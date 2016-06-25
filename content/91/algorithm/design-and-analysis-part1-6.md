+++
date = "2016-06-25T12:54:52+09:00"
prev = "../design-and-analysis-part1-5"
title = "Design and Analysis: Hash Table, Universal Hashing, Bloom filters"
toc = true
weight = 16
aliases = [
    "/hash-table-universal-hashing-bloom-filters"
]
+++

### Hash Table

해시 테이블의 연산은 *key* 를 이용해 이런 작업들을 한다.

(1) **insert:** add new record  
(2) **delete:** delete existing record  
(2) **lookup:** check for a particular record

가끔 사람들이 *dictionary* 라 부르기도 하는데, 해시테이블은 알파벳 순서같은 특정 *order* 로 데이터를 저장하진 않는다.

이 3가지 연산이 거의 `O(1)` 라 보면 된다. 물론 이건 해시테이블을 잘 설계 했을때다. 슬프게도, 해시테이블은 *잘못* 구현하기 쉽다. 해시테이블이 `O(1)` 성능이 나오려면

- properly implemented
- non-pathological data

여기서 *non-pathological* 이란 *collision* 을 만들지 않는 데이터를 말한다.

#### Application

(1) 주어진 *object stream* 을 *de-duplication* 하기 위해 해시테이블을 쓸 수 있다. 해시테이블에 들어오는 객체를 *lookup* 해 보고 없으면 채워 넣고, 있으면 무시한다.

(2) *2-Sum Problem* 에도 해시테이블을 쓸 수 있다. *2-Sum* 을 푸는 `O(n logn)` 방법은, (`x + y = t`)

먼저 정렬 후 `A` 의 원소 `x` 에 대해 `t - x` 를 이진탐색하는 방법이다. 이렇게 하면 `O(n logn)` 으로 해결할 수 있다.

해시테이블을 이용하면 정렬 할 필요도 없고, 이진탐색 대신 *lookup* 으로 `O(1)` 시간에 원소를 검색할 수 있으므로 더 빨라진다.	 

해시 테이블을 만드는데 `O(n)`, 탐색에 `O(1 * n)` 에서, `O(n)` 만에 *2-Sum* 을 해결할 수 있다.

(3) 이외에도

- symbol tables in compilers
- blocking network traffic
- search algorithm (e.g **game tree exploration**)

### Hash Table Implementation

모든 집합을 의미하는 *universe `u`* 에 대해 a reasonable size* 의 *evolving set* `s <= u`* 을 유지하면 된다.

- 배열로 구현할 경우 *lookup* 은 `O(1)` 이지만 메모리가 `O(|u|)` 다.
- 리스트로 구현할 경우 `O(|s|)` 의 메모리를 차지하지만, *lookup* 이 `O(|s|)` 다.

더 나은 방법은 없을까?

(1) *bucket size* 인 `n ~ |s|` 인 `n` 을 고른다. 이 때 `|s|` 는 그렇게 많이 안 변한다고 가정하자.  
(2) 그 후 *hash function* `h: u -> {0, 1, ..., n-1}` 인 `h` 를 고르면 된다.  
(3) 길이 `n` 의 배열 `A` 에, `A[h(x)]` 위치에 `x` 를 저장하면 된다.

이제 충돌 문제를 고민해 보자. 한 방에 `23` 명만 있어도, 생일이 같은 2명이 존재할 확률이 `50%` 가 넘으므로, 

`n` 에 비해 그리 크지 않은 *input size* 에 대해서도 충돌이 발생할 확률이 꽤 높다.

> **Collision:** dinstinct `x, y in u` such that `h(x) = h(y)`

### Resolving Collisions

충돌을 해결하기 위한 첫 번째 방법은

(1) **Chaining:**

`A[h(x)]` 을 리스토로 만들어 충돌이 발생하는 원소를 리스트에 저장한다.

(2) **Open Addressing:**

