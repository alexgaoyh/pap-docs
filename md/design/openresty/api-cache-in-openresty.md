## 概述

&ensp;&ensp;《思考 Openresty 接口缓存》 本例记录了在 Windows 环境下使用 Openresty 完成了接口数据缓存的功能。

## 背景

&ensp;&ensp; 近段时间在做一些整理工作，突然看到很早之前写的多级缓存的文字，结合此背景，考虑在 Openresty 层再加一层缓存。

## 实现

&ensp;&ensp; 详见如下文件： nginx.conf。

&ensp;&ensp; 在进行数据访问的时候，先去判断是否有缓存，如有则直接返回，没有调整到后端进行执行。

```html
	lua_shared_dict cache54321 10m;
	
	server {
		listen       54321;
		server_name  127.0.0.1;
	

		location / {
			content_by_lua_block {
				local cache = ngx.shared.cache54321
				local cache_key = ngx.var.request_uri
				local data, stale_data, flags = cache:get(cache_key)

				if data then
					ngx.header['Content-Type'] = 'application/json;charset=UTF-8'
					ngx.say(data)
					return
				end

				local backend_response = ngx.location.capture("/backend" .. ngx.var.uri, {
					method = ngx.HTTP_GET,
				})

				if backend_response.status == 200 then
					local body = backend_response.body
					ngx.header['Content-Type'] = 'application/json;charset=UTF-8'
					local success = cache:set(cache_key, body, 60)
					ngx.say(body)
				else
					ngx.exit(backend_response.status)
				end
			}
		}

		location /backend {
			rewrite ^/backend(.*) $1 break;
			proxy_pass http://localhost:30000;
		}
	}
```

## 参考

1. http://pap-docs.pap.net.cn/
2. https://gitee.com/alexgaoyh/
