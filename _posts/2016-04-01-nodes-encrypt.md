---
layout: post
title: "express加密模块"
date: 2016-04-01 15:14:54
categories: express
---

    /**
    * @用户相关的业务逻辑操作
    *
    * */
    var usersMod = require('../module/usersModule');
    var useEncrypt = require('../module/encrypt');
    /**
     * @用户登录验证
     * @param req 用户请求数据
     * @param res 相应用户数据
     * @next 调用其他中间件
     * */
    exports.login = function(req, res, next){
        var obj = req.body;
        
        
        if(username == obj.username && password == obj.password){
            res.json({res:'1'});
        } else {
            res.json({res:'0'});
        }
    }
    
    /**
     * @用户注册
     * @param req 用户请求数据
     * @param res 相应用户数据
     * @next 调用其他中间件
     * */
    exports.reg = function(req, res, next){
        var obj = req.body;
        console.log(obj);
        console.log(req.session.code);
        //判断用户的验证码
        if (obj.code != req.session.code) {
            res.json({res:'验证码不对!'});
        } else { //执行数据库操作,并返回成功
            //对密码加密
            var pass = useEncrypt.getSha1(obj.pass);
            //封装添加的数据
            var data = {
                _phone: obj.phone,
                _password: pass
            }
           var addRes = usersMod.reg(data, function(err, id){
               if (err) {
                   res.json({res:'注册失败'});
               } else {
                   //注册成功保存session信息
                   req.session.user = {phone:obj.phone};
                   res.json({res:'注册成功'});
               }
           });
        }
    }
