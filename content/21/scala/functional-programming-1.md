+++
date = "2015-01-07T00:46:26+09:00"
next = "../functional-programming-2"
prev = "../"
title = "Functional Programming 1"
toc = true
weight = 111
aliases = [
    "/functional-programming-in-scala-chapter-1"
]
+++

### 1.1 Programming Paradigms

#### Imperative Programming

- modifying mutable variables  
- using assignment  
- and control str such as if-then-else, loop breakl continue, return  

절차적인 프로그래밍은 폰 노이만 구조랑 비슷한데,

- Mutable var = memory cells  
- variable deferences = load instructions  
- var assginment = store instsruction  
- control structure = jumps  

그런데 이런 instruction 들이 `word` 로 구성되어있으므로, 문제는  
> "Scaling up, How can we avoid conceptualizign programs word by word?"

결국 pure Imperative Programming 은 폰노이만 구조처럼 제한을 받는다고 볼 수 있다.

> "One tends to conceptualize data structures word-by-words"

그렇기 때문에 컬렉션, 다항식, 문자열과 같은 high-level abstraction 을 정의할 방법이 필요한데, 이상적으로는 **Theory** 를 만들면 해결할 수 있다. Theory 는 다음을 포함하는데

- one or more data types  
- operations on these types  
- laws that describe the relationships between values and opertions

그러나, 일반적으로 **Theory** 는 mutation 에 대해서는 describe 하지 않지만 Imperative Programming 에서는 mutation 때문에 theories 가 부서질(break) 수 있음. 따라서 다음과 같은 것들이 필요하다.

- Concentrate on defining theories for operators expressed as function  
- avoid mutations  
- have powerful way to abstract and compose functions  

정리하자면, Imperative Programming 에서는 high-level abstraction 을 위해 theory 를 이용할 수 있는데, Imperative Programming 에서는 mutation variable 을 이용하므로 theory 의 law 를 break 할 수 있다. 따라서 이런 단점을 해결하기 위해 나온 것이 Functional Programming 이다.

Functional Programming 에서는 상태가 Immutable 이기 때문에 **Theory** 를 구성하는 operator 를 만드는 것에 집중할 수 있다.

#### Functional Programming

- without mutable variables, assignments, loops  
- focuses on functions

FP offers the folloing benefits

- simpler reasoning principles  
- better modularity  
- good for exploiting parallelism fo mlticore and clod compting  

### 1.2 Elements of Programming

#### Expression

대다수의 언어들은 **expression** 과 관련해서 다음의 기능들을 제공한다

- primitive expressions, representing the simplest elements  
- ways to combine expressions  
- ways to abstract expressions, which introduce a name for an expression by which it can then be referred to.

#### Evaluation

**Non-primitive** expression 은 최종적으로 value 를 만들기 전까지 다음과 같은 방식으로 evaluated 된다

1. Take the leftmost operator
2. Evaluate its operands (left before right)
3. Apply the operator to the operands

**A name** is evaluatd by replacing it with the right hand side of its definition.

그러나 모든 expression 이 finite value 를 가지는 것은 아니다.

```scala
def loop: Int = loop
```

#### Evaluation of Function Applications

Applications of parameterized functions 은 다음과 같은 방식으로 evaluated 된다. operator 와 얼추 비슷하다

(1) Evaluate all function arguments, from left to right  
(2.1) Replace the function application by the function's right-hand side, and, at the same time
(2.2) Replace the formal parameters of the function by the actual arguments

```scala
def square(x: Int) = x * *
def sumOfSquares(x: Int, : Int) = square(x) + square(x)

sumOfSquares(3, 2+2)

// sumOfSquares(3, 4) : step (1)
// square(3) + square(4) : step (2)
// 3 * 3 + square(4) : step (1), (2)
// 9 + square(4)
// 9 + 4 * 4
// 9 + 16
// 25
```

인자를 먼저 평가하지 않을 경우 `square(2+2)` 가 `(2*2) + (2*2)` 로 reduced 되어 더 많은 계산을 야기할 수 있다.

#### The substitution model

