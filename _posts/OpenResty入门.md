---
title: OpenResty入门
date: 2016-09-28 21:35:47
categories: OpenResty
tags: [OpenResty Nginx]
---

## OpenResty介绍

 - Nginx的非阻塞网络模型，高并发、高性能
 - Lua脚本语言的灵活、小巧、高效
 - Lua协程与Nginx非阻塞网络模型融合，以同步的方式写非阻塞代码
 - OpenResty生态日趋完善，大量应用在高并发场景下，大量优秀的模块
 - 参考
   * https://openresty.org/en/download.html
   * http://nginx.org/en/docs/
   * http://luajit.org/index.html
   * http://www.lua.org/manual/5.1/
   * https://www.gitbook.com/book/moonbingbing/openresty-best-practices/details(最佳实践)
   * https://github.com/openresty/lua-nginx-module
   * https://github.com/Iresty/nginx-lua-module-zh-wiki(上一官方文档中文翻译)
   * https://github.com/bungle/awesome-resty(相关资源整理汇总)

## 安装配置命令(ubuntu)
### 安装步骤
 1. 依赖安装
 
 ```bash
 sudo apt-get install libreadline-dev libncurses5-dev libpcre3-dev ibssl-dev perl make build-essential
 ```
 2. 源码解压
 
 ```bash
 tar zxvf openresty-version.tar.gz && cd openresty-version/
 ```
 3. configure

 ```bash
 ./configure --prefix=/opt/openresty --with-luajit --withhttp_iconv_module -j2
 ```

 4. 编译
 
 ```bash
 make -j2
 ```
 5. 安装
 
 ```bash
 sudo make install
 ```
 
### 命令
```bash
./configure --help      # 查看安装配置参数
nginx                   # 启动
nginx -p /opt/openresty/nginx   # 指定启动prefix路径
nginx -c /opt/openresty/nginx/conf/nginx.conf       # 指定启动配置文件
nginx -t                # 检查nginx.conf配置文件里的配置项合法否
nginx -V                # 查看版本及编译进去的模块
nginx -s stop           # 强制退出
nginx -s quit           # 优雅退出
nginx -s reload         # 重新加载配置文件 优雅重启
nginx -s reopen         # 重新打开日志文件, 用于切换日志
```

### 配置
```nginx
user  resty;    # worker进程运行用户 涉及到权限
worker_processes  auto;     # worker进程数为cpu核数
worker_cpu_affinity auto;   # worker进程cpu亲和性

error_log  logs/error.log;  # 日志

pid        logs/nginx.pid;  # master进程ID 用户nginx -s quit等命令

worker_rlimit_nofile 65535; # 绕过系统单进程最大文件句柄打开数

events {
    use epoll;              # epoll网络模型
    worker_connections  65535;      # worker单进程最大连接数
    multi_accept on;        # 惊群
}

http {
    include       mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';     # access_log格式

    access_log  logs/access.log  main;      # access_log格式

    sendfile        on;
    tcp_nopush     on;
    tcp_nodelay on;

    #keepalive_timeout  0;
    keepalive_timeout  65;      # http长连接
    keepalive_disable none;     # 不再通过 User-Agent 中的浏览器信息，来决定是否 keepalive

    # 压缩
    gzip on;
    gzip_min_length 1k;
    gzip_buffers 4 16k;
    gzip_http_version 1.0;
    gzip_comp_level 4;
    gzip_types text/plain application/x-javascript text/css application/xml;
    gzip_vary on;

    server {
        listen       80;
        server_name  coder-zhuyu.info;

        charset utf8;

        #access_log  logs/host.access.log  main;
        root html/blog;

        location / {
            # root   html;
            index  index.html index.htm;
        }

        # 缓存文件
        location ~ .*\.(gif|jpg|jpeg|png|bmp|swf|mp3)$ {
            expires      30d;
        }

        location ~ .*\.(js|css)?$ {
            expires      30d;
        }

        location ~ .*\.(svg)$ {
            root html;
            expires     30d;
        }

        #error_page  404              /404.html;

        # redirect server error pages to the static page /50x.html
        #
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
    }

}
stream {
    # TCP UDP
}
```

## hello world
```lua
location /helloworld {
    content_by_lua_block {
        ngx.say("Hello, World!")
    }
}
```

## 技能图谱
![技能图谱][1]

  [1]: https://coder-zhuyu.github.io/images/blog/png-OpenResty-by-StuQ.png
