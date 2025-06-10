# Webhooks 设计规范

> **重要性**: 中优先级，实现事件驱动架构和系统解耦  
> **适用范围**: 需要实时通知和事件驱动的业务场景

## 概览

Webhooks 是实现事件驱动架构的重要机制，允许系统在特定事件发生时主动推送数据到外部服务，实现松耦合的系统集成。

## 1. Webhooks 基础架构

### 1.1 核心概念

**Webhook 生命周期**:

```
事件触发 → 事件过滤 → 负载构建 → 签名生成 → HTTP投递 → 重试机制 → 状态更新
```

**关键组件**:

- **事件发布器**: 触发和发布事件
- **事件队列**: 缓存和排队事件
- **投递服务**: 执行 HTTP 投递
- **重试服务**: 处理失败重试
- **监控服务**: 跟踪投递状态

### 1.2 系统架构

```
┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│   业务服务   │───▶│  事件总线    │───▶│ Webhook服务 │
└─────────────┘    └─────────────┘    └─────────────┘
                                              │
                                              ▼
                                    ┌─────────────┐
                                    │  HTTP客户端  │
                                    └─────────────┘
                                              │
                                              ▼
                                    ┌─────────────┐
                                    │   目标端点   │
                                    └─────────────┘
```

## 2. 事件模型设计

### 2.1 事件类型定义

**标准事件类别**:

```json
{
  "user": {
    "user.created": "用户创建",
    "user.updated": "用户更新",
    "user.deleted": "用户删除",
    "user.login": "用户登录",
    "user.logout": "用户登出"
  },
  "order": {
    "order.created": "订单创建",
    "order.paid": "订单支付",
    "order.shipped": "订单发货",
    "order.delivered": "订单送达",
    "order.cancelled": "订单取消"
  },
  "payment": {
    "payment.succeeded": "支付成功",
    "payment.failed": "支付失败",
    "payment.refunded": "支付退款"
  }
}
```

### 2.2 事件负载格式

**标准事件结构**:

```json
{
  "id": "evt_1234567890",
  "type": "user.created",
  "version": "1.0",
  "timestamp": "2024-01-01T12:00:00Z",
  "source": "user-service",
  "specversion": "1.0",
  "datacontenttype": "application/json",
  "subject": "user/123",
  "data": {
    "userId": 123,
    "email": "user@example.com",
    "name": "张三",
    "createdAt": "2024-01-01T12:00:00Z"
  },
  "meta": {
    "correlationId": "corr-abc123",
    "requestId": "req-def456",
    "retryCount": 0
  }
}
```

### 2.3 事件版本控制

**版本演进策略**:

```json
{
  "v1.0": {
    "type": "user.created",
    "data": {
      "id": 123,
      "name": "张三",
      "email": "user@example.com"
    }
  },
  "v1.1": {
    "type": "user.created",
    "data": {
      "id": 123,
      "name": "张三",
      "email": "user@example.com",
      "avatar": "https://cdn.example.com/avatar.jpg" // 新增字段
    }
  },
  "v2.0": {
    "type": "user.created",
    "data": {
      "userId": 123, // 字段重命名
      "fullName": "张三", // 字段重命名
      "emailAddress": "user@example.com", // 字段重命名
      "profile": {
        // 结构调整
        "avatar": "https://cdn.example.com/avatar.jpg"
      }
    }
  }
}
```

## 3. Webhook 配置管理

### 3.1 端点注册

**注册 Webhook 端点**:

```http
POST /api/v1/webhooks
{
  "url": "https://api.partner.com/webhooks/events",
  "events": ["user.created", "user.updated", "order.*"],
  "version": "1.0",
  "secret": "whsec_abc123",
  "active": true,
  "meta": {
    "description": "Partner API webhook",
    "environment": "production"
  }
}
```

**响应**:

```http
HTTP/1.1 201 Created
{
  "id": "wh_1234567890",
  "url": "https://api.partner.com/webhooks/events",
  "events": ["user.created", "user.updated", "order.*"],
  "version": "1.0",
  "secret": "whsec_abc123",
  "active": true,
  "created_at": "2024-01-01T12:00:00Z",
  "updated_at": "2024-01-01T12:00:00Z"
}
```

### 3.2 事件过滤配置

**高级过滤规则**:

