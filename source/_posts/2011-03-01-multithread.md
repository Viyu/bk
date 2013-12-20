title: 并发和多线程
date: 2011-03-01
tags: Concurrence
---
<b>某个线程获得对象的锁之后，只能阻止其他线程获得同一个锁，并不能阻止其他线程通过另外的锁来访问对象的变量</b>

每个可变或共享的变量都该由同一个锁来保护；

简单粗暴的全部synchronized会使得并发程序变成串行程序，影响性能，多核CPU会有空载运行，应该缩小同步块的大小，将不影响共享状态并且执行时间较长的操作从同步代码块中分离出去。

简单性（对整个方法同步）与并发性（对尽可能短的代码路径进行同步）之间的平衡；

当执行较长时间或无法快速完成的操作时，比如网络I/O，一定不要持有锁，不然会影响活跃性和性能。

---------------------------------------------------

<b>并发关注的就是：共享+可变的状态</b>

多个线程之间，不仅要防止错误地修改和读取了状态，还要共享状态，即一个线程修改了状态，另一个线程要被通知到这个修改；

重排序：无同步的多线程程序中，无法正确判断代码的执行顺序。

Sleep和Yield都是让cpu不让锁，跟锁没关系；Wait是让锁。

非volatile的long和double不是线程安全的，64位的读写会被分解，不是原子性的；

“加锁的含义不仅仅局限于互斥行为，还包括内存可见性。为了确保所有线程都能看到共享变量的最新值，所有执行读写操作的线程都必须在同一锁上同步。”

<!-- more -->

---------------------------------------------------

<b>线程安全性的需求不一定来自于对线程的直接使用，而是来源于对比如Servlet这种框架的使用</B>

无状态对象一定是线程安全的；有状态但是状态对象是线程安全的，比如是原子的，也是线程安全的。但如果有多个状态对象，则线程不安全，除非在一个原子操作中同时更新了所有状态对象。synchronized(obj){}就可以保证代码块的原子性。如果一个线程试图获取被另一个线程池有的锁，则该线程会阻塞；如果一个线程试图获取已经被自己池有的锁，“重入”机制使得该线程可以获取；

{% code %}
public class Widget {

public synchronized void dosth() {
}

}

public class Loggingwidget extends Widget {

public synchronized void dosth() {

System.out.println("do sth");

super.dosht();

}
}
{% endcode %}

---------------------------------------------------

<b>volatile是比synchronized更轻量级的同步</b>

olatile，确保将变量的更新操作通知到其他线程；

从内存可见性的角度，写入volatile变量相当于退出同步代码块，读取volatile变量相当于进入同步代码块。

加锁既可以保证可见性又可确保原子性，而volatile只确保可见性；

---------------------------------------------------

<b>逸出</b>

{% code %}
private String[] strs = {"Hello", "World"};

public String getStrs() {

return this.strs;
}
{% endcode %}

strs是被设计为private的，但是通过getStrs方法，外部类可以修改strs的内容，这就是逸出。

不要再构造过程中使thisy引用逸出:

