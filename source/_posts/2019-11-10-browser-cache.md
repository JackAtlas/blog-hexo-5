title: 浏览器缓存小结
date: 2019-11-10 12:38:07
---

关于浏览器缓存的小结。

<!-- more -->

浏览器收到服务器的响应后，会检查响应头信息，一般用于缓存相关的头信息有 4 个：

- ETag
- Cache-Control
- Expires
- Last-Modified

## ETag

> ETag（Entity Tag）是一个用于缓存校验的 token 字符串，通常以文件的哈希值来表示。

浏览器在文件的响应头中接收到 ETag，当文件过期后，浏览器再次向服务器请求该文件时会附上先前接收到的 ETag；服务器通过对比 ETag 和文件的哈希值判断资源是否变化，如果没有则返回 304，浏览器就知道缓存中的资源是安全的。

**注意：**仅当缓存中的文件过期了，ETag 才会被用在请求中。

## Cache-Control

> Cache-Control 常用于设置资源的缓存行为、过期时间、验证等。

### 缓存行为

- `Cache-Control: public`：资源可以被任意缓存（浏览器、CDN 等）
- `Cache-Control: private`：资源只能被浏览器缓存
- `Cache-Control: no-store`：资源不被缓存，浏览器总是请求服务器获取最新文件
- `Cache-Control: no-cache`：资源被浏览器缓存，但使用前总会先请求服务器检查文件（常用于 html 文件）

### 过期时间

- `Cache-Control: max-age=60`：指定资源应该被缓存多少秒
- `Cache-Control: s-max-age=60`：同上，用于中间缓存（CDN 等）
- `Cache-Control: must-revalidate`：资源被使用前必须去验证过期状态

## Expires

Expires 来自 http 1.0 时代，该字段指定了一个日期，超过这个日期就代表资源是无效的，如果已经指定了 Cache-Control 中的 max-age，则浏览器就会忽略 expires。

```
Expires: Wed, 25 Jul 2018 21:00:00 GMT
```

## Last-Modified

Last-Modified 也是来自 http 1.0 时代，该字段包含了资源最后修改的日期和时间。

```
Last-Modified: Mon, 12 Dec 2016 14:45:00 GMT
```

## HTML Meta Tag

在 HTML5 版本之前，在 html 中使用元标签（meta）指定 cache-control 是一个有效的方式

```html
<meta http-equiv="Cache-control" content="no-cache">
```

但在 HTML5 中不推荐这么做，因为只有浏览器可以识别这个标签，中间缓存（CDN）是识别不了的。

## HTTP Response

一个简单的 http 响应：

```
Accept-Ranges: bytes
Cache-Control: max-age=3600
Connection: Keep-Alive
Content-Length: 4361
Content-Type: image/png
Date: Tue, 25 Jul 2017 17:26:16 GMT
ETag: "1109-554221c5c8540"
Expires: Tue, 25 Jul 2017 18:26:16 GMT
Keep-Alive: timeout=5, max=93
Last-Modified: Wed, 12 Jul 2017 17:26:05 GMT
Server: Apache
```

- 第 2 行告诉我们 max-age 为 1 小时
- 第 5 行告诉我们这是一个 png 图片资源
- 第 7 行告诉我们 ETag 将在 1 个消失后去验证资源是否改变过
- 第 8 行，Expires 头会被忽略，因为已经设置了 Cache-Control: max-age=3600
- 第 10 行，Last-Modified 头展示了图片资源的最后修改时间

## Cache Busting

Cache Busting 会使一个资源文件失效，强制浏览器去服务器端重新获取数据。

我们可以通过改变文件名来命令浏览器去避开缓存，对于浏览器来说，这是一份全新的资源，所以会去请求最新的数据。

### 版本号

我们可以给资源添加一个版本号。

```
assets/js/app-v2.min.js
```

### 指纹（fingerpring）

我们可以基于文件内容添加一个指纹值。

```
assets/js/app-d41d8cd98f00b204e9800998ecf8427e.min.js
```

### 拼接查询参数

我们可以在文件名的末尾拼接一个查询参数。

```
assets/js/app.min.js?version=2
```

## 最佳实践

### 推荐

- 对于静态资源来说，使用 Cache-Control 和 ETag 头部来控制缓存的行为
- 设置一个比较长的 max-age 值，以此来获得浏览器缓存带来的好处
- 针对 Cache Busting，使用指纹值（或 hash 值）和版本号的方式

### 不推荐

- 使用 html 元标签去指定缓存的行为
- 使用查询参数的方式实现 Cache Busting
