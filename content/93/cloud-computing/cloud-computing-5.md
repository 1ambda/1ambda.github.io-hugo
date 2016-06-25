+++
date = "2016-06-25T14:42:45+09:00"
next = "../cloud-computing-6"
prev = "../cloud-computing-4"
title = "CC 05: Global Snapshot"
toc = true
weight = 15
aliases = [
    "/cloud-computing-snapshot"
]
+++

![](http://ook.co/wp-content/uploads/cloudcomputing.png)

이번시간에는 *Distributed Snapshot* 에 대해서 배웁니다. 클라우드 환경에서 각 어플리케이션(혹은 서비스) 는 여러개의 서버 위에서 돌아갑니다. 각 서버는 *concurrent events* 를 다루며, 서로 상호작용합니다. 이런 환경에서 *global snapshot* 을 캡쳐할 수 있다면

- **check pointing:** can restart distributed application on failure
- **garbage collection of objects:** object at servers that don't have any other objects(ay any servers) with pointers to them
- **deadlock detection:** useful in database transaction systems
- **termination of computation:** useful in batch computing systems like Folding@Homes, SETI@Home

*global snapshot* 은 두 가지를 포함합니다.

(1) Individual state of each process 
(2) Individual state of each communication channel 

*global snapshot* 을 만드는 한가지 방법은 모든 프로세스의 *clock* 을 동기화 하는 것입니다. 그래서 모든 프로세스에게 *time* `t` 에서의 자신의 상태를 기록하도록 요구할 수 있습니다. 그러나

- Time synchorization always has error
- Doesn't not record the state of meesages in the channels

지난 시간에 보았듯이, *synchronization* 이 아니라 *casuality* 로도 충분합니다. 프로세스가 **명령을 실행하거나**, **메시지를 받거나**, **메시지를 보낼때마다** *global system* 가 변합니다. 이를 저장하기 위해서 *casuality* 를 기록하는 방법을 알아보겠습니다.

<br/>

### Chandy-Lamport Algorithm

시작 전에 *system model* 을 정의하면

- N Processes in the system
- There are two uni-directional communication channels between each ordered process pair `P_j -> P_i`, `P_i -> P_j`
- communication channels are **FIFO** ordered
- **No failure**
- All messages arribe intact, and are not duplicated

*requirements* 는

- *snapshot* 때문에 *application* 의 작업에 방해가 일어나서는 안됩니다
- 각 프로세스는 자신의 *state* 를 저장할 수 있어야 합니다
- *global state* 는 분산회되어 저장됩니다 (collected in a distributed manner)
- 어떤 프로세스든지, *snapshot* 작업을 시작할 수 있습니다

![](https://raw.githubusercontent.com/1ambda/1ambda.github.io/master/assets/images/cloud-computing-concept-1/week5/Chandy_Lamport1.png)

- 프로세스 `P_i` 가 *market* 메세지를 만들고, 자신을 제외한 다른 `N-1` 개의 프로세스에게 보냅니다
- 동시에 `P_i` 는 *incoming channel* 을 레코딩하기 시작합니다

![](https://raw.githubusercontent.com/1ambda/1ambda.github.io/master/assets/images/cloud-computing-concept-1/week5/Chandy_Lamport2.png)

(1) 만약 `P_i` 가 *marker* 메시지를 처음 받는다면

- 만약메시지를 받은 프로세스 `P_i` 에서는 자신의 *state* 를 기록하고
- 자신을 제외한 프로세스들에게 *marker* 보내고
- 는 *incoming channel* 을 레코딩하기 시작합니다

(2) `P_i` 가 이미 *market* 메세지를 받은적이 있다면

- 이미 해당 채널의 모든 메세지를 기록중이었으므로, 레코딩을 끝냅니다

![](https://raw.githubusercontent.com/1ambda/1ambda.github.io/master/assets/images/cloud-computing-concept-1/week5/Chandy_Lamport3.png)

이 알고리즘은 모든 프로세스가 자신의 *state* 와 모든 *channel* 을 저장하면 종료됩니다. 

![](https://raw.githubusercontent.com/1ambda/1ambda.github.io/master/assets/images/cloud-computing-concept-1/week5/Chandy_Lamport_Example1.png)

![](https://raw.githubusercontent.com/1ambda/1ambda.github.io/master/assets/images/cloud-computing-concept-1/week5/Chandy_Lamport_Example2.png)

![](https://raw.githubusercontent.com/1ambda/1ambda.github.io/master/assets/images/cloud-computing-concept-1/week5/Chandy_Lamport_Example3.png)

![](https://raw.githubusercontent.com/1ambda/1ambda.github.io/master/assets/images/cloud-computing-concept-1/week5/Chandy_Lamport_Example4.png)

![](https://raw.githubusercontent.com/1ambda/1ambda.github.io/master/assets/images/cloud-computing-concept-1/week5/Chandy_Lamport_Example5.png)

![](https://raw.githubusercontent.com/1ambda/1ambda.github.io/master/assets/images/cloud-computing-concept-1/week5/Chandy_Lamport_Example6.png)

<br/>

### Consistent Cuts

*Chandy-Lamport* 알고리즘은 *casuality* 를 보장합니다. 이에 대해 증명하기 전에 먼저, *consistent cut* 이란 개념을 보고 가겠습니다.

- **Cut:** time frontier at each process and at each channel. Events at the process/channel that happen before the cut are **in the cut** and happening after the cut are **out of the cut**

- **Consistent Cut:** a cut that obeys casuality. A cut `C` is a consistent cut iff for each pair of event `e` `f` in the system, such that event `e` is in the cur `C` and if `f -> e`

다시 말해서 `e` 가 `C` 내에 있고, `f -> e` 라면 `f` 도 `C` 에 있어야만 *consistent cut* 이란 뜻입니다.

![](https://raw.githubusercontent.com/1ambda/1ambda.github.io/master/assets/images/cloud-computing-concept-1/week5/consistent_cut1.png)

`F` 가 *cut* 내에 있지만, 올바르게 캡쳐되어 메시지 큐 내에서 전송중임을 *snapshot* 에서 보장합니다. 하지만 `G -> D` 같은 경우는, `D` 가 *cut* 내에 있지만 `G` 가 그렇지 않아 *inconsistent cut* 입니다.

![](https://raw.githubusercontent.com/1ambda/1ambda.github.io/master/assets/images/cloud-computing-concept-1/week5/consistent_cut2.png)

*Chandy-Lamport Global Snapshot* 알고리즘은 항상 *consistent cut* 을 만듭니다. 왜 그런가 증명을 보면

![](https://raw.githubusercontent.com/1ambda/1ambda.github.io/master/assets/images/cloud-computing-concept-1/week5/Chandy_Lamport_Proof1.png)

`ei -> ej` 를 보장한다는 말은 스냅샷 안에 두 이벤트가 있다는 뜻입니다. 따라서 `ej -> <P_j records its state>` 일때 당연히 `ei -> <P_i records its state>` 와 같은 말입니다.

![](https://raw.githubusercontent.com/1ambda/1ambda.github.io/master/assets/images/cloud-computing-concept-1/week5/Chandy_Lamport_Proof2.png)

만약 `ej -> <P_j records its state>` 일때 `<P_i records its state> -> ei` 라 합시다.

그러면 `ei -> ej` 로 가는 *regular app message* 경로를 생각해 봤을때, `P_i` 가 먼저 자신의 상태를 기록하기 시작했으므로 *marker* 메세지가 먼저 날라갈겁니다. (FIFO) 그러면 위에서 말한 `ei -> ej` 경로를 타고 *marker* 메세지가 먼저 가게되고 `P_j` 는 자신의 상태를 먼저 기록하게 됩니다. 따라서 `P_j` 에서 `ej` 보다 자신의 상태를 기록하는 것이 먼저이므로 `ej` 는 *out of cut* 이고, 모순입니다.

<br/>

### Safety and Liveness

분산시스템의 *correctness* 와 관련해서 *safety* 와 *liveness* 란 개념이 있습니다. 이 둘은 주로 혼동되어 사용되는데, 둘을 구별하는 것은 매우 중요합니다.

![](https://raw.githubusercontent.com/1ambda/1ambda.github.io/master/assets/images/cloud-computing-concept-1/week5/liveness.png)

- distributed computation will terminate eventually 
- every failure is eventually deteced by some non-faulty process

![](https://raw.githubusercontent.com/1ambda/1ambda.github.io/master/assets/images/cloud-computing-concept-1/week5/safety.png)

- there is no deadlock in a distributed transaction system
- no object is orphaned
- **accuracy** in failure detector
- no two processes decide on different values

![](https://raw.githubusercontent.com/1ambda/1ambda.github.io/master/assets/images/cloud-computing-concept-1/week5/liveness_and_safety.png)

*failure detector* 나 *concensus* 의 경우에서 볼 수 있듯이 *completeness* 와 *accuracy* 두 가지를 모두 충족하긴 힘듭니다.

<br/>

![](https://raw.githubusercontent.com/1ambda/1ambda.github.io/master/assets/images/cloud-computing-concept-1/week5/language_of_global_state.png)

*global snapshot* 은 한 상태 `S` 이고, 여기서 다른 스냅샷으로의 이동은 *casual step* 을 따라 이동하는 것입니다. 따라서 *liveness* 와, *safety* 와 관련해 다음과 같은 특징이 있습니다.

![](https://raw.githubusercontent.com/1ambda/1ambda.github.io/master/assets/images/cloud-computing-concept-1/week5/using_global_snapshot.png)


*Chandy-Lamport* 알고리즘은 *stable* 한지를 검사하기 위해 사용할 수도 있습니다. 여기서 *stable* 하다는 것은, 한번 참이면 그 이후에는 계속 참인 것을 말합니다. 이는 알고리즘이 *casual correctness* 를 가지기 때문입니다.

<br/>

### Refs

(1) [Title Image](http://ook.co/solutions/cloud-computing/)  
(2) **Cloud Computing Concept 1** by *Indranil Gupta*, Coursera  
