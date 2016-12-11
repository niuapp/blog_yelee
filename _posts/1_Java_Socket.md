---
title: Java中的网络编程初试
date: 2016-12-09 12:36:00
categories:
	- Java
tags:
	- 学习笔记
	- 网络
---

![网络编程](/assets/img/cover/网络编程.png)

关键字：TCP UDP Socket 网络编程
<!-- more --> 
<br/>   
## 概述
<br/>

> 网络编程：编写通过网络建立联系的程序

可以分析这个"网络编程"

要建立连接必然就是2个或以上的个体，可以先把每一个个体当做一个叫"Socket"的对象。

如果它们之间想建立联系，其中一个Socket就需要知道另一方在哪儿，所以每个Socket得有个位置属性，在计算机中，"ip"确定网络上的位置，"端口号"确定程序在计算机中的位置，这个属性分解成"ip"和"端口号"来确定。

接着两个Socket要开始互相交流了，需要一个交流规则，这个规则就是"协议"。

<br/>
**总结一下：**网络编程，就是两个可以通过ip和端口号确定位置的Socket使用相同的协议来进行交流。

> 网络通信三要素: **ip  -  端口号  -  协议**

"ip"和"端口号"，这两个暂且略过不提，这里只需要知道一个指网络中的位置，一个指计算机中的位置就好。

> 插入一句，我自己现在的理解，面向对象的开发，就是创造对象，梳理对象之间的关系，要注重无中生有和拿来主义，只是在必要的时候去进行比较接近底层的实现(关键现在也没那个能力啊，努力中...)，所以先重点放到分析对象和关系，构造对象和关系上。


协议，在这里分两部分用java代码实现UDP和TCP协议支持的小小程序，而它们所支持的应用层协议可以之后去稍微了解下。

分析：

1. 因为要两个或以上的人来建立连接，所以需要创建出两个或以上的这个对象，一个发送，一个接收
2. 这两个对象需要有位置标识，需要确定
3. 这两个对象需要一个协议来传输数据

<br/>
## 基于UDP协议的"DatagramSocket"
<br/>

UDP协议是面向无连接的，可以把它当做广播、通知来看，一方发给另一方就是 把消息直接丢给它，不管对方收没收到，我发了就好了，所以发送端和接收端都是比较简单的，它们没有建立起连接(不指联系)，就单是通过ip端口号确定目标，发送 接收。

UDP一个使用最多的地方就是网络聊天了。接收端接收消息，发送端发送消息，当然实际中的聊天工具肯定不是这么简单，中间要经过一个服务器去转发处理什么的，但这个是基础，通过这个明白一些原理还是不错的。

Java中通过 "DatagramSocket" 来创建基于UDP的Socket对象，而Socket中又有了ip和端口号属性。

这一个玩意儿就把 ip、端口号、协议都确定了，所以就拿来这个直接使用就好了。

> 好了，开始写代码。

<br/>
### 发送端

当然是先去 看api，查这个类的构造方法。

发现这个它有构造方法能直接确定端口号，然后还有一个有 "InetAddress
"参数，查发现，这个表示ip，如果使用这个构造就直接确定了自己Socket的端口号和ip，但这里有个问题，现在是想发送给另一个Socket，所以使用这个构造和对方位置是没什么关系的，所以继续查api，找有没有什么发送的方法

果然，有个`void send(DatagramPacket p)`这个方法，但它需要一个参数，看名字这应该是一个数据包，发送一个数据包，对就是这样，所以继续看这个类是怎么回事。

看构造方法，这里也有可以设置端口号和ip的位置，这下就可以确定发送目标了，然后还有byte数组，这肯定是要发送的数据了。

走完一圈，从创建 DatagramSocket对象，然后到它发送数据，整个流程就是这样了，所以代码就有了。

```java

String msg = "hello!!"; //消息内容
DatagramSocket ds = new DatagramSocket(); //发送端Socket对象
ds.send(new DatagramPacket(msg.getBytes(), msg.getBytes().length, InetAddress.getByName("192.168.0.66"), 16666)); //发送给 192.168.0.66：16666
```