```json
{
  "webhookId": "wh_1234567890",
  "filters": {
    "events": ["user.created", "user.updated"],
    "conditions": {
      "user.created": {
        "data.email": { "endsWith": "@company.com" },
        "data.role": { "in": ["admin", "manager"] }
      },
      "user.updated": {
        "data.status": { "equals": "active" }
      }
    },
    "meta": {
      "source": { "equals": "user-service" }
    }
  }
}
```

### 3.3 批量配置管理

**批量 Webhook 操作**:

```http
# 批量创建
POST /api/v1/webhooks/batch
{
  "webhooks": [
    {
      "url": "https://api.partner1.com/webhooks",
      "events": ["user.created"]
    },
    {
      "url": "https://api.partner2.com/webhooks",
      "events": ["order.*"]
    }
  ]
}

# 批量更新状态
PATCH /api/v1/webhooks/batch
{
  "operation": "activate",
  "webhookIds": ["wh_123", "wh_456", "wh_789"]
}
```

## 4. 安全机制

### 4.1 签名验证

**HMAC-SHA256 签名**:

```javascript
// 服务端生成签名
function generateSignature(payload, secret) {
  const timestamp = Math.floor(Date.now() / 1000);
  const signedPayload = `${timestamp}.${payload}`;
  const signature = crypto
    .createHmac("sha256", secret)
    .update(signedPayload, "utf8")
    .digest("hex");

  return {
    timestamp,
    signature: `v1=${signature}`,
  };
}

// 发送Webhook
const { timestamp, signature } = generateSignature(
  JSON.stringify(event),
  secret,
);

await fetch(webhookUrl, {
  method: "POST",
  headers: {
    "Content-Type": "application/json",
    "X-Webhook-Timestamp": timestamp.toString(),
    "X-Webhook-Signature": signature,
  },
  body: JSON.stringify(event),
});
```

**客户端验证签名**:

```javascript
// 客户端验证签名
function verifySignature(payload, signature, timestamp, secret) {
  const currentTime = Math.floor(Date.now() / 1000);

  // 检查时间戳，防止重放攻击
  if (Math.abs(currentTime - timestamp) > 300) {
    // 5分钟容差
    throw new Error("Request timestamp too old");
  }

  const signedPayload = `${timestamp}.${payload}`;
  const expectedSignature = crypto
    .createHmac("sha256", secret)
    .update(signedPayload, "utf8")
    .digest("hex");

  const providedSignature = signature.replace("v1=", "");

  return crypto.timingSafeEqual(
    Buffer.from(expectedSignature, "hex"),
    Buffer.from(providedSignature, "hex"),
  );
}
```

### 4.2 访问控制

**IP 白名单**:

```json
{
  "webhookId": "wh_1234567890",
  "security": {
    "ipWhitelist": ["192.168.1.0/24", "10.0.0.0/8", "203.0.113.0/24"],
    "userAgent": "MyApp-Webhook/1.0",
    "rateLimiting": {
      "maxRequests": 1000,
      "timeWindow": "1h"
    }
  }
}
```

### 4.3 传输安全

**HTTPS 要求**:

```yaml
webhook_security:
  https_required: true
  tls_version: "1.2+"
  certificate_validation: true
  timeout_seconds: 30

  headers:
    user_agent: "MyApp-Webhook/1.0"
    x_api_version: "v1"
    x_source: "webhook-service"
```

## 5. 投递机制

### 5.1 重试策略

**指数退避重试**:

```javascript
class WebhookDelivery {
  constructor() {
    this.maxRetries = 5;
    this.baseDelay = 1000; // 1秒
    this.maxDelay = 60000; // 60秒
  }

  async deliverWebhook(webhook, event) {
    let retryCount = 0;
    let lastError;

    while (retryCount <= this.maxRetries) {
      try {
        const response = await this.sendRequest(webhook, event);

        if (response.status >= 200 && response.status < 300) {
          await this.markSuccess(webhook.id, event.id);
          return response;
        }

        throw new Error(`HTTP ${response.status}: ${response.statusText}`);
      } catch (error) {
        lastError = error;
        retryCount++;

        if (retryCount <= this.maxRetries) {
          const delay = this.calculateDelay(retryCount);
          await this.scheduleRetry(webhook, event, delay);
          await this.sleep(delay);
        }
      }
    }

    await this.markFailed(webhook.id, event.id, lastError);
    throw lastError;
  }

  calculateDelay(retryCount) {
    // 指数退避：1s, 2s, 4s, 8s, 16s
    const delay = Math.min(
      this.baseDelay * Math.pow(2, retryCount - 1),
      this.maxDelay,
    );

    // 添加随机抖动，避免雷群效应
    const jitter = Math.random() * 0.1 * delay;
    return delay + jitter;
  }
}
```

