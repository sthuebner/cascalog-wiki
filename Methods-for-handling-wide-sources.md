In many real world examples, the number of tuple fields produced by some generator can grow to be quite cumbersome.

Often, a subquery will only want to deal with a small subset of the fields produced. Cascalog provides a few methods that make it quite easy to deal with these wide taps.

(Further examples of these solutions can be found in Nathan Marz's [cascalog workshop repository](https://github.com/nathanmarz/cascalog-workshop/blob/master/src/clj/workshop/wide.clj).)

## Underscore ##

Take this example query, designed to add the first two fields produced by the generator and assign the result to `?sum`:

    (<- [?a ?b ?sum]
        (+ ?a ?b :> ?sum)
        (generator ?a ?b ?c ?d ?e ?f ?g ?h ?i))

Not only does this query take up a lot of space; the user also has to think up dummy names for the unused variables. One solution Cascalog allows is the use of underscores for ignored variables:

    (<- [?a ?b ?sum]
        (+ ?a ?b :> ?sum)
        (generator ?a ?b _ _ _ _ _ _ _))

This works, but is still a bit clunky. Next choice: `select-fields`.

## Fields by Name ##

If `generator` is some other subquery with named tuple fields, we can use `select-fields` to pull `?a` and `?b` without knowing their position.

    (<- [?a ?b ?sum]
        (+ ?a ?b :> ?sum)
        ((select-fields generator ["?a" "?b"]) ?a ?b))

Note that `select-fields` takes a generator and a sequence of (string representations of) variables, and returns a new generator that produces only those variables. 

## Fields by Position ##

Our final method of attacking wide taps involves the predicate operator `:#>`. (All other predicate operators are discussed in [[Predicate operators]].) Here's an example:

    (<- [?a ?b ?sum]
        (+ ?a ?b :> ?sum)
        (generator :#> 9 {0 ?a, 1 ?b}))

`:#>` sits to the right of our generator, and accepts two items: the total number of fields produced by the generator, and a map of variable names keyed by position within the generator. Variables names don't have to match the names on the original output tap, as position is already specified.