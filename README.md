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
└── v2 // restapi 第二个版本  在基础上 由作者主导编写
    ├── README.md // 当前文档
    ├── advance_design.md // 高级设计规范
    ├── basic_design.md // 基础设计规范
    ├── cache_advanced.md // 缓存策略
    ├── data_validation.md // 数据验证
    ├── detail_error_format.md // 详细错误格式
    ├── error_handling.md // 错误处理
    ├── glossary.md // 术语表
    ├── im_design.md // IM 特定设计
    ├── issues.md // 问题记录
    ├── monitoring.md // 监控规范
    ├── rate_limiting.md // 限流设计
    ├── response_format_standard.md // 响应格式标准
    ├── security.md // 安全规范
    ├── url_naming_rationale.md // URL 命名规则说明
    ├── versioning.md // 版本控制
    └── webhooks.md // Webhook 设计
```
