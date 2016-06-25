+++
date = "2016-06-25T14:20:43+09:00"
next = "../intro-to-data-science-3"
prev = "../intro-to-data-science-1"
title = "Intro to Data Science 2"
toc = true
weight = 22
aliases = [
    "/edx-600-2x-2"
]
+++

> Computational systems are so very convenient for modeling behaviors of noisier, uncertain systems, **especially in estimating the values of parameters of those systems**.

### Monte Carlo Simulation

> Monte Carlo simulation is a method of **estimating the value of an unknown quantity using the principles of inferential statistics**

이전에 잠깐 *deterministic model* 과 *stochastic model* 언급 했었는데, 다시 한번 알아보자면

> In **deterministic models**, the output of the model is  fully determined by the parameter values and the
initial conditions.

> **Stochastic models** possess some inherent randomness. The same set of parameter values and initial conditions will lead to an ensemble of different outputs

때때로 사람들이 *deterministic model* 은 *uncertainty* 를 다루지 않는다고 말하곤 하는데 이건 엄밀히 말하면 틀렸다. *deterministic model* 내부적으로는 *randomness* 가 없지만, 모델 외부에 *uncertainty* 가 있을 수 있다.  

몬테 카를로 시뮬레이션이 바로 *deterministic model* 에 대해 *random input* 을 이용해 분포를 얻어내는 방법이다. 

> I have heard people say that "a stocahstic model handles uncertainty, a deterministic model doesn't". This is not strictly correct. The correct statement should be: 
a stochastic model has the capacity to handle then uncertainty in the inputs built into it, for a deterministic model, the uncertainties are extenal to the model. The uncertainties in the inputs to a deterministic model can be handled through use of a Monte Carlo simulation (note that this does not make it a stochastic model). This is computationally inefficient however.


### Finding PI

```python
def stdDev(X):
    mean = sum(X) / float(len(X))
    total = 0.0
    for x in X:
        total += (x - mean) ** 2
    return (total / len(X)) ** 0.5


def dropNeedles(num):
    inCircle = 0
    for needles in xrange(1, num + 1, 1):
        x = random.random()
        y = random.random()

        if (x*x + y*y) ** 0.5 <= 1.0:
            inCircle += 1

    return 4 * (inCircle / float(num))


def estimate(numOfNeedles, trials):
    estimates = []
    for i in range(trials):
        pi = dropNeedles(numOfNeedles)
        estimates.append(pi)

    sd = stdDev(estimates)
    est = sum(estimates) / len(estimates)

    return (est, sd)


def simulate(precision, trials):
    numOfNeedles = 1000
    sd = precision

    # 95% of the values lie within precision of the mean
    while sd >= (precision / 2.0):
        est, sd = estimate(numOfNeedles, trials)
        print 'PI est =', est, "sd =", round(sd, 6), "needles =", numOfNeedles
        numOfNeedles *= 2

    return est


random.seed(0)
simulate(0.005, 100)
```

반지름이 1인 원 안에 바늘을 떨어트려, 해당 원 안에 있을 경우와 직사각형에 있을 경우의 비율에 직사각형의 넓이를 곱하면, 원의 넓이 즉 `PI` 값이 나온다.

실제 돌려보면

```
PI est = 3.14844 sd = 0.047886 needles = 1000
PI est = 3.13918 sd = 0.035495 needles = 2000
PI est = 3.14108 sd = 0.02713 needles = 4000
PI est = 3.141435 sd = 0.016805 needles = 8000
PI est = 3.141355 sd = 0.0137 needles = 16000
PI est = 3.14131375 sd = 0.008476 needles = 32000
PI est = 3.141171875 sd = 0.007028 needles = 64000
PI est = 3.1415896875 sd = 0.004035 needles = 128000
PI est = 3.14174140625 sd = 0.003536 needles = 256000
PI est = 3.14155671875 sd = 0.002101 needles = 512000
```

`32000` 개와 `64000` 개의 바늘을 떨어트린 샘플을 보면 실제 샘플도 후자가 많고, 표준편차도 후자가 작음에도 실제 추정값은 더 나쁘게 나왔다.

