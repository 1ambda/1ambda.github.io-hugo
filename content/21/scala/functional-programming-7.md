+++
date = "2016-06-25T00:47:14+09:00"
prev = "../functional-programming-6"
title = "Functional Programming 7"
toc = true
weight = 117
aliases = [
    "/functional-programming-in-scala-chapter-7"
]
+++

# Functional Programming in Scala 7

7주차에 걸친 대장정의 마지막이다. 이번시간에는 *stream*, *lazy evaluation* 에 대해 배우고 이걸 이용해 길이가 무한인 컬렉션을 만들어 보기도 하고 계산을 늦추는 것을 다양한 예제에 적용해 본다.

### Structural Induction on Trees

지난번엔 함수가 올바르게 동작함을 증명하기 위해 *induction* 을 사용했었는데 이번시간엔 *tree* 에 대해 *induction* 을 사용한다.

모든 트리 `t` 에 대해서 속성 `P(t)` 가 참임을 증명하려면

(1) 먼저 트리의 모든 *leave* 에 대해 `P(1)` 임을 보인다.  
(2) 서브트리 `s1, ..., s` 을 가진 *internal node* 에 대해 `P(s1) ^ ... ^ P(sn)` 임을 보인다.

이제 몇 주 전에 만들었던 `IntSet` 을 증명해 보자.

```scala
  abstract class IntSet {
    def incl(x: Int): IntSet
    def contains(x: Int): Boolean
  }

  object Empty extends IntSet {
    def contains(x: Int): Boolean = false
    def incl(x: Int): IntSet = NonEmpty(x, Empty, Empty)
  }

  case class NonEmpty(elem: Int, left: IntSet, right: IntSet) extends IntSet {
    def contains(x: Int): Boolean =
      if (x < elem) left contains x
      else if (x > elem) right contains x
      else true

    def incl(x: Int): IntSet =
      if (x < elem) NonEmpty(elem, left incl x, right)
      else if (x > elem) NonEmpty(elem, left, right incl x)
      else this
  }
```

구현에 대한 증명을 한다는건 구현이 포함하는 *law* 를 증명한다는 것을 의미한다. 예를들면

```scala
Empty contains x == false
(s incl x) contains x == true
(s incl x) contains y == s contains y (if x != y)
```

`(s incl x) contains x == true` 부터 증명하자. ~~쉬우니까~~


(1) *base case (`P(1)`)* 은 `Empty` 를 `s` 에 집어넣으면 된다.

`NonEmpty(x, Empty, Empty) contains x` 이므로 `true` 다.

(2) *induction step: `s` 가 `NonEmpty(z, l, r)`* 에 대해서 증명하자.

`NonEmpty(z, l, r) incl x contains x` 인데, `z == x`, `z < x`, `z > x` 인 3 가지 경우로 나눠 참임을 보이면 된다. 뒤의 두개는 같은 증명이며 각각의 마지막 단계에서 *induction hypothesis* 를 사용하면 된다.

```scala
(NonEmpty(z, l, r) incl x) contians x // z < x
(NonEmpty(z, l, r incl x) contains x
(r incl x) contians x // by def of NonEmpty.contains
true // by induction hypothesis
```

`(xs incl y) contains x == xs contians x`, `(if x != y)` 를 증명해 보자.

마찬가지로 *base case* 는 `Empty` 를 이용하면 된다. `y < x`, `y > x` 인 두 가지 경우로 나누어 참임을 보이자.

다음으로 *inductive step* 인데, `xs` 가 `NonEmpty(z, l, r)` 일때다. 아래의 5가지 경우를 고려해야 한다.

```scala
z = x
z = y
z < y < x
y < z < x
y < x < z
```

마찬가지로 `z = x, z = y` 도 `y < x, y > x` 로 나누어 풀자. 나머지 증명은

