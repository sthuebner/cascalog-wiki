# Cascalog

Cascalog is a fully-featured data processing and querying library for Clojure. The main use cases for Cascalog are processing "Big Data" on top of Hadoop or doing analysis on your local computer from the Clojure REPL. Cascalog is a replacement for tools like Pig, Hive, and Cascading.

Cascalog operates at a significantly higher level of abstraction than a tool like SQL. More importantly, its tight integration with Clojure gives you the power to use abstraction and composition techniques with your data processing code just like you would with any other code. It's this latter point that sets Cascalog far above any other tool in terms of expressive power.

Follow the getting started steps, check out the tutorials, and you'll be running Cascalog queries on your local computer within 5 minutes.

Cascalog is hosted on Github at [[http://github.com/nathanmarz/cascalog]].

## Getting help

- [Cascalog Google Group](http://groups.google.com/group/cascalog-user)
- `#cascalog` room on Freenode

## Documentation

- [[Getting Started]]
- [[Defining and executing queries]]
- [[How Cascalog executes a query]]
- [[Guide to custom operations]]
- [[Predicate operators]]
- [[Methods for handling wide sources]]
- [[Joins in Cascalog]]

## Articles around the web

- [Introductory Tutorial, Part 1](http://nathanmarz.com/blog/introducing-cascalog-a-clojure-based-query-language-for-hado.html)
- [Introductory Tutorial, Part 2](http://nathanmarz.com/blog/new-cascalog-features-outer-joins-combiners-sorting-and-more.html)
- [Developing and deploying a Cascalog query on a Hadoop cluster](http://nathanmarz.com/blog/news-feed-in-38-lines-of-code-using-cascalog.html)
- [Why Yieldbot chose Cascalog over Pig for Hadoop processing](http://tech.backtype.com/52456836)
- [Summarizing next-gen sequencing variation statistics with Hadoop using Cascalog](http://bcbio.wordpress.com/2011/07/04/summarizing-next-gen-sequencing-variation-statistics-with-hadoop-using-cascalog/)
- [Which operation def macro should I use in Cascalog?](http://entxtech.blogspot.com/2010/12/which-operation-def-macro-should-i-use.html)
- [Predicate macros](http://groups.google.com/group/cascalog-user/browse_thread/thread/33f9b69bf18c9bdc)
- [Cascalog made easier](http://jimdrannbauer.com/2011/02/04/cascalog-made-easier/)
- [Generator as filter / negations](http://groups.google.com/group/cascalog-user/browse_thread/thread/17bbe772159b8ffa)
- [Catching errors with traps](http://groups.google.com/group/cascalog-user/browse_thread/thread/f9257bf8002e053a)
- [Testing Cascalog with Midje](http://sritchie.github.com/2011/09/30/testing-cascalog-with-midje.html)

## Documentation Todo

- [[Option predicates]]
- [[Predicate macros]]
- [[Building queries dynamically]]
- [[Global sorting]]
- [[Built-in operations and query builders]]
- [[Negations and generators-as-sets]]
- [[Tuple Serialization]]

## Other

- [Who's using Cascalog](https://www.assembla.com/spaces/cascalog/wiki/Who's_using_Cascalog)
