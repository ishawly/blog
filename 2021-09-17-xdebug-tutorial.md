---
title: Xdebug 环境配置
date: 2021-09-17 20:41:00
description: Xdebug 环境配置及简单说明
categories:
- 开发工具
tags:
- xdebug
- php
---

# 安装 Xdebug

> 官方地址：https://2.xdebug.org/docs/install



- 通过 PECL 安装

- 在 Windows 上安装

- 编译安装

  

这里只介绍编译安装，官方提供一个[安装向导工具 https://xdebug.org/wizard](https://xdebug.org/wizard)，通过该工具用户可以确定下载哪个版本及如何配置使其运行起来。

1. 复制整个 phpinfo() 的输出的页面或者`php -i`的输出到文本框中
2. 分析处理
3. 根据分析结果配置
4. 配置完成后执行`php -v`，若有 Xdebug 提示则表示安装完成



# 配置 Xdebug

## php.ini 配置

找到 `zend_extension="[/xx/xxx/]xdebug.so"`，在附近配置相关参数。

### Xdebug 2 配置

详细请参考：https://2.xdebug.org/docs/remote

```
; 调试通信建立过程中
; 包含 Xdebug 的 PHP 扮演客户端 client 的角色，
; IDE 扮演服务器 server 的角色
; 与通常B/S模式的认识相反，这里很容易弄混因此以下配置中 remote 实际表示 IDE
;
; 启用Xdebug, boolean值，默认值false
xdebug.remote_enable=true
;
; 调试客户端即IDE所在的位置，可以是域名、IP、'unix:///path/to/sock' for a Unix domain socket
; 默认值 localhost
; 当 xdebug.remote_connect_back 启用时将会被忽略
;xdebug.remote_host=localhost
; 
; 调试客户端即IDE使用的端口
; 默认值 9000
xdebug.remote_port=9003
;
; xdebug 从请求中获取IP地址
; boolean值 默认值false
xdebug.remote_connect_back=true
xdebug.idekey = "PHPSTORM"
;xdebug.remote_handler=dbgp
; 
; boolean值 默认值false
; 启用则所有请求都会触发调试会话，并忽略浏览器等客户端是否配置
;xdebug.remote_autostart=true
```



### Xdebug 3 配置

这里介绍PHP/Xdebug和IDE在同一个主机的情况，详细请参考：https://xdebug.org/docs/step_debug

```
; 启用debug模式 https://xdebug.org/docs/all_settings#mode
;xdebug.mode=debug
;
; https://xdebug.org/docs/step_debug#start_with_request
; yes:表示所有请求都触发调试会话，浏览器不用配置插件，CLI不用配置变量
; no: 请求开始不会触发
; trigger: 
;   1. 通过 $_ENV, $_GET, $_POST, $_COOKIE 中的变量名XDEBUG_TRIGGER触发
;   2. 兼容老版本的XDEBUG_SESSION
;   3. XDEBUG_SESSION_START管理调试会话依然有效        
;   4. 配置了 xdebug.trigger_value 则相应值才能触发，为空则表示所有
;xdebug.start_with_request=yes

;xdebug.client_host=127.0.0.1
;xdebug.client_port=9003
;xdebug.log=/tmp/xdebug.log
```



## 浏览器配置

<h3>Chrome 配置</h3>

扩展安装地址：https://chrome.google.com/webstore/detail/xdebug-helper/eadndfjplgieldjbigjakmdgkmoaaaoc



配置IDE Key

![chrome-xdebug-helper-config.png](/images/chrome-xdebug-helper-config.png)



启用：在Chrome浏览器工具打开该插件的调试模式。

![chrome-xdebug-helper-enable](/images/chrome-xdebug-helper-enable.png)





Firefox、Safari配置见：https://xdebug.org/docs/step_debug 中"Browser Extension Initiation"小节。



### Postman 配置

单独启用调试会话可以在get、post请求参数中添加`XDEBUG_SESSION=session_name`即可触发，若`xdebug.session_value`未配置任何值则`session_name`可为任何值。  

若需所有请求都启用调试会话，则可在cookie中设置`XDEBUG_SESSION=session_name`。

![postman-xdebug-config](/images/postman-xdebug-config.png)



## IDE 配置

### Phpstorm 配置

![phpstorm-xdebug-config](/images/phpstorm-xdebug-config.png)



开启调试

![phpstorm-enable-xdebug](/images/phpstorm-enable-xdebug.png)

更多请参考：

> Zero-configuration debugging: https://www.jetbrains.com/help/phpstorm/2021.1/zero-configuration-debugging.html
>
> Configure Xdebug: https://www.jetbrains.com/help/phpstorm/2021.1/configuring-xdebug.html