{% code %}
public class ThisEscape {

public ThisEscape(EventSource source) {

    source.registerListener() { new EventListener() {

        public void onEvent(Event e) {

        }
...
{% endcode %}

上面的内部类把尚未构造完成的this给逸出了；

在构造方法中使用内部类，会逸出this;

在构造方法中启动线程，会逸出this;

在构造方法中调用可改写的实例方法（非private非final），会逸出this；

---------------------------------------------------

<b>调用栈和线程</b>

基本类型的局部变量的固有属性之一就是封闭在执行线程中，它们位于执行线程的栈中，其他线程无法访问这些栈；非基本类型的局部变量，可能被各种方式逸出，比如把该非基本类型的局部变量传递给其他的方法，而基本类型是无法被引用的，所以不会被逸出；

而全局变量，是被每一个线程-调用栈，共享的；

---------------------------------------------------

除非需要更高的可见性，否则应将所有的域都声明为私有域；

除非需要某个域是可变的，否则应将其声明为final域；

某种情况下，volatile+不可变对象，保证了原子性和可见性，

volatile保证可见性不保证原子性，而不可变对象保证了一种弱于synchronized的原子性；

当对象的引用对所有访问该对象的线程是可见时，对象发布时的状态对于所有线程也将是可见的；

---------------------------------------------------

<b>Java线程的实现和调度</b>

如果使用内核线程，则线程的创建、切换、调度就由系统自动完成了，但是花费高；

如果使用用户线程，花费低，但是线程的维护得线程自己完成，实现复杂；

Java线程是抢占式调度，只能让时间（sleep或yield），不能要时间；

Java的线程优先级并不是太靠谱，因为Java自定义的优先级与系统的优先级要映射，如果前者数量比后者多，则会出现Java的几种优先级被系统映射成同一个优先级的情况；

---------------------------------------------------

线程安全的实现方法--互斥同步(阻塞同步)

互斥是因，同步是果，互斥是方法，同步是目的；

如果synchronized明确指定了锁定/解锁（字节码指令monitorenter和monitorexit需要一个reference类型的参数来指定锁定解锁对象）的对象参数，那就是这个对象的reference；如果没有指定，则如果synchronized修饰的是实例方法，取对应的实例，如果synchronized修饰的是类方法，取对应的class对象作为锁对象；

synchronized是重量级Heavyweight的操作，因为java线程是映射到操作系统的原生线程之上的，状态转换消耗的时间可能比用户代码执行的时间还长；

java.util.concurrent.ReentrantLock是API层面的互斥，比原生的synchronized增加了一些高级功能：等待可中断、可实现公平锁、锁可以绑定多个条件。

阻塞同步是悲观的并发策略，非阻塞同步是乐观的并发策略；

---------------------------------------------------

线程安全的实现方法--无同步方案

有一些代码天生就是线程安全的，比如：

1，可重入代码,如果一个方法，它的返回值是可预测的，可根据输入推测出输出，则这个方法就是可重入的代码，是线程安全的；

2，线程本地存储，共享数据的可见范围限制在一个线程之内。当使用ThreadLocal维护变量时，ThreadLocal为每个使用该变量的线程提供独立的变量副本，所以每一个线程都可以独立地改变自己的副本，而不会影响其它线程所对应的副本。

---------------------------------------------------

Amdahl定律：CPU数量和串行比例对并发优化的影响

加速比 = 优化前系统耗时 / 优化后系统耗时 

          <= 1 / (F + (1 - F) / N)； 其中F是系统内必须保持串行化执行的代码比重，N是CPU核数；



A: 100s -> B: 100s -> C: 100s -> D: 100s -> E: 100s

串行执行，上述系统总耗时：500s；

假如B和E两个步骤是可以并发的，这里是说，B和E分别可以被多个CPU执行，而不是说B和E两个并发执行；

那么，假如有1个CPU，耗时仍然是500s;

假如有两个CPU，则B和E的执行都减少了一半，分别都是50s，总耗时400s；

依此类推，假如CPU无穷个，那么总耗时趋于300s但不会小于300s；

从上述公式也能推算出这个结论来；

最终结论：CPU数量越多，串行化比重越低，优化效果越好；仅靠提高CPU数量而不降低程序的串行化比例，也无法有效提高系统性能；

单CPU下的多线程看似是并发，但其实比串行更耗时，因为它最终还是靠一个CPU跑，还得多出来线程切换的开销，这么说的前提是程序是单纯的CPU操作，如果涉及了磁盘、网络等访问，则单CPU下的多线程仍然有意义，因为它把阻塞的时间用来执行别的操作，降低了CPU的闲置率。

---------------------------------------------------

<b>线程池的意义和实现</b>

线程池的意义：

为了节省多线程并发环境下，大量创建和销毁线程所带来的额外开销；

线程池就是为了线程复用；

线程在run运行结束后会自动被销毁，而线程池中的线程是要复用的，不能退出，如何实现？

用无线循环，让一个Thread的run永远不结束，一直等待新的任务；

{% code %}
class ReusableThread extends Thread {

    private ThreadPool pool = null;

    private Runnable target = null;

    private boolean isShutdown = false;

    private boolean isIdle = false;

    public ReusableThread(Runnable target, String name, ThreadPool pool) {

        super(name);

        this.pool = pool;

        this.target = target;

    }

    public Runnable getTarget() {

        return target;

    }

    public boolean isIdle() {

        return isIdle;

    }

    public void run() {

        while(!isShutdown) {

            isIdle = false;

            if(target != null) {

                target.run();

            }

            isIdle = true;

            try {

                pool.repool(this);

                synchronized(this) {

                        wait();

                 }

            } catch(InterruptedException e) {};

        }

        isIdle = false;

    }

    public synchronized void setTarget(Runnable target) {

        this.target = target;

        notifyAll();

    }

    public synchronized void shutdown() {

        isShutdown = true;

        notifyAll();

    }

}

public class ThreadPool {

    private List<ReusableThread> threads;

    private int count;

    private boolean isShutdown = false;

    private ThreadPool() {

        this.threads = new Vector<ReusableThread>();

        count = 0;

    }

    private static class SingletonHolder {

        private static ThreadPool instance = new ThreadPool();

    }

    public static ThreadPool getInstance() {

        return SingletonHolder.instance;

    }

    public int getCount() {

        return count;

    }

    protected synchronized void repool(ReusableThread thread) {

        if(!isShutdown) {

            threads.add(thread);

        } else {

            thread.shutdown();

        }

    }

    public synchronized void shutdown() {

        isShutdown = true;

        for(ReusableThread t : threads) {

            t.shutdown();

        }

    }

    public synchronized void start(Runnable target) {

        ReusableThread thread = null;

        if(threads.size() > 0) {

            thread = threads.remove(threads.size() - 1);

            thread.setTarget(target);

        } else {

        count++;

        thread = new ReusableThread(target, "Reusable Thread # " + count, this);

        thread.start();

    }

}

public static void main(String[] args) {

    {

        long old = System.currentTimeMillis();

        for(int i = 0; i < 1000; i ++) {

            ThreadPool.getInstance().start(new Runnable() {

                public void run() {

                    try {    

                        Thread.sleep(1000);

                    } catch (InterruptedException e) {}

                }

            });

        }

        System.out.println(System.currentTimeMillis() - old);

        ThreadPool.getInstance().shutdown();

    }

    {

        long old = System.currentTimeMillis();

        for(int i = 0; i < 1000; i ++) {

            (new Thread(new Runnable() {

                public void run() {

                    try {

                        Thread.sleep(1000);

                    } catch (InterruptedException e) {}

                }

            })).start();

        }

        System.out.println(System.currentTimeMillis() - old);

        }

    }
}
{% endcode %}

JDK内置了线程池，java.util.concurrent.ExecutorService；

---------------------------------------------------

<b>生产者消费者模型</b>

生产者消费者模型的意义：

核心组件是共享缓冲区，它将生产者和消费者解耦；

并且缓解了生产者和消费者之间的性能差；

只能在同步控制方法或同步块中调用wait()、notify()和notifyAll()；

synchronized的生产和消费方法是共享缓冲区的，而非生产者和消费者线程的；

wait将调用自己的线程阻塞并让锁，notify使得等待的同样以缓冲区对象为锁的另一个线程获得锁；wait是阻塞自己，notify是释放别人；

缓冲区空，需要生产，假如消费线程先到达，就会顺序被压入wait栈，这时候生产线程不需要被notify，因为它们还没被压入wait栈，所以生产线程直接生产，然后调用notify把wait栈的消费线程激活了，这时候如果缓冲区还没满，生产线程和消费线程抢夺锁来交替执行，偶尔的机会，消费线程消费太满，可能缓冲区满了，此时生产线程被顺序压入wait栈，等待消费线程消费后调用notify来唤醒它们；

生产者和消费者是交织在一起的，是不停歇的，所以生产和消费动作都可以放在while(true)下；

生产者和消费者不是两个线程，而是两类线程，是很多个这种两类线程之间的交织；

{% code %}
class Storage {

    private int count = 0;

    public synchronized void produce() {

        while (count == 10) 

            wait();            

        count++;

        notify();

    }

    public synchronized void consume() {

        while (count == 0) {

            wait();

        }

        count--;

        notify();

    }

class ProducerThread implements Runnable {

    public void run() {

        while (true) 

            resource.produce();

    }

}

class ConsumerThread implements Runnable {

    public void run() {

        while (true) 

            resource.consume();

    }

}

new Thread(new ProducerThread(resource)).start();

...

new Thread(new ConsumerThread(resource)).start();

...
{% endcode %}

---------------------------------------------------

