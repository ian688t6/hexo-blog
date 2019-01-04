---
title: Openwrt 创建本地工程
date: 2019-01-03 10:47:40
categories: openwrt
---

-------------------
#### 工程目录结构
* 下面的工程结构主要是为了能将本地修改添加和官方源码分开，一是为了目录结构清晰，二是好对比修改；
``` bash
$ config.mk  config-mt7620a  config-mt7621  dl  Makefile  openwrt-patch  openwrt-src.tar.gz  README.md
```
* config.mk openwrt环境变量配置；

* config-mt7620a openwrt 的配置；

* dl 预先下载的开源包，openwrt编译耗时很长，在编译的同时也会根据依赖关系在编译过程中下载开源软件，所以预先下载需要的依赖包；

* Makefile 工程的第一层makefile, 功能是backport openwrt的源码；

* openwrt-patch 本地对openwrt官方代码的修改或添加patch;

* openwrt-src.tar.gz openwrt的官方代码；

-------------------
#### 工程的Makefile模板
* config.mk 功能设置用到的一些环境变量
``` bash
export TOP := $(shell pwd)
export OPENWRT_SOURCE		:= openwrt-src
export OPENWRT_SOURCE_PATCH	:= openwrt-patch
export OPENWRT_SOC			:= mt7620a
```
* Makefile 功能依次是解压源码，打补丁，编译
``` bash
include config.mk

.PHONY:all cleanall prep build clean
all:
	@echo "rfbrt all"
	pushd $(OPENWRT_SOURCE);make -j8;popd

cleanall:
	@echo "rfbrt clean all"
	@rm -rf $(OPENWRT_SOURCE)

prep:
	@tar xvpf $(OPENWRT_SOURCE).tar.gz 
	@cp -R dl $(OPENWRT_SOURCE)
	@cp -rf $(OPENWRT_SOURCE_PATCH)/* $(OPENWRT_SOURCE)
ifeq ($(OPENWRT_SOC),mt7620)
	@cp config-mt7620a $(OPENWRT_SOURCE)/.config
endif

ifeq ($(OPENWRT_SOC),mt7621)
	@cp config-mt7621 $(OPENWRT_SOURCE)/.config
endif

clean:
	pushd $(OPENWRT_SOURCE);make clean;popd
```

------------------
#### 结束