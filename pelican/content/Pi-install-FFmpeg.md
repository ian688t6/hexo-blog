---
title: Pi 安装 FFmpeg
date: 2019-01-06 20:12:48
Category: pi
---

-------------------------
#### 安装x264
``` bash
$ git clone git://git.videolan.org/x264
$ cd x264
$ ./configure --host=arm-unknown-linux-gnueabi --enable-static --disable-opencl
$ make
$ sudo make install
```

-------------------------
#### 安装ffmpeg
``` bash
$ git clone https://github.com/FFmpeg/FFmpeg.git
$ cd FFmpeg
$ sudo ./configure --arch=armel --target-os=linux --enable-gpl --enable-libx264 --enable-nonfree
$ make
$ sudo make install
```

--------------------------
#### 结束

