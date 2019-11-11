# Kubernetes

> 记录一些k8s相关的学习

# 第二章 Kubernetes 核心概念

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


