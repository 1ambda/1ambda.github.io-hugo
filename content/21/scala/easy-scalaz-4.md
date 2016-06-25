+++
date = "2016-01-03T00:16:35+09:00"
next = "../easy-scalaz-5"
prev = "../easy-scalaz-3"
title = "Easy Scalaz 4"
toc = true
weight = 104
aliases = [
    "/easy-scalaz-4-yoneda-and-free-monad"
]
+++

![](https://raw.githubusercontent.com/1ambda/1ambda.github.io/master/assets/images/easy-scalaz/1-state/haskell.png)

## ReaderWriterState with Kleisli

*Composition* (합성) 은 함수형 언어에서 중요한 테마중 하나인데요, 이번 시간에는 *Kleisli* 를 이용해 어떻게 함수를 타입으로 표현하고, 합성할 수 있는지 살펴보겠습니다. 그리고 나서, *Reader*, *Writer* 에 대해 알아보고, 이것들과 *State* 를 같이 사용하는 *RWST* 에 대해 알아보겠습니다.

## Kleisli

*State* 가 `(S) => (S, A)` 를 타입클래스로 표현한 것이라면, `A => B` 를 타입클래스로 표현한 것도 있지 않을까요? 그렇게 되면, 스칼라에서 지원하는 `andThen`, `compose` 을 이용해서 함수를 조합하는 것처럼, 타입 클래스를 조합할 수 있을겁니다. `Kleisli` 가 바로, 그런 역할을 하는 타입 클래스입니다.

> Kleisli represents a function `A => M[B]`

타입을 보면, 단순히 `A => B` 이 아니라 `A => M[B]` 를 나타냅니다. 이는 `Kleisli` 가 `M` 을 해석하고, 조합할 수 있는 방법을 제공한다는 것을 의미합니다. 실제 구현을 보면,

```scala
// https://github.com/scalaz/scalaz/blob/series/7.2.x/core/src/main/scala/scalaz/Kleisli.scala#L8

final case class Kleisli[M[_], A, B](run: A => M[B]) { self =>
  ...

  def >=>[C](k: Kleisli[M, B, C])(implicit b: Bind[M]): Kleisli[M, A, C] =  kleisli((a: A) => b.bind(this(a))(k.run))

  def andThen[C](k: Kleisli[M, B, C])(implicit b: Bind[M]): Kleisli[M, A, C] = this >=> k

  def >==>[C](k: B => M[C])(implicit b: Bind[M]): Kleisli[M, A, C] = this >=> kleisli(k)

  def andThenK[C](k: B => M[C])(implicit b: Bind[M]): Kleisli[M, A, C] = this >==> k

  /** alias for `compose` */
  def <=<[C](k: Kleisli[M, C, A])(implicit b: Bind[M]): Kleisli[M, C, B] = k >=> this

  def compose[C](k: Kleisli[M, C, A])(implicit b: Bind[M]): Kleisli[M, C, B] = k >=> this

  def <==<[C](k: C => M[A])(implicit b: Bind[M]): Kleisli[M, C, B] = kleisli(k) >=> this

  def composeK[C](k: C => M[A])(implicit b: Bind[M]): Kleisli[M, C, B] = this <==< k
  ...
}
```

[Kleisli Example](https://github.com/scalaz/scalaz/blob/series/7.2.x/example/src/main/scala/scalaz/example/KleisliUsage.scala) 에서 간단한 예제를 가져와서 사용법을 살펴보도록 하겠습니다.

```scala
// https://github.com/scalaz/scalaz/blob/series/7.2.x/example/src/main/scala/scalaz/example/KleisliUsage.scala

case class Continent(name: String, countries: List[Country] = List.empty)
case class Country(name: String, cities: List[City] = List.empty)
case class City(name: String, isCapital: Boolean = false, inhabitants: Int = 20)

val data: List[Continent] = List(
  Continent("Europe"),
  Continent("America",
    List(
      Country("Canada",
        List(
          City("Ottawa"), City("Vancouver"))),
      Country("USA",
        List(
          City("Washington"), City("New York"))))),
  Continent("Asia",
    List(
      Country("India",
        List(City("New Dehli"), City("Calcutta"))))))
```

여기에 다음의 ¨수를 정의하면

```scala
def continents(name: String): List[Continent] =
  data.filter(k => k.name.contains(name))

def countries(continent: Continent): List[Country] = continent.countries

def cities(country: Country): List[City] = country.cities

def save(cities: List[City]): Try[Unit] =
  Try {
    // do IO or some side-effectful operations
    cities.foreach(c => println("Saving " + c.name))
  }

def inhabitants(c: City): Int = c.inhabitants
```

이제 `A => M[B]` 형태의 여러 함수들을 만들었으므로 이를 `Kleisli` 를 이용해 조합할 수 있습니다. (이 예제에서 `M == List`)

```scala
// Kleisli[List, String, City]
val allCities = kleisli(continents) >==> countries >==> cities

// Kleisli[List, String, Int]
val cityInhabitants = allCities map inhabitants
```

`allCities` 는 `String` 을 인자로 받기도 하고, `M == List` 의 `Kleisli` 기 때문에 `List` 를 인자로 받을 수도 있습니다. (`=<<`)

```scala
allCities("America") map(println)

// output
City(Ottawa,false,20)
City(Vancouver,false,20)
City(Washington,false,20)
City(New York,false,20)

(allCities =<< List("America", "Asia")).map(println)

// output
City(Ottawa,false,20)
City(Vancouver,false,20)
City(Washington,false,20)
City(New York,false,20)
City(New Dehli,false,20)
City(Calcutta,false,20)
```

`Kleisli` 가 제공하는 함수를 다시 살펴보면,

```scala
def =<<(a: M[A])(implicit m: Bind[M]): M[B] = m.bind(a)(run)

def map[C](f: B => C)(implicit M: Functor[M]): Kleisli[M, A, C] =
  kleisli(a => M.map(run(a))(f))

def mapK[N[_], C](f: M[B] => N[C]): Kleisli[N, A, C] =
  kleisli(run andThen f)

def flatMapK[C](f: B => M[C])(implicit M: Bind[M]): Kleisli[M, A, C] =
  kleisli(a => M.bind(run(a))(f))

def flatMap[C](f: B => Kleisli[M, A, C])(implicit M: Bind[M]): Kleisli[M, A, C] =
  kleisli((r: A) => M.bind[B, C](run(r))(((b: B) => f(b).run(r))))
```

여기서 `mapK :: M[B] => N[C]` 를 이용하면 현재 `Kleisli[M, _, _]` 를 `Kleisli[N, _, _]` 로 변경할 수 있습니다.

위에서 정한 `save` 함수는 `List[A]` 를 받아 `Try[Unit]` 를 여기에 사용할 수 있습니다.

```scala
// Kleisli[Try, String, Unit]
val getAndSaveCities = allCities mapK save
```

`local` 을 이용하면 함수를 *prepend* 할 수 있습니다.

```scala
// def local[AA](f: AA => A): Kleisli[M, AA, B] =
//   kleisli(f andThen run)

def index(i: Int): String = data(i).name

// Kleisli[List, Int, City]
val allCitiesWithIndex = allCities local index

allCitiesWithIndex(1) map(println)

// output
City(Ottawa,false,20)
City(Vancouver,false,20)
City(Washington,false,20)
City(New York,false,20)
```

`Kleisli` 에 대한 더 읽을거리는 아래 링크를 참조해주세요.

- [Scalaz Arrow](http://eed3si9n.com/learning-scalaz/Arrow.html)
- [Scalaz - Kleisli.scala#KleisliArrow](https://github.com/scalaz/scalaz/blob/series/7.2.x/core/src/main/scala/scalaz/Kleisli.scala#L209)

## Reader

`Kleisli` 가 `A => M[B]` 를 나타낸다면, `Reader` 는 `A => B` (`Function1`) 를 의미하는 타입클래스입니다. 얼핏 생각하기에 `Kleisli[Id, A, B]` 일것 같죠? 실제 구현을 보면 (*scalaz* 에서 타입 얼라이어스는 `package.scala` 에 정의되어 있습니다.)

```scala
// https://github.com/scalaz/scalaz/blob/series/7.2.x/core/src/main/scala/scalaz/package.scala

type ReaderT[F[_], E, A] = Kleisli[F, E, A]
val ReaderT = Kleisli
type Reader[E, A] = ReaderT[Id, E, A]

object Reader {
    def apply[E, A](f: E => A): Reader[E, A] = Kleisli[Id, E, A](f)
  }
```

`Reader` 도 `Klelsli` 이므로, `Reader[A, B] >==> Reader[B, C]` 는 `Reader[A, C]` 가 됩니다. 게다가 `Kleisli` 는 `flatMap` 을 정의하고 있으므로 *monadic composition* 을 작성할 수 있습니다.

> The point of a `Reader` is to supply some configuration object without having to manually (or *implicitly*) pass i around all the functions.

요는, 함수 사이의 체인을 엮어 새로운 함수를 만들수 있고 이로인해 직접 파라미터를 넘겨줄 필요가 없습니다. 예를 들어

```scala
type URI = String
type Key = String
type Value = String

val uri: Reader[Get, URI]
val queryString: Reader[URI, String]
val body: Reader[String, Map[Key, Value]

// Get => Map[Key, Value]
val queryStringToBody = uri >==> queryString >==> body
```

간단히 구현을 해보겠습니다. 예외 처리는 외부에서 `Try` 혹은 `\/.fromTryCatchThrowable` 등으로 한다 가정하고 로직에만 집중해보면,

```scala
// model
trait HttpRequest {
  def url: String
}
case class GET(url: String) extends HttpRequest
case class POST(url: String, body: Map[String, String]) extends HttpRequest

val uri: Reader[GET, String] = Reader { req: GET => req.url }
val queryString: Reader[String, String] = Reader { url: String => url.split("\\?")(1) }
val body: Reader[String, Map[String, String]] = Reader { queries: String =>
  val qs = queries.split("&").toList
  qs.foldLeft(Map.empty[String, String]) { (acc: Map[String, String], q) =>
    val kv = q.split("=")
    acc.updated(kv(0), kv(1))
  }
}

val queryStringToBody: Reader[GET, Map[String, String]] = uri >==> queryString >==> body
```

`queryStringToBody` 를 사용해 보면,

```scala
val get1 = GET("http://www.google.com/search?query=scalaz&site=github")
val post1 = POST("http://www.google.com/search", Map("query" -> "scalaz", "site" -> "github"))
val post2 = POST("https://www.google.com/search", Map("query" -> "scalaz", "site" -> "github"))

queryStringToBody.run(get1) shouldBe Map("query" -> "scalaz", "site" -> "github")
```

함수를 몇개 더 작성해보면,

```scala

val toHttpsRequest = Reader { url: String => url.replaceAll("http://$", "https://") }
val sslProxy: Reader[_ >: readerwriterstate.HttpRequest, readerwriterstate.HttpRequest] = Reader { req: readerwriterstate.HttpRequest =>
  req match {
    case request if request.url.startsWith("https://") => request
    case request: POST => request.copy(url = toHttpsRequest(request.url))
    case request: GET  => request.copy(url = toHttpsRequest(request.url))
  }
}

val convertGetToPost: Reader[_ >: readerwriterstate.HttpRequest, POST] = Reader { req : readerwriterstate.HttpRequest =>
  req match {
    case get: GET =>
      val split = get.url.split("\\?")
      val (path, query) = (split(0), split(1))
      val postBody = body.run(query)

      POST(path, postBody)

    case post: POST => post
  }
}
```

이제 `HttpRequest` 서브타입을 받아, 프록시를 적용하고, `GET` 이면 `POST` 로 변경하는 함수를 조합해보면 아래와 같습니다.

(`:>` 등 *Type Bound* 에 대해서는 [Scala School - Type & Polymorphism](http://twitter.github.io/scala_school/type-basics.html) 과 [Scala School - Advanced Types](http://twitter.github.io/scala_school/advanced-types.html) 를 참조해주세요.)

```scala
val proxiedPost: Reader[_ >: HttpRequest, POST] = sslProxy >==> convertGetToPost

// spec
proxiedPost.run(get1) shouldBe post2
```

## flatMap for Reader

`Reader` 는 `Kleisli` 고, 이것간의 합성은 `>==>` 을 이용한다는것을 확인했습니다. 그럼 `flatMap` 은 어디에 쓰는걸까요?

```scala
type ReaderT[F[_], E, A] = Kleisli[F, E, A]
type Reader[E, A] = ReaderT[Id, E, A]

final case class Kleisli[M[_], A, B](run: A => M[B]) { self =>
  ...

  // andThen
  def >=>[C](k: Kleisli[M, B, C])(implicit b: Bind[M]): Kleisli[M, A, C] =  kleisli((a: A) => b.bind(this(a))(k.run))

  def >==>[C](k: B => M[C])(implicit b: Bind[M]): Kleisli[M, A, C] = this >=> kleisli(k)

  def flatMapK[C](f: B => M[C])(implicit M: Bind[M]): Kleisli[M, A, C] =
    kleisli(a => M.bind(run(a))(f))

  def flatMap[C](f: B => Kleisli[M, A, C])(implicit M: Bind[M]): Kleisli[M, A, C] =
    kleisli((r: A) => M.bind[B, C](run(r))(((b: B) => f(b).run(r))))

  ...
}
```

`flatMap` 을 보면 재미난 점이 보입니다. `Kleisli[M, A, B]` 와 `Kleisli[M, A, C]` 를 `flatMap` 으로 엮는데, `r: A` 를 넣어서 `run(r)` 을 실행하는걸 보실 수 있습니다. `Kleisli[M, A, C]` 까지도요!

즉 `A` 자체가 일종의 설정(*Configuration*) 값으로써 모든 `Kleisli` 에서 사용됩니다. 그렇°에

- `Reader[A, B]` 와 `Reader[B, C]` 는 `>==>` 으로
- `Reader[A, B]` 와 `Reader[A, C]` 는 `flatMap` 으로 엮을 수 있습니다.

### Dependency Injection using Reader

`Reader` 를 이용하면 스칼라에서 별도의 라이브러리 없이 *Dependency Injection* (이하 *DI*) 를 구현할 수 있습니다. 이는 위에서 보았던 `flatMap` 의 특징을 이용하면 됩니다. 다음과 같은 모델이 있다고 할  때,

```scala
case class User(id: Long,
                name: String,
                age: Int,
                email: String,
                supervisorId: Long)

trait UserRepository {
  def get(id: Long): User
  def find(name: String): User
}

trait UserService {
  def getUser(id: Long): Reader[UserRepository, User] =
    Reader(repo => repo.get(id))

  def findUser(userName: String): Reader[UserRepository, User] =
    Reader(repo => repo.find(userName))

  def getUserInfo(userName: String): Reader[UserRepository, Map[String, String]] = for {
    user <- findUser(userName)
    supervisor <- getUser(user.supervisorId)
  } yield Map(
    "email" -> s"${user.email}",
    "boss"  -> s"${supervisor.name}"
  )
}
```

다음처럼 주입할 수 있습니다.

```scala
object UserRepositoryDummyImpl extends UserRepository {
  override def get(id: Long): User = ???
  override def find(name: String): User = ???
}

class UserApplication(userRepository: UserRepository) extends UserService
object UserApplication extends UserApplication(UserRepositoryDummyImpl)
```

이외에도 스칼라에서 언어 자체의 기능만으로 DI 를 구현하는 방법으로 *Cake Pattern* , *Implicit* 등이 있습니다. ([Scala Dependency Injection using Reader](http://blog.originate.com/blog/2013/10/21/reader-monad-for-dependency-injection/) 참조)

위의 두 방법과 `Reader` 를 사용한 방법을 비교하면,

- *Cake Pattern* 에 비해 코드가 짧고
- *Implicit* 를 이용하지 않으므로 함수 시그니쳐가 간단합니다.

## Writer


`Writer[W, A]` 는 `run: (W, A)` 을 값¼로 가지는 *case class* 입니다. 재미난 점은, `flatMap` 을 이용해 두개의 `Writer` 를 엮으면 각각의 값인 `(w1, a1)`, `(w2, a2)` 에 대해서 사용자가 다루는 값인 `a1, a2` 를 제하고 `w1` 과 `w2` 가 일종의 [State](http://1ambda.github.io/easy-scalaz-1-state/) 처럼 관리되어 자동으로 *append* 된다는 점입니다. 따라서 많은 튜토리얼들이 *logging* 을 예로 들어 `Writer` 를 설명하곤 합니다.

```scala
test("WriterOps") {
  val w1: Writer[String, Int] = 10.set("w1 created")
  val w2: Writer[String, Int] = 20.set("w2 created")

  val result: Writer[String, Int] = for {
    n1 <- w1
    n2 <- w2
  } yield n1 + n2

  // What if we use `List[String]` instead of `String`?
  result.run shouldBe ("w1 createdw2 created", 30)
}
```

*Scalaz* 구현을 보면

```scala
type Writer[W, A] = WriterT[Id, W, A]

final case class WriterT[F[_], W, A](run: F[(W, A)]) { self =>
  ...

  def flatMap[B](f: A => WriterT[F, W, B])(implicit F: Bind[F], s: Semigroup[W]): WriterT[F, W, B] =
    flatMapF(f.andThen(_.run))

  def flatMapF[B](f: A => F[(W, B)])(implicit F: Bind[F], s: Semigroup[W]): WriterT[F, W, B] =
    writerT(F.bind(run){wa =>
      val z = f(wa._2)
      F.map(z)(wb => (s.append(wa._1, wb._1), wb._2))
    })

  ...
```

`WriterT` 에서 `F` 를 `Id` 라 하면 `Writer` 가 되고 `flatMap` 로직은 다음처럼 단순화 할 수 있습니다.

```scala
case class Writer[W, A](run: (W, A)) { self =>
  def flatMap[B](f: A => Writer[W, B])(implicit s: Semigroup[W]) {
    val (w1, a) = self.run
    val (w2, b) = f(a)
    (s.append(w1, w2), b)
  }
}
```

여기서 [Semigroup.scala](https://github.com/scalaz/scalaz/blob/series/7.1.x/core/src/main/scala/scalaz/Semigroup.scala) 은, *Associativity* (결합법칙) 을 만족하는 *binary operator* 를 정의하는 타입 클래스입니다. (위에서 `append`)

```scala
// https://github.com/scalaz/scalaz/blob/series/7.1.x/core/src/main/scala/scalaz/Semigroup.scala#L55

 /**
   * A semigroup in type F must satisfy two laws:
    *
    *  - '''closure''': `∀ a, b in F, append(a, b)` is also in `F`. This is enforced by the type system.
    *  - '''associativity''': `∀ a, b, c` in `F`, the equation `append(append(a, b), c) = append(a, append(b , c))` holds.
   */
  trait SemigroupLaw {
    def associative(f1: F, f2: F, f3: F)(implicit F: Equal[F]): Boolean =
      F.equal(append(f1, append(f2, f3)), append(append(f1, f2), f3))
  }
```


[Monoid](https://github.com/scalaz/scalaz/blob/series/7.1.x/core/src/main/scala/scalaz/Monoid.scala) 는 결합법칙을 만족하는 덧셈 연산과, 항등원 연산을 정의하는 타입 클래스인데, *Scalaz* 에서는 `Monoid` 가 `Semigroup` 을 상속받습니다.

```scala
trait Monoid[F] extends Semigroup[F] { self =>
  ...
```

![https://raw.githubusercontent.com/1ambda/1ambda.github.io/master/assets/images/about-type-class/Typeclassopedia-diagram.png](https://raw.githubusercontent.com/1ambda/1ambda.github.io/master/assets/images/about-type-class/Typeclassopedia-diagram.png)

<br/>

따라서 `Writer[W, A]` 의 `flatMap` 을 이용하기 위해서는 `W` 가 `Semigroup` 여야 하고 그래야만 `flatMap` 내부에서 자동으로 `W` 를 *append* 할 수 있습니다.

스칼라에서 제공하는 `List` 등의 기본 타입은 *Scalaz* 에서 `Monoid` 를 제공합니다. ([scalaz.std.List](https://github.com/scalaz/scalaz/blob/series/7.1.x/core/src/main/scala/scalaz/std/List.scala#L109), [scalaz.std](https://github.com/scalaz/scalaz/tree/series/7.1.x/core/src/main/scala/scalaz/std) 참조)

정리하면, `Writer[W, A]` 를 이용하면 값인 `A` 를 조작하면서 `W` 를 신경쓰지 않고, 자동으로 `append` 시킬 수 있습니다. (e.g *logging*)

## Writer Example

간단한 모델을 만들면,

```scala
import scalaz._, Scalaz._

trait ThreadState
case object Waiting    extends ThreadState
case object Running    extends ThreadState
case object Terminated extends ThreadState
case class Thread(tid: String, name: String, state: ThreadState)
case class Process(pid: String, threads: List[Thread])

object Process {
  type Logger[A] = Writer[Vector[String], A]

  def genRandomID: String = java.util.UUID.randomUUID().toString.replace("-", "")

  def createThread(name: String): Logger[Thread] = {
    val tid = genRandomID
    Thread(tid, name, Waiting).set(Vector(s"Thread [$tid] was created"))
  }

  def createEmptyProcess: Logger[Process] = {
    val pid = genRandomID
    Process(pid, Nil).set(Vector(s"Empty Process [$pid] was created"))
  }

  def createNewProcess: Logger[Process] = for {
    mainThread <- createThread("main")
    process <- createEmptyProcess
    _ <- Vector(s"Main Thread [${mainThread.tid}] was added to Process [${process.pid}").tell
  } yield process.copy(threads = mainThread.copy(state = Running) :: process.threads)
}
```

여기서 `W` 로 `List[String]` 대신 `Vector[String]` 을 사용하는 이유는, *append* 가 더 빠르기 때문입니다. ([Scala Collection Performance Characteristics](http://docs.scala-lang.org/overviews/collections/performance-characteristics.html) 참조)

```scala
test("Writer usage2") {
  import readerwriterstate.Process._

  val (written, process) = createNewProcess.run

  process.threads.length shouldBe 1
  process.threads.head.name shouldBe "main"

  /* map lets you map over the value side */
  val ts: Logger[List[Thread]] = createNewProcess.map(p => p.threads)
  ts.value.length shouldBe 1

  /* with mapWritten you can map over the written side */
  val edited: Vector[String] = createNewProcess.mapWritten(_.map { log => "[LOG]" + log }).written
  println(edited.mkString("\n"))

  /** output
   * [LOG]Thread [557ad5bd0f3b4d49bac85b05ebedcd7b] was created
   * [LOG]Empty Process [710bd940ebdd4a82b949a32b585a12d9] was created
   * [LOG]Main Thread [557ad5bd0f3b4d49bac85b05ebedcd7b] was added to Process [710bd940ebdd4a82b949a32b585a12d9]
   */

  /* with mapValue, you can map over both sides */
  createNewProcess.mapValue { case (log, p) =>
    (log :+ "Add an IO thread",
     p.copy(threads = Thread(genRandomID, "IO-1", Waiting) :: p.threads))
  }

  // `:++>` `:++>>`, `<++:`, `<<++:`
  createNewProcess :++> Vector("add some log")
  val emptyWithLog = createEmptyProcess :++>> { process =>
    Vector(s"${process.pid} is an empty process")
  }

   println(emptyWithLog.written)

  // output: Vector(Empty Process [cf211fc366ab4d20a0c25a27d173accd] was created, cf211fc366ab4d20a0c25a27d173accd is an empty process)

  // Writer is an applicative
  val emptyProcesses: Logger[List[readerwriterstate.Process]] =
    (createEmptyProcess |@| createEmptyProcess) { List(_) |+| List(_) }

  val ps = emptyProcesses.value
  ps.length shouldBe 2
}
```

[Applicative Builder](http://eed3si9n.com/learning-scalaz/Applicative+Builder.html), [WriterT Functions](https://github.com/scalaz/scalaz/blob/series/7.1.x/core/src/main/scala/scalaz/WriterT.scala#L30) 를 참고하시면 이해가 더 쉽습니다.

## RWST

[ReaderWriterState](https://github.com/scalaz/scalaz/blob/series/7.1.x/core/src/main/scala/scalaz/ReaderWriterStateT.scala) 는 다름이 아니라, 이제까지 보았던 `Reader`, `Writer`, `State` 를 모두 이용하는 타입 클래스입니다. `Reader` 로 설정값을 읽고, `Writer` 로 중간 과정을 기록하고, `State` 로 상태를 변경 또는 유지해 가며 연산을 수행할 수 있습니다. *Scalaz* 에서는 예제로 [ReaderWriterStateTUsage.scala](https://github.com/scalaz/scalaz/blob/series/7.2.x/example/src/main/scala/scalaz/example/ReaderWriterStateTUsage.scala) 를 제공하고 있습니다.

이제까지 늘 그래왔듯이, `ReaderWriterState[R, W, S, A]` 또한 `ReaderWriterStateT[Id, R, W, S, A]` 의 *type alias* 입니다. `Reader`, `Writer`, `State` 에서 사용했었던 함수들도 같이 제공됩니다.

```scala
type ReaderWriterState[-R, W, S, A] = ReaderWriterStateT[Id, R, W, S, A]
type ReaderWriterStateT[F[_], -R, W, S, A] = IndexedReaderWriterStateT[F, R, W, S, S, A]

object ReaderWriterState extends ReaderWriterStateTInstances with ReaderWriterStateTFunctions {
  def apply[R, W, S, A](f: (R, S) => (W, A, S)): ReaderWriterState[R, W, S, A] = IndexedReaderWriterStateT[Id, R, W, S, S, A] { (r: R, s: S) => f(r, s) }
}
```

`apply` 를 보면, `ReaderWriterState` 는 타입 `(R, S) => (W, A, S)` 함수를 넘겨주어 생성할 수 있습니다. `Reader`, `State` 를 받고, `Writer`, `A` (결과값), `State` 를 돌려주는 것으로 해석할 수 있습니다.

`ReadwrWriterState.flatMap` 은 `State`, `Writer`, `Reader` 의 `flatMap` 을 모두 조합한것처럼 생겼습니다. 하는일도 그렇구요.

```scala
/** A monad transformer stack yielding `(R, S1) => F[(W, A, S2)]`. */
sealed abstract class IndexedReaderWriterStateT[F[_], -R, W, -S1, S2, A] {

  ...

  def flatMap[B, RR <: R, S3](f: A => IndexedReaderWriterStateT[F, RR, W, S2, S3, B])(implicit F: Bind[F], W: Semigroup[W]): IndexedReaderWriterStateT[F, RR, W, S1, S3, B] =
    new IndexedReaderWriterStateT[F, RR, W, S1, S3, B] {
      def run(r: RR, s1: S1): F[(W, B, S3)] = {
        F.bind(self.run(r, s1)) {
          case (w1, a, s2) => {
            F.map(f(a).run(r, s2)) {
              case (w2, b, s3) => (W.append(w1, w2), b, s3)
            }
          }
        }
      }
    }

  ...
```

- [Scalaz - IndexedReaderWriterStateT](https://github.com/scalaz/scalaz/blob/series/7.2.x/core/src/main/scala/scalaz/ReaderWriterStateT.scala#L4)
- [Scalaz - ReaderWriterStateTMonad](https://github.com/scalaz/scalaz/blob/series/7.2.x/core/src/main/scala/scalaz/ReaderWriterStateT.scala#L179)

## RWST Example

예제를 위해 간단한 모델을 만들어 보겠습니다.

- `Reader` 로 `DatabaseConfig` 를
- `Writer` 로 `Vector[String]` 을
- `State` 로 `Connection` 을 이용하고

결과값으로 타입 `A` 를 돌려주는 `Task[A]` 를 만들면 아래와 같습니다.

```scala
object Database {
  type Task[A] = ReaderWriterState[DatabaseConfig, Vector[String] /* log */, Connection, A]
  ...
```

여기에 몇 가지 제약조건을 걸어보겠습니다.

- `DatabaseConfig.operationTimeoutMillis` 에 의해서 타임아(`OperationTimeoutException`) 발생
- `OperationTimeoutException` 발생시, 연산을 즉시 중단하고, 오류 없이 수행이 되었을 경우 *commit*
- *Post Commit Action* 등록을 할 수 있어야 하며, *commit* 후 순차대로 자동 실행

이제 필요한 몇몇 클래스를 만들고

```scala
type Action = () => Unit
case class PostCommitAction(id: String, action: Action)
case class DatabaseConfig(operationTimeoutMillis: Long)
case class ResultSet() /* dummy */

case class Connection(id: String,
                      actions: List[PostCommitAction] = Nil) {

  def commit = {}
  def executeAndReturn(query: String): ResultSet = ResultSet()
  def execute(query: String): Unit = {}
}

class OperationTimeoutException private(ex: RuntimeException) extends RuntimeException(ex) {
  def this(message:String) = this(new RuntimeException(message))
  def this(message:String, throwable: Throwable) = this(new RuntimeException(message, throwable))
}

object OperationTimeoutException {
  def apply(message:String) = new OperationTimeoutException(message)
  def apply(message:String, throwable: Throwable) = new OperationTimeoutException(message, throwable)
}
```

이제 사용자가 API 를 사용하는 것을 한번 상상해보겠습니다. *commit* 이 어쨌건, 사용자가 하고싶은 일은 쿼리를 실행해서 결과값을 받아오거나, 필요한 *post commit action* 을 등록하는 일일겁니다. 나머지는 다 알아서 해주겠거니 하고 기대하고 있겠지요. 아래와 같은 API 가 있다면,

```scala
def createTask[A](f: Connection => A): Task[A]
def addPostCommitAction(action: Action): Task[Unit]
def run[A](task: Task[A]): Option[A]
```

사용자들이 이런 방식으로 사용할 수 있습니다.

```scala
case class Person(name: String, address: Address)
case class Address(street: String)

def getPerson(name: String): Task[Person] = createTask { conn =>
  val rs: ResultSet = conn.executeAndReturn(s"SELECT * FROM USER WHERE name == '$name'")

  /* get a person using the result set */
  ...
}

def updateAddress(person : Person): Task[Unit] = createTask { conn =>
  /* do something */
  conn.execute(
    s"UPDATE ADDRESS SET street = '${person.address.street}' where person_name = '${person.name}'")
}

val getAndUpdatePersonTask: Task[Person] = for {
  p <- getPerson("1ambda")
  updatedP = p.copy(address = Address("BACON STREET 234"))
  _ <- addPostCommitAction(() => println("post commit action1"))
  _ <- updateAddress(updatedP)
  _ <- addPostCommitAction(() => println("post commit action2"))
} yield updatedP

val person: Option[Person] = Database.run(getAndUpdatePersonTask)
```

이제 상상했던 함수를 구현해 보면,

```scala
// https://github.com/1ambda/scala/blob/master/learning-scalaz/src/main/scala/readerwriterstate/Database.scala

import java.util.UUID
import scalaz._, Scalaz._
import Database._
import com.github.nscala_time.time.Imports._

object Database {

  ...
  object Implicit {
    implicit def defaultConnection: Connection = Connection(genRandomUUID)
    implicit def defaultConfig = DatabaseConfig(500)
  }

  private def genRandomUUID: String = UUID.randomUUID().toString

  private def execute[A](f: => A, conf: DatabaseConfig): A = {
    val start = DateTime.now

    val a = f

    val end = DateTime.now

    val time: Long = (start to end).millis

    if (time > conf.operationTimeoutMillis)
      throw OperationTimeoutException(s"Operation timeout: $time millis")

    a
  }

  def createTask[A](f: Connection => A): Task[A] =
    ReaderWriterState { (conf, conn) =>
      val a = execute(f(conn), conf)
      (Vector(s"Task was created with connection[${conn.id}]"), a, conn)
    }

  def addPostCommitAction(action: Action): Task[Unit] =
    ReaderWriterState { (conf, conn: Connection) =>

      val postCommitAction = PostCommitAction(genRandomUUID, action)
      (Vector(s"Add PostCommitAction(${postCommitAction.id})"),
        Unit,
        conn.copy(actions = conn.actions :+ postCommitAction))
    }

  def run[A](task: Task[A])
            (implicit defaultConf: DatabaseConfig, defaultConn: Connection): Option[A] = {

    \/.fromTryCatchThrowable[(Vector[String], A, Connection), Throwable](
      task.run(defaultConf, defaultConn)
    ) match {
      case -\/(t) =>
        println(s"Operation failed due to ${t.getMessage}") /* logging */
        none[A]

      case \/-((log: Vector[String], a: A, conn: Connection)) =>
        conn.commit /* close connection */

        log.foreach { text => println(s"[LOG] $text")} /* logging */

        /* run post commit actions */
        conn.actions foreach { _.action() }

        a.some
    }
  }
```

이제 실제로 *500 ms* 를 초과하는 연산을 실행하면, 예외가 발생하는 것을 확인할 수 있습니다.

```scala
  test("Database example") {

    val slowQuery: Task[Person] = createTask { conn =>
      sleep(600)
      Person("Sherlock", Address("BACON ST 221-B"))
    }

    val getPeopleTask: Task[List[Person]] = for {
      p1 <- getPerson("Mycroft")
      p2 <- getPerson("Watson")
      p3 <- slowQuery
      _ <- addPostCommitAction(() => println("post commit1"))
    } yield p1 :: p2 :: p3 :: Nil

    import Database.Implicit._
    val people = Database.run(getPeopleTask)

    // log: Operation failed due to java.lang.RuntimeException: Operation timeout: 603 millis
    people shouldBe None
}
```

## Previous Posts

- [Easy Scalaz 1, State](http://1ambda.github.io/easy-scalaz-1-state/)
- [Easy Scalaz 2, Monad Transformer](http://1ambda.github.io/easy-scalaz-2-monad-transformer/)


## References

- [Haskell Image](http://cs.lth.se/edan40)
- [Tooling The Reader Monad](https://coderwall.com/p/ye_s_w/tooling-the-reader-monad)
- [Reader Monad For Dependency Injection](http://blog.originate.com/blog/2013/10/21/reader-monad-for-dependency-injection/)
- [Slideshare: Reader Monad](http://slides.com/danielbedo/reader-monad)
- [Typeclassopedia Image](https://wiki.haskell.org/Typeclassopedia)
- [Scala Collection Performance Characteristics](http://docs.scala-lang.org/overviews/collections/performance-characteristics.html)
