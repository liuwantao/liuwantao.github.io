---
layout: post
title: "nodejs搭建简单的http服务器"
date: 2016-03-28 14:14:54
categories: nodejs
---

## 搭建简单的nodejs服务器
通过一些简单配置我们就可以搭建一台基于nodejs的http服务器

### 通过switch配置url路由的方法
	// 这是一个简单的Node HTTP服务器,能处理当前目录的文件
	// 并能实现两种特殊的URL用于测试
	// 用HTTP://localhost:8000或http://127.0.0.1:8000连接这个服务器
	
	// 首先加载所有需要用到的模块
	var http = require('http');        // 加载http服务api模块
	var fs = require('fs');            // 加载fs文件服务api模块
	var server = new http.Server();    // 创建新的HTTP服务器
	var urlapi = require('url');        // 创建url路由api模块
	server.listen(8000);              // 监听端口8000

	// 使用on方法注册事件处理,该事件一直被监听,任何的请求都会进入回调函数,执行相应的操作
	server.on('request', function(request, response) { // 当有request请求的时候触发处理函数    
	    // 解析请求的URL
	    var url = urlapi.parse(request.url);
	    
	    //监听请求的网站,以当前脚本目录为根目录的url地址
	    console.log(url.pathname);
	    
	    // 特殊URL会让服务器在发送响应前先等待
	    switch(url.pathname) {    //判断请求的路径信息
	    case ''||'/' : // 处理请求的网站根目录,指定加载对应的文件夹,一般以根目录的index.html为默认,nodejs是高效流处理的方案,也可以通过配置文件来配置
	        fs.readFile("./page/index.html", function(err, content){  //打开请求的文件
	            if(err) {  //输出错误信息,也可以自定义错误信息
	                response.writeHead(404, { 'Content-Type':'text/plain; charset="UTF-8"' });
	                response.write(err.message);
	                response.end();
	            } else {  //请求成功返回数据
	                response.writeHead(200, { 'Content-Type' : 'text/html; charset=UTF-8' });  //告诉相应头文件,返回数据的类型
	                response.write(content); //返回的内容,有时候还会加上buter数据类型
	                response.end();  //结束响应,不写的话,会一直处于响应状态,页面不会显示内容
	            }
	        });
	        break;
	    case '/test/delay':// 此处用于模拟缓慢的网络连接
	        // 使用查询字符串来获取延迟时长,或者2000毫秒
	        var delay = parseInt(url.query) || 2000;
	        // 设置响应状态和头
	        response.writeHead(200, {'Content-type':'text/plain; charset=UTF-8'});
	        // 立即开始编写响应主体
	        response.write('Sleeping for' + delay + ' milliseconds...');
	        // 在之后调用的另一个函数中完成响应
	        setTimeout(function(){
	            response.write('done.');
	            response.end();
	        }, delay);
	        break;
	
	    case '/test/mirror':// 如果请求是test/mirror,则原文返回它
	        // 响应状态和头
	        response.writeHead(200, {'Content-type':'text/plain; charset=UTF-8'});
	        // 用请求的内容开始编写响应主体
	        response.write(request.mothod + ' ' + request.url + ' HTTP/' + request.httpVersion + '\r\n');
	        // 所有的请求头
	        for (var h in request.headers) {
	            response.write(h + ':' + request.headers[h] + '\r\n');
	        }
	        response.write('\r\n');// 使用额外的空白行来结束头
	        // 在这些事件处理程序函数中完成响应
	        // 当请求主体的数据块完成时,把其写入响应中
	        request.on('data', function(chunk) { response.write(chunk); });
	        // 当请求结束时,响应也完成
	        request.on('end', function(chunk){ response.end(); });
	        break;
	        
	    case '/json' : // 模拟JSON数据返回
	        // 响应状态和头
	        response.writeHead(200, {'Content-type':'application/json; charset=UTF-8'});
	        response.write(JSON.stringify({test:'success'}));
	        response.end();
	        break;
	        
	    default:// 处理来自本地目录的文件,主要是一些静态资源文件,搭建静态服务器还有其他的方法
	        var filename = url.pathname.substring(1);    // 去掉前导'/'
	        var type = getType(filename.substring(filename.lastIndexOf('.')+1));
	        console.log(filename);  //取得文件类型  css   js  ....
	        // 异步读取文件,并将内容作为单独的数据模块传给回调函数
	        // 对于确实很大的文件,使用流API fs.createReadStream()更好
	        fs.readFile(filename, function(err, content){
	            if(err) {
	                response.writeHead(404, { 'Content-Type':'text/plain; charset="UTF-8"' });
	                response.write(err.message);
	                response.end();
	            } else {
	                response.writeHead(200, { 'Content-Type' : type });
	                response.write(content);
	                response.end();
	            }
	        });
	        break;
	    }     
	});
	
	//这里定义了一个用来判断文件类型的函数
	function getType(endTag){
	    var type=null;
	    switch(endTag){
	    case 'html' :
    		  type = 'text/html; charset=UTF-8';
	        break;
	    case 'htm' :
	        type = 'text/html; charset=UTF-8';
	        break;
	    case 'js' : 
	        type = 'application/javascript; charset="UTF-8"';
	        break;
	    case 'css' :
	        type = 'text/css; charset="UTF-8"';
	        break;
	    case 'txt' :
	        type = 'text/plain; charset="UTF-8"';
	        break;
	    case 'manifest' :
	        type = 'text/cache-manifest; charset="UTF-8"';
	        break;
	    default :
	        type = 'application/octet-stream';
	        break;
	    }
	    return type;
	}


