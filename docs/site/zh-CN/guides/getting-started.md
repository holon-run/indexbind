---
title: 快速开始
order: 10
date: 2026-03-25
summary: 安装 indexbind，构建原生工件与规范化 bundle，并端到端跑通第一个查询。
---

# 快速开始

本指南走一条最短的完整路径：

1. 安装 `indexbind`
2. 从一个极小的文档目录构建原生 SQLite 工件
3. 从 Node 查询它
4. 从同一批文档构建规范化 bundle
5. 用 web 运行时查询该 bundle
6. 了解面向反复本地重建的增量缓存路径

## 安装

```bash
npm install indexbind
```

支持的原生预构建目标：

- macOS arm64
- macOS x64
- Linux x64（glibc）

在 Windows 上，请使用 WSL 完成安装、构建与本地 Node 查询。未发布 Windows 原生预构建。

如果你的平台没有预构建原生插件，请在 Rust 工具链环境中本地构建原生包：

```bash
npm run build:native:release
```

## 创建一个极小的文档集

创建一个最小目录：

```text
docs/
  rust.md
  workers.md
```

示例内容：

```md
# Rust Guide

Rust retrieval guide for local search.
```

```md
# Cloudflare Workers Guide

Workers deployment notes for retrieval.
```

## 构建原生 SQLite 工件

针对本地文档目录：

```bash
npx indexbind build ./docs
```

默认把工件写入 `./docs/.indexbind/index.sqlite`。

## 从 Node 查询

```ts
import { openIndex } from 'indexbind';

const index = await openIndex('./docs/.indexbind/index.sqlite');
const hits = await index.search('rust guide', {
  topK: 5,
  mode: 'hybrid',
  reranker: {
    kind: 'embedding-v1',
    candidatePoolSize: 25,
  },
});

console.log(hits[0]);
```

你会看到大致如下形状的命中：

```ts
{
  relativePath: 'rust.md',
  title: 'Rust Guide',
  score: 0.9,
  bestMatch: {
    excerpt: 'Rust retrieval guide for local search.',
    ...
  },
  ...
}
```

当你的运行时是 Node 且想要最简单的本地设置时，使用原生 SQLite 工件。

也可以用 CLI 对工件做健全性检查：

```bash
npx indexbind search ./docs/.indexbind/index.sqlite "rust guide"
npx indexbind search ./docs/.indexbind/index.sqlite "rust guide" --text
```

CLI 命令默认输出 JSON，便于脚本与智能体消费。加 `--text` 可得到更短的终端摘要。

## 构建规范化 Bundle

规范化 bundle 是面向浏览器与 worker 的可移植工件：

```bash
npx indexbind build-bundle ./docs
```

默认把 bundle 写入 `./docs/.indexbind/index.bundle/`。

也可以用编程方式构建同一个 bundle：

```ts
import { buildCanonicalBundle } from 'indexbind/build';

await buildCanonicalBundle('./index.bundle', [
  {
    relativePath: 'guides/rust.md',
    canonicalUrl: '/guides/rust',
    title: 'Rust Guide',
    summary: 'A minimal retrieval guide.',
    content: '# Rust Guide\n\nRust retrieval guide.',
    metadata: { lang: 'rust' },
  },
], {
  embeddingBackend: 'model2vec',
});
```

当你想要 `indexbind` 的最佳检索质量时，`model2vec` 是默认推荐后端。`hashing` 仍可作为更轻量、偏兼容取向的后端使用。

## 可选：增量构建缓存

如果你会反复重建同一个本地语料，保留一个可变缓存，并从它导出全新工件：

```bash
npx indexbind update-cache ./docs --git-diff
npx indexbind export-artifact ./index.sqlite --cache-file ./docs/.indexbind/build-cache.sqlite
npx indexbind export-bundle ./index.bundle --cache-file ./docs/.indexbind/build-cache.sqlite
```

适用于：

- 语料基本稳定
- 你在反复迭代本地内容
- 宿主应用或脚本希望增量触发重建

## 可选：索引级约定文件

如果某个被索引的目录需要少量宿主特定的塑形，把约定文件放在它的 `.indexbind/` 输出旁边：

```text
docs/
  indexbind.build.js
  indexbind.search.js
  .indexbind/
```

`indexbind.build.js` 可以扩展默认目录扫描器而不替换它：

```js
export function includeDocument(relativePath) {
  return relativePath !== 'draft.md';
}

export function transformDocument(document) {
  return {
    ...document,
    canonicalUrl: `https://example.com/${document.relativePath.replace(/\.md$/i, '')}`,
    metadata: {
      ...(document.metadata ?? {}),
      is_default_searchable: 'true',
      directory_weight: 1.0,
    },
  };
}
```

`indexbind.search.js` 可以定义默认搜索 profile 与轻量查询改写：

```js
export const profiles = {
  default: {
    metadata: {
      is_default_searchable: 'true',
    },
    scoreAdjustment: {
      metadataNumericMultiplier: 'directory_weight',
    },
  },
};

export function transformQuery(query) {
  return {
    query: query.replace(/btc/ig, 'bitcoin'),
  };
}
```

这些文件是索引级的：

- 如果你索引 `./docs`，就把它们放在 `./docs/`
- 它们只影响那个被索引的根目录
- 不存在仓库根目录回退行为

## 在 Web 运行时查询 Bundle

```ts
import { openWebIndex } from 'indexbind/web';

const index = await openWebIndex('./docs/.indexbind/index.bundle');
const hits = await index.search('rust guide');
```

`indexbind/web` 要求 wasm 初始化成功，不会静默回退到另一套 JS 查询引擎。

当你希望同一份检索数据工作在浏览器、标准 worker 或 Cloudflare Workers 中时，使用规范化 bundle。

## 选择工件形态

- 本地 Node 检索用原生 SQLite 工件。
- 浏览器与 worker 运行时用规范化 bundle。
- 需要反复本地重建而不把运行时工件当可变存储时，用增量构建缓存。
- 如果你的产品横跨两种环境，从同一批文档同时构建两者。

## 检查与基准测试

检查原生 SQLite 工件：

```bash
npx indexbind inspect ./docs/.indexbind/index.sqlite
npx indexbind inspect ./docs/.indexbind/index.sqlite --text
```

运行内置回归 fixture：

```bash
npm run benchmark:basic
```