超级简单的一个发送端就写好了，当然，这里只是把主体过一次，最后会有一个小demo来总结下。

<br/>
### 接收端

继续看 DatagramSocket类的方法，有一个接收的方法 `void receive(DatagramPacket p)`,同样需要一个数据包参数，这里这个参数不是其他地方传过来的，如果是自己创建肯定是没有数据的，因为还没接收到啊，所以这里应该是一个用来接收数据的空包，什么是空的包，构造数据包的参数，恩，是个空的byte数组就好了。

问题来了，这个byte数组要多大呢，肯定是越大越好，要不数据接收不全咋办，可是这个大没有极限啊，而且还占地方，所以只能是尽量大了，这可以根据实际情况决定，比如这边我知道只是接收一个短字符串，我弄个byte[100]就不少了，具体情况具体看待，一般可以byte[1024]，UDP面向无连接，所以丢包也就很正常了，毕竟快哈。

还有一个问题，接收的参数不是传过来的，而是你创建的空包，所以什么时候接收呢，怎么就知道人家发送了？...实际上答案就是"不知道"。人家又没通知你，哪里知道要开始接收啊。所以这个接收端还要保持长时间开着，不停得保持接收状态，就好像人的耳朵一样，接收广播通知，这也是UDP的特点了，所以可以来一个死循环，保持这个接收端一直开着。

```java

DatagramSocket ds = new DatagramSocket(16666);//确定端口号，ip计算机会帮你确定

//用循环无限接收
while (true)
{
	//接收包
	byte[] bys = new byte[1024];
	DatagramPacket dp = new DatagramPacket(bys, bys.length);
	
	//接收
	ds.receive(dp);
	//解析为字符串
	String ss = new String(dp.getData(), 0, dp.getLength());

	//获取发送者ip 和主机名
	String name = dp.getAddress().getHostName();
	String ip = dp.getAddress().getHostAddress();

	System.out.println(name+" "+ip+" : "+ss);
}
```

一个超级简单的接收端就好了

<br/>
> UDP小demo
接下来是一个完整的demo，可以放到一个 .java文件中编译运行下试试，包含了发送端 接收端，当然也可以分开写

```java

import java.net.DatagramSocket;
import java.net.DatagramPacket;
import java.net.InetAddress;
import java.io.IOException;
import java.io.BufferedReader;
import java.io.InputStreamReader;

//发送端
class SendT implements Runnable
{	
	private DatagramSocket ds;
	
	//通过测试类传过来的DatagramSocket对象进行初始化
	public SendT(DatagramSocket ds)
	{
		this.ds = ds;
	}

	public void run()
	{
		try
		{
			//封装键盘录入
			BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
			String line = null;

			
			while (true)
			{
				//System.out.print("我说：");
				line = br.readLine();
				
				if (line.equals("null"))
				{
					break;
				}
				//把录入的数据打包
				byte[] bys = line.getBytes();
				DatagramPacket dp = new DatagramPacket(bys, bys.length, InetAddress.getByName("192.168.22.68"), 19999);

				//发送
				ds.send(dp);
			}
			}
		catch (IOException e)
		{
			System.out.println("---------");
		}
		finally
		{
			try
			{
				//关闭流
				ds.close();
			}
			catch (Exception e)
			{
			}
		}	
	}
}

//接收端
class ReceiveT implements Runnable
{
	private DatagramSocket ds;
	
	//通过测试类传过来的DatagramSocket对象进行初始化
	public ReceiveT(DatagramSocket ds)
	{
		this.ds = ds;
	}

	public void run()
	{
		try
		{
			while (true)//一直等待接收
			{
				//缓存包
				byte[] bys = new byte[10];
				DatagramPacket dp = new DatagramPacket(bys, bys.length);

				//接收
				ds.receive(dp);

				//转成字符串
				String s = new String(dp.getData(), 0, dp.getLength());

				String name = dp.getAddress().getHostName();
				String ip = dp.getAddress().getHostAddress();
				
				System.out.println("---------------");
				System.out.println(name+"-> "+s);
				System.out.println("---------------");
				
			}
			}
		catch (IOException e)
		{
			System.out.println("---------");
		}

		//不关闭流
	}
}

//测试主类
public class UDPTest
{
	public static void main (String[] args) throws IOException
	{
		//创建发送端和接收端对象
		SendT st = new SendT(new DatagramSocket());
		ReceiveT rt = new ReceiveT(new DatagramSocket(19999));

		//开启线程
		new Thread(rt).start();
		new Thread(st).start();
		
	}
}
```