충돌이 발생하면 새로운 *bucket* 을 찾도록 해 하나의 *bucket* 당 하나의 원소만 들어갈 수 있도록 한다.

> hash function now specifies probe sequence `h_1(x), h_2(x), ...` keep trying til find open slot.

<br/>

*open addressing* 에서는 *probing, 탐사* 방식을 통해 비어있는 *bucket* 을 찾는다. 몇 가지 방법이 있는데

- **linear probing:** 순차적으로 탐색한다. 캐쉬 히트는 높으나, 클러스터링에 취약하다.
- **double hashing probing:** 해쉬 함수 충돌이 발생하면 2차 해쉬 함수를 이용한다. 계산 비용이 비싸고, 캐쉬효율도 낮지만, 클러스터링에 영향을 받지 않는다.
- **quadratic probing:** 2차 함수를 이용해서 탐색을 위치를 찾는데, 캐싱과 클러스터링에서 두 방식의 중간정도의 성능을 보여준다.

#### Clustering

*key `k`* 에 대한 최초의 해쉬 함수 값 `h(k)` *home position* 이라 부르는데, 같은 *home position* 를 갖는 *key* 들을 모아 *cluster* 라 부른다. *cluster* 가 커지면 커질수록, 클러스터의 중간을 *home position* 으로 하는 키가 들어올 확률도 높아지고, 인접한 클러스터와 합쳐지는 속도도 빨라진다.

결국 *linear probing* 의 경우 *load factor* 가 높아질수록 해쉬 테이블의 성능이 `O(n)` 으로 떨어진다.

#### Chaining vs Open-addessing