```scala
// z < y < x, to show NonEmpty(z, l, r) contains x
NonEmpty(z, l, r incl y) contains x
(r incl y) contains x
r contains x  // by induction hypothesis
NonEmpty(z, l, r) contains x // by def of NonEmpty.contains

// y < z < x
NonEmpty(z, l incl y, r) contains x
r contains x
NonEmpty(z, l, r) contains x // by def of NonEmpty.contains

// y < x < z
(NonEmpty(z, l, r) incl y) contains x
NonEmpty(z, l incl y, r) contains x
(l incl y) contains x
l contains x // by induction hypothesis
NonEmpty(z, l, r) contains x
```

조금 더 어려운 증명으로 `union` 이 참임을 보일 수 있다.

```scala
(xs union ys) contains x = xs contains x || ys contains x
```

마찬가지로 `xs` 에 대해 *structural induction* 을 이용하면 된다.

### Streams

1000 부터 10000 사이에 있는 소수 중 두번 째 것을 찾는다 하자.

```scala
((1000 to 10000) filter isPrime)(1)
```

짧고 엘레강스 하지만 문제가 하나 있다. 1000 부터 10000 까지의 소수를 모두 찾은 뒤 2번째 원소에 접근하므로 성능 문제가 생긴다. 필요없는 나머지 소수도 같이 찾아버리는 것이다.

그래서 스칼라에서는 *stream* 을 지원한다.

> Avoid computing the tail of a sequence until it is needed for the evaluation result which might be never

다시 말해서 *sequence* 에서 각 부분이 *evaluation (평가)* 되기 전까지 계산을 미루는 클래스를 스칼라에서는 *stream* 이란 이름으로 만들어 놨다.

스트림은 리스트와 유사하지만 *tail* 부분이 *on-demand* 로 *evaluation* 된다.

```scala
> Stream(1, 2, 3)
scala.collection.immutable.Stream[Int] = Stream(1, ?)

> Stream.empty
res1: scala.collection.immutable.Stream[Nothing] = Stream()

> (1 to 1000).toStream
scala.collection.immutable.Stream[Int] = Stream(1, ?)

> val xs = Stream(1, Stream.cons(2,  Stream.empty))
xs: scala.collection.immutable.Stream[Any] = Stream(1, ?)
```

보면 알겠지만 진짜로 *tail* 부분이 `?` 로 되어있다.

```scala
  def streamRange(l: Int, h: Int): Stream[Int] = {
    if (l >= h) Stream.empty
    else Stream.cons(l, streamRange(l + 1, h))
  }

  def listRange(l:Int, h: Int): List[Int] = {
    if (l >= h) Nil
    else l :: listRange(l + 1, h)
  }
```

스트림을 만드는 `streamRange` 와 리스트를 만드는 `listRange` 를 보면 하는 생긴건 비슷하지만 실제로는 완전히 다른 일을 한다.

`listRange` 는 마지막 원소가 `Nil` 인 리스트를 만들지만 `streamRange` 는 두번 째 원소가 `?` 인 스트림을 만든다.

다시 처음의 예제로 돌아와서

```scala
((1000 to 10000).toStream filter isPrime)(1)
```

*stream* 은 리스트와 관련된 메소드를 모두 사용할 수 있다. 하나만 빼고, 바로 `cons ::` 다.`Stream.cons` 는 `#::` 다.

마찬가지로 `#::` 도 패턴으로 사용할 수 있다.

스트림의 구현은 대부분 리스트와 비슷한데, `empty` 가 좀 다르다.  `Stream.empty` 처럼 스트림 내부에 정의되어있다.

그리고 가장 중요한 차이점은 `Stream.cons` 의 파라미터다.

```scala
object Stream {
  def cons[T](hd: T, tl: => Stream[T]) = new Stream[T] {
    def isEmpty = false
    def head = hd
    def tail = tl
  }
}
```

`tl: => Stream[T]` 를 보면 알 수 있듯이 `tl` 은 *call-by-name* 이다. 따라서 바로 평가되지 않으며 사용하는 시점에 평가된다.

