## Java并发基础篇

重点难点，加油！！！

### 1 进程和线程

进程是程序的一次执行过程，是系统运行程序的基本单位，因此进程是动态的。

一个进程在其执行的过程中可以产生多个线程。与进程不同的是同类的多个线程**共享进程的堆和方法区资源，但每个线程有自己的程序计数器、虚拟机栈和本地方法栈**，所以系统在产生一个线程，或是在各个线程之间作切换工作时，负担要比进程小得多。

**进程让操作系统的并发性成为了可能，而线程让进程的内部并发成为了可能。**

#### 1.1 进程和线程的区别

本质的区别是**是否单独占有内存地址空间及其它系统资源（比如I/O）**

- 进程单独占有一定的内存地址空间，所以进程间存在内存隔离，数据是分开的，数据共享复杂但是同步简单，各个进程之间互不干扰；而线程共享所属进程占有的内存地址空间和资源，数据共享简单，但是同步复杂。
- 进程单独占有一定的内存地址空间，一个进程出现问题不会影响其他进程，不影响主程序的稳定性，可靠性高；一个线程崩溃可能影响整个程序的稳定性，可靠性较低。
- 进程单独占有一定的内存地址空间，进程的创建和销毁不仅需要保存寄存器和栈信息，还需要资源的分配回收以及页调度，开销较大；线程只需要保存寄存器和栈信息，开销较小。
- **进程是操作系统进行资源分配的基本单位，而线程是操作系统进行调度的基本单位**，即CPU分配时间的单位 

### 2 Java多线程入门类和接口

实现线程类，两种方式：

- **继承`Thread`类，并重写`run`方法。**在程序里面调用了start()方法后，虚拟机会先为我们创建一个线程，然后等到这个线程第一次得到时间片时再调用run()方法。

- **实现`Runnable`接口的`run`方法。**==可以使用Java8的函数式编程简化代码。==

`Thread`类是一个`Runnable`接口的实现类。

#### 2.1 Thread类常用方法

`currentThread()`：静态方法，返回对当前正在执行的线程对象的引用；

`start()`：开始执行线程的方法，Java虚拟机会调用线程内的run()方法；

`yield()`：当前线程愿意让出对当前处理器的占用。需要注意的是，就算当前线程调用了yield()方法，程序在调度的时候，也还有可能继续运行这个线程的；

`sleep()`：静态方法，使当前线程睡眠一段时间；

`join()`：使当前线程等待另一个线程执行完毕之后再继续执行，内部调用的是Object类的wait方法实现的；

#### 2.2 继承Thread还是实现Runnable

- 由于Java“单继承，多实现”的特性，Runnable接口使用起来比Thread更灵活。

- Runnable接口出现更符合面向对象，将线程单独进行对象的封装。

- Runnable接口出现，降低了线程对象和线程任务的耦合性。

- 如果使用线程时不需要使用Thread类的诸多方法，显然使用Runnable接口更为轻量。

### 3 Callable、Future与FutureTask

使用`Runnable`和`Thread`来创建一个新的线程，`run`方法是没有返回值的。而有时候我们希望开启一个线程去执行一个任务，并且这个任务执行完成后有一个返回值。

JDK提供了`Callable`接口与`Future`类为我们解决这个问题，这也是所谓的“异步”模型。

#### 3.1 Callable接口

==这一部分没有结合实际代码有点难应用，马住。==

`Callable`与`Runnable`类似，同样是只有一个抽象方法的函数式接口。不同的是，`Callable`提供的方法是**有返回值**的，而且支持**泛型**。`Callable`一般是配合线程池工具`ExecutorService`来使用的。这里只介绍`ExecutorService`可以使用`submit`方法来让一个`Callable`接口执行。它会返回一个`Future`，我们后续的程序可以通过这个`Future`的`get`方法得到结果。

#### 3.2 Future接口

Future是Callable的返回值接口。

```java
public abstract interface Future<V> {
    public abstract boolean cancel(boolean paramBoolean);
    public abstract boolean isCancelled();
    public abstract boolean isDone();
    public abstract V get() throws InterruptedException, ExecutionException;
    public abstract V get(long paramLong, TimeUnit paramTimeUnit)
            throws InterruptedException, ExecutionException, TimeoutException;
}
```

`cancel`方法是试图取消一个线程的执行。

注意是**试图**取消，**并不一定能取消成功**。因为任务可能已完成、已取消、或者一些其它因素不能取消，存在取消失败的可能。`boolean`类型的返回值是“是否取消成功”的意思。参数`paramBoolean`表示是否采用中断的方式取消线程执行。

==实用Tips:==

所以有时候，为了让任务有能够取消的功能，就使用`Callable`来代替`Runnable`。如果为了可取消性而使用 `Future`但又不提供可用的结果，则可以声明 `Future<?>`形式类型、并返回 `null`作为底层任务的结果。

#### 3.3 FutureTask类

Future接口的实现类，

