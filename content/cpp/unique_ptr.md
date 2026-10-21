---
title: "unique_ptr"
date: 2026-10-12T14:30:00+08:00
draft: false
weight: 2
---

# unique_ptr

独占所有权，开销接近裸指针，适合作为默认选择。

# shared_ptr 与 weak_ptr

引用计数共享所有权；`weak_ptr` 用于打破循环引用，或观察对象是否仍存活。

# 不要混用 new 与 make_shared

优先 `std::make_shared`，一次分配控制块与对象，异常安全更好。
