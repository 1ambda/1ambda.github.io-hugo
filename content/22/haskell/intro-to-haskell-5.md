+++
date = "2016-06-25T01:21:46+09:00"
next = "../intro-to-haskell-6"
prev = "../intro-to-haskell-4"
title = "하스켈로 배우는 함수형 언어 5"
toc = true
weight = 105
aliases = [
    "/haskell-intro5"
]
+++

키보드를 읽거나 화면에 무엇인가 쓰는 *intertactive program* 은 *side-effect* 를 만듭니다. 그런데, 하스켈은 *side-effect* 가 없지요. 그럼 입출력이 불가능한 것일까요? 

당연히 그렇지 않습니다. **IO 모나드** 를 사용할겁니다.

*pure expression* 부분과 *side-effect* 를 만들어내는 *impure action* 을 구분하여 하스켈에서 입출력을 할 수 있습니다.

> Interactive program can be written in Haskell using types to distinguish pure expressions from impure actions that may involve side effects

예를 들어 `IO a` 는 `a` 타입을 리턴하는 *action* 입니다.

몇 가지 예를 보면, `IO Char` 은 캐릭터를 리턴하는 액션입니다. `IO ()` 는 *unit* 을 돌려주는데 이건 절차형 언어에서의 *void* 와 같다고 보면 됩니다. 다시 말해서 `IO ()` 는 다른 것엔 아무것도 관심 없고 입출력에만 관심이 있다는 뜻이지요.

지난 시간에 언급 했듯이 *IO 모나드* 는 사실 *State 모나드* 입니다.

```haskell
State -> (a, State)
```

스크린이나, 키보드 버퍼등 다양한 State 를 변화시켜 가면서 `a` 타입의 값을 리턴할 수 있죠. 위에서 본 `IO ()` 는 *purely side-effecting action* 입니다.

### Basic Actions

`getChar` 는 키보드로부터 글자를 하나 읽어 캐릭터를 리턴합니다. 다른 언어에서는 `() -> Char` 처럼 정의되었겠죠?

```haskell
getChar :: IO Char
```

다른 *action* 도 좀 살펴볼까요?

```haskell
puChar :: Char -> IO ()
return :: a -> IO a
```

### Sequencing

*action* 들을 `do` 로 조합할 수 있습니다.

```haskell
a :: IO (Char, Char)
a = do x <- getChar
    getChar
    y <- getChar
    return (x, y)
    
getLine :: IO String
getLine = do x <- getChar
             if x == '\n' then
               return []
             else
               do xs <- getLine
                  return (x:xs)
```

몇 가지 더 볼까요?

```haskell
putStr :: String -> IO ()
putStr [] = return ()
putStr (x:xs) = do putChar x
                   putStr xs
                   
putStrLn :: String -> IO ()
putStrLn xs = do putStr xs
                 putChar '\n'
```

모나드의 산을 넘고 넘어야 IO 의 간결함이 이해가 되니, 아이러니 하죠? 본래 입출력은 정말 기초적인 부분인데 말이지요.

참고로 *list comprehension* 을 이용하면 `putStr` 은 이렇게 정의할 수 있습니다.

```haskell
seqn :: [IO a] -> IO ()
seqn [] = return ()
seqn (x:xs) = do x
                 seqn xs

putStr xs = seqn [putChar x | x <- xs]
```

조금 더 블럭을 쌓아봅시다. 문자열을 키보드로 부터 입력받아 화면에 그 길이를 띄워주는 함수를 작성해 봅시다.

```haskell
strlen :: IO ()
strlen = do putStr "Enter a string: "
            xs <- getLine
            putStr "The string has "
            putStr (show (length xs))
            putStrLn " characters"
            
> strlen
-- Enter a string: Hello World!
-- the string has 12 characters
```

`strlen` 은 `IO ()` 타입이니까, 아무것도 돌려주지 않습니다. 입출력에만 관심이 있지요.

### Hangman

이제까지 배운것을 응용해서 자그마한 행맨 게임을 만들어 봅시다. *top down* 방식으로 접근할 겁니다.

```haskell
hangman :: IO ()
hangman = do putStrLn "Think of a word :"
             word <- sgetLine
             putStrLn "Try to guess it:"
             guess word
```

여기서 `sgetLine` 은 키보드로부터 문자를 입력받아 `-` 를 화면에 출력합니다.

