---
title: "Minecraft Deployment"
description: "Deploy Minecraft Server on ubuntu"
pubDate: "Jul 08 2022"
# heroImage: "../../assets/blog-placeholder-3.jpg"
---

# Minecraft

使用MCSManager管理安装MC

## 安装MCSManager

### 下载MCSManager

首先下载MCSManager。MCSManager可以开启并管理多个实例。

参照 [Docs](https://docs.mcsmanager.com/zh_cn/)安装。

使用 root 账户运行

    sudo su -c "wget -qO- https://script.mcsmanager.com/setup_cn.sh | bash"

接着在防火墙中放行23333和24444两个默认端口，25565为MC java默认端口，27015为L4D2默认端口。

    sudo ufw status

    # 如果显示未启用，可以启用
    # 只要有一个防火墙工作就可以，无论是云服务器管理还是服务器本地的
    sudo ufw enable
    # 添加SSH端口
    sudo ufw allow OpenSSH
    # 或者指定端口号
    sudo ufw allow 22/tcp

    sudo ufw allow 23333/tcp
    sudo ufw allow 24444/tcp
    sudo ufw allow 25565/tcp
    sudo ufw allow 27015/udp

在浏览器中输入 服务器公网ip:23333即可进入MCSManager面板控制台。

## 搭建MC服务器

应用实例-新建应用-MC Java服务端，上传服务端，以paper为例，在[paper](https://papermc.io/downloads)下载得到对应版本的jar文件，上传文件，

启动命令填写

    java -Dfile.encoding=UTF-8 -jar "刚刚下载的jar文件名，例如：paper-1.19.4-516.jar"

不想加环境变量可以填写

    /usr/local/java/jdk-21.0.5/bin/java -Dfile.encoding=UTF-8 -Xms1024m -Xmx2560m -jar paper-1.21.1-131.jar

### 安装java

在ubuntu中使用

    sudo apt-get install openjdk-21-jdk

Ubuntu可以使用apt-get安装，CentOS则太老，需要手动安装。在[Java](https://www.oracle.com/downloads/#category-java)下载对应的java文件得到一个压缩包，将压缩包解压到/usr/local/java下，现在应该有/usr/local/java/jdk-21这样的路径（版本号可能不同）.

修改/etc/profile，添加

    JAVA_HOME=/usr/local/java/jdk-21
    CLASSPATH=.:$JAVA_HOME/lib.tools.jar
    PATH=$JAVA_HOME/bin:$PATH
    export JAVA_HOME CLASSPATH PATH

，运行`source /etc/profile`生效配置，运行`java -version`，有版本信息输出则说明成功安装java。

### Start

第一次运行后修改eula.txt同意协议，可以通过MCSManager的服务端配置文件直接修改，也可以在文件路径下找到用vim修改。修改server.properties，配置正版验证、白名单一类的。

再次启动，等待服务端开启。

进入游戏后可以在服务端输入 op 游戏id给予自己管理员权限。
