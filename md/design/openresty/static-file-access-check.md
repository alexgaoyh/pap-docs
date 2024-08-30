## 概述

&ensp;&ensp;《思考 Openresty 静态资源访问控制》 本例记录了在 Windows 环境下使用 Openresty 完成了静态资源访问控制的功能。

## 背景

&ensp;&ensp; 近段时间陆陆续续接收到不少关于信息安全的信息，结合此背景，在静态资源访问的时候，增加一些校验的功能。

## 实现

&ensp;&ensp; 详见如下文件： nginx.conf。

&ensp;&ensp; 在访问静态资源的时候，需要增加 ?token=pap 这个参数如果未添加，则返回403.jpg, 如何访问资源不存在则返回404.jpg。

&ensp;&ensp; 在扩展阶段，可以对 args.token 进行一些解析，比如进行简单的加密等功能。

```html
	server {
		listen       9999;
		server_name  127.0.0.1;
		
		charset utf-8;
		
		gzip on;
		gzip_types text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript image/jpeg image/png image/gif;

		location / {
		
			access_by_lua_block {
				local args = ngx.req.get_uri_args()

				if not args.token or args.token ~= "pap" then
					ngx.status = ngx.HTTP_FORBIDDEN
					ngx.header.content_type = "image/jpeg"

					local file = io.open("D:/pap/403.jpg", "rb")
					if file then
						local content = file:read("*all")
						file:close()
						ngx.say(content)
					else
						ngx.say("403 Forbidden")
					end

					return ngx.exit(ngx.HTTP_OK)
				end

			}
			
			root D:/pap;
			try_files $uri /404.jpg =404;
		}
		
		error_page 403 /403.jpg;

		location = /403.jpg {
			root D:/pap;
		}
		
		error_page 404 =200 /404.jpg;

		location /404.jpg {
			root   D:/pap;
		}
		
		sendfile on;
		tcp_nopush on;
		tcp_nodelay on;

		keepalive_timeout 65;
	}
```

## 参考

1. http://pap-docs.pap.net.cn/
2. https://gitee.com/alexgaoyh/