<br/>
## 基于TCP协议的"Socket"和"ServerSocket"
<br/>

TCP协议是面向连接的，所以和UDP协议最大的不同就是在于，它需要保持一个长连接，通过这个连接通道来进行联系。这个看起来就很繁琐，所以一般可以用来传输文件什么的，做一些比较耗时的事。比起UDP就可靠多了。

Java中用"Socket"和"ServerSocket"两个类来建立基于TCP协议的联系。

虽然看起来比UDP复杂，但Java代码实现却简单多了，它内部已经做好了很多事，所以只需要有一个通过连接通道进行传输的概念就好了。

<br/>
### 发送端

Socket类的对象。

查api，看构造方法，可以直接new出来 或者设置一些参数，因为要保持长连接，通过连接通道交流，发送端要和服务端先建立连接通道，所以需要选择带端口号和ip的构造来创建对象，在构造时java就会帮忙去和指定的ip端口建立连接。

在构造完后，其实两端已经建立好了连接通道(java都帮搞定了)，所以就可以开始通过通道发送数据了，看api有一个`OutputStream getOutputStream()`方法，这下就好了，IO操作，写入数据，关闭流完事，下边代码：

```java

	//客户端Socket对象
	Socket s = new Socket("192.168.22.66", 16666);
	
	//获取输出流
	OutputStream os = s.getOutputStream();
	
	//写入数据
	os.write("AAAAA".getBytes());
	
	//关闭流
	s.close();
```

非常简单，一个发送端就好了，TCP中发送端和接收端是可以互相通信的，所以在上服务端的反馈的话，一个简单的客户端就能搞定了，继续看接收端

<br/>
### 接收端

ServerSocket对象。

同样的看api，也和UDP一样，创建当然需要一个端口号来确定位置。

创建之后，想接收到发送端的消息，看api发现没什么对应的方法来直接获取消息，但有一个返回Socket对象的方法。看方法名，接收，那就是了，这个方法就是拿到 连接到这个接收端的代表发送端的Socket。

有了Socket之后，一切就又很简单了，Socket中除了获取输出流，还有获取输入流的方法`InputStream getInputStream()`，那个这个就是对应发送端的输出流了，读这个流就可以获取发送信息了。

接收端也是可以去给发送端发消息的，所以其实接收端可以当做一个小小的服务器，发送端就是一个要和这个服务器连接的客户端了。它们之间是可以互相通信的。

简单代码：

```java

	ServerSocket ss = new ServerSocket(16666);
	
	//监听
	Socket s = ss.accept();
	
	//获取输入流
	InputStream is = s.getInputStream();
	
	//读数据
	byte[] bys = new byte[1024];
	int l = 0;
	while ((l=is.read(bys)) != -1)
	{
		System.out.println(new String(bys, 0, l));
	}
	
	//不关，保持服务 或者调用 close()关闭服务
```

TCP的demo就没了，如果只是简单的发送接收上边的两小段就可以了，但它可不是为了聊天存在的，应该当做客户端和服务端，用来传输文件，这样其实关于Socket的代码也大体是这些(多了停止输入或输出两个方法，查api)，只不过多了一些IO的代码，那些都比较固定，练java的IO操作可以试着写写。

<br/>
*完*
<br/>

> 总结:
> java做了绝大部分事情，只是把可能需要变动的给暴露出来，所以也就没有关于怎么实现的描述了(其实我是不太懂)，但基本过程还是可以稍微看下的，可当做回顾复习之类，第一次就写这么多，竟然花了4个小时，还是很有成就感的，哈哈。
 
 
<br/>
在csdn上看到一句话：

> **代码是最为耐心、最能忍耐和最令人愉快的伙伴，在任何艰难困苦的时刻，它都不会抛弃你。**