```haskell
sgetLine :: IO String
sgetLine = do x <- getCh
              if x == '\n'
                then do putChar x
                        return []
                else do putChar '-'
                        xs <- sgetLine
                        return (x:xs)
```

`getCh` 는 문자열을 키보드로 부터 읽지만 화면에 출력하진 않지요.

```haskell
import System.IO

getCh :: IO Char
getCh = do hSetEcho stdin False
           c <- getChar
           hSetCho stdin True
           return c
           
```

여기서 잘 보면 `c <- getChar` 이 할당(`=`)처럼 보일텐데, 사실은 그렇지 않습니다. 우린 어떠한 *mutable* 도 변수도 사용하고 있지 않습니다. 비록 우리가 작성한 코드가 절차형 언어처럼 보일지라도요!

이제 마지막 퍼즐인 `guess` 함수를 작성해 볼까요?

```haskell
guess :: String -> IO ()
guess word = do putStr "> "
                xs <- getLine
                if xs == word
                  then putStrLn "You got it!"
                  else do putStrLn (diff word xs)
                          guess word

diff :: String -> String -> String
diff xs ys = [if elem x ys then x else '-' | x <- xs]
```

`diff` 를 잠깐 실행해 보면

```haskell
> diff "haskell" "pascal"
-- "-as--ll"
```

### Calculator

시작 전에 몇 가지 보조 함수를 정의하면,

```haskell
getCh :: IO Char
getCh =  do hSetEcho stdin False
            c <- getChar
            hSetEcho stdin True
            return c

beep :: IO ()
beep = putStr "\BEL"

cls :: IO ()
cls = putStr "\ESC[2J"

type Pos = (Int, Int)

goto :: Pos -> IO ()
goto (x, y) = putStr ("\ESC["  ++ show y ++ ";" ++ show x ++ "H")

writeAt :: Pos -> String -> IO ()
writeAt p xs = do goto p
                  putStr xs
```

콘솔 창에서 문자의 위치는 좌표 `(Int, Int)` 에 의해 결정됩니다. `goto` 는 그 위치로 커서를 옮기고 `writeAt` 는 해당 좌표에 입력받은 문자열을 출력합니다.

