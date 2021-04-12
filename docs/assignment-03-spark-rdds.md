# Spark RDDs
In the following paragraphs I will note down the most important insights I gained while working with Spark RDDs

## Lazy evaluation
Or in other words: Work is only done if and at the time a result is requested. This goes so far that even reading data from files is done at the moment a result is requested.   A quick example:  
I create a file with content "Original", then I define the RDD with the text file as the input. As a next step I change the file to contain "Changed". If I know print the RDD the naive expectation would be to read "Original" now, but in fact I read "Changed" due to the lazy nature of Spark.


## Caching RRDs

## Effect of shuffling and partitions

## How to avoid shuffling

![namenode-ui]


[namenode-ui]: images/lazy_eval.PNG "Namenode UI"
