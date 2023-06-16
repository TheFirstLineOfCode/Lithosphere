# Hello, XMPP!
让我们开始Lithosphere IoT平台的学习！<br><br>
在这第一篇教程中，我们来了解Lithosphere IoT平台提供的一些基础技术设施。通过这篇教程的学习，我们会初步接触了解Granite XMPP Server和Chalk客户端XMPP库。<br><br>
我们编写一个简单的应用，来熟悉Chalk和Granite的使用，学习如何通过编写插件来扩展Lithosphere平台功能。<br><br>

## 1 前置条件：
**Java >= 11**<br>
**Granite Lite Mini XMPP Server**

点击这里下载[Granite Lite Mini XMPP Server](https://github.com/TheFirstLineOfCode/granite/releases/download/1.0.3-RELEASE/granite-lite-mini-1.0.3-RELEASE.zip)

## 2 检查Granite Lite Mini XMPP Server
### 2.1 解压granite-lite-mini-server
```
unzip granite-lite-mini-1.0.3-RELEASE.zip
```
### 2.2 启动Granite Lite XMPP Server
#### 2.2.1 配置Domain
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
![](http://120.78.128.21:8080/files/granite_server_console.png)
<img src="http://120.78.128.21:8080/files/granite_server_console.png">
我们可以在Console输入services命令来检查Granite XMPP Server的状态。
```
$services

```
如果能看到所有的services的状态都是available，说明granite lite server已经被正常的启动了。
![](http://120.78.128.21:8080/files/granite_server_console_services.png)
可以用plugins命令，来检查可用的plugins。
```
$plugins
```
![](http://120.78.128.21:8080/files/granite_server_console_plugins.png)
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
![](http://120.78.128.21:8080/files/granite_server_console_exit.png)

## 3 编写第一个插件
XMPP协议基于典型的C/S架构模式，客户端需要一个服务器上的账号，才能登录到服务器进行通讯。<br><br>
如何在Granite XMPP Server上创建一个用户呢？<br><br>
Granite XMPP Server完全基于插件架构。我们可以通过编写插件，扩展Granite Server Console的功能，来帮助实现创建用户的任务。
### 3.1 创建hello-xmpp-server工程
我们创建一个maven工程。创建hello-xmpp-server目录，在目录下，建立pom.xml文件。<br><br>
pom.xml的内容如下：
```
<?xml version="1.0" encoding="UTF-8"?>

<project xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns="http://maven.apache.org/POM/4.0.0"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
	<modelVersion>4.0.0</modelVersion>
	
	<parent>
		<groupId>com.thefirstlineofcode.granite</groupId>
		<artifactId>com.thefirstlineofcode.granite</artifactId>
		<version>1.0.3-RELEASE</version>
	</parent>

	<groupId>com.thefirstlineofcode.lithosphere.tutorials.helloxmpp</groupId>
	<artifactId>hello-xmpp-server</artifactId>
	<version>0.0.1-RELEASE</version>
	<name>Hello XMPP server plugin</name>

	<dependencies>
		<dependency>
			<groupId>com.thefirstlineofcode.granite.framework</groupId>
			<artifactId>granite-framework-core</artifactId>
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

>**代码说明**<br>
>* 我们引用com.thefirstlineofcode.granite:com.thefirstlineofcode.granite作为parent POM，这样可以简化我们pom文件的build配置，例如插件配置和依赖库版本配置。
>>```
>><parent>
>>	<groupId>com.thefirstlineofcode.granite</groupId>
>>	<artifactId>com.thefirstlineofcode.granite</artifactId>
>>	<version>1.0.3-RELEASE</version>
>></parent>
>>```
><br><br>
>* 目前，Lithosphere的开源库，仅被部署在TheFirstLineOfCode的私有的maven服务器上。为了构建时能够正确找到开源依赖库，需要配置com.thefirstlineofcode.releases的repository。
>>```
>><repositories>
>>	<repository>
>>	<id>com.thefirstlineofcode.releases</id>
>>	<name>TheFirstLineOfCode Repository - Releases</name>
>>	<url>http://120.25.166.188:9090/repository/maven-releases</url>
>>	</repository>
>></repositories>
>>```
><br><br>
>* hello-xmpp-server插件依赖com.thefirstlineofcode.granite.framework:granite-framework-core包，因为我们需要用到granite framework库里的ICommandProcessor扩展点来扩展granite server console功能。
>>```
>><dependencies>
>>	<dependency>
>>		<groupId>com.thefirstlineofcode.granite.framework</groupId>
>>		<artifactId>granite-framework-core</artifactId>
>>	</dependency>
>></dependencies>
>>```
>>注意：这里我们并不需要指定granite-framework-core包的版本号，因为依赖版本号在parent POM中，已经被定义。

### 3.2 编写插件的代码
我们创建一个名为HelloXmppCommandsProcessor的类，它继承AbstractCommandsProcessor类。
```
package com.thefirstlineofcode.lithosphere.tutorials.helloxmpp;

import org.pf4j.Extension;
import com.thefirstlineofcode.granite.framework.core.annotations.BeanDependency;
import com.thefirstlineofcode.granite.framework.core.auth.IAccountManager;
import com.thefirstlineofcode.granite.framework.core.console.AbstractCommandsProcessor;
import com.thefirstlineofcode.granite.framework.core.console.IConsoleSystem;

@Extension
public class HelloXmppCommandsProcessor extends AbstractCommandsProcessor {
	private static final String COMMAND_GROUP_SAND_DEMO = "hello-xmpp";
	private static final String COMMANDS_GROUP_INTRODUCTION = "Commands for hello XMPP tutorial.";
	private static final String USER_NAME = "geologist";
	private static final String USER_PASSWORD = "I like lithosphere.";
	
	@BeanDependency
	private IAccountManager accountManager;

	@Override
	public String getGroup() {
		return COMMAND_GROUP_SAND_DEMO;
	}

	@Override
	public String[] getCommands() {
		return new String[] {"help", "create-test-user"};
	}

	@Override
	public String getIntroduction() {
		return COMMANDS_GROUP_INTRODUCTION;
	}

	@Override
	public void printHelp(IConsoleSystem consoleSystem) {
		consoleSystem.printTitleLine(String.format("%s Available commands:", getIntroduction()));
		consoleSystem.printContentLine("hello-xmpp help - Display the help information for hello XMPP tutorial command group.");
		consoleSystem.printContentLine("hello-xmpp create-test-user - Create a test user for hello XMPP tutorial.");
	}
	
	public void processCreateTestUser(IConsoleSystem consoleSystem) {
		if (accountManager.exists(USER_NAME)) {
			consoleSystem.printMessageLine("Test user for hello XMPP tutorial has already existed in system. Ignore to execute the command.");			
		} else {
			accountManager.add(USER_NAME, USER_PASSWORD);
			
			consoleSystem.printMessageLine("The test user for hello XMPP tutorial has been created.");
		}
	}
}
```

>**代码说明**<br>
>* AbstractCommandsProcessor是实现ICommandsProcessor接口的一个抽象基类。当我们实现ICommandsProcessor接口时，一般会继承这个抽象基类，以简化代码的编写。
><br><br>
>* 我们看到在HelloXmppCommandsProcessor类的类名定义行上，有一个@Extension标注。
>>```
>>@Extension
>>public class HelloXmppCommandsProcessor extends AbstractCommandsProcessor {
>>```
>>这里的@Extension标注来自pf4j插件框架，表示HelloXmppCommandsProcessor是一个插件扩展。我们在Granite项目中，使用了pf4j插件框架来为XMPP Server提供插件管理功能。<br><br>
>>关于pf4j插件框架d的更多信息，可以参考[pf4j官方网站](https://pf4j.org/)<br><br>
>>在这里，我们只要简单的理解，@Extension表示HelloXmppCommandsProcessor是一个ICommandProcessor的扩展，这个扩展会给Granite Server Console贡献功能。
><br><br>
>* 我们看到打了@BeanDependency标注的一个IAccountManager实例引用。
>>```
>> @BeanDependency
>> private IAccountManager accountManager;
>>```
>> @BeanDependency标注来自Granite Framework，表达对Spring Bean的引用。<br><br>
>> Granite无缝集成了Spring Framework。我们可以使用Granite Framework提供的@BeanDependency标注来引用Spring的Bean。<br><br>
>>在这里，@BeanDependency的作用和spring framework里的@Resource标注类似。
