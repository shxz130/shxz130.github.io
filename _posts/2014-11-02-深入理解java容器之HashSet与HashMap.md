---
layout : life
title : "深入理解java容器之HashMap与HashSet"
category : 深入理解java
duoshuo: true
date : 2014-11-02
---

-----
map的**keyset()**方法会返回key的集合，map的键不能重复，所以**keyset()**方法，而map的值是可以重复的，所以**values()**方法可以容纳重复的元素。所以values()返回类型是Collection，可以容纳重复的元素。

-------
##遍历map:
-------

* 取得keyset的值，然后遍历keySet得到对应的value。
* **entrySet()**方法，获取集合，遍历得到。集合里面实质是Map.Entry(内部类)类型，getKey(),getValue()方法。

{% highlight java %}

public static void main(String[] args)
{
	Map hash=new HashMap();
	hash.put("1", "kewew");
	hash.put("2", "value");
	Set set=hash.entrySet();						   
	Iterator iterator=set.iterator();						
	while(iterator.hasNext())			
	{										
		Map.Entry entry=(Entry) iterator.next();				    
		System.out.println(entry.getValue());				     
	}
}
{% endhighlight %}
--------
##HashSet的实现
* 底层是用HashMap实现的。
* 当使用add方法时，实际上是将该对象作为底层做维护的Map对象的key，而value的值作为同一个Object对象来代替。
* iterator()方法是返回keySet()集合。

---------
##HashMap的实现
-----------

* HashMap底层维护着一个数组Entry，初始化长度为1<<4也就是16的长度。每个Entry有一个key ,一个value的值，有一个类型为Entry的对象next的引用,还有一个**负载因子**（hash值，默认为0.75L）。当HashMap中数据已经超过初始定义的负载限额时，会重新做一次哈希也就是说再散列一次，重新生成一个更大的数组（原数组的**二倍**）。

* 当我们向HashMap中put一个键值时，它会根据key的hashCode值计算出一个位置，该位置就是此对象准备往数组中存放的位置。

* 如果该位置没有对象存在，就此对象直接存放在该位置。如果有对象存在了，则数组顺着此存在的对象的链，开始寻找（Entry类型的next成员变量，指向类该对象的下一个对象)。如果词链上有对象的话，使用equals方法进行比较，如果此链上的某个对象的equals方法比较为false，则将该对象放咋数组中，然后将该位置以前存放的那个对象链接到此对象的后面。 

------------
