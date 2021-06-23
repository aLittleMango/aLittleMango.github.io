---
layout: post
title: centos 配置 docker
category: Utils
tags: centos
keywords: centos docker
description:
---

### 用户添加到 docker 组

```bash
sudo gpasswd -a user docker
newgrp docker               # 更新用户组
```

### 重启 docker

#### centos7

```bash
systemctl daemon-reload

systemctl restart docker
```

### 修改 docker 工作目录

*注意：目前网上大多数配置参数是  --graph ，其实这是老版本中的使用方法，新版本已经抛弃，改用了 --data-root ，具体可以通过命令 dockerd --help 查看使用的参数。*

#### centos7

1. 修改配置文件

    - 方法 1: 新建或者编辑 /etc/docker/daemon.json

        ```bash
        {
            "data-root": "/data/docker"
        }
        ```

    - 方法 2: 编辑文件 /usr/lib/systemd/system/docker.service

        ```bash
        [Service]
        Type=notify
        # the default is not to use systemd for cgroups because the delegate issues still
        # exists and systemd currently does not support the cgroup feature set required
        # for containers run by docker
        ExecStart=/usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock --data-root=/data/docker
        ExecReload=/bin/kill -s HUP $MAINPID
        TimeoutSec=0
        RestartSec=2……
        ```

2. 重启 docker

    ```bash
    systemctl daemon-reload

    systemctl restart docker
    ```

