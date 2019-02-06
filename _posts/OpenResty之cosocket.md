---
title: OpenResty之cosocket
date: 2016-10-09 22:03:50
categories: OpenResty
tags: [OpenResty cosocket]
---

cosocket是OpenResty世界中很重要的一个技术，cosocket是依赖Lua协程 + Nginx事件通知两个重要特性拼的。Lua脚本运行在独享的协程之上，可以在需要的时候暂停自己(yield)，在条件满足的时候被唤醒(resume)。网络操作时，暂停自己，把网络事件注册到Nginx监听列表中，把cpu让出，运行权限交给Nginx。当有Nginx注册网络事件达到触发条件时，唤醒对应的协程继续处理。

大量的模块都是基于cosocket，例如:

 - lua-resty-mysql
 - lua-resty-redis
 - lua-resty-websocket
 - lua-resty-http
 - ngx_stream_lua_module
 - ......

# ngx.socket.tcp的一个例子
```lua
-- server
stream {
    server {  
        listen 1234;                   
        content_by_lua_block {         
            local sock, err = ngx.req.socket()
            if not sock then           
                ngx.exit(500)          
            end   
            while true do              
                local data             
                data, err = sock:receive('*l')
                if not data then       
                    return ngx.exit(200)   
                end                    
                local sent             
                sent, err = sock:send(data .. '\n')
                if err then            
                    return ngx.exit(200)
                end
            end
        }
    }
}                                                  
```

```lua
-- client
local sock = ngx.socket.tcp()
local ok, err = sock:connect("127.0.0.1", 1234)
if not ok then   
    ngx.say("failed to connect: ", err)
    return   
end
local req_data = "hello world\r\n"
local bytes, err = sock:send(req_data)
if err then  
    ngx.say("failed to send: ", err)
    return   
end
local data, err, partial = sock:receive('*l')
if err then  
    ngx.say("failed to recieve: ", err)
    return   
end
sock:close() 
ngx.say("response first line: ", data)  

```
