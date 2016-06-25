+++
date = "2016-06-25T12:54:48+09:00"
next = "../design-and-analysis-part1-4"
prev = "../design-and-analysis-part1-2"
title = "Design and Analysis: Graph Contraction Algorithm"
toc = true
weight = 13
aliases = [
    "/graphs-the-contraction-algorithm"
]
+++

이번엔 지난시간에 배운 *randomized algorithm* 을 새로운 *domain* 인 그래프에 적용해 보고, *contraction algorithm* 이 무엇인지 알아본다.

### Graphs

용어 정리부터 시작하자. *edge* `(E)` 는 *pair of vertices* 와 같은 말이다. `(E)` 는 *directed or undirected* 일 수 있으므로 *unordered pair* 또는 *ordered pair* 일 수 있다. *directed edges* 는 다른말로 *arcs* 라 부르기도 한다.

*cut* 은 그래프를 비어있지 않은 두개의 그룹으로 분리하는 것을 말한다.

>A cut of a graph `(V, E)` is a partition of `V` into two non-empty sets `A` and `B`

따라서 *vertice* 가 `n` 개라면 `2^n - 2` 개의 *cut* 을 만들 수 있다.

### Minimum Cut Problem

*crossing edge* 를 최소로 하는 *cut* 을 찾는 문제다. 이걸 어디다 쓸 수 있을까?

(1) identify network bottlenecks / weaknesses  
(2) community detection in social network  

두 사람 혹은 집단간 강하게 결합되고 나머지와는 약하게 결합된 부분(*mimimum cut*) 을 찾으면 두 개체간 관련성이 있다고 볼 수 있다.	

(3) image segmentation  

이미지를 *2D grid* 라 *grid edge* 를 만들어 *same object* 에서 왔을 가능성을 나타내는 가중치를 부여해 *min cut* 을 하면 쓸모 없는 부분이 잘려나간다.

### Graph representation

그래프가 *sparse graphs*, *dense graphs* 냐에 따라 알고리즘이 성능이 잘 나올수도 있고 아닐수도 있기 때문에 이 두 가지를 구분해 보자.

`n` 을 *the number of vetices*, `m` 을 *the number of edges* 라 하자. 대부분의 경우에 `m` 은 `Omega(n)`, `O(n^2)` 이다.

*sparse graph* 는 `m` 이 `O(n)` 에 가깝고 *dense graphs* 는 `m` 이 `O(n^2)` 에 가깝다.

### Adjacency Matrix

그래프를 자료구조로 표현하는 몇 가지 방법이 있는데 *Adjacency matrix (인접행렬)* 의 경우에는 노드 수, `n` 에 대해 `n x n` 의 행렬 `A` 를 만들어서 `A_ij` 를 `i` 노드와 `j` 노드가 연결되었다면 값을 `1` 채운다

몇 가지 변형이 있을 수 있는데 *parallel edges* 가 허용된다면 `A_ij` 는 연결된 엣지 수 일 수 있고, `A_ij` 에 가중치를 담는 경우도 있다. *directed graph* 면 `i -> j` 냐 `j -> i` 냐에 따라 `-1 or +1` 을 값으로 사용할 수 있다.

어떤 경우든 *adjacency matrix* 방식 자체는 *edge* 수와는 관계 없이 *vertice* 수의 제곱에 비례하는 공간이 필요하다. 따라서 *sparse graphs* 에서는 사용하지 않는 편이 낫다.

### Adjacency List
 
*Adjacency list (인접 리스트)* 로 그래프를 표현할 경우엔 

(1) array (or list) of vertices (`theta(n)`)  
(2) array (or list) of edges (`theta(m)`)  
(3) each edge points to its endpoint (`theta(m)`)  
(4) each vertex points edges (`theta(m)`)

(4) 의 경우 *undirected graph* 라면 명확한데, *directed graph* 의 경우에는 *tail* 만 저장 한다던지 몇가지 방법을 쓸 수 있다. 

