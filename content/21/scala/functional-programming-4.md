+++
date = "2016-06-25T00:47:10+09:00"
next = "../functional-programming-5"
prev = "../functional-programming-3"
title = "Functional Programming 4"
toc = true
weight = 114
aliases = [
    "/functional-programming-in-scala-chapter-4"
]
+++

## Types and Pattern Matching

### Functions as Objects


> In fact function values are treated as objects in Scala

```scala
trait Function1[A, B] {
  def apply(x: A): B
}
```

결국, `function` 은 `apply` 메소드를 가진 오브젝트다.

예를 들어서 `(x: Int) => x * x` 는 다음과 같이 `Function1` ` *trait* 를 구현한 클래스 된다.

```scala
{
  class AnonFun extends Function1[Int, Int] (
    def apply(x: Int) = x * x
  }

  new AnonFun
}

// or using anonymous class syntax
// trait can be instanciated

new Function1[Int, Int] {
  def apply(x: Int) = x * x
}
```

따라서 이런 정의를 보면, `f(a, b)` 는 `f.apply(a, b)` 다. 그런데 만약 메소드인 `def apply` 자체도 오브젝트일까? 그렇지 않다. 만약 `apply` 자체도 오브젝트라면, 그 오브젝트도 `apply` 메소드를 가지고 있어야 하고, 또 다시 그렇게 반복될 수 있다.

#### Functions and Methods

따라서 `def f(x: Int): Int = ...` 메소드 자체는 **Function** 이 아니¤. (*Method != Function*) 그러나

> If `f` is used in a place where a `Function` type is expected, it is converted automatically to the function value

예를 들어, `(x: Int) = f(x)` 는 다음과 같이 확장된다.

```scala
new Function1[Int Int] {
 def apply(x: Int) = f(x)
}
```

이렇게 *Method* 가 *Function* 으로 변환되는 과정을 *lambda calculus* 에서는 **eta-expansion** 이라 부른다.

그리고 위에서 보았겠지만 `apply` 메소드는, 오브젝트에 있을때 오브젝트 이름 자체로 호출될 수 있도록 해준다. 예를들어 지난 시간에 만들었던 `List` *trait* 에 대해서,

```scala
trait List[T] {
  def isEmpty: Boolean
  def head: T
  def tail: List[T]
}

class Cons[T](val head: T, val tail: List[T]) extends List[T] {
  def isEmpty = false
}

class Nil[T] extends List[T] {
  def isEmpty = true
  def head = throw new NoSuchElementException("Nil.head")
  def tail = throw new NoSuchElementException("Nil.tail")
}
```

아래와 같은 호출을 한다면

```scala
val e = List()
val e3 = List(3)
val e34 = List(3, 4)
```

다음과 같이 `List` *Object* 와 `apply` *Method* 를 정의할 수 있다.

```scala
object List {
  def apply() = new Nil
  def apply(x: Int) = new Cons(x, new Nil)
  def apply(x: Int, y: Int) = new Cons(x, new Cons(y, new Nil))
}
```

### Subtyping and Generics

지난시간에는 *Polymorphism* 의 두가지 형태에 대해 배웠었다. 하나는 **Subtyping** 이고, 다른 하나는 **Generics** 다. 기억을 더듬어 보면

> **Subtyping:** Instance of a subclass can be passed to a base class  
> **Generics:** Instance of a function or class are created  by type parameterazation

이 중에서 *subtyping* 은 OOP 에서 먼저 온 것이고, *generics* 는 *FP* 에서 먼저 온 것이라는 이야기 까지 했다.

#### Type Bounds

`assertAllPos` 메소드가 있다고 하자. `IntSet` 을 취해서, 모든 Element 가 양수면 `IntSet` 을 리턴하고 아니면 예외를 던진다. 다음과 같이 정의할 수 있겠다.

`def assertAllPos(s: IntSet): IntSet`

근데, 만약에 이 메소드가 `Empty` 를 받으면 `Empty` 를, `NonEmpty` 를 받으면 `NonEmpty` 를 돌려주게 하려면 어떻게 해야할까? 메소드를 2개를 더 만들어야 할까? *Type Bound* 를 이용해 문제를 해결할 수 있다.

`def assertAllPos[S <: IntSet](r: S): S = ...`

여기서 `S <: IntSet` 은 `S` 가 `IntSet` 의 서브타입임을 말하고, 이것을 **Upper bound** 라 부른다(`IntSet` 기준). 즉 `S` 의 상위 타입을 지정하는 것이다.

반대로 `S :> IntSet` 도 있을 수 있다. 이것은 **Lower bound** 라 부르며(`IntSet` 기준), 이 메소드가 `IntSet` 의 상위 타입 `S` 를 이용한다는 것을 컴파일 타임에 지정한다. `S` 는 `IntSet`, `AnyRef`, `Any` 가 될 수 있다. 정리 하자면,

> `S <: T` means: **S is a subtype of T**  
> `S :> T` means: **S is a supertype of T, or T is a subtype of S**

**Mixed Bound** 도 있다. `[S >: NonEmpty <: IntSet]`

#### Covariance

그런데, `NonEmpty <: IntSet` 일때 `List[NonEmpty] <: List[IntSet]` 이면, **Covariant** 하다고 말한다. 직관적으로 보면 그럴듯 하다.

이거 정말 문제가 없을까? `List` 말고 자바의 `Array` 를 예로 들어보자.

```java
NonEmpty[] a =
  new NonEmpty[]{new NonEmpty(1, Empty, Empty)}

