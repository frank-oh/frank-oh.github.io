---
layout:     post
title:      "(FP,terms,scala) Partial Function"
date:       2017-11-14 15:08:00
summary:    Learning Scala by terms - Partial Function
categories: FP terms scala
---

Anyone whe starts to learn Scala(or other functional programming language) would have some difficulties with the jargons the FP people are using. I want to explain Scala features based on those terms used. The first one is *Partial Function*.

The concept of partial function is not so huge for Scala, and you can become familiier with the concepts by knowing the definition and usage of partial functions.


## Definition of partial function

Mathematically, a partial function is a mapping whose value is defined on just part of its domain. For example, if we have to sets `X = {1,2,3,4}`, `Y = {A,B,C}`, the mapping `(1,A),(2,B),(3,C)` is a partial function because it does not any function value defined when x = 4. Sometimes partial function is also called as *partially defined function*. The conecpt itself can be thought as a generalisation of the concept of function.

In scala, a `PartialFunction` is also a `Function. So we can use `PartialFunction` instead of `Function`. You can use `PartialFunction` wherever you can use a `Function`.

Every `PartialFunction` should provide the `isDefinedAt(value)` method. This method returns `true` iff(if and only if) the function value is defined at `value`.

### Why partial function is necessary?

What can we do if we have some function which can make abnormal result. How can we describe the abnormal result and process it? There can be some options:

1. Throw an exception. Period.
2. Use some special value to describe the abnormal situation. For example we can use values of `Option` or `Either` type to distinguish normal return and abnormal one. In Java you might use the `null` to represent the abnormal return value. But  that is not recommended.

In Java you must use `throws` clause to express the exceptions your function may raise. But in Scala you don't have to(We have `@throws` annotation for this purpose, but if you don't want to use `@throws` you may not). So in Scala it is recommended to use types such as `Option`, `Either`, or `Try`. Everytime you see those types from the function signature, you can immediatly know that the function can return some special value for exceptional cases.

But if you have a function that can work on some specific values, how can you describe that function in your program? The possible options are:

1. Throw `InvalidArgumentException`.
2. Return some special value describing the argument error.
3. Make some `PartialFunction`. Before calling that `PartialFunction`, you need to call `isDefinedAt` on that function to check whether you can get right return value or not. If `isDefinedAt` returns `true`, you can call the `PartialFunction` without concerning about any exceptional situations.

The first option is somewhat traditional in Java. The second option is not recommended because you need to check every function return and you cannot seperate the normal flow from the abnormal flow. Using the third option the reader of your program can know the function is partial function by checking the function's type.

One of the advantages in using the `PartialFunction` is we can freely compose the partial function to form a new partial function. We have two predefined composition functions in `PartialFunction[A]`:

- `def andThen[C](k: (B) ⇒ C): PartialFunction[A, C]`
- `def orElse[A1 <: A, B1 >: B](that: PartialFunction[A1, B1]): PartialFunction[A1, B1]`

`andThen` applies the partial function(referred by `this`) only if the input is valid(by using `isDefinedAt`). And apply the result to function received by `k` argument. The composed function has the same domain as the original function(`this`). But the return value will be the composition of k and `this`.

`orElse` returns the result of applying the input to the original function(`this`), but if `this` is not defined at the input value `orElse` check the `that` function's `isDefinedAt` for the input value. And call `that` if `that` is defined at the input value. In effect, `orElse` unions the domains of the two functions with `this` some precedence. The result function will be defined at the union of domain of `this` and `that`.

### The contract the `PartialFunction` need to keep

The contract is: 

> If a function is a `PartialFunction`, it should provide `isDefinedAt`. And if the return value of `f.isDefinedAt(x)` is `true` for a `PartialFunction` `f`, `f(x)` should return normal value

But the compiler cannot force a `PartialFunction` to keep those contract. So the programmer need to be careful to keep the contract, providing correct implementions of both `isDefinedAt()` and `apply()`. Then the use of the `PartialFunction` can check `isDefinedAt()` before calling the function.

## Examples

### Simple Example

Let's check an example. In finance, *Rule of 72* gives you a convient way to calculate how long an investment(or debt) will take to double using some fixed interest rate by compounding.

```scala
// Rule of 72 calculation
val doublePeriod = new PartialFunction[Int, Int] {
  def apply(d: Int) = 72 / d
  // The rull can be applied to somewhat low interest rates
  def isDefinedAt(d: Int) = d > 0 && d < 40 
}
```

Then we can use this `PartialFunction` as below:

```scala
scala> doublePeriod(10)
res0: Int = 7

