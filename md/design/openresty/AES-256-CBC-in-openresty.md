## 概述

&ensp;&ensp;《记录 Openresty 中 AES-256-CBC 的使用》 本例记录了在 Windows 环境下使用 Openresty 实现 AES-256-CBC 的功能。

## 背景

&ensp;&ensp; 当前仅仅是脚本的记录，用来对字符串进行加解密操作，之所以选择这种方式，是因为其他编程语言也可以达到相同的效果。

## 实现

&ensp;&ensp; 详见如下文件： nginx.conf。

&ensp;&ensp; 同时新增了 crypto.lua 的脚本。

```html
server {
    listen 9878;
    server_name  127.0.0.1;

    # 加密接口: 127.0.0.1:9878/encrypt?text=alexgaoyh
    location /encrypt {
        content_by_lua_block {
            local mycrypto = require "conf/lua/crypto"
            local cjson = require "cjson.safe"

            local args = ngx.req.get_uri_args()
            local text = args.text or "default text"

            local encrypted = mycrypto.encrypt(text)

            ngx.header.content_type = "application/json; charset=utf-8"
            ngx.say(cjson.encode({
                success = true,
                action  = "encrypt",
                input   = text,
                output  = encrypted
            }))
        }
    }

    # 解密接口: 127.0.0.1:9878/decrypt?data=LOteFgO22MXi7b0YhmlG9Q==
    location /decrypt {
        content_by_lua_block {
            local mycrypto = require "conf/lua/crypto"
            local cjson = require "cjson.safe"

            local args = ngx.req.get_uri_args()
            local data = args.data

            if not data then
                ngx.header.content_type = "application/json; charset=utf-8"
                ngx.say(cjson.encode({
                    success = false,
                    error   = "缺少参数: ?data=xxx(base64)"
                }))
                return
            end

            local decrypted, err = mycrypto.decrypt(data)
            ngx.header.content_type = "application/json; charset=utf-8"

            if not decrypted then
                ngx.say(cjson.encode({
                    success = false,
                    action  = "decrypt",
                    input   = data,
                    error   = err
                }))
                return
            end

            ngx.say(cjson.encode({
                success = true,
                action  = "decrypt",
                input   = data,
                output  = decrypted
            }))
        }
    }
}
```

```html
-- 文件: conf/lua/crypto.lua
local aes = require "resty.aes"
local str = require "resty.string"

local _M = {}

-- AES 配置
local key = "1234567890abcdef1234567890abcdef"  -- 32字节
local iv  = "abcdef1357924680"                 -- 16字节

local aes_256_cbc = assert(aes:new(
    key, nil, aes.cipher(256, "cbc"), { iv = iv }
))

-- 加密方法
function _M.encrypt(data)
    local encrypted = aes_256_cbc:encrypt(data)
    return ngx.encode_base64(encrypted)
end

-- 解密方法
function _M.decrypt(base64_str)
    local encrypted = ngx.decode_base64(base64_str)
    if not encrypted then
        return nil, "base64 解码失败"
    end
    local decrypted = aes_256_cbc:decrypt(encrypted)
    if not decrypted then
        return nil, "解密失败"
    end
    return decrypted
end

return _M
```

## 参考

1. http://pap-docs.pap.net.cn/
2. https://gitee.com/alexgaoyh/
