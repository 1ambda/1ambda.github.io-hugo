+++
date = "2016-01-06T00:47:07+09:00"
next = "../functional-programming-3"
prev = "../functional-programming-1"
title = "Functional Programming 2"
toc = true
weight = 112
aliases = [
    "/functional-programming-in-scala-chapter-2"
]
+++

### 2.1 Higher-Order Functions

#### Higher-Order Functions

Functional PL 에서는 함수를 *first-class* 로 다루는데, 이는 함수를 파라미터로 넘기거나 결과로 리턴할 수 있다는 소리다.

이렇게 함수를 파라미터로 받거나, 혹은 함수를 리턴하는 함수를 **Higher order functions** 라 부른다.

#### Function Types

> type A => B is the type of a function that thaks an arg of type A and return a result of type B.

#### Anonymous Functions

> The type of the parameter can be omittered if it can be infferred by the compiler

### 2.2 Currying

2.1 에서 우리는 **Higher Order Functions** 를 만들었다. 다음과 같은 Tail-Recursive 버전의 `sum` 이 있다고 하자.


```scala
def TailRecursiveSum(f: Int => Int, a: Int, b: Int): Int = {
  def loop(a: Int, acc: Int): Int = {
    if (a > b) acc
    else loop(a + 1, f(a) + acc)
  }
  loop(a, 0);
}

def sumInts(a: Int, b: Int) =  sum(x => x, a, b)
def sumCubes(a: Int, b: Int) =  sum(x => x * x * x, a, b)
def sumFactorials(a: Int, b: Int) =  sum(fact, a, b)
```

`a` 와 `b`는 `sumInts` 와 `sumCubes` 로 부터 `sum` 으로 변하지 않고 넘어간다. 제거할 수 없을까?  
답은 간단하다. `sum` 이 `a, b` 를 받는 함수를 리턴하면 된다.

```scala
def sum(f: Int => Int): (Int, Int) => Int = {
  def sumF(a: Int, b: Int): Int = {
    def loop(a: Int, acc: Int): Int = {
      if (a > b) acc
      else loop(a + 1, f(a) +  acc)
    }
    loop(a, 0)
  }
  sumF
}
```

그러면 이렇게 인자를 숨길 수 있다.

```scala

def sumInts = Currying.sum(x => x)
assert(sumInts(1, 10) == 55)
```

`sum` 이 인자로 받은 `f` 를 적용한 새로운 함수 `sumF` 를 돌려주므로 다음과 같이 호출도 가능하다.

```scala
def cube = (x: Int) => x * x * x
sum(cube)(1, 10)
```

함수를 리턴하는 함수는 유용하기 때문에, 스칼라에서는 이를 위한 특별한 문법을 제공한다. 다음의 두 함수 `sum1`과 `sum2` 는 동일하다.

```scala
def sum1(f: Int => Int): (Int, Int) => Int = {
  def sumF(a: Int, b: Int): Int = {
    if (a > b) 0
    else f(a) + sum(a + 1, b)
  }
}

def sum2(f: Int => Int)(a: Int, b: Int): Int = {
  if (a > b) 0
  else f(a) + sum2(f)(a + 1, b)
}
```

#### Expansion of Multiple Parameter Lists

`def f(args1)...(argsn) = E` 가 있을때 이건 다음과 같이 함수로 감싸고 그 함수를 다시 돌려주면, 원 함수 `f` 에서 파라미터를 하나 줄일 수 있다.

`def f(args1)...(argsn-1) = { def g(argn) = E; g }` 만약 익명함수로 표현한다면,

`def f(args1)...(argsn-1) = (argsn => E)` 와 같이 표현할 수 있다. 따라서 이와 같이 함수로 감싸 원 함수 `f` 에서 파라미터를 반복적으로 줄이다 보면

`def f(args1)...(argsn) = E` 는 인자를 1개씩 받는 N개의 익명함수로 표현할 수 있다.

`def f = (args1 => (args2 => ...(argn => E) ...))` 이러한 스타일을 **currying** 이라 부른다.

#### More Function Types

그렇다면, 위에서 본 `sum` 함수의 타입 무엇일까? `def sum(f: Int => Int)(a: Int, b: Int): Int`

`(Int => Int) => Int, Int => Int` 로 표현할 수 있다. 근데 스칼라에서 **functional types associate to the right.** 이므로,

`(Int => Int) => (Int, Int => Int)` 와 동일하다.

#### Exercise

> (1). Write a `product` function that calculates the product of the value of a function for the points on a given interval

```scala
def product(f: Int => Int)(a: Int, b: Int): Int = {
  if (a > b) 1
  else f(a) * product(f)(a + 1, b)
}
```

> (2). Write `factorial` in terms of `product`

```scala
def factorial(n: Int): Int = {
  product(x => x)(1, n)
}
```

