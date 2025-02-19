## 概述

&ensp;&ensp;《记录 Openresty 中 Redis 的使用》 本例记录了在 Windows 环境下使用 Openresty 操作 Redis 的功能。

## 背景

&ensp;&ensp; 当前仅仅是脚本的记录，后续会扩展《思考 Openresty 静态资源访问控制》，从而限制访问码只能使用一次，并且不再使用本地缓存。

## 实现

&ensp;&ensp; 详见如下文件： nginx.conf。

&ensp;&ensp; 在访问资源的时候，需要增加 ?key=pap 这个参数，并根据情况将值保存至Redis中。

```html
	server {
		listen 9969;
		server_name 127.0.0.1;

		location /redis_test {
			content_by_lua_block {
				local redis = require "resty.redis"
                local red = redis:new()

                red:set_timeout(1000)

				ngx.header["Content-Type"] = "text/html; charset=utf-8"
                local ok, err = red:connect("127.0.0.1", 6379)
                if not ok then
                    ngx.say("failed to connect to redis: ", err)
                    return
                end

                local key = ngx.var.arg_key
                if not key then
                    ngx.say("key is required")
                    return
                end

                local res, err = red:get(key)
                if res == ngx.null then
                    -- Key does not exist, add it to Redis
                    local ok, err = red:set(key, "1", "EX", 5)
                    if not ok then
                        ngx.say("failed to set key: ", err)
                        return
                    end
                    ngx.say("key added to redis: ", key)
                else
                    -- Key exists, return error
                    ngx.status = ngx.HTTP_BAD_REQUEST
                    ngx.say("key already exists: ", key)
                    return
                end

                -- Put the connection back into the connection pool
                local ok, err = red:set_keepalive(10000, 100)
                if not ok then
                    ngx.say("failed to set keepalive: ", err)
                    return
                end
			}
		}
	}
```

## 参考

1. http://pap-docs.pap.net.cn/
2. https://gitee.com/alexgaoyh/
