---
layout: post
title: Docker sudo 权限问题
category: Utils
tags: docker
keywords: docker
description:
---

```bash
# 创建 docker 组
sudo groupadd docker

# 将用户加入 docker 组
sudo gpasswd -a ${USER} docker

# 重启 docker 服务
sudo systemctl restart docker

# 切换当前会话到新 group 或者重启 X 会话
newgrp  docker
```
