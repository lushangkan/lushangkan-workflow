# OpenSpec 边界规则

OpenSpec 用于垂直切片，不用于描述整个项目。

## 项目级文档

项目级描述放在 OpenSpec 外部，例如：

- `README.md`
- `PROJECT.md`
- `docs/`

项目级文档描述：项目为什么存在、服务谁、第一版范围、非目标、领域语言和长期约束。

## OpenSpec change

OpenSpec change 描述一次可独立推进的垂直切片。

每个 change 应该能独立完成：

- 设计
- 实现
- 验证
- 归档

## 禁止混用

不要把这些内容塞进 OpenSpec change：

- 整个项目愿景。
- 完整 PRD。
- 平台级路线图。
- 与本次切片无关的架构总览。
- 跨多个独立功能的实施计划。

## 引用方式

OpenSpec change 可以引用项目级文档，但不要复制长文。

正确方式：

```txt
本切片来自 PROJECT.md 的“第一版范围 / 上传文档后获取检测结果”。
```

错误方式：

```txt
把 PROJECT.md 的完整项目愿景复制进 proposal.md。
```
