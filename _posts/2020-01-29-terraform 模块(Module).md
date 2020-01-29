---
layout: post
title: Terraform 模块(Module)
category: Terraform
keywords: terraform, module, orchestration, 编排, 开源
---

## 什么是模块(Module)

模块是多个配置文件（模板）的集合，最大限度保证Terraform配置的**灵活性**和**可重用性**。每一个Terraform配置至少有一个模块，即根模块（root模块）。模块之间可以相互依赖、相互引用，不同的workspace之间的模块也可以相互访问。

## 模块的定义
>
	module "servers" {
	  source = "./app-cluster"
>
	  servers = 5
	}

`module`是模块定义的关键字，`"servers"`是terraform定义的逻辑名称，可以被其它模块调用。`{ }`里的内容就是模块的具体定义。

`source = "./app-cluster"`是指引用`./app-cluster`作为父模板，`servers = 5`是在`./app-cluster`中需要填写的入参。

`source`后的路径可以是本地路径，也可以是远端路径。

***如果模块被修改，必须用`terraform init`来初始化，如果模块已经在workspace内被安装，必须用`terraform init -upgrade`来更新***

## 访问模块的输出参数

在模块内定义的资源是不能被直接访问的，需要通过模块的输出参数（Output Values）来访问。

举个栗子：

如果`./app-cluster`定义了`instance_ids`这个输出参数，通过如下方式可以访问：

>
	resource "aws_elb" "example" {
	  # ...
>
	  instances = module.servers.instance_ids
	}
	
## 模块版本号

***模块的版本号，只支持在[Terraform Cloud Registry](!https://registry.terraform.io/)仓库里边注册的模块，其它的方式的模块并不支持***

>
	module "consul" {
	  source  = "hashicorp/consul/aws"
	  version = "0.0.5"
>
	  servers = 3
	}

如上述栗子，在代码中用`version`属性可以直接指定使用的模板版本号。

 * = 1.2.0: 指定使用1.2.0版本
 * >= 1.2.0: 1.2.0或者更新版本
 * <= 1.2.0: 1.2.0或者更老的版本
 * ~> 1.2.0: >= 1.2.0 到 < 1.3.0之间的非beta版本, 例如：1.2.X
 * ~> 1.2: >= 1.2.0 到 < 2.0.0之间的非beta版本, 比如1.X.Y
 * >= 1.0.0, <= 2.0.0: 1.0.0 到 2.0.0 之间的任意版本

## 一个完成的模块定义

>
	$ tree complete-module/
	.
	├── README.md
	├── main.tf
	├── variables.tf
	├── outputs.tf
	├── ...
	├── modules/
	│   ├── nestedA/
	│   │   ├── README.md
	│   │   ├── variables.tf
	│   │   ├── main.tf
	│   │   ├── outputs.tf
	│   ├── nestedB/
	│   ├── .../
	├── examples/
	│   ├── exampleA/
	│   │   ├── main.tf
	│   ├── exampleB/
	│   ├── .../
	
## 同一个module的多实例化

>
	# my_buckets.tf
>
	module "assets_bucket" {
	  source = "./publish_bucket"
	  name   = "assets"
	}
>
	module "media_bucket" {
	  source = "./publish_bucket"
	  name   = "media"
	}

以下是module定义：
>
	# publish_bucket/bucket-and-cloudfront.tf
>
	variable "name" {} # this is the input parameter of the module
>
	resource "aws_s3_bucket" "example" {
	  # ...
	}
>
	resource "aws_iam_user" "deploy_user" {
	  # ...
	}
	
可以看出，同一个`./publish_bucket`可以被多次引用。

## 模块发布

模块可以通过[Terraform Cloud Registry](!https://registry.terraform.io/)直接发布，但是需要公网才能访问，因此对私有云用户不太合适。