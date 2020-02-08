---
layout: post
title: Terraform Data Source
category: Terraform
keywords: terraform, data source, orchestration, 编排, 开源
---

## Data Source

Data Srouce允许获取或计算数据，以便在Terraform配置的其他地方使用。数据源的使用允许一个Terraform配置使用定义在Terraform外部的信息，或者由另一个单独的Terraform配置定义的信息。

>
	# Find the latest available AMI that is tagged with Component = web
	data "aws_ami" "web" {
	  filter {
	    name   = "state"
	    values = ["available"]
	  }
>
	  filter {
	    name   = "tag:Component"
	    values = ["web"]
	  }
>
	  most_recent = true
	}
