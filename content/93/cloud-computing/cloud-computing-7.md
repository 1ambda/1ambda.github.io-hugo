+++
date = "2016-06-25T14:42:47+09:00"
prev = "../cloud-computing-6"
title = "CC 07: Paxos"
toc = true
weight = 17
aliases = [
    "/cloud-computing-paxos"
]
+++

![](http://ook.co/wp-content/uploads/cloudcomputing.png)

대부분의 분산 서버 벤더들은 `99.99999%` 의 *reliability* 를 보장하지만, `100%`는 아닙니다. 왜그럴까요? 그들이 못해서가 아니라 *consensus* 문제 때문입니다.

> The fault lies in the impossibility of consensus

*Consensus* 문제가 중요한 이유는, 많은 분산 시스템이 *consensus* 문제이기 때문입니다. 

- Perfect Failure Detection
- Leader Election
- Agreement (harder than consensus)

<br/>

일반적으로 서버가 많으면 다음의 일들을 해야합니다.

- **Reliable Multicast:** Make sure that all of them receive the same updates in the same order as each other
- **Membership/Failure Detection:** To keep their own local lists where they know about each other, and when anyone leaves or fails, everyone is updated simultaneously
- **Leader Election:** Elect a leader among them, and let everyone in the group know about it
- **Mutual Exclusion:** To ensure mutually exclusive access to a critical resource like a file

이 문제들은 대부분 *consensus* 와 연관되어 있습니다. 더 직접적으로 연관되어 있는 문제들은

- The ordering of messages
- The up/down status of a suspected failed process
- Who the leader is
- Who has access to the critical resource


<br/>

### Consensus Problem

![](https://raw.githubusercontent.com/1ambda/1ambda.github.io/master/assets/images/cloud-computing-concept-1/week5/consensus_problem.png)

![](https://raw.githubusercontent.com/1ambda/1ambda.github.io/master/assets/images/cloud-computing-concept-1/week5/consensus_problem2.png)

모든 프로세스(노드, 서버)가 같은 *value* 를 만들도록 해야 하는데, 몇 가지 제약조건이 있습니다.

- **validity:** if everyone propose same value, then that's what's decided
- **integrity:** decided value must have been proposed by some process
- **non-triviality:** there is at least one initial system state that leads to each of the all-`0`'s or all-`1`'s outcomes

*non-triviality* 는 쉽게 말해서, 모두 `0` 이거나 모두 `1` 일 수 있는 상태가 있어야 한다는 뜻입니다. 왜냐하면 항상 `0` 이거나 `1` 만 나오면 *trivial* 하기 때문입니다. 별 의미가 없죠.

<br/>

### Models

*consensus* 문제는 분산 시스템 모델에 따라 달라집니다. 모델은 크게 2가지로 나눌 수 있는데

(1) Synchronous Distributed System Model

- Each message is received within bounded time
- Drift of each process' local clock has a known bound
- Each step in a process takes `lb < time < ub`

동기 시스템 모델에서는 *consensus* 문제를 풀 수 있습니다.

(2) Asynchronous Distributed System Model

- Nobounds on process execution
- The drift rate of a clock is arbitrary
- No bounds on message transmission delay

일반적으로 비동기 분산 시스템 모델이 더 일반적입니다, 그리고 더 어렵죠. 비동기를 위한 프로토콜은 동기 모델 위에서 작동할 수도 있으나, 그 역은 잘 성립하지 않습니다.

비동기 분산 시스템 모델에서는 *consensus* 문제는 풀 수 **없습니다**

- Whatever protocol/algorithm you suggest, there is always a worst-case possible execution with failures and message delays that prevens the system from reaching consensus
- Powerful result(see the **FLP** proof)
- Subsequently, safe and **probabilistic** solution have become popular (e.g Paxos)

<br/>

### Paxos in Syncronous Systems

동기 시스템이라 가정합니다. 따라서

- bounds on message dealy
- bounds on upper bound on clock drift rates
- bounds on max time for each process step
- processes can fail by stopping

![](https://raw.githubusercontent.com/1ambda/1ambda.github.io/master/assets/images/cloud-computing-concept-1/week5/consensus_in_sync_system.png)

- 아무리 많아야 `f` 개의 프로세서에서 *crash* 가 나고
- 모든 프로세서는 *round* 단위로 동기화 되고, 동작하며
- *reliable communication* 을 통해 서로 통신합니다

*value_i^r* 을 *round* `r` 의 시작에 `P_i` 에게 알려진 *value* 의 집합이라 라 하겠습니다.

![](https://raw.githubusercontent.com/1ambda/1ambda.github.io/master/assets/images/cloud-computing-concept-1/week5/paxos1.png)

![](https://raw.githubusercontent.com/1ambda/1ambda.github.io/master/assets/images/cloud-computing-concept-1/week5/paxos2.png)

`f+1` 라운드 후에 모든 *correct* 프로세스는 같은 값의 집합을 가지게 되는데, 귀류법으로 쉽게 증명할 수 있습니다.

<br/>

![](https://raw.githubusercontent.com/1ambda/1ambda.github.io/master/assets/images/cloud-computing-concept-1/week5/consensus_in_async.png)

비동기 환경에서는, 아주아주아주아주아주 느린 프로세서와 *failed* 프로세서를 구분할 수 없기 때문에, 나머지 프로세서들이 이것을 결정하기 위해 영원히 기다려야 할지도 모릅니다. 이것이 기본적인 *FLP Proof* 의 아이디어입니다. 그렇다면, *consensus* 문제를 정말 풀기는 불가능한걸까요?

풀 수 있습니다. 널리 알려진 *consensus-solving* 알고리즘이 있습니다. 실제로는 불가능한 *consensus* 문제를 풀려는 것이 아니라, *safety* 와 *eventual liveness* 를 제공합니다. 야후의 *zookeeper* 나 구글의 *chubby* 등이 이 알고리즘을 이용합니다.

![](https://raw.githubusercontent.com/1ambda/1ambda.github.io/master/assets/images/cloud-computing-concept-1/week5/yes_we_can_with_paxos.png)

*safety* 는 서로 다른 두개의 프로세서가 다른 값을 제출하지 않는것을 보장하고, (*No two non-faulty processes decide different values*) *eventual liveness* 는 운이 좋다면 언젠가는 합의에 도달한다는 것을 말합니다. 근데 실제로는 꽤 빨리 *consensus* 문제를 풀 수 있습니다.

![](https://raw.githubusercontent.com/1ambda/1ambda.github.io/master/assets/images/cloud-computing-concept-1/week5/paxos_simple.png)

본래는 최적화때문에 더 복잡한데, 위 슬라이드에서는 간략화된 *paxos* 가 나와있습니다. *paxos* 의 *round* 마다 고유한 *ballot id* 가 할당되고, 각 *round* 는 크게 3개의 비동기적인 *phase* 로 분류할 수 있습니다.

- **election:** a leader is elected
- **bill:** leader proposes a value, processes ack
- **law:** leader multicasts final value

![](https://raw.githubusercontent.com/1ambda/1ambda.github.io/master/assets/images/cloud-computing-concept-1/week5/election.png)

먼저 *potential leader* 가 *unique ballot id* 를 고르고, 다른 프로세서들에게 보냅니다. 다른 프로세스들의 반응에 의해서 선출될 수도 있고, 선출되지 않으면 새로운 라운드를 시작합니다. 

- Because becoming a leader requires a majority of votes, and any two majorities intersect in at least one process, and each process can only vote once.

![](https://raw.githubusercontent.com/1ambda/1ambda.github.io/master/assets/images/cloud-computing-concept-1/week5/bill.png)

리더가 다른 프로세스들에게 `v` 를 제안하고, 프로세스들은 지난 라운드에 `v'` 를 결정했었으면 `v=v'` 를 이용해 값을 결정합니다.

![](https://raw.githubusercontent.com/1ambda/1ambda.github.io/master/assets/images/cloud-computing-concept-1/week5/decision.png)

만약 리더가 *majority* 의 긍정적인 반응을 얻으면 모두에게 그 결정을 알리고 각 프로세서는 합의된 내용을 전달받고, 로그에 기록하게 됩니다. 

![](https://raw.githubusercontent.com/1ambda/1ambda.github.io/master/assets/images/cloud-computing-concept-1/week5/paxos_no_return.png)

사실 이 과정은 응답을 리더가 받는 단계에서 결정되는 것이 아니라, 프로세서들이 *proposed value* 를 듣는순간 결정됩니다. 따라서 리더에서 *failure* 가 일어나도, 이전에 결정되었던 `v'` 을 이용할 수 있습니다.

<br/>

![](https://raw.githubusercontent.com/1ambda/1ambda.github.io/master/assets/images/cloud-computing-concept-1/week5/paxos_safety.png)

이전에도 언급했듯이 *safety* 는 두개의 서로 다른 프로세서의 의해서 다른 값이 선택되지 않음을 보장합니다. 이는 잠재적 리더가 있다 하더라도 현재 리더와, 잠재적 리더에게 응답하는 *majority* (반수 이상) 을 교차하면 적어도 하나는 `v'` 를 응답하기 때문에 *bill phase* 에서 정의한대로 이전 결과인 `v'` 가 사용됩니다.

![](https://raw.githubusercontent.com/1ambda/1ambda.github.io/master/assets/images/cloud-computing-concept-1/week5/paxsos_liveness.png)

그림에서 볼 수 있듯이 영원히 끝나지 않을수도 있지만, 실제로는 꽤 빠른시간 내에 합의에 도달합니다. (eventualy-live in async systems)

<br/>

### Refs

(1) [Title Image](http://ook.co/solutions/cloud-computing/)  
(2) **Cloud Computing Concept 1** by *Indranil Gupta*, Coursera  
