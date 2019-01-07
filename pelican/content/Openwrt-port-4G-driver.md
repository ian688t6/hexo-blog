---
title: Openwrt 移植EC20 4G模块
date: 2019-01-03 13:47:17
Category: openwrt
---

----------------------------
#### 简要说明
* Openwrt 版本是18.06, 内核版本为linux-4.14.90, 4G模块为Quectel EC20

----------------------------
#### 主要参考
* 参考Quectel官方文档，《Quectel_UC20_Embedded_Linux_USB_Driver_User_Guide_V1.0.pdf》，《Quectel_WCDMA&LTE_Linux_USB_Driver_User_Guide_V1.7.pdf》，文档均可在Quectel官网下载；

----------------------------
#### 内核驱动移植
* Backport: drivers/net/usb/qmi_wwan.c
``` c
--- a/drivers/net/usb/qmi_wwan.c	2018-12-21 21:13:19.000000000 +0800
+++ b/drivers/net/usb/qmi_wwan.c	2018-12-26 17:14:55.372927727 +0800
@@ -524,6 +524,23 @@
 	return (data[0] & 0xd0) == 0x40;
 }
 
+struct sk_buff *qmi_wwan_tx_fixup(struct usbnet *dev, struct sk_buff *skb, gfp_t flags)
+{
+	if (cpu_to_le16(0x2C7C) != dev->udev->descriptor.idVendor) {
+		return skb;
+	}
+
+	if (skb_pull(skb, ETH_HLEN)) {
+		return skb;
+	} else {
+		dev_err(&dev->intf->dev, "Packet Droped");
+	}
+	dev_kfree_skb_any(skb);
+
+	return NULL;
+}
+
+
 /* disallow addresses which may be confused with IP headers */
 static int qmi_wwan_mac_addr(struct net_device *dev, void *p)
 {
@@ -1285,7 +1302,7 @@
 	{QMI_GOBI_DEVICE(0x05c6, 0x9225)},	/* Sony Gobi 2000 Modem device (N0279, VU730) */
 	{QMI_GOBI_DEVICE(0x05c6, 0x9245)},	/* Samsung Gobi 2000 Modem device (VL176) */
 	{QMI_GOBI_DEVICE(0x03f0, 0x251d)},	/* HP Gobi 2000 Modem device (VP412) */
-	{QMI_GOBI_DEVICE(0x05c6, 0x9215)},	/* Acer Gobi 2000 Modem device (VP413) */
+	/* {QMI_GOBI_DEVICE(0x05c6, 0x9215)},	 Acer Gobi 2000 Modem device (VP413) */
 	{QMI_FIXED_INTF(0x05c6, 0x9215, 4)},	/* Quectel EC20 Mini PCIe */
 	{QMI_GOBI_DEVICE(0x05c6, 0x9265)},	/* Asus Gobi 2000 Modem device (VR305) */
 	{QMI_GOBI_DEVICE(0x05c6, 0x9235)},	/* Top Global Gobi 2000 Modem device (VR306) */
@@ -1314,6 +1331,25 @@
 	{QMI_GOBI_DEVICE(0x12d1, 0x14f1)},	/* Sony Gobi 3000 Composite */
 	{QMI_GOBI_DEVICE(0x1410, 0xa021)},	/* Foxconn Gobi 3000 Modem device (Novatel E396) */
 
+#ifndef QMI_FIXED_INTF
+#define QMI_FIXED_INTF(vend,prod,num) \
+	.match_flags 			= USB_DEVICE_ID_MATCH_DEVICE | USB_DEVICE_ID_MATCH_INT_INFO, \
+	.idVendor			 	= vend, \
+	.idProduct				= prod, \
+	.bInterfaceClass		= 0xff, \
+	.bInterfaceSubClass		= 0xff, \
+	.bInterfaceProtocol		= 0xff, \
+	.driver_info			= (unsigned long)&qmi_wwan_force_int##num,
+#endif
+	{ QMI_FIXED_INTF(0x05C6, 0x9003, 4) },	/* Quectel UC20 */
+	{ QMI_FIXED_INTF(0x2C7C, 0x0125, 4) },	/* Quectel EC25/EC20 R2.0 */
+	{ QMI_FIXED_INTF(0x2C7C, 0x0121, 4) },	/* Quectel EC21 */
+	{ QMI_FIXED_INTF(0x05C6, 0x9215, 4) },	/* Quectel EC20 */
+	{ QMI_FIXED_INTF(0x2C7C, 0x0191, 4) },	/* Quectel EG91 */
+	{ QMI_FIXED_INTF(0x2C7C, 0x0195, 4) },	/* Quectel EG95 */
+	{ QMI_FIXED_INTF(0x2C7C, 0x0306, 4) },	/* Quectel EG06/EP06/EM06 */
+	{ QMI_FIXED_INTF(0x2C7C, 0x0296, 4) },	/* Quectel BG96 */
+
 	{ }					/* END */
 };

```

