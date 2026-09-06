---
title: 打包与分发
order: 30
date: 2026-03-25
summary: npm 包边界、原生包、wasm 资产与规范化 bundle 模型文件。
---

# 打包与分发

发布的 npm 版本拆分为：

- `indexbind`，包含 TypeScript API、wasm 运行时文件与原生加载器
- 平台包，例如 `@indexbind/native-darwin-x64`，包含预构建的 NAPI 二进制

正常使用时只需安装 `indexbind`。存在受支持的预构建目标时，npm 会自动解析匹配的原生包。

## 当前原生支持

已发布的原生预构建目前覆盖：

- macOS arm64
- macOS x64
- Linux x64（glibc）

未发布 Windows 原生预构建。在 Windows 上，请使用 WSL 完成：

- `npm install indexbind`
- 本地构建命令
- 通过原生插件打开 SQLite 工件的本地 Node 查询

如果你的环境没有可用的预构建包，请在 Rust 工具链环境中安装并构建，而不是假设 npm 能解析到匹配的原生二进制。

## npm 包里有什么

根包包含：

- 运行时入口，例如 `indexbind`、`indexbind/build`、`indexbind/web` 与 `indexbind/cloudflare`
- `dist/wasm` 与 `dist/wasm-bundler` 中的 wasm 运行时文件
- 在预构建平台包存在时解析它们的原生加载器

即使你的宿主开发机是 Windows，浏览器与 worker 入口仍来自根包。当前指引只是把安装与构建放在 WSL 里完成。

## 规范化 Bundle 里有什么

规范化 bundle 包含你的检索数据：

- manifest
- 文档
- 分块
- 向量
- postings
- 可选的模型资产

当你用 `model2vec` 构建时，以下文件会被复制进 bundle：

- `model/tokenizer.json`
- `model/config.json`
- `model/model.safetensors`

这些模型文件不随根 npm 包分发。
