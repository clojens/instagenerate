# instagenerate

An implementation of the core functionality of instaparse written in core.logic. While this version
is obviously not nearly as performant, with this model we can solve other problems like generating
all possible strings for a parser, or getting the string that will parse to a certain parse tree.

## Usage

	[instagenerate "0.1.0-SNAPSHOT"]
	(use 'instagenerate.core)

Note that this library must be used with instaparse and clojure.core.logic.

In `instagenerate.core` there is now a contraint, `instaparseo`, which given a parser,
constrains a string to a parse tree.

	=> (take 3 (run* [input output]
	                 (instaparseo (insta/parser "S = 'ab' C
	                                             C = 'c'+")
	                              input output)))
	([(\a \b \c) (:S "ab" (:C "c"))]
	 [(\a \b \c \c) (:S "ab" (:C "c" "c"))]
	 [(\a \b \c \c \c) (:S "ab" (:C "c" "c" "c"))])

Note that in this model, the "input" is actually a list of strings, and each string is a
chunk of characters that is parsed in each "string" combinator. This turns out
to not be a problem because we typically would use this model to _generate_ strings rather than
consume them, and once they are generated it's trivial to just append them together with
`(apply str)`.

With this model it's simple to ask what strings can be generated with a given parser (essentially
we just ignore the output):

	=> (run* [input]
	         (fresh [output]
	                (instaparseo (insta/parser "S = 'a' | 'b'") input output)))
	((\a) (\b))

As Zack Maril suggested, one may want to generate input strings given a parse tree, or given 
a parse tree "skeleton" with missing information.

	=> (def grammar
         (insta/parser
           "S = name <' '> business
            name = 'Steve' | 'Mary' | 'Bob'
            business = 'Bakery' | 'Shop'"))
       (map (partial apply str)
            (run* [input]
                  (fresh [name business]
                    (instaparseo grammar input
                                 [:S [:name name]
                                     [:business business]]))))
    ("Steve Bakery"
	 "Steve Shop"
	 "Mary Bakery"
	 "Bob Bakery"
	 "Mary Shop"
	 "Bob Shop")

There are two included functions that use the core.logic constraint above:

	(generate-strings-for-parse-tree instaparser parse-tree)
	; returns all possible strings that will result in the given parse tree.
	
	(generate-all-possible-strings instaparser parse-tree)
	; returns all possible strings that the parser can parse.

## Problems / things to note

- Negative lookahead and ordered choice are not implemented, because both concepts
are difficult to reason about in core.logic.
- If a parser is recursive in an obnoxious way (e.g. `S = S`), instagenerate will not catch
that, and it will result in an infinite loop.
- It is recommended that you check that your parser and inputs/outputs you pass to
`instaparseo` are actually possible inputs/outputs; there are some known cases in which
parser recursion will throw the solver into an infinite loop if a solution is unobtainable.


## License

Copyright © 2014

Distributed under the Eclipse Public License, the same as Clojure.
