---
title: Openwrt 添加一个新设备
date: 2019-01-07 15:23:51
categories: openwrt
---

--------------------------
#### 简要说明
* 参考taget/linux下其他设备的目录结构, Openwrt 添加新设备 RuiFeng Bus Wrt (cpu=mt7620a, flash=16M)

#### 添加target.mk配置
``` bash
$ cd openwrt/target/linux/ramips/image
$ vim mt7620.mk
```

``` bash
define Device/rf-bwrt
  DTS := RF-BWRT
  IMAGE_SIZE := $(ralink_default_fw_size_16M)
  SUPPORTED_DEVICES += rf-bwrt
  DEVICE_TITLE := RuiFeng Bus Wrt (16M)
  DEVICE_PACKAGES := kmod-mt76x2 kmod-usb2 kmod-usb-ohci kmod-sdhci-mt7620
endef
TARGET_DEVICES += rf-bwrt
```
--------------------------------
#### 添加dts配置文件
``` bash
$ cd openwrt/target/linux/ramips/dts
$ touch RF-BWRT.dts  RF-BWRT.dtsi
```

``` bash 
$ vim RF-BWRT.dts
/dts-v1/;

#include "RF-BWRT.dtsi"

/ {
	compatible = "rflink,rf-bwrt", "ralink,mt7620a-soc";
	model = "RF-BWRT";
};

&spi0 {
	status = "okay";

	en25q128@0 {
		compatible = "jedec,spi-nor";
		reg = <0>;
		spi-max-frequency = <10000000>;

		partitions {
			compatible = "fixed-partitions";
			#address-cells = <1>;
			#size-cells = <1>;

			partition@0 {
				label = "u-boot";
				reg = <0x0 0x30000>;
				read-only;
			};

			partition@30000 {
				label = "u-boot-env";
				reg = <0x30000 0x10000>;
				read-only;
			};

			factory: partition@40000 {
				label = "factory";
				reg = <0x40000 0x10000>;
				read-only;
			};

			firmware: partition@50000 {
				compatible = "denx,uimage";
				label = "firmware";
				reg = <0x50000 0xfb0000>;
			};
		};
	};
};
```

``` bash 
$ vim RF-BWRT.dts
#include "mt7620a.dtsi"

#include <dt-bindings/gpio/gpio.h>
#include <dt-bindings/input/input.h>

/ {
	compatible = "rflink,rf-bwrt", "ralink,mt7620a-soc";

	aliases {
		led-boot = &led_power;
		led-failsafe = &led_power;
		led-running = &led_power;
		led-upgrade = &led_power;
	};

	chosen {
		bootargs = "console=ttyS0,57600";
	};

	gpio-leds {
		compatible = "gpio-leds";
		led_power: power {
			label = "rf-bwrt:green:power";
			gpios = <&gpio1 14 GPIO_ACTIVE_HIGH>;
		};
		usb {
			label = "rf-bwrt:green:usb";
			gpios = <&gpio1 15 GPIO_ACTIVE_HIGH>;
			trigger-sources = <&ohci_port1>, <&ehci_port1>;
			linux,default-trigger = "usbport";
		};
		air {
			label = "rf-bwrt:green:wifi";
			gpios = <&gpio3 0 GPIO_ACTIVE_LOW>;
		};
	};

	gpio-keys-polled {
		compatible = "gpio-keys-polled";
		poll-interval = <20>;
		reset {
			label = "reset";
			gpios = <&gpio0 1 GPIO_ACTIVE_LOW>;
			linux,code = <KEY_RESTART>;
		};
	};
};

&gpio0 {
	status = "okay";
};

&gpio1 {
	status = "okay";
};

&gpio3 {
	status = "okay";
};

&sdhci {
	status = "okay";
};

&ehci {
	status = "okay";
};

&ohci {
	status = "okay";
};

&ethernet {
	mtd-mac-address = <&factory 0x4>;
	mediatek,portmap = "wllll";
};

&wmac {
	ralink,mtd-eeprom = <&factory 0>;
};

&pinctrl {
	state_default: pinctrl0 {
		default {
			ralink,group = "i2c", "uartf", "wled", "spi refclk", "pa";
			ralink,function = "gpio";
		};
	};
};

&pcie {
	status = "okay";
};
```