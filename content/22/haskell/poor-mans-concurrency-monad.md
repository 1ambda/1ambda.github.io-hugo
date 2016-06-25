+++
date = "2016-06-25T01:22:05+09:00"
prev = "../intro-to-haskell-9"
title = "Poor Man's Concurrency Monad"
toc = true
weight = 110
aliases = [
    "/a-poor-mans-concurrency-monad"
]
+++

*FP 101x* 의 최종 보스입니다. ~~Rose Tree 는 거들뿐~~ *Koen Claessen* 가 1999년에 발표한 [*Poor Man's Concurrenc Monad*](http://citeseerx.ist.psu.edu/viewdoc/summary?doi=10.1.1.39.8039) 를 배경으로 하는 과제인데, 언어에 *primitive* 추가 없이 *concurrency* 를 모델링 하는 방법을 보여줍니다. 

### Continuation

먼저 용어부터 정의하고 가면, *continuation* 은 실행 가능한 *computation* 입니다. 필요할 때 사용할려고 미뤄둔 계산인데, 이게 프로세스를 모델링 하기에 적당합니다. 왜냐하면 프로세스도 멈추었다가, 나중에 다시 실행을 해야 하니까요!

나중에 쓰려고 미뤄둔 계산, 즉 *continuation* 을 지속적으로 넘겨가면서 사용하는 방식을 *continuation passing style* 이라 부릅니다. *CPS* 로 작성된 함수는 리턴하는 법이 없습니다. 다만 자신의 계산을 *continuation* 으로 만들어 넘겨줄 뿐이지요.

코드를 먼저 보시지요. 피타고라스 계산을 하스켈에서 *CPS* 로 작성하는 방법입니다. 

```haskell
square :: Int -> Int
square x = x * x

add :: Int -> Int -> Int
add x y = x + y

square_cps :: Int -> (Int -> r) -> r
square_cps x = \cont -> cont (square x)

add_cps :: Int -> Int -> (Int -> r) -> r
add_cps x y = \cont -> cont (add x y)

pythagoras_cps :: Int -> Int -> (Int -> r) -> r
pythagoras_cps x y = \cont ->
  square_cps x $ \squared_x ->
  square_cps y $ \squared_y ->
  add_cps squared_x squared_y cont
  
> square_cps 3 print
-- "9"

> add_cps 3 4 print
-- "7"

> pythagoras_cps 3 4 print
-- "25"
```

위 예제에서는 `print` 가 나중에 쓸려고 모셔둔 계산, 즉 *continuation* 입니다. 이 타입 `(Int -> r) -> r` 을 잘 기억해 두세요.

### Process Modeling

프로세스를 모델링 하려면 상태와 작업 두 가지를 나타내야 합니다. 먼저 프로세스가 하는 작업에 대해서 모델링을 해 보겠습니다. 프로세스는 의 작업은 `Action` 이라 부르겠습니다. `Action` 은 `Atom` 이라 부르는 `IO` 연산일 수도 있고, 자식을 만드는 `Fork` 나, 프로세스를 멈추는 `Stop` 이 될 수 있습니다.

`Atom` 은 *side-effect* 를 만드는 *atomic* 연산이라 보면 됩니다.

```haskell
data Action = 
   = Atom (IO Action)
   | Fork Action Action
   | Stop
```

프로세스는 상태를 모델링 하기 위해 프로세스의 동작에 대해서 조금 논의해 봅시다. 프로세스는 자신의 작업이 있습니다. 우리는 `Action` 으로 표현했지요. 프로세스가 어떤 이유에서든지 중단된다면, 나중을 위해서 이 `Action` 을 기억해 둬야 합니다. 다시 작업을 해야하니까요!

아까 위에서 보았던 `(Int -> r) -> r` 기억 나시나요? *continuation* `Int - r` 을 이용해 결과 `r` 을 만들어 냈던 타입이지요. 이 타입을 잘 보면, *continuation* 이 공급될 때 *result `r`*  을 얻을 수 있습니다. 여기서 결과인 `r` 은 다른 프로세스에게 밀려 중단된 작업 `Action` 이라 보시면 되고, 공급되는 *continuation* 은 *CPU* 와 같은 리소스라 보시면 됩니다. (그렇게 생각하는 편이 ~~정신건강에~~ 좋습니다.)

그러면, 비슷하게, 이런 타입을 생각해 볼 수 있습니다.

```haskell
data Concurrent a = ((a -> Action) -> Action)
```

이 타입은 `a -> Action` *continuation* 을 받아, 결과 `Action` 을 돌려줍니다. 

그러면 프로세스의 **미뤄진 작업의 상태**를 표현하는 `Concurrent` 에 *continuation* 을 공급해 **미뤄진 작업** `Action` 을 얻어내는 `action` 이란 함수를 만들 수 있습니다.

```haskell
action :: Concurrent a -> Action
action (Concurrent concur) = concur (\a -> Stop)
```

또한 어떤 *continuation* 을 받던 무조건 멈추는 `Action` 을 돌려주는 `stop` 함수도 생각해 볼 수 있겠죠. 이건 **멈춰진 작업의 상태** 를 표현하는 `Concurrent` 라 보셔도 좋습니다.

```
stop :: Concurrent 
stop = Concurrent (\cont -> Stop)
```

이제 `IO` 를 `Concurrent` 로 표현하기 위해 `IO a -> Concurrent a` 로 변환해주는 `atom` 을 만들겁니다. 다시 말해서 이 함수는 **멈춰진 `IO` 연산** 을 돌려줘 하므로 `Concurrent` 내에 `Atom (IO Action)` 을 담아야 합니다. 

`cont a` 가 `Action` 이므로, `do` 내에서 `return (cont a)` 이면 `IO Action` 타입을 얻을 수 있겠죠? 쉽게 생각해서 *continuation* 인 `cont` 가 공급될 때 `IO` 를 수행한다 보면 되겠습니다.

```haskell
atom :: IO a -> Concurrent a
atom \io -> Concurrent $ \cont -> Atom $ do a <- io
                                            return (cont a)
```

이제 프로세스를 분할하는 `Fork` 작업을 생각해 봅시다. 타입만 보면 `Fork Action Aciton`  입니다. 즉 두개의 `Action` 을 `Concurrent` 내에 담아야 합니다.

```haskell
fork :: Concurrent a -> Concurrent ()
fork concur = Concurrent $ \cont -> Fork (action concur) (cont ())
```

보면, `action concur` 로 현재 미뤄진 작업에 대한 `Action` 을 추출하고, *continuation* 를 받아 `cont ()` 로 *continuation* 에 있는 다음 `Action` 을 뽑아냅니다. *continuation* 의 타입이 `a -> Action` 인거 기억 나시죠?

비슷하게, 두개의 미루어진 작업을 받아 `Fork` 로 만드는 `par` 함수도 만들어 봅시다.

```haskell
par :: Concurrent a -> Concurrent a -> Concurrent a
par (Concurrent a) (Concurrent b) = Concurrent $ \cont -> Fork (a cont) (b con))
```

이제 `Concurrent` 간 *composition* 을 위해 `>>=`, `return` 을 구현하면

```haskell
instance Monad Concurrent where
    -- g :: \a -> Concurrent b
    (Concurrent A) >>= g = 
      \contB -> A (\contA -> case g a of (Concurrent B) -> B contB  
```

직관적인 이해는, `>>=` 자체는 두 `Concurrent` 간 연결입니다. 서로 다른 타입 `a, b` 에 대해서 `Concurrent` 가 어떻게 연결되야 하는지 생각해 보면 됩니다. 

`Concurrent a` 의 `Action` 을 얻기 위한  *continuation* 은, 다음 작업을 의미하는데 이 *continuation* `a' -> Action` 에서의 `Action` 이 `Concurrent b` 의 `Action` 이라 보면 됩니다.

다시 말해서, `Concurrent a` 의 `Action` 의 다음 작업이 `Concurrent b` 의 `Action` 이란 뜻이지요. 

마지막으로 `Action` 을 라운드 로빈 방식으로 스케쥴링하는 `roundRobin` 함수와, 실제로 `Concurrent a` 을 이용해 `roundRobin` 함수를 이용하는 `run` 함수를 보면,

```haskell
roundRobin :: [Action] -> IO ()
roundRobin [] = return ()
roundRobin (Atom x:xs) = x >>= \ac -> roundRobin (xs ++ [ac])
roundRobin (Fork x y : xs) = roundRobin (xs ++ [x, y])
roundRobin (Stop : xs) = roundRobin xs

run :: Concurrent a -> IO ()
run x = roundRobin [action x]
```

몇개의 헬퍼 함수와 테스트 코드도 좀 보겠습니다.

```haskell
genRandom :: Int -> [Int]
genRandom 1337 = [1, 96, 36, 11, 42, 47, 9, 1, 62, 73]
genRandom 7331 = [17, 73, 92, 36, 22, 72, 19, 35, 6, 74]
genRandom 2600 = [83, 98, 35, 84, 44, 61, 54, 35, 83, 9]
genRandom 42   = [71, 71, 17, 14, 16, 91, 18, 71, 58, 75]

loop :: [Int] -> Concurrent ()
loop xs = mapM_ (atom . putStr . show) xs

ex0 :: Concurrent ()
ex0 = par (loop (genRandom 1337)) (loop (genRandom 2600) >> atom (putStrLn ""))

ex1 :: Concurrent ()
ex1 = do atom (putStr "Haskell")
         fork (loop $ genRandom 7331) 
         loop $ genRandom 42
         atom (putStrLn "")

myex0 = run $ (ho >> ho >> ho) >>
              (hi >> hi >> hi) >> atom (putStr "\n")
  where ho = atom (putStr "ho")
        hi = atom (putStr "hi")

myex1 = run $ fork (ho >> ho >> ho) >>
              (hi >> hi >> hi) >> atom (putStr "\n")
  where ho = atom (putStr "ho")
        hi = atom (putStr "hi")
        
myex2 = run $ fork (put3 "ba") >> fork (put3 "di") >>
        put3 "bu" >> atom (putStr "\n")
  where put3 = sequence . take 3 . repeat . atom . putStr
        
myex3 = run $ par (put3 "ba") (put3 "di" >> stop) >>
        atom (putStr "\n")
  where put3 = sequence . take 3 . repeat . atom . putStr
        
myex4 = run $ (par (put3 "ba") (put3 "di")) >>
        atom (putStr "\n")
  where put3 = sequence . take 3 . repeat . atom . putStr

myex5 :: Concurrent ()
myex5 = do fork (atom $ putStrLn "test")
           atom $ putStrLn "hello"

myex6 :: Concurrent ()
myex6 = do val <- par (atom $ return "hi") (atom $ return "hello")
           atom $ putStrLn val
```

## References

(1) **DelftX FP 101x**   
(2) *Programming in Haskell*  
(3) [A Poor Man's Concurrency Monad](http://citeseerx.ist.psu.edu/viewdoc/summary?doi=10.1.1.39.8039)
