+++
date = "2016-06-25T12:54:47+09:00"
next = "../design-and-analysis-part1-3"
prev = "../design-and-analysis-part1-1"
title = "Design and Analysis: Randomized Selection"
toc = true
weight = 12
aliases = [
    "/randomized-selection"
]
+++

### Intuition

중복이 없는 `n` 개의 원소를 가진 배열에서 `i` 번째로 큰 원소를 얻고 싶다고 하자. 간단한 방법은 먼저 정렬을 한 뒤 거기서 `i` 번째 원소를 고르면 된다. 이 방법을 *reduction* 이라 부르는데 *selection* 문제를 *sorting* 문제로 바꾸어 푼 것이다. 이 경우 정렬에 머지소트를 사용한다면 `O(n logn)` 만큼의 시간이 걸릴 것이다.

*selection* 문제는 `O(n)` 시간 안에 *deterministic* 하게 해결할 수 있다. 지난시간에 잠깐 논의했던 *randomization* 을 이용하면 된다. 어떻게 그럴 수 있을까? 저기서 정렬을 더 개선할 수 없다는건 모두가 알고 있는 사실인데

*quick sort* 를 수정해서 *pivot* 을 *median of medians* 로 고르면 된다. ~~아니 의사양반 이게 무슨 개소리요!~~

더 정확히 말해서 이 문제는 **정렬 문제가 아니기 때문에** 더 개선할 여지가 있다. *pivot* `P` 를 기준으로 좌측이나 우측 한쪽만 선택하면 되는 *selection* 문제다.

*worst case* 는 당연히 매 재귀호출마다 문제 수가 1씩 줄어드는 경우이므로 `O(n^2)` 일테다. 만약에, *bast case* 로 문제가 절반씩 줄어든다면? *master method* 를 이용하면 `a = 1, b = 2, d = 1` 에서 `T(n) = O(n^1)` 이다.

