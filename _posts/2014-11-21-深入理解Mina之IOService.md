---
layout : life
title : "深入理解Mina之IoService"
category : 深入理解Mina
duoshuo: true
date : 2014-11-21
---

-----------
* IoService提供了一个I/O服务和管理I/Osession的一个功能，是mina框架最重要的一个部分,为实现了IoService的类或接口提供了一个很强大的功能用于业务操作。
* IoService接口提供了一些连接常用接口，在server端IoAcceptor继承了IoService，在client端 IoConnector 继承了IoService 完成客户端功能，IoService是创建服务的顶层接口，无论客户端还是服务端，都是从它继承实现的。

-----------

##IoService的功能

-----------

* session 管理 ： 创建、删除session、监测session 是否失效
* 过滤器链管理 ：管理过滤器连，并且用户可以很方便的自定义创建过滤器
* 调用业务handler：在消息接收完成之后，调用业务handler进行处理
* 统计管理： 统计消息发送量(发送对象、字节....)
* 监听网络连接：一直监听绑定端口是否有新的链接
* 数据传输： 管理 server端和 client 数据传输
IoService是IoConnector's 和 IoAcceptor's的父接口，他所定义的方法都是和I/O操作息息相关的
-------------
 
![gengsufa](/life/picture/mina3.png)

-----------

##IoService的使用方法

----------------

* void addListener(IoServiceListener listener)
 * 这个方法可以为IoService 增加一个监听器，用于监听IoService 的创建、活动、失效、空闲、销毁，具体可以参考IoServiceListener 接口中的方法，这为你参与IoService 的生命周期提供了机会。  
* void removeListener(IoServiceListener listener)
 * 这个方法用于移除addListener的方法添加的监听器。
* boolean isDisposing()
 * 返回isDisposed()的状态，当且仅当 isDisposed() 方法被调用完毕才返回TRUE
* boolean isDisposed()
 * 返回service的状态，当且仅当service 的当前进程的所有资源已经释放完毕才返回 TRUE
* void dispose()
 * 这个方法用于释放service分配的资源，他可能要花费一些时间，用户应该调用 isDisposing() 和 isDisposed() 判断资源是否释放完成，当一个service被关闭的时候都应该调用该方法来进行资源释放
* IoHandler getHandler()
 * 返回当前进程serbice关联的handler
* void setHandler(IoHandler handler)
 * 这个方法用于向IoService 注册IoHandler，同时有getHandler()方法获取Handler
* Map<Long,IoSession> getManagedSessions()
 * 这个方法获取IoService 上管理的所有IoSession，Map 的key 是IoSession 的id。
* int getManagedSessionCount()
 * 返回当前service 上绑定的session数量
* IoSessionConfig getSessionConfig()
 * 这个方法用于获取IoSession 的配置对象，通过IoSessionConfig 对象可以设置Socket 连接的一些选项。
* IoFilterChainBuilder getFilterChainBuilder()
 * 返回当前service的拦截器连，用户可以在改链上添加自己的过滤器，在service绑定之前
* void setFilterChainBuilder(IoFilterChainBuilder builder)
 * 定义service的拦截器链
* DefaultIoFilterChainBuilder getFilterChain()
 * 返回当前service默认的那个filterchain，是getFilterChainBuilder() 快捷方式
* boolean isActive()
 * 当前service是否在活动
* long getActivationTime()
 * 返回这个service 被激活现在的时间，也就是service存活了多久.  如果这个service 不是活动状态咋返回它最好一次活动的时间
* Set<WriteFuture>broadcast(Object message)
 * 向所有注册了的session 广播消息
* void setSessionDataStructureFactory(IoSessionDataStructureFactory sessionDataStructureFactory)
 * 向新注册的service 放一些初始化的数据
* int getScheduledWriteMessages()
 * 返回信息数量(这里的信息时在内存等待socket向外写的)
* IoServiceStatistics getStatistics()
 * 返回service的 IoServiceStatistics 对象.
 
--------------

##IoService的实现

----------------
* IoService 在mina中时一个很重要的接口，同时呢它又被两个接口所继承，分别是 IoAcceptor 和 IoConnector，分别代表了 服务端和客户端的连接，所有我们在建立server和client的时候必须要实现两个接口或者是继承他们的实现类或适配器类

---------------

##IoAcceptor

----------------
* IoAcceptor主要负责在server和client之间建立一个新的连接或者是接收client发送的请求，据说在mina3.0 中会将这个接口命名为 Server。有时候我们可能不是基于某一种协议开发我们的应用，可能会用到多种协议开发(TCP/UDP...),那么这时候mina就为我们提供了很多其他的一些实现，用于不用的需求

* **IoAcceptor** 有如下几个实现类
 * NioSocketAcceptor : 非阻塞的server端的Socket
 * NioDatagramAcceptor: 非阻塞的server端的Socket  (基于UDP协议)
 * AprScoketAcceptor  : 基于APR阻塞式的socket
 * VmPipeScoketAcceptor  :基于管道的Socket
* 常用方法
 * **connect(SocketAddress remoteAddress):用于用户发起一个请求**
 * **setConnectTimeout(int connectTimeout)：连接超时的设置**
 
* 关系图

![gengsuanfa](/life/picture/mina4.png)

--------------


##IoConnector

--------------

* IoConnector 是用于client端的一些方法定义，在写client端的时候必须实现改接口或者其实现类 
* **实现类** 
 * NioScoketConnector : 非阻塞的client端的Socket
 * NioDatagramConnector: 基于UDP协议 非阻塞的client端的Socket
 * AprScoketConnector :基于Apr的scoket
 * ProxyConnector :提供代理的scoket
 * SerialScoket : 提供多个端口连接的scoket
 * VmPipeConnector:基于管道的socket

* 关系图

![gengsuanfa](/life/picture/mina5.png)

---------------

