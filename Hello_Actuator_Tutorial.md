## Hello, Actuator!!!
欢迎进入IoT的世界！！！让我们来学习如何让硬件板上的LED灯闪起来！<br><br>
在这第一篇教程中，让我们使用一块树莓派硬件板，通过GPIO连接LED灯。然后，我们使用Lithosphere IoT平台提供的Actuator插件，只需要做简单的开发，我们就可以用App遥控IoT硬件板设备来让LED闪灯了。

<br><br>
## 1 前置条件：
**Java = 8**（树莓派端）<br>
**Java >= 11**（服务器端）<br>
**Granite Lite IoT XMPP Server**<br>
点击这里下载[Granite Lite IoT XMPP Server](https://github.com/TheFirstLineOfCode/granite/releases/download/1.0.3-RELEASE/granite-lite-iot-1.0.3-RELEASE.zip)<br>
**Raspberry Pi Zero W硬件板**<br>
**LED模块**<br>
**几颗杜邦线**<br>
![](https://dongger-s-img-repo.oss-cn-shenzhen.aliyuncs.com/images/hello_actuator_hardwares.jpg)

<br><be>
## 2 概念
这篇教程，会涉及到硬件控制接口，以及IoT终端设备的分发部署。简单介绍相关概念如下。

<br><br>
### 2.1 GPIO
GPIO是英文General-Purpose Input/Output的缩写。
<br><br>
简单来说，GPIO就是硬件板上一组引针，这些引针可以用来控制电信号输入输出。
<br><br>
由于这些引针是可以编程来控制的，从而可以实现和外部的电路模块板通讯。
<br><br>
下图是Raspberry Pi Zero W上的GPIO接口。
![](https://dongger-s-img-repo.oss-cn-shenzhen.aliyuncs.com/images/gpio.png)
<br><br>
在本教程中，我们使用GPIO来连接外部的LED模块，从而实现LED小灯的程序控制。

<br><br>
### 2.2 授权入网

<br><br>
## 3 安装和配置Raspberry Pi
树莓派Zero W，我在淘宝上买153元RMB，记住要买一个SD卡来插上。<br><br>
我们需要做的第一件事，是给树莓派装上系统。<br><br>
在没有监视器和键盘的情况下，给树莓派装上OS。在Raspberry Pi的官方的文档里，把这叫做Headless Setting Up。可以参考下面的链接，了解如何做Headless Setting Up。<br>
[官方Headless Setting Up文档](https://www.raspberrypi.com/documentation/computers/configuration.html#setting-up-a-headless-raspberry-pi)
<br><br>
如果你不想阅读英文，可以看XDongger这篇关于树莓派安装和配置的文章。<br>
[安装和配置树莓派](https://juejin.cn/post/7247024731443101755)<br><br>

## 4 连接硬件
我们先来看看LED模块长啥样。
![](https://dongger-s-img-repo.oss-cn-shenzhen.aliyuncs.com/images/led_module.jpg)
这个LED模块在淘宝上买，5块钱一个。
<br><br>
我们可以看到，它有3个引脚：
| 引脚名        | 作用 |
| --------- | ---------- |
| GND     | 地线 |
| VCC     | 5V电源 |
| IN      | LED控制 |
<br><br>
我们需要将LED模块，通过树莓派硬件板上的GPIO接口，连接到硬件板上。
<br><br>
如何对接呢？让我们我们来看看树莓派的GPIO接口。
![](https://dongger-s-img-repo.oss-cn-shenzhen.aliyuncs.com/images/connect_led_to_raspbery_pi_by_gpio.png)
<br><br>
我们将LED的VCC接口，接在GPIO的5V Powerd引脚上。
我们将LED的GND接口，接在GPIO的Ground引脚上。
我们将LED的IN接口，接在GPIO 2引脚上。
<br><br>
接好后，看上去是这样的。
![](https://dongger-s-img-repo.oss-cn-shenzhen.aliyuncs.com/images/connect_led_to_raspberry_pi_by_gpio_2.jpg)

<br><br>
## 5 配置硬件板基础软件环境
在[2 安装和配置Raspberry Pi](#2-安装和配置raspberry-pi)步骤中，我们只是得到了一个初始化版本的RaspBerry Pi OS操作系统。要想在上面能运行Lithosphere IoT平台的终端程序，还得做以下的配置。

<br><br>
### 5.1 安装Open JDK
登录到树莓派硬件板。
```
ssh pi@192.168.1.180
```
> **注：**<br>
>* 192.168.1.180是树莓派板的网络地址。请改为你配置树莓派板时，指定的静态IP地址。
>* pi为树莓派用户。请改为你配置树莓派时，初始化创建的用户名。

<br><br>
执行以下指令安装Open JDK8。
```
sudo apt-get install openjdk-8-jdk
```
> **注：**
>* 为何要安装Open JDK 8版本？
>>Raspberry Pi OS默认自带的Open JDK版本是Open JDK 11。这意味着，如果你执行以下指令。系统会默认安装Open JDK 11。
>>```
>>sudo apt-get install default-jdk
>>```
>>这个默认版本的JDK，在Raspberry Pi Zero W上不能正常工作。原因是因为Raspberry Zero W使用ARMv6版本CPU。而Raspberry Pi OS默认带的OPen JDK 11，仅适用于ARMv7和ARMv8版本的CPU。<br><br>
>>当然，有一些其它办法可以在Raspberry Pi Zero W上来安装JDK 11。<br><br>
>>在这篇教程里，我选择改装Open JDK 8的解决方案。看上去，这是一个简单有效的解决方案。

<br><br>
### 5.2 安装WiringPi
我们在后续开发中，会使用Pi4J库V1.3版本来控制GPIO。Pi4J是一个开源Java库，它使用JNI来调用C程序编写的WiringPi库来控制GPIO。
<br><br>
所以，我们需要安装WiringPi。
<br><br>
Raspberry Pi OS并没有自带WiringPi软件包。我们通过以下指令安装WiringPi。
```
wget https://github.com/WiringPi/WiringPi/releases/download/2.61-1/wiringpi-2.61-1-armhf.deb

sudo dpkg -i wiringpi-2.61-1-armhf.deb
```

## 6 开发协议包
为何我们需要单独开发一个包含协议对象的协议包？<br><br>
这是因为我们会使用叫OXM（Object-XMPP Mapping）的技术。简单来说，我们希望在开发中，能够屏蔽掉XMPP协议实现的细节，简化开发过程。<br><br>
关于OXM，可以参考概念文档中的[OXM章节](./Concepts.md#OXM)
<br><br>
使用OXM，我们会定义的协议对象，这些协议对象，在客户端和服务器端，都会被使用。协议对象是可复用的。<br><br>
所以，我们最好定一个单独的协议包，已便于在后面客户端和服务器端复用这些协议对象。
<br><br>
#### 5.2.1 创建协议工程
创建hello-actuator-protocol工程，pom.xml如下：
```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns="http://maven.apache.org/POM/4.0.0"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">

	<modelVersion>4.0.0</modelVersion>

	<parent>
		<groupId>com.thefirstlineofcode.basalt</groupId>
		<artifactId>com.thefirstlineofcode.basalt</artifactId>
		<version>1.1.0-RELEASE</version>
	</parent>

	<groupId>com.thefirstlineofcode.lithosphere.tutorials.helloactuator</groupId>
	<artifactId>hello-actuator-protocol</artifactId>
	<name>Hello actuator protocol</name>
	<version>0.0.1-RELEASE</version>
	
	<dependencies>
		<dependency>
			<groupId>com.thefirstlineofcode.basalt</groupId>
			<artifactId>basalt-oxm</artifactId>
		</dependency>
	</dependencies>
	
	<repositories>
		<repository>
			<id>com.thefirstlineofcode.releases</id>
			<name>TheFirstLineOfCode Repository - Releases</name>
			<url>http://120.25.166.188:9090/repository/maven-releases/</url>
		</repository>
	</repositories>
	
</project>
```

> **代码说明**
>* 指定parent为com.thefirstlineofcode.basalt:com.thefirstlineofcode.basalt，这样可以直接引用basalt parent pom的依赖管理配置。
><br><br>
>* 因为要使用OXM(Object-XMPP Mapping)，所以须依赖basalt-oxm库。
>* 目前，Lithosphere的开源库，仅被部署在TheFirstLineOfCode的私有的maven服务器上。为了构建时能够正确找到开源依赖库，需要配置com.thefirstlineofcode.releases的repository。
>>>```
>>><repositories>
>>>	<repository>
>>>	<id>com.thefirstlineofcode.releases</id>
>>>	<name>TheFirstLineOfCode Repository - Releases</name>
>>>	<url>http://120.25.166.188:9090/repository/maven-releases</url>
>>>	</repository>
>>></repositories>
>>>```
><br><br>