표준편차가 작으면 실제 값에 근접한 추정값이 나왔다는 뜻이 아닌가? 

> Having the small standard deviation doesn't mean we have a good estimate.

그렇지 않다. 표준편차가 작다는 것이, 우리가 얻은 추정값이 실제 값과 같다는 뜻은 아니다.

> All this means is that if we were to draw more samples from the same distribution, we can be reasonably confident that we would get a similar value.

단지 같은 분포에서 더 많은 샘플을 이용하면 *현재 값과 비슷한 값 (!= 실제값)* 을 얻을 수 있다는 말이다.

우리가 구한 값이 실제 `PI` 와 근사하다고 믿기 전에 3가지를 먼저 확인해야한다.

(1) **conceptual model** (이 경우 `PI` 를 위한 계산)  
(2) **implementation**  
(3) **enough samples**  

만약에 `4 * (inCircle / float(num)` 대신에 `2 * (inCircle / float(num)` 를 사용해 잘못된 *conceptual model* 을 가진다면 (버그라 볼 수도 있겠다.)

```
PI est = 1.57422 sd = 0.023943 needles = 1000
PI est = 1.56959 sd = 0.017748 needles = 2000
PI est = 1.57054 sd = 0.013565 needles = 4000
PI est = 1.5707175 sd = 0.008402 needles = 8000
PI est = 1.5706775 sd = 0.00685 needles = 16000
PI est = 1.570656875 sd = 0.004238 needles = 32000
PI est = 1.5705859375 sd = 0.003514 needles = 64000
PI est = 1.57079484375 sd = 0.002017 needles = 128000
```

보면 알겠지만, 표준편차는 충분히 작음에도 우리가 구한 추정값이 `PI` 와는 상당히 다르다.

> Whenever possible, one should attempt to validate results against realilty

### Normal Distribution

> Instead of estimating an unknown parameter by a single value, a **confidence interval** provides a range that is likely to contain the unknown value and a confidence level that the unknown value lays within that range


### Common Pattern in Science and Engineering

보통 두 가지 작업이 있는데,

(1) develop a hypothesis  
(2) design an experiment, take measurements  
(3) use computation to  
- evaluate hypothesis,  
- determin values of unknowns,  
- predict consequences

두가지는 `1 -> 2` 순서일 수 있고, 때때로 뒤 바뀔 수도 있다. 예를 한번 살펴보면

먼저 16세기에 *Hooke* 는 *"용수철에 가해진 힘은 그 길이에 비례한다는 가설"* 을 세웠다. 이를 증명하기 위해 실험을 고안했는데, 천장에 서로 다른 길이의 스프링을 연결하고 거기에 저울 추를 달았다.

늘어난 길이 `x` 에 대해 용수철 상수 `k` 를 `kx = mg` 를 이용해 계산하면, 얼추 맞는다. 그런데 몇몇 샘플에 대해서는 용수철 상수 `k` 가 상당히 다르게 나온다. `[11.41, 14.49 ...]` 왜 그럴까? 용수철 상수가 매번 변한다는걸까? 

다음 데이터에 대해 그래프를 그려보면

```
Distance (m) Mass (kg)
0.0865 0.1
0.1015 0.15
0.1106 0.2
0.1279 0.25
0.1892 0.3
0.2695 0.35
0.2888 0.4
0.2425 0.45
0.3465 0.5
0.3225 0.55
0.3764 0.6
0.4263 0.65
0.4562 0.7
0.4502 0.75
0.4499 0.8
0.4534 0.85
0.4416 0.9
0.4304 0.95
0.437 1.0
```

*distance* 에 대한 예측 `ma / k` 와 실제 값이 일치하지 않는다. 이른바 *error (오류)* 가 있는 것인데, 이들 오류는 *small randomness* 에 대한 축적의 결과로 이루어 진 것이다.

오류에 대한 *probabilty density function* 로 `y = x - 1 (-1 <= x < 0)`, `y = -x + 1 (0 < x <= 1)` 을 가정했을때 시뮬레이션을 좀 해보자. `random.triangular` 를 이용하면 *triangle distribution* 을 얻을 수 있다.

