---
layout: post
title: "MAC下搭建ios和android开发环境"
date: 2016-03-28 15:14:54
categories: cordova
---

## ios环境搭建

这个比较简单,只要确定cordova和xcode安装的没问题,直接这样就行:

	$  cordova platforms ios
	
	$  cordova build ios
	
	$  cordova emulate ios  //测试
	
>如果报错,找到对应的文件,把权限改成普通用户也有读写的权限

## android环境搭建

首先安装java编译器: jdk 1.8 以上 ,去网上下就行了

	$  javac -version //查看java版本

然后安装android集成开发环境 : android studio

	就是个pkg包,双击安装就行了
	
>这个过程会下载很多东西,所以要耐心等待,并不是卡死了

安装完成后添加android进行测试:

	$  cordova platforms android
	
	$  cordova buil android
	
	$  cordova emulate android  //测试


