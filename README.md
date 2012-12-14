node-taobao
===========

taobao api的nodejs sdk

构建中，勿用于生产环境

###支持的api（增加中）

this.taobaoke=this._taobaoke();
    this.products=this._products();
    this.user=this._user();
    this.users=this._users();
    this.item=this._item();

###调用示例（oauth认证和测试一个淘客接口）

```
var Taobao = require("../lib/index.js");
var underscore=require("underscore");
var path=require("path")
var config = {
    app_key:"21261604",
    app_secret:"31ef78b08e193a496c6647bedf0dfa3a",
    redirect_uri:"http://127.0.0.1:8080/sina_auth_cb"
}
var formatjson = require('formatjson');
var app_auth = {
    auth:function (req, res) {
        var api = new Taobao(config);
        var auth_url = api.oauth.authorize();
        res.redirect(auth_url);
        res.end();
    },
    sina_auth_cb:function (req, res) {
        var code = req.query.code;
        var api = new Taobao(config);
        api.oauth.accesstoken(code, function (error,data) {
            res.cookie("token", data.access_token);
            res.redirect('oauth');
            res.end();
        })

    }
}
//import some libs 
var express = require('express');
var cons = require('consolidate');
//init express app
var app = express();
app.use(express.logger({
    format:':method :url :status'
}));
//设置文件上传临时文件夹
app.use(express.bodyParser({
    uploadDir:'./uploads'
}));
app.use(express.cookieParser());
app.use(express.session({
    secret:'yutou'
}));
app.use(app.router);
app.use(express.errorHandler({
    dumpExceptions:true,
    showStack:true
}));
app.error = function (err, req, res) {
    console.log("500:" + err + " file:" + req.url)
    res.render('500');
}
//设置模板引擎为mustache，这里使用了consolidate库
app.engine("html", cons.mustache);
//设置模板路径
app.set('views', __dirname + '/views');
app.set('view engine', 'html');
app.set('view options', {
    layout:false
})
app.listen("8080")
//获取authorize url
app.get("/auth", app_auth.auth)
//获取accesstoken ,存储，并设置userid到cookie
app.get("/sina_auth_cb", app_auth.sina_auth_cb)
//中间页面，提醒用户认证成功
app.get('/oauth', function (req, res) {
    var config = {
        app_key:"21261604",
        app_secret:"31ef78b08e193a496c6647bedf0dfa3a",
        redirect_uri:"http://127.0.0.1:8080/sina_auth_cb",
        access_token:req.cookies.token
    }

    var api=new Taobao(config)
    api.taobaoke['items.detail.get']({
        nick:"xinyu1987326",
        num_iids:"13210257235",
        fields:"click_url,title,nick,shop_click_url,item_img"
    },function(error,data){
        console.log(formatjson(data))
    })
    res.render("oauth.html")
});

```