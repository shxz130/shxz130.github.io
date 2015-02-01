---
layout : life
title : "深入理解SpringMVC框架"
category : 深入理解之java Web框架
duoshuo: true
date : 2015-02-01
---

![gengsuanfa](/life/picture/springmvc.jpg)

---------------

###SpringMVC中三个用户必须定义和扩展的组件
 
--------------

* **HandlerMapping**: 定义URL的映射规则
 * 首先用户发送请求到web容器，web容器根据路径映射到DispatcherServlet（url-pattern为/）进行处理。
 * DispatcherServlet->HandlerMapping进行请求到处理的映射,会映射到controller层。
 * 注册和查找。在HandlerMapping的实现中，持有一个handlerMap这样一个HashMap<String, Object>，其中key是http请求的path信息，value可以是一个字符串，或者是一个处理请求的HandlerExecutionChain，如果是String类型，则会将其视为Spring的bean名称。在HandlerMapping对象的创建中，IoC容器执行了一个容器回调方法setApplicationContext，在这个方法中调用initApplicationContext方法进行初始化，各个子类可以根据需求的不同覆写这个方法。关于handlerMap信息的注册就是在initApplicationContext方法中被执行的。
* **HandlerAdapter**:处理请求的映射 
 * 实现业务逻辑的Handler实例对象 
 * 处理器功能处理方法的调用，HandlerAdapter将会根据适配的结果调用真正的处理器的功能处理方法，完成功能处理；并返回一个ModelAndView对象（包含模型数据、逻辑视图名)ModelAndView 逻辑视图名
* **ViewResolver**
 * ViewResolver将把逻辑视图名解析为具体的View，通过这种策略模式，很容易更换其他视图技术  
 * View,渲染，View会根据传进来的Model模型数据进行渲染，此处的Model实际是一个Map数据结构，因此很容易支持其他视图技术；

---------------
