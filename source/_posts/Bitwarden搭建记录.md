---
title: Bitwarden搭建记录
date: 2021-02-19 14:39:32
tags:
---

密码管理方案从最早的浏览器管理 -> lastpass -> 1password -> keepass，本地保存和 syncthing 共享 keepass 数据。

kepassxc 在 mbp 上一直无法正常使用指纹解锁数据库，正好最近收到 Bitwarden 的安利，遂部署了该服务

## 部署文件

docker-compose.yml 文件如下：

```yaml
version: '3'

services:
  bitwarden:
    image: bitwardenrs/server
    restart: always
    ports:
      - 10001:10001
    volumes:
      - '{{ bitwarden_data }}:/data'
    environment:
      ROCKET_PORT: 10001
      SIGNUPS_ALLOWED: 'true'   # 注册完账号之后，更改为 false 并重启容器
      DOMAIN: "https://bitwarden.example.com/base-dir"

```

> 开启子路由转发，需要添加：DOMAIN: "https://bitwarden.example.com/base-dir"

nginx conf 文件：

```
    location /base-dir/ {
        proxy_set_header Host $http_host;
        proxy_pass http://127.0.0.1:10001;
    }

```

> 注意不能漏掉后面的 /

## 问题

客户端和浏览器插件是独立的，体验很差

## 参考链接

* [bitwarden_rs/wiki](https://github.com/dani-garcia/bitwarden_rs/wiki)

* [Using an alternate base dir](https://github.com/dani-garcia/bitwarden_rs/wiki/Using-an-alternate-base-dir)
