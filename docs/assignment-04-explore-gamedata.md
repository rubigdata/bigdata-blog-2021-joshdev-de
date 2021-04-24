# Exploring Steam Game Data
In this blog post I will explore [this](https://data.world/craigkelly/steam-game-data) dataset about steam games.

## The question to answer
I want to find out whether there is a relation between the release date of a game on Steam and its compatability with different operating systems. The dataset contains the release date and the information whether the game runs on Windows, macOS and Linux. Hence the data is suitable and I can continue.

## Preparing the data
After importing the CSV file to spark all the fields are strings by default, but I need them to be and an integer for the release year and booleans for the fact whether they are compatible with the operating systems. In order to archive that I use some selfmade conversion functions. Converting strings to booleans is rather easy, but does not happen automatically so the function used for that is:  
```Scala
val tBoolean = udf((f: String) => f.toBoolean)
```
Getting the release year from the date string is a little more complicated though. The provided format is in the form "6 Nov 2004" but there are also cases like "~2007" or "To be announced" so my approach is to take the last for characters of the string and try to cast them to integer. In case of failure the value -1 is assigned, which allows me to remove those lines afterwards.
```Scala
def toInt(s: String): Int = {
  try {
    s.toInt
  } catch {
    case e: Exception => -1
  }
}

val getYear = udf((f: String) => toInt(f.takeRight(4)))
```
That conversion functions allow me to get a clean dataset with typed columns
```Scala
case class OS(name:String, release:Int, windows:Boolean, mac:Boolean, linux:Boolean)

val osDF = gamedata.select($"ResponseName" as "name",
                           getYear($"ReleaseDate") as "release",
                           tBoolean($"PlatformWindows") as "windows",
                           tBoolean($"PlatformMac") as "mac",
                           tBoolean($"PlatformLinux") as "linux")
                           .as[OS].cache()
```
## Answering the question
When I now look at the averages I notice that all my boolean columns are missing in the summary.
```
osDF.describe().show()
```

|summary|                name|          release|  
|-------|--------------------|-----------------|  
|  count|               13357|            13357|  
|   mean|  3333996.3333333335|1975.145017593771|  
| stddev|   5772928.580294436|278.9970496372111|  
|    min|! That Bastard Is...|               -1|  
|    max|zTime (Danger Noo...|             2019|  

So using the values as booleans might not be the best decision here. In order to get easy results numeric values are much easier, so a float will help here.
The following functions do the job for me.
```Scala
val tFloat = udf((f: Boolean) => if (f) 1 else 0)

case class osNumeric(name:String, release:Int, windows:Int, mac:Int, linux:Int)

val osNum = gamedata.select($"ResponseName" as "name",
                           getYear($"ReleaseDate") as "release",
                           tFloat(tBoolean($"PlatformWindows")) as "windows",
                           tFloat(tBoolean($"PlatformMac")) as "mac",
                           tFloat(tBoolean($"PlatformLinux")) as "linux")
                           .as[osNumeric].filter("release != -1").cache()
``` 
Now the summary looks very promising and I can already tell that the average game was released in 2014 and the overall propability of a game in my cleaned dataset being compatible with Windows is 99.98%, with macOS is 34.29% and with Linux is 23.07%.
```
osNum.describe().show()
```

|summary|                name|           release|             windows|                mac|              linux|
|-------|--------------------|------------------|--------------------|-------------------|-------------------|
|  count|               13096|             13096|               13096|              13096|              13096|
|   mean|  3333996.3333333335|2014.5290928527795|  0.9998472816127062|0.34285277947464876|0.23068112400733048|
| stddev|   5772928.580294436| 2.231519764634232|0.012357456249179848|0.47468089964107957| 0.4212848149792494|
|    min|! That Bastard Is...|              1997|                   0|                  0|                  0|
|    max|zTime (Danger Noo...|              2019|                   1|                  1|                  1|


From the information about the dataset I know that it was collected in December of 2016, so I do not want to look at years further than 2017, because announced games become fewer after 2017. I also omit years before 2005, because there are not many games released before that and my statistic should not be obfuscated by those outliers. But finally here is the graph for the cleaned and limited dataset.
```SQL
select release, count(release) as games, avg(windows), avg(mac), avg(linux)
from osNum
where release > 2005 and release < 2018
group by release
order by release
```
![graph]
## The answer
So what does the graph tell me? Firstly almost every game is compatible with Windows, even though the blue line is barely visible at the very top of the diagram it is there and only very, very slightly dips down in 2012. The next fact clearly visible is that macOS compatability is better than Linux compatability in every year, but they seem to be related. Whenever compatability for macOS rises or falls Linux follows, with one exception. From 2014 to 2015 macOS compatability falls while Linux' rises. It is also very clear that compatability was very bad starting in 2006 and slowly rose until 2013, which was the peak. It then dropped to about 30% for macOS and 20% for Linux in 2016. After that it rises again. <br> <br>
**The answer is yes, there seems to be a strong relation between the release year of a game on Steam and its compatability with different operation systems!**<br> <br>  
Some aspects that lower my success: The timeframe is very limited and the data was not confirmed by my research or Steam. 

## Fun fact
From the moment I have seen that the compatability with Windows is not 100%, I want to know which game is not compatible with Windows.
```SQL
select name from osNum where windows = 0
```
|name |
|---------------------------|
|Call of Duty: Black Ops - Mac Edition|
|WeaponizedChess|


[graph]: https://github.com/rubigdata/bigdata-blog-2021-joshdev-de/raw/master/docs/images/game-compatability.png "graph"
