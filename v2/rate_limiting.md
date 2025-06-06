# API 频率限制设计指南

> **适用人群**: 需要实现 API 频率限制的架构师和后端开发者  
> **前置知识**: 基础的 HTTP 协议、缓存系统、分布式系统概念

## 概览

API 频率限制（Rate Limiting）是一个复杂的系统工程，涉及算法选择、架构设计、业务策略等多个维度。本文档提供全面的设计指导。

## 1. 为什么需要频率限制

### 1.1 保护系统资源

```
防止系统过载
├── CPU 使用率控制
├── 内存使用控制
├── 数据库连接数限制
└── 网络带宽保护
```

### 1.2 防范恶意攻击

```
安全防护
├── DDoS 攻击防护
├── 爬虫限制
├── API 滥用防止
└── 暴力破解防护
```

### 1.3 商业模式支持

```
业务分级
├── 免费用户限制
├── 付费用户特权
├── API 套餐差异化
└── 公平使用原则
```

## 2. 频率限制算法

### 2.1 固定窗口算法（Fixed Window）

#### 原理

在固定时间窗口内限制请求数量。

```javascript
class FixedWindowRateLimit {
  constructor(limit, windowSize) {
    this.limit = limit; // 限制数量
    this.windowSize = windowSize; // 窗口大小（秒）
    this.windows = new Map(); // 存储窗口数据
  }

  isAllowed(key) {
    const now = Date.now();
    const windowStart =
      Math.floor(now / (this.windowSize * 1000)) * this.windowSize * 1000;

    if (!this.windows.has(key)) {
      this.windows.set(key, { start: windowStart, count: 0 });
    }

    const window = this.windows.get(key);

    // 新窗口开始
    if (window.start < windowStart) {
      window.start = windowStart;
      window.count = 0;
    }

    if (window.count >= this.limit) {
      return false; // 超过限制
    }

    window.count++;
    return true;
  }
}
```

#### 优缺点

**优点：**

- 实现简单
- 内存消耗低
- 性能高

**缺点：**

- 边界效应：窗口切换时可能出现请求激增
- 不够平滑

### 2.2 滑动窗口算法（Sliding Window）

#### 原理

使用多个小的时间窗口，平滑地限制请求速率。

```javascript
class SlidingWindowRateLimit {
  constructor(limit, windowSize, buckets = 10) {
    this.limit = limit;
    this.windowSize = windowSize;
    this.buckets = buckets;
    this.bucketSize = windowSize / buckets;
    this.windows = new Map();
  }

  isAllowed(key) {
    const now = Date.now();

    if (!this.windows.has(key)) {
      this.windows.set(key, []);
    }

    const window = this.windows.get(key);
    const cutoff = now - this.windowSize * 1000;

    // 清理过期的桶
    while (window.length > 0 && window[0].timestamp < cutoff) {
      window.shift();
    }

    // 计算当前总数
    const totalCount = window.reduce((sum, bucket) => sum + bucket.count, 0);

    if (totalCount >= this.limit) {
      return false;
    }

    // 添加到当前桶
    const bucketIndex = Math.floor(now / (this.bucketSize * 1000));
    const existingBucket = window.find((b) => b.index === bucketIndex);

    if (existingBucket) {
      existingBucket.count++;
    } else {
      window.push({
        index: bucketIndex,
        timestamp: now,
        count: 1,
      });
    }

    return true;
  }
}
```

### 2.3 令牌桶算法（Token Bucket）

#### 原理

以恒定速率向桶中添加令牌，请求消耗令牌。

```javascript
class TokenBucketRateLimit {
  constructor(capacity, refillRate) {
    this.capacity = capacity; // 桶容量
    this.refillRate = refillRate; // 令牌添加速率（个/秒）
    this.buckets = new Map();
  }

  isAllowed(key, tokens = 1) {
    const now = Date.now();

    if (!this.buckets.has(key)) {
      this.buckets.set(key, {
        tokens: this.capacity,
        lastRefill: now,
      });
    }

    const bucket = this.buckets.get(key);

    // 计算应该添加的令牌数
    const timePassed = (now - bucket.lastRefill) / 1000;
    const tokensToAdd = Math.floor(timePassed * this.refillRate);

    if (tokensToAdd > 0) {
      bucket.tokens = Math.min(this.capacity, bucket.tokens + tokensToAdd);
      bucket.lastRefill = now;
    }

    if (bucket.tokens >= tokens) {
      bucket.tokens -= tokens;
      return true;
    }

    return false;
  }
}
```

#### 优点

- 允许突发流量
- 平滑的速率控制
- 灵活的配置

### 2.4 漏桶算法（Leaky Bucket）

#### 原理

请求进入桶中排队，以恒定速率处理。

