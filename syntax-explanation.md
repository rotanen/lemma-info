# Lemma Syntax

## Summary
In general, Lemma's syntax looks a lot like Haskell's syntax, but is not identical.
Notably, it is a "square bracket language" - it uses square brackets to enclose
scoping blocks, lambdas, and similar constructs, which in other languages may be enclosed in curly brackets or indicated by indentation.

This was inspired by Smalltalk's block syntax, but the syntax for lambdas in Lemma is slightly different than Smalltalk. I can describe it as Nix's lambda syntax enclosed in square brackets, but it's a bit more powerful than that, and I designed it before I had heard of Nix.

I prefer square brackets over curly brackets because they are easier to type on most keyboards. Curly brackets are still used in Lemma, but only for lists, in order to avoid ambiguity with the constructs that use square brackets.

Lemma has syntactically meaningful newlines, but not syntactically meaningful indentation. I consider the off-side rule too brittle. In my experience, Lemma's square-bracket syntax is just as easy to type as a Python-style indentation syntax.

I do have a working parser for Lemma syntax (written in Haskell) and it is evolving into a compiler - I have written the renamer and am mostly finished with the type inference module. I also have a "repl" to test the parser and type checker interactively. I admit that I hyperfocused on syntax design because parsers are very easy for me to write.

In this document, many syntax examples appear on one line, but that doesn't necessarily mean that newlines are disallowed in that construct. It just means that I don't want to go into details about where newlines are allowed and where they are not.

## Hello World!
```Lemma
module HelloWorld

@ [IO] ()
main = print "Hello World!"
```
## Basic Expressions
Function calls use whitespace-separated arguments, as in Haskell and many other
functional programming languages:
```Lemma
map show list
```
Parentheses are used to enclose expressions.
```Lemma
2 * (3 + 4)
```
## Comments

Comments starting with `#` continue to the end of the line.
```Lemma
# This is a line comment
```
Block comments are enclosed between pairs of `###`

```Lemma
###
Block comments can contain multiple lines.
------------------------------------------
###
```

Block comments are not nestable. Earlier versions of Lemma had nestable
comments but I now consider nestable comments to be a misfeature.

## Conditionals
If-else conditionals look like this:
```Lemma
if a > b: a else b
```

These are expressions, as in most functional languages.

However, `if` without `else` is a statement, not an expression, so it can only appear inside of blocks.

```Lemma
[if True: print "This runs"]
```
### Multi-Way If
In general, `else if` can be omitted if a newline is used instead:

```Lemma
if c > 40: "Scorching"
   c > 30: "Hot"
   c > 20: "Mild"
   c > 10: "Cool"
   c > 0 : "Chilly"
else       "Freezing"
```

## Term Definitions:
The syntax for term and function definitions is basically the same as Haskell's:

```Lemma
foo = 1
bar a b = a + b
```
Patterns can appear in the parameters for functions:

```Lemma
baz (Point a b) x = Point (a + x) (b + x)
```
As in Haskell, a function definition can have more than one case alternative:

```Lemma
map f {} = {}
map f {h | t} = {f h | map f t}
```

However, Lemma currently does not have pattern guards.

## Scoping Blocks
Scoping blocks are enclosed in square brackets. They can contain locally-scoped
term definitions, statements, and end in an expression. The value of the last
expression is the value of the entire block.

```Lemma
[
  x = 1
  y = 2
  x + y
]
```

```Lemma
[
  print "Hello"
  print "World!"
]
```
## Case Blocks

Case blocks use the following syntax:

```Lemma
case x: [
  Some a: a
  None: 0
]
```


### Multi-Scrutinee Case

To pattern-match on multiple scrutinees, multiple colons are used:

```Lemma
case x: y: [
  Some a: Some b: a == b
  None:   None:   True
  _:      _:      False
]
```

## Lambda Blocks
Like scoping blocks, lambdas are enclosed in square brackets. Colons appear after
each parameter.

```Lemma
[x: y: x + y]
```

The expression body of a lambda can be a scoping block without requiring extra square brackets
for that block:

```Lemma
[x: y:
  z = x + y
  z * z
]
```

### Case Lambdas
Pattern matching can be used in lambda parameters:

```Lemma
[Vector a b: Vector c d: Vector (a + c) (b + d)]
```

Lambdas can contain multiple case alternatives, separated by newlines:

```Lemma
[
  Some a: Some b: a == b
  None:   None:   True
  _:      _:      False
]
```

Note that if multiple case alternatives are present, any expression body with a scoping block will need its own square brackets for the scoping block. This is because I struggled to write a parser where this is unnecessary, and it may be logically impossible.

