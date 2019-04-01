# 2. Kubernetes 基础

## 2.1 Kubernetes 概述

### Kubernetes 是什么

Kubernetes(k8s)是Google开源由CNCF基金会管理的容器集群管理系统。为容器化的应用提供部署运行、资源调度、服务发现和动态伸缩等一系列完整功能。Kubernetes 被称为现在化的数据中心操作系统，大家都知道，操作系统是用来管理硬件资源比如 CPU、内存、磁盘、输入输出设备等资源的，并完成对这些资源的调度。而 Kubernetes 是将数据中心里的物理节点作为管理对象，并对物理节点上的资源进行调度完成编排任务。操作系统的另一个特性就是容易扩展，比如，当 GPU 这种硬件出现后，操作系统应该很容扩展去管理这种资源，Kubernetes 作为数据中心操作系统同样也有这样的特性，它也非常容易扩展。

## 2.2 Kubernetes 架构

Kubernetes 的架构从宏观层面的来讲分为两个平面，即控制平面和计算平面，控制平面又称为 Master 节点，Master 节点是整个集群的大脑，负责控制、调度集群资源；计算平面又称为 Node 节点，Node 节点负责运行工作负载，是 Master 节点调度的对象。kubectl 作为 Kubernetes 客户端通过和 Master 节点直接进行通信来控制 Kubernetes 集群。

