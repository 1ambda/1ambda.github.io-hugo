+++
date = "2016-06-25T01:21:50+09:00"
next = "../intro-to-haskell-8"
prev = "../intro-to-haskell-6"
title = "하스켈로 배우는 함수형 언어 7"
toc = true
weight = 107
aliases = [
    "/haskell-intro7"
]
+++

*the countdown problem* 은 프랑스 퀴즈 프로그램에서 유래한 문제입니다. 주어진 양수를 단 한번씩만 이용하여 특정 숫자를 만드는 문제입니다. 사용가능한 연산자는 `+, *, -, /` 입니다.

예를 들어 `(25 - 10) * (50 + 1) = 765` 입니다. 

사람이 풀기엔 *search space* 가 좀 넓어서 답을 한번에 찾기 어렵지만, 컴퓨터는 무한한 인내심을 가지고 있기 때문에 풀기에 적합한 문제입니다.

### Evaluating Expressions

이번시간엔 *bottom-up* 으로 접근해 볼까요? 먼저 연산자타입과 이를 적용하는 함수 `apply` 를 만들어보면

```haskell
data Op = Add | Sub | Mul | Div

apply :: Op -> Int -> Int -> Int
apply Add x y = x + y
apply Sub x y = x - y
apply Mul x y = x * y
apply Div x y = x `div` y
```

그리고 우리가 가진건 양수이기 때문에, 연산의 결과가 양수인지 체크하기 위한 `valid` 함수를 만들어 보겠습니다. 

```haskell
valid :: Op -> Int -> Int -> Bool
valid Add _ _ = True
valid Sub x y = x > y
valid Mul _ _ = True
valid Div x y = x `mod` y == 0
```

이제 수식을 나타내는 `Expr` 타입과 평가하기 위한 `eval` 함수를 만들면

```haskell
data Expr = Val Int = App Op Expr Expr

eval :: Expr -> [Int]
eval (Val n) = [n | n > 0]
eval (App o l r) = [apply o x y | x <- eval l,
                                  y <- eval r,
                                  valid o x y]
```

여기선 연산이 실패했음을 나타내기 위해 `[]` 를 사용했습니다. `Maybe` 타입 대신 리스트를 쓸 때의 장점은, *list comprehension* 을 이용할 수 있다는 점이지요!

### Formalizing The Problem

우리가 풀어야할 문제는 가능한 모든 조합을 탐색해야하기 때문에 다양한 조합을 만들기 위한 `choices` 함수를 만들겠습니다.

```haskell
-- subs [1, 2] -> [[], [1], [2], [1, 2]]
subs :: [a] -> [[a]]
subs [] = [[]]
subs (x:xs) = yss ++ map (x:) yss
  where yss = subs xs

-- interleave 1 [2, 3] -> [[1, 2, 3], [2, 1, 3], [2, 3, 1]]
interleave :: a -> [a] -> [[a]]
interleave x [] = [[x]]
interleave x (y:ys) = (x:y:ys) : map (y:) (interleave x ys)

-- perm [1, 2, 3] = [[1, 2, 3], [1, 3, 2], [2, 3, 1], ..]
perm :: [a] -> [[a]]
perm [] = [[]]
perm (x:xs) = concat (map (interleave x) (perm xs))

-- choices [1, 2] -> [[], [1], [2], [1, 2], [2, 1]]
choices :: [a] -> [[a]]
choices xs = concat (map (perm) (subs xs))
```

여기서 `subs` 함수는 순서를 고려하지 않은 부분집합을, `perm` 는 순열을 돌려줍니다. `choices` 는 이 두 함수를 조합하여 부분집합의 순열리스트를 돌려줍니다.

이제 입력한 수식이 정답인지 알려주는 `solution` 함수를 볼까요? 입력한 수식의 결과가 주어진 수 `n` 과 같아야 하고, 수식에 있는 숫자가 주어진 숫자들의 나열 `ns` 와 같아야 합니다.

```haskell
values :: Expr -> [Int]
values (Val n) = [n]
values (App _ l r) = values l ++ values r

solution :: Expr -> [Int] -> Int -> Bool
solution e ns n = elem (values e) (choices ns) && eval e == [n]
```

### Brute Force

브루트 포스 방법으로 풀려면, 사용가능한 수들을 받아, 가능한 모든 수식을 돌려주면 됩니다.

