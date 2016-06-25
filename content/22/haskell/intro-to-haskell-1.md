+++
date = "2016-06-25T01:21:41+09:00"
next = "../intro-to-haskell-2"
prev = "../"
title = "하스켈로 배우는 함수형 언어 1"
toc = true
weight = 101
aliases = [
    "/haskell-intro1"
]
+++

먼저 이 글은 **edx** 의 *FP101.x (Introduction to Functional Programming)* 수업을 기반으로 작성되었음을 알려드립니다.

시작에 앞서서, 하스켈을 설치하려면 [Haskell Platform](http://www.haskell.org/platform/linux.html) 을 설치하신 후 터미널에서 `ghci` 를 입력하면 됩니다. 하스켈 플랫폼은 하스켈 구현체로 [Glasgow Haskell Compiler, *GHC*](http://www.haskell.org/ghc/) 를 포함하고 있습니다. `ghci` 를 입력하면 하스켈 인터프리터를 사용할 수 있습니다.

깔기 귀찮으시다면 [https://www.fpcomplete.com/](https://www.fpcomplete.com/) 여기서 웹으로 코드를 작성하고 컴파일 할 수 있습니다.

### Basics

먼저 `sum [1..10]` 을 인터프리터에 입력하면 1부터 10 까지의 합, `55` 을 돌려줍니다. `sum` 은 리스트의 원소를 모두 합한 값을 구하는 함수고 `[1..10]` 은 1-10 을 포함하는 리스트를 생성합니다.

이제 인터프리터를 잠깐 끈 후에 *quick-sort* 함수를 파일에 작성한 뒤 로드해 봅시다. 아참, `ghci` 를 종료하려면 `ctrl + d` 를 입력합니다.

```haskell
-- quicksort.hs

f [] = []
f (x:xs) = f ys ++ [x] ++ zs
           where
             ys = [a | a <- xs, a <= x]
             zs = [b | b <- xs, b > x]
```

수학 표기법 같은데, 놀랍게도 잘 동작합니다. 다시 `ghci` 실행하고 `:load quicksort.hs` 를 입력합니다. 그러면 우리가 작성한 `f` 함수가 인터프리터에 로드 됩니다. *퀵-소트* 를 사용할 수 있다는 뜻이죠.

```
ghci > f [3, 2, 9, 5, 4]
[2, 3, 4, 5, 9]
```

제대로 정렬 되는걸 확인할 수 있습니다. 매번 인터프리터를 종료했다가 다시 실행시키긴 귀찮으니 파일을 다시 수정했을때 `:reload` 명령어를 사용해서 다시 로드합니다. 이외에도 다른 명령어를 보고싶다면 `ghci` 에 `:help` 를 입력해보세요.

이제 대충 어떻게 돌아가는지 알았으니 기본적인 함수를 좀 알아볼까요? 다른 함수형 언어처럼 하스켈도 `List` 에 대한 기본적인 연산들이 있습니다.

```haskell
head [2, 3, 4] -- 2
tail [2, 3, 4] -- [3, 4]

head [2] -- []
tail [2] -- []

init [2, 3, 4] -- [2, 3]
last [2, 3, 4] -- 4

take 2 [2, 3, 4] -- [2, 3]
drop 2 [2, 3, 4] -- [4]
```

리스트 원소에 접근하려면 `!!` 를 이용합니다. 다른 언어와 마찬가지로 `0` 부터 인덱스가 시작합니다. 

```haskell
[1, 2, 3, 4] !! 1 -- 2
[1, 2, 3, 4] !! 2 -- 3
[1, 2, 3, 4, 5] !! 2 -- 3
[1, 2, 3, 4, 5] !! 0 -- 1
```

하스켈의 리스트는 자료구조의 그것처럼, `n` 번째 원소에 접근하려면 `O(n)` 의 비용이 듭니다. 그런 이유에서 리스트의 원소를 구하는 `length` 도 `O(n)` 의 비용이 듭니다. 자바의 배열과는 좀 다르죠? 

이제 리스트에 대한 몇 가지 연산을 더 알아봅시다.

```haskell
product [1, 2, 3, 4, 5] -- 120
[1, 2, 3] ++ [4, 5] -- [1, 2, 3, 4, 5]
reverse [1, 2, 3, 4] -- [4, 3, 2, 1]
```

`++` 는 *append* 연산입니다. 앞에 온 리스트에, 뒤에 온 리스트를 붙여줍니다. 


### Function Application

이제 함수를 사용하는 법을 좀 알아볼까요? 일반적으로 `f` 라는 함수에 파라미터 `a` 를 적용하려면 `f(a)` 이렇게 쓸 텐데요. 하스켈에선 `f a` 라고 작성해도 됩니다. 왜냐하면 함수의 우선순위가 다른 연산자들보다 높거든요

> Function application is assumed to have higer priority than all other operators

이런 이유에서 `f a + b` 는 `f(a) + b` 입니다. `f(a + b)` 를 의도했다면 `f(a + b)` 라고 쓰셔야 합니다.

```haskell
f x -- f(x)
f x y -- f(x, y)
f (g x) -- f(g(x))
f x (g y) -- f(x g(y))
f x * g y -- f(x) * g(y)
```

이제 좀 스크립트에 다른 함수를 작성해 볼까요?

```haskell
-- test.hs

double x = x + x
quadruple x = double (double x)
```

`double` 은 *function* 이고 왼쪽에 오는 `x` 는 *argument* 입니다. `=` 뒤에 오는건 함수의 *body* 로 함수가 무슨일을 하는지 기록한 것이죠.

이제 `:load test.hs` 후 `take (double 2) [1, 2, 3, 4, 5, 6]`을 입력하면 `[1, 2, 3, 4]]` 를 돌려줄겁니다. 다른 함수를 좀 더 만들어 봅시다.

```haskell
-- test.hs

factorial n = product [1..n]
avg ns = sum ns `div` length ns
```

`factorial` 은 별로 놀랍지 않죠? 그런데 평균을 구하는 `avg` 는 문법이 좀 신기합니다. `ns` 는 리스트입니다. `sum` 을 적용해야 하니까요. 그리고 끝에 `s` 를 붙이는건 관례인데, 일반적으로 변수 끝에 `s` 를 붙이면 리스트란 뜻입니다. `ns` 란 이름은 *정수 리스트* 라고 알려주는 것이지요. 

`sum ns` 는 `ns` 의 합을 구하고, `length ns` 는 `ns` 의 길이를 구합니다. `div` 함수를 *back-quote* 감싼 것은 함수가 두개의 파라미터를 가질때 가운데 위치할 수 있도록 해 줍니다. 

그래서 *back quote* 로 감싼  함수를 만나면 하스켈 컴파일러는 **"왼쪽과 오른쪽에 있는 것들을 파라미터로 가지는 함수"** 임을 깨닫죠. 이런 문법은 `3 / 5` 처럼 함수를 *operator (연산자)* 쓰듯이 사용할 수 있게 해줍니다. 따라서 위에 나온 코드의 본래 모양은 이렇습니다.

```haskell
avg ns = div (sum ns) (length ns)
```

> *x `f` y* is just **syntatic sugar** for *`f` x y*

아참 그리고 함수의 이름은 소문자로 시작해야 합니다. `Avg` 는 함수 이름으로 쓸 수 없어요. 그리고 `nss` 처럼 변수 이름이 `s` 두개로 끝나면, 그건 리스트의 리스트라는 것을 뜻합니다. 물론 관례죠. 필수는 아닙니다. 그러나 우리가 작성하는건 남들이 볼 수도 있으니 따라주는 편이 좋습니다.

그리고 변수의 정의에 대해 *layout* 을 지키면 `{}` 블럭을 안 사용해도 됩니다. 예를 들어 다음의 두 코드는 똑같은 코드입니다.

```haskell
a = b + c
    where
      b = 1
      c = 2
d = a * 2

a = b + c
    where
      { b = 1
      c = 2 }
d = a * 2
```

### Boolean, Tuple

하스켈은 대 소문자를 구분하기 때문에 *Boolean false* 를 얻을려면 `False` 를 입력해야 합니다.

```haskell
False || True -- True
```

괄호로 감싼 두개의 값들을 *tuple* 이라 부릅니다. *list* 는 같은 종류만 담을 수 있지만 *tuple* 은 달라도 상관 없지요. `fst`, `snd` 는 *tuple* 에서 첫 번째 와 두 번째 값을 돌려주는 함수입니다.

```haskell
fst (1, "Hello") -- 1
snd (1, "Hello") -- "Hello"
fst (snd (1, (2, 3))) -- 2
```

아참, `ghci` 에서 `:t` 을 입력하면 *expression* 의 타입을 알 수 있습니다. `:t False`, `:t length`, `:t head`, `:t (1, "Hello")` 등을 입력해보세요.

### Types

다른 언어처럼 하스켈도 타입이 있습니다. `False` 와 `True` 은 `Bool` 타입이죠. `ghci` 에서 `:t False` 를 입력하면 `False :: Bool` 이란 결과를 돌려줍니다. `e :: t` 는 `e` 가 `t` 타입을 가지고 있다는 뜻입니다.

모든 *expression (식)* 은 타입을 가지고 있습니다. 하스켈 컴파일러는 컴파일 타임에 *type inference (타입 추론)* 을 통해서 타입을 찾아냅니다. 만약 함수의 타입과 불일치하는 인자가 파라미터로 넘겨진다면 타입 에러가 발생하는 것이지요.

하스켈에서 문자열을 나타내는 타입인 `String` 은 사실 문자 하나를 의미하는 `Char` 의 리스트입니다. 

그리고 *fixed-precision integer* 를 의미하는 `Int` 이외에도  파이썬 처럼 *arbitrary-precision integer* 를 위한 `Integer` 타입이 있습니다. 실수는 `Float` 타입을 이용해 나타냅니다.

앞서 언급했듯이 *Tuple* 과 달리 *List* 는 같은 타입만 가질 수 있습니다. 그래서  `[False, False, True]` 의 경우  `[Bool]` 타입입니다. 문자열은 문자의 리스트이므로 `String` 타입은 `[Char]` 입니다. 스트링의 리스트는 `[String]` 이고 더 정확히는 `[[Char]]` 입니다.

그리고 아까는 *Tuple* 을 두개의 원소만 가질 수 있다고 했지만 사실은 두개 이상의 원소를 가질 수 있습니다 `n` 개의 원소를 가진 *tuple* 을 *n-tuple* 이라 부릅니다. 예를 들어 이런 *tuple* 도 있을 수 있습니다.

```haskell
(False, 'a', True) :: (Bool, Char, Bool)
(True, ['a', 'b']) :: (Bool, [Char])
```

### Function Type

함수는 한 타입의 값을 다른 타입의 값으로 매핑합니다.

> A *function* is a mapping from values of one type to values of another types

예를 들어 `:t not` 을 `ghci` 에서 입력하면 아래처럼 출력됩니다. 
`not` 함수는 `Bool` 타입을 받아 `Bool` 타입을 돌려준다는 뜻입니다.

```haskell
not :: Bool -> Bool
```

이제 파일에 타입까지 같이 명시해서 함수를 작성해 봅시다.

```haskell
add :: (Int, Int) -> Int
add (x, y) = x + y

zeroto :: Int -> [Int]
zeroto n :: [0..n]
```

함수가 만약 *tuple* 을 인자로 받으면 괄호가 필요합니다.

### Curried Function

여기서 잠깐 *Currying* 을 설명할 텐데요, 하스켈에서 빠질 수 없는 부분이니 이해가 어렵다면 다른 글을 찾아서라도 이해를 하시는 편이 좋습니다.

*Currying* 의 기본 개념은 이렇습니다. 함수가 `(Int, Int) -> Int` 타입이라면 **계산을 끝내기 위해 두개의 정수를 받아** 하나의 `Int` 를 돌려줄겁니다. 

다시 한번 반복하자면, 계산을 끝내기 위해서는 두개의 정수가 필요합니다. 그렇다면 하나의 정수만 받고, 나머지는 계산이 필요한 시점에 받으면 안될까요?

```haskell
Int -> (Int -> Int)
```

바꿔 말해서, 인자 하나를 받아서 계산을 일정부분 해 내고, 나머지는 **인자를 하나 더 받아 계산을 마무리 해 돌려주는 함수** 를 리턴해도 괜찮지 않을까요? 나머지 계산은 그 함수에서 할거니까, 결국 인자 2개로 계산을 해 내는 건 똑같으니까요.

이게 *Currying* 의 기본 개념입니다. 다시 말해서 다음의 두 타입은 같은 일을 한다는 거죠.

```haskell
(Int, Int) -> (Int)

-- same as
Int -> (Int -> Int)
```

이게 언제 유용할까요? 인자를 부분적으로 채운 함수가 필요할 때를 생각 해 봅시다. 

```haskell
add :: Int -> (Int -> Int)
add x y = (x + y)

add3 = add 3
add3 4 -- 7
```

3을 미리 더 해놓은 `add3` 이란걸 만들어서 써먹었습니다. 이렇게 *Currying* 을 활용할 수 있죠. `:t` 를 입력하면 `Int -> Int` 를 확인할 수 있습니다. `:take 5` 도 *curried function* 이죠.

**결국 *Currying* 은 `n` 개의 인자를 가진 함수를 `n-1` 개의 부분적으로 계산된 함수로 바꿀 때** 쓸 수 있습니다. 단지 `n` 개의 파라미터만 가진 **하나의 함수** 보다 **여러 개의 부분적인 함수**를 만들어 놓으면 재활용 할 수 있으므로 훨씬 좋죠. 인자에 함수를 넘길 수 있는 함수형 언어에서는 함수를 재활용 할 수 있다는 건 정말 좋은 일입니다..

하스켈에서는 *Currying* 을 아주 중요하게 여깁니다. 얼마나 중요하게 생각하는지, 함수 타입에 괄호가 없으면 자동으로 *curried function* 이 되게끔 해 놓았지요. 아래의 두 함수는 같습니다.

```haskell
add :: Int -> (Int -> Int)

-- same as

add :: Int -> Int -> Int
```

다시 말해 함수의 타입은 *right-associative (우측 결합)* 입니다. 반대로 함수의 호출은 *left-associative (좌측 결합)* 이죠.

```haskell
add 3 4

-- same as

(add 3) 4
```

그래서 그냥 함수 타입에 괄호를 안쓰시고 인자의 갯수가 `n` 개라면 이 함수를 호출 할 때 `n-1` 개의 *curried function* 을 호출하신다고 생각하면 됩니다.

### Generics

*polymorphic* 은 *of many form (다양한 형태의)* 라는 뜻입니다. 프로그래밍에선 주로 다양한 타입을 이야기 할때 사용한 용어인데요, `length` 같은 경우 타입을 보면 이렇습니다.

```haskell
length :: [a] -> Int
```

`a` 라는 타입은 없으므로 여기선 **아무 타입이나** 와도 된다는 뜻입니다. 다시 말해 `length` 는 *polymorphic function* 입니다. 다양한 타입을 취할 수 있는 함수죠. `length` 는 사실 무슨 타입이 오든 관심이 없습니다. 갯수를 세는데만 정신이 팔려있는 함수지요.

여기서 `a`는 *type variable* 이라 부릅니다. `a` 가 아니라 `b`, `c` 등 소문자이기만 하면 됩니다.

참고로 프로그래밍에서 *polymorphism* 은 *generics* 와 *sub-typing* 의 두 가지 개념을 모두 포함하는 용어입니다. *generics* 는 *type parameterization* 을, *sub-typing* 은 *type-hierarchy* 를 의미하지요.

이제 몇 가지 *polymorphic function* 을 살펴봅시다.

```haskell
*Main> :t fst
fst :: (a, b) -> a
*Main> :t head
head :: [a] -> a
*Main> :t take
take :: Int -> [a] -> [a]
*Main> :t zip
zip :: [a] -> [b] -> [(a, b)]

*Main> [1, 2, 3] `zip` ['a', 'b', 'c']
[(1,'a'),(2,'b'),(3,'c')]
```

*polymorphic function* 이 특정 타입에 대해 제한 되었는 것을 하스켈에서는 *overloaded* 되었다고 부릅니다. 다시 말해 특정타입에만 해당 함수를 사용할 수 있다는 거죠. 이게 *C++* 에서의 *오버로딩* 과는 좀 의미가 다릅니다. 

> It really means that you're restricting the types of the parameters.

숫자에 대해서만 사용할 수 있는 `sum` 을 좀 살펴볼까요?

```haskell
sum :: Num => [a] -> a
```

`sum` 은 정수나 실수에 대해 모두 사용할 수 있지만 문자열에 대해서는 불가능합니다. 이게 바로 *overloaded* 된 것이죠. 그 부분이 바로 `Num => ` 이 의미하는 바입니다.

하스켈은 *overloading* 에 사용할 수 있는 다양한 타입 클래스가 있습니다. `Num`, `Eq`, `Ord` 등이 그것이죠. `ghci` 에서 `==` 를 타입 검사 해볼까요? 아참, `==` 은 `:t (==)` 로 검사해야합니다.

```haskell
*Main> :t (<) -- Ordered types
(<) :: Ord a => a -> a -> Bool
*Main> :t (==) -- Equality types
(==) :: Eq a => a -> a -> Bool
*Main> :t (+) -- Numeric types
(+) :: Num a => a -> a -> a
```

예를 들어 다음의 `palindrome` 함수 타입을 잘 보세요.

```haskell
palindrome xs = reverse xs == xs

:t palindrome -- Eq [a] => [a] -> Bool
```

다시 말해서 `[a]` 가 비교가 가능한 `Eq` 타입이어야 한다는 뜻입니다.

### Basic Classes

`Eq` 는 해당 클래스가 비교할 수 있음을, `Ord` 는 순서가 있음을 나타냅니다. 기본 타입인 `Bool`, `Char`, `String`, `Int`, `Integer`, `Float` 와 `Tuple`, `List` 는 모두 `Ord` 와 `Eq` 클래스의 인스턴스입니다. 참고로 하스켈에서 *Not equal* 은 `/=` 입니다.

`(>)`, `(>=)`, `(<)`, `(<=)`, `max`, `min` 등이 `Ord` 클래스에 대해 적용할 수 있는 함수입니다. 그리고 `String`, `Tuple`, `List` 는 *사전 편찬 순서 (lexicographically)* 를 기준으로 비교됩니다. 

### Showable Type

다른 언어의 `toString` 이 그렇듯이, `Show` 클래스의 인스턴스들은 `show` 함수를 이용하면 `String` 형태로 출력됩니다.

```haskell
:t show -- Show a => a -> String
show False -- "False"
show [1, 2, 3] -- "[1, 2, 3]"
```

### Readable Type

`Read` 타입 인스턴스들은 `read` 함수를 이용해 `String` 으로부터 얻어질 수 있습니다. 

```haskell
:t read -- Read a => String -> a

read "False" :: Bool
read "[1, 2, 3]" :: [Int] -- [1, 2, 3]
```

### Integral, Fractional, Rational

단순히 숫자 클래스로 `Num` 만 있는건 아니고 정수나, 분수, 유리수 등을 나타내는 다양한 클래스가 있고 여기에 적용할 수 있는 함수가 있고, 불가능한 함수가 있습니다. 예를 들어 `mod` 나 `div` 는 `Integral` 에만 적용 가능합니다. `(/)` 는 `Fractional` 에만 적용 가능합니다.

### Guarded Equations

`if` 대신에 더 좀 편하게 사용할 수 있는 *guarded equation* 이 있습니다. **다른 계산 없이** `if` 만 바로 사용한다면 다음과 같이 바꿔 쓸 수 있죠.

```haskell

abs n = if n >= 0 then n else -n
signum n = if n > 0 then 1 else
		     if n < 0 then -1 else 0

-- same as

abs n | n >= 0 = n
      | otherwise = -n 
       
signum n | n > 0 = 1
         | n < 0 = -1
         | otherwise = 0 
```

### Pattern Matching

*Scala* 나 *C#* 과는 다르게 하스켈은 패턴매칭을 바로 이용합니다. `case` 나 `switch` 없이요! 예를 들어

```haskell
not :: Bool -> Bool
not False = True
not True = False
```

패턴 매칭은 위에서 부터 순서대로 적용됩니다. 예를 들어 `not False = False` 가 코드에 제일 위에 있다면 항상 `False` 만 돌려줄거에요. 우울한 일이 될겁니다.

이제 2개의 피 연산자를 갖는 연산자 `&&` 를 패턴매칭을 이용해 만들어 볼까요? 좌 우에 피 연산자를 가지는 함수를 만들기 위해서는 `(*)` 처럼 괄호로 감싸면 됩니다. 

```haskell
(&&) :: Bool -> Bool -> Bool

True && True = True
False && False = False 
False && True = False
True && False = False
```

더 간단히는 `_` 를 이용할 수 있죠. *wild card pattern* 이라고 부릅니다. `_` 는 *아무거나 (Anything)* 이라고 생각하시면 됩니다. 스칼라를 배우셨다면 익숙하신 기호죠! $

```haskell
(&&) :: Bool -> Bool -> Bool
True && True = True
_ && _ = False
```

그러나 위 정의보다 더 효율적인 `(&&)` 를 만드는 방법이 있습니다. 위 패턴매칭에서 둘 다 `True` 일 경우 우측 `True` 는 사실 평가할 필요가 없습니다. 매칭 시켜봤자 어차피 `True` 니까요.

```haskell
True && b = b
False && _ = False
```

이 경우 패턴매칭에서 `b` 는 변수입니다. 왼쪽 피연산자가 `False` 일 경우도 우측 피 연산자 `_` 를 *evaluation (평가)* 하지 않고 바로 `False` 를 줍니다. 다시 말해 더 효율적이란 뜻이지요.

아참 그리고 패턴매칭에서 동일한 변수를 두번 사용할수 없습니다. `b && b = b` 는 에러를 내뿜습니다. 주의하세요

### List Patterns

함수형 언어에서는 리스트가 주된 자료구조 이기 때문에 패턴매칭에 리스트를 사용할 수 있다면 프로그래머의 삶을 편하게 만들 수 있습니다. 

`:` 는 *Cons* 라 부르는 연산자인데요, `[1, 2, 3, 4]` 는 `1 : 2 : 3 : 4 : []` 더 정확히는 `1 : (2 : ( 3: (4 : []))))` 입니다. 즉 원소와 뒤에오는 리스트를 연결해주는 것이죠. 요로코롬 패턴매칭에 활용할 수 있습니다.

```haskell
head (x : _) = x
tail (_ : xs) = xs
```

`head` 나 `tail` 모두 패턴이 *비어있지 않은 리스트* 이므로 빈 리스트가 인자로 올때는 에러가 납니다. 그리고, 함수 적용의 우선순위가 다른 연산자들 보다 높기 때문에 `head x:_` 는 `(head x) : _` 가 되어 에러가 납니다. 괄호를 잊지마세요.

### Lambda Expression

*lambda (람다)* 는 이제 유명합니다. *Java 8* 에도 추가되었으니 인기있는 대부분의 언어는 람다를 가지고 있죠. 익명함수라고 불리기도 하는데, 하스켈에선 `\` 를 이용해 람다를 표시합니다. 이를테면, `\x -> x + 1` 처럼요. 람다가 왜 유용하나면

(1) *currying* 을 이용해 정의된 함수를 좀더 의미있게 표현할 수 있습니다.

> Lambda expressions can be used to give a formal meaning to functions defined using *currying*

이를테면 다음의 두 함수는 동일합니다. 아래 람다를 이용해 정의한 식이 함수를 리턴하는 함수라는 의미가 더 강하죠.

```haskell
add x y = x + y

-- same as

add = \x -> (\y -> x + y)

const :: a -> b -> a
const x _ = x

-- same as

const :: a -> (b -> a)
const x 
```

(2) 람다를 이용하면 컴퓨터 과학의 가장 큰 난제중 하나인 *naming (명명)* 을 피할 수 있습니다!

```haskell
odds n = map f [0..n]
          where
	    f x = x `mod` 2 /= 0

odds1 n = map (\x -> x `mod` 2 /= 0) [0..n]
```

### Sections

*in-fix operator* 는 괄호를 더해 맨 앞으로 끌어올 수 있습니다.

> An operator written between its two arguments can be converted int a curried function written before its two arguments by using parentheses.

```haskell
1 + 2 -- 3

-- same as

(+) 1 2 -- 3
```

이렇게 연산자를 앞으로 옮길 수 있게 되면 피 연산자 하나와 괄호를 엮을 수 있습니다. `(x+)` 또는 `(+x)` 처럼요. 이렇게 피연산와 연산자가 괄호로 묶인 것을 *section* 이라 부릅니다. `x + y` 를 이용하면 두개의 섹션을 만들 수 있습니다. `(x+)` 와 `(+y)` 입니다.

*section (섹션)* 은 언제 유용할까요? 우리는 섹션을 이용하면 람다를 만들어 주는 대신에 좀 더 의미있는 *partiall applied function (부분 함수)* 를 만들 수 있습니다.

이를테면 `double (k (/2))` 처럼요

이외에도 섹션은 연산자의 타입을 기술하거나 `(&&) ::`, 연산자가 다른 함수의 인자로 들어갈때 필요합니다.

### References

(1) *Programming Haskell*
