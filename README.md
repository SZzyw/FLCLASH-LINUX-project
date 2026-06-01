# FlClash 无图形纯终端部署包

> 适用系统：Ubuntu 22.04+ / Debian 12+ / 任何 Linux x86_64 系统
> 
> **无需安装 Go、无需编译、开箱即用。**
>
> 源码地址：https://github.com/SZzyw/FLCLASH-LINUX

---

## 文件结构

```
flcc-project/
├── headless/
│   └── flclash-headless    # 终端控制程序 (已预编译, 10MB)
├── FlClashCore              # 代理核心引擎 (已预编译, 53MB)
├── assets/
│   └── data/                # 路由数据库 (GEOIP/GEOSITE)
│       ├── GEOIP.dat
│       ├── GEOSITE.dat
│       ├── GEOIP.metadb
│       └── ASN.mmdb
└── 正确.md                   # 本说明文件
```

---

## 快速开始

```bash
# 1. 上传 flcc-project/ 到 Linux 机器任意位置

# 2. 给执行权限 /
cd flcc-project
chmod +x FlClashCore headless/flclash-headless

# 3. 启动后台服务 /flcc-project
cd headless
sudo -v
sudo ./flclash-headless daemon &

# 4. 打开 TUI 交互界面(开一个新的终端) /flcc-project/headless/
sudo ./flclash-headless tui
```

> 所有操作都在 `flcc-project/headless/` 目录下完成。

---

## TUI 首次使用

```
仪表盘 (按 1) → 配置管理 (按 6 或 c) → 输入订阅 URL → 确认导入
返回仪表盘 (按 1) → 启动核心 (按 r) → 确认启动
```

### 界面导航

| 按键 | 功能 |
|------|------|
| `1` | 仪表盘（流量、速度、内存） |
| `2` | 查看并切换代理节点组 |
| `3` | 选择全局出口 |
| `4` | 切换运行模式（规则/全局/直连） |
| `5` | 查看关键日志 |
| `6` / `c` | 管理配置文件和订阅 |
| `r` | 启动/停止核心 |
| `n` | 切换 TUN |
| `q` | 退出 TUI |

---

## CLI 命令 (daemon 运行时)

所有命令在 `flcc-project/headless/` 下执行：

```bash
sudo ./flclash-headless status            # 查看运行状态
sudo ./flclash-headless start             # 启动代理核心
sudo ./flclash-headless stop              # 停止
sudo ./flclash-headless restart           # 重启
sudo ./flclash-headless global <出口名>   # 切换全局出口
sudo ./flclash-headless tun on|off        # TUN 虚拟网卡
sudo ./flclash-headless logs              # 查看日志
```

---

## 原理说明

```
                    ┌───────────────────────────────┐
  headless/         │  flclash-headless             │
  ./flclash-headless│  ├── daemon  (后台服务)       │
                    │  ├── tui     (交互界面)       │
                    │  ├── status  (查看状态)       │
                    │  └── start/stop/...(CLI命令)  │
                    │         │                     │
                    │         │ Unix Socket 通信     │
                    │         ▼                     │
                    │  ┌──────────────────────┐    │
                    │  │  FlClashCore         │    │
                    │  │  混合端口 :7890      │    │
                    │  │  控制端口 :9090      │    │
                    │  └──────────────────────┘    │
                    └───────────────────────────────┘
```

---

## 关闭 daemon 服务

停止核心并退出 daemon 后台进程：

```bash
cd flcc-project/headless

# 1. 停止代理核心
sudo ./flclash-headless stop

# 2. 查找并杀死 daemon 进程
sudo pkill -f "flclash-headless daemon"

# 或者查看进程后手动杀
# ps aux | grep flclash-headless
# sudo kill <PID>
```

> 如果 TUI 还开着，先按 `q` 退出 TUI。

---

## 完全卸载

```bash
# 1. 先关闭 daemon（见上方步骤）
# 2. 删除程序目录
rm -rf flcc-project/

# 3. 清理运行时数据（配置、日志、缓存等）
rm -rf ~/.local/share/flclash-headless/
```

> 运行时数据目录包含导入的订阅配置、日志文件和缓存，卸载时按需保留或删除即可。

---

## 常见问题

| 现象 | 原因 | 解决 |
|------|------|------|
| `fork/exec ../FlClashCore: permission denied` | FlClashCore 没加执行权限 | `chmod +x ../FlClashCore` |
| `核心启动失败` | 订阅配置无效或网络不通 | 在 TUI 按 `c` → 重新导入订阅 |
| TUN 无法使用 | 需要 root 权限 | 加 `sudo` |