이제 `filter` 의 구현을 좀 보자.

```scala
def filter(p: T => Boolean): Stream[T] =
  if (isEmpty) this
  else if (p(head)) cons(head, tail.filter(p))
  else tail.filter(p)
}
```

여기서 중요한 부분이 `cons(head, tail.filter(p))` 다. 스트림의 `cons` 의 두번째 인자로 `tail.filter(p)` 를 넘겨주기 때문에 이 식의 평가는 나중에 호출될 때 이루어진다. 다시 말해 지금 당장은 `head` 만 *predicate `p`* 가 적용된다는 뜻이다.

따라서 `streamRange` 를 이렇게 수정하면 어떤 값이 출력될까?

```scala
  def streamRange(l: Int, h: Int): Stream[Int] = {
    println("l: " + l)
    if (l >= h) Stream.empty
    else Stream.cons(l, streamRange(l + 1, h))
  }

streamRange(1, 10).take(3).toList
```

답은 `1 2 3` 이다. `take(3)` 가 아니라 `toList` 때문에 그렇다. `take(3)` 는 여전히 *stream* 이다. 따라서 첫 번째 원소인 `1` 만 평가된 상태이며 리스트로 바꾸는 순간 `1, 2, 3` 이 모두 평가되어야 하므로 그때 출력된다.

### Lazy Evaluation

이제 까지 본 *stream* 구현은 필요 없는 `tail` 부분을 계산하지 않게 해 주었지만 심각한 결함이 있다. `tail` 이 여러번 호출된다면 어떻게 될까? 매번 다시 계산되야 한다.

그래서 첫 번째 `tail` 을 계산할 때 결과를 저장해 놓고, 그 다음부터는 재활용 하는 방법으로 이 문제를 해결해 보자.

사실 이건 함수형 프로그래밍에서 함수는 매번 같은 결과를 반환한다는 원칙에도 부합한다.

이걸 *lazy evaluation* 이라 부른다. 우리가 이제 까지 보아왔던 것은 *by-name evaluation* 이다. 필요할 때 평가하긴 하지만 매번 다시 계산해야하기 때문에 성능이 떨어진다. 그리고 *strict evaluation* 은 바로 평가되는 일반 `val` 변수라 보면 된¤.

하스켈은 *lazy evaluation* 이 디폴트지만 스칼라는 *strict evaluation* 이 기본이다.

```scala
lazy val x = expr
def x expr
```

다음 두 식의 차이는 무엇일까? 둘 다 *on-demand* 로 평가되지만 `def` 는 매번 다시 계산되고 `lazy val` 은 처음의 계산을 재활용한다.

그런 이유로 다음의 코드를 실행시키면

```scala
  def expr = {
    val x = { println("x"); 1 }
    lazy val y = { println("y"); 2 }
    def z = { println("z"); 3 }
    z + y + x + z + y + x
  }

  expr
  // x
  // z
  // y
  // z
```

`xzyz` 가 출력된다.

이제 *lazy evaluation* 을 *stream* 구현에 적용하자

```scala
object Stream {
  def cons[T](hd: T, tl: => Stream[T]) = new Stream[T] {
    def isEmpty = false
    def head = hd
    lazy val tail = tl
  }
}
```

근데 이렇게 작성한 코드가 실제로 *on-demand* 로 평가되는지 어떻게 알까? *substitution model* 에 적용해보자.

```scala
(streamRange(1000, 10000) filter isPrime) apply 1

// will be expanded

cons(1000, streamRange(1000 + 1, 10000)).filter(isPrime).apply(1)
```

이제 `cons(1000, streamRange(1000 + 1, 10000))` 를 `C1` 이라 하자.

```scala
C1.filter(isPrime).apply(1)

// same as
(if (isPrirme(C1.head))
  cons(C1.head, C1.tail.filter(isPrime))
else C1.tail.filter(isPrime))
apply(1)
```