scala> doublePeriod(0)
java.lang.ArithmeticException: / by zero
  at $anon$1.apply$mcII$sp(<console>:12)
  ... 32 elided

scala> doublePeriod(50)
res2: Int = 1

scala> doublePeriod.isDefinedAt(50)
res3: Boolean = false

scala> doublePeriod.isDefinedAt(0)
res4: Boolean = false
```

As you can see, there is no limitation calling the function even if `isDefinedAt()` returns false. So the **caller** should take care of calling the `isDefinedAt()` and not call if that function returns `false`.

Let's check `andThen` and `orElse` example. First, make two partial functions.

```scala
val oddFt = new PartialFunction[Int, Int] {
  def apply(x: Int) = x + 1
  def isDefinedAt(x: Int) = x%2 == 1
}

val evenFt = new PartialFunction[Int, Int] {
  def apply(x: Int) = x * 2 + 1
  def isDefinedAt(x: Int) = x%2 == 0
} 
```

Let's do a few experiments using the above two partial functions:

```scala
scala> val range = 1 to 10
range: scala.collection.immutable.Range.Inclusive = Range(1, 2, 3, 4, 5, 6, 7, 8, 9, 10)


scala> range.map(oddFt.isDefinedAt)
res5: scala.collection.immutable.IndexedSeq[Boolean] = Vector(true, false, true, false, true, false, true, false, true, false)

scala> range.map(evenFt.isDefinedAt)
res6: scala.collection.immutable.IndexedSeq[Boolean] = Vector(false, true, false, true, false, true, false, true, false, true)

scala> oddFt(1)
res9: Int = 2

scala> evenFt(2)
res10: Int = 5

scala> val composed = oddFt andThen evenFt
composed: PartialFunction[Int,Int] = <function1>

scala> composed(1) = 5

scala> val merged = oddFt orElse evenFt
merged: PartialFunction[Int,Int] = <function1>

scala> merged(1)
res16: Int = 2

scala> merged(2)
res17: Int = 5

