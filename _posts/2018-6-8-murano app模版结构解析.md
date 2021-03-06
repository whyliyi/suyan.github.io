---
layout: post
title: murano app模版结构解析
category: murano
keywords: murano, app, 应用 
---

# Zabbix Overview
Zabbix是Alexei Vladishev创建。提供开源的、企业级的分布式监控解决方案，遵循GPL v2开源协议。

* Zabbix可以监控很多网络的参数，也可以监控服务器各项指标。

* Zabbix灵活的通知机制可以让用户配置E-Mail来接收任意事件消息。

* Zabbix提供非常非常强劲的报表和数据可视化能力。

* Zabbix支持web的访问方式，更方便用户查看。

# Zabbix 特性（Features）
### 数据采集
* 可用性和性能检测
* 支持SNMP (trapping、polling), IPMI, JMX, VMware监控
* 自定义采集
* 自定义采集周期
* performed by server/proxy and by agents

### 灵活的门限定义
* 用户可根据需求，自定义触发器的门限

### 高度可配的告警机制
* 可以定制化发送通知的内容，包括时间、接收者、媒介类型
* 通知支持宏参数，让通知具体，对用户更有帮助。
* 包括远程命令在内的自动化Action

### 实时绘图
* 支持监控数据实时绘图

### 网络监控能力
* Zabbix可以模拟鼠标动作访问指定的URL地址来检查功能和返回值

### 丰富的可视化选项
* 支持将多个监控项整合到一个页面的定制化画图
* 支持网络地图
* custom screens and slide shows for a dashboard-style overview
* 报表
* high-level (business) view of monitored resources

### 历史数据存储
* 数据库存储
* 可配置的历史数据
* built-in housekeeping procedure

### 更易配置
* add monitored devices as hosts
* hosts are picked up for monitoring, once in the database
* apply templates to monitored devices

### 模板
* 模版可定义check组
* 模版可以继承

### 网络发现
* 网络设备自动发现
* agent自动注册
* 发现文件系统、网络接口和SNMP OID

### Zabbix API
* 标准的API接口，方便第三方对接。

### 鉴权系统
* 安全鉴权机制
* 控制用户访问权限，比如某个用户只能看到某个视图（view）

### 可扩展的agent
* 支持Linux和Windows

# 参考
[Zabbix documentation](https://www.zabbix.com/documentation/3.0/)

 