이 때 `C1.head == 1000` 이므로 소수가 아니다. 따라서 이 식은

```scala
C1.tail.filter(isPrime).apply(1)

// same as
streamRange(1001, 10000).filter(isPrime).apply(1)
```

이렇게 첫 번째 소수를 찾을 때 까지 반복된다. `cons(1009, streamRange(1009 + 1, 10000)` 을 `C2` 라 부르면

```scala
C2.filter(isPrime).apply(1)

// same as
cons(1009, C2.tail.filter(isPrime)).apply(1)
```

이 된다 스트림의 `apply` 가 다음과 같이 구현되어 있다고 하자.

```scala
def apply(n: Int): T =
  if (n == 0) head
  else tail.apply(n - 1)
```

그럼 위 식은 이렇게 확장된다.

```scala
cons(1009, C2.tail.filter(isPrime)).tail.apply(0)


// same as
C2.tail.filter(isPrime).apply(0)

// same as
streamRange(1010, 10000)).filter(isPrime).apply(0)
```

마찬가지로 다음 번 소수 `1013` 을 찾을 때 까지 `streamRange.tail` 와 `filter` 가 반복되며 식이 확장된다.

```scala
cons(1013, streamRange(1013 + 1, 10000)).filter(isPrime).apply(0)
```

여기서 `cons(1013, streamRange(1013 + 1, 10000))` 를 `C3` 라 하자.

```scala
C3.filter(isPrime).apply(0)

// same as
cons(1013, C3.tail.filter(isPrime)).apply(0)
```

따라서 `1013` 이 나온다. 결국 모든 과정에서 *stream* 의 `cons` 내부의 `tail` 은 `apply` 나 `filter` 이후에 호출됨을 알 수 있다.

### Computing with Infinite Sequences

스트림에서 *tail* 이 필요할 때만 평가되기 때문에 *infinite stream* 을 만드는 것도 가능하다!

```scala
def from(n: Int): Stream[Int] = n #:: from(n + 1)


val nats = from(0) // natural number  
val m4s = nats map (_ * 4)

(m4s take 3).toList // List(0, 4, 8)
```

이제 무한한 길이의 스트림을 만드는 방법을 배웠으니 이걸 이용해 우주에 존재하는 모든 소수를 포함하는 스트림을 만들어보자.

고대의 소수 계산 방법인 *The Sieve of Eratosthenes* 를 이용한다. 2 부터 시작해서 하나씩 숫자를 증가 시키며 해당 숫자의 배수를 모두 제거하는 방식으로 소수를 찾는다. ~~무려 몇천년 뒤의 컴퓨터 기술을 고려하고  알고리즘을 구현한 갓 고대인들~~

```scala
def sieve(s: Stream[Int]): Stream[Int] =
  s.head #:: sieve(s.tail filter (_ % s.head != 0))

val primes = sieve(from(2))
primes.take(4).toList
// List(2, 3, 5, 7)
```

