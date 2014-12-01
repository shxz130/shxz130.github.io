---
layout : life
title : "深入理解Java之线程Runnable/Callable/Future/FutureTask"
category : java并行优化
duoshuo: true
date : 2014-11-20
---

* **Runnable** 接口
 * 它只有一个run()函数，用于将耗时操作写在其中，**该函数没有返回值**。然后使用某个线程去执行该runnable即可实现多线程，Thread类在调用start()函数后就是执行的是Runnable的run()函数。
* **Callable** 接口
 * 与Runnable的功能大致相似，Callable中有一个call()函数，但是**call()函数有返回值**，而Runnable的run()函数不能将结果返回给客户程序。
* **Future** 接口
 * Executor就是Runnable和Callable的调度容器，Future就是对于具体的Runnable或者Callable任务的执行结果进行取消、查询是否完成、获取结果、设置结果操作。get方法会阻塞，直到任务返回结果。
* FutureTask 类
 * FutureTask则是一个RunnableFuture<V>，而RunnableFuture实现了Runnbale又实现了Futrue<V>这两个接口.

------------
{% highlight java %}
     package com.shxz;  
      
    import java.util.concurrent.Callable;  
    import java.util.concurrent.ExecutionException;  
    import java.util.concurrent.ExecutorService;  
    import java.util.concurrent.Executors;  
    import java.util.concurrent.Future;  
    import java.util.concurrent.FutureTask;  
      
    /** 
     *  
     * @author mrsimple 
     * 
     */  
    public class Text {  
        static ExecutorService mExecutor = Executors.newSingleThreadExecutor();  
        public static void main(String[] args) {  
            runnableDemo();  
            futureDemo();  
        }  
      
        /** 
         * runnable, 无返回值 
         */  
        static void runnableDemo() {  
      
            new Thread(new Runnable() {  
      
                @Override  
                public void run() {  
                    System.out.println("runnable demo : " + fibc(20));  
                }  
            }).start();  
        }  
      
        /** 
         * 其中Runnable实现的是void run()方法，无返回值；Callable实现的是 V 
         * call()方法，并且可以返回执行结果。其中Runnable可以提交给Thread来包装下 
         * ，直接启动一个线程来执行，而Callable则一般都是提交给ExecuteService来执行。 
         */  
        static void futureDemo() {  
            try {  
                /** 
                 * 提交runnable则没有返回值, future没有数据 
                 */  
                Future<?> result = mExecutor.submit(new Runnable() {  
      
                    @Override  
                    public void run() {  
                        fibc(20);  
                    }  
                });  
      
                System.out.println("future result from runnable : " + result.get());  
      
                /** 
                 * 提交Callable, 有返回值, future中能够获取返回值 
                 */  
                Future<Integer> result2 = mExecutor.submit(new Callable<Integer>() {  
                    @Override  
                    public Integer call() throws Exception {  
                        return fibc(20);  
                    }  
                });  
      
                System.out  
                        .println("future result from callable : " + result2.get());  
      
                /** 
                 * FutureTask则是一个RunnableFuture<V>，即实现了Runnbale又实现了Futrue<V>这两个接口， 
                 * 另外它还可以包装Runnable(实际上会转换为Callable)和Callable 
                 * <V>，所以一般来讲是一个符合体了，它可以通过Thread包装来直接执行，也可以提交给ExecuteService来执行 
                 * ，并且还可以通过v get()返回执行结果，在线程体没有执行完成的时候，主线程一直阻塞等待，执行完则直接返回结果。 
                 */  
                FutureTask<Integer> futureTask = new FutureTask<Integer>(  
                        new Callable<Integer>() {  
                            @Override  
                            public Integer call() throws Exception {  
                                return fibc(20);  
                            }  
                        });  
                // 提交futureTask  
                mExecutor.submit(futureTask) ;  
                System.out.println("future result from futureTask : "  
                        + futureTask.get());  
      
            } catch (InterruptedException e) {  
                e.printStackTrace();  
            } catch (ExecutionException e) {  
                e.printStackTrace();  
            }  
        }  
      
        /** 
         * 效率底下的斐波那契数列, 耗时的操作 
         *  
         * @param num 
         * @return 
         */  
        static int fibc(int num) {  
            if (num == 0) {  
                return 0;  
            }  
            if (num == 1) {  
                return 1;  
            }  
            return fibc(num - 1) + fibc(num - 2);  
        }  
      
    }  
{% endhighlight %}
------------

###运行结果：

------------

>runnable demo : 6765
>future result from runnable : null
>future result from callable : 6765
>future result from futureTask : 6765

------------

[参考博文](http://blog.csdn.net/bboyfeiyu/article/details/24851847)