+++
date = "2016-06-25T01:21:52+09:00"
next = "../intro-to-haskell-9"
prev = "../intro-to-haskell-7"
title = "하스켈로 배우는 함수형 언어 8"
toc = true
weight = 108
aliases = [
    "/haskell-intro8"
]
+++

고차함수가 있는 다른언어와 비교했을 때 하스켈은 무슨 특징이 있을까요? 하스켈은 *expression* 을 평가하기 위해 디폴트로 *lazy evaluation* 을 사용한다는 점에서 다른 언어들과 다릅니다.

이번시간엔 *evaluation* 의 개념부터 시작해서, 다양한 종류의 *evaluation* 전략들을 살펴보겠습니다.

### Evaluation

- Avoid doing **unnecessay evaluation**
- Allows programs to be **more modular**
- Allows us to program with **infinite lists**

하스켈은 *lazy evaluation* 을 이용해 위에 나열한 것들을 제공합니다. *lazy evaluation* 을 이야기 하기 전에 먼저 *evaluation* 이 무엇인지 살펴봅시다.

> Basically, expressions are evaluated or reduced by successively applying definitions until no further simplification is possible

예를 들어서 `square n = n * n ` 이란 *definition* 이 있을때, *expression* `square(3 + 4)` 는 이렇게 두 가지 방식으로 평가될 수 있습니다.

```haskell
square (3 + 4)
square 7
7 * 7
49

-- bad
square (3 + 4)
(3 + 4) * (3 + 4)
7 * (3 + 4)
7 * 7
49
```

만약에 아래 버전처럼 `(3 + 4) * (3 + 4)` 로 평가된다면, 똑같은 계산을 두번이나 하게 될 겁니다. 더 심각한 문제는 *side effect* 가 발생한다면 값이 달라질 수도 있다는 것이지요!

아래 예제를 한번 봅시다. *evaluation* 전략에 따라 값이 달라지는 것을 보여줍니다.

```haskell
-- initially, n := 0

-- left first
n + (n := 1)
0 + (n := 1)
0 + 1
1

-- right first
n + (n := 1)
n + 1
1 + 1
2
```

> **FACT:** In Haskell, two diffrent (but terminating) ways of evaluating the same expression will always give the same final result.

다행히도 하스켈은 어떤 전략을 사용하든 *terminating expression* 에 대해서는 항상 같은 결과를 돌려줍니다. 

### Reduction Strategies

일반적으로 평가방법은 크게 두 가지로 나눌 수 있습니다. 어떤 *reducible subexpression (redex)* 를 선택하냐에 따라 

(1) **Innermost reduction:** An inner most redex is always reduced  
(2) **Outermost reduction:** An outermost redex is always reduced
	
```haskell
loop = tail loop

// innermost reduction
fst (1, loop)
fst (1, tail loop)
fst (1, tail (tail loop))
...

// outermost reduction
fst (1, lop)
1
```

위 결과를 보면 *innermost* 가 종료되지 않는 경우에도, *outermost* 는 결과를 돌려줄 수 있다는 사실을 알 수 있습니다. 

또한 어느 하나의 *reduction sequence* 라도 종료된다면 *outermost reduction* 도 종료됩니다. 같은 결과를 돌려주면서요. 원문을 첨부하면,

> For a given expression if there exists any reduction sequence that terminates, then outermost reduction **also** terminates, with the same result

*innermost* 에 비해 더 많은 경우에 종료되므로 *outermost* 가 좋다고 볼 수도 있겠습니다. 그러나, *outermost reduction* 은 좀 비효율적입니다.

```haskell
// innermost
square (3 + 4)
square 7
7 * 7
49

// outermost
square (3 + 4)
(3 + 4) * (3 + 4)
7 * (3 + 4)
7 * 7
49
```

따라서 하스켈에서는 *outermost* 에 *sharing* 을 더해 *lazy evalution* 이라 부르고 이 방법을 *evalution* 에 이용합니다.

