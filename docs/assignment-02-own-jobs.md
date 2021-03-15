# My Hadoop Jobs

[part 1](https://rubigdata.github.io/bigdata-blog-2021-joshdev-de/assignment-02-setup) of the assignment

## Preprocess text
When inspecting the results of the WordCount example I noticed that a lot distinct keys are created due to special characters or capitalized letters. In order to tackle that problem I came up with a set of preprocessing functions.

```java
  private static String normalizeString(String s) {
    return s
      .replaceAll("[^a-zA-Z0-9]", " ")
      .toLowerCase();
  }
```