```haskell
-- brute force
split :: [a] -> [([a], [a])]
split xs = [splitAt i xs | i <- [1..(n-1)]]
  where n = length xs

exprs :: [Int] -> [Expr]
exprs [] = []
exprs [n] = [Val n]
exprs ns = [e | (ls, rs) <- split ns
              , l <- exprs ls
              , r <- exprs rs
              , e <- combine l r]

combine :: Expr -> Expr -> [Expr]
combine l r = [App o l r | o <- [Add, Sub, Mul, Div]]

-- brute force solutions
bSolutions :: [Int] -> Int -> [Expr]
bSolutions ns n = [e | ns' <- choices ns
                     , e <- exprs ns'
                     , eval e == [n]]
```

아주아주아주아주 느립니다. 제 컴퓨터에서는 2분이 지나도 답이 안나오네요.

```haskell
> length (Bolutions [1, 3, 7, 10, 25, 50] 765)
```


### Fast version

어느부분을 고쳐야 더 빨라질까요? 한가지 개선할 부분은, `valid` 가 너무 늦게 호출된다는 점입니다. 우리가 어마어마한 식을 만드는 반면, 답이 780개란 사실은 대부분의 식이 값보다는 형태에 의해 필터링 된다는 뜻입니다. 따라서 `valid` 를 좀 더 땡길 수 있다면 계산이 훨씬 빨라질겁니다.

```haskell
eval :: Expr -> [Int]
eval (Val n) = [n | n > 0]
eval (App o l r) = [apply o x y | x <- eval l,
                                  y <- eval r,
                                  valid o x y]

exprs :: [Int] -> [Expr]
exprs [] = []
exprs [n] = [Val n]
exprs ns = [e | (ls, rs) <- split ns
              , l <- exprs ls
              , r <- exprs rs
              , e <- combine l r]

bSolutions :: [Int] -> Int -> [Expr]
bSolutions ns n = [e | ns' <- choices ns
                     , e <- exprs ns'
                     , eval e == [n]]


```

이 부분을 좀 고쳐보겠습니다. 

```haskell
results :: [Int] -> [Result]
results [] = []
results [n] = [(Val n, n) | n > 0]
results ns = [res | (ls, rs) <- split ns
                  , lx <- results ls
                  , ry <- results rs
                  , res <- combine' lx ry]

combine' :: Result -> Result -> [Result]
combine' (l,x) (r, y) =
  [(App o l r, apply o x y) | o <- [Add, Sub, Mul, Div]
                            , valid o x y]

fastSolutions :: [Int] -> Int -> [Expr]
fastSolutions ns n = [e | ns' <- choices ns
                       , (e, m) <- results ns'
                       , m == n]
```

값을 평가하기 전에 먼저 `valid` 를 호출하고 계산된 값을 튜플에 저장해 놓았다가 나중에 비교합니다.


```haskell
> length (Bolutions [1, 3, 7, 10, 25, 50] 765)
-- 780
```

더 개선할 수 있을까요? 음.. 생각해보니 `x * y = y * x` 이기도 하고 `x * 1` 은 `x` 이기도 하네요. 이런것들을 좀 줄일수 있을겁니다. `valid` 함수를 고쳐보도록 하지요.

```haskell
valid :: Op -> Int -> Int -> Bool
valid Add _ _ = True
valid Sub x y = x > y
valid Mul _ _ = True
valid Div x y = x `mod` y == 0

-- modified
valid :: Op -> Int -> Int -> Bool
valid Add x y = x <= y 
valid Sub x y = x > y
valid Mul x y = x <= y && x /= 1 && y /= 1
valid Div x y = x `mod` y == 0 && y /= 1
```

`x <= y` 로 만들어 중복을 제거하고 `x /= 1` 을 이용해 1을 곱한 수식을 제거했습니다. 결과가 정말 빠르게 나옵니다. 

책에서 말하기를 브루트 포스 방법은 44초, 그 다음버전은 4초, 마지막 버전은 0.44 초 만에 계산이 끝난다고 합니다. 연산 시간이 어마어마하게 줄어들었죠?

```haskell
> length (Bolutions [1, 3, 7, 10, 25, 50] 765)
-- 49
```

### References

(1) **DelftX FP 101x**   
(2) *Programming in Haskell*  