```haskell
square (3 + 4) -- sharing, n = (3 + 4)
= n * n -- reduced shared expression `n` into 7
= 7 * 7
= 49
```

*innermost, outermost* 예제를 좀 더 살펴봅시다.

```haskell
mult :: (Int, Int) -> Int
mult (x, y) = x * y
```

이제 `mult(1 + 2, 3 + 4)` 를 *innermost* 로 평가한다고 한다면,

```haskell
mult(1 + 2, 3 + 4)
mult(3, 3 + 4) -- conventionally, we select left innermost
mult(3, 7)
3 * 7 -- apply outermost
24
```

*innermost* 는 *argument (인자)* 가 먼저 평가 되어야 하기 때문에, 인자가 *value* 인 경우 사용할 수 있습니다. 반대로 *outermost* 전략을 사용한다고 결정하려면 인자가 *name* 이어야 합니다. 

어떤 함수들의 경우는 *outermost* 를 사용함에도 먼저 인자가 평가되어야 합니다. 예를 들어 `*, +` 같은 *built-in operator* 는 무조건 인자가 먼저 평가되야 합니다. 이런 함수들을 *strict* 하다고 말 합니다.

좀 더 엄밀한 정의는

> A function f is said to be strict if, when applied to a nonterminating expression, it also fails to terminate.

`mult` 를 *curried function* 으로 재 작성해 봅시다.

```haskell
mult :: Int -> Int -> Int
mult x = \y -> x * y

-- evaluation
mult (1 + 2) (3 + 4)
mult 3 (3 + 4)
(\y -> 3 * y)(3 + 4)
(\y -> 3 * y)(7)
3 * 7
```

이제 인자가 한턴에 하나씩 계산됩니다. 이는 `mult 3 (3 + 4)` 에서 *left, innermost redex* 가 `mult 3` 이기 때문입니다. `mult (3, 3 + 4)` 에선 `3 + 4` 가 *left, innermost redex* 였지만요.

참고로 하스켈에서 *lambda expression* 내부의 *redex* 를 선택하는건 불가능합니다. 이는 람다도 함수이고, 함수 내부는 볼 수 없는 *black box* 이기 때문입니다.

> Note that in Haskell, the selection of redexes within lambda expressions
is prohibited. The rational for not “reducing under lambdas” is that functions are viewed as black boxes that we are not permitted to look inside.

일반적으로 *innermost* 전략을 *call by value*, *outermost* 전략을 *call by name* 이라 부릅니다.

### Infinite List

여기 `1` 의 무한한 나열을 표현하는 `ones` 에 대해 *expression* `head ones` 가 어떻게 평가되는지 *innermost* 와 *lazy evaluation* 의 두 가지 방법을 비교해 봅시다. 

```haskell
ones :: [Int]
ones = 1 : ones

-- innermost
head one
head (1 : one)
head (1 : 1 : one)
...

-- lazy evaluation
head one
head (1: ones)
1
```

*innermost* 의 경우에는 *evaluation* 이 끝나지 않습니다. 반면 *lazy evaluation* 은 식이 끝나면서 결과를 얻을 수 있죠.

> Using **lazy evaluation**, expressions are only evaluated as much as required to produce the final result

즉 필요한 만큼만 평가됩니다. 따라서 *lazy evaluation* 을 이용한 평가방법이 있으므로 `ones = 1 : ones` 처럼 무한할 **가능성이 있는** 데이터를 표현할 수 있습니다.

### Modular Programming

```haskell
take 5 ones
-- [1, 1, 1, 1, 1]
```

위의 예제에서 볼 수 있듯이 *lazy evaluation* 을 이용하면 *expression* 을 두 부분으로 나눕니다. 

- **Control Part:** `take 5`
- **Data:** `ones`

인자를 받아 주어진 숫자만큼 복사하는 `replicate` 함수도 만들어 볼까요?

```haskell
replicate' :: Int -> a -> [a]
replicate' 0 _ = []
replicate' n x = x : replicate' (n - 1) x
```

### Generate Primes