다른 곳에도 좀 활용 해보자. 오래전에 [1강](http://1ambda.github.io/functional-programming-in-scala-chapter-1/) 에서 *뉴튼-랩슨* 법으로 제곱근을 구하는 함수를 작성한 적이 있었다. 그 때 제곱근의 값이 한 재귀 호출마다 특정 값 미만으로 변하는지 검사하는 `isGoodEnough` 함수를 작성했었고 이를 이용해서 제³±근 구하는 함수가 무한 재귀에 빠지지 않도록 했었다.

*lazy evaluation* 을 이용하면 계산을 미룰 수 있기 때문에 무한 재귀에 빠지는걸 막을 수 있다.

```scala
  def sqrtStream(x: Double): Stream[Double] = {
    def improve(guess: Double) = (guess + x / guess) / 2
    lazy val guesses: Stream[Double] = 1 #:: (guesses map improve)
    guesses
  }

  sqrt(4).take(10).toList  
```

참고로 원리가 알고싶다면 [여기](https://www.google.co.kr/search?q=y+%3D+1%2Fx&oq=y+%3D+1%2Fx&aqs=chrome..69i57j69i65l2j69i64l2.3051j0j1&sourceid=chrome&es_sm=122&ie=UTF-8#newwindow=1&q=y+%3D+(4%2Fx+%2B+x)+%2F+2) 그래프를 한번 보라.

물론 `sqrtStream` 은 *stream* 을 리턴하므로 좀 더 세밀한 값을 얻기 위해 `isGoodEnough` 를 활용할 수도 있다.

```scala
def isGoodEnough(x: Double, guess: Double) =
  math.abs((guess * guess - x ) / x) < 0.0001

sqrtStream(4).filter(isGoodEnough(_, 4)).take(10).toList
```

그럼 이제 무한한 길이를 가진 컬렉션을 `map` 과 `filter` 두 가지 방법으로 구현할 수 있다는 사실을 알게 되었을 텐데, 어느게 더 빠를까?

```scala
val xs = from(1) map (_ * N)
val ys = from(1) filter (_ % N == 0)
```

당연히 `map` 이 더 빠르다. 필터는 원소를 돌면서 필터링 하는거고, 맵은 바로 곱해서 값을 구한다.

### The Water Pouring Problem

기본적인 아이디어는 물컵에 대한 액션(`Move`)을 `Empty, Fill, Pour` 로, 액션을 수행할 상태를 `type State = Vector[Int]` 모델링한다.

최초 상태 `initialState` 에 대해 가능한 모든 종류의 `Move` 를 미리 `moves` 에 만들어 놓고 (쉽게 만들 수 있다) 한번 씩 수행해 가면서 답이 있는지 검사한다.

이 과정에서 만들어지는 `List[Move]` 를 `Path` 라 볼 수 있다. 쉽게 연산하기 위해(리스트의 컨싱을 이용) `Path` 의 `head` 가 제일 마지막 `Move` 라 하자.

이러면 하나의 `Path` 는 우리가 가진 `initialState` 에 대해 `List[Move]` 를 적용해 마지막 상태를 알 수 있다. 이 것을 `endState` 함수로 구현한다. 이 때 마지막 `Move` 가 먼저 적용되야 하므로 `foldRight` 를 이용하면 *one-liner* 로 구현할 수 있다.

`Path` 가 가진 `List[Move]` 에 새로운 `Move` 를 컨싱으로 연결하는 `extend` 메소드를 만들고, 우리가 풀어야 할 문제의 답은 하나의 `Path` 이므로 출력을 위해 `toString` 도 오버라이드 하자.  

하나의 `Path` 에서 `moves` 를 적용하면 다수의 `Path` 가 나온다. 이걸 `Paths: Set[Path]` 라 부르면 한 단계 한 단계 액션을 적용할 때마다 `Set[Path]` 가 생기는것이다.

그런데, 재귀를 이용해 구현한다 해도 무한정 계산할 수 없으므로 스트림을 이용해 다음단계의 계산은 필요할때로 미룰 수 있다. `Set[Path]` 를 받아 `Stream[Set[Path]]` 를 돌려주는 `from` 함수를 만들자.

```scala
class Pouring(capacity: Vector[Int]) {

  // State
  type State = Vector[Int]
  val initialState = capacity map { _ => 0 }

  // Move
  trait Move {
    def change(s: State): State
  }
  case class Empty(glass: Int) extends Move {
    def change(s: State) = s updated(glass, 0)
  }
  case class Fill(glass: Int) extends Move {
    def change(s: State) = s updated(glass, capacity(glass))
  }
  case class Pour(from: Int, to: Int) extends Move {
    def change(s: State) = {
      val amount = s(from) min (capacity(to) - s(to))
      s updated (from, s(from) - amount) updated (to, s(to) + amount)
    }
  }

  val glasses = 0 until capacity.length

  val moves =
    (for (glass <- glasses) yield Empty(glass)) ++
    (for (glass <- glasses) yield Fill(glass)) ++
    (for (from <- glasses; to <- glasses if from != to) yield Pour(from, to))

  // Path

  class Path(history: List[Move]) {
    def endState: State = (history foldRight initialState)(_ change _)
    def extend(move: Move): Path = new Path(move :: history)
    override def toString = "\n" + (history.reverse mkString " ") + "-->" + endState
  }

  val initialPath = new Path(Nil)

  def from(paths: Set[Path]): Stream[Set[Path]] = {
    if (paths.isEmpty) Stream.empty
    else {
      val more = for {
        path <- paths
        next <- moves map path.extend
      } yield next

      paths #:: from(more)
    }
  }

  val pathSets = from(Set(initialPath))

  def solutions(target: Int): Stream[Path] = {
    for {
      pathSet <- pathSets
      path <- pathSet
      if path.endState contains target
    } yield path
  }
}
```

그런데, 실제로 코드를 돌려보면 상당히 느리다.

```scala
val problem = new Pouring(Vector(4, 7, 8))
println(problem.solutions(6))
```

이는 `from` 함수에서 `path` 를 `extend` 할 때 기존에 있던 `path`  도 포함하기 때문이다. 따라서 각 `path` 에서 `endState` 를 담은 `explored` 를 인자에 넘겨주고, 다시 넘겨 받으면

```scala
  def from(paths: Set[Path], explored: Set[State]): Stream[Set[Path]] = {
    if (paths.isEmpty) Stream.empty
    else {
      val more = for {
        path <- paths
        next <- moves map path.extend
      } yield next

      paths #:: from(more, explored ++ (more map (_.endState)))
    }
  }

  val pathSets = from(Set(initialPath), Set(initialState))
```

그리고 `endState` 도 재귀함수로서 자주 호출되는데 매번 다시 계산되야 하므로 변수로 저장하면

```scala
  class Path(history: List[Move], val endState: State) {
    def extend(move: Move): Path = new Path(move :: history, move change endState)
    override def toString = "\n" + (history.reverse mkString " ") + "-->" + endState
  }

  val initialPath = new Path(Nil, initialState)
```

~~내 경우는 옵티마이징 한 결과가 더 느렸다;;~~

여기서 중요한 아이디어 두개를 짚어보면

(1) `List[Move]` 에서 앞쪽 `Move` 가 더 나중에 적용되는 것을 통해 매 `extend` 마다 전체 리스트를 순회할 필요가 없도록 하고 계산은 `foldRight` 를 이용해 쉽게 구현  
(2) 무한한 재귀를 *stream* 을 이용해 계산을 미룬 `solution` 이나 `from` 함수.

특히 `solution` 은 `Stream[Path]` 를 리턴하도록 하여 첫번째 답까지 찾는 계산만 수행하는데, 이는 스트림에 `for-loop` 을 돌려도 스트림을 돌려준다는 사실을 알려준다. (*filter* 와 같다고 생각하면 편하다.*)

```scala
def ex =
  for(i <- (1 to 10).toStream if i % 2 == 0)

ex.take(3).toList // List(2, 4, 6)
```

### Course Summary

이제 까지 우리가 다뤄 온 것들은

- higher-order functions
- case classes and pattern matching
- immutable collections
- absence of mutable state
- flexible evaluation strategies: *strict / lazy / by-name*

앞으로 다룰 것들은 (아마 *reactive programming* 에서)

- **functional programming and state**  
*what does it mean to have mutable state?*  
*what changes if we add it?*  

- **parallelism**
*how to exploit immutablility for parallel execution*  

- **DSL**  
*high-level libraries as embedded DSLs*  
*interpretation techniques for external DSLs*


### References

2014-11-04, **Functional Programming in Scala**, Coursera
