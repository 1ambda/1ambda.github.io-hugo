+++
date = "2016-06-25T01:21:42+09:00"
next = "../intro-to-haskell-3"
prev = "../intro-to-haskell-1"
title = "하스켈로 배우는 함수형 언어 2"
toc = true
weight = 102
aliases = [
    "/haskell-intro2"
]
+++

이번시간엔 *list comprehension* 을 배웁니다. 수학에서는 집합의 원소를 이용해 새로운 집합을 만들 때 사용하는데요,

> In mathematics, the *comprehension* notion can be used to construct new sets from old sets.

비슷하게 하스켈에선 컬렉션에다 사용 할 수 있죠.

> In Haskell, a similar comprehension notion can be used to construct new lists from old lists

```haskell
[x^2 | x <- [1..3]] -- [1, 4, 9]
```

여기서 `x <- [1..5]` 같은 *expression* 을 **generator** 라 부릅니다. *comprehension* 은 한 개 이상의 *generator* 를 가질 수 있습니다.

```haskell
[(x, y) | x <- [1..3], y <- [4..5]]
-- [(1,4),(1,5),(2,4),(2,5),(3,4),(3,5)]
```

*generator* 의 순서를 바꿈으로써 생성되는 원소들의 순서도 바꿀 수 있습니다.

```haskell
[(x, y) | y <- [4..5], x <- [1..3]]
-- [(1,4),(2,4),(3,4),(1,5),(2,5),(3,5)]
```

보면 알겠지만 *multiple generators* 는 *nested loop* 와 비슷합니다. 뒤에 오는 *generator* 가 *inner-loop* 처럼 동작하죠.

그럼 `j = i + 1` 과 같은 변수도 *generator* 로 표현 가능할까요? 그럼요!

```haskell
[(x, y) | x <- [1..3], y <- [x + 1]]
-- [(1,2),(2,3),(3,4)]
```

앞에 오는 *generator* 의 변수를 뒤에 오는 *generator* 에서 사용 할 수 있습니다. *dependant generator* 라 부릅니다.

이제 *dependant generator* 를 이용해서 `concat` 함수를 만들어 봅시다.

```haskell
concat :: [[a]] -> [a]
concat xss = [x | xs <- xss, x <- xs]

concat [[1, 2, 3], [4, 5], [6]]
-- [1,2,3,4,5,6]
```

### Guards

*generator* 에서 변수를 걸러내기 위해 *guards* 를 사용할 수 있습니다.

```haskell
[x | x <- [1..10], even x]
-- [2, 4, 6, 8, 10]
```

약수를 골라내는 `factors` 함수도 만들어 볼 수 있겠죠? 그리고 소수를 판별하는 `prime` 도 같이 작성합시다.

```haskell
factors :: Int -> [Int]
factors n = [x | x <- [1..n], n `mod` x == 0]

factors 17 -- [1,17]
factors 15 -- [1,3,5,15]

prime :: Int -> Bool
prime n = factors n == [1, n]
```

소수를 찾아주는 `primes` 도 만들어볼까요?

```haskell
primes :: Int -> [Int]
primes n = [x | x <- [2..n], prime x]

primes 40
-- [2,3,5,7,11,13,17,19,23,29,31,37]
```

### zip

`zip` 함수는 두 개의 리스트를 받아 하나의 리스트를 만듭니다. 

```haskell
-- zip :: [a] -> [b] -> [(a, b)]

zip [1, 2, 3]  ['a', 'b', 'c', 'd']
[(1,'a'),(2,'b'),(3,'c')]
```

`pairs` 와 같은 함수도 만들어 볼 수 있겠죠?

```haskell
pairs :: [a] -> [(a, a)]
pairs xs = zip xs (tail xs)

pairs [1, 2, 3, 4]
-- [(1,2),(2,3),(3,4)]
```

`pairs` 함수를 이용하면 하나의 리스트에 있는 한 원소와 그 다음 원소의 *pair* 를 구할 수 있으므로 리스트가 정렬되었는지를 검사하는 `sorted` 함수에 사용할 수 있습니다.