여기에 [지난번](http://1ambda.github.io/haskell-intro4/)에 만들었던 파서가 `-`, `/` 도 처리할 수 있게 조금 업그레이드 하면

```haskell
int :: Parser Int
int =  do char '-'
          n <- nat
          return (-n)
        +++ nat

natural :: Parser Int
natural =  token nat

integer :: Parser Int
integer =  token int

expr :: Parser Int
expr = do t <- term
          do symbol "+"
             e <- expr
             return (t + e)
           +++ do symbol "-"
                  e <- expr
                  return (t - e)
           +++ return t

term :: Parser Int
term = do f <- factor
          do symbol "*"
             t <- term
             return (f * t)
           +++ do symbol "/"
                  t <- term
                  return (f `div` t)
           +++ return f

factor :: Parser Int
factor = do symbol "("
            e <- expr
            symbol ")"
            return e
          +++ natural
```

이제 간단한 계산기를 문자열로 나타내 보면

```haskell
box :: [String]
box =  ["+---------------+",
       "|               |",
       "+---+---+---+---+",
       "| q | c | d | = |",
       "+---+---+---+---+",
       "| 1 | 2 | 3 | + |",
       "+---+---+---+---+",
       "| 4 | 5 | 6 | - |",
       "+---+---+---+---+",
       "| 7 | 8 | 9 | * |",
       "+---+---+---+---+",
       "| 0 | ( | ) | / |",
       "+---+---+---+---+"]
```

`q, c, d, =` 는 *quit*, *clear*, *delete* *evaluation* 를 의미합니다. 나머지 버튼은 식을 입력하는데 사용하지요. 이제 박스를화면에 그려주는 `showbox` 함수를 작성합시다.

```haskell
seqn :: [IO a] -> IO ()
seqn [] = return ()
seqn (a:as) = do a
			     seqn as
                 
buttons :: [Char]
buttons = standard ++ extra
          where
            standard = "qcd=123+456-789*0()/"
            extra = "QCD \ESC\BS\DEL\n"

showbox :: IO ()
showbox = 
  seqn [writeAt (1, y) line | (y, line) <- zip [1..13] box]
```

`buttons` 에서 `extra` 는 좀 더 유연한 버튼 인터페이스를 위해 사용합니다. 무슨 말인고 하니 `q` 뿐만 아니라 `Q` 를 눌러도 계산기가 종료되게끔요. 

이제 수식을 표현하는 부분을 출력해줄 `display` 함수를 만듭시다. 입력받은 문자열을, 뒤에서부터 13개만 짤라서 `(3, 2)` 위치에 출력해줍니다.

```haskell
display :: String -> IO ()
display xs = do writeAt (3, 2) "             "
                writeAt (3, 2) (reverse (take 13 (reverse xs)))
```

이제 사용자로부터 문자를 입력받아 화면에 출력해주는 로직을 구현한 `calc` 함수를 보면

```haskell

calc :: String -> IO ()
calc xs = do display xs
             c <- getCh
             if elem c buttons
               then process c xs
               else do beep
                       calc xs

process :: Char -> String -> IO ()
process c xs
  | elem c "qQ\ESC" = quit
  | elem c "dD\BS\DEL" = delete xs
  | elem c "=\n" = eval xs
  | elem c "cC" = clear
  | otherwise = press c xs
```

`calc` 에서는 현재 수식창에 입력된 데이터 `xs` 와, 사용자로부터 받은 `c` 를 이용해 작업을 합니다. `c` 가 만약 `buttons` 내부에 없다면 다시 `calc xs` 를 호출해서 새로운 입력을 받습니다.

만약 `c` 가 `buttons` 내에 있는 문자들 중 하나라면 `process c xs` 를 호출하는데, 여기서는 버튼의 종류에 따라 다른 `IO ()` 를 돌려줍니다.

```haskell
quit :: IO ()
quit = goto (1, 14)

delete :: String -> IO ()
delete "" = calc ""
delete xs = calc (init xs)

eval :: String -> IO ()
eval xs = case parse expr xs of
           [(n, "")] -> calc (show n)
           _ -> do beep
                   calc xs

clear :: IO ()
clear = calc ""

press :: Char -> String -> IO ()
press c xs = calc (xs ++ [c])
```

(1) `quit` 는 다시 `calc` 호출 없이 현재 커서를 14번째 라인으로 이동해 계산기를 종료합니다.   
(2) `delete` 는 현재 `xs` 에서 마지막 문자를 제거한 `init xs` 를 `calc` 에 넘겨줌으로써 수식 입력창에서 마지막 문자를 지웁니다.  
(3) `eval` 는 `parse expr xs` 의 결과로 올바른 계산 값을 얻으면 `calc` 에 그 숫자를 문자열로 변환한 결과를 넘겨주어 계산값을 표시합니다. (`show n`) 아니라면, 계산이 안되므로 비프음을 뿜고 다시 `calc xs` 를 호출해 새로운 입력을 기다립니다.  
(4) `clear` 는 수식 입력창에 있는 값을 `""` 를 돌려줌으로써 비웁니다.  
(5) `press` 는 현재 수식 입력창에 있는 데이터 `xs` 에 `c` 를 이어 붙입니다.  

잘 보시면 현재 가지고 있는 데이터는 `xs` 로 표시되고, 이외의 `IO ()` 를 조합해 가며 화면의 상태(*State*) 를 변화시킵니다. 이 과정에서 **화면을 변화시키는 부분과, 데이터 `xs` 가 변하는 부분이 서로 분리** 되어 있습니다.

마지막으로 계산기를 실행시키는 함수 `run` 을 만들겠습니다.

```haskell
run :: IO ()
run = do cls
         showbox
         clear
```

### Game of Life

~~인생게임은 아닙니다~~ 세포의 생존게임이라 생각하면 이해하기 쉽습니다. `n * m` 보드에서 각 칸마다 세포가 위치할 수 있습니다.

> 1. a living cell survives if it has precisely two or three neighbouring squares that contain living cells, and dies (becomes empty) otherwise.

> 2. an empty square gives birth to a living cell if it has precisely neighbours that contain living cells, and remains empty otherwise.

각 칸마다 균등한 기회를 주기 위해 모서리에 있는 칸 또한 8개의 이웃한 칸을 가졌다고 합시다. *torus (3차원의 도넛모양 )* 을 생각하심 됩니다.

![](http://upload.wikimedia.org/wikipedia/commons/thumb/c/c6/Simple_Torus.svg/310px-Simple_Torus.svg.png)
<p align="center">(http://commons.wikimedia.org/wiki/File:Simple_Torus.svg)</p>

초기값에 따라 턴을 반복하면서 다양한 종류의 결과물이 나옵니다. 그 중에서 초기값이 몇번의 턴을 지나면서 지속적으로 대각선으로 움직이는 패턴을 *glider* 라 부릅니다.

![](https://camo.githubusercontent.com/f865db6a304d36aa7fef6c060729a2d635cd5c14/687474703a2f2f7777772d726f68616e2e736473752e6564752f7e72636172726574652f7465616368696e672f4d2d3539365f706174742f696d616765732f676c696465722e676966)
<p align="center">(https://gist.github.com/boggle/10390842)</p>

이제 *row* 를 `x`, *column* 을 `y` 로 해서 1 부터 시작하는 `5 x 5` 의 *glider* 보드를 만들면

```haskell
width :: Int
width = 5

height :: Int
height = 5

type Board = [Pos]

glider :: Board
glider = [(4,2),(2,3),(4,3),(3,4),(4,4)]

showCells :: Board -> IO ()
showCells b = seqn [writeAt p "O" | p <- b]

isAlive :: Board -> Pos -> Bool
isAlive b p = elem p b

isEmpty :: Board -> Pos -> Bool
isEmpty = not (isAlive b p)
```

여기에 해당 칸의 세포가 살았는지 죽었는지 검사하는 `isAlive`, `isEmpty` 와 보드를 출력하는 `showCells` 함수도 만들었습니다.

이제 어떤 `(x, y)` 를 입력 받아 그 주변 8개의 이웃 세포 좌표를 돌려주는 함수를 만들면

```haskell
neighbs :: Pos -> [Pos]
neighbs (x,y) =  map wrap [(x-1,y-1), (x,y-1),
                           (x+1,y-1), (x-1,y),
                           (x+1,y)  , (x-1,y+1),
                           (x,y+1)  , (x+1,y+1)] 

wrap :: Pos -> Pos
wrap (x,y) =  (((x-1) `mod` width) + 1, ((y-1) `mod` height + 1))
```

`wrap` 은 `mod` 연산을 이용해서, 판의 범위를 벗어난 이웃 세포의 좌표를 판 내에 있는 이웃으로 만들어 돌려줍니다. 예를 들어 

```haskell
> wrap (0, 1)
-- (5,1)
```

이제 살아있는 이웃 세포의 개수를 돌려주는 `liveNeighbs` 와, 살아있는 세포들(인접한 살아있는 세포가 2, 3개인) 좌표를 돌려주는 `survivors` 함수를 만듭시다.

```haskell
liveNeighbs :: Board -> Pos -> Int
liveNeighbs b = length . filter (isAlive b) . neighbs

survivors :: Board -> [Pos]
survivors b = [p | p <- b, elem (liveNeighbs b p) [2, 3]]
```

그리고 죽은 세포에 대해 인접한 살아있는 세포가 3개일 때만 살아있는 세포로 변경하는 `births` 함수를 만들면

```haskell
births :: Board
births :: Board -> [Pos]
births b = [p | p <- rmdups (concat (map neighbs b)),
            isEmpty b p,
            liveNeighbs b p == 3]

rmdups :: Eq a => [a] -> [a]
rmdups [] = []
rmdups (x:xs) = x : filter (/= x) xs
```

중복을 제거하기 위해 `rmdups` 함수를 만들어서 사용했습니다.

이렇게 되면, 다음 턴에서의 *board* 는 `survivors` 와 `births` 의 원소들 이므로

```haskell
nextGen :: Board -> Board
nextGen b = survivors b ++ births b
```

이제 화면 출력을 위한 몇 가지 함수를 더 만들면

```haskell
life :: Board -> IO ()
life b = do cls
            showCells b
            wait 5000
            life (nextGen b)

wait :: Int -> IO ()
wait n = seqn [return () | _ <- [1..n]]
```

### References

(1) **DelftX FP 101x**   
(2) *Programming in Haskell*  
(3) [gist.github.com/boggle](https://gist.github.com/boggle/10390842)
