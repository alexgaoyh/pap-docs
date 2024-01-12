## 背景

&ensp;&ensp;2016-2017年的时候，使用JavaFX做过客户端，最近需要在此基础上，打出来一个安装程序，本文不牵扯任何技术，只做记录。

## 实现效果示例

&ensp;&ensp;分为两步，第一步是先将程序转为可执行EXE，但是这一步仍需要单独安装Java环境，第二步是将EXE转为安装程序，将JRE打进安装程序，不需要再单独安装JAVA环境。

&ensp;&ensp;如果安装程序需要添加开机启动的配置，如下：

```html
[Tasks]
Name: "startupicon"; Description: "When the system starts up automatically run this program"; GroupDescription: "{cm:AdditionalIcons}"; Flags: unchecked

[Icons]
Name: "{autostartup}\{#MyAppName}"; Filename: "{app}\{#MyAppExeName}"; Tasks: startupicon
```

![spring-boot-fatjar-to-exe-min.jpg](https://s2.loli.net/2024/01/12/pZh9NbUxVFcqjQ3.jpg)
![java-project-exe-to-install-program-min.jpg](https://s2.loli.net/2024/01/12/k1s36GwXrbVnLvP.jpg)

&ensp;&ensp;图像备份: [访问](https://gitee.com/alexgaoyh/pap-docs/blob/master/md/other/image/javafx)

## 参考
1. http://pap-docs.pap.net.cn/
2. https://gitee.com/alexgaoyh
