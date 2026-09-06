---
title: 搜索质量控制
order: 30
date: 2026-03-26
summary: 理解哪些搜索选项影响召回、重排、过滤与最终排序。
---

# 搜索质量控制

`indexbind` 暴露一小组搜索旋钮。关键区别在于每个旋钮作用在哪一层：

- 召回：允许哪些候选进入候选池
- 重排：候选池如何被重新排序
- 最终排序：已排序的命中在返回前如何被调整

## 召回控制

### `mode`

```ts
const hits = await index.search('rust guide', {
  mode: 'hybrid',
});
```

`mode: 'hybrid'` 在构建最终排序列表前，合并向量与词法检索。

当你想在精确匹配与语义匹配之间获得更稳的默认行为时使用它。

```ts
const hits = await index.search('rust guide', {
  mode: 'vector',
});
```

`mode: 'vector'` 表示仅向量检索。

```ts
const hits = await index.search('rust guide', {
  mode: 'lexical',
});
```

`mode: 'lexical'` 表示仅词法检索。

### `relativePathPrefix`

```ts
const hits = await index.search('rust guide', {
  relativePathPrefix: 'guides/',
});
```

它在排序定稿前，把候选文档限制在某个路径前缀内。

当你的应用已经知道应搜索哪个子树时使用它。

### `metadata`

```ts
const hits = await index.search('rust guide', {
  metadata: {
    lang: 'rust',
    visibility: 'public',
  },
});
```

元数据过滤是精确匹配过滤。它在最终结果列表返回前收窄候选集。

当宿主应用需要产品级过滤（例如以下维度）时使用：

- 语言
- 租户
- 内容类型
- 发布状态

## 重排控制

### `reranker.kind`

```ts
const hits = await index.search('rust guide', {
  reranker: { kind: 'heuristic-v1' },
});
```

可用类型：

- `heuristic-v1`
- `embedding-v1`

`heuristic-v1` 是轻量重排器，偏向强标题与标题层级匹配。

`embedding-v1` 使用嵌入层重排候选池，带有更强的语义判断。

### `reranker.candidatePoolSize`

```ts
const hits = await index.search('rust guide', {
  topK: 5,
  reranker: {
    kind: 'embedding-v1',
    candidatePoolSize: 25,
  },
});
```

它控制在最终 `topK` 截断之前，有多少候选进入重排器。

以下情况增大它：

- 目标文档相关却总进不了最终前排
- 你的集合噪声较大
- 你使用语义重排器，需要更多提升空间

## 最终排序控制

### `scoreAdjustment.metadataNumericMultiplier`

```ts
const hits = await index.search('rust guide', {
  scoreAdjustment: {
    metadataNumericMultiplier: 'directory_weight',
  },
});
```

它把最终得分乘以每个命中上的数值型元数据字段。

用于宿主定义的排序先验，例如：

- 来源重要性
- 内容质量
- 信任分
- 目录或集合权重

这是刻意保持通用的。`indexbind` 不定义该元数据字段的含义。

### `minScore`

```ts
const hits = await index.search('rust guide', {
  topK: 10,
  minScore: 0.05,
});
```

它在重排与得分调整之后丢弃低分尾部命中。

当你想要以下效果时使用：

- 尾部更少弱的语义近邻
- 置信度低时返回少于 `topK` 条命中
- 为固定检索 profile 提供稳定截断

`minScore` 在索引、嵌入后端、重排器选择与得分调整 profile 都固定时最有用。把它当作 profile 调优的尾部截断，而不是跨所有检索配置的普适置信度。

## 推荐默认值

对许多内嵌产品，一个好的起点是：

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

然后只在宿主应用有明确产品规则应影响排序时，再叠加 `metadata` 过滤与 `scoreAdjustment`。

当你有了稳定的搜索 profile、想裁掉弱尾部匹配时，再加 `minScore`。

## 这些旋钮解决不了什么

这些控制项帮助提升检索质量，但不能替代：

- 面向特定领域的文档规范化
- 宿主侧查询改写
- 自定义摘要渲染
- 产品级导航或渲染逻辑

这些关注点仍然属于内嵌 `indexbind` 的应用本身。
