# Exploring Steam Game Data
In this blog post I will explore [this](https://data.world/craigkelly/steam-game-data) dataset about steam games.

## The question to answer
I want to find out whether there is a relation between the release date of a game and its compatability with different operating systems. The dataset contains the release date and the information whether the game runs on Windows, macOS and Linux. Hence the data is suitable and I can continue.

## Preparing the data
After importing the CSV file to spark all the fields are strings by default, but I need them to be and an integer for the release year and booleans for the fact whether they are compatible with the operating systems. In order to archive that I use some selfmade conversion functions. Converting strings to booleans is rather easy, but does not happen automatically so the function used for that is:  
```Scala
val tBoolean = udf((f: String) => f.toBoolean)
```



[lazy-eval]: https://github.com/rubigdata/bigdata-blog-2021-joshdev-de/raw/master/docs/images/lazy_eval.PNG "Lazy Evaluation"
[uncached]: https://github.com/rubigdata/bigdata-blog-2021-joshdev-de/raw/master/docs/images/uncached.png "Uncached"
[cached]: https://github.com/rubigdata/bigdata-blog-2021-joshdev-de/raw/master/docs/images/cached.png "Cached"