```python
def testErrors(ntrials=10000,npts=100):
    results = [0] * ntrials
    for i in xrange(ntrials):
        s = 0   # sum of random points
        for j in xrange(npts):
            s += random.triangular(-1,1)
        results[i] =s
    # plot results in a histogram
    pylab.hist(results,bins=50)
    pylab.title('Sum of 100 random points -- Triangular PDF (10,000 trials)')
    pylab.xlabel('Sum')
    pylab.ylabel('Number of trials')

testErrors()
pylab.show()
```

실행해 보면 에러의 합의 분포가 정규 분포와 비슷하다. 우리가 어떤 에러 분포를 고르든지 간에 *finite mean, variance* 를 가지고 있다면 에러의 분포는 정규분포다. 

실제 그런가 `random.triangular` 말고 `random.uniform` 을 이용해 보면 똑같이 정규분포를 얻는다. 이는 *central limit theorem (중심극한정리)* 를 의미하는데 위키에서 인용하면

>  동일한 확률분포를 가진 독립 확률 변수 n개의 평균값은 n이 적당히 크다면 정규분포에 가까워진다는 정리

결국 우리가 이전에 스프링을 이용해 봤던 실험에서 발생한 에러는 *small random error* 의 *accumulation* 이므로, 우리는 이 에러의 분포를 정규분포라 말할 수 있다는 것이다.

**결국 오차 역시 평균 주변에 몰려있는 값이므로, 참값을 상당히 높은 확률로 추측해 낼 수 있다.**

> 정규분포는 19세기의 가장 위대한 수학자인 가우스(C. F. Gauss, 1777-1855)에 의해 새롭게 해석된다. 가우스는 관측에 따른 오차의 정도가 대체로 평균값 주변에서 발생한다는 점에 착안하여 정규분포에 따른 확률 밀도 함수와 똑 같은 식을 얻을 수 있었다. 이것은 **관측 오차 역시 정규분포를 따른다는 것으로, 이후 실험으로 구한 관측값에서 참값을 추정해내는 근본적인 원리**로 자리잡게 된다. 이런 점에서 위의 종모양 곡선을 오차곡선(error curve)라고도 부른다.

*normal distribution* 의 식은 

