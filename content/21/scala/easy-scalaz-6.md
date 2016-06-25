+++
date = "2016-01-01T00:16:53+09:00"
prev = "../easy-scalaz-5"
title = "Easy Scalaz 6"
toc = true
weight = 106
+++

![](https://raw.githubusercontent.com/1ambda/1ambda.github.io/master/assets/images/easy-scalaz/1-state/haskell.png)

## Playing with Monoids

이번 글에서는 모노이드를 가지고 놀면서, 아래 나열된 라이브러리 및 언어적 특성을 살펴보겠습니다.

- Boolean Monoid operations with **[Spire](https://github.com/non/spire)**
- Algebraic Data Types using **[Value Class](http://docs.scala-lang.org/overviews/core/value-classes.html)**, **[Scalaz.Tag](https://github.com/scalaz/scalaz/blob/series/7.3.x/core/src/main/scala/scalaz/Tag.scala)**
- Creating Monoid for all subclasses using **[Shapeless](https://github.com/milessabin/shapeless)**
- **[Context Bound](http://docs.scala-lang.org/tutorials/FAQ/context-and-view-bounds.html)**
- **[Path Dependent Type](http://danielwestheide.com/blog/2013/02/13/the-neophytes-guide-to-scala-part-13-path-dependent-types.html)**
## Monoid

[Easy Scalaz 4 - Yoneda and Free Monad: Monoid](http://1ambda.github.io/easy-scalaz-4-yoneda-and-free-monad/#monoid) 부분에서 발췌하면,

어떤 집합 `S` 에 대한 닫힌 연산 `*`, 집합 내의 어떤 원소 `e` 가 다음을 만족할 경우 모노이드라 부릅니다.

- `e * a = a = a * e` (*identity*)
- `(a * b) * c = a * (b * c)` (*associativity*)

일반적으로 `e` 를 항등원이라 부릅니다. `Option[A]` 도 `None` 을 항등원으로 사용하고, *associativity* 를 만족하는 `A` 의 연산을 사용하면 모노이드입니다. 따라서 `A` 가 모노이드면 `Option[A]` 도 모노이드입니다.

*Scalaz* 에서는 모노이드 연산 `*` 를, `|+|` 로 표시합니다. 우리가 알고 있는 *primitives* 대부분이 모노이드입니다.

```scala
> load.ivy("org.scalaz" % "scalaz-core_2.11" % "7.2.0-M5")

> import scalaz._, Scalaz._
import scalaz._, Scalaz._
> implicitly[Monoid[String]]
res4: Monoid[String] = scalaz.std.StringInstances$stringInstance$@5590d10f
> implicitly[Monoid[Int]]
res5: Monoid[Int] = scalaz.std.AnyValInstances$$anon$5@4b9f2522
> implicitly[Monoid[Set[Int]]]
res6: Monoid[Set[Int]] = scalaz.std.SetInstances$$anon$3@5b1965ea

> "1" |+| "2"
res7: String = "12"
> 1.0 |+| 2.0
Compilation Failed
Main.scala:1459: value |+| is not a member of Double
1.0 |+| 2.0
    ^
> 1 |+| 2
res8: Int = 3

> 1.some |+| 2.some
res11: Option[Int] = Some(3)
> 1.some |+| none
res12: Option[Int] = Some(1)
> none[Int] |+| 1.some
res13: Option[Int] = Some(1)
```

`Map[A, B]` 는 `A` 를 *Key* 로 잡고, `B` 의 모노이드 연산과 항등원을 이용하는 모노이드입니다.

```scala
> val m1 = Map("a" -> 1, "b" -> 2)
m1: Map[String, Int] = Map("a" -> 1, "b" -> 2)
> val m2 = Map("a" -> 1, "c" -> 2)
m2: Map[String, Int] = Map("a" -> 1, "c" -> 2)
> m1 |+| m2
res16: Map[String, Int] = Map("a" -> 2, "c" -> 2, "b" -> 2)
```

## Boolean Monoid

`Boolean` 의 경우에는, 두 가지 모노이드가 존재할 수 있습니다.

- `&&` 를 연산으로 사용하고, `true` 를 항등원으로 사용하는 경우
- `||` 를 연산으로 사용하고, `false` 를 항등원으로 사용하는 경우

첫 번째를 *Conjunction* 이라 부르고 두 번째를 *Disjunction* 이라 부릅니다. 즉, `Boolean` 은 두 개의 모노이드가 존재할 수 있기 때문에 아래처럼 *scalaz* 의 `|+|` 를 바로 이용할 수 없습니다. *Disjunction* 인지 *Conjunction* 인지 골라야 하기 때문입니다.

```scala
> false |+| false
Compilation Failed
Main.scala:1468: value |+| is not a member of Boolean
false |+| false
      ^

// import 를 하지 않으면, scalaz.Tags.Disjunction 이 아니라 scalaz.Disjunction 을 사용하므로 주의
> import scalaz.Tags._
import scalaz.Tags._
> import scalaz.syntax.tag._
import scalaz.syntax.tag._
> Disjunction(false)
res22: Boolean @@ Disjunction = false
> Conjunction(false)
res23: Boolean @@ Conjunction = false

> implicitly[Monoid[Boolean @@ Disjunction]]
res27: Monoid[Boolean @@ Disjunction] = scalaz.std.AnyValInstances$$anon$7@79a6c868
> implicitly[Monoid[Boolean @@ Conjunction]]
res28: Monoid[Boolean @@ Conjunction] = scalaz.std.AnyValInstances$$anon$8@6e49df4a

> Disjunction(false) |+| Disjunction(true)
res29: Boolean @@ Disjunction = true
> Disjunction(true) |+| Disjunction(false)
res30: Boolean @@ Disjunction = true
> Conjunction(true) |+| Conjunction(true)
res31: Boolean @@ Conjunction = true
> Conjunction(true) |+| Conjunction(false)
res32: Boolean @@ Conjunction = false

> List(false, false, true, false)
res37: List[Boolean] = List(false, false, true, false)
> Disjunction.subst(res37).suml
res38: Boolean @@ Disjunction = true
> Conjunction.subst(res37).suml
res39: Boolean @@ Conjunction = false
```

실제로 `scalaz.std.AnyVal` 을 확인해 보면,

```scala
// https://github.com/scalaz/scalaz/blob/series/7.2.x/core/src/main/scala/scalaz/std/AnyVal.scala#L52

object conjunction extends Monoid[Boolean] {
  def append(f1: Boolean, f2: => Boolean) = f1 && f2
  def zero: Boolean = true
}

object disjunction extends Monoid[Boolean] {
  def append(f1: Boolean, f2: => Boolean) = f1 || f2
  def zero = false
}
```

그렇다면 `Int` 의 경우에도 `*` 등 다른 ª¨노이드가 있는데 왜 `+` 연산과 `0` 항등원만 `|+|` 에서 사용하는걸까요? 이는 `+` 가 너무 보편적이기 때문이며, `*` (곱셈) 등은 위에서 본 `Tag` 를 이용해 모노이드 연산으로 지정할 수 있습니다.

## Tag

```scala
// https://github.com/scalaz/scalaz/blob/series/7.2.x/core/src/main/scala/scalaz/Tags.scala

object Tags {

  ...

  /** Type tag to choose a [[scalaz.Monoid]] instance that selects the lesser of two operands, ignoring `zero`. */
  sealed trait Min

  val Min = Tag.of[Min]

  /** Type tag to choose a [[scalaz.Monoid]] instance that selects the greater of two operands, ignoring `zero`. */
  sealed trait Max

  val Max = Tag.of[Max]

  /** Type tag to choose a [[scalaz.Monoid]] instance for a numeric type that performs multiplication,
   *  rather than the default monoid for these types which by convention performs addition. */
  sealed trait Multiplication

  val Multiplication = Tag.of[Multiplication]

  ...
}
```

`Multiplication` 을 이©하면,

```scala
> Multiplication(2) |+| Multiplication(6)
res3: Int @@ Multiplication = 12

> implicitly[Monoid[Int @@ Multiplication]]
res4: Monoid[Int @@ Multiplication] = scalaz.std.AnyValInstances$$anon$12@5910ca72
```

`AnyValInstances` 를 찾아보면 `byteMultiplicationNewType`, `intMultiplicationNewType` 등 `A @@ Multiplication` 을 위한 인스턴스들이 구현되어 있습니다.

```scala
// https://github.com/scalaz/scalaz/blob/series/7.2.x/core/src/main/scala/scalaz/std/AnyVal.scala#L253

trait AnyValInstances {

  implicit val shortMultiplicationNewType: Monoid[Short @@ Multiplication] with Enum[Short @@ Multiplication] = new Monoid[Short @@ Multiplication] with Enum[Short @@ Multiplication] {
    ...
  }

  implicit val intMultiplicationNewType: Monoid[Int @@ Multiplication] with Enum[Int @@ Multiplication] = new Monoid[Int @@ Multiplication] with Enum[Int @@ Multiplication] {
    ...
  }
}
```

`Tag` 는 이렇게 생겼습니다.

```scala
// https://github.com/scalaz/scalaz/blob/series/7.2.x/core/src/main/scala/scalaz/package.scala#L99

package object scalaz {
  ...

  private[scalaz] type Tagged[A, T] = {type Tag = T; type Self = A}

  /**
   * Tag a type `T` with `Tag`.
   *
   * The resulting type is used to discriminate between type class instances.
   *
   * @see [[scalaz.Tag]] and [[scalaz.Tags]]
   *
   * Credit to Miles Sabin for the idea.
   */
  type @@[T, Tag] = Tagged[T, Tag]

  ...
}
```

`@@[A, T]` 를 생성하기 위해 `Tag.apply` 를 값을 추출하기 위해 `unwrap` 을 이용할 수 있습니다.

```scala
// https://github.com/scalaz/scalaz/blob/series/7.2.x/core/src/main/scala/scalaz/Tag.scala
object Tag {
  /** `subst` specialized to `Id`.
    *
    * @todo According to Miles, @specialized doesn't help here. Maybe manually specialize.
    */
  @inline def apply[@specialized A, T](a: A): A @@ T = a.asInstanceOf[A @@ T]

  /** `unsubst` specialized to `Id`. */
  @inline def unwrap[@specialized A, T](a: A @@ T): A = unsubst[A, Id, T](a)

  /** Add a tag `T` to `A`.
    *
    * NB: It is unsafe to `subst` or `unsubst` a tag in an `F` that is
    * sensitive to the `A` type within.  For example, if `F` is a
    * GADT, rather than a normal ADT, it is probably unsafe.  For
    * "normal" types like `List` and function types, it is safe.  More
    * broadly, if it is possible to write a ''legal''
    * [[scalaz.InvariantFunctor]] over the parameter, `subst` of that
    * parameter is safe.
    *
    * We do not have a
    * <a href="https://ghc.haskell.org/trac/ghc/wiki/Roles">type role</a>
    * system in Scala with which to declare the exact situations under
    * which `subst` is safe.  If we did, we would declare that `subst`
    * is safe if and only if the parameter has "representational" or
    * "phantom" role.
    */
  def subst[A, F[_], T](fa: F[A]): F[A @@ T] = fa.asInstanceOf[F[A @@ T]]

  ...
}
```

`Tag` 는 [*Value Class*](http://docs.scala-lang.org/overviews/core/value-classes.html) 처럼 활용할 수도 있는데요,

```scala
// http://eed3si9n.com/learning-scalaz/Tagged+type.html

sealed trait USD
sealed trait EUR
def USD[A](amount: A): A @@ USD = Tag[A, USD](amount)
def EUR[A](amount: A): A @@ EUR = Tag[A, EUR](amount)

val oneUSD = USD(1)
```

태깅된 타입을 이용하면 *implicit* 를 선택할 수 있습니다. 예를 들어

```scala
implicit val anonymousUserWriter = Writer[User @@ Anonymous] { ... }
implicit val loggedInUserWriter  = Writer[User @@ LoggedIn]  { ... }
```

그러나 `type B = A @@ T` 에서 `B` 는 `A` 의 서브타입으로 취급되므로 주의하여 사용해야 합니다. 예를 들어, *scalatest* 의 `===`, `shouldBe` 는 런타임값만 체크하므로 아래는 항상 참입니다.

```scala
def convertUSDtoEUR[A](usd: A @@ USD, rate: A)
                      (implicit M: Monoid[A @@ Multiplication]): A @@ EUR =
  EUR((Multiplication(usd.unwrap) |+| Multiplication(rate)).unwrap)

convertUSDtoEUR(USD(1), 2) === EUR(2) // true
convertUSDtoEUR(USD(1), 2) === USD(2) // true

convertUSDtoEUR(USD(1), 2) shouldBe EUR(2) // true
convertUSDtoEUR(USD(1), 2) shouldBe USD(2) // true

2 shouldBe USD(2) // true
2 shouldBe EUR(2) // true
```

따라서 `=:=` 를 만들어 사용하면 `EUR` 과 `USD` 비교시 컴파일 예외를 발생시킬 수 있습니다. (더 정확히는 *scalaz* 의 `===` 또는 `org.scalactic.TypeCheckedTripleEquals` 를 사용하면 되는데, `org.scalactic.TripleEqualSupports` 를 `FunSuite`  내에서 하이딩 시킬 방법을 찾지 못해서 아래처럼 구현했습니다.)

```scala
// impilcit class 로 만들고 import 해서 사용해도 상관없음
trait TestImplicits {
  final case class StrictEqualOps[A](val a: A) {
    def =:=(aa: A) = assert(a == aa)
    def =/=(aa: A) = assert(!(a == aa))
  }

  implicit def toStrictEqualOps[A](a: A) = StrictEqualOps(a)
}

// spec
convertUSDtoEUR(USD(1), 2) =:= EUR(2)
convertUSDtoEUR(USD(1), 2) =:= EUR(3) // will fail
convertUSDtoEUR(USD(1), 2) =:= USD(3) // compile error
```

`Tag` 을 이용하면 같은 *primitive type* 이어도 별도의 *wrapper* 를 §들지 않으면서 다른 타입으로 만들 수 있습니다. 예를 들어 `Job` 을 `Agent` 가 수행한다고 하면, 다음과 같이 간단한 모델을 만들어 볼 수 있는데

```scala
// ref - http://www.slideshare.net/IainHull/improving-correctness-with-types

case class Agent(id: String, /* agent id */
                 status: String, /* agent status */
                 jobType: String)

case class Job(id: String, /* job id */
               maybeAgentId: Option[String], /* agent id */
               status: String, /* job status */
               jobType: String)
```

여기서 *Sum* 을 먼저 추출하면, (*Algebraic Data Type* 관련해서는 [Sum and Product](https://gleichmann.wordpress.com/2011/02/05/functional-scala-algebraic-datatypes-sum-and-product-types/) 참조)

```scala
sealed abstract class AgentStatus(val value: String)
case object Waiting    extends AgentStatus("WAITING")
case object Processing extends AgentStatus("PROCESSING")

sealed abstract class JobStatus(val value: String)
case object Created   extends JobStatus("CREATED")
case object Allocated extends JobStatus("ALLOCATED")
case object Completed extends JobStatus("COMPLETED")

sealed abstract class JobType(val value: String)
case object Small extends JobType("SMALL")
case object Large extends JobType("LARGE")
case object Batch extends JobType("BATCH")

case class Agent(id: String, /* agent id */
                 status: AgentStatus,
                 jobType: JobType)

case class Job(id: String, /* job id */
               maybeAgentId: Option[String], /* agent id */
               status: JobStatus,
               jobType: JobType)
```

여기서 오류의 소지가 다분한 `id` 에 태깅을 하면 다음과 같습니다.

```scala
import scalaz._

case class Agent(id: String @@ Agent,
                 status: AgentStatus,
                 jobType: JobType)

case class Job(id: String @@ Job,
               maybeAgentId: Option[String @@ Agent],
               status: JobStatus,
               jobType: JobType)

Agent(Tag[String, Agent]("03"), Waiting, Small)
Job(Tag[String, Job]("03"), None, Created, Small)
```

조금 더 개선할 여지는, `maybeAgentId` 에 `Option` 을 이용하는 대신, *agent* 에 할당된 *job* 과 아닌 *job* 을 서브타입으로 분리하면, `Job` 을 다루는 함수에서 `Option` 처리를 피할 수 있습니다.

물론 이는 디자인적 결정입니다. `Option` 을 허용하되 수퍼클래스를 인자로 받을것인가, 아니면 허용하지 않을것인가의 문제죠. 개인적으로는 프로그래밍 과정에서 타입을 점점 좁혀가면 오류의 여지를 줄일 수 있기 때문에 후자를 선호합니다. 그렇지 않으면 강력한 타입시스템을 갖춘 언어를 굳이 사용할 필요가 없겠지요.

타입을 이용한 오류방지 방법 관련해서 [Improving Correctness with Types](http://www.slideshare.net/IainHull/improving-correctness-with-types) 를 읽어보시길 권합니다.

## Monoid Example: Filter

간단한 `Monoid` 예제를 나 만들어 보겠습니다. `User` 클래스가 있고, 필터링을 하고 싶을 때

```scala
// http://www.slideshare.net/oxbow_lakes/practical-scalaz

case class User(name: String, city: String)
type Filter[A] = A => Boolean // Function1, same as Reader[A, Boolean]

val london: Filter[User] = _.city endsWith(".LONDON")
val ny: Filter[User]     = _.city endsWith(".NY")

val inLondon = users filter london
val inNY = users filter ny
```

이 때 만약 `Filter[A]` 가 `OR (||)` 연산에 대한 모노이드라면, 이렇게 쓸 수 있지 않을까요?

```scala
users filter (london |+| ny)
```

그런데 `Filter[A]` 는 모노이드가 아니기 때문에 그럴 수 없습니다. 우린 모노이드를 배운 사람들이니까 ~~지성인~~ 한 번 만들어 보겠습니다.

```scala
implicit def booleanMonoid[A] = new Monoid[Filter[A]] = {
  override def zero: Filter[A] =
    false
  override def append(f1: Filter[A], f2: => Filter[A]): Filter[A] =
    a => f1(a) || f2(a)
}
```

*disjunction* ´죠? *Scalaz* 어딘가에 구현되어 있을것 같습니다.

```scala
impilcit def booleanMonoid[A] =
  function1Monoid[A, Boolean](booleanInstance.disjunction)
```

`function1Monoid[A, R]` 은 결과값 `R` 에 대한 모노이드 `Monoid[R]` 를 필요로 하고 여기에 위에서 봤던 `Monoid[Boolean]` 인 `booleanInstance.disjunction` 을 넣으면, 우리가 원했던 `Monoid[Filter[A]` 가 완성됩니다.

```scala
implicit def function1Monoid[A, R](implicit R0: Monoid[R]): Monoid[A => R] = new Function1Monoid[A, R] {
  implicit def R = R0
}

private trait Function1Monoid[A, R] extends Monoid[A => R] with Function1Semigroup[A, R] {
  implicit def R: Monoid[R]
  def zero = a => R.zero
}

object disjunction extends Monoid[Boolean] {
    def append(f1: Boolean, f2: => Boolean) = f1 || f2
    def zero = false
}
```

그러면 이제 요구사항을 좀 더 까다롭게 해서, **런던에 사는 켈리 또는 뉴욕에 사는 켈리** 만 뽑아내려면 어떻게 해야할까요?

```scala
// if we have `|*|` representing `Conjunction`

val kelly: Filter[User] = _.name.endsWith("Kelly")
val myFriendKelly = (london |*| kelly) |+| (ny |*| kelly)
users filter myFriendKelly
```

그런데, *scalaz* 에서 할당한 모노이드 연산자는 `|+|` 하나뿐입니다. 따라서 *Implicit Class* 를 추가하면

```scala
implicit class FilterOps[A](fa: Function1[A, Boolean]) {
  def |*|(other: Function1[A, Boolean]): Function1[A, Boolean] =
    function1Monoid[A, Boolean](booleanInstance.conjunction).append(fa, other)
}

val users = List(
  User("Kelly", ".LONDON"),
  User("John", ".NY"),
  User("Cark", ".SEOUL"),
  User("Kelly", ".NY"),
  User("Kelly", ".SEOUL")
)

val ks1 = users filter ((london |*| isKelly) |+| (ny |*| isKelly))
val ks1.size shouldBe 2

// 더 짧게 줄이면,
val ks2 = users filter ((london |+| ny) |*| isKelly)
```

`scalaz.Monoid` 가 `|+|` 만을 지원하는 반면, 대수타입에 특화된 *Spire* 는 `Boolean` 에 대해 `*, +` 두 가지 연산을 모두 지원합니다.

```scala
import spire.algebra.Rig

implicit def filterRig[A] = new Rig[Filter[A]] {
  def plus(x: Filter[A], y: Filter[A]): Filter[A] = v => x(v) || y(v)
  def one: Filter[A] = Function.const(true)
  def times(x: Filter[A], y: Filter[A]): Filter[A] = v => x(v) && y(v)
  def zero: Filter[A] = Function.const(false)
}

import spire.syntax.rig._

users filter ((london + ny) * kelly)
```

## Monoid with BooleanW, OptionW and Endo

`Boolean` 과 `Option` 은, 연산에 `if-else`, `getOrElse` 처럼  **다른 경우** 를 내포하기 때문에, `Monoid.zero` 와 엮으면 쏠쏠하게 써먹을 수 있습니다.

```scala
> load.ivy("org.scalaz" % "scalaz-core_2.11" % "7.2.0-M5")

> import scalaz._, Scalaz._
import scalaz._, Scalaz._

> ~ 1.some      // Some(1).getOrElse(Monoid[Int].zero)
res5: Int = 1
> ~ none[Int]   // None.getOrElse(Monoid[Int].zero)
res6: Int = 0
> none[Int] | 3 // None.getOrElse(3)
res7: Int = 3
```

`Boolean` 연산도 살펴보면,

```scala
(true  ? 1 | 2) shouldBe 1
(false ? 1 | 2) shouldBe 2
(true  ?? 1) shouldBe 1
(false ?? 1) shouldBe 0 /* raise into zero */
(true  !? 1) shouldBe 0 /* reversed `??` */
(false !? 1) shouldBe 1
```

`??` 는 조건이 참일경우, `A` 를 아닐 경우 `Monoid[A].zero` 를 돌려줍니다.

```scala
final class BooleanOps(self: Boolean) {
  ...
  final def ??[A](a: => A)(implicit z: Monoid[A]): A = b.valueOrZero(self)(a)
  final def !?[A](a: => A)(implicit z: Monoid[A]): A = b.zeroOrValue(self)(a)
  ...
}

trait BooleanFunctions {
  ...
  final def valueOrZero[A](cond: Boolean)(value: => A)(implicit z: Monoid[A]): A =
    if (cond) value else z.zero
  final def zeroOrValue[A](cond: Boolean)(value: => A)(implicit z: Monoid[A]): A =
    if (!cond) value else z.zero
  ...
}
```

[Practical Scalaz](http://www.slideshare.net/oxbow_lakes/practical-scalaz) 에서는 `Endo` 와 엮어 다음처럼 사용하는걸 보여줍니다. (`new Filter` 부분을 추출하는것이 더 나은것 같습니다만, 그냥 이렇게도 사용할 수 있다 정도로 알고만 계시면 될 것 같습니다.)

```xml
// http://www.slideshare.net/oxbow_lakes/practical-scalaz

<instruments filter="incl">
  <symbol value="VOD.L" />
  <symbol value="MSFT.O" />
</instruments>
```

```scala
// before
for {
  e <- xml \ "instrument"
  f <- e.attribute("filter")
} yield
  (if f == "incl") new Filter(instr(e)) else new Filter(instr(e)).neg)

// after
val reverseFilter = Endo[Filter](_.neg)

for {
  e <- xml \ "instrument"
  f <- e.attribute("filter")
} yield
  (f == "incl") !? reverseFilter apply new Filter(instr(e))
```

참고로 `Endo` 는 `Function1[A, A]` 입니다. 따라서 `Monoid[Endo[A]]` 는 *identity function* 입니다.

```scala
final case class Endo[A](run: A => A) {
  final def apply(a: A): A = run(a)

  /** Do `other`, than call myself with its result. */
  final def compose(other: Endo[A]): Endo[A] = Endo.endo(run compose other.run)

  /** Call `other` with my result. */
  final def andThen(other: Endo[A]): Endo[A] = other compose this
}

trait EndoFunctions {
  /** Alias for `Endo.apply`. */
  final def endo[A](f: A => A): Endo[A] = Endo(f)

  /** Alias for `Monoid[Endo[A]].zero`. */
  final def idEndo[A]: Endo[A] = endo[A](a => a)

  ...
}
```

## Example: Currency

이제까지 배워왔던 바를 적용해서, 통화를 나타내는 `Currency` 모델을 만들어 보겠습니다. 위에선 `Tag` 를 이용했었으니, 이번엔 *[Value Class](http://docs.scala-lang.org/overviews/core/value-classes.html)* 로 만들어 보겠습니다.

```scala
object Currency {
  sealed trait Currency extends Any
  final case class EUR[A](amount: A) extends AnyVal with Currency
  final case class USD[A](amount: A) extends AnyVal with Currency
}

// spec
USD(1) =:= USD(1)
USD(3) =:= EUR(2) // compile error
```

이제 `1.USD` 등 의 문법을 위해 *implicit class* 를 추가하면,

```scala
Object Currency {
  ...

  implicit class CurrencyOps[A](amount: A) {
    def EUR = Currency3.EUR(amount)
    def USD = Currency3.USD(amount)
  }
}

// spec
10.USD =:= 10.USD
```

이제 같은 통간 덧셈을 위해, `Monoid[USD[A]]` 등을 추가할 수 있습니다. `|+|` 는 기존의 `Monoid[A]` 를 이용하면 됩니다.

```scala
object Currency {
  import scalaz._, Scalaz._

  ...
  implicit def usdMonoid[A](implicit M: Monoid[A]) = new Monoid[USD[A]] {
    override def zero: USD[A] =
      USD(M.zero)

    override def append(u1: USD[A], u2: => USD[A]): USD[A] =
      USD(M.append(u1.amount, u2.amount))
  }
}

// spec
(10.USD |+| 10.USD) =:= 20.USD
```

이제 `EUR` 를 위한 모노이드를 만들어 보겠습니다. 재미삼아 *[context bound](http://docs.scala-lang.org/tutorials/FAQ/context-and-view-bounds.html)* 를 이용해 보면,

```scala
object Currency {
  ...

  implicit def eurMonoid[A : Monoid] = new Monoid[EUR[A]] {
    override def zero: EUR[A] =
      EUR(implicitly[Monoid[A]].zero)

    override def append(e1: EUR[A], e2: => EUR[A]): EUR[A] =
      EUR(implicitly[Monoid[A]].append(e1.amount, e2.amount))
  }
}
```

통화가 추가될때 마다 매번 반복적으로 ª¨노이드를 추가해야된다는 것이 귀찮으므로, `Currency` 용 모노이드를 만들겠습니다. *[Shapeless](https://github.com/milessabin/shapeless)* 를 이용하면, (*Shapeless* 의 `Generic`, `Aux` 는 아래에서 설명하겠습니다)

```scala
object Currency {
  import scalaz._, Scalaz._
  import shapeless._

  ...
  implicit def currencyMonoid[A : Monoid, C[_] <: Currency]
  (implicit G: Generic.Aux[C[A], A :: HNil]) = new Monoid[C[A]] {
    override def zero: C[A] =
      G.from(implicitly[Monoid[A]].zero :: HNil)

    override def append(c1: C[A], c2: => C[A]): C[A] = {
      val a1: A = G.to(c1).head
      val a2: A = G.to(c2).head

      G.from(implicitly[Monoid[A]].append(a1, a2) :: HNil)
    }
  }
}
```

이제 통화간 변환을 위한 함수를 추가해보도록 하겠습니다. 이런 문법은 어떨까요?

```scala
12.USD to EUR
```

그런데, 현재 우리가 가진 디자인에서 `EUR` 은 *case class* 이므로 `EUR` 생성없이 타입만 지정하려면 이정 문법으로 타협할 수 있겠네요.

```scala
24.USD to[EUR]
```

`Currency` 에서 `to` 구현을 하려면, `to[C[_] <: Currency[_]]` 정도로 하위 클래스는 퉁친다 해도, 하위 클래스 인스턴스 생성시에 `A` 가 필요하므로 `Currency` 를 `Currency[A]` 로 변경해야 합니다.

```scala
object Currency {
  sealed trait Currency[A] extends Any {
    def amount: A
  }

  final case class EUR[A](amount: A) extends AnyVal with Currency[A]
  final case class USD[A](amount: A) extends AnyVal with Currency[A]

  implicit class CurrencyOps[A](amount: A) {
    def EUR = Currency3.EUR(amount)
    def USD = Currency3.USD(amount)
  }

  implicit def currencyMonoid[A : Monoid, C[A] <: Currency[A]]
  (implicit G: Generic.Aux[C[A], A :: HNil]) = new Monoid[C[A]] {
    override def zero: C[A] =
      G.from(implicitly[Monoid[A]].zero :: HNil)

    override def append(c1: C[A], c2: => C[A]): C[A] = {
      val a1: A = G.to(c1).head
      val a2: A = G.to(c2).head

      G.from(implicitly[Monoid[A]].append(a1, a2) :: HNil)
    }
  }
}
```

이제 `Currency` 에 `to` 를 추가하면,

```scala
object Currency {
  ...

  sealed trait Currency[A] extends Any {
    def amount: A
    def to[C[A] <: Currency[A]](implicit G: Generic.Aux[C[A], A :: HNil]): C[A] =
      G.from(amount :: HNil)
  }

  ...
}

// spec
(10.USD.to[EUR]) =:= 10.EUR
```

`to` 에 `implicit` 로 통화간 환율을 담고있는 `R: Rate` 등을 추가하고 `Rate` 내에서 `Monoid[A @@ Multiplcation` 을 이용하면 컴파일타임에

- `USD -> EUR` 변환이 정의되어 있는지 ([Shapeless Heterogenous Maps](https://github.com/milessabin/shapeless/wiki/Feature-overview:-shapeless-2.0.0#heterogenous-maps))
- `A` 에 대한 곱셈 연산 `Monoid[A @@ Multiplication]` 이 정의 되어있는지를 검사할 수 있습니다.

~~구현은 숙제로.. 제가 귀찮아서가 절대 아닙니다~~

디자인적인 결정이겠으나, `USD`, `EUR` 등을 `object` 로 만들고 `case class Money[A](amount: A, currency: Currency)` 로 구현할수도 있겠습니다. 관심 있으신 분은 [github.com/lambdista/money](https://github.com/lambdista/money) 를 참조하시면 됩니다.

## Shapeless

*Shapeless* 는 많은 기능을 가지고 있기 때문에 여기서 모든걸 설명하긴 어렵고, 위에서 사용한 `Generic`, `Aux` 에 대해 간단히 소개만 하겠습니다. (관심 있으신 분은 [Shapeless - Feature 2.0.0](https://github.com/milessabin/shapeless/wiki/Feature-overview:-shapeless-2.0.0) 를 참조하시면 됩니다.)

```scala
// https://github.com/milessabin/shapeless/blob/master/core/src/main/scala/shapeless/generic.scala

> load.ivy("com.chuusai" %% "shapeless" % "2.2.5")

> import shapeless._
import shapeless._

> case class Cat(name: String, catAge: Double)
defined class Cat
> Generic[Cat]
res4: Generic[Cat] {
  type Repr =
    shapeless.::[String,shapeless.::[Double,shapeless.HNil]]
} = ...
```

`Generic[A]` 는 [Path-Dependent Type](http://danielwestheide.com/blog/2013/02/13/the-neophytes-guide-to-scala-part-13-path-dependent-types.html) 으로 `Repr` 을 가지고 있습니다. 이는 `A` 에 따라 달라지는 값인데, 보통 `R` 로 표기합니다.

```scala
// https://github.com/milessabin/shapeless/blob/master/core/src/main/scala/shapeless/generic.scala#L103

trait Generic[T] extends Serializable {
  /** The generic representation type for {T}, which will be composed of {Coproduct} and {HList} types  */
  type Repr

  /** Convert an instance of the concrete type to the generic value representation */
  def to(t : T) : Repr

  /** Convert an instance of the generic representation to an instance of the concrete type */
  def from(r : Repr) : T
}
```

`Generic.Aux[A, R]` 는 `Generic[A]` 의 `Repr` 에 `R` 을 사용하는것으로, `Generic[A] { type Repr = R }` 과 동일합니다.

```scala
// https://github.com/milessabin/shapeless/blob/master/core/src/main/scala/shapeless/generic.scala#L148

object Generic {
  ...

  type Aux[T, Repr0] = Generic[T] { type Repr = Repr0 }

  ...
}
```

`Generic.Aux[A, R]` 을 이용하면, 타입수준의 표현 `R` 과 실제 타입 `A` 간*isomorphic* 변환을 수행할 수 있습니다. 위에서 봤던 `to` 와 `from` 기억 하시죠?

만약 `R` 이 기본적인 타입이어서, `Generic.Aux[A, R]` 이 Shapeless 에서 자동 생성해 줄 경우 `Currency` 예제에서 보았듯이 `implicit` 로 가져오면, 바로 이용할 수 있습니다.

*primitive* 는 물론 *case class* 도 `Generic[Cat]` 처럼 자동생성되어 바로 가져다 쓸 수 있습니다. 중첩된것두 가능하구요.

```scala
> case class EnhancedCat(catType: String, cat: Cat)
defined class EnhancedCat

> Generic[EnhancedCat]
res6: Generic[EnhancedCat] {
  type Repr = shapeless.::[String,shapeless.::[cmd3.Cat,shapeless.HNil]]
} = ...
```

여기서 `HList` 는 ([Heterogenous List](https://github.com/milessabin/shapeless/wiki/Feature-overview:-shapeless-2.0.0#heterogenous-lists)) 여러 타입을 담을 수 있는 리스트입니다.

이제 `to` 와 `from` 예제를 보´

```scala
> val c1 = Cat("odie", 1.0)
c1: Cat = Cat("odie", 1.0)

> Generic[Cat].to(c1)
res9: String :: Double :: HNil = ::("odie", ::(1.0, HNil))

> val reconstructed = Generic[Cat].from(res9)
reconstructed: Cat = Cat("odie", 1.0)

> case class Dog(name: String, dogAge: Double)
defined class Dog

> val d1 = Dog("dog odie", 1.0)
d1: Dog = Dog("dog odie", 1.0)

> Generic[Dog].to(d1)
res13: String :: Double :: HNil = ::("dog odie", ::(1.0, HNil))

> val reconstructedFromDog = Generic[Cat].from(res13)
reconstructedFromDog: Cat = Cat("dog odie", 1.0)
```

[metaplasm.us - Type Classes and Generic Derivation](https://meta.plasm.us/posts/2015/11/08/type-classes-and-generic-derivation/) 에서는 *Shapeless* 를 이용해서 문자열로부터 *case class* 를 자동생성하는 파서를 만드는 법을 보여줍니다.

`CaseClassParser` 가 있을 때, 문자열 `"odie, 1.2"` 를 `Dog` 로 파싱하기 위해 `CaseClassParser[Dog]("odie, 1.2")` 처럼 쓰고싶다고 하면,

```scala
// ref - https://meta.plasm.us/posts/2015/11/08/type-classes-and-generic-derivation/

object CaseClassParser {
  import shapeless._

  trait Parser[A] {
    def apply(s: String): Option[A]
  }

  def apply[A](s: String)(implicit P: Parser[A]): Option[A] = P(s)
}
```

이 때 `shapeless.Generic[A]` 를 이용하면 위에서 보았듯이 `A` 를 `HList` 로 ([Heterogenous List](https://github.com/milessabin/shapeless/wiki/Feature-overview:-shapeless-2.0.0#heterogenous-lists)) 로 변경할 수 있으므로 `Parser[HList]` 만 있으면 됩니다.

`HList` 도 `List` 처럼 `cons` 와 `nil` 로 구성되어 있습니다. `HNil` 과 `HList` 파서를 만들면,

```scala
// ref - https://meta.plasm.us/posts/2015/11/08/type-classes-and-generic-derivation/

object CaseClassParser {
  ...

  implicit val hnilParser = new Parser[HNil] {
    override def apply(s: String): Option[HNil] =
      if (s.isEmpty) Some(HNil) else None
  }

  implicit def hlistParser[H : Parser, T <: HList : Parser] = new Parser[H :: T] {
    override def apply(s: String): Option[H :: T] =
      s.split(",").toList match {
        case cell +: rest /* use `+:` instead of :: */ => for {
          head <- implicitly[Parser[H]].apply(cell)
          tail <- implicitly[Parser[T]].apply(rest.mkString(","))
        } yield head :: tail
      }
  }
}
```

그리고 `implicitly[Parser[H]]` 에서 사용할 개별 타입별 파서를 만들면

```scala
// ref - https://meta.plasm.us/posts/2015/11/08/type-classes-and-generic-derivation/

object CaseClassParser {
  ...

  implicit val intParser = new Parser[Int] {
    override def apply(s: String): Option[Int] = Try(s.toInt).toOption
  }

  implicit val stringParser = new Parser[String] {
    override def apply(s: String): Option[String] = Some(s)
  }

  implicit val doubleParser = new Parser[Double] {
    override def apply(s: String): Option[Double] = Try(s.toDouble).toOption
  }
}
```

마지막으로, *case class* 를 `HList` 로 만들어줄 `caseClassParser` 만 만들면 됩니다.

```scala
// ref - https://meta.plasm.us/posts/2015/11/08/type-classes-and-generic-derivation/

object CaseClassParser {
  ...

  implicit def caseClassParser[C, R <: HList]
  (implicit G: Generic.Aux[C, R], reprParser: Parser[R]): Parser[C] = new Parser[C] {
    override def apply(s: String): Option[C] = reprParser.apply(s).map(G.from)
  }
}
```

`reprParser.apply(s)` 는 `Option[R]` 이므로 `G.from` 을 이용해 변환해주면 됩니다.

## Previous Posts

- [Easy Scalaz 1, State](http://1ambda.github.io/easy-scalaz-1-state/)
- [Easy Scalaz 2, Monad Transformer](http://1ambda.github.io/easy-scalaz-2-monad-transformer/)
- [Easy Scalaz 3, ReaderWriterState with Kleisli](http://1ambda.github.io/easy-scalaz-3-readerwriterstate-with-kleisli/)
- [Easy Scalaz 4, Yoneda and Free Monad](http://1ambda.github.io/easy-scalaz-4-yoneda-and-free-monad/)

## References

- [Haskell Image](http://cs.lth.se/edan40)
- [Learning Scalaz: Tagged Type](http://eed3si9n.com/learning-scalaz/Tagged+type.html)
- [Scalaz 7.2: Tag.scala](https://github.com/scalaz/scalaz/blob/series/7.2.x/core/src/main/scala/scalaz/Tag.scala)
- [Scalaz Google Groups: Value Class vs Tag](https://groups.google.com/forum/#!topic/scalaz/Py_IIfp9d2Q)
- [Scala Docs: Value Classes](http://docs.scala-lang.org/overviews/core/value-classes.html)
- [Scala Docs: Context Bound](http://docs.scala-lang.org/tutorials/FAQ/context-and-view-bounds.html)
- [Practical uses for Unboxed Tagged Types](http://etorreborre.blogspot.kr/2011/11/practical-uses-for-unboxed-tagged-types.html)
- [Improving Correctness with Types](http://www.slideshare.net/IainHull/improving-correctness-with-types)
- [Underscore: Unboxed Tagged Angst](http://underscore.io/blog/posts/2014/01/29/unboxed-tagged-angst.html)
- [Slideshare: Practical Scalaz](http://www.slideshare.net/oxbow_lakes/practical-scalaz)
- [Stackoverflow: Tagged Type Comprarison in Scalaz](http://stackoverflow.com/questions/34266285/tagged-type-comparison-in-scalaz)
- [Shapeless: Feature 2.0.0](https://github.com/milessabin/shapeless/wiki/Feature-overview:-shapeless-2.0.0)
- [The Neophyte's Guide to Scala Part 13: Path-Dependent Type](http://danielwestheide.com/blog/2013/02/13/the-neophytes-guide-to-scala-part-13-path-dependent-types.html)
- [metaplasm.us: Type classes and generic derivation](https://meta.plasm.us/posts/2015/11/08/type-classes-and-generic-derivation/)
- [Stackoverflow: Constructing simple Scala case classes from Strings, strictly without boiler-plate](http://stackoverflow.com/questions/33585441/constructing-simple-scala-case-classes-from-strings-strictly-without-boiler-pla/33586304#33586304)
