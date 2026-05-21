# 工程规范（示例）

## 编码约定

- 模块分层：`src/http/` 入口层 / `src/service/` 业务逻辑层 / `src/repository/` 数据层
- 命名：接口响应字段统一 `snake_case`；内部 TypeScript 类型使用 `camelCase`
- 错误：使用统一的 `AppError` 类，不直接抛原始 `Error`
- 日志：使用 pino（或其他结构化 logger），禁止 `console.log` 入生产代码

## API 约定

- 成功响应：`{ code: 0, data: ... }`
- 错误响应：`{ code: <非 0 整数>, message: "..." }`
- HTTP 状态码与 code 字段解耦：HTTP 层用标准状态码，业务错误通过 code 字段表达

## 本地自验命令

- 静态检查：`npm run type-check`（TypeScript 类型）+ `npm run lint`（ESLint/Biome）
- 单元测试：`npm test`（对应文件：`npm test -- {file}`）
- 集成测试：`npm run test:integration`
- 构建：`npm run build`

## 提交前门禁

- 前置检查（format + type-check）必须通过
- 涉及 `src/` 运行时代码的改动必须走 code-reviewer
- 涉及 API 或 schema 变更的改动必须走 doc-refresher

## 迁移与破坏性变更

- 数据库 migration 通过 SQL 文件管理，一个变更一个文件
- 接口破坏性变更必须在 PR 描述里显式标记 `BREAKING CHANGE:`
- 字段删除走"先标记废弃 → 下次发版删除"两步走
