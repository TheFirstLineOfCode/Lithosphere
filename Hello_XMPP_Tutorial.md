## Hello, XMPP!
让我们开始Lithosphere IoT平台的学习！<br><br>
在这第一篇教程中，我们来了解Lithosphere IoT平台提供的一些基础技术设施。通过这篇教程的学习，我们会初步接触了解Granite XMPP Server和Chalk客户端XMPP库。<br><br>
我们编写一个简单的应用，来熟悉Chalk和Granite的使用，学习如何通过编写插件来扩展Lithosphere平台功能。<br><br>

### 1 前置条件：
**Java >= 11**<br>
**Granite Lite Mini XMPP Server**

点击这里下载[Granite Lite Mini XMPP Server](https://github.com/TheFirstLineOfCode/granite/releases/download/1.0.3-RELEASE/granite-lite-mini-1.0.3-RELEASE.zip)

### 2 检查Granite Lite Mini XMPP Server
#### 2.1 解压granite-lite-mini-server
```
unzip granite-lite-mini-1.0.3-RELEASE.zip
```
#### 2.2 启动Granite Lite XMPP Server
##### 2.2.1 配置Domain
根据XMPP规范要求，每个XMPP Server必须指定Domain。<br>
用户可以在${GRANITE_LITE_SERVER_HOME}/configuration/server.ini文件中配置Domain。<br><br>
在server.ini文件中，找到domain.name的配置行
```
domain.name=localhost
```
这一行修改为你服务器的域名：
```
domain.name=${MY_XMPP_SERVER_DOMAIN_NAME}
```

如果你是在局域网内启动Granite XMPP Server，请使用你当前电脑的IP作为域名。假设你的电脑IP是192.168.1.80，修改后的配置应为：
```
domain.name=192.168.1.80
```
#### 2.2.2 启动并检查Granite Lite XMPP Server的状态
启动Granite Lite XMPP Server
```
cd granite-lite-mini-1.0.3
java -jar granite-server-1.0.3-RELEASE.jar -console
```
带-console参数启动Granite Lite XMPP Serverw之后，能够看到Granite Server Console的界面。
![](https://github.com/TheFirstLineOfCode/Lithosphere/blob/main/granite_server_console.png)
我们可以在Console输入services命令来检查Granite XMPP Server的状态。
```
$services

```
如果能看到所有的services的状态都是available，说明granite lite server已经被正常的启动了。
[](https://github.com/TheFirstLineOfCode/Lithosphere/blob/main/granite_server_console_services.png)
可以用plugins命令，来检查可用的plugins。
```
$plugins
```
![](https://github.com/TheFirstLineOfCode/Lithosphere/blob/main/granite_server_console_plugins.png)
我们会看到，当前的服务器为最小部署版本，部署了最基本的5个插件：
* granite-lite-auth
* granite-lite-dba
* granite-lite-pipeline
* granite-lite-session
* granite-stream-standard

我们用exit指令，可以退出Granite Server Console，并关闭Granite XMPP Server。
```
$exit
```
![](https://github.com/TheFirstLineOfCode/Lithosphere/blob/main/granite_server_console_exit.png)

#### 2.3 编写第一个插件