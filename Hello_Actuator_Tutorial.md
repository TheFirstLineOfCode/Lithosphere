## Hello, Actuator!!!
欢迎进入IoT的世界！！！让我们来学习如何让硬件板上的LED闪起来！<br><br>
在这第一篇教程中，让我们使用一块树莓派硬件板连接LED灯。然后，我们使用Lithosphere IoT平台提供的Actuator插件，只需要做简单的开发，我们就可以用App遥控IoT硬件板设备来控制LED灯了<br><br>

## 1 前置条件：
**Java >= 11**<br>
**Granite Lite IoT XMPP Server**<br>
点击这里下载[Granite Lite IoT XMPP Server](https://github.com/TheFirstLineOfCode/granite/releases/download/1.0.3-RELEASE/granite-lite-iot-1.0.3-RELEASE.zip)<br>
**Raspberry Pi Zero W硬件板**<br>
**LED灯模块**<br>
**几颗杜邦线**<br>
![](https://dongger-s-img-repo.oss-cn-shenzhen.aliyuncs.com/images/hello_actuator_hardwares.jpg)

## 2 安装和配置Raspberry Pi
树莓派Zero W，我在淘宝上买153元RMB，记住要买一个SD卡来插上。<br><br>
我们需要做的第一件事，是给树莓派装上系统。<br><br>
在没有监视器和键盘的情况下，给树莓派装上OS。在Raspberry Pi的官方的文档里，把这叫做Headless Setting Up。可以参考下面的链接，了解如何做Headless Setting Up。<br>
[官方Headless Setting Up文档](https://www.raspberrypi.com/documentation/computers/configuration.html#setting-up-a-headless-raspberry-pi)
<br><br>
如果你不想阅读英文，可以看XDongger这篇关于树莓派安装和配置的文章。<br>
[安装和配置树莓派](https://juejin.cn/post/7247024731443101755)<br><br>
