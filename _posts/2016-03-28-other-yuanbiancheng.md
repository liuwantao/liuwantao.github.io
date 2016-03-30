
---
layout: post
title: "元编程小demo"
date: 2016-03-28 16:14:54
categories: other
---

	//元编程的小demo
	//参数1:公鸡单价
	//参数2:母鸡单价
	//参数3:小鸡单价
	//参数4:购买总数
	//参数5:购买的总钱
	//参数6:要编译的语言
	//尽量写整数,小数还没处理好
	function getji(gmoney, mmoney, xmoney, allnum, allmoney, totype){
		var fs = require('fs');
		var content = "";
		if (totype == "js") {
			content = "for(var g=0;"+gmoney+"*g<=100;g++){for(var m=0;"+mmoney+"*m<=100;m++){for(var x=0;x<=100*"+xmoney+";x++){if((g*"+gmoney+"+m*"+mmoney+"+x/"+xmoney+"=="+allmoney+")&&(g+m+x=="+allnum+")){console.log('公鸡个数'+g+'---母鸡个数'+m+'---小鸡个数'+x);}}}}";
			fs.writeFile('jj.js',content,function(err){
				if (err) {
					throw err;
				} else {
					console.log('编译成功!,文件名为jj.js,请使用node jj.js执行!');
				}
			});
		} else if (totype == "php") {
			//原理一样,以后再写
		}
	}
	
	//接受控制台输入的参数
	var parm1 = process.argv[2];
	var parm2 = process.argv[3];
	var parm3 = process.argv[4];
	var parm4 = process.argv[5];
	var parm5 = process.argv[6];
	var parm6 = process.argv[7];
	
	//控制台提示信息
	if (!parm1 || !parm2 || !parm3 || !parm4 || !parm5 || !parm6) {
		console.log("//参数1:公鸡单价//参数2:母鸡单价//参数3:小鸡单价//参数4:购买总数//参数5:购买的总钱//参数6:要编译的语言//多个参数以空格分开//尽量写整数,小数还没处理好");
	} else {
		//调用函数
		getji(parm1,parm2,parm3,parm4,parm5,parm6);
	}