```haskell
sorted :: Ord a => [a] -> Bool
sorted =
  and [x <= y | (x, y) <- paris xs]
  
sorted [1, 2, 3, 4] --True
sorted [1, 2, 5, 3, 4] --False
```

하스켈 리스트은 배열과는 달라서 인덱스가 없습니다. 리스트에서 주어진 값과 같은 값을 가지는 원소들의 리스트를 구하는 `positions` 함수를 `zip` 을 이용해 만들어봅시다.

```haskell
positions :: Eq a => a -> [a] => [Int]
positions x xs = 
  [i | (x', i) <- zip xs [0..n], x == x']
  where n = (length xs) - 1
  
positions 0 [0, 1, 0, 1, 1, 1, 1, 0]
-- [0,2,7]  
```

### String comprehensions

하스켈에선 *스트링 (문자열)* 이 *캐릭터* 의 리스트라는 걸 지난시간에 이야기 했습니다. 따라서 리스트를 인자로 받는 *polymorphic* 함수에 스트링을 적용할 수 있습니다.

```haskell
zip "abc" [1, 2, 3] -- [('a',1),('b',2),('c',3)]
*Main> take 3 "asdasd" -- "asd"
length "adasd" -- 5
```

그런 이유에서 *list comprehension* 을 이용해 스트링을 조작할 수 있습니다.

```haskell
import Data.Char

lowers :: String -> Int
lowers xs = length [x | x <- xs, isLower x]
```

### The Caesar cipher

*Caesar cipher (시저 암호)* 는 간단한 치환 암호입니다. 알파벳을 특정 자리수 만큼 밀어 인코딩된 새로운 문자열을 만들어 내지요.

간단하게 구현하기 위해 모든 문자가 소문자라 가정하겠습니다. 알파벳은 26개이니 `a` 를 `0` 에 매핑하지요.

```haskell
import Data.Char

let2int :: Char -> Int
let2int c = ord c - ord 'a'

int2let :: Int -> Char
int2let n = chr(n + ord 'a')
```

`ord` 함수는 캐릭터를 받아 아스키 숫자로 변환하고 `chr` 함수는 그 반대의 역할을 합니다. 위 코드에서 `let2int` 는 캐릭터를 받아 `0` 부터 `25` 사이의 숫자로 변환합니다. 물론 `c` 가 소문자임을 가정합니다. `int2let` 은 반대의 역할을 하고요.

이제 주어진 소문자 알파벳을 `n` 번 만큼 이동시키는 `shift` 함수를 만들어 봅시다.

```haskell
shift :: Int -> Char -> Char
shift n c | isLower c = int2let((let2int c + n) `mod` 26)
          | otherwise = c
          
shift (-1) 'a' -- 'z'
shift (3) 'a' -- 'd'
```

이제 *list comprehension* 을 이용해 `encode` 함수를 만들어 봅시다.

```haskell
encode :: Int -> String -> String
encode n cs = [shift n c | c <- cs]

encode 1 "abc"
-- "bcd"
encode 3 "haskell is fun"
-- "kdvnhoo lv ixq"
encode (-3) "kdvnhoo lv ixq"
-- "haskell is fun"
```

### Cracking the Ciper

시저 암호를 깨는 방법은 다음과 같습니다.

(1) 대량의 텍스트를 분석해 각 알파벳이 문장속에서 나올 확률을 가지고 있는 *frequency table* 을 준비합니다.  
(2) 인코딩된 암호를 0 부터 25 까지 시프팅 해 가면서 우리가 준비한 *frequency table* 과 같은 비율을 가지고 있는지 검사합니다.

물론 이 방법은 텍스트가 너무 짧거나, 아니면 우리가 가지고 있는 *frequency table* 과 다른 분포를 가지고 있는 텍스트를 복호화 하지 못합니다. 

일단 한번 해 봅시다.

```haskell
table :: [Float]
table = [8.2, 1.5, 2.8, 4.3, 12.7, 2.2, 2.0, 6.1, 7.0, 0.2, 0.8, 4.0, 2.4,
        6.7, 7.5, 1.9, 0.1, 6.0, 6.3, 9.1, 2.8, 1.0, 2.4, 0.2, 2.0, 0.1]

count :: Eq a => a -> [a] -> Int
count x xs = length [x' | x' <- xs, x == x']

lowers :: String -> Int
lowers cs = length [c | c <- cs, isLower c]

percent :: Int -> Int -> Float
percent n m = (fromIntegral n / fromIntegral m) * 100

freqs :: String -> [Float]
freqs xs = [percent (count x xs) n | x <- ['a'..'z']]
           where n = lowers xs
```

