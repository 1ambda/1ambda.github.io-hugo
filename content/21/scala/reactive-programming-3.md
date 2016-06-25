+++
date = "2016-06-25T01:08:47+09:00"
next = "../reactive-programming-4"
prev = "../reactive-programming-2"
title = "Reactive Programming 3"
toc = true
weight = 123
aliases = [
    "/reactive-programming-3"
]
+++

## Try, Future, Promise 

이번시간엔 *Try*, *Future*, *Awaitable*, *Async*, *Promise* 에 대해 알아본다. ~~모나드가 삶을 윤택하게 하리라~~

### Monads and Effects

프로그래밍에서 4가지 본질적 *effects* 는

```
                  One           Many
                  
Synchronous     T/Try[T]     Iterable[T]
Asynchronous    Future[T]    Observable[T]
```

*asynchoronous computation* 을 살펴보기전에 *synchronous* 부터 살펴볼건데 간단한 어드벤쳐 게임으로 시작하자.

```scala
trait Adventure {
  def collectCoins(): List[Coin]
  def buyTreasure(coins: List[Coin]): Treasure
}

val adventu
re = Adventrue()
val coins = adventure.collectCoins()
val treasure = adventure.buyTreasure(coins)
```

여기서 `collecCoins, buyTreasure` 는 구현에 따라 실패할 수도 있다. 예를 들어

```scala
def collectCoins(): List[Coin] = {
  if (eatenByMonster(this)) throw new GameOverException("Ooops")
  List(Silver, Silver, Gold)
}
def buyTreasure(coins: List[Coin]): Treasure = {
  if (coins.sumBy(_.value) < treasureCost)
    throw new GameOverException("Nice try!")
  Diamond
}
```

그런데, 타입상으로는 `collectCoins, buyTreasure` 는 함수가 실패할 것이라는 어떠한 정보도 주지 않는다. `Try` 를 이용해 함수가 예외를 던질수도 있다는 것을 타입에 표시하자.

#### Try

아래는 `Try` 의 정의다.

```scala
abstract class Try[T]
case class Success[T](elem: T) extends Try[T]
case class Failure[T](t: Throwable) extends Try[Nothing]
```

이걸 게임 함수의 리턴값에 적용하면

```scala
import scala.util.{Try, Success, Failure}

def collectCoins(): Try[List[Coin]] = {
  if (eatenByMonster(this)) throw new GameOverException("Ooops")
  List(Silver, Silver, Gold)
}
def buyTreasure(coins: List[Coin]): Try[Treasure] = {
  if (coins.sumBy(_.value) < treasureCost)
    throw new GameOverException("Nice try!")
  Diamond
}

val adventure = Adventrue()
val coins: Try[List[Coin]] = adventure.collectCoins()
val treasure = coins match {
  case Success(cs) => adventure.buyTreasure(cs)
  case failure @ Failure(t) => failure
}
```

좀 귀찮다. 많이 귀찮다. *higher order function* 의 은혜를 받아 삶을 좀 윤택하게 해 보자.

```scala
def flatMap[S](f: T => Try[S]): Try[S]

def flatten[U <: Try[T]]: TRy[U]

def map[S](f: T => S): Try[T]

def filter(p: T => Boolean): Try[T]

def recoverWith(f: PartialFunction[Throwable, Try[T]]): Try[T]
```

여기서 `flatMap` 을 이용하면 코드가 상당히 이뻐질 것 같다.

사실 비밀을 하나 공개하자면, `Try[T]` 는 모나드다. 그 중에서 *exception* 을 다루는 모나드.

`Try` 모나드를 이용하면 *exception* 부분(`Try[T]`)은 알아서 다루어 주고, 우리가 다뤄야 할 `T` 부분에 집중하게 해준다.

`flatMap` 이 코드를 어떻게 바꾸는가 한번 보자.

```scala
val treasure: Try[Treasure] = 
  adventure.collectCoins().flatMap(coins => {
    adventure.buyTreasure(coins)
  }
```

리턴타입은 `Try[Treasure]` 인데 `Try` 패턴매칭이 사라졌다? 그게 바로 `flatMap` 이 해주는 일이다. 타입을 다시 보자.

