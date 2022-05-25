---
title: Nginx 通过客户端标识过滤请求
date: 2022-05-25 17:00:00
# description: redis 在项目中
categories:
- 服务器
tags:
- nginx
---

&emsp;&emsp;网页爬虫对站点收录和退关有好处，但大量蜘蛛涌入可能会造成网站瘫痪，通过配置 Nginx 可以限制爬虫请求。在 Nginx 的配置 `conf.d` 目录下增加请求频率限制定义文件 `user-agent-rate-limit.conf`。

```
# define two rules: hard and soft 

map $http_user_agent $rate_bot {
    default "";
    "~Googlebot" 1;
    "~Bingbot" 2;
    
    # could add more spiders
}

// 429: too many requests
limit_req_status 429;

# soft rate limit
map $rate_bot $rate_bot_soft {
    default "";
    1 "ratebot_soft";
}

limit_req_zone $rate_bot_soft zone=ratebot_soft:16m rate=5r/s;

# hard rate limit
map $rate_bot $rate_bot_hard {
    default "";
    2 "ratebot_hard";
}

limit_req_zone $rate_bot_hard zone=ratebot_hard:16m rate=1r/s;

```

应用规则

```
server {

    # ....
    limit_req zone=ratebot_soft nodelay;
    limit_req zone=ratebot_hard nodelay;
    # ...
    
}    
```

参考：

- [Module ngx_http_limit_req_module](https://nginx.org/en/docs/http/ngx_http_limit_req_module.html)
- [Module ngx_http_map_module](https://nginx.org/en/docs/http/ngx_http_map_module.html)
- [Rate limit NGINX by User-Agent](https://urlund.com/blog/rate-limit-nginx-by-user-agent/)
- [Rate Limiting with NGINX and NGINX Plus](https://www.nginx.com/blog/rate-limiting-nginx)