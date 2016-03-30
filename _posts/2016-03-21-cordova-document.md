---
layout: post
title: "cordova简单使用"
date: 2016-03-21 23:14:54
categories: cordova
---

## 安装和基本语法

### 优先安装nodejs

### 开始安装

	$  sudo npm install -g cordova

>要有耐心,因为文件很大

### 创建一个项目

	$  cordova create test com.lwt.test Test
							本地目录   唯一ID    安装后显示的应用名字
							
### 添加平台

进入到项目目录下

	$  cd test
	
添加平台

	$  cordova platforms add ios
	
查看可添加平台

	$  cordova plateforms list
	
### 初始化项目

	$  cordova build ios
	
### 测试项目

真机测试
	
	$  cordova run ios
	
模拟器测试

	$  cordova emulate ios
	
	
## 中文手册地址

[cordova中文手册](http://wiki.jikexueyuan.com/project/apache-cordova-tutorial/setting-workshop-files.html)





