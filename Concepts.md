# 概念
Lithosphere IoT平台主要基于以下一些概念设计。
## 1 XMPP
XMPP是Extensible Messaging and Prensence Protocol。以下是一些XMPP协议的关键特征：
* 基于XML消息格式
* 实时通讯协议
* 开放标准

### 1.1 架构
在XMPP网络中，客户端通过TCP连接，使用全局唯一名登录到服务器上。<br><br>
客户端可以向服务器上的其它客户端发送消息，服务器提供中转和路由功能，将消息转发给目标客户端。<br><br>
客户端和服务器之间的连接，被称为C2S连接。<br><br>
在XMPP网络中，还允许服务器之间的连接，这被称为S2S连接。通过S2S连接，登录到不同的服务器上的客户端，可以互相通讯。<br>
```
example.net <--- S 2 S ---> example.com <--- C 2 S ---> romeo@example.com
     ^                         ^
     |                         |
     C                         C
     2                         2
     S                         S
     |                         |
     v                         v
paris@example.net       juliet@example.com
```
* 如上图所示，客户端juliet@example.com和romeo@example.com，通过C2S连接登录到服务器上example.com上，所以这两个客户端之间，可以互相通讯。
* 服务器example.com和example.net之间，建立了S2S的连接。所以，登录到不同服务器上的客户端juliet@example.com和paris@example.net，也可以互相通讯。
### 1.2 Jabber ID
在XMPP网络中，任何实体对象都有一个全局唯一的地址标识。由于历史原因，这个标识被称为Jabber ID或JID。<br>

Jabber ID的格式为：`node@domain/resource`  <br>
在这个Jabber ID表达式里，node和resource都是可选项的。

Jabber ID用来表示XMPP网络里的一切。包括：用户账号、客户端连接、服务器、资源、XMPP服务等。<br>
例如，在domain名为example.com的XMPP网络里：<br>

| 实体        | Jabber ID |
| --------- | ---------- |
| 服务器主机     | exmple.com |
| 用户romeo账号 | romeo@exmple.com |
| 用户romeo手机客户端连接 | romeo@exmple.com/mobile |
| 用户romeo桌面客户端连接 | romeo@exmple.com/desktop |
| 名为happy_life的聊天室 | happy_life@muc.exmple.com |
| .... | .... |

### 1.3 Stream
XMPP协议是一个异步消息通讯协议，在XMPP核心协议里，定义了stream和stanza的概念来实现异步通讯。<br><br>
简单来说，可以认为stream是XMPP中消息交换的通道，而stanza是XMPP里实体之间交换信息使用的最基本消息单元。<br><br>
在XMPP网络里，一个实体要通过XMPP协议跟其它实体进行通讯，需要在连接通道上创建并打开stream。经过协商，stream被打开。<br><br>
然后，实体可以在这个stream通道上，通过发送stanza跟其它实体进行通讯。<br><br>
下图展示了两个实体之间进行的会话。请注意，在两个客户端实体之间，最少需要一个XMPP服务器存在。<br>
* 在下图中，左边的客户端使用DaveBowman账号发起会话，它先在服务器上打开了一个stream通道。
* stream通道被打开后，DaveBowman发送一条message类型的stanza给右边的账号为Hal的客户端。
* Hal读到信息之后，回复了一条message。
* 然后DaveBowman关闭了stream通道，会话结束。<br>
![](https://github.com/TheFirstLineOfCode/Lithosphere/blob/main/stream_and_stanza.png)
### 1.4 Stanza
当XMPP实体建立连接并打开stream后，它就可以给其它XMPP实体发送stanza类型的消息了。
stanza是XMPP协议里进行信息交换的顶级元素。有3种类型的stanza：Message，Presence，IQ(Info/Query)。<br><br>
在标准XMPP协议中，Message一般用于表达实时通讯消息。
```
<message type='chat'
    from='juliet@example.com/balcony'
    to='romeo@example.net'
    xml:lang='en'>
  <body>Wherefore art thou, Romeo?</body>
</message>
```
Presense一般用于表示实体的当前状态。
```
<presence
    from='benvolio@example.org/pda'
    to='romeo@example.net/orchard'
    xml:lang='en'>
  <show>dnd</show>
  <status>gallivanting</status>
</presence>
```
按照XMPP的开发惯例，IQ一般被使用来扩展XMPP协议。
```
<iq from='juliet@example.com/balcony'
    type='set' id='roster_2'>
  <query xmlns='jabber:iq:roster'>
    <item jid='nurse@example.com' name='Nurse'>
      <group>Servants</group>
    </item>
  </query>
</iq>
```
一个XMPP协议的标准实现，除了实现RFC中XMPP Core，XMPP IM等标准规范之外，还可以选择性实现XMPP扩展协议（XEPs）。XMPP扩展协议XEPs的英文全称是XMPP Extension Protocols。<br><br>
XMPP协议的灵活性和扩展性，就充分的被体现在丰富的XEPs上。大部分的XEPs，都采用IQ stanza来做实现。<br><br>
Lithosphere平台，也主要使用IQ来实现IoT扩展协议。

## 2 Plugin-Architecture
插件架构将系统分为两部分：内核系统、插件。<br><br>
![](https://github.com/TheFirstLineOfCode/Lithosphere/blob/main/plugin_architecture.png)<br>
插件架构的主要架构特征是高度模块化和极强的扩展性。<br><br>
经过精心设计的插件系统，每个插件相互之间是相互解耦的，再加上插件在运行时才绑定到主程序，这使得它易于扩展，并可以被灵活部署。
### 2.1 Extension Point
系统可以通过扩展点（Extension Point）来定义可扩展功能接口。例如，一个Extension Point的定义如下：
```
public interface ICommandsProcessor extends ExtensionPoint {
	public static final String DEFAULT_COMMAND_GROUP = "";
	
	String getGroup();
	String[] getCommands();
	String getIntroduction();
	void printHelp(IConsoleSystem consoleSystem);
	boolean process(IConsoleSystem console, String command, String... args) throws Exception;
}

```
### 2.2 Extension
扩展（Extension）是扩展点（Extension Point）的具体实现。通过在插件中实现扩展点，插件就可以系统提供扩展功能。
### 2.3 Why Plugin All
Lithosphere在整个平台中，采用了全插件架构。<br><br>
为什么叫全插件架构呢？简单的解释就是，内核系统只定义开发框架，不实现任何应用层功能。所有的应用层功能，全部是通过插件来实现的。<br><br>
采用这样的架构实现，主要是因为Lithosphere平台基于XMPP协议。<br><br>
在XMPP协议中，除了核心协议XMPP Core、XMPP IM之外，其它的所有的协议都被定义在XEPs协议族中。<br><br>
XEPs定义的协议内容之丰富，从最简单的Ping协议到复杂的协议如多人聊天（MUC）和实时流媒体通讯（Jingle）等。 <br><br>
XMPP的灵活性，就来这种微内核（XMPP Core、XMPP IM） + 丰富的扩展协议族（XEPs）的协议体系设计上。<br><br>
遵循XMPP体系的设计原则，Lithosphere在所有可以采用插件架构的地方，使用插件架构，以获得绝对的灵活性和扩展性。

## 3 IoT


### Actuator


