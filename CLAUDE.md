# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 开发规则 / Development Rules

- **使用中文回答** - 用中文与用户交流，代码注释和文档可以使用英文
- **开发环境** - 当前为 Windows 系统，批处理脚本(.bat)是主要脚本类型
- **BAT 脚本编码** - BAT 脚本中不要出现中文字符（注释和文档输出除外），避免编码问题
- **修复原则** - 修复脚本时在原有文件上修改，非必需不要创建新脚本

## Project Overview

This is a **LightLink** certificate generation toolkit for NATS messaging system. It provides Windows batch scripts to automatically generate TLS certificates for secure NATS server-client communication.

The certificates are organized for:
- **NATS Server** - Server-side certificates (`nats-server/`)
- **Client Services** - Shared client certificates for all services (`client/`)

All certificates are signed by a self-generated CA root certificate.

## Commands

### Generate All Certificates
```bat
setup-certs.bat
```
This is the main entry point. Double-click to run or execute from command line. It:
- Creates a timestamped output folder (`certs-YYYY-MM-DD_HH-MM/`)
- Generates CA, server, and client certificates
- Organizes files into `nats-server/` and `client/` folders
- Creates README documentation in each folder

### Diagnose OpenSSL Installation
```bat
find-openssl.bat
```
Use this if `setup-certs.bat` fails to find OpenSSL. It searches common installation paths and validates the OpenSSL binary.

## Certificate Architecture

The script generates a certificate chain with these components:

| Certificate | CN (Common Name) | Used By | Files |
|-------------|------------------|---------|-------|
| CA Root | `LightLink CA` | Signs all certs | `ca.key`, `ca.crt` |
| NATS Server | `nats-server` | NATS server only | `server.key`, `server.crt` |
| Client | `lightlink-client` | All client services | `client.key`, `client.crt` |

**Important**: Clients must verify `server_name="nats-server"` to match the server certificate's CN.

## Output Structure

Each run creates a new timestamped folder:
```
certs-YYYY-MM-DD_HH-MM/
├── README.txt           # Main documentation
├── ca.crt, ca.key       # CA files (kept at root, not deployed)
├── nats-server/         # Deploy to NATS server
│   ├── ca.crt
│   ├── server.crt
│   ├── server.key
│   └── README.txt
└── client/              # Deploy to ANY client service
    ├── ca.crt
    ├── client.crt
    ├── client.key
    └── README.txt
```

## OpenSSL Dependency

Scripts automatically detect OpenSSL from these locations (in order):
1. System PATH
2. `C:\Program Files\Git\mingw64\bin\openssl.exe`
3. `C:\Program Files\Git\usr\bin\openssl.exe`
4. `C:\Program Files\OpenSSL-Win64\bin\openssl.exe`

If OpenSSL is not found, the script provides download instructions.

## Certificate Details

- **Validity**: 30 years (10950 days)
- **Key size**: 2048-bit RSA
- **CA self-signed**: Yes
- **Server cert CN**: `nats-server` (clients must verify this)

## Security Practices

- Private keys (`.key` files) are never committed to version control
- The `ca.key` remains on the certificate generation machine only
- The `nats-server/` folder contains the server's private key - deploy securely
- The `client/` folder contains the client's private key - all services share the same client certificate

## Language Notes

The repository contains both English and Chinese documentation:
- `setup-certs-依赖说明.txt` - Chinese dependency instructions
- Script comments and generated READMEs are in English
