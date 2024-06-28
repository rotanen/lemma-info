# Lemma syntax

## Summary
In general, Lemma's syntax looks a lot like Haskell's syntax, but is not identical.
Notably, it is a "square bracket language" - it uses square brackets to enclose
scoping blocks, lambdas, and similar constructs, which in other languages may be enclosed in curly brackets or indicated by indentation.

This was inspired by Smalltalk's block syntax, but the syntax for lambdas in Lemma is slightly different than Smalltalk.

I prefer square brackets over curly brackets because they are easier to type on most keyboards. The earliest curly-bracket languages emerged when this was not the case (both square and curly brackets were difficult to type on the keyboards of the time.)

Lemma has syntactically meaningful newlines, but not syntactically meaningful indentation. I consider the off-side rule too brittle. In my experience, Lemma's square-bracket syntax is just as easy to type as a Python-style indentation syntax.

I do have a working parser for Lemma syntax (written in Haskell) and it is evolving into a compiler. I also have a "repl" to test the parser interactively. I admit that I hyperfocused on syntax design because parsers are very easy for me to write.

## Hello World!

    module HelloWorld

    @ [IO] ()
    main = print "Hello World!"

## Basic expressions
Function calls use whitespace-separated arguments, as in Haskell and many other
functional programming languages:

    map double list

Parentheses are used to enclose expressions.

    2 * (3 + 4)

## Comments

Comments starting with `#` continue to the end of the line.

    # This is a line comment

Block comments are enclosed between pairs of `###`

    ###
    Block comments can contain multiple lines.
    ------------------------------------------
    ###

Block comments are not nestable. Earlier versions of Lemma had nestable
comments but I now consider nestable comments to be a misfeature.

## Conditionals
If-else conditionals look like this:

    if a > b: a else b

These are expressions, as in most functional languages.

In general, "else if" can be omitted if a newline is used instead:

    if x > 100: "Too much"
       x > 50:  "A decent amount"
    else        "Not enough"


## Term definitions:
The syntax for term and function definitions is basically the same as Haskell's:

    foo = 1
    bar a b = a + b

Patterns can appear in the parameters for functions:

    baz (Point a b) x = Point (a + x) (b + x)

As in Haskell, a function definition can have more than one case alternative:

    map f {} = {}
    map f {h | t} = {f h | map f t}

However, Lemma currently does not have pattern guards.

## Scoping blocks
Scoping blocks are enclosed in square brackets. They can contain locally-scoped
term definitions, statements, and end in an expression. The value of the last
expression is the value of the entire block.

    [
      x = 1
      y = 2
      x + y
    ]

## Case blocks

Case blocks use the following syntax:

    case x: [
      Some a: a
      None: 0
    ]

To pattern-match on multiple scrutinees, multiple colons are used:

    case x: y: [
      Some a: Some b: a == b
      None:   None:   True
      _:      _:      False
    ]

## Lambda blocks
Like scoping blocks, lambdas are enclosed in square brackets. Colons appear after
each parameter.

    [x: y: x + y]

The expression body of a lambda can be a scoping block without requiring extra square brackets
for that block:

    [x: y:
      z = x + y
      z * z
    ]

Pattern matching can be used in lambda parameters:

    [Vector a b: Vector c d: Vector (a + c) (b + d)]

Lambdas can contain multiple case alternatives, separated by newlines:

    [
      Some a: Some b: a == b
      None:   None:   True
      _:      _:      False
    ]

Note that if multiple case alternatives are present, any expression body with
a scoping block will need its own square brackets for the scoping block. This is because I struggled to write a parser where this is unnecessary, and it may be logically impossible.

Operator sections are enclosed in square brackets too:

    [+]     # equivalent to [x: y: x + y]
    [1+]    # equivalent to [x: 1 + x]
    [+1]    # equivalent to [x: x + 1]

## List expressions
In Lemma, lists are enclosed in curly brackets, with comma-separated values:

    {1, 2, 3, 4}

List "constructor cells" use a vertical bar to separate the head from the tail:

    {h | t}
    {1, 2, 3 | {4, 5, 6}}

This syntax is based on Prolog and Erlang, but curly brackets are used instead of square
brackets to avoid ambiguity with the other syntactic constructs that use square brackets (particularly record field puns.)

The list concatenation operator is ~

## List comprehensions

List comprehensions are enclosed in curly brackets and use a for-loop-like syntax:

    {for x in xs, y in ys if x > y: x + y}

## Type annotations

Type annotations begin with the @ symbol. To annotate a term definition, they appear
above the definition they annotate:

    @ (a -> b) -> List a -> List b
    map f = [
      {}: {}
      {h | t}: {f h | map f t}
    ]

A short same-line type annotation syntax is available for term definitions with no parameters:


    x @ Int = 1

## Datatype definitions

As in Haskell, type and constructor names must begin with capital letters.
Term identifiers and type variable names must begin with lowercase letters.

