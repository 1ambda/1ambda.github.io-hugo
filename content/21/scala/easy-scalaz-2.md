+++
date = "2016-01-05T00:16:24+09:00"
next = "../easy-scalaz-3"
prev = "../easy-scalaz-1"
title = "Easy Scalaz 2"
toc = true
weight = 102
aliases = [
    "/easy-scalaz-2-monad-transformer"
]
+++

![](https://raw.githubusercontent.com/1ambda/1ambda.github.io/master/assets/images/easy-scalaz/1-state/haskell.png)

## State Monad

`State` 를 설명하는 수많은 문구들이 있지만, 타입만큼 간단한건 없습니다.

```scala
State[S, A] :: S => (S, A)
```

> A state transition, representing a **function**

즉 `S` 를 받아 `(S, A)` 를 돌려주는 함수를, 타입클래스 `State[S, A]` 로 표현합니다.

더 엄밀히는, (*scalaz*  구현에서는) `type State[S, A] = StateT[Id, S, A] where Id[+X] = X` 인데 이것은 나중에 `StateT` 에서 다시 보겠습니다.

우선 기억해둘 것은 `State` 가 **함수** 를 나타낸다는 사실입니다. 상태 `S` 를 변경하면서 `A` 를 만들어내는 함수를 말이지요. 즉, `State` 는 더도 말고 덜도 말고, 상태를 조작하는 **함수** 입니다. 여기에 모나드라고 하니, `flatMap` 같은 몇몇 함수가 추가된 것 뿐이지요.

### State Basics

`State` 코드를 들춰보면, 아래와 같이 생겼습니다.

```scala
object State extends StateFunctions {
  def apply[S, A](f: S => (S, A)): State[S, A] = new StateT[Id, S, A] {
    def apply(s: S) = f(s)
  }
}

trait StateFunctions extends IndexedStateFunctions {
  def constantState[S, A](a: A, s: => S): State[S, A] = State((_: S) => (s, a))
  def state[S, A](a: A): State[S, A] = State((_ : S, a))
  def init[S]: State[S, S] = State(s => (s, s))
  def get[S]: State[S, S] = init
  def gets[S, T](f: S => T): State[S, T] = State(s => (s, f(s)))
  def put[S](s: S): State[S, Unit] = State(_ => (s, ()))
  def modify[S](f: S => S): State[S, Unit] = State(s => {
    val r = f(s);
    (r, ())
  })
}
```

- `State.apply` 에 상태 `S` 를 조작하는 함수 `f` 를 먹이면 `StateT` 가 나오고
- `StateT.apply` 에 초기 상태 `S` 를 먹이면 최종 결과물인 `(S, A)` 가 나옵니다

그리고 코드를 조금 만 더 따라가다 보면 `apply` 의 *alias* 로 `run` 이라는 함수가 제공되는걸 알 수 있습니다. [(Scalaz StateT.scala #L10)](https://github.com/scalaz/scalaz/blob/series/7.2.x/core/src/main/scala/scalaz/StateT.scala#L10)

`flatMap` 으로 상태 조작함수 `f` 여러개를 엮다가 하다가 마지막에 `run` 으로 실행시킬것 같다는 느낌이 들죠?

<br/>

이제 `StateFunctions` *trait* 로 제공되는 함수를 사용해 볼까요? 그냥 써보면 재미 없으니, Github 에서 각 Repository 마다 존재하는 *star* 를 가져오는 것을 간단히 모델링 해보겠습니다. 매번 네트워크 요청을 통해 가져오면 느리니까, `Map[String, Int]` 타입의 캐시도 포함시켜서요.

```scala
import scalaz._, Scalaz._ /* import all */

type Cache = Map[String, Int]

"create, run State" in {
  val s: State[Cache, Int] = State { c => (c, c.getOrElse("1ambda/scala", 0))}
  val c: Cache = Map("1ambda/scala" -> 1)

  // def run(s: S): (S, A)
  val (c1, star1) = s.run(c)
  val (c2, star2) = s.run(Map.empty)

  (c1, star1) shouldBe (Map("1ambda/scala" -> 1), 1)
  (c2, star2) shouldBe (Map(), 0)
}
```

이 작은 코드에서 우리가 다루는 상태는 `Cache` 입니다. 아직은 `State { c => ... }` 에서 받은 `c: Cache` 를 수정하지 않기 때문에 `run` 에서 돌려주는 상태 (*State*) 는 `run` 에 넘긴 것과 동일합니다. 그런고로 `c == c1 == c2` 입니다.

이번엔 상태를 변경하는 함수를 만들어 보겠습니다. 캐시에서 데이터를 가져오면, 캐시를 그대로 돌려주고 미스가 발생하면 캐시에 레포지토리 URL 을 추가하겠습니다.

```scala
def getStargazer(url: String): State[Cache, Int] = State { c =>
  c.get(url) match {
    case Some(count) => (c, count)
    case None        => (c.updated(url, 0), 0)
  }
}

"getStargazer" in {
  val c: Cache = Map("1ambda/scala" -> 1)

  val s1 = getStargazer("1ambda/haskell")
  val (c1, star) = s1.run(c)

  (c1, star) shouldBe (c.updated("1ambda/haskell", 0), 0)
}
```

`State` 는 모나드기 때문에, `for` 내에서 이용할 수 있습니다. 아래에서 더 자세히 살펴³´겠습니다.

### State Monad, Applicative and Functor

모나드는 `return` 과 `bind` 를 가지고 특정한 규칙을 만족하는 타입 클래스를 말하는데요, *scala* 에서는 `bind` 는 `flatMap` 이란 이름으로 제공되는 것 아시죠?

```scala
trait Monad[A] {
  // sometimes called `unit`
  def return(a: A): M[A]
  def flatMap[B](f: A => M[B]): M[B]
}
```

*scalaz* 에선 `Monad` 는 아래의 두 타입클래스를 상속받아 구현됩니다.

- `Applicative.point` (= `return`)
- `Bind.bind` (= `bind`)

```scala
trait Bind[F[_]] extends Apply[F] { self =>
  ...
  def bind[A, B](fa: F[A])(f: A => F[B]): F[B]
  ...
}

trait Applicative[F[_]] extends Apply[F] { self =>
  ...
  def point[A](a: => A): F[A]
  ...
}
```

게다가 `Apply` 가 `Functor` 를 상속받으므로

```scala
trait Apply[F[_]] extends Functor[F] { self =>
  def ap[A,B](fa: => F[A])(f: => F[A => B]): F[B]
  ...
```

*scalaz* 에서 `State` 는 `Functor` 이면서, `Applicative` 이고, `Monad` 입니다.

아래는 [doobie](https://github.com/tpolecat/doobie) 를 만든 [@tpolecat](https://github.com/tpolecat) 의 블로그에서 가져온 *scalaz* 타입 클래스 계층인데, 이 그림을 보면 왜 그런지 알 수 있습니다. (http://tpolecat.github.io/assets/scalaz.svg)

![](https://raw.githubusercontent.com/1ambda/1ambda.github.io/master/assets/images/easy-scalaz/1-state/scalaz.png)

이제 `State` 가 모나드라는 사실을 알았으니, 위에서 작성했던 `getStargazer` 함수를 다시 작성해보겠습니다. *for comprehension* 을 사용할건데요,

- 먼저 `State[Cache, Int]` 의 상태인 `Cache` 를 얻어와야 하므로 `get` 을 이용하고
- 상태를 변경해야 하므로 `modify` 를 호출하겠습니다.

```scala
// State helper functions defined in `StateFunctions` trait
def state[S, A](a: A): State[S, A] = State((_ : S, a))
def init[S]: State[S, S] = State(s => (s, s)) /* 상태 S 를 아웃풋 A 위치로 꺼냄 */
def get[S]: State[S, S] = init
def gets[S, T](f: S => T): State[S, T] = State(s => (s, f(s)))
def put[S](s: S): State[S, Unit] = State(_ => (s, ()))
def modify[S](f: S => S): State[S, Unit] = State(s => {
  /* 상태 S 를 변경하는 함수를 받아, 적용하고 A 위치에 `()` 를 돌려줌 */
  val r = f(s);
  (r, ())
})

def getStargazer(url: String): State[Cache, Int] = State { c =>
  c.get(url) match {
    case Some(count) => (c, count)
    case None        => (c.updated(url, 0), 0)
  }
}

def getStargazerWithFor(url: String): State[Cache, Int] =
  for {
    c <- State.get[Cache]
    optCount = c.get(url)
    _ <- modify { c: Cache =>
      // same as `if (optCount.isDefined) c else c.updated(url, 0)`
      optCount match {
        case Some(count) => c
        case None        => c.updated(url, 0)
      }
    }
  } yield optCount.getOrElse(0)
```

### When to use State

그러면, 언제 `State` 가 필요할까요? 하나의 **상태** (*State*) 를 지속적으로 변경, 공유하면서 연산을 실행할 때 사용할 수 있습니다.

> Building computations from sequences of operations that require a shared state.

예를 들어 HTTP 요청과 응답, 트랜잭션 등을 `State` 로 다루면서 연산을 조합해서 사용할 수 있습니다.

- HttpRequest, HttpResponse, HttpSession
- Database Transaction
- Random Number Generator

### Github Service Example

그러면 위에서 보았던 `Cache` 에 약간의 기능을 추가해 볼까요? 캐시 히트, 미스도 저장하고 캐시 히트는 최대 5분까지만 인정하기로 하지요. 오래된 캐시를 삭제하는 기능을 빼고 만들어 보면,

```scala
type URL = String
type StarCount = Int

case class Timestamped(count: StarCount, time: DateTime)

case class Cache(hits: Int, misses: Int, map: Map[URL, Timestamped]) {
  def get(url: URL): Option[Timestamped] = map.get(url)
  def update(url: URL, timestamp: Timestamped): Cache = {
    val m = map + (url -> timestamp)
    this.copy(map = m)
  }
}

object Cache {
  def empty = Cache(0, 0, Map())
}
```

만약 `State` 가 없¤면, 우리가 다루는 상태인 `Cache` 를 명시적으로 넘겨주고, 리턴받기 위해 이렇게 코드를 작성해야 할테지요. 여기서 `c1` 대신 `c` 를 쓰는 오타라도 발생한다면..

```scala
def stargazerCount(url: URL, c: Cache): (Cache, StarCount) = {
  val (c1, optCount) = checkCache(url, c)

  optCount match {
    case Some(count) => (c1, count)
    case None => retrieve(url, c1)
  }
}

def checkCache(url: URL, c: Cache): (Cache, Option[StarCount]) =
  c.get(url) match {
    case Some(Timestamped(count, time)) if !stale(time) =>
      (c.copy(hits = c.hits + 1), Some(count))
    case _ =>
      (c.copy(misses = c.misses + 1), None)
  }

def retrieve(url: URL, c: Cache): (Cache, StarCount) = {
  val count = getStarCountFromWebService(url)
  val timestamp = Timestamped(count, DateTime.now)
  (c.update(url, timestamp), count)
}

def stale(then: DateTime): Boolean = DateTime.now > then + 5.minutes
def getStarCountFromWebService(url: URL): StarCount = ...
```

<br/>

여기에 `State` 를 하나씩 적용해 보겠습니다.

```scala
def stargazerCount(url: URL, c: Cache): (Cache, StarCount) = {
  val (c1, optCount) = checkCache(url, c)

  optCount match {
    case Some(count) => (c1, count)
    case None => retrieve(url, c1)
  }
}
```

먼저 `State` 타입을 적용하고, 그 후에 `for` 문을 적용한 뒤에, `State.state` 를 이용해서 조금 더 깔끔하게 바꾸면

```scala
// applying State
def stargazerCount(url: URL): State[Cache, StarCount] =
  checkCache(url) flatMap { optCount =>
    optCount match {
      case Some(count) => State { c => (c, count) }
      case None        => retrieve(url)
    }
  }

// use for-comprehension
def stargazerCount2(url: URL): State[Cache, StarCount] = for {
  optCount <- checkCache(url)
  count <- optCount match {
    case Some(count) => State[Cache, StarCount] { c => (c, count) }
    case None        => retrieve(url)
  }
} yield count

// State.state
def stargazerCount(url: URL): State[Cache, StarCount] = for {
  optCount <- checkCache(url)
  count <- optCount
    .map(State.state[Cache, StarCount])
    .getOrElse(retrieve(url))
} yield count
```

`checkCache` 함수에도 적용해 보겠습니다.

```scala
def checkCacheOrigin(url: URL, c: Cache): (Cache, Option[StarCount]) =
  c.get(url) match {
    case Some(Timestamped(count, time)) if !stale(time) =>
      (c.copy(hits = c.hits + 1), Some(count))
    case _ =>
      (c.copy(misses = c.misses + 1), None)
  }

def checkCache1(url: URL): State[Cache, Option[StarCount]] = State { c =>
  c.get(url) match {
    case Some(Timestamped(count, time)) if !stale(time) =>
      (c.copy(hits = c.hits + 1), Some(count))
    case _ =>
      (c.copy(misses = c.misses + 1), None)
  }
}

/**
 *  Has potential bug.
 *  Always use `State.gets` and `State.modify`.
 */
def checkCache2(url: URL): State[Cache, Option[StarCount]] = for {
  c <- State.get[Cache]
  optCount <- State.state {
    c.get(url) collect { case Timestamped(count, time) if !stale(time) => count }
  }
  _ <- State.put(optCount ? c.copy(hits = c.hits + 1) | c.copy(misses = c.misses + 1))
} yield optCount

def checkCache(url: URL): State[Cache, Option[StarCount]] = for {
  optCount <- State.gets { c: Cache =>
    c.get(url) collect { case Timestamped(count, time) if !stale(time) => count }
  }
  _ <- State.modify { c: Cache =>
    optCount ? c.copy(hits = c.hits + 1) | c.copy(misses = c.misses + 1)
  }
} yield optCount
```

 `checkCache2` 는 `State.get` `State.put` 때문에 버그가 발생할 수 있습니다. `get` 으로 꺼낸 뒤에 `put` 으로 넣으면, 이전에 어떤 상태가 있었든지, 덮어 씌우기 때문에 주의가 필요합니다. 일반적으로는 `put` 대신 `modify` 를 이용합니다.

```scala
def init[S]: State[S, S] = State(s => (s, s))
def get[S]: State[S, S] = init
def put[S](s: S): State[S, Unit] = State(_ => (s, ()))

def gets[S, T](f: S => T): State[S, T] = State(s => (s, f(s)))
def modify[S](f: S => S): State[S, Unit] = State(s => {
```

마지막으로 `retrieve` 함수도 수정해볼까요

```scala
def retrieveOrigin(url: URL, c: Cache): (Cache, StarCount) = {
  val count = getStarCountFromWebService(url)
  val timestamp = Timestamped(count, DateTime.now)
  (c.update(url, timestamp), count)
}

def retrieve1(url: URL): State[Cache, StarCount] = State { c =>
  val count = getStarCountFromWebService(url)
  val timestamp = Timestamped(count, DateTime.now)
  (c.update(url, timestamp), count)
}

def retrieve(url: URL): State[Cache, StarCount] = for {
  count <- State.state { getStarCountFromWebService(url) }
  timestamp = Timestamped(count, DateTime.now)
  _ <- State.modify[Cache] { _.update(url, timestamp) }
} yield count
```

### References

- [State Monad in Scalaz](https://speakerdeck.com/mpilquist/scalaz-state-monad)
- [Scalaz Typeclass Hierarchy](http://tpolecat.github.io/assets/scalaz.svg)
- [Haskell Image](http://cs.lth.se/edan40)
- [fpinscala - Monad](https://github.com/fpinscala/fpinscala/wiki/Chapter-11:-Monads)
- [Haskell Monad](https://wiki.haskell.org/All_About_Monads#The_IO_monad)
