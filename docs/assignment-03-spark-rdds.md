# Spark RDDs
In the following paragraphs I will note down the most important insights I gained while working with Spark RDDs.

## Lazy evaluation
Or in other words: Work is only done if and at the time a result is requested. This goes so far that even reading data from files is done at the moment a result is requested.   A quick example:  
I create a file with content "Original", then I define the RDD with the text file as the input. As a next step I change the file to contain "Changed". If I know print the RDD the naive expectation would be to read "Original" now, but in fact I read "Changed" due to the lazy nature of Spark.
![lazy-eval]


## Caching RRDs
Spark does not recompute everything everytime, if intermediate results were computed already they are used for the next computation. However this behaviour is not consistent, in order to enforce it I apply ".cache()" to the RDD. The first picture shows the first request and the second one shows a second request which can reuse the already mapped RDD.  
![uncached]
![cached]

## Effect of shuffling and partitions
Shuffling describes the process of shipping data from one node to another one. Shipping data around is always more expensive than just having it at hand.  
If I have an RDD of key value pairs for example and divide it into 4 partitions using the standard partitioner. If I then want to count the occurence of a special key Spark has no information about where those keys are located so it searches in all 4 partitions.  
Fortunately there is a way to let Spark know in which partitions a special key resides. The method used are specialized partitioners. One example is the HashPartitioner which can put all keys with the same hash into one partition. If I know want to count the occurence of a special key, Spark knows where to look for the key. It just computes the hash and searches in the RDD that contains all the keys having that hash.  
But what happens if I modify the RDD now? If I just do a map Spark cannot be sure that the keys are not changed and defaults to the standard partitioner. The knowledge about keys Spark had is lost. So I do a mapValue instead to make sure and tell Spark that the only things changed are the values and the keys remain the same. In this case Spark carries the chosen partitioner on the resulting RDD. 

## How to avoid shuffling and optimize
I could not really test how shuffling impacts performance, but it should be avoided by choosing the right partitioner, like a HashPartitioner.
In order to manage partitions and increase efficiency by choosing the right amount of partitions there are functions to help. ".repartition(n)" is there to partition the RDD into n partitions. ".coalesce(n)" on the other hand merges every n partitions into one partition.

[lazy-eval]: https://github.com/rubigdata/bigdata-blog-2021-joshdev-de/raw/master/docs/images/lazy_eval.PNG "Lazy Evaluation"
[uncached]: https://github.com/rubigdata/bigdata-blog-2021-joshdev-de/raw/master/docs/images/uncached.png "Uncached"
[cached]: https://github.com/rubigdata/bigdata-blog-2021-joshdev-de/raw/master/docs/images/cached.png "Cached"
