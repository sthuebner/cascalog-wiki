Cascalog provides various helper operators in the `cascalog.ops` namespace. It is common to `(:require [cascalog.ops :as c])` within your namespace declaration. As such, you will frequently see such references as `c/each` or `c/sum` in sample code and documentation.

## all
A filter that is equivalent to boolean AND operator such that every function in the parameter must return true for the tuple to be kept. Can take multiple functions and multiple input fields. 

```clojure
(<- [!a !b]
    (nums !a !b)
    ((c/all #'even? #'big?) !a !b)))
```

# any
A filter such that it returns true if any of the passed in function returns true.

```clojure
(<- [!a !b]
    (nums !a !b)
    ((c/any #'even? #'big?) !a !b)))
```

## avg
Average.

```clojure
(<- [?avg]
    (src ?user ?cnt)
    (c/avg ?cnt :> ?avg)))
```

## comp
Composition of functions. Executes function from right to left.

```clojure
(<- [!y]
    (nums !x)
    ((c/comp #'double #'exp) !x :> !y)))
```
is equivalent to:

```clojure
(<- [!y]
    (nums !x)
    (#'exp !x :> !x1)
    (#'double !x1 :> !y))
```

## !count
`!count` takes in one input variable. Null values are interpreted as "0" and non-null values are interpreted as "1". !count returns the sum of those interpreted values. `!count` counts the number of _non-null values_ for that variable.

```clojure
(<- [?count] 
    (source !val) 
    (c/!count !val :> ?count))
```

## count

Similar to `!count`, but count values regardless whether they are null or not.

```clojure
(<- [?count] 
    (source !val) 
    (c/count !val :> ?count))
```

## distinct-count
Similar to `count`, but only count distinct items. Null values would be counted as one.

## each
Apply the specified function to each of the input variable. Number of inputs must equal number of output fields if the function is expected to return a value, otherwise there is no output variables.

```clojure
((c/each #'double) ?a ?b ?c :> ?x ?y ?z)
```
Would apply `double` to `?a :> ?x`, `?b :> ?y`, and `?c :> ?z`.

## first-n
Returns a subquery getting the first n elements. Can pass in sorting arguments.

Say `wordcount-tap` is a subquery with fields `[?word ?count]` and we want to pull the top 100 words by count. Here's how we do that with `first-n`:

```clojure
(defn top-100 [file-path]
  (c/first-n (wordcount-tap file-path)
             100
             :sort ["?count"]
             :reverse true))

(defmain Top100 [tuple-path results-path]
  (?- (hfs-textline results-path)
      (top-100 tuple-path)))
```

## fixed-sample

## fixed-sample-agg

## juxt

## lazy-generator

## limit
An efficient buffer that does most work in mappers to return the top N tuples. 

Some examples using the playground: 

Get the top 3 integers: 

```clojure
(?<- (stdout) [?n-out]
     (integer ?n) (:sort ?n) (:reverse true) 
     (c/limit [3] ?n :> ?n-out)) 
```

Get at most one friend for each person: 

```clojure
(?<- (stdout) [?p ?f-out] 
     (follows ?p ?p2) 
     (c/limit [1] ?p2 :> ?f-out)) 
```

Get 5 follows relationships: 

```clojure
(?<- (stdout) [?p-out ?p2-out] 
     (follows ?p ?p2) (c/limit [5] ?p ?p2 :> ?p-out ?p2-out)) 
```

## limit-rank
Similar to `limit` but also emit the "rank" of each item (useful when sorting): 

Get the top 3 integers with rank: 

```clojure
(?<- (stdout) [?n-out ?r] 
     (integer ?n) 
     (:sort ?n) (:reverse true)
     (c/limit-rank [3] ?n :> ?n-out ?r)) 
```

## max

## min

## negate

## partial

## sum

## with-timeout

## [re-parse [pattern]]