---
layout: post
title: git clone 速度很慢的解决方法
category: Git
tags: git
keywords:
description:
---

1. 浏览器访问<https://www.ipaddress.com/>，获取 github.global.ssl.fastly.net、global-ssl.fastly.net、assets-cdn.github.com 和 github.com 的 ip
2. 修改 hosts，增加 host 映射

    xxx.xxx.xxx.xxx github.global.ssl.fastly.net
    xxx.xxx.xxx.xxx global-ssl.fastly.net
    xxx.xxx.xxx.xxx assets-cdn.github.com
    xxx.xxx.xxx.xxx github.com

3. 更新 DNS 缓存

    # macOS
    dscacheutil -flushcache
    # Windows
    ipconfig /flushdns
    # Linux
    service nscd restart
    # Ubuntu
    sudo /etc/init.d/dns-clean start

4. 只获取代码的最新版，再获取完整历史信息

    git clone --depth=1 https://github.com/xxx/xxx.git
    cd xxx
    git fetch --unshallow