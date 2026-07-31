# luci-app-rustdeskd

[English](README.md) | [中文](README_zh.md)

**luci-app-rustdeskd** is a web interface for OpenWrt routers to manage your own **RustDesk Remote Desktop Server** (an open-source alternative to TeamViewer). By installing this plugin, you can easily turn your router into a private, self-hosted remote control server.

## Core Features

- **One-Click Core Installer**: No need for complex terminal commands. Download and update the official RustDesk server core (`hbbs` and `hbbr`) directly from the web UI.
- **Data Backup & Restore**: Fully integrates with OpenWrt's backup system. Easily backup and restore your encryption keys and database with a single click.
- **Visual Configuration**: Manage ports, keys, and view real-time service logs directly from the LuCI interface.

## Installation

1. Copy the application files to your OpenWrt device:
   ```bash
   cp -r htdocs/luci-static /www/luci-static/
   cp -r root/* /
   chmod +x /etc/init.d/rustdeskd
   ```

2. Reload rpcd and clear LuCI cache:
   ```bash
   /etc/init.d/rpcd reload
   rm -rf /tmp/luci-*
   ```

3. Access the interface at: **Services → RustDesk Server**
