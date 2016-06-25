+++
date = "2016-06-25T01:08:50+09:00"
prev = "../reactive-programming-4"
title = "Reactive Programming 5"
toc = true
weight = 125
aliases = [
    "/reactive-programming-5"
]
+++

## Akka, Actor 

![](http://prabhubuzz.files.wordpress.com/2012/09/demo1.png)
<p align="center">(http://prabhubuzz.wordpress.com)</p>

*Actor* 는 원래 1973년에 인공지능 연구를 위해 개발되었는데, 1995년에는 *Erlang/OTP* 에서 텔레커뮤니케이션 플랫폼을 위해 사용되기도했다. 2006년에는 스칼라 스탠다드 라이브러리로 구현되었고, 2009년에는 *Akka* 가 만들어졌다.

### Why Actors?

액터가 왜 필요한지를 기존의 스레드를 사용하는 방법과 비교해 알아보자. 지난번에 배웠던 *bank account* 예제를 들고오면

```scala
class BankAccount {
  private var balance = 0
  
  def deposit(amount: Int): Unit = 
    if (amount > 0) balance = balance + amount
    
  def withraw(amount: Int): Int = 
    if (0 < amount && amount <= balance) {
      balance = balance - amount
      balance
    } else throw new Error("insufficient funds")
}
```

이 예제를 두개의 스레드로 동시에 돌리면 충돌이 난다. 잔고가 얼마인지는 ´떤 스레드가 나중에 실행되느냐에 따라 달라질 수 있다. 

이를 해결하기 위한 일반적인 방법은 *synchronization* 을 이용하는 것이다. *lock, mutex, semaphore* 등을 이용할 수 있다.

스칼라도 이를 지원하기 위해 모든 오브젝트마다 *synchronization* 블럭을 설정할 수 있다.

```scala
class BankAccount {
  private var balance = 0
  
  def deposit(amount: Int): Unit = this.synchronized { 
    if (amount > 0) balance = balance + amount
  }
    
  def withraw(amount: Int): Int = this.synchronized {
    if (0 < amount && amount <= balance) {
      balance = balance - amount
      balance
    } else throw new Error("insufficient funds")
  }
}
```

`deposit` 도 `balance` 를 수정하기 때문에 *synchronization* 블럭이 필요하다. 이 말은 결국 `balance` 를 수정하는 모든 곳에서 동기화 블럭을 작성해야 한다는 뜻이다.

```scala
def transfer(from: BankAccount, to: BankAccount, amount: Int): Unit = {
  from.synchronized {
    to.synchronized {
      from.withdraw(amount)
      to.deposit(amount)
    }
  }
}
```

과도한 동기화는 데드락을 만들 수 있다. 이를 피하기 위해 *ordered lock* 을 이용하는 등 다양한 방법이 있다. 그러나 코드가 복잡해진다. 이건 간단한 예제라 별로 복잡해질게 없지만, 더 커다란 예제라면 끔찍해진다.

- blocking synchronization introduces **dead-lock**
- blocking is bad for CPU utilization
- synchronous communication couples sender and receiver

*non-blocking object* 를 이용하되, 병렬로 실행할 수 있는 방법은 없을까? 그게 바로 액터다.

### The Actor Model

> The Actor Model represents objects and their interactions, resembling human organizations are built upon the laws of physics

- Actor is an object with **identity**
- Actor **has a behavior**
- Actor only interacts using **asynchonous message passing** 

```scala
type Receive = PartialFunction[Any, Unit]

trait Actor {
  def receive: Receive
  ...
}
```

액터 타입에서 말해주듯이 메세지를 받는다. 액터는 `PartialFunction[Any, Unit]` 에서 볼 수 있듯이 어떤 인자든 처리할 수 있지만 아무것도 돌려주진 않는다. 타입 자체가 *asynchronous* 하게 메세지를 처리함을 보여준다.

간단한 액터를 만들어 보면

```scala
class Counter extends Actor {
  var count = 0 
  def receive = {
    case "incr" => count += 1
  }
}
```

`"incr"` 만 받고, 자신은 메세지를 주지 않기 때문에 별로 할게 없다. 좀 더 예제를 키워보면

```scala
class Counter extends Actor {
  var count = 0 
  def receive = {
    case "incr" => count += 1
    case ("get", customer: ActorRef) => customer ! count
  }
}
```

*akka* 에서 액터는 `ActorRef` 라는 *address* 를 이용해서 메세지를 보낼 수 있다. `!` 는 *tell* 이라 읽는다. 메세지를 보낸단 뜻이다.

```scala
trait Actor {
  implicit val self: ActorRef
  def sender: ActorRef
  ...
}

abstract class ActorRef {
  def !(msg: Any)(implicit sender ActorRef = Actor.noSender): Unit
  def tell(msg: Any, sender: ActorRef) = this.!(msg)(sender)
  ...
}
```

얼랭에서는 매번 `sender` 를 지정하는 모양이다. 그러나 이건 자주 사용되는 패턴이기때문에 메세지를 보낼때 `sender` 를 *implicit* 하게 보내 이렇게 코드를 작성할 수 있다.

```scala
def receive = {
    case "incr" => count += 1
    case ("get", customer: ActorRef) => customer ! count
}

// same
def receive = {
    case "incr" => count += 1
    case "get" => sender ! count
}
```

#### The Actor's Context

```scala
trait ActorContext {
  def become(behavior: Receive, discardOld: Boolean = true): Unit
  def unbecome(): Unit
  ...
}

trait Actor {
  implicit val context: ActorContext
}
```

액터는 `ActorContext` 라는 *behavior stack* 을 가지고 있는데, `become` 이용해 *push* 하거나 `unbecome` 을 이용해 *pop* 할 수 있다.

```scala
class Counter extends Actor {
  def counter(n: Int): Receive = {
    case "incr" => context.become(counter(n + 1))
    case "get" => sender ! n
  }
  
  def receive = counter(0)
}
```

여기서 볼 수 있는 것은

- state change is explicit (`become`)
- state is scoped to current behavior (`context`)

일종의 *asynchronous tail-recursion* 과 비슷하다.

#### Creating and Stopping

액터는 액터에 의해서 생성되고, 자기 자신에 의해서도 수행이 중단될 수 있다.

```scala
trait ActorContext {
  def actorOf(p: Props, name: String): ActorRef
  def stop(a: ActorRef): Unit
}
```

이제 `Counter` 에게 메세지를 보내는 `CounterMain` 을 만들면

```scala
class CounterMain extends Actor {
  val counter = context.actorOf(Props[Counter], "counter")
  
  counter ! "incr"
  counter ! "incr"
  counter ! "incr"
  counter ! "get"
  
  def receive = {
    case count: Int = >
      println(s"count was $count")
      context.stop(self)
  }
}
```

돌리려면 `sbt "run-akka akka.Main CounterMain"` 처럼 실행해야 한다. *sbt* 세팅은

```scaka
import sbt._
import Process._
import Keys._

name := "reactive-programming"

version := "1.0"

scalaVersion := "2.11.2"

resolvers += "Typesafe Repository" at "http://repo.typesafe.com/typesafe/releases/"

libraryDependencies ++= Seq(
  "org.scalatest" % "scalatest_2.11" % "2.2.1" % "test",
  "io.reactivex" %% "rxscala" % "0.22.0",
  "com.typesafe.akka" %% "akka-actor" % "2.3.8"
)

testOptions in Test += Tests.Argument("-oI")
```

전체 코드는

```scala
package coursera.chapter5

import akka.actor.Actor
import akka.actor.Props

class Counter extends Actor {
  var count = 0

  def receive = {
    case "incr" =>
      println("Counter received 'incr'")
      count += 1
    case "get" => sender ! count
  }
}

// sbt run-main akka.Main coursera.chapter.CounterMain
class CounterMain extends Actor {

  val counter = context.actorOf(Props[Counter], "counter")

  counter ! "incr"
  counter ! "incr"
  counter ! "incr"
  counter ! "get"

  def receive = {
    case count: Int =>
      println(s"count was $count")
      context.stop(self)
  }
}

// run main actor
> run-main akka.Main coursera.chapter5.CounterMain
[info] Running akka.Main coursera.chapter5.CounterMain
Counter received 'incr'
Counter received 'incr'
Counter received 'incr'
count was 3
[INFO] [12/27/2014 16:23:40.182] [Main-akka.actor.default-dispatcher-2] [akka://Main/user/app-terminator] application supervisor has terminated, shutting down
[success] Total time: 1 s, completed Dec 27, 2014 4:23:40 PM
>
```

정리하자면 액터 모델에서의 *computation* 은

- send message
- create actors
- designate the behavior for the next message

### Message Processcing Semantics

액터의 상태를 직접적으로 변경할 수 있는 방법은 없다. 상태를 변경하려면 메세지를 보내야 한다. 그리고 메세지를 보내려면 *address, 주소* `ActorRef` 를 알고 있어야 한다.

- every actor knows its own address `self`
- creating an actor returns its address
- addresses can be sent within be sent within messages (e.g `sender`)

액터를  보면 서로 독립적인 *computation, 연산* 을 수행한다. 이들은 병렬적으로 실행되며, 서로 커뮤니케이션 할 수 있는 유일한 방법은 메세지를 보내는 것 뿐이다.

- local execution, no notion of global synchronization
- all actors run fully concurrently
- message-passing primitive is one-way communication

그리고 액터 하나를 기준으로 보면, *single-threaded* 로 처리 될 수 있다.

- messages are recived sequentially
- behavior change is effective before processing the next message
- processing one message is the atomic unit of execution

메세지를 처리하는 메소드는 *synchronized method* 처럼 동작하지만 블러킹 없이 큐에 메세지를 넣는것으로 대신한다.

#### Bank Account using Actor

액터의 메세지를 *companion object* 에 정의하는 것 부터 시작하자. 

```scala
package coursera.chapter5.banking

import akka.actor.Actor
import akka.actor.Props

object Account {
  case class Deposit(amount: BigInt) {
    require(amount > 0)
  }
  
  case class Withdraw(amount: BigInt) {
    require(amount > 0)
  }
  
  case object Done
  case object Failed
}

class Account extends Actor {

  import Account._

  var balance = BigInt(0)
  
  def receive = {
    case Deposit(amount) => 
      balance += amount
      sender ! Done                      
    case Withdraw(amount) if amount <= balance =>
      balance -= amount
      sender ! Done
    case _ =>
      sender ! Failed
  }
}
```

*transfer* 를 구현하려면 한 액터(`BankAccount`) 로 부터 인출하고, 다른 액터에게 같은 금액을 입금하라고 명령을 내리는 액터를 만들면 된다.

```scala
package coursera.chapter5.banking

import akka.actor.{Actor, Props, ActorRef}

object WireTransfer {
  case class Transfer(from: ActorRef, to: ActorRef, amount: BigInt)
  case object Done
  case object Failed
}

class WireTransfer extends Actor {
  import WireTransfer._

  def receive: Receive = {
    case Transfer(from, to, amount) =>
      from ! Account.Withdraw(amount)
      context.become(awaitFrom(to, amount, sender))
  }

  def awaitFrom(to: ActorRef, amount: BigInt, customer: ActorRef): Receive = {
    case Account.Done =>
      to ! Account.Deposit(amount)
      context.become(awaitTo(customer))
    case Account.Failed => 
      customer ! Failed
      context.stop(self)
  }

  def awaitTo(customer: ActorRef): Receive = {
    case Account.Done =>
      customer ! Done
      context.stop(self)
    case Account.Failed =>
      customer ! Failed
      context.stop(self)
  }
}
```

디버깅 위해 `akka.event.LoggingReceive` 를 구현한 `Main` 을 보자.

```scala
package coursera.chapter5.banking

import akka.actor.Actor
import akka.actor.Props
import akka.event.LoggingReceive

// in sbt
// > run-main akka.Main coursera.chapter5.banking.Main
class Main extends Actor {

  val accA = context.actorOf(Props[Account], "accA")
  val accB = context.actorOf(Props[Account], "accB")

  accA ! Account.Deposit(50)

  def receive = LoggingReceive {
    case Account.Done => transfer(50)
  }

  def transfer(amount: BigInt): Unit = {
    // transcation
    val tx = context.actorOf(Props[WireTransfer], "tx")

    tx ! WireTransfer.Transfer(accA, accB, amount)
    context.become(LoggingReceive {
      case WireTransfer.Done =>
        println("successfully transfered")
        context.stop(self)
    })
  }
}
```

실행하려면 세팅을 좀 바꿔야한다. 나는 `main` 에서 그냥 실행하니까, `main/resources` 밑에 `application.conf` 를 만들고 이 세팅을 넣으면 된다. `test` 에서 실행하면 마찬가지로 `test/resources` 에 넣으면 되고.

```
akka {
  loglevel = "DEBUG"
    actor {
      debug {
        receive = on
      }
    }
}
```

잡다구리한거 제거하고 로그를 뽑으면

```scala
> run-main akka.Main coursera.chapter5.banking.Main
[accountA] received handled message Deposit(50)
[Main] received handled message Done
[accountA] received handled message Withdraw(50)
[accountB] received handled message Deposit(50)
successfully transfered
[Main] received handled message Done
```

액터 모델에서 메세지 전송은 

> all communication is inherently unreliable

즉, 메세지가 전달되지 않을수도 있다는 뜻이다. 근데 이건 액터 모델뿐만 아니라 *synchronous* 에서도 마찬가지다. 컴퓨터가 크래시 나거나, 네트워크가 끊기거나.

그래서 프로토콜을 만들어 메세지가 실제로 전송되었는지 확인해야 한다.

> delivery of a message requires eventual availability of channel, recipient

3가지 전략을 사용할 수 있는데

- **at-most-once:** sending once deilvers `[0, 1]` times
- **at-least-once:** resending until acknoledged delivers `[1, ~` times
- **exactly-once:** processing only first reception delivers 1 time

두번째 전략의 경우 센더가 메세지를 다시 보내기 위해 가지고 있어야 한다. 세번째 전략은 가장 비용이 큰 전략인데, 리시버가 메세지가 이미 처리되었는지 아닌지를 판별할 수 있어야 한다.

*reliability* 를 위해 

- 모든 메세지가 *persisted* 되고 (저장된다는 뜻인듯)
- *unique correlation ID* 를 지정할 수 있고
- 성공할때까지 메세지를 계속 보낼 수 있다.

> Reliability can only be ensured by **business-level acknowledgement**

근데 실제로는 리시버로부터 응답이 오기전까진 제대로 보내졌는지 알 수 없으므로 비즈니스 레벨 측면에서 신뢰성을 정의해야한다는 뜻인것 같다.

은행계좌 예제에 지금까지 논의한 *reliability* 를 추가하기 위해

- log acivities of `WireTransfer` to persistent storage
- each transfer has a unique ID
- add ID to `Withdraw` and `Deposit`
- store IDs of completed actions within `BankAccount`

#### Message Ordering

본래 액터 모델에는 순서와 관련된 스펙이 없다고 한다. *akka* 에서는 이 부분을 수정하여 똑같은 목적지로 메세지를 보냈다면, 보낸 순서대로 메세지가 도착한다고 한다.

> If an actor sends multiple messages to the same destination, they will not arrive out of order (this is **Akka-specific**)

*E* 언어에서는 더 강화된 룰이 있다고 한다. *akka* 와는 좀 다른측면에 집중하기 때문이라고 함.

### Designing Actor System

액터 모델을 따르는 크롤러를 구현해보자. 웹페이지를 긁어오는 부분부터 보면

#### Web Client

```scala
val client = new AsyncHttpClient
def get(url: String): String = {
  val res = client.prepareGet(rul).execute().get
  if (res.getStatusCode < 400)
    res.getResponseBodyExcerpt(131072) // 128KB
  else throw BadStatus(response.getStatusCode)
}
```

근데, 긁어오는 부분이 블러킹이라 좀 그렇다. 

- 액터에서 이 코드를 사용하면 긁는동안 다른 요청에 반응을 못하고, 
- 이 액터가 반응을 못하면 다른 액터로 그 영향이 전파된다. 수천개의 액터를 가지고있다면..

```scala
private val client = new AsyncHttpClient

def ge(url: String)(implicit exec: Executor): Future[String] = {
  val f = client.prepareGet(url).execute();
  val p = Promis[String]()
  
  f.addListener(new Runnable {
    def run = {
      val res = f.get
      if (res.getStatusCode < 400)
        p.success(res.getResponseBodyExcerpt(131072))
      else p.failure(BadStatus(response.getStatusCode))
    }
  }, exec)
  
  p.future
}
```

#### Finding Links

```scala
val A_TAG = "(?i)<a ([^>]+)>.+?</a>".r
val HREF_ATTR = """\s*(?i)href\s*=\s*(?:"([^"]*)"|'([^']*)'|([^'">\s]+))\s*""".r

def findLinks(body: String): Iterator[String] = {
  for {
    anchor <- A_TAG.findAllMatchIn(body)
    HREF_ATTR(dquot, quot, bare) <- anchor.subgroups
  } yield if (dquot != null) dquot
  else if (quot != null) quot
  else bare
} 
```

`<a href=` 부분이 쌍따옴표나, 따옴표로 감싸져있거나, 아니면 그냥 링크부분이 문자열일 수 있으므로 나눠서 찾아본다.


#### Getter Actor

```scala
class Getter(url: String, depth: Int) extends Actor {
  implicit val exec = 
    context.dispatcher.asInstanceOf[Executor with ExecutionContext]
    
  val future = WebClient.get(url)
  
  future.onComplete {
    case Success(body) => self ! body
    case Failure(err)  => self ! Status.Failure(err)
  }
  
  ...
  
}
```

이건 너무나 자주 나오는 패턴매칭이므로 *akka* 에서는 `pipeTo` 를 이용해 이렇게 줄일 수 있다.

```scala
class Getter(url: String, depth: Int) extends Actor {
  implicit val exec = 
    context.dispatcher.asInstanceOf[Executor with ExecutionContext]
    
  val future = WebClient.get(url)
  future.pipeTo(self)
  
  // or
  WebClient get url pipeTo self
  ... 
}
```

여기서 `get` 과 `pipeTo` 가 비동기로 실행되야하기 때문에 `Executor` 가 필요하다. 그리고 이 `dispatcher` 는 상당히 중요한데, 이 디스패처가 액터를 실행하고, 퓨처를 실행한다. 그리고 이 디스패처는 공유될 수 있다.

> Actors are run by a dispatcher (potentially shared) which can also run Futures

메세지를 받는 부분은

```scala
class Getter(url: String, depth: Int) extends Actor {
  ...
  
  def receive = {
    case body: String =>
      for (link <- findLinks(body))
        context.parent ! Controller.Check(link, depth)
      stop()
    case _: Status.Failure => stop()
  }
  
  def stop(): Unit = {
    context.parent ! Done
    context.stop(self)
  }
}
```

모든 액터는 다른 액터에 의해 만들어지기 때문에 `context.parent` 로 접근할 수 있다.

#### Actor-Based Logging

- Logging includes *IO* which can block indefinitely
- *Akka*'s logging passes that task to dedicated actors

```scala
class Controller extends Actor with AtcorLogging {
  var cache = Set.empty[String]
  var children = Set.empty[ActorRef]
  
  def receive = {
    case Check(url, depth) =>
      log.debug("{} checking{}", depth, url)
      if (!cache(url) && depth > 0)
        children += context.actorOf(Props(new Getter(url, depth - 1)))
      cache += url
    case Getter.Done =>
      children -= sender
      if (children.isEmpty) context.parent ! Result(cache)
  }
}
```


다른 쿼리를 처리하는 도중에 `Result(cache)` 가 부모에게 전송되었다고 하자. *mutable set* 이면 끔찍하다. 그러나 *immutable set* 을 사용하기 때문에 공유해도 문제 없다.

#### Handling Timeouts

웹서버가 어마어마하게 늦게 응답한다면 어떻게 할까?

```scala
class Controller extends Actor with AtcorLogging {
  context.setReceiveTimeout(10 seconds)
  
  ...
  
  def receive = {
    case Check       ...
    case Getter.Done ...
    case ReceiveTimeout => children foreach (_ ! Getter.Abort)
  }
}

class Getter(url: String, depth: Int) extends Actor {
  def receive = {
    ...
    case Abort => stop()
  }
  
  ...
}
```

#### Scheduler

*akka* 는 *high volume*, *short duration*, *frequent cancellation* 을 위한 *time service* 를 제공한다. 스케쥴러도 그 중 하나다

```scala
trait Scheduler {
  def scheduleOnce(delay: FiniteDuration, target: ActorRef, msg: Any)
                   (implicit ec: ExecutionContext): Cancellable
  def scheduleOnce(delay: FiniteDuration)(block: => Unit)
                   (implicit ec: ExecutionContext): Cancellable
  def scheduleOnce(delay: FiniteDuration, run: Runnable)
                   (implicit ec: ExecutionContext): Cancellable                 
}
```

태스크의 실행과 취소가 리소스를 두고 경쟁할 수 있기 때문에 취소가 호출된 후 메세지를 받는 일도 생긴다. 그러나 취소요청된 메세지는 저장되기때문에 필터링 할 수 있어 별 문제가 안된다.

이 스케쥴러를 이용하면 위에서 본 취소 코드를 이렇게도 작성하면...**안된다**

```scala
class Controller extends Actor with AtcorLogging {
  import context.dispatcher
  var children = Set.empty[ActorRef]
  
  context.system.scheduler.scheduleOnce(10 seconds) {
    children foreach (_ ! Getter.Abort)
  }
  
  ...
}
```

스케쥴러와 액터의 컨텍스트가 다르기 때문에 동시에 `children` 을 수정하는 코드다. **not thread-safe** 다. 이건 컴파일러 에러도 안준다.

액터의 컨텍스트에 접근하려면 메세지를 보내야 한다.

```scala
class Controller extends Actor with AtcorLogging {
  import context.dispatcher
  var children = Set.empty[ActorRef]
  
  context.system.scheduler.scheduleOnce(10 seconds, self, Timeout)
  
  def receive {
    ...
    case Timeout => children foreach (_ ! Getter.Abort)
  }
  
  ...
}
```

비슷한 문제를 더 살펴보자.

```scala
class Cache extends Actor {
  val cache = Map.empty[String, String]
  def receive = {
    case Get(url) =>
      if (cache contains url) sender ! cache(url)
      else
        WebClient get url foreach { body =>
          cache += url -> body // buggy
          sender ! body
        }  
  }
}
```

이것도 액터 컨텍스트 밖에서 `cache` 에 접근하고 있다. `get` 은 `Future` 를 돌려주는데, 이건 분명히 액터의 컨텍스트가 아니다. `cache += url -> body` 이부분

이전과 마찬가지로 액터에게 메세지를 ³´내는 방식으로 해결할 수 있다. 명심하자 액터 내부의 변수는 액터로 메세지를 보내 변경하자.

```scala
  val cache = Map.empty[String, String]
  def receive = {
    case Get(url) =>
      if (cache contains url) sender ! cache(url)
      else
        WebClient get url map(Result(sender, url, _)) pipeTo self
     
     case Result(client, url, body) =>
       cache += url -> body
       client ! body
```

근데 이것도 문제가 있다. `map` 이 `Future` 에 의해 실행되기 때문에, `sender` 가 메세지를 보낸 사람이란걸 보장할 수가 없다.

> Do not refer to actor state from code running asynchoronously

```scala
  val cache = Map.empty[String, String]
  def receive = {
    case Get(url) =>
      if (cache contains url) sender ! cache(url)
      else
        val client = sender
        WebClient get url map(Result(client, url, _)) pipeTo self
     
     case Result(client, url, body) =>
       cache += url -> body
       client ! body
```

#### The Receptionist

```scala
class Receptionist extends Actor {
  def receive = wating
  
  // upon Get(url) start a traversal and become running
  val wating: Receive = {
    case Get(url) => context.become(runNext(Vector(Job(sender, url)))
  }
  
  // upon Get(url) append that to queue and keep running
  // upon Controller.Result(links) ship that to client
  // and run next job from queue if any
  def running(queue: Vector[Job]): Receive = {
    case Controller.Result(links) =>
      val job = queue.head
      job.client ! Result(job.url, links)
      context.stop(sender)
      context.become(runNext(queue.tail))
    case Get(url) =>
      context.become(enqueueJob(queue, Job(sender, url)))
  }
}

case class Job(client: ActorRef, url: String)
var reqNo = 0

def runNext(queue: Vector[Job]): Receive = {
  reqNo += 1
  if (queue.isEmpty) wating
  else {
    val controller = context.actorOf(Props[Controller], s"c$reqNo")
    controller ! Controller.Check[queue.head.url, 2)
    running(queue)
  }
}

def enqueueJob(queue: Vector[Job], job: Job): Receive = {
  if (queue.size > 3) {
    sender ! Failed(job.url)
    running(queue)
  } else running(queue :+ job)
}
```

`Receive` 가 `Actor` 의 *state, 상태* 를 나타낸다. 어떤 메세지를 받을 수 있는지 정의하면서

#### Main

```scala
import akka.actor.{Actor, Props, ReceiveTimeout}
import scala.concurrent.duration._

class Main extends Actor {
  import Receptionist._
  import context.dispatcher

  val receptionist = context.actorOf(Props[Receptionist], "receptionist")

  receptionist ! Get("http://www.google.com")

  context.system.scheduler.scheduleOnce(10 seconds, self, ReceiveTimeout)

  def receive = {
    case Result(url, links) =>
      println(links.toVector.sorted.mkString(s"Results for '$url':\n", "\n", "\n"))
    case Failed(url) =>
      println(s"Failed to fetch '$url'\n")
    case ReceiveTimeout =>
      context.stop(self)
  }

  override def postStop(): Unit = {
    WebClient.shutdown()
  }
}
```

전체 코드는 [여기](https://github.com/ardumont/scala-lab/tree/master/src/main/scala/concurrency/actor/crawler) 서 확인할 수 있다.

#### Summary

(1) A reactive application is **non-blokcing** & **event-driven** top to bottom  
(2) Actors are run by a dispatcher (potentially shared) which can also run Futures  
(3) Prefer imuutable data structures, since they can be shared  
(4) Do not refer to actor state from code running asynchronously  
(5) Prefre `context.become` for different states, with data local to the behavior

### Testing Actor System

테스팅은 오직 외부의 *observable effects* 만 가능하다.

> Tests can only verify externally observable effects

여기 `Toggle` 이란 액터를 하나 만들어 보자.

```scala
import akka.actor.Actor

class Toggle extends Actor {
  // happy state
  def happy: Receive = {
    case "How are you?" =>
      sender ! "happy"
      context become sad
  }

  // sad state
  def sad: Receive = {
    case "How are you?" =>
      sender ! "sad"
      context become happy
  }

  // initial state: happy
  def receive = happy
}
```

이 액터를 테스트할 수 있는 유일한 방법은 메세지를 보내고, 그 응답을 확인하는 것이다. *akka* 의 테스트킷은 `TestProbe` 를 제공하는데, 일종의 *remote controlled actor* 다.

*SBT* 파일에 이렇게 디펜던시를 추가하고

```scala
resolvers += "Typesafe Repository" at "http://repo.typesafe.com/typesafe/releases/"

libraryDependencies ++= Seq(
  ...
  "com.typesafe.akka" %% "akka-testkit" % "2.3.8",
  ...
)
```

아래처럼 사용할 수 있다.

```scala
implicit val system = ActorSystem("TestSys")
val toggle = sytem.actorOf(Props[Toggle])
val p = TestProbe()

p.send(toggle, "How are you?")
p.expectMsg("happy")
p.send(toggle, "unknown")
p.expectNoMsg(1 second)
...
...

system.shutdown()
```

이렇게 더 편하게 쓸 수 있다.

```scala
new TestKit(ActorSystem("TestSys")) with ImplicitSender {
  val toggle = system.actorOf(Props[Toggle])
  
  toggle ! "How are you"
  expectMsg("happy")
  ...
  ...
  system.shutdown()
}
```

`ImplicitSender` 를 이용하면 테스트 시스템 내에서 자동으로 *sender* 가 *test actor* 로 지정된다.

동작하는 코드는 [여기](https://github.com/ardumont/scala-lab/blob/master/src/main/scala/concurrency/actor/toggle/happySad.scala)로

<br/>

일반적으로 테스트를 작성할때는 독립적인 모듈을 먼저 테스트한다. 그리고 필요한경우 가짜 데이터를 돌려주는 *mock* 등을 만든다.

데이터베이스에서 데이터를 가져오는 모듈을 테스트하고 싶다면 그것만 테스트하면 된다. 실제 데이터베이스에 연결 해볼 필요는 없다. 

앞서 크롤러 같은 경우 `Receptionist` 를 테스팅하기 위해 가짜 `Controller` 를 만들 수 있다.

```scala
class FakeController extends Actor {
  def receive = {
    case Controller.Check(url, depth) =>
      context.system.scheduler.scheduleONce(1 seconds,
                                            sender,
                                            Controller.Result(Set(url)))
  }
}
```

### References

(1) *Reactive Programming* by **Martin Ordersky**  
(2) [http://prabhubuzz.wordpress.com](http://prabhubuzz.wordpress.com/2012/09/28/akka-really-mountains-of-concurrency/)


