---
layout : life
title : "深入理解Mina之IOFilter"
category : 深入理解Mina
duoshuo: true
date : 2014-11-21
---

-------------

* 对应用程序和网络这块的传输,就是二进制数据和对象之间的相互转化,有相应的解码和编码器。属于过滤器一种。
* 过滤器还可以做日志。消息确认等功能。
* 在应用层和我们业务层之家过滤层
--------------

* **类结构图**
 
--------------

![gengsuanfa](/life/picture/filter.png)

--------------

* IoFilterAdpater：该类提供了IoFilter所有方法的方法体，但是没有任何逻辑处理，你可以根据你具体的需求继承该类，并重写相关的方法。IoFilterAdpater是在过滤器中使用的较多的一个类。
* ReferenceCountingIoFilter:该类封装IoFilter的实例，它使用监视使用该IoFilter的对象的数量，当没有任何对象使用该IoFilter时，该类会销毁该IoFilter。
* IoFilterAdpater有三个子类，它们的作用分别如下：
 * LoggingFilter:日志工具，该类处理记录IoFilter每个状态触发时的日志信息外不对数据做任何处理。它实现了IoFilter接口的所有方法。你可以通过阅读该类的源码学习如何实现你自己的IoFilter。
 * ExcuterFilter:这个Mina自身提供的一个线程池，在 Mina中你可以使用这个类配置你自己的线程池，由于创建和销毁一个线程，需要耗费很多资源，特别是在高性能的程序中这点尤其重要，因此在你的程序中配置一个线程池是很重要的。它有助于你提高你的应用程序的性能。关于配置Mina的线程池在后续的文档中会给出详细的配置方法。
 * ProtocolFilter:该类是Mina提供的一个协议编解码器，在socket通信中最重要的就是协议的编码和解码工作，Mina提供了几个默认的编解码器的实现，在下面的例子中使用了 ObjectSerializationCodecFactory，这是Mina提供的一个Java对象的序列化和反序列化方法。使用这个编解码器，你可以在你的Java客户端和服务器之间传递任何类型的Java对象。但是对于不同的平台之间的数据传递需要自己定义编解码器。

--------------

##IoFilter API

------------------

* init() filter初始化方法，在下面4个on方法调用之前必须先被调用。
* destroy() 该方法被ReferenceCountingFilter引用到，如果不继承ReferenceCountingFilter，不需实现该方法。
* onPreAdd(IoFilterChain, String, NextFilter) 过滤器被添加之前调用。
* onPostAdd(IoFilterChain, String, NextFilter) 过滤器被添加之后调用。
* onPreRemove(IoFilterChain, String, NextFilter) 过滤器被移除之前调用。
* onPostRemove(IoFilterChain, String, NextFilter) 过滤器被移除之后调用。
* sessionCreated(NextFilter, IoSession) 
* sessionOpened(NextFilter, IoSession)
* sessionClosed(NextFilter, IoSession)
* sessionIdle(NextFilter, IoSession, IdleStatus)
* exceptionCaught(NextFilter, IoSession, Throwable)
* messageReceived(NextFilter, IoSession, Object)
* messageSent(NextFilter, IoSession, WriteRequest)
* filterClose(NextFilter, IoSession)  session.close(）方法中调用。
* filterWrite(NextFilter, IoSession, WriteRequest) session.write(）方法中调用。
 
-----------------