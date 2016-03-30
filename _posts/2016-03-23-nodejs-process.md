---
layout: post
title: "process"
date: 2016-03-23 15:14:54
categories: nodejs
---

## node.js api-process

nodejs的process是一个全局对象，提供了一些方法和属性

### 监听一个事件

	process.on('内置事件或自定义事件',function(参数){
		执行的代码；
	});
	
	process.emit('事件名字'，‘参数’，参数2); ／／触发一个事件

### 事件‘exit’

当进程将要退出时触发。这是一个在固定时间检查模块状态（如单元测试）的好时机。需要注意的是 'exit' 的回调结束后，主事件循环将不再运行，所以计时器也会失效：

	process.on('exit', function() {
	  // 设置一个延迟执行
	  setTimeout(function() {
	    console.log('主事件循环已停止，所以不会执行');
	  }, 0);
	  console.log('退出前执行');
	});
	setTimeout(function() {
	  console.log('1');
	}, 500);
### 事件'uncaughtException'

捕获那些我们没有 try catch 的异常错误：

	process.on('uncaughtException', function() {
	    console.log('捕获到一个异常');
	});
	var a = '123';
	a.a(); //触发异常事件
	console.log('这句话扑街了，不会显示出来，因为去执行异常函数去了');
	
不建议这样粗略的使用这个函数，建议使用domains
	
### 事件'SIGINT'
	
捕获当前进程接收到的信号（如按下了 ctrl + c）：

	process.on('SIGINT', function() {
	  console.log('收到 SIGINT 信号。');
	});
	console.log('试着按下 ctrl + C');
	setTimeout(function() {
	  console.log('end');
	}, 50000);
	
还可以自定义事件名字来监听process事件：

	 process.on('SIGUSR1', function (d) { //这里监听 SIGUSR1 事件
	      console.log('Bye-'+d); //这里将输出Bye-Bye,然后推出进程
	      process.exit(0);
	    }); 
	  process.emit('SIGUSR1', 'Bye'); //利用emit触发SIGUSR1，然后传参数为Bye

### process.stdout

一个指向标准输出流(stdout)的 可写的流(Writable Stream)：

	console.log = function (d) {
	  process.stdout.write(d + '\n');
	};
	
另可使用 process.stdout.isTTY 来判断当前是否处于 TTY 上下文。

### process.stderr

一个指向标准错误流(stderr)的 可写的流(Writable Stream):

	process.stderr.write('输出一行标准错误流，效果跟stdout没差');
	
### process.stdin

一个指向 标准输入流(stdin) 的可读流(Readable Stream)。标准输入流默认是暂停 (pause) 的，所以必须要调用 process.stdin.resume() 来恢复 (resume) 接收:

	process.stdin.on('end', function() {
	  process.stdout.write('end');
	});
	function gets(cb){
	  process.stdin.setEncoding('utf8');
	  //输入进入流模式（flowing-mode，默认关闭，需用resume开启），注意开启后将无法read到数据
	  //见 https://github.com/nodejs/node-v0.x-archive/issues/5813
	  process.stdin.resume();
	  process.stdin.on('data', function(chunk) {
	    console.log('start!');
	    //去掉下一行可一直监听输入，即保持标准输入流为开启模式
	    process.stdin.pause();
	    cb(chunk);
	  });
	  console.log('试着在键盘敲几个字然后按回车吧');
	}
	gets(function(reuslt){
	  console.log("["+reuslt+"]");
	  //process.stdin.emit('end'); //触发end事件
	});

>注意：会连回车一起输入，所以字符串会多一个\n，在使用输入的
>字符串的时候，要记得用trim()删除两端的空白

### process.argv