![](https://acrocontext.files.wordpress.com/2014/01/master-method.png?w=300&h=160)

그럼 이제 문제는 어떻게 사이즈를 `1/2` 로, 더 정확히는 *median* 을 *pivot* 으로 삼느냐다.

### Analysis

*randomized selection* 문제를 풀기 위해 구현한 함수를 `rSelect` 라 하자. 매 재귀마다 문제 사이즈가 `n` 이라고 하면, 각 재귀에서의 `rSelect` 의 연산은 `c * n` 보다 작거나 같다. (`c` 는 상수)

이제 본격적인 분석전에  잠깐 *notation* 을 하나 만들고 가면 `phase j` 는 문제의 사이즈가 `(3/4)^j+1 * n` 과 `(3/4)^j * n` 사이에 있는 `rSelect` 다. 따라서 문제의 사이즈가 `n` 부터 `3/4` 가 되기 전까지의 모든 `rSelect` 는 `phase 0` 에 있다.

그리고 `Xj` 를 `phase j` 에 있는 `rSelect` 호출의 수라 정의하면

![](http://latex.codecogs.com/gif.latex?T%28n%29%20%5Cleq%20%5Csum_%7Bphase%20j%7D%20X_j%20*%20c%20*%20%28%7B3%20%5Cover%204%7D%29%5Ej%20*%20n)

이렇게 정의해 놓으면 재밌는 조건을 하나 쓸 수 있다. 바로 *pivot* 이 `25%-75%` 사이로 분할만 해주면, 다시 말해서 반으로 갈린 문제 중 작은 한쪽이 적어도 `25%` 가 넘으면 현재 *phase* 가 끝난다. 그럼 이제 전체 알고리즘의 기대값을 구하기 위해 *linearity of expectation* 을 이용해서 `E(Xj)` 를 구하면 된다. 

`25-75%` 로 피벗이 걸릴 확률 `P(25-75%) = 1/2` 이고 그럴때의 `Xj = 1` 이다. 반면 두번째에 피벗이 제대로 걸릴 확률은 `1/4` 이고, 세번째에 피벗이 제대로 걸릴 확률은 `1/2^3` 이다.

기대값은 이 모든 각각 확률변수값과 그 확률의 곱이므로 계산하면

![](http://latex.codecogs.com/gif.latex?%7B1%20%5Cover%202%7D%20&plus;%20%7B1%20%5Cover%202%5E2%7D%20&plus;%20%7B1%20%5Cover%202%5E3%7D%20&plus;%20%5Ccdots%20%5Cleq%202)

이것 말고 더 재밌는 계산법도 있다. 자세한 건 강의 내용을 참조 

![](http://latex.codecogs.com/gif.latex?E%28X_j%29%20%3D%201%20&plus;%20%7B1%20%5Cover%202%7D%20*%20E%28X_j%29)

이제 *average running* 타임을 구하기 위해 `T(n)` 의 평균을 구하면

![](http://latex.codecogs.com/gif.latex?T%28n%29%20%5Cleq%20E%5Bc%20*%20n%20*%20%5Csum_%7Bphase%20j%7D%20%28%7B3%20%5Cover%204%7D%29%5Ej%20*%20X_j%5D)

여기서 *linearity of expectation (기대값의 선형성)* 을 이용하면

![](http://latex.codecogs.com/gif.latex?T%28n%29%20%5Cleq%20c%20*%20n%20*%20%5Csum_%7Bphase%20j%7D%20%28%7B3%20%5Cover%204%7D%29%5Ej%20*%20E%28X_j%29)

![](http://latex.codecogs.com/gif.latex?T%28n%29%20%5Cleq%20c%20*%20n%20*%20%5Csum_%7Bphase%20j%7D%202%20*%20%28%7B3%20%5Cover%204%7D%29%5Ej)

무한급수 공식을 적용하면,

![](http://latex.codecogs.com/gif.latex?T%28n%29%20%5Cleq%20c%20*%20n%20*%204)

~~얼마나 멋진가?~~

### Deterministic Selection

만약에 *randomization* 을 이용할 수 없다면? 그럼 이제 문제는 *good pivot*, 즉 `50/50` 에 최대한 가깝게 잘라내는 *pivot* 을 찾아야 한다. *median of medians* 를 이용하면 해낼 수 있다.

*deterministic selection* 알고리즘을 구현한 함수를 `dSelect` 라 부르면

```
dSelect(array A, length n, order statistic i)
```

(1) Break `A` into groups of 5, sort each group  
(2) C = the `n/5` "middle elements"  
(3) p = `dSelect(C, n/5, n/10)`, recursivly computes median of C  
(4) Partition `A` around `p`  
(5) if `j = i` return `p`  
(6) if `j < i` return `dSelect(1st part of A, j-1, i)`  
(7) if `j > i` return `dSelect(2nd part of A, j-j, i-j)`  

`4-7` 스텝은 *randomized selection* 과 똑같다. 더 복잡해진 부분은 앞의 `1-3` 스텝에서 피벗을 고르는 일이다.

퍼포먼스를 다시 이야기 해 보자 *randomized selection* 은 *pivot* 이 정말 나쁘게 선택되면 `O(n^2)` 이 될 수 있다. 

반면 *deterministic selection* 은 모든 경우에 `O(n)` 을 보장한다. 그러나 실제로는 *randomized* 보다 성능이 나쁜데, 이유는 알고리즘에서 볼 수 있듯이 새로운 배열 `C` 가 필요하고 (*not in-place*), 표기법에는 상수가 생략되는데 *deterministic selection* 은 이 상수가 꽤나 커질 수 있다.

### Analysis

이제 좀 더 자세히 살펴보자.

(1) Break `A` into groups of 5, sort each group  

이건 얼마의 시간이 걸릴까? 주어진 배열을 5개씩 짜르고, 각각의 그룹을 정렬하는데 걸리는 시간은? `O(n)` 이다.

먼저 `n = 120` 이라 하자. 정렬에 *merge sort* 를 사용하면 *merge sort* 연산 수 공식은

![](http://latex.codecogs.com/gif.latex?6n%20*%20log_2%28n&plus;1%29)

따라서 잘려진 5개짜리를 정렬하는데 걸리는 시간은 `30 * log_2(6)` 에서, 이 값은 적어도 120 보다는 작음을 알 수 있다. 따라서 전체 그룹의개수 `n/5` 를 곱하면, `24n` 으로 `O(n)` 임을 알 수 있다. 비록 상수가 좀 크긴 하지만

(2) C = the `n/5` "middle elements"  
(3) p = `dSelect(C, n/5, n/10)`, recursivly computes median of C  
(4) Partition `A` around `p`  
(5) if `j = i` return `p`  
(6) if `j < i` return `dSelect(1st part of A, j-1, i)`  
(7) if `j > i` return `dSelect(2nd part of A, j-j, i-j)`  

(2), (4) 는 `O(n)` 임을 알 수 있고, (3) 은 `T(n/5)` 다. 문제는 (6), (7) 이다. 둘 중에 하나만 호출되긴 하지만 선택되는 *pivot* `p` 에 따라서 문제의 사이즈가 달라진다. 모르니까 `T(?)` 라 두자 그러면 *determinitic selection* 의 *running time* 은

![](http://latex.codecogs.com/gif.latex?T%28n%29%20%5Cleq%20cn%20&plus;%20T%28n/5%29%20&plus;%20T%28%3F%29)

간단한 가설을 세워보자. 

>**두번째 `dSelect` 호출의 input size 는 `7/10 * n` 보다 작거나 같다**

그러면 수식을 이렇게 바꿀 수 있다.

![](http://latex.codecogs.com/gif.latex?T%28n%29%20%5Cleq%20cn%20&plus;%20T%28n/5%29%20&plus;%20T%287n/10%29)

(2) 에서 *medians* 를 찾고, 이걸 (3)에서 재귀에 한번 더 넘기면 *median of medians* 을 찾게된다. 이게 어떤 효과가 있냐면, 모든 원소를 5개씩 짤라 아래에서 위로 정렬, *medians* 는 좌에서 우로 정렬하면 다음과 같은 행렬이 나오는데

![](http://i.imgur.com/gaOxb1A.jpg?1)<p align="center">(http://functionspace.org/articles/19)</p>

모든 원소 중 좌측 하단에 있는 `30%` 는 *median of medians* 보다 분명히 작다. 그리고 우측 상단 `30%` 는 *medians of medians* 보다 분명히 크다. 따라서 나머지 40% 값이 어쨌던건 간에 적어도 `30-70%` 분할은 해주므로 문제의 사이즈가 (6) 스텝에서 `7n/10` 보다 작거나 같다는 것을 분명히 보장해준다. 따라서 아래 식은 참이다.

![](http://latex.codecogs.com/gif.latex?T%28n%29%20%5Cleq%20cn%20&plus;%20T%28n/5%29%20&plus;%20T%287n/10%29)

쉽게 *master method* 를 이용하고 싶은데 문제가 서로 다른 사이즈로 분할되니까 사용할 수 없다. *induction* 을 이용하자. 아래가 참임을 보이면 된다.

![](http://latex.codecogs.com/gif.latex?T%28n%29%20%5Cleq%20an)

우선 *base case* 는 `T(1) = 1` 이므로 `T(1) <= a (where a >= 1)` 에서 참이다.

![](http://latex.codecogs.com/gif.latex?T%28n%29%20%5Cleq%20cn%20&plus;%20T%28n/5%29%20&plus;%20T%287n/10%29)

이제 위 식에서 *induction hypothesis* 를 이용하고, 정리하면

![](http://latex.codecogs.com/gif.latex?T%28n%29%20%5Cleq%20cn%20&plus;%20a%28n/5%29%20&plus;%20a%287n/10%29)

![](http://latex.codecogs.com/gif.latex?T%28n%29%20%5Cleq%20n%20*%20%289a/10%29)

이 때 `c` 는 상수이므로 `c = a / 10` 이라 하면 

![](http://latex.codecogs.com/gif.latex?T%28n%29%20%5Cleq%20an)

따라서 *deterministic selection* 의 성능은 `O(n)` 이다.

### lower bound for sorting

*comparison-based sorting* 의 *lower bound* 는 

![](http://latex.codecogs.com/gif.latex?%0A%5COmega%20(n*log%20n))

여기 해당되는 정렬들은 *merge sort, quick sort, heap sort* 등이 있다. 이런 정렬들은 데이터가 어떠할 것이라는 가정 없이 정렬을 해낸다. 

반면 데이터의 분포를 안다면 *bucket sort* 같은 경우 *linear time* 으로 해결할 수 있다. *counting sort* 나 *radix sort* 같은 정렬도 데이터에 대한 정보(정수)라는 것을 이미 알고 있는 경우이므로 `O(n)` 으로 정렬 가능하다.

데이터에 대한 정보를 모른다고 해 보자. `1, 2, ..., n` 까지의 데이터를 가지고 있다면 이 데이터들이 배열 안에 담겨있을 수 있는 경우의 수는 `n!` 이다.

`n!` 개의 모든 종류의 인풋에 대해서 `k` 번만큼, 혹은 그보다 더 적게 비교가 일어난다고 하자. 그럼 모든 `n!` 종류의 인풋에 대해서 `2^k` 개의 서로 다른 *execution* 이 생긴다.

> Suppose algorithm always makes <= k comparisons to correctly sort these `n!` inputs. Across all `n!` possile inputs algorithms exhibits <= `2^k` distinct executions

쉽게 생각해서 `k-bit` 문자열이 있을때 이걸로 얻을 수 있는 문자열은 `2^k` 개수다. 즉 어떤 문자는 없을수도 있다.

비둘기 집 원리를 생각해 보자. 우리는 `n!` 비둘기가 있고, `2^k` 개의 비둘기 집이 있다. 만약에 `k` 가 작아 `2^k < n!` 이면 서로 다른 두개의 인풋에 대해서 같은 종류의 *execution* 을 공유 한다는 뜻이다. 따라서 둘 중 하나는 제대로 정렬되고, 나머지 하나는 제대로 정렬되지 않는다. 

따라서 `2^k >= n!` 이다. 이때 

![](http://latex.codecogs.com/gif.latex?n%21%20%5Cgeq%20n%20*%20%28n-1%29%20*%20%28n-2%29%20%5Ccdots%20%28n/2%29%20%5Cgeq%20%28n/2%29%5E%7B%28n/2%29%7D)

이므로 

![](http://latex.codecogs.com/gif.latex?2%5Ek%20%5Cgeq%20%28n/2%29%5E%7Bn/2%7D)

![](http://latex.codecogs.com/gif.latex?k%20%5Cgeq%20%28n/2%29*%20log_2%28n/2%29)

![](http://latex.codecogs.com/gif.latex?k%20%5Cgeq%20%5COmega%28n*logn%29)

`k` 가 연산 수 이므로 *comparison-based sorting* 의 *lower bound* 는 `Omega(n logn)` 이다.

### References

(1) *Algorithms: Design and Analysis, Part 1* by **Tim Roughgarden**  
(2) [http://functionspace.org/articles/19](http://functionspace.org/articles/19)
