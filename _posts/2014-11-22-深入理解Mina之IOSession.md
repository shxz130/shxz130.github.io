---
layout : life
title : "深入理解Mina之IOSession/IOBuffer"
category : 深入理解Mina
duoshuo: true
date : 2014-11-22
---

----------

* **IOSession**：
 * 主要描述我们的网络通信双方所建立的连接之间描述.IOSession的作用:可以完成对于连接的一些管理.可以发送或则读取数据，并且可以设置我们会话的 上下文信息.
* **IOSessionConfig**:
 * 提供我们对连接的配置信息的描述.比如读缓冲区的设置等等.IOSessionConfig，设置读写缓冲区的一些信息,读和写的空闲时间，以及设置读写超时信息.
* **IOSession**：
 * **getAttribute(Object key)**  根据key获得设置的上下文属性.
 * **setAttribute(Object key, Object value)**  设置上下文属性
 * **removeAttribute(Object key)**    删除上下文属性
 * **write(Object message)**  发送数据
 * **read()** 读取数据
* **IOSessionConfig**
 * **getBothIdleTime()**  获得读写通用的空闲时间
 * **setIdleTime(IdleStatus status, int idleTime)**   设置我们的读或则写的空闲时间.
 * **setReadBufferSize(int readBufferSize)**   设置读缓冲区大小
 * **setWriteTimeout(int writeTimeout)**   设置我们的写超时时间
* **Processor**：
 * 是以NIO为基础实现的以多线程的方式来完成我们读写工作.Processor的作用,是为我们的filter读写原始数据的多线程环境，如果mina不去实现的话 ，我们自己来实现NIo的话，需要自己写一个非阻塞读写的多线程的环境.
* 配置Processor的多线程环境.
 * 通过**NioSocketAcceptor(int processorCount)** 构造函数可以指定多线程的个数.
 * 通过**NioSocketConnector(int processorCount)** 构造函数也可以指定多线程的个数.

---------------

##IOBuffer

------------------

* 基于javaNio中的**ByteBuffer**做了封装，用户操作缓冲区中的数据，包括基本数据类型以及字节数组和一些对象.其本质就是一个可动扩展的byte数组.
* 索引属性
 * capacity：代表当前缓冲区的大小
 * position:理解成当前读写位置，也可以理解成下一个可读数据单位的位置.Position<=Capacity的时候可以完成数据的读写操作.
 * limit:就是下一个不可以被读写缓冲区单元的位置.Limit<=Capacity.

![gengsufa](/life/picture/iobuffer.jpg)
 
* IOBuffer常用api:
 * static   allocate(int capacity)   已指定的大小开辟缓冲区的空间.
 * setAutoExpand(boolean autoExpand)   可以设置是否支持动态的扩展.
 * putShort(int index, short value) 
 * putString(CharSequence val, CharsetEncoder encoder)
 * putInt(int value) 等等方法实现让缓冲区中放入数据.  PutXXX();
 * flip:就是让我们的limit=position，position=0；为我们读取缓冲区的数据做好准备，因为有时候，limit！=position，一般在发送数据之前调用.
 * hasRemaining() :缓冲区中是否有数据：boolean是关于position<=limit=true,否则返回fals
 * remaining():返回的是缓冲区中可读数据的大小,limit-position的值.
 * Rest和clear
	* reset() ：实现清空数据.
	* Clear():实现数据的覆盖.position=0;重新开始读我们缓冲区的数据.

-----------


