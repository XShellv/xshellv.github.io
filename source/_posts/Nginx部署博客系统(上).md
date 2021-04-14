---
title: Nginx部署博客系统(上)
date: 2020-09-20
author:     "Xshellv"
catalog:    true
header-style: text
tags:
    - Next.js
    - Nginx
---

> 博客展示页面已经完成了，现在需要把它从从本地的3000端口，通过Nginx部署到远程服务器，通过访问域名即可打开我们的应用，同时开启压缩、https访问、www域名和顶级域名访问重定向等配置。

## Nginx反向代理

第一步需要做的就是将我们在服务器访问80端口，nginx需要帮助我们转发到3000端口，这一步比较简单。

```nginx
server {
    # ...
    # 博客展示页代理80根路径至3000端口下
    location / {
        proxy_pass http://127.0.0.1:3000;
        proxy_redirect off;
        proxy_set_header Host $host:10001;
        proxy_set_header Host $host:$server_port;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```

完成这一步后重启我们的nginx服务器，浏览器访问http://www.xshellv.com即可正常访问。下面加上我们的域名证书，实现https加密访问。

## https访问配置

```nginx
server {
    listen 443 default_server ssl;
    # 换成自己的域名
    server_name  www.xshellv.com;
    # crt和key结尾的证书，注意这里我是放在当前目录下的，所以直接引用。不同的位置需要引入相应的路径
    ssl_certificate      www.xshellv.com.crt; 
    ssl_certificate_key  www.xshellv.com.key;

    ssl_session_cache    shared:SSL:1m;
    ssl_session_timeout  5m;

    ssl_protocols TLSv1 TLSv1.1 TLSv1.2; 
    #请按照这个套件配置，配置加密套件，写法遵循 openssl 标准。
    ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:HIGH:!aNULL:!MD5:!RC4:!DHE; 
    ssl_prefer_server_ciphers  on;
}
```

## 开启压缩

专业的事情交给专业的工具去做，这里文件压缩没有使用next的配置，而是开启Nginx压缩配置。

```nginx
http {
    gzip  on; # 开启压缩
    gzip_min_length 1k; #低于1kb的资源不压缩
    gzip_comp_level 6; #压缩级别【1-9】，越大压缩率越高，同时消耗cpu资源也越多，建议设置在4左右。
    gzip_types text/plain application/javascript application/x-javascript text/javascript text/xml text/css;  #需要压缩哪些响应类型的资源，多个空格隔开。不建议压缩图片，下面会讲为什么。
    gzip_disable "MSIE [1-6]\.";  #配置禁用gzip条件，支持正则。此处表示ie6及以下不启用gzip（因为ie低版本不支持）
    gzip_vary on;  #是否添加“Vary: Accept-Encoding”响应头
```

## 重定向配置

我们的目的很明确，就是通过访问`http://www.xshellv.com`和`xshellv.com`可以重定向到我们的`https://www.xshellv.com`。（因为忘记配置`xshellv.com`导致我捣鼓半天硬是不知道哪里出问题了。）

### 前提

在阿里云提前对`www`和`@`的主机记录配置。

![阿里云域名解析](https://cdn.xshellv.com/develop/deploy1/resolution_post)

### 配置

```nginx
server {
   # 监听http请求80端口下的xshellv.com和www.xshellv.com，统一重定向至https://www.xshellv.com
   listen 80; 
   server_name xshellv.com,www.xshellv.com;
   return 301 https://www.xshellv.com$request_uri;
}

server {
    # 不要忘记https协议下的xshellv.com也需要进行处理。
    listen 443;
    server_name xshellv.com;
    return 301 https://www.xshellv.com$request_uri;
}
```
