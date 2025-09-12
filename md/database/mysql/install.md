# MySQL 安装指南

## 系统架构检查

首先检查系统架构，以确定需要下载的 MySQL 版本：
    
```shell
uname -m
```

如果是 x86_64 → 需要下载 mysql-x.x.x-linux-glibc2.12-x86_64.tar.gz

如果是 aarch64 → 需要下载 mysql-x.x.x-linux-glibc2.17-aarch64.tar.gz

## 安装步骤

1. 上传 tar.gz 文件到 /mnt 文件夹并解压：

```shell
cd /mnt
sudo tar -xvf mysql-8.3.0-linux-glibc2.28-aarch64.tar.xz
sudo mv mysql-8.3.0-linux-glibc2.28-aarch64 mysql
```

2. 创建必要的目录

```shell
sudo mkdir -p /mnt/mysql/data
sudo mkdir -p /mnt/mysql/logs
sudo mkdir -p /mnt/mysql/tmp
sudo mkdir -p /mnt/mysql/conf
```

3. 设置目录权限

```shell
sudo chown -R mysql:mysql /mnt/mysql
sudo chmod -R 755 /mnt/mysql
```
4. 安装依赖包

```shell
sudo apt update
sudo apt install libaio1 libncurses5 libtinfo5 libnuma1
```

5. 创建 MySQL 用户和组

```shell
sudo groupadd mysql
sudo useradd -r -g mysql -s /bin/false mysql
```

6. 创建配置文件 /mnt/mysql/conf/my.cnf：

```shell
sudo vim /mnt/mysql/conf/my.cnf
```
```html
    [mysqld]
    # 基础设置
    basedir=/mnt/mysql
    datadir=/mnt/mysql/data
    tmpdir=/mnt/mysql/tmp
    log-error=/mnt/mysql/logs/mysql-error.log
    slow_query_log_file=/mnt/mysql/logs/mysql-slow.log
    
    # 网络设置
    port=3306
    socket=/mnt/mysql/mysql.sock
    
    # 安全设置
    user=mysql
    
    # 内存设置
    innodb_buffer_pool_size=128M
    
    [client]
    socket=/mnt/mysql/mysql.sock
    
    [mysql]
    socket=/mnt/mysql/mysql.sock
```

7. 初始化 MySQL

```shell
cd mysql
sudo -u mysql bin/mysqld --defaults-file=/mnt/mysql/conf/my.cnf --initialize --user=mysql --basedir=/mnt/mysql --datadir=/mnt/mysql/data
```

8. 查看临时密码：

```shell
sudo cat /mnt/mysql/logs/mysql-error.log
```
查找类似以下内容：A temporary password is generated for root@localhost: 9q>XwNrkd3w)

9. 启动 MySQL

```shell
# 方式1：使用 mysqld_safe 启动（推荐）
cd /mnt/mysql
sudo -u mysql bin/mysqld_safe --defaults-file=/mnt/mysql/conf/my.cnf &

# 方式2：使用 mysqld 直接启动
cd /mnt/mysql
sudo -u mysql bin/mysqld --defaults-file=/mnt/mysql/conf/my.cnf &
```

10. 检查 MySQL 状态

```shell
# 检查进程是否运行：
ps aux | grep mysql
# 检查端口是否监听：
netstat -ln | grep 3306
# 或者使用
sudo lsof -i :3306
```

11. 停止 MySQL

```shell
# 方式1：使用 mysqladmin 停止（推荐）
cd /mnt/mysql
sudo bin/mysqladmin -u root -p shutdown --socket=/mnt/mysql/mysql.sock
# 方式2：查找进程ID并kill
sudo pkill -f mysqld
# 方式3：强制停止所有MySQL进程
sudo killall mysqld
```

12. 初始配置

```shell
# 登录 MySQL
bin/mysql -u root -p --socket=/mnt/mysql/mysql.sock
# 输入临时密码后进入 MySQL 命令行。

# 修改 root 密码和创建用户
# 修改 root 密码
ALTER USER 'root'@'localhost' IDENTIFIED BY '1q2w!Q@W';
FLUSH PRIVILEGES;

# 创建新用户
CREATE USER 'alexgaoyh'@'%' IDENTIFIED BY '1q2w!Q@W';

# 授予所有权限（生产环境建议按需授权）
GRANT ALL PRIVILEGES ON *.* TO 'alexgaoyh'@'%' WITH GRANT OPTION;

# 刷新权限
FLUSH PRIVILEGES;
```