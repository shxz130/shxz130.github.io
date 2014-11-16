---
layout : life
title : "深入理解设计模式之责任链模式"
category : 深入理解设计模式
duoshuo: true
date : 2014-11-16
---

##责任链模式

-------------
* 使多个对象都有机会处理请求，从而避免请求的发送者和接受者之间的耦合关系。将这个对象连成一条链，并沿着这条链传递该请求，直到有一个对象处理他为止。
* 适合使用责任链模式的情况
 * 在许多对象可以处理用户请求的时候
 * 希望用户不必明确处理者的时候，同时向多个处理者发送请求
 * 程序希望动态的订制可处理用户请求的集合对象
* **应用场景**：
 * Struts2中的**拦截器**，**过滤器**的使用。

 
-------------------

{% highlight java %}
package com.shxz.filter;

public interface Filter {
	String dosomething(String request,FilterChain filterChain);
}

{% endhighlight %}
-----------

{% highlight java %}
package com.shxz.filter;

public class FaceFilter implements Filter{

	@Override
	public String dosomething(String request, FilterChain filterChain) {
		// TODO Auto-generated method stub
		System.out.println("我是FaceFilter,我将黄色改**");
		String temp=request.replace("黄色", "**");
		
		String response=filterChain.dosomething(temp, filterChain);
		System.out.println("我是FaceFilter,处理完毕");
		return response;
	}

}

{% endhighlight %}
-----------


{% highlight java %}
package com.shxz.filter;

import javax.swing.RepaintManager;

public class HtmlFilter implements Filter{

	@Override
	public String dosomething(String request, FilterChain filterChain) {
		// TODO Auto-generated method stub
		System.out.println("我是htmlFilter,我将<html>改为我是标签");
		String temp=request.replace("<html>", "我是标签");

		String response=filterChain.dosomething(temp, filterChain);
		System.out.println("我是htmlFilter,处理完毕");
		return response;
	}

}
	
{% endhighlight %}
-----------


{% highlight java %}
package com.shxz.filter;

public class SesstiveFilter implements Filter{

	@Override
	public String dosomething(String request, FilterChain filterChain) {
		// TODO Auto-generated method stub
		System.out.println("我是SesstiveFilter,我将郭老板改为郭金康");
		String temp=request.replace("郭老板", "郭金康");

		String response=filterChain.dosomething(temp, filterChain);
		System.out.println("我是SesstiveFilter,处理完毕");
		return response;
	}

}	
{% endhighlight %}
-----------


{% highlight java %}
package com.shxz.filter;

import java.util.ArrayList;

public class FilterChain implements Filter{
	ArrayList<Filter> list=new ArrayList<>();	
	int index=0;
	public void add(Filter filter){
		list.add(filter);
	}
	public void remove(Filter filter){
		list.remove(filter);
	}
	@Override
	public String dosomething(String request, FilterChain filterChain) {
		// TODO Auto-generated method stub
		if(index==list.size())
		{
			return request;
		}
		else{
		Filter filter=list.get(index);
		index++;
		return filter.dosomething(request, filterChain);
		
		}
	}
	
	
}	
{% endhighlight %}
-----------


{% highlight java %}
public static void main(String[] args) {
		FilterChain chain=new FilterChain();
		chain.add(new HtmlFilter());
		chain.add(new FaceFilter());
		chain.add(new SesstiveFilter());
		
		String request="<html> 郭老板喜欢看黄色电影</html>";
		System.out.println(request);
		String getrequest=chain.dosomething(request, chain);
		System.out.println(getrequest);

	}	
{% endhighlight %}
-----------

##运行结果：

----------------
><html> 郭老板喜欢看黄色电影</html>
>我是htmlFilter,我将<html>改为我是标签
>我是FaceFilter,我将黄色改**
>我是SesstiveFilter,我将郭老板改为郭金康
>我是SesstiveFilter,处理完毕
>我是FaceFilter,处理完毕
>我是htmlFilter,处理完毕
>我是标签 郭金康喜欢看**电影</html>

----------------