# Nginx 安装指南

## 安装步骤

1. 更新系统并安装 Nginx

```shell
sudo apt update
sudo apt install nginx
```

2. 启动并启用 Nginx 服务

```shell
sudo systemctl start nginx
sudo systemctl enable nginx
sudo systemctl status nginx
# 停止
sudo systemctl stop nginx
```

3. 主配置文件

```shell
/etc/nginx/nginx.conf
# 可以使用 sudo nginx -t 来检查配置文件的路径
# nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
# nginx: configuration file /etc/nginx/nginx.conf test is successful
```