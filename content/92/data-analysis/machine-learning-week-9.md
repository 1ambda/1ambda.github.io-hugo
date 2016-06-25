+++
date = "2016-06-25T14:25:34+09:00"
next = "../machine-learning-week-10"
prev = "../machine-learning-week-8"
title = "ML 09: Anomaly Detection, Recommender System"
toc = true
weight = 19
aliases = [
    "/machine-learning-week-9"
]
+++

이번시간엔 *anomaly detection* 과 *recommender system* 을 배운다.

### Anomaly Dectection

![](http://img.my.csdn.net/uploads/201302/19/1361236753_7590.png)

![](http://img.my.csdn.net/uploads/201302/19/1361236757_2205.png)
<p align="center">(http://blog.csdn.net/linuxcumt1)</p>

*anomaly* 는 정상집단에서 떨어진 데이터라 보면 된다. 공장에서 품질이 떨어지는 제품을 골라낼때 사용할 수 있는데, 위 그림은 비행기 엔진 공장을 예로 들어 설명한다.

데이터로부터 `p(x)` 를 만들어, 검사할 데이터가 *threshold* 를 넘는지 안넘는지 검사해 *anomaly* 로 판정할 수 있다.

참고로, *anomaly* 가 너무 많으면, *false positive* 가 높은 것인데 이 때는  *threshold* 를 줄이면 된다.

![](http://img.my.csdn.net/uploads/201302/19/1361236761_2830.png)
<p align="center">(http://blog.csdn.net/linuxcumt1)</p>

*anomaly detection* 은 *fraud detection* 에 많이 사용된다. 데이터로부터 모델 `p(x)` 를 만들고 *unusual user* 를 검사하기 위해 `p(x) < e` 인지 검사하면 된다.

이외에도 항공기 엔진 예제처럼 제품의 품질 관리나, 데이터 센터에서의 노드 과부하 탐지등에 사용할 수 있다.

### Gaussian Distribution

![](http://img.my.csdn.net/uploads/201302/19/1361236829_8964.png)

![](http://img.my.csdn.net/uploads/201302/19/1361236829_8964.png)
<p align="center">(http://blog.csdn.net/linuxcumt1)</p>

*gaussian density* 공식은

![](http://latex.codecogs.com/gif.latex?P%28x%3B%20%5Cmu%2C%20%5Csigma%5E2%29%5C%5C%20%5C%5C%20%3D%20%7B1%20%5Cover%20%5Csqrt%7B2%5Cpi%5Csigma%5E2%7D%7D%20%5C%20%5Cexp%28-%20%7B%28x%20-%20%5Cmu%29%5E2%20%5Cover%202%5Csigma%5E2%7D%29)

![](http://img.my.csdn.net/uploads/201302/19/1361236839_1788.png)
<p align="center">(http://blog.csdn.net/linuxcumt1)</p>

평균과 분산은

![](http://latex.codecogs.com/gif.latex?%5Cmu%20%3D%20%7B1%20%5Cover%20m%7D%20%5C%20%5Csum_%7Bi%20%3D%201%7D%5Em%20x%5E%7B%28i%29%7D)

![](http://latex.codecogs.com/gif.latex?%5Csigma%5E2%20%3D%20%7B1%20%5Cover%20m%7D%20%5Csum_%7Bi%20%3D%201%7D%5Em%20%28x%5E%7B%28i%29%7D%20-%20%5Cmu%29)

<br/>

### Anomaly Detection Algorithm

![](http://img.my.csdn.net/uploads/201302/19/1361236899_7015.png)
<p align="center">(http://blog.csdn.net/linuxcumt1)</p>

각 *feature* 가 가우시안 분포를 따른다고 하면, 

![](http://latex.codecogs.com/gif.latex?p%28x%29%20%5C%5C%20%5C%5C%20%3D%20p%28x_1%3B%20%5Cmu_1%2C%20%5Csigma_1%5E2%29%5C%20p%28x_2%3B%20%5Cmu_2%2C%20%5Csigma_1%5E2%29%20%5Ccdots%5C%20p%28x_n%3B%20%5Cmu_n%2C%20%5Csigma_1%5En%29%20%5C%5C%20%5C%5C%20%3D%20%5Cprod_%7Bj%20%3D%201%7D%5En%20p%28x_j%3B%20%5Cmu_j%2C%20%5Csigma_j%5E2%29)

이렇게 가정하려면, 각 *feature* 가 독립적이어야 하지만 실제로는 독립적이지 않더라도 어느정도 동작한다. 이 때 

![](http://latex.codecogs.com/gif.latex?%5Cmu_j%20%3D%20%7B1%20%5Cover%20m%7D%20%5Csum_%7Bi%20%3D%201%7D%5Em%20x_j%5E%7B%28i%29%7D)

![](http://latex.codecogs.com/gif.latex?%5Csigma_j%5E2%20%3D%20%7B1%20%5Cover%20m%7D%20%5Csum_%7Bi%20%3D%201%7D%5Em%20%7B%28x_j%5E%7B%28i%29%7D%20-%20%5Cmu_j%29%5E2%7D)

![](http://img.my.csdn.net/uploads/201302/19/1361236904_6921.png)
<p align="center">(http://blog.csdn.net/linuxcumt1)</p>

따라서 `p(x)` 는 아래 식이 된다. `p(x)` 는 *feature* 가 나올 확률로 이해하면 된다. 이 때 `p(x)` 가 상당히 작으면, 평균에 가깝지 않은 *feature* 가 많이 나왔다는 뜻이므로 *anomaly* 라 볼 수 있다.

![](http://latex.codecogs.com/gif.latex?p%28x%29%20%5C%5C%20%5C%5C%20%3D%20%5Cprod_%7Bj%3D1%7D%5En%20%5C%20%7B1%20%5Cover%20%5Csqrt%7B2%5Cpi%5Csigma_j%5E2%7D%7D%20%5C%20%5Cexp%28-%7B%28x_j%20-%20%5Cmu_j%29%5E2%20%5Cover%202%5Csigma_j%5E2%7D%29)

<br/>

![](http://img.my.csdn.net/uploads/201302/19/1361236907_7102.png)
<p align="center">(http://blog.csdn.net/linuxcumt1)</p>

두 *feature* `x1, x2` 의 가우시안 분포를 3차원으로 조합하면 `p(x)` 가 좌측 하단 3차원 원뿔의 높이가 된다.

### Evaluating Anomaly Detection

![](http://img.my.csdn.net/uploads/201302/19/1361236992_3664.png)

![](http://img.my.csdn.net/uploads/201302/19/1361236996_4034.png)
<p align="center">(http://blog.csdn.net/linuxcumt1)</p>

*anomaly* 를 잘 나타낼거 같은 *feature* 를 골라내고, 이를 이용해 모델을 만든다. 

우리가 가진 데이터가 *anomaly* 를 알려주는 `y` 가 있다면, 위 그림처럼 *training set* 으로 *non-anomalous* 을 이용하고, *CV, Test Set* 으로 나머지를 반반씩 분할하면 된다.

즉 *good example* 로 모델을 만들고, *anomaly* 가 섞여있는 *cv, test set* 으로 평가한다.

![](http://img.my.csdn.net/uploads/201302/19/1361237001_5250.png)
<p align="center">(http://blog.csdn.net/linuxcumt1)</p>

이 때 *skewed classess* 이기 때문에 (`y = 0` 이 대다수, `y = 1` 은 희박) 단순히 정확도로 평가하긴 좀 무리가 있다. *precision, recall, f1 score* 등을 이용해 평가해야 한다.

*threshold* 인 `e` (엡실론) 를 고르기 위해 *cross validation* 을 이용할 수 있다. *f1 score* 를 최대화 하는 `e` 를 고른다거나.

### Anomaly Dectection vs Supervised Learning

`y` 값이 있는 데이터라면, 왜 *supervised learning* 을 이용하지 않을까? 

![](http://img.my.csdn.net/uploads/201302/19/1361242897_8389.png)
<p align="center">(http://blog.csdn.net/linuxcumt1)</p>

#### Anomaly Detection

*anomaly detection* *skewed class* 가 있을 때 사용한다.

> Many different **types** of anomalies. Hard for any algorithm to learn from positive examples what the anomalies look like

> Future anomalies may look nothing like any of the anomalous examples we've seen so far

보면 알겠지만 *anomaly* 가 굉장히 다양할 수 있기 때문에 *anomaly* 를 특정 형태로 구분짓는 알고리즘을 쓰긴 좀 힘들다.

게다가, 가지고 있는 데이터 셋에서 보지 못했던 새로운 종류의 *anomaly* 가 나올 수도 있다.

#### Supervised Learning

*positive, negative example* 이 많을 때 사용한다.

> Enough positive examples for algorithms to get a sense of what positive examples are like, futre positive example likly to be similar to ones in training set

*supervised learning* 에서 *positive example* 은 어떤 특정 형태기 때문에, 미래에 발견할 *positive example* 도 비슷한 형태라 생각될 때 사용한다.

*SPAM filtering* 에서는 다양한 타입의 *positive example* 이 있어도, 우리가 충분한 양의 *positive example* 이 있기 때문에 커버할 수 있어 *supervised learning* 을 사용한다.

<br/>

![](http://img.my.csdn.net/uploads/201302/19/1361243087_2169.png)
<p align="center">(http://blog.csdn.net/linuxcumt1)</p>

<br/>

### Choosing What Features to Use

![](http://img.my.csdn.net/uploads/201302/19/1361244210_3429.png)
<p align="center">(http://blog.csdn.net/linuxcumt1)</p>

*feature* 의 분포가 가우시안이면 고맙지만, 아닐경우 변환이 필요하다. 왼쪽 아래 분포에 로그를 씌우면, 가우시안 분포 비슷하게 보인다.

다른 방법으로는 `log(x_2 + c)`, `sqrt(x_3)` 등등이 있다.

![](http://img.my.csdn.net/uploads/201302/19/1361245473_5316.png)

흔한 에러는 `p(x)` 가 *normal, anomalous* 에 대해서 모두 높은 경우인데, 슬라이드의 아래쪽에서 볼 수 있듯이 `x2` 라는 *feature* 를 만들어서 *anomaly* 를 발견하는 알고리즘을 만들 수 있다.

![](http://img.my.csdn.net/uploads/201302/19/1361246077_9679.png)
<p align="center">(http://blog.csdn.net/linuxcumt1)</p>

*anomaly* 를 위한 *feature* 를 고를 때 특이하게 높거나, 낮을 수 있는 것을 고르면 된다. 데이터 센터 예제에서는 *CPU load / network traffic* 등이 있을 수 있다. 네트워크 트래픽이 낮은데 *CPU load* 가 높다면 확실히 *anomaly* 기 때문이다.

### Multivariate Gaussian Distribution

![](http://img.my.csdn.net/uploads/201302/19/1361257865_7961.png)
<p align="center">(http://blog.csdn.net/linuxcumt1)</p>

*feature* 를 *CPU laod, memory use* 로 했을 때 낮은 CPU 부하에도 메모리 사용량이 높으면 *anomaly* 라 볼 수 있다.

그런데, 슬라이드의 왼쪽 그림에서 녹색으로 표시한 *anomaly* 는 지금까지 설명했던 알고리즘으로 찾기가 힘들다. 적당한 수준의 *memory use* 와 그리 낮지 않은 *cpu load* 를 가지기 때문이다.

실제 *normal example* 이 타원형이기 때문에, 원으로 *anomaly* 를 찾기는 어렵다. 

![](http://img.my.csdn.net/uploads/201302/19/1361258533_5107.png)

따라서 `p(x_1)p(x_2)...` 을 이용한 모델 말고 다른 방법으로 모델을 만들어야 한다. 

`u` 를 `n` 벡터라 하고, `Sigma` 를 `u` 의 *convariance matrix* 라 하자. 그러면

![](http://latex.codecogs.com/gif.latex?p%28x%3B%20%5Cmu%2C%20%5CSigma%29%20%5C%5C%20%5C%5C%20%3D%20%7B1%20%5Cover%20%282%5Cpi%29%5E%7Bn/2%7D%20%5C%20%7C%5CSigma%7C%5E%7B1/2%7D%7D%20%5C%20%5Cexp%28-%7B1%5Cover%202%7D%28x%20-%20%5Cmu%29%5ET%20%5C%20%5CSigma%5E%7B-1%7D%28x%20-%20%5Cmu%29%29)

여기서 `|Sigma|` 는 `Sigma` 의 행렬식인데, 여기를 참고하자.

- [행렬](http://ghebook.blogspot.com/2011/06/matrix.html)
- [행렬식](http://ghebook.blogspot.com/2011/06/determinant.html)
- [행렬식의 기하학적 의미](http://ghebook.blogspot.kr/2011/06/geometric-meaning-of-determinant.html)
- [행렬식과 기하학적 활용](http://darkpgmr.tistory.com/104)

이제 위 식을 이용해서 나온 `p(x)` 를 3차원, 2차원으로 보면

![](http://img.my.csdn.net/uploads/201302/19/1361259228_7695.png)

![](http://img.my.csdn.net/uploads/201302/19/1361259243_2967.png)

![](http://img.my.csdn.net/uploads/201302/19/1361259236_1052.png)

<br/>

![](http://img.my.csdn.net/uploads/201302/19/1361259583_5151.png)
<p align="center">(http://blog.csdn.net/linuxcumt1)</p>

![](http://latex.codecogs.com/gif.latex?%5Cmu%20%3D%20%7B1%20%5Cover%20m%7D%20%5Csum_%7Bi%20%3D%201%7D%5Em%20x%5E%7B%28i%29%7D)

![](http://latex.codecogs.com/gif.latex?%5CSigma%20%3D%20%7B1%20%5Cover%20m%7D%20%5Csum_%7Bi%3D1%7D%5Em%20%5C%20%28x%5E%7B%28i%29%7D%20-%20%5Cmu%29%28x%5E%7B%28i%29%7D%20-%20%5Cmu%29%5ET)

<br/>

![](http://img.my.csdn.net/uploads/201302/19/1361259728_6035.png)

`u, Sigma` 를 찾아 `p(x)` 를 만들고, 테스트 데이터에 대해 `p(x) < e` 인지 비교한다.

![](http://img.my.csdn.net/uploads/201302/19/1361260300_1768.png)

*original model* 은 *multivariate model* 에서 각 *feature* 간 상관 관계가 없는 (독립), 즉 *covariance matrix* 가 *diagonal matrix* 인 경우다. (

![](http://img.my.csdn.net/uploads/201302/19/1361260755_3407.png)
<p align="center">(http://blog.csdn.net/linuxcumt1)</p>

- **Original model**

수동으로 *feature* 를 만들때 사용할 수 있다. 또는 적은 연산을 원할때, 다시 말해서 `n` 이 커서 연산이 무지막지하게 클 때 좋다.

`m` 이 작아도 쓸 수 있다.

- **Multivariate Gaussian**

계산 비용이 비싸지만, 자동으로 *feature* 간 상관관계를 모델에 포함시킨다.

`Sigma` 가 *invertible* 이기 위해서는 `m > n` 이어야 한다. 실제로는 `m` 이 `n` 보다 훨씬 클 때 사용하는 경우가 많다. (e.g. `m >= 10n`)

만약에 `m > n` 인데, `Sigma` 가 *non-invertible* 이면 *redundant feature* 가 있는 경우니 확인해 보자. (흔한 오류라고 함)

### Recommender System

![](http://img.my.csdn.net/uploads/201302/20/1361324993_7588.png)

### Content Based Recommendations

![](http://img.my.csdn.net/uploads/201302/20/1361325560_4034.png)
<p align="center">(http://blog.csdn.net/linuxcumt1)</p>

위 슬라이드는 유저 `j` 로 부터 `theta^(j)` 를 얻어, *feature* `x` 와 곱함으로써 *linear regression* 문제로 변경했다.

![](http://img.my.csdn.net/uploads/201302/20/1361326084_2070.png)

`theta^(j)` 는 어떻게 훈련시킬까?

![](http://latex.codecogs.com/gif.latex?min_%7B%5Ctheta%5E%7B%28j%29%7D%7D%20%5C%20%5Csum_%7Bi%3A%20%5C%20%28ri%2C%20j%29%20%3D%201%20%7D%20%7B1%20%5Cover%202m%5E%7B%28j%29%7D%7D%5C%20%5B%28%5Ctheta%5E%7B%28j%29%7D%29%5ET%28x%5E%7B%28i%29%7D%29%20-%20y%5E%7B%28i%2C%20j%29%7D%5D%5E2%20%5C%20&plus;%20%7B%5Clambda%20%5Cover%202m%5E%7B%28j%29%7D%7D%5Csum_%7Bk%3D1%7D%5En%28%5Ctheta_k%5E%7B%28j%29%7D%29%5E2)

여기서 `m^(j)` 는 유저 `j` 에 의해 점수를 받은 영화의 수인데, 어차피 상수이므로 제거하면

![](http://latex.codecogs.com/gif.latex?min_%7B%5Ctheta%5E%7B%28j%29%7D%7D%20%5C%20%5Csum_%7Bi%3A%20%5C%20%28ri%2C%20j%29%20%3D%201%20%7D%20%7B1%20%5Cover%202%7D%5C%20%5B%28%5Ctheta%5E%7B%28j%29%7D%29%5ET%28x%5E%7B%28i%29%7D%29%20-%20y%5E%7B%28i%2C%20j%29%7D%5D%5E2%20%5C%20&plus;%20%7B%5Clambda%20%5Cover%202%7D%5Csum_%7Bk%3D1%7D%5En%28%5Ctheta_k%5E%7B%28j%29%7D%29%5E2)

![](http://img.my.csdn.net/uploads/201302/20/1361326247_4648.png)
<p align="center">(http://blog.csdn.net/linuxcumt1)</p>

이 때 각 유저마다의 `theta(j)` 를 합 해 최소화 시키는 방식으로 전체 `theta` 를 훈련시킬 수 있다.

![](http://latex.codecogs.com/gif.latex?min_%7B%5Ctheta%5E%7B%28j%29%7D%2C%20%5Ccdots%20%5Ctheta%5E%7B%28n_u%29%7D%7D%20%5C%5C%20%5C%5C%20%3D%20%7B1%20%5Cover%202%7D%5Csum_%7Bj%3D1%7D%5E%7Bn_u%7D%20%5Csum_%7Bi%3A%20%5C%20%28ri%2C%20j%29%20%3D%201%20%7D%20%5B%28%5Ctheta%5E%7B%28j%29%7D%29%5ET%28x%5E%7B%28i%29%7D%29%20-%20y%5E%7B%28i%2C%20j%29%7D%5D%5E2%20%5C%20&plus;%20%7B%5Clambda%20%5Cover%202%7D%5Csum_%7Bj%3D1%7D%5E%7Bn_u%7D%5Csum_%7Bk%3D1%7D%5En%28%5Ctheta_k%5E%7B%28j%29%7D%29%5E2)

![](http://img.my.csdn.net/uploads/201302/20/1361326573_9477.png)
<p align="center">(http://blog.csdn.net/linuxcumt1)</p>

*gradient descent* 는

![](http://latex.codecogs.com/gif.latex?%5Ctheta_k%5E%7B%28j%29%7D%20%3A%3D%20%5Ctheta_k%5E%7B%28j%29%7D%20-%20%5Calpha%5Csum_%7Bi%3A%5C%20r%28i%2C%20j%29%20%3D%201%7D%20%28%28%5Ctheta%5E%7B%28j%29%7D%29%5ET%28x%5E%7B%28i%29%7D%29%20-%20y%5E%7B%28i%2C%20j%29%7D%29x_k%5E%7B%28i%29%7D%20%5C%20%28for%5C%20k%20%3D%200%29)

![](http://latex.codecogs.com/gif.latex?%5Ctheta_k%5E%7B%28j%29%7D%20%3A%3D%20%5Ctheta_k%5E%7B%28j%29%7D%20-%20%5Calpha%5Csum_%7Bi%3A%5C%20r%28i%2C%20j%29%20%3D%201%7D%20%28%28%5Ctheta%5E%7B%28j%29%7D%29%5ET%28x%5E%7B%28i%29%7D%29%20-%20y%5E%7B%28i%2C%20j%29%7D%29x_k%5E%7B%28i%29%7D%20&plus;%20%5Clambda%5Ctheta_k%5E%7B%28j%29%7D%5C%20%28for%5C%20k%20%5Cneq%200%29)

### Collaborative Filtering

![](http://img.my.csdn.net/uploads/201302/20/1361327928_4438.png)
<p align="center">(http://blog.csdn.net/linuxcumt1)</p>

*content-based recommendation* 에서 *feature* 를 구하긴 사실 어려운 일이다. 누가 이 영화가 얼마만큼 로맨스고, 아닌지를 판별해줄까? 

문제를 좀 변경해서, 만약에 유저로부터 `theta(j)` 를 얻어낼 수 있다면 그 정보로 부터 *feature* `x(i)` 를 추출할 수 있다. 왜냐하면 `(\theta^(j))^T * x^(i) ~ y^(i, j)` 이기 때문이다.

![](http://img.my.csdn.net/uploads/201302/20/1361328443_4320.png)

`x^(i)` 를 얻기 위해, 

![](http://latex.codecogs.com/gif.latex?min_%7Bx%5E%7B%28j%29%7D%2C%20%5Ccdots%20x%5E%7B%28n_m%29%7D%7D%20%5C%5C%20%5C%5C%20%3D%20%7B1%20%5Cover%202%7D%5Csum_%7Bi%3D1%7D%5E%7Bn_m%7D%20%5Csum_%7Bi%3A%20%5C%20r%28i%2C%20j%29%20%3D%201%20%7D%20%5B%28%5Ctheta%5E%7B%28j%29%7D%29%5ET%28x%5E%7B%28i%29%7D%29%20-%20y%5E%7B%28i%2C%20j%29%7D%5D%5E2%20%5C%20&plus;%20%7B%5Clambda%20%5Cover%202%7D%5Csum_%7Bi%3D1%7D%5E%7Bn_m%7D%5Csum_%7Bk%3D1%7D%5En%28x_k%5E%7B%28i%29%7D%29%5E2)

![](http://img.my.csdn.net/uploads/201302/20/1361330430_7394.png)
<p align="center">(http://blog.csdn.net/linuxcumt1)</p>

- `theta` 가 주어지면 `x` 를 훈련할 수 있고
- `x` 가 주어지면 `theta` 를 훈련할 수 있다.

따라서 최초의 랜덤 `theta` 에 대해 `x` 를 훈련하고, 다시 `theta` 를 훈련하고, 반복하면 된다. 

![](http://img.my.csdn.net/uploads/201302/22/1361495687_3476.jpg)

`theta` 와 `x` 를 반복해서 훈련시키는 것보다, 동시에 훈련시키는 것이 좀 더 효율적이다. 따라서

![](http://latex.codecogs.com/gif.latex?J%28%7Bx%5E%7B%28j%29%7D%2C%20%5Ccdots%20x%5E%7B%28n_m%29%7D%2C%20%5Ctheta%5E%7B%28i%29%7D%2C%20%5Ccdots%20%5Ctheta%5E%7B%28n_u%29%7D%7D%29%20%5C%5C%20%5C%5C%20%3D%20%7B1%20%5Cover%202%7D%5Csum_%7B%28i%2C%20j%29%3A%20%5C%20r%28i%2C%20j%29%20%3D%201%20%7D%20%5B%28%5Ctheta%5E%7B%28j%29%7D%29%5ET%28x%5E%7B%28i%29%7D%29%20-%20y%5E%7B%28i%2C%20j%29%7D%5D%5E2%20%5C%20&plus;%20%7B%5Clambda%20%5Cover%202%7D%5Csum_%7Bi%3D1%7D%5E%7Bn_m%7D%5Csum_%7Bk%3D1%7D%5En%28x_k%5E%7B%28i%29%7D%29%5E2%20&plus;%20%7B%5Clambda%20%5Cover%202%7D%5Csum_%7Bj%3D1%7D%5E%7Bn_u%7D%5Csum_%7Bk%3D1%7D%5En%28%5Ctheta_k%5E%7B%28j%29%7D%29%5E2)

를 최소화 시키면 된다. 참고로 `x_0` 은 *collaborative filtering* 에서 필요가 없다. 알고리즘 자체가 *feature* 를 직접 찾아내니 *hard coded* 된 *feature* 는 사용하지 않는다.

![](http://img.my.csdn.net/uploads/201302/22/1361495692_7530.jpg)
<p align="center">(http://blog.csdn.net/linuxcumt1)</p>

(1) `x`, `theta` 를 작은 값으로 초기화 한다.

이는 *symmetry breaking* 을 하기 위함이다. 작은 랜덤값들로 초기화 하여 `x^(i)` 가 서로 다른 값들을 가지도록 도와준다.

(2) *cost function* `J` 를 *gradient descent* 등으로 최소화 시킨다.

![](http://latex.codecogs.com/gif.latex?x_k%5E%7B%28i%29%7D%20%3A%3D%20x_k%5E%7B%28i%29%7D%20-%20%5Calpha%20%5Csum_%7Bj%3A%5C%20r%28i%2C%20j%29%20%3D%201%7D%5B%28%5Ctheta%5E%7B%28j%29%7D%29%5ET%28x%5E%7B%28i%29%7D%29%20-%20y%5E%7B%28i%2C%20j%29%7D%5D%5Ctheta_k%5E%7B%28j%29%7D%20&plus;%20%5Clambda%20x_k%5E%7B%28i%29%7D)

![](http://latex.codecogs.com/gif.latex?%5Ctheta_k%5E%7B%28i%29%7D%20%3A%3D%20%5Ctheta_k%5E%7B%28i%29%7D%20-%20%5Calpha%20%5Csum_%7Bi%3A%5C%20r%28i%2C%20j%29%20%3D%201%7D%5B%28%5Ctheta%5E%7B%28j%29%7D%29%5ET%28x%5E%7B%28i%29%7D%29%20-%20y%5E%7B%28i%2C%20j%29%7D%5D%5Cx_k%5E%7B%28i%29%7D%20&plus;%20%5Clambda%20%5Ctheta_k%5E%7B%28j%29%7D)

(3) 유저의 *parameter* `theta` 와 영화의 *feature* `x` 에 대해 `theta^T * x` 를 이용해 예측하면 된다.

### Vectorization: Low Rank Matrix Factorization

![](http://img.my.csdn.net/uploads/201302/22/1361496844_8727.jpg)

![](http://img.my.csdn.net/uploads/201302/22/1361496849_5252.jpg)

*collaborative filtering* 은 *low rank matrix factoriazation* 이라 부르기도 한다. 위 슬라이드처럼 `X, THETA` 를 구성하고 `X * THETA^T` 를 구하면 된다.

![](http://img.my.csdn.net/uploads/201302/22/1361496854_2443.jpg)
<p align="center">(http://blog.csdn.net/linuxcumt1)</p>

*low rank matrix factorization* 을 이용해서 *feature* 를 찾으면, 두 영화 `i, j` 가 얼마나 유사한지 `||x^(i) - x^(j)||` 으로 판단할 수 있다.

### Implementation Detail: Mean Normalization

![](http://img.my.csdn.net/uploads/201302/22/1361497832_3797.jpg)

만약 위 슬라이드의 `Eve` 처럼 아무 영화도 평가 안한 사람에게는, `theta` 가 `0` 으로 나온다. (첫번째 *term* 이 `0` 이고, *regularization term* 은 `theta` 를 최소화한다.)

그렇게 되면, 어떤 영화도 높은 *rating* 을 받을 수 없으므로 (`theta^T * x`). 추천할 거리가 없다. 이건 별로 좋은 상황이 아닌데, *mean normalization* 을 이용하면 이 문제를 해결할 수 있다.

![](http://img.my.csdn.net/uploads/201302/22/1361497813_3878.jpg)
<p align="center">(http://blog.csdn.net/linuxcumt1)</p>

*mean normalized* 데이터를 이용하면, 추천 안한 사람이 `theta = 0` 을 갖더라도, 남들이 추천한 선호도 `u` 에 따라서 영화를 추천받을 수 있다.

![](http://latex.codecogs.com/gif.latex?%28%5Ctheta%5E%7B%28j%29%7D%29%5ET%28x%5E%7B%28i%29%7D%29%20&plus;%20%5Cmu_i)

잘보면 *feature scaling* 과는 다르게 특정 *range* 로 나누질 않는데, 이건 이미 *rating* 자체가 일정 범위 `1-5` 를 갖기 때문이다.

### References

(1) *Machine Learning* by **Andrew NG**  
(2) [http://blog.csdn.net/linuxcumt](http://blog.csdn.net/linuxcumt)  
(3) [http://blog.csdn.net/abcjennifer](http://blog.csdn.net/abcjennifer)  
(4) [http://ghebook.blogspot.com](http://ghebook.blogspot.com/2011/06/matrix.html)  
(5) [http://darkpgmr.tistory.com](http://darkpgmr.tistory.com/104)  