이렇게 Evaluation 해 나가는 과정을 **The substitution model** 이라 부른다. 이 모델의 근간이 되는 아이디어는 **reducing an expression to a value** 이고, **side-effect** 가 없는 한 모든 expressions 에  용할 수 있다. 참고로 이 모델은 Functional Programming 의 근간이 되는 **lambda-calculus** 사용한 것이다.

#### Call-by-value, Call-by-name

`square(4)` 처럼 `sumOfSquares` 의 인자가 먼저 평가되는 방식을 **Call-by-value**, `square(2+2)` 처럼 나중에 인자가 평가되는 방식을 **Call-by-name** 이라 부른다.

두 가지 방식 아래 조건이 지켜지는 한 모두 expression 을 value 로 reduce 한다

- the reuced expression consists of pure functions, and  
- both evaluations terminate

- **Call-by-value** has the advantage that it evaluates every function argument only once  
- **Call-by-name** has the advantage that a function argument is not evaluated if the corresponding parameter is unused in the evaluation of the function body  

#### Examples

```scala
def test(x: Int, y: Int) = x *x
```

위와 같은 함수가 있다고 할때, `test(3, 4)` 는 똑같은 속도지만 `test(2+2, 3)` 은  **Call-by-value** 가, `test(2, 3+2)` 는 **Call-by-name** 이 더 빠르다

### 1.3 Evaluation Strategies and Termination

But what if termination is not guaranteed?

```scala
def test (x: Int, y: Int) = x * x
def loop () = loop
```

위의 예제에서 `test(3, loop)` 라는 expressions 은 **Call-by-name** 방식으로 평가될 수 있지만, **Call-by-value** 방식으로는 아니다.

Scala 은 일반적으로 re-computation 을 피하기 위해 **Call-by-value** 을 사용한다. 그러나 `def constOne(x: Int, y: => Int) = 1` 처럼 function parameter 가 `=>` 로 시작하면 해당 인자는 **Call-by-name** 을 이용한다.

```scala
def constOne(x: Int, y: => Int) = 1
def loop() = loop

constOne(1+2, loop) // will be evaluated
constOne(loop, 1+2) // will not be evaluated
```

### 1.4 Conditionals and Value Definitions

#### Conditional Expression

```scala
def abs(x: Int) = if (x >= 0) x else -x
```

In the above example, `x >= 0 ` is a predicate, of type Boolean. and `If-else` is an expression not a statement

#### short-circuit evaluation

Boolean 을 위한 Rule 을 다시 만들수 있다.

```
!true      --> false
!false     --> true
true && e  --> e
false && e --> false
true || e  --> true
false || e --> e
```

`&&` 와 `||` 의 경우에는 언제나 오른쪽 operand 가 평가되야 하는건 아닌데, 이러한 expression 을 보고 **short-circuit evaluation** 을 사용한다고 말한다.

#### Value Definitions

```scala
def x loop(): Booelan = loop
```

위의 식이 평가되는걸 보면 `def` 는 **Call-by-value** 를 이용하는걸 알 수 있다. 반대로 `val` 은 **Call-by-value** 를 사용한다. `val x = loop` 식을 평가하면, 무한 루프가 도는것을 확인할 수 있다.


#### Exercise

`&&` 와 `||` 없이 `and` 함수를 구현하려다 보면, 다음과 같이 구현하는 경우가 있는데,

```scala
def and(x: Boolean, y: Boolean = if (x) y else false
```

이 경우 `and(false, loop)` 를 평가하면 올바르게 동작하지 않고 무한루프에 걸린다 따라서 두번 째 인자가 **Call-by-name** 을 이용해 평가되도록, 아래와 같이 작성해야 한다.

```scala
def and(x: Boolean, y: => Booelan) = if (x) y else false
def or(x: Boolean, y: => Boolean) = if (x) true else y
```

### 1.5 Example: square roots with Newton's method

일단 시작 전에 먼저 말하자면, Scala 에서 recursive function 의 경우에는 explicit return type 이 필요하다.

