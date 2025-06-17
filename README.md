# 项目介绍

## 结构

```sh
.
├── README.md
├── ai
│   └── v1 // 由 AI 根据 规则，根据 IM 场景业务，编写的具体规范
├── input
│   ├── v1 // easemob 的 IM 的 restapi 文档
│   └── v2 // easemob 的 IM 的 restapi 当前最新文档
├── output
│   └── v1 // 由 AI 根据 basic 和 advance 等文档规则 ，针对 input/v1 做出的评价
├── ref.md // 参考文档
├── res // 资源
│   ├── 每个类别应该有overview.png
│   └── 平台架构和业务分类不一致.png
├── tree.log // 目录结构记录
├── v1 // restapi 第一个版本  由AI 根据最佳实践 生成
│   ├── 01-基础概念.md
│   ├── 02-URL设计规范.md
│   ├── 03-HTTP方法和状态码.md
│   ├── 04-响应格式和错误处理.md
│   ├── 05-最佳实践和示例.md
│   └── README.md
├── v2 // restapi 第二个版本  在基础上 由作者主导编写
│   ├── README.md // 当前文档
│   ├── advance_design.md // 高级设计规范
│   ├── basic_design.md // 基础设计规范
│   ├── cache_advanced.md // 缓存策略
│   ├── data_validation.md // 数据验证
│   ├── detail_error_format.md // 详细错误格式
│   ├── error_handling.md // 错误处理
│   ├── glossary.md // 术语表
│   ├── im_design.md // IM 特定设计
│   ├── issues.md // 问题记录
│   ├── monitoring.md // 监控规范
│   ├── rate_limiting.md // 限流设计
│   ├── response_format_standard.md // 响应格式标准
│   ├── security.md // 安全规范
│   ├── url_naming_rationale.md // URL 命名规则说明
│   ├── versioning.md // 版本控制
│   └── webhooks.md // Webhook 设计
├── v3 // restapi 第三个版本  针对IM业务场景，进行了迭代
│   ├── im_naming_conventions.md // 命名规范
│   ├── im_design.md // IM 设计
│   ├── im_security_overview.md // 安全概述
│   ├── im_error_code.md // 错误码
│   ├── im_response_format.md // 响应格式
│   ├── im_rate_limiting.md // 限流规范
│   ├── im_batch_specification.md // 批量操作规范
│   ├── im_sort_specification.md // 排序规范
│   ├── im_url_specification.md // URL 规范
│   ├── im_parameter_specification.md // 参数规范
│   ├── im_search_specification.md // 搜索规范
│   ├── im_pagination_specification.md // 分页规范
│   └── security/ // 安全相关文档
│       ├── im_data_security.md // 数据安全
│       ├── im_transport_security.md // 传输安全
│       ├── im_authentication.md // 认证
│       └── im_audit_security.md // 审计安全
└── v4 // restapi 第四个版本  内容与v3相同，但文件结构调整
    ├── im_design.md // IM 设计
    └── im/ // IM 规范文档
        ├── im_naming_conventions.md // 命名规范
        ├── im_security_overview.md // 安全概述
        ├── im_error_code.md // 错误码
        ├── im_response_format.md // 响应格式
        ├── im_rate_limiting.md // 限流规范
        ├── im_batch_specification.md // 批量操作规范
        ├── im_sort_specification.md // 排序规范
        ├── im_url_specification.md // URL 规范
        ├── im_parameter_specification.md // 参数规范
        ├── im_search_specification.md // 搜索规范
        ├── im_pagination_specification.md // 分页规范
        └── security/ // 安全相关文档
            ├── im_data_security.md // 数据安全
            ├── im_transport_security.md // 传输安全
            ├── im_authentication.md // 认证
            └── im_audit_security.md // 审计安全
```
