---
title: CLI
order: 20
date: 2026-03-25
summary: 构建、检查、搜索与基准测试 indexbind 工件的命令。
---

# CLI

安装 npm 包后，通过 `npx indexbind ...` 或你包管理器的本地 bin 运行公共 CLI。

主要命令：

- `npx indexbind build [input-dir] [output-file] [--backend <hashing|model-id>]`
- `npx indexbind build-bundle [input-dir] [output-dir] [--backend <hashing|model-id>]`
- `npx indexbind update-cache [input-dir] [cache-file] [--backend <hashing|model-id>] [--git-diff] [--git-base <rev>]`
- `npx indexbind export-artifact <output-file> [--cache-file <path>]`
- `npx indexbind export-bundle <output-dir> [--cache-file <path>]`
- `npx indexbind inspect <artifact-file>`
- `npx indexbind search <artifact-file> <query> [flags]`
- `npx indexbind benchmark <artifact-file> <queries-json>`

示例：

```bash
npx indexbind build ./docs
npx indexbind build . ./index.sqlite --backend hashing
npx indexbind build-bundle ./docs
npx indexbind update-cache ./docs --git-diff
npx indexbind export-artifact ./index.sqlite --cache-file ./docs/.indexbind/build-cache.sqlite
npx indexbind export-bundle ./index.bundle --cache-file ./docs/.indexbind/build-cache.sqlite
npx indexbind inspect ./docs/.indexbind/index.sqlite
npx indexbind search ./docs/.indexbind/index.sqlite "rust guide"
npx indexbind search ./docs/.indexbind/index.sqlite "rust guide" --text
npx indexbind benchmark ./docs/.indexbind/index.sqlite fixtures/benchmark/basic/queries.json
```

## 输出模式

命令默认输出 JSON。

当你想要面向终端的紧凑摘要时，加 `--text`：

```bash
npx indexbind inspect ./docs/.indexbind/index.sqlite --text
npx indexbind search ./docs/.indexbind/index.sqlite "rust guide" --text
```

这个默认值是有意为之，让智能体、shell 脚本与 CI 任务无需额外解析即可消费 CLI 输出。

默认路径规则：

- 省略 `input-dir` 表示当前目录
- 构建命令省略 `output-file`、`output-dir` 或 `cache-file` 时写入 `<input-dir>/.indexbind/`
- `export-*` 仍要求显式输出路径；省略 `--cache-file` 时使用 `./.indexbind/build-cache.sqlite`

扫描默认值：

- 忽略隐藏文件与目录
- 尊重嵌套的 `.gitignore` 规则
- 忽略常见的生成物或依赖目录，例如 `node_modules/`、`target/`、`dist/` 与 `build/`

索引级约定文件：

- `indexbind.build.js` 与 `indexbind.search.js` 只从确切的 `input-dir` 或工件 `sourceRoot` 自动发现
- 如果你索引 `./docs`，把它们放在 `./docs/`，即 `./docs/.indexbind/` 旁边
- 它们只影响那个被索引的根；不存在仓库根目录或当前工作目录回退

嵌入后端选择：

- 传入 `--backend hashing`
- 或用 `--backend <model-id>` 传入任意 `model2vec` 模型 id

省略 `--backend` 时使用当前默认后端。

## 增量缓存流程

推荐顺序：

1. `update-cache` 刷新可变构建缓存
2. `export-artifact` 写出全新 SQLite 工件
3. 需要时用 `export-bundle` 写出全新规范化 bundle

`update-cache` 默认做全量目录扫描。加 `--git-diff` 用 Git 作为变更检测快速路径。当你想相对特定修订做 diff 且复用同一缓存时，加 `--git-base <rev>`。

## 构建约定

可选的 `indexbind.build.js` 导出可以扩展默认目录扫描器：

```js
export function includeDocument(relativePath, ctx) {
  return true;
}

export async function transformDocument(document, ctx) {
  return document;
}
```

`transformDocument()` 接收 frontmatter 解析之后的默认规范化文档形态。它是推导 `canonicalUrl`、注入元数据或规范化标题/摘要的合适位置，且不需要替换扫描器。

## 搜索约定

可选的 `indexbind.search.js` 导出可以为 `indexbind search` 定义默认 profile：

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

export function transformQuery(query, ctx) {
  return { query };
}
```

先应用 `profiles.default`，然后显式 CLI 标志覆盖它。`transformQuery()` 只改写查询字符串。

## 搜索标志

用 `search` 在已构建的 SQLite 工件上实验检索设置。

支持的标志：

- `--top-k <n>`
- `--mode <hybrid|vector|lexical>`
- `--reranker embedding-v1|heuristic-v1`
- `--candidate-pool-size <n>`
- `--relative-path-prefix <prefix>`
- `--metadata key=value`（可重复）
- `--score-adjust-metadata-multiplier <field>`
- `--min-score <float>`
- `--text`

示例：

```bash
npx indexbind search ./docs/.indexbind/index.sqlite "rust guide" \
  --top-k 5 \
  --mode vector \
  --reranker heuristic-v1 \
  --candidate-pool-size 25 \
  --min-score 0.05 \
  --text
```

- `--mode vector` 表示仅向量检索。
- `--mode lexical` 表示仅词法检索。

## 触发示例

一种简单的本地钩子模式是在分支变更后更新缓存：

```bash
#!/usr/bin/env bash
set -euo pipefail

npx indexbind update-cache ./docs --git-diff
npx indexbind export-artifact ./index.sqlite --cache-file ./docs/.indexbind/build-cache.sqlite
```

这只是一个适配器示例。缓存逻辑仍在共享的增量引擎里，因此同一流程也可以由智能体脚本、任务运行器或文件 watcher 调用。
