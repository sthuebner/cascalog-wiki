A Cascalog query is defined by a list of output vars and a list of predicates that define and constrain the output vars. Think of these predicates as "facts" about the data; As such, the ordering of the predicates doesn't matter. (See [[Guide to custom operations]] for descriptions of the various types of predicates available.)

Let's use the following query as an example. This query gets all the words from the sentence dataset that occur more than 5 times:

```clojure
    (<- [?word]
         (sentence ?sentence)
         (split ?sentence :> ?word)
         (c/count ?count)
         (> ?count 5))
```

Every Cascalog query is executed in 3 steps: 

1. Pre-aggregation 
2. Aggregation 
3. Post-aggregation 

In step 1, Cascalog joins all the generators together while applying as many functions and filters as it can in the process. The only functions/filters that can't be applied are those dependent on the results of an aggregator.

In our example, Cascalog starts with the generator `sentence`. Since all the input vars are satisfied for `split`, it applies the split operation which satisfies another var: `?word`. At this point, Cascalog can't apply the `(> ?count 5)` predicate because the `?count` var hasn't yet been satisfied. As no other operations can be applied, Cascalog then moves into the aggregation phase.

In step 2, Cascalog partitions the tuples by any output variables that have already been satisfied. If no variables are satisfied yet, Cascalog does a single global partition containing all tuples. Cascalog then executes the aggregators of the query for each partition.

In this example, because the `?word` output variable has already been satisfied in step 1, Cascalog executes the count aggregator to create a `?count` var for every value of `?word`.
 
In step 3, Cascalog executes the remaining filters/functions which are dependent on aggregator output. 

In this example, the `?count` variable is now satisfied, so the `(> ?count 5)` predicate can now be applied.
