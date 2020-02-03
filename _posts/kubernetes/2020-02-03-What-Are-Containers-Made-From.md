---
layout: post
title:  "What Are Containers Made From"
description: ""
categories: Kubernetes
tags: [Docker,Container]
comments: true

---



Docker와 함께 떠오른 Container 기술은 사실 훨씬 오래 전부터 존재했다. 하지만 Docker가 곧 Container다 라고 아는 사람이 많았을 정도로 Docker는 Container 시대 부흥의 시작을 열었고 산업을 주도하는 container 플랫폼이어왔다. 지금은 그 역할을 Kubernetes가 넘겨받았다고 해야겠지만.

![docker_components]({{ site.baseurl }}/images/docker_gt.jpg#center)


<br>
<br>
## Ground Technologies
- #### CGroup
- #### Namespace
- #### Union File System

<br>
Container라는 격리된 공간을 만들어주는 데에 가장 핵심 역할을 하는 기술은 바로 cgroup과 namespace. 간단하게는 namespace가
칸막이를 쳐서 Filesystem, PID, Hostname 등이 Host나 다른 Container와 충돌하지 않게 독립된 공간을 만들어주고, cgroup이
CPU, Memory, I/O 등의 자원을 할당해주는 방식이다. VM처럼 hardware-level 가상화가 아닌 OS-level로, Host OS에 종속적이고
VM에 비해 독립성 측면에서는 부족하지만, 벽을 시공해서 막는 느낌이 아니라 간이 칸막이를 세우는 느낌이라 여러 가지 의미로
훨씬 가벼운(?) 용도로 쓸 수 있다. 위의 그림에 나와있는 libcontainer는 밑에 적을 kernel feature들을 가지고 container를 만드는 역할을 하는 container runtime인데 이후에 따로 정리하려고 한다.

각각 좀 더 자세히 보자면,



## 1. CGroup

cgroup(Control Group)은 단일 Process나 Process 그룹에 대한 자원 할당을 제어하는 기술이며, limit, priority를 부여해서
Process group의 자원 사용을 제어하고 사용량 측정이나 Process 동작 제어(일시 중단) 등이 가능하다. CPU로 예를 들면, 특정 자원만 사용하게 한다던가(cgroup0 -> CPU0), 두 개 이상의 group이 있을 때 ratio를 지정해서 비율 조정(A:B:C = 1:1:2),  hard limit을 지정해서(10000ms 마다 200ms만 사용) 쓸 수 있다.

cgroup은 아래 표처럼 각 자원을 담당하는 subsystem들로 구성된다.

| subsystem | description                                                  |
| --------- | ------------------------------------------------------------ |
| cpu       | CPU 자원 사용 제어                                           |
| cpuacct   | 그룹 단위 CPU 자원 사용 통계 제공                            |
| cpuset    | 개별 processor 및 memory node를 cgroup에 binding             |
| memory    | memory 사용을 제어하고 통계 제공                             |
| blkio     | block device에 대한 I/O 사용량을 제어하고 측정               |
| devices   | 그룹의 device 파일 Read/Write 권한 제어                      |
| freezer   | cgroup에 대한 작업을 cgroup의 작업을 일시적으로 중지.  cluster batch scheduling, process migration, 그리고 debugging 등을 위함. |
| net_cls   | packet에 tag(classid)를 붙여 cgroup을 식별해서 traffic 제어 |
| net_prio  | network interface 별 우선순위를 부여해서 traffic 제어 |
| hugetlb   | Hugepage 사용 제어                                           |
| pid       | cgroup 내에서 생성 될 수 있는 process 개수 제어              |

![cgroup_cli]({{ site.baseurl }}/images/cgroup_cli.png#center)
cgroups of Ubuntu 18.04
{: style="font-size: 80%; text-align: center;"}

*****


## 2. Namespace

외부와 격리된 공간을 만들어서 작업 공간을 독립시켜주는 kernel feature로, mount 경로만 다르게 보여주는 chroot와는 달리
아래 표처럼 더 많은 영역에 대해서 가능하다. Namespace 덕분에 동일한 PID나 hostname 등을 여러 container에서 사용 할 수
있다.

| type                             | description                                 |
| -------------------------------- | ------------------------------------------- |
| PID(Process)                     | process ID                                  |
| NET(Network)                     | network resources(device, stack, port, ...) |
| MNT(Mount)                       | file system mount points                    |
| IPC(Inter-Process Communication) | SystemV IPC, POSIX message queues           |
| UTS(Unix Time Sharing)           | hostname, NIS domain name                   |
| USER                             | UID, GID                                    |
| CGROUP                           | cgroup                                      |

![namespace_cli]({{ site.baseurl }}/images/namespace_cli.png#center)
namespaces of Ubuntu 18.04
{: style="font-size: 80%; text-align: center;"}  

*****

## 3. Union File System
UFS는 말 그대로 여러 filesystem들을 묶어서 사용하는 것이다. Docker에서 container의 filesystem은 container 내부에서 보면 평범해보이지만, 사실 UFS를 통해서 한 개 이상의 image(read only)와 하나의 container layer(R/W)의 조합이 하나의
Filesystem이 된 것이다. Union mount를 지원하는 filesystem은 AUFS, Device Mapper 등 더 많이 있지만 OverlayFS가 현재 가장 많이 쓰이고, storage driver로 overlay2가 쓰인다.
![docker_image_struct]({{ site.baseurl }}/images/docker_image_struct.png#center)

위의 그림처럼 web app이라는 image로 container를 띄우면, base image인 ubuntu부터 nginx, web app source layer를 모두
read-only로 mount 시키고 그 위에 container layer만 하나 더 갖는 형태의 구조를 갖게 된다. 여기서 새로운 파일을 추가하거나 기존 파일을 수정하거나 삭제하는 변경사항들은 container layer에만 기록이 된다. 새로 추가하는 건 특별할 게 없고, 기존 파일을 수정하는 과정은 CoW 방식으로 lower layer에서 복사해온 다음에 그 부분에 수정을 해서 사용하게 되고, 기존 파일의 삭제는 whiteout 파일로 lower layer의 해당 파일을 가려둬서 container에서 그 파일의 존재를 없애는 방식으로 동작한다.

>**Note**:  
>```CoW(Copy-On-Write)```  
>미리 다 복사해두지 말고 Write 동작 시에 Copy 시에 필요한 부분만 복사해서 수정하는 기법이다.
>![cow]({{ site.baseurl }}/images/cow.jpg#center)
>위의 그림처럼 fork()가 호출 됐을 때 child process가 parent의 자원을 복제해서 갖는 것이 아니라 page를 공유해서 사용하게 하고 page C를 수정 했을 때 해당 page를 수정하는 것이 아니라 복사본에 쓰게 하고 process의 pointer를 copy를 가르키도록 변경시켜준다. 이렇게 되면 process2는 아무런 간섭도 받지 않게 된다.
>
>```Whiteout File & Opaque Dir```  
> whiteout은 file, opaque는 directory 대상일 뿐 동일한 역할을 하는데, container에서 file이나 directory를 지웠을 때 실제로 해당 image 파일을 지우는 것이 아니라 지워진 파일/폴더라고 마킹하고 지운 것처럼 처리한다.

*****


## Reference

<https://medium.com/@nagarwal/understanding-the-docker-internals-7ccb052ce9fe>
<https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/resource_management_guide/index>
<https://www.terriblecode.com/blog/how-docker-images-work-union-file-systems-for-dummies/>
<https://blog.knoldus.com/unionfs-a-file-system-of-a-container/>
{: style="font-size: 80%;"}  
