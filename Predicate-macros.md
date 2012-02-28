Predicate macros allow you to arbitrarily compose together other predicates. Predicate macros are defined when input or output vars are explicitly defined using `:>` or `:<` within the declared variables of a query. 

When a predicate macro is used, it expands to one or more predicates.
 
For example, here's the how you can compose together `sum` and `count` to create `average`: 

```clojure
(def average 
  (<- [!val :> !avg] 
      (c/count !c)
      (c/sum !val :> !s)
      (div !s !c :> !avg))) 
```
Here's an example of how `average` is expanded within a query: 

```clojure
(<- [?avg-age] 
    (age _ ?a)
    (average ?a :> ?avg-age))
```
expands to: 

```clojure
(<- [?avg-age]
    (age _ ?a)
    (count !c_1)
    (sum ?a :> !s_1)
    (div !s_1 !c_1 :> ?avg-age))
```
Any non-declared variables used in a predicate macro, like `!s` and `!c` in average, are given unique names so as not to conflict with other variables when expanded. 

Another example of a predicate macro is "distinct count", which will count the number of unique occurrences of a value. distinct-count secondary sorts the values and then does the computation in a single scan of the values through a custom aggregator:

```clojure
(def distinct-count 
  (<- [!v :> !c] 
      (:sort !v) 
      (distinct-count-agg !v :> !c))) 
```