### 5.2 失败处理

**失败分类和处理**:

```javascript
const failureHandlers = {
  // 4xx错误 - 客户端错误，不重试
  "4xx": {
    retry: false,
    action: "disable_webhook",
    notification: true,
  },

  // 5xx错误 - 服务器错误，重试
  "5xx": {
    retry: true,
    maxRetries: 5,
    action: "retry_with_backoff",
  },

  // 超时错误 - 重试
  timeout: {
    retry: true,
    maxRetries: 3,
    action: "retry_with_shorter_timeout",
  },

  // 网络错误 - 重试
  network: {
    retry: true,
    maxRetries: 5,
    action: "retry_with_backoff",
  },
};
```

### 5.3 批量投递

**批量事件处理**:

```json
{
  "batchId": "batch_1234567890",
  "events": [
    {
      "id": "evt_001",
      "type": "user.created",
      "data": {...}
    },
    {
      "id": "evt_002",
      "type": "user.updated",
      "data": {...}
    }
  ],
  "meta": {
    "batchSize": 2,
    "processingTime": "2024-01-01T12:00:00Z"
  }
}
```

## 6. 监控和调试

### 6.1 投递状态跟踪

**状态枚举**:

```json
{
  "deliveryStates": {
    "pending": "等待投递",
    "sending": "正在发送",
    "delivered": "投递成功",
    "failed": "投递失败",
    "retrying": "重试中",
    "disabled": "已禁用"
  }
}
```

**投递日志**:

```json
{
  "deliveryId": "del_1234567890",
  "webhookId": "wh_1234567890",
  "eventId": "evt_1234567890",
  "status": "delivered",
  "attempts": [
    {
      "attemptNumber": 1,
      "timestamp": "2024-01-01T12:00:00Z",
      "httpStatus": 500,
      "responseTime": 1500,
      "error": "Internal Server Error"
    },
    {
      "attemptNumber": 2,
      "timestamp": "2024-01-01T12:01:00Z",
      "httpStatus": 200,
      "responseTime": 250,
      "success": true
    }
  ],
  "totalDuration": 61500,
  "finalStatus": "delivered"
}
```

### 6.2 性能指标

**关键指标监控**:

```javascript
const webhookMetrics = {
  delivery: {
    successRate: 0.98, // 投递成功率：98%
    averageLatency: 250, // 平均延迟：250ms
    p95Latency: 800, // P95延迟：800ms
    throughput: 1000, // 吞吐量：1000 webhooks/min
  },

  reliability: {
    retryRate: 0.15, // 重试率：15%
    failureRate: 0.02, // 失败率：2%
    timeoutRate: 0.005, // 超时率：0.5%
  },

  endpoints: {
    totalActive: 150, // 活跃端点数
    averageEventsPerEndpoint: 50, // 平均事件数/端点
    endpointUptime: 0.995, // 端点可用率：99.5%
  },
};
```

### 6.3 调试工具

**Webhook 测试端点**:

```http
POST /api/v1/webhooks/test
{
  "webhookId": "wh_1234567890",
  "eventType": "user.created",
  "testData": {
    "userId": 999,
    "email": "test@example.com",
    "name": "测试用户"
  }
}
```

**事件重放功能**:

```http
POST /api/v1/webhooks/{webhookId}/replay
{
  "eventIds": ["evt_123", "evt_456"],
  "timeRange": {
    "start": "2024-01-01T00:00:00Z",
    "end": "2024-01-01T23:59:59Z"
  }
}
```

## 7. 客户端最佳实践

### 7.1 接收端点实现

**Node.js 示例**:

```javascript
const express = require('express');
const crypto = require('crypto');
const app = express();

// Webhook接收端点
app.post('/webhooks/events', express.raw({type: 'application/json'}), (req, res) => {
  const signature = req.headers['x-webhook-signature'];
  const timestamp = req.headers['x-webhook-timestamp'];
  const payload = req.body;

  try {
    // 验证签名
    if (!verifySignature(payload, signature, timestamp, webhookSecret)) {
      return res.status(401).send('Invalid signature');
    }

    // 解析事件
    const event = JSON.parse(payload);

    // 幂等性检查
    if (await isEventProcessed(event.id)) {
      return res.status(200).send('Event already processed');
    }

    // 处理事件
    await processEvent(event);

    // 标记为已处理
    await markEventProcessed(event.id);

    res.status(200).send('OK');

  } catch (error) {
    console.error('Webhook processing error:', error);
    res.status(500).send('Internal Server Error');
  }
});

async function processEvent(event) {
  switch (event.type) {
    case 'user.created':
      await handleUserCreated(event.data);
      break;
    case 'order.paid':
      await handleOrderPaid(event.data);
      break;
    default:
      console.log(`Unhandled event type: ${event.type}`);
  }
}
```

### 7.2 幂等性处理

**事件去重机制**:

```javascript
class EventProcessor {
  constructor() {
    this.processedEvents = new Set(); // 或使用Redis
  }

  async processWebhook(event) {
    // 幂等性检查
    if (this.processedEvents.has(event.id)) {
      console.log(`Event ${event.id} already processed`);
      return { status: "already_processed" };
    }

    try {
      // 处理事件
      const result = await this.handleEvent(event);

      // 标记为已处理
      this.processedEvents.add(event.id);

      return { status: "processed", result };
    } catch (error) {
      console.error(`Failed to process event ${event.id}:`, error);
      throw error;
    }
  }
}
```

### 7.3 错误处理和重试

**客户端错误响应**:

```javascript
app.post("/webhooks/events", async (req, res) => {
  try {
    await processWebhook(req.body);
    res.status(200).json({ message: "Success" });
  } catch (error) {
    if (error.code === "TEMPORARY_ERROR") {
      // 临时错误，返回5xx让服务端重试
      res.status(503).json({
        error: "Temporary error, please retry",
        retryAfter: 60,
      });
    } else if (error.code === "VALIDATION_ERROR") {
      // 验证错误，返回4xx不要重试
      res.status(400).json({
        error: "Invalid webhook data",
        details: error.details,
      });
    } else {
      // 其他错误
      res.status(500).json({ error: "Internal error" });
    }
  }
});
```

## 8. 管理和运维

### 8.1 Webhook 管理 API

**获取 Webhook 列表**:

```http
GET /api/v1/webhooks?status=active&event_type=user.*
{
  "webhooks": [
    {
      "id": "wh_1234567890",
      "url": "https://api.partner.com/webhooks",
      "events": ["user.created", "user.updated"],
      "status": "active",
      "created_at": "2024-01-01T12:00:00Z"
    }
  ],
  "pagination": {
    "page": 1,
    "limit": 10,
    "total": 25
  }
}
```

**更新 Webhook 配置**:

```http
PATCH /api/v1/webhooks/{webhookId}
{
  "events": ["user.created", "user.updated", "user.deleted"],
  "active": true,
  "meta": {
    "description": "Updated webhook configuration"
  }
}
```

### 8.2 批量运维操作

**批量禁用 Webhook**:

```http
POST /api/v1/webhooks/bulk-actions
{
  "action": "disable",
  "filters": {
    "url_pattern": "*.old-domain.com",
    "last_success_before": "2024-01-01T00:00:00Z"
  }
}
```

### 8.3 告警和通知

**告警规则配置**:

```yaml
webhook_alerts:
  - name: high_failure_rate
    condition: failure_rate > 0.10
    duration: 5m
    notification: email

  - name: endpoint_unreachable
    condition: consecutive_failures > 10
    duration: 1m
    notification: slack

  - name: processing_delay
    condition: avg_delivery_time > 5000ms
    duration: 10m
    notification: pagerduty
```

## 总结

完善的 Webhooks 系统需要考虑：

1. **可靠投递**: 实现重试机制和失败处理
2. **安全性**: 签名验证和访问控制
3. **可观测性**: 全面的监控和日志记录
4. **可扩展性**: 支持大规模事件处理
5. **易用性**: 提供友好的管理界面和 API
6. **兼容性**: 支持事件格式版本演进

通过系统化的 Webhooks 设计，可以构建高效、可靠的事件驱动架构，实现系统间的松耦合集成。