* Backport: drivers/usb/serial/qcserial.c
``` c
--- a/drivers/usb/serial/qcserial.c	2018-12-21 21:13:19.000000000 +0800
+++ b/drivers/usb/serial/qcserial.c	2018-12-26 17:14:55.372927727 +0800
@@ -92,7 +92,7 @@
 	{USB_DEVICE(0x03f0, 0x241d)},	/* HP Gobi 2000 QDL device (VP412) */
 	{USB_DEVICE(0x03f0, 0x251d)},	/* HP Gobi 2000 Modem device (VP412) */
 	{USB_DEVICE(0x05c6, 0x9214)},	/* Acer Gobi 2000 QDL device (VP413) */
-	{USB_DEVICE(0x05c6, 0x9215)},	/* Acer Gobi 2000 Modem device (VP413) */
+	/* {USB_DEVICE(0x05c6, 0x9215)},	// Acer Gobi 2000 Modem device (VP413) */
 	{USB_DEVICE(0x05c6, 0x9264)},	/* Asus Gobi 2000 QDL device (VR305) */
 	{USB_DEVICE(0x05c6, 0x9265)},	/* Asus Gobi 2000 Modem device (VR305) */
 	{USB_DEVICE(0x05c6, 0x9234)},	/* Top Global Gobi 2000 QDL device (VR306) */

```

* Backport: drivers/usb/serial/usb_wwan.c
``` c
--- a/drivers/usb/serial/usb_wwan.c	2018-12-21 21:13:19.000000000 +0800
+++ b/drivers/usb/serial/usb_wwan.c	2018-12-26 17:14:55.372927727 +0800
@@ -502,6 +502,23 @@
 			  usb_sndbulkpipe(serial->dev, endpoint) | dir,
 			  buf, len, callback, ctx);
 
+	/* add for Quectel for zero packet */
+	if (USB_DIR_OUT == dir) {
+		struct usb_device_descriptor *desc = &serial->dev->descriptor;
+		if ((cpu_to_le16(0x05C6) == desc->idVendor) && (cpu_to_le16(0x9090) == desc->idProduct)) {
+			urb->transfer_flags |= URB_ZERO_PACKET;
+		}
+		if ((cpu_to_le16(0x05C6) == desc->idVendor) && (cpu_to_le16(0x9003) == desc->idProduct)) {
+			urb->transfer_flags |= URB_ZERO_PACKET;
+		}
+		if ((cpu_to_le16(0x05C6) == desc->idVendor) && (cpu_to_le16(0x9215) == desc->idProduct)) {
+			urb->transfer_flags |= URB_ZERO_PACKET;
+		}
+		if (cpu_to_le16(0x2C7C) == desc->idVendor) {
+			urb->transfer_flags |= URB_ZERO_PACKET;
+		}
+	}
+
 	return urb;
 }
 

```

