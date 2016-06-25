+++
date = "2016-06-25T01:21:49+09:00"
next = "../intro-to-haskell-7"
prev = "../intro-to-haskell-5"
title = "하스켈로 배우는 함수형 언어 6"
toc = true
weight = 106
aliases = [
    "/haskell-intro6"
]
+++

이번시간엔 어떻게 *type* 과 *class* 를 정의하는지 배울겁니다. 이렇게 *commonality* 를  추출해서 *type* 과 *class* 로 만듦으로써 작업의 양을 줄일 수 있습니다. 이 과정을 추상화라 부르기도 합니다.

마지막엔 이제까지 배운바를 적용해 봅시다. 항상 참인 명제를 검사하는 **tautology checker** 와 평가 시점을 조절하는 **abstract machine** 을 만들어 보겠습니다.

### Type Declarations

하스켈에선 존재하는 타입을 이용해서 새로운 타입을 만들 수 있습니다.

```haskell
type String = [Char]
```

지난시간에 2차원 좌표계를 구현할 때 만들었던 `Pos` 타입 기억 나시죠?

```haskell
type Pos = (Int, Int)

origin :: Pos
origin = (0, 0)

left :: Pos -> Pos
left (x, y) = (x-1, y)
```

*type* 은 함수와 마찬가지로 다양한 타입을 사용할 수 있습니다.

```haskell
type Pair a = (a, a)

mult :: Pair Int -> Int
mult (a, b) = a * b

copy :: Int -> Pair Int
copy a = (a, a)
```

여러개의 타입도 사용할 수 있습니다.

```haskell
type Assoc k v = [(k, v)]

find :: Eq k => k -> Assoc k v -> v
find k xs = head [v | (k', v) <- xs, k == k']

> find 2 [(1, 'a'), (2, 'c'), (3, 'f')]
-- 'c'
```

그리고 *nested (중첩)* 될 수 있습니다.

```haskell
type Trans = Pos -> Pos

left :: Trans
left (x, y) = (x-1, y)
```

하지만 *recursive* 로 정의될 수는 없습니다. 왜냐하면 *type* 이 단지 *synonym* 이기 때문입니다.

```haskell
-- doesn't work
type Tree = (Int, [Tree

-- ghci

Cycle in type synonym declarations:
  lecture9.hs:22:1-25: type Tree = (Int, [Tree])
Failed, modules loaded: none.
```

그러나 하스켈에선 재귀적으로 타입을 정의할 수 있는 방법이 있긴 있습니다! 다만 *nominal type* 을 이용해야 합니다.(`data` 키워드를 사용합니다.) 많은 언어들이 이와 비슷한 제약조건을 가지고 있습니다.

*object-oriented language* 에서는 전형적으로 *nominal type system* 을 사용합니다. 이는 *"두 타입이 같은지"*, *"한 타입이 다른 타입의 서브타입인지"* 검사하기 쉽기 때문입니다. 반면 *purely sructural type system* 에서는 이게 조금 어려워집니다. (참고로 *nominal vs structure* 은, *dynamic static* 과는 다른 문제입니다.)

### Data Declarations

기존타입과 관련없는 새로운 타입을 만들려면 `data` 키워드를 사용하면 됩니다.

```haskell
data Bool = False | True	
```

이제 `Bool` 은 새로운 *type* 이고, 여기에 `False, True` 의 *value* 를 사용할 수 있습니다.

여기서 `False`, `True` 를 *type* `Bool` 을 위한 *constructor* 라 부릅니다. *type constructor* 의 이름은 반드시 대문자로 시작해야합니다.

새로운 타입을 조금 더 만들어 봅시다.

```haskell
data Answer = Yes | No | Unknown

answers :: [Answer]
answers = [Yes, No, Unknown]

flip :: Answer -> Answer
flip Yes = No
flip No = Yes
flip Unknown = Unknown
```

좌표의 움직임을 추상화한 타입 `Move` 도 만들어 봅시다.

```haskell
data Move = Left | Right | Up | Down

move :: Move -> Pos -> Pos
move Up (x, y) = (x, y-1)
move Left (x, y) = (x-1, y)
move Down (x, y) = (x, y+1)
move Right (x, y) = (x+1, y)

moves :: [Move] -> Pos -> Pos
moves [] p = p
moves (m:ms) p = moves ms (move m p)

> move Left (1, 1)
-- (0,1)

> moves [Left, Right, Up, Down, Left] (0, 0)
--(-1,0)
```

*data declaration* 내에 있는 *constructor* 는 파라미터를 가질 수 있습니다.