IntSet[] b = a
b[0] = Empty
NonEmpty s = a[0]
```

`b[0] = Empty` 가 문제가 된다. 여기서 런타임 예외가 발생하는데, 자바에서는 배열이 생성될때, 이 배열이 어떤 타입으로 생성되었는지 내부적으로 태그를 붙인다. 그런데, `NonEmpty` 로 태그가 붙은 배열에 호환되지 않는 `Empty` 를 넣고 있기 때문이다.

자바 1.5 이전에는 Generics 가 없었기 때문에 정렬을 위해서는 `sort(Object[] a)` 처럼 주어야 했는데, 이를 위해서는 자바의 배열이 **Covariant** 여야 했다.

#### The Liskov Substitution Principle

**Liskov Substitution Princile(리스코프 치환원칙)** 은 언제 한 입이 다른 타입의 서브타입이 될 수 있는지 말해준다.

> If `A <: B`, then everything one can to do with a value of type `B` one should also be able to do with a value of type `A`

자 이제, Scala 에서 위의 코드를 작성하면 어디서 에러가 나는지 확인 해 보자.

```scala
val a: Array[NonEmpty] = Array(new NonEmpty(1, Empty, Empty))
val b Array[IntSet] = a
b(0) = Empty
val s: NonEmpty = a(0)
```

`val b: Array[IntSet] = a` 에서 컴파일 타임 에러가 난다. 왜냐하면 Scala 의 `Array` 는 **Not** *covariant* 이기 때문이다.

### Variance

어떤 타입은 *covariant* 고 어떤 타입은 그렇지 않은걸까? 엄격히 말해서, elements 들의 *mutation* 을 허용하는 타입은 *not-covariant* 여야 한다.

반면 *immutable types* 은 조건이 갖춰지면 *covariant* 일 수 있다. 위에서 `List` 는 되고, `Array`는 안되었던 것처럼.

`C[T]` 가 있고, `A <: B` 일때 다음과 같은 정의를 내릴 수 있다.

- `C[A] <: C[B]` 이면, **C is covariant**, `C[+A]` 로 표시
- `C[A] >: C[B]` 이면, **C is contravariant**, `C[-A]` 로 표시  
- `C[A]` 와 `C[B]` 가 상관이 없으면, **C is non-variant**, , `C[A]` 로 표시

그렇다면 다음과 같은 두개의 타입이 있을때, 어떤것이 서브타입이고 어떤  것이 슈퍼타입일까?

```scala
type A = IntSet => NonEmpty
type B = NonEmpty => IntSet
```

설명을 조금 자세히 하면, 함수의 파라미터는 *Contravariant* 하고, 함수의 리턴타입은 *Covariant* 하다. `Function1[-A, +B]` 를 보면 알 수 있다. 따라서 `A <: B` 다. 왜 그럴까? 여기 [Scala School](https://twitter.github.io/scala_school/type-basics.html#variance) 의 예제를 좀 보자.

```scala
class Animal {
  val sound = "rustle"
  def name = "animal"
}

