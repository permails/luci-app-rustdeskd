# luci-app-rustdeskd

[English](README.md) | [中文](README_zh.md)

**luci-app-rustdeskd** 是一款专为 OpenWrt 路由器打造的 **RustDesk 远程桌面服务器** 可视化管理插件。通过安装这个插件，你可以轻松地把自己的路由器变成一台私有的、完全由你掌控的远程控制中继服务器（开源的 TeamViewer 替代方案）。

## 核心功能

- **一键下载内核**：告别繁琐的命令行操作，支持在网页后台一键下载和更新官方 RustDesk 服务器核心（`hbbs` 和 `hbbr`）。
- **原生备份与恢复**：完美整合 OpenWrt 系统机制，一键无损备份/恢复你的服务器加密公私钥和连接数据，换机重装毫无压力。
- **傻瓜式可视化管理**：提供直观的运行状态、实时日志查看器，让修改端口、管理密钥变得轻而易举。

## 安装指南

1. 将文件直接放入 OpenWrt 对应目录：
   ```bash
   cp -r htdocs/luci-static /www/luci-static/
   cp -r root/* /
   chmod +x /etc/init.d/rustdeskd
   ```

2. 刷新服务和缓存：
   ```bash
   /etc/init.d/rpcd reload
   rm -rf /tmp/luci-*
   ```

3. 访问后台：**服务 → RustDesk Server** (中文环境下名为 **远程桌面**)
