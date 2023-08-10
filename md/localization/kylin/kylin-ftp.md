# [国产化-银河麒麟v10桌面版]FTP适配(FtpClient)

## 介绍

&ensp;&ensp;作为一个码农，近期看到很多关于国产化的新闻，便使用虚拟机装了一台银河麒麟v10桌面版操作系统(Kylin-Desktop-V10-SP1-Release-2107-x86_64)，并计划对目前经常使用的基础组件(ftp redis db ……)进行适配，本文主要针对 FTP 服务。

## 安装与配置

```shell
sudo apt install vsftpd
sudo apt install vim

sudo vim /etc/vsftpd.conf 

sudo useradd -d /home/ftpuser ftpuser 	
sudo passwd ftpuser

sudo chown ftpuser:ftpuser -R /home/ftpuser/

sudo systemctl daemon-reload
sudo systemctl restart vsftpd.service
```

```html
## 修改 sudo vim /etc/vsftpd.conf 文件，添加写入权限，设置编码，设置被动模式和端口范围
write_enable=YES

utf8_filesystem=YES

pasv_enable=YES
pasv_min_port=30000
pasv_max_port=30005
```

&ensp;&ensp;在进行安装配置完之后，作者最开始使用命令行的方式开放端口(ftp对应的21端口、被动端口30000-30005)，但是不可用，最终是通过桌面环境进行的端口开放(开启防火墙 麒麟操作系统任务栏左下角开始 - 点击设置 - 选择安全与更新 - 安全中心 - 网络保护 -  防火墙 - 自定义(添加ftp的21端口，被动链接的 30001-30005 的5个端口))


## 常见问题

### 阿里云

&ensp;&ensp;作者同时在阿里云上租用了一台ECS，并选择'云市场三方-银河麒麟高级服务器操作系统（X86版）V10'作为镜像。

&ensp;&ensp;与上述文档不同之处在于：
1. 使用 yum 而不是 apt 命令进行安装(原因未知，怀疑是三方镜像问题)；
2. 阿里云上/etc/vsftpd.conf 文件的写入权限默认是开启的，并且未配置编码和被动模式；

### FtpClient(commons-net)

&ensp;&ensp;作者在使用java语言进行FTP测试的时候，出现无法查看目录、文件等问题

&ensp;&ensp;解决方案如下，作者同时使用了如下两种方案：
1. 按照如上方式修改配置文件，设置被动模式和端口范围，并开放对应端口范围；
2. 互联网有一种方案是修改时区，sudo vim /lib/systemd/system/vsftpd.service   在 [Service] 下面添加环境变量 Environment=LC_TIME=en_US.UTF-8

## 总结

&ensp;&ensp;不同操作系统下配置 ftp 之后，遇到的情况有细节的偏差，比如ubuntu下没有显示配置被动模式和端口也能正常使用，然而本文使用的银河麒麟v10桌面版操作系统却需要进行配置。

## 参考

1. https://blog.csdn.net/clear4521/article/details/130846360
2. https://gitee.com/alexgaoyh/pap-bean-ftp-starter/blob/master/ReadMe-kylin.MD
3. https://pap-docs.pap.net.cn/
