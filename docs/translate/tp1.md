# 【翻译计划1】Know Kubernetes — Pictorially

Created: Feb 26, 2021 7:44 PM
Year: 2019
link: https://medium.com/tarkalabs/know-kubernetes-pictorially-f6e6a0052dd0
writer: Sudhakar Rayavaram
标签: kubernetes, 技术

上一次说到翻译计划，似乎还是16年在点石团队的时候。“Open Eyes Project”，那时候缺乏了推动力。这次重新开始，翻译一些技术文章或者，感兴趣的内容。暂时不给他一个名字吧。`新翻译计划` 它来了。这是本系列的第一篇翻译文章。

这篇文章发表在2019年发表在medium上，作者是Sudhakar Rayavaram，主要目的通过图片漫画语言更好的帮助大家理解k8s的基本组件。而翻译他的初衷也是因为，译者正在学习k8s相关的内容。

# 通过漫画-认识Kubernetes

最近，我开始了我的Kubernetes之旅，希望更好的去理解它的内部原理。我做了一个关于这个话题的演讲，这里是他的博客版本。

## Container

在我们弄清楚什么是Kubernetes之前，我们花一点时间来弄清楚容器到底是什么，以及为什么它们这么受欢迎。毕竟在不知道什么是容器的情况下，谈论容器编排(containers orchestrator, Kubernetes)是没有任何意义的。