```javascript
class LeakyBucketRateLimit {
  constructor(capacity, leakRate) {
    this.capacity = capacity; // 桶容量
    this.leakRate = leakRate; // 漏出速率（个/秒）
    this.buckets = new Map();
  }

  isAllowed(key) {
    const now = Date.now();

    if (!this.buckets.has(key)) {
      this.buckets.set(key, {
        level: 0,
        lastLeak: now,
      });
    }

    const bucket = this.buckets.get(key);

    // 计算漏出的请求数
    const timePassed = (now - bucket.lastLeak) / 1000;
    const leaked = Math.floor(timePassed * this.leakRate);

    if (leaked > 0) {
      bucket.level = Math.max(0, bucket.level - leaked);
      bucket.lastLeak = now;
    }

    if (bucket.level >= this.capacity) {
      return false; // 桶已满
    }

    bucket.level++;
    return true;
  }
}
```

### 2.5 算法选择指南

| 算法     | 适用场景             | 优点                 | 缺点       |
| -------- | -------------------- | -------------------- | ---------- |
| 固定窗口 | 简单限制、性能要求高 | 实现简单、性能好     | 边界效应   |
| 滑动窗口 | 需要平滑限制         | 更精确、避免边界效应 | 复杂度较高 |
| 令牌桶   | 允许突发、灵活控制   | 支持突发、灵活       | 实现复杂   |
| 漏桶     | 严格速率控制         | 平滑输出             | 不支持突发 |

## 3. 限制维度设计

### 3.1 单维度限制

```yaml
# 全局限制
global:
  requests_per_minute: 10000

# 用户限制
user:
  requests_per_minute: 100

# IP限制
ip:
  requests_per_minute: 1000

# API端点限制
endpoint:
  "/api/v1/search":
    requests_per_minute: 50
  "/api/v1/upload":
    requests_per_minute: 10
```

### 3.2 多维度组合限制

```yaml
# 用户+端点组合
user_endpoint:
  "user:123:/api/v1/search":
    requests_per_minute: 30

# IP+端点组合
ip_endpoint:
  "192.168.1.1:/api/v1/upload":
    requests_per_minute: 5

# 层级限制（优先级）
hierarchy:
  - user_endpoint # 最高优先级
  - user
  - ip_endpoint
  - endpoint
  - global # 最低优先级
```

### 3.3 动态策略

```javascript
class DynamicRateLimit {
  getRateLimit(context) {
    const { user, ip, endpoint, time } = context;

    // 基础限制
    let limit = this.getBaseLimit(endpoint);

    // 用户等级调整
    if (user.isPremium) {
      limit *= 5;
    } else if (user.isFree) {
      limit *= 0.5;
    }

    // 时间段调整
    const hour = new Date(time).getHours();
    if (hour >= 9 && hour <= 17) {
      limit *= 0.8; // 工作时间降低限制
    }

    // 地理位置调整
    if (this.isHighRiskRegion(ip)) {
      limit *= 0.3;
    }

    return Math.floor(limit);
  }
}
```

## 4. 分布式实现

### 4.1 基于 Redis 的实现

```javascript
class RedisRateLimit {
  constructor(redis, algorithm = "sliding_window") {
    this.redis = redis;
    this.algorithm = algorithm;
  }

  async isAllowed(key, limit, windowSize) {
    const script = this.getScript(this.algorithm);
    const result = await this.redis.eval(
      script,
      1,
      key,
      limit,
      windowSize,
      Date.now(),
    );

    return result[0] === 1; // 1: allowed, 0: denied
  }

  getScript(algorithm) {
    switch (algorithm) {
      case "sliding_window":
        return `
          local key = KEYS[1]
          local limit = tonumber(ARGV[1])
          local window = tonumber(ARGV[2])
          local now = tonumber(ARGV[3])
          
          -- 清理过期数据
          redis.call('ZREMRANGEBYSCORE', key, 0, now - window * 1000)
          
          -- 检查当前计数
          local current = redis.call('ZCARD', key)
          if current < limit then
            -- 添加当前请求
            redis.call('ZADD', key, now, now)
            redis.call('EXPIRE', key, window)
            return {1, limit - current - 1}
          else
            return {0, 0}
          end
        `;

      case "token_bucket":
        return `
          local key = KEYS[1]
          local capacity = tonumber(ARGV[1])
          local refill_rate = tonumber(ARGV[2])
          local now = tonumber(ARGV[3])
          
          local bucket = redis.call('HMGET', key, 'tokens', 'last_refill')
          local tokens = tonumber(bucket[1]) or capacity
          local last_refill = tonumber(bucket[2]) or now
          
          -- 计算应添加的令牌
          local time_passed = (now - last_refill) / 1000
          local tokens_to_add = math.floor(time_passed * refill_rate)
          
          if tokens_to_add > 0 then
            tokens = math.min(capacity, tokens + tokens_to_add)
            last_refill = now
          end
          
          if tokens >= 1 then
            tokens = tokens - 1
            redis.call('HMSET', key, 'tokens', tokens, 'last_refill', last_refill)
            redis.call('EXPIRE', key, 3600)
            return {1, tokens}
          else
            redis.call('HMSET', key, 'tokens', tokens, 'last_refill', last_refill)
            redis.call('EXPIRE', key, 3600)
            return {0, 0}
          end
        `;
    }
  }
}
```

