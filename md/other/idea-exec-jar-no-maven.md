# [杂谈]IDE-idea-可执行JAR项目创建

## 介绍

&ensp;&ensp;近期总结了前段时间的一个桌面应用开发的场景，想到多年前在没有Maven的时候，也同样进行过开发，想回过头来对比一下，故编写此文，使用 IDEA 这个 IDE 创建一个简单的项目，并打包成可执行JAR，本例无技术难点，仅做一个记录。

## 创建步骤

1. 使用IDEA新建一个项目，选择 New Project，输入项目名
2. 在左侧项目名称上右键选择 Project Structure，在 Artifacts 页签执行新增，选择Add-JAR-From modules with dependencies……
3. 选中对应的 Main Class，本例由于未对新创建的项目做更改，则选择了 Main.java
4. 在左侧项目名称上右键选择 Rebuild Module 'pap-jar'，就会生成 out 文件夹，里面会输出对应的编译后文件；
5. 菜单栏选择Build - Build Artifacts……，之后在弹出的 Build Artifact 上执行 JAR 文件生成；
6. 最终便生成了可执行pap-jar.jar 文件

## 截图示例

&ensp;&ensp;安装上一章节'创建步骤'所述，截图如下所示。

![idea-simple-project.jpg](https://s2.loli.net/2024/01/28/LzpvkPHAJxQDi62.jpg)

&ensp;&ensp;图像备份: [访问](https://gitee.com/alexgaoyh/pap-docs/blob/master/md/other/img/idea-simple-project.jpg)
    
## 参考
1. http://pap-docs.pap.net.cn/
2. https://gitee.com/alexgaoyh
