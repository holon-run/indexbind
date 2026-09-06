---
title: 运行时模型
order: 20
date: 2026-03-25
summary: 跨 Node、浏览器与 Cloudflare Workers 的只检索运行时边界。
---

# 运行时模型

`indexbind` 刻意只聚焦检索，不做内容加载。

也就是说，运行时返回：

- `docId`
- `relativePath`
- `canonicalUrl`
- `title`
- `summary`
- `metadata`
- `score`
- `bestMatch`

## 为什么只做检索

这让运行时保持可移植。

如果库试图直接读取内容，它就需要为以下场景定义宿主特定契约：

- 本地文件系统路径
- 浏览器 fetch 位置
- Cloudflare 资产路由

那会削弱公共 API 表面，也会让 web 运行时变得不再干净。

取而代之的是，应用拿到命中元数据后，自行决定如何渲染、取回或跳转到源内容。

## 运行时划分

- `indexbind`
  面向 Node 的原生 SQLite 查询运行时
- `indexbind/build`
  面向 Node 的构建 API，负责规范化 bundle 构建与增量缓存导出
- `indexbind/web`
  面向浏览器与标准 worker 的 wasm 规范化 bundle 运行时
- `indexbind/cloudflare`
  面向 Cloudflare Workers 的 wasm 规范化 bundle 运行时
