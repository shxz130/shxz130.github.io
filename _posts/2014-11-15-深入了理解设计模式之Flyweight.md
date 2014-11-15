---
layout : life
title : "深入理解设计模式之享元模式"
category : 深入理解设计模式
duoshuo: true
date : 2014-11-15
---

##享元模式

-------------
* 一个系统中如果有多个相同的对象，那么只共享一份就可以了，不必每个都去实例化一个对象
* 享元对象能做到共享的关键：
 * **内蕴状态(Internal State)**：储在享元对象内部的，并且是不会随环境的改变而有所不同。因此，一个享元可以具有内蕴状态并可以共享。
 * **外蕴状态(External State)**：随环境的改变而改变的、不可以共享的。享元对象的外蕴状态必须由客户端保存，并在享元对象被创建之后，在需要使用的时候再传入到享元对象内部。外蕴状态不可以影响享元对象的内蕴状态，它们是相互独立的。
* java中，**String类型**就是用了享元。String对象是final类型，对象一旦创建就不可改变。在JAVA中字符串常量都是存在常量池中的，JAVA会确保一个字符串常量在常量池中只有一个拷贝。String a="abc"，其中"abc"就是一个字符串常量。

--------------

##享元模式的优缺点

--------------

 * 享元模式的优点在于它**大幅度地降低内存中对象的数量**。但是，它做到这一点所付出的代价也是很高的：
 * **享元模式使得系统更加复杂**。为了使对象可以共享，需要将一些状态外部化，这使得程序的逻辑复杂化。
 * **享元模式将享元对象的状态外部化，而读取外部状态使得运行时间稍微变长。**
 
-------------

{% highlight java %}
public class Test {

    public static void main(String[] args) {
        
        String a = "abc";
        String b = "abc";
        System.out.println(a==b);
        
    }
}
{% endhighlight %}





-----------

##享元模式的角色

-------------
* **抽象享元(Flyweight)角色** ：给出一个抽象接口，以规定出所有具体享元角色需要实现的方法。
* **具体享元(ConcreteFlyweight)角色**：实现抽象享元角色所规定出的接口。如果有内蕴状态的话，必须负责为内蕴状态提供存储空间。
* **享元工厂(FlyweightFactory)角色** ：本角色负责创建和管理享元角色。本角色必须保证享元对象可以被系统适当地共享。当一个客户端对象调用一个享元对象的时候，享元工厂角色会检查系统中是否已经有一个符合要求的享元对象。如果已经有了，享元工厂角色就应当提供这个已有的享元对象；如果系统中没有一个适当的享元对象的话，享元工厂角色就应当创建一个合适的享元对象。


{% highlight java %}
package com.shxz.flyweight;


public interface FlyWeight {
    //一个示意性方法，参数state是外蕴状态
    public void operation(String state);
}

{% endhighlight %}
-----------

{% highlight java %}
package com.shxz.flyweight;


	public class ConcreteFlyWeight implements FlyWeight {
	    private Character intrinsicState = null;
	    /**
	     * 构造函数，内蕴状态作为参数传入
	     * @param state
	     */
	    public ConcreteFlyWeight(Character state){
	        this.intrinsicState = state;
	    }	    
	    /**
	     * 外蕴状态作为参数传入方法中，改变方法的行为，
	     * 但是并不改变对象的内蕴状态。
	     */
	    @Override
	    public void operation(String state) {
	        // TODO Auto-generated method stub
	        System.out.println("Intrinsic State = " + this.intrinsicState);
	        System.out.println("Extrinsic State = " + state);
	    }

	}

{% endhighlight %}
-----------
{% highlight java %}
package com.shxz.flyweight;

import java.util.HashMap;
import java.util.Map;

public class FlyWeightFactory {
	 private Map<Character,FlyWeight> files = new HashMap<Character,FlyWeight>();
	    
	    public FlyWeight factory(Character state){
	        //先从缓存中查找对象
	        FlyWeight fly = files.get(state);
	        if(fly == null){
	            //如果对象不存在则创建一个新的Flyweight对象
	            fly = new ConcreteFlyWeight(state);
	            //把这个新的Flyweight对象添加到缓存中
	            files.put(state, fly);
	        }
	        return fly;
	    }
	    
	 
}

{% endhighlight %}
-----------
{% highlight java %}
   public static void main(String[] args) {
	        // TODO Auto-generated method stub
	        FlyWeightFactory factory = new FlyWeightFactory();
	        FlyWeight fly = factory.factory(new Character('a'));
	        fly.operation("First Call");
	        
	        fly = factory.factory(new Character('b'));
	        fly.operation("Second Call");
	        
	        fly = factory.factory(new Character('a'));
	        fly.operation("Third Call");
	    }
{% endhighlight %}
-----------

##运行结果：

----------------
>Intrinsic State = a
>Extrinsic State = First Call
>Intrinsic State = b
>Extrinsic State = Second Call
>Intrinsic State = a
>Extrinsic State = Third Call

----------------