---
title: 采用示例
order: 15
date: 2026-03-29
summary: 把 indexbind 映射到文档、发布与本地知识库工作流，同时不放弃宿主控制权。
---

# 采用示例

这些示例展示 `indexbind` 在更大系统中的位置。

它们的模式是一致的：

- 宿主应用决定文档集合是什么
- `indexbind` 构建或更新检索工件
- 宿主应用仍拥有路由、过滤、排序策略与渲染

## 带浏览器搜索的文档站点

当你希望文档站点把搜索工件随站点一起交付时，使用这种形态。

典型流程：

1. 从文档语料构建规范化 bundle
2. 把 bundle 与站点一起发布
3. 在浏览器或 worker 运行时中加载

```bash
npx indexbind build-bundle ./docs ./public/index.bundle
```

```ts
import { openWebIndex } from 'indexbind/web';

const index = await openWebIndex('/index.bundle');
const hits = await index.search('canonical bundle');
```

宿主应用仍然拥有：

- 路由生成
- URL 结构
- 摘要渲染
- UI 状态与过滤器
- 叠加在上层的任何宿主特定排序规则

这与 `indexbind` 文档站点今天的组织方式很接近。

## 发布或博客系统

当你已经有一套规范化内容流水线、希望检索留在流水线内部时，使用这种形态。

典型流程：

1. 在宿主应用中解析 frontmatter 与 markdown
2. 把规范化后的文档传入 `indexbind/build`
3. 导出与最终部署目标匹配的运行时工件

```ts
import {
  buildCanonicalBundle,
} from 'indexbind/build';

await buildCanonicalBundle('./dist/search.bundle', [
  {
    relativePath: 'posts/retrieval.md',
    canonicalUrl: '/posts/retrieval',
    title: 'Retrieval Notes',
    summary: 'How the publishing pipeline builds search artifacts.',
    content: '# Retrieval Notes\n\nBuild artifacts during publish.',
    metadata: {
      section: 'blog',
      visibility: 'public',
    },
  },
], {
  embeddingBackend: 'hashing',
});
```

当宿主已经拥有以下能力时，这种形态很合适：

- frontmatter 解析
- 规范 URL
- 分类体系
- 发布状态
- 产品级排序先验

当博客或发布系统希望搜索只是构建产物之一、而不是一个独立搜索服务时，这也是个好选择。

## 面向基本默认仓库的索引级约定

当默认目录扫描器已经够用、只是某个被索引的根目录还需要少量仓库特定行为时，使用这种形态。

典型流程：

1. 继续使用 `indexbind build`、`build-bundle` 或 `update-cache`
2. 把 `indexbind.build.js` 放在被索引根目录旁边
3. 当 CLI 或 Node 搜索需要默认 profile 时，把 `indexbind.search.js` 也放在那里

```text
docs/
  indexbind.build.js
  indexbind.search.js
  .indexbind/
```

`indexbind.build.js` 示例：

```js
export function includeDocument(relativePath) {
  return relativePath !== 'archive/notes.md';
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

`indexbind.search.js` 示例：

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

当宿主只需要以下能力时，这种形态很合适：

- 从默认扫描中跳过少数文件
- 推导 `canonicalUrl`
- 注入宿主元数据，例如 `source_root`、`content_kind`、`is_default_searchable` 或 `directory_weight`
- 定义默认的 CLI/Node 搜索 profile
- 增加轻量的查询别名扩展

当仓库不需要完整的自定义构建脚本、但零配置默认又不够时，这是恰当的中间地带。

## 混合本地知识库的自定义索引构建器

当宿主应用希望精确决定扫描哪些目录、如何分类文档、以及把哪些元数据或权重规则写进索引时，使用这种形态。

典型流程：

1. 遍历宿主特定的内容根
2. 把每个 markdown 文件规范化为 `BuildDocument`
3. 推断元数据，例如来源根、内容类型、可见性或目录权重
4. 把规范化后的文档传入 `indexbind/build`

```ts
import { buildCanonicalBundle } from 'indexbind/build';

const documents = [
  {
    docId: 'public/post-a/README.md',
    sourcePath: '/workspace/public/post-a/README.md',
    relativePath: 'public/post-a/README.md',
    canonicalUrl: 'https://example.com/post-a/',
    title: 'Post A',
    summary: 'Host-defined summary for workspace search.',
    content: '# Post A\n\nHost-controlled markdown content for the public post.',
    metadata: {
      source_root: 'public',
      content_kind: 'public_post',
      is_default_searchable: true,
      directory_weight: 1.0,
    },
  },
  {
    docId: 'research/notes/layer2.md',
    sourcePath: '/workspace/research/notes/layer2.md',
    relativePath: 'research/notes/layer2.md',
    title: 'Layer2 Notes',
    content: '# Layer2 Notes\n\nHost-controlled markdown content for research search.',
    metadata: {
      source_root: 'research',
      content_kind: 'research',
      is_default_searchable: true,
      directory_weight: 0.92,
    },
  },
];

await buildCanonicalBundle('./dist/workspace.bundle', documents, {
  embeddingBackend: 'model2vec',
  sourceRootId: 'workspace',
  sourceRootPath: process.cwd(),
});
```

当宿主希望拥有以下控制权时，这种形态很合适：

- 多根目录选择
- frontmatter 解析与自定义标题、摘要规则
- 内容分类，例如 `public_post`、`draft`、`research` 或 `archive_doc`
- 元数据驱动的排序提示，例如目录权重或可见性标志
- 分离的搜索 profile，例如默认搜索与穷尽搜索

这与 `workspace` 项目使用 `indexbind` 的方式很接近：宿主先规范化异构内容，再把受控的文档集交给 `indexbind/build`。

## 本地知识库或智能体工作区

当文档反复变化、宿主希望增量触发索引时，使用这种形态。

典型流程：

1. 刷新构建缓存
2. 从缓存导出全新运行时工件
3. 让宿主工具在本地打开新工件

```bash
npx indexbind update-cache ./workspace-docs --git-diff
npx indexbind export-artifact ./workspace.sqlite --cache-file ./workspace-docs/.indexbind/build-cache.sqlite
```

```ts
import { openIndex } from 'indexbind';

const index = await openIndex('./workspace.sqlite');
const hits = await index.search('incremental indexing');
```

适合：

- 宿主定义工作流的本地知识库
- 智能体驱动的文档刷新
- git 钩子或任务运行器触发的重建
- 希望内嵌检索层、而不是可变本地存储搜索产品的产品

## 五种形态之间如何选

- 运行时目标以浏览器或 worker 为主时，优先文档站点模式。
- 宿主已有结构化内容流水线时，优先发布流水线模式。
- 默认扫描器已接近需求、只需给一个被索引根挂少量宿主策略时，优先索引级约定模式。
- 宿主需要在索引前对混合本地内容分类时，优先自定义构建器模式。
- 增量重建与本地 Node 查询比浏览器分发更重要时，优先本地知识库模式。

如果需要完整的决策框架，回到[如何选择 indexbind](./choosing-indexbind.md)。想看本地参考数据与当前自用说明，参见[基准测试与案例](./benchmarks-and-case-studies.md)。
