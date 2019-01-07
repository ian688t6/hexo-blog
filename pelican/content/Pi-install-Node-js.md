---
title: Pi 安装 Node.js
date: 2019-01-06 19:17:58
Category: Pi
---

--------------------------
#### 获取node.js package最新的稳定版本
``` bash
$ wget http://node-arm.herokuapp.com/node_latest_armhf.deb
```

--------------------------
#### 安装
``` bash
$ sudo dpkg -i node_latest_armhf.deb
```
--------------------------
#### 验证是否安装成功
``` bash
$ node -v
```

--------------------------
#### 结束

