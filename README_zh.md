# luci-app-rustdeskd

[English](README.md) | [中文](README_zh.md)

用于在 OpenWrt 上管理 [RustDesk Server](https://github.com/rustdesk/rustdeskd) 的 LuCI 网页界面。

RustDesk 是一款功能齐全的开源远程控制替代方案（类似 TeamViewer 和 AnyDesk）。此 LuCI 插件提供了一个基于 Web 的管理界面，用于在 OpenWrt 路由器上配置和管理自建的 RustDesk 服务器组件（hbbs 和 hbbr）。

## 功能特性

- **服务管理** - 直接从用户界面启动/停止/重启服务
- **核心下载** - 支持在界面一键下载和更新官方 RustDesk 服务器核心文件
- **状态与日志** - 实时监控服务运行状态并提供可视化的日志查看器
- **快速配置** - 轻松管理端口、加密密钥和中继参数
- **数据备份** - 完美整合 OpenWrt 原生备份恢复机制，一键备份数据库和密钥
- **安全可靠** - 内置严格的数据验证和防注入机制

## 运行要求
- OpenWrt 23.05 或更高版本（需安装 LuCI）
- 依赖项：`luci-base`, `rpcd`, `rpcd-mod-ucode`

## 安装说明

1. 将应用文件复制到 OpenWrt 设备：
   ```bash
   cp -r htdocs/luci-static /www/luci-static/
   cp -r root/* /
   chmod +x /etc/init.d/rustdeskd
   ```

2. 重新加载 rpcd 并清除缓存：
   ```bash
   /etc/init.d/rpcd reload
   rm -rf /tmp/luci-*
   ```

3. 访问界面：**服务 → RustDesk Server**（中文环境下可能名为 **远程桌面**）

## 使用指南

1. 进入界面的 **核心管理** 选项卡，点击下载核心程序（`hbbs` 和 `hbbr`）。
2. 在基础设置中，勾选启用 ID 服务器和中继服务器。
3. 保存并应用，从状态栏复制 **公钥 (Public Key)** 填入客户端即可连接。

## 致谢 (Acknowledgements)

本项目深受 [**superzjg**](https://github.com/superzjg) 开发的 [luci-app-rustdesk-server](https://github.com/superzjg/luci-app-rustdesk-server) 启发，并向其致敬。我们由衷感谢 superzjg 为开源社区做出的卓越贡献！
