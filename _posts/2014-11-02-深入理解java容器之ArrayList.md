---
layout : life
title : "深入理解java容器之ArrayList"
category : 深入理解java
duoshuo: true
date : 2014-11-02
---

------
* size：用来返回Arraylist里面容纳的个数。默认构造长度为10.
* elementDate：（transient）类型，将Arraylist内容缓存进这个数组用来存放对象的引用。底层是一个数组。
* 实现的接口：List,Seriaizable,Cloneale,RandomAccess.
 * RandomAccess是一个标记接口，实现该接口表示支持快速访问，get方法比Iterator.nex访问速度快。
  * Cloneable用来指明被创建的一个允许对对象进行位复制，如果试图用一个不支持Cloneable接口的类调用clone()方法，将引发一个CloneNotSupportedException异常。没有调用被复制对象的构造函数，副本是原对象的一个简单精确的拷贝（浅复制）。
* ArrayList底层采用数组实现，当使用不带参数的构造方法生成ArrayList对象时，实际上会在底层生成一个长度为10的Object类型数组。
* 如果增加的元素个数超过10个的话，那么ArrayList底层会新生成一个数组，长度变为原来数组长度的1.5倍+1，然后将原数组的内容复制到新数组当中，然后后续增加的内容都会放在新数组中，当新数组无法容增加的元素时，重复该过程。
* 删除元素时，需要将被删除元素的后续元素向前移动，代价比较高。
* 集合当中只能放置对象的引用，无法放置原生类型，我们需要使用原生数据类型的包装类才能加入到集合中。
* 集合当中放置的是Object类型,因此取出来的也是Object类型，那么必须用强制类型转换将其为真正的类型。

------
