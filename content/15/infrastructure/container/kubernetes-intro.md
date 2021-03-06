+++
date = "2016-08-06T20:41:38+09:00"
prev = "../../"
title = "Kubernetes: Intro"
toc = true
weight = 50
+++

![](https://raw.githubusercontent.com/1ambda/1ambda.github.io/master/assets/images/infra-kubernetes/intro/logo.png)

## Kubernetes: Intro

### Caution

1. 작성자는 Container 전문가가 아니며 최대한 정확한 내용을 기록하려 했으나, 주말동안 짧게 찾아본 내용이므로 오류가 있을 수 있습니다. Production 적용을 위해서는, 더 많은 자료를 참고 부탁드립니다

2. 이하의 내용은 docker, docker-compose, docker-swarm 등에 대해 이해하고 있다는 가정 하에 쓰여졌습니다.
잘 모르신다면 [Docker Self-Paced Online Learning](https://training.docker.com/self-paced-training) 을 보고 오시면 더욱 쉽게 이해할 수 있습니다. 

3. 이 글에서는 kubernetes 의 구성요소, 인터널 등을 간략히 다룹니다. [Autopilot Pattern](http://autopilotpattern.io/) 등의 볼륨 매니지먼트 기술을 찾아 오셨다면, 아래의 글 참고 부탁드립니다. 

- [Joyent: Persistent storage patterns for Docker in Production](https://www.joyent.com/blog/persistent-storage-patterns) 
- [Joyent: MySQL on Autopilot](https://www.joyent.com/blog/dbaas-simplicity-no-lock-in)

<br/>

### Kubernetes

[kubernetes](kubernetes.io) (이하 k8s) 는 container orchestration 툴입니다. 
(e.g [docker-swarm](https://docs.docker.com/swarm/overview/),  [marathon](https://mesosphere.github.io/marathon/)) 

- 여러 host ([= node in k8s](http://kubernetes.io/docs/admin/node/)) 를 묶어 클러스터를 구성하고 
- container 를 적절한 위치에 배포하고 (auto-placement)
- container 가 죽으면 자동으로 복구하며 (auto-restart)
- 필요에 따라 container 를 매끄럽게 추가(scaling), 복제(replication), 업데이트(rolling update), 롤백(rollback) 할 수 있습니다
- 이 외에도 수 많은 기능이 있으며, [What is k8s?](http://kubernetes.io/docs/whatisk8s/) 에서 확인할 수 있습니다 

<br/>

k8s 를 사용하려면, 다음과 같은 내용을 알아야 합니다.

1. k8s object (e.g [pod](http://kubernetes.io/docs/user-guide/pods/), [pet set](http://kubernetes.io/docs/user-guide/petset/), [service](http://kubernetes.io/docs/user-guide/services/), [selector](http://kubernetes.io/docs/user-guide/labels/) 등)
2. multi-host 위에서 container 실행시 고려해야 할 것들 (e.g service discovery, networking, volume management, log aggregation)  
3. k8s internal

다행히도 문서가 장황하지 않습니다. 필요한 내용을, 필요한 만큼만 설명하고 있기 때문에 날 잡아서 쭈욱 읽기에도 괜찮습니다.

<br/>

내용을 설명하기에 앞서 짧게나마 느낀점을 요약하면 다음과 같습니다

- docker 1.12 기준으로 [docker-swarm: built-in orchestration](https://blog.docker.com/2016/06/docker-1-12-built-in-orchestration/) 이 대폭 개선되었음에도 불구하고, k8s 가 더 많은 기능과 자세한 세팅을 제공합니다.
실제 production 적용시에 다양한 요구사항이 생길 수 밖에 없기 때문에, k8s 의 다양한 기능은 큰 장점으로 보입니다
- GCE 를 쓰지 않아도 됩니다. github issue 를 살펴보다 보면 AWS 에서 사용하는 사람도 많은것 같습니다
([running k8s on AWS](http://kubernetes.io/docs/getting-started-guides/aws/))
일부 기능들 (e.g Service type LoadBalancer) 을 사용할 수 없으나, 세팅이 많은 만큼 우회가 가능하며
AWS 를 직접 지원하기도 합니다. (e.g AWSElasticBlockStore, SSL 등 ~~대인배~~)
- [ingress](http://kubernetes.io/docs/user-guide/ingress/) 등의 기능과 Future Work 등을 보고 있노라면
정말 다양한 기능을 빠르게 추가하려고 노력한다는 느낌을 받습니다

지금 production 에 적용할거라면 swarm 보다는 k8s 에 베팅하고 올라 타겠습니다. (물론 [marathone](https://mesosphere.github.io/marathon/) 도 살펴보겠습니다만..) 

<br/>

### Getting Started

1. GCE 가입하신 후 [Kubernetes: Hello World Walkthrough](http://kubernetes.io/docs/hellonode/) 를 진행하고 오시면 아래의 내용을 이해하시는데 더욱 도움이 됩니다. 

2. [minikube](https://github.com/kubernetes/minikube/blob/master/README.md) 를 설치하면 로컬에서 k8s 를 실행해볼 수 있습니다.  

3. kubectl 1.3.0+ 의 경우에는 bash 와 zsh completion 이 들어있습니다. 이걸 이용하면 편합니다. (`source <(kubectl completion zsh)
`) gcloud 이용시 1.2.5 버전이 깔릴 수 있습니다. 버전이 낮을시 [kubectl isntall](http://kubernetes.io/docs/user-guide/prereqs/) 참고하시어 설치하면 됩니다.

```
$ kubectl version

Client Version: version.Info{Major:"1", Minor:"3", GitVersion:"v1.3.0", GitCommit:"283137936a498aed572ee22af6774b6fb6e9fd94", GitTreeState:"clean", BuildDate:"2016-07-01T19:26:38Z", GoVersion:"go1.6.2", Compiler:"gc", Platform:"darwin/amd64"}

Server Version: version.Info{Major:"1", Minor:"3", GitVersion:"v1.3.3", GitCommit:"c6411395e09da356c608896d3d9725acab821418", GitTreeState:"dirty", BuildDate:"1970-01-01T00:00:00Z", GoVersion:"go1.6.2", Compiler:"gc", Platform:"linux/amd64"}
```

<br/>

### Kubernetes Internal (abbreviated) 

- [k8s.info: Cheat Sheet](http://k8s.info/cs.html)
- [k8s design docs: Architecture](https://github.com/kubernetes/kubernetes/blob/release-1.3/docs/design/architecture.md) 

시작 전에 간단히 구성 요소를 짚고 넘어가겠습니다. 아래의 설명 대신 위에 있는 링크를 읽고 오셔도 됩니다.

![](https://raw.githubusercontent.com/1ambda/1ambda.github.io/master/assets/images/infra-kubernetes/intro/physical-layout.png)

하나의 k8s 클러스터는 하나의 master 와 여러개의 node 로 구성되어 있습니다. 
개발자는 kubectl 을 이용해서 master 에 명령을 내리고, node 를 관리합니다. 
반면 사용자 (endpoint user) 는 node 에 접속해 서비스를 이용합니다. 

![](https://raw.githubusercontent.com/1ambda/1ambda.github.io/master/assets/images/infra-kubernetes/intro/arc_k8s_simple.jpg)

조금 더 자세히 보면, 

- master 에는 작업을 위한 [api server](http://kubernetes.io/docs/admin/kube-apiserver/), 
state 를 관리하기 위한 분산 저장소 (default 로 [etcd](https://coreos.com/etcd/)), 
[scheduler](http://kubernetes.io/docs/admin/kube-scheduler/), 
[controller manager](http://kubernetes.io/docs/admin/kube-controller-manager/) 등이 있습니다. (현재는 master 가 단일 노드이지만 추후 multi-node master 가 지원될 예정)
- node (= minion) 에는 master 와 통신하는 [kubelet](http://kubernetes.io/docs/admin/kubelet/) (agent, 현재는 containerized 되어있지 않음)이 있고, 
외부의 요청을 처리하는 [kube-proxy](http://kubernetes.io/docs/admin/kube-proxy/), 
container 리소스 모니터링을 위한 [cAdviser](https://github.com/google/cadvisor) 등이 있습니다.

이제 다시 큰 그림에서 보면, 다음과 같습니다.

![](https://raw.githubusercontent.com/1ambda/1ambda.github.io/master/assets/images/infra-kubernetes/intro/arc_official.png)

<br/>

### Pod 

![](https://raw.githubusercontent.com/1ambda/1ambda.github.io/master/assets/images/infra-kubernetes/intro/pods.png)

하나 또는 여러개의 container 묶음을 [pod](http://kubernetes.io/docs/user-guide/pods) 이라 부릅니다. 
docker 에서 container 끼리 통신하려면 같은 network 위에 있도록 구성해야 하는 반면 (compose 도 동일), 하나의 pod 내에 있는 contianer 끼리는 그럴 필요가 없습니다.

- 같은 IP 와 port space 를 가지기 때문에 `localhost` 로 통신이 가능하며
- [volume](http://kubernetes.io/docs/user-guide/volumes/) 을 공유합니다
만약 어느 container 가 죽고 재시작되어도 pod 이 살아있는 한 shared volume 은 유지됩니다. 

> In terms of Docker constructs, a pod is modelled as a group of Docker containers with shared namespaces and shared volumes. PID namespace sharing is not yet implemented in Docker.

pod 의 종료, 삭제 관련해서는 [Termination of Pods](http://kubernetes.io/docs/user-guide/pods/#termination-of-pods) 를 참고하시면 됩니다.

<br/>

위에서 k8s 가 auto-restart 등을 해준다고 했었는데 테스트 해보겠습니다. 그 전에 먼저 클러스터가 정상적으로 세팅이 되었는지 확인해 보겠습니다. 저는 minikube 를 이용해서 로컬에서 실행했으므로 아래와 같은 결과가 나옵니다. 
kubectl 을 여러 클러스터 중 하나에 붙어서 커맨드를 날릴 수 있도록 도와주는 docker-machine 정도로 이해하시면 됩니다. (단위가 다르지만)
 
```
$ kubectl config view

apiVersion: v1
clusters:
- cluster:
    certificate-authority: /Users/1ambda/.minikube/ca.crt
    server: https://192.168.64.2:8443
  name: minikube
contexts:
- context:
    cluster: minikube
    user: minikube
  name: minikube
current-context: minikube
kind: Config
preferences: {}
users:
- name: minikube
  user:
    client-certificate: /Users/1ambda/.minikube/apiserver.crt
    client-key: /Users/1ambda/.minikube/apiserver.key
```

이제 아래와 같이 `nginx.yaml` 을 만들겠습니다.

```yaml
apiVersion: v1
kind: ReplicationController
metadata:
  name: nginx
spec:
  replicas: 3
  selector:
    app: nginx
  template:
    metadata:
      name: nginx
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
```

이제 실행하면,

```
$ kubectl create -f nginx.yaml
replicationcontroller "nginx" created

$ kubectl get replicationcontroller nginx
NAME      DESIRED   CURRENT   AGE
nginx     3         3         41s

$ kubectl get pods
NAME          READY     STATUS    RESTARTS   AGE
nginx-5aa7m   1/1       Running   0          45s
nginx-5frtc   1/1       Running   0          45s
nginx-sg1s5   1/1       Running   0          45s
```

pod 하나를 죽여보겠습니다. 재생성되는걸 확인할 수 있습니다.

```
$ kubectl delete pod nginx-5aa7m
pod "nginx-5aa7m" deleted

$ kubectl get pods
NAME          READY     STATUS              RESTARTS   AGE
nginx-5frtc   1/1       Running             0          1m
nginx-6tub7   0/1       ContainerCreating   0          1s
nginx-sg1s5   1/1       Running             0          1m


$ kubectl get pods
NAME          READY     STATUS    RESTARTS   AGE
nginx-5frtc   1/1       Running   0          1m
nginx-6tub7   1/1       Running   0          5s
nginx-sg1s5   1/1       Running   0          1m
```

<br/>

### Service

![](https://raw.githubusercontent.com/1ambda/1ambda.github.io/master/assets/images/infra-kubernetes/intro/abstraction.png)

pod 은 생성/삭제 될 수 있습니다. [replication-controller](http://kubernetes.io/docs/user-guide/replication-controller/) 를 이용하면 심지어 동적으로도 scale up/down 이 가능한데,
이럴 경우 IP 가 변경/추가/제거 되므로, k8s 는 external <-> pods 이나 pods <-> pods 간의 안정적인 통신을 위해 [service](http://kubernetes.io/docs/user-guide/services/) 라는 object 를 도입했습니다.

> Services provide a single, stable name and address for a set of pods. They act as basic load balancers.
    
위 그림을 잘 보면, pod 에 있는 [label](http://kubernetes.io/docs/user-guide/labels/) 과 동일한 컬러의 것이 service 에도 있고,
해당 service 가 같은 label 컬러를 가진 pod 을 위한 것임을 쉽게 알 수 있습니다.

위에서 생성한 nginx pods 를 위한 service 를 `nginx-svc.yaml` 이란 이름으로 만들어 보겠습니다. `selector` 를 잘 보세요. 

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-svc
spec:
  ports:
    - name: nginx-svc
      port: 8090
      targetPort: 80
  type: NodePort
  selector:
    app: nginx
```

이제 생성하면,  `nginx-svc` service 의 EXTERNAL-IP 가 `<nodes>` 로 보입니다. 
전체 node 에 대해서 외부 포트를 열어서 그런데, 
[NodePort](http://kubernetes.io/docs/user-guide/services/#type-nodeport) 
대신 [LoadBalancer](type-loadbalancer) (cloud provider 가 지원할 경우만 사용가능) 타입을 이용하거나 [ingress](http://kubernetes.io/docs/user-guide/ingress/) 를 이용할 수 있습니다.

> As of Kubernetes v1.0, Services are a “layer 3” (TCP/UDP over IP) construct. In Kubernetes v1.1 the Ingress API was added (beta) to represent “layer 7” (HTTP) services.
  
```
$ kubectl create -f nginx-svc.yaml
service "nginx-svc" created

$ kubtctl get service
NAME         CLUSTER-IP   EXTERNAL-IP   PORT(S)    AGE
kubernetes   10.0.0.1     <none>        443/TCP    3d
nginx-svc    10.0.0.196   <nodes>       8090/TCP   1m

$ kubectl describe service nginx-svc
  ...
  Endpoints:              172.17.0.3:80,172.17.0.4:80,172.17.0.5:80
  
$ kubectl get node
NAME          STATUS    AGE
boot2docker   Ready     3d

$ kubectl describe node boot2docker | grep Address
Addresses:              192.168.64.2,192.168.64.2

$ curl 192.168.64.2:31968

 <!DOCTYPE html>
 <html>
 <head>
 <title>Welcome to nginx!</title>
  ...
```

**192.168.64.2:31968** 을 접근해보면, nginx 가 떠있음을 확인할 수 있습니다.

<br/>

pod 이 생성될때 active service 에 대해서 kubelet 이 service 의 IP, port 와 관련된 환경변수를 pod 에 주입합니다. (service 가 먼저 생성되어 있어야 함) 
예를 들어 service name 이 `redis-master` 라면 다음과 같은 값들이 주입됩니다.
  
```
REDIS_MASTER_SERVICE_HOST=10.0.0.11
REDIS_MASTER_SERVICE_PORT=6379
REDIS_MASTER_PORT=tcp://10.0.0.11:6379
REDIS_MASTER_PORT_6379_TCP=tcp://10.0.0.11:6379
REDIS_MASTER_PORT_6379_TCP_PROTO=tcp
REDIS_MASTER_PORT_6379_TCP_PORT=6379
REDIS_MASTER_PORT_6379_TCP_ADDR=10.0.0.11
```

하여 pod 간 통신에는 env variable 을 이용할 수 있으나, [DNS](http://kubernetes.io/docs/user-guide/services/#dns) 를 이용하는 것이 더 권장됩니다.

추가적으로, label 값은 [kubernetes/example/guestbook](https://github.com/kubernetes/kubernetes/tree/master/examples/guestbook) 처럼 붙이는 것이 권장됩니다.

```yaml
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: redis-master
spec:
  replicas: 1
  template:
    metadata:
      labels:
        app: redis
        role: master
        tier: backend
```

<br/>

### Deployment

[deployment](http://kubernetes.io/docs/user-guide/deployments/) 는 [Kubernetes: Hello World Walkthrough](http://kubernetes.io/docs/hellonode/) 를 진행하셨다면 감이 오셨을수도 있겠습니다.
rolling update, rollback 등을 지원하는 pod, replica set 입니다. 

```yaml
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 3
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.7.9
        ports:
        - containerPort: 80
```

위와 같이 `nginx-deployment.yaml` 을 만들고, 배포하면

```
$ kubectl delete replicationcontroller nginx
$ kubectl get pod

$ kubectl create -f nginx-deployment.yaml
deployment "nginx-deployment" created

$ kubectl get deployment
NAME               DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   3         3         3            3           8h

$ kubectl get rs
NAME                          DESIRED   CURRENT   AGE
nginx-deployment-1159050644   3         3         8h

$ kubectl get pods
NAME                                READY     STATUS    RESTARTS   AGE
nginx-deployment-1159050644-1bhfw   1/1       Running   0          8h
nginx-deployment-1159050644-hxaxs   1/1       Running   0          8h
nginx-deployment-1159050644-zc5tc   1/1       Running   0          8h

$ kubectl rollout status deployment/nginx-deployment
  deployment nginx-deployment successfully rolled out
```

정상적으로 배포되었는지는 **rollout status** 커맨드를 이용해서 확인할 수 있습니다.

이제 container 의 nginx 버전을 올려보겠습니다. 

```
$ kubectl set image deployment/nginx-deployment nginx=nginx:1.9.1
deployment "nginx-deployment" image updated

$ # also, we can use `edit` 
$ # kubectl edit deployment/nginx-deployment

$ kubectl rollout status deployment/nginx-deployment
deployment nginx-deployment successfully rolled out

$ kubectl get deployment
NAME               DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   3         3         3            3           9h

$ kubectl get rs
NAME                          DESIRED   CURRENT   AGE
nginx-deployment-1159050644   0         0         8h
nginx-deployment-671724942    3         3         1m
```

모든 pod 을 한번에 생성하고, 한번에 오래된 pod 을 죽이는 방식으로 일어나는 것이 아니라 rolling update 처럼 하나하나씩 진행됩니다.

> Deployment can ensure that only a certain number of Pods may be down while they are being updated. By default, it ensures that at least 1 less than the desired number of Pods are up (1 max unavailable).  

> Deployment can also ensure that only a certain number of Pods may be created above the desired number of Pods. By default, it ensures that at most 1 more than the desired number of Pods are up (1 max surge).

```
$ kubectl get pods
NAME                               READY     STATUS    RESTARTS   AGE
nginx-deployment-671724942-1hytl   1/1       Running   0          41s
nginx-deployment-671724942-7a0f1   1/1       Running   0          41s
nginx-deployment-671724942-jzgm0   1/1       Running   0          40s

$ kubectl describe deployments
Name:                   nginx-deployment
Namespace:              default
CreationTimestamp:      Sun, 07 Aug 2016 00:48:18 +0900
Labels:                 app=nginx
Selector:               app=nginx
Replicas:               3 updated | 3 total | 3 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  1 max unavailable, 1 max surge
OldReplicaSets:         <none>
NewReplicaSet:          nginx-deployment-671724942 (3/3 replicas created)
Events:
  FirstSeen     LastSeen        Count   From                            SubobjectPath   Type        Reason                   Message
  ---------     --------        -----   ----                            -------------   --------    ------                   -------
  51s           51s             1       {deployment-controller }                        Normal      ScalingReplicaSet        Scaled up replica set nginx-deployment-671724942 to 1
  51s           51s             1       {deployment-controller }                        Normal      ScalingReplicaSet        Scaled down replica set nginx-deployment-1159050644 to 2
  51s           51s             1       {deployment-controller }                        Normal      ScalingReplicaSet        Scaled up replica set nginx-deployment-671724942 to 2
  50s           50s             1       {deployment-controller }                        Normal      ScalingReplicaSet        Scaled down replica set nginx-deployment-1159050644 to 1
  50s           50s             1       {deployment-controller }                        Normal      ScalingReplicaSet        Scaled up replica set nginx-deployment-671724942 to 3
  50s           50s             1       {deployment-controller }                        Normal      ScalingReplicaSet        Scaled down replica set nginx-deployment-1159050644 to 0
```

보면 replica set 은 2개지만 `nginx-deployment-1159050644` 은 업데이트 전 버전인 1.7.9 pod 이 하나도 없고 (**desired = 0**), `nginx-deployment-671724942` 만 3 개의 pod 을 가지고 있습니다. 
이전 replica set 을 유지하는 이유는 rollback 을 위해서 인데요
  
```
$ kubectl rollout history deployment/nginx-deployment
deployments "nginx-deployment":
REVISION        CHANGE-CAUSE
1               <none>
2               <none>
```

여기서 `CHANGE-CAUSE` 가 **none** 인 이유는 deployment 커맨드 이용해 **\-\-record** 를 사용하지 않아서 그렇습니다. revision 값을 지정해 살펴보면

```
$ kubectl rollout history deployment/nginx-deployment --revision=1
deployments "nginx-deployment" revision 1
  Labels:       app=nginx
        pod-template-hash=1159050644
  Containers:
   nginx:
    Image:      nginx:1.7.9
    Port:       80/TCP
    Environment Variables:      <none>
  No volumes.
  
$ kubectl rollout history deployment/nginx-deployment --revision=2
deployments "nginx-deployment" revision 2
  Labels:       app=nginx
        pod-template-hash=671724942
  Containers:
   nginx:
    Image:      nginx:1.9.1
    Port:       80/TCP
    Environment Variables:      <none>
  No volumes.
```

이제 rollback 해 보겠습니다.
  
```
$ kubectl rollout undo deployment/nginx-deployment --to-revision=1
deployment "nginx-deployment" rolled back

$ kb get pods
NAME                                READY     STATUS              RESTARTS   AGE
nginx-deployment-1159050644-mg5e9   1/1       Running             0          2s
nginx-deployment-1159050644-vsoaw   1/1       Running             0          2s
nginx-deployment-1159050644-x0319   0/1       ContainerCreating   0          1s
nginx-deployment-671724942-1hytl    1/1       Terminating         0          23m

$ kb get deployment
NAME               DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   3         3         3            3           9h

$ kb get rs
NAME                          DESIRED   CURRENT   AGE
nginx-deployment-1159050644   3         3         9h
nginx-deployment-671724942    0         0         24m
```

이전 pod 이 죽고, 새로운 pod 이 생성되는 과정을 볼 수 있습니다.

<br/>

### Pet Set

stateless application 의 경우에는 기존의 pod, replica set 등을 이용해 쉽게 배포하고 확장할 수 있었지만,
 cluster 로 동작하는 경우에는 각 instance 간 networking 을 위해 reliable name (e.g index based, advertised hostname) 등의 기능이 필요했습니다.  

[pet set](http://kubernetes.io/docs/user-guide/petset/) 은 stateful (e.g clustering) applications 의 지원을 위해 1.3 버전에서 alpha 기능으로 추가되었습니다. (see [CHANGELOG#1.3](https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG.md#v130))

[1000 Instances of Cassandra using Kubernetes Pet Set](http://blog.kubernetes.io/2016/07/thousand-instances-of-cassandra-using-kubernetes-pet-set.html) 에서는 alpha 기능인 pet set 을 이용해 1000 개의 카산드라 인스턴스를 배포하고 sample job 을 돌린 경험을 공유하고 있습니다. 

> We deployed 1,000 pets. Technically with the Cassandra setup, **we could have lost 333 nodes without service or data loss**.  
> **8,072 Cores**. The master used 24, minion nodes used the rest  
> **100,510 GB** persistent disk used by the Minions and the Master  
> **380,020 GB SSD** disk persistent disk. 20 GB for the master and 340 GB per Cassandra Pet.  

<br/>

팀 내에서 20+ broker 로 kafka cluster 를 운영하고 있는데다가 container 기반 기술에 관심이 많아 적용을 해보고 싶긴 한데,
아직 레퍼런스가 없어 안타깝습니다. github issue 를 보면 example 을 만들기 위해 논의는 진행중인것 같습니다.

- [kubernetes #5017: Example: Kafka/Zookeeper](https://github.com/kubernetes/kubernetes/issues/5017)
- [SO: Kafka on k8s multi node](http://stackoverflow.com/questions/32140025/kafka-on-kubernetes-multi-node)
- [Running a ZK and kafka clusters on k8s](http://www.defuze.org/archives/351-running-a-zookeeper-and-kafka-cluster-with-kubernetes-on-aws.html)
- [Github: kubernetes-kafka](https://github.com/CloudTrackInc/kubernetes-kafka/issues)

[Github: kubernetes-contrib/pets](https://github.com/kubernetes/contrib/tree/master/pets) 를 보시면 pet set 을 이용해 cluster 를 구성하는 샘플이 몇개 있습니다. 
현재까지는 mysql, redis, ZK 정도가 있네요. [Github: kubernetes/examples](https://github.com/kubernetes/kubernetes/tree/master/examples) 도 다양한 샘플이 있으므로 한번 쭈욱 둘러보시면 도움이 될듯 합니다.

<br/>

### Other Objects

- [Volume](http://kubernetes.io/docs/user-guide/volumes/), [Persistent Volume](http://kubernetes.io/docs/user-guide/persistent-volumes/)

위에서는 pod 과 volume 의 life cycle 이 동일하다고 했지만, 사실 꼭 그렇지는 않습니다. 다양한 type 의 volume 이 지원되기 때문인데요,
`emptyDir` 이외의 volume type 을 이용하면 pod 이 죽더라도, 데이터를 유지할 수 있습니다. (e.g [gcePersistentDisk](http://kubernetes.io/docs/user-guide/volumes/#gcepersistentdisk), [flocker](https://clusterhq.com/flocker/introduction/))

persistent volume (PV) 는 일종의 networked storage 로 pod lifecycle 을 벗어나 존재할수 있으면서도, 사용자가 리소스의 양을 특정지어 요청할 수 있는 volume 입니다.
 
> Managing storage is a distinct problem from managing compute. The PersistentVolume subsystem provides an API for users and administrators that abstracts details of how storage is provided from how it is consumed

- [Daemon Set](http://kubernetes.io/docs/admin/daemons/)

daemon set 은 node 마다 추가되야 하는 프로세스가 있을때 사용할 수 있습니다.

> running a cluster storage daemon, such as **glusterd**, **ceph**, on each node  
> running a logs collection daemon on every node, such as **fluentd** or **logstash**  
> ...

- [Job](http://kubernetes.io/docs/user-guide/jobs/) 

배치 작업처럼, 실행후 종료되는 Job 을 의미합니다.

기타 오브젝트는 [Reference](http://kubernetes.io/docs/reference/) 의 [Glossary](http://kubernetes.io/docs/user-guide/annotations/) 를 참조하시면 됩니다.

<br/>

### Etc

A. Log Aggregation 은 아래의 내용을 참고하실 수 있습니다. 

- http://kubernetes.io/docs/getting-started-guides/logging/
- https://github.com/kubernetes/kubernetes/issues/1071
- https://github.com/kubernetes/kubernetes/issues/24677

B. k8s cluster 간 연결은 [k8s cluster federation](https://github.com/kubernetes/kubernetes/blob/master/docs/proposals/federation.md) 이라 불립니다.  

C. k8s production 적용을 위한 [guide](http://kubernetes.io/docs/user-guide/production-pods/#liveness-and-readiness-probes-aka-health-checks) 도 있습니다.

D. 이런 저런 CI 를 테스팅 해봤는데 [Circle CI](circleci.com) 가 썩 괜찮습니다. 
싼 가격에, 연동 잘되고 기능 많습니다. 아래와 같이 build (scala) 세팅하면 [google container registry](https://cloud.google.com/container-registry/) 에 push 하고 
gcloud 커맨드 까지 직접 내릴 수 있으니, develop branch 정도라면 k8s [deployment](kubernetes.io/docs/user-guide/deployments/) 이용해서 바로 롤링 업그레이드 가능합니다. (만약 SBT 가 느리다면 [courseir](https://github.com/alexarchambault/coursier) 나 circle CI build cache 등을 이용해보세요.) 

```yaml
machine:
  environment:
    SBT_VERSION: 0.13.8
    SBT_OPTS: "-Xms512M -Xmx1536M -Xss1M -XX:+CMSClassUnloadingEnabled
-XX:MaxPermSize=256M"
    PROJECT_NAME: dmm-common
  services:
    - docker
  java:
    version: oraclejdk8

dependencies:
  cache_directories:
    - "~/.sbt"
  pre:
    # install SBT
    - wget --output-document=$HOME/bin/sbt-launch.jar
      https://repo.typesafe.com/typesafe/ivy-releases/org.scala-sbt/sbt-launch/"$SBT_VERSION"/sbt-launch.jar
    - echo "java $SBT_OPTS -jar \`dirname \$0\`/sbt-launch.jar \"\$@\"" > $HOME/bin/sbt
    - chmod u+x $HOME/bin/sbt

    # install gcloud SDK, kubectl
    - sudo /opt/google-cloud-sdk/bin/gcloud --quiet components update
    - sudo /opt/google-cloud-sdk/bin/gcloud --quiet components update kubectl

  override:
    - sbt sbt-version

test:
  override:
    - sbt test
    - sbt docker

deployment:
  development:
    branch: /feature.*/
    commands:
      - echo $GCE_KEY > gcloud-key.json
      - gcloud auth activate-service-account --key-file gcloud-key.json
      - docker tag 1ambda/sample gcr.io/1ambda/sample:$CIRCLE_SHA1
      - gcloud docker push gcr.io/1ambda/sample:$CIRCLE_SHA1
```

## References

- [Title Image](http://devops.com/2015/11/09/9-more-open-source-devops-tools-we-love/)
- [What is k8s?](http://kubernetes.io/docs/whatisk8s/)
- [k8s.info](http://k8s.info/cs.html)
- [SO: How to expose a Kubernetes service externally using NodePort](http://stackoverflow.com/questions/33970251/how-to-expose-a-kubernetes-service-externally-using-nodeport)
- [k8s pod image](http://nshani.blogspot.kr/2016/02/getting-started-with-kubernetes.html)
- [k8s architecture slide1](http://www.slideshare.net/erialc_w/kubernetes-50626679)
- [k8s architecture slide2](http://www.slideshare.net/imesh/apache-stratos-410-architecture)
- [1000 Instances of Cassandra using Kubernetes Pet Set](http://blog.kubernetes.io/2016/07/thousand-instances-of-cassandra-using-kubernetes-pet-set.html)
- [k8s design docs: Architecture](https://github.com/kubernetes/kubernetes/blob/release-1.3/docs/design/architecture.md) 

<br/>