```haskell
data Shape = Circle Float
           | Rect Float Float

square :: Float -> Shape
square n = Rect n n

area :: Shape -> Float
area (Circle r) = pi * r^2
area (Rect x y) = x * y
```

여기서 *constructor* 를 함수라 볼 수도 있습니다.

```haskell
Circle :: Float -> Shape
Rect :: Float Float -> Shape
```

*constructor* 뿐만 아니라 *data declaration* 그 자체도 파라미터를 가질 수 있습니다.

```haskell
data Maybe a = Nothing | Just a

safediv :: Int -> Int -> Maybe Int
safediv _ 0 = Nothing
safediv x y = Just (x `div` y)

safehead :: [a] -> Maybe a
safehead [] = Nothing
safehead (x:xs) = Just x
```

### Recursive Types

재귀적인 타입의 예를 한번 볼까요?

```haskell
data Nat = Zero | Succ Nat
```

여기서 `Zero :: Nat`, `Succ :: Nat -> Nat` 라 보면 됩니다. 따라서 다음처럼 확장이 가능하지요.

```haskell
Zero -- 0
Succ Zero -- 1
Succ (Succ Zero) -- 2
```

보시면 알겠지만, 우리는 단 한 줄로 자연수를 표현하는 데이터 타입 `Nat` 를 만들었습니다. 숫자와 `Nat` 타입을 변환하는 함수를 만들어 봅시다.

```haskell
data Nat = Zero | Succ Nat

nat2int :: Nat -> Int
nat2int Zero = 0
nat2int (Succ nat) = 1 + nat2int nat

int2nat :: Int -> Nat
int2nat 0 = Zero
int2nat n = Succ (int2nat (n-1))

> nat2int (int2nat 10)
-- 10
```

재귀를 이용하면 `Nat` 간 덧셈을 위에서 만든 변환함수 없이도 만들수 있습니다.

```haskell
add :: Nat -> Nat -> Nat
add Zero n = n
add (Succ n) s = Succ (add n s)

> nat2int (add (int2nat 2) (int2nat 3))
-- 5
```

### List

임의의 타입을 갖는 리스트를 나타내는 `List` 타입을 만들어 보죠.

```haskell
data List a = Nil | Cons a (List a)

len :: List a -> Int
len Nil = 0
len (Cons h t) = 1 + len t

>  len (Cons 4 (Cons 3 Nil))
-- 2

> len Nil
-- 0
```

### Arithmetic Expressions

기본적인 `+, *` 과 정수와 연산 *expression (식)* 을 타입으로 만들면 어떻게 될까요?