> (3). Can you write a more general funtion, which generalizes both `sum` and `product`

```scala
def mapReduce(f: Int => Int, combine: (Int, Int) => Int, init: Int)(a: Int, b: Int): Int = {
  if (a > b) init
  else combine(f(a), mapReduce(f, combine, init)(a + 1, b))
}

def sumUsingMapReduce(f: Int => Int)(a: Int, b: Int) =
  mapReduce(f, (x: Int, y: Int) => x + y, 0)(a, b)

def productUsingMapReduce(f: Int => Int)(a: Int, b: Int) =
  mapReduce(f, (x: Int, y: Int) => x * y, 1)(a, b)

```

### 2.3 Example: Finding Fixed Points

#### Finding a fixed point of a function

> A number is called a **fixed point** of a function f  if `f(x) = x`

어떤 `f` 들에 대해서는 `f(x)` 를 반복적으로 적용하면서 변하지 않거나 변화량이 충분히 작아질때를 찾아 **fixed point** 를 찾을 수 있다.

1장에서 만들었던 제곱근을 구하는 함수로 돌아가 보자. 사실 이 함수는 **fixed point** 와 관련이 있다. `sqrt(x) = y` 라고 했을때 `y * y = x` 이므로 `y = x / y` 다. 따라서 `sqrt(x)` 는 함수 `y = x / y` 를 꾸준히 적용해서 찾아낼 수 있으므로 `y = x / y` 의 **fixed point** 다.

다음과 같은 `fixedPoint` 함수가 있다고 하자.

```scala
val tolerance = 0.0001 // = 1.0E-4
def isCloseEnough(x: Double, y: Double) = {
  abs((x - y) / x) / x < tolerance
}

def fixedPoint(f: Double => Double)(firstGuess: Double): Double = {
  def iterate(guess: Double): Double = {
    val next = f(guess)
    if (isCloseEnough(guess, next)) next
    else iterate(next)
  }
  iterate(firstGuess)
}
```

이 함수를 이용해서

`sqrt(x: Int) = fixedPoint(y => x / y)(1.0)`

와 같은 제곱근을 구하는 함수를 만들어 볼 수 있겠다. 그러나 `sqrt(2)`
 를 실행하면 `guess` 값이 `1.0`과 `2.0` 사이를 널뛰기 하면서 무한 루프를 돈다.

이건 `guess` 값이 너무나 많이 변하기 때문인데, `f` 를 적용하는 시퀀스에서 연속적인 두개의 `guess` 값의 평균을 구하는 `f` 를 만듦으로서 이 문제를 피할 수 있다. (잘못된 해석일 수 있으므로 원문을 첨부한다.)

```scala
One way to control such oscillations is to prevent the estimation from varying too much. This is done by averaging successive values of the original sequence
```

#### Functions as return values

여태까지는 함수를 인자로 사용했을때 언어에서 어떤 이점을 얻을 수 있는가에 대한 설명이었고, 이제부터는 함수를 리턴값으로 사용할때의 장점을 알아 보자.

아까와 같이 averaging 함으로써 stabilizing 하는 기법을 **Average Damping** 이라 부르는데, 아래와 같은 함수를 만들어서 인자로 넘길 수 있다.

`def avgDamp(f: Double => Double)(x: Doube) = (x + f(x)) / 2`

따라서 `sqrt` 함수는 다음과 같이 새로 작성할 수 있다.

```scala
def avgDamp(f: Double => Double)(x: Double) = (x + f(x)) / 2
def sqrt(x: Double): Double = fixedPoint(avgDamp(y => x / y))(1)
```

`avgDamp` 는 `x` 를 다시 인자로 받는 함수를 돌려준다. 원래라면 `def` 를 이용해서 새로 함수를 만들고 리턴했어야 하나, 스칼라에서 지원해주는 문법을 이용해서 `(x: Double)` 을 추가하는 것 만으로 편하게 만들었다.

이번장에선 Higher order function 을 이용해서 함수를 combine 하면 더 강력한 **abstraction(추상화)** 를 얻을 수 있다는 법을 배웠다. 록 이 방법이 항상 좋은건  아니지만 배우면 다 쓸데가 있기 마련이다.

덤으로 하나 더 정리하자면, **Currying** 은 함수의 인자를 반복적으로 쪼개어 익명 함수로 만든 뒤 재활용 할 수 있도록 만드는 기술이라 보면 된다. 왜냐면 N개의 인자가 있다고 가정할때, 이전에는 1개의 함수에 다양한 인자를 줘야했지만, 커링을 이용하면 N개의 함수로 쪼갤 수 있고, 각각의 리턴되는 함수를 저장할 수 있으므로 각각을 재활용 할 수 있다.


### 2.5 Functions and Data

