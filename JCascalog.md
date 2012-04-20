JCascalog is a pure-Java interface to Cascalog that comes bundled with Cascalog as of version 1.8.7. All the functionality available in Cascalog is available via the JCascalog interface (although due to it being Java it's naturally more verbose). Moreso, Cascalog and JCascalog are perfectly interoperable: JCascalog subqueries, operations, and predicate macros can be used in regular Cascalog code and vice-versa.

The goals of JCascalog are three-fold:

1. Make Cascalog an option for a larger audience. The concepts behind Cascalog are not Clojure-specific.
2. For non-Clojure people, make the learning curve for Cascalog less steep. Rather than have to learn a whole new language to learn Cascalog, you can start with JCascalog and then perhaps move to regular Cascalog when you want something more concise.
3. Make Cascalog an easier sell within a larger organization. In a larger organization, Clojure will be a barrier to using Cascalog for many people. But due to the tight integration between JCascalog and Cascalog, Cascalog is now an easier sell. Some people can use Java, others can use Clojure, and the tight integration between them still lets people share operations, subqueries, and routines with each other.

## Basics

Let's take a look at some simple queries. You can see the datasets we'll be querying [here](https://github.com/nathanmarz/cascalog/tree/master/src/clj/cascalog/playground.clj), and all the example code here is available in [this file](https://github.com/nathanmarz/cascalog/tree/master/src/jvm/jcascalog/example/Examples.java). Here's a query that gets all the people that are 25 years old:

```java
Api.execute(
  new StdoutTap(),
  new Subquery(new Fields("?person"),
    new Predicate(Playground.AGE, new Fields("?person", 25))
    ));
```

When you run this query, you'll get this as output:

```
RESULTS
-----------------------
david
emily
-----------------------
```

What this query says is:

1. I want to execute the following "subquery" and put the results into an instance of `StdoutTap`.
2. `StdoutTap` is a type of `cascading.tap.Tap` which is an abstraction for reading and/or writing data. This particular tap just writes tuples written to it into stdout.
3. A `Subquery` represents a set of tuples that is computed by doing computation over other sets of tuples. This `Subquery` says:
  - I emit output tuples containing one field called "?person"
  - I read from the Playground.AGE dataset, binding the first field of the dataset to "?person" and only keeping tuples where the second field is 25
4. A `Subquery` contains an "output fields declaration", the names for the output tuples of the subquery, and then a list of `Predicate` which define and constrain those output variables. Everything you could imagine doing: sourcing data, functions, filtering, aggregations, grouping, secondary sorting... is done via this single `Predicate` abstraction.

Here's a slightly more complicated query that prints all the people less than 30 years old:

```java
Api.execute(
  new StdoutTap(),
  new Subquery(new Fields("?person"),
    new Predicate(Playground.AGE, new Fields("?person", "?age")),
    new Predicate(new LT(), new Fields("?age", 30))
    ));
```

This query works like this:

1. The `Subquery` says "I emit tuples with one field called '?person'"
2. It reads from the AGE dataset and binds the first field to "?person" and the second field to "?age"
3. The second predicate creates a filter over the tuples. It says "Only keep tuples for which the value of '?age' is less than 30"

This query only prints out the names of the people. If you want to emit 2-tuples that contain the age as well, you simply add "?age" to the output fields declaration, like so:

```java
Api.execute(
  new StdoutTap(),
  new Subquery(new Fields("?person", "?age"),
    new Predicate(Playground.AGE, new Fields("?person", "?age")),
    new Predicate(new LT(), new Fields("?age", 30))
    ));
```

This query will print out:

```
RESULTS
-----------------------
david	25
emily	25
kumar	27
alice	28
gary	28
-----------------------
```

So far you've seen how to read tuples and filter them. Let's look at an example of using a function to create new variables in a query. The following query reads from the AGE dataset and emits each person's name, age, and the age value multiplied by 2:

```java
Api.execute(
  new StdoutTap(),
  new Subquery(new Fields("?person", "?age", "?double-age"),
    new Predicate(Playground.AGE, new Fields("?person", "?age")),
    new Predicate(new Multiply(), new Fields("?age", 2), new Fields("?double-age"))
    ));
```

The interesting thing in this query is the second predicate doing the Multiply operation. This predicate says:

1. I execute the `Multiply` function
2. The function takes as input the `?age` variable and the number `2`.
3. The function emits a single output variable called `?double-age`.

Unlike the previous predicates you saw, this predicate takes in a set of input fields and a set of output fields.

There are two kinds of fields you can use in Cascalog:

1. "Vars": These are strings that start with "?", "!", or "!!". A var binds to the output of a predicate and represents all the values for that portion of all tuples emitted by that predicate.
2. "Constants": Constants are everything else. When used as an input field, it will serve as input to that function. When used as an output field, it acts as a filter by filtering out any tuples that did not produce that constant as output.

By defaults, Cascalog auto-distincts the output of queries (that don't contain any aggregators – more on this in a bit). For instance, take a look at the FOLLOWS dataset:

```
["alice" "david"]
["alice" "bob"]
["alice" "emily"]
["bob" "david"]
["bob" "george"]
["bob" "luanne"]
["david" "alice"]
["david" "luanne"]
["emily" "alice"]
["emily" "bob"]
["emily" "george"]
["emily" "gary"]
["george" "gary"]
["harold" "bob"]
["luanne" "harold"]
["luanne" "gary"]
```

Now let's say you want all the values of the first field. If you run this query:

```java
Api.execute(
  new StdoutTap(),
  new Subquery(new Fields("?person"),
    new Predicate(Playground.FOLLOWS, new Fields("?person", "_"))
    ));
```

You get the unique set of values for that first field:

```
RESULTS
-----------------------
alice
bob
david
emily
george
harold
luanne
-----------------------
```

In contrast, if you run this query:

```java
Api.execute(
  new StdoutTap(),
  new Subquery(new Fields("?person"),
    new Predicate(Playground.FOLLOWS, new Fields("?person", "_")),
    Option.distinct(false)
    ));
```

You get a non-distinct result:

```
RESULTS
-----------------------
alice
alice
alice
bob
bob
bob
david
david
emily
emily
emily
emily
george
harold
luanne
luanne
-----------------------
```

The only difference between the queries is the addition of the `Option.distinct(false)` clause. "Option predicates" are a special kind of predicate which tweak various aspects of how a query executes.

Also notice the use of the underscore for binding to the second field of the FOLLOWS dataset. "_" is a special field that will ignore the values for that field.

## Joins

Let's look at an example of how you would do a join between two datasets. Here's a query that finds all the male people that "emily" follows by doing a join over the FOLLOWS and GENDER datasets:

```java
Api.execute(
  new StdoutTap(),
  new Subquery(new Fields("?person"),
    new Predicate(Playground.FOLLOWS, new Fields("emily", "?person")),
    new Predicate(Playground.GENDER, new Fields("?person", "m"))
    ));
```

Joins are implicit in Cascalog. What causes a join is two different data sources sharing a variable name. In this case, the `?person` var is shared between the output of FOLLOWS and GENDER so Cascalog does an inner join between the two datasets.

Also note the use of constants in this query for constraining the output to only the male people that "emily" follows.

## Multi-level queries

So far the queries you've seen have have all been relatively simple. Let's look at a query that's more complicated and requires multiple levels of subqueries.

Here's how you would do a query that emits all follows relationships where each person in the relationship follows more than two people:

```java
Subquery manyFollows =
        new Subquery(
            new Fields("?person"),
            new Predicate(Playground.FOLLOWS, new Fields("?person", "_")),
            new Predicate(new Count(), new Fields("?count")),
            new Predicate(new GT(), new Fields("?count", 2))
            );
Api.execute(
  new StdoutTap(),
  new Subquery(new Fields("?person1", "?person2"),
    new Predicate(manyFollows, new Fields("?person1")),
    new Predicate(manyFollows, new Fields("?person2")),
    new Predicate(Playground.FOLLOWS, new Fields("?person1", "?person2"))
    ));
```

First the subquery "manyFollows" is created that emits all people that follow more than two people. Then, a query is executed which does joins against "manyFollows" to filter the FOLLOWS records.

The key to "multi level" queries is that subqueries and taps work exactly the same way within queries. They are sources of tuples. Cascalog calls a source of tuples a "generator". 

Finally, here's how you could make the last query slightly more concise:

```java
Subquery manyFollows =
        new Subquery(
            new Fields("?person"),
            new Predicate(Playground.FOLLOWS, new Fields("?person", "_")),
            new Predicate(new Count(), new Fields("?count")),
            new Predicate(new GT(), new Fields("?count", 2))
            );
Api.execute(
  new StdoutTap(),
  new Subquery(new Fields("?person1", "?person2"),
    new Predicate(Api.each(manyFollows), new Fields("?person1", "?person2")),
    new Predicate(Playground.FOLLOWS, new Fields("?person1", "?person2"))
    ));
```

Note the `Api.each` call. This creates something called a "predicate macro" that will expand that predicate into the two predicates from the first implementation of this query by applying `manyFollows` to each input var to the predicate. More on predicate macros later.

## Operations

Let's take a look at implementing your own operation. Let's say you want all the unique words from the SENTENCE dataset. To do this, you'll need an operation that takes a sentence and splits it into its constituent words. Here's how you can define that operation:

```java
public class Split extends CascalogFunction {
    public void operate(FlowProcess flowProcess, FunctionCall fnCall) {
        String sentence = fnCall.getArguments().getString(0);
        for(String word: sentence.split(" ")) {
            fnCall.getOutputCollector().add(new Tuple(word));
        }
    }
}
```

This operation grabs the first field of its input as the sentence and then emits for each word a single field containing the word. Here's how you would use this function:

```java
Api.execute(
  new StdoutTap(),
  new Subquery(new Fields("?word"),
    new Predicate(Playground.SENTENCE, new Fields("?sentence")),
    new Predicate(new Split(), new Fields("?sentence"), new Fields("?word"))
    ));
```

Now let's say you want to implement word count over the SENTENCE dataset, the canonical MapReduce query. Here's how you implement it:

```java
Api.execute(
  new StdoutTap(),
  new Subquery(new Fields("?word", "?count"),
    new Predicate(Playground.SENTENCE, new Fields("?sentence")),
    new Predicate(new Split(), new Fields("?sentence"), new Fields("?word")),
    new Predicate(new Count(), new Fields("?count"))
    ));
```

The only difference with the last query is the addition of the "Count" predicate.

You might be wondering "Where is it doing a GROUP-BY in this query?". Grouping is implicit in Cascalog. You can read more about how grouping and aggregation works [here](https://github.com/nathanmarz/cascalog/wiki/How-cascalog-executes-a-query).

There are 6 kinds of operations you can implement in JCascalog:

1. `CascalogFunction`: These operations operate on one tuple at a time and produce 0 or more output tuples. If it produces 0 tuples, the input tuple will be filtered out. If it produces more than 1 tuple, the input fields will be duplicated into multiple tuples.
2. `CascalogFilter`: These operations return true if the input tuple should be kept and false if it should be filtered out.
3. `CascalogAggregator`: A type of aggregator. These work the same as [Cascading aggregators](http://www.cascading.org/javadoc/cascading/operation/Aggregator.html)
4. `CascalogBuffer`: Another type of aggregator that can't be chained with other aggregators. These work the same as [Cascading buffers](http://www.cascading.org/javadoc/cascading/operation/Buffer.html)
5. `ParallelAgg`: A more restricted form of aggregator that will automatically insert combiners on the map-side to speed up the computation.
6. `ClojureOp`: Used for referencing operations defined in Clojure. Takes in a namespace and a var name.

The `jcascalog.op` package contains a useful set of common operations.

## Reading from and writing to files

So far all the queries have only been working in-memory by reading from in-memory sources and writing to stdout. Here's a query that reads all the text files in the "src/jvm/jcascalog/example" directory and emits a single tuple containing the total line count for all files in that directory. The output will be emitted as a textfile to the "/tmp/myresults" directory:

```java
Api.execute(
  Api.hfsTextline("/tmp/myresults"),
  new Subquery(new Fields("?count"),
    new Predicate(Api.hfsTextline("src/jvm/jcascalog/example"), new Fields("_")),
    new Predicate(new Count(), new Fields("?count"))
    ));
```

`Api.hfsTextline` is a helper function to create a [Cascading Tap](http://www.cascading.org/1.2/userguide/html/ch03s03.html) that reads from HDFS using the `TextLine` format. Any Cascading tap can be used in Cascalog queries.

## Api static methods

The bulk of the JCascalog API are static methods in the `jcascalog.Api` class. These methods wrap functionality available in the [api.clj](https://github.com/nathanmarz/cascalog/blob/master/src/clj/cascalog/api.clj) and [ops.clj](https://github.com/nathanmarz/cascalog/blob/master/src/clj/cascalog/ops.clj) files. Look to those files for documentation on the functions available. Some other functions to note:

1. `union` and `combine` are used to concatenate two or more separate sets of data into one set of data. They require that each dataset contain tuples with the same number of fields. `union` will in addition unique the tuples from the sets of data.
2. `firstN`: This functions takes in a subquery, a number of tuples, and optional sorting arguments and returns the "first n" tuples from the set it sees. The implementation is extremely efficient and works across arbitrarily sized datasets.
3. `setApplicationConf`: Takes in a map of `JobConf` parameters that will be automatically used for every executed query from now on.

## Predicate Macros

- [Predicate macro interface](https://github.com/nathanmarz/cascalog/blob/master/src/jvm/jcascalog/PredicateMacro.java)
- TODO: finish
