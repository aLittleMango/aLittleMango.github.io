---
layout: post
title: CUDA & CUDNN 查询指令
category: CUDA
tags: cuda
description:
---

## 版本查看

### 1. 查看cuda版本

```bash
$ cat /usr/local/cuda/version.txt
```

### 2. 查看cudnn版本
```bash
$ cat /usr/local/cuda/include/cudnn.h | grep CUDNN_MAJOR -A 2
```