# 简介

[Kubernetes](https://www.kubernetes.org.cn/)是一个开源的，用于管理云平台中多个主机上的容器化的应用，Kubernetes的目标是让部署容器化的应用简单并且高效（powerful）,Kubernetes提供了应用部署，规划，更新，维护的一种机制。

Kubernetes一个核心的特点就是能够自主的管理容器来保证云平台中的容器按照用户的期望状态运行着（比如用户想让apache一直运行，用户不需要关心怎么去做，Kubernetes会自动去监控，然后去重启，新建，总之，让apache一直提供服务），管理员可以加载一个微型服务，让规划器来找到合适的位置，同时，Kubernetes也系统提升工具以及人性化方面，让用户能够方便的部署自己的应用（就像canary deployments）。



在Kubenetes中，所有的容器均在[Pod](https://www.kubernetes.org.cn/tags/pod)中运行,一个Pod可以承载一个或者多个相关的容器，在后边的案例中，同一个Pod中的容器会部署在同一个物理机器上并且能够共享资源。一个Pod也可以包含O个或者多个磁盘卷组（volumes）,这些卷组将会以目录的形式提供给一个容器，或者被所有Pod中的容器共享，对于用户创建的每个Pod,系统会自动选择那个健康并且有足够容量的机器，然后创建类似容器的容器,当容器创建失败的时候，容器会被node agent自动的重启,这个node agent叫kubelet,但是，如果是Pod失败或者机器，它不会自动的转移并且启动，除非用户定义了 replication controller。



![img](https://raw.githubusercontent.com/kubernetes/kubernetes/release-1.2/docs/design/architecture.png)

# 容器(Container)

在Kubernetes上运行的程序被打包成Linux容器。容器是一个被广泛接受的标准，因此已经有许多预先构建的映像可以部署在Kubernetes上。

容器化允许你创建自足式的Linux执行环境。任何程序和它的所有依赖项都可以打包成一个文件，然后在网络上共享。任何人都可以下载该容器并在其基础设施上部署它，所需的设置非常少。创建一个容器可以通过编程方式完成，从而形成强大的CI和CD管道。

可以将多个程序添加到单个容器中，但是如果可能的话，你应该将自己限制为每个容器的一个进程。拥有很多小容器比一个大容器好。如果每个容器都有一个紧密的焦点，那么更新更容易部署，并且问题更容易诊断。

# POD

在Kubernetes中，最小的管理元素不是一个个独立的容器，而是Pod,Pod是最小的，管理，创建，计划的最小单元.

一个[Pod](https://www.kubernetes.org.cn/tags/pod)（就像一群鲸鱼，或者一个豌豆夹）相当于一个共享context的配置组，在同一个context下，应用可能还会有独立的cgroup隔离机制，一个Pod是一个容器环境下的“逻辑主机”，它可能包含一个或者多个紧密相连的应用，这些应用可能是在同一个物理主机或虚拟机上。





# Services

Kubernete Service 是一个定义了一组[Pod](https://www.kubernetes.org.cn/tags/pod)的策略的抽象，我们也有时候叫做宏观服务。这些被服务标记的Pod都是（一般）通过label Selector决定的（下面我们会讲到我们为什么需要一个没有label selector的服务）。