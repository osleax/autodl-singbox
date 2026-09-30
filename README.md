# AutoDL Sing-box

AutoDL 专用 sing-box 自动配置与代理管理工具。

主要功能：

- 自动检测 AutoDL 环境
- 利用 `/etc/network_turbo` 下载 sing-box
- 自动识别 amd64 / arm64
- 自动获取 sing-box 官方稳定版
- 支持 VLESS 订阅解析
- 全节点自动测速
- 自动选择最快节点
- GitHub 代理测试
- Git 全局代理管理
- sing-box 启动、停止、重启和日志管理

## 安装

在 AutoDL 终端执行：

```bash
source /etc/network_turbo

git clone https://github.com/osleax/autodl-singbox.git
cd autodl-singbox

chmod +x sb
./sb install
```

安装完成后：

```bash
sb setup
```

根据提示粘贴自己的订阅地址。

> 订阅地址输入时不会显示，也不会保存到 GitHub。

## 常用命令

```bash
# 查看帮助
sb help

# 首次配置
sb setup

# 查看状态
sb status

# 测试当前代理
sb test

# 所有节点重新测速
sb speedtest

# 更新订阅
sb update

# 启动
sb start

# 停止
sb stop

# 重启
sb restart

# 查看日志
sb log

# 开启 Git 代理
sb git-on

# 关闭 Git 代理
sb git-off

# 查看版本
sb version
```

## 文件位置

安装后：

```text
/usr/local/bin/sb
/usr/local/bin/sing-box
/etc/sing-box/config.json
/etc/sing-box/config.json.bak
/var/log/sing-box.log
/run/sing-box.pid
```

默认代理地址：

```text
http://127.0.0.1:1080
```

## 更新节点

如果当前节点速度变慢：

```bash
sb speedtest
```

程序会重新测试现有 VLESS 节点，并自动切换到测速最快的节点。

如果订阅内容发生变化：

```bash
sb update
```

重新输入订阅地址即可。

## 安全提示

请勿将以下内容提交到 GitHub：

- `config.json`
- 订阅链接
- VLESS 链接
- UUID
- 机场 Token

`.gitignore` 已配置忽略常见敏感配置和临时文件。

## AutoDL

本项目针对 AutoDL 容器环境设计，不使用 TUN 模式，而是通过本机 mixed 代理：

```text
127.0.0.1:1080
```

为 Git、curl 等程序提供代理。
