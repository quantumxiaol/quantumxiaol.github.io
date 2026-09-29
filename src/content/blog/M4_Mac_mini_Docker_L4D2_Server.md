---
title: "L4D2 server on Mac Mini"
description: "在 M4 Mac mini 上用 Docker 搭建求生之路 2 服务器"
pubDate: "Sept 29 2026"
# heroImage: "../../assets/blog-placeholder-3.jpg"
---

# 在 M4 Mac mini 上用 Docker 搭建求生之路 2 服务器

> 适用环境：Apple Silicon Mac（M1/M2/M3/M4，本文实测 M4 16GB）+ OrbStack（或 Docker Desktop）+ frp 内网穿透。
> 目标：开箱即用的 L4D2 专用服务器（srcds），支持公网联机、RCON 管理。

---

## 0. 原理：为什么不能直接用 Rosetta

- L4D2 专用服务器（Steam AppID **222860**）的 `srcds_linux` 是 **32 位（i386）x86 程序**，连 SteamCMD 本体也是 32 位的；Valve 至今没有为 L4D2 提供 64 位服务端。
- macOS 的 Rosetta（包括 Docker Desktop / OrbStack 里的 Rosetta 选项）**只翻译 x86_64，不支持 32 位**。所以 `--platform linux/amd64` + Rosetta 这条路是死的。
- 本教程采用的方案：**ARM64 容器 + Box64/Box32 动态翻译**（Box64 跑 64 位 x86，内置的 Box32 组件跑 32 位），性能足够带动这个 2009 年的老引擎。
- 下载环节：SteamCMD 在模拟环境下对 222860 无法解析下载配置（`Missing configuration` 报错），改用 **DepotDownloader**（原生 ARM64 程序，零模拟）下载服务端文件。

备选方案：QEMU 用户态全模拟 i386 容器（兼容性最好、速度较慢），见文末附录。

---

## 1. 准备工作

### 1.1 安装 OrbStack（或 Docker Desktop）

OrbStack 官网下载安装即可，它提供完全兼容的 `docker` 命令。验证：

```bash
docker --version
```

> **内存提示**：OrbStack 默认内存上限 8GB（Settings 里可调）。L4D2 服务端本体只占 1GB 左右，但如果还跑着 Minecraft 服务端这类吃内存的服务，注意 16GB 总内存的分配，建议错开高峰期或调大上限。

### 1.2 规划目录

```bash
mkdir -p ~/Desktop/Server/l4d2/data
cd ~/Desktop/Server/l4d2
```

`data` 目录会挂载进容器，服务端的全部文件都持久化在这里——**容器随便删，数据不会丢**。

路径对应关系（很重要，后面所有操作都要分清自己在哪一侧）：

| 位置 | 路径 |
|---|---|
| Mac 上 | `~/Desktop/Server/l4d2/data/` |
| 容器内 | `/home/steam/l4d2/` |

---

## 2. 拉镜像、创建容器

使用社区维护的 `steamcmd-arm64` 镜像（自带 Box64/Box32，含 M 系列芯片优化构建变体）：

```bash
docker run -it --name l4d2 \
  -p 27015:27015/tcp -p 27015:27015/udp \
  -e ARM64_DEVICE=m1 \
  -v "$HOME/Desktop/Server/l4d2/data:/home/steam/l4d2" \
  ghcr.io/sonroyaalmerol/steamcmd-arm64 bash
```

- `-e ARM64_DEVICE=m1`：启用针对 M 系列芯片的 Box64 构建
- 看到 `steam@xxxx:~/steamcmd$` 提示符，说明已进入容器

---

## 3. 下载服务端文件（DepotDownloader）

> **为什么不直接用 SteamCMD？** 实测 SteamCMD 在 Box32 下对 AppID 222860 会报 `ERROR! Failed to install app '222860' (Missing configuration)`，握手阶段还经常卡死。DepotDownloader 是原生 ARM64 程序，满带宽、零玄学。SteamCMD 只在下载时需要，服务端运行不依赖它。

### 3.1 在 Mac 侧下载 DepotDownloader（另开一个终端）

```bash
cd ~/Desktop/Server/l4d2
curl -L -o dd.zip https://github.com/SteamRE/DepotDownloader/releases/latest/download/DepotDownloader-linux-arm64.zip
unzip dd.zip -d data/depotdl
```

因为 `data` 挂载进容器，这个二进制在容器里直接可用。

