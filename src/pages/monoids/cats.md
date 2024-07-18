## Monoids in Cats

Now we've seen what monoids are,
let's look at their implementation in Cats.
Once again we'll look at the three main aspects of the implementation:
the *type class*, the *instances*, and the *interface*.

### The Monoid Type Class

The monoid type class is `cats.kernel.Monoid`,
which is aliased as [`cats.Monoid`][cats.kernel.Monoid].
`Monoid` extends `cats.kernel.Semigroup`,
which is aliased as [`cats.Semigroup`][cats.kernel.Semigroup].
When using Cats we normally import type classes
from the [`cats`][cats.package] package:

```scala mdoc:silent
import cats.Monoid
import cats.Semigroup
```

or just

```scala mdoc:reset:silent
import cats.*
```

<div class="callout callout-info">
#### Cats Kernel? {-}

Cats Kernel is a subproject of Cats
providing a small set of typeclasses
for libraries that don't require the full Cats toolbox.
While these core type classes are technically
defined in the [`cats.kernel`][cats.kernel.package] package,
they are all aliased to the [`cats`][cats.package] package
so we rarely need to be aware of the distinction.

The Cats Kernel type classes covered in this book are
[`Eq`][cats.kernel.Eq],
[`Semigroup`][cats.kernel.Semigroup],
and [`Monoid`][cats.kernel.Monoid].
All the other type classes we cover
are part of the main Cats project and
are defined directly in the [`cats`][cats.package] package.
</div>


### Monoid Instances {#sec:monoid-instances}

`Monoid` follows the standard Cats pattern for the user interface:
the companion object has an `apply` method
that returns the type class instance for a particular type.
For example, if we want the monoid instance for `String`,
and we have the correct given instances in scope,
we can write the following:

```scala mdoc:silent
import cats.Monoid
```

```scala mdoc
Monoid[String].combine("Hi ", "there")
Monoid[String].empty
```

which is equivalent to:

```scala mdoc
Monoid.apply[String].combine("Hi ", "there")
Monoid.apply[String].empty
```

As we know, `Monoid` extends `Semigroup`.
If we don't need `empty` we can equivalently write:

```scala mdoc:silent
import cats.Semigroup
```

```scala mdoc
Semigroup[String].combine("Hi ", "there")
```

The standard type class instances for `Monoid`
are all found on the appropriate companion objects,
and so are automatically in the given scope with no further imports required.


### Monoid Syntax {#sec:monoid-syntax}

Cats provides syntax for the `combine` method
in the form of the `|+|` operator.
Because `combine` technically comes from `Semigroup`,
we access the syntax by importing from [`cats.syntax.semigroup`][cats.syntax.semigroup]:

```scala mdoc:silent
import cats.syntax.semigroup.* // for |+|
```

```scala mdoc
val stringResult = "Hi " |+| "there" |+| Monoid[String].empty

val intResult = 1 |+| 2 |+| Monoid[Int].empty
```

As always,
unless there is compelling reason not, we recommend importing all the syntax with

```scala mdoc:silent
import cats.syntax.all.*
```


#### Exercise: Adding All The Things

The cutting edge *SuperAdder v3.5a-32* is the world's first choice for adding together numbers.
The main function in the program has signature `def add(items: List[Int]): Int`.
In a tragic accident this code is deleted! Rewrite the method and save the day!

<div class="solution">
We can write the addition as a `foldLeft` using `0` and the `+` operator:

```scala mdoc:silent
def add(items: List[Int]): Int =
  items.foldLeft(0)(_ + _)
```

We can alternatively write the fold using `Monoids`,
although there's not a compelling use case for this yet:

```scala mdoc:silent:reset-object
import cats.Monoid
import cats.syntax.all.*

def add(items: List[Int]): Int =
  items.foldLeft(Monoid[Int].empty)(_ |+| _)
```
</div>

Well done! SuperAdder's market share continues to grow,
and now there is demand for additional functionality.
People now want to add `List[Option[Int]]`.
Change `add` so this is possible.
The SuperAdder code base is of the highest quality,
so make sure there is no code duplication!

<div class="solution">
Now there is a use case for `Monoids`.
We need a single method that adds `Ints` and instances of `Option[Int]`.
We can write this as a generic method that accepts an implicit `Monoid` as a parameter:

```scala mdoc:silent:reset-object
import cats.Monoid
import cats.syntax.all.*

def add[A](items: List[A])(using monoid: Monoid[A]): A =
  items.foldLeft(monoid.empty)(_ |+| _)
```

We can optionally use Scala's *context bound* syntax to write the same code in a shorter way:

```scala mdoc:invisible:reset-object
import cats.Monoid
import cats.syntax.all.* 
```
```scala mdoc:silent
def add[A: Monoid](items: List[A]): A =
  items.foldLeft(Monoid[A].empty)(_ |+| _)
```

We can use this code to add values of type `Int` and `Option[Int]` as requested:

```scala mdoc
add(List(1, 2, 3))
```

```scala mdoc
add(List(Some(1), None, Some(2), None, Some(3)))
```

Note that if we try to add a list consisting entirely of `Some` values,
we get a compile error:

```scala mdoc:fail
add(List(Some(1), Some(2), Some(3)))
```

This happens because the inferred type of the list is `List[Some[Int]]`,
while Cats will only generate a `Monoid` for `Option[Int]`.
We'll see how to get around this in a moment.
</div>

SuperAdder is entering the POS (point-of-sale, not the other POS) market.
Now we want to add up `Orders`:

```scala mdoc:silent
case class Order(totalCost: Double, quantity: Double)
```

We need to release this code really soon so we can't make any modifications to `add`.
Make it so!

<div class="solution">
Easy---we simply define a monoid instance for `Order`!

```scala mdoc:silent
given monoid: Monoid[Order] with {
  def combine(o1: Order, o2: Order) =
    Order(
      o1.totalCost + o2.totalCost,
      o1.quantity + o2.quantity
    )

  def empty = Order(0, 0)
}
```
</div>
