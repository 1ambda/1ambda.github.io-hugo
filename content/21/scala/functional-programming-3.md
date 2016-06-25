+++
date = "2016-06-25T00:47:08+09:00"
next = "../functional-programming-4"
prev = "../functional-programming-2"
title = "Functional Programming 3"
toc = true
weight = 113
aliases = [
    "/functional-programming-in-scala-chapter-3"
]
+++

### 3.1 Class Hierarchies

#### Abstract Classes

**abstract class** 는 다른 언어의 그것과 같다.

```scala
abstract class IntSet {
  def contains(x: Int): Boolean
  def incl(x: Int): IntSet
}

object EmptySet extends IntSet {
  def contains(x: Int) = false
  def incl(x: Int) = new NonEmptySet(x, EmptySet, EmptySet)
}

class NonEmptySet(elem: Int, left: IntSet, right: IntSet) extends IntSet {
  def contains(x: Int) = {
    if (x < elem) left contains x
    else if (x > elem) right contains x
    else true
  }

  def incl(x: Int) = {
    if (x < elem) new NonEmptySet(elem, left incl x, right)
    else if (x > elem) new NonEmptySet(elem, left, right incl x)
    else this
  }
}
```

여기서 재밌는 점은 `incl` 을 수행할때 새로운 서브트리를 매번 만든다는 것인데, 이건 이 프로그램이 `immutable` 하다는 것을 말한다.

이전의 데이터들을 변경하지 않으므로 **persistent data structures** 라 볼 수 있다.

클래스에서는 다른 언어와 마찬지로 **override** 를 이용해서 existing 혹은 non-abstract definition 을 서브클래스에서 **재정의(redefined)** 할 수 있다.  

#### Object Definitions

`object` 키워드를 이용하면 **singleton object** 를 만든다. 그리고 **singleton object** 는 value 이기 때문에, 그 자체로 evaluate 된다. 다시말해 evaluation step 이 수행 될 필요가 없다.

```scala
object EmptySet extends IntSet {
  def contains(x: Int) = false
  def incl(x: Int) = new NonEmptySet(x, EmptySet, EmptySet)
}
```

#### Exercisee
> Write a method `union` for forming the union of two sets. You should implement the following abstract class

```scala
  def union(other: IntSet): IntSet = {
    ((left union right) union other) incl elem
  }
```

해석하면 terminal node 의 경우, union 연산이 `other incl elem` 이 된다. 게다가 매번의 `left union right` 연산은 현재보다 더 작은 단위를 호출하고, `(left union right) union other` 은 적어도 좌측 operand 가 적어 현재보다 1개 작은 elem 을 가지고 있기 때문에 최소한 자기 자신을 다시 호출하지 않는다는 것을 알 수 있다. 따라서 이런 점을 고려하면 함수는 언젠가 끝난다는 것을 알 수 있다.

#### Dynamic Binding

> Object-oriented language implement dynamic method dispatch. This means that the code invoked by a method call depends on the runtime type of the object that contains the method

이렇게 보면, **Dynamic dispatch** 는 ** higher-orher functions** 와 유사한데, 둘 다 static 타임에 어떤 함수가 실행될 지 알 수 없다. 그럼 둘을 섞으면 어떻게 될까?

### 3.2 How Classes Are Organized

스칼라에서 class 들은 **package** 로 관리된다. 스칼라에서 자동으로 임포트하는 것들은

- All members of package `scala` like scala.Int  
- All members of package `java.lang` like java.lang.Object  
- All members of the singleton object  `scala.Predef` like scala.Predef.require  

#### Trait

> A trait is declared like an abstract class, just with trait instead of abstract class

```scala
trait Planar {
  def height: Int
  def width: Int
  def surface = height * width
}
```

**Trait** 는 자바의 **Interface** 와 비슷하지만, fields 와 concrete methods 를 포함할 수 있다는 점에서 더 강력하다. 반면 **Trait** 는 parameter 를 가질 수 없다.

#### Scala's Hierarchy

<p><img src="http://librairie.immateriel.fr/baw/9780596155957/httpatomoreillycomsourceoreillyimages322250.png" /></p>
<p align="center">(http://librairie.immateriel.fr/fr/read_book/9780596155957/ch07s04#scalas-type-hierarchy)</p>

그림을 보면 알겠지만, 가장 상위에 `scala.Any` 가 있고, 그 아래로 기본 타입들은 `scala.AnyVal` 아래에 위치한다. 스칼라의 `Double` 은 자바의 `double` 와 일치한다. `java.lang.Double` 과는 다르다. `java.lang.Double` 은 아래 설명을 보면 알겠지만, `AnyRef` 하위에 위치한다.

레퍼런스 타입은 `scala.AnyRef` 아래에 위치 한다. 그리고 `scala.AnyRef` 는 자바의 `java.lang.Object` 와 동일하다. 모든 스칼라 오브젝트들은 `scala.AnyRef` 를 하위에 위치한다. 받는다.

정리하자면 primitive type 은 `scala.AnyVal` 하위에 있고, object 는 `scala.AnyRef` 하위에 있다고 보면 된다.  

그리고 하위에 보면 `scala.Nothing` 과 `scala.Null` 이 **Trait** 로 존재하는 걸 확인할 수 있다.

<br/>
<p><img src="http://docs.scala-lang.org/resources/images/classhierarchy.img_assist_custom.png" /></p>
<p align="center">http://docs.scala-lang.org/tutorials/tour/unified-types.html</p>

그림이 좀 작긴 한데, 자세히 보면 dotted arrow  가 있는걸 볼 수 있다. 이건 해당 타입이 화살표가 이어진 곳에 있는 타입으로 자동으로 converted 될 수 있는지의 여부다. 따라서 아래와 같은 **REPL** 실행 결과를 얻을 수 있다.

```scala
scala> val a: Byte = 1
a: Byte = 1

scala> val b: Short = a
b: Short = 1

scala> b
res0: Short = 1

scala> val c: Byte = b
<console>:9: error: type mismatch;
 found   : Short
 required: Byte
       val c: Byte = b
                     ^
```

#### Top Types

`Any` 은 모든 타입의 베이스 타입으로, `==`, `!=`, `equals`, `hashCode`, `toString` 등의 메소드를 포함하고 있다.

#### The Nothing Type

<br/>
<p><img src="http://docs.scala-lang.org/resources/images/classhierarchy.img_assist_custom.png" /></p>
<p align="center">http://docs.scala-lang.org/tutorials/tour/unified-types.html</p>


`Nothing` 은 Scala's type hierarchy 가장 아래쪽에 위치하는데, **모든 타입의 subtype** 이다. `Nothing` 은 또한 값이 없는데, 다음의 두 가지 경우 유용하다.

(1). To signal abnormal termination  
(2). As an element type of empty collections. ex) `Set[Nothing]`

