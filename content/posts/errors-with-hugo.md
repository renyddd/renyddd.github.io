---
title: "Errors With Hugo"
date: 2021-06-15T17:08:06+08:00
draft: false
tags: ["hugo"]
categories: ["hugo"]
series: ["hugo系列"]
---

## 开始使用

### 一篇新的博客

```bash
cd WOWRKDIR
hugo new posts/gdb-use.md
```

 

## 一些错误记录

### 1 部署后未收录文章
仔细阅读[官方文档](https://gohugo.io/getting-started/quick-start/)
> Drafts do not get deployed; once you finish a post, update the header of the post to say draft: false. More info here.

### 2 选择发布分支
github 端 gh-pages。

### 3 Github Actions
在同一个仓库的时候直接无脑复制官方 .github/workflows/main.yml 文件就好了。
