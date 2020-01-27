---
layout: post
title: Terraform resource
category: Terraform
keywords: terraform, orchestration, 编排, 开源
---

## Resource是神马

在Terraform语言中，resource是最最最重要的元素，没有之一，其它的元素都是围绕着resource展开的，无非是为了简化resource的编写难度，让resource更加灵活。

resource定义需要在某个基础云环境上需要创建的系列资源、以及资源之间的各种依赖关系，`terraform apply`就是根据模板创建实例资源的过程。

## resource如何定义

>
	resource "aws_instance" "web" {
	  ami           = "ami-a1b2c3d4"
	  instance_type = "t2.micro"
	}

在基本概念中已经写过，不写了。

## resource的相互依赖

resource的相互依赖可以分为隐式依赖和显示依赖两种。

### 隐式依赖

隐式依赖，实际上是资源定义时通过参数引用、资源引用来实现的一种依赖，比如，创建虚拟机依赖于先创建网络。

举个栗子：
>
	variable "server_name" {
	  default = "test_1"
	}
>
	resource "openstack_compute_flavor_v2" "test-flavor" {
	  name  = "my-flavor"
	  ram   = "8096"
	  vcpus = "2"
	  disk  = "20"
	}
>
	resource "openstack_compute_instance_v2" "basic" {
	  name            = var.server_name
	  image_id        = "e5939e45-6be2-462c-a4c2-5b725a5f8b7d"
	  flavor_id       = openstack_compute_flavor_v2.test-flavor
	  security_groups = ["default"]
>
	  network {
	    name = "AT_tempest_net"
	  }
	}

如上述模板，`name = var.server_name`是引用参数server_name，`flavor_id = openstack_compute_flavor_v2.test-flavor`是引用test-flavor实例资源的ID，因此在此发生了隐式依赖，`basic`资源的创建依赖于`test-flavor`资源创建，换句话说，只有`test-flavor`创建完成后，`basic`资源的`flavor_id`才会有值，才能创建。

### 显示依赖

在resource定义中，可以用`depends_on`关键字来定义资源间的相互依赖。depends_on是reosurce的一个元参数（Meta-Arguments）

举个栗子：
>
	variable "server_name" {
	  default = "test_1"
	}
>
	resource "openstack_compute_flavor_v2" "test-flavor" {
	  name  = "my-flavor"
	  ram   = "8096"
	  vcpus = "2"
	  disk  = "20"
	}
>
	resource "openstack_compute_instance_v2" "basic" {
	  name            = var.server_name
	  image_id        = "e5939e45-6be2-462c-a4c2-5b725a5f8b7d"
	  flavor_id       = "m1.tiny"
	  security_groups = ["default"]
>
	  network {
	    name = "AT_tempest_net"
	  }
>	  
	  depends_on = [
	    openstack_compute_flavor_v2.test-flavor,
	  ]
	}

如上模板所述，`basic`资源并没有隐式依赖于`test-flavor`资源，但是我们在`basic`资源中使用了`depends_on`关键字，因此，资源创建顺序依然是`test-flavor`先创建，然后再创建`basic`资源。

## resource的count元参数

在默认情况下，`terraform apply`在创建资源实例的时候，只会创建一个。如果指定count、for_each属性，可以同时创建多个资源实例。

举个栗子：
>
	resource "aws_instance" "server" {
	  count = 4 # create four similar EC2 instances
>
	  ami           = "ami-a1b2c3d4"
	  instance_type = "t2.micro"
>
	  tags = {
	    Name = "Server ${count.index}"
	  }
	}
	
如上所述，加入了`count = 4`后，相同的资源实例会创建4个，同时`Name = "Server ${count.index}"`为4个资源打上不同的标签，`${count.index}`是引用count的方式（以0开始）。

count的值只能是数字(numeric)，不能是其它任何值。

### > 引用count下的资源
>
	<TYPE>.<NAME>[<INDEX>]

比如aws_instance.server[0], aws_instance.server[1]等。

## resource的for_each元参数

如果您的资源实例几乎相同，那么count是合适的。如果它们的一些参数需要不同的值，而这些值不能直接从整数派生出来，那么使用for_each会更安全。

举个栗子：
>
	variable "subnet_ids" {
	  type = list(string)
	}
>
	resource "aws_instance" "server" {
	  # Create one instance for each subnet
	  count = length(var.subnet_ids)
>
	  ami           = "ami-a1b2c3d4"
	  instance_type = "t2.micro"
 	  subnet_id     = var.subnet_ids[count.index]
>
	  tags = {
	    Name = "Server ${count.index}"
	  }
	}

如上模板，这个模板的定义相对来讲比较脆弱，因为资源实例是通过它们的索引而不是列表中的字符串值来标识的。如果从列表中间删除一个元素，那么该元素之后的每个实例都将看到其subnet_id值的变化，从而导致比预期更多的远程对象变化。使用for_each可以提供相同的灵活性，而不需要额外的麻烦。

>
	variable "subnet_ids" {
	  type = list(string)
	}
>
	resource "aws_instance" "server" {
	  for_each = toset(var.subnet_ids)
>
	  ami           = "ami-a1b2c3d4"
	  instance_type = "t2.micro"
	  subnet_id     = each.key # note: each.key and each.value are the same for a set
>
	  tags = {
	    Name = "Server ${each.key}"
	  }
	}
	
***注：toset方法是terraform的一个内置函数，将list转化成set***

## resource的provider元参数

在一个完成的Terraform模板中，provider可以有多个，在资源中可以指定使用某个provider：
>
	# default configuration
	provider "google" {
	  region = "us-central1"
	}
>
	# alternative, aliased configuration
	provider "google" {
	  alias  = "europe"
	  region = "europe-west1"
	}
>
	resource "google_compute_instance" "example" {
	  # This "provider" meta-argument selects the google provider
	  # configuration whose alias is "europe", rather than the
	  # default configuration.
	  provider = google.europe
>
	  # ...
	}

## resource的lifecycle元参数

没深入看，还不了解！：)

## resource的provision和connection元参数

没深入看，还不了解！：)

## 本地资源（Local-Only resource）

有些资源只保存在terraform的state文件中，并不会在云环境中进行创建，所以称之为本地资源。比如random provider里边的random_id、random_integer、random_password等等资源。

## 资源操作超时设置

某些资源可以设置`timeouts`参数，用来自定义资源操作（creating、updating、deleting等）超时时间。
>
	resource "aws_db_instance" "example" {
	  # ...
	  timeouts {
	    create = "60m"
	    delete = "2h"
	  }
	}

***60m表示60分钟，2h表示2小时，200s表示200秒***
