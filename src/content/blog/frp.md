---
title: "FRP Deployment"
description: "Deploy FRP Server and FRP Client"
pubDate: "Jul 08 2025"
heroImage: "../../assets/blog-placeholder-3.jpg"
---

# Frp Deployment

使用一台带公网的服务器作为frp的ubuntu server，自己的性能比较高的电脑安装ubuntu作为本地服务器。

使用阿里云进行内网穿透是有一定风险的，购买云服务器时指明了不能将服务器用作内网穿透服务器，但是不要有特别大的公网流量天天跑满应该问题不大。

[GoFrp](https://gofrp.org/)

[GoFrp Docs](https://gofrp.org/zh-cn/docs/)

[参考](https://blog.csdn.net/aakk007/article/details/147692482)

## Server

### 下载
下载地址：[GitHub](https://github.com/fatedier/frp/releases)

使用的是Linux Ubuntu2204，目前最新的为[frp_0.63.0_linux_amd64](https://github.com/fatedier/frp/releases/download/v0.63.0/frp_0.63.0_linux_amd64.tar.gz)的版本。

可以本地下载再上传服务器。如果服务器本身可以下载，使用

    wget https://github.com/fatedier/frp/releases/download/v0.63.0/frp_0.63.0_linux_amd64.tar.gz

### 解压安装

安装在/usr/local和/home/username的区别在于，/usr/local下的文件是系统级文件，/home/username下的文件是普通用户文件。

在/usr/local下创建frp目录

    mkdir /usr/local/frp

将下载的frp_0.63.0_linux_amd64.tar.gz解压到/usr/local/frp目录下

    tar -zxvf frp_0.63.0_linux_amd64.tar.gz -C /usr/local/frp

### 配置

进入安装目录

    cd /usr/local/frp/frp_0.63.0_linux_amd64

编辑frps配置文件 

    vi frps.toml
    # or
    nano frps.toml

添加下面内容

    # 客户端与服务连接端口 
    bindPort = 7000 
    # 客户端连接服务端时认证的密码 
    auth.token = "abcjc"     #自行修改为自己的token
    # http协议监听端口 
    vhostHTTPPort = 28080 
    # web界面配置 
    webServer.addr = "0.0.0.0" 
    webServer.port = 7500 
    webServer.user = "admin" 
    webServer.password = "admin"    #修改为自己的密码

vi 使用命令i进入编辑模式，Esc退出编辑模式，:wq保存并退出
nano 使用ctrl+O保存写入，ctrl+X保存并退出

之后查看账号密码可以

    nano /usr/local/frp/frp_0.63.0_linux_amd64/frps.toml

这些端口需要在云服务器管理的防火墙、本地的防火墙放行。

    firewall-cmd --permanent --add-port=7000/tcp
    firewall-cmd --permanent --add-port=7500/tcp
    firewall-cmd --permanent --add-port=28080/tcp
    firewall-cmd --reload

### 运行frps服务

创建 frps.service 文件

使用文本编辑器 (如 vi、nano) 在 /etc/systemd/system 目录下创建一个 frps.service 文件，用于配置 frps 服务。

    sudo vi /etc/systemd/system/frps.service
    or
    sudo nano /etc/systemd/system/frps.service

添加以下内容

    [Unit] 
    Description=frp server 
    After=network.target syslog.target 
    Wants=network.target 
    [Service] 
    Type=simple 
    ExecStart=/usr/local/frp/frp_0.63.0_linux_amd64/frps -c /usr/local/frp/frp_0.63.0_linux_amd64/frps.toml 
    [Install] 
    WantedBy=multi-user.target

保存退出启动服务

    sudo systemctl status frps.service

### 检测服务

浏览器进入http://server-public-ip:7500/检测，输入上面配置的账号密码

### FRP server 管理

    # 启动frp 
    sudo systemctl start frps 
    # 停止frp 
    sudo systemctl stop frps 
    # 重启frp 
    sudo systemctl restart frps
    #设置开机自启
    sudo systemctl enable frps

## 客户端

### 下载解压

和服务端用的一个东西，直接下载解压，路径也一样。

### 配置

进入安装目录

    cd /usr/local/frp/frp_0.63.0_linux_amd64

编辑frps配置文件 

    vi frpc.toml
    sudo vi /usr/local/frp/frp_0.63.0_linux_amd64/frpc.toml

    # or
    nano frpc.toml
    sudo nano /usr/local/frp/frp_0.63.0_linux_amd64/frpc.toml


添加内容

    serverAddr = "xxx.xxx.xxx.xxx" #云服务器的IP ""代表字符串
    serverPort = 7000
    
    auth.method = "token"
    auth.token = "token-XXXXXX" #token
    
    
    [[proxies]]
    name = "ssh"       
    type = "tcp"
    localIP = "127.0.0.1"
    localPort = 22
    remotePort = 6000    #ssh工具使用此端口访问

    # 添加其他服务的端口，示例，尽量不要使用默认服务的端口，会被攻击
    # L4D2
    [[proxies]]
    name = "L4D2"
    type = "udp"
    localIP = "127.0.0.1"
    localPort = 27015
    remotePort = 27015
    
    # Minecraft
    [[proxies]]
    name = "Minecraft"
    type = "tcp"
    localIP = "127.0.0.1"
    localPort = 25565
    remotePort = 25565

    # MCSManager
    [[proxies]]
    name = "MCSManager1"
    type = "tcp"
    localIP = "127.0.0.1"
    localPort = 23333
    remotePort = 23333

    # MCSManager
    [[proxies]]
    name = "MCSManager2"
    type = "tcp"
    localIP = "127.0.0.1"
    localPort = 24444
    remotePort = 24444

验证文件是否正确

    /usr/local/frp/frp_0.63.0_linux_amd64/frpc verify -c /usr/local/frp/frp_0.63.0_linux_amd64/frpc.toml

### 启动

    sudo chmod +x /usr/local/frp/frp_0.63.0_linux_amd64/frpc

    sudo chmod +x frpc

    ./frpc -c frpc.toml

    /usr/local/frp/frp_0.63.0_linux_amd64/frpc -c frpc.toml

创建 Systemd 服务文件

    sudo nano /etc/systemd/system/frpc.service

添加下面的路径，和上面的区别在于一个是c，一个是s

    [Unit]
    Description=Frp Client Service
    After=network.target
    
    [Service]
    Type=simple
    User=nobody
    ExecStart=/usr/local/frp/frp_0.63.0_linux_amd64/frpc -c /usr/local/frp/frp_0.63.0_linux_amd64/frpc.toml
    Restart=on-failure
    RestartSec=5s
    
    [Install]
    WantedBy=multi-user.target

### FRP client 管理

    # 启动frp 
    sudo systemctl start frpc
    # 停止frp 
    sudo systemctl stop frpc
    # 重启frp 
    sudo systemctl restart frpc
    # 设置开机自启
    sudo systemctl enable frpc
    # 查看状态
    sudo systemctl status frpc
