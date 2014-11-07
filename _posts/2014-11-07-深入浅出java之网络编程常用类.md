---
layout : life
title : "深入浅出java网络编程之常用类URL/InetAddress"
category : 深入理解java
duoshuo: true
date : 2014-11-07
---
------------

因为自己本人是学网络工程的，所以在网络编程基础知识这块不在这里做介绍，后面将会在计算机网络知识模块中介绍，在这里只是简单的介绍介绍常用的类和方法。

--------------

##java.net.URL

--------------

* 构造方法：
 * URL ( String url)//url代表一个绝对地址，URL对象直接指向这个资源，如：
 * URL ( URL baseURL , String relativeURL)//　其中，baseURL代表绝对地址，relativeURL代表相对地址。如：
 * URL ( String protocol , String host , String file)//其中，protocol代表通信协议，host代表主机名，file代表文件名。
 * URL ( String protocol , String host , int port , String file)//protocol代表通信协议，host代表主机名，port代表端口号，file代表文件名。
* 获取URL属性
 * getDefaultPort()： 返回默认的**端口号**。
 * getFile()： 获得URL指定资源的**完整文件名**。
 * getHost()： 返回**主机名**。
 * getPath()： 返回指定资源的**文件目录和文件名**。
 * getPort()： 返回端口号，默认为**-1**。
 * getProtocol()： 返回表示URL中**协议的字符串对象**。
 * getRef()： 返回URL中的HTML文档标记，即#号标记。
 * getUserInfo： 返回**用户信息**。
 * toString： 返回完整的URL字符串。
* 常用方法：
 * **openConnection()**：打开一个I/O的connection连接，方便I/O操作。
 * **getContent()**：方法来获取资源的内容，返回内容为Object类型，通过openConnection().getContent()**源码得知**;

--------------

##java.net.InetAddress

--------------

InetAddress类没有提供返回构造函数，所以不能用new()方法来创建它的对象，而只可以调用静态方法getLocalHost()、getByName()、getByAddress()等来生成InetAddress类的实质。

--------------

* getAddress()： 返回IP地址的字节形式。
* getAllByName()： 返回指定主机名的IP地址。
* getbyAddress()： 返回指定字节数组的IP地址形式。
* getByName()： 返回指定主机名的IP地址对象。
* getHostAddress()： 返回主机地址的字符串形式。
* getLocalHost()： 返回当前主机名。
* hashCode()： 返回InetAddress对象的哈希码。
* toString()： 返回地址转换成的字符串。

------------












 
　