scala> range.map(merged.isDefinedAt)
res18: scala.collection.immutable.IndexedSeq[Boolean] = Vector(true, true, true, true, true, true, true, true, true, true)
```

## Real Scala `PartialFunction` examples

### scala collections

Many Scala collections are implementing `PartialFunction`. I will just check the `Map` for the classical partial function example. You may check the "Known Subclasses" section of the [PartialFunction scaladoc](https://www.scala-lang.org/api/current/scala/PartialFunction.html) to get the full list of classes which derive PartialFunction.

#### `Map[K,+V]`

As the name implies, `Map` is a mapping from keys to values. So inherently it is very similiar to function. And if the key value set of a `Map` is finite, the `Map` can be considered as a `PartialFunction`. So `Map` has the `andThen` and `orElse` and also other methods from the `PartialFunction`:

- `def andThen[C](k: (B) ⇒ C): PartialFunction[A, C]`: Composes this partial function with a transformation function that gets applied to results of this partial function.
- `def applyOrElse[A1 <: A, B1 >: B](x: A1, default: (A1) ⇒ B1): B1`: Applies this partial function to the given argument when it is contained in the function domain.
- `def compose[A](g: (A) ⇒ A): (A) ⇒ B`: Composes two instances of Function1 in a new Function1, with this function applied last.
- `def lift: (A) ⇒ Option[B]`: Turns this partial function into a plain function returning an Option result.
- `def orElse[A1 <: A, B1 >: B](that: PartialFunction[A1, B1]): PartialFunction[A1, B1]`: Composes this partial function with a fallback partial function which gets applied where this partial function is not defined.
- `def runWith[U](action: (B) ⇒ U): (A) ⇒ Boolean`: Composes this partial function with an action function which gets applied to results of this partial function.

Moreover there are some other methods as well:

- `def collect[B](pf: PartialFunction[A, B]): Map[B]`: Builds a new collection by applying a partial function to all elements of this map on which the function is defined.
- `def collectFirst[B](pf: PartialFunction[(K, V), B]): Option[B]`: Finds the first element of the traversable or iterator for which the given partial function is defined, and applies the partial function to it.

### `receive` of `Actor`

Except the collections, the most famous `PartialFunction` in Scala would be the `receive` of an `Actor`. An `Actor` can receive any message, the `ActorSystem` should know whether the `Actor` can process the message or not. If you check the [scaladoc of Akka actor](https://doc.akka.io/api/akka/current/akka/actor/Actor.html), you may notice:

- `type Receive = PartialFunction[Any, Unit]`
- `abstract def receive: Actor.Receive` : Scala API: This defines the initial actor behavior, it must return a partial function with the actor logic.

Here, the type of `receive` is `PartialFunction[Any,Unit]`. So the `ActorSystem` can call `isDefinedAt()` on the partial function which `Actor`'s `receive` returns.

### `^?` of Scala Parser Combinator

If you check [Scala `Parser` API doc](https://static.javadoc.io/org.scala-lang.modules/scala-parser-combinators_2.12/1.0.6/scala/util/parsing/combinator/Parsers$Parser.html), you can find:

- `def ^?[U](f: PartialFunction[T, U]): Parser[U]` : `p ^? f` succeeds if `p` succeeds AND `f` is defined at the result of `p`; in that case, it returns `f` applied to the result of `p`.
- `def ^?[U](f: PartialFunction[T, U], error: (T) ⇒ String): Parser[U]` : `p ^? (f, error)` succeeds if `p` succeeds AND `f` is defined at the result of `p`; in that case, it returns f applied to the result of `p`. If `f` is not applicable, `error`(the result of `p`) should explain why.

## Three ways to declare a `PartialFunction`

### First style: using the `PartialFunction` trait

Obviously, the first way to declare a `PartialFunction` is creating a class deriving `PartialFunction` and overriding necessary methods such as `isDefinedAt()` and `apply()`. We saw that in the previous examples. You may mix-in the `PartialFunction` trait into your class as well.

But there are two more ways to make `PartialFunction` easily. Let's check those.

### Second style: Using `match` expression

Let's start by checking an example:

```scala
// save this file as match.scala
object MatchTest {
  def main(args: Array[String]):Unit = {
    val x = 1
    val y = (x:Int) => x match {
        case 1 => "One"
        case 2 => "Two"
        case 3 => "Three"
      }
    println(y(x))
    // Same lambda, different type
    val z:PartialFunction[Int,String] = (x:Int) => x match {
        case 1 => "One"
        case 2 => "Two"
        case 3 => "Three"
      }
    println(z.isDefinedAt(x))
  }
}
```

If you check the compiler generated output using `scalac -Xprint:all .\match.scala`, you can see:

```scala
[[syntax trees at end of                    parser]] // match.scala
package <empty> {
  object MatchTest extends scala.AnyRef {
    def <init>() = {
      super.<init>();
      ()
    };
    def main(args: Array[String]): Unit = {
      val x = 1;
      val y = ((x: Int) => x match {
        case 1 => "One"
        case 2 => "Two"
        case 3 => "Three"
      });
      println(y(x));
      val z: PartialFunction[Int, String] = ((x: Int) => x match {
        case 1 => "One"
        case 2 => "Two"
        case 3 => "Three"
      });
      println(z.isDefinedAt(x))
    }
  }
}