무한한 길이의 원소를 표현할 수 있다는 법을 배웠습니다. 이 방법을 이용해 존재하는 모든 소수의 집합을 표현하는 리스트를 만들어 볼까요? 

*Sieve of Eratosthenes (에라토스테네스의 체)* 란 방법을 사용하겠습니다. 알고리즘은 [여기](http://en.wikipedia.org/wiki/Sieve_of_Eratosthenes) 를 참조하세요.

```haskell
primes :: [Int]
primes = seive [2..]

seive :: [Int] -> [Int]
seive (p : xs) = p : [x | x <- xs, x `mod` p /= 0]

take 10 primes
-- [2,3,5,7,9,11,13,15,17,19]

takeWhile (<15) primes
-- [2, 3, 5, 7, 11, 13]
```

### Strict Application

하스켈에선 *lazy evaluation* 이 기본이지만, *strict* 버전으로 함수를 적용할 수 있는 방법도 제공합니다. `$!` 키워드를 이용하면 되는데요, `f $! x` 같은 경우 `f` 를 적용하기 전에 `x` 가 모두 평가되야 합니다.

더 엄밀히 말하면 *top-level of evaluation* 이 이루어지는데요, 인자 `x` 의 타입이 `Int` 나 `Bool` 같은 *basic type* 일 경우는 *complete evaluation* 이 이루어집니다.

반대로, `(Int, Bool)` 같은 복합타입이라면 *pair of expression* 이 얻어질 때 까지만 평가가 이루어집니다. 유사하게 타입이 리스트라면 `[]` 나 `a : b` 같은 컨싱이 얻어질때까지만 평가가 이루어집니다.

> More formally, an expression of the form `f $! x` is only a redex once evaluation of the argument x, using lay evaluaion as normal, has reached the point where it is known that the result is not an undefined value, at which point the expression can be reduced to the normal application `f x`

예를 들어 `square $! (1 + 2)` 의 경우

```haskell
square $! (1 + 2)
square $! 3
square 3
3 * 3
9 
```

다수개의 인자를 갖는 *curried function* 과 `$!` 가 쓰일 경우에는 다양한 형태가 될 수 있습니다.

```haskell
(f $! x) y    -- forces top-level evaluation of x
(f x) $! y    -- forces top-level evaluation of y
(f $! x) $! y -- forces top-level evaluation of x and y
```

하스켈에서 *strict application* 은 주로 프로그램의 *space performance* 을 개선하기 위해 사용됩니다. 예를 들어 다음과 같은 `sumWith` 함수가 있다고 합시다. 

```haskell
sumWith :: Int -> [Int] -> Int
sumWith v [] = v
sumWith v (x:xs) sumWith (v + x) xs
```

*lazy evaluation* 에서는

```haskell
sumWith 0 [1, 2, 3]
sumWith (0 + 1) [2, 3]
sumWith ((0 + 1) + 2) [3]
sumWith (((0 + 1) + 2) + 3) []
(((0 + 1) + 2) + 3)
...
...
6
```

계산 전에 `(((0 + 1) + 2) + 3)` 가 만들어 지는걸 볼 수 있습니다. `sumWith 0 [1.. 10000]` 같은 큰 수의 계산일 경우 공간이 좀 아까울 수 있지요.

따라서 `sumWith` 에 `$!` 를 이용하면

```haskell
sumWith v [] = v
sumWith v (x:xs) = (sumWith $! (v + x)) xs

sumWith 0 [1, 2, 3]
sumWtih $! (0 + 1) [2, 3]
sumWith $! 1 [2, 3]
sumWith 1 [2, 3]
...
```

`sumWith` 뿐만 아니라 고차함수인 `foldl` 등에도 적용해 볼 수 있습니다.

```haskell
foldl' :: (a -> b -> a) -> a -> [b] -> a
foldl' f v [] = v
foldl' f v (x:xs) ((foldl' f) $! (f v x)) xs
```

이러면, `sumWith` 를 `foldl' (+)` 로 정의할 수 있습니다.

## References

(1) **DelftX FP 101x**   
(2) *Programming in Haskell*  


