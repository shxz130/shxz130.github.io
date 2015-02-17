---
layout : life
title : "深入理解Session与Cookie"
category : 浅谈http协议
duoshuo: true
date : 2015-01-25
---

###主要存在的问题：

--------------

* 当系统大到需要很多Cookie的时候，需要考虑Http协议对Cookie数量和大小的限制。
* 当我们的应用系统有几百台服务器的时候，如何解决Session共享问题。
* 安全问题：Cookie被盗，COokie伪造。
* **Session与Cookie的作用:保持访问用户和后端服务器的交互状态**

---------------

###理解Cookie

-----------------------

* 用户通过Http协议访问一个服务器，服务器会将一些key/value键值对返回给客户端浏览器，并将数据添加一些限制条件，在条件符合时，该用户下次访问原服务器时，数据会被完整的带回到服务器。
* http协议是一种无状态协议，当用户的一次访问请求结束后，后端服务器无从判断下次来访的是不是上次来访的用户。对于两次访问是不是同一人对于程序的设计和性能有很大的不同。如果与用户相关的数据被频繁的访问，可以针对该数据做缓冲，可以提高数据的访问性能。

-----------------------

###Cookie的常见属性：

--------------------

|   属性   |        属性项介绍                          |
| -------- |:------------------------------------------:|
|  domain  | 生成该cookie的域名                         | 
|  path    | 生成路径                                   |  
|  secure  | 如果设置了，只会在ssh连接时才会回传该cookie|
 
---------------------

* 如何将cookie添加到Http的Header中 
 * tomcat调用addCookie方法
  * HttpServletResponse.addCookie().addCookieInternal().addHeader()。
  * Headers.addValue(name).setString(value)。设置值。
* 当我们每次通过response创建多个Cookie时，cookie最终是在一个Header项中还是以独立的Header存在的，也就是说我们每次创建的Cookie都是创建一个header。
* 从客户端获取cookie信息
 * 当我们请求某个Url路径将符合条件的Cookie信息放在Request请求头部传回到服务端，服务端通过Request.getCookies()来取得所有Cookie。

-----------------------

* 下图为淘宝网的cookies信息
![gengsuanfa](/life/picture/cookies.jpg)

--------------------

* cookie的限制 ：浏览器一般为50个域名

---------------------

|   浏览器版本   | Cookie数限制    | cookie总大小限制      |
| -------------- |:---------------:|----------------------:|
|  ie7           | 20个/每个域名   | 4095个字节            | 
|  ie9           | 50个/每个域名   | 4095个字节            |  
|  chrome        | 50个/每个域名   | 大于80000			   |
|  FireFox       | 50个/每个域名   | 4095个字节 		   |

------------

###深入理解Session

-------------

* cookie可以让服务端程序跟踪每个客户端的访问，但是每次客户端的访问都必须传回这些cookie,如果cookie很多，无形的增加了客户端和服务器端的数据传输量，而session正是解决此问题。
* 同一个客户端每次与服务端交互时，不需要每次都传回所有的Cookie，而是只要传一个ID，而ID是客户端第一次访问服务端时生成的，而且是唯一的，每个客户端只有一个唯一的ID，客户端只要传回这个ID就好了，这个ID通常Name为JSESIONID的一个Cookie.
* **如何让Session基于Cookie来工作**。
 * 基于URL Path Parameter.
 * 基于Cookie，如果没有修改Context容器的cookies标识。
 * 基于SSL，默认不支持。
* **session如何工作**
 * request.getSession()方法，如果当前的SessionId没有对应的HttpSession对象，那么就创建一个新的，并添加到org.apache.catalina.Manager的session容器中保存。

--------------------

###表单重复提交

--------------------

* 为了防止表单重复提交，需要标识用户的每一次访问，使得每一次访问对服务端来说都是唯一确定的，为了标识每一次访问，需要请求一个表单域的时候增加一个隐藏表单项，填写唯一的token，用户在发出请求的时候，生成token，将token存放在session中，用户提交的时候检查这个token和当前的session中保存的token是否一致，如果一致说明没有重复提交，如果不一致就判断为重复提交。

------------------

###总结

------------------

* Cookie和Session都是为了保持用户访问的连续状态，之所以要保持这种状态，一方面为了方便业务实现，另一方面为了简化服务端程序设计，提高访问性能。

