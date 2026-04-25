---
title: "L4D2 Server Deployment"
description: "在Linux上部署求生之路服务器"
pubDate: "Jul 08 2025"
# heroImage: "../../assets/blog-placeholder-3.jpg"
---

# 安装Steam

`sudo apt install libgl1-mesa-dri:i386 libgl1-mesa-glx:i386`安装32位库
在download下运行`sudo dpkg -i ~/Downloads/steam_latest.deb`
`sudo apt --fix-broken install`修复依赖

这个下载的是Steam本体，可以用来玩

## 安装 server

~对于使用带GUI的ubuntu，安装了Steam可以下载left 4 dead 2 dedicated server~没法运行

## L4D2 Server

参考[bilibili](https://www.bilibili.com/opus/736922474423255104/?from=readlist)

### 安装Steam CMD

安装SteamCMD所需的依赖库

    sudo apt install lib32gcc-s1

添加用户 l4d2，此用户没有密码，不能登陆，只能用来启动l4d2


    adduser l4d2

    su l4d2

    mkdir /home/l4d2/Steam

    sudo wget -O /home/l4d2/Steam/steamcmd_linux.tar.gz https://steamcdn-a.akamaihd.net/client/installer/steamcmd_linux.tar.gz

    sudo tar -xvf /home/l4d2/Steam/steamcmd_linux.tar.gz -C /home/l4d2/Steam

    sudo rm /home/l4d2/Steam/steamcmd_linux.tar.gz

    sudo chown -R l4d2:l4d2 /home/l4d2/Steam

    sudo -u l4d2 -s

    cd /home/l4d2/Steam/

    ./steamcmd.sh

在Steam>后输入命令

    force_install_dir /home/l4d2/Steam/l4d2

    login anonymous

使用个人steam账号（确认此账号已购买L4D2）登录SteamCMD后再下载，原来的 login anonymous 替换为 login abc 123 ，其中 abc 替换为自己的steam账户名，将 123 替换为steam密码。如果有绑定steam手机令牌，接下来会显示 Two-factor code: ，这里填上验证码即可登录，登录后再输入命令 app_update 222860 validate 继续下载安装L4D2服务器

    app_update 222860 validate


### 插件

以下是常用的3个插件，均下载linux版本，注意分辨 “l4d” 和 “l4d2”

[SourceMOD](https://www.sourcemod.net/downloads.php?branch=stable)

[MetaMOD](http://metamodsource.net/downloads.php?branch=stable)

[Tickrate Enabler](https://github.com/accelerator74/Tickrate-Enabler/releases/tag/build)

使用下载

    wget https://mms.alliedmods.net/mmsdrop/1.12/mmsource-1.12.0-git1219-linux.tar.gz

    wget https://sm.alliedmods.net/smdrop/1.12/sourcemod-1.12.0-git7210-linux.tar.gz

解压后得到addons和cfg两个文件夹，将这两个文件夹里的所有东西分别传输到系统/home/l4d2/Steam/l4d2/left4dead2/路径下的addons和cfg

配置插件

    vim /home/l4d2/Steam/l4d2/left4dead2/addons/sourcemod/configs/admins_simple.ini

在文档末端另起一行，写入自己的[steamid](https://steamid.io/lookup)：

    "STEAM_x:x:xxxxxx"  "99:z"

### 配置server.cfg

    cd /home/l4d2/Steam/l4d2/left4dead2/cfg

    vim server.cfg
    nano server.cfg

写入

    rcon_password "" //在引号内填写远程管理密码，引号内不填即为不设密码。使用rcon的两个必要条件：① 在服务器端开放游戏端口的tcp协议 ② 在l4d2服务器启动项里添加一条 +ip 0.0.0.0
    sv_password "" //在引号内填写服务器密码，引号内不填即为不设密码
    sv_allow_lobby_connect_only 0 //不允许从大厅选择组服务器来连接
    sv_tags hidden //在服务器浏览列表的中隐藏（防止别人恶意攻击服务器）
    //coop合作；versus对抗；survival生还者；realism写实；scavenge清道夫
    sv_gametypes "coop,versus,survival,realism" //设定服务器可用的游戏模式
    sm_cvar mp_gamemode realism //设定当前游戏模式为合作战役
    z_difficulty Impossible //游戏难度：easy简单；normal普通；hard高级；impossible专家
    sv_region 4 //设定服务器地区为亚洲
    sv_lan 0 //非局域网
    sv_consistency 0 //关闭模型(MOD)冲突
    motd_enabled 1 //玩家进入服务器自动打开[今日消息]界面
    sv_cheats 0 //关闭作弊

### 运行方式

    cd /home/l4d2/Steam/l4d2
    ./srcds_run -game left4dead2 -insecure +hostport 27015 -condebug +map c1m2_streets +exec server.cfg -nomaster

    或者使用脚本`nano /home/l4d2/Steam/l4d2/start.sh`

    /home/l4d2/Steam/l4d2/srcds_run -game left4dead2 -insecure +hostport 27015 -condebug +map c1m2_streets +exec server.cfg -nomaster

### 今日内容

服务器端的motd.txt中，文件所在路径为：/home/l4d2/Steam/l4d2/left4dead2

如需自定义内容，建议在同路径下新建一个motd1.txt文档，将要展示的内容写在里面，同时在服务器端的server.cfg中添加一条指令：motdfile "motd1.txt"