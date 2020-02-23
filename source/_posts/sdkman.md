---
title: 使用 SDKMAN 管理开发环境.
date: 2019-12-23 13:59:58
tags: tools
---

# 简介
最近需要在 Mac 上同时使用两个不同的开发环境. 配置起来还比较麻烦. 使用 SDKMAN 则可以比较方便的管理.
* 官网: https://sdkman.io/

# 安装 SDKMAN
安装只需要运行一下命令:
```shell
curl -s "https://get.sdkman.io" | bash
```

# 搜索和安装 JDK
跟常见的包管理工具类似, 使用 SDKMAN 安装只需要使用 install 命令, 运行下面的命令就可以安装最新版本的 
jdk.
```shell
sdk install java
```

需要搜索的时则可以使用 list 命令. 比如列出可用的 java 版本:
```shell
sdk list java
```

结果如下:
![](/images/sdk_list_java.png)

安装指定的版本则需要在工具名后面添加 Identifier. 比如安装 jdk 8 需要用一下命令:
```shell
sdk install java 8.0.232.j9-adpt
```

# 切换 jdk 的版本

安装完之后可以利用 use 命令切换不同的版本, 比如切换到 jdk8:
```shell
sdk use java 8.0.232.j9-adpt
```

也可以使用 default 命令将 jdk8 设为默认的版本:
```shell
sdk default java 8.0.232.j9-adpt
```

使用 current 命令则可以查看当前版本:
```shell
sdk current java
```












