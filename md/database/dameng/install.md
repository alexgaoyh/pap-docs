# 达梦 安装指南

## 安装步骤

1. 上传 zip 文件 挂载 ISO 文件 ：

```shell
# 创建挂载点
sudo mkdir /mnt/dm8

# 挂载 ISO 文件
sudo mount -o loop dm8_20250506_x86_rh7_64.iso /mnt/dm8
```

2. 创建安装用户和组

```shell
# 创建用户组
sudo groupadd dinstall

# 创建用户
sudo useradd -g dinstall -m -d /home/dmdba -s /bin/bash dmdba

# 设置密码 1q2w!Q@W
sudo passwd dmdba
```

3. 创建安装目录

```shell
# 创建安装目录
sudo mkdir /opt/dm8
sudo chown dmdba:dinstall /opt/dm8
sudo chmod 755 /opt/dm8
```

4. 设置环境限制

```shell
# 编辑 limits.conf
sudo vim /etc/security/limits.conf

# 添加以下内容：
dmdba soft nofile 65536
dmdba hard nofile 65536
dmdba soft nproc 65536
dmdba hard nproc 65536
```

4. 执行安装

```shell
# 切换用户
sudo su - dmdba
# 设置环境变量
export DM_INSTALL_TMPDIR=/tmp
export DM_INSTALL_PATH=/opt/dm8
# 运行安装程序
cd /mnt/dm8
./DMInstall.bin -i
```

5. 命令行安装交互

```shell
是否输入Key文件路径? (Y/y:是 N/n:否) [Y/y]: n
是否设置时区? (Y/y:是 N/n:否) [Y/y]: y
请设置时区 [21]: 21
安装类型 (1:典型 2:服务器 3:客户端 4:自定义) [1]: 1
安装路径 [/home/dmdba/dmdbms/]: /home/dmdba/dmdbms
确认安装? (Y/y:是 N/n:否): y
```

6. 启动数据库服务

```shell
# 注册服务（需要 root 权限）
sudo /home/dmdba/dmdbms/script/root/root_installer.sh
# 如果数据目录为空或不存在，需要初始化
cd /home/dmdba/dmdbms/bin
./dminit path=/opt/dm8/data db_name=DAMENG instance_name=DMSERVER port_num=5236 SYSDBA_PWD=Dameng123 SYSAUDITOR_PWD=Dameng123
# 初始化成功后的验证 检查数据文件是否生成 应该看到以下文件： dm.ini, dm.ctl, SYSTEM.dbf, ROLL.dbf, MAIN.dbf 等
ls -la /opt/dm8/data/DAMENG/
# 直接启动数据库
./dmserver /opt/dm8/data/DAMENG/dm.ini
# 启动服务  DmServiceDMSERVER
#sudo systemctl start DmAPService
# 查看服务状态  DmServiceDMSERVER
#sudo systemctl status DmAPService
# 连接测试
# 使用 disql 连接
cd /home/dmdba/dmdbms/bin
./disql SYSDBA/Dameng123@127.0.0.1:5236
# 环境变量配置
export DM_HOME=/home/dmdba/dmdbms/
export PATH=$DM_HOME/bin:$DM_HOME/tool:$PATH
export LD_LIBRARY_PATH=$DM_HOME/bin:$DM_HOME/lib:$LD_LIBRARY_PATH
```

6. 创建用户和赋权

```sql
-- 创建用户
CREATE USER dmtest1 IDENTIFIED BY "Dameng123";
-- 授予创建会话（连接数据库）的权限
GRANT CREATE SESSION TO dmtest1;

-- 授予在自身模式（dmtest1）下创建表的权限
GRANT CREATE TABLE TO dmtest1;

-- 授予在自身模式下创建视图、索引、存储过程等对象的权限（按需授予）
GRANT CREATE VIEW, CREATE INDEX, CREATE PROCEDURE TO dmtest1;

-- 非常重要：授予该用户对其模式下的表空间配额
-- 这里假设使用 MAIN 表空间，允许无限制使用
ALTER USER dmtest1 QUOTA UNLIMITED ON MAIN;
```
