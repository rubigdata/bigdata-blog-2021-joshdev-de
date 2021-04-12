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
Unfortunately I could not test the effects of shuffling and partitions, because no matter how hard I tried I could not get Spark to NOT be efficient.
I tried using ".map()" instead of ".mapValues()" but Spark carried the partitioner on to the next RDD anyway.

## How to avoid shuffling
In order to manage partitions on the other hand there are the functions "repartition(n)" to partition the RDD into n partitions. "coalesce(n)" on the other hand merges every n partitions into one partition.

[lazy-eval]: images/lazy_eval.PNG "Lazy Evaluation"partitioner functions and coalesse
[uncached]: images/uncached.png "Uncached"
[cached]:images/cached.png "Cached"
