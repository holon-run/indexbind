---
title: API
order: 10
date: 2026-03-25
summary: Node、构建、web 与 Cloudflare 入口，以及当前搜索选项面。
---

# API

`indexbind` 有四个面向运行时的入口：

- `indexbind`
- `indexbind/build`
- `indexbind/web`
- `indexbind/cloudflare`

Node 侧的目录与工件 API 还支持可选的索引级约定文件：

- `indexbind.build.js`
- `indexbind.search.js`

这些文件位于被索引根目录的 `.indexbind/` 目录旁边，且只从那个确切的根自动发现。

## `indexbind`

面向原生 SQLite 工件的 Node 入口：

```ts
import { openIndex } from 'indexbind';

const index = await openIndex('./index.sqlite');
const hits = await index.search('rust guide', {
  topK: 5,
  mode: 'hybrid',
  reranker: { kind: 'embedding-v1', candidatePoolSize: 25 },
  relativePathPrefix: 'guides/',
});
```

### `openIndex(artifactPath, options?)`

打开原生 SQLite 工件并返回一个 `Index`。

打开选项：

- `modeProfile?`：`'hybrid'` 或 `'lexical'`

当这个 `Index` 实例应保持仅词法时使用 `modeProfile: 'lexical'`。在该 profile 下，`index.search()` 默认词法模式，并拒绝 `mode: 'hybrid'` / `mode: 'vector'`。

当工件 `sourceRoot` 含有 `indexbind.search.js` 时，`openIndex()` 会应用：

- `profiles.default` 作为默认搜索 profile
- `transformQuery()` 作为轻量查询改写钩子

### `index.info()`

返回工件元数据，例如：

- `schemaVersion`
- `builtAt`
- `embeddingBackend`
- `lexicalTokenizer`
- `sourceRoot`
- `documentCount`
- `chunkCount`

### `index.search(query, options?)`

主要选项：

- `topK?`：返回的命中数量
- `mode?`：`'hybrid'`、`'vector'` 或 `'lexical'`
- `minScore?`：在最终评分后修剪低置信尾部命中
- `reranker?`：可选的最终重排阶段
- `relativePathPrefix?`：把检索限制到某个路径子树
- `metadata?`：精确匹配的元数据过滤
- `scoreAdjustment?`：用元数据驱动的乘数调整最终排序

在 Node 入口上，`metadata` 当前在 TypeScript API 中暴露为字符串到字符串的映射。

重排器选项：

- `kind?`：`embedding-v1` 或 `heuristic-v1`
- `candidatePoolSize?`：在最终 `topK` 之前传入重排器的候选数量

得分调整选项：

- `metadataNumericMultiplier?`：其数值应乘以最终得分的元数据字段名

返回的命中包括：

- `docId`
- `relativePath`
- `canonicalUrl?`
- `title?`
- `summary?`
- `metadata`
- `score`
- `bestMatch`

`bestMatch` 包含：

- `chunkId`
- `excerpt`
- `headingPath`
- `charStart`
- `charEnd`
- `score`

## `indexbind/build`

编程式构建与增量缓存 API：

```ts
import {
  buildCanonicalBundle,
  updateBuildCache,
  exportArtifactFromBuildCache,
  exportCanonicalBundleFromBuildCache,
} from 'indexbind/build';
```

主要输入形态：

- `docId?`
- `sourcePath?`
- `relativePath`
- `canonicalUrl?`
- `title?`
- `summary?`
- `content`
- `metadata?`

当宿主应用已经拥有规范化文档集、希望直接从代码构建而不是通过 CLI 扫描目录时，使用这个入口。

关于宿主先分类文档、再注入元数据的更大混合内容示例，参见[采用示例](../guides/adoption-examples.md)。

可用辅助函数：

- `buildCanonicalBundle(outputDir, documents, options?)`
- `buildFromDirectory(inputDir, outputPath, options?)`
- `buildCanonicalBundleFromDirectory(inputDir, outputDir, options?)`
- `updateBuildCache(cachePath, documents, options?, removedRelativePaths?)`
- `updateBuildCacheFromDirectory(inputDir, cachePath, options?, updateMode?)`
- `exportArtifactFromBuildCache(cachePath, outputPath)`
- `exportCanonicalBundleFromBuildCache(cachePath, outputDir)`
- `inspectArtifact(artifactPath)`
- `benchmarkArtifact(artifactPath, queriesJsonPath)`

