## 概述

&ensp;&ensp;《思考 Openresty 上传与进度》 本例记录了在 Windows 环境下使用 Openresty 完成了资源上传与进度的处理逻辑。

## 背景

&ensp;&ensp; 很多年前使用 [nginx-upload-module](https://github.com/fdintino/nginx-upload-module) 处理过文件上传，但是仓库已经很久没有更新了，为了做对比实现，尝试了一下 Openresty 并进行如下记录。

## 实现

&ensp;&ensp; 详见如下文件： nginx.conf、 upload.lua、 upload_progress.lua、 upload.html

```html
http {
	
    lua_shared_dict upload_progress 10m;

	client_body_buffer_size 8k;
	client_max_body_size 2048m;
			
    server {
        listen       80;
        server_name  localhost;

        location / {
            root   html;
            index  index.html index.htm;
        }
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
        location /upload {
			add_header 'Access-Control-Allow-Origin' '*';
			add_header 'Access-Control-Allow-Methods' 'GET, POST, OPTIONS';
			add_header 'Access-Control-Allow-Headers' 'Origin, Content-Type, Accept, Content-Length, Content-Disposition, X-File-Size';
			add_header 'Access-Control-Allow-Credentials' 'true';
			proxy_set_header X-Content-Length $content_length;
            content_by_lua_file conf/lua/upload.lua;
        }
		location /progress {
			add_header 'Access-Control-Allow-Origin' '*';
			add_header 'Access-Control-Allow-Methods' 'GET, POST, OPTIONS';
			add_header 'Access-Control-Allow-Headers' 'Origin, Content-Type, Accept, Content-Length, Content-Disposition, X-File-Size';
			add_header 'Access-Control-Allow-Credentials' 'true';
            content_by_lua_file conf/lua/upload_progress.lua;
        }
		location /preview/ {
            alias "D:/knowledge/";
            autoindex on;
            try_files $uri $uri/ =404;
        }
    }

}
```

```html
local upload = require "resty.upload"
local cjson = require "cjson"
local upload_progress = ngx.shared.upload_progress

local chunk_size = 16384000  -- 每次读取的块大小
local upload_root_path = "D:/knowledge/"

local function get_filename_extension(res)
    local filename = res:match('filename="(.-)"$')
    local extension = filename:match("^.+(%..+)$")
    return extension
end

local function save_file(filepath, content)
    local file = io.open(filepath, "wb")
    if not file then
        return nil, "Failed to open file for writing"
    end

    file:write(content)
    file:close()
    return true
end

local function to_hex(str)
    return (str:gsub('.', function (c)
        return string.format('%02x', string.byte(c))
    end))
end

local function process_upload()
    local form, err = upload:new(chunk_size)
    if not form then
        ngx.say(cjson.encode({ success = false, error = err }))
        return
    end

    local file_content = {}
    local filename
    local extension
    local total_size = tonumber(ngx.var.http_content_length)
    local uploaded_size = 0
    local upload_id = ngx.var.arg_upload_id
	

    if not upload_id then
        ngx.say(cjson.encode({ success = false, error = "Missing upload_id parameter" }))
        return
    end

    form:set_timeout(3000)  -- 3秒超时

    while true do
        local typ, res, err = form:read()
        if not typ then
            ngx.say(cjson.encode({ success = false, error = err }))
            return
        end

        if typ == "header" then
            if res[1] == "Content-Disposition" then
                extension = get_filename_extension(res[2])
            end

        elseif typ == "body" then
            table.insert(file_content, res)
            uploaded_size = uploaded_size + #res

            if total_size > 0 then
                local progress = (uploaded_size / total_size) * 100
                if progress > 100 then
                    progress = 100
                end
                upload_progress:set(upload_id, progress)
            end

        elseif typ == "part_end" then
            if extension then
                local timestamp = tostring(os.time())
                local date = os.date("*t")
                local date_path = string.format("%04d%02d%02d", date.year, date.month, date.day)
                filename = timestamp .. (extension or "")
                os.execute("mkdir \"" .. upload_root_path .. "/" .. date_path .. "\"")
                local filepath = upload_root_path .. "/" .. date_path .. "/" .. filename
                local full_content = table.concat(file_content)
                local ok, err = save_file(filepath, full_content)
                if not ok then
                    ngx.say(cjson.encode({ success = false, error = err }))
                    return
                end
                upload_progress:delete(upload_id)
				
                local md5_hash = to_hex(ngx.md5_bin(full_content))
                ngx.say(cjson.encode({ success = true, upload_id = upload_id, filename = filename, path = date_path, md5 = md5_hash }))
                return
            end

        elseif typ == "eof" then
            break
        end
    end
end

process_upload()

```

```html
local cjson = require "cjson"
local upload_progress = ngx.shared.upload_progress

local function get_upload_progress()
    local upload_id = ngx.var.arg_upload_id
    if not upload_id then
        ngx.say(cjson.encode({ success = false, error = "Missing upload_id parameter" }))
        return
    end

    local progress = upload_progress:get(upload_id)
    if not progress then
        ngx.say(cjson.encode({ success = false, error = "No upload progress found for the given upload_id" }))
        return
    end

    if progress ~= progress or progress == math.huge or progress == -math.huge then
        ngx.say(cjson.encode({ success = false, error = "Invalid progress value" }))
        return
    end

    ngx.say(cjson.encode({ success = true, upload_id = upload_id, progress = progress }))
end

get_upload_progress()

```


```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>File Upload</title>
</head>
<body>
    <input type="file" id="fileInput" />
    <button onclick="uploadFile()">Upload</button>
    <script>
        function uploadFile() {
            var fileInput = document.getElementById('fileInput');
            var file = fileInput.files[0];
            var uploadId = "123456";  // 可以动态生成或从其他地方获取的唯一ID
            var url = 'http://localhost/upload?upload_id=' + uploadId;
            var xhr = new XMLHttpRequest();
            var formData = new FormData();
            formData.append('file', file);
            xhr.open('POST', url, true);
			// 设置请求头，传递文件大小
			xhr.setRequestHeader('X-File-Size', file.size);
            xhr.upload.onprogress = function (e) {
                if (e.lengthComputable) {
                    var percentComplete = (e.loaded / e.total) * 100;
                    console.log('Progress: ' + percentComplete.toFixed(2) + '%');
                }
            };
            xhr.onload = function () {
                if (xhr.status === 200) {
                    console.log('Upload complete: ', xhr.responseText);
                } else {
                    console.error('Upload failed: ', xhr.statusText);
                }
            };
            xhr.send(formData);
        }
    </script>
</body>
</html>
```
