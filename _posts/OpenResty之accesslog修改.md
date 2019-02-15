---
title: OpenResty之AccessLog记录response
date: 2016-10-14 22:15:30
categories: OpenResty
tags: [OpenResty accesslog]
---
Nginx的access log是在请求处理完成才写入日志文件的，下面将使用OpenResty将response加入到access log里面。
```bash
http {
    log_format log_req_resp '$remote_addr - $remote_user [$time_local] '
        '"$request" $status $body_bytes_sent '
        '"$http_referer" "$http_user_agent" $request_time
        req_body:"$request_body" resp_body:"$resp_body"';
    server {
        listen       80;
        server_name  localhost;
        access_log /tmp/nginx.resp.access.log log_req_resp;
        lua_need_request_body on;
        set $resp_body "";      # set resp_body 为空
        location / {
            body_filter_by_lua_block {
                local resp_body = ngx.arg[1]
                ngx.ctx.buffered = (ngx.ctx.buffered or "") .. resp_body
                if ngx.arg[2] then      # 判断是否是结尾
                     ngx.var.resp_body = ngx.ctx.buffered
                end
            }
    
            proxy_pass http://xxx.xxx.xxx.xxx/;
        }
    }
}

```

access log中文会被escape，可以对源码做修改来正常显示中文:
```c
//ngx_http_log_module.c

static u_char *
ngx_http_log_variable(ngx_http_request_t *r, u_char *buf, ngx_http_log_op_t *op)
{
    ngx_http_log_loc_conf_t    *lcf;
    ngx_http_variable_value_t  *value;

    value = ngx_http_get_indexed_variable(r, op->data);

    if (value == NULL || value->not_found) {
        *buf = '-';
        return buf + 1;
    }
    
    return ngx_cpymem(buf, value->data, value->len);
// 注释下面代码
#if 0
    if (value->escape == 0) {
        return ngx_cpymem(buf, value->data, value->len);

    } else {
        lcf = ngx_http_get_module_loc_conf(r, ngx_http_log_module);
        return (u_char *) ngx_http_log_escape(lcf, buf, value->data, value->len);
    }
#endif
}
```
