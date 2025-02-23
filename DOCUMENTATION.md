Table of contents
=================

- [Motivation](#motivation)
- [Installation](#installation)
- [Introduction](#introduction)
- [Primitives](#primitives)
    - [Let](#let)
    - [Lambda](#lambda)
    - [Self](#self)
    - [Annotation](#annotation)
    - [Hole](#hole)
    - [Log](#log)
    - [Import](#import)
- [Datatypes](#datatypes)
    - [Basics](#basics)
    - [Fields](#fields)
    - [Move](#move)
    - [Recursion](#recursion)
    - [Polymorphism](#polymorphism)
    - [Indices](#indices)
    - [Motive](#motive)
    - [Encoding](#encoding)
- [Advanced](#advanced)
    - [Proofs](#proofs)
- [Theory](#theory)
    - [Formality Core](#formality-core)
    - [Formality Calculus](#formality-calculus)
    - [Formality Net](#formality-net)
    - [Compilation](#compilation)

**Note: this documentation is slightly outdated after the
December 6, 2019 update. Some examples here may not work. It
will be reviewed and updated next week. If you like learning
by example, please ignore this docs and check
[Base.fm](https://github.com/moonad/Base.fm); the code there
is very representative and, if you already know FP, browsing
it is the fastest way to pick up the language.**

Motivation
==========

> Knowing mathematics and programming might one day be one... it fills you with determination.

Formality exists to fill a hole in the current market: there aren't many
languages featuring *theorem proving* that are simple, user-friendly and
efficient. To accomplish that goal, we rely on several design philosophies:

An accessible syntax
--------------------

Proof languages often have complex syntaxes that make them needlessly
inaccessible, as if the subject wasn’t hard enough already. Coq, for example,
uses 3 different languages with different rules and an overall heavy syntax.
Agda is clean and beautiful, but relies heavily on unicode and agda-mode,
making it essentially unusable outside of EMACs, which is arguably a “hardcore”
editor. Formality aims to keep a simple, familiar syntax that is much closer to
common languages like Python and JavaScript. A regular TypeScript developer
should, for example, be able to read our
[Functor](https://github.com/moonad/Formality-Base/blob/master/Control.Functor.fm)
formalization without extensive training. While we may not be quite there,
we’re making fast progress towards that goal.

Fast and portable "by design"
-----------------------------

 Some languages are inherently slow, by design. JavaScript, for example, is
 slower than C: all things equal, its mandatory garbage collector will be an
 unavoidable disadvantage. Formality is meant to be as fast as theoretically
 possible. For example, it has affine lambdas, allowing it to be
 garbage-collection-free. It has a strongly confluent interaction-net runtime,
 allowing it to be evaluated in massively parallel architectures. It doesn’t
 require De Bruijn bookkeeping, making it the fastest “closure chunker” around. It
 is lazy, it has a clear cost model for blockchains, it has a minuscule ([435
 LOC](https://github.com/moonad/Formality/blob/master/src/fm-net.js)) runtime
 that can easily be ported to multiple platforms. Right now, Formality’s
 compiler isn’t as mature as the ones found in decades-old languages, but it
 has endless room for improvements since the language is fast “by design”.

An elegant underlying Type Theory
---------------------------------

Formality's unique approach to termination is conjectured to allow it to have
elegant, powerful type-level features that would be otherwise impossible
without causing logical inconsistencies. For example, instead of built-in
datatypes, we rely on [Self
Types](https://www.semanticscholar.org/paper/Self-Types-for-Dependently-Typed-Lambda-Encodings-Fu-Stump/652f673e13b889e0fd7adbd480c2fdf290621f66),
which allows us to implement inductive families with native lambdas. As history
tells, having elegant foundations often pays back. We've not only managed to
port several proofs from other assistants, but found techniques to [emulate
Coq's structural
recursion](https://github.com/moonad/Formality-Base/commit/b777d806c6fa37f2ce306fbe87b3ed267152b90c),
to perform large eliminations, and even a hypothetical encoding of [higher
inductive
types](https://github.com/moonad/Formality-Base/blob/master/Example.HigherInductiveType.fm);
and we've barely begun exploring the system.

An optimal high-order evaluator
-------------------------------

Formality's substitution algorithm is **asymptotically faster** than Haskell's,
Clojure's, JavaScript's and other closure implementations. This makes it
extremely fast at evaluating high-order programs, combining a Haskell-like
high-level feel with a Rust-like low-level performance curve. For example,
Haskell's stream fusion, a hard-coded, important optimization, happens
naturally, [at
runtime](https://medium.com/@maiavictor/solving-the-mystery-behind-abstract-algorithms-magical-optimizations-144225164b07),
on Formality. This also allows us to explore new ways to develop algorithms,
such as this "impossibly efficient" [exp-mod
implementation](https://medium.com/@maiavictor/calling-a-function-a-googol-times-53933c072e3a).
Who knows if this may lead to new breakthroughs in complexity theory?

![](https://raw.githubusercontent.com/moonad/Assets/master/images/inet-simulation.gif)

Installation
============

Right now, Formality can only be installed through npm. Install npm following
[this guide](https://www.npmjs.com/get-npm). Then, go to the command-line and
type:

```
$ npm i -g formality-lang
```

Using [`node2nix`](https://github.com/svanderburg/node2nix#installation), we
can also install Formality using the Nix package manager:

```
$ git clone git@github.com:moonad/Formality.git
$ cd Formality
$ nix-channel --add https://nixos.org/channels/nixpkgs-unstable unstable
$ nix-env -f '<unstable>' -iA nodePackages.node2nix
$ node2nix --nodejs-12
$ sed -i 's/nixpkgs/unstable/g' default.nix
$ nix-env -f default.nix -iA package
```

This should be all you need. In order to test if it worked, type `fm` on the
terminal. If you see Formality's command-line options, then it has been
successfully installed in your system. If you have any problem during this
process, please [open an issue](https://github.com/moonad/Formality/issues).


Introduction
============

This is the "Hello, World!" in Formality:

```haskell
import Base#

main : String
  "Hello, world!"
```

A Formality file is just a list of imports followed by a series of top-level
definitions. Here, we have one top-level definition, `main`, with type
`String`, and body `"Hello, world"`. Save this file as `hello.fm`.

To run it, type `fm hello/main`. This will evaluate `main` using an interpreter
in debug mode and output `"Hello, world!"`. You can also use `fm -o hello/main`
to evaluate it with the interaction-net runtime. This will be faster, but
you'll lose information like variable names and logs.

To type-check it, type `fm -t hello/main`. This will check if the program's
type is correct and print `String ✔`. If the type is incorrect, it will print
an error message instead. For example, if you change `"Hello, world!"` to `true`,
it will print:

```haskell
Type mismatch.
- Found type... Bool
- Instead of... String
- When checking 7
- On line 4, col 9, file hello.fm:
  1| import Base#
  2|
  3| main : String
  4|   print(true)
  5|
```

Because `true` is a `Bool`, but the `print` function expects
a `String`. Since Formality is a proof language, types can
be seen as theorems and well-typed terms can be seen a
proof. So, for example, this is a proof that `"cat" == "cat"`:

```haskell
import Base#

main : "cat" == "cat"
  equal(__)
```

Running it with `fm -t file/cat_is_cat` will output
`Equal(String, "cat", "cat") ✔`, which means Formality is
convinced that "cat" is equal to "cat". Of course, that is
obvious, but it could be something much more important such
as `OnlyOwnerCanWithdrawal(owner, contract) ✔`. How proofs
can be used to make your programs safer will be explored
later. Let's now go through all of Formality's primitives.
Don't worry: since it is designed to be a very simple
language, there aren't many!

Primitives
==========

Lambda
------

syntax | description
--- | ---
`(x : A, y : B, z : C, ...) -> D` | Function type with args `x : A`, `y : B`, `z : C`, returning `D`
`(x, y, z, ...) => body` | A function that receives the arguments `x`, `y`, `z` and returns `body`
`f(x, y, z, ...)` | Applies the function `f` to the arguments `x`, `y`, `z` (curried)

The lambda (also called function) is the only computational
primitives of Formality. Everything, from booleans, to list,
to complex algorithms, data structures and mathematical
proofs are compiled to plain lambdas.

As on other functional languages, lambdas are curried. `(x,
y, z, ...) => body` is the same as `(x) => (y) => (z) => ...
body`, and `f(x, y, z)` is the same as `f(x)(y)(z)...`.

The type of a function is written as `A -> B -> C -> D`,
like on Haskell, but it can also be written with names, as
`(x : A, y : B, z : C ...) -> D`.

You can define a top-level function as below:

```haskell
import Base#

same : String -> String
  (x) => x

main : String
  same("Hello, world!")
```

You can also include the variable names before the `:`:

```haskell
import Base#

same(x : String) : String
  x

main : String
  same("Hello, world!")
```

Functions can be inlined:

```haskell
import Base#

call(f : String -> String, x : String) : String
  f(x)

main : String
  call((x) => same(x), "Hello, world!")
```

If Formality can't infer the type of `x`, you can add it after the name:

```haskell
import Base#

main : String
  ((x : String) => x)("Hello, world!")
```

Or after the function, with an explicit annotation (`::`):

```haskell
import Base#

main : String
  (((x) => x) :: String -> String)("Hello, world!")
```

Types are optional. This won't type-check, but you can still run it:

```haskell
main
  ((x) => x)("Hello, world!")
```

Lambdas and applications can be erased with a `;`, which
causes them to vanish from the compiled output. This is
useful, for example, to write polymorphic functions without
extra runtime costs. For example, on the code below, `foo`
is compiled to `(x) => x`, and `main` is compiled to
`foo("Hello, world!")`. The first argument disappears from the
runtime.

```haskell
foo(T : Type; x : T) : T
  x

main : String
  foo(String; "Hello, world!")
```


Formality functions are **affine**, which means you can't use a variable more
than once. For example, the program below isn't allowed, because `b` is used
twice:

```haskell
import Base#

self_or(b : Bool) : Bool
  or(b, b)

main : Bool
  self_or(true)
```

Multiple ways to circumvent this limitation will be explained through this
documentation.


Let
---

Allows you to give local names to terms.

```haskell
import Base#

main : String
  let hello = "Hello, world!"
  print(hello)
```

`let` expressions can be infinitely nested.

```haskell
import Base#

main : Output
  let output =
    let hello = "Hello, world!"
    print(hello)
  output
```

`let` has no computational effect, it simply performs a parse-time substitution.

Self
----

Formality also has [Self
Types](http://homepage.divms.uiowa.edu/~astump/papers/fu-stump-rta-tlca-14.pdf),
which allow us to implement inductive datatypes with λ-encodings:

syntax | description
--- | ---
`${self} T(self)` | `T` is a type that can access its own value
`new(T) t` | Constructs an instance of a `T` with value `t`
`use(t)` | Consumes a self-type `t`, giving its type access to its value

Self Types allow a type to access *its own value*. This
allows us to do us to encode inductive datatypes with
lambdas, as will be explained later.

Annotation
----------

You can also explicitly annotate the type of a term:

syntax | description
--- | ---
`term :: Type` | Annotates `term` with type `Type`

This is useful when the type-checker can't infer the type of an expression.

```haskell
main : String
  (((x) => x) :: String -> String)("Hello, world!")
```

Nontermination
--------------

Formality has first-class nontermination, which disables the termination
checker in delimited sections of your programs.

syntax | description
------ | ----------------------------------------
-A     | Nonterminating term of type `A`
\<t\>  | Converts `t : A` to `t : -A`
+t     | Converts `t : -A` to `t : A`

Inside `<...>` (that is, on nonterminating terms), you can
use unrestricted recursion, as well as ignore the
limitations imposed by affine lambdas and stratified
duplications. So, for example, while the `List(Bool)` type
represents a finite list of words, the `-List(Bool)` type
can be infinite:

```javascript
trues : -List(Bool)
  <cons(Bool; true, +trues)>
```

Note that nonterminating terms can't be compiled to interaction nets, and can't
be interpreted as mathematical proofs. That's because nontermination can be
used to create paradoxes, allowing you to inhabit a type with a loop:

```haskell
  main : -Empty
    main
```

A powerful aspect of Formality is that you can still prove theorems about
nonterminating programs, as long as the proofs themselves are terminating. This
idea was first explored in ["A dependently typed language with
nontermination"](https://www.cis.upenn.edu/~sweirich/papers/sjoberg-thesis.pdf).
Nonterminating terms are also very useful when you have a function argument
that will only be used inside a type such as the `Nat` index of a Vector, and
they also play an important role in the implementation of inductive datatypes
with self-types.

Hole
----

Formality also features holes, which are very useful for development, debugging
and cleaning up type annotations from your code. A hole can be used to fill a
part of your program that you don't want to implement yet. It can be written
anywhere as `?name`, with the name being optional. If you give it a name, it
will cause Formality to print the type expected on the hole location, its
context (scope variables), and possibly a value, if Formality can fill it for
you. For example, the program below:

```haskell
import Base#

main(x : Bool) : Bool
  or(true, ?a)
```

Will output:

```haskell
Found hole: 'a'.
- With goal... Bool
- With context:
- x : Bool

(x : Bool) -> Bool ✔
```

This tells you that, on the location of the hole, you should have a `Bool`.
Note that this is only automatic if Formality can infer the expected type of
the hole's location. Otherwise, you must give it an explicit annotation, as in
`?hole :: MyType`.

When formality can infer the value of a hole for you, you don't need to complete
it; you can leave a `?` in your code. For example, below:

```haskell
main : List(Bool)
  cons(?; true, nil(?;))
```

We don't need to write `Bool`, since Formality can infer that. We can leave it
as `?`. Since erased arguments are often inferrable, a `_` can be used as a
shortcut for `?;`, as in:

```haskell
main : List(Bool)
  cons(_ true, nil(_))
```

Log
------

Another handy feature is `log(x)`. When running a program, it will print the
normal form of `x`, similarly to haskell's `console.log` and haskell's `print`,
but for anything (not only strings). When type-checking a program, it tells you
the normal-form and the type of `x`. This is useful when you want to know what
type an expression would have inside certain context. For example:

```haskell
import Base#

main(f : Bool -> Nat) : Nat
  log(f(true))
  ?a
```

Type-checking the program above will cause Formality to output:

```haskell
[LOG]
Term: f(true)
Type: Nat

Found hole: 'a'.
- With goal... Nat
- With context:
- f : (:Bool) -> Nat
```

This tells you that, inside the body of `main`, the type of `f(true)` is `Nat`.
Since it coincides with the goal, you can complete the program above with it:

```haskell
import Base#

main(f : Bool -> Nat) : Nat
  f(true)
```

Compile-time logs are extremelly useful for development. We highly recommend
you to use them as much as possible!

Import
------

The `import` statement can be used to include local files. For example, save an
`Answers.fm` file in the same directory as `hello.fm`, with the following
contents:

```haskell
import Base#

everything : String
  "42"
```

Then save a `test.fm` file as:

```haskell
import Base#
import Answers

main : Output
  print(everything)
```

And run it with `fm test/main`. You should see `42`.

If multiple imports have conflicting names, you can disambiguate with
`File/name`, or with a qualified import, using `as`:


```haskell
import Base#
import Answers as A

main : Output
  print(A/everything)
```

Formality also has a file-based package manager. You can use it to share files
with other people. A file can be saved globally with `fm -s file`. This will
give it a unique name with a version, such as `file#7u3k`. Once given a unique
name, the file contents will never change, so `file#7u3k` will always refer to
that exact file. As soon as it is saved globally, you can import it from any
other computer. For example, remove `Answers.fm` and change `hello.fm` to:

```haskell
import Base#
import Answers#xxxx

main : Output
  print(everything)
```

This will load `Answers#xxxx.fm` inside the `fm_modules` directory and load it.
Any import ending with `#id` refers to a unique, immutable, permanent global
file. That prevents the infamous "dependency hell", and is useful for many
applications.

Right now, global imports are uploaded to our servers, but, in the future,
they'll upload files to decentralized storage such as IPFS/Swarm, and give it a
unique name using Ethereum's naming system.

Datatypes
=========

Formality includes a powerful datatype system. A new datatype can be defined
with the `T` syntax, which is similar to Haskell's `data`. It creates global
definitions for the type and its constructors. To pattern-match against a value
of a datatype, you must use a `case` expression.

Basics
------

Datatypes can be defined and used as follows:

```haskell
import Base#

T Suit
| clubs
| diamonds
| hearts
| spades

print_suit(suit : Suit) : Output
  case suit
  | clubs    => print("First rule: you do not talk about Fight Club.")
  | diamonds => print("Queen shines more than diamond.")
  | hearts   => print("You always had mine.")
  | spades   => print("The only card I need is the Ace of Spades! \m/")
  : Output

main : Output
  print_suit(spades)
```

The program above creates a datatype, `Suit`, with 4 possible values. In
Formality, we call those values **constructors**. It then pattern-matches a
suit and outputs a different sentence depending on it. Notice that on this
`case` expression, we annotated the return type, `: Output`. That's not
always necessary, but it is very important for theorem proving. The annotated
type of a case expression is called its **motive**.

Fields
------

Datatype constructors can have fields, allowing them to store values:

```haskell
import Base#

T Person
| person(age : Nat, name : String)

get_name(p : Person) : String
  case p
  | person => p.name

main : Output
  let john = person(26, "John")
  print(get_name(john))
```

As you can see, fields can be accessed inside `case` expressions. Notice that
`p.name` is not a field accessor, but just a single variable: the `.` is part
of its name. When Formality doesn't know the name of the matched value, you
must must explicitly name it using the `as` keyword:

```haskell
main(p : Person) : Nat
  case person(26n, "John") as john
  | person => john.age
```

Move
----

Since Formality functions are affine, you can't use an argument more than once.
So, for example, the function below isn't allowed:

```haskell
import Base#

main(a : Bool, b : Bool) : Bool
  case a
  | true  => b
  | false => not(b)
```

But, since we used `b` in two different branches, we don't need to copy it:
we can instead tell Formality to move it to each branch with a `+`:

```haskell
import Base#

main(a : Bool, b : Bool) : Bool
  case a
  + b : Bool
  | true  => b
  | false => not(b)
```

Under the hoods, this just adds an extra lambda on each branch:

```haskell
import Base#

main(a : Bool, b : Bool) : Bool
  (case a
  | true  => (b : Bool) => b
  | false => (b : Bool) => not(b))(b)
```

Recursion
---------

Fields can refer to the datatype being defined:

```haskell
T Nat
| zero
| succ(pred : Nat)
```

This creates a recursive datatype. Since `Nat` is so common, there is a
syntax-sugar for it, `3n`, which expands to `succ(succ(succ(zero)))`.

Mutual recursion is allowed:

```haskell
T Foo
| foo(bar : Foo)

T Bar
| bar(foo : Bar)
```

Recursive functions can be written as usual:

```haskell
mul2(n : Nat) : Nat
  case n
  | zero => zero
  | succ => succ(succ(mul2(n.pred)))
  : Nat
```

With one caveat: recursive occurrences must be applied to structurally smaller
values, in order to prevent nontermination. Right now, though, while
Formality's termination checker can prevent loops such as `λx.(x x) λx.(x x)`
from being constructed (for example, through Russel's paradox), it still
can't prevent loops from recursive calls. This feature is to be developed
soon. Until then, type-checking recursive functions will raise a warning,
signaling that Formality is unsure if your program halts. If you can't wait,
you can use inductive datatypes such as `INat` from Base instead.

Note: while you can use recursive calls as much as you want, it is wise to
treat them as normal variables and use `move` to pass them to branches. This
can be a major optimization in some cases.

Polymorphism
------------

Polymorphism allows us to create multiple instances of the same datatype with
different contained types.

```haskell
import Base#

// Imported from Base
// T Pair<A, B>
// | pair(fst : A, snd : B)

main : Nat
  let a = pair(Bool; Nat; true, 7n)

  case a
  | pair => a.snd
```

The `<A, B>` syntax after the datatype declares two polymorphic Type variables,
`A` and `B`, allowing us to create pairs with different contained types without
having to write multiple `Pair` definitions. 

Each polymorphic variable adds an implicit, erased argument to each constructor,
so, instead of `pair(true, 7)`, you must write `pair(Bool; Nat; true, 7)`;
or, with holes, `pair(__ true, 7)`.

One of the most popular polymorphic types is the linked `List`:

```haskell
// Imported from Base
// T List<A>
// | nil
// | cons(head : A, tail : List(A))

main : List(Nat)
  cons(_ 1n, cons(_ 2n, cons(_ 3n, nil(_))))
```

Notice we used holes to avoid having to write `Nat` repeatedly. Since lists
are so common, it is part of the Base library, and there is a built-in
syntax-sugar for them:

```haskell
import Base#

main : List(Nat)
  [1n, 2n, 3n]
```

Or, if the type can be inferred:

```haskell
import Base#

main : List(Nat)
  [1n, 2n, 3n]
```

Indices
-------

Indices are like polymorphic variables, except that, rather than constant
types, they are **computed values** that can depend on each constructor's
arguments. That gives us a lot of type-level power and is one of the reasons
Formality is a great proof language. For example:

```haskell
import Base#

T IsEven (x : Nat)
| make_even(half : Nat) : IsEven(mul(2n, half))
```

This datatype has one index, `n`, of type `Nat`. Its constructor, `is_even`,
has one field, `half : Nat`. When you write `make_even(3n)`, the number `3n`
is multiplied by two and moved to the type-level, resulting in a value of type
`IsEven(6n)`. This makes it impossible to create a value of type `IsEven(5n)`,
because you'd need a `n` such that `mul(2n, n)` is `5n`, but that's impossible.
To visualize this, see the code below:

```haskell
even_0n : IsEven(0n)
  make_even(0n)

even_2n : IsEven(2n)
  make_even(1n)

even_4n : IsEven(4n)
  make_even(2n)

even_6n : IsEven(6n)
  make_even(3n)
```

This allows us, for example, to create a `half` function that can only receive
even values:

```haskell
div2(n : Nat) : Nat
  case n
  | zero => zero
  | succ => case n.pred as np
    | zero => zero
    | succ => succ(div2(np.pred))
    : Nat
  : Nat

half(n : Nat, is_even : IsEven(n);) : Nat
  div2(n)
```

You can't call `half` on odd values because you can't construct an `IsEven` for
them. An good exercise might be to implement a proof of `Equal(Nat,
double(half(n, ie;)), n)`. Why this is possible for `half`, but not for `div2`?

Another example is the Vector, which is a `List` with a statically known length.
Every time you add an element to a Vector, the length on its type increases:

```haskell
import Base#

T Vector<A> (len: -Nat)
| vnil : Vector(A, zero)
| vcons(len; head: A, tail: Vector(A, len)) : Vector(A, succ(len))

main : Vector(String, 3n)
   vcons(__ "A", vcons(__ "B", vcons(__ "C", vnil(_))))
```

This has many applications such as creating a type-safe `vhead` function that
can't be called on non-empty vectors.

As the last example, this defines a list with all elements being true:

```haskell
import Base#

T AllTrue (xs : List(Bool))
| at_nil                                      : AllTrue(nil(Bool;))
| al_cons (xs : List(Bool), tt : AllTrue(xs)) : AllTrue(cons(Bool; true, xs))
```

It says that the empty list is a list of trues (`at_nil`) and that appending
true to a list of trues is still a list of trues (`at_cons`). It is impossible
to create a `AllTrue(list)` with a false element. You can make all sorts of
specifications using indexed datatypes, making them extremely powerful.

Motive
------

Until now, we've almost always omitted the type annotation of case expressions;
that is, its motive. But motives are important when you want the return type of
a case expression to depend on the matched value. For example:

```haskell
import Base#

CaseType(x : Bool) : Type
  case x
  | true  => Nat
  | false => String

main : Nat
  case true as x
  | true  => 42n
  | false => "hello"
  : CaseType(x)
```

The way this works is that, in order to check if a `case` expression is
well-typed, Formality first specializes the return type for the specific value
of each branch. For example, on the `true` case, `CaseType(true)` returns
`Nat`, so we can write `42`. On the `false` case, `CaseType(false)` returns
`String`, so we can write `"hello"`. Finally, the whole expression returns
`CaseType(x)`, which, in this case, is `Nat`, because `x` is `true`.

In other words, this means that, if we prove a theorem for every specific
possible value of `x`, then this theorem holds for `x` itself. This mechanism
is the heart of theorem proving, and understanding it well is essential to
develop mathematical proofs in Formality. Let's go through some examples.

#### Example: proving equalities

Formality's base libraries include a type for equality
proofs called `Equal`.  For example, `Equal(Nat, <2n>, <2n>)`
is the statement that `2` is equal `2`. It is not a proof:
you can write `Equal(Nat, <2n>, <3n>)`, which is just the
**statement** that `2` is equal to `3`.  To prove an
equality, you must use `equal(A; x;)`, which, for any `x :
A`, proves `Equal(A, <x>, <x>)`. In other words, `equal` is a
proof that every value is equal to itself. As such, we can
prove that `true` is equal to `true` like this:

```haskell
true_is_true : Equal(Bool, <true>, <true>)
  equal(Bool; <true>;)
```

Note that holes can often be used to avoid writing the arguments of `equal`.
Moreover, Formality includes a syntax sugar for `Equal`, `a == b`, which
expands to `Equal(?, <a>, <b>)`. As such, the program above can be written as:

```haskell
true_is_true : true == true
  equal(__)
```

A natural question is: if the only way to prove an equality is by using `equal`,
which proves that an element is equal to itself, then what is the point? The
insight is that, with motives, we can prove that two expressions are equal even
when they're not literally identical. For example, suppose you want to prove
that, for any boolean `b`, `not(not(b))` is equal to `b`. This can be stated as
such:

```haskell
import Base#

not_not_is_same(b : Bool) : Equal(Bool, not(not(b)), b)
  ?a
```

But here you can't use `equal(Bool; <b>;)`, because that'd be a proof of
`Equal(Bool, <b>, <b>)`, not of `Equal(Bool, <not(not(b))>, <b>)`. The sides aren't
identical! The problem is that the function call is stuck on a variable, `b`,
causing both sides to be different. That's when dependent motives help: if you
pattern-match on `b`, Formality will specialize the equation for both specific
values of `b`, that is, `true` and `false`:

```haskell
import Base#

not_not_is_same(b : Bool) : Equal(Bool, <not(not(b))>, <b>)
  case b
  | true  => ?a
  | false => ?b
  : Equal(Bool, <not(not(b))>, <b>)
```

So, on the `true` case, it asks you to prove `Equal(Bool, <not(not(true))>,
<true>)`, because `b` was specialized to true on that branch. Since
`not(not(true))` reduces to `true`, you only need to prove `Equal(Bool, <true>,
<true>)`, which can be done with `equal`. The same holds for the `false` case.
Once you prove the theorem for both possible cases of `b`, then Formality
returns the motive generalized for `b` itself:

```haskell
import Base#

not_not_is_same(b : Bool) : Equal(Bool, <not(not(b))>, <b>)
  case b
  | true  => equal(Bool; <true>;)
  | false => equal(Bool; <false>;)
  : Equal(Bool, <not(not(b))>, <b>)
```

This proof wouldn't be possible without using `b` on the motive. With syntax
sugars, it can be written as:

```haskell
main(b : Bool) : not(not(b)) == b
  case b
  | true  => equal(__)
  | false => equal(__)
  : not(not(b)) == b
```

#### Example: proving absurds

Another interesting example comes from the `Empty` datatype:

```haskell
T Empty
```

This code isn't incomplete, the datatype has zero constructors. This means it
is impossible to construct a term `t` with type `Empty`, but we can still
accept it as a function argument. So, what happens if we pattern match against
it?

```haskell
import Base#

wtf(e : Empty) : ?
  case e : ?
```

Simple: we can replace `?` with anything, and the program will check. That's
because, technically, we proved all the cases, so the case expression just
returns the motive directly, allowing us to write anything on it! That can be
used to derive any theorem given a value of type Empty:

```haskell
import Base#

one_is_two(e : Empty) : Equal(Nat, <1n>, <2n>)
  case e : Equal(Nat, <1n>, <2n>)
```

In other words, if we managed to call `one_is_two`, we'd have a proof that `1`
is equal to `2`. Of course, we can't call it, because there is no way to
construct a value of type `Empty`. Interestingly, the opposite holds too: given
a proof that `1` is equal to `2`, we can make an element of type `Empty`. Can
you prove this?

An interesting thing about `Empty` is that it can be used to prove that a
theorem is false by implementing a function that receives it as the input and
uses it to derive `Empty`. Our base libraries export many functions to help
you with that. For example:

- [`absurd`](https://github.com/moonad/Formality-Base/blob/master/Empty.fm):
  allows you to prove any theorem from an `e : Empty`.

- [`true_isnt_false`](https://github.com/moonad/Formality-Base/blob/master/Bool.fm):
  allows you to turn a proof that `true` is `false` into a proof of `Empty`.

- [`apply`](https://github.com/moonad/Formality-Base/blob/master/Equal.fm):
  allows you to apply a function to both sides of an equation.

- [`rewrite`](https://github.com/moonad/Formality-Base/blob/master/Equal.fm):
  allows you to rewrite provably equal expressions inside types.


Those building blocks allow you to prove more complicate absurdities  by
manipulating equations with higher level functions. For example, here is a
proof that `"dogs"` and `"horses"` are different strings:

```haskell
import Base#

main : "dogs" != "horses"
  (input) =>
  let e0 = input                         -- "dogs" == "horses"
  let e1 = apply(____ length(_); e0)     -- n4 == n6
  let e2 = apply(____ nat_equal(4n); e1) -- true == false
  let e3 = true_isnt_false(e2)           -- Empty
  e3
```

#### Example: avoiding unreachable branches

Notice the program below:

```haskell
import Base#

main : Nat
  case true
  | true  => 10n
  | false => ?
```

Here, we're matching against `true`, so we know the `false` case is
unreachable, but we still need to fill it with some number. With motives, we
can avoid that. The idea is that we will send, to each branch, a proof that
`Equal(Bool, x, true)`. On the `true` branch, it will be specialized to
`Equal(Bool, true, true)`, which is useless... but, on the `false` branch, it
will be specialized to `Equal(Bool, false, true)`. We can then use
`false_isnt_true` and `absurd` to fill the branch without ever providing a
number:

```haskell
import Base#

main : Nat
  case true as x
  + refl(__) as e : Equal(Bool, x, true)
  | true  => 10n
  | false => absurd(false_isnt_true(e), _)
```

(Remember that `+` is a shorthand for adding an extra variable to the motive.)

Of course, in this example we could just have written any number, but, in some
cases, such as on the `nil` branch of a `head` function, we simply don't have
a value to insert. In those cases, finding a way to construct `Empty` on that
branch is equivalent to saying it is unreachable. As such, `absurd` can be used
to "skip" it.

Encoding
--------

Interestingly, none of the features above are part of Formality's type theory.
Instead, they are lightweight syntax-sugars that elaborate to plain-old
lambdas. To be specific, a datatype is encoded as is own inductive hypothesis,
with "Self Types". For example, the `Bool` datatype desugars to:

```haskell
Bool : Type
  ${self} (
    P     : Bool -> Type;
    true  : P(true),
    false : P(false)
  ) -> P(self)

true : Bool
  new(Bool) (P; true, false) => true

false : Bool
  new(Bool) (P; true, false) => false

case_of(b : Bool, P : Bool -> Type; t : P(true), f : P(false)) : P(b)
  use(b)(P; t, f)
```

Here, `${self} ...`, `new(T) val` and `use(b)` are the type, introduction, and
elimination of Self Types, respectively. You can see how any datatype is
encoded under the hoods by asking `fm` to evaluate its type, as in, `fm
Data.Bool#/Bool -W` (inside the `fm_modules` directory). The `-W` flag asks
Formality to not evaluate fully since `Bool` is recursive. While you probably
won't need to deal with self-encodings yourself, knowing how they work is
valuable, since it allows you to express types not covered by the built-in
syntax.

Proofs
======

Types in Formality can be used to express mathematical theorems. This allows us
to statically prove invariants about our programs, making them flawless. In a
way, proofs can be seen as a generalization of tests. With tests, we can assert
that specific expressions have the values we expect. For example, consider the
program below:

```javascript
// Recursively doubles a number
function mul2(n) {
  if (n <= 0) {
    return 0;
  } else {
    return 2 + mul2(n - 1.01);
  }
}

// Tests
it("Works for 10", () => {
  console.assert(mul2(10) === 20);
})


it("Works for 44", () => {
  console.assert(mul2(44) === 88);
});

it("Works for 17", () => {
  console.assert(mul2(17) === 34);
});

it("Works for 12", () => {
  console.assert(mul2(12) === 24);
});

it("Works for 20", () => {
  console.assert(mul2(20) === 40);
});
```

It allows us to test our implementation of `mul2` by checking if the invariant
that `n * 2` is equal to `n + n` holds for specific values of `n`. The problem
with this approach is that it only gives us partial confidence. No matter how
many tests we write, there could be still some input which causes our function
to misbehave. In this example, `mul2(200)` returns `398`, which is different
from `200 + 200`. The the implementation is incorrect, despite all tests
passing!


With formal proofs, we can write tests too:

```haskell
import Base#

mul2(n : Number) : Number
  if n .<. 0 .|. n .==. 0 then
    0
  else
    2 .+. mul2(n .-. 1.01)

it_works_for_10 : mul2(10) == 20
  refl(__)

it_works_for_44 : mul2(44) == 88
  refl(__)

it_works_for_17 : mul2(17) == 34
  refl(__)

it_works_for_12 : mul2(12) == 24
  refl(__)

it_works_for_20 : mul2(20) == 40
  refl(__)
```

> TODO: this section is outdated after the removal of Numbers; update

Here, we're using `==` to make an assertion that `mul2(10)`
is equal to `20` and so on. Since those are true by
reduction, we can complete the proofs with `refl`. This
essentially implements a type-level test suite. But with
proofs, we can go further: we can prove that a general
property holds for every possible input, not just a few. To
explain, let's first re-implement `mul2` for `Nat`, which
will posteriorly allows us to write a `case` with a
`motive`, an essential proof technique:

```haskell
import Base#

mul2(n : Nat) : Nat
  case n
  | zero => zero
  | succ => succ(succ(mul2(n.pred)))
```

Let's start by adding a few tests just to be sure:

```haskell
it_works_for_0 : mul2(0n) == 0n
  refl(__)

it_works_for_1 : mul2(1n) == 2n
  refl(__)

it_works_for_2 : mul2(2n) == 4n
  refl(__)
```

Since those pass, we can now try to prove the more general statement that the
double of any `n` is equal to `x .+. x`. We start by writing the type of our
invariant:

```haskell
it_works_for_all_n(n : Nat) : mul2(n) == add(n, n)
  ?a
```

As you can see, this is just like a test, except that it includes a variable,
`n`, which can be any `Nat`. As such, it will only "pass" if we manage to
convince Formality that `mul2(n) == add(n, n)` holds for every `n`,
not just a few. To do it, we can start by type-checking the program above and
seeing what Formality has to say:

```haskell
Found hole: 'a'.
- With goal... Equal(Nat, mul2(n), add(n, n))
- With context:
- n : Nat

(n : Nat) -> Equal(Nat, mul2(n), add(n, n)) ✔
```

This is telling us that our theorem is correct as long as we can replace the
hole `?a` with a proof that `mul2(n) == add(n, n)`. In other words,
Formality is asking us to to prove what we claimed to be true. Let's try to do
it with a `refl`:

```haskell
it_works_for_all_n(n : Nat) : mul2(n) == add(n, n)
  refl(__)
```

This time, it doesn't work, and we get the following error:

```haskell
Type mismatch.
- Found type... Equal(Nat, add(n, n), add(n, n))
- Instead of... Equal(Nat, mul2(n), add(n, n))
```

That's because `refl` proves that a value is equal to itself, but `mul2(n)` and
`add(n, n)` are different expressions. The problem is that, unlike on the
previous tests, the equation now has a variable, `n`, which causes both sides to
get "stuck", so they don't become identical by mere reduction. We need to
"unstuck" the equation by inspecting the value of `n` with a case expression.

```haskell
it_works_for_all_n(n : Nat) : mul2(n) == add(n, n)
  case n
  | zero => ?a
  | succ => ?b
  : mul2(n) == add(n, n)
```

Let's type-check it again:

```haskell
Found hole: 'a'.
- With goal... Equal(Nat, mul2(zero), add(zero, zero))
- Couldn't find a solution.
- With context:
- n : Nat

Found hole: 'b'.
- With goal... Equal(Nat, mul2(succ(n.pred)), add(succ(n.pred), succ(n.pred)))
- Couldn't find a solution.
- With context:
- n      : Nat
- n.pred : Nat
```

Notice that, now, we have two holes, one for each possible value of `n` (`zero`
or `succ(n.pred)`). The first hole is now asking a proof that `Equal(Nat,
mul2(zero), add(zero, zero))`.  See how it was specialized to the value of `n`
on the branch?  That's very important, because now both sides evaluate to
`zero`.  This allows us to prove that case with a `refl`!

```haskell
it_works_for_all_n(n : Nat) : mul2(n) == add(n, n)
  case n
  | zero => refl(__)
  | succ => ?b
  : mul2(n) == add(n, n)
```

Now, we only have one hole:

```haskell
Found hole: 'b'.
- With goal... Equal(Nat, mul2(succ(n.pred)), add(succ(n.pred), succ(n.pred)))
- Couldn't find a solution.
- With context:
- n      : Nat
- n.pred : Nat
```

This demands a proof that `Equal(Nat, mul2(succ(n.pred)), add(succ(n.pred),
succ(n.pred)))`. Due to the way `mul2` and `add` are defined, this reduces to
`Equal(Nat, succ(succ(mul2(n.pred))), succ(succ(add(n.pred, succ(n.pred)))))`,
so that's what we need to prove. This looks... complex. But, since we're in a
recursive branch, there is a cool thing we can do: call the function
recursively to the predecessor of `n`. That's the mathematical equivalent of
applying the inductive hypothesis, and it is almost always the key to proving
theorems about recursive datatypes like `Nat`. Let's do that and `log` it to
see what we get:

```haskell
it_works_for_all_n(n : Nat) : mul2(n) == add(n, n)
  case n
  | zero => refl(__)
  | succ =>
    let ind_hyp = it_works_for_all_n(n.pred)
    log(ind_hyp)
    ?a
  : mul2(n) == add(n, n)
```

This outputs:

```haskell
[LOG]
Term: it_works_for_all_n(n.pred)
Type: Equal(Nat, mul2(n.pred), add(n.pred, n.pred))
```

In other words, we gained "for free" a proof, `ind_hyp`, that `Equal(Nat,
mul2(n.pred), add(n.pred, n.pred))`! Now, remember that our goal is `Equal(Nat,
succ(succ(mul2(n.pred))), succ(succ(add(n.pred, succ(n.pred)))))`. Take a
moment to observe that, to turn `ind_hyp` into our goal, all we need to do is
add `succ(succ(...))` to both sides of the equation. This can be done with the
`cong` function from the Base library. That function accepts an equality and a
function, and applies the function to both sides of the equality. Like this:

```haskell
it_works_for_all_n(n : Nat) : mul2(n) == add(n, n)
  case n
  | zero => refl(__)
  | succ =>
    let ind_hyp = it_works_for_all_n(n.pred)
    let add_two = (x : Nat) => succ(succ(x))
    let new_hyp = cong(____ add_two; ind_hyp)
    log(new_hyp)
    ?a
  : mul2(n) == add(n, n)
```

Let's log `new_hyp` to see what we have now:

```haskell
[LOG]
Term: cong(?_a/line24_11; ?_a/line24_12; ?_a/line24_13; ?_a/line24_14; (x : Nat) => succ(succ(x)); it_works_for_all_n(n.pred))
Type: Equal(Nat, succ(succ(mul2(n.pred))), succ(succ(add(n.pred, n.pred))))
```

As you can see, `new_hyp` has the same type as our goal! As such, we can
complete this branch with it:

```haskell
it_works_for_all_n(n : Nat) : Equal(Nat, mul2(n), add(n, n))
  case n
  | zero => refl(__)
  | succ =>
    let ind_hyp = it_works_for_all_n(n.pred)
    let add_two = (x : Nat) => succ(succ(x))
    let new_hyp = cong(____ add_two; ind_hyp)
    new_hyp
  : mul2(n) == add(n, n)
```

And done! Since we've proven our theorem for both possible values of `n`
(`zero` or `succ(n.pred)`), then we've proven it for every `n`. And that's how
most proofs are done. Proving an equation is often just a game of "opening"
variables with `case` expressions so that the sides gets unstuck until we're
able to finish the proof with a `refl`, or with an inductive hypothesis.

The cool thing is, since our program passes the type-checker now, that means
we've proven that `2 * n` is equal to `n + n` for all `n`. This is an extreme
validation that our implementation of `mul2` is correct: it is as if we've
written infinite tests, one for each `n`. Not only that, but the fact that
`mul2` and `add` match perfectly despite being very different functions
reinforces the correctness of them both, mutually. Here is the complete file
with our implementation and its correctness proof:

```haskell
import Base#

-- Multiplies a number by two
mul2(n : Nat) : Nat
  case n
  | zero => zero
  | succ => succ(succ(mul2(n.pred)))

-- Tests that it works as expected for `n = 0`
it_works_for_0 : mul2(0n) == 0n
  refl(__)

-- Tests that it works as expected for `n = 2`
it_works_for_1 : mul2(1n) == 2n
  refl(__)

-- Tests that it works as expected for `n = 4`
it_works_for_2 : mul2(2n) == 4n
  refl(__)

-- Proves that it works as expected for any `n` up to infinity
-- using symbolic manipulation and inductive reasoning
it_works_for_all_n(n : Nat) : mul2(n) == add(n, n)
  case n
  | zero => refl(__)
  | succ =>
    let ind_hyp = it_works_for_all_n(n.pred)
    let add_two = (x : Nat) => succ(succ(x))
    let new_hyp = cong(____ add_two; ind_hyp)
    new_hyp
  : mul2(n) == add(n, n)
```

An interesting point to note is that proofs are often much longer than
theorems. In this example, the theorem had just one line, but the proof had 8.
Proofs are laborious to write and require a set of advanced programming skills.
But, once they're done, they're undeniably correct. This is extremely
valuable. For example, think of a huge smart-contract: its code could be big
and complex, but, as long as its developers publish proofs of a few essential
properties, users can trust it won't go wrong. In a way, proofs can be seen as
trustless correctness assets, in the sense, you can use them to convince people
that your code is correct without needing them to trust you.

Theory
======

Formality is a dependently typed programming language based on extrinsic type
theory. It is based on elementary terms are just annotated versions of the
Elementary Affine Calculus (EAC), which is itself a terminating subset
of the lambda-calculus with great computational characteristics. In particular,
EAC is compatible with the most efficient version of the optimal reduction
algorithm [citation]. This is important because, while asymptotically optimal,
most implementations of the algorithm carry an extra, significant constant
overhead caused by a book-keeping machinery. This made them too slow in
practice, decreasing interest in the area. By relying on EAC, we can avoid the
book-keeping overhead, making it much faster than all alternatives. Below is a
table comparing the number of native operations (graph rewrites) required to
reduce λ-terms in other implementations and Formality:

 Term | GeomOpt | GeomImpl | YALE | IntComb | Formality
  --- |     --- |      --- |  --- |     --- |       ---
 22II |     204 |       56 |   38 |      66 |        21
222II |     789 |      304 |  127 |     278 |        50
  3II |      75 |       17 |   17 |      32 |         9
 33II |     649 |      332 |   87 |     322 |        49
322II |    7055 |     4457 |  383 |    3268 |        85
223II |    1750 |     1046 |  213 |     869 |        69
 44II |    3456 |     2816 |  148 |    2447 |        89

Not only Formality requires orders of magnitude less native operations, but its
native operations are much simpler.

Unlike most proof languages, Formality doesn't include a datatype system, since
not only it is very complex to implement, which goes against our second goal
(simplicity), but can also cause more serious issues. For example, it was
discovered that type preservation does not hold in Coq due to its treatment of
coinductive datatypes [citation], and both Coq and Agda were found to be
incompatible with Homotopy Type Theory due to the K-axiom, used by native
pattern-matching on dependent datatypes [citation]. Instead, Formality relies
on lambda encodings, an alternative to a datatype system that uses plain lambdas
to encode arbitrary data. While attractive, this approach wasn't adopted in
practical type theories for several reasons:

1. There is no encoding that provides both recursion and fast pattern-matching.

    - Church encodings provide recursion, but pattern-matching is slow (`O(n)`).

    - Scott encodings provide fast pattern-matching, but no recursion.

    - Alternative encodings that provide both consume too much memory.

2. We cannot prove `0 != 1` with the usual definition of `!=`.

3. Induction is not derivable.

Formality solves the first issue by simply using Church encodings for recursion
and Scott encodings for data, the second issue by collapsing universes, and the
last issue by combining self-types and type-level recursion [citation]. It is
well-known that, in most proof languages, collapsing universes and type-level
recursion can lead to logical paradoxes, but the remarkable coincidence is that
the same feature that allows the language to be compatible with optimal
reductions, Elementary Affine Logic, also makes its type-theory sound in the
presence of those [proof]. This allows the theory to be remarkably simple and
powerful.

Formality Core
--------------

Formality Core (FM-Core) can be seen as a distillation of FM-Lang with just the
most essential primitives, excluding aspects such as datatypes, which are
implemented with more primitive features such as lambdas and self. Its syntax
is defined as follows:

```
name ::=
  <any alphanumeric string>

term ::=
  -- Lambdas
  (name : term) -> term -- lambda type (dependent product)
  (name : term) => term -- lambda term
  term(term)            -- lambda application

  -- Boxes
  ! term                -- boxed type
  # term                -- boxed term
  dup name = term; term -- boxed duplication

  -- Selfies
  ${name} term           -- self type
  new(term) term         -- self term
  use(term)              -- self elimination

  -- Language
  name                   -- variable
  Type                   -- universe

file ::=
  name term : term; file -- top-level definition
  \eof                   -- end-of-file
```

Note that the actual implementation of FM-Core includes numbers and pairs for
efficiency reasons, but they'll be omitted from this documentation for the sake
of simplicity. Also, we'll use the Formality notation and write `(x0 : A0, x1,
A1, ...) => t` as a synonym for `(x0 : A0) => (x1 : A1) => ...  => t`, and
`f(x,y,z)` as a synonym for `f(x)(y)(z)`. When `x` isn't used in `B`, we'll
write `(x : A) -> B` as `A -> B` instead. We will also sometimes omit types of
lambda-bound arguments when they can be inferred and write `(x0) => t` instead.
Our typing rules are:

```
-- Lambdas

Γ |- A : Type    Γ, x : A |- B : Type
------------------------------------- lambda type
Γ |- (x : A) -> B : Type

Γ |- A : Type    Γ, x : A |- t : B
---------------------------------- lambda term
Γ |- (x : A) => t : (x : A) -> B

Γ |- f : (x : A) -> B    Γ |- a : A
----------------------------------- lambda application
Γ |- f(a) : B[x <- a]

-- Boxes

Γ |- A : Type
-------------- boxed type
Γ |- !A : Type

Γ |- t : A
------------ boxed term
Γ |- #A : !A

Γ |- t : !A    Γ, x : A |- u : B
-------------------------------- boxed duplication
Γ |- dup x = t; u : B

-- Selfies

Γ, s : ${x} A |- A : Type
----------------------- self type
Γ |- ${x} A : Type

Γ, |- t : A[x <- t]    Γ |- ${x} A : Type
----------------------------------------- self term
Γ |- new(${x} A) t : ${x} A

Γ |- t : ${x} A
----------------------- self elimination
Γ |- use(t) : A[x <- t]

-- Language

(x : T) ∈ Γ
----------- variable
Γ |- x : T

Ø
---------------- type-in-type
Γ |- Type : Type
```

**Theorem: induction principle.**

Induction can be proved on FM-Core as follows:

```
Nat : Type
  ${self}
  ( P    : Nat -> Type
  , succ : (r : Nat, i : P(r)) -> P(succ(r))
  , zero : P(zero)
  ) -> P(self)

succ : (n : Nat) -> Nat
  new(Nat) (P, succ, zero) => succ(n, use(n)(P, succ, zero))

zero : Nat
  new(Nat) (P, succ, zero) => zero

nat_ind : (n : Nat, P : Nat -> Type, s : (n : Nat, i : P(n)) -> P(succ(n)), z : P(zero)) -> P(n)
  (n) => use(n)
```

This defines an inductive lambda encoded datatype, Nat. The key to understand
why this works is to realize that `Nat` is defined as its own induction scheme.
Mutual recursion is needed because the induction scheme of any datatype refers
to its constructors, which then refer to the defined datatype. Self types are
used because the type returned by the induction scheme can refer to the term
being induced. Together, those two components allow us to define inductive
datatypes with just plain lambdas, and to induce on `n : Nat` you don't require
any additional proof, all you need to do is eliminate its self-type with `use(n)`.
Note that this `Nat` is complicated by the fact it is recursive (Church
encoded). We could have a non-recursive (Scott encoded) `Nat` as:

```javascript
Nat Type
  ${self} (
    P    : Nat -> Type;
    succ : (n : Nat) -> P(succ(n)),
    zero : P(zero)
  ) -> P(self)

succ(n : Nat) -> Nat
  new(Nat) (P; succ, zero) => succ(n)

zero Nat
  new(Nat) (P; succ, zero) => zero

nat_ind(n : Nat, P : Nat -> Type; s : (n : Nat, i : P(n)) -> P(succ(n)), z : P(zero)) -> P(n)
  (n) => use(n)
```

Which is simpler and has the benefit of having constant-time pattern-matching.
Even simpler types include Bool:

```
Bool Type
  ${self} (
    P     : Bool -> Type;
    true  : P(true),
    false : P(false)
  ) -> P(self)

true Bool
  new(Bool) (P; true, false) => true

false Bool
  new(Bool) (P; true, false) => false

bool_induction(b : Bool, P : Bool -> Type; t : P(true), f : P(false)) -> P(b)
  (b) => use(b)
```

Unit:

```
Unit Type
  ${self}
  ( P     : Unit -> Type;
    unit  : P(unit)
  ) -> P(self)

unit Unit
  new(Unit) (P; unit) => unit

unit_induction(b : Unit, P : Unit -> Type; u : P(unit)) -> P(u)
  (u) => use(u)
```

And Empty:

```
Empty Type
  ${self} (
    P : Empty -> Type;
  ) -> P(self)

empty_induction(b : Empty, P : Empty -> Type;) -> P(b)
  (e) => use(e)
```

Notice how, for example, `bool_induction` is equivalent to the elimination
principle of the Bool datatype on Coq or Agda, except that this was obtained
without needing a complex datatype system to be implemented as a native
language feature. This type theory is capable of expressing any inductive
datatype seen on traditional proof languages. For example, Vector (a List with
statistically known length) can, too, be encoded as its dependent elimination:

```
Vector(A : Type; len : Nat) -> Type
  ${self} (
    P     : (len : Nat) -> Vector(A, len) -> Type;
    vcons : (len : Nat; x : A, xs : Vector(A, len)) -> P(succ(len), vcons(A; len; x, xs)),
    vnil  : P(zero, vnil(A;)),
  ) -> P(len, self)

vcons(A : Type; len : Nat; head : A, tail : Vector(A, len)) -> Vector(A, succ(len))
  new(Vector(A, succ(len))) (P; vcons, vnil) => vcons(len; head, tail)

vnil(A : Type;) -> Vector(A, zero)
  new(Vector(A, zero)) (P; vcons, vnil) => vnil
```

And an identity type for propositional equality is just the J axiom wrapped by
a self type:

```
Id(A : Type, a : A, b : A) -> Type
  ${self} (
    P    : (b : A, eq : Id(A, a, b)) -> Type;
    refl : P(a, refl(A; a;))
  ) -> P(b, self)

refl(A : Type; a : A;) -> Id(A, a, a)
  new(Id(A, a, a)) (P; refl) => refl
```

We're also able to express datatypes that traditional proof languages can't.
For example, intervals, i.e., booleans that can only be eliminated if both
provided cases are equal, can be written as:

```
Interval Type
  ${self} (
    P  : (i : Interval) -> Type;
    I0 : P(i0),
    I1 : P(i1),
    SG : I0 == I1
  ) -> P(self)

i0 Interval
  new(Interval) (P; i0, i1, sg) => i0

i1 Interval
  new(Interval) (P; i0, i1, sg) => i1
```

And we can easily prove troublesome theorems like `0 != 1`: 

```
true_isnt_false(e : Id(Bool, true, false)) -> Empty
  use(e)((b, e.b) => use(e.b)((b) => Type; Unit, Empty); unit)
```

That much type-level power comes at a cost: if Formality was based on the
conventional lambda calculus, it wouldn't be sound. Since it is based on EAC,
a terminating untyped language, its normalization is completely independent
of types, so it is impossible to derive paradoxes by exploiting
non-terminating computations. A good exercise is attempting to write `λx. (x x)
λx. (x x)` on EAC, which is impossible. The tradeoff is that EAC is
considerably less computationally powerful than the lambda calculus, imposing
a severe restriction on how terms can be duplicated. This limitation isn't very
problematic in practice since the language is still capable of implementing any
algorithm a normal programming language, but doing so involves a resource-usage
discipline that can be annoying to users. Formality is currently being
implemented in Agda, on which the consistency argument will be formalized.

TODO: complete with consistency proofs (being developed on the
[Formality-Agda](https://github.com/moonad/formality-agda) repository).


Formality Calculus
------------------

The Formality Calculus (FM-Calc) is a further distillation of FM-Core, without
types. It is similar to the λ-calculus, but it is both terminating and
compatible with the book-keeping free optimal reduction algorithm used by
our runtime.

```
name ::=
  <any alphanumeric string>


term ::=
  -- Lambdas
  (name) => term        -- lambda term
  term(term)            -- lambda application

  -- Boxes
  # term                -- boxed term
  dup name = term; term -- boxed duplication

  -- Language
  name                  -- variable
```

The actual implementation of FM-Calc also features numbers and pairs, which are
omitted here for simplicity.  The first 3 constructors are the usual
λ-calculus. The last two introduce boxes, which just wrap terms, and
duplications, which copies the contents of a box. The FM-Calc has the following
reduction rules:

```
1. application: ((x) => a)(b) ~> [b/x]a
2. duplication: dup x = #a; b ~> [a/x]b
3. appdup-swap: ((dup x = a; b) c) ~> dup x = a; (b c)
4. dupdup-swap: dup x = (dup y = a; b); c ~> dup y = a; dup x = b; c
```

The first rule is the usual beta-reduction. The second rule allows us to copy
the contents of a box. The last two are just permutations, allowing inner
`dup`s to be exposed. An FM-Calc term is said to be stratified when it follows the
following conditions:

1. Lambda-bound variables can only be used at most once.

2. There must be exactly 0 boxes between a variable bound by a lambda and its occurrence.

3. There must be exactly 1 box between a variable bound by a duplication and its occurrence.

The first condition says that lambdas must be affine. The second condition says
that lambdas can't substitute inside boxes. The third condition says that you
can't unbox, or add boxes, to a term by duplicating it. Stratified FM-Calc terms
are strongly normalizing. Proof: 

- Define the level of a term as the number of boxes surrounding it.

- Define the number of redex in a level by counting its reducible apps/dups.

- Prove that level is reduction-invariant.

- Prove that applications reduce the number of redexes in a level.

- Prove that duplications reduce the number of redexes in a level.

- Prove that reduction eventually consumes all redexes on level 0.

- Prove that, if level N is normalized, reduction eventually consumes all redexes on level N+1. 

TODO: include the formalization of this proof (being developed on the
[Formality-Agda](https://github.com/moonad/formality-agda) repository).

TODO: mention impressive performance results obtained by runtime fusion:

- https://medium.com/@maiavictor/solving-the-mystery-behind-abstract-algorithms-magical-optimizations-144225164b07

- https://medium.com/@maiavictor/calling-a-function-a-googol-times-53933c072e3a

Formality Net
-------------

Formality terms are compiled to a memory-efficient interaction net system,
FM-Net. Interaction nets are just graphs where nodes have labeled ports, one
being the main one, plus a list of "rewrite rules" that are activated whenever
two nodes are connected by their main ports. Our system includes 6 types of
nodes, ERA, CON, OP1, OP2, ITE, NUM.

![](https://raw.githubusercontent.com/moonad/Assets/master/images/fm-net-node-types.png)

- `CON` has 3 ports and an integer label. It is used to represent lambdas,
  applications, boxes (implicitly) and duplications. Since FM-Core is based on
  EAL, there is no book-keeping machinery to keep track of Bruijn indices, just
  `CON` is enough for beta-reduction.

- `ERA` has 1 port and is used to free empty memory, which happens when a
  function that doesn't use its bound variable is applied to an argument.

- `NUM` has 1 port and stores an integer and is used to represent native
  numbers.

- `OP1` has 2 ports and stores one integer and an operation id. `OP2` has 3
  ports and an operation id. They are used for numeric operations such as
  addition and multiplication.

- `ITE` has 3 ports and an integer label. It is used for if-then-else and is
  required to enable number-based branching.

Note that the position of the port matters. The port on top is called the `main`
port. The first port counter-clockwise to the main port (i.e., to the left on
this drawing) is the `aux0` port, and the first port clockwise to the main port
(i.e., to the right on this drawing) is the `aux1` port.

#### Rewrite rules

In order to perform computations, FM-Net has a set of rewrite rules that are
triggered whenever two nodes are connected by their main ports. This is an
extensive list of those rules:

![](https://raw.githubusercontent.com/moonad/Assets/master/images/fm-net-rewrite-rules.png)

Note that, while there are many rules (since we need to know what to do on each
combination of a node), most of those have the same "shape" (such as OP2-OP2,
ITE-ITE), so they can reuse the same code. There are only 5 actually relevant
rules:

#### Erasure

When an `ERA` or a `NUM` node collides with anything, it "destroys" the other
node, and propagates itself to destroy all nodes connected to it.

#### Substitution

When two `CON` nodes of equal label collide, and also on the `OP2-OP2` /
`ITE-ITE` cases, both nodes are destroyed, and their neighbors are connected.
That's the rule that performs beta-reduction because it allows connecting the
body of a lambda (which is represented with `CON`) to the argument of an
application (which is, too, represented with `CON`). Note that on the OP2-OP2
and ITE-ITE cases, that's just a default rule that doesn't matter, since those
cases can't happen on valid FM-Calc programs.

#### Duplication

When different nodes collide, they "pass-through" each other, duplicating
themselves in the process. This allows, for example, `CON` nodes with a label
`>1` to be used to perform deep copies of any term, with `dup x = val; ...`. It
can copy lambdas and applications because they are represented with `CON` nodes
with a label `0`, pairs and pair-accessors, because they are represented with
`CON` nodes with a label `1`, and `ITE`, `OP1`, `OP2`, because they are
different nodes.

It also allows duplications to duplicate terms that are partially duplicated
(i.e., which must duplicate, say, a λ-bound variable), as long as the `CON`
labels are different, otherwise, the `CON` nodes would instead fall in the
substitution case, destroying each other and connecting neighbors, which isn't
correct. That's why FMC's box system is necessary: to prevent concurrent
duplication processes to interfere with each other by ensuring that, whenever
you duplicate a term with `dup x = val; ...`, all the duplication `CON` nodes of
`val` will have labels higher than the one used by that `dup`.

#### If-Then-Else

When an `ITE` node collides with a `NUM` node, it becomes a `CON` node with one
of its ports connected to an `ERA` node. That's because then/else branches are
actually stored in a pair, and this allows you to select either the `fst` or the
`snd` value of that pair and discard the other branch.

#### Num-Operation

When `OP2` collides with a `NUM`, it becomes an `OP1` node and stores the number
inside it; i.e., the binary operation becomes a unary operation with `NUM`
partially applied. When that `OP1` collides with another `NUM`, then it performs
the binary operation on both operands, and return a new `NUM` with the result.
Those rules allow us to add, multiply, divide and so on native numbers.

#### Implementation

In our implementation, we use a buffer of 32-bit unsigned integers to represent
nodes, as follows:

- `CON`: represented by 4 consecutive uints. The first 3 represent the `main`,
  `aux0` and `aux1` ports. The last one represents the node type (3 bits),
  whether its ports are pointers or unboxed numbers (3 bits), and the label (26
  bits).

- `OP1`: represented by 4 consecutive uints. The first and third represent
  the `main` and `aux0` ports. The second represents the stored number. The
  last one represents the node type (2 bits), whether its ports are pointers
  or unboxed numbers (3 bits, 1 unused), and the operation (26 bits).

- `OP2`: represented by 4 consecutive uints. The first 3 represent the `main`,
  `aux0` and `aux1` ports. The last one represents the node type (3 bits),
  whether its ports are pointers or unboxed numbers (3 bits), and the operation
  (26 bits).

- `ITE`: represented by 4 consecutive uints. The first 3 represent the `main`,
  `aux0` and `aux1` ports. The last one represents the node type (3 bits),
  whether its ports are pointers or unboxed numbers (3 bits), and the label
  (26 bits).

- `ERA`: is stored inside other nodes and does not use any extra space. An `ERA`
  node is represented by a pointer port that points to itself. That's because
  `ERA`'s rewrite rules coincide with what we'd get if we allowed ports to point
  to themselves.

- `NUM`: is stored inside other nodes and does not use any extra space. A `NUM`
  node is represented by a numeric port. In order to know if a port is a number
  or a pointer, each node reserves 3 bits of its last uint to store that
  information.

#### Rewrites

TODO: explain how `rewrite` works, how `link_ports` is used, and why
`unlink_ports` is necessary to avoid invalid states.

#### Strict evaluation

The strict evaluation algorithm is very simple. First, we must keep a set of
redexes, i.e., nodes connected by their main ports. In order to do that,
whenever we link two main ports, we must add the address of the smallest nodes
to that set. We then perform a loop to rewrite all redexes. This will give us a
new set of redexes, which must then be reduced again, over and over, until there
are no redexes left. This is the pseudocode:

```python
while (len(net.redexes) > 0):
  for redex in net.redexes:
    net.rewrite(redex)
```

The strict reduction is interesting because it doesn't require graph walking nor
garbage collection passes, and because the inner `for`-loop can be performed in
parallel. That is, every `redex` in `net.redexes` can be rewritten at the same
time.

In order to do that, though, one must be cautious with intersection areas. For
example, in the graph below, B-C and D-E are redexes. If we reduce them in
parallel, both threads will attempt to read/write from C's and D's `aux0` and
`aux1` ports, potentially causing synchronization errors.

![](https://raw.githubusercontent.com/moonad/Assets/master/images/Absal/sk_problem_2x.png)

This can be avoided through locks, or by performing rewrites in two steps. On
the first step, each thread reads/writes the ports of its own active pair as
usual, except that, when it would need to connect a neighbor, it instead turns
its own node into a "redirector" which points to where the neighbor was supposed
to point. For example, substitution and duplication would be performed as
follows:

![](https://raw.githubusercontent.com/moonad/Assets/master/images/Absal/sk_local_rewrites_2x.png)

Notice that `P`, `Q`, `R` and `S` (neighbor ports) weren't touched: they keep
pointing to the same ports, but now those ports point to where they should point
to. Then, a second parallel step is performed. This time, we spawn a thread for
each neighbor port and walk through the graph until we find a non-redirector
node. We then point that neighbor to it. Here is a full example:

![](https://raw.githubusercontent.com/moonad/Assets/master/images/Absal/sk_local_rewrites_ex_2x.png)

Notice, for example, the port `C` of the node `A`. It is on the neighborhoods of
a redex (`B-C`), but isn't a redex itself. On the first step, two threads
rewrite the nodes `B-C` and `D-E`, turning them into redirectors, and without
touching that port. On the second step, a thread starts from port `C` of node
`A`, towards port `B` of node `B` (a redirector), towards port `C` of node `D`
(a redirector), towards port `B` of node ` F`. Since that isn't a redirector,
the thread will make `C` point to `B`. The same is done for each neighbor port
(in parallel), completing the parallel reduction.

#### Lazy evaluation

The lazy evaluation algorithm is very different from the strict one. It works by
traversing the graph, exploring it to find redexes that are "visible" on the
normal form of the term, skipping unnecessary branches. It is interesting
because it allows avoiding wasting work; for example, `({a b}b (F 42) 7)` would
quickly evaluate to `7`, no matter how long `(F 42)` takes to compute. In
exchange, it is "less parallel" than the strict algorithm (we can't reduce all
redexes since we don't know if they're necessary), and it requires global
garbage collection (since erasure nodes are ignored).

To skip unnecessary branches, we must walk through the graph from  port to port,
using a strategy very similar to the denotational semantics of symmetric
interaction combinators. First, we start walking from the root port to its
target port. Then, until we get back to the root, do as follows:

1. If we're walking from an aux port towards an aux port of a node, add the aux
   we're coming from to a stack, and move towards the main port of that node.

2. If we're walking from a main port to an auxiliary port, then we just found a
   node that is part of the normal form of the graph! If we're performing a
   weak-head normal form reduction, stop. Otherwise, start walking towards each
   auxiliary port (either recursively, or in parallel).

3. If we're walking towards the root, halt.

This is a rough pseudocode:

```python
def reduce_lazy(net, start):
  back = []
  prev = start
  next = net.enter(prev)

  while not(net.is_root(next)):
    if slot_of(prev) == 0 and slot_of(next) == 0:
      net.rewrite(prev, next)
    elif slot_of(next) == 0:
      for aux_n from 0 til net.aux_ports_of(next):
        net.reduce_lazy(Pointer(node_of(next), aux_n))
    else:
      back.push(prev)
      prev = Pointer(node_of(next), 0)
      next = net.enter(prev)
```

While the lazy algorithm is inherently sequential, there is still an opportunity
to explore parallelism whenever we find a node that is part of the normal form
of the graph (i.e., case `2`). In that case, we can spawn a thread to walk
towards each auxiliary port in parallel; i.e., the `for`-loop of the pseudocode
can be executed in parallel like the one on the strict version. This would allow
the algorithm to have many threads walking through the graph at the same time.
Again, caution must be taken to avoid conflicts.

Compilation
-----------

The process of compiling a Formality program to an interaction net can be
summarized as:

```
FM-Lang -> FM-Core -> FM-Calc -> FM-Net
```

That is, the user-facing textual syntax, FM-Lang, is desugared into a simple
type-theory, FM-Core, which is type-checked. If checking succeeds, types are
erased, turning it into FM-Calc, which is then compiled to FM-Net. The process
of compiling FM-Calc to FM-Net can be defined by the following function
`k_b(net)`:

![](https://raw.githubusercontent.com/moonad/Assets/master/images/fm-net-compilation.png)

This function recursively walks through a term, creating nodes and "temporary
variables" (`x_b`) in the process. It also keeps track of the number of boxes it
passed through, `b`. For example, on the lambda (`{x}f`) case, the procedure
creates a `CON` node with a label `0`, creates a "temporary variable" `x_b` on
the `aux0` port, recurses towards the body of the function, `f` on the `aux1`
port, and then returns the `main` port (because there is a black ball on it).
Notice that there isn't a case for `VAR`. That's what those "temporary
variables" are for. On the `VAR` case, two things can happen:

1. If the corresponding "temporary variable" `x_b` was never used, simply return
   a pointer to it.

2. If the corresponding "temporary variable" `x_b` was used, create a "CON" node
   with a label `2 + b`, connect its main port to the old location of `x_b`, its
   `aux0` to the port `x_b` pointed to, and return a pointer to its `aux1` port.

This process allows us to create as many `CON` nodes as needed to duplicate
`dup`-bound variables, and labels those nodes with the layer of that `dup` (plus
2, since labels 0 and 1 are used for lambdas/applications and
pairs/projections). Note that this process is capable of duplicating λ-bound
variables, but this isn't safe in practice, and won't happen in well-typed
inputs.
