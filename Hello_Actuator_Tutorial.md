## Hello, Actuator!!!
欢迎进入IoT的世界！！！让我们来学习如何让硬件板上的LED灯闪起来<br><br>
在这第一篇教程中，让我们使用一块树莓派硬件板，通过GPIO连接LED灯。<br><br>
然后，我们使用Lithosphere IoT平台提供的Actuator插件，只需要做简单的开发，我们就可以用App遥控IoT硬件板设备来让LED闪灯了。

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

<br><br>
## 2 概念
这篇教程，会涉及到硬件控制接口，以及IoT终端设备的分发部署。简单介绍相关概念如下。

<br><br>
### 2.1 GPIO
GPIO是英文General-Purpose Input/Output的缩写。<br><br>
简单来说，GPIO就是硬件板上一组引针，这些引针可以用来控制电信号输入输出。<br><br>
由于这些引针是可以编程来控制的，从而可以实现和外部的电路模块板通讯。<br><br>
下图是Raspberry Pi Zero W上的GPIO接口。
![](https://dongger-s-img-repo.oss-cn-shenzhen.aliyuncs.com/images/gpio_pins.png)
<br><br>
在本教程中，我们使用GPIO来连接外部的LED模块，从而实现LED小灯的程序控制。

<br><br>
### 2.2 授权入网
在真实的IoT应用中，当一个IoT设备从库房中拿出来时，它并不能接通电源后立马就开始工作。<br><br>
基于安全性和利于管理的考虑，IoT设备要能够开始工作，需要经过一个“授权入网”的步骤。<br><br>
在这个“授权入网”的过程中，应用一般会：
* 检查设备的合法性
* 检查设备是否在此时被允许入网
* 登记设备相关信息到系统中
* 配置设备 - 例如对设备进行网络配置；发放安全token等。

<br><br>
在不同的应用和不同标准中，“授权入网”可能会有不同的术语和叫法。<br><br>
例如，在LoRaWAN标准中，“授权入网”被叫做"End Device Activation"。官方文档里，这样描述End Device Activation：<br>
All end devices that participate in a LoRaWAN network must be activated. There are two methods of activation you can choose: over-the-air activation (OTAA) or activation by personalization (ABP).<br><br>
Lithosphere IoT Platform基于XMPP通讯协议。XMPP里，对于参与IM网络的用户，需要先做用户注册。<br><br>
在标准XMPP协议扩展XEP-0077(In-Band Registration)里，定义了如何实现IM用户在线注册的功能。<br><br>
遵从XMPP的习惯，Lithosphere里的“授权入网”，被称为IBTR(In-Band Thing Registration)，智能物件在线注册。<br><br>
Lithosphere平台，已经内置实现了IBTR。基于服务器端和客户端的IBTR插件，可以快速开发极度灵活的设备“授权入网”功能。<br><br>
在本篇教程中，我们会看到相关的内容。

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
### 5.1 安装OpenJDK
登录到树莓派硬件板。
```
ssh pi@192.168.1.180
```
> **注：**<br>
>* 192.168.1.180是树莓派板的网络地址。请改为你配置树莓派板时，指定的静态IP地址。
>* pi为树莓派用户。请改为你配置树莓派时，初始化创建的用户名。

<br><br>
执行以下指令安装OpenJDK 8。
```
sudo apt-get install openjdk-8-jdk
```
> **注：**
>* 为何要安装OpenJDK 8版本？
>>Raspberry Pi OS默认自带的OpenJDK版本是OpenJDK 11。这意味着，如果你执行以下指令。系统会默认安装OpenJDK 11。
>>```
>>sudo apt-get install default-jdk
>>```
>>这个默认版本的JDK，在Raspberry Pi Zero W上不能正常工作。原因是因为Raspberry Zero W使用ARMv6版本CPU。而Raspberry Pi OS默认带的OPenJDK 11，仅适用于ARMv7和ARMv8版本的CPU。<br><br>
>>当然，有一些其它办法可以在Raspberry Pi Zero W上来安装OpenJDK 11。<br><br>
>>在这篇教程里，我选择改装OpenJDK 8的解决方案。看上去，这是一个简单有效的解决方案。

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

<br><br>
## 6 开发协议包
为何我们需要单独开发一个包含协议对象的协议包？<br><br>
这是因为我们使用叫OXM（Object-XMPP Mapping）的技术。简单来说，我们希望在开发中，能够屏蔽掉XMPP协议实现的细节，简化开发过程。<br><br>
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
>* 因为要使用OXM(Object-XMPP Mapping)，所以依赖basalt-oxm库。
><br><br>
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

<br><br>
#### 5.2.2 定义协议对象
在Hello，Actuator例子里，我们想用App来遥控IoT设备上的LED灯。我们希望可以遥控LED执行：亮灯、熄灯、闪灯。<br><br>
为此，我们使用以下3个协议对象：
TurnOn
```
@ProtocolObject(namespace="urn:leps:things:simple-light", localName="turn-on")
public class TurnOn {
	public static final Protocol PROTOCOL = new Protocol("urn:leps:things:simple-light", "turn-on");
}
```
<br><br>
TurnOff
```
@ProtocolObject(namespace="urn:leps:things:simple-light", localName="turn-off")
public class TurnOff {
	public static final Protocol PROTOCOL = new Protocol("urn:leps:things:simple-light", "turn-off");
}
```
<br><br>
Flash
```
@ProtocolObject(namespace="urn:leps:things:simple-light", localName="flash")
public class Flash {
	public static final Protocol PROTOCOL = new Protocol("urn:leps:things:simple-light", "flash");
	
	private int repeat;
	
	public Flash() {
		repeat = 1;
	}
	
	public Flash(int repeat) {
		setRepeat(repeat);
	}
	
	public int getRepeat() {
		return repeat;
	}

	public void setRepeat(int repeat) {
		if (repeat < 1)
			throw new IllegalArgumentException("Attribute repeat must be a positive integer.");
		
		this.repeat = repeat;
	}
	
	@Override
	public String toString() {
		return String.format("Flash[repeat=%d]", repeat);
	}
}
```
> **代码说明**
>* OXM框架会帮我们将Project Object转换成XMPP协议文档。当Flash对象的repeat属性值设置成5时，会生成下面的XMPP协议文档：
>>>```
>>><flash xmlns="urn:leps:things:simple-light" repeat=5/>
>>>```
><br><br>
>* 不需要担心"urn:leps:things:simple-light"字符串太长了，会带来传输效率的低下。在Lithosphere IoT平台下，如果使用BXMPP技术，ProtocolObject(namespace="urn:leps:things:simple-light", localName="flash")这段协议头信息，会被转化成3字节的二进制协议信息。
><br><br>
>* TurnOn和TurnOff协议，表达一个不带任何参数的指令，所以它们被设计成不带任何实例属性的空对象。

<br><br>
#### 5.2.3 定义Model Descriptor
让我们来给要控制的IoT设备，定义一个Model Descriptor。<br><br>
为何需要定义Model Descriptor？<br><br>
这是因为Lithosphere完全基于插件架构。在插件架构下，系统功能并不是写死固化的。当一个插件被部署时，它为系统提供了一组应用功能。当这个插件被卸载掉后，它所提供的功能就消失了。<br><br>
正是因为这种插件架构的特性，默认系统是不认识任何IoT设备的，也不能识别任何IoT控制/数据协议。<br><br>
当我们基于插件技术，使用XMPP扩展协议，开发了某种IoT设备的一组应用功能。我们需要使用平台提供的扩展点，将插件提供的应用功能注册到系统中。我们需要明确的告知系统：现在，我们有这些功能了，相关功能由XXX插件提供。<br><br>
这个事说起来啰嗦，实操起来其实比较简单。我们需要在协议包里，定义一个Model Descriptor类。
```
public class HatModelDescriptor extends SimpleThingModelDescriptor {
	public static final String MODEL_NAME = "HAT";
	public static final String DESCRIPTION = "Hello acuator thing";
	
	public HatModelDescriptor() {
		super(MODEL_NAME, DESCRIPTION, false, null, null, createSupportedActions());
	}
	
	private static Map<Protocol, Class<?>> createSupportedActions() {
		Map<Protocol, Class<?>> supportedActions = new HashMap<>();
		supportedActions.put(Flash.PROTOCOL, Flash.class);
		supportedActions.put(TurnOn.PROTOCOL, TurnOn.class);
		supportedActions.put(TurnOff.PROTOCOL, TurnOff.class);
		
		return supportedActions;
	}
}
```
> **代码说明**
>* 继承SimpleThingModelDescriptor，这个基类实现了IThingModelDescriptor接口，提供了一些复用代码，可以让我们编写Model Descriptor时更省事。
><br><br>
>* IoT设备的型号名。在这里，我们把这个被遥控闪灯的IoT设备型号叫做HAT，Hello Actuator Thing的缩写。
><br><br>
>* 这个IoT设备是一个Actuator（执行器）设备，它接受Action指令，然后执行指令对应的操作。它不是Sensor，不需要上报数据。它也不触发事件，不需要事件通知功能。所以构造器里的参数都是false，null。只有最后一个构造器参数，我们用createSupportedActions()，将这个Actuator设备支持的Action都登记到Model Descriptor中。当然，在这个例子里，IoT设备只支持3个指令，TurnOn，TurnOff，Flash。

<br><br>
#### 5.2.4 构建安装协议包
因为需要在后续开发客户端和服务器端插件包时，引用协议包，所以我们在hello-actuator-protocol工程里，执行构建安装指令，把协议包安装到本地maven仓库。
```
cd hello-actuator-protocol
mvn clean install
```
<br><br>
协议包已经开发完成，你可以参考官方开源仓库代码[hello-actuator-protocol协议包工程源码](https://github.com/TheFirstLineOfCode/hello-lithosphere-tutorials/tree/main/hello-actuator/hello-actuator-protocol)

<br><br>
### 5.3 开发服务器端插件
#### 5.3.1 服务器端插件工程
创建hello-actuator-server目录，添加pom.xml文件。
```
<?xml version="1.0" encoding="UTF-8"?>

<project xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns="http://maven.apache.org/POM/4.0.0"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
	<modelVersion>4.0.0</modelVersion>
	
	<parent>
		<groupId>com.thefirstlineofcode.sand</groupId>
		<artifactId>sand-server</artifactId>
		<version>1.0.0-BETA2</version>
	</parent>

	<groupId>com.thefirstlineofcode.lithosphere.tutorials.helloactuator</groupId>
	<artifactId>hello-actuator-server</artifactId>
	<version>0.0.1-RELEASE</version>
	<name>Hello actuator server plugin</name>

	<dependencies>
		<dependency>
			<groupId>com.thefirstlineofcode.sand.server</groupId>
			<artifactId>sand-server-things</artifactId>
		</dependency>
		<dependency>
			<groupId>com.thefirstlineofcode.lithosphere.tutorials.helloactuator</groupId>
			<artifactId>hello-actuator-protocol</artifactId>
			<version>0.0.1-RELEASE</version>
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
>**代码说明**
>* parent设置为com.thefirstlineofcode.sand:sand-server，可引用parent POM里的以来配置管理。
><br><br>
>* 依赖com.thefirstlineofcode.sand.server:sand-server-things库，因为我们要使用这个库里的接口IThingProvider和IThingRegistrationCustomizer来实现相关功能。
>* 依赖hello-actuator-protocol协议包。

<br><br>
#### 5.3.2 实现IThingProvider
```
@Extension
public class ThingModelsProvider implements IThingModelsProvider {

	@Override
	public IThingModelDescriptor[] provide() {
		return new IThingModelDescriptor[] {
			new HatModelDescriptor()
		};
	}

}
```
>**代码说明**
>* 实现IThingProvider接口，在接口方法里，我们将前面开发的HatModelDescripotor登记到服务器中。
><br><br>
>* 请注意@Extension标注，它申明了这个类是PF4J的插件扩展。

<br><br>
#### 5.3.2 实现IThingRegistrationCustomizer
```
@Extension
public class ThingRegistrationCustomizer extends AbstractThingRegistrationCustomizer {
	private static final String HARD_CODED_REGISTRATION_CODE = "abcdefghijkl";
	
	@Override
	public boolean isUnregisteredThing(String thingId, String registrationCode) {
		if (!super.isUnregisteredThing(thingId, registrationCode))
			return false;
		
		return HARD_CODED_REGISTRATION_CODE.equals(registrationCode);
	}
}
```
