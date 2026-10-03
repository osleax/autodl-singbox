# AutoDL Sing-box

面向 AutoDL 容器环境的 sing-box 自动配置与代理管理工具。

## 快速使用

### 新 AutoDL 实例

```bash
source /etc/network_turbo
git clone https://github.com/osleax/autodl-singbox.git
cd autodl-singbox
./sb install

unset http_proxy https_proxy HTTP_PROXY HTTPS_PROXY ALL_PROXY all_proxy

sb setup
sb status
```

`sb setup` 会自动解析 VLESS 订阅、过滤订阅信息条目并筛选节点。

当前节点选择策略以 **倍率优先、稳定性门槛、实际下载速度决胜**：

1. 严格按 `0.01 倍率 → 0.1 倍率 → 1 倍率 → 其他倍率` 逐组测试
2. 未明确标注倍率的节点统一按 `1 倍率` 处理
3. VLESS 和 Hysteria2 节点使用相同优先级，不按协议或地区加权
4. 当前倍率组的节点最多 5 个并发进行 GitHub HEAD 稳定性测试，每个节点测试 5 次
5. 成功率至少达到 80% 才允许进入下载测速候选；低于 80% 的节点直接跳过下载测速
6. 自动测速先按成功率和延迟预筛最多 3 个候选
7. 下载测速保持串行，每个候选约下载 500 KB，避免并发下载互相抢占带宽
8. 同倍率合格候选最终优先选择实际下载速度最高的节点
9. 当前倍率组存在下载成功节点后立即停止，不再测试更高倍率
10. 当前倍率组没有达到 80% 的节点，或所有下载候选均失败时，才继续下一倍率组
11. 所有倍率组均无稳定且下载成功的节点时保留原配置

支持从节点名称识别 `0.01倍`、`0.1倍`、`0.1倍率`、`0.1x`、`×0.1`、`倍率: 0.1` 等倍率标注。没有明确倍率标注时按 `1 倍率` 处理。

支持解析 **VLESS** 和 **Hysteria2 / hy2** 节点。协议和地区不会改变节点优先级，节点选择首先由倍率决定。


### 日常使用

```bash
sb status       # 查看状态
sb test         # 测试当前 GitHub 代理
sb select       # 按分组树状菜单手动选择节点
sb speedtest    # 重新筛选稳定节点
sb update       # 更新订阅
sb restart      # 重启 sing-box
sb log          # 查看日志
sb git-on       # 开启 Git 全局代理
sb git-off      # 关闭 Git 全局代理
sb version      # 查看版本
```

当前节点不稳定、GitHub 经常超时或速度明显下降时，运行：

```bash
sb speedtest
```

需要排查问题时：

```bash
sb log
```

## 功能

- 自动检测 AutoDL 环境
- 安装时自动利用 `/etc/network_turbo` 下载官方 sing-box
- 自动识别 `amd64` / `arm64`
- 自动获取 sing-box 官方稳定版
- 支持 VLESS 订阅解析
- 稳定性优先的低流量节点筛选
- 倍率优先：0.01 倍 → 0.1 倍 → 1 倍 → 其他倍率，无合格节点时逐级回退
- 每节点多次 GitHub 连通性检查，降低网络抖动影响
- 自动过滤剩余流量、套餐到期、重置时间等订阅信息条目
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
- 筛选稳定节点
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
sb speedtest     # 稳定性优先重新筛选现有节点
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

程序会重新测试配置中的 VLESS 节点，以 GitHub 稳定性和延迟为主要依据筛选节点。

网络质量存在波动，因此不同时间筛选出的推荐节点可能不同。

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

## 手动选择节点

运行 `sb select`，按下载优先级显示有节点的分组。先输入分组编号，再输入节点编号即可切换；节点菜单输入 `0` 返回分组，分组菜单输入 `0` 退出。

仅显示当前配置中 proxy 选择器可用的 VLESS 节点。切换前检查配置并备份旧配置，随后启动或重启 sing-box。此操作不自动测速；切换后可执行 `sb test` 检查连接。
