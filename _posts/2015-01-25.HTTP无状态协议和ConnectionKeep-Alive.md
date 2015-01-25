---
layout : life
title : "HTTP无状态协议和ConnectionKeep-Alive"
category : 深入理解java Web
duoshuo: true
date : 2015-01-25
---

###HTTP无状态

--------------

* 无状态是指协议对于事务处理没有记忆能力，服务器不知道客户端是什么状态。从另一方面讲，打开一个服务器上的网页和你之前打开这个服务器上的网页之间没有任何联系。
* 如果你要实现一个购物车，需要借助于Cookie或Session或服务器端API（如NSAPI and ISAPI）记录这些信息，请求服务器结算页面时同时将这些信息提交到服务器 
* 当你登录到一个网站时，你的登录状态也是由Cookie或Session来“记忆”的，因为服务器并不知道你是否登录。
* **优点：**服务器不用为每个客户端连接分配内存来记忆大量状态，也不用在客户端失去连接时去清理内存，以更高效地去处理WEB业务。
* **缺点：**客户端的每次请求都需要携带相应参数，服务器需要处理这些参数。 

--------------

###Keep-Alive

-------------

* HTTP是一个无状态的面向连接的协议，无状态不代表HTTP不能保持TCP连接，更不能代表HTTP使用的是UDP协议（无连接） 
* 从HTTP/1.1起，默认都开启了Keep-Alive，保持连接特性，简单地说，当一个网页打开完成后，客户端和服务器之间用于传输HTTP数据的TCP连接不会关闭，如果客户端再次访问这个服务器上的网页，会继续使用这一条已经建立的连接 
* Keep-Alive不会永久保持连接，它有一个保持时间，可以在不同的服务器软件（如Apache）中设定这个时间 
* **什么情况下该开启Keep-Alive?**
 * 在网页开发过程中，Keep-Alive是HTTP协议中非常重要的一个属性。大家知道HTTP构建在TCP之上。在HTTP早期实现中，每个 HTTP请求都要打开一个socket连接。这种做效率很低，因为一个Web 页面中的很多HTTP请求都指向同一个服务器。例如，很多为Web页面中的图片发起的请求都指向一个通用的图片服务器。持久连接的引入解决了多对已请求服务器导致的socket连接低效性的问题。它使浏览器可以再一个单独的连接上进行多个请求。浏览器和服务器使用Connection头来指出对 Keep-Alive的支持。  
* 笔者在去年遇到一个跟Keep-Alive的问题： 
* **问题现象**： 一个JSP页面，居然要耗时40多秒。网页中有大量的图片的CSS 
* **问题解决**： 原因也找了半天，原来Apache配置里面，把Keep-Alive的开关关闭了。这个是个大问题，工程师为什么要关闭它，原来他考虑的太简单了，我们知 道Apache适合处于短连接的请求，处理时间越短，并发数才能上去，原来他是这么考虑，但是没有办法，只能这样了，还是打开Keep-Alive开关 吧。 
* 当然，不是所有的情况都设置KeepAlive为On，下面的文字总结比较好： 
* 在使用apache的过程中，KeepAlive属性我一直保持为默认值On，其实，该属性设置为On还是Off还是要具体问题具体分析的，在生产环境中的影响还是蛮大的。
* **KeepAlive选项到底有什么用处？**如果你用过Mysql ，应该知道Mysql的连接属性中有一个与KeepAlive 类似的Persistent Connection，即：长连接(PConnect)。该属性打开的话，可以使一次TCP连接为同一用户的多次请求服务，提高了响应速度。比如很多网页中图片、CSS、JS、Html都在一台Server上，当用户访问其中的Html网页时，网页中的图片、Css、Js都构成了访问请求，打开KeepAlive 属性可以有效地降低TCP握手的次数(当然浏览器对同一域下同时请求的图片数有限制，一般是2)，减少httpd进程数，从而降低内存的使用(假定prefork模式)。MaxKeepAliveRequests 和KeepAliveTimeOut 两个属性在KeepAlive =On时起作用，可以控制持久连接的生存时间和最大服务请求数。 
不过，上面说的只是一种情形，那就是静态网页居多的情况下，并且网页中的其他请求与网页在同一台Server上。当你的应用动态程序(比如：php )居多，用户访问时由动态程序即时生成html内容，html内容中图片素材和Css、Js等比较少或者散列在其他Server上时，KeepAlive =On反而会降低Apache 的性能。为什么呢？前面提到过，KeepAlive =On时，每次用户访问，打开一个TCP连接，Apache 都会保持该连接一段时间，以便该连接能连续为同一client服务，在KeepAliveTimeOut还没到期并且MaxKeepAliveRequests还没到阈值之前，Apache 必然要有一个httpd进程来维持该连接，httpd进程不是廉价的，他要消耗内存和CPU时间片的。假如当前Apache 每秒响应100个用户访问，KeepAliveTimeOut=5，此时httpd进程数就是100*5=500个(prefork 模式)，一个httpd进程消耗5M内存的话，就是500*5M=2500M=2.5G，夸张吧？当然，Apache 与Client只进行了100次TCP连接。如果你的内存够大，系统负载不会太高，如果你的内存小于2.5G，就会用到Swap，频繁的Swap切换会加重CPU的Load。 
现在我们关掉KeepAlive ，Apache 仍然每秒响应100个用户访问，因为我们将图片、js、css等分离出去了，每次访问只有1个request，此时httpd的进程数是100*1=100个，使用内存100*5M=500M，此时Apache 与Client也是进行了100次TCP连接。性能却提升了太多。 
* **总结**： 
 * 当你的Server内存充足时，KeepAlive =On还是Off对系统性能影响不大。 
 * 当你的Server上静态网页(Html、图片、Css、Js)居多时，建议打开KeepAlive 。 
 * 当你的Server多为动态请求(因为连接数据库，对文件系统访问较多)，KeepAlive 关掉，会节省一定的内存，节省的内存正好可以作为文件系统的Cache(vmstat命令中cache一列)，降低I/O压力。 
PS：当KeepAlive =On时，KeepAliveTimeOut的设置其实也是一个问题，设置的过短，会导致Apache 频繁建立连接，给Cpu造成压力，设置的过长，系统中就会堆积无用的Http连接，消耗掉大量内存，具体设置多少，可以进行不断的调节，因你的网站浏览和服务器配置 而异。

------------------

[转自推酷](http://www.tuicool.com/articles/eAnQZn)
