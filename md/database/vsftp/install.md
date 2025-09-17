# Vsftp 安装指南

## 安装步骤

1. 更新系统并安装 vsftpd

```shell
sudo apt update
sudo apt install vsftpd
```

2. 启动并启用 vsftpd 服务

```shell
sudo systemctl start vsftpd
sudo systemctl enable vsftpd
sudo systemctl status vsftpd
```

3. 创建 FTP 用户

```shell
sudo useradd -m -d /home/bj -s /bin/bash bj
sudo passwd bj
# 输入并确认密码

```
4. 编辑 vsftpd 配置文件

```shell
sudo vim /etc/vsftpd.conf
```

```shell
# 允许本地用户登录
local_enable=YES

# 启用写入权限
write_enable=YES

# 将本地用户限制在其主目录中（重要安全设置）
chroot_local_user=YES

# 设置本地用户的umask，022意味着新创建的文件权限为644，目录为755
local_umask=022

# 启用被动模式(PASV)并设置端口范围
pasv_enable=YES
pasv_min_port=30000
pasv_max_port=31000

# 强制使用UTF-8文件系统编码
utf8_filesystem=YES

# 可选：解决某些客户端连接问题
allow_writeable_chroot=YES
```

5. 设置目录权限

```shell
sudo chown -R bj:bj /home/bj/
sudo chmod a-w /home/bj/
```

6. 配置防火墙（如果启用）

```shell
sudo ufw allow 20/tcp
sudo ufw allow 21/tcp
sudo ufw allow 30000:31000/tcp
sudo ufw reload
```

7. 重启 vsftpd 服务应用配置

```shell
sudo systemctl restart vsftpd
sudo systemctl status vsftpd
```

8. 权限

```shell
sudo chown -R bj:bj /home/bj
# vsftpd 要求根目录不能有写权限，755 的操作可以忽略
# sudo chmod -R 755 /home/bj

# 现在根目录不能写了，用户无法上传文件。这是为了安全，但我们也需要提供上传功能。标准的做法是在根目录下创建一个拥有写权限的子目录，例如 upload
# sudo mkdir /home/bj/upload
# 将这个子目录的所有权和权限设置为可写
# sudo chown bj:bj /home/bj/upload
# sudo chmod 770 /home/bj/upload # 或者 755，根据你的用户/组需求
```