그럼 *adjacency list* 는 얼마의 공간을 차지할까? (3) 의 경우는 위에 표시했듯이 (`theta(m)`) 인데, 각각의 *edge* 는 2 개의 *vertex* 를 저장하지만 `2` 는 상수 취급한다.

(4) 가 노드마다 간선 수가 달라 계산이 어려울 수 있는데, (3) 과 1:1 대응이라 보면 된다. 노드가 가리키는 간선이나, 간선이 가리키는 노드나 수는 같다. 따라서 (`theta(m)`) 이므로 전체 메모리 사용은 (`theta(m + n)`) 이다. 

그러면 인접 행렬과 인접 행렬중 어떤게 더 나을까? 둘 다 장단이 있지만 *graph search* 는 단연 인접 행렬이 더 낫고, 요즘엔 *node* 는 정말 많은 반면 *edge* 는 좀 적기 때문에 인접 리스트가 더 낫다.

간단히 웹만 생각해봐도 노드 자체는 엄청나게 많은 반면 간선은 적다. 만약 인접 행렬로 그래프를 표현하면 노드 수의 제곱에 비례하는 메모리가 필요한데, 이건 리소스 문제를 겪을 수 있다.

### Random Contraction Algorithm

*min cut* 을 해결하기 위해 *quick sort*, *randomized selection* 에서 보았던 랜덤 샘플링을 이용할건데, 이 문제는 랜덤 샘플링이 그래프 문제에도 얼마나 효과적인지 보여준다. 알고리즘은 이렇다.

(1) while there are more than 2 vertices  
(2) pick a remaining edge `(u, v)` uniformly at random  
(3) merge (or "contract") `u` and `v` into a single vertex  
(4) remove self-loops  
(5) return cut represented by final 2 vertices  

해보면 알겠지만 이 알고리즘은 *min cut* 을 답으로 제공할 수도, 아닐 수도 있다. 따라서 문제는, *What is prob of success?* 를 계산하는 것으로 바뀐다.

### Analysis: Contraction Algorithm

분석 전에 몇 가지 용어를 정의하고 가자. *graph* `G = (V, E)` 에 대해 `n` 개의 *vertices*, `m` 개의 *edges* 가 있다. 그리고 *minimum cut* `(A, B)` 는 `G` 를 두개의 비어있지 않은 그룹 `A`, `B` 로 나눈다. 그리고 `k` 를 `(A, B)` 의 *crossing edges* 숫자라 하자. 그리고 이들 *crossing edges* 를 `F` 라 부르자.

만약에 `F` 중 하나의 *edge* 가 *contraction* 알고리즘 중에 선택 된다면 `(A, B)` 는 섞여버린다. 

따라서 이터레이션 동안 `A` 내부에 있는 *vertex* 끼리만, 그리고 `B` 내부에 있는 *vertex* 끼리만 *contraction* 이 일어나야 한다. 그래야만 *minimum cut* 을 찾을 수 있다.

따라서 올바른 `(A, B)` 를 아웃풋으로 얻을 확률은 `F` 중 어느 *edge*도 선택되지 않을 확률과 같다.

