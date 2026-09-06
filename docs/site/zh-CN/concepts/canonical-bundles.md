---
title: 规范化 Bundle
order: 10
date: 2026-03-25
summary: web 与 worker 运行时共享的可移植文件 bundle 工件。
---

# 规范化 Bundle

规范化 bundle（canonical bundle）是跨运行时查询的产品级工件。

它是一个目录型 bundle，而不是 SQLite 数据库。当前的最小形态为：

- `manifest.json`
- `documents.json`
- `chunks.json`
- `vectors.bin`
- `postings.json`
- 可选的 `model/`

## 它为什么存在

SQLite 很适合原生本地查询，但把它当作长期跨运行时契约是错误的格式。

规范化 bundle 更容易：

- 从静态托管直接分发
- 在开发期直接检查
- 在浏览器与 worker 中加载
- 演进时不会把 SQLite 特有的 schema 决策泄漏进公共 API

## Bundle 里存了什么

- 规范化的文档元数据
- 分块边界与摘录
- 供语义检索使用的稠密向量
- 供词法评分使用的 postings
- 可选的 `model2vec` 资产，用于 wasm 中的查询嵌入

bundle 是只检索的。它不打算暴露任何宿主特定的文件读取契约。
