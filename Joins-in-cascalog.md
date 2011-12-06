Cascalog joins datasets that share variable names; this turns out to be quite a concise way to express complex relationships between datasets. Let's start our discussion by defining `age` and `gender`, two datasets that link customer names to these attributes. We'll use these datasets to discuss inner and outer joins.

```clojure
    (def age [["alice" 28]
              ["bob" 33]
              ["chris" 40]
              ["david" 25]
              ["emily" 25]
              ["george" 31]
              ["gary" 28]
              ["kumar" 27]
              ["luanne" 36]])
 
    (def gender [["alice" "f"]
                 ["bob" "m"]
                 ["chris" "m"]
                 ["david" "m"]
                 ["emily" "f"]
                 ["george" "m"]
                 ["gary" "m"]
                 ["harold" "m"]
                 ["luanne" "f"]])
```

## Inner Join ##

An inner join between two datasets returns all tuples that satisfy the "join condition"; in Cascalog, join conditions are specified through shared variable names. 

An example query that triggers an inner join between `age` and `gender` through a shared `?name` variable would be

    (?<- (stdout)
         [?name ?age ?gender]
         (age ?name ?age)
         (gender ?name ?gender))

Executing this query produces these results:

      RESULTS
      -----------------------
      emily   25	   f
      david	  25	   m
      alice	  28	   f
      gary	  28	   m
      george  31	   m
      bob	  33	   m
      luanne  36	   f
      chris	  40	   m
      -----------------------

The inner join drops Harold's and Kumar's records from the result set, as they each weren't present in both source datasets.

Here's a more interesting query that generates all female customers over the age of 25:

    (?<- (stdout)
         [?name ?age]
         (age ?name ?age)
         (gender ?name "f")
         (> ?age 25))

This query joins on the `?name` field, filter `?age` explicitly with clojure's `>` operator, and perform an implicit filtering constraint on gender by constraining the `?gender` var to the static value of `"f"`. (Powerful stuff!) This query produces

      RESULTS
      -----------------------
      emily	  25
      alice	  28
      luanne  36
      -----------------------

## Outer Joins ##

The inner join discussed above will drop tuples that don't exist in all datasets. To keep all data, Cascalog allows you to perform outer joins by using "ungrounding variables", prefixed with `!!`. In the following query, `!!age` and `!!gender` are ungrounding variables:

    (?<- (stdout)
         [?person !!age !!gender]
         (age ?person !!age)
         (gender ?person !!gender))

Executing this query generates the following results:

      -----------------------
      kumar	   27 	null
      alice	   28 	f
      emily	   25 	f
      luanne	36 	f
      bob	   33 	m
      chris	   40 	m
      david	   25 	m
      gary	   28 	m
      george	31 	m
      harold	null	m
      -----------------------

A predicate that contains an ungrounding variable is called an "unground predicate", and a predicate that does not contain an ungrounding variable is called a "ground predicate". Joining together two unground predicates results in a full outer join, while joining a ground predicate to an unground predicate results in a left join.

Let's introduce `person` (our customer list) and `follows` (a dataset of folks our customers are interested in):

    (def person [["alice"]
                 ["bob"]
                 ["chris"]
                 ["david"]
                 ["emily"]
                 ["george"]
                 ["gary"]
                 ["harold"]
                 ["kumar"]
                 ["luanne"]])

    (def follows [["alice" "david"]
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
                  ["luanne" "gary"]])

Here's an example of a left join. To get all the follow relationships for each person in our dataset, or null if the person has no follow relationships, we run:

    (?<- (stdout)
         [?person1 !!person2]
         (person ?person1)
         (follows ?person1 !!person2))

To get all the people who do not have a follows relationship, we can run:

    (?<- (stdout)
         [?person]
         (person ?person)
         (follows ?person !!p2)
         (nil? !!p2))

Notice that the `(nil? !!p2)` predicate gets applied after `!!p2` gets joined to a ground predicate. This is an important part of the semantics of outer joins in Cascalog.

Now let's say we want the follows count for each person. A normal "count" aggregation won't work because it counts the number of tuples and doesn't distinguish between null and non-null follows. In this case, we want null follows to be counted as 0 and non-null follows to be counted as 1. Cascalog has an aggregator called `!count` that does exactly this:

    (?<- (stdout)
         [?person ?count]
         (person ?person)
         (follows ?person !!p2)
         (c/!count !!p2 :> ?count))

## Cross Joins ##

Inner joins return all records satisfying the equality constraint; how do we join datasets that have nothing at all in common?

This is called a cross join, and produces behavior similar to clojure's `for` macro:

    (for [name ["wendy" "sam"], food ["cake" "burger"]]
      [name food])

    => (["wendy" "cake"] ["wendy" "burger"] ["sam" "cake"] ["sam" "burger"])

This is usually a mistake, and causes a massive tuple blowup, but if you must do this, it's possible to trigger a cross-join by including `(cross-join)` as a predicate in your query:

    (let [name [["wendy"] ["sam"]]
          food [["cake"] ["burger"]]]
      (??<- [?name ?food]
            (name ?name)
            (food ?food)
            (cross-join)))

    => (["sam" "burger"] ["sam" "cake"] ["wendy" "burger"] ["wendy" "cake"])

Note that `??<-` defines a query, executes it, and returns the result in the form of a clojure sequence, as discussed [[Defining and executing queries|here]].
