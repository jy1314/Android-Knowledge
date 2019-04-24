# Java线程基本概念
## 定义任务
java中的任务被抽象成了一个Runnable接口：

```
public interface Runnable {
    public void run();
}
```
我们的自定义任务需要去实现这个接口，并把任务的详细内容写在覆盖的run方法里，比如我们定义一个输出一个字符串的任务：

```
public class PrintTask implements Runnable {

    @Override
    public void run() {
        System.out.println("输出一行字");
    }
}
```
## Thread类

java中的Thread类来代表一个线程，我们需要关注它的这几种构造方法：

```
Thread(Runnable target, String name)
```
在创建线程对象的时候传入需要执行的任务以及这个线程的名称。

```
Thread(Runnable target)
```
只传入需要执行的任务，名称是系统自动生成的，或者可以在创建对象后再通过别的方法修改名称。

```
Thread(String name)
```
只传入待创建线程的名称。

```
Thread()
```
啥都不传，就是单纯构造一个线程对象而已～

## 执行任务

Thread类的start()方法负责开始执行一个线程，让一个线程运行起来有这么两种方法：
创建Thread对象的时候指定需要执行的任务:

```
public class Test {

    public static void main(String[] args) {
        new Thread(new PrintTask()).start();
    }
}
```

通过继承Thread类并覆盖run方法：

Thread类本身就代表了一个Runnable任务，我们看Thread类的定义：

```
public class Thread implements Runnable {

    private Runnable target;

    @Override
    public void run() {
        if (target != null) {
            target.run();
        }
    }

    // ... 为省略篇幅，省略其他方法和字段
}
```
其中的target就是在构造方法里传入的，如果构造方法不传这个字段的话，很显然run方法就是一个空实现，所以如果我们想运行这个线程，就继承它并且覆盖一下run方法。

```
public class PrintThread extends Thread {

    @Override
    public void run() {
        System.out.println("输出一行字");
    }
}
```
因为PrintThread中已经有一个任务了，所以直接调用start方法运行它就好：
```
public class Test {

    public static void main(String[] args) {
        new PrintThread().start();
    }
}
```
使用继承Thread类并且覆盖run方法的方式把线程和任务给弄到了一块儿了，不可分割了，也就是所谓的耦合了，所以我们平时更倾向于使用任务和线程分开处理的第1种执行任务的方式。当然，有时候为了演示的方便，也是会使用继承Thread类并且覆盖run方法的方式。

## 线程相关方法
Thread类提供了许多方法来方便我们获取线程的信息或者控制线程，下边来一下都有哪些重要的方法吧：

### 1.获取线程ID
long getId()：系统会为每个线程自动分配一个long类型的id值，通过getId方法可以获取这个值。
### 2.获取和设置线程名称
+ void setName(String name)：设置线程的名称。
+ String getName()：获取线程的名称。
### 3.获取和设置线程优先级
处理器会从就绪队列里挑一个已经就绪的线程去执行，每个线程都可以有不同的优先级，优先级越高，越容易被处理器选中去执行。
void setPriority(int newPriority)：设置线程优先级。

java中的优先级是用一个正数表示，共有1~10十个等级，其中，设计java的大叔们用了三个静态变量表示我们常用的：

+ Thread.MIN_PRIORITY = 1
+ Thread.NORM_PRIORITY = 5;
+ Thread.MAX_PRIORITY = 10;

一般情况下，我们就用这三个变量去表示优先级就够用了～

int getPriority()：获取线程的优先级。

### 4.休眠

如果想在线程执行过程中让程序停一段时间之后再执行，这个停止一段时间也叫做休眠，就好像睡一段时间然后再醒来。可以通过调用sleep方法，来实现休眠：

```
static void sleep(long millis) throws InterruptedException
```

程序在指定的毫秒数内让当前正在执行的线程休眠，也就是暂停执行。

```
static void sleep(long millis, int nanos) throws InterruptedException
```
程序在指定的毫秒数加纳秒数内让当前正在执行的线程休眠，也就是暂停执行。

这个sleep方法是一个静态方法，它会让**当前线程**暂停指定的时间。这个所谓的暂停，或者说休眠其实只是把正在运行的线程阻塞掉，放到阻塞队列里，等指定的时间一到，再从阻塞队列里出来而已。
### 5.获取当前正在执行的线程
不同的线程可以共享同一个进程中的同一段内存，也就意味着不同的线程可以执行同一段代码，有些时候有必要获取一下代码在哪个线程中执行。

static Thread currentThread()：获取当前正在执行的线程对象的引用。
### 6.守护线程

线程可以被分成两种类型，一种叫**普通线程**，另一种就叫**守护线程**，守护线程也被称为后台线程。守护线程在程序执行过程中并不是不可或缺的，它主要是为普通线程提供便利服务的，比方说java中不需要我们手动去释放某个对象的内存，它有一种传说中的垃圾收集器在不停的去释放程序中不需要的内存，在我们启动程序的时候系统会帮助我们创建一个负责收集垃圾的线程，这个线程就是一个守护线程。如果所有的普通线程全部死掉了，那守护线程也会跟着死掉，也就是程序就退出了，比方说在所有的普通线程停止了之后，这个负责收集垃圾的守护线程也会自动退出的。反过来说只要有一个普通线程活着，程序就不会退出。我们可以通过下边这些方法来判断一个线程是否是守护线程或者把一个线程设置为守护线程：

+ boolean isDaemon()：判断该线程是否为守护线程。
+ void setDaemon(boolean on)：将该线程标记为守护线程或者普通线程。

**只有在线程未启动的时候才能设置该线程是否为守护线程**，否则的话会抛出异常。
另外，**从普通线程中生成的线程默认是普通线程，从守护线程中生成的线程默认是守护线程**。

### 7.让出本次处理器时间片
static void yield()：表示放弃此次时间片时间，等待下次执行。
但是，这个yield方法只是**建议处理器不要在此次时间片时间内继续执行本线程**，最后实际怎么着还不一定呢，另外，yield是一个**静态方法**，表示让出当前线程本次时间片的时间。

### 8.等待另一线程执行结束
有的时候需要等待一个线程执行完了，另一个线程才能继续执行，Thread类提供了这样的方法：

```
void join()
```
等待该线程终止才能继续执行。

```
void join(long millis)
```
在指定毫秒数内等待该线程终止，如果到达了指定时间就继续之行了。

```
void join(long millis, int nanos)
```
跟上边方法一个意思，只不过加了个纳秒限制。

## 总结：

（1）、当我们运行一个java程序时，系统会为我们默认创建一个名叫main的线程。

（2）、 java中的任务被抽象成了一个Runnable接口，我们的自定义任务需要去实现这个接口，并把任务的详细内容写在覆盖的run方法里。

（3）、java中的Thread类来代表一个线程，它提供设置线程名、线程要执行的任务等相关构造方法。

（4）、 调用Thread对象的start方法可以执行一个线程，使用某个线程执行某个任务的方式有两种：

- 	创建Thread对象的时候指定需要执行的任务。
-  通过继承	Thread类并覆盖run方法。

（5）、Thread类提供一系列方法来方便我们获取线程的信息或者控制线程，包括下边这些功能：

- 获取线程ID
- 获取和设置线程名称
- 获取和设置线程优先级
- 线程休眠
- 获取当前正在执行的线程
- 守护线程的设置和判断相关方法
- 让出本次处理器时间片
- 等待另一线程执行结束

（6）、可以为某个线程设置异常处理器来捕获线程中没有catch的异常。







