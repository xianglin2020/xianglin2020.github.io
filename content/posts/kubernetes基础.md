---
title: "Kubernetes基础"
date: 2024-09-20T08:19:52+08:00
categories:
  - learn
tags:
  - kubernetes
summary: "kubernetes 基础知识。"
description: "kubernetes 基础知识。"
author: [ "XiangLin" ]
cover:
  image: "https://kubernetes.io/images/docs/kubernetes-cluster-architecture.svg"
  caption: ""
  alt: ""
  relative: false
---

https://kubernetes.io/zh-cn/docs/home/

## 工作负载

### Pod

Pod 是可以在 Kubernetes 中创建和管理的、最小的可部署的计算单元。Pod 是一组（一个或多个）容器，这些容器共享存储、网络、以及怎样运行这些容器的规约。Pod
的共享上下文包括一组 Linux 名字空间、控制组（cgroup）和可能一些其他的隔离。

Kubernetes 集群中的 Pod 主要有两种用法：

* 运行单个容器的 Pod。每个 Pod 一个容器，可以将 Pod 看作单个容器的包装器，并且 Kubernetes 直接管理 Pod，而不是容器。
* 运行多个协同工作的容器的 Pod。Pod 可以封装由紧密耦合且需要共享资源的多个并置容器组成的应用，这些位于同一位置的容器构成一个内聚单元。

#### Pod 的生命周期

Pod 遵循预定义的生命周期，起始于 Pending 阶段，如果至少其中有一个主要容器正常启动，则进入 Running，之后取决于 Pod
中是否有容器以失败状态结束而进入 Succeeded 或者 Failed 阶段。

Pod 在其生命周期中只会被调度一次。将 Pod 分配到特定节点的过程称为绑定，而选择使用哪个节点的过程称为调度。

Pod 的 status 字段是一个 PodStatus 对象，其中包含一个 phase 字段。

| 取值            | 描述                                                                       |
|---------------|--------------------------------------------------------------------------|
| Pending（悬决）   | Pod 已被 Kubernetes 系统接受，但有一个或多个容器尚未创建亦未运行。此阶段包括等待 Pod 被调度的时间和通过网络下载镜像的时间。 |
| Running（运行中）  | Pod 已经绑定到某个节点，Pod 中所有容器都已经被创建。至少有一个容器仍在运行，或者正处于启动或重启状态。                  |
| Succeeded（成功） | Pod 中的所有容器都已经成功终止，并且不会再重启。                                               |
| Failed（失败）    | Pod 中的所有容器都已终止，并且至少有一个容器是因为失败终止。也就是说，容器以非 0 状态退出或者被系统终止，且未被设置为自动重启。      |
| Unknown       | 因为某些原因无法取得 Pod 的状态，通常因为与 Pod 所在主机通信失败。                                   |

#### 容器状态

一旦调度器将 Pod 分派给某个节点，kubelet 就通过容器运行时开始为 Pod 创建容器。容器的状态有三种：Waiting（等待）、Running（运行中）和
Terminated（已终止）。

处于 Waiting 状态的容器仍在运行它完成启动所需要的操作：例如，从某个容器镜像仓库拉取容器镜像，或者向容器应用 Secret 数据。Reason 字段给出了容器处于等待状态的原因。

Running 状态表明容器正在执行状态并且没有问题发生。如果配置了 postStart 回调，那么该回调已经执行且已完成。

处于 Terminated（已完成）状态的容器表明容器运行至正常结束或者因为某些原因失败。如果容器配置了 preStop 回调，则该回调会在容器进入 Terminated 状态之前执行。

#### 容器重启策略

Pod 的 spec 中包含一个 restartPolicy 字段，其可能取值包括 Always、OnFailure 和 Never。默认值是 Always。

* Always：只要容器终止就自动重启容器。
* OnFailure：只有在容器错误退出（退出状态非 0）时才重启容器。
* Never：不会自动重启已终止的容器。

Kubernetes 通过 restartPolicy 管理 Pod 内容器出现的失效，其顺序如下：

1. 最初的崩溃：Kubernetes 尝试根据 Pod 的 restartPolicy 立即重新启动。
2. 反复的崩溃：在最初的崩溃之后，Kubernetes 对于后续重新启动的容器采用指数级回退延迟机制（10 秒、20 秒、40 秒）重启容器，上限为
   300 秒（5分钟）。这一机制可以防止快速、重复的重新启动尝试导致系统过载。