### 3.2 在容器内执行下载

```bash
chmod +x /home/steam/l4d2/depotdl/DepotDownloader

# 自动重试循环：崩溃自动重启下载（断点续传），直到成功
until /home/steam/l4d2/depotdl/DepotDownloader -app 222860 -dir /home/steam/l4d2/server -validate; do
  echo "crashed, retrying in 3s..."
  sleep 3
done
```

**注意事项：**

- **不要加 `-os linux` 参数**。加了只会下载 Linux 专属 depot（约 315MB），漏掉装着地图和资源的公共 depot，服务端起不来。不加参数下载全部 depot，总量约 **6~7GB**。
- 下载中途偶发 `Illegal instruction` 崩溃（与虚拟机内存紧张有关），重试循环会自动续传。
- SSH 长跑任务建议套一层 `tmux`，防止断线丢失进度。

### 3.3 验证下载完整性

```bash
du -sh /home/steam/l4d2/server            # 应超过 6GB（实测 9.3G）
ls /home/steam/l4d2/server/left4dead2/maps/ | grep c1m1   # 应看到 c1m1_hotel.bsp
```

---

## 4. 编写 server.cfg

容器内执行：

```bash
mkdir -p /home/steam/l4d2/server/left4dead2/cfg
cat > /home/steam/l4d2/server/left4dead2/cfg/server.cfg <<'EOF'
hostname "My L4D2 Server"
sv_lan 0
sv_cheats 0
rcon_password "改成你自己的密码"
EOF
```

也可以直接在 Mac 上编辑 `~/Desktop/Server/l4d2/data/server/left4dead2/cfg/server.cfg`，效果一样。

---

## 5. 前台启动测试

容器内：

```bash
cd /home/steam/l4d2/server
export LD_LIBRARY_PATH=/home/steam/l4d2/server/bin:$LD_LIBRARY_PATH
box64 ./srcds_linux -game left4dead2 -console +map c1m1_hotel +maxplayers 8 -port 27015 +exec server.cfg
```

**成功的标志**：输出滚动一阵后看到

```
Connection to Steam servers successful.
   VAC secure mode is activated.
```

在 srcds 控制台输入 `status` 能看到地图名和端口，即为运行正常。

**本机/局域网实测**：另一台电脑打开 L4D2，启用开发者控制台（选项 → 键盘/鼠标 → 允许开发者控制台），按 `~` 打开控制台，输入：

```
connect Mac-mini的局域网IP:27015
```

能进游戏、能跑动开枪，服务端这一关就过了。Ctrl+C 停服。

> 若出现 Box64 崩溃/段错误，附加稳定性参数再试：
> `export BOX64_DYNAREC_BIGBLOCK=0 BOX64_DYNAREC_SAFEFLAGS=2 BOX64_DYNAREC_STRONGMEM=3 BOX64_DYNAREC_FASTROUND=0 BOX64_DYNAREC_FASTNAN=0 BOX64_DYNAREC_X87DOUBLE=1`

---

## 6. 后台常驻 + 一键脚本

### 6.1 重建为守护模式容器

Mac 终端执行：

```bash
docker rm -f l4d2
docker run -d --name l4d2 --restart unless-stopped \
  -p 27015:27015/tcp -p 27015:27015/udp \
  -e ARM64_DEVICE=m1 \
  -v "$HOME/Desktop/Server/l4d2/data:/home/steam/l4d2" \
  ghcr.io/sonroyaalmerol/steamcmd-arm64 \
  bash -c 'cd /home/steam/l4d2/server && export LD_LIBRARY_PATH=/home/steam/l4d2/server/bin:$LD_LIBRARY_PATH && exec box64 ./srcds_linux -game left4dead2 -console +map c1m1_hotel +maxplayers 8 -port 27015 +exec server.cfg'
```

`--restart unless-stopped` 意味着：只要 OrbStack 在运行，容器崩溃/重启后都会自动拉起。配合 OrbStack 的 **Settings → Start at login**，Mac 重启后服务器全自动恢复。

### 6.2 一键启动脚本

`~/Desktop/Server/l4d2/start_server.sh`：

