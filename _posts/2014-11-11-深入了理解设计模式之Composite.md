---
layout : life
title : "深入理解设计模式之组合模式"
category : 深入理解设计模式
duoshuo: true
date : 2014-11-11
---

##组合模式

-------------

* **定义**：又叫做**部分-整体模式**，一般用来处理类似树形结构的问题。它模糊处理元素与部分整体，使得客户端能够统一处理简单或者复杂的元素，从而使得客户端程序同元素的结构之间解耦。 
* **作用**：
 * 统一元素与部分整体，简化处理代码
 * 将元素内部结构同处理程序解耦，从而一致的对待元素与部分整体。
* 实际上，组合模式在应用中其实非常广泛，像文件系统、企业结构等都可以看做是组合模式的典型应用。

* **应用环境**
 * 在对象与部分整体之间，想要通过统一的方式对其进行处理，模糊处理其差异的时候可以选用组合模式
 * 当客户端忽视结构层次，无差异的看待元素与部分整体，不关心元素和部分整体之间的层次结构，想要实现对统一接口编程的时候
 * 对象的变化是动态，而客户端想要一致的处理对象的时候
综上，在上述3种情况下可以考虑使用组合模式来设计系统程序。

-------------

##案例分析：
一个公司有分公司也有部门，分公司下面也有公司和部门。
![onepiece](/life/picture/zuhemoshi.png)

{% highlight java %}
package com.shxz.Composite;

public abstract class GongSi {
	protected String name;
	public GongSi(String name)
	{
		this.name=name;
	}
	public void ShowName()
	{
		System.out.println(name);
	}
	public abstract void add(GongSi gongSi);
	public abstract void remove(GongSi gongSi);
	public abstract void iterate();
}
{% endhighlight %}
--------

{% highlight java %}
package com.shxz.Composite;

import java.util.ArrayList;
import java.util.Iterator;

public class FenGongSi extends GongSi{

	private ArrayList<GongSi> list=new ArrayList<GongSi>();
	public FenGongSi(String name) {
		super(name);
		// TODO Auto-generated constructor stub
	}

	@Override
	public void add(GongSi zongGongSi) {
		// TODO Auto-generated method stub
		list.add(zongGongSi);
	}
	@Override
	public void remove(GongSi gongSi) {
		// TODO Auto-generated method stub
		list.remove(gongSi);
	}

	@Override
	public void iterate() {
		// TODO Auto-generated method stub
		System.out.println(name);
		  for (GongSi c : list) {  
             c.iterate();
          }  
	}
}

{% endhighlight %}
--------

{% highlight java %}
package com.shxz.Composite;

public class BuMen extends  GongSi{	
	public BuMen(String name) {
		super(name);
		// TODO Auto-generated constructor stub
	}

	@Override
	public void add(GongSi gongSi) {
		// TODO Auto-generated method stub
	}

	@Override
	public void remove(GongSi gongSi) {
		// TODO Auto-generated method stub
	}

	@Override
	public void iterate() {
		// TODO Auto-generated method stub
		System.out.println(name);
	}

}

{% endhighlight %}
--------

{% highlight java %}
package com.shxz.Composite;

public class Text {
	public static void main(String[] args) {
		FenGongSi fenGongSi=new FenGongSi("总公司");
		fenGongSi.add(new BuMen("总公司下部门1"));
		fenGongSi.add(new BuMen("总公司下部门2"));
		FenGongSi fengongsi1=new FenGongSi("总公司下分公司1");
		fenGongSi.add(fengongsi1);
		FenGongSi fengongsi2=new FenGongSi("总公司下分公司2");
		fenGongSi.add(fengongsi2);
		
		FenGongSi fengongsi11=new FenGongSi("总公司下分公司1分部门1");
		fengongsi1.add(fengongsi11);
		FenGongSi fengongsi12=new FenGongSi("总公司下分公司1分部门2");
		fengongsi1.add(fengongsi12);
		
		FenGongSi fengongsi21=new FenGongSi("总公司下分公司2分部门1");
		fengongsi2.add(fengongsi21);
		FenGongSi fengongsi22=new FenGongSi("总公司下分公司2分部门2");
		fengongsi2.add(fengongsi22);
		fenGongSi.iterate();
	}
}
{% endhighlight %}
-----------

##运行结果：

----------------

>总公司
>总公司下部门1
>总公司下部门2
>总公司下分公司1
>总公司下分公司1分部门1
>总公司下分公司1分部门2
>总公司下分公司2
>总公司下分公司2分部门1
>总公司下分公司2分部门2

----------------