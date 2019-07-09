---
layout: post
title: Kubernetes基本概念
category: Kubernetes
keywords: kubernetes, k8s, container
---

容器（container）是一种便携式、轻量级的操作系统级虚拟化技术。核心技术点：

* 利用操作系统的cgroup隔离不同的软件运行环境；
* 通过容器镜像打包软件的运行环境；

这些特性使得容器可以方便的在任何地方运行。

# 优点

* 体积小
* 启动快
* 不需要外部依赖
* 没有环境依赖（因为没有依赖，所以从测试环境到生产环境非常好迁移，满足当前DevOps的需求）
* 相比虚拟机，容器更易用、更高效，更符合敏捷的要求
* 升级回滚更为简单
* 应用与基础架构分析
* 测试环境到生产环境的一致性保证
* 更好的可移植性
* 从传统的操作系统部署转变为应用的部署
* 资源隔离
* 资源利用更高效、高密度

# 基本概念

## Pod

Kubernetes使用Pod来管理容器，每一个Pod中可以运行多个容器（container）。

从设计上来讲，一个Pod是一组紧密关联的容器集合，容器之间共享PID、IPC、网络和namespace，是Kubernetes调度的最小单元；

Pod内的多个容器共享网络和文件系统，可以通过进程间通信和文件共享这种简单高效的方式组合完成服务。

Kubernetes种使用manifest（支持yaml、json）来定义，以下是一个简单的nginx服务模板：
> 
	apiVersion: v1
	kind: Pod	
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

## Node

Node是Pod运行的主机，可以是虚拟机，也可以是物理机。

每个Node必须要运行Container runtime(docker或者rkt)、kubelet、kube-proxy三个服务。


## Namespace

Namespace是对一组资源和对象的抽象集合，pods, services, replication conrtroller, deployments都属于某一个namespace，node、persistentVolumes不属于任何namespace。

## Services

Service是应用服务对象，通过Labels为应用服务提供负载均衡和服务发现。匹配labels的Pod IP和端口列表组成endpoint，由kube-proxy负责负载分发。

每个Service都会自动分配一个cluster IP（仅在集群内部可以访问的虚拟地址）和DNS名，其他容器可以通过该地址或DNS来访问服务。

## Label

Label是识别kubernetes对象的标签，以key-value方式附加在对象上；

相同的标签认为是同一个应用；

用Label selector选择label组，支持以下几种方式：

* 等式：app=nginx 或者env!=production
* 集合：env in (production, test)
* 多个label：app=nginx,env=test

## Annotations

annotation用来记录一些附加信息，用来辅助应用部署、安全策略及调度策略，比如deployment用annotaion来记录rolling update的状态。