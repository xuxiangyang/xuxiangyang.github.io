---
layout: post
title: "React经验"
date: 2016-02-28 01:16:58 +0800
comments: true
categories: React
---
## Promise 低版本不支持。安卓4.4不支持 Promise语法。需要安装`babel-polyfill` 然后require

## Nginx 未转发 router里的请求

nginx 需要特殊设定，才能转发所有router。总体思路就是把所有请求都交给index.html处理

```
server {
        listen 80;
        server_name cet.kechenggezi.com;
        access_log  logs/cet_web.kechenggezi.com.access.log;
        root /var/www/cet_web/production/current;
        location / {
                try_files $uri /index.html =404;
                expires    -5y;
                add_header Pragma no-cache;
                add_header Cache-Control "no-cache, no-store, must-revalidate";
        }
}
```

##  index.html（html入口）浏览器中开缓存，导致更新失效。

一般来说修改nginx就ok了，在nginx中加入

```
expires    -5y;
add_header Pragma no-cache;
add_header Cache-Control "no-cache, no-store, must-revalidate";
```

也可以在html header中加入如下meta

```
<meta http-equiv="Cache-Control" content="no-cache, no-store, must-revalidate" />
<meta http-equiv="Pragma" content="no-cache" />
<meta http-equiv="Expires" content="0" />
```

关闭缓存

## 服务端cors，请求跨域

需要服务端处理。如果是rails，推荐使用下面的gem

```
gem 'rack-cors', "~> 0.4.0", :require => 'rack/cors'
```

然后在不同的environment中进行自己的配置。如下：

```
  config.middleware.insert_before 0, "Rack::Cors" do
    allow do
      origins 'cet.kechenggezi.com', "web.cet.kechenggezi.com"
      resource '*',
        :headers => :any,
        :methods => [:get, :post, :options, :delete, :put],
        :max_age => 1728000
    end
  end
```

不推荐允许origins 为 *

## 不要用jquery-mobile

这个库特别慢。加载了它打开页面需要好久。如果需要手机上的手势可以使用`hammerjs`库实现。



## 好的JS库

1. immutable： js的hash、array等数据结构实现。它的每一个操作都是返回一个新的对象。适合用来表示state
2. isomorphic-fetch: fetch的模拟实现。fetch是ecma6才有的。绝大多数浏览器并不支持。这个库实现了fetch，可以让老版本浏览器支持
3. redux-router/react-router: 路由。将路由与redux状态同步
4. babel-polyfill：ecma6仿真。依旧是让老版本可以支持
5. react-addons-css-transition-group: react页面间动画。可以用它来模拟ios右滑动画效果。配合react-static-container使用
6. hammerjs：手机手势仿真
