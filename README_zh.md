# luci-app-rustdeskd

[English](README.md) | [中文](README_zh.md)

用于在 OpenWrt 上管理 [RustDesk Server](https://github.com/rustdesk/rustdeskd) 的 LuCI 网页界面。

RustDesk 是一款功能齐全的开源远程控制替代方案（类似 TeamViewer 和 AnyDesk）。此 LuCI 插件提供了一个基于 Web 的管理界面，用于在 OpenWrt 路由器上配置和管理自建的 RustDesk 服务器组件（hbbs 和 hbbr）。

## 功能特性

- **服务管理** - 直接从用户界面启动/停止/重启服务
- **开机自启** - 切换服务是否开机自动启动
- **状态监控** - 实时轮询 HBBS 和 HBBR 服务的运行状态
- **公钥显示** - 查看并复制生成的公钥以便于客户端配置
- **密钥重置** - 在需要时重新生成加密密钥
- **日志查看** - 带有自动刷新和自动滚动功能的服务日志查看器
- **防火墙提示** - 显示所需端口，方便手动配置防火墙
- **选项卡配置** - 分类管理 ID 服务器（hbbs）和中继服务器（hbbr）设置
- **输入验证** - 验证路径、端口和配置值的合法性
- **多语言支持** - 完全支持 i18n 翻译（包含 POT 模板）
- **核心一键下载** - 支持在界面一键下载官方 RustDesk 服务器核心文件

## 架构

```
luci-app-rustdeskd/
├── Makefile                      # OpenWrt 软件包构建文件
├── htdocs/luci-static/resources/view/rustdeskd/
│   └── general.js                # 主 UI 界面 (JavaScript)
├── po/templates/
│   └── rustdeskd.pot             # 翻译模板
└── root/
    ├── etc/
    │   ├── config/rustdeskd      # UCI 配置文件
    │   ├── init.d/rustdeskd      # procd 启动脚本
    │   └── uci-defaults/50-luci-rustdeskd  # 首次运行设置
    └── usr/share/
        ├── luci/menu.d/luci-app-rustdeskd.json  # 菜单入口
        └── rpcd/
            ├── acl.d/luci-app-rustdeskd.json    # ACL 权限
            └── ucode/rustdeskd.uc               # RPC 后端
```

## 运行要求

### OpenWrt 依赖项
- OpenWrt 23.05 或更高版本（需安装 LuCI）
- `luci-base` - LuCI 核心框架
- `rpcd` - RPC 守护进程
- `rpcd-mod-ucode` - rpcd 的 ucode 支持

### RustDesk 服务器核心
RustDesk 服务器核心程序（`hbbs`、`hbbr`）必须单独安装，本软件包**不包含**核心文件。你也可以直接在插件界面中点击“下载 / 更新核心”按钮进行自动下载和安装。

#### 手动安装 RustDesk 服务器核心

1. **从 GitHub Releases 下载：**
   ```bash
   # 检查你的架构
   uname -m
   
   # 从以下地址下载对应的核心：
   # https://github.com/rustdesk/rustdeskd/releases
   
   # 以 aarch64 为例：
   wget https://github.com/rustdesk/rustdeskd/releases/download/1.1.11/rustdeskd-linux-arm64v8.zip
   unzip rustdeskd-linux-arm64v8.zip
   cp amd64/hbbs amd64/hbbr /usr/bin/
   chmod +x /usr/bin/hbbs /usr/bin/hbbr
   ```

2. **验证安装：**
   ```bash
   /usr/bin/hbbs --version
   /usr/bin/hbbr --version
   ```

## 安装说明

### 从 OpenWrt 软件源安装
```bash
opkg update
opkg install luci-app-rustdeskd
```

### 源码编译安装 (开发环境)
```bash
# 克隆 LuCI 仓库
git clone https://github.com/openwrt/luci.git
cd luci

# 编译软件包
make package/luci-app-rustdeskd/compile
```

### 手动安装
1. 将应用文件复制到 OpenWrt 设备：
   ```bash
   # 复制 htdocs 到 /www
   cp -r htdocs/luci-static /www/luci-static/
   
   # 复制 root 文件
   cp -r root/* /
   
   # 设置权限
   chmod +x /etc/init.d/rustdeskd
   ```

2. 重新加载 rpcd 以注册新的 RPC 方法：
   ```bash
   /etc/init.d/rpcd reload
   ```

3. 清除 LuCI 缓存：
   ```bash
   rm -rf /tmp/luci-*
   ```

4. 访问界面：**服务 → RustDesk Server**（或者中文界面下的 **远程桌面**）

## 配置指南

### 核心路径
插件默认 `hbbs` 和 `hbbr` 存放在 `/usr/bin` 目录下。

### 防火墙配置
必须在 **网络 → 防火墙 → 通信规则** 中手动配置防火墙规则。插件的状态栏中会显示默认所需的端口。

标准 RustDesk 端口分配如下：

| 端口 | 协议 | 服务 | 计算方式 |
|------|----------|---------|-------------|
| HBBS-1 | TCP | NAT 类型测试 | server_port - 1 |
| HBBS | TCP/UDP | ID 服务器 / 打洞 | server_port |
| HBBS+2 | TCP | Web 客户端支持 | server_port + 2 |
| HBBR | TCP | 中继服务器 | relay_port |
| HBBR+2 | TCP | Web 客户端支持 | relay_port + 2 |

**示例：** 使用默认端口（`server_port=21116` 和 `relay_port=21117`）：
- TCP 端口：21115, 21116, 21117, 21118, 21119
- UDP 端口：21116

### 日志记录
在常规设置中启用日志记录即可将服务输出保存至 `/var/log/rustdeskd.log`。可通过“日志”选项卡实时查看。

### 数据库路径
数据库存储在 `/tmp/rustdesk_db_v2.sqlite3`。此目录不会持久化保存，设备重启后会清空。这是为了适应 OpenWrt 等嵌入式系统存储空间有限的特性。

## 客户端配置

启动服务后：

1. 进入 LuCI 界面，记下你的路由器 IP 地址
2. 从“服务状态”中复制 **Public Key (公钥)**
3. 在 RustDesk 客户端的网络设置中，填写：
   - **ID 服务器**：路由器的 IP:21116（或自定义端口）
   - **中继服务器**：路由器的 IP:21117（或自定义端口）
   - **Key (密钥)**：第 2 步复制的公钥

## 安全性说明

此插件实现了多层输入验证与过滤，以防止 Shell 注入攻击：

- **前端验证 (JavaScript)**：所有用户输入在保存至 UCI 配置之前，都会经过类型、字符范围和格式的严格校验。
- **后端验证 (Init 脚本)**：启动脚本会再次校验端口范围、字符合法性、URL 格式等，过滤任何潜在的危险字符（如 `;|&$\`(){}[]<>'"\\!` 等）。
- **RPC 后端验证 (ucode)**：严格限制了允许执行的服务动作白名单。
