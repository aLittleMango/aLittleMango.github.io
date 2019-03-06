---
layout: post
title: CUDA & CUDNN 安装
category: CUDA
tags: cuda
description:
---

## 一、cuda安装

### 1. 安装cuda_8.0.61

安装cuda_8.0.61到指定路径  ${HOME}/local/cuda_8.0.61
```bash
$ chmod a+x cuda_8.0.61_375.26_linux.run
$ ./cuda_8.0.61_375.26_linux.run
```

### 2. 添加本地环境变量

```bash
vim ~/.bashrc
```
末尾添加

    export PATH=/home/michael/local/cuda_8.0.61/bin${PATH:+:${PATH}}
    export LD_LIBRARY_PATH=/home/michael/local/cuda_8.0.61/lib64${LD_LIBRARY_PATH:+:${LD_LIBRARY_PATH}}
    export C_INCLUDE_PATH=/home/michael/local/cuda_8.0.61/include${C_INCLUDE_PATH:+:${C_INCLUDE_PATH}}
    export CPLUS_INCLUDE_PATH=/home/michael/local/cuda_8.0.61/include${CPLUS_INCLUDE_PATH:+:${CPLUS_INCLUDE_PATH}}

### 3. 更新环境变量
```bash
$ source ~/.bashrc
```
### 4. 验证

```bash
$ nvcc -V
```

## 二、cudnn安装

```bash
tar -zxvf cudnn-8.0-linux-x64-v6.0.tgz    #解压出来的文件夹名称就叫cuda

cd cuda/include
cp cudnn.h /home/michael/local/cuda_8.0.61/include
cd ../lib64
cp -a lib* /home/michael/local/cuda_8.0.61/lib64
```
这里cudnn已经加入到cuda的包含目录里面了，也可以不放进cuda里面，放外面之后要改环境变量