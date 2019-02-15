---
title: lua-resty-mysql事务封装
date: 2016-10-22 20:23:05
categories: OpenResty
tags: [OpenResty lua-resty-mysql]
---

lua-resty-mysql没有提供事务封装，下面提供一个事务的封装例子的主要代码部分。事务是基于session，主要将事务相关语句在同一个session上执行。
```lua
local mysql = require "resty.mysql"
local config = require "config"
local log = require "utils.log"
local cjson = require "cjson"
local u_string = require "utils.string"

local _M = {}

local mt = {__index = _M}

function _M.new(self)
    local db, err = mysql:new()
    if not db then
        log.err("failed to instantiate mysql: ", err)
        return nil
    end

    -- 连接超时时间
    db:set_timeout(config.get('mysql_conn_timeout'))

    -- 连接数据库
    local ok, err, errno, sqlstate = db:connect(config.get('mysql_conn'))
    if not ok then
        log.err("failed to connect: ", err, ": ", errno, ": ", sqlstate)
        return nil
    end

    return setmetatable({ db = db }, mt)
end


-- 开启事务
function _M.transaction_start(self)
    local db = self.db

    local res, err, errno, sqlstate = db:query("START TRANSACTION")
    if not res then
        log.err("START TRANSACTION failed: ", err, ": ", errno, ": ", sqlstate)
        return nil
    end

    return true
end


-- 提交事务
function _M.transaction_commit(self)
    local db = self.db

    local res, err, errno, sqlstate = db:query("COMMIT")
    if not res then
        log.err("COMMIT failed: ", err, ": ", errno, ": ", sqlstate)
        return nil
    end

    -- 连接池
    local pool = config.get('mysql_pool')
    local ok, err = db:set_keepalive(pool.timeout, pool.size)
    if not ok then
        log.err("failed to set keepalive: " .. err)
    end

    return true
end


-- 回滚事务
function _M.transaction_rollback(self)
    local db = self.db

    local res, err, errno, sqlstate = db:query("ROLLBACK")
    if not res then
        log.err("ROLLBACK failed: ", err, ": ", errno, ": ", sqlstate)
        return nil
    end

    -- 连接池
    local pool = config.get('mysql_pool')
    local ok, err = db:set_keepalive(pool.timeout, pool.size)
    if not ok then
        log.err("failed to set keepalive: " .. err)
    end

    return true
end


-- 执行sql
function _M.execute(self, sql, params)
    local sql = u_string.parse_sql(sql, params)
    if not sql then
        log.err("sql format error: ", sql, ": ", cjson.encode(params))
        return nil
    end

    local db = self.db

    local res, err, errno, sqlstate = db:query(sql)
    if not res then
        log.err("sql execute failed: ", err, ": ", errno, ": ", sqlstate)
        return nil, errno
    end

    return res
end


function _M.select(self, sql, params)
    return self:execute(sql, params)
end

function _M.insert(self, sql, params)
    local res, errno = self:execute(sql, params)
    if res and res.affected_rows > 0 then
        return true
    else
        return false, errno
    end
end

function _M.update(self, sql, params)
    local res, errno = self:execute(sql, params)
    if res and res.affected_rows > 0 then
        return true
    else
        return false, errno
    end
end

function _M.delete(self, sql, params)
    local res, errno = self:execute(sql, params)
    if res and res.affected_rows > 0 then
        return true
    else
        return false, errno
    end
end

return _M
```

sql格式化代码：
```lua
-- sql 格式化
function _M.parse_sql(sql, params)
    if not params or not u_table.is_array(params) or #params == 0 then
        return sql
    end

    if not sql then return nil end

    local new_params = {}
    for _, v in ipairs(params) do
        if type(v) == 'string' then
            tab_insert(new_params, ngx_quote_sql_str(v))
        else
            tab_insert(new_params, v)
        end
    end

    sql = str_format(sql, unpack(new_params))
    return sql
end
```