![](http://latex.codecogs.com/gif.latex?P_r%5Boutput%20%5C%20is%20%5C%20%28A%2C%20B%29%5D%20%3D%20P_r%5Bnever%5C%20contracts%5C%20an%5C%20edge%5C%20of%20F%5D)

~~Tex 에 맛들려서 이미지를 추가한건 아니요!~~

`S_i` 를 `F` 에 있는 *edge* 가 이터레이션 `i` 에서 *contracted* 되는 *event (사건)* 이라 하자. 그럼 우리의 목표는 다음의 확률을 계산하는 것이다.

![](http://latex.codecogs.com/gif.latex?P_r%5B%5Cneg%20S_1%20%5Ccap%20%5Cneg%20S_2%20%5Ccap%20%5Ccdots%20%5Ccap%20%5Cneg%20S_%7Bn-2%7D%5D)

증명에 사용할 재미난 그래프의 특징이 하나 있다. 모든 *vertex* 의 *incident edges, degree* 의 값은 `k` 보다 크거나 같다. 왜냐하면 모든 *vertex* 는 그 자신과 나머지를 분리하는 *cut* 을 가지는데, 이게 `k` 라면 *min cut* 이고 아니라면 `k` 보다 크기 때문이다.	

> degree of each vertex is at least `k`

그리고 모든 *vertex* 의 *degree* 는 `2m`, 즉 모든 *edge* 수의 2배이기 때문에 아래 식은 참이고, 

![](http://latex.codecogs.com/gif.latex?%5Csum_%7Bv%7Ddegree%28v%29%20%3D%202m)

위 식과 각 *degree* 합은 `kn` 보다 크거나 같으므로 `2m` 도 `kn` 보다 크거나 같다.

![](http://latex.codecogs.com/gif.latex?2m%20%5Cgeq%20kn)

![](http://latex.codecogs.com/gif.latex?m%20%5Cgeq%20%28kn/2%29)

여기서 처음 이터레이션에서 `F` 내에 있는 *edge* 가 선택될 확률인 `P(S_1) = k / m` 이기 때문에 

![](http://latex.codecogs.com/gif.latex?%7B2%20%5Cover%20n%7D%20%5Cgeq%20%7Bk%20%5Cover%20m%7D)

![](http://latex.codecogs.com/gif.latex?P_r%5BS_1%5D%20%5Cleq%20%7B2%20%5Cover%20n%7D)

이제 `P(S_1)` 을 구했으니, 두번째 이터레이션에서 `F` 내에 있는 *edge* 가 선택되지 않을 확률을 구해보자. 조건부 확률 공식을 이용하면, 

![](http://latex.codecogs.com/gif.latex?P_r%5B%5Cneg%20S_1%20%5Ccap%20%5Cneg%20S_2%5D%20%3D%20P_r%5B%5Cneg%20S_2%20%7C%20%5Cneg%20S_1%5D%20*%20P_r%5B%5Cneg%20S_1%5D)

이때 `P(~S_1)` 이 `n/2` 보다 작거나 같으므로

![](http://latex.codecogs.com/gif.latex?P_r%5B%5Cneg%20S_1%5D%20%5Cgeq%20%281%20-%20%7B2%20%5Cover%20n%7D%29)

나머지 `P(~S_2 | ~S_1)` 을 구하려다 보니 남아있는 *edge* 가 얼만지 알 수가 없다. 

![](http://latex.codecogs.com/gif.latex?P_r%5B%5Cneg%20S_2%20%7C%20%5Cneg%20S_1%5D%20%3D%201%20-%7Bk%20%5Cover%20number%20%5C%20of%5C%20remaining%20%5C%20edges%7D)

그런데, 본래의 그래프가 모든 *vertex* 에 대해 *at least* `k` 개의 *edge* 를 가졌으면 *contracted* 된 그래프도 모든 *vertex* 에 대해 *at least* `k` 개의 *edge* 를 가져야 한다. (우리는 `F` 내의 *edge* 를 선택하지 않았기 때문)

따라서 *remaining edge* 는 `1/2 * k * (n-1)` 보다 크다. (1/2 은 `n` 으로 *edge* 수를 세면 두번씩 카운팅하기 때문에 필요)

![](http://latex.codecogs.com/gif.latex?%7Bnumber%20%5C%20of%5C%20remaining%20%5C%20edges%7D%20%5Cgeq%201/2%20*%20k%20*%20%28n-1%29)

*denominator* 의 *lower bound* 를 구했기 때문에 *fraction* 의 *upper bound* 를 구한셈이 된다.

![](http://latex.codecogs.com/gif.latex?P_r%5B%5Cneg%20S_2%20%7C%20%5Cneg%20S_1%5D%20%5Cgeq%201%20-%7B2%20%5Cover%20n%20-%201%7D)

이제 규칙성이 보인다. 우리가 구하려는 값은 

![](http://latex.codecogs.com/gif.latex?P_r%5B%5Cneg%20S_1%20%5Ccap%20%5Cneg%20S_2%20%5Ccap%20%5Ccdots%20%5Ccap%20%5Cneg%20S_%7Bn-2%7D%5D)

![](http://latex.codecogs.com/gif.latex?%3D%20P_r%5B%5Cneg%20S_1%5D%20*%20P_r%5B%5Cneg%20S_2%20%7C%20S_1%5D%20*%20P_r%5B%5Cneg%20S_3%20%7C%20%5Cneg%20S_2%20%5Ccap%20%5Cneg%20S_1%5D%20*%20%5Ccdots%20*%20P_r%5B%5Cneg%20S_%7Bn-2%7D%20%7C%20%5Cneg%20S_1%20%5Ccap%20%5Ccdots%20%5Ccap%20%5Cneg%20S_%7Bn-3%7D%5D)

![](http://latex.codecogs.com/gif.latex?%5Cgeq%20%281%20-%20%7B2%20%5Cover%20n%7D%29*%281%20-%20%7B2%20%5Cover%20%28n%20-%201%29%7D%29*%281%20-%20%7B2%20%5Cover%20%28n%20-%202%29%7D%29*%5Ccdots*%20%281%20-%20%7B2%20%5Cover%20%28n%20-%20%28n-4%29%29%7D%29%20*%20%281%20-%20%7B2%20%5Cover%20%28n%20-%20%28n-3%29%29%7D%29)

정리하면

![](http://latex.codecogs.com/gif.latex?%3D%20%7B2%20%5Cover%20n%20%5C%20%28n-1%29%7D%20%5Cgeq%20%7B1%20%5Cover%20n%5E2%7D)

따라서 *contraction* 알고리즘이 성공할 확률은 `n` 이 크면 굉장히 낮다. 근데 이게 *brute-force* 에 비하면 놀랍게도 굉장히 높은 성공률이다. 

본래 `n` 개의 *vertex* 가 있으면 모든 *cut* 을 다 해 보려면 `2^n` 의 시도가 필요하다. 따라서 *contraction* 알고리즘은 꽤 높은 확률을 보장하는 알고리즘이다. 

`T_i` 를 `i` 번째 *trial* 에서 *min cut* 을 찾아낼 확률이라 하자. `N` 번의 *trial* 동안 *min cut* 을 찾지 못할 확률은, 매 *trial* 이 독립적이기 때문에 

![](http://latex.codecogs.com/gif.latex?P_r%5B%5Cneg%20T_1%20%5Ccap%20%5Cneg%20T_2%20%5Ccap%20%5Ccdots%20%5Ccap%20%5Cneg%20T_N%20%5Ccap%20%5D%20%3D%20%5Cprod_%7Bi%20%3D%201%7D%5EN%20P_r%5B%5Cneg%20T_i%5D)

![](http://latex.codecogs.com/gif.latex?%5Cprod_%7Bi%20%3D%201%7D%5EN%20P_r%5B%5Cneg%20T_i%5D%20%5Cleq%20%281%20-%20%7B1%20%5Cover%20n%5E2%20%7D%29%5EN)

이 때 `1 + x <= e^x` 란 사실을 이용하면 좀 더 간단한 *upper bound* 를 찾으 수 있다.

![](http://latex.codecogs.com/gif.latex?%281%20-%20%7B1%20%5Cover%20n%5E2%20%7D%29%5EN%20%5Cleq%20%28e%5E%7B-1%20%5Cover%20n%5E2%7D%29%5EN)

이때 `N = n^2` 이라면 `N` 번째까지 실패할 확률은 `1/e` 보다 작거나 같다. 만약에 `N = n^2 lnn` 이면 `1/n` 까지 내려간다.

따라서 단순히 계산을 반복하는 것만으로도 성공 확률을 `1/n^2` 에서 `1 - 1/n` 까지 올릴 수 있다.

*running time* 은 `Omega(n^2 * m)` 쯤 된다. `n^2` 정도의 *trial* 이 필요하고 매 *trial* 마다 `m` 의 *edge* 를 살펴봐야 한다.

여전히 느리다. 이후에는 단순히 *trial* 을 늘리는 것 뿐만 아니라 다양한 옵티마이제이션 기법을 활용하는법을 배워보자. 거의 `O(n^2)` 까지 줄일 수 있다.

### Counting Minimum Cuts

그래프를 그려보면 알겠지만 *min cut* 은 한개가 아니라 여러개 일 수 있다. 그러면 `n` 개의 *vertice* 를 가진 그래프에서 최대로 가질 수 있는 *min cut* 은 몇개 일까? 

그래프에서 각 노드마다 *edge* 가 하나밖에 없을땐 `n-1` 이고, 아무리 *cut* 이 많아봐야 `2^n - 2` 보다 적으니까 이 사이에 있는건 분명하다.

답은 *n choose 2*, `(n * (n - 1)) / 2` 다.

먼저 *lower bound* 부터 보자. *n-cycle* 그래프를 보면 2개를 끊으면 되므로 `nC2` 다. 

따라서 `n` 개의 *vectice* 를 가진 모든 그래프 중에서 가장 많은 *min-cut* 을 가진 그래프들은 적어도 이것보다는 많은 *min-cut* 을 가져야 한다.

*upper bound* 를 보자. `(A1, B1), (A2, B2), ..., (At, Bt)` 만큼의 *min cut* 이 있다 하자. 이 때 특정 *min cut* 인 `(Ai, Bi)` 가 나올 확률은 위의 증명을 다시 보면 `1/n^2` 보다 큰 `2/(n(n-1))` 이다. 이건 `nC2` 를 뒤집은 수다.

다시 말해서 *min cut* 을 뽑아낼 확률이

![](http://latex.codecogs.com/gif.latex?P%5Boutput%5C%20%3D%5C%20%28A_i%2C%20B_i%29%5D%20%5Cgeq%20%7B2%20%5Cover%20n%28n-1%29%7D%20%3D%20%7B1%20%5Cover%20%5Cbinom%7Bn%7D%7B2%7D%7D)

이 때 `S_i` 를 `(A_i, B_i)` 가 나오는 사건이라 하면 `S_i` 각각은 *disjoint* 다.

중요하니까 다시 한번 반복하면, `S_i` 는 *disjoin* 고 이로인해 모든 `S_i` 를 합하면 `1` 이다. 따라서 

![](http://latex.codecogs.com/gif.latex?%7Bt%20%5Cover%20%5Cbinom%7Bn%7D%7B2%7D%7D%20%5Cleq%201)

![](http://latex.codecogs.com/gif.latex?%7Bt%7D%20%5Cleq%20%5Cbinom%7Bn%7D%7B2%7D)

이건 *upper bound* 다. *lower bound* 와 같으므로 모든 `n` 개의 *vertice* 를 가진 그래프는 최대 `nC2` 의 *min cut* 을 가진다.

### Conditional Prob

중간에 잠깐 조건 부 확률과 독립성, 그리고 기대값에 대해 나오는데 반-직관적인 예제를 교수님이 소개해 주셔서 적어볼까 한다.

![](http://latex.codecogs.com/gif.latex?X_1%2C%20X_2%20%5Cin%20%5C%7B%200%2C%201%20%5C%7D%20%5C%20and%20%5C%20X_3%20%3D%20X_1%20%5Coplus%20X_3)

일때 `X_1` 과 `X_3` 는 독립이고, `X_1, X_3` 와 `X_2` 는 독립이 아니다. 기대값을 이용하면 쉽게 증명이 가능하다.

![](http://latex.codecogs.com/gif.latex?E%5BX_1%2C%20X_2%2C%20X_3%5D%20%5Cneq%20E%5BX_1%2C%20X_2%5D%20*%20E%5BX_3%5D)


### References

(1) *Algorithms: Design and Analysis, Part 1* by **Tim Roughgarden**  
