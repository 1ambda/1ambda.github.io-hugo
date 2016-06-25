+++
date = "2016-01-06T00:37:23+09:00"
next = "../easy-scalaz-2"
prev = "../"
title = "Easy Scalaz 1"
toc = true
weight = 101
aliases = [
    "/easy-scalaz-1-state"
]
+++

![](https://raw.githubusercontent.com/1ambda/1ambda.github.io/master/assets/images/about-type-class/Typeclassopedia-diagram.png)

## About Type Classes

프로그래머가 하는 행위를 극도로 단순화해서 표현하면 **저수준** 의 데이터를 **고수준** 데이터로 변환하는 일입니다.

여기서 저수준이란, *Stream*, *Byte*, *JSON*, *String* 등 현실세계의 데이터를, 고수준이라 함은 비즈니스 로직, 제약조건 등이 추가된 도메인 객체, 모델 등 데이터를 말합니다.

이로 인해

1. 저수준을 고수준으로 변환하는건 조건이 충족되지 않은 데이터와 연산 과정에서 일어나는 시스템 오류를 처리해야하기 때문에 힘든일입니다

2. 갖은 고생 끝에 데이터를 고수준으로 끌어올린 뒤에야, 그 데이터를 프로그래머 자신의 세상에서 마음껏 주무를 수 있습니다

3. 프로그래머가 작업을 끝낼 시점이 되면, 데이터를 저수준으로 변환해서 저장 또는 전송해야 하는데, 이미 제약조건이 충족 되었기 때문에 이는 손쉬운 일입니다

따라서 핵심은 다음의 두가지 입니다.

- 쉽게 고수준으로 변환할 수 있는가 (**연산**)
- 변환된 고수준 데이터가 얼마나 다루기 편한가 (**추상**)

프로그래머가 *적절한 연산* 을 선택하면 힘들이지 않고 변환을 해낼것이고, *적절한 추상 (혹은 모델링)* 을 한다면 직관적인 코드로 데이터를 주무를 수 있게 되는데, 이 것을 도와주는 것이 바로 **타입 클래스** 입니다.

타입 클래스를 이용하면,

- `if null`  을 Option 으로,
- `S => (S, A)` 을 State[S, A] 로
- `if if if` 를 Applicative 로
- *fail-slow*, *fail-fast* 로직은 ValidationNel 과 Either 로
- `F[G[A]]` 을 `G[F[A]]` 로의 변경은 Traversal 로
- `setC{applyB{getA}}` 를 getA > applyB > setC 로(Kleisli)
 표기할 수 있습니다.

이렇게 연산을 각각의 타입으로 표시하기 때문에 로직을 파악하고, 분 하기 쉽습니다. 그리고 연산을 작성하는 과정이 타입을 조합하는 과정과 동일하기 때문에 컴파일러의 도움을 받을수 있구요.

타입클래스는 **연산이 어떠해야 하는지** 를 다루기 때문에 연산을 조합할 수 있는 다양한 함수들이 포함되어 있습니다. 이것을 이용하면 직관적인 방식으로 데이터를 다룰 수 있는데, 예를 들어 다음은 코드 실행과정에서 예외 발생 시에만 롤백을 수행하고, 예외를 돌려주는 코드입니다. (간략화 하였습니다.)

```scala
\/.fromTryCatch {
  val result = runQuery;
  commit;
  result
} leftMap(err => rollback; err};
```

만약 `if null`  보다  `Option` 을 쓰는것이 더 편하고 익숙하다면, `Applicative` 부터 천천히 시작해보는건 어떨까요?

### Reference

- [Typeclassopedia Image - Haskell Wiki](https://wiki.haskell.org/Typeclassopedia)
