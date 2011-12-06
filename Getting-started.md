# Make sure you have java 1.6
# export JAVA_OPTS=-Xmx768m

Cascalog is hosted at the Clojars maven repo "here":http://clojars.org/cascalog

h2. Learning Cascalog

Read the tutorials and follow along in your REPL. Experiment with Cascalog's playground dataset. The tutorials can be found "here":http://nathanmarz.com/blog/introducing-cascalog-a-clojure-based-query-language-for-hado.html and "here":http://nathanmarz.com/blog/new-cascalog-features-outer-joins-combiners-sorting-and-more.html.

Nathan's "tech talk at LinkedIn":http://sna-projects.com/blog/2010/11/clojure-at-backtype/ goes through an in-depth example of using Cascalog to perform a complex query on real-world data. Watching this talk in full is highly recommended.

The Cascalog api is defined in the [[url:https://github.com/nathanmarz/cascalog/blob/master/src/clj/cascalog/api.clj|api.clj]] and [[url:https://github.com/nathanmarz/cascalog/blob/master/src/clj/cascalog/ops.clj|ops.clj]] source code files. The files are pretty short , and it's recommended that you read through those files and familiarize yourself with the API.

After you've gone through the tutorials, read through the documentation on this wiki.

h2. Local mode

Cascalog can be run from the REPL on your local machine. In this case Hadoop runs in "local mode" which just means it's completely in process. This is useful for experimentation and for doing local analysis with small datasets.

Cascalog comes with some "playground" datasets which are useful for learning how to use the tool. These datasets are used in the introductory tutorials and you can see them by looking at the playground.clj file in the Cascalog source.

h2. Running Cascalog queries on a Hadoop cluster

See "this tutorial":http://nathanmarz.com/blog/news-feed-in-38-lines-of-code-using-cascalog.html for information about developing and running a Cascalog query on a cluster.