![](http://latex.codecogs.com/gif.latex?f%28x%29%20%3D%20%7B1%20%5Cover%20%5Csqrt%7B2%5Cpi%5Csigma%5E2%7D%7D%20%5C%20e%5E%7B-%28x%20-%20%5Cmu%29%5E2%20%5Cover%20%5Csigma%5E2%7D)

`mu = 0, sigma = 1` 인 경우에 *standard normal* 혹은 *unit normal* 이라 부른다.

> So when observation errors are due to the accumlation of many small random perturbations

![](http://latex.codecogs.com/gif.latex?f%28x%29%20%3D%20%7B1%20%5Cover%20%5Csqrt%7B2%5Cpi%5Csigma%5E2%7D%7D%20%5C%20e%5E%7B-%28x%29%5E2%20%5Cover%20%5Csigma%5E2%7D)

다시 말해 작은 랜덤의 누적으로 발생한 관측 오차의 경우에는 `mu = 0` 이다. 그리고 식이 말해주는 바는, 큰 에러의 경우에는 확률이 *expnentially less likely* 하다는 것이다.

이제 각 에러가 일어날 확률 곱을 다음과 같이 구할 수 있다. 

![](http://latex.codecogs.com/gif.latex?%5Cprod_%7Bi%20%3D%200%7D%5E%7Blen%28obj%29%20-%201%7D%20%5C%20L_%7Berr%7D%20%28obs_i%20-%20pred_i%29)

이때 이 값을 최대화한다는 것은 각 에러가 나올 확률이 가장 높아야 한다. 다시 말해서 가장 평균적인 에러만 나와야 한다는 뜻이다. 이 값의 최대를 구하는 것은 뒤집은 식의 최소값을 찾는 것과 같으므로

![](http://latex.codecogs.com/gif.latex?min%20%5C%20%7B1%20%5Cover%20%5Cprod_%7Bi%20%3D%200%7D%5E%7Blen%28obj%29%20-%201%7D%20%5C%20L_%7Berr%7D%20%28obs_i%20-%20pred_i%29%7D)

여기에 자연 로그를 씌우면 확률 변수의 곱이 덧셈으로 변한다.

![](http://latex.codecogs.com/gif.latex?ln%20%28%7B1%20%5Cover%20%5Cprod%20%5C%20L_%7Berr%7D%20%28obs_i%20-%20pred_i%29%7D%29%20%5C%5C%20%5C%5C%20%5C%5C%20%3D%20-%20%5Csum%20ln%20%28L_%7Berr%7D%28obs_i%20-%20pred_i%29%29)

이 값을 최소화 하면 된다. 여기서 *pdf* 식을 적용하면

![](http://latex.codecogs.com/gif.latex?min%20-%20%5Csum%20ln%20%28%7B1%20%5Cover%20%5Csqrt%7B2%5Cpi%5Csigma%5E2%7D%7D%20e%5E%7B%28obs_i%20-%20pred_i%29%5E2%20%5Cover%20%5Csigma%5E2%7D%29)

로그를 씌우면 

![](http://latex.codecogs.com/gif.latex?%5Csum%20%5Bln%20%7B%5Csqrt%7B2%5Cpi%5Csigma%5E2%7D%7D%20&plus;%20ln%20%7B%28obs_i%20-%20pred_i%29%5E2%20%5Cover%20%5Csigma%5E2%7D%5D)

이 때 다른 상수를 제외하고 실제 최소화 해야 할 부분만 고려하면

![](http://latex.codecogs.com/gif.latex?min%20%5Csum%20%28obs_i%20-%20pred_i%29%5E2)

따라서 이 값을 최소화 하면 *most likely observations* 가 된다. 이런 이유에서 오차 제곱의 합의 최소가 되는 파라미터가 바로 가장 신뢰할만한 파라미터가 된다.

  
<br/>
처음부터 정리하자면, 중심 극한 정리에 따라 에러의 분포가 어떠하든 간에 에러가 확률변수라면 이것의 평균은 정규분포다. 따라서 *pdf* 식을 적용할 수 있고 이때 `mean = 0` 이다.

에러가 나올 확률의 곱이 최대이면, 모든 에러에 대해 보편적인 에러를 얻었다는 뜻이므로 이에 대해 식을 정리하면,

*sum of squared of erros (SSE, least square)* 를 최소화 하는 파라미터를 선택하면 가장 신뢰할 만한 관측 결과를 얻어낼 수 있다는 결론을 얻게된다.

<br/>
파이선에서는 `pylab.plotfit` 을 이용해 값을 최소화 하는 파라미터를 뽑아낼 수 있다. 예를 들어서 `y = ax + b` 의 합을 최소화 하는 `a, b` 를 찾으려면 

```python
a, b = pylab.ployfir(xvals, yvals, 1)
```

`y = ax^2 + bx + c` 에 대해서는

```python
a, b, c = pylab.polyfit(xvals, yvals, 2)
```

### Firing Arrow

이제 위에서 얻은 개념을 다른 예제에 적용해보면서 가설이 얼마나 *잘 맞는가* 를 어떻게 측정할건지를 좀 생각해 보자. (*Measuring "goodness" of fit*)

화살이 날라가는 거리에 따른 높이를 측정한 데이터다.

```
Distance (yds) height (ins) height height height
30  0 0 0 0
29 2.25 3.25 4.5 6.5
28 5.25 6.5 6.5 8.75
27 7.5 7.75 8.25 9.25
26 8.75 9.25 9.5 10.5
25 12 12.25 12.5 14.75
24 13.75 16 16 16.5
23 14.75 15.25 15.5 17.5
22 15.5 16 16.6 16.75
21 17 17 17.5 19.25
20 17.5 18.5 18.5 19
15 19.5 20 20.25 20.5
10 18.5 18.5 19 19
5 13 13 13 13
0 0 0 0 0
```

```python

def getTrajectoryData(fileName):
    dataFile = open(fileName, 'r')
    distances = []
    heights1, heights2, heights3, heights4 = [],[],[],[]
    discardHeader = dataFile.readline()
    for line in dataFile:
        d, h1, h2, h3, h4 = line.split()
        distances.append(float(d))
        heights1.append(float(h1))
        heights2.append(float(h2))
        heights3.append(float(h3))
        heights4.append(float(h4))
    dataFile.close()
    return (distances, [heights1, heights2, heights3, heights4])

def tryFits(fName):
    distances, heights = getTrajectoryData(fName)
    distances = pylab.array(distances)*36 # convert yard to
    totHeights = pylab.array([0]*len(distances))
    for h in heights:
        totHeights = totHeights + pylab.array(h)
    pylab.title('Trajectory of Projectile (Mean of 4 Trials)')
    pylab.xlabel('Inches from Launch Point')
    pylab.ylabel('Inches Above Launch Point')
    meanHeights = totHeights/float(len(heights))
    pylab.plot(distances, meanHeights, 'bo')
    a,b = pylab.polyfit(distances, meanHeights, 1)
    altitudes = a*distances + b
    pylab.plot(distances, altitudes, 'r',
               label = 'Linear Fit')
    a,b,c = pylab.polyfit(distances, meanHeights, 2)
    altitudes = a*(distances**2) + b*distances + c
    pylab.plot(distances, altitudes, 'g',
               label = 'Quadratic Fit')
    pylab.legend()

```

위 코드를 돌려보면 이차함수가 일차함수보다 더 *fit* 한 걸 볼 수 있다. 그럼 문제는 매번 그래프로 그릴수도 없고 어떻게 측정할거냐 하는건데, *variabilty* 를 이용하는 방법이 있다. 다시 말해 에러가 얼마나 많이 변하냐는 것이다.

*variability of errors* 는 SEE, 즉 관측 데이터와 예측값 간 차이의 제곱의 합으로 구할 수 있다. 그리고 *variability of data* 는 관측값과 관측값의 평균의 차이의 제곱의 합으로 구할 수 있다. 그리고 이 두 변수간 비율로 모델이 얼마나 잘 맞는지를 판단할 수 있다.

![](http://latex.codecogs.com/gif.latex?1%20-%20%7B%5Csigma_%7Berr%7D%5E2%20%5Cover%20%5Csigma_%7Bdata%7D%5E2%7D)

이 값을 `R^2` 또는 *coefficient of determination* 이라 부른다. 이 값이 `1` 에 근접하면 모델이 데이터와 잘 맞고, `0` 에 가까우면 거의 안맞는다는 뜻이다.

그러나 주의해야 할 점이 하나 있다. `R^2` 값이 높은 모델이라고 해서 좋은 모델이라는 뜻은 아니다. 지금 현재 가진 데이터에 *fit* 된다는 거지, 실제 데이터에 적용하면 어떻게 될지 모른다. *overfitting* 할 수도 있다는 이야기다.

파이썬에서 `R^2` 를 구하는 함수를 만들면

```python
def rSquare(measured, estimated):
    """measured: one dimensional array of measured values
       estimate: one dimensional array of predicted values"""
    SEE = ((estimated - measured)**2).sum()
    mMean = measured.sum()/float(len(measured))
    MV = ((mMean - measured)**2).sum()
    return 1 - SEE/MV
```

### References

(1) *MIT 6.00.2 2x* in **edx**  
(2) [http://ko.wikipedia.org](http://ko.wikipedia.org/wiki/%EC%A4%91%EC%8B%AC%EA%B7%B9%ED%95%9C%EC%A0%95%EB%A6%AC)  
(3) [http://www.financedoctor.co.kr](http://www.financedoctor.co.kr/finance/view.php?f_idx=14587&b_code=8&m_code=0&s_code=0)  
(4) [www.researchgate.net/](http://www.researchgate.net/post/What_is_the_difference_among_Deterministic_model_Stochastic_model_and_Hybrid_model)  
(5) [www4.stat.ncsu.edu](http://www4.stat.ncsu.edu/~gross/BIO560%20webpage/slides/Jan102013.pdf)
