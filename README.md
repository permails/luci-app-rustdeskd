# luci-app-rustdeskd

[English](README.md) | [中文](README_zh.md)

LuCI web interface for managing [RustDesk Server](https://github.com/rustdesk/rustdeskd) on OpenWrt.

RustDesk is a full-featured open source remote control alternative to TeamViewer and AnyDesk. This LuCI application provides a web-based interface to configure and manage the self-hosted RustDesk server components (hbbs and hbbr) on OpenWrt routers.

## Features

- **Service Management** - Start/Stop/Restart services directly from the UI
- **Core Downloader** - One-click download and update for the official RustDesk server core
- **Status & Logs** - Real-time service monitoring and log viewer
- **Configuration** - Easy management of ports, keys, and advanced options
- **Native Backup** - Fully integrates with OpenWrt's native backup/restore system
- **Security** - Built-in data validation and safe execution

## Requirements
- OpenWrt 23.05 or later with LuCI installed
- `luci-base`, `rpcd`, `rpcd-mod-ucode`

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

## Usage

1. Go to the **Core Management** tab to download the `hbbs` and `hbbr` binaries.
2. Enable the ID Server and Relay Server in the General Settings.
3. Apply settings and copy the **Public Key** for your clients.

## Acknowledgements

This project is heavily inspired by and pays tribute to the original [luci-app-rustdesk-server](https://github.com/superzjg/luci-app-rustdesk-server) created by [**superzjg**](https://github.com/superzjg). We sincerely thank superzjg for pioneering the OpenWrt integration for RustDesk Server!
