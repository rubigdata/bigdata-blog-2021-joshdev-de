# Exploring a web crawl
In this blog post I will describe how to process web crawl data.  
I describe the process and problems that occured.
In the last section I give some ressources which explain the process of getting the scala program to run on a cluster.

## The question to answer
Crawl data holds a lot of information, so the choice of a question was not easy.  
I decided to answer on of the most crucial questions:  
**Xbox or Playstation?!**  
In order to answer this question I simply count how often the words xbox and playstation occur in the html pages stored in the web crawl.  
Later I also decided to count the amount of webpages each word occurs on.

## First steps, preparing the data
The first step is to find a way to make spark handle warc files. Warc is the format in which web crawls are stored nowadays and they hold metadata about the web query and the answer as well as the answer itself. I use a premade library and some spark magic of define a spark session with a custom configuration and data frame.

```scala
import org.apache.spark.SparkConf
import org.apache.spark.sql.SparkSession

import org.apache.hadoop.io.NullWritable

import de.l3s.concatgz.io.warc.{WarcGzInputFormat,WarcWritable}
import de.l3s.concatgz.data.WarcRecord

val sparkConf = new SparkConf()
                          .setAppName("Relevance Counter")
                          .set("spark.memory.offHeap.enabled", "true")  // allow heap data to be stored outside the JVM memory segment
                          .set("spark.memory.offHeap.size", "8g")       // specifies size of off heap space
                          .set("spark.serializer", "org.apache.spark.serializer.KryoSerializer")
                          .registerKryoClasses(Array(classOf[WarcRecord]))

implicit val sparkSession = SparkSession.builder().config(sparkConf).getOrCreate()
val sc = sparkSession.sparkContext

val warcs = sc.newAPIHadoopFile(
              warcLocation,
              classOf[WarcGzInputFormat],             // InputFormat
              classOf[NullWritable],                  // Key
              classOf[WarcWritable]                   // Value
    )
```





For my question I am only interested in the webpages itself so the only thing I care about are responses which hold html files. Another important step is to filter out any invalid entries to not get undefined behaviour. The steps of preparing the data can be seen here:

```scala
val plainHTML = warcs.map { wr => wr._2}
            .filter{ _.isValid() }
            .map{ _.getRecord() }
            .filter{ _.getHeader.getHeaderValue("WARC-Type") == "response" }
            .filter{ _.getHttpMimeType() == "text/html" }
            .map{ _.getHttpStringBody() }
```

Now that I have a data frame of plain html pages I can count the words "xbox" and "playstation" easily right?  
No it is not easy and you will read why in the following section.

## Counting words efficiently is hard!
My first attempt on counting the words was like the following:
I parse the HTML page to only get the visible text and put that to lower case. Then I split the text by spacebars to create an array of words. the last step is to filter the array to only keep the words that contain either "xbox" or "playstation" and then count the remaining words. In the end all the word counts in the different texts are summed up.
```scala
val parsed = plainHTML.map{ wr => Jsoup.parse(wr).text().toLowerCase().split(" ")}
val xboxCount = parsed.map{ _.filter( x => x.contains("xbox")).length }.sum()
val playstationCount = parsed.map{ _.filter( x => x.contains("playstation")).length }.sum()
```
The strategy works well on small amounts of data, but when querying even a fraction of a full web crawl spark runs out of memory.  
In order to solve this the first step is to make the word counting more effiecent with the help of regular expressions like this:
Using `.r` the string is turned into a regular expression which can then be found in the text. The size of the resulting MatchIterator just gives the amount of found matches.
```scala
val xboxCount = plainHTML.map{ text => "xbox".r.findAllIn(text).size }.sum()
```
But that improvement brings no success and I am greeted with the following log entry again even though special memory settings are applied to give spark more memory space to work with:  
![oom-pic]  
The next step is to eliminate the step of parsing the HTML page and work with the plain file:
```scala
val parsed = plainHTML.map{ _.toLowerCase()}
```
As I learn quickly even the step of putting the text to lower case is too memory intensive. Luckily regular expressions can be made case insensitive like this:
```scala
val xboxNum = plainHTML.map{ text => "(?i)xbox".r.findAllIn(text).size }.sum()
val playstationNum = plainHTML.map{ text => "(?i)playstation".r.findAllIn(text).size }.sum()
```

Like this two passes over the data are needed and that seems inefficient so I merge the two queries into one like this:
Instead of creating two values I create a touple and reduce the touples by summing the first and second entries individually.
```scala
val matches = plainHTML.map{ text => ("(?i)xbox".r.findAllIn(text).size, "(?i)playstation".r.findAllIn(text).size )}
                .reduce({case((x1, p1), (x2, p2)) => (x1 + x2, p1 + p2)})
```
Finally! This program runs on a large fraction of the web crawl and gives me the following result:
```
Count of xbox: 		     27064534
Count of playstation:  2686489
```

