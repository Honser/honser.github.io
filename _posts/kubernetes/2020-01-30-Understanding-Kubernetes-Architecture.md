---
layout: post
title:  "Understanding Kubernetes Architecture"
description: ""
categories: Kubernetes
tags: [Kubernetes]
comments: true
---

![kubernetes_architecture]({{ site.baseurl }}/images/kubernetes_components2.png#center)

Kubernetes를 배포 할 때 클러스터를 구성하게 되는데, 최소 하나의 Master Node와 Worker Node가 포함된다. Master Node가 Worker Node와 전체 Pod들을 관리하는데 HA를 위해 Production 목적에서는 최소 3개(three-node cluster)로 구성하는 편이다.
<br>
<br>

## Master Node(Control Plane)

### 1. kube-apiserver
Kubernetes 내부의 모든 Operation이나 Communication은 REST API를 통해 호출되는데, 대부분의 Operation은 kubectl CLI를 통해 수행되고 kubeadm(Kubernetes 환경 구성 할 때) 같은 다른 Command-line tool이나 Dashboard(GUI)도 쓰인다. 물론 REST Call을 직접 사용해 API에 접근도 가능하다.


### 2. kube-controller-manager
간단하게 Cluster 전체의 객체와 자원들을 모니터링하여 Desired State가 유지되도록 관리하는 역할이다. 하나의 binary file로 실행되지만 세부적으로는 아래처럼 여러 Controller로 구성된다.
<br>
* **Node Controller** : Node 상태 모니터링
* **Replication Contoller** : 사용자가 정의한 개수만큼의 Pod을 유지
* **Endpoints Controller** : 객체들의 endpoint 관리
* **Service Account & Token Controller** : 새로운 namespace에 대한 default 계정과 API access 토큰 생성

>**Note**:  
```Desired State```  
Kubernetes에서는 사용자로부터 요청 받은 Workflow에 따라 하나 하나 Action을 취하는 것이 아니라, 사용자가 원하는 상태(Desired State)를 미리 전달하고 Kubernetes가 Current State를 계속 감시하고 있다가 Desired State와 다른 상태가 되었을 때 자체적으로 Action을 취해 원하는 상태로 유지하는 방식으로 동작한다.
<br>

### 3. cloud-controller-manager
원래 Kube Controller Manager에 포함되어 있었지만 1.6 버전에서 따로 도입됐고, Cloud 공급자가 Plugin을 통해서 Kubernetes와의 통합을 좀 더 손쉽게 할 수 있도록 돕기 위해 만들어졌다.
<br>
* **Node Controller**: Cloud 상에서 Node를 관리하기 위해 사용
* **Route Controller**: Cloud 인프라 환경에서 Routing 구성
* **Service Controller**: Cloud가 제공하는 load balancer Create/Update/Delete 관리
* **Volume Controller**: Cloud가 제공하는 Volume의 Creat/Attach/Mount

### 4. kube-scheduller
가용 노드 안에서 Pod Placement를 담당하고 Node의 자원 상황에 따라 적절하게 Pod을 분배하여 Scheduling 한다. Affinity나 Anti-Afiinity Rule에 따라서 예외적으로 원하는/원하지 않는 Node/Pod을 지정하여 스케쥴링 할 수도 있다.

### 5. etcd
Cluster 내 모든 데이터를 관리하는 Key-value store 방식의 저장공간. 가용성이나 복구 관점에서 생각 할 때 제일 신경써야 되는 Component 이기 때문에, 여유가 있다면 Production grade에서 Five node로 구성하는 게 best라고 Kubernetes 공식 블로그에는 적혀있지만 경험상 three node 정도로만 쓰이는 듯하다. 
<br>
<br>

## Worker Node

### 1. container runtime
실질적으로 container를 띄우는 역할. 일단은 OpenStack에서의 Hypervisor 같은 역할로 생각해두자. Kubernetes에서는 Docker가 default로 쓰이고 있지만, 여러 platform에서 cotainerd나 CRI-O로 넘어가는 추세다.

### 2. kubelet
Master와 통신하는 Node agent. Pod에 속하는 Container들이 항상 유지될 수 있게 하는 역할을 하고, 당연한 얘기지만 kubernetes를 통해 생성된 container만 관리한다.

### 3. kube-proxy
Kubernetes는 Cluster 내부에 별도의 가상 네트워클르 설정하고 관리하는데, kube-proxy가 가상 네트워크를 구성하고 관리하는 실질적인 Process다. Host의 network rule도 관리하며, API Server한테 Service가 추가되거나 삭제되는 정보를 받아서 route를 갱신한다.
<br>
<br>

## Addons
말그대로 추가로 사용 가능한 Component들인데 대표적으로 명시되어 있는 건 아래 네 가지이고, >다른 것들은 필수는 아니지만 DNS server는 꼭 있어야 하는데 kubeadm과 함께 default로 설치된다. 그리고 1.11 이후 버전에서는 CoreDNS가 권장된다.
* **DNS**
* **Web UI(Dashboard)**
* **CRM(Container Resource Monitoring)**
* **Cluster-level Logging**
<br>
<br>
