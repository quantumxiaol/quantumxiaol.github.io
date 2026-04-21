---
title: "Python环境管理"
description: "用conda miniconda venv uv等配置环境"
pubDate: "Jul 08 2025"
heroImage: "../../assets/blog-placeholder-3.jpg"
---

# Python环境管理

安装虚拟环境可以用conda miniconda venv uv等

## Python

安装Python

Linux:
```bash
sudo apt-get update
sudo apt-get install python3 python3-dev
```
MacOS:
```bash
brew install python@3.12
```
Windows:
```bash
# 查看可用版本
winget search Python.Python

# 安装最新稳定版（例如 Python 3.12）
winget install --id Python.Python.3.12 -e
```

## Conda

Conda 是一个开源的包和环境管理系统，支持多语言（Python、R、Node.js 等），主要用于数据科学、机器学习等领域。

可以创建隔离的虚拟环境。
跨平台支持（Windows/macOS/Linux）。
不仅管理 Python 包，还能管理 C/C++ 库等二进制依赖。

### 安装Conda

#### Windows

在官方网站上下载anaconda安装包，并安装[anaconda](https://www.anaconda.com/products/individual)

或者在清华镜像站下载anaconda安装包，并安装[tuna/anaconda](https://mirrors.tuna.tsinghua.edu.cn/anaconda/archive/)

#### Linux
```bash
# On Linux
wget https://repo.anaconda.com/archive/Anaconda3-2024.10-1-Linux-x86_64.sh
# chmod +x Anaconda3-2024.10-1-Linux-x86_64.sh
# bash Anaconda3-2024.10-1-Linux-x86_64.sh
./Anaconda3-2024.10-1-Linux-x86_64.sh
```
### 使用
```bash
conda create -n myenv python=3.10
conda activate myenv
conda install numpy pandas
```
## Miniconda

Miniconda 是 Conda 的轻量版本，只包含基础的 Conda 和 Python，没有预装大量库。适合希望自定义安装内容的用户。

Anaconda = Miniconda + 预装数百个库（如 NumPy, Pandas, Scikit-learn 等）

### 安装Miniconda
```bash
# On Linux
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh
# chmod +x Miniconda3-latest-Linux-x86_64.sh
# bash Miniconda3-latest-Linux-x86_64.sh
./Miniconda3-latest-Linux-x86_64.sh
```
### 使用Miniconda

和anaconda一样

## venv

venv 是 Python 标准库中自带的虚拟环境工具，用于创建轻量级的隔离环境，适合纯 Python 项目。

Python 自带，无需额外安装（Python 3.3+）。
更轻量，但不能管理非 Python 依赖（如 C 库）。

### 安装

venv模块是Python标准库的一部分，从Python 3.3版本开始就已包含在内

```bash
sudo apt install python3.x-venv
```
### 使用venv
```bash
python3 -m venv .venv
source .venv/bin/activate
pip install fastapi uvicorn
```
## uv (by Astral)

[uv](https://github.com/astral-sh/uv)是一个新的超快 Python 包管理器和构建工具，由 Rust 编写，旨在替代 pip、poetry、setuptools 等传统工具。

比 pip 快 10~100x（得益于 Rust 实现）。
支持虚拟环境创建。
兼容 pip、Poetry、PEP 621 等标准。
支持 lock 文件生成、依赖解析、安装、冻结等功能。

### 安装uv
```bash
# On macOS and Linux.
curl -LsSf https://astral.sh/uv/install.sh | sh
# On Windows.
powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"
# With pip.
pip install uv
# or
cargo install --git https://github.com/astral-sh/uv uv

uv --version
```
### 添加环境变量
```bash
export PATH="$HOME/.local/bin:$PATH"
```
### 使用uv
```bash

# 默认创建 .venv 目录
uv venv  

# 使用 --prefix 指定环境目录：
uv venv --prefix ./myenv

# 指定 Python 版本
uv venv --python 3.11  

# 指定路径创建虚拟环境
uv venv --python 3.11 /root/.venv

# 创建轻量级虚拟环境（无需复制标准库）：
uv venv --seed

# 清除所有缓存文件
uv clean  

# 激活虚拟环境
source /root/.venv/bin/activate

# 退出虚拟环境
deactivate

# pip安装
uv pip install -r requirements.txt
```
创建uvlock

```bash
# 从测试好的环境中生成uvlock
uv pip freeze > requirements_lock.txt
# uv pip compile requirements.txt --universal --output-file requirements_lock.txt

# 获取最小的依赖包极其版本
python envextract.py requirements.txt requirements_lock.txt requirements_mini.txt


# 创建 pyproject.toml
uv init

# 将包复制进pyproject.toml，填到dependencies[]中
python envconvert.py requirements_mini.txt >> pyproject.toml

# 锁定依赖，创建uv.lock
uv lock

# 在其他机器上运行，创建venv
uv sync
```