`freqs` 함수는 주어진 문자열에 대해 *frequency table* 을 돌려줍니다. `count` 와 `percent`, `lowers` 함수를 이용해서 만들었습니다. 

이제 우리가 가지고 있는 `table` (`es`, *expected*) 과 인코딩된 텍스트를 시프팅 해서 얻은 `os` (*observed*) 테이블과의 차를 구하는 `chisqr` 함수를 만들겠습니다. 이 차이가 가장 작으면 `os` 가 우리가 가진 테이블에 가장 근접한 *frequency table* 을 가지는 테이블이라는 뜻이죠. 

카이 제곱 분포를 이용할 건데, 공식은 다음과 같습니다.

![http://www.maritzresearch.com/maritzstats/HelpFiles/Formula_ChiSquareTest.htm](http://www.maritzresearch.com/maritzstats/HelpFiles/images/ChiSquareFormula-ChiSquareValue.bmp)


```haskell
chisqr :: [Float] -> [Float] -> Float
chisqr os es = sum [(o- e)^2 / e | (o, e) <- zip os es]
```

하스켈 참 쉽죠? 이제 본래 인코딩 된 텍스트를 왼쪽으로 시프팅 하는 함수를 만들겁니다. `rotate` 라고 부릅시다. `take`, `drop` 을 이용하면

```haskell
rotate :: Int -> [a] -> [a]
rotate n xs = drop n xs ++ take n xs
```

이제 인코딩된 텍스트에 대해 `[0..25]` 번 `rotate` 해 가며 `chisqr` 을 호출한 결과 중 가장 작은 값을 가지는 `factor` 를 찾아 `encode` 에  `-factor` 로 입력하면 암호가 깨집니다. 이런 일을 하는 함수를 `crack` 이라 부르고 작성해 봅시다.

```haskell
positions :: Eq a => a -> [a] -> [Int]
positions x xs = [i | (x', i) <- zip xs [0..n], x' == x]
  where n = (length xs) - 1

crack :: String -> String
crack xs = encode (-factor) xs
  where
    factor = head (positions (minimum chiTab) chiTab)
    chiTab = [chisqr (rotate n table') table | n <- [0..25]]
    table' = freqs xs
```

이제 이 함수를 사용해 봅시다.

```haskell
crack "kdvnhoo lv ixq"
-- "haskell is fun"
crack "vscd mywzboroxcsyxs kbo esopev"
-- "list comprehensioni are uieful"
crack "vscd mywzboroxcsyxs kbo ecopev"
-- "list comprehensioni are useful"

crack (encode 3 "haskell")
-- "piasmtt"
crack (encode 3 "boxing wizards jump quickly")
-- "wjsdib rduvmyn ephk lpdxfgt"
```

아래 두개의 예제는 복호화의 실패한 경우를 보여줍니다. 인코딩된 텍스트의 테이블이 우리가 가진 테이블과 많이 다르기 때문이지요.

### Exercise

두개의 *generator* 를 가진 하나의 *comprehension* 은 하나의 *generator* 를 가진 두개 이상의 *comprehension* 으로 작성할 수 있습니다. 다음의 두 라인은 동일합니다.

```haskell
[(x, y) | x <- [1, 2, 3], y <- [4, 5, 6]]

concat [[(x, y) | y <- [4, 5, 6]] | x <- [1, 2, 3]] 
```

### References

(1) **DelftX FP 101x** in *edx*  
(2) Chapter 5, Programming in haskell  
(3) [*Caesar cipher*](http://ko.wikipedia.org/wiki/%EC%B9%B4%EC%9D%B4%EC%82%AC%EB%A5%B4_%EC%95%94%ED%98%B8)  
(4) [http://www.maritzresearch.com](http://www.maritzresearch.com/maritzstats/HelpFiles/Formula_ChiSquareTest.htm)

