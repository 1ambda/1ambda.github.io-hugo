+++
date = "2016-06-25T00:16:24+09:00"
next = "../easy-scalaz-4"
prev = "../easy-scalaz-2"
title = "Easy Scalaz 3"
toc = true
weight = 103
aliases = [
    "/easy-scalaz-3-readerwriterstate-with-kleisli"
]
+++

![](https://raw.githubusercontent.com/1ambda/1ambda.github.io/master/assets/images/easy-scalaz/1-state/haskell.png)

## Monad Transformer

지난 시간엔 *State Monad* 를 다루었습니다. 그러나 *State* 만 이용해서는 유용한 프로그램을 작성할 수 없습니다. 우리가 다루는 연산은 *Option*, *Future* 등 다양한 *side-effect* 가 필요하기 때문인데요,

서로 다른 `Monad` 를 조합할 수 있다면 좋겠지만, 아쉽게도  `Functor`, `Applicative` 와 달리 모나드는 *composing* 이 불가능합니다. [Monad Do Not Compose](http://tonymorris.github.io/blog/posts/monads-do-not-compose)

여러 모나드를 조합해서 사용하려면 *Monad Transformer* 가 필요합니다.

> Monad transformers are useful for enabling interaction between different types of monads by "nesting" them into a higher-level monadic abstraction.

*Monad Transformer* 란 여러 모나드의 *effect* 를 엮어 새로운 모나드를 만들때 쓸 수 있습니다. 예를 들어

- 어떤 임의의 모나드 M 을 사용하면서 `State` 효과를 주고 싶을 때 `StateT` 를 이용할 수 있습니다
- `State` 를 다루면서, `for` 내에서 `Option` 처럼 로직을 다루고 싶다면, `OptionT[State, A]` 를 이용할 수 있습니다

대략 감이 오시죠? (`State` 에 대한 자세한 설명은 [Easy Scalaz 1 - State](http://1ambda.github.io/easy-scalaz-1-state/) 을 참조)

*scalaz* 에는 기본적으로 여러 모나드 트랜스포머가 정의되어 있습니다. ([scalaz.core.*](https://github.com/scalaz/scalaz/tree/de0516dffadb4ccd2066fe2b132a6d2ba6e38bc0/core/src/main/scala/scalaz)) `ListT`, `MaybeT` 등등. 이번 글에서는 아래 3개의 모나드 트랜스포머만 다룰 예정입니다.

- [Scalaz - OptionT.scala](https://github.com/scalaz/scalaz/blob/de0516dffadb4ccd2066fe2b132a6d2ba6e38bc0/core/src/main/scala/scalaz/OptionT.scala)
- [Scalaz - EitherT.scala](https://github.com/scalaz/scalaz/blob/de0516dffadb4ccd2066fe2b132a6d2ba6e38bc0/core/src/main/scala/scalaz/EitherT.scala)
- [Scalaz - StateT.scala](https://github.com/scalaz/scalaz/blob/de0516dffadb4ccd2066fe2b132a6d2ba6e38bc0/core/src/main/scala/scalaz/StateT.scala)


## The Problem

모나드 트랜스포머를 설명하기 위해, 사용자의 Github Repository 에 어느 언어가 쓰였는지를 알려주는 `findLanguage` 함수를 작성해보겠습니다.

```scala
// ref - https://softwarecorner.wordpress.com/2013/12/06/scalaz-optiont-monad-transformer/

import scalaz._, Scalaz._

case class User(name: String, repositories: List[Repository])
case class Repository(name: String, languages: List[Language])
case class Language(name: String, line: Long)

object GithubService {
  def findLanguage(users: List[User],
                    userName: String,
                    repoName: String,
                    langName: String): Option[Language] =
    for {
      u <- users          find { _.name === userName }
      r <- u.repositories find { _.name === repoName }
      l <- r.languages    find { _.name === langName }
    } yield l
}
```

`List[User]` 를 받아 해당 유저의 레포지토리에서 특정 언어가 있는지, 없는지를 검사하는 간단한 함수입니다.

```scala
val u1 = User(
  "1ambda", List(
    Repository("akka", List(
      Language("scala", 4990),
      Language("java",  12801)
    )),

    Repository("scalaz", List(
      Language("scala", 1451),
      Language("java",  291)
    ))
  )
)

val u2 = User(
  "2ambda", List()
)

val users = List(u1, u2)

// spec
"findLanguage" in {
  val l1 = findLanguage(users, "1ambda", "akka", "scala")
  val l2 = findLanguage(users, "1ambda", "akka", "haskell")
  val l3 = findLanguage(users, "1ambda", "rx-scala", "scala")
  val l4 = findLanguage(users, "adbma1", "rx-scala", "scala")

  l1.isDefined shouldBe true
  l2.isDefined shouldBe false
  l3.isDefined shouldBe false
  l4.isDefined shouldBe false
  }
```

그런데, 요구사항이 갑자기 변경되어 많이 쓰이는 언어도 찾아내야 합니다. **검사한 것 중 1000 줄이 넘는 언어리¤트를 상태로 다루면**,

```scala
type LangState = State[List[Language], Option[Language]]
```

이제 `findLanguage` 를 다시 작성하면,

```scala
def findLanguage2(users: List[User],
                  userName: String,
                  repoName: String,
                  langName: String): LangState =
  for {
    u <- users.find(_.name === userName).point[LangState]
    r  <- u.repositories.find(_.name === repoName).point[LangState]
    l <- r.languages.find(_.name === langName).point[LangState]
    _ <- modify(langs: List[Language] => if (l.line >= 1000) l :: langs else langs)
  } yield song
```

당연히 컴파일이 되지 않습니다. 이는 `u`, `r`, `l` 이 각각 `User`, `Repository`, `Language` 가 아니라 `Option[User]`, `Option[Repository]`, `Option[Language]` 이기 때문입니다. 패턴 매칭을 적용하면 아래와 같은 코드가 만들어집니다.

```scala
def findLanguage(users: List[User],
                  userName: String,
                  repoName: String,
                  langName: String): LangState[Option[Language]] =
  for {
    optUser <- (users.find { _.name === userName }).point[LangState]
    optRepository <- (
      optUser match {
        case Some(u) => u.repositories.find(_.name === repoName)
        case None => none[Repository] // same as Option.empty[Repository]
      }).point[LangState]
    optLanguage <- (optRepository match {
      case Some(r) => r.languages.find(_.name === langName)
      case None    => none[Language]
    }).point[LangState]
    _ <- modify { langs: List[Language] => optLanguage match {
      case Some(l) if l.line => 1000 => l :: langs
      case _                         => langs
    }}
  } yield optLanguage
```

위 코드에서 중복되는 부분을 발견할 수 있는데요, 바로 `State[S, Option[A]]` 에 대해 매번 패턴 매칭을 수행하는 부분이 중복입니다. 이를 제거하기 위해 새로운 모나드 `LangStateOption` 을 만들면

```sacla
case class LangStateOption[A](run: LangState[Option[A]])
```

이제 모나드를 구현하면

```scala
implicit val LangStateOptionMonad = new Monad[LangStateOption] {
  override def point[A](a: => A): LangStateOption[A] =
    LangStateOption(a.point[Option].point[LangState])

  override def bind[A, B](fa: LangStateOption[A])(f: (A) => LangStateOption[B]): LangStateOption[B] =
    LangStateOption(fa.run.flatMap { (o: Option[A]) => o match {
      case Some(a) => f(a).run
      case None    => (none[B]).point[LangState] /* same as `(None: Option[B]).point[LangState]` */
    }})
}

// findLanguage impl
def findLanguage3(users: List[User],
                  userName: String,
                  repoName: String,
                  langName: String): LangStateOption[Language] =
  for {
    u <- LangStateOption((users.find { _.name === userName }).point[LangState])
    r <- LangStateOption((u.repositories.find { _.name === repoName }).point[LangState])
    l <- LangStateOption((r.languages.find { _.name === langName }).point[LangState])
    _ <- LangStateOption((modify { langs: List[Language] =>
      if (l.line >= 1000) l :: langs else langs
    }) map (_ => none[Language]))
  } yield l
```

여기서 잘 보셔야 할 두 가지 부분이 있습니다

A. 우리가 임의의 모나드와 `Option` 을 엮은 새로운 모나드를 생성한다면, `LangStateOption` 타입만 다르고 모두 동일한 형태의 코드를 가지게 됩니다.

그런고로 *scalaz* 에서는 `Option` 과 임의의 모나드 `M` 을 조합한 타입을 `OptionT[M[_], A]` 로 제공합니다.

B. `State` 와 `Option` 을 엮어서 `State[S, Option[A]]` 를 엮을 경우 `State` 가 먼저 실행되고, 그 후에야 `Option` 이 효과를 발휘합니다. (`fa.run.flatMap { o => ... `}

따라서 어떤 모나드 트랜스포머와, 모나드를 엮냐에 따라서 의미가 달라집니다. 예를 들어 *scalaz* 에서 제공해주는 모나드 트랜스포머 `OptionT` 와 `StateT` 에 대해

- `OptionT[LangState, A]` 는 `run: LangState[Option[A]]` 이기 때문에 *optional value* 를 돌려주는 *state action* 을 의미하고
- 반면 `StateT[Option, List[Language], A]]` 는 `run: Option[State[List[Language], A]]` 기 때문에 존재하지 않을 수 있는 (`None`) 일 수 있는 *state action* 을 의미합니다

## MonadTrans

지금까지 우리가 했던 일을 살펴보면,

> `M[A]` -> `M[N[A]]` -> `NT[M[N[_]], A]`

즉 하나의 모나드 `M` 이 있을때 `A` 를 `N[A]` 로 *lifting* 하는 `N` 을 위한 모나드 트랜스포머를 `NT` 를 정의했습니다. *scalaz* 에서 사용된 모나드 트랜스포머 구현인 `MonadTrans`, `OptionT` 을 보면 다음과 같습니다.

```scala
// https://github.com/scalaz/scalaz/blob/series/7.2.x/core/src/main/scala/scalaz/MonadTrans.scala
trait MonadTrans[F[_[_], _]] {
  def liftM[G[_]: Monad, A](g: G[A]): F[G, A]

  ...
}

// OptionT `liftM` implementation (F == Option)
// https://github.com/scalaz/scalaz/blob/series/7.2.x/core/src/main/scala/scalaz/OptionT.scala#L213

def liftM[G[_], A](a: G[A])(implicit G: Monad[G]): OptionT[G, A]) =
  OptionT[G, A](G.map[A, Option[A]](a) { (a: A) =>
    a.point[Option]
  }
```

**Monad Transformer** 또한 **Monad** 기 때문에 또 다른 **Monad Transformer** 와 중첩이 가능합니다. 예를 들어

```scala
// ref - http://www.slideshare.net/StackMob/monad-transformers-in-the-wild
type VIO[A] = ValidationT[IO, Throwable, A]
def doIO: VIO[Option[String]
val r = OptionT[VIO, String] = optionT[VIO](doIO)

// OptionT[ValidationT[IO, Throwable, A]
// == IO[Validation[Throwable, Option[A]]
```

## OptionT

이제 모나드 트랜스포머가 무엇인지 알았으니, `OptionT` 를 사용해 볼까요?

```scala
// https://github.com/scalaz/scalaz/blob/series/7.2.x/core/src/main/scala/scalaz/OptionT.scala

final case class OptionT[F[_], A](run: F[Option[A]]) {
  self =>

  private def mapO[B](f: Option[A] => B)(implicit F: Functor[F]) = F.map(run)(f)

  def map[B](f: A => B)(implicit F: Functor[F]): OptionT[F, B] = new OptionT[F, B](mapO(_ map f))

  def flatMap[B](f: A => OptionT[F, B])(implicit F: Monad[F]): OptionT[F, B] = new OptionT[F, B](
    F.bind(self.run) {
      case None    => F.point(None: Option[B])
      case Some(z) => f(z).run
    }
  )

  def flatMapF[B](f: A => F[B])(implicit F: Monad[F]): OptionT[F, B] = new OptionT[F, B](
    F.bind(self.run) {
      case None    => F.point(none[B])
      case Some(z) => F.map(f(z))(b => some(b))
    }
  )
```

`OptionT` 는 두 가지 방법으로 생성할 수 있습니다.

- `val ma: M[A]` 가 있을 때 `ma.liftM[OptionT]`
- `val oa: Option[A]` 가 있을 때 `OptionT(oa.point[M])`

```scala
// type LangState[A] = State[List[Language], A]
val l = Language("lisp", 309)
val os1: OptionT[LangState, Language] = l.point[LangState].liftM[OptionT]
val os2: OptionT[LangState, Language] = OptionT(l.some.point[LangState])

os1 === os2
os1.run === os2.run
os1.run.runZero[List[Language]] === os2.run.runZero[List[Language]]
```

이제 `findLanguage` 함수를 `OptionT` 로 작성할 수 있습니다.

```scala
def findLanguage(users: List[User],
                  userName: String,
                  repoName: String,
                  langName: String): OptionT[LangState, Language] =
  for {
    u <- OptionT((users.find { _.name === userName }).point[LangState])
    r <- OptionT((u.repositories.find { _.name === repoName }).point[LangState])
    l <- OptionT((r.languages.find { _.name === langName }).point[LangState])
    _ <- modify { langs: List[Language] =>
      if (l.line >= 1000) l :: langs else langs
    }.liftM[OptionT]
  } yield l
```

### Sequencing OptionT

`findLanguage` 를 이용해서, findLanguage**s** 를 작성하는 것이 가능할까요?

```scala
case class LanguageLookup(userName: String, repoName: String, langName: String)

// Option[List[Language]] 를 돌려주는 All or Nothing 버전
def findLanguages(users: List[User],
                     lookups: List[LanguageLookup]): OptionT[LangState, List[Language]] = ???

// List[Option[Language]] 를 돌려주는 덜 엄격한 버전
def findLanguages(users: List[User],
                     lookups: List[LanguageLookup]): LangState[List[Option[Language]]] = ???
```

일단 `OptionT[LangState, List[Language]]` 를 돌려주는 것 부터 작성해 보겠습니다.

```scala
def findLanguages1(users: List[User],
                   lookups: List[LanguageLookup]): OptionT[LangState, List[Language]] =
  lookups map { lookup =>
    findLanguage(users, lookup.userName, lookup.repoName, lookup.langName)
  }

// compile error
Error:(87, 13) type mismatch;

 found   : List[scalaz.OptionT[LangState, Language]]
 required: scalaz.OptionT[LangState,List[Language]]
    lookups map { lookup =>
            ^
```

우리는 `OptionT[LangState, List[Language]]` 를 돌려줘야 하는데, 단순히 `map` 만 적용해서는 `List[OptionT[LangState, Language]]` 밖에 못 얻습니다. 따라서 `Traversable.traverseU` 를 이용하면

```scala
def findLanguages1(users: List[User],
                   lookups: List[LanguageLookup]): OptionT[LangState, List[Language]] =
  lookups.traverseU { lookup =>
    findLanguage(users, lookup.userName, lookup.repoName, lookup.langName)
  }
```

여기서 `traverseU(f)` 가 하는 일은

- `map(f)`: 함수 `f` 를 적용합니다.
- `List[OptionT[LangState, Language]]` 를 `OptionT[LangState, List[Language]]` 를 변환합니다. **Option 모나드의 효과를 적용하면서요** (**sequence**)

일반적으로 `F[G[B]]` 를 `G[F[B]]` 로 변경하는 함수를 `sequence` 라 부릅니다. (`F` 는 *Monad*, `G` 는 *applicative*)

```scala
final def sequence[G[_], B](implicit ev: A === G[B], G: Applicative[G]): G[F[B]] = {
  ...
}
```

`map` 후 `sequence` 를 호출하는 함수가 바로 위에서 보았던 `traverse` 입니다. 그런데, 더 높은 추상에서 보면 방금 말했던 것과는 반대로, `sequence` 가 *identity* 함수를 `map` 한 `traverse` 입니다. **scalaz** 에도 실제로 이렇게 구현되어 있습니다.

```scala
// https://github.com/scalaz/scalaz/blob/series/7.2.x/core/src/main/scala/scalaz/syntax/TraverseSyntax.scala#L25

  final def traverse[G[_], B](f: A => G[B])(implicit G: Applicative[G]): G[F[B]] =
    G.traverse(self)(f)

  /** Traverse with the identity function */
  final def sequence[G[_], B](implicit ev: A === G[B], G: Applicative[G]): G[F[B]] = {
    val fgb: F[G[B]] = ev.subst[F](self)
    F.sequence(fgb)
  }
```

위에서 `traverse` 가 아니라 `traverseU` 를 호출한 이유는 `OptionT` 에 대한 타입추론을 이용하기 위해서 입니다.

<br/>

이제 덜 엄격한 `findLanguages` 함수를 작성해보겠습니다.

```scala
def findLanguages2(users: List[User],
                   lookups: List[LanguageLookup]): LangState[List[Option[Language]]] =
  lookups.traverseS { lookup =>
    findLanguage(users, lookup.userName, lookup.repoName, lookup.langName).run
  }
```

`traverseS` 는 *state* 버전의 `traverse` 입니다. `map` 을 적용한 `List[OptionT[LangState, Language]]` 에 대해 `LangState[List[Option[Language]]` 를 돌려줍니다.

```scala
/** A version of `traverse` specialized for `State` */
final def traverseS[S, B](f: A => State[S, B]): State[S, F[B]] = F.traverseS[S, A, B](self)(f)
```

`State[S, A]` 에 대해서

- `State[S, Option[List[A]]` 를 얻고 싶다면 (**all or nothing**) `traverseU` 를
- `State[S, List[Option[A]]` 를 얻고 싶다면 `B = Option[A]` 를 `List` 로 감싸야 하므로 `State[S, F[B]]` 를 돌려주는 위해 `traverseS` 를 사용하면 됩니다.

### EitherT

`EitherT` 는 *scalaz* 의 `Either` 에 대한 모나드 트랜스포머입니다. 참고로, `scalaz.Either` 은 `scala.Either` 과 달리 *right-biased* 입니다. `Option` 처럼요.

> `A \/ B` is isomorphic to `scala.Either[A, B]`, but `\/` is right-biased, so methods such as `map` and `flatMap` apply only in the context of the "right" case.

`scalaz.Either` 에 대한 기본적인 설명은 [Learning Scalaz - Either](http://eed3si9n.com/learning-scalaz/Either.html) 에서 보실 수 있습니다.

<br/>

`EitherT` 를 위한 간단한 모델을 만들어 보겠습니다.

- 쿼리를 파싱하³ , 실행하는 과정에서 *상태* 인 `QueryState` 를 이용하고
- 쿼리 파싱에 실패하면 수행하지 않고 종료하기 위해 `scalaz.Either` 를 사용합니다

```scala
// ref - https://speakerdeck.com/mpilquist/scalaz-state-monad

import scalaz._, Scalaz._

trait Model
trait Query
trait QueryResult

object QueryService {
  def runQuery(s: String, model: Model): String \/ QueryResult = for {
    query <- parseQuery(s)
    result <- performQuery(query, model)
  } yield result

  def parseQuery(s: String): String \/ Query = "TODO".left
  def performQuery(q: Query, m: Model): String \/ QueryResult = "TODO".left
}
```

위 코드에 *State* 와 `EitherT` 를 추가하면

```scala
trait Model
trait Query
trait QueryResult
trait Transaction

object QueryService {
  type TransactionState[A] = State[Transaction, A]
  type Transactional[A] = EitherT[TransactionState, String, A]

  def runQuery(s: String, model: Model): Transactional[QueryResult] = for {
    query <- EitherT(parseQuery(s).point[TransactionState])
    result <- EitherT(performQuery(query, model).point[TransactionState])
  } yield result

  def parseQuery(s: String): String \/ Query = ???
  def performQuery(q: Query, m: Model): String \/ QueryResult = ???
}
```

여기에 약간의 헬퍼 함수를 더하면,

```scala
def runQuery(s: String, model: Model): Transactional[QueryResult] = for {
  query <- Transactional(parseQuery(s))
  result <- Transactional(performQuery(query, model))
} yield result

object Transactional {
  import QueryService._
  def apply[A](e: String \/ A): Transactional[A] = liftE(e)
  def liftE[A](e: String \/ A): Transactional[A] =
    EitherT(e.point[TransactionState])
}
```

이제 `Transactional` 이 이름 그대로의 역할을 할 수 있게 간단한 커넥션도 모델링 해 보겠습니다.

```scala
trait Transaction {
  def closeConnection: Unit
  def commit: Unit = closeConnection
  def rollback: Unit = closeConnection
}

object QueryService {
  type TransactionState[A] = State[Transaction, A]
  type EitherStringT[F[_], A] = EitherT[F, String, A]
  type Transactional[A] = EitherStringT[TransactionState, A]

  def parseQuery(s: String): String \/ Query =
    if (s.startsWith("SELECT")) s"Invalid Query: $s".left[Query]
    else (new Query {}).right[String]

  def performQuery(q: Query, m: Model): String \/ QueryResult =
    new QueryResult {}.right

  def runQuery(s: String, model: Model): Transactional[QueryResult] = for {
    query <- Transactional(parseQuery(s))
    result <- Transactional(performQuery(query, model))
    _ <- (modify { t: Transaction => t.commit; t }).liftM[EitherStringT]
  } yield result
}
```

여기서 `EitherStringT` 타입을 새로 만든건, `liftM` 을 사용하기 위해서입니다. 만약 `liftM[EitherT]` 를 이용해 리프팅을 하면, 다음과 같은 예외가 발생합니다.

```scala
Error:(37, 59) scalaz.EitherT takes three type parameters, expected: two
    _ <- (modify { t: Transaction => t.commit; t }).liftM[EitherT]
                                                          ^
```

이제 `parseQuery` 와 `performQuery` 실패시 `rollback` 을 호출하는것을 구현하고, `commit` 을 헬퍼 함수로 변경하겠습니다.

```scala
def runQuery(s: String, model: Model): Transactional[QueryResult] = for {
  query <- Transactional(parseQuery(s))
  result <- Transactional(performQuery(query, model))
  _ <- commit
} yield result

def commit: Transactional[Unit] =
  (modify { t: Transaction => t.commit; t }).liftM[EitherStringT]

object Transactional {
  import QueryService._
  def apply[A](e: String \/ A): Transactional[A] = e match {
    case -\/(error) =>
      /* logging error and... */
      liftTS(State[Transaction, String \/ A] { t => t.rollback; (t, e) })
    case \/-(a) => liftE(e)
  }

  def liftE[A](e: String \/ A): Transactional[A] =
    EitherT(e.point[TransactionState])

  def liftTS[A](tse: TransactionState[String \/ A]): Transactional[A] =
    EitherT(tse)
}
```

이제 다음처럼 실패시 롤백이 호출되고 `for` 자동으로 스탑되것을 확인할 수 있습니다.

```scala
val t = new Transaction {}
val model = new Model {}
val result1 = runQuery("qqq", model).run.eval(t)
println(result)

// output
parseQuery
rollback
-\/(Invalid Query: qqq)

val result2 = runQuery("SELECT", model).run.eval(t)
println(result2)

// output
parseQuery
performQuery
\/-(QueryService$$anon$2@36804139)
```

만약 `Transaction` 에 `committed`, `rollbacked` 등의 값을 추가하면 `eval` 대신 `exec` (`run` 도 가능) 으로 최종 상태인 `Transaction` 을 얻어 확인할 수 있습니다.

```scala
// https://github.com/scalaz/scalaz/blob/series/7.2.x/core/src/main/scala/scalaz/StateT.scala#L17

  /** An alias for `apply` */
  def run(initial: S1): F[(S2, A)] = apply(initial)

  /** Calls `run` using `Monoid[S].zero` as the initial state */
  def runZero[S <: S1](implicit S: Monoid[S]): F[(S2, A)] =
    run(S.zero)

  /** Run, discard the final state, and return the final value in the context of `F` */
  def eval(initial: S1)(implicit F: Functor[F]): F[A] =
    F.map(apply(initial))(_._2)

  /** Calls `eval` using `Monoid[S].zero` as the initial state */
  def evalZero[S <: S1](implicit F: Functor[F], S: Monoid[S]): F[A] =
    eval(S.zero)

  /** Run, discard the final value, and return the final state in the context of `F` */
  def exec(initial: S1)(implicit F: Functor[F]): F[S2] =
    F.map(apply(initial))(_._1)

  /** Calls `exec` using `Monoid[S].zero` as the initial state */
  def execZero[S <: S1](implicit F: Functor[F], S: Monoid[S]): F[S2] =
    exec(S.zero)
```

### StateT

[Easy Scalaz 1 - State](http://1ambda.github.io/easy-scalaz-1-state/) 에서 언급했던 것 처럼

```scala
type State[S, A] = StateT[Id, S, A]
type Id[+X] = X

// 더 엄밀히는,

type StateT[F[_], S, A] = IndexedStateT[F, S, S, A]
type IndexedState[-S1, S2, A] = IndexedStateT[Id, S1, S2, A]
```

`StateT` 에다가 혼합할 모나드 `F` 에 `Id` 를 준것이 `State` 입니다.

여기에 함수 `replicateM` 을 적용하면,

```scala
// https://speakerdeck.com/mpilquist/scalaz-state-monad
  "replicateM(10)" in {

    // def replicateM(n: Int): F[List[A]]
    val getAndIncrement: State[Int, Int] = State { s => (s + 1, s) }
    getAndIncrement.replicateM(10).run(0) shouldBe (10, (0 until 10).toList)
  }
```

따라서 `State` 를 `F[_]` 라 보면 이걸 `F[List[_]]` 로 만들어 주므로 여러개의 `flatMap` 이 중첩된 형태가 됩니다.

따라서 `replicateM(100000)` 등의 코드는 *Stackoverflow* 가 발생합니다.

이 문제를 해결하기 위해 `Trampoline` 을 이용할 수 있습니다.

> Scalaz provides the `Free` data type, which when used with Function0, trade heap for stack

이럴때 `Trampoline` 을 사용하면, *stackoverflow* 를 피할 수 있습니다. (그만큼의 힙을 사용해서)

```scala
// type Trampoline[+A] = Free[Function0, A]

"replicateM(1000)" in {

  import scalaz.Free._

  val getAndIncrement: State[Int, Int] = State { s => (s + 1, s) }
  getAndIncrement.lift[Trampoline].replicateM(1000).run(0).run shouldBe (1000, (0 until 1000).toList)
}
```

`Trampoline` 은 후에 `Free` 를 살펴보면서 다시 보겠습니다.


### References

- [Tony Morris - Monad Do Not Compose](http://tonymorris.github.io/blog/posts/monads-do-not-compose)
- [Scalaz OptionT Monad Transformer](https://softwarecorner.wordpress.com/2013/12/06/scalaz-optiont-monad-transformer/)
- [State Monad in Scalaz](https://speakerdeck.com/mpilquist/scalaz-state-monad)
- [scala-scratchpad: Monad Transformer in Scala](https://github.com/earldouglas/scala-scratchpad/tree/master/category-theory/monad-transformers)
- [Scalaz Typeclass Hierarchy](http://tpolecat.github.io/assets/scalaz.svg)
- [Stackoverflow - traverseU, traverseM](http://stackoverflow.com/questions/26602611/how-to-understand-traverse-traverseu-and-traversem)
- [Haskell Image](http://cs.lth.se/edan40)
- [Haskell Wiki - All About Monads](https://wiki.haskell.org/All_About_Monads#The_IO_monad)
