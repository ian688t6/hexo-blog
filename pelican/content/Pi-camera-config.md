---
title: Pi 配置摄像头
date: 2019-01-06 17:43:16
Category: pi
---

----------------------
#### 安装
* 摄像头模组安装到pi的mipi接口上

----------------------
#### 配置
``` bash
$ sudo raspi-config
```

``` bash
5 Intefacing Options 	Configure connections to peripherals
	P1 Camera 	Enable/Disable connection to Raspberry Pi Camera
```

----------------------
#### 测试
``` bash
$ raspstill -v -f
```

----------------------
#### 结束
