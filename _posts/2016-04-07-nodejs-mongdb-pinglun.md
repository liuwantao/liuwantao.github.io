---

layout: post
title:  "评论盖楼两种方式"
date:   2016-04-07 00:00:00
categories: nodejs

---

盖楼两种实现方式
-------------------

先分析一下思路:

先说前台的处理,我们需要后台返回一个多维的数组对象,这个数组对象包含:

对这个商品的所有评论,在每一条的评论中,应该有一个字段,来存放对这条评论的所有回复的一个数组,

大概是这个样子的:

    [
      obj1{    //对商品的评价
        username:'user',
        gid:'goodid',
        review:[           //对每一条评论的回复
          robj1,
          robj2,
          ...
        ],
        addtime:'time'
      },
      obj2{
        username:'user',
        gid:'goodid',
        review:[
          robj1,
          robj2,
          ...
        ],
        addtime:'time'
      },
      ...
    ]
    
这样我们在前台遍历盖楼的时候,只需要判断review是否为空,来决定是否输出子回复的楼层了

nodejs后台实现方式
-----------------------

### 方式1:在插入数据的时候进行数据处理

这种方式是我们在数据写入数据库的时候就把数据处理好,这样查询到数据后,直接返回前台就可以用了

>特点:插入费点事,查询简单

这种方式的核心地方在于表结构的设计和数据的插入:

    表结构:
    remark 表
     {
        _gid : {type:String},    //商品ID,对哪个商品的评价
        _contents : [{           //针对每一个评价的回复,一个数组集合
            _userPhone : {type:String},   //哪个用户评论的
            _content : {type:String},    //评论内容
            _addTime : {type:Date, default:Date.now()}  //评论时间
        }]
      }
        插入语句:
        /**
         * @评论插入语句
         * @json 数据对象,拼装的用户输入的数据
         * @callBack 回调函数
         */
        var db = mongoose.createConnection('localhost', 'dbname'); 
        exports.disAdd = function (json, callBack) {
            var schema = new mongoose.Schema({   //实例化schema对象
                上面的表结构
                ...
            });
            var goodsModel = db.model('tableName',schema);  //model对象
            goodsModel.findById(json.id,function(err,result){//注意这里的result,它是查询到得model的一个子对象,是可以调用自己的方法的
            if (err) {                                      //不要单单理解成一条数据
                    callBack('error');   //向调用这个函数的中间价传递信息
                } else {
                    result._contents.push({  //每当有新数据,就向表中的_contens数组集合压入数据
                        _userPhone : json.phone,
                        _content : json.con
                    });
                    result.save();           // 保存我们的修改
                    callBack('success');
                }
            });
        }
        
        查询语句:
        /**
         * @获取某个商品的品论
         * @gid 商品id
         * @callBack 回调函数
         */
        exports.getDiscuss = function (gid, callBack) {
            var schema = new mongoose.Schema({   //实例化schema对象
                上面的表结构
                ...
            });
            var goodsModel = db.model('tableName',schema);  //model对象
            goodsModel.find({_gid:gid}, function (err, goods) {   // 查询数据
                if (err) {
                    callBack('error');
                } else {
                    callBack(goods);  //把查到数据传给调用该函数的中间件(当然,如果你的路由没有分开,也可以直接在这里res处理)
                }
            });
        }
        
### 方式2:在查询数据的时候进行数据处理

这种方式是我们在查出来数据的时候进行进行数据处理

>特点:插入简单,数据查询费事

