---
title: Openwrt 调试配置4G模组
date: 2019-01-03 16:09:27
categories: openwrt
---

-------------------
#### 简要说明
* Openwrt 版本是18.06, 内核版本为linux-4.14.90, 4G模块为Quectel EC20;

* 移植好4G模块的内核驱动后，主要是对4G模块进行配置；

* 4G模块至少有两种配置协议，一种是ppp拨号方式，第二种是qmi方式；

* 如果路由器或者是modem只支持AT命令的方式那么就只能通过拨号方式上网了；

* QMI方式会比ppp拨号方式要稳定，并且连接更快；

-------------------
#### openwrt 路由器需要安装的包

    usb-modeswitch - It will automatically issue a “special” command to the modem for switching it into the “Working” state
    kmod-mii - Mii driver
    kmod-usb-net - USB to Ethernet
    kmod-usb-wdm
    kmod-usb-net-qmi-wwan
    uqmi - Control utility

``` bash
$ opkg update
$ opkg install usb-modeswitch kmod-mii kmod-usb-net kmod-usb-wdm kmod-usb-net-qmi-wwan uqmi

```

-------------------
#### openwrt 可选择安装的包
``` bash
$ opkg update
$ opkg install kmod-usb-net-cdc-mbim umbim
$ opkg update
$ opkg install kmod-usb-serial-option kmod-usb-serial kmod-usb-serial-wwan
```

-------------------
#### network 配置
``` bash
$ vi /etc/config/network
config interface 'wwan'
        option ifname 'wwan0'
        option proto 'dhcp'
```

-------------------
#### iptables 配置源地址伪装，以及端口转发
* 可直接在openwrt的防火墙配置界面进行配置
``` bash
    go to Network → Firewall, scroll down to wan and click the Edit button
    add a checkmark to the wwan box under Covered networks heading, click Save & Apply
```

------------------
#### 结束