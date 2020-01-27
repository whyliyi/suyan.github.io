---
layout: post
title: Terraform基本概念
category: Terraform
keywords: terraform, orchestration, 编排, 开源
---

Terraform是一个开源编排工具，支持的插件众多，相对来讲比较灵活，兼容很多的公有云、私有云。

Terraform有自己的配置语言（Configuration Language）。配置语言最主要的功能就是描述需要编排的资源（resource），其它的都是围绕resource展开，其存在意义就是让resource的定义更加灵活、方便。


# Terraform Language基本语法

>
	resource "aws_vpc" "main" {
 	 	cidr_block = var.base_cidr_block
	}
>	
	<BLOCK TYPE> "<BLOCK LABEL>" "<BLOCK LABEL>" {
  		# Block body
  		<IDENTIFIER> = <EXPRESSION> # Argument
	}
	
* 遵循Terraform配置语言语法描述的一个资源集合，称之为模板，或配置文件，为了方便，统称为模板
* 模板以`.tf`、`.tf.json`扩展名结尾，语法遵循Terraform语言语法

举个栗子：
>
	variable "aws_region" {}
>	
	variable "base_cidr_block" {
	  description = "A /16 CIDR range definition, such as 10.1.0.0/16, that 	the VPC will use"
	  default = "10.1.0.0/16"
	}
>
	variable "availability_zones" {
	  description = "A list of availability zones in which to create subnets"
	  type = list(string)
	}
>
	provider "aws" {
	  region = var.aws_region
	}
>
	resource "aws_vpc" "main" {
	  # Referencing the base_cidr_block variable allows the network address
	  # to be changed without modifying the configuration.
	  cidr_block = var.base_cidr_block
	}
>
	resource "aws_subnet" "az" {
	  # Create one subnet for each given availability zone.
	  count = length(var.availability_zones)
>
	  # For each subnet, use one of the specified availability zones.
	  availability_zone = var.availability_zones[count.index]
>
	  # By referencing the aws_vpc.main object, Terraform knows that the subnet
	  # must be created only after the VPC is created.
	  vpc_id = aws_vpc.main.id
>
	  # Built-in functions and operators can be used for simple transformations of
	  # values, such as computing a subnet address. Here we create a /20 prefix for
	  # each subnet, using consecutive addresses for each availability zone,
	  # such as 10.1.16.0/20 .
	  cidr_block = cidrsubnet(aws_vpc.main.cidr_block, 4, count.index+1)
	}

上述模板仅仅是资源描述的集合，并不会在云环境中创建实例，在使用terraform apply命令后，会在云环境中创建实例。当然，实例模板不是完整模板，直接apply是会出错的。

# 基本概念

## 资源（Resource）

在Terraform语言中，resource是最重要的元素，没有之一
>
	resource "aws_instance" "web" {
	  ami           = "ami-a1b2c3d4"
	  instance_type = "t2.micro"
	}

上述栗子中，

* resource是Terraform关键字，表明此代码块定义的是一个Terraform资源
* aws_instance是指名资源类型，是[AWS服务中的虚拟机实例](!https://www.terraform.io/docs/providers/aws/r/instance.html)。
* web是Terraform运行时的一个内部资源名称，在同一个module中资源名称不允许重复，在module中其它的资源可以引用这个名称。**资源名称只能以字母、下划线开头，只能包含字母、数字、下划线、中划线**
* 在`{}`中是资源本身在创建的时候需要的配置参数，参数的定义在provider中定义


## Provider

这个用中文怎么翻译呢？？还是不翻译了，就是Provider

Resource定义的是资源参数，但是资源可以执行的操作全部在Provider中定义。每一个Provider提供一套资源类型，定义每个资源可以指定的参数、如何调用云端API实现资源的各种操作等；
>
	provider "google" {
	  project = "acme-app"
	  region  = "us-central1"
	}

* provider是Terraform语言中的关键字，指名当前代码块定义的是一个provider
* google是provider的名字
* `{}`代码快中定义的是调用google云API需要的必选参数

在某个资源中，可以指定provider创建：
>
	resource "aws_instance" "foo" {
	  provider = aws.west
>
	  # ...
	}
* Provider可以用户自定义，参考[自定义Provider](!https://www.terraform.io/docs/extend/writing-custom-providers.html)
* 自定义Provider不能通过terraform init来初始化，需要手动的安装，直接把自定义的可执行文件放到`~/.terraform.d/plugins`目录下，此时terrafrom init可以获取到Provider
* Provider可执行文件命名规则：`terraform-provider-<NAME>_vX.Y.Z`，其中`<Name>`是Provider名称，`vX.Y.Z`是Provider的版本号。

## 入参（Input Variables）

>
	variable "region" {
	  default = "us-east-1"
	}
如上，入参是为了编码我们的Terraform配置文件（模板）中可以变化的参数定义，避免模板中出现硬编码，将可能变化或者需要用户写入的参数以入参形势暴露给用户，方便用户输入。

对于定义的入参，可以以以下方式引用：
>
	provider "aws" {
	  region     = var.region
	}
在调用terraform apply命令时可以修改参数的值：
>
	$ terraform apply \
	  -var 'region=us-east-2'
	# ...
也可以定义`terraform.tfvars`文件，作为参数传入方式，文件内容如下：
>
	region = "us-east-2"
可以定义多个`.tfvars`参数文件，在terraform apply时直接引用文件即可：
>
	$ terraform apply \
	  -var-file="secret.tfvars" \
	  -var-file="production.tfvars"	
还可以通过环境变量的方式定义参数值，形式如：`TF_VAR_name`，比如`TF_VAR_region`，修改`TF_VAR_region`环境变量的值可以修改region参数的值

## 出参（Output Values）

在很多时候，我们需要将在云环境上创建的实例某些属性暴露给用户，比如创建的虚拟机需要暴露网卡地址，创建的web server需要暴露访问地址，因此有了出参（Output Values），出参可以将实例的属性或者组合的属性暴露给用户，最大程度降低用户访问实例资源的可能。

举个栗子：我们需要创建一个mysql数据库，用户实际上不需要关心mysql创建的虚拟机是什么，但是用户需要知道mysql虚拟的IP、访问端口等，因此我们通过定义Output来将mysql的相关信息返回：
>
	output "mysql_ip" {
	  value = aws_instance.server.private_ip
	}
>	
	output "mysql_port" {
	  value = var.port
	}
其中第一个参数mysql_ip是创建的虚拟机实例的IP地址，第二个参数mysql_port是用户输入或者默认的端口地址

Output可以有很多设置，比如`sensitive`可以保证输出参数在控制台不可见，但是在模块内部可见。`depends_on`可以显示指定出参的依赖资源。等等

## 局部参数（Local Values）

局部参数可以在某一个模块内定义，主要用途是方便在同一个模块中多次使用，一般来说是常量。
>
	locals {
	  # Ids for multiple sets of EC2 instances, merged together
	  instance_ids = concat(aws_instance.blue.*.id, aws_instance.green.*.id)
	}
>
	locals {
	  # Common tags to be assigned to all resources
	  common_tags = {
	    Service = local.service_name
	    Owner   = local.owner
	  }
	}
## 模块（Modules）

模块可以包含多个Terraform配置文件（模板），主要是为了方便资源重用、复杂场景模块化需要。在同一个工作目录内定义一系列`.tf`文件，来整合一个复杂场景，充分利用一些基础资源等等。

module有众多好处，一两句话也说不清除，后续重点学习一下，写一篇专门的学习笔记。

##数据定义（Data Sources）

没看懂，先不写了。:)

以上就是一些基本概念吧。