```bash
#!/bin/zsh
# L4D2 服务端一键启动：容器不存在则创建，存在则启动

DATA_DIR="$HOME/Desktop/Server/l4d2/data"

if docker ps -a --format '{{.Names}}' | grep -qx l4d2; then
  echo "容器已存在，直接启动"
  docker start l4d2
else
  echo "首次创建容器"
  docker run -d --name l4d2 --restart unless-stopped \
    -p 27015:27015/tcp -p 27015:27015/udp \
    -e ARM64_DEVICE=m1 \
    -v "$DATA_DIR:/home/steam/l4d2" \
    ghcr.io/sonroyaalmerol/steamcmd-arm64 \
    bash -c 'cd /home/steam/l4d2/server && export LD_LIBRARY_PATH=/home/steam/l4d2/server/bin:$LD_LIBRARY_PATH && exec box64 ./srcds_linux -game left4dead2 -console +map c1m1_hotel +maxplayers 8 -port 27015 +exec server.cfg'
fi

docker ps --filter name=l4d2
```

```bash
chmod +x ~/Desktop/Server/l4d2/start_server.sh
```

---

## 7. frp 公网开放

L4D2 需要 **27015 的 TCP 和 UDP 双协议转发**（UDP 承载游戏数据；TCP 给 RCON 和服务器浏览器查询用）。在 Mac mini 的 frpc 配置（toml）中加入：

```toml
[[proxies]]
name = "l4d2-tcp"
type = "tcp"
localIP = "127.0.0.1"
localPort = 27015
remotePort = 27015

[[proxies]]
name = "l4d2-udp"
type = "udp"
localIP = "127.0.0.1"
localPort = 27015
remotePort = 27015
```

然后重启 frpc，并确认 VPS 的防火墙/安全组放行了 27015 的 TCP+UDP。朋友在游戏控制台输入 `connect VPS_IP:27015` 即可进服。

**关于隧道协议**：用 frp 默认的 TCP 隧道即可。KCP/QUIC（UDP 隧道）只在烂线路上有优势，而且需要在安全组放行对应的 UDP 端口，很多人 KCP 连不上其实是 VPS 只放行了 TCP 端口，并非"运营商封了 UDP"。

**如何验证你的网络 UDP 质量**（以校园网为例）：在 VPS 上 `iperf3 -s`（放行 5201 TCP/UDP），Mac 上 `brew install iperf3`，然后：

```bash
iperf3 -c VPS_IP -u -b 20M -t 30      # 上行
iperf3 -c VPS_IP -u -b 20M -t 30 -R   # 下行
```

丢包 <1% 优秀，1%~5% 能玩，>5% 建议换线路或改用局域网直连。校园网拥塞通常出现在晚上。

---

## 8. 管理员：RCON 远程控制台

原版服务端没有账号体系，管理靠 RCON（`server.cfg` 里的 `rcon_password` 就是钥匙）。游戏内控制台：

```
rcon_password 你的密码       # 先登录
rcon status                  # 玩家列表和 ID
rcon kick 玩家名              # 踢人
rcon changelevel c2m1_highway  # 换地图
rcon say 服务器消息            # 以服务器身份喊话
rcon sv_cheats 1             # 开作弊（全局生效，全员无成就）
```

RCON 走 TCP 27015，frp 的 TCP 转发正好覆盖。

**关于 SourceMod/MetaMod 插件平台**：MetaMod 可装且运行稳定（见第 9 节），但 **SourceMod 目前在 Box32 下不可用**——它的 `sourcepawn.jit.x86.so` 需要 `pthread_cond_clockwait` 符号，Box32 的 32 位 libpthread 尚未提供（上游已知问题，未修复），强行加载会段错误崩溃。小型服务器用 RCON 管理完全够用。

---

## 9.（可选）安装 MetaMod

容器内（先停服）：

```bash
cd /tmp
MMF=$(curl -s https://mms.alliedmods.net/mmsdrop/1.12/mmsource-latest-linux)
curl -LO "https://mms.alliedmods.net/mmsdrop/1.12/$MMF"
tar -xzf "$MMF" -C /home/steam/l4d2/server/left4dead2
```

编辑 `/home/steam/l4d2/server/left4dead2/gameinfo.txt`，在 `SearchPaths` 段的 `{` 之后第一行插入（**保留原有所有条目**）：

```
        GameBin |gameinfo_path|addons/metamod/bin
```

**关键补丁**：Box32 环境下引擎的平台探测会误判为 64 位，执意加载 `addons/metamod/bin/linux64/server.so`。解决办法是用 32 位文件覆盖 linux64 目录里的同名文件：

