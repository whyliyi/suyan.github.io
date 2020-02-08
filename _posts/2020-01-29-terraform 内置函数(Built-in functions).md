---
layout: post
title: Terraform 内置函数（built-in functions）
category: Terraform
keywords: terraform, built-in functions, orchestration, 编排, 开源, 内置函数, 函数
---

## Built-in Functions

Terraform包含了很多内置函数，内置函数可以在terraform控制端使用，比如：

>
	> max(5, 12, 9)
	12 

## 数字类的函数

* abs：取数字的绝对值

>
	> abs(23)
	23
	> abs(0)
	0
	> abs(-12.4)
	12.4
	
* ceil：上取整

> 
	> ceil(5)
	5
	> ceil(5.1)
	6
	
* floor：下取整

>
	> floor(5)
	5
	> floor(4.9)
	4

* log： 取对数

>
	> log(16, 2)
	4

* max/min：取最大/最小

>
	> min(12, 54, 3)
	3
	
* 	parseint：将字符串转换成对应进制的数字

> 
	> parseint("100", 10)
	100
>
	> parseint("FF", 16)
	255
>
	> parseint("-10", 16)
	-16
>
	> parseint("1011111011101111", 2)
	48879

* pow：求幂

>
	> pow(3, 2)
	9

* signum：求符号

>
	> signum(-13)
	-1
	> signum(0)
	0
	> signum(344)
	1	

## 字符串函数

* chomp：删除换行符

> 
	> chomp("hello\n\n")
	hello

* format：[格式化字符串](!https://www.terraform.io/docs/configuration/functions/format.html)

> 
	> format("Hello, %s!", var.name)
	Hello, Valentina!

* formatlist：[格式化字符串列表](!https://www.terraform.io/docs/configuration/functions/formatlist.html)

>
	> formatlist("%s, %s!", "Salutations", ["Valentina", "Ander", "Olivia", "Sam"])
	[
	  "Salutations, Valentina!",
	  "Salutations, Ander!",
	  "Salutations, Olivia!",
	  "Salutations, Sam!",
	]
	
* indent：多行字符串缩进指定字符串数

>
	> "  items: %{indent(2, "[\n  foo,\n  bar,\n]\n")}"
	  items: [
	    foo,
	    bar,
	  ]

* join：用指定分隔符连接列表中的字符串

>
	> join(", ", ["foo", "bar", "baz"])
	foo, bar, baz

* split：按照指定分隔符分割字符串

>
	> split("+", "1+2+3")
	[1, 2, 3,]

* lower：字符串转化为全小写
* upper：字符串转化为全大写
* regex：[按照正则表达式匹配字符串](!https://www.terraform.io/docs/configuration/functions/regex.html)，并且返回第一个匹配成功的子字符串

>
	> regex("[a-z]+", "53453453.345345aaabbbccc23454")
	aaabbbccc
	
* regexall：[按照正则表达式匹配字符串，并返回匹配的子字符串列表](!https://www.terraform.io/docs/configuration/functions/regexall.html)

>
	> regexall("[a-z]+", "1234abcd5678efgh9")
	[
	  "abcd",
	  "efgh",
	]
	
* replace：替换字符串中符合条件的子字符串

>
	> replace("1 + 2 + 3", "+", "-")
	1 - 2 - 3
	
* strrev：字符串逆序

>
	> strrev("hello")
	olleh

* substr：字符串分片

>
	> substr("hello world", 1, 4)
	ello
	
* title：单词的首字母大写

>
	> title("hello world")
	Hello World
	
* trim：删除字符串首尾指定字符串

>
	> trim("?!hello?!", "!?")
	hello
	
* trimprefix：删除字符串指定前缀

>
	> trimprefix("helloworld", "hello")
	world
	
* trimsuffix：删除字符串指定后缀

>
	> trimsuffix("helloworld", "world")
	hello

* trimspace：删除前后空格

>
	> trimspace("  hello\n\n")
	hello

## [集合类函数](!https://www.terraform.io/docs/configuration/functions/chunklist.html)

## [字符串编码函数](!https://www.terraform.io/docs/configuration/functions/base64decode.html)

## [文件系统函数](!https://www.terraform.io/docs/configuration/functions/abspath.html)

## [时间函数](!https://www.terraform.io/docs/configuration/functions/formatdate.html)

## [加密函数](!https://www.terraform.io/docs/configuration/functions/base64sha256.html)

## [网络相关函数](!https://www.terraform.io/docs/configuration/functions/cidrhost.html)

## [类型转换函数](!https://www.terraform.io/docs/configuration/functions/can.html)
	
* can：计算给定的表达式，是否能够无错误的算出结果

>
	> local.foo
	{
	  "bar" = "baz"
	}
	> can(local.foo.bar)
	true
	> can(local.foo.boop)
	false
	> can(local.nonexist)
>
	Error: Reference to undeclared local value
>
	A local value with the name "nonexist" has not been declared.

* tobool：转换成bool类型，仅仅是true、false、"true"、"false"可以转换
* tolist：将参数转化成列表类型

>
	> tolist(["a", "b", 3])
	[
	  "a",
	  "b",
	  "3",
	]	

* tomap：转换成map类型

>
	> tomap({"a" = "foo", "b" = true})
	{
	  "a" = "foo"
	  "b" = "true"
	}

* tonumber：转换成数字

>
	> tonumber(1)
	1
	> tonumber("1")
	1
	> tonumber("no")
	Error: Invalid function argument

* toset：转换成set类型

>
	> toset(["c", "b", "b", 3])
	[
	  "b",
	  "c",
	  "3",
	]

* tostring：转换成字符串类型, 只对string、number、bool类型有效

>
	> tostring("hello")
	hello
	> tostring(1)
	1
	> tostring(true)
	true
	> tostring([])
	Error: Invalid function argument
	
* try：计算所有的表达式，并返回第一个计算成功的结果，并且不会抛出任何错误

>
	> local.foo
	{
	  "bar" = "baz"
	}
	> try(local.foo.bar, "fallback")
	baz
	> try(local.foo.boop, "fallback")
	fallback

-------
以上