class Bird extends Animal {
  override val sound = "call"
  def name = "bird"
}
class Chicken extends Bird {
  override val sound = "cluck"
  def name = "chicken"
}
```

이때, `val getTweet: (Bird => String)` 에 `(c: Chicken =>  c.chicken` 과 같이 넘겨주고, 나중에 `getTweet(new Bird)` 를 호출하면 에러가 난다. 반면 `a: Animal => a.name` 을 주고, `getTweek(new Bird)` 는 상관 없다. 어차피 `Bird` 는 `Animal` 이니까

따라서 함수의 파라미터는 현재와 같은 타입이거나, 혹은 그 슈퍼타입이어야 한다, 다시말해서 **Contravariant** 해야 한다.

#### Variance Checks

`Array` 의 경우 `update` 연산이 문제가 될 수 있다는걸 위에서 논의 했었는데, 이걸 정리하자면

> *The problematic combination is  
> **the covariant type parameter T**  
> **which appears in parameter position of method `update`**

즉 *covariant* 타입 `T` 가 `update` 연산에 나타날때 문제가 된다. 그래서 Scala 는 컴파일 타임에 이런 문제가 생기지 않는지 검사를 한다.

> *covariant* type parameters can only appear in method results  
> *contravariant* type parameters can only appear in method parameters  
> *invariant* type parameters can appear anywhere  

`Function1` *Trait* 는 그래서 사실 이런 모양이다.

```scala
package scala
trait Function1[-T, +U] {
  def apply(x: T): U
}
```

`T` 는 *contravariant* 이므로 파라미터에만, `U` 는 *covaraint* 이므로 리턴타입에만 나타난다. 이제 지난시간에 만들었던 `Nil` 클래스를 *Object* 로 만들어 보자.

```scala
trait List[+T] {
  def isEmpty: Boolean
  def head: T
  def tail: List[T]
}

class Cons[T](val head: T, val tail: List[T]) extends List[T] {
  def isEmpty = false
}

object Nil extends List[Nothing] {
  def isEmpty = true
  def head = throw new NoSuchElementException("Nil.head")
  def tail = throw new NoSuchElementException("Nil.tail")
}
```

`Nil` 이 `List[Nothing]` 을 상속하게 해, 모든 리스트의 서브타입이 될 수 있도록 했다. 그러나 이것만으로는 부족하다. `trait List[T]` 로 만들면, `List[Something]` 과 `List[Nothing]` 과는 아무 관련이 없는 *non-variant* 다. 따라서 `List[+T]` 로 만들어, `List[Nothing] <: List[Something]` 이 되도록 해야한다.

이제, 다음과 같은 `prepend` 메소드를 고려 해 보자.

```scala
trait List[+T] {
  ...
  def preprend(elem: T): List[T] = new Cons(elem, this)
  ...
}
```

이 경우에는 *covariant* `T` 가 파라미터에 나오므로, 컴파일이 실패한다. 그러나 우리의 `List` 는 *immutable* 한데, 이 경우 파라미터에 `T` 가 나오면 안되나?

#### Prepend Violates LSP

```scala
val xs = new List[IntSet]
xs.prepend(Empty)

val ys = new List[NonEmpty]
xs.prepend(Empty) // compilation fail
```

따라서, `List[IntSet]` 으로 할 수 있는걸 `List[NonEmpty]`로 할 수 없으니, **LSP** 에 따라서, `List[NonEmpty]` 는 `List[IntSet]` 의 서브타입이 될 수 없다.

`List` 는 *covariant* 하고, `update` 연산이 있는것도 아니므로 `immutable` 한데, `prepend` 메소드의 타입체킹이 문제다. 어떻게 하면 *covaraint* 타입 `T` 를 메소드 파라미터로 나타나게   수 있을까? **Lower Bound** 를 이용하면 된다.

```scala
def prepend[U :> T](elem: U): List[U] = new Cons(elem, this)
```

이 경우 `List[NonEmpty].prepend(Empty)` 의 리턴값은 `List[IntSet]` 이 될것이다. 따라서 룰을 정리하면

> (1) **covariant type parameters** may appear in **lower bounds** of method type parameters  
> (2) **contravariant type parameters** may appear in **upper bounds** of method

### Objects Everywhere

#### Pure Object Orientation

*Pure OO language* 는 모든 *value* 가 *object* 다. 스칼라는 얼핏 보기에 *primitive type* 을 사용하는 것 같지만 잘 보면 `scala.Boolean`, `scala.Int` 처럼 기본 타입이 클래스화 되어있다. (참고로 `Int` 는 성능을 위해 32-bit Integer 로 되어있다.)

`scala.Boolean` 대신, 직접 만든 `Boolean` 을 사용해 보자.

```scala
abstract class cBoolean {

  def IfThenElse[T](t: T, e: T): T

  def &&(other: cBoolean) = IfThenElse(other, False)
  def ||(other: cBoolean) = IfThenElse(True, other)
  def unary_! : cBoolean = IfThenElse(False, True)

  def ==(other: cBoolean) = IfThenElse(other, other.unary_!)
  def !=(other: cBoolean) = IfThenElse(other.unary_!, other)

  def <(other: cBoolean) = IfThenElse(False, other)
  def >(other: cBoolean) = IfThenElse(other.unary_!, False)
}

object True extends cBoolean {
  def IfThenElse[T](t: T, e:T) = t
}

object False extends cBoolean {
  def IfThenElse[T](t: T, e:T) = e
}
```

그럼 과연, *primitive type* 없이 언어의 모든 부분을 클래스와 오브젝트로 구성하는것이 가능할까? `Boolean` 은 우리가 `cBoolean` 으로 대체했다. `Int` 부터 더 자그마한 `Nat`, 즉 자연수 범위부터 시작해보자.

> Can it be represented as a class from first principles (i.e not using primitie ints)

```scala
abstract class Nat {
  def isZero: Boolean
  def predecessor: Nat
  def successor = new Succ(this)
  def + (that: Nat): Nat
  def - (that: Nat): Nat
}

object Zero extends Nat {
  def isZero = true
  def predecessor = throw new RuntimeException("Zero.predecessor");
  def + (that: Nat) = that
  def - (that: Nat) = {
    if (that.isZero) this
    else throw new RuntimeException("Zero.-")
  }
}

class Succ(n: Nat) extends Nat {
  def isZero = false
  def predecessor: Nat = n
  def + (that: Nat) = new Succ(n + that)
  def - (that: Nat) = if(that.isZero) this else n - that.predecessor
}
```

숫자가 없을때 숫자를 추상화(abstraction) 할 방법을 찾아야 하는데, 놀랍게도 인스턴스의 중첩을, 숫자로 표현했다. 개인적으로 기가막힌다. 4강 초반부에서 집합을 predicate function 의 접합(`||`, `&&`)으로 표현했을때도 놀라웠는데..


위 코드에서 `Succ.-` 메소드 같은 경우 `new Succ(n - that)` 을 할 수 있는데, `n` 이 `Zero` 즉, 현재 `Succ` 인스턴스가 1인 경우를 고려해야 한다. 이 경우 런타임 예외가 발생하므로, `n - that.predecessor` 가 적절하다. 여기에 `that` 이 `Zero` 일 경우를 고려하면 된다.

테스트를 작성할 경우 `Zero` 를 제외하고는 나머지는 다 인스턴스라서, 비교가 어렵다. 그래서 다음과 같은 연속적인 `predecessor` 를 호출할 수 있는데,

```scala
  val one = Zero.successor
  val two = one + one
  val three = two + one
  val four = three + one

  "One + One" should "be Two" in {
    assert(two.predecessor.predecessor == Zero)
  }

  "two + two" should "be four" in {
    assert(four.predecessor.predecessor.predecessor.predecessor == Zero)
  }

```

아래와 같은 테스트용 유틸리티 함수를 만들면 편하다. (아니면 `==`를 오버라이딩 하거나.)

```scala
  def number = {
    def count(n: Int, succ: Nat): Int = {
      if (succ == Zero) n
      else count(n + 1, succ.predecessor)
    }

    count(0, this)
  }
```

이렇게, 실제 타입에 대한 *primitive value* 없이 *abstraction* 만으로 타입을 구성할 수 있다. 위에서 구현한 `Nat` 클래스를 기술적으로는 **Peano numbers** 라 부른다. 다시 말해서,

> The properties of the natural numbers can e derived from the **Peano axioms**

자세한건 [여기](http://en.wikipedia.org/wiki/Natural_number#Peano_axioms)로


### Decomposition

프로그래밍의 많은 부분이 *Decomposition* 이다. 타입을 비교하고 타입에 따라 처리하는 일들. 다음과 같은 아주 자그마한 컴파일러가 있다고 해 보자.

```scala
object Decomposition {
  def eval(e: Expr): Int = {
    if (e.isNumber) e.numValue
    else if (e.isSum) eval(e.leftOp) + eval(e.rightOp)
    else throw new Error(s"unknown Expr $e")
  }
}

trait Expr {
  def isNumber: Boolean
  def isSum: Boolean
  def numValue: Int
  def leftOp: Expr
  def rightOp: Expr
}

class Number(n: Int) extends Expr {
  def isNumber = true
  def isSum = false
  def numValue = n
  def leftOp = throw new Error("Number.leftOp")
  def rightOp = throw new Error("Number.righOp")
}

class Sum(l: Expr, r: Expr) extends Expr {
  def isNumber = false
  def isSum = true
  def numValue = throw new Error("Sum.numValue")
  def leftOp = l
  def rightOp = r
}
```

다음과 같은 테스트코드를 작성하면, 잘 돌아간다.

```sclaa
  import Decomposition._

  "Sum(Number(3), Number(4))" should "be eql 7" in {
    val three = new Number(3)
    val four = new Number(4)
    val sum = new Sum(three, four)

    assert(eval(sum) == 7)
  }

```

위에서, `isNumber` 과 같은 것들을 **Classification**, `rightOp`, `numValue` 같은 것들을 **Accessor** 라 부른다.

문제는 만약 `Prod` 나 `Var` 같은 클래스들이 `Expr` 을 상속했을 때 새로운 **Classification** 과 **Accessor** 를 작성 해야 한다는 거다. 무려 **25** 개나! 단 두개의 클래스만 추가했을 뿐인데..

일반적으로 새롭게 클래스를 정의했을때 메소드는 *quadratic* 으로 증가한다. 이건 큰 문제다.

이걸 해결하는 한가지 방법은, *Type Cast* 와 *Type Test* 를 이용하는거다. 자바에서 사용하는 아래의 두 메소드는

```java
x instansceOf T
(T) x
```

¤칼라에서 다음과 같다.

```
x.isInstanceOf[T]
x.asInstanceOf[T]
```

이 방법을 이용하면 `eval` 함수에서 `if (e.isInstanceOf[Number])` 와 같이 비교할 수 있기 때문에 *Classification Method* 가 필요없다. 그러나, 타입캐스팅에 실패했을 경우 런타임 에러가 발생할 수 있다. 다른 방법은 없을까?

#### Object-Oriented Decomposition

다른 한 가지 방법은, `eval` 에서 타입체킹을 하는게 아니라 각 클래스에 `eval` 메소드를 만드는거다.


```scala
trait Expr {
  def eval: Int
}
```

이 방법의 문제는 `Expr` 에 새로운 메소드를 추가했을때 *Hierarchy* 내에 있는 모든 클래스에 같은 메소드를 작성해야 한다는 것이다.

게다가 `a * b + a * c` 를 `a * (b + 3)` 로 축약하기가 어렵다. 이건 *Non-local simplification* 이기 때문에, sub-tree 를 모두 테스트하고 접근해야한다. *OO Decomposition* 은 `eval` 메소드 구현엔 좋지만, 이런 점에선 문제가 다.

### Pattern Matching

우리는 *Decomposition* 을 해결하기 위해서 3가지 방법을 시도해봤다.

(1) *Classification* and *acess* methods: **quadratic explosion**  
(2) *Type tests* and *Type casts*: **unsafe**, **low-level**  
(3) *OO Decomposition*: **need to touch all classes to add a new method**, **does not work always**  

#### Functional Decomposition with Pattern Matching

사실 *Classification* 이나 *Access* 의 목적은 다음의 두가지라 봐도 충분하다.

> (1) Which subclass was used?  
> (2) What were the arugmnets of the constructor?

따라서 Scala 에서는 **Pattern Matching**, 그리고 그 과정에서 사용하는 **Case class** 를 통해 *Decomposition* 을 우아하게 자동화 한다.

```scala
trait Expr
case class Number(n: Int) extends Expr
case class Sum(l: Expr, r: Expr) extends Expr

def eval(e: Expr): Expr = {
  e match {
    case Number(n) => n
    case Sum(e1, e2) => eval(e1) + eval(e2)
  }
}
```

(1) 패턴에서 사용하는 *Variable(변수)* 는 소문자로 시작해야 한다.
(2) *Variable* 은 두번 사용될 수 없다. `Sum(a, a)` 는 잘못된 패턴이다.  
(3) *Constant(상수)* 는 대문자로 시작해야 하는데, 예외는 `null`, `true`, `false`

`Expr` *Trait* 내부에 `eval` 을 삽입하는 것도 가능하다

```scala
trait Expr {
  def eval: Int = {
    this match {
      case Number(n) => n
      case Sum(e1, e2) => e1.eval + e2.eval
    }
  }
}
```

*Object-Oriented Decomposition* 과 *Functional Decomposition* 모두 장단이 있는데, 만약 메소드를 많이 만드는 경우라면 *Functional Decomposition* 이 더 우월하다. 매 클라스마다 메소드를 만들 필요가 없기 때문이다. 반대로, 클래스를 많이 만드는 경우라면, *OO Decomposition* 이 더 낫다. 클래스를 만들때마다 매번 `eval` 함수를 수정 할 필요 없기 `eval` 을 가진 클래스를 만들면 된다.


### Lists

Scala 에서 `List` 와 `Array` 는 크게 두가지 면에서 다르다, 먼저 `List` 는 *immutable* 이고, *recursive* 인 반면 `Array` 는 *mutable*, *flat* 하다.

```scala
List("apple", "oranges", "pears")
"apple" :: ("oranges" :: ("pears" :: Nil))

List()
Nil
```

#### Right Associativity

좀 더 편하게 하기 위해서, Scala 는 다음과 같은 문법을 제공한다.

```scala
A :: (B :: C)
A :: B :: C
```

이는 Scala 에서 `:` 로 끝나는 *operator* 는 **Right-associative** 이기 때문이다.

또한 `:` 로 끝나는 *operator* 에서는, 우측에 오는것이 본래의 *operand* 다

> Operators ending in `:` are also difference in the they are seen as method calls of the right-hand operand

따라서 다음의 세 라인은 모두 같다.

```scala
1 :: 2 :: 3 :: 4 :: Nil
1 :: (2 :: (3 :: (4 :: Nil)))
Nil.::(4).::(3).::(2).::(1)
```

그러므로 `::` 를 `prepend` 메소드라 보면 된다.

#### List Patterns

`::`(Cons) 연산자를 이용하면 다음과 같은 패턴이 가능하다.

```scala
1 :: 2 :: xs
x :: Nil
List(x) // same as x :: nil
List() // Nil
List(2 :: xs)
```

#### Sorting Lists

**Insertion Sort** 는 재귀를 이용하면 다음과 같이 구현할 수 있다.

```scala
  def isort(xs: List[Int]): List[Int] = {
    xs match {
      case Nil => List()
      case y :: ys => insert(y, isort(ys))
    }
  }

  def insert(x: Int, xs: List[Int]): List[Int] = {
    xs match {
      case Nil => List(x)
      case y :: ys => {
        if (x < y)  x :: xs
        else y :: insert(x, ys)
      }
    }
  }
```

### References

(1) https://twitter.github.io/scala_school/type-basics.html#variance  