3. CrashLoopBackOff 状态：表明回退延迟机制正在生效。
4. 回退重置：如果容器成功运行了一定时间（如 10 分钟），Kubernetes 会重置回退延迟机制，将新的崩溃视为第一次崩溃。

#### 容器探针

probe 是由 kubelet 对容器执行的定期诊断。使用探针来检查容器有四种不同的方法：

* exec：在容器内执行指定命令，如果命令退出时返回码为 0 则认为诊断成功。
* grpc：使用 gRPC 执行一个远程过程调用。目标应该实现 gRPC 健康检查，如果响应的状态是 SERVING 则认为诊断成功。
* httpGet：对容器的 IP 地址上指定端口和路径执行 HTTP GET 请求。如果响应码在 [200, 400)，则认为诊断成功。
* tcpSocket：对容器的 IP 地址上指定端口执行 TCP 检查。如果端口打开，则诊断被认为是成功的。

每次探测都将获得以下三种结果之一：

* Success：成功
* Failure：失败
* Unknown：未知

针对运行中的容器，kubelet 可以选择执行以下三种探针：

* livenessProbe：指示容器是否正在运行。如果存活探针探测失败，则 kubelet 会杀死容器，并根据重启策略处理。如果容器不提供存活探针，则默认状态为
  Success。
* readinessProbe：指示容器是否准备好为请求提供服务。如果就绪态探测失败，端点控制器将从与 Pod 匹配的所有服务的端点列表中删除该
  Pod 的 IP 地址。初始延迟之前的就绪的状态值默认为 Failure。如果容器不提供就绪态探针，则默认状态为 Success。
* startupProbe：指示容器中的应用是否已经启动。如果提供了启动探针，则所有其他探针会被禁用，直到此探针成功为止。如果启动探测失败，则
  kubelet 会杀死容器，并根据重启策略处理。如果容器不提供启动探针，则默认状态为 Success。

#### Init 容器

Init 容器是一种特殊容器，在 Pod 内的应用容器启动之前运行，Init 容器可以包括一些应用镜像中不存在的实用工具和安装脚本。常规的
Init 容器不支持 lifecycle、livenessProbe、readinessProbe 或 startupProbe 字段。Init 容器必须在 Pod 准备就绪之前完成运行，如果一个
Pod 指定了多个 Init 容器，这些容器会按顺序逐个运行。每个 Init 容器必须运行成功，下一个才能够运行。当所有的 Init
容器运行完成时，Kubernetes 才会为 Pod 初始化应用容器并像平常一样运行。

实用 Init 容器的一些场景：

* 等待一个 Service 完成创建
* 注册这个 Pod 到远程服务器
* 在启动应用容器之前等待一段时间
* 克隆 Git 仓库到卷中
* 将配置值放到配置文件中，运行模板工具为主应用容器动态地生成配置文件

#### 边车容器

边车容器是与主应用容器在同一个 Pod 中运行的辅助容器。这些容器通过提供额外的服务或功能（如日志记录、监控、安全性或数据同步）来增强或扩展主应用容器的功能，而无需直接修改主应用代码。边车容器与同一
Pod 中的应用容器并行运行，不过边车容器不执行主应用逻辑，而是为主应用提供支持功能。边车容器具有独立的生命周期，他们可以独立于应用容器启动、停止和重启。边车容器与主应用容器共享相同的网络和存储命名空间。

### 工作负载管理

通常不需要直接管理每个 Pod，而是使用负载资源来代替用户管理一组 Pod。这些负载资源通过配置控制器来确保正确类型的、处于运行状态的
Pod 个数是正确的，与用户指定的状态一致。用于管理工作负载的内置 API 包括：

* Deployment（也间接包括 ReplicaSet）是在集群上运行应用的最常见方式。Deployment 适合在集群上管理无状态应用工作负载，其中
  Deployment 中的任何 Pod 都是可互换的，可以在需要时进行替换。
