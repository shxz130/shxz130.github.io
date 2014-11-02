---
layout : life
title : "深入理解java容器之LinkList"
category : 深入理解java
duoshuo: true
date : 2014-11-02
---

------
* **equals**
 * 自反性：x.equals(x)应该返回true.
 * 对称性：x.equals(y)为true，则y.equals(x)为true.
 * 传递性：x.equals(y)为true，y.equals(z)为true.则x.equals(z)为true.
 * 一致性：x.equals(y)为true，那么x.equals(y)的第二次，第三次,第n次调用也应该为true.前提条件是在比较之间没有修改x,y。
  * 对于非空引用x,x.equals(null)为false。
* **hashcode**
 * 在java应用的一次执行过程中，对于同一个对象的hashcode方法的多次调用，返回相同的值。
 * 对于两个对象而言，如果使用equals方法比较返回false，那么这两个对象的hashCode值不要求一定不同，可以相同也可以不同。但是如果不同可以提高效率。
 * 对于Object类来说，不同的Object对象的hashCode值是不同的(Object类的hashCode值表示是对象的地址)。
 * 当使用HashSet时，hashCode()方法就会得到调用，判断已经存储在集合中的对象的hash code值是否与增加的对象的hash code值是否一致，如果不一致，直接加进去，如果一致，再进行equals方法的比较，equals方法如果返回true，表示对象已经加进去了，就不会再增加新的对象。否则加进去。
 * 如果我们重写equals方法，也就要重写hashCode()方法，主要在容器中用的。
--------

