---
title: "WSL 安装环境"
description: "在WSL(Windows Subsystem for Linux)上安装开发环境"
pubDate: "Jul 08 2025"
# heroImage: "../../assets/blog-placeholder-3.jpg"
---

# 在WSL(Windows Subsystem for Linux)上安装虚拟环境

## 启用WSL
管理员身份打开 PowerShell 或命令提示符

    wsl --install
    
这会自动启用 WSL 功能、安装 Linux 内核更新包，并设置默认的 Linux 发行版

这是手动启用相关功能的命令，也可以到控制面板中去勾选

    dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
    dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart

如果要使用WSL2，同时Windows为专业版，可以开启Hyper-V

    dism.exe /online /enable-feature /featurename:Microsoft-Hyper-V-All /all /norestart

启用WSL 2

    wsl --set-default-version 2

## 使用VSCode连接WSL

重启系统，打开VSCode，会提醒下载WSL的插件。

使用远程连接进入WSL，会提示下载Linux发行版，选择ubuntu下载。

当输出Please create a default UNIX user account.时，创建自己的用户名和密码。

当通过 VS Code 的 Remote - WSL 扩展进入 WSL 时，默认会位于 Linux 用户主目录（/home/<你的用户名>/）。

WSL 的 Linux 文件系统存储在 Windows 的隐藏目录中，但建议通过 WSL 终端或 /home/ 路径访问文件。

如果需要访问 Windows 文件，可以通过 /mnt/c/ 路径操作。

输入 `exit` 退出终端

输入 `wsl --shutdown` 关闭WSL

## 安装CUDA
在 WSL 中安装 CUDA 工具包

添加 NVIDIA 的 CUDA 包仓库

    wget https://developer.download.nvidia.com/compute/cuda/repos/wsl-ubuntu/x86_64/cuda-wsl-ubuntu.pin
    sudo mv cuda-wsl-ubuntu.pin /etc/apt/preferences.d/cuda-repository-pin-600

下载并安装包

CUDA12.3

    wget https://developer.download.nvidia.com/compute/cuda/12.3.2/local_installers/cuda-repo-wsl-ubuntu-12-3-local_12.3.2-1_amd64.deb
    sudo dpkg -i cuda-repo-wsl-ubuntu-12-3-local_12.3.2-1_amd64.deb
    sudo cp /var/cuda-repo-wsl-ubuntu-12-3-local/cuda-*-keyring.gpg /usr/share/keyrings/

CUDA11.7

    wget https://developer.download.nvidia.com/compute/cuda/11.7.1/local_installers/cuda-repo-wsl-ubuntu-11-7-local_11.7.1-1_amd64.deb
    sudo dpkg -i cuda-repo-wsl-ubuntu-11-7-local_11.7.1-1_amd64.deb
    sudo cp /var/cuda-repo-wsl-ubuntu-11-7-local/cuda-*-keyring.gpg /usr/share/keyrings/

安装 CUDA 工具包

    sudo apt-get update
    sudo apt-get install -y cuda

配置环境变量

    nano ~/.bashrc
在文件最后添加以下内容：

    export PATH=/usr/local/cuda/bin:$PATH
    export LD_LIBRARY_PATH=/usr/local/cuda/lib64:$LD_LIBRARY_PATH

运行 `source ~/.bashrc` 使环境变量生效

验证安装

    nvidia-smi
    nvcc --version
输出GPU和CUDA的信息

## 安装 Anaconda

访问 Anaconda 官方下载页面 获取 Linux 版本的安装脚本链接

    wget https://repo.anaconda.com/archive/Anaconda3-2024.10-1-Linux-x86_64.sh

使用脚本安装

    chmod +x Anaconda3-2024.10-1-Linux-x86_64.sh
    ./Anaconda3-2024.10-1-Linux-x86_64.sh
默认安装路径为 ~/anaconda3，参数-p指定路径。
例如`./Anaconda3-2024.10-1-Linux-x86_64.sh -p /mnt/.anaconda`将安装到/mnt/.anaconda

初始化

    source ~/anaconda3/bin/activate
验证安装

    conda --version

>Do you wish to update your shell profile to automatically initialize conda?
This will activate conda on startup and change the command prompt when activated.
If you'd prefer that conda's base environment not be activated on startup,
   run the following command when conda is activated:

conda config --set auto_activate_base false

You can undo this by running `conda init --reverse $SHELL`? [yes|no]

这个提示的意思是询问是否希望 自动初始化 Conda。如果选择 yes，Conda 会在每次启动终端时自动激活其 base 环境，并修改命令提示符（例如显示 (base)）。如果你选择 no，则需要手动激活 Conda。

选择no需要手动初始化

临时激活命令

    eval "$(/home/quantum/anaconda3/bin/conda shell.bash hook)" 
    conda init
    source ~/.bashrc

## 在WSL中使用代理
当代理服务器运行在Windows上且监听localhost时，在WSL中直接访问这些代理可能会遇到问题，因为WSL不能直接通过localhost访问Windows上的服务。
>wsl: 检测到 localhost 代理配置，但未镜像到 WSL。NAT 模式下的 WSL 不支持 localhost 代理。

找到Windows主机的IP地址：
在WSL中执行 ` hostname -I | awk '{print $1}' ` 获取Windows主机的IP地址：

或者简单地使用172.16.17.32（对于大多数情况下的IPv4地址，这是WSL 2分配给Windows主机的默认网关地址）。

使用找到的Windows主机IP地址而不是localhost来配置Git代理。

    git config --global http.proxy 'http://172.16.17.32:代理端口号'
    git config --global https.proxy 'http://172.16.17.32:代理端口号'

或者可以windows下好包，在文件管理中拖到Linux中。

## 查看WSL 2的资源使用情况
对于正在运行的WSL 2实例，你可以使用Linux命令来查看内存和CPU的使用情况。例如：

使用`top`或`htop`（如果已安装）查看实时的资源使用情况。
使用`free -m`查看总的和可用的内存。
使用`cat /proc/meminfo`获取详细的内存信息。
但是，这些命令仅能显示当前使用的资源量，并不能直接显示分配给WSL 2的最大资源限制。

要查看或设置分配给WSL 2的资源限制（如内存、处理器和交换空间），需要编辑.wslconfig文件。这个文件一般放在用户目录下（即C:\Users\<用户名>\），其格式如下：

    memory=4GB   # 分配给WSL 2的内存
    processors=4 # 可以使用的处理器核心数
    swap=2GB     # 交换分区大小
    localhostForwarding=true # 允许本地主机转发