实现的核心在于查询数据的处理:

    表结构:
    remark 表
    {
      goodsId:{type: String, default: ''},   //对哪个商品评论
      userName:String,  //哪个用户评论的
      toremark:{type: String, default: ''},   //对哪个评论进行的回复
      content:String,   //评论的内容
      addtime:{type: Date, default: Date.now()},   //评论时间
      review:{type: String, default: ''}   //  用来存该条商品评论的所有回复
    }
    插入语句:
    接收到数据后的处理:
    如果是对商品的评论,这样处理数据:
    data = {   //这是对商品的评价,是没有回复ID的,即不写toremark字段
       goodsId:req.body.goodid,  //商品ID
       userName:req.session.userName,  //用户名
       content:req.body.content   //内容
    }
    data = {  //这是对评价的回复,是没有商品ID的,即不写goodsId字段
      userName:req.session.userName,  //用户名
      toremark:req.body.toremark,    //对哪个评价进行的回复
      content:req.body.content  //内容
    }
    /**
     * @插入评论的方法
     * @param data json 要插入的数据
     * @param callback function 回调函数返回执行结果
     * */
    exports.saveReview = function (data, callback) {
    	var schema = new mongoose.Schema({   //实例化schema对象
            上面的表结构
            ...
        });
      var goodsModel = db.model('tableName',schema);  //model对象
    	goodsModel.create(data, function(err){  //执行添加
    		callback(err);   //回调信息
    	});
    }
    查询语句:
    /**
     * @查询评价信息
     * @param where json 查询条件
     * @param callback function 回调函数返回执行结果
     * */
    exports.getReview = function (where, callback) {
    	var schema = new mongoose.Schema({   //实例化schema对象
            上面的表结构
            ...
        });
      var goodsModel = db.model('tableName',schema);  //model对象  
    	goodsModel.find(where, function(err, data){    	//查询有关商品的所有回复
    		var i = 0;  初始化变量
    		getCallbakc(i, model,data,callback);  	//因为要循环查询每一个评价的回复,所有通过特殊的方式处理回调函数的异步
    	});
    }
    //这个函数处理回调函数的异步,很实用哦
      function getCallbakc (i, model, obj, callback) {
    	if (i < obj.length) {  //判断当前商品的评价数量,用来判断是否终止递归
    		if (obj[i].toremark != '') {    //判断评价是否存在回复,存在保存数据
    			var id =(obj[i]._id).toString();   //id转为字符串类型(不转的话我记得会报错,你可以试试看)
    			model.find({toremark: id}, function (err, doc) {
    				var test = obj[i].toObject();  //将当前评论对象化,便于下面写入数据
    				obj[i] = {
    					_id:id,
    					goodsId:test.goodsId,
    					userName:test.userName,
    					content:test.content,
    					addtime:test.addtime,
    					review:doc  //把查到的回复数组集合保存在字段里面
    				}
    				getCallbakc(i + 1, model, obj, callback);  //查完继续调用函数去查下一条的
    			});
    		} else {
    			getCallbakc(i + 1, model, obj, callback);  //这个一定要写,不然碰到一条没有回复,怎么能继续查下一条呢?
    		}
    	} else {
    		callback(obj);   //直到查完所有数据,把数据通过回调函数传递回去
    	}
    }
  
前台遍历输出的两种方式
-----------------------
    
### 方式1:padding内间距法

大概思路就是一种通过内嵌div,每个div都设定固定的padding,靠padding来撑起来楼层的效果:
  
