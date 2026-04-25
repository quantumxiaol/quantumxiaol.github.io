---
title: "Linux配置环境"
description: "配置linux系统的几种方法"
pubDate: "Jul 08 2025"
# heroImage: "../../assets/blog-placeholder-3.jpg"
---

# Linux配置环境

## 配置linux系统的几种方法

### 1 WSL

在windows11后全面支持Windows Subsystem for Linux，使得在Windows系统下可以直接运行linux系统无需虚拟机。
在管理员模式下向powershell中输入

    dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
安装WSL必备组件，并输入

    dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart 
打开hyper-V虚拟化后即可在应用商店中下载linux系统。

如果出现参考的对象类型不支持尝试的操作，重置网络net winsock reset即可。
缺点是不原生支持GUI界面。

### 2 实体机安装双系统

以ubuntu为例，在ubuntu官网中下载镜像，使用烧录软件烧录到U盘中制作安装系统的引导盘，在实体机中安装双系统。在重启电脑后进入bios，以引导盘作为优先启动项，在引导系统的指引下安装到硬盘的空闲位置。

也可以使用Ventoy烧录U盘，制作多个启动镜像，甚至可以直接在其中启动虚拟机。Ventoy中可以直接运行vhdx镜像文件，免去了安装虚拟机。

缺点是每次开机时要调整或选择启动项。

### 3 Windows安装虚拟机

可以使用VMware或VirtualBox安装Linux系统，使用iso文件在虚拟机中安装ubuntu系统。接下来安装过程同在实体机过程基本一致。

特别的，由于hyper-V的存在，虚拟机和WSL（以及WSA）会冲突。

### 4 购买云服务器

可以在阿里云、腾讯云或华为云中购买云服务器（轻量应用服务器），选定linux系统，购买后，在防火墙中开启22端口后，在本地输入云服务器ip，使用VSCode或XShell等SSH连接到服务器，之后像本地操作一样使用linux系统。如过要GUI，可以开启VNC服务后在本地使用VNC viewer远程连接。使用Xftp或WinSCP上传下载文件。
缺点是带宽昂贵，默认为5M，上传下载文件不方便。~~而且这个服务器多半是跑不了深度学习的~~

特别的，选的GPU服务器时可以指定CUDA和其他环境，一般是默认装好的，开盖即用。

## Linux 系统安装

安装Ubuntu系统，以双系统为例，假定已经存在了windows系统，且拥有windows的PE盘，用于恢复引导。

### 制作启动盘

