# Exploring Steam Game Data
In this blog post I will explore [this](https://data.world/craigkelly/steam-game-data) dataset about steam games.

## The question to answer
I want to find out whether there is a relation between the release date of a game and its compatability with different operating systems. The dataset contains the release date and the information whether the game runs on Windows, macOS and Linux. Hence the data is suitable and I can continue.

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

+-------+--------------------+-----------------+
|summary|                name|          release|
+-------+--------------------+-----------------+
|  count|               13357|            13357|
|   mean|  3333996.3333333335|1975.145017593771|
| stddev|   5772928.580294436|278.9970496372111|
|    min|! That Bastard Is...|               -1|
|    max|zTime (Danger Noo...|             2019|
+-------+--------------------+-----------------+
```

[lazy-eval]: https://github.com/rubigdata/bigdata-blog-2021-joshdev-de/raw/master/docs/images/lazy_eval.PNG "Lazy Evaluation"
[uncached]: https://github.com/rubigdata/bigdata-blog-2021-joshdev-de/raw/master/docs/images/uncached.png "Uncached"
[cached]: https://github.com/rubigdata/bigdata-blog-2021-joshdev-de/raw/master/docs/images/cached.png "Cached"