![https://dist.lyneee.com/blog/2021030300know-k8s-p1-container.png](https://dist.lyneee.com/blog/2021030300know-k8s-p1-container.png)

Container 容器

"Container"即是一个容器，……用来装所有你放进去的东西。

这些东西包括了你的应用程序代码、依赖库以及以及它所依赖的一些环境直到内核。这中间的关键则是隔离。把你所有的东西和其他的环境、程序等等隔离开，这样你就可以更好的控制他们。

- Workspace isolation(Process, Network) - 工作区隔离（进程，网络）
- Resource isolation(CPU, Memory) - 资源隔离（CPU，内存）
- File System isolation(Union File System$ ^1$, Union FS) - [文件系统隔离（联合文件系统）](https://en.wikipedia.org/wiki/UnionFS)

把容器看作精简的虚拟机VM。它们精简，能够快速启动，而且这一切都不是从零建立起来的。相反，它们使用了linux中已有的工具（如cgroups, namespaces）建立了一个很好的抽象层。
>

现在，我们已经知道容器是什么了，很容易理解为什么它们这么受欢迎。相比于仅仅发布你的二进制应用程序或者代码，这使得发布能够运行你的程序的整个环境成为了可能，因为容器可以被构建为非常小的单元。这完美修复了“It works in my machine”的问题。

## 什么时候使用Kubernetes？

容器一切都不错，软件开发人员的日子也比以前更好过了。可是，为什么我们还需要像Kubernetes这样的容器编排技术呢？

![https://dist.lyneee.com/blog/2021030300know-k8s-p2-toomany-containers.png](https://dist.lyneee.com/blog/2021030300know-k8s-p2-toomany-containers.png)

too-many-containers

当你达到上图这种状态时，你就需要他了，因为你有太多的容器需要管理。

Q：我的前端容器在哪里，我运行了多个前端容器？

A：很难回答。请使用容器编排。

Q：如何让我的前端代码容器和后端容器进行通讯？

A：很难回答。或者你可以使用容器编排。

Q：如何进行滚动升级(rolling upgrade)？

A：很难回答。请使用容器编排。

## 为什么我更喜欢Kubernetes？

有很多容器编排器可以选择，比如docker swarm、Mesos、Kubernets。我的选择是Kubernetes，因为Kubernetes像是……

![https://dist.lyneee.com/blog/2021030720know-k8s-p3-likeblocks.png](https://dist.lyneee.com/blog/2021030720know-k8s-p3-likeblocks.png)

like blocks

……乐高积木一样。它不仅有运行大规模容器编排的组建，同时也能够灵活地对不同组建进行客制化的替换。当你需要一个自定义的调度器，当然只需要插入其中即可。需要一个新的资源类型？写一个CRD$^2$([CustomResourceDefinition](https://jimmysong.io/kubernetes-handbook/concepts/crd.html))就可以。此外，Kubernetes社区非常的活跃，各种工具也在迅猛的发展。

## Kubernetes的架构

![https://dist.lyneee.com/blog/2021030720know-k8s-p4-Architecture.png](https://dist.lyneee.com/blog/2021030720know-k8s-p4-Architecture.png)

Architecture

每一个Kubernetes集群都有两种节点（机器）：主节点（Master）和Worker节点（Worker）。主节点，工作节点。顾名思义，主节点是控制和监控集群即工作节点上运行的负载情况和应用程序。

一个集群可以只有单个的节点，但是最好还是有三个节点来确保高可用性（即所说的HA Clusters，高可用集群）。

然后我么仔细看看**主节点（Master）**他是怎么构成的：

![https://dist.lyneee.com/blog/2021030721know-k8s-p5-master.png](https://dist.lyneee.com/blog/2021030721know-k8s-p5-master.png)

master node

**etcd**：用于存储Kubernetes对象的所有数据、它们的状态、访问信息以及其他节点的配置信息。

**API Server：**RESTFul API服务器暴露了End Points以便于控制集群。几乎主节点工作节点上的所有组建都会于该服务器通讯以执行它们各自的功能。

**Scheduler**：调度器，决定了应用需要在哪一台机器上运行。

**control manager**： 控制器，是一个控制循环，它监视集群的状态（通过从API Server调取的数据），并采取操作保证达到预期的状态。

**工作节点：**

![https://dist.lyneee.com/blog/2021030721know-k8s-p6-worker.png](https://dist.lyneee.com/blog/2021030721know-k8s-p6-worker.png)

worker node

**kubelet：**是工作节点的心脏。它与主节点的API服务器通信，并为其节运行容器调度器。

**kube Proxy：**使用IP表/IPVS来处理pods的网络需求。

**Pod：**Kubernetes中的执行者，它运行了你的所有容器。没有POD的结构，你不能在Kubernetes中运行容器。Pod增加了让Kuberenetes在容器之间建立网络的方式至关重要的功能

![https://dist.lyneee.com/blog/2021030721know-k8s-p7-pod.png](https://dist.lyneee.com/blog/2021030721know-k8s-p7-pod.png)

Pod

一个Pod可以有多个容器，并且所有运行在这些容器中的服务器能够把彼此看作本地主机。这使得我们能够非常方便的将应用程序的不同服务分割到单个容器中，然后将他们装载到一个Pod中。Pod的模式也有多种，比如sidecar、proxy以及ambassador以方便满足不同的需求。可以查看这篇文章来了解更多关于Pod模式的信息：[Multi-Container Pod Design Patterns in Kubernetes](https://matthewpalmer.net/kubernetes-app-developer/articles/multi-container-pod-design-patterns.html)$^3$。

Pod网络接口提供了一种机制，能够将同一节点中的其他pod或者其他工作节点中的Pod联网。

![https://dist.lyneee.com/blog/2021030722know-k8s-p8-inside-pod.png](https://dist.lyneee.com/blog/2021030722know-k8s-p8-inside-pod.png)

pod network

此外，每个pod都会被分配自己的IP地址，能够被kube-proxy用于路由通信。这些IP地址只能够在集群中可见。

pod中挂在的数据卷（volume）也是对所有容器可见的，有时这些数据卷可以被用于不同pod之间的异步通信。举一个例子，假设你的应用是一个图片上传应用（可能像Instagram一样），它能够保存这些文件在数据卷中，同一个pod中的其他容器能够监视数据卷中的新文件，并将他们处理成多种大小格式的文件，并上传到云存储中。

## Controllers 控制器

在Kubernetes中，有很多种的控制器，比如ReplicaSet、Replication Controllers、Deployments、statefulset和Service。这些对象能够以这样或那样的方式控制Pod。让我们看看其中相对重要的一些。

### ReplicaSet

![https://dist.lyneee.com/blog/2021030722know-k8s-p9-replicaset.png](https://dist.lyneee.com/blog/2021030722know-k8s-p9-replicaset.png)

ReplicaSet的主要工作就是复制pod

这个控制器的主要职责是给特定的pod创建副本。如果一个pod因为某种愿意死亡了，控制器会接收到通知，并立即开始行动产生一个新的pod。

### Deployment

![https://dist.lyneee.com/blog/2021030722know-k8s-p10-deployment.png](https://dist.lyneee.com/blog/2021030722know-k8s-p10-deployment.png)

Deployment通过控制ReplicaSet

Deployment是一个高阶对象，它使用ReplicaSet去管理副本。它能够通过产生一个新的ReplicaSet来，逐渐缩小（最终删除）现有的ReplicaSet来实现滚动升级。

### Service

![https://dist.lyneee.com/blog/2021030722know-k8s-p11-service.png](https://dist.lyneee.com/blog/2021030722know-k8s-p11-service.png)

Service被表示为无人机向相应的Pod中运送数据包

Service的主要指责是作为一个负载均衡器将数据包分发给相应的节点。它是一个基本的控制器结构，用于分组相似的pod（通常由pod标签进行区分）在工作节点之间。

假设你的前端应用想与后端应用机械能通信，可能每个应用程序都有多个正在运行的实例。与其担心对每一个后端pod进行编码，不如直接将数据包发送给后端Service，让它来决定随后如何去平衡负载并进行转发。

注意，Service更像是一个虚拟实体，因为所有的包都是通过IP表/IPVS/CNI插件进行处理的。它只是让我们更容易把它看作是一个真实的实体，使得我们更容易的去理解他在Kubernetes生态中的角色。

### Ingress

![https://dist.lyneee.com/blog/2021030722know-k8s-p12-Ingress.png](https://dist.lyneee.com/blog/2021030722know-k8s-p12-Ingress.png)

Ingress浮动的平台，所有的数据都通过该平台流入集群中

Ingress控制器是一个与外部世界进行联系的单点，用于与集群内运行的所有服务进行通信。这使得我们很容易的在其中设置安全策略、进行监视以及日志记录。

*P.S:在Kubernetes中还有很多其他的控制器对象，比如DaemonSet、StatefulSet和Jobs。还有很多对象比如，Secrets、ConfigMaps能够用于存储应用密钥和配置文件。*

【翻译计划2】下一篇文章，会继续跟进service mesh。进一步了解这些没有介绍到的服务。

## More

[1]Union File System$^1$: “联合文件系统是一个Linux，FreeBSD，NetBSD的文件系统服务，它实现了针对与其他文件的联合挂载。它允许单独的文件系统的文件和目录（分支）被透明的覆盖，形成一个一致的文件系统。被合并的分支中如果具有相同的路径，会出现在一个合并的目录中，并位于新的虚拟文件系统的目录中” —WikiPedia

[2]CRD$^2$: CustomResourceDefinition，link：[使用CRD拓展Kubernetes API](https://jimmysong.io/kubernetes-handbook/concepts/crd.html)

[3]Pod多容器设计模式，link：[Multi-Container Pod Design Patterns in Kubernetes](https://matthewpalmer.net/kubernetes-app-developer/articles/multi-container-pod-design-patterns.html)