참고로, `Exception` 의 타입도 `Nothing` 이다.

### Null

`Null` 은 모든 `scala.AnyRef` 하위에 있는 타입의 서브타입이다.

> Every reference class type also has null as a value.

`null` 의 type 이 바로 `Null` 이다. `Null` 은 `java.lang.Object`, 즉 `scala.AnyRef` 를 상속받는 모든 클래스의 서브타입이기 때문에 `scala.AnyVal` 과는 incompatible 하다.

```scala

scala> null
res1: Null = null

scala> val a:String = null
a: String = null

scala> val b: Int =  null
<console>:7: error: an expression of type Null is ineligible for implicit conversion
       val b: Int =  null
```

참고로 `if (true) 1 else false` 의 타입은 `AnyVal` 인데 `1`과 `false` 의 공통적인 상위 타입은 `AnyVal` 이기 때문이다.

### 3.3 Polymorphism

대부분의 함수형 언어에서 기본적인 데이터 구조는 **immutable linked list** 다. 이건 **Nil** 과 **Cons** 로 구성되어 있는데, **Nil** 은 empty list 를, **Cons** 는 element 를 담고있는 부분을 말한다. 리습의 그것과 같다.

<br/>
<p><img src="http://upload.wikimedia.org/wikipedia/commons/thumb/1/1b/Cons-cells.svg/525px-Cons-cells.svg.png" /></p><p align="center">http://en.wikipedia.org/wiki/Cons</p>

이번시간엔, `Cons` 와 `Nil` 들을 구현해 보자.

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

여기서 클래스에 있는 파라미터, `val head: T` 를 ***Value Parameter*** 라 부른다. `val` 의 경우에는 자동으로 **public getter** 를 만들어 준다.

그리고 `[T]` 에서 `T` 는 ***Type Parameter*** 다. 타입 파라미터는 클래스 뿐만 아니라 함수에도 적용할 수 있는데,

`def signleton[T](elem:T) = new Cons[T](elem, new Nil[T])`

는 정상적으로 컴파일 된다. 물론 스칼라는 강력한 ***Type Inference*** 를 지원하기 때문에

`singleton[Boolean](true)` 대신 스칼라 컴파일러는 type inference 를 이용해서 `singleton(true)` 혹은 `singleton(1)` 를 받아들인다.

#### Types and Evaluation

> Type parameters do not affect evaluation in Scala

재밌게도 Scala 프로그램이 evaluation 될 때  `[T]` 와 같은 **Type parameters** 는 전혀 영향을 미치지 않는다. 왜냐하면 Scala 가 evaluation 전에 모든 **Type parameters** 와 **Type arguments** 를 제거하기 때문이다.

이 과정은 ***Type erasure*** 로 불린다. Java, Scala, Haskell, ML, OCaml 등은 ***Type erasure*** 를 이용하고, 런타임에도 **Type parameters** 를 유지하는 언어는 C++, C#, F# 등이 있다.  

#### Polymorphism

**Polymorphism** 은 *"in many forms"* 라는 뜻이다. 프로그래밍에서는 다음과 같은 의미를 가진다.

> (1) the function can be applied to arguments of many types or <br/>
> (2) the type can have instances of many types

이 정의로부터 두 가지 사실을 끌어낼 수 있는데,

> (1) **subtyping:** instances of a subclass can be passed to a base class  <br/>
> (2) **generics:** instances of a function or class are created by type parameterization

사실 **subtyping** 은 OOP 언어에서 먼저 온 것이고, **generics** 는 함수형 언어에서 온 것이나, Scala 는 모두 사용한다.

중요한 내용이므로 다시 한 번 정리 하면 **다형성** 이란, 다양한 형태를 가지고 있다는 뜻인데, 프로그래밍에서는 다음과 같은 의미를 지닌다.

(1) 함수는 다양한 타입의 인자를 받아들일 수 있다.  
(2) 타입은 다양한 타입의 인스턴스를 가질 수 있다.

따라서 함수나 클래스의 인스턴스는 ***type parameterization*** 을 통해 생성될 수 있으며, 하위 클래스의 인스턴스는 상위 클래스로서 동작할 수 있다.

#### Exercise

> Write a function `nth` that takes an interger `n` and `a` list and selects the n'th element of the list<br/> <br/>
> If index is outside the range from 0 up the length of the list minus one, a `IndexOutOfBoundsException` should be thrown

```scala
def nth[T](n: Int, list: List[T]): T = {
  if (list.isEmpty) throw new IndexOutOfBoundsException("out of bound index")
  else if (n == 0) list.head
  else nth(n - 1, list.tail)
}
```
