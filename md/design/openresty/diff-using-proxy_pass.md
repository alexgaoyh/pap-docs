## 概述

&ensp;&ensp;《记录 Openresty 中 proxy_pass 的一个用法》 本例记录了在 Windows 环境下使用 Openresty 中一个 proxy_pass 的用法。

## 实现

&ensp;&ensp; 详见如下文件： nginx.conf。

&ensp;&ensp; 可以通过在浏览器端设置一个特定的cookie，之后如果遇到这个cookie，则跳转到特定的后端服务。

&ensp;&ensp; 如下配置为当cookie中设置了 special_service=true 的话，跳转到一个后端服务，否则都是其他的后端服务。

```html
	upstream cookie_backend1 {
		server 127.0.0.1:8080;
	}
	upstream cookie_backend2 {
		server 192.168.1.180:8080;
	}

	server {
		listen 9868;
		server_name 127.0.0.1;
		
		location / {
            set_by_lua_block $proxy_upstream {
				# cookie 的名称为 special_service
                local cookie_value = ngx.var.cookie_special_service
                if cookie_value == "true" then
                    return "cookie_backend1"
                else
                    return "cookie_backend2"
                end
            }
			
			proxy_pass http://$proxy_upstream;
			
			# 关键：设置正确的头部传递真实 IP
			proxy_set_header Host $host;
			proxy_set_header X-Real-IP $remote_addr;
			proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
			proxy_set_header X-Forwarded-Proto $scheme;
			proxy_set_header X-Forwarded-Host $host;
			proxy_set_header X-Forwarded-Port $server_port;
		}
	}
```

## 参考

1. http://pap-docs.pap.net.cn/
2. https://gitee.com/alexgaoyh/