이번 장에서는 클래스를 사용한다. 유리수 계산을 하기 위해 `def add(n1: Int, d1: Int, n2: Int, d2: Int): Int` 와 같은 함수를 만드는 것이 아니라, 데이터를 추상화 하기 위한 방법으로 다음과 같은 클래스를 만들 수 있다.

```scala
class Rational(x: Int, y: Int) {
  def numer = x
  def denom = y

  def add(that: Rational) = {
    new Rational(
      numer * that.denom + that.numer * denom,
      denom * that.denom
    )
  }
}
```

참고로, 스칼라는 **type** 과 **value** 를 서로 다른네임스페이스에서 관리하기 때문에 충돌할 걱정을 할 필요가 없다.

#### Exercise

> (1). In your worksheet, add a method `neg` to class Rational that is used like this `x.neg // -x`

> (2). Add a method sub to subtract two rational numbers

> (3). With the values of x, y, z as given in the previous slide, what is the result of `x - y - z`

```scala
class Rational(x: Int, y: Int) {
  def numer = x
  def denom = y

  def add(that: Rational) = {
    new Rational(
      numer * that.denom + that.numer * denom,
      denom * that.denom
    )
  }

  def sub(that: Rational) = {
    add(that.neg)
  }

  def neg = new Rational(-numer, denom)
}
```

### 2.6 More Fun With Rationals

이전에 만든 `Rational` 클래스는 약분된 형태로 표현되지 않기 때문에 이런 기능을 추가할 필요가 있다.

> reduce them to their smallest numerator and denominator by dividing both with a divisor

다양한 방법으로 구현할 수 있겠지만, 가장 쉬운 방법은 `Rational` 오브젝트가 생성될때 약분 하는 방법이다.

```scala
  private def gcd(a: Int, b: Int): Int = if (b == 0) a else gcd(b, a % b)
  private val g = abs(gcd(x, y))

  def numer = x / g
  def denom = y / g
```

#### Self-reference

그리고, `less`, `max` 와 같은 함수도 만들어 볼 수 있다. `max` 가 **self-referencing** 을 위해 `this` 키워드를 사용한다는 점에 주목하자.

```scala
  def less(that: Rational) = numer * that.denom < that.numer * denom
  def max(that: Rational) = if (this.less(that)) that else this
```

#### Pre-condition

`new Rational(1, 0)` 을 시도하면 `0` 으로 나눌수 없기 때문에 에러를 뿜는다. 검사하기 위해 `require` 함수를 사용할 수 있다.

`require(y > 0, "denum must be != 0`

`assert` 도 사용할 수 있는데, `require` 와는 차이가 있다. `AssertionError` 가 나오고, `require` 에는 `IllegalArgumentException` 이 나온다. 그래서 서로 다른 의도로 쓰이게끔 만들어졌다는걸 알 수 있다. (오역이 있을 수 있기 때문에 원문을 첨부한다.)

> The reflects a difference in intent </br><br/>
> - `require` is used to enforce a precondition on the caller of a function <br/>
> - `assert` is used as to check the code of the function itself

만약 예외를 테스트한다면 `scalatest` 에서는 `intercept` 를 이용하면 된다.

```scala
  "Rational(1, 0)" should "throw IllegalArgumentException" in {
    intercept[IllegalArgumentException] {
      new Rational(1, 0)
    }
  }
```

#### Constructor

스칼라에서는 암시적(implicitly) 인 생성자를 도입했는데, 다시 말해 코드상에 없어도 **Primary Constructor** 가 존재하는데, 이 **Primary Constructor** 는 클래스의 파라미터를 받아서 클래스 바디의 모든 문장을 실행한다. 만약 다른 생성자를 만들고 싶으면, 다음과 같이 작성하면 된¤.

`def this(x: Int) = this(x, 1)`

우측에 나오는 `this` 는 **implicit primary constructor** 다.


#### Exercise

> Modify the `Rational` class so that rational numbers are kept unsimplified internally, but the simplification is applied when numbers are converted to strings. Do clients observe the same behavior when interacting with rational class? </br>

primary constructor 에서 약분을 하지 않고, `toString` 에서 약분을 할때 과연 제대로 되겠느냐인데, 답은 **아니오** 다. integer overflow 를 생각하면 쉽다. 최대한 약분할 수 있을때 먼저 해버리는것이 낫다. 커다란 수 `a` ... `z` 에 대해서 연산 해버리면, 마지막 `toString` 에서만 약분이 될텐데. 제대로 되지 않을 가능성이 있다.

> Yes for small sizes of denominators and nominators and small numbers of operations

### 2.7 Evaluation and Operators

#### Classes and Substitutions

(단어 오역이 있을 수 있어서 용어를 그대로 씀)

