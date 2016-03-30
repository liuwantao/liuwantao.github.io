---
layout: post
title: "exress中间件mongoose"
date: 2016-03-31 4:14:54
categories: nodejs
---

## express中间件mongoose基本的CRUD

	//导入基本的中间件
	var express = require('express');
	var router = express.Router();
	var mongoose = require('mongoose');
	
	//链接数据库
	var db = mongoose.createConnection('localhost','lwtTest');


### 定义chema，也就是传统意义的表结构

	var personSchema = new mongoose.Schema({
	    _name:String,   //定义一个属性name，类型为String
	    _phone:Number
	});

### 定义Model

	var personModel = db.model('lwtArr',personSchema);

## 常用CRUD操作

### 增加测试数据

	var obj = {
	    _name:'lwt',
	    _phone:111
	}

### 方法1

	var personEntity = new personModel(obj);
	personEntity.save(function(err){
	    if (err) {
	        console.log('insert failed');
	    } else {
	        console.log('insert success');
	    }
	
	});

### 方法2

	personModel.create(obj, function(err){
	    if (err) {
	        console.log('insert failed');
	    } else {
	        console.log('insert success');
	    }
	});

>注意:
>在添加数据的时候,如果不指定id属性,id会默认为一个字符串对象
>区别:
>Entity是model的对象,用它来添加数据的时候会把隐藏属性一起存入数据库,有几率报错
>model只能添加纯净的json对象,不能添加它创建的实体


## 修改操作

### 方法1

	personModel.update({_phone:222},{$set:{_name:'把222的都改了'}},{ multi: true },function(err){
	    if (err) {
	        console.log('update failed');
	    } else {
	        console.log('update success');
	    }
	});

### 方法2

	var id = "56fc1bbe7c2bce52c38fc5f8";
	personModel.findById(id,function(err,person){
	    if (err) {
	        console.log('update failed');
	    } else {
	        person._name = 'from_id';
	        person.save(function(err){});
	        console.log('update success');
	    }
	});
	
>注意:
>如果更新的数据比较少的话,用方法一很方便的,其中的{ multi: true }表示同时更新所有数据,{ multi: false }或者省略则只跟新一条


## 查询数据

### 直接查询

	personModel.findOne({'_phone':111},function(err,person){
	    if (err) {
	        console.log('select failed');
	    } else {
	        console.log(person);
	    }
	});

### 查询一条

	var query = personModel.findOne({'_phone':111});
	query.select();
	query.exec(function(err,person){
	    if (err) {
	        console.log('select failed');
	    } else {
	        console.log(person);
	    }
	});

### 查询多条

	var query = personModel.find({'_phone':222});
	query.select();
	query.exec(function(err,person){
	    if (err) {
	        console.log('select failed');
	    } else {
	        console.log(person);
	    }
	});

### 链式查询

	personModel
	    .where('_phone')
	    .where('_name').equals('aaa')
	    .limit(2)
	    .select()
	    .exec(function(err,person){
	        if (err) {
	            console.log('select failed');
	        } else {
	            console.log(person);
	        }
	    });
    
### 另外附详细链式查询表

	personModel            //model 对象
	    .find({ occupation: /host/ })   //条件
	    .where('name.last').equals('Ghost')  //字段  等于  值
	    .where('age').gt(17).lt(66)   //  范围
	    .where('likes').in(['vaporizing', 'talking'])  // 范围
	    .limit(10)  //  查几条
	    .skip(20)  //跳过几条
	    .asc('age')  //  排序
	    .sort('-occupation')  //
	    .slaveOk()  //
	     .hint({ age: 1, name: 1 })
	    .select('name occupation')  //执行查询 ()中的值可以省略,只是别名
	    .exec(callback);  //回调函数,处理查询结果

## 删除操作

	var personEntity = new personModel();
	personModel.remove({_id:"56fc281ab64732d8c33b4db7"},function(err){
	    if (err) {
	        console.log('delete failed');
	    } else {
	        console.log('detete success');
	    }
	
	});
	
>注意:
>删除条件一定是唯一的



