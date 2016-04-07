---
layout: post
title: "mongodb的一些细节"
date: 2016-4-6 12:44:12
categories: sql
---

## 关于mongodb查询出来的数据类型 ##

### id类型 ###

mongodb 的id是一个特殊的对象:

>主键，一种特殊而且非常重要的类型，每个Schema都会默认配置这个属性，属性名为_id，除非自己定义，方可覆盖主键，
>一种特殊而且非常重要的类型，每个Schema都会默认配置这个属性，属性名为_id，除非自己定义，方可覆盖

    var mongoose = require('mongoose');
    var ObjectId = mongoose.Schema.Types.ObjectId;
    var StudentSchema = new Schema({}); //默认会有_id:ObjectId
    var TeacherSchema = new Schema({id:ObjectId});//只有id:ObjectId

如果想用id直接做关联查询,我们可以这样做:
    
    var ExampleSchema = new Schema({
      _pk:{type:Schema.Types.ObjectId,ref:'tableName'}
    });
    
>这样就关联了一张表,在查询的时候,会自动把这个ID替换成响应表中对应id的数据

如果只是想把id做为字符串和其他表关联,这样做:

    因为id是一个特殊的对象类型,所以我们在使用的时候要进行响应的转换:
    
    var id =(obj._id).toString();  //把id转换成字符串
    
>这是为了解决如果你的表中,pid定义的是字符串类型,在取得另一张表的id直接往这张表里面存的时候,
>mongodb会自动将你插入的ID转换成表中定义好得String类型,这样以后你再拿取出来的对象类型的id去查
>数据的时候是查询不到的,需要进行字符串转换

### findOne类型 ###

findOne查询到的是一条数据,是字符串类型的,比如:

    {name:'aaa',age:'55'}  //查出来的数据是字符串类型,不能当做对象用
  
我们如果想直接调用其中的属性,要先把查到的字符串转换成json对象:

    var json = data.toObject(); //这样我们就可以当做json对象使用了
  
### find 类型 ###

find查询到的数据是一个数组(因为查询多条呀),是这样的:

    arr = [{name:'aaa',age:'55'},{name:'aaa',age:'55'},{name:'aaa',age:'55'}]
    
我们可以直接用for 循环取出其中的数据

    for (var i=0;i<arr.length;i++) {
        console.log(arr[i]);
    }
    
>因为是一个数组,所以它里面只是存得json形式的字符串罢了,如果想用它的数据和属性,这样做:

    for (var i=0;i<arr.length;i++) {
        var obj = arr[i].toObject();  //这样obj就是一个一个json对象了,我们就可以为所欲为了
        console.log(obj);
    }
    
## 静态方法 ##

我们在通过Schema实例化一个Model之前,可以对即将生成的Model封装自定义的静态方法:
    
    //一个新的Schema对象
    var Schema = new Schema(
      {
        name:String,
        age:Number
      }
    );
    
    //定义静态方法
    Schema.static.myFunctionName = function(name, callback){
        this.find({name:name},callback); //一定要有回调函数,不然异步的查询,数据你就获取不到了
    }
    
    //使用静态方法
    var myModel = mongoose.model('tableName',Schema);
    myModel.myFunctionName('tom',function(err, data){  //找到所有名字为tom的人
        console.log(data);  //通过自己定义的方法拿到数据了
    });
    
### 验证器 ###
    
在定义表结构的时候,我们可以添加一些内置的函数来验证数据:

* required 非空验证
* min/max 范围验证
* enmu/match 枚举验证/匹配验证
* validate 自定义验证


    var PersonSchema = new Schema({
      name:{
        type:'String',
        required:true //姓名非空
      },
      age:{
        type:'Nunmer',
        min:18,       //年龄最小18
        max:120     //年龄最大120
      },
      city:{
        type:'String',
        enum:['北京','上海']  //只能是北京、上海人
      },
      other:{
        type:'String',
        validate:[validator,err]  //validator是一个验证函数，err是验证失败的错误信息
      }
    });
        

如果验证失败,返回err:

    err.errors                  //错误集合（对象）
    err.errors.color            //错误属性(Schema的color属性)
    err.errors.color.message    //错误属性信息
    err.errors.path            //错误属性路径
    err.errors.type            //错误类型
    err.name                   //错误名称
    err.message               //错误消息
          
        
---

