---
layout : life
title : "深入理解Java之IO/NIO/AIO"
category : 深入理解Java
duoshuo: true
date : 2014-11-21
---

-----------

* 面向流与面向缓冲
 * Java NIO和IO之间第一个最大的区别是，**IO是面向流的**，**NIO是面向缓冲区的**。 Java IO面向流意味着每次从流中读一个或多个字节，直至读取所有字节，它们没有被缓存在任何地方。此外，它不能前后移动流中的数据。如果需要前后移动从流中读取的数据，需要先将它缓存到一个缓冲区。 Java NIO的缓冲导向方法略有不同。数据读取到一个它稍后处理的缓冲区，需要时可在缓冲区中前后移动。这就增加了处理过程中的灵活性。但是，还需要检查是否该缓冲区中包含所有您需要处理的数据。而且，需确保当更多的数据读入缓冲区时，不要覆盖缓冲区里尚未处理的数据。
* 阻塞与非阻塞
 * 阻塞和非阻塞是进程在访问数据的时候，数据内是否准备就绪的一种处理方式,当数据没有准备的时候
 * **阻塞**：往往需要等待缓冲区中的数据准备好过后才处理其他的事情，否则一直等待在那里
 * **非阻塞**:当我们的进程访问我们的数据缓冲区的时候，数据没有准备好的时候直接返回，不需要等待.数据有的时候也直接返回.
* 同步和异步的方式: 
 * 同步和异步都是基于应用程序和操作系统处理IO时间锁采用的方式.比如同步应用程序要直接参与IO读写的操作,
 * **异步**：所有的IO读写交给操作系统去处理. 同步的方式在处理IO事件的时候，必须阻塞在某个方法上面等待我们的IO时间完成(阻塞IO事件或则通过轮询IO事件的方式),对于异步来说，所有的IO读写都交给了操作系统，这个时候，我们可以去做其他的事情，并不需要去完成真正的IO操作，当操作完成IO后，给我们的应用程序一个通知就可以的，
 * **同步**: 阻塞到IO事件阻塞到read或者write. 这个时候我们就完全不能做自己的事情.让读写方法加入到线程里面，然后阻塞线程来实现.对线程的性能开销比较大.
* IO事件的轮询  --多路复用技术(select模式)
 * 读写事件交给一个单独的线程来处理，这个完成IO事件的注册供，还有就是不断的去轮询我们的读写缓冲区，看是否有数据准备好。通知我们通知相应读写线程. 这样的话，以前的读写线程就可以做其他的事情，这个时候阻塞的不是所有的IO线程，阻塞的select这个线程.

--------------

##Java IO 模型

------------

* BIO:JDK1.4以前版本，使用BIO,**阻塞IO**。阻塞到读写方法，阻塞到线程提供性能，对于线程的开销本来就是性能的浪费。
* NIO:JDK1.4,借用了Linux 多路复用技术(select模式) 实现IO事件的轮询方式：**同步非阻塞**的模式。目前**主流的网络通信模式**。
 * Mina Netty 现在主流的网络通信框架(基于NIO)，比直接写NIO容易，代码可读性好。
* AIO：(NIO2)真正实现aio,借鉴LInux epoll模式。**异步非阻塞**

-----------

###NIO AIO原理的解读

------------
* 对于网络通信而言NIO，AIO并没有改变网通通信的基本步骤，只是在其原来的基础上(serverscoket,socket)做了一个改进.
Socket     <----建立连接需要三次握手----->           serversocket 
* 对于三次握手的方式建立稳定的连接性能开销比较大.解决方案从思想上来说比较容易,就是**减少连接的次数**.对我们的读写通信管道进行一个抽象.                 

-------------

###NIO原理

------------

* 通过selctor（选择器）就相当管家 ，管理所有的IO事件
 * Connction   
 * accept    
 * 客服端和服务端的读写.(IO事件)

* selctor（选择器）如何进行管理IO事件
 * 当IO事件注册给我们的选择器的时候,选择器会给他们分配一个key（可以简单的理解成一个时间的标签），当IO事件完成过通过key值来找到相应的管道，然后通过管道发送数据和接收数据等操作.

* 数据缓冲区：
 * 通过bytebuffer，提供很多读写的方法put()get()
* 服务端：ServerSocketChannel  
* 客户端:  SocketChannel  
* 选择器:  Selector  selector=Select.open();这样就打开了我们的选择器.
	
