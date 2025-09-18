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

-- 判断字符串是否为JSON格式
local function is_json_string(str)
    if type(str) ~= "string" then
        return false
    end
    -- 简单的JSON格式检测
    local trimmed = str:match("^%s*(.-)%s*$")
    return (trimmed:sub(1,1) == "[" and trimmed:sub(-1) == "]") or 
           (trimmed:sub(1,1) == "{" and trimmed:sub(-1) == "}")
end

-- 处理所有可能包含JSON字符串的字段，将其转换为Lua table
local function process_json_fields(result)
    if not result or #result == 0 then
        return result
    end
    
    for _, row in ipairs(result) do
        for key, value in pairs(row) do
            if type(value) == "string" and is_json_string(value) then
                -- 尝试解析JSON字符串
                local ok, parsed_json = pcall(cjson.decode, value)
                if ok then
                    row[key] = parsed_json
                else
                    -- 如果解析失败，保持原样
                    ngx.log(ngx.WARN, "Failed to parse JSON field '", key, "': ", value)
                end
            end
        end
    end
    return result
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

-- 处理所有JSON字段
local processed_res = process_json_fields(res)

-- 返回查询结果
ngx.say(cjson.encode({
	code = 200,
    success = true,
    data = processed_res,
    count = #processed_res
}))
```

```shell
curl --location 'http://127.0.0.1:9874/api/query' \
--header 'Content-Type: application/json' \
--data '{
    "sql": "SELECT * FROM t_student;"
}'

# 这里添加一下额外的支持， 比如查询结果是一个一对多的数据对象，那么在数据返回的时候，也要正确的展示出来对象的结构。
curl --location 'http://127.0.0.1:9874/api/query' \
--header 'Content-Type: application/json' \
--data '{
    "sql": "SELECT 
    dd.DICT_ID, dd.DICT_CODE, dd.DICT_NAME,
    JSON_ARRAYAGG(
        JSON_OBJECT(
            '\''DICT__DETAIL_ID'\'', ddd.DICT__DETAIL_ID,
            '\''DICT__DETAIL_CODE'\'', ddd.DICT__DETAIL_CODE,
            '\''DICT__DETAIL_NAME'\'', ddd.DICT__DETAIL_NAME
        )
    ) AS details
FROM t_data_dict dd
LEFT JOIN t_data_dict_detail ddd ON dd.DICT_ID = ddd.DICT_ID
GROUP BY dd.DICT_ID, dd.DICT_CODE, dd.DICT_NAME;"
}'
```

## 参考

1. http://pap-docs.pap.net.cn/
2. https://gitee.com/alexgaoyh/
