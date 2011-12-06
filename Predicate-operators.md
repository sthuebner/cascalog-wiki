Cascalog includes two basic operators for specification of predicate input and output variables: `:<` and `:>`. Each of these has an equivalent version (`:<<` and `:>>` respectively) that allows for specification of arrays of input and output variables. We'll start with the more familiar operators, `:<` and `:>`.

## :< and :> ##

The `:<` predicate operator treats the variables on its right as input to the function on its left, as here:

    (some-op :< ?a :> ?b)

Similarly, the familiar `:>` operator, located after the initial function and input vars (if any exist), marks variables to its right as outputs.

`:>` can be found in many of the available Cascalog predicate examples, including all at [[Guide to custom operations]]. Predicates in these examples typically take one of two forms: `(<function> ?in-var :> ?out-var)` for operations, and `(<function> ?some-var)` for generators and filters.

The first of these makes intuitive sense: an operation is acting on the input variables and generating a new set of output variables.

    (some-op ?a ?b :> ?output)

The first bit, `(some-op ?a ?b ...`, looks like standard Clojure function application! The truth is that Cascalog is making certain assumptions about our predicates, and letting us drop operators that might make what's happening more clear to the uninitiated.

Let's use the following dataset and query as an example. (For a refresher on query structure, see [[Defining and executing queries]].)

    (def digits [[1 2]
                 [3 4]])

    (<- [?a ?b ?c]
        (digits ?a ?b)
        (+ ?a ?b :> ?c))

The `+` function receives `?a` and `?b`, and generates `?c`. A more explicit form of this predicate would be:

    (+ :< ?a ?b :> ?c)

The `:<` ushers the variables to its right into the function to its left. We can leave it out in almost all queries because Cascalog is smart enough to detect where `:<` should go; at the break between the first function and the following dynamic variables. `:>` is necessary, as there is no other way to distinguish between inputs and outputs.

This should help us with our understanding of the other form we've seen for predicates. This second form involves a single function and a series of vars:

    (generator ?out-a ?out-b ?out-c)
    (some-filter ?in-a)

Both generators and filters are written in this fashion; the implicit difference between the two is that `?out-a`, `?out-b` and `?out-c` are output variables emitted from `generator`, while `?in-a` is an input variable.

Again, Cascalog infers the presence of, respectively, the `:>` and `:<` operators. Our generator could be written like so:

    (generator :> ?out-a ?out-b ?out-c)

After all, what is a generator but a function with no input vars? Similarly, our filtering predicate can be written as:

    (some-filter :< ?in-a)

Cascalog implicitly makes the right choice, as it can detect the difference between a generator and a filter (or a vanilla clojure function.)

### :<< and :>> ###

The `:<<` and `:>>` operators are equivalent to `:<` and `:>` discussed above, except that they're used to denote sequences of input and output variables. A thorough understanding of the implicit positioning of `:<` and `:>` will go a long way toward understanding how to use the predicate array operators.

As an example, the following query is functionally equivalent to the one discussed above:

    (def my-vars ["?a" "?b"])

    (<- [?a ?b ?c]
        (digits :>> my-vars)
        (+ :<< my-vars :> ?c))

Note that variables within `my-vars` are strings. Trying to put straight symbols into a sequence results in the expected error:

    user=> [?a ?b]
    java.lang.Exception: Unable to resolve symbol: ?a in this context (NO_SOURCE_FILE:0)

### :#> ###

Our final predicate operator, `:#>`, allows the user to pull specific tuple fields out of a generator by referencing position. Its use is described in [[Methods for handling wide sources]].