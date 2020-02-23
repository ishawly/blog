---
title: 一个备忘录小程序从0到1
date: 2018-09-22 11:17:40
tags:
---

# 前言
&emsp;&emsp;最近突然想写个备忘录来记录脑海里突然闪现的思维火花，思考一番觉得做成小程序最佳。顺便实战下laravel开发RESTful API。本文描述了大略实现思路过程和相关参考文档。

<!-- more -->

# 功能定义
&emsp;&emsp;从备忘录信息中引申，一条信息主要是其信息主体，附属包括创建时间，创建人等。围绕这条信息，需要至少用户登录，录入，历史列表浏览三个界面即可满足。数据库中创建用户和备忘信息两张表与之对应。  
&emsp;&emsp;线上环境配置：centos7，nginx1.4.3，php7.2，mariadb5.5.6，composer1.7，laravel5.5  

基础储备：  
1. [轻松学会Laravel-基础篇](https://www.imooc.com/learn/697)
2. [composer基础用法](https://docs.phpcomposer.com/01-basic-usage.html)

# 服务端实现
## 定义api路由
&emsp;&emsp;目前RESTful API是一套比较成熟的互联网应用程序的API[设计理论](http://www.ruanyifeng.com/blog/2014/05/restful_api.html)，恰好laravel对其也有[良好支持](https://laravel-china.org/docs/laravel/5.5/controllers/1296#resource-controllers)。定义备忘录信息（后以想法或idea代替）api只需一条命令即可`php artisan make:controller IdeaController --resource`, 完成后在路由文件/routes/api.php中加入一条路由`Route::resource('ideas', 'IdeaController');`同理添加用户路由。资源控制器操作处理如下表所示。

| 动作 | URI | 行为 | 路由名称 |
| --- | --- | --- | --- |
| GET       | `/ideas`              | index   | ideas.index   |
| GET       | `/ideas/create`       | create  | ideas.create  |
| POST      | `/ideas`              | store   | ideas.store   |
| GET       | `/ideas/{photo}`      | show    | ideas.show    |
| GET       | `/ideas/{photo}/edit` | edit    | ideas.edit    |
| PUT/PATCH | `/ideas/{photo}`      | update  | ideas.update  |
| DELETE    | `/ideas/{photo}`      | destroy | ideas.destroy |

*疑问1：路由如何访问？*  
*答：命令行中确保php命令已经引入，进入Laravel根目录，执行`php artisan route:list`生成的列中URI即是*

## 功能开发
&emsp;&emsp;API开发通常会涉及到用户身份验证，有些使用token验证身份，有些使用OAuth2.0。网上有种说发，给别人使用的使用OAuth，自己使用的用token。而且Laravel自带token验证。修改下资源路由加入api中间件和前缀并限制使用的方法。
```
Route::middleware('auth:api')->prefix('v1')->group(function(){
    Route::resource('ideas', 'IdeaController', [
        'only'  => ["index", "store", "show", "destroy"]
    ]);
});
```
&emsp;&emsp;想法储存使用IdeaController中的store方法
```
public function store(Request $request)
    {
        // 获取想法数据
        $content = $request->post('idea');
        // 数据安全检测

        // 入库成功，返回处理结果（格式根据项目或团队决定）
        $rtn = [
            'status'=> 0,
            'msg'   => '操作成功',
            'data'  => null
        ]
        // 一般返回json格式
        return response()->json($rtn);
    }
```
参考：  
[API Token Authentication](https://segmentfault.com/a/1190000006215513)  
[Laravel 的用户认证系统](https://laravel-china.org/docs/laravel/5.5/authentication/1308)  
[Laravel API Token 体验](https://laravel-china.org/articles/6381/laravel-api-tutorial-practice)

# 小程序实现
&emsp;&emsp;个人开发者也可以属于自己的小程序，具有一定前端知识（不需要很多）一定要尝试一下。这或许是现目前较好的混合App一种实现方式。参照[官方视觉规范](https://developers.weixin.qq.com/miniprogram/design/#%E8%A7%86%E8%A7%89%E8%A7%84%E8%8C%83)在不能专门设计的情况下也可以画出看的过去的界面。实现三个页面过程大同小异：组装请求数据，接口调用，渲染页面。

* 用户登录授权页：获取用户用户信息通过openid识别用户并返回用户的token，其余页面通过token验证身份。该页作为小程序入口页，其余页面通过底部导航切换。
* 想法录入页：提供录入接口。
* 想法列表页：查看历史记录。

## 开发难点
&emsp;&emsp;由于微信接口调整不能直接在小程序中获取用户的openid。应对方法，小程序调用`wx.login(Object object)`获取code，然后在服务器后台向微信服务器调用 [code2Session](https://developers.weixin.qq.com/miniprogram/dev/api/open-api/login/code2Session.html)，使用 code 换取 openid 和 session_key 等信息。
&emsp;&emsp;备忘录不在个人开发者的适用范围内。目前正处理这个难点。

---
*更新记录*
* 2018-10-03 内容重构
