title: 记一次 nginx 配置错误排查
date: 2018-11-01 18:25:04
---

Nginx 入门级选手记录某日配置错误的排查经过。

<!-- more -->

进入正文前先介绍一下本站的基本架构：我把本站的几个基础业务分在了不同的二级域名下，通过 nginx 进行反向代理。如接口在 api 二级域名下，管理后台在 admin 下等。

由于浏览器的同源策略，当 admin 的页面访问 api 的接口时会报出跨域的错误，需要通过在 nginx 设置 CORS 来解决。

```
# 前略

if ($http_origin = 'http://admin.jackatlas.com') {
    add_header 'Access-Control-Allow-Origin' $http_origin;
    add_header 'Access-Control-Allow-Headers' 'Authorization,Content-Type,Accept,Origin,User-Agent,DNT,Cache-Control,X-Mx-ReqToken,X-Requested-With';
    add_header 'Access-Control-Allow-Methods' 'GET, POST, OPTIONS, PUT, DELETE';
}

if ($request_method = 'OPTIONS') {
    add_header 'Access-Control-Allow-Origin' $http_origin;
    add_header 'Access-Control-Allow-Headers' 'Authorization,Content-Type,Accept,Origin,User-Agent,DNT,Cache-Control,X-Mx-ReqToken,X-Requested-With';
    add_header 'Access-Control-Max-Age' 1728000;
    add_header 'Content-Length' 0;
    add_header 'Content-Type' 'text/plain, charset=utf-8';
    return 204;
}

# 后略
```

如上配置后，在 admin 访问 api 下的 `login` 接口一切正常，再访问 `user` 接口获取用户信息则报出两个错误：服务器 500 及跨域。

查阅 [nginx 官方文档](https://nginx.org/en/docs/http/ngx_http_headers_module.html#add_header)发现默认配置下只在返回 20X 和 30X 状态码时会返回自定义的请求头，即 500 状态下不会返回上面配置的请求头。

继续排查发现 500 的原因是 session 为空，则 session 中的用户信息 `undefined` 引起了错误。

原因是没有配置允许 cookies 跨域，需要在配置中加入：

```
add_header 'Access-Control-Allow-Credentials' 'true';
```

并在前端访问时加上 `withCredentials: true` 的请求头。

如此配置后，admin 可以顺利访问 api 的接口了。但还有若干疑问尚未解决：

1. 有几条重复的添加请求头的语句，如果把 `OPTIONS` 条件分支下的那几条重复的删掉，`prelight` 就会不通过；
2. 尝试过把 `OPTIONS` 条件分支及其中的 `prelight` 配置都放进前面的条件分支语句中，运行 `nginx -t` 会报出语法错误。

PS. 感谢朱西同学的协助和解答。
