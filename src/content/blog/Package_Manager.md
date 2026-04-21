---
title: "包管理工具"
description: "Linux、MacOS、Windows的包管理工具"
pubDate: "Jul 08 2025"
heroImage: "../../assets/blog-placeholder-3.jpg"
---


# 包管理平台

## MacOS

MacOS使用Homebrew进行包管理。

[Homebrew](https://brew.sh/)

```bash
# 安装 Homebrew（如果未安装）
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

## Linux

在Linux上，可以使用apt-get进行包管理。
APT（Advanced Package Tool）是 Debian 及其衍生发行版（如 Ubuntu）的标准包管理工具。

snap	Ubuntu 推广的通用包格式，自动更新，跨发行版（如 snap install code）

flatpak	社区驱动的通用包格式，支持沙箱（如 flatpak install flathub org.gimp.GIMP）

dpkg	底层工具，用于安装 .deb 文件（不处理依赖）

## Windows

在Windows上，可以使用Winget（Windows 10 1809+ / Windows 11自带）或Chocolatey（第三方管理）进行包管理。

包 ID 可通过 winget search 查找。

安装Chocolatey
```bash
Set-ExecutionPolicy Bypass -Scope Process -Force; 
iex ((New-Object System.Net.WebClient).DownloadString('https://community.chocolatey.org/install.ps1'))
```