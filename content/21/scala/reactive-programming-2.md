+++
date = "2016-06-25T01:08:45+09:00"
next = "../reactive-programming-3"
prev = "../reactive-programming-1"
title = "Reactive Programming 2"
toc = true
weight = 122
aliases = [
    "/reactive-programming-2"
]
+++

## Stateful Object

지금까지 우리가 작성한 프로그램은 *side-effect free* 였기 때문에, **time** 이 중요한 요소가 아니였다. 무슨말인고 하니, 모든 프로그램은 *sequence of actions* 에 대해 항상 같은 결과를 주게 되어있었다.

이건 *substitution model* 에 반영되어 있다.

### Substitution Model

*substitution model* 을 복습해 보면, 프로그램의 *evaluation* 은 *rewriting* 이다.

```scala
def f(x1, ..., xn) = B; ... f(v1, ..., vn)
```

은 다음처럼 평가된다. 여기서 `B` 는 펑션 바디. 

```scala
def f(x1, ..., xn) = B; ... f(v1/x1, ..., vn/xn) B
```

예를 들어

```scala
def interate(n: Int, f: Int => Int, x: Int) =
  if (n == 0) x else iterate(n-1, f, f(x))
  
def square(x: int) = x * x
```

`iterate(1, square, 3)` 은 이렇게 평가된다.

```scala
if (1 == 0) 3 else iterate(1-1, square, square(3))
iterate(0, square, square(3))
iterate(0, square, 3 * 3)
iterate(0, square, 9)
if (0 == 0) 9 else iterate(0-1, square, square(9))
9
```

·¸런데 여기서 재미난 부분이 있다. *rewriting* 은 어느 *term* 에서나 일어날 수 있고, 모든 *종료되는* *rewriting* 은 같은 결과를 만든다.

> Rewriting can be done anywhere in a term, and all rewritings which terminated lead to the same solution

그리고 이 개념이 람다 대수와, 함수형 프로그래밍의 기반이다. 아래의 두 식은 같은 식이다. 

```scala
if (1 == 0) 3 iterate(1 - 1, square, square(3))

// ==
iterate(0, square, square(3))

// ==
if (1 == 0) 3 else iterate(1 - 1, square, 3 * 3)
```

