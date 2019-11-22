# Kubernetes

> 记录一些k8s相关的学习,感谢 [阿里大佬](https://developer.aliyun.com/learning/roadmap/cloudnative?spm=5176.13257455.1389354.1.604b7facM746tv)

# Kubernetes 核心概念

## 什么是K8s

**_一个自动化的容器编排平台（部署+弹性+管理）_**

核心功能：

* 服务发现与负载均衡
* **容器自动装箱**
* 存储编排
* **自动容器恢复**
* 自动发布与回滚
* 配置与密文管理
* 批量执行
* **水平伸缩**

## K8s架构

![](http://image.wangdy.cn/k8s/k8s%E6%9E%B6%E6%9E%84.png)

* API Server：顾名思义是用来处理 API 操作的，Kubernetes 中所有的组件都会和 API Server 进行连接，组件与组件之间一般不进行独立的连接，都依赖于 API Server 进行消息的传送；
 
* Controller：是控制器，它用来完成对集群状态的一些管理。比如刚刚我们提到的两个例子之中，第一个自动对容器进行修复、第二个自动进行水平扩张，都是由 Kubernetes 中的 Controller 来进行完成的；
 
* Scheduler：是调度器，“调度器”顾名思义就是完成调度的操作，就是我们刚才介绍的第一个例子中，把一个用户提交的 Container，依据它对 CPU、对 memory 请求大小，找一台合适的节点，进行放置；
 
* etcd：是一个分布式的一个存储系统，API Server 中所需要的这些原信息都被放置在 etcd 中，etcd 本身是一个高可用系统，通过 etcd 保证整个 Kubernetes 的 Master 组件的高可用性。

![](http://image.wangdy.cn/k8s/k8s_master.png)

Kubernetes 的 Node 是真正运行业务负载的，每个业务负载会以 Pod 的形式运行。

在 OS 上去创建容器所需要运行的环境，最终把容器或者 Pod 运行起来，也需要对存储跟网络进行管理。
Kubernetes 并不会直接进行网络存储的操作，他们会靠 Storage Plugin 或者是网络的 Plugin 来进行操作。用户自己或者云厂商都会去写相应的 Storage Plugin 或者 Network Plugin，去完成存储操作或网络操作。

![](http://image.wangdy.cn/k8s/k8s_node.png)

POD部署流程：

当`API Server`收到CLI或者UI的POD部署请求时，将信息写入存储系统`etcd`，并通知到调度器`Scheduler`。

`Scheduler` 根据内存等资源状态做一次调度决策，将调度信息告诉`API Server`，并被后入到`etcd`，再通知到NODE的`Kubelet`真正起一个容器。
![](http://image.wangdy.cn/k8s/k8s_pod_ex.png)

## K8s核心概念与API

#### POD

Pod 是 Kubernetes 的一个最小调度以及资源单元。
用户可以通过 Kubernetes 的 Pod API 生产一个 Pod，让 Kubernetes 对这个 Pod 进行调度，也就是把它放在某一个 Kubernetes 管理的节点上运行起来。
一个 Pod 简单来说是对一组容器的抽象，它里面会包含一个或多个容器。

#### Volume

Volume 就是卷的概念，它是用来管理 Kubernetes 存储的，是用来声明在 Pod 中的容器可以访问文件目录的，一个卷可以被挂载在 Pod 中一个或者多个容器的指定路径下面。

Volume 本身是一个抽象的概念，一个 Volume 可以去支持多种的后端的存储

#### Deployment

Deployment 是在 Pod 这个抽象上更为上层的一个抽象，它可以定义一组 Pod 的副本数目、以及这个 Pod 的版本。
一般大家用 Deployment 这个抽象来做应用的真正的管理，而 Pod 是组成 Deployment 最小的单元。

Kubernetes 是通过 Controller 去维护 Deployment 中 Pod 的数目，它也会去帮助 Deployment 自动恢复失败的 Pod。

#### Service

Service 提供了一个或者多个 Pod 实例的稳定访问地址。

* Cluster IP
* nodePort
* LoadBalancer

#### Namespace

Namespace 是用来做一个集群内部的逻辑隔离的，它包括鉴权、资源管理等。
Kubernetes 的每个资源，比如刚才讲的 Pod、Deployment、Service 都属于一个 Namespace，同一个 Namespace 中的资源需要命名的唯一性，不同的 Namespace 中的资源可以重名。



# Pod与容器设计模式

## 容器

**容器的本质实际上是一个进程，是一个视图被隔离，资源受限的进程。**

* 容器里 PID = 1 的进程就是应用本身

## 理解Pod

#### 进程组

例子：Helloworld 程序由四个进程组成，这些进程之间会共享一些资源和文件。那么现在有一个问题：假如说现在把 Helloworld 程序用容器跑起来，你会怎么去做？

自然的一个解法就是，我现在就启动一个 Docker 容器，里面运行四个进程。可是这样会有一个问题，这种情况下容器里面 PID=1 的进程该是谁? 比如说，它应该是我的 main 进程，那么问题来了，“谁”又负责去管理剩余的 3 个进程呢？

由于容器实际上是一个“单进程”模型，所以如果你在容器里启动多个进程，只有一个可以作为 PID=1 的进程，而这时候，如果这个 PID=1 的进程挂了，或者说失败退出了，那么其他三个进程就会自然而然的成为孤儿，没有人能够管理它们，没有人能够回收它们的资源，这是一个非常不好的情况。

> 注意：Linux 容器的“单进程”模型，指的是容器的生命周期等同于 PID=1 的进程（容器应用进程）的生命周期，而不是说容器里不能创建多进程。当然，一般情况下，容器应用进程并不具备进程管理能力，所以你通过 exec 或者 ssh 在容器里创建的其他进程，一旦异常退出（比如 ssh 终止）是很容易变成孤儿进程的。

![](http://image.wangdy.cn/k8s/ex1.png)

在真实的操作系统里面，一个程序往往是根据进程组来进行管理的。Kubernetes 把它类比为一个操作系统，比如说 Linux。
针对于容器我们前面提到可以类比为进程，就是前面的 Linux 线程。

那么 Pod 又是什么呢？实际上 Pod 就是我们刚刚提到的进程组，也就是 Linux 里的线程组。
![](http://image.wangdy.cn/k8s/9C4359DC-CBFD-4ADF-9F7A-B5434CF43D7F.png)

![](http://image.wangdy.cn/k8s/%E8%BF%9B%E7%A8%8B%E7%BB%84.png)

#### 为什么 Pod 必须是原子调度单位？

Pod 是原子调度单位，Pod 里面的容器是“超亲密关系”。

超亲密关系大致分为以下几类：
* 比如说两个进程之间会发生文件交换，前面提到的例子就是这样，一个写日志，一个读日志；
* 两个进程之间需要通过 localhost 或者说是本地的 Socket 去进行通信，这种本地通信也是超亲密关系；
* 这两个容器或者是微服务之间，需要发生非常频繁的 RPC 调用，出于性能的考虑，也希望它们是超亲密关系；
* 两个容器或者是应用，它们需要共享某些 Linux Namespace。最简单常见的一个例子，就是我有一个容器需要加入另一个容器的 Network Namespace。这样我就能看到另一个容器的网络设备，和它的网络信息。

## Pod实现机制

1. 共享网络
        
比如说现在有一个 Pod，其中包含了一个容器 A 和一个容器 B，它们两个就要共享 Network Namespace。在 Kubernetes 里的解法是这样的：它会在每个 Pod 里，额外起一个 Infra container 小容器来共享整个 Pod 的  Network Namespace。

Infra container 是一个非常小的镜像，大概 100~200KB 左右，是一个汇编语言写的、永远处于“暂停”状态的容器。由于有了这样一个 Infra container 之后，其他所有容器都会通过 Join Namespace 的方式加入到 Infra container 的 Network Namespace 中。

所以说一个 Pod 里面的所有容器，它们看到的网络视图是完全一样的。即：它们看到的网络设备、IP地址、Mac地址等等，跟网络相关的信息，其实全是一份，这一份都来自于 Pod 第一次创建的这个 Infra container。这就是 Pod 解决网络共享的一个解法。

整个 Pod 里面，必然是 Infra container 第一个启动。并且整个 Pod 的生命周期是等同于 Infra container 的生命周期的，与容器 A 和 B 是无关的。这也是为什么在 Kubernetes 里面，它是允许去单独更新 Pod 里的某一个镜像的，即：做这个操作，整个 Pod 不会重建，也不会重启，这是非常重要的一个设计。

2. 共享存储

把 volume 变成了 Pod level

## 详解容器设计模式

**Pod 是 Kubernetes 项目里实现“容器设计模式”的核心机制；**

#### Sidecar

什么是 Sidecar？就是说其实在 Pod 里面，可以定义一些专门的容器，来执行主业务容器所需要的一些辅助工作。

这种做法一个非常明显的优势就是在于其实将辅助功能从我的业务容器解耦了。

其它有哪些操作呢？比如说：

* 原本需要在容器里面执行 SSH 需要干的一些事情，可以写脚本、一些前置的条件，其实都可以通过像 Init Container 或者另外像 Sidecar 的方式去解决；
* 当然还有一个典型例子就是我的日志收集，日志收集本身是一个进程，是一个小容器，那么就可以把它打包进 Pod 里面去做这个收集工作；
* 还有一个非常重要的东西就是 Debug 应用，实际上现在 Debug 整个应用都可以在应用 Pod 里面再次定义一个额外的小的 Container，它可以去 exec 应用 pod 的 namespace；
* 查看其他容器的工作状态，这也是它可以做的事情。不再需要去 SSH 登陆到容器里去看，只要把监控组件装到额外的小容器里面就可以了，然后把它作为一个 Sidecar 启动起来，跟主业务容器进行协作，所以同样业务监控也都可以通过 Sidecar 方式来去做。


# 应用编排管理

## K8s资源对象

* Spec 期望的状态
* Status 观察到的状态
* Metadata
    1. Labels 标示型的Key:Value元数据，多被 Selector 使用
    2. Annotations 存储资源的非标识性信息，扩展资源的spec/status
    3. OwnerReference 集合类资源，比如 Replicaset 创建的Pods就会有 OwnerReference 信息。方便反向查找创建资源对象，方便进行及联删除
    

`kubectl get pods --show-labels -l 'env in (test,dev)'`

`--show-labels` 代表展示pods的标签，
`-l` 后面是 Selector 某个标签，支持集合型查询 `'env in (test,dev)'` ，以及相等型 `env=dev,xxx=xxx` 这种，后者多个label是"与"的关系。
    
## 控制器模式

![](http://image.wangdy.cn/k8s/%E6%8E%A7%E5%88%B6%E5%99%A8%E6%A8%A1%E5%BC%8F.png)

* 由声明式的API驱动-描述K8s资源对象
* 由控制器异步地控制系统向终态驱动
* 使系统的自动化和无人值守化成为可能
* 便于扩展-自定义资源和控制器


# Deployment

管理部署发布的控制器

- 定义一组Pod期望数量
- 配置Pod发布方式，可以保证更新过程中不可用的Pod数量可控
- 回滚发布

常用命令：

1. 修改镜像 `kubectl set image deployment/xxx yyy=zzz`, 其中 `xxx` 代表某个 deployment 名字， `yyy` 代表要更新的容器名字， `zzz` 为新的镜像
2. 回滚发布 `kubectl rollout undo deployment/xxx`

## DeploymentStatus

![](http://image.wangdy.cn/k8s/deploymentstatus.png)

## 管理模式

![](http://image.wangdy.cn/k8s/deploy_replicaset.png)


# DaemonSet 守护进程控制器

DaemonSet Controller 可以帮助我们：
- 保证集群内每一个（或一些）节点都运行一组相同的Pod
- 跟踪集群节点状态，保证新加入的节点创建对应Pod
- 跟踪集群节点状态，保证移除的节点删除对应Pod
- 跟踪Pod状态，保证每个节点Pod处于运行状态

使用场景：
1. 集群存储进程
2. 日志收集进程
3. 需要在每个节点运行监控收集器


# 应用配置管理

![](http://image.wangdy.cn/k8s/pod%E9%85%8D%E7%BD%AE%E7%AE%A1%E7%90%86.png)

#### Secret

Secret 用于存储一些敏感信息。

`kubectl create secret generic [NAME][DATA][TYPE]` 

DATA可以指定文件(--from-file=key=/config.json)/键值对(--from-literal=key=value) 默认TYPE为 Opaque

#### ServiceAccount

ServiceAccount 主要用于解决 Pod 在集群中的身份认证问题，其中认证使用的授权信息，可以利用 Secret(type=kubernetes.io/service-account-token) 进行管理

#### Resources

Pod 运行的一些资源配置，有 request/limit 两种类型
- CPU
- Memory
- ephemeral-storage (临时存储，单位Byte)

服务质量(Qos)配置
![](http://image.wangdy.cn/k8s/3DAC9206-A843-41DB-B7AE-6288CAA2C1BD.png)


# PV/PVC

因为 Pod Volume 的生命周期与 Pod 相同，所以面对无法满足 Pod 层面的一些存储需求，比如宿主机故障迁移不丢失数据，多个 Pod 共享 Volume，数据功能扩展等。

(PV)PersistentVolumes 将存储与计算分离，使用不同的组建(Controllers)管理存储与计算资源，解耦 Pod 与 Volume 的生命周期关联。

_PVC_ 与 _PV_ 的关系类似于软件工程中的 **_接口_** 与 **_实现_**，用户只需关心使用 PVC(PersistentVolumesClaim)

#### 静态配置

系统管理员预先创建 PV

![](http://image.wangdy.cn/k8s/201911161127-A654-41C2-A63D-C1838A223665.png)

#### 动态配置

系统管理员创建 StorageClass 模版

![](http://image.wangdy.cn/k8s/201911161128-4B3A-460E-9609-1EAF8583E6DD.png)

#### PVC Spec

![](http://image.wangdy.cn/k8s/201911161138-3CCB-4142-9627-FCD452573FA3.png)

PVC,PV,Pod创建使用完整流程

![](http://image.wangdy.cn/k8s/201911161202-421B-4E3B-A167-AFA02B1E2972.png)

# 存储快照与拓扑调度

## 存储快照

产生背景：
1. 如何保证重要数据在误操作之后可以快速恢复？ 
2. 如何进行数据的快速复制和迁移

解决办法：
- Kubernetes CSI Snapshotter Controller

类似于PV/PVC

用户定义 `VolumeSnapshot` ，管理员定义 `VolumeSnapshotClass` ，系统会动态生成 `VolumeSnapShotContent`

恢复数据时，配置 PVC 的 `dataSource` 为已创建好的 `VolumeSnapshot`

## 拓扑调度

> 这里的拓扑指K8s集群中 node 位置关系的一种人为划分规则

产生背景：

如果创建的 PV 有 `.spec.nodeAffinity` 的要求，即只有特定的 Nodes 上才能访问 PV 资源，
原有创建 Pod 的流程与但创建 PV 的流程是分离的，所以无法保证新创建的 Pod 能被调度到可以访问 PV 资源的 Node 上，导致 Pod 起不来。

本质上讲，PV 在Binding或者Dynamic Provisioning时，不知道使用此 PV 的 Pod 调度在哪个 Node 上，但 PV 的访问对 Node 的拓扑有限制。

解决方法：

- 将 PV 的 Binding / Dynamic Provisioning 操作**_延迟_**到 Pod 调度确定结果之后

#### 实现流程

![](http://image.wangdy.cn/k8s/201911172038-89AF-4F50-826B-D257C63AFAF8.png)

CSI是Container Storage Interface（容器存储接口）的简写，目的是定义行业标准“容器存储接口”，使存储供应商（SP）能够开发一个符合CSI标准的插件并使其可以在多个容器编排（CO）系统中工作。

上图中 csi-snapshottor 和 csi-provisioner 部分为社区推动实现的 controller 部分，右边为 SP 实现的不同的 Plugin(Driver)，controller 与 plugin 之间通过 grpc 调用，后者通过自己的 openAPI 完成真实的云服务创建。
 

# 应用健康

应用健康状态
- Readiness Probe 就绪指针
- Liveness Probe 存活指针

![](http://image.wangdy.cn/k8s/201911212144-C5F8-4C25-B674-7B7FDF31850B.png)

![](http://image.wangdy.cn/k8s/201911212147-92B3-42B0-9055-452B3F73AAA0.png)

常见异常：
- Pod 处于 Pending，表示调度器没有介入，可以查看事件排查，一般与资源使用相关。
- Pod 处于 waiting，一般表示镜像没有正常拉取，可能与私有镜像仓库、地址不存在等相关。
- Pod 不断被拉起且可以看到 crashing，表示 Pod 已被正常调度但启动失败，可能是权限、配置的问题，检查日志查看。


# K8s Service

K8s 通过 Service 做服务发现和负载均衡

#### 为什么需要服务发现

1. Pod 生命周期短，IP随时发生变化
2. Deployment 需要统一访问入口和做负载均衡
3. 应用间在不同环境部署时需要保持同样的部署拓扑和访问方式

#### 向集群外暴露 Service

Service 的类型：

- ClusterIP
- ExternalName
- NodePort
- LoadBalancer

其中 NodePort 和 LoadBalancer 可以向集群外暴露服务

![](http://image.wangdy.cn/k8s/201911221721-3300-45D5-9EB4-57ED2842D2C3.png)

![](http://image.wangdy.cn/k8s/201911221722-5F18-4BD6-B496-05C3885A701D.png)


 








