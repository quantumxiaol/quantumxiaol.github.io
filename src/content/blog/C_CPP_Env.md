---
title: "C/CPP环境安装"
description: "C CPP开发环境安装配置"
pubDate: "Jul 08 2025"
# heroImage: "../../assets/blog-placeholder-3.jpg"
---


# C CPP环境安装

## 默认编译器

Linux的默认编译器是gcc、g++，Windows可以装MSVC、MinGW，MacOS自带clang。

### Linux
```bash
# 安装 GCC（C/C++ 编译器）和构建工具
sudo apt-get install build-essential
# build-essential 包含：
#   gcc, g++, make, libc6-dev 等
```

### MacOS
MacOS有自带的apple-clang，但有时可能需要LLVM的clang
```bash
# 安装命令行开发工具（包含 clang 编译器）
xcode-select --install

# 验证安装
clang --version
```

### Windows
Windows可以安装Visual Studio或者MinGW，安装VS后安装C++桌面开发工具包。
也可以安装MSYS2到C:\msys64。
也可以直接安装MinGW。
```bash
# 安装 MSYS2 后运行：
pacman -S mingw-w64-x86_64-gcc

# 或使用 winget 安装 MinGW-w64
winget install WinDev.Mingw

# 验证安装
gcc --version
g++ --version

cl
```