### 4.2 集群一致性处理

```javascript
class ClusterRateLimit {
  constructor(nodes, consistencyLevel = "eventual") {
    this.nodes = nodes;
    this.consistencyLevel = consistencyLevel;
  }

  async isAllowed(key, limit) {
    switch (this.consistencyLevel) {
      case "strong":
        return this.strongConsistency(key, limit);
      case "eventual":
        return this.eventualConsistency(key, limit);
      default:
        throw new Error("Unknown consistency level");
    }
  }

  async strongConsistency(key, limit) {
    // 需要多数节点同意
    const promises = this.nodes.map((node) =>
      node.checkAndIncrement(key, limit),
    );

    const results = await Promise.all(promises);
    const allowed = results.filter((r) => r.allowed).length;

    if (allowed > this.nodes.length / 2) {
      return { allowed: true };
    } else {
      // 回滚已执行的操作
      await this.rollback(key, results);
      return { allowed: false };
    }
  }

  async eventualConsistency(key, limit) {
    // 使用主节点决策，异步同步到其他节点
    const primary = this.nodes[0];
    const result = await primary.checkAndIncrement(key, limit);

    // 异步同步到其他节点
    this.syncToSecondaries(key, result);

    return result;
  }
}
```

## 5. HTTP 响应规范

### 5.1 标准响应头

```http
# 成功请求
HTTP/1.1 200 OK
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 999
X-RateLimit-Reset: 1640998800
X-RateLimit-Policy: "1000;w=3600"

# 超过限制
HTTP/1.1 429 Too Many Requests
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 0
X-RateLimit-Reset: 1640998800
Retry-After: 3600

{
  "error": {
    "code": "RATE_LIMIT_EXCEEDED",
    "message": "API rate limit exceeded",
    "details": {
      "limit": 1000,
      "remaining": 0,
      "reset": 1640998800
    }
  }
}
```

### 5.2 扩展响应头

```http
# 多维度限制信息
X-RateLimit-User-Limit: 100
X-RateLimit-User-Remaining: 95
X-RateLimit-User-Reset: 1640998800

X-RateLimit-IP-Limit: 1000
X-RateLimit-IP-Remaining: 900
X-RateLimit-IP-Reset: 1640998800

X-RateLimit-Endpoint-Limit: 50
X-RateLimit-Endpoint-Remaining: 45
X-RateLimit-Endpoint-Reset: 1640998800

# 策略信息
X-RateLimit-Policy: "100;w=3600;comment=user, 1000;w=3600;comment=ip"
```

## 6. 客户端处理策略

### 6.1 智能重试

```javascript
class RateLimitAwareClient {
  constructor(baseURL, options = {}) {
    this.baseURL = baseURL;
    this.maxRetries = options.maxRetries || 3;
    this.backoffMultiplier = options.backoffMultiplier || 2;
  }

  async request(endpoint, options = {}) {
    let attempt = 0;

    while (attempt <= this.maxRetries) {
      try {
        const response = await fetch(`${this.baseURL}${endpoint}`, options);

        if (response.status === 429) {
          const retryAfter = this.getRetryAfter(response);

          if (attempt < this.maxRetries) {
            await this.sleep(retryAfter * 1000);
            attempt++;
            continue;
          }

          throw new RateLimitError("Rate limit exceeded", response);
        }

        return response;
      } catch (error) {
        if (attempt === this.maxRetries) {
          throw error;
        }

        const delay = Math.pow(this.backoffMultiplier, attempt) * 1000;
        await this.sleep(delay);
        attempt++;
      }
    }
  }

  getRetryAfter(response) {
    const retryAfter = response.headers.get("Retry-After");
    const resetTime = response.headers.get("X-RateLimit-Reset");

    if (retryAfter) {
      return parseInt(retryAfter);
    }

    if (resetTime) {
      const now = Math.floor(Date.now() / 1000);
      return Math.max(0, parseInt(resetTime) - now);
    }

    return 60; // 默认等待1分钟
  }

  sleep(ms) {
    return new Promise((resolve) => setTimeout(resolve, ms));
  }
}
```

### 6.2 预测性限制