返回当前命令行指令参数 ，但不包括node特殊(node-specific) 的命令行选项（参数）。常规第一个元素会是 'node'， 第二个元素将是 .Js 文件的名称。接下来的元素依次是命令行传入的参数：

	//试着执行 $node --harmony argv.js a b
	console.log(process.argv); //[ 'node', 'E:\\github\\nodeAPI\\process\\argv.js', 'a', 'b' ]
	process.argv.forEach(function(val, index, array) {
	    console.log(index + ': ' + val);
	});
	
### process.execArgv

与 process.argv 类似，不过是用于返回 node特殊(node-specific) 的命令行选项（参数）。另外所有文件名之后的参数都会被忽视：

	//试着执行 $node --harmony execArgv a b --version
	console.log(process.execArgv); //[ '--harmony' ]
	process.execArgv.forEach(function(val, index, array) {
	    console.log(index + ': ' + val);
	});
	
### process.abort()

触发node的abort事件，退出当前进程：

	process.abort();
	console.log('在输出这句话之前就退出了');
	
### process.execPath

返回当前node.js进程的启动命令路径，也就是node.js安装目录下的node命令路径，是一个绝对路径:

	console.log(process.execPath); //url: /usr/local/bin/node
	
### process.cwd

返回当前进程的工作目录：

	console.log('当前目录：' + process.cwd()); //当前目录：E:\github\nodeAPI\process

### process.chdir(directory)

改变进程的当前进程的工作目录（该目录必须已存在），若操作失败则抛出异常：

	var path = require('path');
	
	console.log('当前目录：' + process.cwd()); //当前目录：E:\github\nodeAPI\process
	try {
	    process.chdir(path.resolve('.','tmp'));
	    console.log('新目录：' + process.cwd()); //新目录：E:\github\nodeAPI\process\tmp
	}
	catch (err) {
	    console.log('chdir: ' + err);
	}
	
>注意：node.js的require()不受这个限制,他是以当前文件的。在
>执行child_process.exec()方法时需要考虑这一点。

### process.env

返回当前linux系统的信息，我可以输入一下代码来看系统信息:

	console.log(JSON.stringify(process.env));
	
>返回的是json格式，通过函数将它转换成我们容易看的模式

还可以：

	console.log(process.env);
	console.log('username: ' + process.env.USERNAME);
	console.log('PATH: ' + process.env.PATH);

### process.exit([code])

终止当前进程并返回给定的 code。如果省略了 code，退出是会默认返回成功的状态码('success' code) 也就是 0：

	process.exit(1); //node的shell将捕获到值为1的返回码
	
### process.exitCode

可以自定义退出进程时node shell捕获到的状态码（必须是正常结束进程或者使用process.exit()指令退出）

	process.exitCode = 4;
	process.exit();
	
### process.version

一个暴露编译时存储版本信息的内置变量 NODE_VERSION 的属性:

	console.log('版本: ' + process.version); //版本: v0.12.7
	
### process.versions

一个暴露存储 node 以及其依赖包 版本信息的属性：

	console.log(process.versions);
	//{ http_parser: '2.3',
	//    node: '0.12.7',
	//    v8: '3.28.71.19',
	//    uv: '1.6.1',
	//    zlib: '1.2.8',
	//    modules: '14',
	//    openssl: '1.0.1p' }
	
### process.config

一个包含用来编译当前 node.exe 的配置选项的对象:
	
	console.log(process.config);
	
### process.pid

获得当前进程的pid：

	console.log(process.pid);

### process.kill(pid, [signal])

结束对应某pid的进程并发送一个信号（若没定义信号值则默认为'SIGTERM'）：

	process.kill(process.pid, 'SIGTERM');

### process.title

获取或设置当前进程的标题名称：

	console.log(process.title);
	// 管理员: E:\Program Files\WebStorm 9.0.1\lib\libpty\win\x86\winpty-agent.exe - node  title
	process.title = 'new title!!!';
	console.log(process.title); //new title!!!

### process.arch

返回当前CPU的架构（'arm'、'ia32' 或者 'x64'）：

	console.log(process.arch); //x64

### process.platform

