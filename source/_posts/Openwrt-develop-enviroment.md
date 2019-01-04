---
title: Openwrt 开发环境搭建
date: 2019-01-02 17:09:42
categories: openwrt
---

#### Host 为 ubuntu-16.04，需要预安装的包:
``` bash
$ sudo apt-get install subversion g++ zlib1g-dev build-essential git python rsync man-db
$ sudo apt-get install libncurses5-dev gawk gettext unzip file libssl-dev wget zip time
```

#### 获取 openwrt 源码：
``` bash
$ git clone https://git.openwrt.org/openwrt/openwrt.git/
$ cd openwrt
$ ./scripts/feeds update -a
$ ./scripts/feeds install -a
$ make menuconfig
```

#### 固件编译：
* “Target System” ⇒ “Atheros AR7xxx/AR9xxx”
* “Target Profile” ⇒ “TP-LINK TL-WR841N/ND v11”

``` bash
$ make
```
or
``` bash
$ make V=s
```

------------------
#### 结束