[[syntax trees at end of                     namer]] // match.scala: tree is unchanged since parser
[[syntax trees at end of            packageobjects]] // match.scala: tree is unchanged since parser
[[syntax trees at end of                     typer]] // match.scala
package <empty> {
  object MatchTest extends scala.AnyRef {
    def <init>(): MatchTest.type = {
      MatchTest.super.<init>();
      ()
    };
    def main(args: Array[String]): Unit = {
      val x: Int = 1;
      val y: Int => String = ((x: Int) => x match {
        case 1 => "One"
        case 2 => "Two"
        case 3 => "Three"
      });
      scala.this.Predef.println(y.apply(x));
      val z: PartialFunction[Int,String] = ({
        @SerialVersionUID(value = 0) final <synthetic> class $anonfun extends scala.runtime.AbstractPartialFunction[Int,String] with Serializable {
          def <init>(): <$anon: Int => String> = {
            $anonfun.super.<init>();
            ()
          };
          final override def applyOrElse[A1 <: Int, B1 >: String](x: A1, default: A1 => B1): B1 = (x: A1 @unchecked) match {
            case 1 => "One"
            case 2 => "Two"
            case 3 => "Three"
            case (defaultCase$ @ _) => default.apply(x)
          };
          final def isDefinedAt(x: Int): Boolean = (x: Int @unchecked) match {
            case 1 => true
            case 2 => true
            case 3 => true
            case (defaultCase$ @ _) => false
          }
        };
        new $anonfun()
      }: PartialFunction[Int,String]);
      scala.this.Predef.println(z.isDefinedAt(x))
    }
  }
}