```scala 
def flatMap[S](f: T => Try[S]): Try[S]
```

`T` 를 받아 `Try[S]` 를 돌려줄 함수만 넣어주면, 실제 `T` 를 이 함수에 넣기 위해 해야할 패턴매칭은 알아서 해준다.

그리고 지난 [1강](http://1ambda.github.io/reactive-programming-1/) 에서 모나드속에 있는 타입을 빼기 위해 *for expression* 을 이용했었다. 마찬가지로 `Try` 도 가능하다. `flatMap` 보다 더 이뻐진다.

```scala
val treasure: Try[Treasure] = for {
  coins <- adventure.collectCoins()
  treasure <- adventure.buyTreasure(coins)
} yield treasure
```

우측에서 `Try[T]` 를 리턴하고, `for` 가 알아서 `Try` 를 제거하고 좌측에 `T` 를 돌려준다.

`Try` 를 다루기 위한 *higher order function* 이 내부적으로 어떻게 돌아가는지 한번 살펴보자.

```scala
def map[S](f: T => S): Try[S] = this match {
  case Succes(value) => Try(f(Value))
  case failure @ Failure(t) => failure
}

def flatMap[S](f: T => Try[S]): Try[S] = this match {
  case Success(value) => try { f(value) } catch { cast t => Failure(t) }
  case failure @ Failure(t) => failure
}

object Try {
  def apply[T](r: => T): Try[T] = {
    try { Success(r) }
    catch { case t => Failure(t) }
  }
}
```

`flatMap` 내부에서 패턴매칭 및 예외 처리를 해준다. 

### Latency as an Effect

```
                  One           Many
                  
Synchronous     T/Try[T]     Iterable[T]
Asynchronous    Future[T]    Observable[T]
```

지금까지 `T/Try[T]` 에 대해서 봤다. 이번엔 *asynchronous* 로 옮겨가 `Future[T]` 를 한번 볼건데, 간단한 네트워크 프로그램을 모델링 하면서 배워보자.

```scala
trait Socket {
  def readFromMemory(): Array[Byte]
  def sendToEurope(packet: Array[Byte]): Array[Byte]
}

val socket = Socket()
val packet = socket.readFromMemory()
val confirmation = socket.sendToEurope(package)
```

이 코드도 이전의 어드벤쳐 게임처럼 실행중 어떤일이 발생할지 모른다. 예외가 발생하지 않았을때만 정상적으로 실행된다. 게다가 `readFromMemory`, `sendToEurope` 동안 함수가 블럭되면 프로그램은 멈춰있다. (*heavy effect*)

이걸 해결하는게 *Future* 모나드다. 이 모나드는 *exception* 과 *latency* 를 다룬다. `Future[T]` 의 정의는

```sala
import scala.concurrent._
import scala.concurrent._ExecutionContext.Implicits.global

trait Future[T] {
  def onComplete(callback: Try[T] => Unit)
     (implicit excutor: ExecutionContext): Unit
}
```

`Try[T]` 를 받는 콜백을 인자로 필요로 하는 `onComplete` 메소드가 있다. 아랫 부분에 `ExecutionContext` 는 백그라운드에서 다른 스레드로 돌리기 위해 사용하고, `implicit` 는 이런 디테일을 숨기기 위함이다.

*Future* 는 다른 버전으로 작성될 수도 있는데,

```scala
trait Future[T] {
  def onComplete(success: T => Unit, 
                 failed Throwale => Unit): Unit
                 
  def onComplete(callback: Observer[T]): Unit
}

trait Observer[T] {
  def onNext(value: T): Unit
  def onError(error: Throwable): Unit
}
```

이건 위 버전에서의 *callback* 을 좀 세분화 한것이다. 어차피 콜백이 `Try[T]` 를 받기 때문에 내부에서 *case* 로 분리해야 하는데, 미리 로직을 분리해서 각각의 경우에 대해 넘겨주는 것이다.

아니면 그 아래 `onComplete` 정의처럼 `Observer` 로 감싸서 줄 수 있다. 이것도 마찬가지로 성공했을때의, 실패했을때의 콜백이다.

이제 처음의 소켓 프로그램으로 돌아와서 *Future* 를 적용하면

```scala
trait Future[T] {
  def onComplete(callback: Try[T] => Unit)
     (implicit executor: ExecutionContext): Unit
}

trait Socket {
  def readFromMemory(): Future[Array[Byte]]
  def sendToEurope(package: ArrayByte]): Future[Array[Byte]]
}
```

이제 `readFromMemory(), sendToEurope()` 의 함수 호출이 긴 시간이 걸릴수 있겠구나 하고 `Future` 가 리턴입에 있음을 보고 알 수 있다.

*future* 는 참 좋은건데, 이걸 사용하면 아까 실행 코드는

```scala
// before
val socket = Socket()
val packet = socket.readFromMemory()
val confirmation = socket.sendToEurope(package)

// after
val socket = Socket()
val packet: Future[Array[Byte]] = socket.readFromMemory()

// can't compile
val confirmation: Future[Array[Byte]] = 
  packet onComplete {
    case Success(p) => socket.sendToEurope(p)
    case Failure(t) => ...
  }
```

잘 보면 `onComplete` 의 리턴타입은 `Unit` 이기 때문에 `confirmation` 은 `Future[Array[Byte]]` 가 될 수 없다.

한 가지 방법은 `confirmation` 을 내부에 넣는건데,   그러면 나머지 밑 부분 코드도 모두 `Success` 내부에 작성해야 한다. ~~자바스크립트 콜백헬~~

```scala
// can't compile
  packet onComplete {
    case Success(p) => 
      val confirmation = socket.sendToEurope(p)
      ...
      ...
      // callback hell
      ...
    case Failure(t) => ...
  }
```

이 문제를 해결하기 위해 *future* 를 만들 수 있다. `Future` 의 *companion object* 정의를 보면

```scala
object Future {
  def apply(body => T)
     (implicit context: ExecutionContext): Future[T]
}
```

예제를 보면

```scala
import scala.concurrent.ExecutionContext.Implicit.global
import akka.serializer._

val memory = Queue[EmailMessage](
  EmailMessage(from = "Erik",   to = "Roland")
  EmailMessage(from = "Martin", to = "Erik")
  EmailMessage(from = "Roland", to = "Martin"))
  
def readFromMemory(): Future[Array[Byte]] = Future {
  val email = queue.dequeue()
  val serializer = serialization.findSerializationFor(email)
  serializer.toBinary(email)
}

val packet: Future[Array[Byte]] = socket.readFromMemory()

packet onSuccess {
  case bs => socket.sendToEurope(p)
}

packet onSuccess {
  case bs => socket.sendToEurope(p)
}
```

이렇게 사용할 수 있다. 이 코드가 모두 실행되면, 이메일 큐에는 두개의 이메일이 남는다. **하나가 아니다!!** `Future`  **미래에 돌려줄 결과**를 가지고 있다고 보면 되는데, 하나의 결과에 대해 두개의 콜백을 호출해도 하나의 결과, 즉 이메일 하나만 뽑아먹었다는 사실은 변하지 않는다.

### Combinators on Futures

이제 *future* 가 무슨일을 하는지 알았으면, 이걸 어떻게 모나드스럽게 사용할지 알아보자. 단골손님 `flatMap` 과 그 친구들이 등장한다.

```scala
trait Awaitable[T] extends AnyRef {
  abstract def ready(atMost: Duration): Unit
  abstract def result(atMost: Duration): T
}

trait Future[T] extends Awaitable[T] {
  def filter(p: T => Boolean): Future[T]
  def flatMap[S](f: T => Future[S]): Future[S]
  def map[S](f: T => S): Future[S]
  def recoverWith(f: PartialFunction[Throwable, Future[T]]): Future[T]
}

objec Future {
  def apply[T](body: => T): Future[T]
}
```

`flatMap` 님을 이용해서 코드를 작성하자.

```scala
val socket = Socket()
val packet: Future[Array[Byte]] = socket.readFromMemor()
val confirmation: Future[ArrayByte]] = 
  packet.flatMap(p => {
    socket.sendToEurope(p)
  }
```

`flatMap` 의 정의를 보면 알겠지만, 함수 `f: T => Future[S]` 만 제공하면 앞의 `Future` 를 껍질을 벗겨, `T` 로 넣어준다. 근데 여기서 재밌는 사실은, `flatMap` 의 리턴 타입이 `Future[S]` 기 때문에 `confirmation` 도 같은 타입이 된다.

즉, `flatMap` 을 이용하면 모나드를 체이닝할 수 있다. 다른 예제도 좀 보자.

```scala
import scala.concurrent.ExecutionContext.Implicits.global
import scala.imaginary.Http._

object Http {
  def apply(url: URL, req: Request): Future[Response] = 
  { .. runs the http request asynchronously }
}

// buggy
def sendToEurope(packet: Array[Byte]): Future[Array[Byte]] = 
  Http(URL("mail.server.eu"), Request(packet))
    .filter(response => response.isOK)
    .map(response => response.toByteArray)
```

마지막 부분의 코드를 다양하게 활용해 보자.

```scala
def sendTo(url: URL, packet: Array[Byte]): Future[Array[Byte]] = 
  Http(URL("mail.server.eu"), Request(packet))
    .filter(response => response.isOK)
    .map(response => response.toByteArray)
    
def sendToAndBackup(packet: Array[Byte]):
  Future[(Array[Byte], Array[Byte])] = {
    val europeConfirm = sendTo(mailServer.europe, packet)
    val usaConfirm    = sendTo(mailServer.usa, packet)
    europeConfirm.zip(usaConfirm)
  }
```

는 정상적인 코드가 아니다. 유럽이나 미국 둘중 하나라도 실패하면, 전체가 실패한다. 다행히도 스칼라 디자이너가 이런 문제를 해결하기 위한 함수들 `recover, recoverWith` 를 준비 해 놓았다.

```scala
def recover(f: PartialFunction[Throwable, T]): Future[T]
def recoverWIth(f: PartialFunction[Throwable, Future[T]): Future[T]
```

타입을 잘보면 예외를 검사해서 다시 `Future[T]` 를 던지는 함수들이다. 특히 `recoverWith` 는 또 다른 비동기 연산을 할 수 있도록 `Future[T]` 를 지원한다.

위의 이메일 예제에 `recover, recoverWith` ¼ 적용해 보자.

```scala
def sendTo(url: URL, packet: Array[Byte]): Future[Array[Byte]] = 
  Http(URL("mail.server.eu"), Request(packet))
    .filter(response => response.isOK)
    .map(response => response.toByteArray)

def sendToAndBackup(packet: Array[Byte]): Future[Array[Byte]] = 
  sendTo(mailServer.europe, packet) recoverWith {
    case europeError => sendTo(mailServer.usa, packet) recover {
      case usaError => usaError.getMessage.toByteArray      
    }
  }
```

근데 마지막 부분에서 `usaError.getMessage.toByteArray` 가 별로 맘에 안든다.  미국으로 보내는건 백업일 뿐이고, 실제로는 유럽에 보내고 싶었다. 그래서 실패한 메세지를 받더라도 유럽쪽 에러를 받고 싶다. 또 다른 문제는 코드가 좀 못생겼다. 이 두가지 문제를 해결해보자.

```scala
def fallbackTo(that: => Future[T]): Future[T] = {
  if this future fails take the successful result
  of that future
  if that future fails too, take the error of
  this future
}
```

이런 메소드가 있다면 다음처럼 작성할 수 있다.

```scala
def sendSafe(packet: Array[Byte]): Future[Array[Byte]] = 
  sendTo(mailServer.europe, packet) fallbackTo {
    sendTo(mailServer.usa, packet)
  } recover {
    case europeError => europeError.getMessage.toByteArray
  }
```

`fallbackTo` 의 구현은 

```scala
def fallbackTo(that: => Future[T]): Future[T] = {
  this recoverWith {
    case _ => that recoverWith { case _ => this }
  }
}
```

`Try` 에 실패했을때 복구하는 `Try` 를 만들 수 있다.

```scala
object Try {
  def apply(f: Future[T]): Future[Try[T]] = 
    f.map(s => Success(s)) recover { case t => Failure(t) }
}
```

#### Awaitable

```scala
trait Awaitable[T] extends AnyRef {
  abstract def ready(atMost: Duration): Unit
  abstract def result(atMost: Duration): T
}

trait Future[T] extends Awaitable[T] {
  def filter(p: T => Boolean): Future[T]
  def flatMap[S](f: T => Future[S]): Future[S]
  def map[S](f: T => S): Future[S]
  def recoverWith(f: PartialFunction[Throwable, Future[T]]): Future[T]
}
```

때때로 *asynchronous* 보다는 *blocking* 을 원할 수 있다. 그럴때는 `Awaitable` 을 사용하면 된다. 지정된 시간동안 블럭 후에 `result` 함수는 모나드를 벗겨 `T` 를 돌려준다.

예를 들어

```scala
val socket = Socket()
val packet: Future[Array[Byte]] = socket.readFromMemory()
val confirmation: Future[Array[Byte]] = 
  packet.flatMap(socket.sendToSafe(_))

val c = Await.result(confirmation, 2 seconds)
println(c.toText)
```

여기 잘 보면 `2 seconds` 라고 썼는데, 진짜 동작하는 코드다.

```scala
import scala.language.postFixOps

object Duration {
  def apply(length: Long, unit: TimeUnit): Duration
}

val fiveYears = 1826 minutes
```

### Composing Futures

```scala
val socket = Socket()
val packet: Future[Array[Byte]] = 
  socket.readFromMemory()
  
val confirmation: Future[Array[Byte]] = 
  packet.flatmap(socket.sendToSafe(_))
```

위에서 이런 코드를 작성했었다. 당연히 *for expression* 으로 변환할 수 있다.

```scala
val socket = Socket()

val confirmation: Future[Array[Byte]] = for {
  packet  <- socket.readFromMemory()
  confirm <- socket.sendToSafe(packet)
} yield confirm
```

여기에 더 많은 *control flow* 를 도입하려면 어떻게 해야할까? `flatMap` 만으로는 좀 부족해보인다. 예를 들어 정해진 횟수만큼 *retry* 를 하고싶다고 하자. 이런 함수를 만들어야 하는데,

```scala
def retry(times: Int)(block: => Future[T]): Future[T]
```

재귀로 구현하면

```scala
def retry(times: Int)(block: => Future[T]): Future[T] = {
  if (times == 0) Future.failed(new Exeception("Sorry")
  else 
    block fallbackTo { 
      retry(times - 1) { block }
    }
}
```

음... 못생겼다. 재귀긴 한데.. 에릭 마이어에 의하면 *recursion* 은 함수형 프로그래밍의 **GOTO** 라고 한다. 재귀 말고 *fold* 를 사용하자.

```scala
def retry(times: Int)(block: => Futurep[T]): Future[T] = {
  val ns: Iterator[Int] = (1 to times).iterator
  val attempts: Iterator[Future[T]] = ns.map(_ => () => block)
  val failed = Future.failed(new Exception)

  attempts.foldLeft(failed)
    ((a, block) => a recoverWith { block() })
}
```

즉, *future* 를 받아 `times` 만큼의 리스트를 만들어 놓고, *fold* 를 이용해 `recoverWith` 를 호출한다. 

따라서 `retry(3) { block }` 코드는 이렇게 확장된다.

```scala
((failed recoverWith block) recoverWith block) recoverWith block
```

만약 *foldRight* 를 이용하면


```scala
def fallbackTo(that: => Future[T]): Future[T] = {
  this recoverWith {
    case _ => that recoverWith { case _ => this }
  }
}

def retry(times: Int)(block: => Futurep[T]): Future[T] = {
  val ns: Iterator[Int] = (1 to times).iterator
  val attempts: Iterator[Future[T]] = ns.map(_ => () => block)
  val failed = Future.failed(new Exception)

  attempts.foldRight(() => failed)
    ((block, a) => () => { block() fallbackTo { a() } })
}

retry(3) { block } ()

// ==
block fallbackTo { block fallbackTo { block fallbackTo { failed }}}
```

잘보면 `foldRight` 부분에서 초기값이 `() => failed` 로 변했다. 이는 우리가 `fallbackTo` 를 이용하기 때문인데, `fallbackTo` 의 로직상 `this` 가 실패하면 `that` 을 시도하게끔 되어있다. `that` 이 성공하면 `that` 을 돌려준다.

우리는 이미 실패한 `block` 을 `a` 에 쌓아놨기 때문에, 이것을 그대로 돌려주려면 `() => ` 로 감싸서 성공할 수 있도록 해야한다.


### Async

타입에 *effect* 를 명시하는건 무슨일이 일어나는지 알려주니까 정말 좋긴 한데, 코드를 작성하기가 까다롭다. 좀 간단하게 할 수 있는 방법은 없을까?

```scala
import scala.async.Async._

def async[T](body: => T)
  (implicit context: ExecutionContext): Future[T]
  
def await[T](future: Future[T]): T
```

여기서 `async` 는 `Future` 의 팩토리라 보면 된다. 위에서 본 코드와의 다른점은, 내부에 `await` 함수를 사용할 수 있다. 얼핏 보면 `await` 은 블럭킹을 위한 `Awaitable` 과 비슷하게 보이기도 한다. `Future` 를 받아 `T` 를 돌려주니까.

```scala
trait Awaitable[T] extends AnyRef {
  abstract def ready(atMost: Duration): Unit
  abstract def result(atMost: Duration): T
}

// usage
Await.result(confirmation, 2 seconds)
```

그러나 놀랍게도 `await` 함수는 블럭되지 않는다. 코드를 보기전에 잠깐 설명서를 좀 보면

> **Illegal Uses**

> - await requires a directly-enclosing async; this means await must not be used inside a closure nested within in an async block, or insdie a nested object, trait, or class

> - await must not be used inside an expression passed as an argument to a by name parameter

> - await must not be used inside a Boolean short-circuit argument

> - return expression are illegal inside an async block

> - await should not be used under a **try / catch**

`try / catch` 구문을 이용할 수 없으므로 `Try` 모나드를 써야한다. 이제 위에서 봤던 `retry` 함수를 `await` 을 이용해서 작성하면

```scala
def retry(times: Int)(block => Future[T]): Future[T] = async {
  val i = 0
  var result: Try[T] = Failure(new Exception("sorry man!"))
  
  while (i < times && result.isFailure) {
    result = await { Try(block) }
    i += 1
  }
  
  result.get
}
```

코드가 좀 더 이해하기 쉬워졌다. 그리고 내부에서는 *mutable state* 를 사용할지라도 외부로는 여전히 *purely functional* 이다.

내친김에 `filter` 도 구현해 보자.

```scala
def async[T](body: => T)
  (implicit context: ExecutionContext): Future[T]
def await[T](future: Future[T]): T

def filter(p: T => Boolean): Future[T] = async {
  val x = await { this }
  
  if (!p(x)) throw new NoSuchElementException()
  else x
}
```

여기서 예외를 던지는 이유는 *empty future* 를 예외로 간주하기 때문이다. 앞서 코드에서도 그랬듯이.

`flatMap` 은 어떨까?

```scala
def async[T](body: => T)
  (implicit context: ExecutionContext): Future[T]
def await[T](future: Future[T]): T

def flatMap[S](f: T => Future[S]): Future[S] =
  async { await { f(await {this}) }}
```

### Promise

`await` 없이 `filter` 를 만들려면 `Promise` 를 사용할 수 있다.

```scala
def filter(pred: T => Boolean): Future[T] = {
  val p = Promise[T]()
  
  this onComplete {
    case Failure(e) => p.failure(e)
    case Success(x) => 
      if (!pred(x)) p.failure(new NoSuchElementException)
      else p.success(x)
  }
  
  p.future
}
```

`Promise` 의 정의를 보면

```scala
trait Promise[T] {
  def future: Future[T]
  def complete(result: Try[T]): Unit
  def tryComplete(result: Try[T]): Boolean
}

trait Future[T] {
  def onCompleted(f: Try[T] => Unit): Unit
}
```

`Promise` 는 `Future` 를 담고 있는데, `Future.onCompleted` 에 등록된 콜백 `f: Try[T] => Unit` 은, `Promise.complete` 에 의해 호출된다. 

`Promise.complete` 는 한번만 호출될 수 있다. 상식적으로 생각해봐도 그렇다. 따라서 `tryComplete` 를 만들어, 이미 완료되으면 `false` 를 얻어 검사한다.

재미난 예제를 하나 더 보자.

```scala
import scala.concurrent.ExecutionContext.Implicits.global

def race[T](left: Future[T], right: Future[T]): Future[T] = {
  val p = Promise[T]()
  
  left  onComplete { p.tryComplete(_) }
  right onComplete { p.tryComplete(_) }
  
  p.future
}
```

두 `left, right` *computation* 중 먼저 끝나는 연산이 돌려주는 `Try[T]` 가 `p.future.onComplete` 의 콜백에 삽입된다. 

어떤 리소스를 얻길 원하는데 로컬 캐싱값과 리모트 값 둘 중 먼저 얻어오는 것을 사용하려고 할 때 이런 코드를 작성할 수 있다. *HTML5* 에도 *worker*(?) 라고 이렇게 활용할 수 있는 기능이 있는걸로 안다.

`Promise` 에는 몇 가지 함수들이 더 있다.

```scala
trait Promise[T] {
  def future: Future[T]
  def complete(result: Try[T]): Unit
  def tryComplete(result: Try[T]): Boolean
  
  // helper method
  def success(value: T): Unit = this.complete(Success(value))
  def failure(t: Throwable): Unit = this.complete(Failure(t))
}
```

이제 아까 `filter` 로 다시 돌아가자.

```scala
// async version
def filter(p: T => Boolean): Future[T] = async {
  val x = await { this }
  
  if (!p(x)) throw new NoSuchElementException()
  else x
}

// promise version
def filter(pred: T => Boolean): Future[T] = {
  val p = Promise[T]()
  
  this onComplete {
    case Failure(e) => p.failure(e)
    case Success(x) => 
      if (!pred(x)) p.failure(new NoSuchElementException)
      else p.success(x)
  }
  
  p.future
}
```

`zip` 도 `Promise` 와 `await` 이용해 작성해 보자.

```scala
// promise version
def zip[S, R](that: Future[S], f: (T, S) => R): Future[R] = {
  val p = Promise[R]()
  
  this onComplete {
    case Failure(e) => p.failure(e)
    case Success(x) => that onComplete {
      case Failure(e) => p.failure(e)
      case Success(y) => p.success(f(x, y))
    }
  }
  
  p.future
}

// async version
def zip[S, R](p: Future[S], f: (T, S) => R): Future[R] = async {
  f(await { this }, await {that })
}
```

~~갓 async~~ 

시퀀스도 `await` 을 이용해서 구현하면

```scala
def sequence[T](fs: List[Future[T]]): Future[List[T]] = async {
  var _fs = fs
  var r = ListBuffer[T]()
  while (_fs != Nil) {
    r += await { _fs.head }
    _fs = _fs.tail
  }
  
  r.result
}
```

즉 `Future[T]` 를 하나씩 *async* 하게 얻어, 리스트로 돌려준다. 만약 이걸 `Promise` 로 구현하면

```scala
def sequence[T](fs: List[Future[T]]): Future[List[T]] = {
  val successful = Promise[List[T]]()
  successful.success(Nil)
  
  fs.foldRight(successful.future) {
    (f, acc) => for {x <- f; xs <- acc} yield x :: xs
  }
}
```

`Future[T]` 를 누적해서 리스트를 만들어야 하기 때문에 `Promise.complete(Nil)` 을 세팅해 이것의 `Promise.future` 를 `foldRight` 의 초기값으로 사용한다.

그리고 *for expression* 에서 `f: Future[T], acc: Future[List[T]]` 다. 따라서 `for` 구문에서 모나드가 벗겨져 `x: T, xs: List[T]` 이며 성공적¼로 `x` 를 가져오면 컨싱한다.

지금까지 `Try` 와 `Future` 를 살펴봤다. 다음엔 하나의 값이 아니라 컬렉션을 *async* 하게 어떻게 처리하나 알아보자.

```
                  One           Many
                  
Synchronous     T/Try[T]     Iterable[T]
Asynchronous    Future[T]    Observable[T]
```

### References

(1) *Reactive Programming* by **Martin Ordersky**  
