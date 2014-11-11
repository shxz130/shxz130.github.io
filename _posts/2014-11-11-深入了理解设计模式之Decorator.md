---
layout : life
title : "深入理解设计模式之装饰者模式"
category : 深入理解设计模式
duoshuo: true
date : 2014-11-11
---

##装饰者模式

-------------

通过继承实现子类的行为是在编译时静态决定的，而且所有子类都会继承到相同的行为;而组合可以在运行时动态的进行扩展，所以应该尽量利用组合的方法扩展对象的行为。运行时的扩展远比编译时期的继承威力大
 * 装饰者和被装饰者必须是一样的类型，即具有共同的超类。 
 * 当我们将装饰者与组件组合时，就是在加入新的行为。
 * java I/O中使用了大量的装饰类
 * 装饰者模式符合开放——关闭模式:允许扩展，但是不允许修改代码。

------------
 
案例分析：

 一杯茶100元，加冰10块，加糖40.以后说不定还会加其他调料，所以要有很好的扩展性。

父类（抽象类）：水
{% highlight java %}
package com.shxz.Decorator;

public abstract class Water {
	private String description="water";
	public abstract double cost();
	public String getDescription() {
		return description;
	}
	public void setDescription(String description) {
		this.description = description;
	}
}


{% endhighlight %}
-----------
{% highlight java %}
package com.shxz.Decorator;

public class Tea extends Water{
	@Override
	public double cost() {
		// TODO Auto-generated method stub
		return 100;
	}
	@Override
	public String getDescription() {
		// TODO Auto-generated method stub
		return "茶";
	}
}


{% endhighlight %}
-----------
{% highlight java %}
package com.shxz.Decorator;

public abstract class PeiLiao extends Water{
	public abstract String getDescription();
}
{% endhighlight %}
-----------
{% highlight java %}
package com.shxz.Decorator;

public class Addsugar extends PeiLiao{
	private Water water;
	public Addsugar(Water water)
	{
		this.water=water;
	}
	@Override
	public String getDescription() {
		// TODO Auto-generated method stub
		return water.getDescription()+ "加糖";
	}
	@Override
	public double cost() {
		// TODO Auto-generated method stub
		return water.cost()+30;
	}	
}

{% endhighlight %}
-----------
{% highlight java %}

package com.shxz.Decorator;

public class AddIce extends PeiLiao{
	private Water water;
	public AddIce(Water water)
	{
		this.water=water;
	}
	@Override
	public String getDescription() {
		// TODO Auto-generated method stub
		return water.getDescription()+"加冰";
	}
	@Override
	public double cost() {
		// TODO Auto-generated method stub
		return water.cost()+10;
	}
}

{% endhighlight %}
-----------

{% highlight java %}
package com.shxz.Decorator;

public class Main {
	public static void main(String[] args) {
		Water water=new Tea();
		System.out.println(water.getDescription());
		System.out.println(water.cost());
		water=new AddIce(water);
		System.out.println(water.getDescription());
		System.out.println(water.cost());
		water=new Addsugar(water);
		System.out.println(water.getDescription());
		System.out.println(water.cost());
	}
}
{% endhighlight %}
-----------

##运行结果：

----------------
>茶加冰
>110.0
>茶加冰加糖
>140.0
----------------