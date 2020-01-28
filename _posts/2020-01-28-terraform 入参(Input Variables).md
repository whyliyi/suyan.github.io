---
layout: post
title: Terraform 入参(Input Variables)
category: Terraform
keywords: terraform, orchestration, 编排, 开源
---

## 入参（Input Variables）

入参（输入参数）作为Terraform的模块参数，是从资源定义中剥离出来的在用户使用的过程中可能会变化的属性，抽取成入参后，在`terraform apply`的时候可以指定参数值。

举个栗子：创建虚拟机是一个通用功能，但是虚拟机最基本的规格、镜像、网络等，都是可以由用户在创建资源实例的时候指定，因此可以把这些属性抽取成一个参数，用户在`terraform apply`的时候直接指定参数值，以灵活定义虚拟机个镜像、规格、网络等。如下：

>
	variable "image_id" {
	  type = string
	  default = "e5939e45-6be2-462c-a4c2-5b725a5f8b7d"
	}
>
	variable "flavor_id" {
	  type = string
	  default = "m1.small"
	}
>
	variable "net_id" {
	  type = string
	  default = "test_network"
	}
	resource "openstack_compute_instance_v2" "basic" {
	  name            = "basic_server"
	  image_id        = var.image_id
	  flavor_id       = var.flavor_id
	  security_groups = ["default"]
>
	  network {
	    name = var.net_id
	  }
	}

## 入参命名规则

* 包含数字、字母、中划线(`-`)、下划线(`_`)，不能以数字开头。
* 以下字符串是Terraform内置关键字，不允许作为入参使用：
	- `source`
	- `version`
	- `providers`
	- `count`
	- `for_each`
	- `lifecycle`
	- `depends_on`
	- `locals`
	- `any`

## 入参支持的参数类型

**基本数据类型：**

* `string`："hello"/"China"等等
* `number`：6.12/3000等等
* `bool`：true/false

以上三种基本数据类型会在必要时强制转换，比如`true`转换成`"true"`、`12`转换成`"12"`反之亦然。

**复杂数据类型-集合类型数据:**

* `list(<TYPE>)`：比如list(string), list(number), list(any)
* `set(<TYPE>)`：set内的元素不允许重复
* `map(<TYPE>)`：map(string)

**复杂数据类型-结构化数据**

* `object({<ATTR NAME> = <TYPE>, ... })`：

> 
	object({ name=string, age=number })
>	
	初始化变量的时候，应该是：
>	
	{
	  name = "John"
	  age  = 52
	}
	
* `tuple([<TYPE>, ...])`

>
	tuple([string, number, bool])
>
	初始化变量的时候，应该是：
>
	["a", 15, true]

**`any`类型占位符**

指定参数的`type`为`any`的时候，表示可以接受任何参数，但是**any不是参数类型**，你可以认为any只是一个不确定参数的类型的占位符，Terraform会根据你传入的值确定参数类型

举个栗子：

>
	list(any)：
>
	+ 如果传值为list(["a", "b", "c"])，那么list(any)会被Terraform解析成list(string)。
	+ 如果传入list([1, 2, 3])，会被Terraform解析成list(number)。
	+ 如果传入list([1, [], 3])，由于`[]`无法转换成和1、3同一个数据类型，因此Terraform会报错。

>

	variable "no_type_constraint" {
	  type = any
	}
	
	也是同样的道理

## 入参校验

>
	variable "image_id" {
	  type        = string
	  description = "The id of the machine image (AMI) to use for the server."
>
	  validation {
	    condition     = length(var.image_id) > 4 && substr(var.image_id, 0, 4) == "ami-"
	    error_message = "The image_id value must be a valid AMI id, starting with \"ami-\"."
	  }
	}
	
从上述栗子可以看出，validation可以校验参数输入是否合法，`length(var.image_id) > 4`表示`image_id`参数的长度需要大于4，`substr(var.image_id, 0, 4) == "ami-"`表示`image_id`要以"ami-"开头。

如果`condition`为`true`，则参数校验通过，如果为`false`，则将`error_message`作为错误原因返回校验失败。

***注：示例中用到了`length`、`substr`函数，这些函数是Terraform内置函数，后续再说***

## 入参赋值方式

入参在`terraform apply`的时候，会有很多方式给参数赋值。

* `-var`传值

>
	terraform apply -var="image_id=ami-abc123"
	terraform apply -var='image_id_list=["ami-abc123","ami-def456"]'
	terraform apply -var='image_id_map={"us-east-1":"ami-abc123","us-east-2":"ami-def456"}'
	
* 变量定义文件`.tfvars`
参数定义文件可以是`.tfvars`或者`.tfvars.json`结尾。用`-var-files`传入文件名称。

>
	terraform apply -var-file="testing.tfvars"

testing.tfvars文件内容如下：
	
>	
	image_id = "ami-abc123"
	availability_zone_names = [
	  "us-east-1a",
	  "us-west-1c",
	]

Terraform会自动加载文件夹中以下文件（如果有）以初始化变量。
>
 	+ 文件名为`terraform.tfvars`或者`terraform.tfvars.json`
	+ 文件以后缀名为`.auto.tfvars`或者`.auto.tfvars.json`

* 环境变量传值

>
	export TF_VAR_image_id=ami-abc123
	
由此可以定义`image_id`的值为ami-abc123

* 入参定义的优先级

按照一下顺序搜索参数的值，后边的参数值会覆盖前边的参数值。

>
	1. 环境变量
	2. terraform.tfvar文件
	3. terraform.tfvars.json文件
	4. *.auto.tfvars或者*.auto.tfvars.json文件
	5. "-var"和"-var-file"指定的参数值或者参数文件


---------------------------
以上