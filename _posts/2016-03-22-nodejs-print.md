---
layout: post
title:  "打印图形小练习"
date:   2016-03-21 15:14:54
categories: nodejs
---

## 千分位格式化字符串 ##

	function comma(str){
		//倒着给字符串添加逗号
		var newstr = "";
		var j = 1;
		for (var i=str.length-1; i>=0; i--) {
			newstr += str[i];
			//每数3个数字加一个逗号，并且过滤掉第一位出现逗号的情况
			if (j%3 == 0 && j != str.length)
				newstr += ','; 
			j++;
		}
	
		//再把新的字符串倒回来就是我们需要的字符串了
		var	str = '';
		for (var k=newstr.length-1; k>=0; k--) {
			str += newstr[k];
		}
		console.log(str);
	}
	
	var str = "1234567890";
	comma(str);

## 菱形 ##

	//简单版本
	function square (num){
		//必须为偶数才行
		num = (num%2==0)?num:num+1;	
		//外层控制打印多少行
		for (var i=1; i<=num; i++) {
			var str = '';
			//内层控制打印多少列
			for (var j=1; j<=num; j++) {
				//通过判断来限制每一行打印什么
				if ( ((i==1 || i==num) && j==num/2) || ((j==num/2-i+1 || j==num/2+i-1) && i>=2 && i<=num/2) || ((j==i-num/2 || j==num+num/2-i) && i>=num/2+1 && i<=num-1)    ) {
					str += '*';
				} else {
					str += ' ';
				}			
			}
			console.log(str);
		}	
	}
	square(24);

## 回形 ##

	//简化版
	function square (num){
		//必须大于1
		if (num <= 1) {
			console.log('参数必须大于1');
			return false;
		}
		var space = Math.ceil(num/4);	
		//外层控制打印多少行
		for (var i=1; i<=num; i++) {
			var str = '';
			//内层控制打印多少列
			for (var j=1; j<=num; j++) {
				//通过判断来限制每一行打印什么
				if ( (i==1 || i==num || j==1 || j==num) || ((i==space || (i==num-space+1)) && j>=space && j<=num-space+1)  ||  ((j==space || (j==num-space+1)) && i>=space && i<=num-space+1)  ) {
					str += '*';
				} else {
					str += ' ';
				}			
			}
			console.log(str);
		}
		
	}
	
	square(10);
	
## 等腰梯形 ##

	function trapezoid(width, height){
		 if (height<3 || height>width/2+1) {
		 	console.log("高度必须大于2，并且高度不能大于宽的一半加1");
		 	return false;
		 }
		//外层控制打印多少行
		for (var i=1; i<=height; i++) {
			var str = '';
			//内层控制打印多少列
			for (var j=1; j<=width; j++) {
				//通过判断来限制每一行打印什么
				if (  ((i==1 && j>=height && j<=width-height+1) || i==height) || (i>1 && i<height && j==height-i+1) ||  (i>1 && i<height && j==width-height+i)  ) {
					str += '*';
				} else {
					str += ' ';
				}			
			}
			console.log(str);
		}
	}
	
	trapezoid(20,8);


