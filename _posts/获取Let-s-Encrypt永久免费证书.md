---
title: 获取Let's Encrypt永久免费证书
date: 2017-07-20 15:36:30
categories: web
tags: [https, 证书]
---
本文介绍如何使用Let's Encrypt获取免费证书，以支持https服务。

## 步骤如下:

### 1. 拉取git上的工具

```bash
# git clonehttps://github.com/letsencrypt/letsencrypt
# cd letsencrypt
# chmod +x letsencrypt-auto
```

### 2. 获取证书
```bash
# ./letsencrypt-auto certonly --email 邮箱 -d 域名

遇到
How would you like to authenticate with the ACME CA?
-------------------------------------------------------------------------------
1: Spin up a temporary webserver (standalone)
2: Place files in webroot directory (webroot)

选择1
```

### 3. 更新证书(证书有效期3个月)

```bash
./letsencrypt-auto renew
```
可以配置crontab 定期更新

### 4. 配置nginx

证书位置: /etc/letsencrypt/live/域名/
```bash
# HTTPS server
#
server {
    listen       443 ssl;
    server_name  域名;

    ssl_certificate      /etc/letsencrypt/live/域名/fullchain.pem;
    ssl_certificate_key  /etc/letsencrypt/live/域名/privkey.pem;

    ssl_session_cache    shared:SSL:1m;
    ssl_session_timeout  5m;

    ssl_protocols       TLSv1 TLSv1.1 TLSv1.2;
    ssl_ciphers  HIGH:!aNULL:!MD5;
    ssl_prefer_server_ciphers  on;

    location / {
        root   html;
        index  index.html index.htm;
    }
}
```
