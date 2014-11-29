---
layout : life
title : "深入理解设计模式之解释器模式"
category : 深入理解设计模式
duoshuo: true
date : 2014-11-29
---

##解释器模式

-------------

* **解释器模式**可以定义出其文法的一种表示，并同时提供一个解释器。客户端可以使用这个解释器来解释这个语言中的句子。
**优点：** 
 * 解释器是一个简单语法分析工具，它最显著的优点就是扩展性，修改语法规则只要修改相应的非终结符表达式就可以了，若扩展语法，则只要增加非终结符类就可以了。
* **缺点**：
 * 解释器模式会引起类膨胀，每个语法都要产生一个非终结符表达式，语法规则比较复杂时，可能产生大量的类文件，难以维护。
 * 解释器模式采用递归调用方法，它导致调试非常复杂。
 * 解释器由于使用了大量的循环和递归，所以当用于解析复杂、冗长的语法时，效率是难以忍受的
* **角色划分**：
 * **抽象表达式(Expression)角色**：声明一个所有的具体表达式角色都需要实现的抽象接口。这个接口主要是一个interpret()方法，称做解释操作。
 * **终结符表达式(Terminal Expression)角色**：实现了抽象表达式角色所要求的接口，主要是一个interpret()方法；文法中的每一个终结符都有一个具体终结表达式与之相对应。比如有一个简单的公式R=R1+R2，在里面R1和R2就是终结符，对应的解析R1和R2的解释器就是终结符表达式。
 * **非终结符表达式(Nonterminal Expression)角色**：文法中的每一条规则都需要一个具体的非终结符表达式，非终结符表达式一般是文法中的运算符或者其他关键字，比如公式R=R1+R2中，“+"就是非终结符，解析“+”的解释器就是一个非终结符表达式。
 * **环境(Context)角色**：这个角色的任务一般是用来存放文法中各个终结符所对应的具体值，比如R=R1+R2，我们给R1赋值100，给R2赋值200。这些信息需要存放到环境角色中，很多情况下我们使用Map来充当环境角色就足够了。

----------------
![gengsuanfa](/life/picture/inter.png) 
----------------

{% highlight java %}
package com.shxz.interpreter;
public abstract class Expression {
    /**
     * 以环境为准，本方法解释给定的任何一个表达式
     */
    public abstract boolean interpret(Context ctx);
    /**
     * 检验两个表达式在结构上是否相同
     */
    public abstract boolean equals(Object obj);
    /**
     * 返回表达式的hash code
     */
    public abstract int hashCode();
    /**
     * 将表达式转换成字符串
     */
    public abstract String toString();
}
{% endhighlight %}
-----------
{% highlight java %}
package com.shxz.interpreter;

import java.util.HashMap;
import java.util.Map;

public class Context {

private Map<Variable,Boolean> map = new HashMap<Variable,Boolean>();
    
    public void assign(Variable var , boolean value){
        map.put(var, new Boolean(value));
    }
    
    public boolean lookup(Variable var) throws IllegalArgumentException{
        Boolean value = map.get(var);
        if(value == null){
            throw new IllegalArgumentException();
        }
        return value.booleanValue();
    }
}
{% endhighlight %}
-----------
{% highlight java %}
package com.shxz.interpreter;

public class Constant extends Expression{
    
    private boolean value;

    public Constant(boolean value){
        this.value = value;
    }
    
    @Override
    public boolean equals(Object obj) {
        
        if(obj != null && obj instanceof Constant){
            return this.value == ((Constant)obj).value;
        }
        return false;
    }

    @Override
    public int hashCode() {
        return this.toString().hashCode();
    }

    @Override
    public boolean interpret(Context ctx) {
        
        return value;
    }

    @Override
    public String toString() {
        return new Boolean(value).toString();
    }
    
}
{% endhighlight %}
-----------
{% highlight java %}
package com.shxz.interpreter;

public class Variable extends Expression {

    private String name;

    public Variable(String name){
        this.name = name;
    }
    @Override
    public boolean equals(Object obj) {
        
        if(obj != null && obj instanceof Variable)
        {
            return this.name.equals(
                    ((Variable)obj).name);
        }
        return false;
    }

    @Override
    public int hashCode() {
        return this.toString().hashCode();
    }

    @Override
    public String toString() {
        return name;
    }

    @Override
    public boolean interpret(Context ctx) {
        return ctx.lookup(this);
    }

}

{% endhighlight %}
-----------
{% highlight java %}
package com.shxz.interpreter;

public class And extends Expression {

    private Expression left,right;
    
    public And(Expression left , Expression right){
        this.left = left;
        this.right = right;
    }
    @Override
    public boolean equals(Object obj) {
        if(obj != null && obj instanceof And)
        {
            return left.equals(((And)obj).left) &&
                right.equals(((And)obj).right);
        }
        return false;
    }

    @Override
    public int hashCode() {
        return this.toString().hashCode();
    }

    @Override
    public boolean interpret(Context ctx) {
        
        return left.interpret(ctx) && right.interpret(ctx);
    }

    @Override
    public String toString() {
        return "(" + left.toString() + " AND " + right.toString() + ")";
    }

}
{% endhighlight %}
-----------
{% highlight java %}
package com.shxz.interpreter;

public class Not extends Expression {

    private Expression exp;
    
    public Not(Expression exp){
        this.exp = exp;
    }
    @Override
    public boolean equals(Object obj) {
        if(obj != null && obj instanceof Not)
        {
            return exp.equals(
                    ((Not)obj).exp);
        }
        return false;
    }

    @Override
    public int hashCode() {
        return this.toString().hashCode();
    }

    @Override
    public boolean interpret(Context ctx) {
        return !exp.interpret(ctx);
    }

    @Override
    public String toString() {
        return "(Not " + exp.toString() + ")";
    }

}

{% endhighlight %}
-----------
{% highlight java %}
package com.shxz.interpreter;

public class Or extends Expression {
    private Expression left,right;

    public Or(Expression left , Expression right){
        this.left = left;
        this.right = right;
    }
    @Override
    public boolean equals(Object obj) {
        if(obj != null && obj instanceof Or)
        {
            return this.left.equals(((Or)obj).left) && this.right.equals(((Or)obj).right);
        }
        return false;
    }

    @Override
    public int hashCode() {
        return this.toString().hashCode();
    }

    @Override
    public boolean interpret(Context ctx) {
        return left.interpret(ctx) || right.interpret(ctx);
    }

    @Override
    public String toString() {
        return "(" + left.toString() + " OR " + right.toString() + ")";
    }

}
{% endhighlight %}
-----------
{% highlight java %}
package com.shxz.interpreter;

public class Client {
	public static void main(String[] args) {
        Context ctx = new Context();
        Variable x = new Variable("x");
        Variable y = new Variable("y");
        Constant c = new Constant(true);
        ctx.assign(x, false);
        ctx.assign(y, true);
        
        Expression exp = new Or(new And(c,x) , new And(y,new Not(x)));
        System.out.println("x=" + x.interpret(ctx));
        System.out.println("y=" + y.interpret(ctx));
        System.out.println(exp.toString() + "=" + exp.interpret(ctx));
    }
}
{% endhighlight %}
-----------

##运行结果：

----------------

>x=false
>y=true
>((true AND x) OR (y AND (Not x)))=true

----------------

[博文参考](http://www.cnblogs.com/java-my-life/archive/2012/06/19/2552617.html)
[博文参考](http://www.cnblogs.com/itTeacher/archive/2012/12/12/2814437.html)