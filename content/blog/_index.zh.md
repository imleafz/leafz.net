---
title: 博客
description: 文章与工程笔记。
comments: false
type: blog
# 顶层栏目：开启 LLMSFULL 全文包（/blog/llms-full.txt）。
# front matter 的 outputs 整体替换站点级 section 列表，需写全原有格式。
outputs: [HTML, RSS, print, markdown, LLMSFULL]
icon: fa-solid fa-blog
sidebar_root_for: self
sidebar_root_link_self: true
menus:
  main:
    identifier: blog
    weight: 10
cascade:
  type: blog
  footer_style: slim
  reading_time: true
---

记录做过的事与想明白的道理。
