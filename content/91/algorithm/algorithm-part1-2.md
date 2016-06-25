+++
date = "2016-06-25T13:01:05+09:00"
prev = "../algorithm-part1-1"
title = "Algorithm: Analysis"
toc = true
weight = 32
aliases = [
    "/analysis-of-algorithms"
]
+++

알고리즘을 분석해야 하는 이유는 

(1) Predict performance  
(2) Compare algorithms  
(3) Provide guarantees  
(4) Understand theoretical basis  

그 중에서도 *performance bug* 를 피하는 것이 무엇보다 중요하다. 이를 통해 내 알고리즘이 *practical large input* 에 적용할 수 있을까? 고민하는 것이, 알고리즘 분석의 주된 동기다.

### Observations

주어진 배열에서 3개를 골라 더했을때 0이 나오는 *3-Sum* 문제를 고려해 보자. 

```java
for (int i = 0; i < N; i++)
  for(int j = i + 1; j < N; j++)
    for(int k = j + 1; k < N; k++) 
      if (array[i] + array[j] + array[k] == 0)
        count++;        
```

입력 대비 출력 시간을 *log-log plot* 으로 만들어, `y = a * N^b` 를 얻을 수 있다. 이것 보다 더 빠르게 찾는 방법은 *input* 을 두배씩 늘려가면서 로그 비율을 찾는 방법인데, *Doubling hypothesis* 라 부른다. 이 경우 `b` 가 3이 나오는걸 확인할 수 있다.

샘플 데이터가 다음과 같다고 하자. (Matlab format)

```matlab
x = [
1000;
2000;
4000;
8000;
16000;
32000;
64000]

y = [
0.0;
0.0;
0.1;
0.3;
1.3;
5.1;
20.5]
```

이 경우 수가 클때, 4배씩 증가하므로 *2-based log ratio* 는 `2` 다. 따라서 `a * N^b` 에서 `b` 를 2로 가정하면, `5.1 / 32000^2` 하면, 대략 `5.0 * 10^-9`


### Mathematical Models

사실 *Running time* 에 있어서 연산의 수행 시간은 컴퓨터마다, 또 연산마다 다르므로 이것을 제하고 *operation number*, *frequency* 의 곱을 *input number* `N` 을 이용하여 표현한 모델을 이용할 수 있다. 예를 들어 *1-Sum* 코드의 경우

```java
int count = 0;
for (int i = 0; i < N; i++)
  if (array[i] == 0)
    count++;
```

> variable declaration = 2  
> assignment = 2  
> less then compare = N + 1  
> equal to compare = N  
> array access = N  
> increment = N to 2N

*2-Sum* 코드의 경우 

```java
int count = 0;
for (int i = 0; i < N; i++)
  for(int j = i + 1; j < N; j++)
    if (array[i] + array[j] == 0)
      count++;
```

> variable declaration = 2 + N 
> assignment = 2 + N
> less then compare = 1/2 * (N + 1) * (N + 2)  
> equal to compare = 1/2 * N * (N - 1)  
> array access = N * (N - 1)  
> increment = 1/2 * N * (N - 1) to N * (N - 1)

세 보면 알겠지만 정말 귀찮다. 세야할 연산의 종류도 많고. 그렇기 때문에, *running-time* 을 간단히 추정하기 위해

(1) Use some basic operation as a proxy for running time  
(2) tilde notation: ignore lower order terms  

따라서 *2-Sum* 의 경우 유효한 연산을 배열 접근으로 잡으면, 연산 회수는 `~N^2` 이고 *3-Sum* 은 `~ 1/6 * N^3` 이다.

*discrete sum* 을 편하게 계산하는 법이 있는데, 시그마를 적분으로 바꿔, 계산하면 된다. 따라서 `1 + 2 + ... + N` 은 `1/2 * N`, `1 + 1/2 + 1/3 + ... + 1/N` 은 `ln N`, 

*3-Sum* 의 *triple loop* 는 `N * 1/2 N* 1/3 N ` 이므로, `~1/6 N` 이다. 여기에 3번의 Array Access 가 일어나므로 곱하면 *3-Sum* 은 `~1/2 N`이다.

```java
int sum = 0;
for (int i = 0; i < N; i++)
  for (int j = i; j < N; j++)
    for (int k = 1; k < N; k = k * 2)
      if (a[i] + a[j] >= a[k]) sum++;
```

