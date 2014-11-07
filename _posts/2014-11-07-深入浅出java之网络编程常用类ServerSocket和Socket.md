---
layout : life
title : "深入浅出java网络编程之常用类ServerSocket/Socket"
category : 深入理解java
duoshuo: true
date : 2014-11-07
---
------------

因为自己本人是学网络工程的，所以在网络编程基础知识这块不在这里做介绍，后面将会在计算机网络知识模块中介绍，在这里只是简单的介绍介绍常用的类和方法。

--------------

##java.net.ServerSocket

--------------

实现服务器套接字。服务器套接字等待请求通过网络传入。它基于该请求执行某些操作，然后可能向请求者返回结果。 

--------------

* 构造方法摘要： 
 * ServerSocket() 创建非绑定服务器套接字。 
 * ServerSocket(int port) 创建绑定到特定端口的服务器套接字。 
 * ServerSocket(int port, int backlog) 利用指定的 backlog 创建服务器套接字并将其绑定到指定的本地端口号。 
 * ServerSocket(int port, int backlog, InetAddress bindAddr)  使用指定的端口、侦听 backlog 和要绑定到的本地 IP 地址创建服务器。
* 常用方法：
 * accept():Socket  等待Socket去连接，当accept函数正常执行完毕，意味着Socket和ServerSocket建立连接。
 * bind(SocketAddress endpoint):void  将 ServerSocket 绑定到特定地址（IP 地址和端口号）。
 * bind(SocketAddress endpoint, int backlog):void  将 ServerSocket 绑定到特定地址（IP 地址和端口号）。 
 * getInetAddress(): InetAddress 返回此服务器套接字的本地地址。
 * setSoTimeout(int timeout):void 通过指定超时值启用/禁用 SO_TIMEOUT，以毫秒为单位。 
 * close():void :关闭连接。
--------------

##java.net.Socket

--------------
 
 套接字Socket是两台机器间通信的端点。Socket的实际工作由 SocketImpl 类的实例执行

--------------

* 构造方法摘要 
 * Socket()通过系统默认类型的 SocketImpl 创建未连接套接字 
 * Socket(InetAddress address, int port)创建一个流套接字并将其连接到指定 IP 地址的指定端口号。 
 * Socket(InetAddress address, int port, InetAddress localAddr, int localPort) 创建一个套接字并将其连接到指定远程地址上的指定远程端口。
 * Socket(String host, int port, InetAddress localAddr, int localPort) 创建一个套接字并将其连接到指定远程主机上的指定远程端口。
* connect(SocketAddress endpoint)  将此套接字连接到服务器。
* getInputStream() 返回此套接字的输入流。
* getInetAddress() 返回套接字连接的地址。
* getLocalAddress() 获取套接字绑定的本地地址。
* getInputStream()返回此套接字的输入流。
* getOutputStream()返回此套接字的输出流。
* getPort()返回此套接字连接到的远程端口。
* boolean isClosed()返回套接字的关闭状态。 
* isConnected()返回套接字的连接状态。
* setKeepAlive(boolean on)启用/禁用 SO_KEEPALIVE。

------------

##服务器客户端互相通信代码

-------------

{% highlight java %}
public class MainServer
{
	public static void main(String[] args) throws Exception
	{
		ServerSocket serverSocket = new ServerSocket(4000);
		
		while(true)
		{
			Socket socket = serverSocket.accept();
			
			//启动读写线程
			new ServerInputThread(socket).start();
			new ServerOutputThread(socket).start();
		}
	}
}

{% endhighlight %}

---------------

{% highlight java %}
public class ServerInputThread extends Thread
{
	private Socket socket;
	
	public ServerInputThread(Socket socket)
	{
		this.socket = socket;
	}
	
	@Override
	public void run()
	{
		try
		{
			InputStream is = socket.getInputStream();
			
			while(true)
			{
				byte[] buffer = new byte[1024];
				
				int length = is.read(buffer);
				
				String str = new String(buffer, 0, length);
				
				System.out.println(str);
			}
		}
		catch (IOException e)
		{
			e.printStackTrace();
		}
	}
}
{% endhighlight %}

--------------

{% highlight java %}

public class ServerOutputThread extends Thread
{
	private Socket socket;

	public ServerOutputThread(Socket socket)
	{
		this.socket = socket;
	}

	@Override
	public void run()
	{
		try
		{
			OutputStream os = socket.getOutputStream();

			while(true)
			{
				BufferedReader reader = new BufferedReader(new InputStreamReader(
						System.in));
				
				String line = reader.readLine();
				
				os.write(line.getBytes());
			}
		}
		catch (IOException e)
		{
			e.printStackTrace();
		}
	}
}
{% endhighlight %}

------------------

{% highlight java %}
public class MainClient
{
	public static void main(String[] args) throws Exception, IOException
	{
		Socket socket = new Socket("127.0.0.1", 9998);

		new ClientInputThread(socket).start();
		new ClientOutputThread(socket).start();
	}
}
{% endhighlight %}

-----------------










 
　