## Convenience features
Now that the program is finally running it is time for some improvments!  
The first one is to make the warc location and searched words which are hardcoded at the moment arguments of the program.  
The second one is to count the number of pages each word occurs on. The program now looks like this (ready for you to copy):  
I use clever pattern matching to introduce the second improvment and simple built in functionality to realize the parameterization.
```scala
import org.apache.spark.SparkConf
import org.apache.spark.sql.SparkSession

import org.apache.hadoop.io.NullWritable

import de.l3s.concatgz.io.warc.{WarcGzInputFormat,WarcWritable}
import de.l3s.concatgz.data.WarcRecord


object RUBigDataApp {
  def main(args: Array[String]) {

    val warcLocation = args(0)
    val word1 = args(1)
    val word2 = args(2)

    val sparkConf = new SparkConf()
                          .setAppName("Relevance Counter")
                          .set("spark.memory.offHeap.enabled", "true")  // allow heap data to be stored outside the JVM memory segment
                          .set("spark.memory.offHeap.size", "8g")       // specifies size of off heap space
                          .set("spark.serializer", "org.apache.spark.serializer.KryoSerializer")
                          .registerKryoClasses(Array(classOf[WarcRecord]))

    implicit val sparkSession = SparkSession.builder().config(sparkConf).getOrCreate()
    val sc = sparkSession.sparkContext

    val warcs = sc.newAPIHadoopFile(
                  warcLocation,
                  classOf[WarcGzInputFormat],             // InputFormat
                  classOf[NullWritable],                  // Key
                  classOf[WarcWritable]                   // Value
        )

    val plainHTML = warcs.map { wr => wr._2}
                .filter{ _.isValid() }
                .map{ _.getRecord() }
                .filter{ _.getHeader.getHeaderValue("WARC-Type") == "response" }
                .filter{ _.getHttpMimeType() == "text/html" }
                .map{ _.getHttpStringBody() }

    val matches = plainHTML.map{ text => (s"(?i)$word1".r.findAllIn(text).size, s"(?i)$word2".r.findAllIn(text).size )}
                .map({
                    case(0, 0) => (0, 0, 0, 0)
                    case(0, p) => (0, p, 0, 1)
                    case(x, 0) => (x, 0, 1, 0)
                    case(x, p) => (x, p, 1, 1)
                })
                .reduce({ case((x1, p1, xo1, po1), (x2, p2, xo2, po2)) => (x1 + x2, p1 + p2, xo1 + xo2, po1 + po2) })

    println(s"Count of $word1: \t" + matches._1)
    println(s"$word1 occured on pages: \t" + matches._3)
    println(s"Count of $word2: \t" + matches._2)
    println(s"$word2 occured on pages: \t" + matches._4)

    sparkSession.stop()
  }
}
```

## Running a (scala-) spark application on a cluster
To be honest the default container given to me for the purpose of this project was enough to compile the program.  
For everyone which is not part of this course the following ressources are helpful to find out about the process:  
 * [creating standalone spark app](http://spark.apache.org/docs/3.1.1/quick-start.html#self-contained-applications)
 * [using sbt to compile scala apps](https://docs.scala-lang.org/getting-started/sbt-track/getting-started-with-scala-and-sbt-on-the-command-line.html)
 * [sbt-assembly for dependecy management](https://github.com/sbt/sbt-assembly)

You then need some spark cluster setup (local or remote) to submit the application.  
[Here](http://spark.apache.org/docs/3.1.1/submitting-applications.html)  you find infomation on that.

## Final answer
Just to keep the excitement some more here are some screenshots of my final program running on the cluster.  
Here you can see how the application itself is held in HDFS.<br><br>
![namenode-fs]  
<br>
**Isn't it beautiful? :)** <br>
Here you see it in the queue dashboard using the ressources (of course in the gold queue, only the best for my app): <br><br>
![yarn-queue]  
<br>

## Thank you!





[oom-pic]: https://github.com/rubigdata/bigdata-blog-2021-joshdev-de/raw/master/docs/images/oom-pic.png "oom-pic"
[namenode-fs]: https://github.com/rubigdata/bigdata-blog-2021-joshdev-de/raw/master/docs/images/namenode-fs.png "stored-app"
[yarn-queue]: https://github.com/rubigdata/bigdata-blog-2021-joshdev-de/raw/master/docs/images/yarn-queue.png "yarn queue"