어느 부분에 집중하냐에 따라 *term* 에서의 *rewriting* 이 달라질 수는 있으나, 결과는 같다. 이걸 *confluence (합류)* 라 부르기도 하고 [Church-Rosser Theorem](http://en.wikipedia.org/wiki/Church%E2%80%93Rosser_theorem) 이라 부르기도 한다.

### Stateful Object

지금까지는 *pure functional world* 의 이야기였다. 이제 좀 바깥 세상 이야기를 해보자. 상태가 변하고 하는것들.

일반적으로 *world* 를 *a set of objects* 로 정의할 수 있으며, 이 *object* 들은 시간이 지남에 따라 *change* 가 일어난다. 

> An object **has a state** if its behavior is influenced by its history

예를 들어서 *은행 계좌* 는 *state* 를 가지고 있다. 왜냐하면 다음 질문의 답이

> **"Can I withdraw 100 CHF?"**

계좌의 이전 상태들이 어땠는지에 따라 달라지기 때문이다. (*may vary over the course of the lifetime of the account*)

모든 *mutable state* 는 *variable* 을 이용해 만들 수 있는데 스칼라에서는 *value definition* 인 `val` 대신에 *variable definition* 인 `var` 을 이용한다.

실제로는 `var` 보다는 다수의 *variable* 을 가진 *object* 를 이용해 오브젝트를 나타낸다. 아래는 은행 계좌 예제

```scala
  class BankAccount {
    private var balance = 0

    def deposit(amount: Int): Unit = {
      if (amount > 0) balance = balance + amount
    }

    def withdraw(amount: Int): Unit = {
      if (0 < amount && amount <= balance) balance = balance - amount
      else throw new Error("insufficient")
    }
  }
```

여기서 `BankAcoount` 는 `deposit` 과 `withdraw` 가 어떻게 얼마나 호출되었는지(히스토리)에 따라 상태 `balance` 가 달라진다. 

여기서 지난번에 배웠던 `Stream` 의 구현을 잠깐 보자. 이건 *stateful object* 일까?

```scala
def const[T](hd: T, tl: => Stream[T]) = new Stream[T] {
  def head = hd
  private var tlOpt = Option[Stream[T]] = None
  def tail: T = tlOpt match {
    case Some(x) => x
    case None => tlOpt = Some(tl); tail
  }
}
```

이건 시스템이 어떠한가 하는 *가정* 에 따라 참일수도, 아닐수도 있다.

(1) 만약에 `tl` 연산에서 *side-effct* 가 없다면 `tlOpt` 를 캐시하기 위한 최적화는 이 클래스 외부에 대해 어떤 영향도 없으므로 스트림은 *stateful object* 가 아니다.

(2) 반대로 `tl` 을 계산하는 과정에서 *print* 나 등등 *side-effect* 가 발생한다´ 첫번째로 `tl` 을 호출하냐, 두번째로 호출하냐에 따라 *print* 가 발생할 수, 아닐수도 있으므로 *stateful object* 다.

그럼 이제 이런 은행계좌 클래스를 생각해 보자.

```scala
class BankAccountProxy(ba: BankAccount) {
  def deposit(amount: Int) = ba.deposit(amount)
  def withdraw(amount: Int) = ba.withdraw(amount)
}
```

사실 이 `BankAccountProxy` 클래스는 어느 *varaible* 도 가지고 있지 않지만, 이 클래스의 *behavior* 이 `ba` 의 *state* 를 결정하기 때문에 *stateful object* 다.

### Identity and Change

이번엔 두 *state* 가 같은지를 판별하는 문제를 생각해 보자.

*assignment* 는 두 *expression* 이 서로 같은지(*the same*)에 대한 질문을 던진다.

예를 들어 `val x = E; val y = E` 라 하자. 여기서 `E` 는 임의의 *expression*. 여기서 *assignment* 가 없다면 아래 식 처럼 바꿔 쓸 수 있다.

```scala
val x = E; val y = x
```

*property* 를 **referential transparency** 라 부른다. 근데 만약 *assigment* 가 있으면, 두 *formulation* 은 다를 수 있다.

```scala
val x = new BankAccount
val y = new BankAccount
```

#### Operational Equivalence

두 변수가 같은지를 판별하기 위해서, *같다* 라는 말을 좀 더 엄밀히 정의해 보자. 

> The precise meaning of **"being the same"** is defined by the property of *operational equivalence**

`x, y` 두 개의 *definition* 을 가지고 있다고 하자. 

> `x` and `y` are operationally equivalent if **no possible test** can disingquish between them

그러므로 우리는 `x, y` 가 같은지 비교하기 위해 

> Execute the definitions followed by an arbitrary sequence `f` of operations that involves `x` an `y`, observing the possible outcomes.

```scala
val x = new BankAccount
val y = new BankAccount
f(x, y)

// another execution
val x = new BankAccount
val y = new BankAccount
f(x, x)
```

> Then, execute the definitions with another sequence `S'` obtains by renaming all occurences of `y` by `x` in `S`

만약 이 실행 결과가 다르다면 `x, y` 는 다른것이고 그 반대로 모든 `(S, S')` 가 같은 결과를 돌려준다면 `x, y` 는 같다. 왜냐하면 더이상 구분할 수 없기 때문이다. 이게 바로 *operationally equivalence*

이제 이 *operational equvalence* 를 이용해서 위에서 나온 질문을 해결해 보자. *counter example* 로,

```scala
// sequence S
val x = new BankAccount
val y = new BankAccount

x deposit 30  // 30
y withdraw 20 // error: insufficient

// sequnece S'
val x = new BankAccount
val y = new BankAccount

x deposit 30  // 30
x withdraw 20 // 10
```

따라서 `x, y` 는 서로 다르다. 반면 `val y = x` 로 정의한다면 어떤 *operation* 도 두 변수를 구분할 수 없기 때문에 똑같다.

#### Assignment and Substitution Model

지금까지 논의한 바를 정리해 보면 *assignment* 가 도입됨에 따라 우리가 가진 *computation model* 이 적용 불가능해 졌다.

```scala
val x = new BankAccount
val y = x

// will be replaced to, but not correct
val x = new BankAccount
val y = new BankAccount
```

위 식은 아래 식처럼 치환되지만, 절대 같은 결과가 아니다. 다른 프로그램이 된다!

> *The substitution model* **ceases** to be valid when we add the *assignment*

*store* 개념을 도입하면 *substituion model* 을 적용 가능하지만, 이건 프로그램을 상당히 복잡하게 만든다.

*purely functional world* 에서 벗어나니 세상이 복잡해졌다. 어떻게 두 세계를 잘 버무릴수 있을까?

### Loops

사실 *loop* 는 *imperative programming* 에서 필수적인 요소는 아니다. *variable* 만으로 절차형 언어를 모델링하기에 충분하긴 한데, 어쨌든 있긴 하니까 함수형 언어에서도 모델링 할 수 있는 방법을 강구해보자. *function* 으로 할 수 있다. 

```scala
def power(x: Double. exp: Int): Double = {
  var r = 1.0
  var i = exp
  
  while (i > 0) { r = r * x; i = i - 1}
  r
}
```

스칼라에서 `while` 은 키워드니까, `WHILE` 을 이용해 루프를 모델링하는 함수를 만들어 보자.

```scala
def WHILE(condition: => Boolean)(body: => Unit): Unit = {
  if (condition) {
    body
    WHILE(condition)(body)
  } else ()
}
```

- *re-evaluation* 을 피하기 위해서 (인자로넘길때) `condition, body` 는 *by name* 으로 넘겨야 한다. 
- `WHILE` 은 *tail-recursive* 이므로 *constant stack-size* 를 가진다.

*repeat* 는 이런식으로

```scala
def REPEAT(body: => Unit)(condition: => Boolean): Unit = {
  body
  if (condition) ()
  else REPEAT(body)(condition)
}
```

*until* 을 만들고 싶으면 내부 함수를 만들면 된다.

```scala
// ref: https://gist.github.com/metasim/7503601
def REPEAT(body: => Unit) = new {
  def UNTIL(condition: => Boolean): Unit = {
    body
    if (condition) ()
    else UNTIL(condition)
  }
}

// test code
REPEAT {
  x = x + 1
} UNTIL (x > 3)
```

#### For-Loops

자바의 *for-loop* 를 단순히 *higher-order function* 를 사용하는것 만으는 모델링하기 어렵다. 왜냐하면 `i` 의 *declaration* 이 포함되어있기 때문이다.

```java
for (int i = 1; i < 3; i = i + 1) { System.out.print(i + " "); }
```

스칼라에서도 비슷한 루프를 제공하긴 하는데, 위 루프보다는 *extended* 루프에 더 가깝다.

```scala
for (i <- 1 until 3) { System.out.print(i + " ") }
```

*for loop* 는 *foreach combinator* 로  번역 된다. 참고로 *for expression* 은 `map, flatMap` 으로 번역된다.

```scala
def foreach(f: T => Unit): Unit = 
  // apply 'f' to each element of the collection
  
// example
for (i <- 1 until 3; j <- "abc") println(i + " " + j)

// translated to
(1 until 3).foreach(i => "abc" foreach (j => println(i + " " + j)))
```

### Discrete Event Simulation

지금까지 *world* 를 모델링 할 수 있는 *state* 와, 여기에 적용할 수 있는 *control structure* 를 살펴봤는데, 이걸 이용해서 시뮬레이션을 해보자.

#### Digital Circuit

*digital circuit* 은 **wires** 와 **functional components** 로 구성된다. 

> Wires transport signals thar are transformed by components

*signal* 을 `True, False` 로 표현하자. 그리고 기본 *components* 로

- Inverter
- AND Gate
- OR Gate

그러면 다른 컴포넌트들은 이 조합으로 만들 수 있다. 그리고 각 컴포넌트는 *delay* 를 가질 수 있다.

```scala
def inverter(input: Wire, output: Wire): Unit
def andGate(a1: Wire, a2: Wire, output: Wire): Unit
def orGate(o1: Wire, o2: Wire, output: Wire): Unit
```

이제 *half adder, HA* 를 만들어 보면 

![](http://i.msdn.microsoft.com/dynimg/IC141779.gif)
<p align="center">(http://msdn.microsoft.com)</p>

```scala
def halfAdder(a: Wire, b: Wire, c: Wire, s: Wire): Unit = {
  val d = new Wire
  val e = new Wire

  orGate(a, b, d)
  andGate(a, b, e)
  inverter(c, e)
  andGate(d, e, s)
}
```

![](http://upload.wikimedia.org/wikipedia/commons/thumb/8/83/Full_Adder_Blocks.svg/2000px-Full_Adder_Blocks.svg.png)
<p align="center">(http://en.wikibooks.org)</p>

![](http://hyperphysics.phy-astr.gsu.edu/hbase/electronic/ietron/fulladd.gif)
<p align="center">(http://hyperphysics.phy-astr.gsu.edu)</p>

그러면 *full adder* 는

```scala
def fullAdder(a: Wire, b: Wire, cin: Wire, sum: Wire, cout: Wire): Unit = {
  val s = new Wire
  val c1 = new Wire
  val c2 = new Wire

  halfAdder(b, cin, s, c1)
  halfAdder(a, s, sum, c2)
  orGate(c1, c2, cout)
}
```

#### Action

*discrete event simulator* 는 특정 *moment* 에서의 수행되는 *actions* 이다. 아무런 파라미터도 필요없는 *action* 은

```scala
type Action = () => Unit
```

어떤 *side-effect* 를 수행하면서 *time* 을 시뮬레이션 할 수 있다.


#### Simulator

```scala
  trait Simulation {
    def currentTime: Int = ???
    def afterDelay(delay: Int)(block: => Unit): Unit = ???
    def run(): Unit = ???
  }
```

- `currentTime` returns the current simulated time
- `afterDelay` registeres an action to perform after a certain delay
- `run` performs the simulation until there are no more actions wating

이렇게 시뮬레이터 *trait* 를 만들고 상속 구조를 만들면

```scala
 Simulation
     |
   Gates     // Wire, AND, OR, INV
     |
  Circuits   // HA, ADDER  
     |
MySimulation // implementation
```

#### Wire

```scala
class Wire {
  private var sigVal = false
  private var actions: List[Action] = List()

  def getSignal: Boolean = sigVal
  def setSignal(s: Boolean): Unit =
    if (s != sigVal) {
      sigVal = s
      actions foreach (_())
    }
    
 def addAcion(a: Action): Unit = {
    actions = a :: actions
    a()
  }
}
```

#### Inverter, AND, OR Gates

*input wire* 로 부터의 입력을 뒤집어서 *output wire* 에다 돌려주는 `inverter` 를 구현할 수 있다. 근데 회로에서는 *delay* 가 있기 때문에 바로는 뒤집기 보다는 `InvertDelay` 이후에 신호를 반전시키는 *action* 을 구현하자. 아까 `Simulation` 내의 `afterDelay` 를 구현하면 된다.

```scala
  def andGate(in1: Wire, in2: Wire, output: Wire): Unit = {
    def andAction(): Unit = {
      val in1Signal = in1.getSignal
      val in2Signal = in2.getSignal
      afterDelay(AddGateDelay) { output setSignal (in1Signal & in2Signal) }
    }

    in1 addAction andAction
    in2 addAction andAction
  }

  def orGate(in1: Wire, in2: Wire, output: Wire): Unit = {
    def orAction(): Unit = {
      val in1Signal = in1.getSignal
      val in2Signal = in2.getSignal
      afterDelay(OrGateDelay) { output setSignal (in1Signal | in2Signal) }
    }

    in1 addAction orAction
    in2 addAction orAction
  }

  def inverter(input: Wire, output: Wire): Unit = {
    def invertAction(): Unit = {
      val inputSig = input.getSignal
      afterDelay(InverterDelay) { output setSignal (!inputSig) }
    }
    input addAction invertAction
  }
```

<br/>

~~혼란이 오기 시작했다~~

> What happens if we compute `in1Sig` and `in2Sig` inline inside `afterDelay` instead of computing them as value?

당연히 *delay* 후의 *signal* 값을 가지고 *action list* 에 추가하므로 제대로 모¸링하지 못한다.

```scala
afterDelay(OrGateDelay) {
  output setSignal (in1.getSignal | in2.getSignal)
}
```

### The Simulation Trait

지금까지를 정리하면, `Wire` 클래스는 *statefule object* 를 나타낸다. 상태 `signal`, `actions` 는 `addAction, setSignal` 호출에 의해 정해진다. 이 함수들의 호출은 일종의 *event* 이며, *delay* 와 *action* 으로 구성된다.

그리고 이 모든 `Event` 는 `Simulation` *trait* 내 리스트로 저장된다. 일종의 할일 목록이나, *history* 로 보면 쉬울듯. 

```scala
type Action = () => Unit
case class Event(time: Int, action: Action)
private type Agenda = List[Event]
private var agenda: Agenda = List()

private var curtime: Int = 0
def currentTime: Int = curtime
```

현재 시뮬레이션 타임을 기록하기 위해 `curtime` 정의하고, 이를 이용해 `Event` 를 정의할 수 있다. `Event(curtime + delay, () => block)` 처럼

```scala
def afterDelay(delay: Int)(block: => Unit): Unit = {
  val item = Event(currentTime + delay, () => block)
  agenda = insert(agenda, item)
}

private def insert(ag: List[Event], item: Event): List[Event] = ag match {
  case first :: rest if first.time <= item.time => first :: insert(rest, item)
  case _ => item :: ag
}
```

그리고 `agenda` 에 있는 액션들을 처리할 `loop` 와 시뮬레이션을 돌릴 `run` 함수를 구현하면

```scala
private def loop(): Unit = agenda match {
  case first :: rest =>
    agenda = rest
    curtime = first.time
    first.action()
    loop()
  case _ =>
}

def run(): Unit = {
  afterDelay(0) {
    println("*** simulation started, time = " + currentTime + " ***")
  }

  loop()
}
```

그리고 시뮬레이션 자체는 아무런 *output* 도 주지 않기 때문에, 디버깅에 유용한 `probe` 함수를 추가하자. 디버깅용 *action* 을 추가한다 생각하면 쉽다. 일종의 *gate* 이기도 하다.

```scala
def probe(name: String, wire: Wire): Unit = {
  def probeAction(): Unit = {
    println(s"$name $currentTime value = ${wire.getSignal}")
  }

  wire addAction probeAction
}
```

이제 각 게이트마다의 딜레이를 위한 *trait* 를 만들자. 회로를 무엇으로 구성하냐에 따라 다를 수 있으므로 *trait* 으로 만드는건 좋은 선택이다.

```scala
trait Parameters {
  def InverterDelay = 2
  def AndGateDelay = 2
  def OrGateDelay = 2
}
```

#### Implementation

```scala

```

#### Test

```scala
trait Simulation {
  type Action = () => Unit
  case class Event(time: Int, action: Action)
  private type Agenda = List[Event]
  private var agenda: Agenda = List()

  private var curtime: Int = 0
  def currentTime: Int = curtime

  def afterDelay(delay: Int)(block: => Unit): Unit = {
    val item = Event(currentTime + delay, () => block)
    agenda = insert(agenda, item)
  }

  private def insert(ag: List[Event], item: Event): List[Event] = ag match {
    case first :: rest if first.time <= item.time => first :: insert(rest, item)
    case _ => item :: ag
  }

  private def loop(): Unit = agenda match {
    case first :: rest =>
      agenda = rest
      curtime = first.time
      first.action()
      loop()
    case _ =>
  }

  def run(): Unit = {
    afterDelay(0) {
      println("*** simulation started, time = " + currentTime + " ***")
    }

    loop()
  }

}

abstract class Gates extends Simulation {

  def AndGateDelay: Int
  def OrGateDelay: Int
  def InverterGateDelay: Int

  class Wire {
    private var sigVal = false
    private var actions: List[Action] = List()

    def getSignal: Boolean = sigVal
    def setSignal(s: Boolean): Unit =
      if (s != sigVal) {
        sigVal = s
        actions foreach (_())
      }

    def addAction(a: Action): Unit = {
      actions = a :: actions
      a()
    }
  }

  def andGate(in1: Wire, in2: Wire, output: Wire): Unit = {
    def andAction(): Unit = {
      val in1Signal = in1.getSignal
      val in2Signal = in2.getSignal
      afterDelay(AndGateDelay) { output setSignal (in1Signal & in2Signal) }
    }

    in1 addAction andAction
    in2 addAction andAction
  }

  def orGate(in1: Wire, in2: Wire, output: Wire): Unit = {
    def orAction(): Unit = {
      val in1Signal = in1.getSignal
      val in2Signal = in2.getSignal
      afterDelay(OrGateDelay) { output setSignal (in1Signal | in2Signal) }
    }

    in1 addAction orAction
    in2 addAction orAction
  }

  def inverter(input: Wire, output: Wire): Unit = {
    def invertAction(): Unit = {
      val inputSig = input.getSignal
      afterDelay(InverterGateDelay) { output setSignal (!inputSig) }
    }
    input addAction invertAction
  }

  def probe(name: String, wire: Wire): Unit = {
    def probeAction(): Unit = {
      println(s"$name $currentTime value = ${wire.getSignal}")
    }

    wire addAction probeAction
  }
}

abstract class Circuits extends Gates {

  // input a, b
  // output sum, carry
  def halfAdder(a: Wire, b: Wire, s: Wire, c: Wire): Unit = {
    val d = new Wire
    val e = new Wire

    orGate(a, b, d)
    andGate(a, b, c)
    inverter(c, e)
    andGate(d, e, s)
  }

  def fullAdder(a: Wire, b: Wire, cin: Wire, sum: Wire, cout: Wire): Unit = {
    val s = new Wire
    val c1 = new Wire
    val c2 = new Wire

    halfAdder(b, cin, s, c1)
    halfAdder(a, s, sum, c2)
    orGate(c1, c2, cout)
  }
}

trait Parameters {
  def InverterGateDelay = 2
  def AndGateDelay = 3
  def OrGateDelay = 5
}
```

테스트코드는

```scala
object test extends Circuits with Parameters
import test._
val in1, in2, sum, carry = new Wire

halfAdder(in1, in2, sum, carry)
probe("sum", sum)
probe("carry", carry)

in1.setSignal(true)
test.run()
in2.setSignal(true)
test.run()
```

결과는

```scala
sum 0 value = false
carry 0 value = false
*** simulation started, time = 0 ***
sum 8 value = true
*** simulation started, time = 8 ***
carry 11 value = true
sum 16 value = false
```

흐름을 정리해 보면,

(1) `halfAdder`  
(2) `orGate, andGate, invertGate`  
(3) `addAction`
(4) `a()` -> `afterDelay`  
(5) `insert` 에서 시간을 고려해 이벤트리스트에 삽입  
(6) `loop` 가 돌아가면서 모  `action` 실행

#### A Variant

*OR gate* 는 *AND, Invert* 로 구성될 수 있다. `a | b = ~(~a & ~b)` 이기 때문인데

```scala
def orGateAlt(in1: Wire, in2: Wire, output: Wire): Unit = {
  def orAction(): Unit = {
    val notIn1, notIn2, notOut = new Wire
    inverter(in1,notIn1);
    inverter(in2,notIn2);
    andGate(notIn1, notIn2, notOut)
    inverter(notOut, output)
  }
}
```

만약에 이 *OR Gate* 를 사용하면 어떻게 될까? 시간은 당연히 달라지고, 추가적인 이벤트도 발생한다.

### Summary

- *state* 와 *assignment* 는 모델을 더 복잡하게 만든다. 
- *referential transparency* 도 포기해야 한다

반면

- *discrete event simulation* 같은 특정 형태의 프로그램을 우아하게 작성할 수 있다
- 시스템은 *mutable list of actions* 로 표현되고
- *action* 이 호출되면 그 효과로 인해 오브젝트의 상태가 변한다
- 미래에 호출될 *action* 을 *install* 할 수 있다.


### References

(1) *Reactive Programming* by **Martin Ordersky**  
(2) [MSDN: Half Adder](http://msdn.microsoft.com/en-us/library/aa288734(v=vs.71).aspx)