返回当前平台类型（'darwin', 'freebsd', 'linux', 'sunos' 或者 'win32'）：

	console.log(process.platform); //win32

### process.memoryUsage()

返回一个对象，它描述了Node进程的内存使用情况，其单位是bytes：

	console.log(process.memoryUsage()); //{ rss: 16875520, heapTotal: 9751808, heapUsed: 3997040 }

### process.nextTick(callback)

算是 process 对象最重要的一个属性方法了，表示在事件循环（EventLoop）的下一次循环中调用 callback 回调函数。

要注意的是它总会在I/O操作（比如查询数据）之前先执行。

示例：

	console.log('开始');
	process.nextTick(function() {
	  console.log('nextTick 回调');
	});
	setTimeout(function(){
	  console.log('新的EventLoop!')
	  }, 2000);
	console.log('当前EventLoop');
	// 输出:
	// 当前EventLoop
	// nextTick 回调
	// 新的EventLoop!

>异步执行callback函数，注意，这比 "setTimeout(fn, 0) " 要
>高效很多。当有一些比较耗时的操作可以用在
>process.nextTick(callback) 中，这样可以不阻塞整个函数执行。

### process.umask([mask])

设置或者读取进程的文件模式的创建掩码（屏蔽字）。子进程从父进程中继承这个掩码。如果设定了参数 mask 那么返回旧的掩码，否则返回当前的掩码：
	
	var newmask = 77,
	oldmask = process.umask(newmask);
	console.log('原掩码: ' + oldmask.toString(8) + '\n' +
	'新掩码: ' + newmask.toString(8));
	//原掩码: 0
	//新掩码: 115
	
	设置进程的user mask值，什么是umask值呢？
	在linux系统有一个系统命令：$ umask，主要作用是修改系统默认的创建文件和文件夹的权限。
	注意这句话：Returns the old mask if mask argument is given, otherwise returns the current mask.
	当对process.umask()方法传递参数时，则返回旧的umask值，否则返回当前的umask值。
	这里如果我们设置：
	var oldmask, newmask = 0644;
	oldmask = process.umask(newmask);
	console.log('Changed umask from: ' + oldmask.toString(8) + ' to ' + newmask.toString(8));
	输出022和0644，当umask = 022时，新建的目录权限是755，新建文件的权限是 644。具体可以参考linux umask手册。这里只是修改由本node.js进程创建的文件和目录的权限。

### process.uptime()

返回 Node 程序已运行的秒数：

	console.log('初始时间是：' + process.uptime());
	
	var arr = new Array(10000000);
	var s = arr.join(',');
	console.log('处理数组后的时间是：' + process.uptime());
	
	//初始时间是：0.436
	//处理数组后的时间是：1.068
	
### process.hrtime()

返回当前的高分辨时间，形式为 [秒，纳秒] 的元组数组。它是相对于在过去的任意时间。该值与日期无关，因此不受时钟漂移的影响。主要用途是可以通过精确的时间间隔，来衡量程序的性能。示例：

	var t1 = process.hrtime();
	var arr = new Array(40000000),
	  s = arr.join(',');
	var t2 = process.hrtime();
	console.log('处理数组共花费了%d秒，详细为%d纳秒', (t2[0] - t1[0]), (t2[1] - t1[1]));
	//处理数组共花费了2秒，详细为227005133纳秒

### process.reallyExit(status)

真实推出本进程，不触发‘exit’事件
	
### process._kill(pid,sig)

用于给指定pid的进程发送指定信号（类似linux下的kill命令）

	var pid=process.pid;
	process._kill(pid,9);

### process.binding(name)

这个方法用于返回指定名称的内置模块。例如下面的代码将打印node_net模块所有的可以调用的方法或变量。

	var binding=process.binding('net');
	console.dir(binding);

### process.end

结束进程的事件，需要用process.emit出发：

	process.emit('end','parm1','parm2');