* Selectionkey
 * 获取事件的keys： Selectionkey  keys=  Selector.selectedkeys();
 * 可以通过它来判断IO事件是否已经就绪.
 * key.isAccptable:是否可以接受客户端的连接
 * Key.isconnctionable:是否可以连接服务端
 * Key.isreadable():缓冲区是否可读
 * Key.iswriteable():缓冲区是否可写
 
* 注册
 * channel.regist(Selector,Selectionkey.OP_Write);
 * channel.regist(Selector,Selectionkey.OP_Read);
 * channel.regist(Selector,Selectionkey.OP_Connct);
 * channel.regist(Selector,Selectionkey.OP_Accept);

---------------


--------------

###AIO
 
----------------

服务端:AsynchronousServerSocketChannel
客户端:AsynchronousSocketChannel
用户处理器:CompletionHandler接口,这个接口实现应用程序向操作系统发起IO请求,当完成后处理具体逻辑，否则做自己该做的事情.

------------------

{% highlight java %} 
    package com.shxz.nio2;  
      
    import java.io.IOException;  
    import java.net.InetSocketAddress;  
    import java.nio.ByteBuffer;  
    import java.nio.channels.SelectionKey;  
    import java.nio.channels.Selector;  
    import java.nio.channels.ServerSocketChannel;  
    import java.nio.channels.SocketChannel;  
    import java.util.Iterator;  
      
    /** 
     * NIO服务端 
     * @author 小路 
     */  
    public class NIOServer {  
        //通道管理器  
        private Selector selector;  
      
        /** 
         * 获得一个ServerSocket通道，并对该通道做一些初始化的工作 
         * @param port  绑定的端口号 
         * @throws IOException 
         */  
        public void initServer(int port) throws IOException {  
            // 获得一个ServerSocket通道  
            ServerSocketChannel serverChannel = ServerSocketChannel.open();  
            // 设置通道为非阻塞  
            serverChannel.configureBlocking(false);  
            // 将该通道对应的ServerSocket绑定到port端口  
            serverChannel.socket().bind(new InetSocketAddress(port));  
            // 获得一个通道管理器  
            this.selector = Selector.open();  
            //将通道管理器和该通道绑定，并为该通道注册SelectionKey.OP_ACCEPT事件,注册该事件后，  
            //当该事件到达时，selector.select()会返回，如果该事件没到达selector.select()会一直阻塞。  
            serverChannel.register(selector, SelectionKey.OP_ACCEPT);  
        }  
      
        /** 
         * 采用轮询的方式监听selector上是否有需要处理的事件，如果有，则进行处理 
         * @throws IOException 
         */  
        @SuppressWarnings("unchecked")  
        public void listen() throws IOException {  
            System.out.println("服务端启动成功！");  
            // 轮询访问selector  
            while (true) {  
                //当注册的事件到达时，方法返回；否则,该方法会一直阻塞  
                selector.select();  
                // 获得selector中选中的项的迭代器，选中的项为注册的事件  
                Iterator ite = this.selector.selectedKeys().iterator();  
                while (ite.hasNext()) {  
                    SelectionKey key = (SelectionKey) ite.next();  
                    // 删除已选的key,以防重复处理  
                    ite.remove();  
                    // 客户端请求连接事件  
                    if (key.isAcceptable()) {  
                        ServerSocketChannel server = (ServerSocketChannel) key  
                                .channel();  
                        // 获得和客户端连接的通道  
                        SocketChannel channel = server.accept();  
                        // 设置成非阻塞  
                        channel.configureBlocking(false);  
      
                        //在这里可以给客户端发送信息哦  
                        channel.write(ByteBuffer.wrap(new String("向客户端发送了一条信息").getBytes()));  
                        //在和客户端连接成功之后，为了可以接收到客户端的信息，需要给通道设置读的权限。  
                        channel.register(this.selector, SelectionKey.OP_READ);  
                          
                        // 获得了可读的事件  
                    } else if (key.isReadable()) {  
                            read(key);  
                    }  
                }  
            }  
        }  
        /** 
         * 处理读取客户端发来的信息 的事件 
         * @param key 
         * @throws IOException  
         */  
        public void read(SelectionKey key) throws IOException{  
            // 服务器可读取消息:得到事件发生的Socket通道  
            SocketChannel channel = (SocketChannel) key.channel();  
            // 创建读取的缓冲区  
            ByteBuffer buffer = ByteBuffer.allocate(10);  
            channel.read(buffer);  
            byte[] data = buffer.array();  
            String msg = new String(data).trim();  
            System.out.println("服务端收到信息："+msg);  
            ByteBuffer outBuffer = ByteBuffer.wrap(msg.getBytes());  
            channel.write(outBuffer);// 将消息回送给客户端  
        }  
          
        /** 
         * 启动服务端测试 
         * @throws IOException  
         */  
        public static void main(String[] args) throws IOException {  
            NIOServer server = new NIOServer();  
            server.initServer(8000);  
            server.listen();  
        }  
      
    }  
{% endhighlight %}
------------
{% highlight java %} 
    package com.shxz.nio2;  
      
    import java.io.IOException;  
