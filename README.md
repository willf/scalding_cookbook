Scalding Cookbook
=================

[Scalding](https://github.com/twitter/scalding) is a Scala API for 
[Cascading](http://www.cascading.org/), which is a Java API for Hadoop. 
"Hadoop is a distributed system for counting words."

Here are some recipes. If you want to add a recipe, create a pull request, and I'll
likely add it.

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

**Convert TSV file to another TSV file format** Same number of lines, mapping some function over the lines, and
projecting out a certain set of columns. This could be written as map(fn).project(columns), but mapTo is ``more efficient.''

```scala
import com.twitter.scalding._

// input (tsv)
// 0   1     2     3    4   5   6
// 22  kinds of	   love	nn2 io  nn1
// 12  large green eyes	jj  jj  nn2
//
// output (tsv)
// 22 of    kinds/nn2_love/nn1
// 12 green large/jj_eyes/nn2

class contextCountJob(args : Args) extends Job(args) {
	val inSchema = ('count, 'w1 ,'w2, 'w3, 'pos1, 'pos2, 'pos3)
	val outSchema = ('count, 'word, 'context)
  Tsv(args("input"),inSchema)
    .mapTo(inSchema -> outSchema) { parts : (String, String, String, String, String, String, String) => {
    	  val (count, w1, w2, w3, pos1, pos2, pos3) = parts
    		val context = "%s/%s_%s/%s".format(w1,pos1,w3,pos3)
    		(count, w2, context)
    	}
    }
    .write(Tsv(args("output")))
  }
```
**Limit or sample the data**  Sometimes, you want to do limit the data you process (for example, in order to 
do a quick check to see if the output is what you expect. Sometimes, you want to sample a percentage of the data
(for example, to create an evaluation set). Use ```limit``` to limit the data (this is much like Scala's
```take```), or ```sample``` to sample it. 

A couple of notes:
1. You have to run this job using either hds, or hds-local (this is a limit of Cascading, the underlying system)
2. Jobs with small limits will not take long to run, but jobs with a sample size set will take as long as processing the entire set
3. Limit takes an integer (the number of items to take)
4. Sample takes a double between 0.0 and 1.0, which is the percentage to take. If the number is greater than 1.0, it's treated as 100%
5. Sample is currently only in the develop branch of Scalding

***Limit*** example
```scala
import com.twitter.scalding._

class WordCountJob(args : Args) extends Job(args) {
  TextLine(args("input"))
    .limit(args("limit").toInt)
    .flatMap('line -> 'word) { line : String => line.split("\\s+") }
    .groupBy('word) { _.size }
    .write(Tsv(args("output")))
}
```

***Sample*** example
```scala
import com.twitter.scalding._

class WordCountJob(args : Args) extends Job(args) {
  TextLine(args("input"))
    .sample(args("sample").toDouble)
    .flatMap('line -> 'word) { line : String => line.split("\\s+") }
    .groupBy('word) { _.size }
    .write(Tsv(args("output")))
}
```
