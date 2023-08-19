## Hello, Lithosphere!!!
按照这个行业的惯例，我们总是应该从Hello, XXX!开始。<br><br>
Hello, Lithosphere是专门为新手准备的一系列教程。<br><br>
教程一共6篇，第一篇教程讲XMPP和插件架构，其余每一篇讲一个IoT开发中的主题。我们在教程中学习如何在Lithosphere IoT平台上，实现这个主题的相关功能。教程包含文章及软硬件示例。<br><br>
按照顺序阅读和学习后，会对Lithosphere IoT平台有一个直观、大概的了解。<br><br>
这6篇教程，分别是：<br><br>
[**Hello, XMPP!**](./Hello_XMPP_Tutorial.md)<br>
Lithosphere IoT平台主要使用XMPP协议和插件架构技术。<br><br>
在这篇教程里，让我们来熟悉一下Granite XMPP Server，和Chalk客户端XMPP库，以及学习如何编写XMPP协议插件。<br><br>
这篇教程不会直接涉及到IoT的开发，但是通过这篇教程，可以熟悉XMPP协议和插件架构等基础概念。这样在后续的教程中，我们学习使用IoT相关插件来开发时，会更加容易理解在背后发生了什么。<br><br>
如果你不想了解Lithosphere IoT平台的技术细节，而只是想尽快的用手机App控制硬件板上的LED，让灯闪起来。那么，你可以先跳过这篇教程，进入下一篇教程。<br><br>
对于大部分常规的IoT应用开发来说，并不需要开发新的XMPP扩展协议，也不需要去开发客户端和服务器端插件，直接使用Lithosphere IoT平台提供的IoT组件，使用Lithosphere IoT开发好的、内置的IoT通讯和数据协议就够用了。<br><br>
当然，如果你想完全的掌控Lithosphere平台，能够应对任意复杂的IoT应用，那么建议阅读这篇教程。<br><br>
你也可以再学习完其它几篇教程后，再回来学习这篇讲Lithosphere IoT平台基础技术的教程。
<br><br>
[**Hello, Actuator!**](./Hello_Actuator_Tutorial.md)<br>
让我们走进IoT的世界！<br><br>
在这篇教程里，我们用一块Raspberry PI Zero W的板子，通过GPIO连接一个LED灯。<br><br>
然后，我们使用Actuator插件，用App来控制这个IoT设备亮灯和闪灯。<br><br>
在这篇教程里，我们还会涉及到IoT设备注册的相关内容。<br><br>
[**Hello, LoRa!**](./Hello_LoRa_Tutorial.md)<br>
IoT系统里，怎么能不出现IoT通讯协议呢？<br><br>
在这篇教程里，我们来学习如何组网使用LoRa协议的通讯设备。<br><br>
我们使用一个Raspberry Pi 3A+的树莓派板来做LoRa通讯网关。当然，我们使用平台提供的LoRa Gateway插件来简化网关的开发。<br><br>
我们用一块Arduino Micro硬件板来连接LED灯。并使用Mud通讯库来让这个IoT终端通过安全授权后加入LoRa网络。<br><br>
同样，在最后，我们使用App来遥控这个使用LoRa协议的终端节点。<br><br>
[**Hello, Sensor!**](./Hello_Sensor_Tutorial.md)<br>
Sensor是IoT应用里最常见的设备。<br><br>
在这篇教程里，我们使用一块Arduino Micro板，通过GPIO连接到一个温度传感器模块。<br><br>
我们通过使用Lithosphere平台自带的Report协议，通过LoRa网关，定时上报温度数据到服务器。<br><br>
最后，我们做一个事，我们在服务器端将温度数据推送到App端，这样，我们就可以在App上，查看实时温度了。<br><br>
在这个范例中，我们主要是使用了Sensor插件。<br><br>
[**Hello, WebRTC**](./Hello_XMPP_Tutorial.md)<br>
IoT应用中，常见的一个需求是实时视频监控。<br><br>
在这篇教程中，我们使用一个Raspberry Pi 3B的边缘设备，给它接上一个USB摄像头。<br><br>
通过使用Webcam插件，我们就可以在App上随时打开这个Webcam设备，查看实时视频流了<br><br>
[**Hello, Friends**](./Hello_Friends_Tutorial.md)<br>
在更复杂的IoT应用中，我们需要让IoT设备之间互相进行通讯。<br><br>
Lithosphere平台为此提供了Firends协议，一个IoT设备可以follow它的Friend设备的事件（Event）。<br><br>
在IoT LAN内，我们可以配置这个Follow为LAN Follow。LAN Follow监听到的事件，会被网关直接在LAN内进行转发，而不需要通Server端绕一圈来进行转发。<br><br>
无疑，LAN Follow是一种常见的边缘设备的使用模式。<br><br>
在这篇教程里，我们学习如何配置和使用Firends协议。