모든 *triple loop* 가 `~n^3` 을 가지는건 아니다. 위의 코드의 경우 `~ 1/2 N^2 * 3 lg  N` 이다.

### Order of Growth Classification

`1` < `lg N` < `N` < `N lg N` < `N^2` < `N^3` < `2^N` 우측으로 갈 수록 알고리즘의 성능이 나쁘다. 일반적으로는 `N lg N` 이 쓸 수 있는 알고리즘이다. 좀 더 알아보자면, 

> **log N:** N 이 절반으로 나누어지는 *binary search* 등  
> **N log N:**, *divide and conquer: merge sort* 등  
> **2^N:** 서브트리를 모두 검사하는 *exhaustive search* 등  

알고리즘의 성능을 비교하기 위해 `T(2N) / T(N)` 을 구할 수 있는데, 

(1) `N` 은 `2` (따라서 lg 2 = 1 이므로, `N^1` 이다.)  
(2) `N^2` 은 `4`  
(3) `N^3` 같은 경우 `8` 이다. 
(4) `2^N` 은 `T(N)` 이다.

잠깐 `~lg N` 의 복잡도를 가지는 *Binary Search* 에 대해 이야기 해 보자.

#### Binary Search

이진트리에 대해 재밌는 사실이 하나 있다. 이진트리가 나온건 1946년인데, 놀랍게도 버그가 없는 버전은 1962년에 처음 나왔다. 그리고 자바의 `Arrays.binarySerach()` 도 2006년에 버그가 발견되었다. 아래의 코드에서 만약 `arr` 에 `[1]` 을 넣고, `key` 에 `1` 을 주면 어떻게 될지 한번 생각 해 보자.


```java
public static int binarySearch(int[] arr, int key) {
  int low = 0, high = arr.length - 1;
  
  while (low <= high) {
    int mid = low + (high - low) / 2;
    
    if (key < arr[mid]) high = mid - 1; // 3-way compares
    else if (key > arr[mid]) low = mid + 1;
    else return mid
  }
  
  return -1;
}
```

*binary search* 를 *2-way compare* 로 구현하면, 다시 말해 `if-else` 만 이용하면 `1 + lg N` 의 복잡도를 가지게 된다. 이는 `T(N) <= 1 + T(N/2)` , `T(N) <= 1 + 1 + T(N/4)`, ... `T(N) = 1 + 1 + ... + T(N/N)` 이기 때문이다.

*binary search* 가 `1 + lg N` 이라는 사실을 이용하면 *3-Sum* 을 `N^2 lg N` 으로 개선할 수 있다.

(1) sorting: `N^2`  
(2) binary search for `-(arr[i] + arr[j])`  : `N^2 log N`  

두번째 스탭에서 `i`, `j` 에 대해 이진탐색을 시도하므로 `N^2 * log N` 이다.

### Theory of Algorithms

이제 **Best case(lower bound on cost)**, **Worst case(upper bound on cost)**, **Average case(expected cost)** 를 고려 해 보자

