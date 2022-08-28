---
title: nginx和跨域
date: 
tags:
---
### nginx

> 内容为 nginx基本使用，nginx代理跨域理解
> 

Nginx是一个轻量级、高性能、稳定性高、并发性好的HTTP和反向代理服务器。

<!--more-->

（1）常用操作
```shell
#启动
nginx
#关闭
nginx -s stop
#修改了配置，重新加载
nginx -s reload
#检查配置文件格式
nginx -t
```
配置文件地址：mac=>/usr/local/etc/nginx/nginx.conf , docker=>/etc/nginx

[配置文件结构说明](https://juejin.im/post/6844903575210967048)

```txt
user  nginx;
worker_processes  1;

error_log  /var/log/nginx/error.log warn;
pid        /var/run/nginx.pid;

events {
    worker_connections  1024;
}

http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    keepalive_timeout  65;

    server {
        listen       80;
        listen  [::]:80;
        server_name  localhost;

        #charset koi8-r;
        #access_log  /var/log/nginx/host.access.log  main;
        #接收到请求，去/usr/share/nginx/html目录下找静态资源，index.html或index.htm
        location / {
            root   /usr/share/nginx/html;
            index  index.html index.htm;
            #proxy_pass http://localhost:8081/;
        }

        #error_page  404              /404.html;

        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   /usr/share/nginx/html;
        }

    }
    #gzip  on;
    #引入conf.d文件夹下以.conf结尾的配置文件
    include /etc/nginx/conf.d/*.conf;
}
```

（2）代理 

数据请求：浏览器=>代理服务器=>目标服务器

数据返回：目标服务器=>代理服务器=>浏览器

例子：科学上网。

**从服务器的角度，没有办法分辨请求来自代理还是浏览器，代理的本质是隐藏了客户端。**

（3）反向代理

数据请求：浏览器=>反向代理服务器=>目标服务器

数据返回：目标服务器=>反向代理服务器=>浏览器

**流程和代理差不多，但是代理在浏览器端配置，而反向代理靠近服务器，在服务端配置。**

**从浏览器的角度，不知道请求最终被哪个服务器所处理，反向代理的本质是隐藏了服务端。**

### 同源策略

（1）同源策略[MDN](https://developer.mozilla.org/zh-CN/docs/Web/Security/Same-origin_policy)

​	浏览器的一个安全策略，限制同源**[协议、域名、端口都一致]**的文档以及它们加载的脚本怎么与另一个源的资源交互。同源策略控制不同源之间的交互，这些交互通常分三类：

- 跨域**写操作**（Cross-origin writes）*一般是被允许的*。例如链接（links），重定向以及表单提交。

- 跨域**资源嵌入**（Cross-origin embedding）一般是被允许（后面会举例说明）。

- 跨域**读操作**（Cross-origin reads）*一般是不被允许的*，但常可以通过内嵌资源来巧妙的进行读取访问

资源嵌入示例：

\<script src="...">\</script> 标签嵌入跨域脚本。语法错误信息只能被同源脚本中捕捉到。
\<link rel="stylesheet" href="..."> 标签嵌入CSS。由于CSS的松散的语法规则，CSS的跨域需要一个设置正确的 HTTP 头部 Content-Type 。
通过 \<img> 展示的图片。支持的图片格式包括PNG,JPEG,GIF,BMP,SVG,...
通过 \<video> 和 \<audio> 播放的多媒体资源。
通过 \<object>、 \<embed> 和 \<applet> 嵌入的插件。
通过 @font-face 引入的字体。一些浏览器允许跨域字体（ cross-origin fonts），一些需要同源字体（same-origin fonts）。
通过\<iframe> 载入的任何资源。站点可以使用 X-Frame-Options 消息头来阻止这种形式的跨域交互。

（2）跨域方案

jsonp
|type|地址|
|--|--|
|前端||
|后端|http://localhost:8500|

前端：

```html
<script type="text/javascript">
  var dataHandle = function(data) {
    alert(
      "jsonp调用服务端结果：" +
        data.result
    );
  };
  // 拼script
  let url = 'http://127.0.0.1:8500?callback=dataHandle&name=jack';
  var script = document.createElement('script');
  script.setAttribute('src', url);
  document.getElementsByTagName('head')[0].appendChild(script);
</script>
```

后端：

```js
const http = require("http");
const url = require("url");
const querystring = require("querystring");
var fs = require('fs')

const hostname = "127.0.0.1";
const port = 8500;

const server = http.createServer();
server.on("clientError", (err, socket) => {
  socket.end("HTTP/1.1 400 Bad Request\r\n\r\n");
});

server.on("request", (req, res) => {
  var arg = url.parse(req.url).query;
  var params = querystring.parse(arg);

  const data = {
      jack:"i am jack",
      tom:"i am tom"
  }
  //识别请求和参数
  if (params.callback && params.callback == "dataHandle") {
    //将查询数据和调用函数表达式拼接返回给客户端
    res.write(`dataHandle({"result":"${data[params.name]}"});`)   
    res.end()
  }
});

server.listen(port, hostname, () => {
  console.log(`Server running at http://${hostname}:${port}/`);
});

```



开启跨域资源共享（Cross-oringin resource sharing，CORS）[MDN](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Access_control_CORS)

|type|地址|
|--|--|
|前端|http://localhost:8081|
|后端|http://localhost:9000|

在后端设置HTTP响应首部字段 Access-Control-Allow-Origin：

```js
  response.setHeader('Access-Control-Allow-Origin', 'http://localhost:8081');
```

**过程**：浏览器通过8081直接请求9000服务端口

nginx反向代理

|type|地址 |
|--|--|
|前端|http://localhost:8081|
|后端|http://localhost:8000|
|nginx|代理8866端口，来自http://localhost:8866的请求转发到8000|

```shell
server {
  listen       8866;
  server_name  127.0.0.1;  # 需要中转的请求
  location / {
    proxy_pass http://127.0.0.1:8000; # 反向代理后端地址
    index index.html index.htm;
    # 跨域配置，给浏览器设置响应头
    add_header Access-Control-Allow-Origin http://localhost:8081; 
    add_header Access-Control-Allow-Credentials true;
  }
}   
```

**过程**：浏览器从8081端口向nginx的8866端口请求数据，nginx反向代理8866端口，将对8866端口的请求转发给真实服务端接口8000.

**解析**：浏览器从8081端口向8866端口请求资源是跨域行为，须在nginx配置跨域如上。nginx到8000的代理请求不经过浏览器，不受浏览器的同源策略影响，可以访问。

### 总结：

​	nginx反向代理和CORS都是通过设置HTTP响应首部字段 Access-Control-Allow-Origin，开启跨域资源共享机制。不同点在于，nginx反向代理是nginx向浏览器开放，因而不需要服务器额外配置，而CORS是服务端向浏览器开放，需在服务端配置。