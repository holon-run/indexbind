---
title: 规范化工件与 WASM
order: 30
date: 2026-03-25
summary: 只检索契约、规范化 bundle 与 wasm 运行时的当前架构方向。
---

# 规范化工件与 WASM

本页是仓库中更长设计工作的简短架构版。

## 方向

当前设计是：

1. 保持 `indexbind` 聚焦检索
2. 为跨运行时查询定义规范化文件工件
3. 用 wasm 作为 web 与 worker 共享的查询运行时
4. 让 SQLite 保持在原生优化路径上，而不是跨运行时公共契约

## 主要决策

### 只检索的 API

运行时返回排序后的命中与元数据。应用使用这些结果，经由自己的存储层与路由层去渲染、取回或跳转内容。

### 规范化工件

可移植工件是一个文件 bundle，包含 manifest、文档、分块、向量、postings 与可选的模型资产。

### WASM 查询运行时

`indexbind/web` 与 `indexbind/cloudflare` 在该规范化 bundle 之上使用 wasm 支撑的查询执行。

### 原生路径

Node 仍通过原生插件支持 SQLite 工件，但它不再是唯一的产品级工件形态。

## 这为什么重要

这个拆分让项目的产品形态更干净：

- 原生查询可以保持快速且实用
- web 与 worker 运行时可以共享一个真正的检索引擎
- 公共 API 不再依赖宿主文件系统假设

完整设计笔记见[仓库架构文档](../../architecture/canonical-artifact-and-wasm.md)。