```javascript
class PredictiveRateLimit {
  constructor() {
    this.requests = [];
    this.limits = {};
  }

  updateLimits(response) {
    const headers = response.headers;
    this.limits = {
      limit: parseInt(headers.get("X-RateLimit-Limit")),
      remaining: parseInt(headers.get("X-RateLimit-Remaining")),
      reset: parseInt(headers.get("X-RateLimit-Reset")),
    };
  }

  shouldDelay() {
    if (!this.limits.limit) return false;

    const now = Date.now();
    const resetTime = this.limits.reset * 1000;
    const timeUntilReset = resetTime - now;

    if (timeUntilReset <= 0) return false;

    // 计算理想的请求间隔
    const idealInterval = timeUntilReset / this.limits.remaining;

    // 检查最近的请求是否太频繁
    const recentRequests = this.requests.filter((t) => now - t < 60000);
    if (recentRequests.length === 0) return false;

    const lastRequestTime = Math.max(...recentRequests);
    const timeSinceLastRequest = now - lastRequestTime;

    return timeSinceLastRequest < idealInterval;
  }

  async makeRequest(requestFn) {
    if (this.shouldDelay()) {
      const delay = this.calculateDelay();
      await this.sleep(delay);
    }

    this.requests.push(Date.now());
    const response = await requestFn();
    this.updateLimits(response);

    return response;
  }
}
```

## 7. 监控和告警

### 7.1 监控指标

```yaml
metrics:
  # 基础指标
  rate_limit_hits_total:
    type: counter
    labels: [endpoint, user_type, action]

  rate_limit_rejections_total:
    type: counter
    labels: [endpoint, user_type, reason]

  rate_limit_current_usage:
    type: gauge
    labels: [key, limit_type]

  # 性能指标
  rate_limit_check_duration:
    type: histogram
    buckets: [0.001, 0.005, 0.01, 0.05, 0.1, 0.5, 1.0]

  rate_limit_redis_errors:
    type: counter
    labels: [operation, error_type]
```

### 7.2 告警规则

```yaml
alerts:
  # 高拒绝率告警
  - name: HighRateLimitRejectionRate
    condition: |
      rate(rate_limit_rejections_total[5m]) / 
      rate(rate_limit_hits_total[5m]) > 0.1
    duration: 2m
    message: "Rate limit rejection rate is {{ $value }}% for endpoint {{ $labels.endpoint }}"

  # Redis错误告警
  - name: RateLimitRedisErrors
    condition: rate(rate_limit_redis_errors[1m]) > 10
    duration: 1m
    message: "High rate of Redis errors in rate limiting: {{ $value }}/min"

  # 检查延迟告警
  - name: RateLimitHighLatency
    condition: |
      histogram_quantile(0.95, rate(rate_limit_check_duration_bucket[5m])) > 0.1
    duration: 5m
    message: "Rate limit check latency is high: {{ $value }}s"
```

## 8. 最佳实践

### 8.1 设计原则

```
1. 宽进严出：初期限制较宽松，逐步收紧
2. 渐进式限制：不同等级用户不同限制
3. 透明性：向用户明确说明限制规则
4. 可预测性：稳定的限制策略，避免频繁变更
5. 降级策略：系统压力大时优雅降级
```

### 8.2 常见陷阱

```
❌ 避免的错误:
1. 过于严格的限制影响正常用户
2. 不考虑时区差异的固定窗口
3. 忽略分布式环境下的数据一致性
4. 缺少监控和可观测性
5. 没有考虑客户端重试对系统的放大效应
```

### 8.3 配置建议

```yaml
# 推荐的分级配置
rate_limits:
  # 免费用户
  free_user:
    requests_per_minute: 60
    requests_per_hour: 1000
    requests_per_day: 10000

  # 付费用户
  premium_user:
    requests_per_minute: 300
    requests_per_hour: 10000
    requests_per_day: 100000

  # 企业用户
  enterprise_user:
    requests_per_minute: 1000
    requests_per_hour: 50000
    requests_per_day: 1000000

  # 特殊端点
  special_endpoints:
    search:
      multiplier: 0.5 # 搜索API限制更严格
    upload:
      multiplier: 0.1 # 上传API限制最严格
    auth:
      multiplier: 0.2 # 认证API防止暴力破解
```

## 总结

频率限制是一个涉及算法、架构、业务的复杂系统。设计时需要考虑：

1. **算法选择**：根据业务特点选择合适的限流算法
2. **架构设计**：考虑分布式环境下的一致性和性能
3. **业务策略**：制定合理的限制规则和分级策略
4. **客户端友好**：提供清晰的限制信息和合理的重试机制
5. **可观测性**：完善的监控和告警机制

正确实现频率限制能够保护系统稳定性，提升用户体验，支持业务发展。
