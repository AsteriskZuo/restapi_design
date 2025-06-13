# IM 传输安全规范

## 1. HTTPS 配置

### 1.1 强制 HTTPS

```nginx
# Nginx 配置示例
server {
    listen 80;
    server_name api.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name api.example.com;

    ssl_certificate /path/to/cert.pem;
    ssl_certificate_key /path/to/key.pem;

    # 安全配置
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256;
    ssl_prefer_server_ciphers on;
}
```

### 1.2 证书要求

- 使用受信任的 CA 机构证书
- 证书有效期不超过 1 年
- 支持多域名（SAN）
- 密钥长度至少 2048 位

## 2. TLS 配置

### 2.1 协议版本

```json
{
  "tls": {
    "minVersion": "TLSv1.2",
    "maxVersion": "TLSv1.3",
    "disabledVersions": ["TLSv1.0", "TLSv1.1"]
  }
}
```

### 2.2 加密套件

```json
{
  "ciphers": [
    "ECDHE-ECDSA-AES128-GCM-SHA256",
    "ECDHE-RSA-AES128-GCM-SHA256",
    "ECDHE-ECDSA-AES256-GCM-SHA384",
    "ECDHE-RSA-AES256-GCM-SHA384"
  ]
}
```

## 3. 安全头部

### 3.1 响应头配置

```http
# 安全相关响应头
Strict-Transport-Security: max-age=31536000; includeSubDomains
X-Content-Type-Options: nosniff
X-Frame-Options: DENY
X-XSS-Protection: 1; mode=block
Content-Security-Policy: default-src 'self'
```

### 3.2 头部说明

| 头部  | 值                 | 说明               |
| ----- | ------------------ | ------------------ |
| HSTS  | max-age=31536000   | 强制 HTTPS         |
| X-CTO | nosniff            | 防止 MIME 类型嗅探 |
| X-FO  | DENY               | 防止点击劫持       |
| CSP   | default-src 'self' | 内容安全策略       |

## 4. 协议安全

### 4.1 WebSocket 安全

```javascript
// WebSocket 连接配置
const ws = new WebSocket("wss://api.example.com/ws", {
  protocols: ["v1.im.secure"],
  headers: {
    Authorization: "Bearer token123",
  },
});
```

### 4.2 长连接安全

```json
{
  "connection": {
    "heartbeat": 30,
    "timeout": 60,
    "reconnect": {
      "maxAttempts": 5,
      "backoff": "exponential"
    }
  }
}
```

## 5. 密钥管理

### 5.1 密钥轮换

```json
{
  "keyRotation": {
    "interval": "90d",
    "overlap": "7d",
    "algorithm": "RSA-2048"
  }
}
```

### 5.2 密钥存储

```json
{
  "keyStorage": {
    "type": "HSM",
    "backup": true,
    "encryption": "AES-256-GCM"
  }
}
```

## 6. 最佳实践

### 6.1 开发建议

- 使用安全的 TLS 配置
- 实现证书自动更新
- 定期安全扫描
- 监控异常连接

### 6.2 运维建议

- 定期更新证书
- 监控 TLS 版本
- 检查加密套件
- 记录安全事件
