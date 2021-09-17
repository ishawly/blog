---
title: 使用 Laradock 搭建PHP开发环境
date: 2021-08-25 16:26:00
description: 搭建Docker的PHP开发环境及遇到的问题处理
categories:
- 环境搭建
tags:
- docker
- laradock
- php
- nginx
- php-fpm
---



# 快速上手



前提已配置好 Docker 环境。



## 1、克隆 Laradock 项目代码

``` 
git clone https://github.com/Laradock/laradock.git
```



## 2、创建配置文件

```
cd laradock
cp .env-example .env
```



## 3、运行容器

```
docker-compose up -d nginx mysql redis beanstalkd
```



## 4、修改配置

```
DB_HOST=mysql
REDIS_HOST=redis
QUEUE_HOST=beanstalkd
```



## 5、配置目录隐射



- 在`laradock`父级目录下创建于`laradock`同级的`wwwroot`目录，并在该目录下运行`laravel new blog`命令创建一个新的`Laravel`应用。
- 编辑`laradock`中`.env`的`APP_CODE_PATH_HOST=../wwwroot/`，这就将`wwwroot`与`docker`的`/var/www`目录建立了软连接。
- 修改`laradock/nginx/sites/defaults.conf`中的指令`root /var/www/blog/public`。
- 重启 Nginx `docker-compose up -d nginx`，通过`http://localhost`就可以访问这个应用了



> 注：更多细节请参考官方文档：[http://laradock.io/documentation/](http://laradock.io/documentation/)，以上参考[使用 Laradock 搭建基于 Docker 的 PHP 开发环境](https://laravelacademy.org/post/7691.html)



# 问题汇总



## 1、网络问题

```
#55 0.237 curl: (7) Failed to connect to raw.githubusercontent.com port 443: Connection refused
#55 0.240 /bin/sh: 1: .: Can't open /home/laradock/.nvm/nvm.sh
```



排查思路：

- ping raw.githubusercontent.com
- telnet 443 端口
- 查看本地 hosts 文件
- nslooup 查看 DNS 解析



这里遇到的情况是 DNS 无法解析，解决方法如下：



- 获取其IP，添加本地hosts文件隐射(该方案不可取)
- 更改本机的DNS设置，添加 114.114.114.114 或 8.8.8.8



> 更多请参考：[【原创】curl: (7) Failed to connect to raw.githubusercontent.com port 443: Connection refused](https://blog.csdn.net/u011700186/article/details/109452684)