함수에서 **subtitution** 에 기반한 **compuation model** 을 이용했는데, 사실 이건 클래스를 인스턴스 할때도 똑같이 적용된다. 즉 `new C(x1, ... ,xn)` 은 `new C(v1, ..., vn)` 과 같다.

그렇다면, 다음과 같이 클래스가 인자 n 개를 받는 함수를 정의했을때 `def f(y1, ... , yn) = b` 이런 식은 어떻게 평가될까?

`new C(v1, ..., vm).f(w1, ... , wn)`

(1). 클래스의 **formal parameter** `x1, ..., xn` 이 **actual parameter** `v1, ... , vm` 으로 **substitution** 된다.<br/>

(2). 함수 `f` 의 **formal parameter** `y1, ... , yn` 이 **actual parameter** `w1, ... , wn` 으로 **substitution** 된다. <br/>

(3) `new Class(v1, ... , vm)` 이 `this` 로 치환되고

(4) `f` 의 바디 `b` 가 평가된다.

따라서 이를 식으로 표현하면, 다음과 같이 쓸 수 있다.

`[v1/x1, ... , vm/xm][w1/y1, ... , wn/yn][new C(v1, ..., vm)/this]b`

그러면, 예제를 통해서 살펴보자.

```scala
new Rational(1, 2).less(new Rational(2, 3))
```

이건 ¤음과 같이 평가된다.

```scala
[1/2, 2/y][new Rational(2, 3)/that][new Rational(1, 2)/this] this.numer * that.denom < that.numer * this denom
```

결국 이건 아래와 같다.

```scala
new Rational(1, 2).numer * new Rational(2, 3).denom < new Rational(2, 3).numer * new Rational(1, 2).denom

// 1 * 3 < 2 * 2
// true
```

#### Operators

`Int` 의 경우에는 `+` 를 사용하면 `3 + 5` 와 같이 표현할 수 있지만 `Rational` 의 경우에는 `r.add(r2)` 와 같이 사용해야 했다. 뭔가 불편하다.

스칼라에는 이런 문제를 해결하기 위해 **Infix Notation** 이 있다. 함수의 인자가 하나라면, 괄호를 생략하는 것이다. 바이너리 오퍼레이터처럼 보일 수 있도록.

```scala
r add s // r.add(s)
r less s // r less s
r max s // r max s
```

#### Relaxed Identifiers

스칼라에서는 **operator** 또한 **identifier** 가 될 수 있다. 스칼라에서 **identifier** 룰은 아래와 같다

> (1). **Alphanumeric:** starting with a letter, followed by a sequence of letters or numbers <br/>

> (2). **Symbolic:** starting with an operator symbol, followed by other character operator symbols <br/>

> (3). The underscore character `_` counts as a letter <br/>

> (4). Alphanumeric identifiers can also end in an underscore, followed by some operator symbols

따라서 `*`, `+?%&`, `vector_++`, `counter_=` 모두 유효한 identifier 들이다.

이제 이런 symbolic identifier 들을 이용해서 `Rational` 클래스의 함수 이름들을 산술연산 처럼 보이도록 변경해 보자.

```scala
class Rational(x: Int, y: Int) {
  require(y > 0, "denom != 0")

  // secondary constructor
  def this(x: Int) = this(x, 1)

  private def gcd(a: Int, b: Int): Int = if (b == 0) a else gcd(b, a % b)
  private val g = abs(gcd(x, y))

  def numer = x / g
  def denom = y / g

  def < (that: Rational) = numer * that.denom < that.numer * denom
  def max(that: Rational) = if (this < that) that else this

  def + (that: Rational) = {
    new Rational(
      numer * that.denom + that.numer * denom,
      denom * that.denom
    )
  }

  def - (that: Rational) = {
    this + -that
  }

  def unary_- = new Rational(-numer, denom)
}
```

재밌는 점은 `neg` 를 **unary operator** `-` 로 만들기 위해 `unary_` 를 이용해서 `unary_-` 로 정의했다는 것이다.

참고로, return 값을 주기 위해서 `unary-_: Rational` 과 같이 정의하면 에러가 난다. `:` 가 포함된 identifier 로 인식하기 때문에 `unary-_ :` 로 스페이스를 꼭 주어야 한다.

#### Precedence Rules

그렇다면 `x * x + y` 와 같은 경우 `*` 가 먼저 계산되어야 하는데, 이런건 어떻게 해결할까? 우리가 만든건 정수 연산자가 아니라 함수인데.

이를 위해 스칼라는 다음과 같은 룰을 만들어 두었다.

> The precedence of an operator is determined by its first character. The following table lists the characters in increasing order of priority precedence

```scala
(all letters) // Alphanumeric
|
^
&
< >
= !
:
+ -
* / %
(all other special characters)
```

identifier 의 첫글자가 미리 정의된 테이블에 있다면 우선순위가 정해지는 룰이다.