* StatefulSet 管理了一个或多个运行相同应用代码，但具有不同身份标识的 Pod。StatefulSet 最常见的用途是能够建立其 Pod
  与其持久化存储之间的关联。例如，可以运行一个将每个 Pod 关联到 PersistentVolume 的 StatefulSet。如果该 StatefulSet 中的一个
  Pod 失败了，Kubernetes 将创建一个新的 Pod，并连接到相同的 PersistentVolume。
* DaemonSet 定义了在特定节点上提供本地设施的 Pod。DaemonSet 中每个 Pod 执行类似与经典 Unix / POSIX 服务器上的系统守护进程的角色。
* 可以使用 Job 或 CronJob 定义一次性任务和定时任务。Job 表示一次性任务，而每个 CronJob 可以根据排期表重复执行。

#### Deployments

Deployment 用于管理运行一个应用的一组 Pod，通常适用于不保持状态的负载。一个 Deployment 为 Pod 和 ReplicaSet
提供声明式的更新能力。你负责描述 Deployment 中的目标状态，而 Deployment 控制器（Controller）以受控速率更改实际状态，使其变为期望状态。你可以定义
Deployment 以创建新的 ReplicaSet，或删除现有 Deployment，并通过新的 Deployment 收养其资源。

#### StatefulSet

StatefulSet 运行一组 Pod，并为每个 Pod 保留一个稳定的标识。这可用于管理需要持久化存储或稳定、唯一网络标识的应用。使用
StatefulSet 的场景：

* 稳定的、唯一的网络标识符。
* 稳定的、持久的存储。
* 有序的、优雅的部署和扩缩。
* 有序的、自动的滚动更新。

StatefulSet Pod 具有唯一的标识，该标识包括顺序标识、稳定的网络标识和稳定的存储。 该标识和 Pod 是绑定的，与该 Pod 调度到哪个节点上无关。

#### DaemonSet

DaemonSet 定义了提供节点本地设施的 Pod。这些设施可能对于集群的运行至关重要，例如网络辅助工具，或者作为 add-on
的一部分。DaemonSet 确保全部（或者某些）节点上运行一个 Pod 的副本。当有节点加入集群时，也会为它们新增一个 Pod。当有节点从集群移除时，这些
Pod 也会被回收。删除 DaemonSet 将会删除它创建的所有 Pod。DaemonSet 的一些典型用法：

* 在每个节点上运行集群守护进程
* 在每个节点上运行日志收集收集守护进程
* 在每个节点上运行监控守护进程

#### Job / CronJob

Job 表示一次性任务，运行完后就会停止。Job 会创建一个或者多个 Pod，并将继续重试 Pod 的执行，直到指定数量的 Pod 成功终止。

CronJob 用于执行排期操作，例如备份、生成报告等。一个 CronJob 对象就像 Unix 系统上的 corntab（cron table）文件中的一行。

```shell
# ┌───────────── 分钟 (0 - 59)
# │ ┌───────────── 小时 (0 - 23)
# │ │ ┌───────────── 月的某天 (1 - 31)
# │ │ │ ┌───────────── 月份 (1 - 12)
# │ │ │ │ ┌───────────── 周的某天 (0 - 6)（周日到周六）
# │ │ │ │ │                          或者是 sun，mon，tue，web，thu，fri，sat
# │ │ │ │ │
# │ │ │ │ │
# * * * * *
```

## 服务、负载均衡和联网

Kubernetes 网络模型由几个部分构成：

* 集群中的每个 Pod 都会获得自己的、独一无二的集群范围 IP 地址。Pod 有自己的私有网络命名空间，Pod 内的所有容器共享这个命名空间。运行在同一个
  Pod 中的不同容器进程彼此之间可以通过 localhost 进行通信。
* Pod 网络（也称为集群网络）处理 Pod 之间的通信。它确保所有 Pod 可以与其他 Pod 进行通信。节点上的代理（例如系统守护进程或
  kubelet）可以与该节点上的所有 Pod 进行通信。
* Service API 允许你为由一个或多个后端 Pod 实现的服务提供一个稳定（长效）的 IP 地址或主机名，其中组成服务的各个 Pod 可以随时变化。
* Gateway API（或 Ingress）使得集群外部的客户端能够访问 Service。
* NetworkPolicy 是一个内置的 Kubernetes API，允许你控制 Pod 之间的流量或 Pod 与外部世界的流量。

### 服务（Service）

