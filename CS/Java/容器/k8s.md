# what

+ Kubernetes（简称K8s）是一个开源的容器编排平台，用于自动化部署、扩展和管理容器化应用程序。它可以管理多个容器化应用程序，并提供自动化部署、负载均衡、自动扩展、自动恢复等功能

# why

- Docker为容器化的应用程序提供了开放标准，但随着容器越来越多出现了一系列新问题

  - 如何协调和调度这些容器？

  - 如何在升级应用程序时不会中断服务？

  - 如何监视应用程序的运行状况？

  - 如何批量重新启动容器里的程序？

- 解决这些问题需要容器编排技术，可以将众多机器抽象，对外呈现出一台超大机器。现在业界比较流行的有：k8s、Mesos、Docker Swarm

#  k8s架构

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6eaf44b8963e40c2ac3b1b2ce80c9972~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

我们看下 k8s 集群的架构，从左到右，分为两部分，第一部分是 Master 节点（也就是图中的 Control Plane），第二部分是 Node 节点

Master 节点一般包括四个组件，apiserver、scheduler、controller-manager、etcd，他们分别的作用是什么

- Apiserver：上知天文下知地理，上连其余组件，下接ETCD，提供各类 api 处理、鉴权，和 Node 上的 kubelet 通信等，只有 apiserver 会连接 ETCD。
- Controller-manager：控制各类 controller，通过控制器模式，致力于将当前状态转变为期望的状态。
- Scheduler：调度，打分，分配资源。
- Etcd：整个集群的数据库，也可以不部署在 Master 节点，单独搭建。

Node 节点一般也包括三个组件，docker，kube-proxy，kubelet

- Docker：具体跑应用的载体
- Kube-proxy：主要负责网络的打通，早期利用 iptables，现在使用 ipvs技术
- Kubelet：agent，负责管理容器的生命周期

总结一下就是 k8s 集群是一个由两部分组件 Master 和 Node 节点组成的架构，其中 Master 节点是整个集群的大脑，Node 节点来运行 Master 节点调度的应用

