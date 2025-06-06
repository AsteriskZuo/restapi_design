# 监控和可观测性

> **重要性**: 中优先级，保证系统可维护性和故障快速定位  
> **适用范围**: 所有 API 服务和基础设施

## 概览

可观测性是现代 API 服务的重要组成部分，通过日志、指标和追踪三大支柱，帮助开发运维团队快速发现问题、定位根因、优化性能。

## 1. 监控体系架构

### 1.1 监控层次

```
应用层监控
├── API响应时间
├── 业务指标
├── 错误率统计
└── 用户行为分析

服务层监控
├── 服务健康状态
├── 依赖服务状态
├── 资源使用情况
└── 性能瓶颈分析

基础设施监控
├── 服务器资源
├── 网络状态
├── 数据库性能
└── 缓存命中率
```

### 1.2 可观测性三大支柱

#### 日志 (Logs)

- 结构化日志记录
- 日志聚合和查询
- 日志告警规则

#### 指标 (Metrics)

- 实时性能指标
- 业务指标监控
- 趋势分析

#### 追踪 (Tracing)

- 分布式链路追踪
- 请求生命周期
- 性能瓶颈定位

## 2. 请求追踪

### 2.1 请求 ID 生成

**唯一请求标识**:

```javascript
// 请求ID生成策略
function generateRequestId() {
  // 方式1：UUID v4
  return crypto.randomUUID();

  // 方式2：时间戳 + 随机数
  const timestamp = Date.now().toString(36);
  const random = Math.random().toString(36).substring(2);
  return `req-${timestamp}-${random}`;

  // 方式3：雪花算法（分布式环境）
  return snowflake.nextId();
}
```

**请求头传递**:

```http
# 客户端发送
GET /api/v1/users/123
X-Request-ID: req-1640995200-abc123

# 服务器响应
HTTP/1.1 200 OK
X-Request-ID: req-1640995200-abc123
X-Response-Time: 123ms
```

### 2.2 用户会话追踪

**会话标识**:

```http
# 会话追踪头部
X-Session-ID: sess-1640995200-def456
X-User-ID: user-123
X-Client-Version: app/1.2.3
X-Platform: ios
```

**追踪上下文传递**:

```javascript
// Express中间件示例
function requestTracking(req, res, next) {
  // 生成或提取请求ID
  const requestId = req.headers["x-request-id"] || generateRequestId();
  const sessionId = req.headers["x-session-id"];
  const userId = req.user?.id;

  // 设置追踪上下文
  req.context = {
    requestId,
    sessionId,
    userId,
    startTime: Date.now(),
    userAgent: req.headers["user-agent"],
    ip: req.ip,
  };

  // 设置响应头
  res.set("X-Request-ID", requestId);

  next();
}
```

## 3. 结构化日志

### 3.1 日志级别定义

| 级别      | 用途                   | 示例场景               |
| --------- | ---------------------- | ---------------------- |
| **ERROR** | 系统错误，需要立即关注 | 系统异常、业务流程失败 |
| **WARN**  | 警告信息，可能影响功能 | 依赖服务超时、配置异常 |
| **INFO**  | 重要业务事件           | 用户登录、订单创建     |
| **DEBUG** | 调试信息，开发环境使用 | 变量值、流程步骤       |

### 3.2 日志格式规范

**标准日志格式**:

```json
{
  "timestamp": "2024-01-01T12:00:00.123Z",
  "level": "INFO",
  "service": "user-api",
  "version": "v1.2.3",
  "environment": "production",
  "requestId": "req-1640995200-abc123",
  "sessionId": "sess-1640995200-def456",
  "userId": 123,
  "operation": "user.create",
  "message": "用户创建成功",
  "data": {
    "userId": 123,
    "email": "user@example.com",
    "ip": "192.168.1.100"
  },
  "duration": 456,
  "tags": ["user", "registration"]
}
```

### 3.3 业务日志示例

#### 用户操作日志

```json
{
  "timestamp": "2024-01-01T12:00:00.123Z",
  "level": "INFO",
  "operation": "user.login",
  "message": "用户登录成功",
  "requestId": "req-abc123",
  "userId": 123,
  "data": {
    "email": "user@example.com",
    "ip": "192.168.1.100",
    "userAgent": "Mozilla/5.0...",
    "loginMethod": "password"
  }
}
```

#### 错误日志

```json
{
  "timestamp": "2024-01-01T12:00:00.123Z",
  "level": "ERROR",
  "operation": "order.create",
  "message": "订单创建失败",
  "requestId": "req-def456",
  "userId": 123,
  "error": {
    "code": "BUSINESS_INSUFFICIENT_STOCK",
    "message": "库存不足",
    "stack": "Error: 库存不足\n    at OrderService.create...",
    "data": {
      "productId": 456,
      "requestedQuantity": 5,
      "availableQuantity": 2
    }
  }
}
```

#### 性能日志