```bash
cd /home/steam/l4d2/server/left4dead2/addons/metamod/bin
cp -f server.so metamod.2.l4d2.so linux64/
```

重启服务端后 `meta list` 能看到 MetaMod 即为成功（`Listing 0 plugins` 也正常，因为没装插件）。

---

## 10. 日常运维速查

| 操作 | 命令 |
|---|---|
| 查看日志 | `docker logs -f l4d2`（Ctrl+C 退出查看，不影响服务） |
| 重启服务 | `docker restart l4d2` |
| 停止服务 | `docker stop l4d2` |
| 进入 srcds 控制台 | `docker attach l4d2`（退出用 Ctrl+P Ctrl+Q，**不要按 Ctrl+C**，否则会把服务端杀掉） |
| 进容器 shell | `docker exec -it l4d2 bash` |
| 修改配置 | 编辑 Mac 上的 `~/Desktop/Server/l4d2/data/server/left4dead2/cfg/server.cfg`，然后 `docker restart l4d2` |
| 更新服务端 | `docker exec -it l4d2 bash` 进容器，重跑第 3.2 节的 DepotDownloader 命令，再 `docker restart l4d2` |
| 换地图 | 游戏内 `rcon changelevel 地图名`，或修改启动参数里的 `+map` |

---

## 11. 踩坑记录（FAQ）

| 症状 | 原因与解法 |
|---|---|
| `--platform linux/amd64` + Rosetta 无法启动 | srcds 是 32 位程序，Rosetta 只支持 x86_64，此路不通 |
| `steamcmd.sh: Couldn't find steamcmd at .../linuxarm64/steamcmd` | 镜像包装脚本在 SteamCMD 自更新后找错路径，绕过它直接 `box64 /home/steam/steamcmd/linux32/steamcmd +...` |
| SteamCMD 卡在 `Loading Steam API...OK` | Box32 下已知毛病，重试或加 Box64 稳定性参数；也可直接放弃 SteamCMD 用 DepotDownloader |
| `ERROR! Failed to install app '222860' (Missing configuration)` | SteamCMD 在模拟下无法解析该 app 的下载配置，换 DepotDownloader |
| DepotDownloader 只下了 315MB | 误加 `-os linux` 参数，漏掉了公共内容 depot；去掉参数重跑（断点续传） |
| DepotDownloader `Illegal instruction` | 虚拟机内存紧张导致，用 until 循环自动重试，并关闭其他吃内存的服务 |
| `meta list` → `Unknown command "meta"` | 引擎误判平台去加载 linux64 目录的 64 位插件；用 32 位文件覆盖 `addons/metamod/bin/linux64/` |
| 装 SourceMod 后 `Segmentation fault` | Box32 缺 `pthread_cond_clockwait` 符号，SourceMod 当前不可用；卸载 `addons/sourcemod` 即可恢复 |

---

## 附录：备选方案 —— QEMU i386 全模拟容器

如果 Box64 路线在你的环境上跑不通，退回 QEMU 用户态模拟（慢一些，但兼容性最好，SourceMod 也能跑）：

```bash
# OrbStack 内置支持 linux/386，一般无需注册；若报 exec format error 则执行：
docker run --rm --privileged tonistiigi/binfmt --install i386

# 验证（输出 i686 即成功）
docker run --rm --platform linux/386 i386/debian:bookworm uname -m

# 起 32 位 Debian 容器，之后完全按 Valve 官方 Linux 文档安装
docker run -it --name l4d2 --platform linux/386 \
  -p 27015:27015/tcp -p 27015:27015/udp \
  -v "$HOME/Desktop/Server/l4d2/data:/home/steam/l4d2" \
  i386/debian:bookworm bash
```

容器内就是标准 x86 流程：`apt install curl lib32gcc-s1` → 下载 SteamCMD → `app_update 222860` → `./srcds_run -game left4dead2 ...`，不需要任何模拟器相关的特殊处理。

---

## 最终架构一览

```
玩家 ──UDP/TCP 27015──> VPS(frps) ══TCP 隧道══> Mac mini(frpc) ──> Docker 容器
                                                                        │
                                                          Box64/Box32 翻译运行
                                                          32 位 srcds_linux
                                                          文件持久化在 ~/Desktop/Server/l4d2/data
```

实测效果：M4 16GB + OrbStack，校园网晚高峰经 frp 中转延迟约 100ms，合作模式体验良好。
