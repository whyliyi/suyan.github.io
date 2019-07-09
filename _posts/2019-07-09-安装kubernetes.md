---
layout: post
title: Kubernetes安装
category: Kubernetes
keywords: kubernetes, k8s, container
---

参考文档：https://yq.aliyun.com/articles/221687

安装docker

yum remove docker docker-common docker-selinux docker-engine 

yum install -y yum-utils device-mapper-persistent-data lvm2

yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo

yum makecache fast

yum -y install docker-ce

systemctl daemon-reload && systemctl enable docker && systemctl restart docker && systemctl status docker

启动minikube
minikube start --registry-mirror=https://registry.docker-cn.com  --vm-driver=none 

