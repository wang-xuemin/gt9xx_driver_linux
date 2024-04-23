# gt9xx_driver_linux
linux driver source code for gt9xx series touch controller.

### 驱动从安卓移植至Linux
安卓驱动链接：https://github.com/goodix/gt9xx_driver_android

移植参考文章：https://blog.51cto.com/u_13360/9348365

适配汇顶 911系列8英寸 分辨率：（1024 * 768）

移植根据厂家提供的cfg进行配置，调整分辨率、适配IC版本根据厂家提供的cfg配置即可

Linux内核版本为：5.15.67

测试工具：evtest
```
evtest
```

查看log，需要在 Makefile 中添加配置。调试完去除即可
```
ccflags-y += -DDEBUG
```

查看log命令
```
cat /dev/kmsg | grep goodix-ts
```

### 驱动安装步骤

1、将gt9xx目录放置至内核源码目录的touchscreen中，路径如下
```
drivers/input/touchscreen/gt9xx
```

2、编辑touchscreen/Kconfig添加
```
source "drivers/input/touchscreen/gt9xx/Kconfig"
```

3、编辑touchscreen/Makefile
```
obj-$(CONFIG_TOUCHSCREEN_GT9XX)	+=  gt9xx/
```

4、设备树desi配置 arch/arm/boot/dts 这里以arm为例，根据实际架构选择对应的目录。
```
&i2c3 {
        pinctrl-names = "default", "sleep";
        i2c-scl-rising-time-ns = <100>;
        i2c-scl-falling-time-ns = <7>;
        status = "okay";
        /delete-property/dmas;
        /delete-property/dma-names;

	goodix-ts@5d {
		compatible = "goodix,gt9xx";
		reg = <0x5d>;
		interrupt-parent = <&gpioi>;
		reset-gpios = <&gpioi 2 GPIO_ACTIVE_LOW>;
		irq-gpios = <&gpioi 7 IRQ_TYPE_EDGE_FALLING>;
		irq-flags = <2>;		
		touchscreen-max-id = <11>;
		touchscreen-size-x = <1024>;
		touchscreen-size-y = <768>;
		goodix,int-sync = <1>;
		goodix,type-a-report = <0>;
		goodix,driver-send-cfg = <1>;
		goodix,cfg-group0 = [63 00 04 00 03 0A 0D 00 01 0F 1E 0F 50 32 03 05 00 00 00 00 00 00 05 00 00 00 00 8C 2E 0E 27 25 F4 0A 00 00 00 9B 03 1D 00 01 00 00 00 00 00 00 00 00 00 19 4B 94 C5 02 07 00 00 04 A4 1C 00 8A 22 00 70 2B 00 5C 36 00 4D 43 00 4D 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 02 04 06 08 0A 0C 0E 10 12 14 16 18 1A 1C 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 02 04 06 08 0A 0C 0F 10 12 13 14 16 18 1C 1D 1E 1F 20 21 22 24 26 28 29 2A 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 85 01];
		status = "okay";	
	};
};
```

5、配置文件配置 arch/arm/configs 这里以arm为例，根据实际架构选择对应的目录
```
CONFIG_TOUCHSCREEN_GT9XX=y
```

6、进入内核目录编译内核驱动即可 fconfig 对应 arch/arm/configs 配置
```
mkdir -p ../build

make ARCH=arm O="$PWD/../build" fconfig

make ARCH=arm uImage vmlinux dtbs LOADADDR=0xC2000040 O="$PWD/../build"

make ARCH=arm modules O="$PWD/../build"
```