[[syntax trees at end of                    patmat]] // match.scala
package <empty> {
  object MatchTest extends scala.AnyRef {
    def <init>(): MatchTest.type = {
      MatchTest.super.<init>();
      ()
    };
    def main(args: Array[String]): Unit = {
      val x: Int = 1;
      val y: Int => String = ((x: Int) => {
        case <synthetic> val x1: Int = x;
        x1 match {
          case 1 => "One"
          case 2 => "Two"
          case 3 => "Three"
          case _ => throw new MatchError(x1)
        }
      });
      scala.this.Predef.println(y.apply(x));
      val z: PartialFunction[Int,String] = ({
        @SerialVersionUID(value = 0) final <synthetic> class $anonfun extends scala.runtime.AbstractPartialFunction[Int,String] with Serializable {
          def <init>(): <$anon: Int => String> = {
            $anonfun.super.<init>();
            ()
          };
          final override def applyOrElse[A1 <: Int, B1 >: String](x: A1, default: A1 => B1): B1 = {
            case <synthetic> val x1: A1 = (x: A1 @unchecked);
            case7(){
              if (1.==(x1))
                matchEnd6("One")
              else
                case8()
            };
            case8(){
              if (2.==(x1))
                matchEnd6("Two")
              else
                case9()
            };
            case9(){
              if (3.==(x1))
                matchEnd6("Three")
              else
                case10()
            };
            case10(){
              matchEnd6(default.apply(x))
            };
            matchEnd6(x: B1){
              x
            }
          };
          final def isDefinedAt(x: Int): Boolean = {
            case <synthetic> val x1: Int = (x: Int @unchecked);
            x1 match {
              case 1 => true
              case 2 => true
              case 3 => true
              case _ => false
            }
          }
        };
        new $anonfun()
      }: PartialFunction[Int,String]);
      scala.this.Predef.println(z.isDefinedAt(x))
    }
  }
}
... omit output ...
```

If you analyse the compiler output, you may notice the difference between the untyped lambda and `PartialFunction` typed lambda. While the *typer* performs type checking, it automatically insert the `isDefinedAt()` and `applyOrElse`. This is where the magic happens. The runtime `AbstractPartialFunction` supplies the `apply()` override method. So the whole lambda can become a `PartialFunction` with all the methods available.

#### abbreviated `match` form: `case` clauses

And in the form we used `val z: PartialFunction[Int, String] = ((x: Int) => x match { ... }`, the `x` parameter does nothing but a boilerplate for defining a lambda. So Scala permits programmers use the `match` expression without providing the only explicit parameter used. To use this shorter form, you only need to use the `case` parts, by omitting the lambda header(parameter and `=>`) and the `x match`, as below:

```scala
val z2:PartialFunction[Int,String] = {
  case 1 => "One"
  case 2 => "Two"
  case 3 => "Three"
}
println(z2.isDefinedAt(x))
```

If you use this abbreviated form instaed of the original lamdba in `match.scala` you can get the same result bytecode. Those case block can used as a `PartialFunction`. And because a `PartialFunction` is also a derived class of `Function`, you can use the `case` block form wherever you need a lambda or a function. But you should be careful because it can raise the `MatchError` if the input does not match anyone of the `case`s.

Because of this, you can easily pass the `case` match expression into HOF using the abbreviated form.

```scala
scala> List(1,2,3) map { 
     | case 1 => "One"
     | case 2 => "Two"
     | case _ => "Other" }
res2: List[String] = List(One, Two, Other)
```

### Third Style: Using Collection

As I mentioned earlier in *Real Scala `PartialFunction` examples*, the scala collections provides the override methods for the `PartialFunction` trait.

Let's consider `List`, `Map`, `Array`:

```scala
scala> val family = List("Frank","Kevin","Joshua")
family: List[String] = List(Frank, Kevin, Joshua)

scala> family(0)
res6: String = Frank

scala> family(3)
java.lang.IndexOutOfBoundsException: 3
  at scala.collection.LinearSeqOptimized$class.apply(LinearSeqOptimized.scala:65)
  at scala.collection.immutable.List.apply(List.scala:84)
  ... 32 elided

scala> val familyMap = Map( "dad" -> "Frank", "mum" -> "Joyce")
familyMap: scala.collection.immutable.Map[String,String] = Map(dad -> Frank, mum -> Joyce)

scala> familyMap("dad")
res8: String = Frank

scala> familyMap("mistress")
java.util.NoSuchElementException: key not found: mistress
  at scala.collection.MapLike$class.default(MapLike.scala:228)
  at scala.collection.AbstractMap.default(Map.scala:59)
  at scala.collection.MapLike$class.apply(MapLike.scala:141)
  at scala.collection.AbstractMap.apply(Map.scala:59)
  ... 32 elided

scala> val arr = Array(1,2,3,4,5)
arr: Array[Int] = Array(1, 2, 3, 4, 5)

scala> arr(0)
res10: Int = 1

scala> arr(-1)
java.lang.ArrayIndexOutOfBoundsException: -1
  ... 32 elided
```

What is the common characteristics of all the element getting operations(in reality, that operation is implemented by the `apply(i)` method by the Scala convention)?

- If `i` is greater or equal to `0` and `i` is less than the number of elements, the operation returns the element at that position. 
- Otherwise, the operation throws `IndexOutOfBoundsException` or `ArrayIndexOutOfBoundsException`.

Similarly, for `Map` the operation `apply(key)` provides:

- If the `key` matches one of the keys in the map, it will return the associated value for the `key`.
- Otherwise, it throws `NoSuchElementException`.


Did you realize something familiar we've alread seen? Yes! That is basically same as the `PartialFunction`. So Scala collections can mix in the `PartialFunction`:

```scala
scala> family.isDefinedAt(100)
res12: Boolean = false

scala> familyMap.isDefinedAt("Hoho")
res13: Boolean = false

scala> arr.isDefinedAt(Int.MaxValue)
res14: Boolean = false
```

So one of the most simple way to define a `PartialFunction` without using `case` block is making a `Map`. You may use `List` or `Array` for providing a `PartialFunction` with consecutive integers from 0 to some number as its domain, but that is very limited usage. So using `Map` is much powerful than other collections.

```scala
scala> val y:PartialFunction[Int, String] = Map( 1->"One", 2->"Two" )
y: PartialFunction[Int,String] = Map(1 -> One, 2 -> Two)
```

Sometimes people confused the *partial function* with the *partially applied* function. So next time I will explain about the *partially applied* function.