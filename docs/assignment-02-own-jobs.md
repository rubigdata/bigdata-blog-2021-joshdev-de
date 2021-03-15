# My Hadoop Jobs

[part 1](https://rubigdata.github.io/bigdata-blog-2021-joshdev-de/assignment-02-setup) of the assignment

## Preprocess text
When inspecting the results of the WordCount example I noticed that a lot distinct keys are created due to special characters or capitalized letters. In order to tackle that problem I came up with a set of preprocessing functions that remove the special characters and set all letters to lower case.

```java
private static String normalizeString(String s) {
  return s
    .replaceAll("[^a-zA-Z0-9]", " ")
    .toLowerCase();
}
```
Using the new normalizing step the word count presented in the ouptut is much more expressive.

## Fun Facts

Using the following mapper I can count the lines and letters, while also answering the question whether Romeo or Juliet appears more often in the script.

```java
  public static class TokenizerMapper
       extends Mapper<Object, Text, Text, IntWritable>{

    private final static IntWritable one = new IntWritable(1);
    private IntWritable count = new IntWritable();
    private final static Text romeo = new Text("Romeo appearances");
    private final static Text juliet = new Text("Juliet appearances");
    private final static Text lines = new Text("lines");
    private final static Text letters = new Text("letters");

    private static String normalizeString(String s) {
      return s
        .replaceAll("[^a-zA-Z0-9]", " ")
        .toLowerCase();
    }

    private static int countLetters(String s) {
      return s
        .replaceAll("\\s+","")
        .length();
    }

    public void map(Object key, Text value, Context context
                    ) throws IOException, InterruptedException {
      context.write(lines, one);

      String line = normalizeString(value.toString());
      
      count.set(countLetters(line));
      context.write(letters, count);

      StringTokenizer itr = new StringTokenizer(line);
      while (itr.hasMoreTokens()) {
        switch (itr.nextToken()) {
          case "juliet": context.write(juliet, one); break;
          case "romeo" : context.write(romeo, one); break;
          default: break;
        }
      }
    }
  }
```