```java
// 自定义Callable
class Task implements Callable<Integer>{
    @Override
    public Integer call() throws Exception {
        // 模拟计算需要一秒
        Thread.sleep(1000);
        return 2;
    }
    //1使用Future
    public static void main(String args[]){
        ExecutorService executor = Executors.newCachedThreadPool();
        Task task = new Task();
        Future<Integer> result = executor.submit(task);
        // 注意调用get方法会阻塞当前线程，直到得到结果。
        // 所以实际编码中建议使用可以设置超时时间的重载get方法。
        System.out.println(result.get()); 
    }
    //2使用FutureTask
     public static void main(String args[]) throws Exception {
        ExecutorService executor = Executors.newCachedThreadPool();
        FutureTask<Integer> futureTask = new FutureTask<>(new Task());
        executor.submit(futureTask);
        System.out.println(futureTask.get());
    }
}
```

使用上`2`与`1`有一点小的区别。首先，调用`submit`方法是没有返回值的。这里实际上是调用的`submit(Runnable task)`方法，而`1`，调用的是`submit(Callable<T> task)`方法。

然后，`2`是使用`FutureTask`直接取`get`取值，而`1`是通过`submit`方法返回的`Future`去取值。

**FutureTask的几个状态：**

```java
private volatile int state;
private static final int NEW          = 0;
private static final int COMPLETING   = 1;
private static final int NORMAL       = 2;
private static final int EXCEPTIONAL  = 3;
private static final int CANCELLED    = 4;
private static final int INTERRUPTING = 5;
private static final int INTERRUPTED  = 6;
```

state表示任务的运行状态，初始状态为NEW。运行状态只会在set、setException、cancel方法中终止。COMPLETING、INTERRUPTING是任务完成后的瞬时状态。



### 4 线程组和线程优先级

#### 4.1线程组

Java中用ThreadGroup来表示线程组，我们可以使用线程组对线程进行批量控制。每个Thread必然存在于一个ThreadGroup中，Thread不能独立于ThreadGroup存在。

总结来说，线程组是一个树状的结构，每个线程组下面可以有多个线程或者线程组。线程组可以起到统一控制线程的优先级和检查线程的权限的作用。

==什么时候该用线程组，线程组和线程池的区别？==

#### 4.2线程优先级

Java中线程优先级可以指定，范围是1~10。不同操作系统支持优先级的划分不同，Java只是给操作系统一个优先级的**参考值**，线程最终**在操作系统的优先级**是多少由操作系统决定。线程的优先级会在线程被调用之前设定。

`getPriority()`和`setPriority()`可获取和设置线程优先级。

如果某个线程优先级无法大于线程所在**线程组的最大优先级**。



### 5 线程状态转换

**操作系统中进程的状态转换**：

![系统进程/线程转换图](./img/系统进程状态转换图.png)

- 就绪(ready)：线程正在等待使用CPU，经调度程序调用之后可进入running状态。
- 运行(running)：线程正在使用CPU。
- 等待(waiting): 线程经过等待事件的调用或者正在等待其他资源（如I/O）。

#### 5.1 Java线程的6个状态

```java
NEW, RUNNABLE, BLOCKED, WAITING, TIMED_WAITING, TERMINATED;
```

- NEW

尚未启动（未`start()`）。

**关于start()的两个引申问题**（==很好的问题==）

1. 反复调用同一个线程的start()方法是否可行？
2. 假如一个线程执行完毕（此时处于TERMINATED状态），再次调用这个线程的start()方法是否可行？

> 两个问题的答案都是不可行，在调用一次start()之后，threadStatus的值会改变（threadStatus !=0），此时再次调用start()方法会抛出IllegalThreadStateException异常。
>
> 比如，threadStatus为2代表当前线程状态为TERMINATED。

- RUNNABLE

表示当前线程正在运行中。处于RUNNABLE状态的线程在Java虚拟机中运行，也有可能在等待CPU分配资源（==进入运行前的状态？==）。

> Java线程的**RUNNABLE**状态其实是包括了传统操作系统线程的**ready**和**running**两个状态的。

- BLOCKED

阻塞状态。处于BLOCKED状态的线程正**等待锁的释放**以进入同步区。

- WAITING

等待状态。处于等待状态的线程变成RUNNABLE状态需要其他线程唤醒。

调用如下3个方法会使线程**进入等待状态**：

1. `Object.wait()`：使当前线程处于等待状态直到另一个线程唤醒它；

2. `Thread.join()`：等待线程执行完毕，底层调用的是Object实例的wait方法；
3. `LockSupport.park()`：除非获得调用许可，否则禁用当前线程进行线程调度。

- TIMED_WAITING

超时等待状态。线程等待一个具体的时间，时间到后会被自动唤醒。

- TERMINATED

终止状态。此时线程已执行完毕。

**线程状态转换图**：

![线程状态转换图](./img/线程状态转换图.png) 

==省略了详细的状态转换，[详见](http://concurrent.redspider.group/article/01/4.html)==

#### 5.2 线程中断



