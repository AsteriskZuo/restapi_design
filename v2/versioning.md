# API 版本管理策略

> **重要性**: 中优先级，保证 API 演进的稳定性和向后兼容性  
> **适用范围**: 所有 API 接口的生命周期管理

## 概览

API 版本管理是确保系统持续演进而不破坏现有客户端的关键策略。本文档定义了完整的版本管理规范，包括版本策略、兼容性管理、迁移指导等。

## 1. 版本控制策略

### 1.1 版本命名规范

**语义化版本控制 (Semantic Versioning)**:

```
格式: MAJOR.MINOR.PATCH
示例: v2.1.3

MAJOR: 不兼容的API更改
MINOR: 向后兼容的功能新增
PATCH: 向后兼容的bug修复
```

**API 版本表示方法**:

```http
# URL路径方式（推荐）
GET /api/v2/users/123

# 查询参数方式
GET /api/users/123?version=2.1

# 请求头方式
GET /api/users/123
API-Version: 2.1

# Accept头方式
GET /api/users/123
Accept: application/vnd.api+json; version=2.1
```

### 1.2 版本生命周期

**版本状态定义**:

| 状态       | 描述         | 支持期  | 新功能开发 | Bug 修复      |
| ---------- | ------------ | ------- | ---------- | ------------- |
| **开发中** | 内部开发版本 | -       | ✅         | ✅            |
| **预发布** | Beta/RC 版本 | 3 个月  | ✅         | ✅            |
| **稳定版** | 生产环境版本 | 24 个月 | ❌         | ✅            |
| **维护版** | 仅关键修复   | 12 个月 | ❌         | 🔶 仅安全问题 |
| **废弃版** | 计划下线     | 6 个月  | ❌         | ❌            |
| **终止版** | 不再支持     | -       | ❌         | ❌            |

### 1.3 版本发布策略

**发布时间表**:

```yaml
版本发布计划:
  major_release:
    frequency: "每年一次"
    timing: "Q1"
    preparation_time: "3个月"

  minor_release:
    frequency: "每季度一次"
    timing: "每季度末"
    preparation_time: "1个月"

  patch_release:
    frequency: "按需发布"
    timing: "发现问题后2周内"
    preparation_time: "1周"
```

## 2. 向后兼容性管理

### 2.1 兼容性原则

**向后兼容的变更**:

- 新增 API 端点
- 新增可选字段
- 新增响应字段
- 扩展枚举值
- 放宽验证规则

**不兼容的变更**:

- 删除 API 端点
- 删除字段
- 修改字段类型
- 修改字段语义
- 收紧验证规则

### 2.2 兼容性检查清单

**API 设计兼容性检查**:

```yaml
兼容性检查项:
  request_format:
    - 是否删除了必需字段？ ❌
    - 是否修改了字段类型？ ❌
    - 是否新增了必需字段？ ❌
    - 是否新增了可选字段？ ✅

  response_format:
    - 是否删除了现有字段？ ❌
    - 是否修改了字段类型？ ❌
    - 是否新增了响应字段？ ✅
    - 是否修改了错误码？ ❌

  behavior:
    - 是否修改了业务逻辑？ 🔶 需评估
    - 是否修改了数据验证规则？ 🔶 需评估
    - 是否修改了认证授权？ 🔶 需评估
```

### 2.3 版本兼容性示例

**v1.0 用户 API**:

```json
{
  "id": 123,
  "name": "张三",
  "email": "zhang@example.com",
  "status": "active"
}
```

**v1.1 向后兼容更新**:

```json
{
  "id": 123,
  "name": "张三",
  "email": "zhang@example.com",
  "status": "active",
  "avatar": "https://cdn.example.com/avatar.jpg", // 新增字段
  "preferences": {
    // 新增嵌套对象
    "language": "zh-CN",
    "timezone": "Asia/Shanghai"
  }
}
```

**v2.0 不兼容更新**:

```json
{
  "userId": 123, // 字段重命名: id -> userId
  "fullName": "张三", // 字段重命名: name -> fullName
  "emailAddress": "zhang@example.com", // 字段重命名: email -> emailAddress
  "accountStatus": "ACTIVE", // 类型变更: "active" -> "ACTIVE"
  "profile": {
    // 结构重组
    "avatar": "https://cdn.example.com/avatar.jpg",
    "preferences": {
      "language": "zh-CN",
      "timezone": "Asia/Shanghai"
    }
  }
}
```

## 3. 版本迁移策略

### 3.1 渐进式迁移

**并行版本支持**:

```http
# 同时支持多个版本
GET /api/v1/users/123  # 老版本
GET /api/v2/users/123  # 新版本

# 版本映射中间层
internal_service.getUser(123) -> {
  if (apiVersion === 'v1') {
    return transformToV1Format(userData);
  } else if (apiVersion === 'v2') {
    return transformToV2Format(userData);
  }
}
```

### 3.2 数据转换层