### Operator Sections

Operator sections are enclosed in square brackets too:

```Lemma
[+]     # equivalent to [x: y: x + y]
[1+]    # equivalent to [x: 1 + x]
[+1]    # equivalent to [x: x + 1]
```

## List Expressions
In Lemma, lists are enclosed in curly brackets, with comma-separated values:

```Lemma
{1, 2, 3, 4}
```

List "constructor cells" use a vertical bar to separate the head from the tail:

```Lemma
{h | t}
{1, 2, 3 | {4, 5, 6}}
```

This syntax is based on Prolog and Erlang, but curly brackets are used instead of square brackets to avoid ambiguity with the other syntactic constructs that use square brackets (particularly record field puns.)

The list concatenation operator is `~`

### List Comprehensions

List comprehensions are enclosed in curly brackets and use a for-loop-like syntax:

```Lemma
{for x in xs for y in ys if x > y: x + y}
```
Newlines are allowed before the keywords `for` and `if`, and after
the colon:

```Lemma
{for x in xs
 for y in ys
 if x > y:
 x + y}
```

## For Loops

For loops use the same syntax as the interior of a list compehension, but they are statements, not expressions, which means that they can only appear in blocks:

```Lemma
[
  xs = {1, 2, 3, 4}
  ys = {2, 4, 5}
  for x in xs for y in ys if x > y: print (x + y)
]
```

For loops aren't expressions in order to prevent a list comprehension from being parsed as a single-element list containing a for loop expression.

## Type Annotations

Type annotations begin with the @ symbol. To annotate a term definition, they appear
above the definition they annotate:

```Lemma
@ (a -> b) -> List a -> List b
map f = [
  {}: {}
  {h | t}: {f h | map f t}
]
```

A short same-line type annotation syntax is available for term definitions with no parameters:

```Lemma
x @ Int = 1
```

## Datatype Definitions

As in Haskell, type and constructor names must begin with capital letters.
Term identifiers and type variable names must begin with lowercase letters.

For algebraic data types, the constructors are enclosed in square brackets
and separated by newlines:

```Lemma
data Maybe a = [
  None
  Some a
]
```

Constructors can have named fields as well:

```Lemma
data Color = [
  RGB[red @ Real, green @ Real, blue @ Real]
  HSV[hue @ Real, sat @ Real, value @ Real]
]
```

To define a datatype with only one constructor, which uses named fields and has the same name as the type itself, a shorthand syntax is available:

```Lemma
data Point = [
  x @ Real
  y @ Real
]
```

Note that in these record field syntaxes, commas and newlines are interchangeable.

### Field Default Values
Records with named fields can have default values:

```Lemma
data Date = [
  year @ Int = 1970
  month @ Month = January
  day @ Int = 1
]

defaultDate = Date[]
ny2024 = Date[year = 2024]
```

## Records

There are two syntaxes for record constructors - positional fields and named fields.

```Lemma
Point 1 2            # positional
Point[x = 1, y = 2]  # named
Point[y = 2, x = 1]  # order does not matter in named field constructors
```

Field "puns" are allowed:

```Lemma
x = 1
y = 2
p = Point[x, y]   # equivalent to Point[x = x, y = y]
```

Field accessors use dot notation:

```Lemma
p = Point[x = 1, y = 2]
a = p.x + p.y
```

Record updaters use a dot followed by square brackets containing one or more field definitions:

```Lemma
p' = p.[x = 3]
```

A record updater lambda is a dot followed by square brackets with one or more field definitions:

```Lemma
.[x = 1]    # equivalent to [a: a.[x = 1]]
```

Similarly, an accessor lambda is a field accessor enclosed in square brackets:

```Lemma
[.name]     # equivalent to [a: a.name]
```

## Tuples

Lemma's tuple syntax is identical to Haskell's:

```Lemma
(1, True, "Hello World!")
```

The empty tuple `()` represents the unit type and its only value.

## Algebraic Effects

Effect definitions look like this:

```Lemma
effect State s = [
  get @ s
  put @ s -> ()
]
```

### The `=!` Operator

The `=!` operator is the "run this and get a value before continuing" operator. It is similar to the `<-` operator in Haskell's monadic do notation.
It indicates that the right-hand side is a computation with side effects, and the left-hand side is a term that gets the resulting value of that computation *after* its effects have run. And nothing in the block that appears after a `=!` statement can be run until the `=!` statement is run, because it might depend on the value or the side effects of the computation.

```Lemma
@ (s -> s) -> [State s] ()
modify f = [
  v =! get
  put (f v)
]
```

