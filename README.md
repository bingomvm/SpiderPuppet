
## spiderPuppet

SpiderPuppet是一个借助puppeteer的服务端渲染服务。

这个项目的动机是为了方便服务端/前端来处理搜索引擎的爬虫请求。当爬虫请求下发时，服务端可以直接将请求的url直接转发到本服务上，稍后会使用puppeteer打开对应页面并将页面内容返回。
这种解决方式不需要业务方来针对某些爬虫使用SSR（服务端渲染），对于他们来说方便、快捷。

目前只支持返回渲染后的页面内容,其他功能.....

## 启动本地服务

```
1. npm install

2. npm start
```

启动服务后可以通过http://127.0.0.1:8362访问服务的展示页面。在页面通过输入url来获取url渲染后的内容。

目前服务支持两种数据方式

1. /render?url=xxx 会直接返回渲染后的页面
2. /api/page/render?url=xxx 会返回包含页面内容的JSON对象

## 使用方式

启动服务后可以将爬虫等请求直接转发到服务接口，然后接口就会返回渲染内容。ngxinx和php的简单示例如下：

#### nginx
```
http {
    map $http_user_agent $is_bot {
        default 1;
        ~[a-z]bot[^a-z] 1;
        ~[sS]pider[^a-z] 1;
        ~[sS]pider[^a-z] 1;
        ~[Ww]get 1;
        'Yahoo! Slurp China' 1;
        'Mediapartners-Google' 1;
    }
    server {
        location / {
            error_page 418 =200 @bots;
            if ($is_bot) {
                return 418;
            }
        }
        location @bots {
            proxy_pass_header Server;
            proxy_set_header Host $http_host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Scheme $scheme;
            proxy_set_header Cookie $http_cookie;
            proxy_set_header User-Agent $http_user_agent;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_read_timeout 3000;
            proxy_redirect off;
            proxy_pass $scheme://yourssrserverhost/render?url=$scheme://$host:$server_port$request_uri;
        }
    }
}
```

#### php 
```
<?php
$ssrHost = 'yourssrserverhost';
$protocol = strpos(strtolower($_server['server_protocol']),'https')  === false ? 'http' : 'https';
$host = $protocol.'://'.$_SERVER["SERVER_NAME"];
$user_agent = urlencode($_SERVER['HTTP_USER_AGENT']);
$port = '';

if($protocol == 'http' && $_SERVER["SERVER_PORT"] != 80){
    $port = ':'.$_SERVER["SERVER_PORT"];
}

if($protocol == 'https' && $_SERVER["SERVER_PORT"] != 443){
    $port = ':'.$_SERVER["SERVER_PORT"];
}
$requestUrl = $host.$port.$_SERVER['REQUEST_URI'];
$result = file_get_contents($ssrHost.'/render?url='.$requestUrl);

echo $result;
?>
```

## 部署服务

通过pm2来部署

```
pm2 startOrReload pm2.json

```
如果想自己部署可以通过 ```node production.js```来启动服务

## 修改代码

本项目是基于ThinkJS+Vue。服务端代码在src目录中，修改后直接生效。

如果想要修改前端代码可以通过以下方式:

```
1. npm run dev 

2. 访问127.0.0.1:8361 （所有请求会通过webpack-dev-server转发到8362端口），进行开发

3. 开发完成执行 npm run build 打包代码

```
## 使用docker

构建镜像

```
npm run docker
```

启动容器

```
docker run -t -p yourport:8362 IMAGEID

```

>注意：如果修改了服务端代码需要重新构建镜像，修改了前端代码在构建镜像前需要执行```npm run build```打包前端代码