![Kubernetes 架构](https://github.com/findsec-cn/k101/raw/master/docs/k8s.jpg)

### 控制节点

![Kubernetes Master](https://github.com/findsec-cn/k101/raw/master/docs/k8s-master.jpg)

- kube-apiserver 对外暴露 Kubernetes API，所有对集群的操作都是通过这组API完成，包括客户端下达应用编排命令给 Kubernetes 集群；kubelet 上报集群资源使用情况；以及各个组件之间的交互都是通过这套 API 完成的。
- kube-controller-manager 负责整个 Kubernetes 的管理工作，保证集群中各种资源处于期望状态，当监控到集群中某个资源状态与期望状态不符时，controller-manager 会触发调度操作。
- kube-scheduler 调度器负责 Kubernetes 集群的具体调度工作，接收来自于controller-manager 触发的调度操作请求，然后根据请求规格、调度约束、整体资源情况进行调度计算，最后将任务发送到目标节点由的kubelet组件执行。
- etcd 是一个高效KV存储系统。在Kubernetes环境中主要用于存储所有需要持久化的数据。

### 计算节点

![Kubernetes Node](https://github.com/findsec-cn/k101/raw/master/docs/k8s-node.jpg)

- kubelet 是 Node 节点上核心组件，负责与 docker daemon 进行交互运行 docker 容器；配置网络和数据卷；监控并上报节点资源使用情况。
- kube-proxy 主要负责 Service Endpoint 到 POD 实例的请求转发及负载均衡的规则管理。

以上这些组件的运行原理在进阶篇中会详细讲述。

## 2.3 Kubernetes 解决了什么问题

Kubernetes 项目最主要的设计思想是，从更宏观的角度，以统一的方式来定义任务之间的各种关系，更甚至我们可以运用 Kubernetes 扩展能力灵活的自定义任务之间的关系。

比如，Kubernetes 项目对容器间的交互关系进行了分类，首先总结出了一类非常常见的“紧密交互”的关系，即：这些应用之间需要直接通过本地文件进行信息交换。

在常规环境下，这些应用往往会被直接部署在同一台机器上，通过 Localhost 通信，通过本地磁盘上的文件进行交互。而在 Kubernetes 项目中，这些容器则会被划分为一个“Pod”，Pod 里的容器共享同一个 Network Namespace、同一组数据卷，从而达到高效率交换信息的目的。

Pod 是 Kubernetes 项目中最基础的一个对象，下一章中我会重中想你介绍 Pod。

而对于另外一种更为常见的需求，比如 Web 应用与数据库之间的访问关系，Kubernetes 项目则提供了一种叫作“Service”的服务。像这样的两个应用，往往故意不部署在同一台机器上，这样即使 Web 应用所在的机器宕机了，数据库也完全不受影响。可是，我们知道，对于一个容器来说，它的 IP 地址等信息不是固定的，那么 Web 应用又怎么找到数据库容器的 Pod 呢？

所以，Kubernetes 项目的做法是给 Pod 绑定一个 Service 服务，而 Service 服务声明的 IP 地址等信息是“终生不变”的。这个Service 服务的主要作用，就是作为 Pod 的代理入口，从而代替 Pod 对外暴露一个固定的网络地址。

除了 Pod、Service 这些基本资源外，Kubernetes 还为我们提供了 Deployment、StatefulSet、Job 等扩展资源，有了这些资源我们就可以轻松的编排我们的服务了，通过这些资源，Kubernetes 为我们提供了应用的水平扩展、滚动升级、自动扩缩容能力，为服务提供了负责均衡、服务监控、服务发现能力。通过这些高级功能以及其灵活的扩展能力使 Kubernetes 逐渐成为了容器编排领域的翘楚。

## 2.4 Kubernetes 集群搭建

### 使用 Docker CE 自带的 Kubernetes

安装 Docker 参考：https://docs.docker.com/install/

- Install Docker for Mac
- Install Docker for Windows

### 使用 minikube

#### MacOS 安装 minikube

- 安装minikube

    brew cask install minikube

- 安装vm驱动

    curl -LO https://storage.googleapis.com/minikube/releases/latest/docker-machine-driver-hyperkit \\n&& sudo install -o root -g wheel -m 4755 docker-machine-driver-hyperkit /usr/local/bin/

- 启动 minikube(minikube会安装kubernetes需要的组件，这些组件的镜像存储在google的常客中，所有需要能够翻墙才能够安装成功)

    minikube start --vm-driver=hyperkit --memory=4096 --insecure-registry="hub.example.com" --registry-mirror="https://m9sl8pb5.mirror.aliyuncs.com" --docker-opt="bip=172.87.0.1/16"

    参数介绍:
    --vm-driver 使用的虚拟机驱动
    --memory 给虚拟机的内存大小
    --cpu 给虚拟机的cpu核数，默认是2核
    --insecure-registry 指定私有仓库地址
    --registry-mirror 指定镜像缓存地址，加快镜像下载速度
    --docker-opt 指定docker的启动参数，bip 指定 docker 网桥使用的网段

- 停止 minikube

    minikube stop

- 删除集群

    minikube delete

- 登录虚拟机

    minikube ssh

#### Windows 安装 minikube

Windows 安装minikube，需要Windows系统支持Hyper-V，目前 Windows 10 Enterprise, Windows 10 Professional, and Windows 10 Education 支持Hyper-V。安装minikube时，最好命令行使用powershell，并且以管理员身份打开并执行。

##### 安装方法 1：

- 安装 https://chocolatey.org/
- choco install minikube kubernetes-cli

##### 安装方法 2：

下载 minikube-installer.exe （https://github.com/kubernetes/minikube/releases/latest）并进行安装。

启动前准备(重要)

- 配置 MINIKUBE_HOME 环境变量及创建目录
- 在 Hyper-V 中为minikube添加虚拟交换机
- 用 chocolatey 安装 OpenSSH（choco install openssh）

启动 minikube

    minikube start --vm-driver=hyperv --hyperv-virtual-switch=minikube --memory=4096 --insecure-registry="hub.example.com" --registry-mirror="https://m9sl8pb5.mirror.aliyuncs.com" --docker-opt="bip=172.87.0.1/16"

## 2.5 访问 Kubernetes

### Dashboard

#### 访问Dashboard

最新版本的 minikube 默认没有安装 Dashboard，执行如下命令 minikube 自动帮我们部署 Dashboard 并启动代理。

    minikube dashboard

访问地址： http://localhost:8001/api/v1/namespaces/kube-system/services/kubernetes-dashboard/proxy/#!/overview?namespace=default

#### 基本概念

##### 集群

- namespace 逻辑隔离空间，可以在不同的 namespace 里运行不同的微服务，默认使用 default 命名空间
- node 部署集群使用的物理节点，包括控制节点和计算节点

##### 工作负载

- deployment 我们实际部署的服务，所谓部署就是用来描述我们待编排的服务的期望状态的，比如我们这个服务启动的副本数，使用的镜像等等，后面会做详细讲解

##### 服务发现

- service 应用想被其他应用访问，需要先定义一个service，kubernetes自动帮我们解析这个service name，然后其他服务就可以通过service name访问这个服务了

##### 配置

- configmap 定义应用的配置文件，在应用容器启动时挂载到容器中
- secret 同 configmap，定义某些敏感的、需要加密的配置文件，在应用容器启动时挂载到容器中

### kubectl

#### 下载安装 kubectl

MacOS 安装：

    brew install kubernetes-cli

Windows 安装：

    choco install kubernetes-cli
    cd C:\users\yourusername
    mkdir .kube
    cd .kube
    New-Item config -type file

#### 配置 kubectl 客户端连接到 kubernetes 集群

kubectl配置文件 ~/.kube/config

- clusters
- users
- contexts

#### 获取集群信息

    kubectl get nodes
    kubectl get pods
    kubectl get pods -n xxx

#### 使用命令行部署nginx服务

    kubectl run nginx --image=nginx:1.15

#### 为nginx创建服务

    kubectl expose deployment nginx --port 80

#### 获取部署列表

    kubectl get svc

#### 启动工具容器进行访问测试

    kubectl run -ti busybox --image=busybox --restart=Never -- sh

#### 测试能否访问服务

    wget http://nginx

## 2.6 小结

本章中我向你讲述了 Kubernetes 是什么，Kubernetes 的架构，以及它解决了什么问题。在工作中，我们的目标是把我们的应用部署到 Kubernetes 容器云中，在部署应用时，首先遇到了应用间亲密关系的难题，于是就扩展到了 Pod；有了 Pod 之后，我们希望能一次启动多个应用实例，这样就需要 Deployment 这个 Pod 的多实例管理器；而有了这样一组相同的 Pod 后，我们又需要通过一个固定的 IP 地址和端口以负载均衡的方式访问它，于是就有了 Service。在以后的章节中我会给你详细讲解 Pod、Deployment、Service 这些资源对象。

最后我演示了如何搭建一个用于测试的 Kubernetes 集群。并通过 Dashboard、kubectl 访问了我们搭建的集群。在下一章我会着重给你讲解如何在 Kubernetes 集群中部署、升级我们的应用。
