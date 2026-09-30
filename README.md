# AutoDL Sing-box

面向 AutoDL 容器环境的 sing-box 自动配置与代理管理工具。

## 功能

- 自动检测 AutoDL 环境
- 安装时自动利用 `/etc/network_turbo` 下载官方 sing-box
- 自动识别 `amd64` / `arm64`
- 自动获取 sing-box 官方稳定版
- 支持 VLESS 订阅解析
- 自动测试全部节点速度
- 自动选择测速最快节点
- GitHub 代理连通性检查
- 健康检查失败自动重试，减少网络抖动造成的误报
- sing-box 启动、停止、重启、状态和日志管理
- 可选 Git 代理管理

## AutoDL 安装

### 1. 开启 AutoDL 学术加速

新 AutoDL 实例先执行：

```bash
source /etc/network_turbo
```

这里主要用于加速 GitHub clone。

### 2. 克隆项目

```bash
git clone https://github.com/osleax/autodl-singbox.git
cd autodl-singbox
```

`sb` 已包含可执行权限，无需再执行 `chmod +x sb`。

### 3. 安装

```bash
./sb install
```

安装程序会：

1. 检测 AutoDL `/etc/network_turbo`
2. 检测 CPU 架构
3. 获取 sing-box 官方稳定版本
4. 下载并安装 sing-box
5. 安装 `sb` 到 `/usr/local/bin/sb`

### 4. 释放当前 Shell 的学术加速代理

如果之前执行过：

```bash
source /etc/network_turbo
```

学术加速的代理环境变量仍可能保留在当前 Shell。

执行：

```bash
unset http_proxy https_proxy HTTP_PROXY HTTPS_PROXY ALL_PROXY all_proxy
```

可以检查：

```bash
env | grep -i proxy
```

只剩 `no_proxy` 没有关系。

### 5. 配置 sing-box

```bash
sb setup
```

根据提示粘贴自己的订阅地址。

程序会自动：

- 下载订阅
- 解析 VLESS 节点
- 测试节点
- 选择最快节点
- 生成 sing-box 配置
- 启动代理
- 测试 GitHub 连通性

> 订阅地址输入时不会显示。请勿将订阅地址、VLESS 链接或生成的配置提交到 GitHub。

### 6. 检查状态

```bash
sb status
```

正常情况下会看到：

```text
========== sing-box ==========
进程: ✓ 运行中
PID: xxxx
代理: http://127.0.0.1:1080
状态: ✓ GitHub 代理正常
==============================
```

## 新 AutoDL 快速部署

完整流程：

```bash
source /etc/network_turbo

git clone https://github.com/osleax/autodl-singbox.git
cd autodl-singbox
./sb install

unset http_proxy https_proxy HTTP_PROXY HTTPS_PROXY ALL_PROXY all_proxy

sb setup
sb status
```

## 常用命令

```bash
sb help          # 查看帮助
sb setup         # 首次配置 / 重新配置订阅
sb status        # 查看运行状态和代理连通性
sb test          # 测试当前节点访问 GitHub
sb speedtest     # 重新测试全部现有节点并选择最快节点
sb update        # 更新订阅
sb start         # 启动 sing-box
sb stop          # 停止 sing-box
sb restart       # 重启 sing-box
sb log           # 查看日志
sb git-on        # 开启 Git 全局代理
sb git-off       # 关闭 Git 全局代理
sb version       # 查看版本
```

## 代理地址

默认 mixed 代理：

```text
http://127.0.0.1:1080
```

例如单独让 curl 使用 sing-box：

```bash
curl --proxy http://127.0.0.1:1080 https://github.com
```

这不会修改 Git 配置。

## Git 代理

如需让 Git 使用 sing-box：

```bash
sb git-on
```

关闭：

```bash
sb git-off
```

也可以完全不修改 Git 全局配置，而只对单次命令指定代理：

```bash
git -c http.proxy=http://127.0.0.1:1080 \
    -c https.proxy=http://127.0.0.1:1080 \
    clone https://github.com/OWNER/REPO.git
```

## 节点测速

当前节点速度变慢时：

```bash
sb speedtest
```

程序会重新测试配置中的 VLESS 节点，并选择本次测速最快的节点。

网络质量存在波动，因此不同时间测速得到的最快节点可能不同。

## 更新订阅

订阅节点发生变化时：

```bash
sb update
```

根据提示重新输入订阅地址。

## 健康检查

`sb start`、`sb restart` 和 `sb status` 会通过：

```text
127.0.0.1:1080
```

检查 GitHub 连通性。

由于代理节点偶尔可能出现连接抖动，健康检查会进行重试，避免单次连接超时直接被判定为代理故障。

如果仍然显示失败，可以检查：

```bash
sb log
```

并手动测试：

```bash
curl --proxy http://127.0.0.1:1080 \
  --connect-timeout 10 \
  --max-time 30 \
  -L -o /dev/null \
  -w 'HTTP: %{http_code}\n速度: %{speed_download} bytes/s\n' \
  https://github.com
```

## 文件位置

安装后的主要文件：

```text
/usr/local/bin/sb
/usr/local/bin/sing-box
/etc/sing-box/config.json
/etc/sing-box/config.json.bak
/var/log/sing-box.log
/run/sing-box.pid
```

## AutoDL 环境说明

AutoDL 容器通常没有可用的 `/dev/net/tun`，因此本项目不依赖 TUN 模式。

sing-box 在本机监听：

```text
127.0.0.1:1080
```

需要代理的程序可以显式使用这个 HTTP / SOCKS mixed 代理。

AutoDL `/etc/network_turbo` 和 sing-box 是两套不同的代理机制：

```text
AutoDL 学术加速
    ↓
主要用于安装前 GitHub clone / 下载

sing-box
    ↓
安装配置完成后的独立代理
```

## 安全提示

请勿提交以下内容到 GitHub：

- 订阅 URL
- VLESS 链接
- UUID
- 节点认证信息
- `config.json`
- `config.json.bak`
- 机场 Token
- sing-box 日志

`.gitignore` 已用于排除常见配置、日志和临时文件。

SSH 私钥同样绝对不要提交：

```text
~/.ssh/id_ed25519
```

公钥 `.pub` 可以添加到 GitHub SSH Keys。
