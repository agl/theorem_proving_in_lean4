Dependent Type Theory
=====================

Dependent type theory is a powerful and expressive language, allowing
you to express complex mathematical assertions, write complex hardware
and software specifications, and reason about both of these in a
natural and uniform way. Lean is based on a version of dependent type
theory known as the *Calculus of Constructions*, with a countable
hierarchy of non-cumulative universes and inductive types. By the end
of this chapter, you will understand much of what this means.

## Simple Type Theory

"Type theory" gets its name from the fact that every expression has an
associated *type*. For example, in a given context, ``x + 0`` may
denote a natural number and ``f`` may denote a function on the natural
numbers. For those who like precise definitions, a Lean natural number
is an arbitrary-precision unsigned integer.

Here are some examples of how you can declare objects in Lean and
check their types.

```lean
/- Declare some constants. -/

constant m  : Nat   -- m is a natural number
constant n  : Nat
constant b1 : Bool  -- b1 is a Boolean
constant b2 : Bool

/- Check their types. -/

#check m            -- output: Nat
#check n
#check n + 0        -- Nat
#check m * (n + 0)  -- Nat
#check b1           -- Bool
#check b1 && b2     -- "&&" is the Boolean and
#check b1 || b2     -- Boolean or
#check true         -- Boolean "true"
```

Any text between ``/-`` and ``-/`` constitutes a comment block that is
ignored by Lean. Similarly, two dashes `--` indicate that the rest of
the line contains a comment that is also ignored. Comment blocks can
be nested, making it possible to "comment out" chunks of code, just as
in many programming languages.

