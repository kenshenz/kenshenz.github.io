---
layout: post
category : lessons
tags : [intro, beginner, jekyll, tutorial]
---
{% include JB/setup %}


#Java并发系列

##Java之多线程

###如何创建Java线程

Java线程的创建方式有两种，一种是继承java.lang.Thread或其子类，例如：

    Thread t = new Thread();
    t.start();

实例化一个Thread对象，然后调用它地start()来执行线程。

第二种方式是实现java.lang.Runnable接口，并作为Thread的构造参数，例如：

    public class MyRunnable implements Runnable {
        public void run() {
            //线程的执行代码
        }
    }

创建一个类MyRunnable，实现Runnable，并重写run方法

    Runnable r = new MyRunnable();
    Thread t = new Thread(r);
    t.start();

实例化一个Runnable对象，然后作为参数传递给Thread，最后调用Thread对象地start()方法，这里地start()实际上就是调用了MyRunnable重写的run()方法。

###run()和start()方法

调用run()和start()看起来效果一样，实际上，run()方法是由当前线程执行的，而start()才是由我们创建的线程执行的，例如：

    创建线程 A B C D
    A.run()
    B.run()
    C.start()
    D.start()

假设线程执行任务需要一定时间，我们会发现A.run()结束之后才会执行B.run()，相当于单线程，而C.start()和D.start()是并行执行的，这才是多线程。