import java.net.InetSocketAddress;  
import java.nio.ByteBuffer;  
import java.nio.channels.SelectionKey;  
import java.nio.channels.Selector;  
import java.nio.channels.SocketChannel;  
import java.util.Iterator;  
      
    /** 
     * NIO客户端 
     * @author 小路 
     */  
    public class NIOClient {  
        //通道管理器  
        private Selector selector;  
      
        /** 
         * 获得一个Socket通道，并对该通道做一些初始化的工作 
         * @param ip 连接的服务器的ip 
         * @param port  连接的服务器的端口号          
         * @throws IOException 
         */  
        public void initClient(String ip,int port) throws IOException {  
            // 获得一个Socket通道  
            SocketChannel channel = SocketChannel.open();  
            // 设置通道为非阻塞  
            channel.configureBlocking(false);  
            // 获得一个通道管理器  
            this.selector = Selector.open();  
              
            // 客户端连接服务器,其实方法执行并没有实现连接，需要在listen（）方法中调  
            //用channel.finishConnect();才能完成连接  
            channel.connect(new InetSocketAddress(ip,port));  
            //将通道管理器和该通道绑定，并为该通道注册SelectionKey.OP_CONNECT事件。  
            channel.register(selector, SelectionKey.OP_CONNECT);  
        }  
      
        /** 
         * 采用轮询的方式监听selector上是否有需要处理的事件，如果有，则进行处理 
         * @throws IOException 
         */  
        @SuppressWarnings("unchecked")  
        public void listen() throws IOException {  
            // 轮询访问selector  
            while (true) {  
                selector.select();  
                // 获得selector中选中的项的迭代器  
                Iterator ite = this.selector.selectedKeys().iterator();  
                while (ite.hasNext()) {  
                    SelectionKey key = (SelectionKey) ite.next();  
                    // 删除已选的key,以防重复处理  
                    ite.remove();  
                    // 连接事件发生  
                    if (key.isConnectable()) {  
                        SocketChannel channel = (SocketChannel) key  
                                .channel();  
                        // 如果正在连接，则完成连接  
                        if(channel.isConnectionPending()){  
                            channel.finishConnect();  
                              
                        }  
                        // 设置成非阻塞  
                        channel.configureBlocking(false);  
      
                        //在这里可以给服务端发送信息哦  
                        channel.write(ByteBuffer.wrap(new String("向服务端发送了一条信息").getBytes()));  
                        //在和服务端连接成功之后，为了可以接收到服务端的信息，需要给通道设置读的权限。  
                        channel.register(this.selector, SelectionKey.OP_READ);  
                          
                        // 获得了可读的事件  
                    } else if (key.isReadable()) {  
                            read(key);  
                    }  
      
                }  
      
            }  
        }  
        /** 
         * 处理读取服务端发来的信息 的事件 
         * @param key 
         * @throws IOException  
         */  
        public void read(SelectionKey key) throws IOException{  
        	  //和服务端的read方法一样  
            SocketChannel channel = (SocketChannel) key.channel();  
            // 创建读取的缓冲区  
            ByteBuffer buffer = ByteBuffer.allocate(10);  
            channel.read(buffer);  
            byte[] data = buffer.array();  
            String msg = new String(data).trim();  
            System.out.println("客户端端收到信息："+msg);  
            ByteBuffer outBuffer = ByteBuffer.wrap(msg.getBytes());  
            channel.write(outBuffer);// 将消息回送给客户端  
          
        }  
        /** 
         * 启动客户端测试 
         * @throws IOException  
         */  
        public static void main(String[] args) throws IOException {  
            NIOClient client = new NIOClient();  
            client.initClient("localhost",8000);  
            client.listen();  
        }  
    }  
{% endhighlight %}
------------


