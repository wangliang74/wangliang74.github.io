---
title: "Hugo 博客搭建"
date: 2026-10-16T08:00:00+08:00
draft: false
weight: 1
---

# 选型

静态站点生成、主题生态成熟；配合 GitHub Actions 可自动部署到 Pages。

# 内容与构建分离

Markdown 存仓库，`public/` 本地生成即可，不必提交构建产物。

# 主题与子模块

PaperMod 以子模块引入，升级主题时更新 submodule 指针即可。
