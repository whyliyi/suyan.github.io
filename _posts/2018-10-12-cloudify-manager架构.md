---
layout: post
title: cloudify-manger architecture
category: cloudify
keywords: cloudify, orchestrator, 应用 
---

# Cloudify Overview
Cloudify是一个开源的云编排框架。Cloudify可以自动管理应用/服务的生命周期，包括在任意云上的部署、应用各方面的监控、检测问题和故障、手动/自动化运维任务。应用程序建模应用程序建模（Application Modeling Application modeling）能够以一种通用的、描述性的方式描述应用程序及其所有资源(基础设施、中间件、应用程序代码、脚本、工具配置、指标和日志)。
# Cloudify Manager Overview
cloudify-manager是Cloudify的核心组件，主要负责cloudify模板的解析、自动化部署、应用监控等任务。是一个有状态的编排核心，主要编排blueprint（cloudify的模板文件）。其主要功能为：

* 解析blueprint模板
* 自动化执行workflow
* 在cloudify-agent执行命令


cloudify-manager支持多云插件，实现简单。

cloudify-manager支持UI、API或者CLI的方式接入。提供标准的RESTful接口。
# Cloudify Manager Architecture
![](https://raw.githubusercontent.com/whyliyi/whyliyi.github.io/master/_img/cloudify/cloudify_flows.png)

* Proxy and File Server

cloudify用nginx作为前端反向代理和文件服务器。

* REST API

cloudify提供标准的RESTful API，覆盖所有的编排、管理功能。

API采用python的flask框架，运行在gUnicorn container中。

* Web GUI

提供cloudify的可视化界面，通过调用cloudify API实现，界面如下图所示：


![](https://raw.githubusercontent.com/whyliyi/whyliyi.github.io/master/_img/cloudify/dashboard.png)

* Workflow Engine

Workflow Engine用来自动化执行内置、客户定制的workflow，做blueprint解析、插件加载、任务定时、编排云资源（创建、操作应用）。

Workflow基于[Celery tasks broker](http://www.celeryproject.org/)实现。

* Runtime Model

Cloudify用Elasticsearch保存部署数据，deployment和runtime数据都以json文件保存。

* Metrics Database

Cloudify使用InfluxDB作为监控的数据库

* Policy Engine

Cloudify提供Policy功能，在运行过程中控制应用，举个例子：

加入一个application中的web service的安装依赖于mysql安装完成，我们可以用start detection来检测mysql已经安装完成。

Policy Engine基于[Riemann.IO CEP](http://riemann.io/)来实现。

Policy支持[Clojure](https://clojure.org/)语法格式。

* Task Broker

Cloudify用[Celery tasks broker](http://www.celeryproject.org/)和RabbitMQ来管理任务的分发和执行，包括blueprint、运行时数据、插件、相关node、操作。cloudify-agent监听RabbitMQ消息。

* Tasks

Task用来执行plugin函数。函数包括一系列参数（运行时参数和node参数上下文）

* Agents

cloudify-agent基于[Celery](http://www.celeryproject.org/)进程或worker，分布式部署，不区分节点，一个deployment可以同时在多个进程上执行。

cloudify-manager侧的agent主要用来调用IaaS层的API实现创建或者其他生命周期的操作。

* Plugins

Plugin是使用python语言来实现的第三方工具。

* Logs and Events

Cloudify的日志和操作日志

* Log & Event gathering mechanism

 

