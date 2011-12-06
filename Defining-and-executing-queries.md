Cascalog includes a number of options for the creation and execution of its queries.

## <- ##

`<-` is the query creation operator. As described in [[How Cascalog executes a query]], `<-` accepts a vector of output variables and a series of predicates. The various predicates allowed are discussed in [[Guide to custom operations]].

For example, given this dataset:

```clojure
    (def people [["ben" 35]
                 ["jerry" 41]])
```

The following query filters out all people in the dataset under 40:

```clojure
    (<- [?name ?age]
        (people ?name ?age)
        (< ?age 40))
```

This query does nothing on its own. It can act as a generator for other queries, as described [[Guide to custom operations|here]], or we can bind it to an output tap with the query execution operator.

## ?- ##

`?-` is the query execution operator. It takes a sequence of `<output tap, query>` pairs, and executes all supplied queries in parallel. For example:

```clojure
    (?- (stdout)
        (<- [?name ?age]
            (people ?name ?age)
            (< ?age 40)))
```

Prints the following:

    RESULTS
    -----------------------
    ben	35
    -----------------------

`(stdout)` is a Cascading tap that prints the string representation of output tuples to the output stream.

## ?<- ##

 `?<-` allows for combined query creation and execution. It accepts a single output tap, a result vector, and a series of predicates, like so:

```clojure
    (?<- (stdout)
         [?name ?age]
         (people ?name ?age)
         (< ?age 40))
```

This query is functionally equivalent to the previous example, under `?-`.

## ??- ##

`??-` accepts any numbers of queries (defined by `<-`), executes them in parallel, and returns a sequence of sequences of the results of each query's execution.

That last bit can be a bit confusing. Let's look at two examples. Here's the result of executing a single query with `??-`:

```clojure
    (def results
       (??- (<- [?name ?age]
                (people ?name ?age)
                (< ?age 40))))

    user=> results
    ((["ben" 35]))
```

Notice that we have an outer sequence with a single entry, corresponding to the single subquery we executed. This entry is a sequence containing that query's output tuples. Had we executed two queries, we would see a sequence with two inner sequences: 

```clojure
    (def multi-results
      (??- (<- [?name ?age]
               (people ?name ?age)
               (< ?age 40))
           (<- [?name ?age]
               (people ?name ?age)
               (< ?age 50))))

    user=> multi-results
    ((["ben" 35]) (["ben" 35] ["jerry" 41]))
```

## ??<- ##

`??<-` allows for combined query creation and execution into a clojure sequence. Because `??<-` only allows execution of a single query, it returns a single sequence of tuples:

```clojure
    (def results-??<-
      (??<- [?name ?age]
            (people ?name ?age)
            (< ?age 50)))

    user=> results-??<-
    (["ben" 35] ["jerry" 41])
```