* Backport: drivers/usb/serial/option.c
``` c
--- a/drivers/usb/serial/option.c	2018-12-21 21:13:19.000000000 +0800
+++ b/drivers/usb/serial/option.c	2018-12-27 07:33:43.644125800 +0800
@@ -569,6 +569,17 @@
 
 
 static const struct usb_device_id option_ids[] = {
+	/* for Quectel */
+	{ USB_DEVICE(0x05C6, 0x9090) },	/* Quectel UC15 */
+	{ USB_DEVICE(0x05C6, 0x9003) }, /* Quectel UC20 */
+	{ USB_DEVICE(0x2C7C, 0x0125) }, /* Quectel EC25/EC20 R2.0 */
+	{ USB_DEVICE(0x2C7C, 0x0121) }, /* Quectel EC21 */
+	{ USB_DEVICE(0x05C6, 0x9215) }, /* Quectel EC20 */
+	{ USB_DEVICE(0x2C7C, 0x0191) }, /* Quectel EG91 */
+	{ USB_DEVICE(0x2C7C, 0x0195) }, /* Quectel EG95 */
+	{ USB_DEVICE(0x2C7C, 0x0306) }, /* Quectel EG06/EP06/EM06 */
+	{ USB_DEVICE(0x2C7C, 0x0296) }, /* Quectel GG96 */
+
 	{ USB_DEVICE(OPTION_VENDOR_ID, OPTION_PRODUCT_COLT) },
 	{ USB_DEVICE(OPTION_VENDOR_ID, OPTION_PRODUCT_RICOLA) },
 	{ USB_DEVICE(OPTION_VENDOR_ID, OPTION_PRODUCT_RICOLA_LIGHT) },
@@ -1927,7 +1938,8 @@
 	{ USB_DEVICE_INTERFACE_CLASS(0x2001, 0x7d01, 0xff) },			/* D-Link DWM-156 (variant) */
 	{ USB_DEVICE_INTERFACE_CLASS(0x2001, 0x7d02, 0xff) },
 	{ USB_DEVICE_INTERFACE_CLASS(0x2001, 0x7d03, 0xff) },
-	{ USB_DEVICE_INTERFACE_CLASS(0x2001, 0x7d04, 0xff) },			/* D-Link DWM-158 */
+	{ USB_DEVICE_INTERFACE_CLASS(0x2001, 0x7d04, 0xff),			/* D-Link DWM-158 */
+	 .driver_info = RSVD(4) | RSVD(5) },
 	{ USB_DEVICE_INTERFACE_CLASS(0x2001, 0x7d0e, 0xff) },			/* D-Link DWM-157 C1 */
 	{ USB_DEVICE_INTERFACE_CLASS(0x2001, 0x7e19, 0xff),			/* D-Link DWM-221 B1 */
 	  .driver_info = RSVD(4) },
@@ -1977,6 +1989,9 @@
 #ifdef CONFIG_PM
 	.suspend           = usb_wwan_suspend,
 	.resume            = usb_wwan_resume,
+	/* add for Quectel */
+	.reset_resume      = usb_wwan_resume,
+
 #endif
 };
 
@@ -2014,12 +2029,25 @@
 	    iface_desc->bInterfaceClass != USB_CLASS_CDC_DATA)
 		return -ENODEV;
 
-	/*
-	 * Allow matching on bNumEndpoints for devices whose interface numbers
-	 * can change (e.g. Quectel EP06).
-	 */
-	if (device_flags & NUMEP2 && iface_desc->bNumEndpoints != 2)
+	/* added by Quectel UC20's interface 4 can be used as USB Network device */
+	if ((cpu_to_le16(0x05C6) == serial->dev->descriptor.idVendor) &&
+		(cpu_to_le16(0x9003) == serial->dev->descriptor.idProduct) &&
+		(4 <= serial->interface->cur_altsetting->desc.bInterfaceNumber)) {
+		return -ENODEV;
+	}
+
+	/* added by Quectel EC20's interface 4 can be used as USB Network device */
+	if ((cpu_to_le16(0x05C6) == serial->dev->descriptor.idVendor) &&
+		(cpu_to_le16(0x9215) == serial->dev->descriptor.idProduct) &&
+		(4 <= serial->interface->cur_altsetting->desc.bInterfaceNumber)) {
+		return -ENODEV;
+	}
+
+	/* added by Quectel EC25/EC21/EC20 R2.0/EG91/EG95/EG06/EP06/EM06/BG96's interface 4 can be used as USB Network device */
+	if ((cpu_to_le16(0x2C7C) == serial->dev->descriptor.idVendor) &&
+		(4 <= serial->interface->cur_altsetting->desc.bInterfaceNumber)) {
 		return -ENODEV;
+	}
 
 	/* Store the device flags so we can use them during attach. */
 	usb_set_serial_data(serial, (void *)device_flags);

```

* 配置内核选项
``` bash
$ make kernel_menuconfig
```
1. 配置 CONFIG_USB_SERIAL_OPTION
``` bash
[*]Device Drivers
    [*]USB Support
        [*]USB Serial Converter support
            [*]USB driver for GSM and CDMA modems
```
2. 配置 CONFIG_USB_ACM
``` bash
[*]Device Drivers
    [*]USB Support
        [*]USB Modem (CDC ACM) support
```

3. 配置 CONFIG_USB_NET_QMI_WWAN
``` bash
[*] Device Drivers →
    -*- Network device support →
            USB Network Adapters →
                {*} Multi-purpose USB Networking Framework
                <*> QMI WWAN driver for Qualcomm MSM based 3G and LTE modems
```
4. 配置 CONFIG_PPP_ASYNC CONFIG_PPP_SYNC_TTY CONFIG_PPP_DEFLATE
``` bash
[*] Device Drivers →
    [*] Network device support →
        [*] PPP (point-to-point protocol) support
```

------------------
#### 结束
