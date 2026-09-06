---
title: Web 与 Cloudflare
order: 20
date: 2026-03-25
summary: 在浏览器、worker 与 Cloudflare Workers 中加载规范化 bundle。
---

# Web 与 Cloudflare

`indexbind` 有两个面向 web 的入口：

- `indexbind/web`
- `indexbind/cloudflare`

它们都查询规范化 bundle，但 Cloudflare Workers 需要独立入口，让 wasm 通过静态 Worker 模块导入加载。

## 浏览器或标准 Worker

```ts
import { openWebIndex } from 'indexbind/web';

const index = await openWebIndex('/search/index.bundle');
const hits = await index.search('cloudflare wasm');
```

如果你的站点始终只用词法检索，可以用词法 profile 打开 bundle，让该运行时实例不加载向量与模型文件：

```ts
const index = await openWebIndex('/search/index.bundle', {
  modeProfile: 'lexical',
});
const hits = await index.search('cloudflare wasm');
```

## Cloudflare Worker

```ts
import { openWebIndex } from 'indexbind/cloudflare';

export default {
  async fetch(request: Request): Promise<Response> {
    const index = await openWebIndex('https://assets.example.com/index.bundle');
    const hits = await index.search(new URL(request.url).searchParams.get('q') ?? '');
    return Response.json(hits);
  },
};
```

如果你的宿主把 bundle 文件虚拟化、而不是暴露公开 bundle URL，传入自定义 `fetch` 实现：

```ts
const index = await openWebIndex(new URL('https://mdorigin-search.invalid/index.bundle/'), {
  fetch: (input, init) => env.ASSETS.fetch(new Request(input, init)),
});
```

如果你的宿主应用通过 Workers Assets 与虚拟基础 URL 提供 bundle 文件，参见手工测试用例：

- `fixtures/manual/cloudflare-worker-issue-18`

它复现了与 `mdorigin` 相同的形态：一个假的 bundle 源，加上把请求临时重定向进 `ASSETS.fetch(...)` 的 `fetch` 实现。

## 嵌入后端

规范化 bundle 当前可以用以下后端构建：

- `hashing`
- `model2vec`

当你想要更高质量的检索路径时，把 `model2vec` 作为默认推荐后端。当你想要更轻、偏兼容取向的后端或更小的构建依赖面时，使用 `hashing`。

对 `model2vec`，构建步骤会把以下文件复制进 bundle：

- `model/tokenizer.json`
- `model/config.json`
- `model/model.safetensors`

这让 wasm 运行时无需宿主文件系统访问就能嵌入查询。

## 包边界

npm 包包含运行时代码与 wasm 文件。

bundle 工件包含你的实际索引数据、向量、postings 与可选的 `model2vec` 模型文件。