对基于目录的辅助函数，如果输入根含有 `indexbind.build.js`，这些 API 会自动应用：

- `includeDocument(relativePath, ctx)`
- `transformDocument(document, ctx)`

这让宿主特定的文档塑形保持挂在共享的目录扫描器与增量缓存流程上。

`updateBuildCache(...)` 返回：

- `scannedDocumentCount`
- `newDocumentCount`
- `changedDocumentCount`
- `unchangedDocumentCount`
- `removedDocumentCount`
- `activeDocumentCount`
- `activeChunkCount`

典型增量流程：

```ts
import {
  updateBuildCache,
  exportArtifactFromBuildCache,
  exportCanonicalBundleFromBuildCache,
} from 'indexbind/build';

await updateBuildCache(
  './.indexbind-cache.sqlite',
  [
    {
      relativePath: 'guides/rust.md',
      title: 'Rust Guide',
      content: '# Rust Guide\n\nRust retrieval guide.',
      metadata: { lang: 'rust' },
    },
  ],
  { embeddingBackend: 'hashing' },
  ['guides/old.md'],
);

await exportArtifactFromBuildCache('./.indexbind-cache.sqlite', './index.sqlite');
await exportCanonicalBundleFromBuildCache('./.indexbind-cache.sqlite', './index.bundle');
```

## `indexbind/web`

面向规范化 bundle 的浏览器与 worker 入口：

```ts
import { openWebIndex } from 'indexbind/web';
```

该路径默认使用 wasm 支撑的 hybrid/vector 运行时。如果你用 `modeProfile: 'lexical'` 打开，它可以留在更轻的仅词法路径上，不加载向量与模型文件。

`openWebIndex(base, options?)` 返回一个 `WebIndex`。

打开选项：

- `fetch?`：自定义资源加载器
- `modeProfile?`：`'hybrid'` 或 `'lexical'`

`modeProfile: 'lexical'` 会让该 `WebIndex` 实例跳过向量/模型加载，并使词法模式成为 `search()` 的默认值。

`WebIndex.info()` 返回规范化 bundle 元数据，例如：

- `schemaVersion`
- `artifactFormat`
- `builtAt`
- `embeddingBackend`
- `documentCount`
- `chunkCount`
- `vectorDimensions`
- `chunking`
- `features`

`WebIndex.search(query, options?)` 接受与 Node 入口相同的搜索选项，只是元数据值可以使用更宽的 JSON 值形态。

## `indexbind/cloudflare`

Cloudflare Worker 入口：

```ts
import { openWebIndex } from 'indexbind/cloudflare';
```

在 Workers 内用它替代 `indexbind/web`，让 wasm 通过 Worker 兼容的静态模块路径加载。

它接受与 `indexbind/web` 相同的可选 `fetch` 覆盖，当宿主希望经由 `ASSETS.fetch(...)` 而不是公开 URL 读取 bundle 文件时很有用。

## 搜索默认值与模式

合理的起点：

```ts
const hits = await index.search(query, {
  topK: 10,
  mode: 'hybrid',
  reranker: {
    kind: 'embedding-v1',
    candidatePoolSize: 25,
  },
});
```

当宿主应用有明确产品边界时，使用元数据过滤：

```ts
const hits = await index.search(query, {
  metadata: {
    lang: 'rust',
    visibility: 'public',
  },
});
```

当应用想要宿主定义的排序先验时，使用基于元数据的得分调整：

```ts
const hits = await index.search(query, {
  scoreAdjustment: {
    metadataNumericMultiplier: 'directory_weight',
  },
});
```

当产品想裁掉弱尾部匹配、允许少于 `topK` 条命中时，使用 `minScore`：

```ts
const hits = await index.search(query, {
  topK: 10,
  minScore: 0.05,
});
```

- `mode: 'vector'` 表示仅向量检索。
- `mode: 'lexical'` 表示仅词法检索。

关于这些旋钮如何相互作用的完整解释，参见[搜索质量控制](../guides/search-quality-controls.md)。