**版本适配器模式**:

```javascript
class APIVersionAdapter {
  constructor(version) {
    this.version = version;
  }

  transformUserResponse(userData) {
    switch (this.version) {
      case "v1":
        return this.toV1Format(userData);
      case "v2":
        return this.toV2Format(userData);
      default:
        throw new Error(`Unsupported version: ${this.version}`);
    }
  }

  toV1Format(data) {
    return {
      id: data.userId,
      name: data.fullName,
      email: data.emailAddress,
      status: data.accountStatus.toLowerCase(),
    };
  }

  toV2Format(data) {
    return {
      userId: data.userId,
      fullName: data.fullName,
      emailAddress: data.emailAddress,
      accountStatus: data.accountStatus,
      profile: data.profile,
    };
  }
}
```

### 3.3 客户端迁移指导

**迁移步骤**:

```markdown
## 从 v1 迁移到 v2

### 第一阶段：兼容性验证

1. 在测试环境使用 v2 API
2. 验证现有功能正常工作
3. 测试新增功能

### 第二阶段：逐步迁移

1. 更新 API 调用 URL: `/api/v1/` -> `/api/v2/`
2. 更新字段映射:
   - `id` -> `userId`
   - `name` -> `fullName`
   - `email` -> `emailAddress`
3. 处理状态码变更: `"active"` -> `"ACTIVE"`

### 第三阶段：完整切换

1. 所有调用切换到 v2
2. 移除 v1 相关代码
3. 充分测试
```

## 4. 版本废弃流程

### 4.1 废弃时间表

**标准废弃流程**:

```
T+0: 发布新版本 (v2.0)
T+6个月: 标记旧版本为废弃 (v1.x)
T+18个月: 停止新功能开发
T+24个月: 仅提供安全更新
T+30个月: 完全停止支持
```

### 4.2 废弃通知机制

**响应头通知**:

```http
HTTP/1.1 200 OK
Warning: 299 - "API version v1 is deprecated. Please migrate to v2. See: https://api.example.com/migration-guide"
Sunset: Wed, 01 Jan 2025 00:00:00 GMT
Deprecation: Wed, 01 Jul 2024 00:00:00 GMT
```

**废弃 API 响应示例**:

```json
{
  "data": {
    "id": 123,
    "name": "张三"
  },
  "meta": {
    "deprecation": {
      "deprecated": true,
      "deprecatedSince": "2024-07-01",
      "sunsetDate": "2025-01-01",
      "migrationGuide": "https://api.example.com/migration-guide",
      "newVersion": "v2"
    }
  }
}
```

### 4.3 通知渠道

**多渠道通知策略**:

```yaml
通知渠道:
  api_response:
    - HTTP Warning头部
    - 响应体中的废弃信息

  documentation:
    - API文档标记
    - 迁移指南
    - 变更日志

  communication:
    - 邮件通知
    - 开发者博客
    - 技术公告

  monitoring:
    - 使用情况统计
    - 迁移进度监控
```

## 5. 版本控制最佳实践

### 5.1 版本设计原则

**设计指导原则**:

1. **最小惊讶原则**: 保持 API 行为的一致性
2. **渐进增强**: 优先考虑向后兼容的更新
3. **明确契约**: 清晰定义 API 契约和变更影响
4. **用户导向**: 考虑客户端迁移成本

### 5.2 版本间数据一致性

**数据模型版本管理**:

```javascript
// 数据库模型支持多版本
class UserModel {
  // 内部统一数据格式
  static async findById(id) {
    const user = await db.users.findById(id);
    return {
      userId: user.id,
      fullName: user.name,
      emailAddress: user.email,
      accountStatus: user.status,
      profile: user.profile || {},
    };
  }

  // 版本特定转换
  static toAPIVersion(userData, version) {
    return new APIVersionAdapter(version).transformUserResponse(userData);
  }
}
```

### 5.3 版本测试策略

**多版本测试**:

```javascript
describe("API Version Compatibility", () => {
  test("v1 and v2 should return consistent core data", async () => {
    const v1Response = await api.get("/api/v1/users/123");
    const v2Response = await api.get("/api/v2/users/123");

    // 验证核心数据一致性
    expect(v1Response.data.id).toBe(v2Response.data.userId);
    expect(v1Response.data.name).toBe(v2Response.data.fullName);
    expect(v1Response.data.email).toBe(v2Response.data.emailAddress);
  });

  test("deprecated version should include deprecation warnings", async () => {
    const response = await api.get("/api/v1/users/123");
    expect(response.headers.warning).toContain("deprecated");
    expect(response.headers.deprecation).toBeDefined();
  });
});
```

## 6. 内容协商

### 6.1 版本选择策略

**版本解析优先级**:

```javascript
function resolveAPIVersion(req) {
  // 1. URL路径中的版本 (最高优先级)
  const pathVersion = req.path.match(/\/api\/v(\d+)/)?.[1];
  if (pathVersion) return `v${pathVersion}`;

  // 2. Accept头中的版本
  const acceptHeader = req.headers.accept;
  const acceptVersion = acceptHeader?.match(/version=(\d+\.\d+)/)?.[1];
  if (acceptVersion) return `v${acceptVersion}`;

  // 3. 自定义API-Version头
  const apiVersionHeader = req.headers["api-version"];
  if (apiVersionHeader) return apiVersionHeader;

  // 4. 查询参数中的版本
  const queryVersion = req.query.version;
  if (queryVersion) return `v${queryVersion}`;

  // 5. 默认版本
  return DEFAULT_API_VERSION;
}
```

### 6.2 版本内容协商

**媒体类型版本控制**:

```http
# 客户端请求
GET /api/users/123
Accept: application/vnd.api.v2+json

# 服务器响应
HTTP/1.1 200 OK
Content-Type: application/vnd.api.v2+json
Vary: Accept

{
  "userId": 123,
  "fullName": "张三"
}
```

## 7. 监控和分析

### 7.1 版本使用统计

**版本使用监控**:

```javascript
const versionMetrics = {
  "v1.0": {
    requests: 50000,
    percentage: 25.0,
    uniqueClients: 150,
    avgResponseTime: 200,
  },
  "v2.0": {
    requests: 120000,
    percentage: 60.0,
    uniqueClients: 400,
    avgResponseTime: 180,
  },
  "v2.1": {
    requests: 30000,
    percentage: 15.0,
    uniqueClients: 100,
    avgResponseTime: 160,
  },
};
```

### 7.2 迁移进度跟踪

**迁移指标监控**:

```json
{
  "migrationProgress": {
    "fromVersion": "v1",
    "toVersion": "v2",
    "timeline": {
      "startDate": "2024-01-01",
      "targetDate": "2024-12-31",
      "currentDate": "2024-06-01"
    },
    "metrics": {
      "totalClients": 500,
      "migratedClients": 300,
      "migrationRate": 0.6,
      "activeV1Clients": 200,
      "newV2Clients": 50
    }
  }
}
```

## 8. 文档管理

### 8.1 版本文档结构

**文档组织方式**:

```
api-docs/
├── v1/
│   ├── overview.md
│   ├── authentication.md
│   ├── endpoints/
│   └── changelog.md
├── v2/
│   ├── overview.md
│   ├── authentication.md
│   ├── endpoints/
│   ├── migration-from-v1.md
│   └── changelog.md
└── common/
    ├── errors.md
    └── rate-limiting.md
```

### 8.2 变更日志

**标准化变更记录**:

```markdown
# API v2.1.0 变更日志

## 发布日期: 2024-06-01

### 新增功能

- 添加用户偏好设置 API
- 支持批量用户操作

### 改进

- 优化用户查询性能
- 增强错误信息详细度

### Bug 修复

- 修复分页参数验证问题
- 解决并发更新竞态条件

### 废弃警告

- `GET /api/v2/users/{id}/settings` 将在 v2.2 中废弃
- 请使用 `GET /api/v2/users/{id}/preferences` 替代

### 破坏性变更

- 无
```

## 9. 应急处理

### 9.1 版本回滚策略

**回滚决策流程**:

```yaml
回滚触发条件:
  - 新版本错误率 > 5%
  - 响应时间增加 > 50%
  - 关键功能不可用
  - 大量客户端报错

回滚步骤: 1. 立即停止新版本流量
  2. 将流量路由到稳定版本
  3. 分析问题根因
  4. 制定修复计划
  5. 重新发布修复版本
```

### 9.2 热修复机制

**紧急修复流程**:

```javascript
// 运行时版本切换
class APIGateway {
  constructor() {
    this.versionRoutes = new Map();
    this.enabledVersions = ["v1", "v2"];
  }

  // 动态禁用有问题的版本
  disableVersion(version, reason) {
    this.enabledVersions = this.enabledVersions.filter((v) => v !== version);
    logger.warn(`Disabled API version ${version}: ${reason}`);
  }

  routeRequest(req, res) {
    const requestedVersion = this.resolveVersion(req);

    if (!this.enabledVersions.includes(requestedVersion)) {
      return res.status(503).json({
        error: "API version temporarily unavailable",
        availableVersions: this.enabledVersions,
      });
    }

    return this.versionRoutes.get(requestedVersion)(req, res);
  }
}
```

## 总结

有效的 API 版本管理需要：

1. **清晰的版本策略**: 明确的版本命名和生命周期
2. **向后兼容性**: 最大化保护现有客户端投资
3. **渐进式迁移**: 提供平滑的升级路径
4. **充分的通知**: 及时告知客户端版本变更
5. **监控和分析**: 跟踪版本使用情况和迁移进度
6. **应急机制**: 快速响应版本相关问题

通过系统化的版本管理，可以在保证 API 持续演进的同时，最大程度减少对现有用户的影响。
