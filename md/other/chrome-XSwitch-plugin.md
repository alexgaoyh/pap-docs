# [杂谈]谷歌浏览器 XSwitch 插件 - 解决本地请求转发

## 介绍

&ensp;&ensp;软件行业从业者在日常工作过程中，经常会遇到请求转发的问题，本文介绍的 [XSwitch](https://github.com/yize/xswitch/releases) 是一款应用于 Chrome 浏览器的请求转发插件。

&ensp;&ensp;对这款插件进行总结就是 ： 将指定的特定 url 重定向 到你想定向的 url。

## 应用场景

1. 将 JS/CSS/JPG 等资源转发到本地进处理。
   1. 在进行异地办公/远程办公的过程中，办公地点A的开发人员基于当前的网络环境发送了向 192.168.20.* 的网络请求, 但是办公地点B并不能正常请求 192.168.20.*，这时就可以使用 XSwitch 进行请求转发。
   2. 在调试过程中，需要将线上环境的脚本或样式转发到本地进行DEBUG，这时候就可以用到 XSwitch 插件，将 *.js/*.css 的请求在本地进行调试。

## 使用方法

1. 安装： 谷歌商店搜索安装，无法访问谷歌商店的话，可以从如下链接获取 [XSwitch](https://github.com/yize/xswitch/releases)。
2. 规则配置：在安装之后，可以进行相关的代码配置，如下代码段落是将所有 192.168.0.26:5555 下的请求转发到 127.0.0.1:5555 下。

```json
{
  "proxy": [
      [
        "http://192.168.0.26:5555/(.*)",
        "http://127.0.0.1:5555/$1",
      ]
  ]
}
```

## 截图示例 

1. 未开启 XSwitch，这个时候可以发现浏览器在访问 192.168.0.26 下的请求过程中，并不能正常获取静态资源。
2. 图片无法正常查看的话， [访问](https://gitee.com/alexgaoyh/pap-docs/blob/master/md/other/img/un-open-XSwitch.png)
   ![未开启 XSwitch](https://s2.loli.net/2023/04/20/76yPSDzmxqCnLQB.png)

2. 开启 XSwitch， 这个时候发现浏览器在访问 192.168.0.26 下的请求过程中，会根据配置，自动将请求转发到 127.0.0.1 下，从浏览器的地址栏也看到请求地址发生了变化。
2. 图片无法正常查看的话， [访问](https://gitee.com/alexgaoyh/pap-docs/blob/master/md/other/img/open-XSwitch.png)
   ![开启 XSwitch](https://s2.loli.net/2023/04/20/o36TW1fhdkRcFvi.png)

## 总结

&ensp;&ensp;作者近期由于 VPN 无法正常运行，在进行代码编写的过程中，发现部分网络请求无法正常访问，故此使用此插件将请求转发到本地进行代码编写与调试。

## 参考

1. http://pap-docs.pap.net.cn/
2. https://gitee.com/alexgaoyh/
