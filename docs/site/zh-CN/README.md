---
title: indexbind
type: page
order: 0
date: 2026-03-25
summary: 面向 Node、浏览器与 Workers 的内嵌式检索工件。
---

# indexbind

`indexbind` 是一个面向固定文档集合的检索库。

它在离线阶段构建检索工件，然后在 Node、浏览器、Web Worker 或 Cloudflare Workers 中本地打开。

如果想走最短路径，从[快速开始](./guides/getting-started.md)入手。如果需要先判断 `indexbind` 是否是合适的工具，先读[如何选择 indexbind](./guides/choosing-indexbind.md)。

## 它为优化什么而生

多数搜索基础设施围绕服务、爬虫或运行时管理的索引来设计。

`indexbind` 选择了另一种立场：

- 文档集合在构建期固定
- 检索工件是确定性且可移植的
- 运行时 API 足够小，可以内嵌进另一个产品
- 同一套检索模型可以运行在 Node、浏览器与 Workers 中
- 宿主应用仍然拥有路由、过滤与排序策略

因此它更适合文档系统、本地工具、由宿主定义工作流的本地知识库、静态部署，以及 [`mdorigin`](https://mdorigin.holon.run) 这类把内嵌检索作为更大发布流程一部分的产品。

## 选择合适的工具

当你需要一层内嵌检索时，`indexbind` 是更合适的选择。它并不想成为：

- 托管搜索服务
- 开箱即用的知识库产品
- 仅面向静态站点的搜索挂件

如果仍难决定，参见[如何选择 indexbind](./guides/choosing-indexbind.md)。

## 主要能力

- 从文档集合构建确定性的检索工件
- 为 Node 提供原生 SQLite 工件
- 为 web 与 worker 运行时提供规范化文件 bundle
- 提供 Node 构建 API，以及 Node、web 与 Cloudflare 的查询 API
- 支持增量构建缓存，并可随时导出全新工件与 bundle
- 让搜索保持为可内嵌的库级关注点，而不是托管服务

## 按需入门

- 想要最短的端到端路径：
  [快速开始](./guides/getting-started.md)
- 想判断它与 Pagefind、qmd 或 Meilisearch 相比是否更合适：
  [如何选择 indexbind](./guides/choosing-indexbind.md)
- 想看文档、发布或本地知识库工作流的具体集成形态：
  [采用示例](./guides/adoption-examples.md)
- 想了解本地基准数据与当前自用模式：
  [基准测试与案例](./guides/benchmarks-and-case-studies.md)
- 想从代码集成：
  [API](./reference/api.md)
- 想用 CLI 驱动构建：
  [CLI](./reference/cli.md)
- 想在浏览器或 Worker 中使用：
  [Web 与 Cloudflare](./guides/web-and-cloudflare.md)
- 想理解打包与工件形态：
  [打包与分发](./reference/packaging.md)

## 当前平台支持

- 原生预构建产物覆盖 macOS arm64、macOS x64 与 Linux x64（glibc）。
- 未发布 Windows 原生预构建；在 Windows 上请使用 WSL 完成安装、构建与本地 Node 查询。
- 规范化 bundle 运行时可跨浏览器、Workers 与 Cloudflare Workers 使用。

## 文档地图

- [快速开始](./guides/getting-started.md)
- [如何选择 indexbind](./guides/choosing-indexbind.md)
- [采用示例](./guides/adoption-examples.md)
- [基准测试与案例](./guides/benchmarks-and-case-studies.md)
- [搜索质量控制](./guides/search-quality-controls.md)
- [Web 与 Cloudflare](./guides/web-and-cloudflare.md)
- [API](./reference/api.md)
- [CLI](./reference/cli.md)
- [打包与分发](./reference/packaging.md)
- [规范化 Bundle](./concepts/canonical-bundles.md)
- [运行时模型](./concepts/runtime-model.md)
- [规范化工件与 WASM](./concepts/canonical-artifact-and-wasm.md)

## 本地预览

如果你想用 [`mdorigin`](https://mdorigin.holon.run) 预览本站点本身：

```bash
npm run docs:index
npm run docs:dev
```

<!-- INDEX:START -->

- [指南](./guides/)
  <!-- mdorigin:index kind=directory -->

- [概念](./concepts/)
  <!-- mdorigin:index kind=directory -->

- [参考](./reference/)
  <!-- mdorigin:index kind=directory -->

<!-- INDEX:END -->