For algebraic data types, the constructors are enclosed in square brackets
and separated by newlines:

    data Maybe a = [
      None
      Some a
    ]

Constructors can have named fields as well:

    data Color = [
      RGB[red @ Real, green @ Real, blue @ Real]
      HSV[hue @ Real, sat @ Real, value @ Real]
    ]

To define a datatype with only one constructor, which uses named fields and has the same name as the type itself, a shorthand syntax is available:

    data Point = [
      x @ Real
      y @ Real
    ]

Note that in these record field syntaxes, commas and newlines are interchangeable.

Records with named fields can have default values:

    data Date = [
      year @ Int = 1970
      month @ Month = January
      day @ Int = 1
    ]

    defaultDate = Date[]

## Records

There are two syntaxes for record constructors - positional fields and named fields.

    Point 1 2
    Point[x = 1, y = 2]
    Point[y = 2, x = 1]

Field "puns" are allowed:

    x = 1
    y = 2
    p = Point[x, y]

Field accessors use dot notation:

    p = Point[x = 1, y = 2]
    a = p.x + p.y

Record updaters use a dot followed by square brackets containing one or more field definitions:

    p' = p.[x = 3]

A record updater lambda is a dot followed by square brackets with one or more field definitions:

    .[x = 1]    # equivalent to [a: a.[x = 1]]

Similarly, an accessor lambda is a field accessor enclosed in square brackets:

    [.name]        # equivalent to [a: a.name]

## Algebraic Effects

Effect definitions look like this:

    effect State s = [
      get @ s
      put @ s -> ()
    ]

Within a computation, the definition operator `=!` works similar to Haskell's `<-` in monadic do-notation.
It indicates that the right-hand side should be run once and its value assigned to the left-hand side before
continuing the computation:

    @ (s -> s) -> [State s] ()
    modify f = [
      v =! get
      put (f v)
    ]

Note that Lemma's syntax for algebraic effects is not entirely "direct style", but uses
the `=!` operator for explicit sequencing. It is closer to Haskell's do-notation without `return`.
This syntax preserves referential transparency but is slightly more verbose than direct style.

The `=!` operator can only be used in local scope blocks and cannot be used in top-level definitions.

In type signatures, the type of an effectful computation is indicated by a set of square brackets containing the effects,
followed by the result type of the computation when completed:

    @ [IO, State Int] String

The above represents the type of a computation with IO and State Int effects, and a result type of String

Effect handlers are functions that pattern-match on suspended computations. The syntax is a bit difficult to explain without an example:

    @ s -> [State s, e] a -> [e] a
    runState init = [
      [pure]:      pure
      [get | k]:   runState init (k init)
      [put s | k]: runState s k
    ]

A completed computation is represented as an identifier pattern in square brackets.
A suspended computation is represented as square brackets containing the effect operation
that was called, a vertical bar, and an identifier representing the continuation to resume
the computation.

Note that Lemma uses "shallow handlers."

## Typeclasses

The syntax for typeclass definitions is similar to record and effect definitions:

    typeclass Eq a = [
      [==] @ a -> a -> Bool
      [/=] @ a -> a -> Bool
      [/=] a b = not (a == b)     # Default definition
    ]

Instance definitions are similar:

    instance Eq Bool = [
      [==] True  True  = True
      [==] False False = True
      [==] _     _     = False
    ]

In type signatures, typeclass constraints appear before a type using this syntax:

    @ if Num a: a -> a -> a -> a
    foo a b c = a + b + c

Multiple constraints in the constraint context are separated by commas.

## Field constraints

Record field constraints are similar to typeclasses and check if a type variable is a record type with the specified fields:

    @ if a.[x @ n, y @ n], Num n: a -> n
    foo a = a.x + a.y

This concept is necessary to have most general types while using field accessors.

Note that Lemma does not have ad-hoc record types, but field constraints can accomplish much of the same polymorphism.

And it avoids adding subtyping to the type system. I am trying to stick with the "Hindley-Milner with constraints" model.

## Miscellaneous operators

Below are some common operators that may be unusual for those familiar with Haskell:

| Operator | Meaning |
| -------- | ------- |
| `&` | logical and |
| `?` | logical or  |
| `~` | concatenation |
| `\` | equivalent to Haskell's `$` |
| `%` | function composition |

## Miscellaneous rules

If a function call has arguments on more than one line, the call must be wrapped in parentheses:

    (functionWithLotsOfArgs 1 2 3
     4 5 6
     7 8 9)

A semicolon can stand in for a newline in many constructions where newlines are a separator, such as scoping blocks.

    [x = 1; y = 2; x + y]

    data Bool = [False; True]

For record definitions and constructors, commas and newlines are interchangeable.

    p = Point[
      x = 1
      y = 2
    ]

The parser is programmed to always parse something like `Circle[radius]` as a record constructor with a field pun, instead of a record constructor with a single positional field that's a block with just an identifier in it. The latter can be expressed as `Circle radius` anyway.