Cascalog provides various helper operators in the `cascalog.ops` namespace. It is common to `(:require [cascalog.ops :as c])` within your namespace declaration. As such, you will frequently see such references as `c/each` or `c/sum` in sample code and documentation.

## all
A filter that is equivalent to boolean AND operator such that every function in the parameter must return true for the tuple to be kept. Can take multiple functions and multiple input fields. 

```clojure
(<- [!a !b]
    (nums !a !b)
    ((c/all #'even? #'big?) !a !b)))
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