Kubernetes 中 Service 是将运行在一个或一组 Pod 上的应用程序公开为网络服务的方法。Service 所对应的 Pod 集合通常由定义的选择算符来确定。

如果工作负载使用 HTTP 通信，可以选择使用 Ingress 来控制 Web 流量如何到达该工作负载。Ingress 不是一种 Service，但它可用作集群的入口点。

#### 服务类型

Service type（类型）值及其行为有：

* ClusterIP：默认值，通过集群的内部 IP 公开 Service，选择该值时 Service 只能够在集群内部访问。可以使用 Ingress 或者 Gateway
  API 向公共互联网公开服务。
* NodePort：通过每个节点上的 IP 和静态端口（NodePort）公开 Service。为了让 Service 可以通过节点端口访问，Kubernetes 会为
  Service 配置集群 IP 地址，相当于请求了 type: ClusterIP 的 Service。
* LoadBalancer：使用云平台的负载均衡器向外部公开 Service。
* ExternalName：将服务映射到 externalName 字段的内容（例如，映射到主机名 api.foo.bar.example）。

#### 服务发现

对于在集群内运行的客户端，Kubernetes 支持两种主要的服务发现模式：环境变量和 DNS。

* 环境变量：当 Pod 运行在某 Node 上时，kubelet 会在其中为每个活跃的 Service 添加一组环境变量。kubelet 会添加环境变量
  `{SVCNAME}_SERVICE_HOST` 和 `{SVCNAME}_SERVICE_PORT` 。
* DNS：DNS 服务器会监视 Kubernetes API 中新的 Service，并为每个 Service 创建一个 DNS 记录。如果在整个集群中都启用了 DNS，则所有
  Pod 都应该能通过 DNS 名称自动解析 Service。

### Ingress

Ingress 提供从集群外部到集群内服务的 HTTP 和 HTTPS 路由，流量路由由 Ingress 资源所定义的规则来控制。Ingress 可以为 Service
提供外部可访问的 URL、对其流量作负载均衡、终止 SSL/TLS。Ingress 控制器负责完成 Ingress
的工作。可选用 [NGINX Ingress 控制器](https://kubernetes.github.io/ingress-nginx/deploy/#quick-start)。

![ingress-diagram](https://kubernetes.io/zh-cn/docs/images/ingress.svg)

### 存储

为集群中的 Pods 提供长期和临时存储的方法。

#### 卷（Volume）

容器中的文件在磁盘上是临时存放的，当容器停止或崩溃时，在容器生命周期内创建或修改的所有文件都将丢失。Kubernetes 支持很多类型的卷，Pod
可以同时使用任意数目的卷类型。临时卷类型的生命周期与 Pod 相同，但持久卷可以比 Pod 的存活期长。对于给定 Pod
中任何类型的卷，在容器重启期间数据都不会丢失。

卷的核心是一个目录，其中可能存有数据，Pod 中的容器可以访问该目录中的数据。

Kubernetes 支持下列类型的卷：

* configMap：提供了向 Pod 注入配置数据的方法。
* emptyDir：在 Pod 被指派到某节点时此卷会被创建。Pod 中的容器都可以读写 emptyDir 卷中相同的文件。当 Pod
  因为某些原因从节点上删除时，emptyDir 卷中的数据也会被永久删除。emptyDir 的一些用途：
    * 缓存空间，例如基于磁盘的归并排序。
    * 为耗时较长的计算任务提供检查点，以便任务能方便地从崩溃前状态恢复执行。
* hostPath：能将主机节点文件系统上的文件或目录挂载到你的 Pod 中。hostPath 的一些用法：
    * 运行一个需要访问节点级系统组件的容器（例如一个将系统日志传输到集中位置的容器，使用只读挂载 `/var/log` 来访问这些日志）。
    * 让存储在主机系统上的配置文件可以被静态 Pod 以只读方式访问（静态 Pod 无法访问 ConfigMap）。
* nfs：将 NFS（网络文件系统）挂载到 Pod 中，nfs 卷的内容在删除 Pod 时会被保存，卷只是被卸载。
* persistentVolumeClaim：用来将持久卷（PersistentVolume）挂载到 Pod 中。持久卷申领是用户在不知道特定云环境细节的情况下”申领“持久存储的一种方法。
* secret：用来给 Pod 传递敏感信息，例如密码。

