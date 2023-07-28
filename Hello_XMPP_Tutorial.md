# Hello, XMPP!
让我们开始Lithosphere IoT平台的学习！<br><br>
在这第一篇教程中，我们来了解Lithosphere IoT平台提供的一些基础技术设施。通过这篇教程的学习，我们会初步接触了解Granite XMPP Server和Chalk客户端XMPP库。<br><br>
我们编写一个简单的应用，来熟悉Chalk和Granite的使用，学习如何通过编写插件来扩展Lithosphere平台功能。

<br><br>
## 1 前置条件
**Java >= 11**<br><br>
**Granite Lite Mini XMPP Server**<br>
点击这里下载[Granite Lite Mini XMPP Server](https://github.com/TheFirstLineOfCode/granite/releases/download/1.0.4-RELEASE/granite-lite-mini-1.0.4-RELEASE.zip)

<br><br>
## 2 检查Granite Lite Mini XMPP Server
### 2.1 解压Granite Lite Mini Server
```
unzip granite-lite-mini-1.0.4-RELEASE.zip
```

<br><br>
### 2.2 启动Granite Lite XMPP Server
#### 2.2.1 配置Domain Name
根据XMPP规范要求，每个XMPP Server必须指定Domain Name。<br><br>
用户可以在${GRANITE_LITE_SERVER_HOME}/configuration/server.ini文件中配置Domain Name。<br><br>
在server.ini文件中，找到domain.name的配置行
```
domain.name=localhost
```
这一行修改为你服务器的域名：
```
domain.name=${MY_XMPP_SERVER_DOMAIN_NAME}
```
<br>
如果你是在局域网内启动Granite XMPP Server，请使用你当前电脑的IP作为域名。假设你的电脑IP是192.168.1.80，修改后的配置应为：

```
domain.name=192.168.1.80
```

<br><br>
#### 2.2.2 启动并检查Granite Lite XMPP Server的状态
启动Granite Lite XMPP Server
```
cd granite-lite-mini-1.0.4
java -jar granite-server-1.0.4-RELEASE.jar -console
```
带-console参数启动Granite Lite XMPP Server之后，能够看到Granite Server Console的界面。
![](https://dongger-s-img-repo.oss-cn-shenzhen.aliyuncs.com/images/granite_server_console.png)

<br><br>
我们可以在Console输入services命令来检查Granite XMPP Server的状态。
```
$services

```
如果能看到所有的services的状态都是available，说明granite lite server已经被正常的启动了。
![](https://dongger-s-img-repo.oss-cn-shenzhen.aliyuncs.com/images/granite_server_console_services.png)
<br><br>
可以用plugins命令，来检查可用的plugins。
```
$plugins
```
![](https://dongger-s-img-repo.oss-cn-shenzhen.aliyuncs.com/images/granite_lite_mini_server_console_plugins.png)
<br><br>
我们会看到，当前的服务器为最小部署版本，部署了最基础的5个插件：
* granite-lite-auth
* granite-lite-dba
* granite-lite-pipeline
* granite-lite-session
* granite-stream-standard

<br><br>
我们用exit指令，可以退出Granite Server Console，并关闭Granite XMPP Server。
```
$exit
```
![](https://dongger-s-img-repo.oss-cn-shenzhen.aliyuncs.com/images/granite_server_console_exit.png)

<br><br>
## 3 编写第一个插件
XMPP协议基于典型的C/S架构模式，客户端需要一个服务器上的账号，才能登录到服务器进行通讯。<br><br>
初始状态的Granite XMPP Server，是没有任何用户的。<br><br>
如何在Granite XMPP Server上创建一个用户呢？<br><br>
有很多方法可以创建用户。比如，我们可以部署IBR插件。IBR插件实现XEPs协议[XEP-0077（In-Band Registration）](https://xmpp.org/extensions/xep-0077.html)，提供IM用户在线注册功能。<br><br>
在这里，我们采用一个更简单的方案。Granite XMPP Server完全基于插件架构。我们可以通过编写插件，扩展Granite Server Console的功能，来帮助实现创建用户的任务。

<br><br>
### 3.1 创建hello-xmpp-server工程
创建hello-xmpp-server目录，在目录下，建立pom.xml文件。<br><br>
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
		<version>1.0.4-RELEASE</version>
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

>**代码说明**
>* 我们引用com.thefirstlineofcode.granite:com.thefirstlineofcode.granite作为parent POM，这样可以简化我们pom文件，例如插件配置和依赖库版本配置。
>>>```
>>><parent>
>>>	<groupId>com.thefirstlineofcode.granite</groupId>
>>>	<artifactId>com.thefirstlineofcode.granite</artifactId>
>>>	<version>1.0.4-RELEASE</version>
>>></parent>
>>>```
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
><br><br>
>* hello-xmpp-server插件依赖com.thefirstlineofcode.granite.framework:granite-framework-core包，因为我们需要用到granite framework库里的ICommandProcessor扩展点来扩展granite server console功能。
>>>```
>>><dependencies>
>>>	<dependency>
>>>		<groupId>com.thefirstlineofcode.granite.framework</groupId>
>>>		<artifactId>granite-framework-core</artifactId>
>>>	</dependency>
>>></dependencies>
>>>```
>>>注意：这里我们并不需要指定granite-framework-core包的版本号，因为依赖版本号在parent POM中，已经被定义。

<br><br>
### 3.2 编写插件的代码
我们创建一个名为HelloXmppCommandsProcessor的类，它继承AbstractCommandsProcessor类。
```
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
>**代码说明**
>* AbstractCommandsProcessor是实现ICommandsProcessor接口的一个抽象基类。当我们实现ICommandsProcessor接口时，一般会继承这个抽象基类，以简化代码的编写。
><br><br>
>* 我们看到在HelloXmppCommandsProcessor类的类名定义行上，有一个@Extension标注。
>>>```
>>>@Extension
>>>public class HelloXmppCommandsProcessor extends AbstractCommandsProcessor {
>>>```
>>这里的@Extension标注来自PF4J插件框架，表示HelloXmppCommandsProcessor是一个插件扩展。我们在Granite项目中，使用了PF4J插件框架来为XMPP Server提供插件管理功能。<br>
>>关于PF4J插件框架的更多信息，可以参考[PF4J官方网站](https://pf4j.org/)<br><br>
>>在这里，我们只要简单的理解，@Extension表示HelloXmppCommandsProcessor是一个ICommandProcessor的扩展，这个扩展会给Granite Server Console贡献功能。
><br><br>
>* 我们看到打了@BeanDependency标注的一个IAccountManager实例引用。
>>>```
>>>@BeanDependency
>>>private IAccountManager accountManager;
>>>```
>>>@BeanDependency标注来自Granite Framework，表达对Spring Bean的引用。<br><br>
>>>Granite无缝集成了Spring Framework。我们可以使用Granite Framework提供的@BeanDependency标注来引用Spring的Bean。<br><br>
>>>在这里，@BeanDependency的作用和spring framework里的@Autowired标注类似。
><br><br>
>* 请注意processCreateTestUser方法的命名。这里采用了CoC（约定优于配置）的设计，当Granite Server Console收到扩展命名组的指令时，默认将create-test-user指令的请求，转发到processCreateTestUser方法进行处理。
><br><br>
>* 我们在HelloXmppCommandsProcessor类中引用的IAccountManager接口来自Granite Framework包，而它的实例实现来自granite-lite-auth插件。IAccountManager提供了添加账户的服务，我们直接使用该接口服务来添加一个账号。
>>>```
>>>accountManager.add(USER_NAME, USER_PASSWORD);
>>>```

<br><br>
### 3.3 编写插件配置文件
在src/main/resources目录下，创建plugin.properties文件，内容如下：
```
plugin.id=hello-xmpp-server
plugin.provider=TheFirstLineOfCode
plugin.version=0.0.1-RELEASE
```
>**代码说明**
>* 我们使用PF4J作为服务器端插件管理器。PF4J要求使用plugin.properties定义插件的配置属性。<br><br>
>* 配置文件最重要的属性是plugin.id，每个插件都必须有自己唯一的插件ID。

<br><br>
### 3.4 打包并部署插件
#### 3.4.1 打包插件
使用maven构建插件。
```
cd hello-xmpp-server
mvn clean package
```

<br><br>
#### 3.4.2 部署插件到Granite Lite Server
将插件包copy到服务器plugins目录下。
```
cp hello-xmpp-server/target/hello-xmpp-server-0.0.1-RELEASE.jar granite-lite-mini-1.0.4/plugins
```

<br><br>
#### 3.4.3 创建测试用户
启动Granite Lite XMPP Server
```
cd granite-lite-mini-1.0.4
java -jar granite-server-1.0.4-RELEASE.jar -console
```

<br><br>
可以用plugins命令，查看新部署的plugins。
```
$plugins
```
我们可以看到，新部署的hello-xmpp-server插件。<br><br>
在Granite Server Console中执行create-test-user命令
```
hello-xmpp create-test-user
```

<br><br>
我们可以看到，测试用户已经被创建。
![](https://dongger-s-img-repo.oss-cn-shenzhen.aliyuncs.com/images/granite_server_console_hello_xmpp_create_test_user.png)

<br><br>
## 4 测试连接服务器
有了测试用户，我们现在可以用客户端连接到服务器了。<br><br>
我们写一个简单的测试程序，来测试客户端到服务器的连通性。

<br><br>
### 4.1 创建hello-xmpp-app工程
创建hello-xmpp-app目录，添加pom.xml文件。
```
<?xml version="1.0" encoding="UTF-8"?>

<project xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns="http://maven.apache.org/POM/4.0.0"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
	<modelVersion>4.0.0</modelVersion>
	
	<parent>
		<groupId>com.thefirstlineofcode.chalk</groupId>
		<artifactId>com.thefirstlineofcode.chalk.parent</artifactId>
		<version>1.0.2-RELEASE</version>
	</parent>

	<groupId>com.thefirstlineofcode.lithosphere.tutorials.helloxmpp</groupId>
	<artifactId>hello-xmpp-app</artifactId>
	<version>0.0.1-RELEASE</version>
	<name>Hello XMPP App</name>

	<dependencies>
		<dependency>
			<groupId>com.thefirstlineofcode.chalk</groupId>
			<artifactId>chalk-core</artifactId>
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
>* 指定com.thefirstlineofcode.chalk:com.thefirstlineofcode.chalk.parent作为POM的parent，这样可以简化我们pom文件，例如插件配置和依赖库版本配置。
>>>```
>>><parent>
>>>	<groupId>com.thefirstlineofcode.chalk</groupId>
>>>	<artifactId>com.thefirstlineofcode.chalk.parent</artifactId>
>>>	<version>1.0.2-RELEASE</version>
>>></parent>
>>>```
><br><br>
>* 同样，我们配置com.thefirstlineofcode.releases仓库，使得项目能够找到chalk库依赖。
><br><br>
>* 为了连接到服务器，我们需要com.thefirstlineofcode.chalk:chalk-corey依赖包。我们不需要指定依赖的版本，因为依赖版本已经在parent的POM定义。
>>>```
>>><dependency>
>>>	<groupId>com.thefirstlineofcode.chalk</groupId>
>>>	<artifactId>chalk-core</artifactId>
>>></dependency>
>>>```

<br><br>
### 4.2 编写Java代码
```
public class Main {
	private static final String USER_NAME = "geologist";
	private static final String USER_PASSWORD = "I like lithosphere.";
	
	public static void main(String[] args) {
		StandardStreamConfig streamConfig = new StandardStreamConfig("192.168.1.80", 5222);
		streamConfig.setResource("my_notebook");
		
		IChatClient chatClient = new StandardChatClient(streamConfig);
		
		try {
			chatClient.connect(new UsernamePasswordToken(USER_NAME, USER_PASSWORD));
			System.out.println("Chat client has connected.");
		} catch (ConnectionException e) {
			throw new RuntimeException("Can't connect to server.", e);
		} catch (AuthFailureException e) {
			throw new RuntimeException("No such user???", e);
		}
		
		if (chatClient != null && chatClient.isConnected()) {			
			chatClient.close();
			System.out.println("Chat client has closed.");
		}
	}
}
```
>**代码说明**
>* 指定Stream配置。我们使用StandardStreamConfig。192.168.1.80为服务器域名。5222是XMPP服务的标准端口。
>>>```
>>>StandardStreamConfig streamConfig = new StandardStreamConfig("192.168.1.80", 5222);
>>>streamConfig.setResource("my_notebook");
>>>```
><br><br>
>* IChatClient是Chalk库的核心API接口，使用它连接到服务器端。

<br><br>
### 4.3 运行测试代码
#### 4.3.1 构建hello-xmpp-app程序。
```
cd hello-xmpp-app
mvn clean package
```

<br><br>
#### 4.3.2 启动Granite Lite Mini Server
```
cd granite-lite-mini-1.0.4-RELEASE
java -jar granite-server-1.0.4-RELEASE.jar -console
```

<br><br>
#### 4.3.3 运行app测试程序
```
java -jar hello-xmpp-app-0.0.1-RELEASE.jar
```
正常的话，可以看到以下的运行结果。
![](https://dongger-s-img-repo.oss-cn-shenzhen.aliyuncs.com/images/connect_to_granite_server.png)

<br><br>
## 5 通过插件扩展XMPP
### 5.1 定义XMPP扩展协议
XMMP协议的强大之处在于它的扩展性。按照官方的说法，大概有350个XEPs扩展协议。
![](https://dongger-s-img-repo.oss-cn-shenzhen.aliyuncs.com/images/how_many_xeps_exists.png)<br><br>
接下来，让我们来尝试扩展XMPP协议。<br><br>
让我们来设计一个简单的扩展协议，如下：
```
<iq from="geologist@192.168.1.180">
	<hello xmlns="urn:lithosphere:tutorials:hello-xmpp"
			geologist-name="Dongger">
		<greeting>
			Hello, XMPP Server!
		</greeting>
	</hello>
</iq>

<iq to="geologist@192.168.1.180">
	<hello xmlns="urn:lithosphere:tutorials:hello-xmpp">
		<greeting>
			Hello, Dongger! I'm Granite XMPP Server.
		</greeting>
	</hello>
</iq>
```
>**注**
>* 客户端向服务器发送一条hello信息。
><br><br>
>* 服务器端收到后，返回使用geologist name的问候，并声明自己是Granite XMPP Server。

<br><br>
这个协议非常之简单，但是包含了完整的XMPP协议元素。它包含了客户端和服务器端的完整交互。服务器读取客户端发来请求的参数，对应这个参数的逻辑处理，然后返回处理结果给客户端。<br><br>
Ok，让我们来用插件技术来实现这个协议。

<br><br>
### 5.2 开发协议包
为何我们需要单独开发一个包含协议对象的协议包？<br><br>
这是因为我们会使用叫OXM（Object-XMPP Mapping）的技术。简单来说，我们希望在开发中，能够屏蔽掉XMPP协议实现的细节，简化开发过程。<br><br>
关于OXM，可以参考概念文档中的[OXM章节](./Concepts.md#OXM)
<br><br>
使用OXM，我们会定义的协议对象，这些协议对象，在客户端和服务器端，都会被使用。协议对象是可复用的。<br><br>
所以，我们最好定一个单独的协议包，已便于在后面客户端和服务器端复用这些协议对象。

<br><br>
#### 5.2.1 创建协议工程
创建hello-xmpp-protocol工程，pom.xml如下：
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

	<groupId>com.thefirstlineofcode.lithosphere.tutorials.helloxmpp</groupId>
	<artifactId>hello-xmpp-protocol</artifactId>
	<name>Hello XMPP protocol</name>
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

<br><br>
#### 5.2.2 定义协议对象
定义一个协议对象Hello。
```
@ProtocolObject(namespace="urn:lithosphere:tutorials:hello-xmpp", localName = "hello")
public class Hello {
	public static final Protocol PROTOCOL = new Protocol("urn:lithosphere:tutorials:hello-xmpp", "hello");

	private String geologistName;
	
	@NotNull
	@TextOnly
	private String greeting;
	
	public String getGeologistName() {
		return name;
	}
	
	public void setGeologistName(String geologistName) {
		this.geologistName = geologistName;
	}
	
	public String getGreeting() {
		return greeting;
	}
	
	public void setGreeting(String greeting) {
		this.greeting = greeting;
	}
}
```
> **代码说明**
>* 使用@ProtocolObject标注，定义XMPP扩展协议的namespace和local-name。
>>>```
>>>@ProtocolObject(namespace = "urn:lithosphere:tutorials:hello-xmpp", localName = "hello")
>>>```
>>框架在读取到ProtocolObject标注时，会把它翻译为对应的XMPP协议扩展语法。
>>>```
>>><hello xmlns="urn:lithosphere:tutorials:hello-xmpp" ...>
>>>...
>>></hello>
>>>```
><br><br>
>* greeting字段上有两个标注，@NotNull和@TextOnly。
>>>```
>>>@NotNull
>>>@TextOnly
>>>private String greeting;
>>>```
>>>在翻译协议时，TextOnly会生成下面格式的XML。
>>>```
>>><greeting>
>>>	Hello, XMPP Server!
>>></greeting>
>>>```
>>>而NotNull是OXM框架提供的一个Validator标签，它检查greeting字段不能为null。
><br><br>
>* OXM框架提供了很多标注，用于定制XMPP协议的XML文档。请参考Basalt项目文档了解更多信息。
><br><br>
>* 我们定义了一个静态协议字段PROTOCOL。因为我们后面需要注册插件扩展时，会经常需要使用到协议信息。定义这么一个公有静态协议字段，会方便后续的扩展注册。
>>关于协议(Protocol)，可以参考概念文档里的[Protocol and Protocol Chain](./Concepts.md#Protocol)

<br><br>
#### 5.2.3 构建安装协议包
因为需要在后续开发客户端和服务器端插件包时，引用协议包，所以我们在hello-xmpp-protocol工程里，执行构建安装指令，把协议包安装到本地maven仓库。
```
cd hello-xmpp-protocol
mvn clean install
```

<br><br>
协议包已经开发完成，你可以参考官方开源仓库代码[hello-xmpp-protocol协议包工程源码](https://github.com/TheFirstLineOfCode/hello-lithosphere-tutorials/tree/main/hello-xmpp/hello-xmpp-protocol)

<br><br>
### 5.3 开发服务器端插件
#### 5.3.1 服务器端插件工程
hello-xmpp-server工程已经存在，我们在前面使用它来创建测试用户。

<br><br>
#### 5.3.2 扩展Pipeline Extenders
创建PipelineExtendersContributor类，代码如下：
```
@Extension
public class PipelineExtendersContributor extends PipelineExtendersConfigurator {
	private static final IqProtocolChain PROTOCOL_CHAIN_HELLO = new IqProtocolChain(Hello.PROTOCOL);

	@Override
	protected void configure(IPipelineExtendersConfigurator configurator) {
		configurator.registerCocParser(PROTOCOL_CHAIN_HELLO, Hello.class);
		configurator.registerCocTranslator(Hello.class);
		
		configurator.registerSingletonXepProcessor(PROTOCOL_CHAIN_HELLO, new HelloProcessor());
	}

}
```
> **代码说明**
>* PipelineExtendersContributor通过继承抽象类PipelineExtendersConfigurator，实现configure抽象方法来配置pipeline extenders扩展。<br><br>
>* Pipeline Extenders是服务器端的主要扩展点接口。Granite服务器除了使用插件架构之外，在设计中，还使用了Pipeline架构设计。Granite Framework提供了一组Pipeline Extenders扩展点，用于扩展Granite插件功能。更多相关信息，可以参考Granite开发文档。
><br><br>
>* 类名上的@Extension表示这是一个PF4J的扩展。<br><br>
>* 我们在configure方法中扩展了3个Pipeline Extenders。<br>
>>> 我们双向注册了Hello协议对象的Protocol Parser和Protocol Translator。<br>
>>> 这是因为Server端需要双向解析翻译Hello协议对象。这意味着，Server端既需要解析客户端传输过来的Hello协议消息，又会向客户端返回Hello协议消息。<br>
所以，我们既要注册做协议解析的Protocol Parser，又需要注册做协议翻译的Protocol Translator。<br><br>
不需要编写parser和translator，我们可以直接使用系统内置的CoC Parser和CoC Translator。
>>>```
>>>configurator.registerCocParser(PROTOCOL_CHAIN_HELLO, Hello.class);
>>>configurator.registerCocTranslator(Hello.class);
>>>```
>>>在这里，我们使用PROTOCOL_CHAIN_HELLO协议链和Hello协议类，来注册协议对象解析器。<br><br>
>>>我们使用Hello协议对象的类对象，来注册协议翻译器。<br><br>
>>>关于协议链(Protocol Chain)，可以参考概念文档里的[Protocol and Protocol Chain](./Concepts.md#Protocol)
>>><br><br>
>>>关于协议解析器和协议翻译器，可以参考概念文档里的[Protocol Parser and Translator](./Concepts.md#Protocol Parser)
>>><br><br>
>>>我们还注册了一个单实例的Xep Processor，HelloProcessor用来处理客户端发过来的Hello协议。
>>>```
>>>configurator.registerSingletonXepProcessor(PROTOCOL_CHAIN_HELLO, new HelloProcessor());
>>>```
>><br><br>
>>Xep Processor可以说是服务器端最重要的Pipeline Extender。我们在前面提到，XEPs扩展协议大概有350个，我们当然需要提供响应的机制，允许注册Xep Processor来处理不同的XMPP扩展协议。

<br><br>
#### 5.3.3 编写HelloProcessor代码
HelloProcessor类实现IXepProcessor接口，它处理客户端发来的Hello协议。
```
public class HelloProcessor implements IXepProcessor<Iq, Hello>, IConfigurationAware {
	private static final String CONFIGURTION_KEY_XMPP_SERVER_NAME = "xmpp.server.name";
	private String xmppServerName;

	@Override
	public void process(IProcessingContext context, Iq iq, Hello hello) {
		if (hello.getGeologistName() == null) {
			throw new ProtocolException(new BadRequest("Null geologist name."));
		}
		
		Hello helloFromServer = new Hello();
		helloFromServer.setGreeting(String.format("Hello! %s. I'm %s.", hello.getGeologistName(), xmppServerName));
		Iq result = Iq.createResult(iq, helloFromServer);
		
		context.write(result);
	}

	@Override
	public void setConfiguration(IConfiguration configuration) {
		xmppServerName = configuration.getString(CONFIGURTION_KEY_XMPP_SERVER_NAME, "Granite XMPP Server");
	}

}
```
> **代码说明**
>* 实现IXepProcessor<Iq, Hello>，在接口方法process里。编写对Hello扩展协议的处理逻辑。<br><br>
>* 先检查协议参数有效性，这里，我们检查geogistName参数是否为null。
>>>```
>>>if (hello.getGeogistName() == null) {
>>>	throw new ProtocolException(new BadRequest("Null geologist name."));
>>>}
>>>```
>>>这里，我们直接抛出封装BadRequest错误的ProtocolException例外。如果例外抛出，这个例外将会被框架处理，返回客户端一个错误如下：
>>>```
>>><iq type='error' id='ij20ftxf'>
>>>	<error type='modify'>
>>>		<bad-request xmlns='urn:ietf:params:xml:ns:xmpp-stanzas'/>
>>>		<text xmlns='urn:ietf:params:xml:ns:xmpp-stanzas'>
>>>			Null geologist name
>>>		</text>
>>>	</error>
>>></iq>
>>>```
>>>关于Granite的错误处理，请参考Granite开发文档。

<br><br>
#### 5.3.4 编写插件配置文件
修改plugin.properties文件，在文件最后增加一行内容：
```
plugin.id=hello-xmpp-server
plugin.provider=TheFirstLineOfCode
plugin.version=0.0.1-RELEASE
non-plugin.dependencies=hello-xmpp-protocol
```
>**代码说明**
>* non-plugin是Granite工程在PF4J插件管理框架上扩展的一个插件配置语法。
>>>```
>>> non-plugin.dependencies=hello-xmpp-protocol
>>>```
>>>简单来说，hello-xmpp-protocol包并不是一个插件包，它并没有定义plugin.properties，它只是一个普通标准的jar包。<br><br>
>>>PF4J并不支持插件包引用非插件包，但是因为Lithosphere的协议包，是被客户端（chalk）和服务器端（granite）两端都要引用的依赖包。协议包在客户端（chalk）里被使用时，它并不需要是PF4J插件格式的插件包，只需要是标准格式的jar包就可以了。<br><br>
>>>Lithosphere对此的解决方案，是为PF4J增加一个non-plugin.dependencies的语法，对于不是插件包的依赖，可以使用这个配置语法来进行配置。

<br><br>
#### 5.3.5 构建部署服务器端插件
构建hello-xmpp-server插件包
```
cd hello-xmpp-server
mvn clean package
```

<br><br>
将hello-xmpp-server插件包和它依赖的hello-xmpp-protocol包，把这两个jar包，copy到服务器的plugins目录下。
```
cp hello-xmpp-protocol/target/hello-xmpp-protocol-0.0.1-RELEASE.jar granite-lite-mini-1.0.4-RELEASE/plugins

cp hello-xmpp-server/target/hello-xmpp-server-0.0.1-RELEASE.jar granite-lite-mini-1.0.4-RELEASE/plugins
```

服务器端插件已经开发完成，你可以参考官方开源仓库代码[hello-xmpp-server服务器端插件包工程源码](https://github.com/TheFirstLineOfCode/hello-lithosphere-tutorials/tree/main/hello-xmpp/hello-xmpp-server)

<br><br>
### 5.4 开发客户端插件
#### 5.4.1 创建客户端工程
建立hello-xmpp-client目录，在目录下，创建pom.xml内容如下：
```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns="http://maven.apache.org/POM/4.0.0"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">

	<modelVersion>4.0.0</modelVersion>

	<parent>
		<groupId>com.thefirstlineofcode.chalk</groupId>
		<artifactId>com.thefirstlineofcode.chalk.parent</artifactId>
		<version>1.0.2-RELEASE</version>
	</parent>

	<groupId>com.thefirstlineofcode.lithosphere.tutorials.helloxmpp</groupId>
	<artifactId>hello-xmpp-client</artifactId>
	<version>0.0.1-RELEASE</version>
	<name>Hello XMPP client plugin</name>
	
	<dependencies>
		<dependency>
			<groupId>com.thefirstlineofcode.chalk</groupId>
			<artifactId>chalk-core</artifactId>
		</dependency>
		<dependency>
			<groupId>com.thefirstlineofcode.lithosphere.tutorials.helloxmpp</groupId>
			<artifactId>hello-xmpp-protocol</artifactId>
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
> **代码说明**
>* parent POM为com.thefirstlineofcode.chalk:com.thefirstlineofcode.chalk.parent。
>>```
>><parent>
>>	<groupId>com.thefirstlineofcode.chalk</groupId>
>>	<artifactId>com.thefirstlineofcode.chalk.parent</artifactId>
>>	<version>1.0.2-RELEASE</version>
>></parent>
>>```
><br><br>
>* 扩展chalk插件，需要依赖com.thefirstlineofcode.chalk:chalk-core。当然，我们还需要依赖协议包。
>>```
>><dependency>
>>	<groupId>com.thefirstlineofcode.chalk</groupId>
>>	<artifactId>chalk-core</artifactId>
>></dependency>
>><dependency>
>>	<groupId>com.thefirstlineofcode.lithosphere.
>>		tutorials.helloxmpp</groupId>
>>	<artifactId>hello-xmpp-protocol</artifactId>
>>	<version>0.0.1-RELEASE</version>
>></dependency>
>>```

<br><br>
#### 5.4.2 定义客户端插件API接口
编写客户端插件的一个重要任务是，我们需要定义插件API给客户端主程序调用。我们定义一个名为ISayHelloToXmpp的API接口。
```
public interface ISayHelloToXmpp {
	public interface IHelloListener {
		void greetingReceived(String greeting);
		void errorReceived(StanzaError error);
	}
	
	void setHelloListener(IHelloListener helloListener);
	void sayHello(String myName);
}
```

<br><br>
#### 5.4.3 实现API
我们编写客户端插件API的实现逻辑。
```
public class SayHelloToXmpp implements ISayHelloToXmpp {
	private IChatServices chatServices;
	
	private IHelloListener helloListener;
	
	@Override
	public void sayHello(final String myName) {
		chatServices.getTaskService().execute(new TaskAdapter<Iq>() {

			@Override
			public void trigger(IUnidirectionalStream<Iq> stream) {
				Iq iq = new Iq(Iq.Type.SET, new Hello(myName));
				stream.send(iq);
			}

			@Override
			public void processResponse(IUnidirectionalStream<Iq> stream, Iq response) {
				if (helloListener != null) {					
					Hello hello = response.getObject();
					helloListener.greetingReceived(hello.getGreeting());
				}
			}
			
			@Override
			public boolean processError(IUnidirectionalStream<Iq> stream, StanzaError error) {
				if (helloListener != null) {					
					helloListener.errorReceived(error);
				}
				
				return true;
			}
		});
	}

	@Override
	public void setHelloListener(IHelloListener helloListener) {
		this.helloListener = helloListener;
	}

}
```
> **代码说明**
>* 插件API实现一般需要使用IChatServices。IChatService封装了XMPP客户端的基础网络通讯服务，客户端插件调用它来简化业务功能的开发。<br><br>
>* IChatService由框架注入给插件API实现。我们只需要在API实现类中，提供一个IChatService的类变量就可以了，框架会自动把IChatService的实现注入。<br><br>
>* 我们使用Task Service来执行一个ITask，我们采用继承TaskAdapter抽象类，来简化实现ITask。
>* 关于Chalk的使用，可以参考[Chalk项目文档](http://github.com/TheFirstLineOfCode/chalk)了解更多。

<br><br>
#### 5.4.4 注册插件信息
我们将编写好的程序逻辑注册到插件中。
```
public class HelloXmppPlugin implements IPlugin {

	@Override
	public void init(IChatSystem chatSystem, Properties properties) {
		chatSystem.registerTranslator(Hello.class, new CocTranslatorFactory<>(Hello.class));
		chatSystem.registerParser(new IqProtocolChain(Hello.PROTOCOL),
				new CocParserFactory<>(Hello.class));
		chatSystem.registerApi(ISayHelloToXmpp.class, SayHelloToXmpp.class);
	}

	@Override
	public void destroy(IChatSystem chatSystem) {
		chatSystem.unregisterApi(ISayHelloToXmpp.class);
		chatSystem.unregisterParser(new IqProtocolChain(Hello.PROTOCOL));
		chatSystem.unregisterTranslator(Hello.class);
	}
}
```
> **代码说明**
>* 实现IPlugin接口。<br><br>
>* 我们在init方法里，注册Hello协议对象的解析器和翻译器。我们使用系统内置的CoC Parser和CoC Translator。CoC的Parser和Translator使用命名约定来翻译XMPP协议文档<br>
>>```
>>chatSystem.registerTranslator(Hello.class,
>>		new CocTranslatorFactory<>(Hello.class));
>>chatSystem.registerParser(
>>		new IqProtocolChain(Hello.PROTOCOL),
>>		new CocParserFactory<>(Hello.class));
>>```
><br><br>
>* 我们为插件注册插件API和API实现。
>>```
>>chatSystem.registerApi(ISayHelloToXmpp.class, SayHelloToXmpp.class);
>>```
><br><br>
>* destory方法，基本上就是init方法的一个反向调用，当销毁插件时，我们将注册的插件相关信息注销掉。

<br><br>
#### 5.4.4 构建安装客户端插件包
客户端应用程序会使用客户端插件包，我们在hello-xmpp-client工程里，执行构建安装指令，把客户端插件包安装到本地maven仓库。
```
cd hello-xmpp-client
mvn clean install
```
<br><br>
客户端插件包已经开发完成，你可以参考官方开源仓库代码[hello-xmpp-client客户端插件包工程源码](https://github.com/TheFirstLineOfCode/hello-lithosphere-tutorials/tree/main/hello-xmpp/hello-xmpp-client)

<br><br>
### 5.5 测试扩展协议
#### 5.5.1 客户端app工程
hello-xmpp-app工程已经存在，在前面，我们用它来测试客户端-服务器端连接性。<br><br>
我们需要增加一些代码逻辑来测试Hello扩展协议。

<br><br>
#### 5.5.2 编写app测试代码
修改hello-xmpp-app的Main类代码如下：
```
public class Main implements IHelloListener {
	private static final String USER_NAME = "geologist";
	private static final String USER_PASSWORD = "I like lithosphere.";
	
	public static void main(String[] args) {
		new Main().run();
	}
	
	private void run() {
		IChatClient chatClient = createChatClient();
		sayHelloToXmpp(chatClient);
		
		if (chatClient != null && chatClient.isConnected())
			chatClient.close();
	}

	private void sayHelloToXmpp(IChatClient chatClient) {
		try {
			chatClient.connect(new UsernamePasswordToken(USER_NAME, USER_PASSWORD));
		} catch (ConnectionException e) {
			throw new RuntimeException("Can't connect to server.", e);
		} catch (AuthFailureException e) {
			throw new RuntimeException("????", e);
		}
			
		ISayHelloToXmpp sayHelloToXmpp = chatClient.createApi(ISayHelloToXmpp.class);
		sayHelloToXmpp.setHelloListener(this);
		sayHelloToXmppAsAnonymous(sayHelloToXmpp);
		
		try {
			Thread.sleep(5000);
		} catch (InterruptedException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
		
		sayHelloToXmppAsDongger(sayHelloToXmpp);
		
		try {
			Thread.sleep(5000);
		} catch (InterruptedException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
	}

	private static void sayHelloToXmppAsDongger(ISayHelloToXmpp sayHelloToXmpp) {
		System.out.println();
		System.out.println();
		System.out.println("Say hello to XMPP as Dongger.");
		sayHelloToXmpp.sayHello("Dongger");
		System.out.println();
	}

	private static void sayHelloToXmppAsAnonymous(ISayHelloToXmpp sayHelloToXmpp) {
		System.out.println();
		System.out.println();
		System.out.println("Say hello to XMPP as anonymous.");
		System.out.println();
		sayHelloToXmpp.sayHello(null);
	}

	private static IChatClient createChatClient() {
		StandardStreamConfig streamConfig = new StandardStreamConfig("192.168.1.80", 5222);
		streamConfig.setTlsPreferred(false);
		streamConfig.setResource("my_desktop");
		IChatClient chatClient = new StandardChatClient(streamConfig);
		
		chatClient.register(HelloXmppPlugin.class);
		
		return chatClient;
	}

	@Override
	public void greetingReceived(String greeting) {
		System.out.println(greeting);
	}

	@Override
	public void errorReceived(StanzaError error) {
		System.out.println(String.format("Received an stanza error: %s." , error));
	}
}
```
>**代码说明**
>* 创建IChatClient时，注册HelloXmppPlugin插件。
>>```
>>chatClient.register(HelloXmppPlugin.class);
>>```
><br><br>
>* sayHelloToXmppAsAnonymous方法发送的Hello协议里的geologistName为null，它会引发服务器端抛出错误例外。这会触发HelloListener收到errorReceived()方法调用。
><br><br>
>* sayHelloToXmppAsDongger方法发送正确的Hello协议，服务器端会返回XMPP Server的greeting。HelloListener会收到greetingReceived()方法调用。

关于hello-xmpp app程序，你可以参考官方开源仓库代码[hello-xmpp-app工程源码](https://github.com/TheFirstLineOfCode/hello-lithosphere-tutorials/tree/main/hello-xmpp/hello-xmpp-app)

<br><br>
#### 5.5.3 运行app测试扩展协议
构建hello-xmpp-app。
```
cd hello-xmpp-app
mvn clean package
```

<br><br>
运行app程序测试扩展协议。
```
cd target
tar -xzvf hello-xmpp-app-0.0.1-RELEASE.tar.gz
cd hello-xmpp-app-0.0.1-RELEASE
java -jar hello-xmpp-app-0.0.1-RELEASE.jar
```

<br><br>
如果一切正常，应该可以看到以下的结果。
![](https://dongger-s-img-repo.oss-cn-shenzhen.aliyuncs.com/images/run_hello_xmpp_app.png)

<br><br>
>**注**
>* 运行hello_xmpp_app之前，需要先启动Granite XMPP Server。

<br><br>
## 6 结论
* Lithosphere平台是一个XMPP平台，并重度依赖于插件架构技术。<br><br>
* 我们在这篇教程里，使用Granite XMPP Lite Server的最小mini版本，它本身并不支持任何的通讯协议。即使最简单的[XMPP Ping（XEP-0199）](https://xmpp.org/extensions/xep-0199.html)协议也不支持。所有XMPP通讯功能，都通过插件机制来实现和提供。<br><br>
* XMPP采用经典的C/S架构，客户端连接服务器，需要有服务器端的账号。在插件架构下，所有功能都可以通过插件机制灵活扩展。我们编写了一个插件来扩展Granite Server Console，帮助创建一个测试用户。<br><br>
* XMPP的强大之处在于它的扩展性和灵活性。我们通过扩展Hello协议来演示在Lithosphere平台下，如何实现扩展协议，如何编写插件来扩展平台功能。<br><br>
* 基于OXM技术，我们可以简单的定义协议对象，而不需要去理解XMPP协议细节。我们将协议对象封装在协议包中，协议包是标准的jar包，被Chalk客户端和Granite服务器端复用。<br><br>
* 我们开发Granite服务器插件，通过Pipeline Extenders，我们扩展Granite服务器端功能。通过简单的注册CoC Parser和CoC Translator，Granite服务器就能够识别协议对象，并做正确的解析和翻译。我们注册协议的协议处理器XEP Processor。在协议处理器中，直接对解析好的业务对象进行逻辑处理。<br><br>
* 对应于服务器端，我们在客户端也开发客户端插件来处理协议逻辑。我们定义客户端API接口，并提供接口实现。Chalk提供封装了XMPP协议细节的网络接口Chat Services，我们可以调用IQ Service，Message Service，Presence Service，Task Service等网络服务，来处理扩展协议相关细节。当然，我们也通过协议注册接口，将解析器、翻译器等信息注册到插件管理系统。<br><br>
* 教程中的例子，看似简单。但要理解的是，所有的Lithosphere平台的功能，都是通过这样的机制扩展出来的。通过这套扩展机制，Lithosphere平台实现了RFC3921和部分XEPs，包括：
	* RFC3921(XMPP IM)
	* XEP-0199(XMPP Ping)
	* XEP-0030(Service Discovery)
	* XEP-0033(Extended Stanza Addressing)
	* XEP-0203(Delayed Delivery)
	* XEP-0004(Data Form)
	* XEP-0059(Result Set Management)
	* XEP-0077(In-Band Registration)
	* XEP-0045(Multi-User Chat)
	* XEP-0066(Out of Band Data)
	* XEP-0114(Jabber Component Protocol)
<br><br>
* 当然，实时通讯显然不是Lithosphere平台的重点。在这套扩展机制下，我们开发了Sand的所有IoT平台功能。特别是Webcam插件和Friends协议，它们证明了Lithosphere平台的插件扩展机制，可以用于实现极其复杂的通讯逻辑。<br><br>
* Lithosphere IoT平台，具备非常好的灵活性，和极强的扩展性，适合用来开发复杂、灵活的IoT应用。
