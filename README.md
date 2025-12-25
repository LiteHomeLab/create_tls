# LightLink Certificate Setup

LightLink NATS 证书一键生成工具 - 为 NATS 消息系统快速生成 TLS 证书。

## 功能特性

- 一键生成 CA 根证书、NATS 服务器证书和客户端证书
- 自动创建带时间戳的输出文件夹
- 证书有效期 30 年
- 支持 NATS 服务器与客户端的 TLS 双向认证

## 快速开始

### 1. 安装依赖

本工具需要 OpenSSL。推荐安装 **Git for Windows**（自带 OpenSSL）：

- 下载地址：https://git-scm.com/download/win
- 验证安装：`openssl version`

### 2. 生成证书

双击运行 `setup-certs.bat`，或在命令行执行：

```bat
setup-certs.bat
```

### 3. 输出结果

脚本会生成一个时间戳文件夹：

```
certs-YYYY-MM-DD_HH-MM/
├── nats-server/     # 部署到 NATS 服务器
│   ├── ca.crt
│   ├── server.crt
│   ├── server.key
│   └── README.txt
└── client/          # 部署到所有客户端服务
    ├── ca.crt
    ├── client.crt
    ├── client.key
    └── README.txt
```

## 文件说明

| 文件 | 说明 |
|------|------|
| `setup-certs.bat` | 主脚本，生成所有证书 |
| `find-openssl.bat` | OpenSSL 诊断工具，用于定位 OpenSSL |
| `setup-certs-依赖说明.txt` | 详细的依赖安装说明（中文） |

## 证书架构

| 证书 | CN (Common Name) | 使用者 |
|------|------------------|--------|
| CA 根证书 | `LightLink CA` | 签发所有证书 |
| NATS 服务器 | `nats-server` | NATS 服务器专用 |
| 客户端 | `lightlink-client` | 所有客户端服务共享 |

> **重要**：客户端连接时必须验证 `server_name="nats-server"` 与服务器证书的 CN 匹配。

## NATS 服务器配置

```nginx
tls {
  ca_file:   "./nats-server/ca.crt"
  cert_file: "./nats-server/server.crt"
  key_file:  "./nats-server/server.key"
}
```

## 客户端连接配置

### Go
```go
tlsConfig := &client.TLSConfig{
    CaFile:     "client/ca.crt",
    CertFile:   "client/client.crt",
    KeyFile:    "client/client.key",
    ServerName: "nats-server",
}
```

### Python
```python
tls_config = TLSConfig(
    ca_file="client/ca.crt",
    cert_file="client/client.crt",
    key_file="client/client.key",
    server_name="nats-server"
)
```

### C#
```csharp
opts.SetCertificate("client/ca.crt", "client/client.crt", "client/client.key");
```

## 故障排查

如果 `setup-certs.bat` 报错找不到 OpenSSL，运行：

```bat
find-openssl.bat
```

该工具会自动搜索并测试系统中的 OpenSSL。

## 安全提示

- `.key` 文件包含私钥，请妥善保管
- 不要将私钥文件提交到版本控制系统
- 仅通过安全渠道传输证书文件

## 许可证

MIT License - 详见 [LICENSE](LICENSE)
