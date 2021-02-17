---
title: NAS整理记录
date: 2021-02-17 13:42:54
tags:
---

由于2020 年贸易战的原因，担心 Google Drive 和 OneDrive 无法继续使用，又暂时没有能力肉翻，遂决定折腾在家部署 NAS。又本身就有淘汰的 CPU i7-4790 加上主板和内存，所以直接购入了机箱 [银欣 CS380](https://item.jd.com/100007972013.html) 和 5 块 [西部数据红盘](https://item.jd.com/694105.html#crumb-wrap)

## 折腾过程

期间折腾过 Unraid、OpenMediaVault、FreeNAS 和 Proxmox，决定使用 Unraid。经过半年时间的使用，Unraid 确实能胜任大部分工作，快捷的应用商店（和 linuxserver 合作的容器部署方案），虚拟机，smb 共享等。但问题也随之而来，部分容器 CPU 占用不正常，比如 qBittorrent 任务一多就会自动停止运行，后来发现 Unriad 容器 CPU 也是虚拟方案，不是直接使用宿主机上的，对于伪强迫症的我，就又开始寻找新的方案

过年期间在 v2ex 上看到了以前收藏的帖子 [开源技术够用了么？我的 NAS 选型与搭建过程](https://www.v2ex.com/t/723211)，决定参照他的搭建过程来重新部署我的 NAS。

### 系统和存储配置

操作系统使用的是 Debian testing，部署在 SSD 硬盘上，5 块 1T 硬盘做 zfs raidz1（也就是raid5）

```shell
sudo zpool create -f -o ashift=12 -m /data mypool raidz \
ata-WDC_WD10EFRX-68FYTN0_WD-WCC4J0SEF254 \
ata-WDC_WD10EFRX-68FYTN0_WD-WCC4J0SEFN9A \
ata-WDC_WD10EFRX-68FYTN0_WD-WCC4J0TDU34X \
ta-WDC_WD10EFRX-68FYTN0_WD-WCC4J0TDUPT3 \
ata-WDC_WD10EFRX-68FYTN0_WD-WCC4J7VF4NUS

sudo zpool status
  pool: mypool
 state: ONLINE
  scan: scrub repaired 0B in 00:26:26 with 0 errors on Sun Feb 14 00:50:28 2021
config:

	NAME                                          STATE     READ WRITE CKSUM
	mypool                                        ONLINE       0     0     0
	  raidz1-0                                    ONLINE       0     0     0
	    ata-WDC_WD10EFRX-68FYTN0_WD-WCC4J0SEF254  ONLINE       0     0     0
	    ata-WDC_WD10EFRX-68FYTN0_WD-WCC4J0SEFN9A  ONLINE       0     0     0
	    ata-WDC_WD10EFRX-68FYTN0_WD-WCC4J0TDU34X  ONLINE       0     0     0
	    ata-WDC_WD10EFRX-68FYTN0_WD-WCC4J0TDUPT3  ONLINE       0     0     0
	    ata-WDC_WD10EFRX-68FYTN0_WD-WCC4J7VF4NUS  ONLINE       0     0     0

errors: No known data errors

sudo zpool list
NAME     SIZE  ALLOC   FREE  CKPOINT  EXPANDSZ   FRAG    CAP  DEDUP    HEALTH  ALTROOT
mypool  4.55T   155G  4.40T        -         -     1%     3%  1.00x    ONLINE  -
```

### 部署服务

用 ansible 部署服务，比如 Nextcloud，qBittorrent（下载PT），qBittorrent-Enhanced-Edition（下载PT），aria2（下载百度网盘）等等，ansible 仓库在 [playbooks](https://github.com/bernylinville/playbooks  )

> 没有选择做 TimeMachine 备份是因为我定期会重装 mac 系统，备份还原参考 [macOS 日常 & 备份 & 还原](https://macosdoc.googo.cc/)

### 外部访问

参考 [docker-cloudflare-ddns](https://github.com/oznu/docker-cloudflare-ddns) 部署了 Cloudflare DDNS

```yaml
version: '3'
services:
  cloudflare-ddns:
    image: oznu/cloudflare-ddns:latest
    restart: always
    environment:
      - EMAIL=<Cloudflare_Email>
      - API_KEY=<Cloudflare_API_KEY>
      - ZONE=<example.com>
      - SUBDOMAIN=subdomain
      - PROXIED=false

```

通过 acme.sh 用 Cloudflare api 的选项，生成并部署 SSL 证书，由于强国家庭网络不支持 80 和 443 端口对外转发，通过路由器配置 10000 以上的端口转发来访问服务

## 参考链接

* [开源技术够用了么？我的 NAS 选型与搭建过程](https://www.v2ex.com/t/723211)