[뉴튼-랩슨 법](http://kevin0960.tistory.com/entry/%EA%B3%A0%EC%B0%A8-%EB%B0%A9%EC%A0%95%EC%8B%9D%EC%9D%98-%ED%95%B4-%EA%B5%AC%ED%95%98%EA%B8%B0-%EB%89%B4%ED%8A%BC-%EB%9E%A9%EC%8A%A8%EB%B2%95-Newton-Rahpson)을 이용해서 제곱근을 구하는 Scala 코드를 작성하면

```scala
  def abs(x: Double) = if (x < 0) -x else x
  def sqrt(x: Int): Double = sqrtIter(1.0, x)
  def sqrtIter(guess: Double, x: Double): Double =
    if (isGoodEnough(guess, x)) guess
    else sqrtIter(improve(guess, x), x)

  def isGoodEnough(guess: Double, x: Double): Boolean =
    abs(guess * guess - x) / x < 0.0001

  def improve(guess: Double, x: Double) =
    (guess + x / guess) / 2
```

실수에 대해 작업할때는 엄청나게 커다란 수와 작은 수에 대해 테스트를 해 보아야 한다. 만약 `isGoodEnough` 의 구현이 `abs(guess * guess -x ) < 0.0001` 이라면 큰수에 대해서는 non-termination 이, 작은 수에 대해서는 invalid 한 값이 나올수 있다.

참고로, **scalatest** 에서는 **floating point number** 에 대한 테스틀 위해 다음과 같은 인터페이스를 지원한다.

```scala
sevenDotOh should equal (6.9 +- 0.2)
sevenDotOh should === (6.9 +- 0.2)
sevenDotOh should be (6.9 +- 0.2)
sevenDotOh shouldEqual 6.9 +- 0.2
sevenDotOh shouldBe 6.9 +- 0.2
```

### 1.6 Blocks and Lexical Scope

위에서 작성한 `sqrt` 를 block scope 를 이용하면 `x` 를 파라미터로 넘기는것을 제거해 간단히 만들 수 있다.

```scala
  def abs(x: Double) = if (x < 0) -x else x
  def sqrt(x: Int): Double = {

    def sqrtIter(guess: Double): Double =
      if (isGoodEnough(guess, x)) guess
      else sqrtIter(improve(guess))

    def isGoodEnough(guess: Double, x: Double): Boolean =
      abs(guess * guess - x) < 0.0001

    def improve(guess: Double) =
      (guess +  x / guess) / 2

    sqrtIter(1.0)
  }
```

#### Semicolons

Scala 에서 세미콜론은 옵션이지만 이와 관련된 이슈가 있다.

```scala
someLongExp
+ someOtherExp
```

이건 다음과 같이 interpreted 될 것이다.

```scala
someLongExp;
+ someOtherExp
```

Expression 이 분리 되는 것을 방지하기 위해 다음과 두 가지 방법을 이용할 수 있다.

```scala
someLongExp +
someOtherExp

// or

(someLongExp
+ someOtherExp)
```

### 1.7 Tail Recursion

```scala
  def gcd(a: Int, b: Int): Int =
    if (b == 0) a else gcd(b, a % b)

  def factorial(n: Int): Int =
    if (n == 0) 1 else n * factorial(n - 1);
```

다음과 같은 재귀 함수가 있을때, 자세히 보면 `factorial(4)` 의 경우 Expression 이 점점 길어진다. `(4 * 3 * 2 * (1 * 1)`

반대로 `gcd(3, 2)` 의 경우 계산과정을 살´보면 `gcd(3, 2)`, `gcd(2, 1)` 로 진행된다. 식이 점점 길어지는게 아니라, 함수에서 변수만 바뀌는걸 알 수 있다. 이 경우 저장해야할 지역변수가 없기 때문에 stack frame 을 재활용 할 수 있으며, 이런 Recursive call 을 **Tail Recursion** 이라 부른다. 영문 설명을 보면,

If a function calls itself as its last action, the function's stack frame can e reused. This is called **Tail Recursion**

> Tail recursive functions are iterative processes

In general, if the last action of a function consists of calling a function (which may be the same), one stack frame would be sufficient for both functions. Such calls are called **tail-calls**

**tail-recursion** 버전의 `factorial` 은 다음과 같다.

```scala
  def tailFactorial(n: Int) = {
    def loop(acc: Int, n: Int): Int =
      if (n == 0) acc else loop(acc * n, n-1)
    loop(1, n);
  }
```