下载 [Ubuntu 镜像](https://ubuntu.com/download/desktop)

使用Rufus或Etcher将iso烧录到U盘

如果电脑是较新的 UEFI 系统，选择 GPT 分区方案。
如果电脑是旧的 BIOS 系统，选择 MBR 分区方案。

### 从U盘启动

进入 BIOS/UEFI 设置，开机时按下特定键（通常是 F2、F12、DEL 或 ESC，具体取决于主板型号）。

在 BIOS 设置中确认启动模式：UEFI 或 Legacy（BIOS）。将 U 盘设置为第一启动项。

保存更改后重启电脑，系统应该会从 U 盘启动。

按不出来可以在windows设置-更新和安全-恢复-高级启动 重启电脑，从USB设备恢复

### 安装Ubuntu

配置系统安装到什么磁盘的区域，对于双系统而言，选择引导加载器安装放在windows盘的efi的区域。挂载root到空闲分区，文件系统类型可以选ext4。
配置完成后，点击安装，安装完成后，重启，移除安装介质，进入Ubuntu系统。
如果出现grub引导的命令行，可以输入

    chainloader (hdX,Y)/EFI/Microsoft/Boot/bootmgfw.efi
    boot

进入Windows，如果错误（找不到系统）可以使用PE盘恢复windows引导。
或者输入

    linux (hdX,Y)/boot/vmlinuz-X.X.X-XX-generic root=/dev/sdXY
    initrd (hdX,Y)/boot/initrd.img-X.X.X-XX-generic
    boot
(hdX,Y)指的是包含Linux内核的分区。不过可能太杂找不到哪个是哪个。

更改正常后进入GRUB界面，一般可以选择ubuntu、windows、ubuntu高级选项，在ubuntu中修改grub以修改默认进入的系统。

    sudo nano /etc/default/grub

可以将 GRUB_DEFAULT 设置为想要默认启动的操作系统在GRUB菜单中的位置索引（从0开始计数）。运行`sudo update-grub`更改生效。

### 添加新用户

运行`sudo adduser newuser`添加新用户，
运行`sudo usermod -aG sudo newuser`给予sudo权限。

## Linux配置虚拟环境[参照WSL](../WSL/readme.md)

全量安装完成后默认是有nvidia的驱动的，可以输入`nvidia-smi`查看。

如果出现
>done.
done.
正在处理用于 dbus (1.12.20-2ubuntu4.1) 的触发器 ...
在处理时有错误发生：
 nvidia-dkms-515
 cuda-drivers-515
 cuda-drivers
 nvidia-driver-515
 cuda-runtime-11-7
 cuda-demo-suite-11-7
 cuda-11-7
 cuda
E: Sub-process /usr/bin/dpkg returned an error code (1)

可以运行`ubuntu-drivers devices`查看可用驱动，如果没有可先添加`sudo add-apt-repository ppa:graphics-drivers/ppa`，运行`sudo apt install nvidia-driver-570`安装一个新的驱动。
如果是内核版本不匹配`dpkg --list | grep linux-image`查看内核，
>rc  linux-image-5.15.0-1079-nvidia              5.15.0-1079.80                          amd64        Signed kernel image nvidia
ii  linux-image-6.2.0-26-generic                6.2.0-26.26~22.04.1                     amd64        Signed kernel image generic
ii  linux-image-6.8.0-60-generic                6.8.0-60.63~22.04.1                     amd64        Signed kernel image generic
ii  linux-image-generic-hwe-22.04               6.8.0-60.63~22.04.1                     amd64        Generic Linux kernel image

进入旧内核卸载新内核
sudo apt remove linux-image-6.8.0-60-generic linux-headers-6.8.0-60-generic
sudo update-grub

其他安装流程类似WSL，但仓库地址不一样，需要修改。主要的差别在于WSL没有nvidia的图形输出。

安装CUDA和Anaconda

为cuda添加仓库（ubuntu22.04，需要更换驱动）

    wget https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2204/x86_64/cuda-ubuntu2204.pin
    sudo mv cuda-ubuntu2204.pin /etc/apt/preferences.d/cuda-repository-pin-600
    wget https://developer.download.nvidia.com/compute/cuda/11.7.0/local_installers/cuda-repo-ubuntu2204-11-7-local_11.7.0-515.43.04-1_amd64.deb
    sudo dpkg -i cuda-repo-ubuntu2204-11-7-local_11.7.0-515.43.04-1_amd64.deb
    sudo cp /var/cuda-repo-ubuntu2204-11-7-local/cuda-*-keyring.gpg /usr/share/keyrings/
    sudo apt-get update

安装 CUDA 工具包

    sudo apt-get update
    ~~sudo apt-get install -y cuda~~
    sudo apt-get install cuda-toolkit-11-7

配置环境变量

    nano ~/.bashrc
在文件最后添加以下内容：

    export PATH=/usr/local/cuda/bin:$PATH
    export LD_LIBRARY_PATH=/usr/local/cuda/lib64:$LD_LIBRARY_PATH
    export CUDA_HOME=/usr/local/cuda

运行 `source ~/.bashrc` 使环境变量生效

验证安装

    nvidia-smi
    nvcc --version
输出GPU和CUDA的信息。

运行`nvcc ./HelloWorld/helloworld_cuda.cu -o helloworld`编译helloworld，运行`./helloworld`查看输出

        Hello World from CPU!
        Hello World from GPU!
说明正常。

## 登陆多用户的实验室服务器

### SSH登陆

#### 使用Ubuntu SSH连接服务器

运行`ssh-keygen -t rsa -b 4096 -C "mail@example.com"`，其中`mail@example.com`是一个标签，通常是你的用户邮箱。

运行`cat ~/.ssh/id_rsa.pub`查看公钥，如果之前配置过也可以直接查看，运行`ls -al ~/.ssh`查看是否配置过。

配置登陆电脑的ssh公钥密钥，把公钥上传到服务器的`~/.ssh/authorized_keys`中。

#### 使用Windows SSH连接服务器

运行`ssh-keygen -t rsa -b 4096 -C "mail@example.com"`

生成的密钥文件`id_rsa`和`id_rsa.pub`一般在`C:\Users\username\.ssh`中。

#### 使用MacOS SSH连接服务器

运行`ssh-keygen -t rsa -b 4096 -C "mail@example.com"`

生成的密钥文件`id_rsa`和`id_rsa.pub`一般在`/Users/username/.ssh`中。

### 配置CUDA

实验室服务器通常别人已经安装了CUDA，这种情况下，运行`ls /usr/local | grep cuda`查看安装位置，添加到bashrc即可。

运行`nvidia-smi`查看GPU信息，可以看到显存和显存使用情况。

运行`nvcc --version`查看CUDA版本。




