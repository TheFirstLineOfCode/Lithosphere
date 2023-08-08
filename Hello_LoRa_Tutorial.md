## Hello, LoRa!!!
欢迎来到IoT的世界。让我们来学习和了解IoT世界里最精彩和有趣的部分。<br><br>
在这篇教程里，我们会搭建和部署一个经典的、完整的IoT应用网络。<br><br>
我们搭建一个LoRa网络。<br><br>
我们使用Mud库来帮助MCU硬件板接入到LoRa网络中。<br><br>
在LoRa网关端，我们使用LoRa Gateway插件来简化网关的开发。LoRa Gateway会给终端MCU硬件板动态分配地址（LoRa DAC插件），并把它添加为下级节点（Concentrator插件）。<br><br>
最后，我们依然通过手机App，来遥控MCU硬件板上的LED，亮灯、熄灯、闪灯。

<br><br>
## 1 前置条件：
**Java >= 11**<br>
**Granite Lite IoT XMPP Server**<br>
点击这里下载[Granite Lite IoT XMPP Server](https://github.com/TheFirstLineOfCode/granite/releases/download/1.0.4-RELEASE/granite-lite-iot-1.0.4-RELEASE.zip)<br>
**Raspberry Pi 3A+硬件板**<br>
**Arduino Micro硬件板**<br>
**LED模块**<br>
**LoRa模块**<br>
**一把杜邦线**<br><br>
下图是这个教程中使用到的硬件。
![](https://dongger-s-img-repo.oss-cn-shenzhen.aliyuncs.com/images/hello_lora_hardwares.jpg)

### ***提示***
在学习这篇教程之前，我们假设读者已经具备以下的基础知识：
* 了解IoT（物联网）相关基础概念。
* 熟悉Linux技术，会安装树莓派操作系统。
* 知道GPIO是什么，了解相关基础知识。
* 熟悉Java语言，有C语言基础。
* 了解掌握的Arduino IDE的基本使用。

在这篇教程里，我们不会再去花时间讲解以上的基础知识。<br><br>
建议在学习这篇教程之前，先学习[Hello, Actuator教程](./Hello_Actuator_Tutorial.md)。<br><br>
如果你不了解Arduino IDE的使用，可以读官方网站这篇[Getting Started with Arduino IDE 2](https://docs.arduino.cc/software/ide-v2/tutorials/getting-started-ide-v2)。

<br><br>
## 2 概念
### 2.1 UART
UART是Universal Asynchronous Receiver/Transmitter。<br><br>
中文名通用异步收发器协议。<br><br>
UART是一种硬件串口通讯协议，常被用于连接嵌入式硬件板和外部模块之间的通讯。<br><br>
当使用UART在两个设备之间进行通讯时，须将设备1的TX接到设备2的RX，将设备1的RX接到设备2的TX。如下图。<br><br>
![](https://dongger-s-img-repo.oss-cn-shenzhen.aliyuncs.com/images/uart_two_way_communication.png)

<br><br>
### 2.2 Concentrator
在IoT系统中，集线器（Concentrator）是指可以添加和管理下级IoT设备的中心节点设备。

<br><br>
### 2.3 LoRa DAC
LoRa DAC是是LoRa Dynamic Address Configuration的缩写。<br><br>
中文为LoRa动态地址配置协议。<br><br>
在同一个LoRa网络中，如果想要精确控制每一个LoRa中终端节点，需要给每个终端节点配置不同的LoRa地址。<br><br>
这和TCP/IP网络很像，在本地局域网内，需要给每个主机节点配置不同的IP地址。<br><br>
地址配置，有静态配置和动态配置两种方法，在TCP/IP网络中，我们使用DHCP服务来动态分配IP地址，简化网络的配置操作。<br><br>
在LoRa网络中，目前还缺少动态分配地址的相关协议。<br><br>
Lithsphere为解决这个问题，提供了LoRa DAC协议。并内置实现了LoRa DAC协议的LoRa DAC Client和LoRa DAC Service的组件。

<br><br>
### 2.4 LoRa网关
#### 2.4.1 为何需要LoRa网关
LoRa协议并不兼容TCP/IP。<br><br>
所以使用LoRa协议的IoT终端节点，并不能直接连接到互联网中。<br><br>
为了让LoRa终端节点连接到互联网，我们需要使用LoRa网关设备。<br><br>
LoRa网关设备，在IoT网络这一端，使用LoRa协议，连接LoRa终端节点。<br><br>
在互联网端，LoRa网关通过WiFi或者运营商网络（4G、5G，....）、连接到互联网云端服务器。<br><br>
在Lithospere里，LoRa Gateway插件，使用Concentrator插件管理下级LoRa终端节点，提供互联网连通服务。<br><br>
LoRa Gateway插件还使用LoRa DAC插件提供的LoRa DAC服务，来分配和管理LoRa终端节点地址。

<br><br>
#### 2.4.2 LoRa网关工作模式
LoRa网关有两种工作模式，DAC模式和ROUTER模式。<br><br>
这是因为Lithospere中的LoRa网关插件，被设计成了可执行成多功能任务的设备，它具备以下两个功能：
* ROUTER功能<br>
LoRa网关，在使用ROUTER工作模式时，可以帮助LoRa网络中的终端节点，连接到互联网服务器。在这个工作模式下，LoRa网关实际上起到了一个连接LoRa网络和Internet网络的路由器作用。
* DAC功能<br>
LoRa网关，在使用DAC工作模式时，它其实是一个辅助组网LoRa网络的工具。<br><br>
使用LoRa协议组网时，一个比较啰嗦的事情，是如何给每个终端节点，分配一个在LoRa网络里唯一、不冲突的LoRa地址。<br><br>
当然，我们可以给LoRa模块静态配置地址，这个就需要严格做设备和地址的分配管理，以避免LoRa组网地址冲突。<br><br>
在工厂里组装LoRa协议智能设备时，还必须有一个给LoRa终端设备配置静态地址的流程，将被管理的不冲突、唯一地址，配置到LoRa协议智能设备中。<br><br>
另外一种方案是在组网时，给终端设备动态分配地址，这类似于在TCP/IP网络里使用DHCP协议。问题在于，LoRa网络协议，目前没有动态地址分配相关的协议。<br><br>
Lithosphere提供了LoRa DAC协议，来帮助简化LoRa组网过程。当LoRa网关处于DAC工作模式下时，它被用于配置和组网LoRa网络。<br><br>
Lithosphere目前的版本，这两种工作模式，不能同时并发使用。<br><br>
我们在本教程案例中，会使用到ChangeWorkingMode指令，来遥控LoRa网关切换工作模式，来跑通整个案例。


<br><br>
### 2.5 TUXP
TUXP是Things Unified and Extensiable Protocol。<br><br>
中文为智能物件统一和可扩展协议。<br><br>
为何会有TUXP？<br><br>
这是因为，在IoT应用中，我们通常会面对多端和多协议带来的复杂性。<br><br>
在IoT应用中，终端部署地点和部署环境，较传统互联网和移动互联网来说，更为复杂和多变。<br><br>
而且在IoT应用架构上，由于引入了IoT专用通讯协议，往往需要部署IoT网关节点来帮助连接到互联网的通讯。<br><br>
较传统互联网应用来说，IoT应用引入了更多端（移动端、服务器端、IoT网关端、终端节点端）和更多协议（BlueTooth，WiFi，LoRa，ZigBee，JSON, XML，MQTT，....）。<br><br>
为了解决IoT应用多端和多协议带来的复杂性问题，Lithosphere平台采用了TUXP。<br><br>
TUXP是一组面向IoT通讯的XMPP扩展协议。<br><br>
目前，TUXP包括以下一些子协议：
* IBTR Protocol
物件在线注册协议
* Notification Protocol<br>
物件事件通知协议。
* Execution Protocol<br>
远程控制执行协议。
* Report Protocol<br>
传感器数据上报协议。
* LoRa DAC<br>
LoRa动态地址分配协议。
* Friends Protocol<br>
友物件事件监听协议
* ... ...

<br><br>
Lithosphere在IoT应用中，统一使用TUXP协议来简化多端和多协议问题。<br><br>
基于Lithosphere来做IoT应用开发，其它所有IoT相关协议都被屏蔽掉了，我们只使用TUXP。<br><br>
No LoRa，No JSON，No XML，No MQTT，No HTTP，... ...<br><br>
Only TUXP。

<br><br>
### 2.6 BXMPP
让我们来聊聊Lithosphere平台里锐利的宝剑，BXMPP！

<br><br>
#### 2.6.1 为何需要BXMPP？<br><br>
一个最简单的回答，当然是：<br><br>
**优化通讯效率**<br><br>
让我们再深入一些分析这个问题。<br><br>
开发IoT应用，就会涉及到多端、多协议问题。<br><br>
* IoT专用通讯协议，需要二进制通讯协议。<br>
一些IoT专用通讯协议，因为省电、低能耗的需求，被设计为低速率。<br><br>
对于JSON，XML来说，这些低速率的通讯协议，速度太慢了！在这些IoT专用网络里，JSON，XML格式数据包太大了！<br><br>
我们需要二进制协议来改善优化网络通讯。<br><br>
* 终端设备环境，需要二进制通讯协议。<br>
IoT的世界，千变万化！IoT里的T - Thing，数量多如泥沙！<br><br>
因为部署环境、功耗、以及成本的考虑，我们常在应用中使用低成本、廉价的MCU解决方案。<br><br>
MCU硬件板的性能、资源，往往有限。许多MCU的板子，甚至不能运行JSON、XML解析器。<br><br>
就算使用性能更好的高端MCU板，有限的资源，也不应该消耗在JSON、XML的解析上。<br><br>
二进制通讯协议，很明显是更好的解决方案。
<br><br>
在本教程中，我们使用Arduino Micro来作为终端设备硬件板，开发教程的例子。它的内存SRAM容量为2.5K。<br><br>
本教程案例，也在Arduino UNO R3上测试通过，这是一款廉价10元板，我在淘宝上购买它的价格是15元，它的内存SRAM容量为2k。

<br><br>
#### 2.6.2 BXMPP究竟是什么？
在IoT开发中，使用XMPP协议，可能会遭到最大的质疑是，基于XML的BXMPP，通讯效率太差了，这个协议不适合用于需要高效通讯的IoT应用。<br><br>
解决方案是什么？其实有一个公司，已经为我们做出了最佳示范。<br><br>
这个公司是WhatsApp。它使用叫名为FunXMPP的XMPP二进制变种协议，来支撑全球20亿IM用户的使用。<br><br>
Lithosphre也提供了XMPP的二进制变种协议，来改善通讯效率问题。这个二进制BXMPP变种协议被命名为BXMPP。<br><br>

#### 2.6.2 BXMPP通讯效率如何？
我们直接来看使用BXMPP的实际效果。<br><br>
在LoRa网络中，LoRa终端设备通过LoRa网关注册到服务器。<br><br>
在这个注册过程中，LoRa终端上报Thing ID（13字节）和Registration Code（12字节）；LoRa网关给LoRa终端动态分配地址并设置数据上报通道；最后网关发消息给终端设备确认注册成功。<br><br>
整个过程LoRa终端设备和LoRa网关总共有4次通讯，全部通讯过程，总计交换94字节数据，完成整个终端设备注册过程。<br><br>
**所以，请放心使用Lithosphere平台开发IoT应用，不用担心通讯效率问题。**

<br><br>
## 3 开发协议包
我们总是先要开发协议包。<br><br>
本教程中开发协议包过程，和[Hello, Actuator教程开发协议包](./Hello_Actuator_Tutorial.md#开发协议包)中类似。<br><br>
只有些细节区别。
<br><br>
### 3.1 创建协议工程
创建hello-lora-protocol工程，pom.xml如下：
```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns="http://maven.apache.org/POM/4.0.0"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">

    ... ...

	<groupId>com.thefirstlineofcode.lithosphere.tutorials.hellolora</groupId>
	<artifactId>hello-lora-protocol</artifactId>
	<name>Hello LoRa protocol</name>
	<version>0.0.1-RELEASE</version>
	
    ... ...
	
</project>
```

> **代码说明**
>* 除了协议包的groupId和artifactId不一样之外，其它内容和[Hello, Actuator教程创建协议工程](./Hello_Actuator_Tutorial.md#创建协议工程)中的pom.xml完全相同。不再赘述。

<br><br>
### 3.2 定义协议对象
和[Hello, Actuator教程定义协议对象](./Hello_Actuator_Tutorial.md#定义协议对象)中完全相同。不再赘述。

<br><br>
### 3.3 定义Model Descriptor
在这个教程案例中，我们有2种型号智能物件需要登记到服务器：
* LoRa智能网关
* 可控终端LED小灯

<br><br>
以下代码定义可控终端LED小灯类型描述器。
```
public class HltModelDescriptor extends SimpleThingModelDescriptor {
	public static final String MODEL_NAME = "HLT";
	public static final String DESCRIPTION = "Hello LoRa thing";
	
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
>* 和[Hello, Actuator教程定义Model Descriptor](./Hello_Actuator_Tutorial.md#定义Model Descriptor)类似，只是设备型号名改为了HLT（Hello LoRa Thing）。不再赘述。

<br><br>
以下代码定义LoRa网关类型描述器。
```
public class HlgModelDescriptor extends SimpleThingModelDescriptor {
	public static final String MODEL_NAME = "HLG";
	public static final String DESCRIPTION = "Hello LoRa Gateway";
	
	public HlgModelDescriptor() {
		super(MODEL_NAME, DESCRIPTION, true, null, null, createSupportedActions());
	}
	
	private static Map<Protocol, Class<?>> createSupportedActions() {
		Map<Protocol, Class<?>> supportedActions = new HashMap<>();
		supportedActions.put(ChangeWorkingMode.PROTOCOL, ChangeWorkingMode.class);
		
		return supportedActions;
	}
}
```
> **代码说明**
>* 设备型号名为HLG（Hello LoRa Gateway）。
>* LoRa网关，只支持一个Action，ChangeWorkingMode，用来遥控LoRa网关切换工作模式。
>* 不需要在协议包中定义ChangeWorkingMode对象。因为LoRa Gateway网关插件中已经内置了ChangeWorkingMode协议对象，我们可以使用系统自带的这个协议对象。

### 3.4 构建安装协议包
因为需要在后续开发客户端和服务器端插件包时，引用协议包，所以我们在hello-lora-protocol工程里，执行构建安装指令，把协议包安装到本地maven仓库。
```
cd hello-lora-protocol
mvn clean install
```
<br><br>
协议包已经开发完成，你可以参考官方开源仓库代码[hello-lora-protocol协议包工程源码](https://github.com/TheFirstLineOfCode/hello-lithosphere-tutorials/tree/main/hello-lora/hello-lora-protocol)

<br><br>
## 4 开发服务器插件
### 4.1 服务器端插件工程
创建hello-lora-server目录，添加pom.xml文件。
```
<?xml version="1.0" encoding="UTF-8"?>

<project xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns="http://maven.apache.org/POM/4.0.0"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
	<modelVersion>4.0.0</modelVersion>
	
    ... ...

	<groupId>com.thefirstlineofcode.lithosphere.tutorials.hellolora</groupId>
	<artifactId>hello-lora-server</artifactId>
	<version>0.0.1-RELEASE</version>
	<name>Hello LoRa server plugin</name>

	... ...
</project>
```
> **代码说明**
>* 除了服务器插件包的groupId和artifactId不一样之外，其它内容和[Hello, Actuator教程服务器端插件工程](./Hello_Actuator_Tutorial.md#服务器端插件工程)中的pom.xml完全相同。不再赘述。

<br><br>
### 4.2 实现IThingProvider
```
@Extension
public class ThingModelsProvider implements IThingModelsProvider {

	@Override
	public IThingModelDescriptor[] provide() {
		return new IThingModelDescriptor[] {
			new HlgModelDescriptor(),
			new HltModelDescriptor()
		};
	}

}
```
>**代码说明**
>* 注册HLG和HLT设备到服务器。
>>>```
>>>public IThingModelDescriptor[] provide() {
>>>  return new IThingModelDescriptor[] {
>>>         new HlgModelDescriptor(),
>>>         new HltModelDescriptor()
>>>     };
>>>}
>>>```

<br><br>
### 4.3 实现IThingRegistrationCustomizer
```
@Extension
public class ThingRegistrationCustomizer extends ThingRegistrationCustomizerAdapter {
	private static final String HARD_CODED_REGISTRATION_CODE = "abcdefghijkl";
	
	@Override
	public boolean isUnregisteredThing(String thingId, String registrationCode) {
		if (!super.isUnregisteredThing(thingId, registrationCode))
			return false;
		
		return HARD_CODED_REGISTRATION_CODE.equals(registrationCode);
	}
	
	@Override
	public boolean isAuthorizationRequired() {
		return false;
	}
	
	@Override
	public boolean isConfirmationRequired() {
		return false;
	}
}
```
>**代码说明**
>* 和Hello, Actuator教程案例里的ThingRegistrationCustomizer相比，只多了一个isConfirmationRequired方法，返回false值。
>>>```
>>>public boolean isConfirmationRequired() {
>>>		return false;
>>>}
>>>```
>>>我们在Hello，Actuator教程里，提到了设备注册的概念。<br><br>
>>>我们在sAuthorizationRequired()方法中返回false，禁止掉了Edge设备（直连Internet服务器的设备）的人工授权步骤。禁止人工授权后，设备仅需要通过Thing ID和Registration Code合法性检查，就会自动注册到服务器<br><br>
>>>对于非直连Internet服务器的设备，例如在这个案例中的HLT（Hello LoRa Thing）设备，它是通过LoRa网关来连接Internet服务器的。则需要在isConfirmationRequired()方法中返回false，来禁止掉了此类设备的人工授权步骤。<br><br>
>* 同样的，我们检查设备的Registration Code必须为"abcdefghijkl"。

### 4.4 编写插件配置文件
在src/main/resources目录下，创建plugin.properties。
```
plugin.id=hello-lora-server
plugin.provider=TheFirstLineOfCode
plugin.version=0.0.1-RELEASE
plugin.dependencies=sand-server-things
non-plugin.dependencies=sand-protocols-lora-gateway,hello-lora-protocol
```
>**代码说明**
>* 和[Hello, Actuator教程服务器编写插件配置文件](./Hello_Actuator_Tutorial.md#编写插件配置文件)类似，仅有细节区别，不再赘述。

<br><br>
### 4.5 构建部署服务器端插件
构建hello-lora-server插件包
```
cd hello-lora-server
mvn clean package
```
<br><br>
将hello-lora-server插件包和它依赖的hello-lora-protocol包，把这两个jar包，copy到服务器的plugins目录下。
```
cp hello-lora-protocol/target/hello-lora-protocol-0.0.1-RELEASE.jar granite-lite-iot-1.0.4-RELEASE/plugins

cp hello-lora-server/target/hello-lora-server-0.0.1-RELEASE.jar granite-lite-iot-1.0.4-RELEASE/plugins
```
<br><br>
服务器端插件已经开发完成，你可以参考官方开源仓库代码[hello-lora-server服务器端插件包工程源码](https://github.com/TheFirstLineOfCode/hello-lithosphere-tutorials/tree/main/hello-lora/hello-lora-server)

<br><br>
### 7.6 其它服务器端工作
#### 7.6.1 检查Granite Lite XMPP Server状态
和[Hello, Actuator教程检查Granite Lite XMPP Server状态](./Hello_Actuator_Tutorial.md#检查Granite Lite XMPP Server状态)类似，不再赘述。

<br><br>
#### 7.6.2 创建测试用户
和[Hello, Actuator教程创建测试用户](./Hello_Actuator_Tutorial.md#创建测试用户)类似，不再赘述。

<br><br>
## 5 连接LoRa网关硬件
让我们来做些硬件组装的工作。<br><br>

我们来看看LoRa网关用到的LoRa模块长啥样。<br><br>
![](https://dongger-s-img-repo.oss-cn-shenzhen.aliyuncs.com/images/as32_ttl_100.jpg)
<br><br>
这个LoRa模块型号AS32-TTL-100，在淘宝上买，零售价还挺贵的，30块钱一个。头上黑黑的、粗粗的东西，是信号天线，5块钱一个。
<br><br>
我们可以看到，它有7个引脚：
| 引脚名        | 作用 |
| --------- | ---------- |
| GND       | 地线 |
| VCC       | 5V电源 |
| MD0       | 模块配置引脚0 |
| MD1       | 模块配置引脚1 |
| AUX       | 模块状态指示引脚 |
| RXD       | UART串口通讯RXD引脚，接硬件板上TX引脚 |
| TXD       | UART串口通讯TXD引脚，接硬件板上RX引脚 |

<br><br>
再来看树莓派硬件板这一端。<br><br>
![](https://dongger-s-img-repo.oss-cn-shenzhen.aliyuncs.com/images/connect_lora_module_to_raspbery_pi_using_gpio.png.png)
<br><br>
LoRa模块上的MD0、MD1控制引脚，以及AUX状态指示引脚，可以接到树莓派任意的有效GPIO引脚上。在本教程案例里，我们把这3个引脚接到了树莓派板的GPIO_02，GPIO_03，GPIO_04引脚上。<br><br>
树莓派硬件板接上LoRa模块之后，看上去是这样的。<br><br>
![](https://dongger-s-img-repo.oss-cn-shenzhen.aliyuncs.com/images/lora_module_connected_to_raspberry_pi.jpg)

<br><br>
## 6 配置树莓派环境
安装了纯净RaspBerry Pi OS操作系统的树莓派，还需要做一些环境配置。

<br><br>
### 6.1 安装JDK
接上电源，启动树莓派。<br><br>
使用ssh登录到树莓派上，然后安装默认的JDK。树莓派OS默认安装的JDK版本是Open JDK11。
```
ssh pi@192.168.1.180
sudo apt-get update
sudo apt-get install default-jdk
```

<br><br>
### 6.2 安装pigpio库
Pi4J通过采用JNI技术访问C库，来提供GPIO，Serial口通讯等功能。<br><br>
从V2版本开始，Pi4J放弃了使用WiringPi库，改用pigpio库作为控制GPIO的底层C库。<br><br>
我们需要先安装pigpio库，才能让Pi4J正常工作。<br><br>
按照以下的步骤安装pigpio库。

<br><br>
### 6.3 配置UART

#### 6.3.1 启用UART
编辑/boot/config.txt文件。
```
sudo vi /boot/config.txt
```
<br><br>
在文件结尾添加以下的两行内容。
```
enable_uart=1
dtoverlay=disable-bt
```
**说明：**
* Raspberry Pi板上有两种类型的UART，PL011和mini UART。其中PL011是全功能的UART；mini UART是一个缩减了功能的版本。<br>
在本教程中使用的Raspberry Pi 3A+版本硬件板，默认将PL011 UART被配置给了蓝牙使用。<br>
我们想使用PL011 UART来连接LoRa模块，需要调整树莓派的默认配置，我们需要关掉蓝牙，让它把PL011 UART让出来用。所以，需要以下的这行配置，关掉蓝牙。
>>>```
>>>dtoverlay=disable-bt
>>>```

<br><br>
#### 6.3.2 移除蓝牙相关系统服务
我们已经在配置中，关掉蓝牙了。但可以再彻底一些，我们移除掉蓝牙相关的系统服务。
```
sudo systemctl disable hciuart
sudo systemctl disable bluetooth
```

#### 6.3.3 禁止UART串口控制台输出
编辑/boot/cmdline.txt文件
```
sudo vi /boot/cmdline.txt
```
将console==serial0,xxxx的内容删除掉，这里的xxxx是配置的波特率。<br><br>
在我的树莓派3A+板上，需要被移除的内容为console=serial0,115200。

<br><br>
## 6 开发LoRa网关程序
让我们来开发LoRa网关程序。

<br><br>
### 6.1 网关端工程
创建hello-lora-gateway目录，添加pom.xml文件。
```
<?xml version="1.0" encoding="UTF-8"?>

<project xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns="http://maven.apache.org/POM/4.0.0"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
	<modelVersion>4.0.0</modelVersion>
	
	<parent>
		<groupId>com.thefirstlineofcode.sand</groupId>
		<artifactId>sand-client</artifactId>
		<version>1.0.0-BETA3</version>
	</parent>

	<groupId>com.thefirstlineofcode.lithosphere.tutorials.hellolora</groupId>
	<artifactId>hello-lora-gateway</artifactId>
	<version>0.0.1-RELEASE</version>
	<name>Hello LoRa Gateway</name>

	<dependencies>
		<dependency>
			<groupId>com.thefirstlineofcode.sand.protocols</groupId>
			<artifactId>sand-protocols-bxmpp-extensions</artifactId>
		</dependency>
		<dependency>
			<groupId>com.thefirstlineofcode.chalk</groupId>
			<artifactId>chalk-logger</artifactId>
		</dependency>
		<dependency>
			<groupId>com.thefirstlineofcode.sand.client</groupId>
			<artifactId>sand-client-edge</artifactId>
		</dependency>
		<dependency>
			<groupId>com.thefirstlineofcode.sand.client</groupId>
			<artifactId>sand-client-lora-gateway</artifactId>
		</dependency>
		<dependency>
			<groupId>com.thefirstlineofcode.sand.client.pi</groupId>
			<artifactId>sand-client-pi-ashining</artifactId>
		</dependency>
		<dependency>
			<groupId>com.thefirstlineofcode.lithosphere.tutorials.hellolora</groupId>
			<artifactId>hello-lora-protocol</artifactId>
			<version>0.0.1-RELEASE</version>
		</dependency>
	</dependencies>
	
	<build>
		<plugins>
			<plugin>
				<groupId>org.apache.maven.plugins</groupId>
				<artifactId>maven-jar-plugin</artifactId>
				<version>2.4</version>
				<configuration>
					<archive>
						<manifest>
							<addClasspath>true</addClasspath>
							<classpathPrefix>libs/</classpathPrefix>
							<mainClass>com.thefirstlineofcode.lithosphere.tutorials.hellolora.gateway.Main</mainClass>
						</manifest>
					</archive>
				</configuration>
			</plugin>
			<plugin>
				<artifactId>maven-assembly-plugin</artifactId>
				<version>3.0.0</version>
				<configuration>
					<appendAssemblyId>false</appendAssemblyId>
					<descriptors>
						<descriptor>src/assembly/descriptor.xml</descriptor>
					</descriptors>
				</configuration>
				<executions>
					<execution>
						<id>make-assembly</id>
						<phase>package</phase>
						<goals>
							<goal>single</goal>
						</goals>
					</execution>
				</executions>
			</plugin>
		</plugins>
	</build>

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
>* POM继承com.thefirstlineofcode.sand:sand-client，以便复用父POM里的依赖配置管理。
><br><br>
>* hello-lora-gateway是一个独立运行的Java程序。我们使用maven-assembly-plugin和maven-jar-plugin来打包和配置可这个可运行程序。
><br><br>
>* 依赖com.thefirstlineofcode.sand.client:sand-client-edge库，我们使用Edge库来帮助网关设备进行设备注册和连接服务器。<br>
>>>```
>>><dependency>
>>>	<groupId>com.thefirstlineofcode.sand.client</groupId>
>>>	<artifactId>sand-client-edge</artifactId>
>>></dependency>
>>>```
><br><br>
>* 依赖com.thefirstlineofcode.sand.client:sand-client-lora-gateway库，我们需要使用Lora Gateway插件。<br>
>>>```
>>><dependency>
>>>	<groupId>com.thefirstlineofcode.sand.client</groupId>
>>>	<artifactId>sand-client-lora-gateway</artifactId>
>>></dependency>
>>>```
><br><br>
>* 我们使用型号为AS32-TTL-100的LoRa模块。Sand项目提供了在树莓派硬件板上对这个型号LoRa模块的硬件通讯器封装，具体实现在com.thefirstlineofcode.sand.client.pi:sand-client-pi-ashining库中。这个库间接引用了Pi4J库，所以我们不需要再另行配置Pi4J库依赖。
>>>```
>>><dependency>
>>>	<groupId>com.thefirstlineofcode.sand.client.pi</groupId>
>>>	<artifactId>sand-client-pi-ashining</artifactId>
>>></dependency>
>>>```
>* 依赖hello-lora-protocol协议包。
>>>```
>>><dependency>
>>>	<groupId>com.thefirstlineofcode.lithosphere.tutorials.hellolora</groupId>
>>>	<artifactId>hello-lora-protocol</artifactId>
>>>	<version>0.0.1-RELEASE</version>
>>></dependency>
>>>```

<br><br>
### 6.2 实现HelloLoraGateway
```
public class HelloLoraGateway extends AbstractEdgeThing {
	public static final String THING_MODEL = HlgModelDescriptor.MODEL_NAME;
	public static final String SOFTWARE_VERSION = "1.0.0-BETA3";
	
	private static final String ATTRIBUTE_NAME_COMMUNICATOR_HAS_CONFIGURED = "communicator_has_configured";
	
	private ICommunicator<LoraAddress, LoraAddress, byte[]> communicator;
	private ILoraGateway loraGateway;
	
	public HelloLoraGateway() {
		super(THING_MODEL, null, true);
		
		communicator = createLoraCommunicator();
	}

	@Override
	public String getSoftwareVersion() {
		return SOFTWARE_VERSION;
	}

	@Override
	protected boolean doProcessAttributes(Map<String, String> attributes) {
		return initializeAndConfigureCommunicator(attributes);
	}
	
	private boolean initializeAndConfigureCommunicator(Map<String, String> attributes) {
		if (!communicator.isInitialized())
			communicator.initialize();
		
		if (!"true".equals(attributes.get(ATTRIBUTE_NAME_COMMUNICATOR_HAS_CONFIGURED))) {
			communicator.configure();
			
			attributes.put(ATTRIBUTE_NAME_COMMUNICATOR_HAS_CONFIGURED, "true");
			return true;
		}
		
		return false;
	}

	@Override
	protected void registerIotPlugins() {
		chatClient.register(LoraGatewayPlugin.class);
	}

	@Override
	protected void startIotComponents() {
		if (loraGateway == null) {
			loraGateway = chatClient.createApi(ILoraGateway.class);
			
			loraGateway.setDownlinkCommunicator(communicator);
			loraGateway.setUplinkCommunicators(Collections.singletonList(communicator));
			
			configureConcentrator(loraGateway);
		}	
		
		loraGateway.start();
	}
	
	private void configureConcentrator(ILoraGateway gateway) {
		IConcentrator concentrator = gateway.getConcentrator();
		
		concentrator.registerLanThingModel(new HltModelDescriptor());
		regiserExecutors(concentrator, gateway);
	}

	private ICommunicator<LoraAddress, LoraAddress, byte[]> createLoraCommunicator() {
		As32Ttl100LoraCommunicator communicator = new As32Ttl100LoraCommunicator();
		communicator.setMd0Pin(2);
		communicator.setMd1Pin(3);
		communicator.setAuxPin(4);
		
		return communicator;
	}

	private void regiserExecutors(IActuator actuator, final ILoraGateway loraGateway) {
		actuator.registerExecutor(ChangeWorkingMode.class, ChangeWorkingModeExecutor.class, loraGateway);
	}

	@Override
	protected void stopIotComponents() {
		loraGateway.stop();
	}

	@Override
	protected String loadThingId() {
		return THING_MODEL + "-" + ThingsUtils.generateRandomId(8);
	}

	@Override
	protected String loadRegistrationCode() {
		return "abcdefghijkl";
	}
}
```
>**代码说明**
>* 继承AbstractEdgeThing。我们使用sand-client-edge库来简化LoRa网关程序的编写。
><br><br>
>* As32Ttl100LoraCommunicator类将树莓派板上型号为AS32-TTL-100的LoRa Module，封装成了标准的ICommunicator接口实现。我们创建这个As32Ttl100LoraCommunicator实例，并根据我们实际的接线，设置树莓派连接到MD0，MD1，AUX的对应GPIO引脚。
>>>```
>>>private ICommunicator<LoraAddress, LoraAddress, byte[]> createLoraCommunicator() {
>>>		As32Ttl100LoraCommunicator communicator = new As32Ttl100LoraCommunicator();
>>>		communicator.setMd0Pin(2);
>>>		communicator.setMd1Pin(3);
>>>		communicator.setAuxPin(4);
>>>		
>>>		return communicator;
>>>}
>>>```
><br><br>
>* AbstractEdgeThing使用一个名为${THING_MODE_NAME}-attributes.propertis的属性配置文件来保存属性参数。并且，留下了一个doProcessAttributes()方法允许子类使用这个属性文件。我们在HelloLoraGateway中重载了这个方法，借用属性配置文件来保存我们用到的通讯器已配置（communicator_has_configured）参数。
>>>```
>>>protected boolean doProcessAttributes(Map<String, String> attributes) {
>>>		return initializeAndConfigureCommunicator(attributes);
>>>}
>>>
>>>private boolean initializeAndConfigureCommunicator(Map<String, String> attributes) {
>>>	if (!communicator.isInitialized())
>>>		communicator.initialize();
>>>
>>>	if (!"true".equals(attributes.get(ATTRIBUTE_NAME_COMMUNICATOR_HAS_CONFIGURED))) {
>>>		communicator.configure();
>>>		attributes.put(ATTRIBUTE_NAME_COMMUNICATOR_HAS_CONFIGURED, "true");
>>>		return true;
>>>	}
>>>
>>>	return false;
>>>}
>>>```
>>>我们使用Communicator之前，需要对它做initialize和configure。<br><br>
>>>communicator.initialize()方法，每次程序启动，都需要用它初始化Communicator。<br><br>
>>>communicator.configure()方法，则只需要调用一次。<br><br>
>>>这里的的逻辑是，LoRa模块提供了持久化保存配置的功能。所以，调用过一次communicator.configure()后，相关配置信息就被LoRa模块的硬件持久化保存记住了。即使所有硬件掉电重启，重启后，LoRa模块依然可以读取到已经持久化保存的配置信息。所以configure()只需要调用一次，之后不再需要反复调用。<br><br>
我们在第一次做configure时，这时候属性配置文件中，还不能读到名为communicator_has_configured的属性参数。我们调用configure()，调用结束后，设置communicator_has_configured属性参数值为"true"。<br><br>
然后，我们返回true，这个返回值表示我们修改了属性配置，需要将最新的属性列表保存到属性配置文件。<br><br>
下次再进入initializeAndConfigureCommunicator()方法，我们能读取到communicator_has_configured的属性值为"true"。我们直接退出，不再重复调用configure()。
><br><br>
>* 在HelloLoraGateway中，我们只需要使用LoRaGateway插件。
>>>```
>>>protected void registerIotPlugins() {
>>>		chatClient.register(LoraGatewayPlugin.class);
>>>}
>>>```
><br><br>
>* 在startIotComponents方法中，我们创建LoRa Gateway组件，然后对它进行配置。我们需要为LoRa Gateway设置Downlink Communicator和Uplink Communicators。其中，被设置的Uplink Communicators参数是一个ICommunicator的列表。在多频道的网关中，我们会指定多个Uplink Communicators，来提升LoRa网关的上行通讯容量。在本教程的案例中，我们只使用一块LoRa通讯芯片，这是一个单频道LoRa网关。所以我们使用Collections.singletonList(communicator)创建一个单元素的列表，然后设置到LoRa Gateway组件。
>>>```
>>>loraGateway.setDownlinkCommunicator(communicator);
>>>loraGateway.setUplinkCommunicators(Collections.singletonList(communicator));
>>>```
><br><br>
>* 我们为网关配置ChangeWorkingMode执行器，这样它才能被遥控切换Working Mode。<br><br>
注意一个细节，我们不需要自己来编写ChangeWorkingModeExecutor。LoRa Gateway插件中，已经内置提供了ChangeWorkingModeExecutor，我们直接使用它。<br><br>
这里需要注意的另一个细节是：Conconcenator组件实现了IActuator接口，我们可以用它来注册Executors。
>>>```
>>>... ...
>>>IConcentrator concentrator = gateway.getConcentrator();
>>>... ...
>>>regiserExecutors(concentrator, gateway);
>>>... ...
>>>private void regiserExecutors(IActuator actuator, final ILoraGateway loraGateway) {
>>>	actuator.registerExecutor(ChangeWorkingMode.class, ChangeWorkingModeExecutor.class, loraGateway);
>>>}
>>>```
>* 我们需要登记HltModeDescriptor到网关。我们在前面的教程里，提到过需要将Mode Descriptor登记到服务器。Granite服务器完全基于插件架构，当我们通过开发插件来提供新的IoT设备功能，这时需要登记设备相关信息到服务器，服务器才能知道如何处理新IoT设备的相关功能。<br><br>
我们在网关端使用Chalk库开发网关设备。同样的，Chalk也完全基于插件架构。所以，我们需要将终端设备的Model Descriptor注册到网关中，网关才能知道如何处理设备的相关通讯功能。我们使用IConcentrator的registerLanThingModel()API来做这个事。我们从LoRa Gateway中拿到Concentrator组件，并登记HLT（Hello LoRa Thing）设备到Concentrator。
>>>```
>>>IConcentrator concentrator = gateway.getConcentrator();
>>>concentrator.registerLanThingModel(new HltModelDescriptor());
>>>```

<br><br>
### 6.3 入口主程序
编写一个简单的Main程序来启动运行LoRa网关。
```
... ...
public static void main(String[] args) {
	new Main().run(args);
}

private void run(String[] args) {
... ...
	LogConfigurator.configure(HlgModelDescriptor.MODEL_NAME, getLogLevel(logLevel));
... ...
	gateway.start();
... ...
}
```
>**代码说明**
>* 我们使用chalk-logger库来配置logger。因为Hello, LoRa教程的例子相较其它几篇教程，涉及到更多端和更多协议。我们配置logger方便调试和跟踪。
>>>```
>>>LogConfigurator.configure(HlgModelDescriptor.MODEL_NAME, getLogLevel(logLevel));
>>>```
>>>LogConfigurator.configure()方法的两个参数是日志文件名，和配置的LogLevel等级。<br><br>
以上代码将日志文件保存为${USER_HOME}/.com.thefirstlineofcode.chalk/logs/${MODE_NAME}.yyyy-MM-DD.log。

<br><br>
### 6.4 BXMPP
让我们来配置使用BXMPP。<br><br>
考虑到LoRa网络的低速率，以及Arduio低端硬件板的资源处理能力，我们需要在LoRa网络里，使用二进制的BXMPP协议。<br><br>
#### 6.4.1 引入sand-protocols-bxmpp-extensions库
sand项目内置的IoT通讯相关协议，都已经配置好。我们只要引入相关BXMPP协议扩展库就Ok了。<br><br>
在项目的pom.xml文件中，我们引入com.thefirstlineofcode.sand.protocols:sand-protocols-bxmpp-extensions依赖库。
```
... ...
<dependency>
	<groupId>com.thefirstlineofcode.sand.protocols</groupId>
	<artifactId>sand-protocols-bxmpp-extensions</artifactId>
</dependency>
... ...
```

#### 6.4.2 配置BXMPP应用协议扩展
我们在hello-lora-protocol协议包中，引入了应用专用的Flash、TurnOn、TurnOff协议对象。我们需要为它们配置BXMPP协议扩展。<br><br>
二进制XMPP的一个关键思路，就是采用替换法，将把XMPP里的长长的字符串常量，都替换成单字节的替换字节。<br><br>
例如，Flash协议对象，当repeat属性被设置为5时，会被OXM框架翻译成以下的XMPP文档：<br>
```
<flash xmlns="urn:leps:things:simple-light" repeat=5/>
```
我们能够看到，flash、xmlns、urn:leps:things:simple-light、repeat这几个字符串都很长，而且它们都是字符串常量。<br><br>
在通讯时，我们可以把这些字符串，都换成单字节的字节替换符，以优化通讯的效率。<br><br>
具体怎么做呢？<br><br>
我们在src/main/resources目录下，创建/META-INF/iot-lan-bxmpp-extensions.txt文件，在这个文件里，我们列出需要引入的BXMPP协议扩展文件。<br><br>
在本教程案例里，我们只需要在iot-lan-bxmpp-extensions.txt文件里，引入hello-lora-protocol协议包的BXMPP协议扩展配置文件。<br><br>
iot-lan-bxmpp-extensions.txt文件内容如下：
```
hello-lora-bxmpp-extension.properties
```
我们在src/main/resources目录下，创建hello-lora-bxmpp-extension.properties文件，在这文件里，我们配置hello-lora-protocol协议的BXMPP替换规则。
```
0xf70x01=urn:leps:things:simple-light
0x00=flash
0x01=repeat
0x02=turn-on
0x03=turn-off
<br><br>
```
**代码说明**
* XMPP协议使用xmlns来扩展协议，不同的xmlns，表示不同的协议。<br><br>
在hello-lora-protocol中，flash协议的local name是flash，它的的xmlns被定义为"urn:leps:things:simple-light"。<br><br>
所以XMPP版本的flash协议，完整的协议名为urn:leps:things:simple-light | flash<br><br>
根据hello-lora-bxmpp-extension.properties的配置，flash协议在它的BXMPP版本中，协议名被替换成3字节替换符0xf70x010x00。<br><br>
* 属性名repeat，被替换成了1字节替换符0x01。
* turn-on和turn-off协议，被翻译成BXMPP协议时，规则同flash协议类似，不再赘述。

<br><br>
### 6.5 测试和注册LoRa网关
我们来测试一下网关程序，启动LoRa网关主程序，将网关设备注册到服务器。<br><br>
将树莓派接通电源启动起来，然后将编译好的LoRa网关程序，用scp拷贝到树莓派上。
```
cd hello-lora-gateway
mvn clean package
scp target/hello-lora-gateway-0.0.1-RELEASE.tar.gz pi@192.168.1.180:/home/pi
```
**注：**
* 192.168.1.180是树莓派板的网络地址。请改为你配置树莓派板时，指定的静态IP地址。
* pi为树莓派用户。请改为你配置树莓派时，初始化创建的用户名。

<br><br>
启动网关程序，检查它是否能够正确注册到服务器。
```
ssh pi@192.168.1.180
tar -xzvf hello-lora-gateway-0.0.1-RELEASE.tar.gz
cd hello-lora-gateway-0.0.1-RELEASE
sudo java -jar hello-lora-gateway-0.0.1-RELEASE.jar --host=192.168.1.80
```
**注：**
* 在启动网关程序之前，先启动Granite XMPP Lite IoT Server。
* 这里，我们需要使用sudo来启动网关程序。这是由于Pi4J库的使用限制。官方文档的说法：Need to run as sudo。下图来自Pi4J官方文档。<br><br>
![](https://dongger-s-img-repo.oss-cn-shenzhen.aliyuncs.com/images/must_run_as_sudo.png)

<br><br>
如果能够看到Thing thing has started，说明网关程序已经成功注册，并连接到服务器。<br><br>
![](https://dongger-s-img-repo.oss-cn-shenzhen.aliyuncs.com/images/lora_gateway_has_started.png)

### 6.4 用App控制网关切换工作模式
我们还是用sand-demo App来遥控LoRa网关。
<br><br>
点击这里下载构建好的[sand-demo App](https://github.com/TheFirstLineOfCode/sand/releases/download/1.0.0-BETA3/sand-demo.apk)。
<br><br>
使用sand-demo用户登录到sand-demo App里后，我们可以看到型号为HLG的一个设备，它是前面注册成功的LoRa网关设备。<br><br>
![](https://dongger-s-img-repo.oss-cn-shenzhen.aliyuncs.com/images/change_lora_gateway_working_mode.jpg)
<br><br>
可以看到它有唯一一个控制菜单项，Change Working Mode。<br><br>
点击菜单改变LoRa网关工作模式，在LoRa网关程序的控制台里，我们可以看到切换网关Working Mode的日志输出。<br><br>
![](https://dongger-s-img-repo.oss-cn-shenzhen.aliyuncs.com/images/change_lora_gateway_working_mode_to_dac.png)

<br><br>
## 7 连接LoRa终端设备硬件
让我们来实现最后一块拼图 - 终端设备。<br><br>
先把硬件组装起来。终端硬件板上要接上LoRa模块和LoRa网关通讯，它还需要接上LED用于执行亮灯、熄灯、闪灯指令<br><br>
我们使用一块Arduino Micro的硬件板子。<br><br>
![](https://dongger-s-img-repo.oss-cn-shenzhen.aliyuncs.com/images/arduino_micro.jpg)
<br><br>
这个硬件板有点贵的，在淘宝上，带USB接口线的版本，零售价47元。<br><br>
为何我们不使用更便宜的板子？<br><br>
这个教程里的终端程序，在Arduino Uno R3的板子上测试可以正常运行。Arduino Uno R3板在淘宝上的价格，15元。<br><br>
问题在于，Arduino Uno R3板，不能在Arduino IDE中使用串口监视器打印调试信息。这是因为这个板子的USB接口线使用了UART串口。那么，因为我们使用的LoRa模块也要使用UART串口，这会产生冲突，除非我们愿意放弃在IDE中打印调试信息，<br><br>
Arduino Nano板，也和Arduino Uno R3板有同样的问题。似乎Arduino的10元板，都有这个问题，在外接设备使用UART串口时，无法同时使用Arduino IDE的的串口监视器来打印调试信息。<br><br>
好吧，为了方便打印调试信息，强烈建议使用Arduino Micro来开发本教程中的终端设备案例。<br><br>
来接线吧，让我们来看看如何将LoRa模块和LED灯接到Arduino Micro板上。<br><br>
![](https://dongger-s-img-repo.oss-cn-shenzhen.aliyuncs.com/images/connect_lora_module_and_led_to_arduino_micro.png)
<br><br>
接好之后，看上去是这样的。<br><br>
![](https://dongger-s-img-repo.oss-cn-shenzhen.aliyuncs.com/images/all_modules_connected_to_arduino_micro.jpg)

<br><br>
## 8 开发LoRa终端程序
让我们开始来写终端程序吧。

<br><br>
### 8.1 安装Arduino IDE
我们使用Arduino IDE来编写和安装hello_lora_thing硬件板程序。<br><br>
如果你的开发机器上，并没有安装Arduino IDE，请从官方网站[下载Arduino IDE 2安装](https://docs.arduino.cc/software/ide-v2)。<br><br>
如果你以往并没有使用Arduino IDE的相关经验，请阅读官方文档[Getting Started with Arduino IDE 2](https://docs.arduino.cc/software/ide-v2/tutorials/getting-started-ide-v2)，稍微熟悉一下这个开发工具。

### 8.2 安装mud和设备适配库
在开发终端程序之前，我们先安装要使用到的用到的依赖库。<br><br>
终端程序依赖以下第三方库：
* mud<br>
Lithosphere平台Mud通讯库。<br><br>
点击这里下载[Mud通讯库](https://github.com/TheFirstLineOfCode/mud/releases/download/1.0.0-BETA2/mud.zip)

* mud_arduino_avr<br>
Arduino AVR硬件板的Mud适配库。<br><br>
点击这里下载[Arduino AVR硬件板的Mud适配库](https://github.com/TheFirstLineOfCode/mud/releases/download/1.0.0-BETA2/mud_arduino_avr.zip)

* mud_as32_ttl_100<br>
AS32-TTL-100型号LoRa模块的Mud适配库。<br><br>
点击这里下载[AS32-TTL-100型号LoRa模块的Mud适配库](https://github.com/TheFirstLineOfCode/mud/releases/download/1.0.0-BETA2/mud_as32_ttl_100.zip)

* arduino_unique_id_generator<br>
Arduino硬件板唯一ID生成器。<br><br>
点击这里下载[Arduino硬件板唯一ID生成器](https://github.com/TheFirstLineOfCode/mud/releases/download/1.0.0-BETA2/arduino_unique_id_generator.zip)

* mud_configuration<br>
Arduino硬件板的Mud适配配置文件<br><br>
https://github.com/TheFirstLineOfCode/mud/releases/download/1.0.0-BETA2/mud_configuration.zip

如果你不知道如何导入一个zip格式Library，那么可以阅读官方文档[Import a .zip Library](https://docs.arduino.cc/software/ide-v1/tutorials/installing-libraries#importing-a-zip-library)

### 8.3 安装ArduinoUniqueID库
Mud提供的arduino_unique_id_generator库，依赖ArduinoUniqueID库提供的Arduino硬件板唯一ID读取功能。<br><br>
ArduinoUniqueID库可以直接从Arduino IDE的Library Manager里安装。<br><br>
打开Library Manager，输入ArduinoUniqueID关键字查询，找到ArduinoUniqueID库后，安装它。<br><br>
![](https://dongger-s-img-repo.oss-cn-shenzhen.aliyuncs.com/images/install_arduino_unique_id_library.png)

<br><br>
### 8.4 编写hello-lora-thing代码
创建hello-lora-thing目录，然后在这个目录下，创建hello-lora-thing.ino文件，代码如下。
```
#include <Arduino_BuiltIn.h>

#include <thing.h>
#include <mcu_board_adaptation.h>
#include <radio_module_adaptation.h>
#include <arduino_unique_id_generator.h>

#define MODEL_NAME "HLT"
#define LED_PIN 12

void setup() {
  configureMcuBoard(MODEL_NAME);
  configureRadioModule();
  
  registerThingIdLoader(loadThingId);
  registerRegistrationCodeLoader(loadRegistrationCode);

  registerThingProtocolsConfigurer(configureThingProtocolsImpl);
  
  pinMode(LED_PIN, OUTPUT);
  
  toBeAThing();
}

char *loadThingId() {
  return generateThingIdUsingUniqueIdLibrary(MODEL_NAME);
}

char *loadRegistrationCode() {
  return "abcdefghijkl";
}

void turnLedOn() {
  digitalWrite(LED_PIN, HIGH);
}

void turnLedOff() {
  digitalWrite(LED_PIN, LOW);
}

void flashLed() {
  digitalWrite(LED_PIN, HIGH);
  delay(200);
  digitalWrite(LED_PIN, LOW);
}

int8_t processFlash(Protocol *protocol) {
  int repeat;
  if (!getIntAttributeValue(protocol, 0x01, &repeat)) {
    repeat = 1;
  }

  if (repeat <= 0 || repeat > 8)
    return -1;

  for (int i = 0; i < repeat; i++) {
    flashLed();
    delay(500);
  }

  return 0;
}

int8_t processTurnOn(Protocol *protocol) {
  turnLedOn();
  return 0;
}

int8_t processTurnOff(Protocol *protocol) {
  turnLedOff();
  return 0;
}

void configureThingProtocolsImpl() {
  ProtocolName pnFlash = {0xf7, 0x01, 0x00};
  registerActionProtocol(pnFlash, processFlash, false);

  ProtocolName pnTurnOn = {0xf7, 0x01, 0x02};
  registerActionProtocol(pnTurnOn, processTurnOn, false);

  ProtocolName pnTurnOff = {0xf7, 0x01, 0x03};
  registerActionProtocol(pnTurnOff, processTurnOff, false);
}

void loop() {
  int result = doWorksAThingShouldDo();
  if (result != 0) {
#ifdef ENABLE_DEBUG
    Serial.print(F("Error occurred when the thing does the works it should do. Error number: "));
    Serial.print(result);
    Serial.println(F("."));    
#endif*
  }
}
```
> **代码说明**
>* 引入mud通讯库、Arduino AVR板mud适配库、AS32-TTL-100 LoRa模块适配库、Arduino硬件板唯一ID生成器库。
>>>```
>>>#include <thing.h>
>>>#include <mcu_board_adaptation.h>
>>>#include <radio_module_adaptation.h>
>>>#include <arduino_unique_id_generator.h>
>>>```
><br><br>
>* 定义常量MODEL_NAME和LED_PIN。LED控制引针在教程中，被连在了GPIO12引针上。如果你接LED的控制的引针不是GPIO12，那请根据自己的接线修改LED_PIN的值。
>>>```
>>>#define MODEL_NAME "HLT"
>>>#define LED_PIN 12
>>>```
><br><br>
>* 使用mud适配库，配置Arduino AVR硬件板和LoRa模块。
>>>```
>>>configureMcuBoard(MODEL_NAME);
>>>configureRadioModule();
>>>```
>* 设备注册时，需要提供Thing ID和Registration Code值。我们注册loadThingId和loadRegistrationCode两个函数指针，使用它们来获取Thing ID和Registration Code值。
>>>```
>>>registerThingIdLoader(loadThingId);
>>>registerRegistrationCodeLoader(loadRegistrationCode);
>>>... ...
>>>char *loadThingId() {
>>>  return generateThingIdUsingUniqueIdLibrary(MODEL_NAME);
>>>}
>>>
>>>char *loadRegistrationCode() {
>>>  return "abcdefghijkl";
>>>}
>>>```
>>>**注：**
>>>* arduino_unique_id_generator库的API函数generateThingIdUsingUniqueIdLibrary()读取Arduino硬件板的唯一ID，组合MODEL_NAME来生成一个Thing ID。
>>>* loadRegistrationCode()直接返回硬编码"abcdefghijkl"字符串。还记得我们在服务器端会检查这个硬编码的Registration Code吗？
><br><br>
>* 我们在configureThingProtocolsImpl()里配置Action处理函数，我们需要将configureThingProtocolsImpl()函数注册为协议配置器。
>>>```
>>>registerThingProtocolsConfigurer(configureThingProtocolsImpl);
>>>... ...
>>>void configureThingProtocolsImpl() {
>>>  ProtocolName pnFlash = {0xf7, 0x01, 0x00};
>>>  registerActionProtocol(pnFlash, processFlash, false);
>>>
>>>  ProtocolName pnTurnOn = {0xf7, 0x01, 0x02};
>>>  registerActionProtocol(pnTurnOn, processTurnOn, false);
>>>
>>>  ProtocolName pnTurnOff = {0xf7, 0x01, 0x03};
>>>  registerActionProtocol(pnTurnOff, processTurnOff, false);
>>>}
>>>```
>>>**注：**
* 我们在协议配置器里，登记三个Action协议Flash、TurnOn，TurnOff对应的处理函数。<br><br>
注意这里3字节的BXMPP协议名定义，它来自我们在前面定义的BXMPP协议扩展配置文件[hello-lora-bxmpp-extension.properties](#配置BXMPP应用协议扩展)。
><br><br>
>* 还是遵循责任分离原则，我们来看看怎么做硬件控制。使用Arduino的API设置LED_PIN的PIN Mode。然后，我们在turnOn()，turnOff()，flashLED()函数里，通过LED_PIN引针控制LED灯。
>>>```
>>>... ...
>>>pinMode(LED_PIN, OUTPUT);
>>>... ...
>>>void turnLedOn() {
>>>	digitalWrite(LED_PIN, HIGH);
>>>}
>>>
>>>void turnLedOff() {
>>>	digitalWrite(LED_PIN, LOW);
>>>}
>>>
>>>void flashLed() {
>>>	digitalWrite(LED_PIN, HIGH);
>>>	delay(200);
>>>	digitalWrite(LED_PIN, LOW);
>>>}
>>>```
>>>**注：**
>>>pinMode()、delay()、digitalWrite()都是Arduino的开发API。Arduino开发比较简单，在本教程中不再赘述，请自行学习。<br><br>
>>>关于Arduino的开发，可以参考Arduino官方文档[Arduino程序语言参考](https://www.arduino.cc/reference/en/)。
><br><br>
>* 现在来处理IoT通讯逻辑，我们使用mud库来简化IoT的通讯。
>>>```
>>>... ...
>>>toBeAThing();
>>>... ...
>>>int result = doWorksAThingShouldDo();
>>>... ...
>>>int8_t processFlash(Protocol *protocol) {
>>>  int repeat;
>>>  if (!getIntAttributeValue(protocol, 0x01, &repeat)) {
>>>    repeat = 1;
>>>  }
>>>
>>>  if (repeat <= 0 || repeat > 8)
>>>    return -1;
>>>
>>>  for (int i = 0; i < repeat; i++) {
>>>    flashLed();
>>>    delay(500);
>>>  }
>>>
>>>  return 0;
>>>}
>>>```
>>>**注：**
>>>* 在setup()里，调用toBeAThing()，让设备变身智能物件。在loop()里，调用doWorksAThingShouldDo()，做智能物件该做的事。魔法就在藏在这两个函数中，它们会搞定一切。<br><br>
>>>* 我们在processFlash()里，处理收到的Flash指令。函数会收到带Protocol数据结构的参数的回调。我们调用getIntAttributeValue()来获取repeat参数。并做响应逻辑处理。<br><br>
>>>* 我们获取repeat参数值时，使用了参数名的替换字符0x01。它来自我们在前面定义的BXMPP协议扩展配置文件[hello-lora-bxmpp-extension.properties](#配置BXMPP应用协议扩展)。<br><br>
>>>* 在指令处理函数里，如果有错误，返回0之外的返回值，表示这个错误的错误码。返回0表示指令已被正常执行。
>>>* trunOn()和turnOff()函数的处理更简单，不再赘述。<br><br>

### 8.5 上传程序到终端硬件板
用Arduino IDE将程序上传到Arduino Micro硬件板。<br><br>
如果你不知道怎么做，请参考官方文档[How to upload a sketch with the Arduino IDE 2](https://docs.arduino.cc/software/ide-v2/tutorials/getting-started/ide-v2-uploading-a-sketch)。

<br><br>
### 8.6 测试终端程序
现在可以来测试终端设备了。

<br><br>
#### 8.6.1 前置准备
将Granite Lite IoT XMPP Server启动起来。<br><br>
然后，登录到树莓派硬件板，将LoRa网关程序启动起来。<br><br>

<br><br>
#### 8.6.2 注册终端设备
登录sand-demo App程序。<br><br>
如果还没有安装sand-demo App，请点击这里下载安装使用[sand-demo App](https://github.com/TheFirstLineOfCode/sand/releases/download/1.0.0-BETA3/sand-demo.apk)。<br><br>

使用LoRa网关设备的Changing Working Mode菜单，修改网关设备的工作模式为DAC。在DAC工作模式下，终端设备才能通过网关注册到服务器。切换工作模式后，应该能够看到网关控制台有"DAC service has started."信息输出。<br><br>
![](https://dongger-s-img-repo.oss-cn-shenzhen.aliyuncs.com/images/lora_gateway_has_changed_its_working_mode_to_dac.png)

在Arduino IDE里，打开Serial Monitor。<br><br>
接通Arduino Micro硬件板，会看到一些终端硬件板协商地址，申请节点添加的调试信息。<br><br>
如果能看到"I'm a thing now!"信息出现在串口监视器里，终端设备已经成功注册。<br><br>
![](https://dongger-s-img-repo.oss-cn-shenzhen.aliyuncs.com/images/I_am_a_thing_now.png)

#### 控制终端硬件闪灯
如果一直sand-demo App一直处于打开状态，终端设备注册成功后，会收到添加终端接到网关成功的提示信息。<br><br>
![](https://dongger-s-img-repo.oss-cn-shenzhen.aliyuncs.com/images/node_added.jpg)<br><br>
点击"是的，刷新智能物件列表"按钮，刷新设备列表。现在可以看到网关设备和终端设备，都出现在App的智能物件列表中了。<br><br>
![](https://dongger-s-img-repo.oss-cn-shenzhen.aliyuncs.com/images/both_gateway_and_thing_have_registered.jpg)<br><br>

使用LoRa网关设备的Changing Working Mode菜单，修改网关设备的工作模式为ROUTER。在ROUTER工作模式下，网关才能够转发远程控制指令。<br><br>
现在，可以使用终端设备的控制菜单，来遥控终端设备亮灯、熄灯、闪灯了。

<br><br>
## 10 当我遇到错误时
一个最简单的解决方法，能不能重来一次啊？<br><br>
可以的！

<br><br>
### 10.1 重置终端hello-lora-thing
在setup()里调用toBeAThing()之前，先调用resetThing();<br><br>
这会重置终端硬件板的状态为初始化状态，它会重新申请设备注册。<br><br>
![](https://dongger-s-img-repo.oss-cn-shenzhen.aliyuncs.com/images/arduino_ide_reset_thing.png)

<br><br>
### 10.2 重置hello-lora-gateway
登录到树莓派网关硬件板上。删除掉Edge Thing的属性配置文件。<br><br>
```
ssh pi@192.168.1.180
sudo -i
rm -rf .com.thefirstlineofcode.sand.client.edge/HLG-attributes.properties
exit
```
**注：**
* 网关程序将状态保存在属性配置文件中，删除这个文件，会重置网关程序状态。<br><br>
* Edge Thing的属性配置文件，被保存为${USER_HOME}/.com.thefirstlineofcode.sand.client.edge/${EDGE_THING_MODEL_NAME}-attributes.properties。 <br><br>
* 在删除这个文件之前，先执行sudo -i切换到root身份。还记得我们因为要使用Pi4J V2版本，必须要使用sudo指令来执行网关程序吗？

<br><br>
### 10.3 重置Granite Lite IoT XMPP Server
删除掉服务器目录下data子目录，可以重置服务器状态，如下：
```
cd granite-lite-iot-1.0.4-RELEASE
rm -rf data
```
**注：**
删除data目录，会重置服务器所有状态。<br><br>
所以重置服务器后，记得要重新执行sand-demo create-test-users，重新创建测试用户。

<br><br>
## 11 总结
通过这篇教程，我们可以学习到以下的内容：
* 使用UART串口通讯，我们可以在Linux系统中，集成使用较复杂的外部通讯模块，例如本教程中的LoRa模块。<br><br>
* IoT专用网络协议，不兼容互联网的TCP/IP。所以，我们需要使用IoT网关来帮助IoT终端连接到互联网。<br><br>
* 相较于传统互联网应用，IoT应用会涉及到更多端和更多协议，这给IoT应用带来了额外的复杂性。<br><br>
为简化这个问题，Lithosphere使用TUXP协议来屏蔽其它的IoT协议。<br><br>
* 在通讯协议层，由于我们使用TUXP和OXM技术，我们只需要定义协议对象，然后在开发API中直接使用它。我们定义的协议对象，从移动端App开始，通过服务器、IoT网关，最终直达IoT终端设备，这个过程不需要做任何协议翻译解析相关的任何工作。<br><br>
* 使用LoRa网关插件，我们用不到100行代码，开发了一个全功能的LoRa网关程序。<br><br>
* 在IoT终端，我们用95行代码，开发了LED灯终端控制程序。这其中大部分是硬件控制、协议配置、协议处理的相关代码。使用Mud库，给终端设备带来LoRa通讯能力所需要的代码数量，是4行代码。<br><br>
* 在IoT专用网络里，我们使用BXMPP协议。通过简单的配置，协议包在IoT网络内被翻译成为二进制XMPP兼容协议。<br><br>
所以，我们不需要担心通讯效率问题。
* 树莓派和Arduino，都不复杂。传统的纯软件开发人员，完全可以通过使用这些硬件平台，进入到IoT应用领域。
