Scalding Cookbook
=================

[Scalding](https://github.com/twitter/scalding) is a Scala API for 
[Cascading](http://www.cascading.org/), which is a Java API for Hadoop. 
"Hadoop is a distributed system for counting words."

Here are some recipes.

**Word Count** Splits on one or more whitespace

```scala
import com.twitter.scalding._

class WordCountJob(args : Args) extends Job(args) {
  TextLine(args("input"))
    .flatMap('line -> 'word) { line : String => line.split("\\s+") }
    .groupBy('word) { _.size }
    .write(Tsv(args("output")))
}
```
**Word Count, specialized tokenizer**
From the [Scalding intro](https://github.com/twitter/scalding/blob/develop/README.md)

```scala
import com.twitter.scalding._

class WordCountJob(args : Args) extends Job(args) {
  TextLine( args("input") )
    .flatMap('line -> 'word) { line : String => tokenize(line) }
    .groupBy('word) { _.size }
    .write( Tsv( args("output") ) )

  def tokenize(text : String) : Array[String] = {
    // Lowercase each word and remove punctuation.
    text.toLowerCase.replaceAll("[^a-zA-Z0-9\\s]", "").split("\\s+")
  }
}
```
