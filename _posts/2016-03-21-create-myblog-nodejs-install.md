---
layout: post
title: "node.js安装和配置"
date: 2016-03-21 22:14:54
categories: nodejs
excerpt: nodejs
---

##安装 （mac ox）

###方式一

直接pkg包安装：

下载地址：[nodejs4.4.0LTS](https://nodejs.org/en/)

双击打开包就安装完成

    node -v  //查看node版本
  
    npm -v  //查看npm版本
  
##设置国内的淘宝源

###临时修改

    npm config set registry http://registry.cnpmjs.org //配置指向源
    
###永久设置

找到npm配置文件：

    vi  ~/.npmrc   //打开配置文件

添加：
  
    registry =https://registry.npm.taobao.org   //写入配置文件
    



