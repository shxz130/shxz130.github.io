---
layout : life
title : "深入理解java容器之LinkList"
category : 深入理解java
duoshuo: true
date : 2014-11-02
---

------
* 继承于：AbstractSequentialList<E>
* 实现的接口：List,Seriaizable,Cloneale,Deque<E>.
 * RandomAccess是一个标记接口，实现该接口表示支持快速访问，get方法比Iterator.nex访问速度快。
  * Cloneable用来指明被创建的一个允许对对象进行位复制，如果试图用一个不支持Cloneable接口的类调用clone()方法，将引发一个CloneNotSupportedException异常。没有调用被复制对象的构造函数，副本是原对象的一个简单精确的拷贝（浅复制）。
   * Deque接口是一个双端队列。支持两端插入和移除元素。
* LinkList底层使用双向链表去实现的。里面有一个一个内部类Entry。
``` 
    private class Entry<E e>
    {
        Entry previous;
        Object Elements;
        Entry Next;
    }
```
------
