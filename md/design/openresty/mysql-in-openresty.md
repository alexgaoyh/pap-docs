## 概述

&ensp;&ensp;《记录 Openresty 中 Mysql 的使用》 本例记录了在 Windows 环境下使用 Openresty 操作 Mysql 的功能。

## 背景

&ensp;&ensp; 当前仅仅是脚本的记录，用来进行 SELECT 语句的操作，返回查询结果的数据。

## 实现

&ensp;&ensp; 详见如下文件： nginx.conf。

&ensp;&ensp; 同时新增了 mysqlapi.lua 的脚本。

```html
	server {
        listen       9874;
        server_name  127.0.0.1;
        
        # 查询API
        location /api/query {
            content_by_lua_file conf/lua/mysqlapi.lua;
        }
        
        # 健康检查
        location /health {
            return 200 '{"status": "ok"}';
        }
    }
```

```html
local cjson = require "cjson"

-- 设置响应头
ngx.header.content_type = "application/json; charset=utf-8"

-- 只允许POST请求
if ngx.var.request_method ~= "POST" then
    ngx.status = ngx.HTTP_METHOD_NOT_ALLOWED
    ngx.say(cjson.encode({
        success = false,
        error = "Only POST method is allowed"
    }))
    return
end

-- 读取请求体
ngx.req.read_body()
local body = ngx.req.get_body_data()

if not body then
    ngx.status = ngx.HTTP_BAD_REQUEST
    ngx.say(cjson.encode({
        success = false,
        error = "Request body is empty"
    }))
    return
end

-- 解析JSON请求
local ok, params = pcall(cjson.decode, body)
if not ok or not params.sql then
    ngx.status = ngx.HTTP_BAD_REQUEST
    ngx.say(cjson.encode({
        success = false,
        error = "Invalid JSON or missing 'sql' parameter"
    }))
    return
end

local sql = params.sql

-- 简单的SQL注入防护：只允许SELECT语句
if not sql:lower():match("^%s*select") then
    ngx.status = ngx.HTTP_BAD_REQUEST
    ngx.say(cjson.encode({
        success = false,
        error = "Only SELECT statements are allowed"
    }))
    return
end

-- MySQL配置
local db_config = {
    host = "127.0.0.1",
    port = 3306,
    database = "cf",
    user = "root",
    password = "alexgaoyh", -- 改为您的MySQL密码
    max_packet_size = 1024 * 1024,
    charset = "utf8"
}

-- 执行MySQL查询的函数
local function mysql_query(sql_query)
    local mysql = require "resty.mysql"
    
    -- 创建MySQL对象
    local db, err = mysql:new()
    if not db then
        return nil, "Failed to create MySQL object: " .. tostring(err)
    end

    -- 设置超时
    db:set_timeout(5000)

    -- 连接数据库
    local ok, err, errcode, sqlstate = db:connect(db_config)
    if not ok then
        return nil, "Connection failed: " .. tostring(err)
    end

    -- 执行查询
    local res, err, errcode, sqlstate = db:query(sql_query)
    
    -- 关闭连接
    local close_ok, close_err = db:close()
    if not close_ok then
        ngx.log(ngx.ERR, "Failed to close connection: ", close_err)
    end

    if not res then
        return nil, "Query failed: " .. tostring(err)
    end

    return res
end

-- 执行查询
local res, err = mysql_query(sql)
if not res then
    ngx.status = ngx.HTTP_INTERNAL_SERVER_ERROR
    ngx.say(cjson.encode({
        success = false,
        error = "Database query failed: " .. tostring(err)
    }))
    return
end

-- 返回查询结果
ngx.say(cjson.encode({
	code = 200,
    success = true,
    data = res,
    count = #res
}))
```

```shell
curl --location 'http://127.0.0.1:9874/api/query' \
--header 'Content-Type: application/json' \
--data '{
    "sql": "SELECT * FROM t_student;"
}'
```

## 参考

1. http://pap-docs.pap.net.cn/
2. https://gitee.com/alexgaoyh/
