# Welcom to Lithosphere

## Lithosphere是什么？
Lithosphere是基于XMPP协议的IoT开发平台。<br><br>
Lithosphere平台的目标，是提供全栈的IoT解决方案，开发者可以基于Lithosphere平台技术，开发复杂、灵活的IoT应用。<br><br>
Lithosphere 作为IoT应用的解决方案，主要有以下的特征：
* **全栈IoT开发框架**<br> 
Lithosphere提供全栈的IoT解决方案，包括IoT服务器，LoT局域网网关，到MCU硬件板通讯库，以及移动端开发框架。<br><br>
Lithosphere采用统一的架构技术和通讯协议，开发者不再需要整合多种开发技术来开发IoT应用，这使得IoT应用的开发变得简单。<br><br>
* **插件架构（Plugin-Architecture）**<br>
Lithosphere的核心子项目Chalk，Granite，Sand等，都基于插件架构构建。采用Lithosphere开发的IoT应用一般会具备以下这些特征：
  * 高度模块化
  * 扩展性良好
  * 部署灵活
  <br><br>
* **高效的通讯协议**<br>
标准XMPP协议使用XML来表达协议消息包，这使得它具备灵活和扩展性强特点的同时，也备受通讯协议冗余和低效的指责。<br><br>
一家公司在解决XMPP效率问题上做出了很好的示范。WhatsApp公司使用二进制的XMPP变种，为全球超过20亿用户提供IM服务。<br><br>
是的，Lithosphere平台也使用二进制XMPP来解决通讯协议效率问题。
<br><br>
* **基于IoT概念组件编程**<br>
Lithosphere提供了一组屏蔽了底层通讯细节，封装良好的IoT组件来提高开发效率。<br><br>
我们可以使用Actuator，Sensor，Concentrator，Gateway，Webcam等IoT概念组件来做开发，而不需要去研究XMPP、LoRa、WebRTC等具体技术的底层实现细节。
<br><br>
## Lithosphere由以下子项目构成：
### [Granite](https://github.com/TheFirstLineOfCode/granite)<br>
Granite是一个基于Java开发的XMMP Server。Granite XMPP Server具有以下特征：
* 标准兼容
* 高度模块化
* 高可用性和高扩展性
* 易于扩展和集成  
Granite基于微内核架构（插件架构），这使得它非常灵活和易于扩展。
<br><br>
### [Chalk](https://github.com/TheFirstLineOfCode/chalk)<br>
Chalk是Java XMPP客户端通讯库，可以用于开发Java桌面和Android的XMPP客户端。Chalk基于插件架构设计，这使得它易于使用及易于扩展。
<br><br>
### [Basalt](https://github.com/TheFirstLineOfCode/basalt)<br>
Basalt是XMPP的Java解析库。Basalt基于OXM（Object-XMPP Mapping）概念，提供XMPP协议消息包和协议对象（Protocol Object）之间的解析转换功能。
<br><br>
### [Sand](https://github.com/TheFirstLineOfCode/sand)<br>
Sand项目提供一组封装良好的IoT插件。这些IoT插件基于Chalk技术（客户端插件）和Granite技术（服务器端插件）开发。包括：
* Actuator<br>
执行器组件。关于执行器，可以参考概念里的[Actuator](./Concepts.md#Actuator)章节内容。
<br><br>
* Sensor<br>
传感器组件。关于传感器，可以参考概念里的[Sensor](./Concepts.md#Sensor)章节内容。
<br><br>
* Edge Thing<br>
边缘设备组件。Lithosphere里的边缘设备组件（Edge Thing）连接到Granite XMPP Server上后，会自动申请注册。一般来说，Gateway，Concentrator等组件，都是Edge Thing。
<br><br>
* LoRa Gateway<br>
封装LoRa协议的网关组件。
<br><br>
* Remoting<br>
远程控制插件。用于在移动App或桌面客户端里做IoT设备的远程控制。
<br><br>
* Operator<br>
运维人员使用的运维功能插件。例如，在运维App里，用于实现授权设备入网，修改客户权限等功能。
<br><br>
* Webcam<br>
基于WebRTC技术的实时监控摄像头组件。
<br><br>
### [Mud](https://github.com/TheFirstLineOfCode/mud)<br>
MCU板通讯库。支持BXMPP，和TUXP协议族Notification、
Execution、Report等协议。使用Mud库，可以为MCU板添加IoT通讯能力。
<br><br>
## 快速入门
如果你并不熟悉XMPP，插件架构，IoT，那你可以通过阅读<br>
[**概念**](./Concepts.md)<br>
来熟悉相关概念 。<br><br>
如果你是第一次接触使用Lithosphere，建议跟随<br>
[**Hello Lithosphere教程**](./Hello_Lithosphere_Tutorials.md)<br>
来学习和了解Lithosphere平台的使用。