```json
{
  "timestamp": "2024-01-01T12:00:00.123Z",
  "level": "INFO",
  "operation": "api.request",
  "message": "API请求完成",
  "requestId": "req-ghi789",
  "performance": {
    "method": "GET",
    "url": "/api/v1/users/123",
    "statusCode": 200,
    "duration": 156,
    "dbQueryTime": 45,
    "cacheHit": true,
    "responseSize": 1024
  }
}
```

## 4. 性能指标监控

### 4.1 核心 API 指标

#### 响应时间指标

```javascript
// 响应时间监控
const responseTime = {
  p50: 120, // 50百分位：120ms
  p90: 250, // 90百分位：250ms
  p95: 400, // 95百分位：400ms
  p99: 800, // 99百分位：800ms
  avg: 180, // 平均值：180ms
  max: 1200, // 最大值：1200ms
};
```

#### 吞吐量指标

```javascript
// 吞吐量监控
const throughput = {
  rps: 150, // 每秒请求数
  rpm: 9000, // 每分钟请求数
  dailyRequests: 12960000, // 日请求量
  concurrentUsers: 500, // 并发用户数
};
```

#### 错误率指标

```javascript
// 错误率监控
const errorRate = {
  total: 0.02, // 总错误率：2%
  "4xx": 0.015, // 4xx错误率：1.5%
  "5xx": 0.005, // 5xx错误率：0.5%
  timeout: 0.001, // 超时错误率：0.1%
  byEndpoint: {
    "/api/v1/users": 0.01,
    "/api/v1/orders": 0.03,
  },
};
```

### 4.2 业务指标监控

#### 用户行为指标

```javascript
const userMetrics = {
  activeUsers: {
    daily: 10000,
    weekly: 50000,
    monthly: 150000,
  },
  sessionMetrics: {
    avgDuration: 1800, // 平均会话时长（秒）
    bounceRate: 0.25, // 跳出率
    pageViews: 5.2, // 平均页面浏览量
  },
  conversionRate: 0.03, // 转化率：3%
};
```

#### 业务流程指标

```javascript
const businessMetrics = {
  registration: {
    attempts: 1000,
    success: 950,
    successRate: 0.95,
  },
  payment: {
    attempts: 500,
    success: 485,
    successRate: 0.97,
    avgAmount: 299.99,
  },
  orderFulfillment: {
    avgProcessingTime: 3600, // 平均处理时间（秒）
    onTimeDeliveryRate: 0.92, // 准时交付率
  },
};
```

### 4.3 系统资源监控

#### 服务器资源

```javascript
const systemMetrics = {
  cpu: {
    usage: 0.65, // CPU使用率：65%
    loadAvg: [1.2, 1.5, 1.8], // 负载平均值
  },
  memory: {
    usage: 0.72, // 内存使用率：72%
    available: 2048, // 可用内存：2GB
    cached: 1024, // 缓存内存：1GB
  },
  disk: {
    usage: 0.45, // 磁盘使用率：45%
    iops: 150, // IOPS
    latency: 5.2, // 磁盘延迟：5.2ms
  },
  network: {
    inbound: 100, // 入站流量：100Mbps
    outbound: 80, // 出站流量：80Mbps
    connections: 500, // 活跃连接数
  },
};
```

## 5. 健康检查

### 5.1 健康检查端点

**基础健康检查**:

```http
GET /health
{
  "status": "healthy",
  "timestamp": "2024-01-01T12:00:00Z",
  "version": "v1.2.3",
  "uptime": 86400,
  "environment": "production"
}
```

**详细健康检查**:

```http
GET /health/detailed
{
  "status": "healthy",
  "timestamp": "2024-01-01T12:00:00Z",
  "checks": {
    "database": {
      "status": "healthy",
      "responseTime": 12,
      "connections": {
        "active": 5,
        "idle": 10,
        "max": 20
      }
    },
    "redis": {
      "status": "healthy",
      "responseTime": 3,
      "memory": {
        "used": "512MB",
        "max": "2GB"
      }
    },
    "externalAPI": {
      "status": "degraded",
      "responseTime": 1200,
      "lastError": "Connection timeout",
      "errorCount": 3
    }
  }
}
```

### 5.2 就绪状态检查

**就绪检查端点**:

```http
GET /ready
{
  "ready": true,
  "timestamp": "2024-01-01T12:00:00Z",
  "dependencies": {
    "database": "ready",
    "cache": "ready",
    "messageQueue": "ready"
  }
}
```

### 5.3 存活状态检查

**存活检查端点**:

```http
GET /alive
{
  "alive": true,
  "timestamp": "2024-01-01T12:00:00Z",
  "pid": 1234,
  "uptime": 86400
}
```

## 6. 告警规则

### 6.1 告警级别定义

| 级别         | 描述         | 响应时间  | 通知方式         |
| ------------ | ------------ | --------- | ---------------- |
| **Critical** | 系统不可用   | 立即      | 电话、短信、邮件 |
| **High**     | 严重性能问题 | 5 分钟内  | 短信、邮件       |
| **Medium**   | 性能下降     | 15 分钟内 | 邮件、即时消息   |
| **Low**      | 预警信息     | 1 小时内  | 邮件             |

