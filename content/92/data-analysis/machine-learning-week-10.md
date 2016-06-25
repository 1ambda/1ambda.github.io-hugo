+++
date = "2016-06-25T14:25:35+09:00"
prev = "../machine-learning-week-9"
title = "ML 10: Stochastic Gradient, Synthetic Data, Ceiling Analysis"
toc = true
weight = 20
aliases = [
    "/machine-learning-week-10"
]
+++

이번 주에는 *mini-batch, stochastic graident descent*, *online learning*, *map-reduce* 등의 개념에 대해 배운다.

### Learning With Large Datasets

![](http://img.my.csdn.net/uploads/201302/22/1361499744_2717.jpg)
<p align="center">(http://blog.csdn.net/linuxcumt)</p>

왜 그렇게 큰 데이터 셋이 필요할까? 좋은 퍼포먼스를 얻기 위한 한 가지 방법이, *low bias* 알고리즘에 *massive data* 를 활용해 훈련하는 것이기 때문이다.


![](http://img.my.csdn.net/uploads/201302/22/1361499747_9327.jpg)
<p align="center">(http://blog.csdn.net/linuxcumt)</p>

그러나 커다란 데이터 셋을 연산하기 위해서는 연산비용이 정말 비싸다. 예를 들어 
`m = 100,000,000` 이라 하면 *gradient* 를 계산하기 위해 매번 `k * m` 의 연산이 필요하다.

따라서 모든 데이터를 이용해 알고리즘을 훈련하기 보다는, 랜덤하게 고른 작은 서브셋에 대해서 알고리즘을 개발하고, 이후에 전체 데이터에 대해서 훈련하는 방법을 쓰기도 한다.

그러면 `m` 이 작아도 알고리즘이 충분히 잘 훈련된다는 것을 어떻게 보장할까? 이는 *learning curve* 를 그려보면 된다.

위 슬라이드에서 우측 하단은 *high bias* 알고리즘인데 `m` 을 많이 투입해도 별다른 이득이 없으므로 적당한 수준에서 `m` 을 정하면 된다.

물론 만든 알고리즘이 우측처럼 *high bias* 로 나오면, 좀 더 자연스러운 생각은 *hidden layer* 를 추가한다거나, 새로운 *feature* 를 도입해서 *bias* 를 낮추는 것이다.

<br/>

### Stochstic Gradient Descent

![](http://img.my.csdn.net/uploads/201302/22/1361500908_4667.jpg)
<p align="center">(http://blog.csdn.net/linuxcumt)</p>

*gradient descent* 를 이용하는 *linear regression* 에서

![](http://latex.codecogs.com/gif.latex?h_%5Ctheta%28x%29%20%3D%20%5Csum_%7Bj%3D0%7D%5En%20%5C%20%5Ctheta_jx_j)

![](http://latex.codecogs.com/gif.latex?J_%7Btrain%7D%28%5Ctheta%29%20%3D%20%7B1%20%5Cover%202m%7D%20%5Csum_%7Bi%3D1%7D%5Em%5C%20%28h_%5Ctheta%28x%29%5E%7B%28i%29%7D%20-%20y%5E%7B%28i%29%7D%29%5E2)

이미 언급했듯이 *batch gradient descent* 의 문제는, `m` 이 크면 `J` 의 연산이 어마어마하게 많아진다는 것이다. 매 스텝마다 `m` 을 읽고, 계산값을 저장하기 때문이다.

![](http://img.my.csdn.net/uploads/201302/22/1361501694_2445.jpg)
<p align="center">(http://blog.csdn.net/linuxcumt)</p>

이 문제를 해결하기 위해 *stochastic gradient descent* 에서는 한 턴에 한 쌍의 `x, y` 에 대해서만 *gradient* 를 계산한다.

*batch gradient decsent* 에서는 한 턴마다 모든 모든 쌍의 `x, y` 에 대해	 *gradient* (= `theta_j`) 를 계산 했었다.

![](http://latex.codecogs.com/gif.latex?cost%28%5Ctheta%2C%20%28x%5E%7B%28i%29%7D%2C%20y%5E%7B%28i%29%7D%29%29%20%3D%20%7B1%20%5Cover%202%7D%5C%20%28h_%5Ctheta%28x%5E%7B%28i%29%7D%20-%20y%5E%7B%28i%29%7D%29%5E2)

![](http://latex.codecogs.com/gif.latex?J_%7Btrain%7D%28%5Ctheta%29%20%3D%20%7B1%20%5Cover%20m%7D%20%5Csum_%7Bi%3D1%7D%5Em%20cost%28%5Ctheta%2C%20%28x%5E%7B%28i%29%7D%2C%20y%5E%7B%28i%29%7D%29%29)

라고 하면

```
- Randomly shuffle dataset  
- Repeat for i = 1 to m
  - 0_j := 0_j - a * derivative of cost(0, (xi, yi)
```

즉 `J_train` 의 미분 대신에 `cost` 의 미분값을 이용해서 연산을 줄이는 방법이다. 이 때 데이터가 이미 랜덤하게 섞였다는 점을 기억하자. 기하학적으로 보면

![](http://img.my.csdn.net/uploads/201302/22/1361502040_9252.jpg)
<p align="center">(http://blog.csdn.net/linuxcumt)</p>

*batch* 에서는 올바른 방향을 향해 가지만, 보폭이 좀 좁다. *stochastic* 은 이리 갔다, 저리 갔다 하지만 결국에는 최저점을 향해 간다. 다만 *global optima* 에 도달하지 못하고 그 근처에 도착할 수 있다.

`m` 이 굉장히 크면, *repeat* 부분의 루프를 1번만 돌려도 될 테지만, 작으면 여러번 돌려서 최대한 좋은 퍼포먼스를 내도록 훈련시킬 수 있다.

<br/>

### Min-Batch Gradient Descent

![](http://img.my.csdn.net/uploads/201302/22/1361503946_9357.jpg)
<p align="center">(http://blog.csdn.net/linuxcumt)</p>

- **Batch gradient descent:** Use all `m` examples in each iteration
- **Stochastic gradient descent:** Use `1` example in each iteration
- **Batch gradient descent:** Use `b` examples in each iteration

![](http://latex.codecogs.com/gif.latex?%5Ctheta_j%20%3A%3D%20%5Ctheta-j%20-%20%7B%5Calpha%20%5Cover%20b%7D%20%5Csum_%7Bk%20%3D%20i%7D%5E%7Bi%20&plus;%20b%20-%201%7D%20%28h_%5Ctheta%28x%5E%7B%28k%29%7D%29%20-%20y%5E%7B%28k%29%7D%29%20x_j%5E%7B%28k%29%7D)

![](http://img.my.csdn.net/uploads/201302/22/1361504164_4561.jpg)
<p align="center">(http://blog.csdn.net/linuxcumt)</p>

`b` 는 보통 `2 - 100` 사이의 값이기 때문에 *batch* 보다 훨씬 빠르다. 또한 *vectorization* 을 이용하면 *gradient computation* 을 *partially parallelize* 할 수 있기 때문에 *stochastic* 보다 더 빠를 수 있다. ~~병렬화의 미덕~~

단점으로는 추가적인 파라미터 `b` 가 필요하다는 점이다. 그러나 *vectorization* 을 잘 이용하면 더 빨라지므로 문제 없다.

<br/>

### Stochastic Gradient Descent Convergence

![](http://img.my.csdn.net/uploads/201302/22/1361519060_5993.jpg)
<p align="center">(http://blog.csdn.net/linuxcumt)</p>

*convergence* 를 검증하는 방법으로, 훈련시키는 동안 얻은 `cost(0, (xi yi)` 평균값을 이용해 플롯을 그릴 수 있다. 	

![](http://img.my.csdn.net/uploads/201302/22/1361519096_5186.jpg)
<p align="center">(http://blog.csdn.net/linuxcumt)</p>

*stochastic* 은 *global optimum* 을 찾아내지 못할 수도 있기 때문에, 그 주변에서 알짱거릴 수도 있다.

더 많은 `m` 을 투입하면, 까끌거리는 선보다 좀 매끄러운 곡선을 얻을 수도 있다.

때때로 알고리즘이 전혀 학습하지 못하는 것 처럼 보일수도 있는데, 그럴 경우 `m` 을 더 투입하면 좀 경사가 낮은 커브로 조금씩 *decreasing* 할 수 있다. 이를 보면 결국 훈련되긴 하는데, 평균값을 플랏으로 그리니 들쭉날쭉 해 보이는 것이다. (물론 학습하지 못하는 경우도 있다. `m` 을 더 늘려서 확인해 보자.)

`cost` 값이 증가한다면 더 작은 *learning late* 값을 이용하자.

![](http://img.my.csdn.net/uploads/201302/22/1361519184_7199.jpg)
<p align="center">(http://blog.csdn.net/linuxcumt)</p>

*learning rate* 와 관련해서 위 슬라이드처럼 식을 만들면, 이터레이션 넘버가 천천히 증가하면서 `alpha` 가 감소해 *converge* 하는 결과를 얻을 수 있다.

> If we reduce learning rate `alpha` (and run stochastic gradient descent long enough), it's possible that we may find a set of better parameters than with large `alpha`

<br/>

### Online Learning

![](http://img.my.csdn.net/uploads/201302/22/1361520551_7175.jpg)

*online learning* 에서는 데이터를 얻어 `theta` 를 업데이트하는데 사용하고, 버린다. 큰 사이트라면 데이터가 지속적으로 들어오기 때문에, *trianing data* 를 볼 필요가 없다. 다시 말해 같은 데이터를 두번 이상 쓰지 않는다는 말이다.

또 다른 장점으로는 사용자의 취향 변화를 빠르게 반영할 수 있다는 점이다.

> Can adopt to changing user preference

![](http://img.my.csdn.net/uploads/201302/22/1361520594_9988.jpg)
<p align="center">(http://blog.csdn.net/linuxcumt)</p>

*product search* 에 *predicted CTR* 를 이용해, 검색어와 잘 매칭되는 상품을 결과로 돌려수 있다. 이때 매 검색마다 돌려주는 검색결과는 일종의 *training set* 이 된다.

- special offers
- customized selection

등에도 사용할 수 있다.

<br/>

### Map Reduce and Data Parallelism

데이터가 어마어마하게 많으면, 하나의 컴퓨터에서 머신러닝 알고리즘을 돌리기가 좀 힘들다. 어떻게 해결할까?

![](http://img.my.csdn.net/uploads/201302/22/1361522176_1942.jpg)
![](http://img.my.csdn.net/uploads/201302/22/1361522180_7521.jpg)

쉽게 말하면, 분산해서 처리할 수 있는 결과는 `map` 으로 해결하고, 이 결과들을 이용해 전체적인 결과는 `reduce` 가 계산한다. (실제로는 `reduce` 도 여러개 일 수 있다)

![](http://img.my.csdn.net/uploads/201302/22/1361522184_5033.jpg)
<p align="center">(http://blog.csdn.net/linuxcumt)</p>

> Many lenaring algorithms can be expressed as computing sums of functions over the training set

이렇기 때문에 *map-reduce* 가 큰 데이터셋에 대한 계산 처리 방법으로 좋은 해결책이 될 수 있다.

![](http://img.my.csdn.net/uploads/201302/22/1361522188_9650.jpg)
<p align="center">(http://blog.csdn.net/linuxcumt)</p>

요즘엔 대부분의 프로세서가 멀티코어이기 때문에, 하나의 컴퓨터에서도 병렬화를 이용해 계산을 빠르게 해 낼 수 있다. 이 경우는 *network latency* 등에 대해 생각을 안해도 된다. 참고로 좋은 라이브러리들은 자동으로 연산을 병렬화한다. 

<br/>

### Photo OCR and Pipeline

머신러닝 예제로 *Photo OCR* 을 알아보자.

![](http://img.my.csdn.net/uploads/201302/27/1361935712_3407.jpg)
<p align="center">(http://blog.csdn.net/linuxcumt)</p>

*Photo OCR pipeline* 은 

- Image
- Text detection
- Character segmentation
- Character recognition

의 단계를 거친다. 각 단계마다 머신러닝을 적용할 수 있다.

<br/>

### Sliding Windows

![](http://img.my.csdn.net/uploads/201302/27/1361936303_5994.jpg)
![](http://img.my.csdn.net/uploads/201302/27/1361936307_1350.jpg)
![](http://img.my.csdn.net/uploads/201302/27/1361937647_2708.jpg)

텍스트나, 보행자등 특정 패턴을 검색하기 위해 이동하는 *rectangle* 의 단위를 *step-size, slide* 라 부른다. *slide* 의 사이즈를 변경해 가면서 패턴을 파악하는 방법을 *sliding window* 라 부른다.

![](http://img.my.csdn.net/uploads/201302/27/1361937654_3410.jpg)
![](http://img.my.csdn.net/uploads/201302/27/1361937664_2466.jpg)
<p align="center">(http://blog.csdn.net/linuxcumt)</p>

텍스트를 인식해서, 근처의 텍스트와 묶는 *expansion* 작업을 하고 *character segmentation* 단계로 넘어간다. 

![](http://img.my.csdn.net/uploads/201302/27/1361937671_6657.jpg)
![](http://img.my.csdn.net/uploads/201302/27/1361937676_4874.jpg)
<p align="center">(http://blog.csdn.net/linuxcumt)</p>

<br/>

### Artificial Data

*low bias* 에 *massive data set* 을 조합하면 좋은 퍼포먼스가 나오긴 하는데, 어디서 커다란 데이터셋을 구할까? 작은 데이터 셋으로 커다란 데이터셋을 인위적으로 만들 수 있을까?

![](http://img.my.csdn.net/uploads/201302/27/1361948453_3756.jpg)
![](http://img.my.csdn.net/uploads/201302/27/1361948457_9107.jpg)
![](http://img.my.csdn.net/uploads/201302/27/1361948462_1113.jpg)
<p align="center">(http://blog.csdn.net/linuxcumt)</p>

스케일링, 로테이션, 디스토션, 백그라운드 수정 등 다양한 조합을 통해 진짜처럼 보이는 *synthetic data* 를 얻을 수 있다. 마찬가지로, *speech recognition* 에도 *synthetic data* 를 만들어 퍼포먼스를 높일 수 있다.

![](http://img.my.csdn.net/uploads/201302/27/1361948466_5032.jpg)

- *synthetic data* 를 만들기 전에 *low bias classifier* 인지 확인하자.
- 데이터를 조합하는데 들어가는 노력이 얼마나 들까 생각해보자
- *crowd source* 를 고려하자. (e.g. Amazon Mechanical Turk)

10 초당 1개의 *example* 을 수동으로 얻는다면, 10000 개를 얻는데 대략 3.5일의 시간이 걸린다.

### Ceiling Analysis

이전의 *Photo OCR* 문제에서 퍼포먼스를 높이려면 파이프라인의 각 단계 중 어느 부분에 가장 많은 노력을 들여야할까? 

![](http://img.my.csdn.net/uploads/201302/27/1361952527_2269.jpg)
<p align="center">(http://blog.csdn.net/linuxcumt)</p>

각 단계에서 수동으로 정확도 100% 를 만들었을때와, 전체적인 정확도를 비교해서 어느 부분을 향상 시켰을때 가장 효율적일지를 파악할수 있다.  

![](http://img.my.csdn.net/uploads/201302/27/1361952535_9451.jpg)
![](http://img.my.csdn.net/uploads/201302/27/1361952540_1528.jpg)
<p align="center">(http://blog.csdn.net/linuxcumt)</p>

<br/>

### Summary

![](http://img.my.csdn.net/uploads/201302/27/1361952938_3624.jpg)
<p align="center">(http://blog.csdn.net/linuxcumt)</p>

**supervised learning** 의 종류로

- linear regresison
- logistic regression
- neural networks
- SVM

**unsupervised learning** 으로

- k-means
- PCA
- anomaly detection

또한 머신 러닝의 응용으로

- recommender system
- large scale ML

마지막으로 머신러닝에 도움이 되는 주제로

- bias vs variance
- regularization
- evaluation technique
- learning curve
- error analysis
- ceiling analysis

등을 배웠다.

![](http://img.my.csdn.net/uploads/201302/27/1361953290_6926.jpg)

### References

(1) *Machine Learning* by **Andrew NG**  
(2) [http://blog.csdn.net/linuxcumt](http://blog.csdn.net/linuxcumt)  
(3) [http://blog.csdn.net/abcjennifer](http://blog.csdn.net/abcjennifer)  