The ``constant`` command introduces new constant symbols into the
working environment.  The ``#check`` command asks Lean to report their
types; in Lean, auxiliary commands that query the system for
information typically begin with the hash (#) symbol. You should try
declaring some constants and type checking some expressions on your
own. Declaring new objects in this way is a good way to experiment
with the system.

What makes simple type theory powerful is that you can build new types
out of others. For example, if ``a`` and ``b`` are types, ``a -> b``
denotes the type of functions from ``a`` to ``b``, and ``a × b``
denotes the type of pairs consisting of an element of ``a`` paired
with an element of ``b``, also known as the *Cartesian product*. Note
that `×` is a Unicode symbol. The judicious use of Unicode improves
legibility, and all modern editors have great support for it. In the
Lean standard library, you often see Greek letters to denote types,
and the Unicode symbol `→` as a more compact version of `->`.

```lean
constant m : Nat
constant n : Nat

constant f  : Nat → Nat         -- type the arrow as "\to" or "\r"
constant f' : Nat -> Nat        -- alternative ASCII notation
constant p  : Nat × Nat         -- type the product as "\times"
constant q  : Prod Nat Nat      -- alternative notation
constant g  : Nat → Nat → Nat
constant g' : Nat → (Nat → Nat) -- has the same type as g!
constant h  : Nat × Nat → Nat
constant F  : (Nat → Nat) → Nat -- a "functional"

#check f            -- Nat → Nat
#check f n          -- Nat
#check g m n        -- Nat
#check g m          -- Nat → Nat
#check (m, n)       -- Nat × Nat
#check p.1          -- Nat
#check p.2          -- Nat
#check (m, n).1     -- Nat
#check (p.1, n)     -- Nat × Nat
#check F f          -- Nat
```

Once again, you should try some examples on your own.

Let's take a look at some basic syntax. You can enter the unicode
arrow ``→`` by typing ``\to`` or ``\r``. You can also use the ASCII
alternative ``->``, so the expressions ``Nat -> Nat`` and ``Nat →
Nat`` mean the same thing. Both expressions denote the type of
functions that take a natural number as input and return a natural
number as output. The unicode symbol ``×`` for the Cartesian product
is entered as ``\times``. You will generally use lower-case Greek
letters like ``α``, ``β``, and ``γ`` to range over types. You can
enter these particular ones with ``\a``, ``\b``, and ``\g``.

There are a few more things to notice here. First, the application of
a function ``f`` to a value ``x`` is denoted ``f x``. Second, when
writing type expressions, arrows associate to the *right*; for
example, the type of ``g`` is ``Nat → (Nat → Nat)``. Thus you can
view ``g`` as a function that takes natural numbers and returns
another function that takes a natural number and returns a natural
number. In type theory, this is generally more convenient than
writing ``g`` as a function that takes a pair of natural numbers as
input and returns a natural number as output. For example, it allows
you to "partially apply" the function ``g``.  The example above shows
that ``g m`` has type ``Nat → Nat``, that is, ``g m`` returns a
function that "waits" for a second argument, ``n``, which is then
equivalent to writing ``g m n``. Taking a function ``h`` of type ``Nat
× Nat → Nat`` and "redefining" it to look like ``g`` is a process
known as *currying*.

You have seen that if you have ``m : Nat`` and ``n : Nat``, then
``(m, n)`` denotes the ordered pair of ``m`` and ``n`` which is of
type ``Nat × Nat``. This gives you a way of creating pairs of natural
numbers. Conversely, if you have ``p : Nat × Nat``, then you can write
``p.1 : Nat`` and ``p.2 : Nat``. This gives you a way of extracting
its two components.

## Types as objects

One way in which Lean's dependent type theory extends simple type
theory is that types themselves --- entities like ``Nat`` and ``Bool``
--- are first-class citizens, which is to say that they themselves are
objects. For that to be the case, each of them also has to have a
type.

```lean
#check Nat               -- Type
#check Bool              -- Type
#check Nat → Bool        -- Type
#check Nat × Bool        -- Type
#check Nat → Nat         -- ...
#check Nat × Nat → Nat
#check Nat → Nat → Nat
#check Nat → (Nat → Nat)
#check Nat → Nat → Bool
#check (Nat → Nat) → Nat
```

You can see that each one of the expressions above is an object of
type ``Type``. You can also declare new constants for types:

```lean
constant α : Type
constant β : Type
constant F : Type → Type
constant G : Type → Type → Type

#check α        -- Type
#check F α      -- Type
#check F Nat    -- Type
#check G α      -- Type → Type
#check G α β    -- Type
#check G α Nat  -- Type
```

Indeed, you have already seen an example of a function of type
``Type → Type → Type``, namely, the Cartesian product:

```lean
constant α : Type
constant β : Type

#check Prod α β       -- Type
#check α × β          -- Type

#check Prod Nat Nat   -- Type
#check Nat × Nat      -- Type
```

Here is another example: given any type ``α``, the type ``List α``
denotes the type of lists of elements of type ``α``.

```lean
constant α : Type

#check List α    -- Type
#check List Nat  -- Type
```

Given that every expression in Lean has a type, it is natural to ask:
what type does ``Type`` itself have?

```lean
#check Type      -- Type 1
```

You have actually come up against one of the most subtle aspects of
Lean's typing system. Lean's underlying foundation has an infinite
hierarchy of types:

```lean
#check Type     -- Type 1
#check Type 1   -- Type 2
#check Type 2   -- Type 3
#check Type 3   -- Type 4
#check Type 4   -- Type 5
```

Think of ``Type 0`` as a universe of "small" or "ordinary" types.
``Type 1`` is then a larger universe of types, which contains ``Type
0`` as an element, and ``Type 2`` is an even larger universe of types,
which contains ``Type 1`` as an element. The list is indefinite, so
that there is a ``Type n`` for every natural number ``n``. ``Type`` is
an abbreviation for ``Type 0``:

```lean
#check Type
#check Type 0
```

Some operations, however, need to be *polymorphic* over type
universes. For example, ``List α`` should make sense for any type
``α``, no matter which type universe ``α`` lives in. This explains the
type annotation of the function ``List``:

```lean
#check List    -- Type u_1 → Type u_1
```

Here ``u_1`` is a variable ranging over type levels. The output of the
``#check`` command means that whenever ``α`` has type ``Type n``,
``List α`` also has type ``Type n``. The function ``Prod`` is
similarly polymorphic:

```lean
#check Prod    -- Type u_1 → Type u_2 → Type (max u_1 u_2)
```

To define polymorphic constants and variables, Lean allows you to
declare universe variables explicitly using the `universe` command:

```lean
universe u
constant α : Type u
#check α
```
Equivalently, you can write ``Type _`` to avoid giving the arbitrary
universe a name:

```lean
constant α : Type _
#check α
```

## Function Abstraction and Evaluation

Lean provides a `fun` (or `λ`) keyword to create a function
from an expression as follows:

```lean
#check fun (x : Nat) => x + 5   -- Nat → Nat
#check λ (x : Nat) => x + 5     -- λ and fun mean the same thing
#check fun x : Nat => x + 5     -- Nat inferred
#check λ x : Nat => x + 5       -- Nat inferred
```

You can evaluate a lamda function by passing the required parameters:

```lean
#eval (λ x : Nat => x + 5) 10    -- 15
```

Creating a function from another expression is a process known as
*lambda abstraction*. Suppose you have the variable ``x : α`` you can
construct an expression ``t : β``. Then the expression ``fun (x : α)
=> t``, or, equivalently, ``λ (x : α) => t``, is an object of type ``α
→ β``. Think of this as the function from ``α`` to ``β`` which maps
any value ``x`` to the value ``t``.

Here are some more examples

```lean
constant f : Nat → Nat
constant h : Nat → Bool → Nat

#check fun x : Nat => fun y : Bool => h (f x) y   -- Nat → Bool → Nat
#check fun (x : Nat) (y : Bool) => h (f x) y      -- Nat → Bool → Nat
#check fun x y => h (f x) y                       -- Nat → Bool → Nat
```

Lean interprets the final three examples as the same expression; in
the last expression, Lean infers the type of ``x`` and ``y`` from the
types of ``f`` and ``h``.

Some mathematically common examples of operations of functions can be
described in terms of lambda abstraction:

```lean
constant f : Nat → String
constant g : String → Bool
constant b : Bool

#check fun x : Nat => x        -- Nat → Nat
#check fun x : Nat => b        -- Nat → Bool
#check fun x : Nat => g (f x)  -- Nat → Bool
#check fun x => g (f x)        -- Nat → Bool
```

Think about what these expressions mean. The expression
``fun x : Nat => x`` denotes the identity function on ``Nat``, the
expression ``fun x : Nat => b`` denotes the constant function that
always returns ``b``, and ``fun x : Nat => g (f x)``, denotes the
composition of ``f`` and ``g``.  You can, in general, leave off the
type annotation and let Lean infer it for you.  So, for example, you
can write ``fun x => g (f x)`` instead of ``fun x : Nat => g (f x)``.

You can pass functions as parameters and by giving them names `f`
and `g` you can then use those functions in the implementation:

```lean
#check fun (g : String → Bool) (f : Nat → String) (x : Nat) => g (f x)
-- (String → Bool) → (Nat → String) → Nat → Bool
```

You can also pass types as parameters:

```lean
#check fun (α β γ : Type) (g : β → γ) (f : α → β) (x : α) => g (f x)
```
The last expression, for example, denotes the function that takes
three types, ``α``, ``β``, and ``γ``, and two functions, ``g : β → γ``
and ``f : α → β``, and returns the composition of ``g`` and ``f``.
(Making sense of the type of this function requires an understanding
of dependent products, which will be explained below.)

The general form of a lambda expression is ``fun x : α => t``, where
the variable ``x`` is a "bound variable": it is really a placeholder,
whose "scope" does not extend beyond the expression ``t``.  For
example, the variable ``b`` in the expression ``fun (b : β) (x : α) => b``
has nothing to do with the constant ``b`` declared earlier.  In fact,
the expression denotes the same function as ``fun (u : β) (z : α), u``.

Formally, the expressions that are the same up to a renaming of bound
variables are called *alpha equivalent*, and are considered "the
same." Lean recognizes this equivalence.

Notice that applying a term ``t : α → β`` to a term ``s : α`` yields
an expression ``t s : β``. Returning to the previous example and
renaming bound variables for clarity, notice the types of the
following expressions:

```lean
#check (fun x : Nat => x) 1     -- Nat
#check (fun x : Nat => true) 1  -- Bool

constant f : Nat → String
constant g : String → Bool

#check
  (fun (α β γ : Type) (u : β → γ) (v : α → β) (x : α) => u (v x)) Nat String Bool g f 0
  -- Bool
```

As expected, the expression ``(fun x : Nat =>  x) 1`` has type ``Nat``.
In fact, more should be true: applying the expression ``(fun x : Nat
=> x)`` to ``1`` should "return" the value ``1``. And, indeed, it does:

```lean
#eval (fun x : Nat => x) 1     -- 1
#eval (fun x : Nat => true) 1  -- true

constant f : Nat → String
constant g : String → Bool
```

You will see later how these terms are evaluated. For now, notice that
this is an important feature of dependent type theory: every term has
a computational behavior, and supports a notion of *normalization*. In
principle, two terms that reduce to the same value are called
*definitionally equal*. They are considered "the same" by Lean's type
checker, and Lean does its best to recognize and support these
identifications.

Lean is a complete programming language. It has a compiler that
generates a binary executable and an interactive interpreter. You can
use the command `#eval` to execute expressions, and it is the
preferred way of testing your functions. Note that `#eval` and
`#reduce` are *not* equivalent. The command `#eval` first compiles
Lean expressions into an intermediate representation (IR) and then
uses an interpreter to execute the generated IR. Some builtin types
(e.g., `Nat`, `String`, `Array`) have a more efficient representation
in the IR. The IR has support for using foreign functions that are
opaque to Lean.

In contrast, the ``#reduce`` command relies on a reduction engine
similar to the one used in Lean's trusted kernel, the part of Lean
that is responsible for checking and verifying the correctness of
expressions and proofs. It is less efficient than ``#eval``, and
treats all foreign functions as opaque constants. You will learn later
that there are some other differences between the two commands.

## Introducing Definitions

The ``def`` keyword provides one important way of declaring new named
objects.

```lean
def double (x : Nat) : Nat :=
  x + x
```

This might look more familiar to you if you know how functions work in
other programming languages. The name `double` is defined as a
function that takes an input parameter `x` of type `Nat`, where the
result of the call is `x + x`, so it is returning type `Nat`. You
can then invoke this function using:

```lean
# def double (x : Nat) : Nat :=
#  x + x
#eval double 3    -- 6
```

In this case you can think of `def` as a kind of named `lambda`.
The following yields the same result:

```lean
def double : Nat → Nat :=
  fun x => x + x

#eval double 3    -- 6
```

You can omit the type declarations when Lean has enough information to
infer it.  Type inference is an important part of Lean:

```lean
def double :=
  fun (x : Nat) => x + x
```

The general form of a definition is ``def foo : α := bar`` where
``α`` is the type returned from the expression ``bar``.  Lean can
usually infer the type ``α``, but it is often a good idea to write it
explicitly.  This clarifies your intention, and Lean will flag an
error if the right-hand side of the definition does not have a matching
type.

The right hand side `bar` can be any expression, not just a lambda.
So `def` can also be used to simply name a value like this:

```lean
def pi := 3.141592654
```

`def` can take multiple input parameters.  Let's create one
that adds two natural numbers:

```lean
def add (x y : Nat) :=
  x + y

#eval add 3 2               -- 5
```

The parameter list can be separated like this:

```lean
# def double (x : Nat) : Nat :=
#  x + x
def add (x : Nat) (y : Nat) :=
  x + y

#eval add (double 3) (7 + 9)  -- 22
```

Notice here we called the `double` function to create the first
parameter to `add`.

You can use other more interesting expressions inside a `def`:

```lean
def greater (x y : Nat) :=
  if x > y then x
  else y
```

You can probably guess what this one will do.

You can also define a function that takes another function as input.
The following calls a given function twice passing the output of the
first invocation to the second:

```lean
# def double (x : Nat) : Nat :=
#  x + x
def doTwice (f : Nat → Nat) (x : Nat) : Nat :=
  f (f x)

#eval doTwice double 2   -- 8
```

Now to get a bit more abstract, you can also specify arguments that
are like type parameters:

```lean
def compose (α β γ : Type) (g : β → γ) (f : α → β) (x : α) : γ :=
  g (f x)
```

This means `compose` is a function that takes any 2 functions as input
arguments, so long as those functions each take only one input.
The type algebra `β → γ` and `α → β` means that it is a requirement
that the type of the output of the second function must match the
type of the input to the first function - which makes sense, otherwise
the two functions would not be composable.

`compose` also takes a 3rd argument of type `α` which
it uses to invoke the second function (locally named `f`) and it
passes the result of that function (which is type `β`) as input to the
first function (locally named `g`).  The first function returns a type
`γ` so that is also the return type of the `compose` function.

`compose` is also very general in that it works over any type
`α β γ`.  This means `compose` can compose just about any 2 functions
so long as they each take one parameter, and so long as the type of
output of the second matches the input of the first.  For example:

```lean
# def compose (α β γ : Type) (g : β → γ) (f : α → β) (x : α) : γ :=
#  g (f x)
# def double (x : Nat) : Nat :=
#  x + x
def square (x : Nat) : Nat :=
  x * x

#eval compose Nat Nat Nat double square 3  -- 18
```

Local Definitions
-----------------

Lean also allows you to introduce "local" definitions using the
``let`` keyword. The expression ``let a := t1; t2`` is
definitionally equal to the result of replacing every occurrence of
``a`` in ``t2`` by ``t1``.

```lean
#check let y := 2 + 2; y * y   -- Nat
#eval  let y := 2 + 2; y * y   -- 16

def twice_double (x : Nat) : Nat :=
  let y := x + x; y * y

#eval twice_double 2   -- 16
```

Here, ``twice_double`` is definitionally equal to the term ``(x + x) * (x + x)``.

You can combine multiple assignments in a single ``let`` statement:

```lean
#check let y := 2 + 2; let z := y + y; z * z   -- Nat
#eval  let y := 2 + 2; let z := y + y; z * z   -- 64
```

The ``;`` can be omitted when a line break is used.
```lean
def t (x : Nat) : Nat :=
  let y := x + x
  y * y
```

Notice that the meaning of the expression ``let a := t1; t2`` is very
similar to the meaning of ``(fun a => t2) t1``, but the two are not
the same. In the first expression, you should think of every instance
of ``a`` in ``t2`` as a syntactic abbreviation for ``t1``. In the
second expression, ``a`` is a variable, and the expression
``fun a => t2`` has to make sense independently of the value of ``a``.
The ``let`` construct is a stronger means of abbreviation, and there
are expressions of the form ``let a := t1; t2`` that cannot be
expressed as ``(fun a => t2) t1``. As an exercise, try to understand
why the definition of ``foo`` below type checks, but the definition of
``bar`` does not.

```lean
def foo := let a := Nat; fun x : a => x + 2
/-
  def bar := (fun a => fun x : a => x + 2) Nat
-/
```
# <a name="_variables_and_sections"></a>Variables and Sections

Consider the following three function definitions:
```lean
def compose (α β γ : Type) (g : β → γ) (f : α → β) (x : α) : γ :=
  g (f x)

def doTwice (α : Type) (h : α → α) (x : α) : α :=
  h (h x)

def doThrice (α : Type) (h : α → α) (x : α) : α :=
  h (h (h x))
```

Lean provides you with the ``variable`` command to make such
declarations look more compact:

```lean
variable (α β γ : Type)

def compose (g : β → γ) (f : α → β) (x : α) : γ :=
  g (f x)

def doTwice (h : α → α) (x : α) : α :=
  h (h x)

def doThrice (h : α → α) (x : α) : α :=
  h (h (h x))
```
You can declare variables of any type, not just ``Type`` itself:
```lean
variable (α β γ : Type)
variable (g : β → γ) (f : α → β) (h : α → α)
variable (x : α)

def compose := g (f x)
def doTwice := h (h x)
def doThrice := h (h (h x))

#print compose
#print doTwice
#print doThrice
```
Printing them out shows that all three groups of definitions have
exactly the same effect.

The ``variable`` command instructs Lean to insert the declared
variables as bound variables in definitions that refer to them by
name. Lean is smart enough to figure out which variables are used
explicitly or implicitly in a definition. You can therefore proceed as
though ``α``, ``β``, ``γ``, ``g``, ``f``, ``h``, and ``x`` are fixed
objects when you write your definitions, and let Lean abstract the
definitions for you automatically.

When declared in this way, a variable stays in scope until the end of
the file you are working on. Sometimes, however, it is useful to limit
the scope of a variable. For that purpose, Lean provides the notion of
a ``section``:

```lean
section useful
  variable (α β γ : Type)
  variable (g : β → γ) (f : α → β) (h : α → α)
  variable (x : α)

  def compose := g (f x)
  def doTwice := h (h x)
  def doThrice := h (h (h x))
end useful
```

When the section is closed, the variables go out of scope, and become
nothing more than a distant memory.

You do not have to indent the lines within a section. Nor do you have
to name a section, which is to say, you can use an anonymous
``section`` / ``end`` pair. If you do name a section, however, you
have to close it using the same name. Sections can also be nested,
which allows you to declare new variables incrementally.

# <a name="_namespaces"></a>Namespaces

Lean provides you with the ability to group definitions into nested,
hierarchical *namespaces*:

```lean
namespace Foo
  def a : Nat := 5
  def f (x : Nat) : Nat := x + 7

  def fa : Nat := f a
  def ffa : Nat := f (f a)

  #check a
  #check f
  #check fa
  #check ffa
  #check Foo.fa
end Foo

-- #check a  -- error
-- #check f  -- error
#check Foo.a
#check Foo.f
#check Foo.fa
#check Foo.ffa

open Foo

#check a
#check f
#check fa
#check Foo.fa
```

When you declare that you are working in the namespace ``Foo``, every
identifier you declare has a full name with prefix "``Foo.``" Within
the namespace, you can refer to identifiers by their shorter names,
but once you end the namespace, you have to use the longer names.
Unlike `section` namespaces require a name, there is only one
anonymous namespace at the root level.

The ``open`` command brings the shorter names into the current
context. Often, when you import a module, you will want to open one or
more of the namespaces it contains, to have access to the short
identifiers. But sometimes you will want to leave this information
protected by a fully qualified name, for example, when they conflict
with identifiers in another namespace you want to use. Thus namespaces
give you a way to manage names in your working environment.

For example, Lean groups definitions and theorems involving lists into
a namespace ``List``.
```lean
#check List.nil
#check List.cons
#check List.map
```
The command ``open List`` allows you to use the shorter names:
```lean
open List

#check nil
#check cons
#check map
```
Like sections, namespaces can be nested:
```lean
namespace Foo
  def a : Nat := 5
  def f (x : Nat) : Nat := x + 7

  def fa : Nat := f a

  namespace Bar
    def ffa : Nat := f (f a)

    #check fa
    #check ffa
  end Bar

  #check fa
  #check Bar.ffa
end Foo

#check Foo.fa
#check Foo.Bar.ffa

open Foo

#check fa
#check Bar.ffa
```
Namespaces that have been closed can later be reopened, even in another file:

```lean
namespace Foo
  def a : Nat := 5
  def f (x : Nat) : Nat := x + 7

  def fa : Nat := f a
end Foo

#check Foo.a
#check Foo.f

namespace Foo
  def ffa : Nat := f (f a)
end Foo
```

Like sections, nested namespaces have to be closed in the order they
are opened. Namespaces and sections serve different purposes:
namespaces organize data and sections declare variables for insertion
in definitions. Sections are also useful for delimiting the scope of
commands such as ``set_option`` and ``open``.

In many respects, however, a ``namespace ... end`` block behaves the
same as a ``section ... end`` block. In particular, if you use the
``variable`` command within a namespace, its scope is limited to the
namespace. Similarly, if you use an ``open`` command within a
namespace, its effects disappear when the namespace is closed.

## What makes dependent type theory dependent?

The short explanation is that types can depend on parameters. You
have already seen a nice example of this: the type ``List α`` depends
on the argument ``α``, and this dependence is what distinguishes
``List Nat`` and ``List Bool``. For another example, consider the
type ``Vector α n``, the type of vectors of elements of ``α`` of
length ``n``.  This type depends on *two* parameters: the type of the
elements in the vector (``α : Type``) and the length of the vector
``n : Nat``.

Suppose you wish to write a function ``cons`` which inserts a new
element at the head of a list. What type should ``cons`` have? Such a
function is *polymorphic*: you expect the ``cons`` function for
``Nat``, ``Bool``, or an arbitrary type ``α`` to behave the same way.
So it makes sense to take the type to be the first argument to
``cons``, so that for any type, ``α``, ``cons α`` is the insertion
function for lists of type ``α``. In other words, for every ``α``,
``cons α`` is the function that takes an element ``a : α`` and a list
``as : List α``, and returns a new list, so you have ``cons α a as : list α``.

It is clear that ``cons α`` should have type ``α → List α → List α``.
But what type should ``cons`` have?  A first guess might be
``Type → α → list α → list α``, but, on reflection, this does not make
sense: the ``α`` in this expression does not refer to anything,
whereas it should refer to the argument of type ``Type``.  In other
words, *assuming* ``α : Type`` is the first argument to the function,
the type of the next two elements are ``α`` and ``List α``. These
types vary depending on the first argument, ``α``.

```
constant α : Type
#check cons α           -- List Type → List Type
#check cons             -- ?m.468 → List ?m.468 → List ?m.468 which is not ``Type → α → list α → list α``
```

This is an instance of a *dependent function type*, or *dependent
arrow type*. Given ``α : Type`` and ``β : α → Type``, think of ``β``
as a family of types over ``α``, that is, a type ``β a`` for each
``a : α``. In that case, the type ``(a : α) → β a`` denotes the type
of functions ``f`` with the property that, for each ``a : α``, ``f a``
is an element of ``β a``. In other words, the type of the value
returned by ``f`` depends on its input.

Notice that ``(a : α) → β`` makes sense for any expression ``β :
Type``. When the value of ``β`` depends on ``a`` (as does, for
example, the expression ``β a`` in the previous paragraph),
``(a : α) → β`` denotes a dependent function type. When ``β`` doesn't
depend on ``a``, ``(a : α) → β`` is no different from the type
``α → β``.  Indeed, in dependent type theory (and in Lean), ``α → β``
is just notation for ``(a : α) → β`` when ``β`` does not depend on ``a``.

Returning to the example of lists, you can use the command `#check` to
inspect the type of the following `List` functions.  The ``@`` symbol
and the difference between the round and curly braces will be
explained momentarily.

```lean
#check @List.cons    -- {α : Type u_1} → α → List α → List α
#check @List.nil     -- {α : Type u_1} → List α
#check @List.length  -- {α : Type u_1} → List α → Nat
#check @List.append  -- {α : Type u_1} → List α → List α → List α
```

Just as dependent function types ``(a : α) → β a`` generalize the
notion of a function type ``α → β`` by allowing ``β`` to depend on
``α``, dependent Cartesian product types ``(a : α) × β a`` generalize
the Cartesian product ``α × β`` in the same way. Dependent products
are also called *sigma* types, and you can also write them as
`Σ a : α, β a`. You can use `⟨a, b⟩` or `Sigma.mk a b` to create a
dependent pair.

```lean
universe u v

def f (α : Type u) (β : α → Type v) (a : α) (b : β a) : (a : α) × β a :=
  ⟨a, b⟩

def g (α : Type u) (β : α → Type v) (a : α) (b : β a) : Σ a : α, β a :=
  Sigma.mk a b

#reduce f
#reduce g

#reduce f Type (fun α => α) Nat 10
#reduce g Type (fun α => α) Nat 10

#reduce (f Type (fun α => α) Nat 10).1 -- Nat
#reduce (g Type (fun α => α) Nat 10).1 -- Nat
#reduce (f Type (fun α => α) Nat 10).2 -- 10
#reduce (g Type (fun α => α) Nat 10).2 -- 10
```
The function `f` and `g` above denote the same function.