### 6.2 告警规则配置

```yaml
# 告警规则示例
alerts:
  # 错误率告警
  - name: high_error_rate
    condition: error_rate_5m > 0.05
    severity: high
    duration: 5m
    message: "API错误率过高: {{ $value }}%"

  # 响应时间告警
  - name: slow_response
    condition: response_time_p95_5m > 2000
    severity: medium
    duration: 10m
    message: "API响应时间过慢: P95 {{ $value }}ms"

  # 服务不可用告警
  - name: service_down
    condition: up == 0
    severity: critical
    duration: 1m
    message: "服务不可用: {{ $labels.service }}"

  # 数据库连接告警
  - name: database_connection_high
    condition: db_connections_active / db_connections_max > 0.8
    severity: high
    duration: 5m
    message: "数据库连接使用率过高: {{ $value }}%"
```

### 6.3 告警抑制和分组

```yaml
# 告警抑制规则
inhibit_rules:
  - source_match:
      severity: "critical"
    target_match:
      severity: "warning"
    equal: ["service", "instance"]

# 告警分组规则
route:
  group_by: ["service", "severity"]
  group_wait: 30s
  group_interval: 5m
  repeat_interval: 12h
```

## 7. 监控仪表板

### 7.1 服务概览仪表板

**关键指标展示**:

- 请求量趋势图
- 响应时间分布
- 错误率变化
- 服务可用性

### 7.2 API 性能仪表板

**详细性能指标**:

- 各端点响应时间对比
- 吞吐量分布
- 错误类型分析
- 缓存命中率

### 7.3 业务监控仪表板

**业务关键指标**:

- 用户注册转化漏斗
- 交易成功率
- 客户满意度指标
- 收入指标

## 8. 日志管理

### 8.1 日志收集

**日志收集策略**:

```yaml
# Fluent Bit配置示例
[INPUT]
    Name tail
    Path /var/log/app/*.log
    Tag app.logs
    Parser json

[FILTER]
    Name modify
    Match app.logs
    Add service api-gateway
    Add environment production

[OUTPUT]
    Name elasticsearch
    Match app.logs
    Host elasticsearch.example.com
    Index api-logs
```

### 8.2 日志存储和保留

**日志保留策略**:

- **热数据**（7 天）：快速检索，SSD 存储
- **温数据**（30 天）：常规检索，HDD 存储
- **冷数据**（1 年）：归档存储，压缩保存
- **法规数据**（5 年）：合规存储，加密保存

### 8.3 日志分析

**常用查询示例**:

```
# 查找特定用户的操作记录
userId:123 AND timestamp:[now-1h TO now]

# 查找所有错误日志
level:ERROR AND timestamp:[now-24h TO now]

# 查找慢查询
operation:"db.query" AND duration:>1000

# 查找特定API的调用情况
operation:"api.request" AND data.url:"/api/v1/users/*"
```

## 9. 性能分析

### 9.1 分布式追踪

**追踪 span 示例**:

```json
{
  "traceId": "trace-abc123",
  "spanId": "span-def456",
  "parentSpanId": "span-ghi789",
  "operationName": "user.create",
  "startTime": 1640995200000,
  "duration": 156,
  "tags": {
    "http.method": "POST",
    "http.url": "/api/v1/users",
    "http.status_code": 201,
    "user.id": 123
  },
  "logs": [
    {
      "timestamp": 1640995200050,
      "message": "Validating user data"
    },
    {
      "timestamp": 1640995200100,
      "message": "Saving to database"
    }
  ]
}
```

### 9.2 性能瓶颈分析

**瓶颈识别指标**:

- 数据库查询时间占比
- 外部 API 调用时间
- 序列化/反序列化时间
- 网络传输时间

### 9.3 性能优化建议

**基于监控数据的优化**:

- 缓存命中率低 → 优化缓存策略
- 数据库查询慢 → 添加索引或优化查询
- 响应时间长 → 考虑异步处理
- 错误率高 → 检查业务逻辑和验证

## 10. 监控最佳实践

### 10.1 监控指标选择

**黄金信号**:

1. **延迟**：请求处理时间
2. **流量**：请求速率
3. **错误**：错误率
4. **饱和度**：资源使用率

### 10.2 告警策略

**告警设计原则**:

- 基于用户影响设计告警
- 避免告警疲劳
- 提供可操作的告警信息
- 建立告警升级机制

### 10.3 可观测性文化

**团队实践**:

- 在开发阶段考虑可观测性
- 建立监控和告警标准
- 定期回顾和优化监控策略
- 培养数据驱动的决策文化

## 总结

完善的监控和可观测性体系需要：

1. **全面覆盖**：从应用到基础设施的全栈监控
2. **实时响应**：快速发现和定位问题
3. **预防性监控**：通过趋势分析预防问题
4. **用户导向**：关注用户体验相关指标
5. **持续改进**：基于监控数据持续优化系统