![](http://upload.wikimedia.org/wikipedia/commons/thumb/9/98/Exp-tree-ex-11.svg/375px-Exp-tree-ex-11.svg.png)
<p align="center">(http://en.wikipedia.org)</p>

```haskell
data Expr = Val Int
          | Add Expr Expr
          | Mul Expr Expr          
```

이제 *expression* 의 사이즈와, 계산 결과를 돌려주는 함수 `size`, `eval` 을 만듭시다.

```haskell
size :: Expr -> Int
size (Val n) = 1
size (Add l r) = size l + size r
size (Mul l r) = size l + size r

eval :: Expr -> Int
eval (Val n) = n
eval (Add l r) = eval l + eval r
eval (Mul l r) = eval l * eval r

> eval (Add (Val 3) (Val 2))
-- 5

> size (Add (Val 3) (Val 2))
-- 2
```

이번엔 이진트리를 표현해 볼까요?

```haskell
data Tree = Leaf Int
          | Node Tree Int Tree
```

이제 트리에서 원하는 숫자가 존재하는지 검사하는 `occurs ` 함수를 만들면

```haskell
occurs :: Int -> Tree -> Bool
occurs n (Leaf k) = n == k
occurs n (Node l k r) =
  (n == k) 
  || occurs n l
  || occurs n r
  
> occurs 3 (Node (Leaf 3) 4 (Leaf 5))
-- True

> occurs 6 (Node (Leaf 3) 4 (Leaf 5))
-- False  
```

트리의 모든 원소를 리스트로 돌려주는 `flatten` 함수도 만들어 봅시다.

```haskell
flatten :: Tree -> [Int]
flatten (Leaf k) = [k]
flatten (Node l k r) = flatten l ++ [k] ++ flatten 

> flatten (Node (Leaf 3) 4 (Leaf 5))
-- [3,4,5]

> flatten (Node (Leaf 6) 4 (Leaf 7))
-- [6,4,7]
```

여기서 재미난 결과를 볼 수 있습니다. `flatten` 함수는 매 재귀마다 왼쪽부터 방문하고, 현재 노드를 방문하고, 마지막으로 오른쪽 노드를 방문합니다. 

그래서 `flatten` 함수의 결과가 *ordered* 이면 트리는 한 노드를 기준으로 한쪽은 현재 노드보다 작고, 다른쪽은 큰 *search-tree* 가 됩니다.

*search-tree* 에서는 만약 찾으려는 수가 현재 노드보다 크면 *right sub-tree* 만, 현재 노드보다 작으면 *left sub-tree* 만 검색하면 됩니다. 따라서 `occurs` 함수를 

```haskell
-- occurs for search-ree
occurs' :: Int -> Tree -> Bool
occurs' n (Leaf k) = k == n
occurs' n (Node l k r) | n == k = True
                       | n < k = occurs' n l
                       | otherwise = occurs' n r


> occurs' 3 (Node (Leaf 3) 4 (Leaf 5))
-- True

> occurs' 5 (Node (Leaf 3) 4 (Leaf 5))
-- True
```

실제로 트리는 값을 어디에 저장하냐에 따라 다양한 형태가 될 수 있습니다.

```haskell
data Tree a = Leaf a | Node (Tree a) (Tree a)
data Tree a = Leaf | Node (Tree a) a (Tree a)
data Tree a b = Leaf a | Node (Tree a b) b (Tree a b)
data Tree a = Node a [Tree a]
```

위에서 부터

(1) *leaf* 에만 값을 저장  
(2) *node* 에만 값을 저장  
(3) *leaf*, *node* 에 모두 값을 저장  
(4) 한 *node* 에 값과 복수개의 트리를 저장  

### Tautology checker

항상 참인 명제를 *tautology* 라고 합니다. 논리학에 대해서는 다음 글을 참조해주세요.

(1) [명제논리의 기초 1 : 소개](http://imnt.tistory.com/91)  
(2) [명제논리의 기초 2 : 진리표](http://imnt.tistory.com/91)  
(3) [명제논리의 기초 3 : tautology, contradiction](http://imnt.tistory.com/91)  

*tautology* 는 여러가지가 있습니다. 한 가지 예를 보면, 참 또는 거짓일 수 있는 명제 `p`, `q` 에 대해 `p -> q ^ q -> p` 는 항상 참입니다.

![](http://s1.hubimg.com/u/3891828_f520.jpg)
<p align="center">(http://julieburke.hubpages.com)</p>

이번에 만들 프로그램에서는 논리학 연산자를 `Not`, `And`, `Imply`, `Or` 4가지로 제한하겠습니다. *proposition (명제)* 는 `A, ..., Z` 이고 각각 `True / False` 일 수 있습니다.

```haskell
data Prop =  Const Bool
          | Var Char
          | Not Prop
          | Or Prop
          | And Prop Prop
          | Imply Prop Porp
```

검사할 4개의 명제를 만들어 보죠.

```haskell
-- A and ~A
p1 :: Prop
p1 = And (Var 'A') (Not (Var 'A'))

-- A and B -> A
p2 :: Prop
p2 = Imply (And (Var 'A') (Var 'B')) (Var 'A')

-- A -> A and B
p3 :: Prop
p3 = Imply (Var 'A') (And (Var 'A') (Var 'B'))

-- (A and (A -> B)) -> B
p4 :: Prop
p4 = Imply (And (Var 'A') (Imply (Var 'A') (Var 'B'))) (Var 'B')
```

각 명제가 참인지 거짓인지 알 수 있는 테이블을 나타내는 타입 `Subst` 를 만듭시다. 진리표라고 생각하면 됩니다. 그리고 여기서 값을 찾는 함수 `find` 도 만들면

```haskell
type Assoc k v = [(k, v)]
type Subst = Assoc Char Bool

find :: Eq k => k -> Assoc k v -> v
find k t = head [v | (k', v) <- t, k' == k]
```

이제 `Prop` 를 평가하는 함수 `eval` 을 만들면

```haskell
eval :: Subst -> Prop -> Bool
eval _ (Const b) = b
eval s (Var x) = find x s
eval s (Not p) = not (eval s p)
eval s (Or p1 p2) = eval s p1 || eval s p2
eval s (And p1 p2) = eval s p1 && eval s p2
eval s (Imply p1 p2) = eval s p2
```

어떤 명제 `Prop` 가 *tautologt* 인지 검사하려면, 명제를 이루는 문장의 모든 참/거짓 경우에 대해 살펴봐야 합니다. 따라서 현재 가진 변수 `A, ..., Z` 에 대해서 참 / 거짓의 모든 경우를 포함한 테이블이 필요합니다. 이 함수를 만들기 위해 작은 함수부터 차근차근 조립해 갑시다.

먼저 현재 `Prop` 에서 모든 변수를 찾는 함수 `vars` 와 중복을 제거하는 함수 `uniq` 를 만들겠습니다.

```haskel
vars :: Prop -> [Char]
vars (Const _) = []
vars (Var x) = [x]
vars (Not p) = vars p
vars (And p1 p2) = vars p1 ++ vars p2
vars (Or p1 p2) = vars p1 ++ vars p2
vars (Imply p1 p2) = vars p1 ++ vars p2

> vars p1
-- "AA"

> vars p2
-- "AA"

> vars p3
-- "ABA"

> vars p4
--"AAB"

uniq :: Eq a => [a] -> [a]
uniq = foldr (\x xs-> if elem x xs then xs else x:xs) []

> uniq (vars p4)
-- "AB"
```

길이를 받으면 해당 길이 만큼 `True, False` 의 모든 조합을 리턴하는 `bools` 함수도 만들죠. 조합이므로 다음 재귀 단계에, 가능한 모든 경우를 더하면 됩니다.

```haskell
bools :: Int -> [[Bool]]
bools 0 = [[]]
bools n = map (False:) prev ++ map (True:) prev
  where prev = bools (n - 1)
```

이제 마지막 퍼즐을 완성하겠습니다. `Prop` 를 받아, `[Subst]` 를 돌려주는 함수 `substs` 와, `Prop` 를 받아 *tautology* 인지 검사하는 함수 `isTaut` 는

```haskell
substs :: Prop -> [Subst]
substs p = map (zip vs) (bools (length vs))
  where vs = uniq (vars p)
  
isTaut :: Prop -> Bool
isTaut p = and [eval s p | s <- substs p]

> isTaut p1
-- True

> isTaut p2
--False

> isTaut p3
--True

> isTaut p4
--False

> isTaut p5
-- True
```

### Abstract Machine

간단한 수식 계산을 위한 *expression* 타입을 생각해 봅시다.

```haskell
data Expr = Val Int 
          | Add Expr Expr
          
value :: Expr -> Int
value (Val n) = n
value (Add l r) = value l + value r
```

이제 덧셈을 실제로 해 보면, 계산이 왼쪽부터 이루어지는걸 확인할수 있습니다.

```haskell
-- 2 + 3 + 4
value (Add (Add (Val 2) (Val 3)) (Val 4))
...
...
(2 + value (Val 3)) + value (Val 4)
```

위에서 알 수 있듯이 왼쪽 인자가 오른쪽 인자보다 먼저 평가됩니다. 이건 우리가 지정한게 아니고, 하스켈이 왼쪽 인자부터 평가하기 때문입니다.

*expression* 에서 평가 시점을 결정하는 *abstract machine* 을 만들어서 해결할 수 있습니다.

> If desired, however, such control information can be made explicit by defining an abstract machine for expressions,
which specifies the step-by-step process of their evaluation.

컨트롤 스택을 위한 타입을 만들고, 값을 평가하는 `eval` 과 실제로 덧셈을 수행하는 `exec` 함수를 만들겠습니다.

```haskel
-- expression
data Expr = Val Int
          | Add Expr Expr
            
-- control stack
type Cont = [Op]
data Op = EVAL Expr
        | ADD Int

eval :: Expr -> Cont -> Int
eval (Val n) c = exec c n -- eval n
eval (Add x y) c = eval x (EVAL y : c) -- eval x before y

exec :: Cont -> Int -> Int
exec [] n = n
exec (EVAL y : c) n = eval y (ADD n : c)
exec (ADD n : c)  m = exec c (n + m)

value :: Expr -> Int
value e = eval e []
```

`eval (Add x y)` 에 대해서 `EVAL y` 가 먼저 스택에 들어가고, `x` 가 먼저 평가됩니다. 그 이후에 `exec` 로 넘어오면서 `ADD` 명령이 스택에 들어가고, 그 이후에야 `y` 가 평가됩니다. 마지막으로 컨트롤 스택에 들어간 `ADD` 명령이 끝납니다.

간단한 예제를 통해 평가되는 과정을 보면

```haskell
eval (Add (Val 3) (Val 5)) []
eval (Val 3) [EVAL (Val 5)]
exec [EVAL (Val 5)] 3
eval (Val 5) [ADD 3]
exec [ADD 3] 5
exec [] (3 + 5)
```

### Class and Instance declaration

마지막으로 `class` 대해 알아보겠습니다. 아참 시작하기 전에 먼저 아셔야 할 사실은, 하스켈에선 기술적인 이유로 `data` 를 이용해 만든 타입만 클래스의 인스턴스가 될 수 있습니다. 

> For technical reasons, only types declared using the data mechanism can be made into instances of classes.

하스켈에선 `Eq` 클래스가 있는데요, 이렇게 정의되어 있습니다.

```haskell
class Eq where
 (==), (/=) :: a -> a -> Bool
 x /= y = not (x == y) 
```

이 말은 `Eq` 의 인스턴스가 되는 `a` 는 `(==)` 연산을 지원해야 한다는 뜻입니다. (`/=` 연산은 디폴트로 정의되어 있습니다.)

그래서 `Eq` 의 인스턴스인 `Bool` 의 경우

```haskell
instance Eq Bool where
  False == False = True
  True == True = True
  _ == _ = False
```

물론 기본 연산은 *overrided* 될 수 있습니다. 어떤 인스턴스의 경우 비교를 위해 `==` 를 재정의해서 사용할 수 있을겁니다.

클래스는 확장될 수 있습니다. 다른언어의 상속처럼요.

```haskell
class Eq a => Ord a where
  (<), (<=), (>), (>=) :: a -> a -> Bool
  min, max :: a -> a -> a

  min x y | x <= y = x
          | otherwise = y

  max x y | x <= y = y
          | otherwise = x
```

상대적인 크고 작음을 의미하는 `Ord` 클래스의 경우 `Eq` 의 연산에 추가적으로 크기 비교를 위한 연산을 가지고 있습니다. `<, <=, >, >=` 4개의 연산을 `Ord` 의 인스턴스는 정의해야 하는데요, 이것만 정의하면 디폴트로 정의된 `min, max` 도 사용할 수 있습니다.

아까 `Imply` 구현할 때 `p <= q` 연산 보셨죠? `Bool` 은 `Ord` 의 인스턴스이기도 한데요

```haskell
instance Ord Bool where
  False < True = True
  _ < _ = False

  b > c = c < b
  b <= c = (b < c) || (b == c)
  b >= c = c <= b
```

### Derived instances

타입을 만들때 *built-in* 클래스의 인스턴스로 만들기 위해 `deriving` 키워드를 사용할 수 있습니다. 그래서 `Bool` 같은 경우 콘솔에 출력도 되고, 문자열에서 변경도 가능하고, 비교도 가능하죠.

```haskell
data Bool = False | True
          deriving (Eq, Ord, Show, Read)
          
> False == False
--True

> False < True
-- True

> show False
-- "False"

> read "False"::Bool
--False
```

한가지 재밌는 사실은 `Bool` 의 *constructor* 중에서 `False` 가 `True` 보다 먼저 나오기 때문에 `False < True` 라는 사실입니다.

```haskell
data Shape = Circle Float | Rect Float Float
data Maybe a = Nothing | Just a
```

그리고 `Float` 가 `Eq` 의 인스턴스이기 때문에 결과적으로 이것을 파라미터로 가지는 *constructor* `Circle`, `Rect` 도 `Eq` 의 인스턴스입니다.

```haskell
> Rect 1.0 4.0 < Rect 2.0 3.0
True

> Rect 1.0 4.0 < Rect 1.0 3.0
False
```

마찬가지로 `Maybe a` 가 `Eq` 의 인스턴스가 되려면 `a` 가 `Eq` 의 인스턴스여야 합니다.

### Examples

몇개의 예제들입니다. 참고해보세요

```haskell
import Data.List
import Data.Char
import Unsafe.Coerce

data Nat = Zero
         | Succ Nat
         deriving Show

nat2int :: Nat -> Integer
nat2int = \n -> genericLength [c | c <- show n, c == 'S']

int2nat 0 = Zero
int2nat n = Succ (int2nat (n-1))

add :: Nat -> Nat -> Nat
add n Zero = n
add n (Succ m) = Succ (add m n)

mult m Zero = Zero
mult m (Succ n) = add m (mult m n)

-- tree
data Tree1 = Leaf Integer
          | Node Tree Tree

leaves (Leaf _) = 1
leaves (Node l r) = leaves l + leaves r
balanced :: Tree -> Bool
balanced (Leaf _) = True
balanced (Node l r) = abs (leaves l - leaves r) <= 1 && balanced l && balanced r

balance :: [Integer] -> Tree
halve xs = splitAt (length xs `div` 2) xs
balance [x] = Leaf x
balance xs = Node (balance ys) (balance zs)
  where (ys, zs) = halve xs
```

### References

(1) **DelftX FP 101x**   
(2) *Programming in Haskell*  
(3) [Wiki - Binary Expression](http://en.wikipedia.org/wiki/Binary_expression_tree)  
(4) [http://imnt.tistory.com](http://imnt.tistory.com)  