The `=!` operator is never necessary for pure values - the ordinary `=` operator can be used to define terms with pure values. Using `=` with a right-hand side that contains side effects defines a term *that is an effect computation*. Effect computations are pure values. Scoping blocks with `=!` statements in them are used to construct effect computations out of other effect computations.

Note that Lemma's syntax for algebraic effects is not entirely "direct style", but uses the `=!` operator for explicit sequencing. It is closer to Haskell's do-notation without `return`. This syntax preserves referential transparency but is slightly more verbose than direct style.

The `=!` operator can only be used in local scope blocks and cannot be used in top-level definitions.

### Effect Computation Types

In type signatures, the type of an effectful computation is indicated by a set of square brackets containing the effects, followed by the result type of the computation when completed:

```Lemma
@ [IO, State Int] String
```

The above represents the type of a computation with `IO` and `State Int` effects, and a result type of `String`.

Effect computation types use row polymorphism, so if the last element in an effect list is a type variable, it represents the rest of the effects in a computation (if any.)

```Lemma
@ [State Int, e] Int
```

The above type is polymorphic and represents any computation that has the `State Int` effect and possibly other effects, which would be captured by the row variable `e`.

Note that an effect list can only have one row variable and it must appear at the end of the list.

### Effect Handlers

Effect handlers are functions that pattern-match on suspended computations. The syntax is a bit difficult to explain without an example:

```Lemma
@ s -> [State s, e] a -> [e] a
runState init = [
  [pure]:      pure
  [get | k]:   runState init (k init)
  [put s | k]: runState s k
]
```

A completed computation is represented as an identifier pattern in square brackets.
A suspended computation is represented as square brackets containing the effect operation
that was called, a vertical bar, and an identifier representing the continuation to resume
the computation.

Note that Lemma uses "shallow handlers."

Effect handlers can be called like ordinary functions, but they take effect computations as a parameter:

```Lemma
six @ Int = runState 0 [
  modify [+1]
  modify [+1]
  modify [*3]
  get
]
```

Note that in the above example, the scoping block contains several `modify [..]` expressions that are functioning as statements. These indicate that these computations run in sequence, but the result value of the computations are discarded. In this case, the result values of these `modify` calls are all `()`, so discarding them makes sense.

## Typeclasses

The syntax for typeclass definitions is similar to record and effect definitions:

```Lemma
typeclass Eq a = [
  [==] @ a -> a -> Bool
  [/=] @ a -> a -> Bool
  [/=] a b = not (a == b)     # Default definition
]
```

Instance definitions are similar:

```Lemma
instance Eq Bool = [
  [==] True  True  = True
  [==] False False = True
  [==] _     _     = False
]
```
In type signatures, typeclass constraints appear before a type using this syntax:

```Lemma
@ if Num a: a -> a -> a -> a
foo a b c = a + b + c
```

Multiple constraints in the constraint context are separated by commas.

### Field Constraints

Record field constraints are similar to typeclasses and check if a type variable is a record type with the specified fields:

```Lemma
@ if a.[x @ n, y @ n], Num n: a -> n
foo a = a.x + a.y
```

This concept is necessary to have most general types while using field accessors.

Note that Lemma does not have ad-hoc record types, but field constraints can accomplish much of the same polymorphism.

And it avoids adding subtyping to the type system. I am trying to stick with the "Hindley-Milner with qualified types" model.

## Miscellaneous Operators

Below are some common operators that may be unusual for those familiar with Haskell:

| Operator | Meaning |
| -------- | ------- |
| `&` | logical and |
| `?` | logical or  |
| `~` | concatenation |
| `\` | equivalent to Haskell's `$` |
| `%` | function composition |

## Miscellaneous Rules

If a function call has arguments on more than one line, the call must be wrapped in parentheses:

```Lemma
(functionWithLotsOfArgs 1 2 3
   4 5 6
   7 8 9)
```

A semicolon can stand in for a newline in many constructions where newlines are a separator, such as scoping blocks.

```Lemma
[x = 1; y = 2; x + y]

data Bool = [False; True]
```

For record definitions and constructors, commas and newlines are interchangeable.

```Lemma
p = Point[
  x = 1
  y = 2
]
```

The parser is programmed to always parse something like `Circle[radius]` as a record constructor with a field pun, instead of a record constructor with a single positional field that's a block with just an identifier in it. The latter can be expressed as `Circle radius` anyway.

This is why square brackets are used for records. Using curly brackets or parentheses would have led either to worse ambiguities (e.g. with lists or tuples) or required ugly syntax for field puns.