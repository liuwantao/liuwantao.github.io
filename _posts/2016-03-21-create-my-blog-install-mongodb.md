---
layout: post
title: "mongodb简介和安装"
date: 2016-03-21 22:14:54
categories: Sql
excerpt: "mongodb简介和安装"
---

##什么是mongodb

mongodb  分布式文档存储数据库

 `mongodb` 是一个介于关系数据库和非关系数据库之间的产品，是非关系数据库当中功能最丰富，最像关系数据库的。他支持的数据结构非常松散，是类似json的bson格式，因此可以存储比较复杂的数据类型。Mongo最大的特点是他支持的查询语言非常强大，其语法有点类似于面向对象的查询语言，几乎可以实现类似关系数据库单表查询的绝大部分功能，而且还支持对数据建立索引。

##特点

它的特点是高性能、易部署、易使用，存储数据非常方便。主要功能特性有：

* 面向集合存储，易存储对象类型的数据。
* mongodb集群
* 模式自由。
* 支持动态查询。
* 支持完全索引，包含内部对象。
* 支持查询。
* 支持复制和故障恢复。
* 使用高效的二进制数据存储，包括大型对象（如视频等）。
* 自动处理碎片，以支持云计算层次的扩展性。
* 支持RUBY，PYTHON，JAVA，C++，PHP，C#等多种语言。
* 文件存储格式为BSON（一种JSON的扩展）。
* 可通过网络访问。

##安装 （基于mac ox）
###方式一：

下载源码包：[3.2.4下载地址](https://www.mongodb.org/downloads?_ga=1.206526102.42745136.1458557150#production)

找个地方解压下载的压缩包：

	$ tar -zxvf mongodb-osx-x86_64-3.2.4.tar /usr/local/
	
根目录创建数据存储的仓库：

	mkdir /data/
	mkdir /data/db
	sudo chmod 777 /data/db -R 

进入解压包：

	$ cd /usr/local/mongodb-osx-x86_64-3.2.4
	$ cd ./bin
	$ ./mongod
	
	此时显示的应该是端口情况和日志文件检测情况，别关这个终端
	
新打开一个终端：

	$ cd /usr/local/mongodb-osx-x86_64-3.2.4/bin
	$ ./mongo  
	
这时候数据库就打开了，我们可以这样查看数据库的信息：
	
	> show databases;  	

 
