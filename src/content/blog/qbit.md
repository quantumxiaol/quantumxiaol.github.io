---
title: "qBittorrent Deployment"
description: "Deploy qBittorrent on ubuntu"
pubDate: "Jul 08 2022"
heroImage: "../../assets/blog-placeholder-3.jpg"
---

# qBittorrent Deployment

在 Ubuntu 上安装 qBittorrent 并通过网页访问它的 Web UI（用户界面）

## 安装qBittorrent

### 创建用户qbittorrent

    sudo adduser --system --group qbittorrent
    sudo usermod -s /usr/sbin/nologin qbittorrent

之后使用`sudo -u qbittorrent`来以qbittorrent用户的名义执行，这样更加安全。

### 安装

    sudo apt-get update
    sudo apt-get install qbittorrent-nox

### 配置

由于 qbittorrent-nox 安装在全局路径下（例如 /usr/bin/qbittorrent-nox），任何用户都可以执行它，qbittorrent用户需要一些配置来执行命令。

    sudo mkdir -p /home/qbittorrent/.config/qBittorrent/

    sudo chown -R qbittorrent:qbittorrent /home/qbittorrent/.config/qBittorrent/

    sudo -u qbittorrent nano /home/qbittorrent/.config/qBittorrent/qBittorrent.conf

    sudo chown -R qbittorrent:qbittorrent /home/qbittorrent
    sudo chmod -R 700 /home/qbittorrent

配置文件中输入下面内容

    [WebUI]
    Enabled=true
    Host=::
    Port=8080
    Username=admin          # 设置一个用户名
    Password=               # 设置一个强密码，首次启动后应立即修改默认密码

    [Network]
    UseProxy=false          # 如果你需要使用HTTP代理，请设置为true
    ProxyType=0             # 0 = HTTP, 1 = SOCKS5
    ProxyIP=127.0.0.1
    ProxyPort=7890           # 根据你的代理服务器端口调整
    ProxyPeerConnections=true
    ProxyAuthEnabled=false   # 如果代理需要认证，请设置为true
    ProxyUsername=           # 代理认证用户名
    ProxyPassword=           # 代理认证密码

    [Preferences]
    DefaultSavePath=/home/qbittorrent/Downloads/
    AutoTMMState=true
    TempPath=/home/qbittorrent/Torrents/
    ExportDir=/home/qbittorrent/Completed/
    export_dir_root=/home/qbittorrent/Completed/
    ScanDirs=@Invalid()

如果手动写明文密码，qBittorrent 会忽略它，仍然使用默认密码 adminadmin，设置Proxy才需要填入上面的conf，一般情况下可以选择初始化时默认创建的东西。

创建并给予这些权限

    sudo mkdir -p /home/qbittorrent/{Downloads,Torrents,Completed}
    sudo chown -R qbittorrent:qbittorrent /home/qbittorrent/{Downloads,Torrents,Completed}

### 创建服务

    sudo nano /etc/systemd/system/qbittorrent.service

添加

    [Unit]
    Description=qBittorrent Daemon
    After=network.target

    [Service]
    User=qbittorrent
    Group=qbittorrent
    Type=simple
    ExecStart=/usr/bin/qbittorrent-nox --profile=/home/qbittorrent/.config/qBittorrent
    Restart=on-failure
    RestartSec=5s
    TimeoutStopSec=20s
    WorkingDirectory=/home/qbittorrent/
    Environment=HOME=/home/qbittorrent
    Environment=LANG=en_US.UTF-8
    Environment=LC_ALL=en_US.UTF-8
    Environment=XDG_CONFIG_HOME=/home/qbittorrent/.config


    [Install]
    WantedBy=multi-user.target

启动服务

    sudo systemctl daemon-reload
    sudo systemctl enable qbittorrent
    sudo systemctl start qbittorrent

    sudo systemctl stop qbittorrent

检查服务状态：

    sudo systemctl status qbittorrent

重置密码

    sudo systemctl stop qbittorrent
    sudo mv /home/qbittorrent/.config/qBittorrent/qBittorrent.conf /home/qbittorrent/.config/qBittorrent/qBittorrent.conf.bak
    sudo -u qbittorrent qbittorrent-nox

重新初始化