css:

    <style>
    .commentsinfo{  padding: 2px;  border: 1px solid #ccc;  background: #ffe;  }
    .comments{  border-top: 1px dotted #ccc;  border-bottom: 1px dotted #ccc;  padding-bottom: 30px;  padding-top: 30px;  }
    .theauthor{  color: #1e50a2;  }
    .commentsnumber, .goodscsheia{  float: right;  }
    </style>
  
html:

    <article id="nest" style="padding: 5px;">
        <!--<div class="comments">
            <p class="theauthor">我是用户名</p>
            <div class="commentsinfo">
                <div class="commentsinfo">
                    <div class="commentsinfo">
                        <p class="theauthor">我是用户名</p> <span class="commentsnumber">1</span>
                        <p>我是内容</p>
                    </div>
                    <p class="theauthor">我是用户名</p> <span class="commentsnumber">2</span>
                    <p>我是内容</p>
                </div>
                <p class="theauthor">我是用户名</p> <span class="commentsnumber">3</span>
                <p>我是内容</p>
            </div>
            <p>我是内容</p>
            <div class="pinglun">
                <a href="#" id="goodscshei" data-rel="popup" data-position-to="window" data-transition="fade">评论</a>
            </div>
        </div>-->
    </article>
     ps: 注释的部分不要,用ajax循环替换
  
js:

        function getDiscuss() {
            $.ajax({
                url:'http://192.168.160.14:3000/admin/getDiscuss',
                data:{gid:goodsId},
                dataType:'json',
                type:'post',
                success:function (data) {
                    var c = data;
                    //$('#two')
                    var a ='<div class="commentsinfo">';
                    var b ='   <p class="theauthor">';
                    var h ='</p> <span class="commentsnumber">';
                    var d ='</span><p>';
                    var e ='</p></div>';
                    var g = '<div class="comments">';
                    var str = '';
                    for(var i=0;i<c.length;i++){
                        var xstr = '';
                        var x =c[i]._contents;
                        var w ='</p><div class="pinglun "><a href="#" class="goodscsheia" data-rel="popup" data-position-to="window" data-transition="fade" data-cidd="'+c[i]._id+'">评论</a></div></div></div>';
                        var ding = '';
                        var shou = '';
                        var shouxia = '';
                        var num = x.length;
                        for(var f=0;f<num;f++){
                            var xxstr ='';
                            console.log(f);
                            if(f == num-1){
                                console.log(x[f])
                                var shou =  g +b +x[f]._userPhone+'</p>';
                                var shouxia = '<p class="theauthor">'+x[f]._content+'</p>';
                            }else{
                                ding +=a;
                                xxstr = b +x[f]._userPhone+h+(f+1)+d+x[f]._content+e;
                            }
                            xstr += xxstr;
                        }
                        str += (shou+ding+xstr+shouxia+w);
                    }
                    $('#nest').html(str);
                    // -----------------回复
                    $('.goodscsheia').click(function () {
                        var content = $('#sendM').val();
                        var cid = $(this).data('cidd');
                        $.ajax({
                            url:'http://192.168.160.14:3000/admin/disAdd',
                            data:{id:cid, con:content},
                            type:'post',
                            dataType:'json',
                            success:function (data) {
                                if (data.result == 'yes') {
                                    $('#sendM').val(' ');
                                    getDiscuss();
                                } else if (data.result == 'noLogin') {
                                    $('#hiddenLogin').click();
                                }
                            }
                        });
                    });
                    // --------------回复结束
                }
            });
        }
 	
    
### 方法2:百分比定位居中法


思路就是给父标签一个相对定位,子标签绝对定位,不断循环减少left和top的值就可以啦

html:

    <section data-role="main" class="ui-content" id="shopReviewDiv">
     </section>
    
css:

    #shopReviewDiv >div {width: 100%;min-height:60px;border-radius: 10px;border:1px solid #ccc;margin-top: 10px;}
    #shopReviewDiv >div img {width: 15%;height:50px;display: inline-block;margin-top:5px;border-radius: 25px;float: left;}
    #shopReviewDiv >div >div {width: 85%;display: inline-block;border-radius: 10%;float: left;}
    #shopReviewDiv >div >div >div{position: absolute;border:1px solid black;}
    #shopReviewDiv >div >div >div >div > a{float:right; text-decoration: none;color:#aaa;font-size: 10px;}
    
js:

    //打开商品的评论
    function toMoreReview(goodid){
      $("#shopReviewDiv").html(' ');//每次加载之前清空原来的内容
      //获取到商品ID之后去后台查询相关评论
      $.ajax({
          url:"http://192.168.160.9:3000/goods/getReview",
          data:{goodid:goodid},
          type:'post',
          dataType:'json',
          success:function(data) {
              for (var i=0;i<data.length;i++) {
                  var str = '';  //用来存放拼接的字符串
                  var l = data[i].review.length;  //子平论的长度
                  var zi = data[i].review;  //子平论的数据
                  //先输出对商品的回复
                  //$("#shopReviewDiv").append("<div><img src=''><div style='position: relative;'></div></div>");
                  if( l != 0 ) {  //判断哪一条数据有子评论,有得话就先把子字评论拼接起来
                      str = "<div style='height:"+(l*50+65)+"px;'><img src=''><div style='position:relative;'>";
                      str += "<div style='left:" + l + "%;top:" + (3*l+6) + "px;width:" + (100-2*l-4) + "%;min-height:50px;'><div style='background:#ffe;margin-top:0px;'><b style='color: #00d6b2'>User: "+data[i].userName+"</b><em style='float:right;'>1&nbsp;</em><br/><span style='color: #bd362f'> "+data[i].content+"</span><a href='#' onclick=toReviewOther('"+data[i]._id+"')>Review</a></div></div>";
                      for (var j = 0; j < l; j++) {  //内层循环带 有子平论的回复
                          str += "<div style='left:" + (l-1-j) + "%;top:" + (3*l +3 - 3*j) + "px;width:" + (98-2*l + 2 * j) + "%;min-height:" + (100 + 50 * j) + "px;'><div style='background:#ffe;margin-top:" + (50 * j+60) + "px;'><b  style='color: #00d6b2'>User: "+zi[j].userName+"</b><em style='float:right;'>"+(j+2)+"&nbsp;</em><br/><span style='color:black'> "+zi[j].content+"</span><a href='#' onclick=toReviewOther('"+data[i]._id+"')>Review</a></div></div>";
                      }
                     mar = 50*l+10;
                  } else {  //如果只有单个的对商品的评价,没有字评价,就用这个样式
                      str = "<div style='margin-top:10px'><img src=''><div style='position:relative;'>";
                      str += "<div style='border:0px;width:100%;min-height:50px;'><div style='background:#ffe;margin-top:10px;'><b style='color: #00d6b2'>User: "+data[i].userName+"</b><br/><span style='color: #bd362f'> "+data[i].content+"</span><a href='#' onclick=toReviewOther('"+data[i]._id+"')>Review</a></div></div>";
                  }
                  str += "</div></div>";
                  $("#shopReviewDiv").append(str);
              }
          },
          error:function(err){
      
              } 
          });
        }
  
---

有不对的地方或者有更好的方法请在下方留言
