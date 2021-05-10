# Spark Structured Streaming
In this blog post I will describe my experience with Spark Structured Streaming

## Difficulties

The main difficultiy I experienced using streams in that the data is not present in the form of files, but needs to be received and stored first (in the case of the assignment). Starting and stopping the process of collecting data manually just does not fit into the conceptual view I developed of Spark.  
Parsing the input stream was quite easy using the regular expression with the premade expression, but completely writing it myself is something that would take much more time as I am not familiar with the notation used, even though I am quite experienced with the concept.  
Unfortunately I was not able to do batch processing.

## Easy steps

Analyzing the data after having the data in the form of dataframes is just as easy as analyzing data that is available staticly.

## Questions in the assignment

The total amount of items sold:
```sql
SELECT count(*) as items FROM runes
```
|items|
|-------| 
|  5595|


The amount of items sold per type
```sql
SELECT tpe, count(*) FROM runes GROUP BY tpe
```

|type|count|
|--|--|
|Sword|416|
|Hasta|449|
|Two-handed sword|447|
|Hatchet|430|
|Dagger|410|
|Halberd|392|
|Warhammer|417|
|Spear|393|
|Longsword|482|
|Mace|440|
|Claw|440|
|Battleaxe|431|
|Scimitar|448|  

the amount of money spent on all types of swords
```sql
SELECT sum(price) FROM runes WHERE tpe = 'Sword' or tpe = 'Two-handed sword' or tpe = 'Longsword'
```
| sum(price) |
|------------|
|16671107|
