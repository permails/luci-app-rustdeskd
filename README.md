# luci-app-rustdeskd

[English](README.md) | [中文](README_zh.md)

LuCI web interface for managing [RustDesk Server](https://github.com/rustdesk/rustdeskd) on OpenWrt.

RustDesk is a full-featured open source remote control alternative to TeamViewer and AnyDesk. This LuCI application provides a web-based interface to configure and manage the self-hosted RustDesk server components (hbbs and hbbr) on OpenWrt routers.

## Features

- **Service Management** - Start/Stop/Restart services directly from the UI
- **Boot Enable/Disable** - Toggle service startup at boot
- **Status Monitoring** - Real-time status of HBBS and HBBR services with live polling
- **Public Key Display** - View and copy the generated public key for client configuration
- **Key Regeneration** - Regenerate encryption keys when needed
- **Log Viewer** - View service logs with auto-refresh and auto-scroll features
- **Firewall Hints** - Displays required ports for manual firewall configuration
- **Tabbed Configuration** - Organized settings for ID Server (hbbs) and Relay Server (hbbr)
- **Input Validation** - Validates paths, ports, and configuration values
- **i18n Ready** - Full translation support with POT template
- **Core Downloader** - Download and update the official RustDesk server core directly from the UI

## Architecture

```
luci-app-rustdeskd/
├── Makefile                      # OpenWrt package build file
├── htdocs/luci-static/resources/view/rustdeskd/
│   └── general.js                # Main UI view (JavaScript)
├── po/templates/
│   └── rustdeskd.pot       # Translation template
└── root/
    ├── etc/
    │   ├── config/rustdeskd        # UCI configuration
    │   ├── init.d/rustdeskd        # procd init script
    │   └── uci-defaults/50-luci-rustdeskd  # First-run setup
    └── usr/share/
        ├── luci/menu.d/luci-app-rustdeskd.json  # Menu entry
        └── rpcd/
            ├── acl.d/luci-app-rustdeskd.json    # ACL permissions
            └── ucode/rustdeskd.uc               # RPC backend
```

## Requirements

### OpenWrt Dependencies
- OpenWrt 23.05 or later with LuCI installed
- `luci-base` - LuCI core framework
- `rpcd` - RPC daemon
- `rpcd-mod-ucode` - ucode support for rpcd

### RustDesk Server Binaries
The RustDesk server binaries (`hbbs`, `hbbr`) must be installed separately. They are **not included** in this package. However, you can download them directly from the UI using the "Download / Update Core" button.

#### Manual Installation of RustDesk Server Binaries

1. **Download from GitHub Releases:**
   ```bash
   # Check your architecture
   uname -m
   
   # Download appropriate binaries from:
   # https://github.com/rustdesk/rustdeskd/releases
   
   # Example for aarch64:
   wget https://github.com/rustdesk/rustdeskd/releases/download/1.1.11/rustdeskd-linux-arm64v8.zip
   unzip rustdeskd-linux-arm64v8.zip
   cp amd64/hbbs amd64/hbbr /usr/bin/
   chmod +x /usr/bin/hbbs /usr/bin/hbbr
   ```

2. **Verify installation:**
   ```bash
   /usr/bin/hbbs --version
   /usr/bin/hbbr --version
   ```

## Installation

### From GitHub Releases (Recommended)
1. Download the latest `.ipk` package from the [Releases page](https://github.com/permails/luci-app-rustdeskd/releases).
2. Upload it to the `/tmp` directory on your router and install:
```bash
opkg update
opkg install /tmp/luci-app-rustdeskd_*.ipk
```

### From Source (Development)
```bash
# Clone the LuCI repository
git clone https://github.com/openwrt/luci.git
cd luci

# Build the package
make package/luci-app-rustdeskd/compile
```

### Manual Installation
1. Copy the application files to your OpenWrt device:
   ```bash
   # Copy htdocs to /www
   cp -r htdocs/luci-static /www/luci-static/
   
   # Copy root files
   cp -r root/* /
   
   # Set permissions
   chmod +x /etc/init.d/rustdeskd
   ```

2. Reload rpcd to register the new RPC methods:
   ```bash
   /etc/init.d/rpcd reload
   ```

3. Clear LuCI cache:
   ```bash
   rm -rf /tmp/luci-*
   ```

4. Access the interface at: **Services → RustDesk Server**

## Configuration

### Binary Location
The application expects `hbbs` and `hbbr` binaries to be installed in `/usr/bin`.

### Firewall Configuration
Firewall rules must be configured manually in **Network → Firewall → Traffic Rules**. The application displays the required ports in the Service Status section.

The standard RustDesk port layout is:

| Port | Protocol | Service | Calculation |
|------|----------|---------|-------------|
| HBBS-1 | TCP | NAT type test | server_port - 1 |
| HBBS | TCP/UDP | ID server / Hole punching | server_port |
| HBBS+2 | TCP | Web client support | server_port + 2 |
| HBBR | TCP | Relay server | relay_port |
| HBBR+2 | TCP | Web client support | relay_port + 2 |

**Example:** With default ports (`server_port=21116` and `relay_port=21117`):
- TCP ports: 21115, 21116, 21117, 21118, 21119
- UDP port: 21116

### Logging
Enable logging in General settings to write service output to `/var/log/rustdeskd.log`. View logs in real-time using the Logs tab.

### Database Location
The database is stored in `/tmp/rustdesk_db_v2.sqlite3`. This is a non-persistent location and will be cleared on reboot. This is intentional for embedded systems like OpenWrt where persistent storage may be limited.

## Client Configuration

After starting the service:

1. Go to the LuCI interface and note your router's IP address
2. Copy the **Public Key** from the Service Status section
3. In RustDesk client settings, configure:
   - **ID Server**: Your router's IP:21116 (or custom port if configured)
   - **Relay Server**: Your router's IP:21117 (or custom port if configured)
   - **Key**: The public key from step 2

## UCI Configuration Reference

The configuration is stored in `/etc/config/rustdeskd`:

```uci
config rustdeskd
    option enabled '1'              # Enable ID server (hbbs)
    option enabled_relay '1'        # Enable Relay server (hbbr)
    
    # HBBS options
    option server_port '21116'      # ID server port
    option server_key ''            # Custom key (optional)
    
    # HBBR options  
    option relay_port '21117'       # Relay server port
    
    # Environment variables
    option server_env_rust_log 'info'
```

## Security Considerations

This application implements multiple layers of input validation and sanitization to prevent shell injection attacks:

### Frontend Validation (JavaScript)
All user inputs are validated before being saved to UCI configuration:

| Field Type | Validation |
|------------|------------|
| Ports | Numeric only, range 1-65535, supports ranges and comma-separated lists |
| CIDR masks | Strict IP/prefix format validation |
| Keys | Alphanumeric and base64 characters only (`A-Za-z0-9+/=`) |
| URLs | Must start with `http://` or `https://`, no shell metacharacters |
| Paths | Must start with `/`, no shell metacharacters (`;|&$\`(){}[]<>'"\\!`) |
| Server lists | Alphanumeric, dots, colons, commas, hyphens, underscores only |
| Numeric fields | Use LuCI's built-in `uinteger` datatype |

### Backend Validation (Init Script)
The init script (`/etc/init.d/rustdeskd`) includes comprehensive validation functions that re-validate all configuration values before using them in shell commands:

- `validate_numeric()` - Ensures values contain only digits
- `validate_port()` - Validates port range (1-65535)
- `validate_path()` - Checks for shell metacharacters and requires leading `/`
- `validate_url()` - Validates URL format and rejects dangerous characters
- `validate_key()` - Allows only base64-safe characters
- `validate_server_list()` - Allows only hostname/IP-safe characters
- `validate_cidr()` - Allows only digits, dots, and slash
- `validate_log_level()` - Whitelist of valid log levels

Invalid values are rejected and logged with warnings to syslog.

### RPC Backend Validation (ucode)
The RPC backend (`rustdeskd.uc`) validates:
- `service_action`: Whitelist of allowed actions (`start`, `stop`, `restart`, `reload`, `enable`, `disable`)
- `get_log` lines parameter: Clamped to range 10-1000