[여기](http://sweeper.egloos.com/viewer/925740)를 인용하면

*chaining* 은 *open addressing* 에 비해 다음의 장점을 가진다.

> 삭제 작업이 간단하다. 삭제 작업이 빈번하다면 *open addressing* 보다는 *chaining* 이 낫다.

> **chaining** 은 클러스터링에 거의 영향을 받지 않아 충돌의 최소화만 고려하면 된다. 반면 **open addressing** 은 클러스터링까지 피해야 하므로 해쉬함수를 구현하기가 쉽지 않다.

> *load factor* 가 높아져도 성능 저하가 선형적이다. 아래 그림에서 볼 수 있듯이, *open-addressing* 방법처럼 급격히 *lookup time* 이 늘지 않는다. 따라서 테이블 확장을 상당히 늦출 수 있다.

![](http://pds5.egloos.com/pds/200702/14/32/d0014632_11023351.jpg)

> 데이터의 크기가 *5 words and more* 이면, *open addressing* 보다 메모리 사용량이 적다. 

반면 *open addressing* 은

> 어떠한 포인터도 저장할 필요가 없고, 테이블 외부에 추가적인 공간이 필요 없으므로 메모리 효율이 높다.

> 특히 *linear probing* 에서 뛰어난 *locality* 때문에 데이터가 캐쉬라인을 채울 정도로 크지 않다면 좋은 성능을 낼 수 있다.

정리하자면,

> open-addressing 방식은 테이블에 모두 저장될 수 있고 캐쉬 라인에 적합할 수 있을 정도로 데이터의 크기가 작을수록 성능이 더 좋아진다. 메모리 비용을 아끼려면, 이 방법이 적합하다.

> 반면 테이블의 높은 load factor가 예상되거나, 데이터가 크거나, 데이터의 길이가 가변일 때 chained 해쉬 테이블은 open-addressing 방식보다 적어도 동등하거나 훨씬 더 뛰어난 성능을 보인다. 삭제가 중요하고, 빈번한 연산이라면 *chianing* 이 더 낫다.

### What Makes a Good Hash Function?

*chaining* 을 생각해 보자. 

- **insert:** `O(1)`
- **lookup, delete:** `O(list length in the bucket)`

이때 하나의 버켓에 들어있는 *list length* 는 `m/n` 부터 `m` 까지 일 수 있기 때문에 (`m` 개의 오브젝트에 대해), 해쉬 함수에 따라 성능이 정말 달라진다.

이로부터 좋은 해쉬함수의 기준을 알 수 있다.

> 1. Should lead to good performance => *"spread data out"*   
>(gold standard: completely random hashing)
> 2. Should be easy to store / very fast to evaluate  

### Quick and Dirty Hash Function

좋은 해쉬함수를 설계할 수 있다면 좋겠지만, 시간이 없을때 객체 `u` 를 받아 정수 `n` 으로 만들어 *bucket* 을 찾는 해쉬함수를 이렇게 디자인할 수 있다.

- `u -> n`: *hash code*
- `n -> bucket`: *compression function* using `mod`

여기서 `n` 은 어떻게 고를까? 

(1) 우리가 *compression function* 으로 `mod` 를 사용하기 때문에 소수여야 한다. 소수가 아니라면, `n` 으로 나누어지는 모든 수는 `mod n == 0` 이 되어, 같은 *bucket* 에 할당될 것이다. 물론 이 수는 너무 커서는 안되고, 객체를 담을 수 있을만한 적당한 숫자여야 한다.

(2) *input data* 의 패턴을 고려해 `n` 을 정해야 한다. 예를 들어 *memory location* 이 4의 배수일 때, 테이블 사이즈 `n` 을 `2^j` 로 정해버리면, `mod n == 0` 이 되는 경우가 많아 *empty bucket* 이 많이 생길 것이다.

그리고 `n` 을 `2^k, 10^k` 로 정해버리는 경우 `mod` 연산이 시프팅으로 쉽게 구현되는데 이는 나머지 데이터를 고려하지 않고 일부의 데이터만으로 버킷을 찾아가므로 별로 좋은 선택이 아니다.

### Load Factor

*evenly spread out* 에 대해 고민해 보았으니, 이제 *non-pathological* 을 생각해 보자.

용어부터 정의하고 가면 *load factor* 는 해시테이블에 들어있는 오브젝트 수를, 버킷 수로 나눈 것이다.

*open addressing* 의 경우에는 *load factor* 가 1보다 클 수 없지만 *chaining* 은 가능하다.

(1) *load factor* `a = O(1)` 이어야 연산이 *constant time* 이다.  
(2) *open addressing* 이라면 `x << 1` 이어야 한다.

따라서 해시 테이블의 성능을 위해서는 *load factor* 를 조절해야 한다.

### Pathological Data Sets

모든 데이터에 대해 *evenly spread out* 할 수 있는 해시함수가 있다면 좋겠지만, **그런 해시 함수는 없다.**

모든 해시 함수는 자신만의 *pathological data set* 이 있다. 이는 쉽게 보일 수 있는데, *universe `u`* 와 대해 버켓 수 `n` 에 대해 해시함수 `h: u -> {0, 1, ..., n-1}` 이 있다고 하자.

비둘기 집 원리에 의해 모든 *bucket* 은 적어도 `|u|/n` 개의 데이터를 담고 있다. 따라서 `u` 중에서 어느 한 *bucket* 에만 담을 수 있는 데이터 셋을 고르면, 그것이 바로 *pathological data set* 이다.

이런 *pathological data set* 은 *service attack* 에 쓰이기도 한다. 따라서 오픈소스라면 리버스엔지니어링 하기 쉽지 않게끔 해시함수를 설계하는 것도 필요하다.

그럼 모든 해시 함수가 이런 데이터 셋을 가지고 있고, 심지어 공격에도 이용할 수 있다면 어떻게 해시함수를 설계해야 이런 문제를 조금이나마 피할 수 있을까?

(1) **use a cryptographic hash function** (e.g., **SHA-2**)  

infeasible to reverse engineer a pathological data set

(2) **use randomization**  

design a family `H` of hash funcitons such that data sets `S`, "almost all" functions `h in H` spread `S` out "pretty evenly" (compare to quicksort guarantee)

이제 *universal hashing* 이 무엇인지 알아보자

### Universal Hashing Functions

> Let `H` be a set of hash functions from `u` to `{0, 1, ..., n-1}`

> `H` is **universal** if and only if,

> for all `x, y in u (x != y)` `P[h(x) = h(y)] <= 1/n` when `h` is chosen unifomly at random from `H` where `n` is the number of buckets. 

#### Hashing IP Addresses

*IP Address* 를 예로 들어 설명해보면 *IP* 를 `(x1, x2, x3, x4)` (`xi = 0 to 255`), *bucket* 수 `n` 을 소수라 하자.

*tuple `a = (a1, a2, a3, a4), where ai in {0, ..., n-1}`* 에 대해서

`h_a` 를 이렇게 정의하자. 이러면 `h_a` 는 `n^4` 개 존재한다.

`h_a(x1, x2, x3, x4) = (a1x2 + a2x2 + a3x3 + a4x4) mod n`

이제 `h_a` 의 집합 `H` 는 *universal* 이다.

`H = { h_a | a1, a2, a3, a4 in {0, 1, ..., n-1} }`

#### Proof

서로 다른 *IP* `(x1, x2, x3, x4), (y1, y2, y3, y4)` 를 생각해보자.

만약 `x4 != y4` 라면, 충돌이 일어날 확률은 얼마일까? 충돌에 대한 식을 좀 정리하면

![](http://latex.codecogs.com/gif.latex?collision%5C%20means%20%5C%5C%20%5C%5C%20%28a_1x_1%20&plus;%20a_2x_2%20&plus;%20a_3x_3%20&plus;%20a_4x_4%29%20%5Cmod%20n%20%3D%20%28a_1y_1%20&plus;%20a_2y_2%20&plus;%20a_3y_3%20&plus;%20a_4y_4%29%20%5Cmod%20n%20%5C%5C%20%5C%5C%20so%2C%20%5C%5C%20%5C%5C%20a_4%28x_4-y_4%29%20%5Cmod%20n%20%3D%20%5Csum_%7Bi%20%3D%201%7D%5E3%20a_i%28y_i%20-%20x_i%29%20%5Cmod%20n)

이 때 `a1, a2, a3` 를 고정하면 얼마나 많은 `a4` 에 대해 아래 식이 성립할까? 

![](http://latex.codecogs.com/gif.latex?a_4%28x_4-y_4%29%20%5Cmod%20n%20%3D%20%5Csum_%7Bi%20%3D%201%7D%5E3%20a_i%28y_i%20-%20x_i%29%20%5Cmod%20n)

`xi, yi, a1, a2, a3` 가 *fixed* 기 때문에 우변은 `{0, ..., n-1}` 사이의 숫자고 `a4` 만 랜덤이다.

이 때

- `x4 != y4` 이므로 `x4 - y4 != 0` 이다
- `n` 이 `ai` 의 최대값보다 큰수이면서 동시에 소수인데다가
- `a4` 가 *uniform at random* 이기 때문에

> left-hand side equally likely to be any of `{0, 1, ..., n-1}`.

따라서 좌변이 특정 숫자인 우변과 같을 확률은 `1/n` 이다.

![](http://latex.codecogs.com/gif.latex?P%5Bh_a%28x%29%20%3D%20h_a%28y%29%5D%20%3D%20%7B1%20%5Cover%20n%7D)

### Analysis of Chaining

*universal hash functions* 의 정의를 한번 더 보고 넘어가면,

`H` 가 해시함수 `u -> {0, ..., n-1}` 의 집합일때 `H` 가 다음을 만족하면 *universal* 하다.

- `x != y` 인 `u` 내의 `x, y` 에 대해 충돌이 일어날 확률 `P <= 1/n` 이고
- `H` 내에서 `h` 가 *uniformly at random* 하게 선택될때

만약 해시 테이블이 *chaining* 을 이용해 구현되었을때, *universal family* `H` 로부터 해시함수 `h` 가 *uniformly a random* 하게 선택되면 모든 연산이 `O(1)` 이다.

그리고, `|S| = O(n)` 다시 말해 *load factor* `alpha = |S| / n = O(1)` 임을, 해시 함수를 평가하는데 `O(1)` 임을 가정한다.

#### Proof

*unsuccessful lookup* 을 분석할건데, 다른 연산이 이보다는 항상 더 빠르므로 다른 연산의 *upper bound* 라 보면 된다.

`S` 를 `|S| = O(n)` 인 데이터셋이라 하자. `x not in S` 인 `x` 를 *lookup* 한다 하면 *running time* 은

`O(1) + O(list length in A[h(x)])` 다. 즉 `h(x)` 를 평가하는데 걸리는 시간과 해당 버킷 내의 리스트를 순회하는 시간의 합이다.

그런데 여기서 `A[h(x)]` 버킷의 리스트 길이를 `L` 이라 하면 이 `L` 은 `h` 선택에 따라 달라지는 *random variable, 확률변수* 다.

그럼 *average list length* 를 구해, `O(L)` 을 구해보자. 기대값의 선형성을 이용할건데, `E(L)` 을 위한 `1 or 0` 의 확률변수를 도입하자.

`x != y` 인 `y in S` 에 대해 `z_y` 를 충돌이 날경우 `1` 로, 아닐 경우를 `0` 으로 하면

![](http://latex.codecogs.com/gif.latex?z_y%3D%20%5Cbegin%7Bcases%7D%201%2C%20%26%20%5Cmbox%7Bif%20%7Dh%28x%29%20%3D%20h%28y%29%20%5C%5C%200%2C%20%26%20%5Cmbox%7Bif%20%7Dh%28x%29%20%5Cneq%20h%28y%29%20%5Cend%7Bcases%7D)

이 때 해시함수가 무엇이든, 충돌이 날 경우에만 같은 버킷으로 들어가므로 버킷의 길이는 

![](http://latex.codecogs.com/gif.latex?L%20%3D%20%5Csum_%7By%5C%20%5Cin%5C%20S%7D%20z_y)

따라서

![](http://latex.codecogs.com/gif.latex?E%28L%29%20%5C%5C%20%5C%5C%20%3D%20E%5B%5Csum_%7By%5C%20%5Cin%5C%20S%7D%20z_y%5D%20%5C%5C%20%5C%5C%20%3D%20%5Csum_%7By%5C%20%5Cin%5C%20S%7D%20E%28z_y%29%20%5C%20%5C%20%5Cmbox%7B%28apply%20linearity%20of%20expectation%29%7D%20%5C%5C%20%5C%5C%20%3D%20%5Csum_%7By%5C%20%5Cin%20%5C%20S%7D%20%7B1%20%5Cover%20n%7D%20%5C%5C%20%5C%5C%20%5Cleq%20%7CS%7C%20*%20%7B1%20%5Cover%20n%7D%20%5C%5C%20%5C%5C%20%3D%20O%281%29)

중간에 `H` 가 *universal* 이므로 `P[h(y) = h(x)] <= 1/n` 이다.

### Open Addressing Performance

*open addressing* 퍼포먼스를 계산할건데, *quick and dirty idealized analysis* 를 위해 *heuristic assumtion* 을 도입하면,

> All `n!` probe sequences equally likely	

이상적인 경우를 가정하면 얻어지는 것은

> expected insertion time ~= `1 / (1 - a)` where `a = load factor`

다시 말해서, `a = 0.5` 라면 새로운 데이터를 집어넣기 위해 `2` 만큼 *probe* 해야한다는 소리다. 반면 `a ~= 1` 이면 (`1`에 가까워지면) *insertion* 타임은 어마어마하게 커진다.

> A random probe finds an empty slot with probability `1 - a`

이 문제를 "*head* 를 얻기 위해 동전을 몇번 뒤집어야 하는가" 로 치환할 수 있다. 여기서 `Pr[heads] = 1 - a` 라 보면

*head* 를 얻기 위해 동전을 뒤집는 수 `N` 에 대해 기대값 `E[N]` 은

![](http://latex.codecogs.com/gif.latex?E%5BN%5D%20%3D%201%20&plus;%20%5Calpha%20%5C%20E%5BN%5D)

식을 풀면

![](http://latex.codecogs.com/gif.latex?E%5BN%5D%20%3D%20%7B1%20%5Cover%201-%20%5Calpha%7D)

#### Linear Probing

*open addressing* 방법으로 *linear probing* 을 사용할 경우, 아까의 *heuristic assumption* 자체가 성립하지 않는다.

따라서 다른 가정으로

> initial probe uniformly random, independent for different keys.

그러면 가정아래,  *expected insertion time* 은 `1 / (1 - a)^2` 에 가까워진다. (*D.E Knuth* 가 발견했다고 한다.)

### Bloom Filter

하던대로 *supported operation* 부터 이야기 하자.

블룸 필터는 해시테이블과 비슷하게 빠른 삽입, 탐색을 지원한다. 해시테이블과 비교했을때 메모리가 덜 든다. 반면 단점은

(1) Can't store an associated object  
(2) No deletions  
(3) small **false positive** pobability (but no false negative)

블룸 필터는 다양한 곳에 사용한다. 

- early spell checkers (original)
- list of forbidden passwords (canonical)
- network routers (mordern)

만약 메모리가 아주 비싸고, *false positive* 를 참을만 하다면 블룸필터는 좋은 선택이다. 연산도 아주 빠르다.

블룸 필터의 구성요소를 보자.

(1) 자료구조는 `n` 비트의 배열이다.  
(2) `k` 개의 해시 함수가 필요하다. (`k` 는 *small constant*)

*insertion* 은 `i = 1, ..., k` 에 대해 `A[h_i(x)] = 1` 로 세팅하면 된다. 이미 `1` 이어도 덮어쓴다. 참고로 덮어쓰기때문에 *false positive* 는 있어도 *false negative* 는 없다. 자그마한 종양만 보여도 무조건 암이라 주장하는 소심한 의사라 보면 이해가 쉽다.

*lookup* 은 `i = 1, ..., k` 에 대해 모든 `A[h_i(x)] = 1` 이면 찾으려는 `x` 가 존재한다.

*false positive* 는 `A[h_i(x)]` 가 다른 *insertion* 에 의해 `1` 로 세팅 되었을때 발생한다.

블룸필터를 이미지로 보면

![](http://upload.wikimedia.org/wikipedia/commons/thumb/a/ac/Bloom_filter.svg/720px-Bloom_filter.svg.png)
<p align="center">(http://en.wikipedia.org)</p>

블룸필터를 쓰는 것이 합리적인 선택이 되려면

(1) `n / |S|` 즉, 오브젝트당 비트 수가 충분히 작아야 한다.  
(2) *false positive*, 즉 에러 확률이 작아야한다.

동시에 두 조건을 작은 값으로, 모두 만족시키지 못한다면 그냥 해시 테이블을 쓰는 것이 더 낫다. 

근데, 자세히 살펴보면 *space* 와 *error prob*, 이 두 조건은 *trade-off* 다.  

### Bloom Filter: Heuristic Analysis

*heuristic assumption* 은

> all `h_i(x)`' is uniformly random and independent (across different `i`'s and `x`'s

`k` 개의 해시함수를 가지는 `n` 비트 블룸 필터에 데이터셋 `S` 를 먼저 넣어놓자. 이제 블룸필터 `A` 의 각 비트가 1일 확률은,

![](http://latex.codecogs.com/gif.latex?1%20-%20%281%20-%7B1%20%5Cover%20n%7D%29%5E%7Bk%7CS%7C%7D)

인데 이것은 한 비트가 `0` 일 확률을 `1` 에서 뺀 것이다. `0` 일 확률은 `k` 개의 해쉬 함수를 `|S|` 개의 모든 원소를 다 집어 넣은 후에도 `0` 인 확률이므로 

![](http://latex.codecogs.com/gif.latex?%281%20-%20%7B1%20%5Cover%20n%7D%29%5E%7Bk%7CS%7C%7D)

이 때 `e^x` 가 `1 + x` 의 *upper bound* 임을 이용하면, 각 비트가 1일 확률은

![](http://latex.codecogs.com/gif.latex?1%20-%20%281%20-%7B1%20%5Cover%20n%7D%29%5E%7Bk%7CS%7C%7D%20%5C%5C%20%5C%5C%20%5Cleq%201%20-%20e%5E%7B-k%7CS%7C%20%5Cover%20n%7D%20%5C%20%5Cmbox%7B%5C%20%5C%20%281/n%20%5Csim%200%29%7D)

이 때 `n / |S| = b`, `b` 는 오브젝트당 비트 수 이므로

![](http://latex.codecogs.com/gif.latex?1%20-%20%281%20-%7B1%20%5Cover%20n%7D%29%5E%7Bk%7CS%7C%7D%20%5C%5C%20%5C%5C%20%5Cleq%201%20-%20e%5E%7B-k%7CS%7C%20%5Cover%20n%7D%20%5C%5C%20%5C%5C%20%3D%201%20-%20e%5E%7B-k%20/%20b%7D)

이제 블룸필터에 한번도 입력되지 않은 데이터에 대해 *false positive* 확률 `P[FP]` 를 계산하면,

![](http://latex.codecogs.com/gif.latex?P%5BFP%5D%20%5C%5C%20%5C%5C%20%5Cleq%20%281%20-%20e%5E%7B-k%20/%20b%7D%29%5Ek%20%5C%5C%20%5C%5C)

이 때 고정된 수 `b` 에 대해 에러일 확률 `P[FP]` 를 최소화 하는 `k` 를 찾으면

`k ~ (ln2) * b` 다. 로그를 계산하면, `k ~ 0.693 * b` 

따라서 

![](http://latex.codecogs.com/gif.latex?P%5BFP%5D%20%5Csim%20%28%7B1%20%5Cover%202%7D%29%5E%7B%28ln2%29b%7D)

이므로 오브젝트당 비트수 `b` 에 따라서 *false positive*, 즉 에러 확률이 *exponentially* 작아진다.

식을 거꾸로 풀면

![](http://latex.codecogs.com/gif.latex?b%20%5Csim%201.44%20*%20log_2%7B1%20%5Cover%20P%5BFP%5D%7D)

이 두 식은 오브젝트당 비트수 `b` 와 *false positive* 의 *trade off* 를 보여준다.

만약 `b = 8` 이고 `k = 5, 6` 이면 에러 확률은 `2%` 정도다. 


### References

(1) *Algorithms: Design and Analysis, Part 1* by **Tim Roughgarden**  
(2) [Hash Table](http://sweeper.egloos.com/viewer/925740)  
(3) [http://www.slideshare.net/tanmaytan21](http://www.slideshare.net/tanmaytan21/application-of-hashing-in-better-alg-design-tanmay)  
(4) [Wikipedia: Hash Table](http://en.wikipedia.org/wiki/Hash_table)  
(5) [http://www.slideshare.net/sajidmarwatt](http://www.slideshare.net/sajidmarwatt/advance-algorithm-hashing-lec-ii)  
(6) [Wikipedia: Primary Clustering](http://en.wikipedia.org/wiki/Primary_clustering)  
(7) [Wikipedia: Bloom Filter](http://en.wikipedia.org/wiki/Bloom_filter)	