이를 위해 새로운 *notation* 을 도입할 수 있다. 교수님이 해주시는 설명을 솔직히 못 알아 듣겠다. 그냥 *big theta*, *big omega*, *big oh* 에 대해 구글링 해서 나오는 [SO 답변](http://stackoverflow.com/questions/471199/what-is-the-difference-between-%CE%98n-and-on) 을 참조했다.

먼저 **asymptotic** 이란 말을 이해해야 하는데 한국어로 표현하면 *점근적* 정도가 된다. ~~m 은 묵음~~

> If an algorithm is of Θ(g(n)), it means that the running time of the algorithm as n (input size) gets larger is proportional to g(n).

> If an algorithm is of O(g(n)), it means that the running time of the algorithm as n gets larger is at most proportional to g(n).

> More technically, O(n) represents upper bound. Θ(n) means tight bound. Ω(n) represents lower bound.

우리가 어떤 알고리즘이 `O(n)` 이라고 할 때 `O(n^2)`, 혹은 `O(n^3)` 이라고 말할 수 있다. *upper bound* 니까. 하지만 어떤 알고리즘(`f(n)`) 이 `Θ(n)` 일때, `Θ(n^2)` 이라 말할 순 없다. 그리고 *Big-Oh* 와 *Big-theta* 가 존재 해야 *Big-omega* 가 존재한다.

> f(x) = Θ(g(x)) iff f(x) = O(g(x)) and f(x) = Ω(g(x))

이런 표기법을 이용하는 이유는, 다시 말해서 *Theory of algorithm* 의 목적은 *optimal algorithm* 을 찾는데 있다. 

*1-sum* 을 예로 들어 보자. *upper bound* 는, 모든 원소를 찾는 것이다. 따라서 `O(n)`.  적어도 모든 원소를 한번씩은 다 뒤져봐야 하므로, *lower bound* 는 `Ω(n)` 이다. 따라서 두개가 *constant factor* 내에서 같으므로 `Θ(n)` 이다.

*3-sum* 을 고려해 보자. *upper bound* 는 `O(n^2 * lg n)` 이다. *lower bound* 는 확실힌 모르겠지만 적어도 모든 원소를 한번씩은 훑어야 하므로 `Ω(n)` 다. [여기](http://cstheory.stackexchange.com/questions/14585/lower-bounds-for-3sum-with-a-free-cache) 보면 *3-sum* 의 *lower bound* 에 관한 논의가 있다.

따라서 *3-sum* 에 대한 확실한 *lower bound* 를 모르므로, *upper bound* 와의 갭이 있고, 아직까진 *optimal algorithm* 이 무엇인지 알 수 없다.

따라서 알고리즘을 디자인하는 접근 방법은,

(1) Develop an algorithm  
(2) Prove a lower bound

그리하여, *lower bound* 와 *upper bound* 사이의 갭이 있다면 *lower bound* 를 증가시키는 새로운 알고리즘이 여지가 있다. 그걸 찾아내면서 *lower bound* 를 올려가면 *optimal algorithm* 을 찾을 수 있다. 

1970년대에는 *upper bound* 를 줄여왔지만, 

(1) 너무 *worst case* 에만 집중헀고  
(2) 이제 더 정확한 성능 측정을 위해서 *to within a constant factor* 보다 더 좋은 무언가가 필요하다.

그리고 많은 사람들이 *Big o* 를 *approximate model of running time* 으로 번역하는 실수를 저질렀는데, 이 수업에서는 *Big o* 대신 *tilde(~)* 를 사용하겠다.

<img src="http://qlx.is.quoracdn.net/main-ce94d194dc6a85d3.png" align="center" />

<p align="center">If</p>

<img src="http://qlx.is.quoracdn.net/main-30ac99d6624fcf8c.png" align="center" />

### Memory

자바에서는 다음과 같은 오버헤드가 있다.

(1) `N` 개의 1차원 배열을 만들때는 *Type \* N + 24 Bytes*  
(2) *Object* 는 *16 Bytes*  
(3) *Reference* 는 *8 Bytes*  
(4) 각 오브젝트는 *8 Bytes* 단위로 패딩된다.

**String** 은 조금 더 복잡한데,

오브젝트 오버헤드, `char` 배열 오버헤드(`2N + 24`)와 함께 `offset`, `count`, `hash` 를 `int` 타입으로 가지므로 *12 Bytes* 와 패딩이 포함된다.

*inner class* 가 있다면 *Object* 에 *8 bytes* 의 메모리가 더 필요하다.

```java
public class WeightedQuickUnionUF {
  private int[] id;
  private int[] sz;
  private int count;
  
  ...
}
```

위 클래스를 인스턴스로 가지고 있을 경우엔 `N` 에 대해서

(1) 16 bytes *object overhead*  
(2) 8 + (4N + 24) for each *int[] array*  
(3) 4 bytes for `count`  
(4) 4 bytes padding

따라서 `8N + 88` 이므로 `~8 N` 이다.

### Summary

**Empirical analysis** 은 실제로 프로그램을 돌려 성능을 얻은 다음 가설을 새워 예측하는 방법이다. 반면 **Mathematical analysis** 는 알고리즘을 분석해 연산 횟수에 기반한 모델을 새우는 방법이다. 

> Emplirical analysis model enables us to **make predictions** and necessary to validate mathematical models

> Mathematical analysis model enables us to **explain behavior** and is independent of a particular system. 
