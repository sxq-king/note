# 1、JUC概述
## 1.1 什么是JUC
> 在Java中，线程部分是一个重点，本篇文章说的JUC 也是关于线程的。JUC就是java.util .concurrent工具包的简称。这是一个处理线程的工具包，JDK1.5开始出现的。

## 1.2 进程和线程的概念

  >进程∶指在系统中正在运行的一个应用程序﹔程序一旦运行就是进程;进程——资源分配的最小单位。

------

  > 线程︰系统分配处理器时间资源的基本单元，或者说进程之内独立执行的一个单元执行流。线程——程序执行的最小单位。
  > 

##  1.3 线程的状态

### 1.3.1线程状态枚举类

```.java
Thread.State

public enum State{
	NEW,(新建)
	RUNNABLE,(准备就绪)
	BLOCKED,(阻塞)
	WATING,(不见不散)
	TIMED_WATING,(过时不候)
	TERMINATED;(终结)
}
```
### 1.3.2 wait和sleep
( 1 ) sleep是Thread的静态方法，wait是Object的方法，任何对象实例都能调用。

( 2 ) sleep不会释放锁，它也不需要占用锁。wait会释放锁，但调用它的前提是当前线程占有锁(即代码要在synchronized 中)。

( 3)它们都可以被interrupted方法中断。

##  1.4 并发与并行
### 1.4.1 串行模式
串行表示所有任务都一一按先后顺序进行。串行意味着必须先装完一车柴才能运送这车柴，只有运送到了，才能卸下这车柴，并且只有完成了这整个三个步骤，才能进行下一个步骤。
**串行是一次只能取得一个任务，并执行这个任务**。

### 1.4.2 并行模式

多个任务同时进行

### 1.4.3 并发

同一时刻，多个线程，访问同一个资源

### 1.4.4 小结(重点)

并发: 

> 同一时刻多个线程在访问同一个资源，多个线程对一个点
> 例子︰春运抢票电商秒杀...

并行:
> 多项工作一起执行，之后再汇总
> 例子︰泡方便面，电水壶烧水，一边撕调料倒入桶中

## 1.5 管程

Monitor监视器 ，就是java中所说的锁

> 是一种同步机制，保证同一个时间，只有一个线程访问被保护的数据或代码

## 1.5  用户线程和守护线程

用户线程：比如、自定义线程



守护线程：比如、垃圾回收

# 2、Lock接口

## 2.1 Synchronized关键字

###  2.1.1Synchronized作用范围

- 1．修饰一个代码块，被修饰的代码块称为同步语句块，其作用的范围是大括号括起来的代码，作用的对象是调用这个代码块的对象
- 2．修饰一个方法，被修饰的方法称为同步方法，其作用的范围是整个方法，作用的对象是调用这个方法的对象
- 3．修饰一个静态的方法，其作用的范围是整个静态方法，作用的对象是这个类的所有对象;
- 4. 修饰一个类，其作用的范围是synchronized 后面括号括起来的部分，作用的对象是这个类的所有对象。

### 2.1.2 Synchroized实现买票例子

3个售票员 卖出30张票

```java
package com.sxq.sync;

/**
 * @author song
 * @version 1.0
 * @date 2023/1/3 18:34
 * @description: TODO
 */
class Ticket{
    //票数
    private int number = 30;
    //操作方法 买票
    public synchronized void sale(){
        //判断：是否有票
        if(number > 0){
            System.out.println(Thread.currentThread().getName() + " : 卖出 : " + number-- + " 剩下 " + number);
        }
    }
}
public class SalTicket {
    //创建多个线程调用卖票方法

    public static void main(String[] args) {
        Ticket ticket = new Ticket();
        //创建线程
        new Thread(new Runnable() {
            @Override
            public void run() {
                for (int i = 0; i < 40; i++) {
                    ticket.sale();
                }
            }
        },"AA").start();
        new Thread(new Runnable() {
            @Override
            public void run() {
                for (int i = 0; i < 40; i++) {
                    ticket.sale();
                }
            }
        },"BB").start();
        new Thread(new Runnable() {
            @Override
            public void run() {
                for (int i = 0; i < 40; i++) {
                    ticket.sale();
                }
            }
        },"CC").start();
    }
}

```
### 2.1.3 多线程编程步骤

上部

- 第一步 创建资源类，在资源类创建属性和操作方法

中部

- 第二步在资源类操作方法
- - (1)判断
  - (2)干活
  - (3)通知

- 第三步 创建多个线程，调用资源类的操作方法

下部

- **防止虚假唤醒**

 

## 2.2 什么是Lock接口

### 2.2.1 Lock接口介绍

Lock锁实现提供了比使用同步方法和语句可以获得的更广泛的锁操作。它们允许更灵活的结构，可能具有非常不同的属性，并且可能支持多个关联的条件对象。Lock提供了比 synchronized更多的功能。

### 2.2.2 Lock实现可重入锁

```java
package com.sxq.lock;

import java.util.concurrent.locks.ReentrantLock;

/**
 * @author song
 * @version 1.0
 * @date 2023/1/3 18:57
 * @description: TODO
 */
class LTicket{
    //票数
    private int number = 30;

    //创建可重入锁
    private final ReentrantLock lock = new ReentrantLock();
    //操作方法 卖票
    public void sale(){
        //上锁
        lock.lock();
        try {
            if(number > 0){
                System.out.println(Thread.currentThread().getName() + " : 卖出 : " + number-- + " 剩下 " + number);
            }
        }catch (Exception e){
            e.printStackTrace();
        }
        finally {
            //解锁
            lock.unlock();
        }

    }
}
public class LSaleTicket {
    public static void main(String[] args) {
        LTicket lTicket = new LTicket();
        new Thread(()->{
            for (int i = 0; i < 40; i++) {
                lTicket.sale();
            }
        },"AA").start();
        new Thread(()->{
            for (int i = 0; i < 40; i++) {
                lTicket.sale();
            }
        },"BB").start();
        new Thread(()->{
            for (int i = 0; i < 40; i++) {
                lTicket.sale();
            }
        },"CC").start();
    }
}

```

### 2.2.3 Lock 和 Synchronized不同
1. Lock是一个接口，而synchronized是Java中的关键字，synchronized是内置的语言实现;
2. synchronized在发生异常时，会自动释放线程占有的锁，因此不会导致死锁现象发生;而Lock在发生异常时，如果没有主动通过unLock(去释放锁，则很可能造成死锁现象，因此使用Lock时需要在finally 块中释放锁;。
3. Lock 可以让等待锁的线程响应中断，而synchronized 却不行，使用synchronized时，等待的线程会一直等待下去，不能够响应中断;·
4. 通过Lock可以知道有没有成功获取锁，而synchronized却无法办到。。
5. Lock可以提高多个线程进行读操作的效率。


## 2.3 创建多线程的多种方式
### 2.3.1 继承Thread类
### 2.3.2 实现Runnable接口
### 2.3.3 使用Callable接口
### 2.3.4 使用线程池

# 3、线程间通信

### 例子
> 两个线程实现对一个初始值0的变量  一个线程+1、一个线程-1（Synchronized实现）

###  （1.Synchronized实现）

```java
package com.sxq.sync;

/**
 * @author song
 * @version 1.0
 * @date 2023/1/7 10:57
 * @description: 两个线程实现对一个初始值0的变量  一个线程+1、一个线程-1 Synchronized实现
 */

//创建资源类
class Share{
    //创建资源
    private int number = 0;

    //+1的方法
    public synchronized void incr() throws InterruptedException {
        //判断 干活 通知
        if(number != 0){
            //判断number值是否是0，如果不是o，等待
            this.wait();
        }
        //如果number是0 干活
        number++;
        System.out.println(Thread.currentThread().getName() + " :: " + number);
        //通知 唤醒等待的线程
        this.notifyAll();
    }

    //-1的方法
    public synchronized void decr() throws InterruptedException {
        //判断 干活 通知
        if(number != 1){
            this.wait();
        }
        number--;
        System.out.println(Thread.currentThread().getName() + " :: " + number);
        //通知 唤醒等待的线程
        this.notifyAll();
    }
}
public class ThreadDemo1 {
    public static void main(String[] args) {
        Share share = new Share();

        new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                try {
                    share.incr();
                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                }
            }
        },"AA").start();

        new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                try {
                    share.decr();
                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                }
            }
        },"BB").start();
    }
}

```

### 出现问题 虚假唤醒

wait会释放锁

1. 当一个AA线程抢到+1，number=1，然后唤醒其他线程，如果CC此时抢到，发现number=1，于是CC线程进入等待。

2. 如果此时AA再次抢到，然后发现number=1，于是AA线程也进入等待，然后CC再次抢到，CC线程从等待的地方唤醒，不在执行if判断此时就出现number=2了。这就是虚假唤醒。

解决方式：将调用wait的地方循环

```java
    //+1的方法
    public synchronized void incr() throws InterruptedException {
        //判断 干活 通知
        while (number != 0){
            //判断number值是否是0，如果不是o，等待
            this.wait();
        }
        //如果number是0 干活
        number++;
        System.out.println(Thread.currentThread().getName() + " :: " + number);
        //通知 唤醒等待的线程
        this.notifyAll();
    }

    //-1的方法
    public synchronized void decr() throws InterruptedException {
        //判断 干活 通知
        while (number != 1){
            this.wait();
        }
        number--;
        System.out.println(Thread.currentThread().getName() + " :: " + number);
        //通知 唤醒等待的线程
        this.notifyAll();
    }
```

### （2.Lock实现）

```java
package com.sxq.lock;

import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.ReentrantLock;

/**
 * @author song
 * @version 1.0
 * @date 2023/1/7 14:50
 * @description: 两个线程实现对一个初始值0的变量  一个线程+1、一个线程-1 Lock实现
 */
//共享资源
class Share{
    private int number = 0;
    //创建Lock
    private final ReentrantLock lock = new ReentrantLock();
    private Condition condition = lock.newCondition();
    //+1操作
    public void incr() throws InterruptedException {
        //上锁
        lock.lock();
        try{
            //判断
            while (number != 0){
                condition.await();
            }
            //干活
            number++;
            System.out.println(Thread.currentThread().getName() + " :: " + number);
            //通知
            condition.signalAll();
        }finally {
            //解锁
            lock.unlock();
        }
    }
    //-1操作
    public void decr() throws InterruptedException {
        lock.lock();
        try{
            while (number != 1){
                condition.await();
            }
            
            number--;
            System.out.println(Thread.currentThread().getName() + " :: " + number);

            condition.signalAll();
        }finally {
            lock.unlock();
        }
    }

}
public class ThreadDemo1 {
    public static void main(String[] args) {
        Share share = new Share();

        new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                try {
                    share.incr();
                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                }
            }
        },"AA").start();

        new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                try {
                    share.decr();
                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                }
            }
        },"BB").start();
        new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                try {
                    share.incr();
                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                }
            }
        },"CC").start();

        new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                try {
                    share.decr();
                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                }
            }
        },"DD").start();
    }
}
```

# 4、线程间定制化通信



### 实现AA线程执行打印5次 ==》BB线程执行打印10次==》CC线程打印15次

```java
package com.sxq.lock;

import javax.swing.*;
import java.util.Locale;
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

/**
 * @author song
 * @version 1.0
 * @date 2023/1/7 15:29
 * @description: 线程间定制化通讯  
 */

//资源类
class ShareResource {
    //定义标志位 1=AA 2=BB 3=CC
    private int flag = 1;
    //创建Lock锁
    private Lock lock = new ReentrantLock();

    //创建三个condition
    private Condition c1 = lock.newCondition();
    private Condition c2 = lock.newCondition();
    private Condition c3 = lock.newCondition();

    //操作方法
    //打印5次，参数第几轮
    public void print5(int loop){
        //上锁
        lock.lock();
        try {
            //判断
            while (flag != 1){
                //等待
                c1.await();
            }
            //干活
            for (int i = 0; i < 5; i++) {
                System.out.println(Thread.currentThread().getName()+" :: " + i + "轮数:" + loop);
            }
            //通知
            flag = 2; //先修改标志位
            c2.signal(); //通知BB线程
        } catch (InterruptedException e) {
            throw new RuntimeException(e);
        } finally {
            //释放锁
            lock.unlock();
        }
    }

    //打印10次，参数第几轮
    public void print10(int loop){
        //上锁
        lock.lock();
        try {
            //判断
            while (flag != 2){
                //等待
                c2.await();
            }
            //干活
            for (int i = 0; i < 10; i++) {
                System.out.println(Thread.currentThread().getName()+" :: " + i + "轮数:" + loop);
            }
            //通知
            flag = 3; //先修改标志位
            c3.signal(); //通知BB线程
        } catch (InterruptedException e) {
            throw new RuntimeException(e);
        } finally {
            //释放锁
            lock.unlock();
        }
    }

    //打印10次，参数第几轮
    public void print15(int loop){
        //上锁
        lock.lock();
        try {
            //判断
            while (flag != 3){
                //等待
                c3.await();
            }
            //干活
            for (int i = 0; i < 15; i++) {
                System.out.println(Thread.currentThread().getName()+" :: " + i + "轮数:" + loop);
            }
            //通知
            flag = 1; //先修改标志位
            c1.signal(); //通知AA线程
        } catch (InterruptedException e) {
            throw new RuntimeException(e);
        } finally {
            //释放锁
            lock.unlock();
        }
    }
}

public class ThreadDemo3 {
    public static void main(String[] args) {
        ShareResource shareResource = new ShareResource();
        new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                shareResource.print5(i);
            }
        },"AA").start();
        new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                shareResource.print10(i);
            }
        },"BB").start();
        new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                shareResource.print15(i);
            }
        },"CC").start();
    }
}

```

 

# 5、集合的线程安全

### 5.1 集合线程不安全演示

```java

public class ThreadDemo4 {

    public static void main(String[] args) {
        //创建集合
        List<String> list = new ArrayList<>();

        for (int i = 0; i < 10; i++) {
            new Thread(()->{
                //向集合中添加内容
                list.add(UUID.randomUUID().toString().substring(0,8));
                //从集合获取内容
                System.out.println(list);
            },String.valueOf(i)).start();
        }
    }
}

==============
    出现异常 java.util.ConcurrentModificationException 并发修改异常
```

#### 5.1.1 解决方案Vector

```java
  public static void main(String[] args) {
        //创建集合
//        List<String> list = new ArrayList<>();
        //Vector解决 不推荐
        List<String> list = new Vector<>();
        //Collections解决
//        List<String> list = Collections.synchronizedList(new ArrayList<>());
        //CopyOnWriteArrayList 写时复制
//        List<String> list = new CopyOnWriteArrayList<>();
        for (int i = 0; i < 30; i++) {
            new Thread(()->{
                //向集合中添加内容
                list.add(UUID.randomUUID().toString().substring(0,8));
                //从集合获取内容
                System.out.println(list);
            },String.valueOf(i)).start();
        }
    }
```



#### 5.1.2 解决方案Collections

```java
 public static void main(String[] args) {
        //创建集合
//        List<String> list = new ArrayList<>();
        //Vector解决 不推荐
//        List<String> list = new Vector<>();
        //Collections解决
        List<String> list = Collections.synchronizedList(new ArrayList<>());
        //CopyOnWriteArrayList 写时复制
//        List<String> list = new CopyOnWriteArrayList<>();
        for (int i = 0; i < 30; i++) {
            new Thread(()->{
                //向集合中添加内容
                list.add(UUID.randomUUID().toString().substring(0,8));
                //从集合获取内容
                System.out.println(list);
            },String.valueOf(i)).start();
        }
    }
```



#### 5.1.3 解决方案CopyOnWriteArrayList

写时复制，可以并发读数据，但是当写入时会复制一份原本的数据，在副本添加新写入的数据，最后将副本与原来的合并。

```java
public static void main(String[] args) {
        //创建集合
//        List<String> list = new ArrayList<>();
        //Vector解决 不推荐
//        List<String> list = new Vector<>();
        //Collections解决
//        List<String> list = Collections.synchronizedList(new ArrayList<>());
        //CopyOnWriteArrayList 写时复制
        List<String> list = new CopyOnWriteArrayList<>();
        for (int i = 0; i < 30; i++) {
            new Thread(()->{
                //向集合中添加内容
                list.add(UUID.randomUUID().toString().substring(0,8));
                //从集合获取内容
                System.out.println(list);
            },String.valueOf(i)).start();
        }
    }
```

### 5.2 HashSet线程不安全演示

```java
  public static void main(String[] args) {
        //演示HashSet
        Set<String> set = new HashSet<>();

        for (int i = 0; i < 30; i++) {
            new Thread(()->{
                //向集合中添加内容
                set.add(UUID.randomUUID().toString().substring(0,8));
                //从集合获取内容
                System.out.println(set);
            },String.valueOf(i)).start();
        }
    }
```

一样的并发修改异常java.util.ConcurrentModificationException

#### 5.2.1 解决方式 CopyOnWriteArraySet

```java
public static void main(String[] args) {
        //演示HashSet
        Set<String> set = new CopyOnWriteArraySet<>();
        for (int i = 0; i < 30; i++) {
            new Thread(()->{
                //向集合中添加内容
                set.add(UUID.randomUUID().toString().substring(0,8));
                //从集合获取内容
                System.out.println(set);
            },String.valueOf(i)).start();
        }
    }
```

### 5.3 HashMap线程不安全演示

```java
public static void main(String[] args) {
        Map<String,String> map = new HashMap<>();

        for (int i = 0; i < 30; i++) {
            String key = String.valueOf(i);
            new Thread(()->{
                //向集合中添加内容
                map.put(key,UUID.randomUUID().toString().substring(0,8));
                //从集合获取内容
                System.out.println(map);
            },String.valueOf(i)).start();
        }
    }
并发修改异常
```

#### 5.3.1 解决方案 ConcurrentHashMap

```java
public static void main(String[] args) {
        Map<String,String> map = new ConcurrentHashMap<>();
        for (int i = 0; i < 30; i++) {
            String key = String.valueOf(i);
            new Thread(()->{
                //向集合中添加内容
                map.put(key,UUID.randomUUID().toString().substring(0,8));
                //从集合获取内容
                System.out.println(map);
            },String.valueOf(i)).start();
        }
    }
```

# 6、多线程锁

### 6.1 锁的八种情况

synchronized实现同步的基础:Java中的每一个对象都可以作为锁具体表现为以下3种形式。

- 对于普通同步方法，锁是当前实例对象。（调用方法的对象本身 this）
- 对于静态同步方法，锁是当前类的Class对象。
- 对于同步方法块，锁是synchonized括号里配置的对象

```java
package com.sxq.sync;

import java.util.concurrent.TimeUnit;

/**
 * @author song
 * @version 1.0
 * @date 2023/1/8 10:16
 * @description: 多线程锁
 */

class Phone {
    public static synchronized void sendSMs() throws Exception {
        //停留4秒
        TimeUnit.SECONDS.sleep(4);
        System.out.println("------sendSMS");
    }

    public  synchronized void sendEmail() throws Exception {
        System.out.println("------sendEmail");
    }

    public void getHello() {
        System.out.println("------getHello");
    }


}

/**
 * @Description: 8锁
 * <p>
 * 1 标准访问，先打印短信还是邮件
 * ------sendSMS
 * ------sendEmail
 * <p>
 * 2 停4秒在短信方法内，先打印短信还是邮件
 * ------sendSMS
 * ------sendEmail
 * <p>
 * 3 新增普通的hello方法，是先打短信还是helLo
 * ------getHello
 * ------sendSMS
 * <p>
 * 4 现在有两部手机，先打印短信还是邮件
 * ------sendEmail
 * ------sendSMS
 * <p>
 * 5 两个静态同步方法，1部手机，先打印短信还是邮件
 * ------sendSMS
 * ------sendEmail
 * <p>
 * 6 两个静态同步方法，2部手机，先打印短信还是邮件
 * ------sendSMS
 * ------sendEmail
 * <p>
 * 7 1个静态同步方法,1个普通同步方法,1部手机，先打印短信还是邮件
 * ------sendEmail
 * ------sendSMS
 * <p>
 * 8 1个静态同步方法1个普通同步方法,2部手机，先打印短信还是邮件
 * ------sendEmail
 * ------sendSMS
 **/
public class Lock_8 {
    public static void main(String[] args) throws InterruptedException {
        Phone phone1 = new Phone();
        Phone phone2 = new Phone();
        new Thread(() -> {
            try {
                phone1.sendSMs();
            } catch (Exception e) {
                e.printStackTrace();
            }
        },"AA").start();
        Thread.sleep(100);
        new Thread(() -> {
            try {
//                phone1.sendEmail();
//                phone1.getHello();
                phone2.sendEmail();
            } catch (Exception e) {
                e.printStackTrace();
            }
        },"BB").start();
    }
}
```

### 6.2 公平锁和非公平锁

创建一个可重入锁

```java
//创建可重入锁
private final ReentrantLock lock = new ReentrantLock();  
```


ReentrantLock的源码，当创建锁不传值。默认为非公平锁

```java
//无参构造
public ReentrantLock() {
        sync = new NonfairSync();
    }
//有参构造
public ReentrantLock(boolean fair) {
        sync = fair ? new FairSync() : new NonfairSync();
    }

//非公平锁
    static final class NonfairSync extends Sync {
        private static final long serialVersionUID = 7316153563782823691L;

        /**
         * Performs lock.  Try immediate barge, backing up to normal
         * acquire on failure.
         */
        final void lock() {
            if (compareAndSetState(0, 1))
                setExclusiveOwnerThread(Thread.currentThread());
            else
                acquire(1);
        }

        protected final boolean tryAcquire(int acquires) {
            return nonfairTryAcquire(acquires);
        }
    }

//公平锁
static final class FairSync extends Sync {
        private static final long serialVersionUID = -3000897897090466540L;

        final void lock() {
            acquire(1);
        }

        /**
         * Fair version of tryAcquire.  Don't grant access unless
         * recursive call or no waiters or is first.
         */
        protected final boolean tryAcquire(int acquires) {
            final Thread current = Thread.currentThread();
            int c = getState();
            if (c == 0) {
                if (!hasQueuedPredecessors() &&
                    compareAndSetState(0, acquires)) {
                    setExclusiveOwnerThread(current);
                    return true;
                }
            }
            else if (current == getExclusiveOwnerThread()) {
                int nextc = c + acquires;
                if (nextc < 0)
                    throw new Error("Maximum lock count exceeded");
                setState(nextc);
                return true;
            }
            return false;
        }
    }
```

- 非公平锁

> 可能会导致线程饿死、效率高

- 公平锁

> 每个线程都会有机会，效率相对较低

公平锁和非公平锁都是排序队列的，但是公平锁的新创建线程会排到所有的就绪队列之后，非公平锁的线程和就绪队列直接竞争资源，插队。

### 6.3 可重入锁

synchronized（隐式）、Lock（显式）都是可重入锁 | 所谓显式隐式指的是是否需要手动申明加锁解锁

> 是同一把锁，进入后可以无障碍访问其他

```java
1.===============   
     //synchronized演示可重入锁
public synchronized void add(){
        add();
    }

    public static void main(String[] args) {
		//通过递归调用自身，运行爆出堆栈溢出Exception in thread "main" java.lang.StackOverflowError
        //说明synchronized可重入(递归锁)
        new SyncLockDemo().add();
    }
2.===============
    public static void main(String[] args) {
        Object o = new Object();
        new Thread(()->{
            synchronized (o){
                System.out.println(Thread.currentThread().getName() + " 外层");
                synchronized (o){
                    System.out.println(Thread.currentThread().getName() + " 中层");
                    synchronized (o){
                        System.out.println(Thread.currentThread().getName() + " 内层");
                    }
                }
            }
        },"t1").start();
    }
```

```java
    public static void main(String[] args) {
        //lock演示可重入锁
        ReentrantLock lock = new ReentrantLock();
        //创建线程
        new Thread(()->{
            try {
                //枷锁
                lock.lock();
                System.out.println(Thread.currentThread().getName() + "外层");

                try {
                    //枷锁
                    lock.lock();
                    System.out.println(Thread.currentThread().getName() + "内层");
                }finally {
                    //释放锁 只是为了演示可重入性 一定记得加锁解锁
//                    lock.unlock();
                }

            }finally {
                //释放锁
                lock.unlock();
            }
        },"lock").start();
    }
```

### 6.4  死锁

1. 什么是死锁?：

>  两个或两个以上进程在执行过程中，因为争夺资源而造成互相等待的现象，如果没有外力干涉，进程无法继续再执行下去。

2. 产生死锁原因

> 第一 系统资源不足
>
> 第二 进程运行顺序不合适
>
> 第三 资源分配不当

#### 演示死锁

```java

public class DeadLock {

    //创建两个对象
    static Object a = new Object();
    static Object b = new Object();

    public static void main(String[] args) {
        new Thread(()->{
            synchronized (a){
                System.out.println(Thread.currentThread().getName() + "持有锁a，试图获取锁b");
                synchronized (b){
                    System.out.println(Thread.currentThread().getName() + "获取锁b");
                }
            }
        },"A").start();

        new Thread(()->{
            synchronized (b){
                System.out.println(Thread.currentThread().getName() + "持有锁b，试图获取锁a");
                synchronized (a){
                    System.out.println(Thread.currentThread().getName() + "获取锁a");
                }
            }
        },"B").start();
    }
}

=====结果
    A持有锁a，试图获取锁b
	B持有锁b，试图获取锁a

```

####     验证是否为死锁

通过 jps -l  命令查看当前进程的进程号

然后通过 jstack 进程号 查看堆栈情况

```
Java stack information for the threads listed above:
===================================================
"B":
        at com.sxq.sync.DeadLock.lambda$main$1(DeadLock.java:30)
        - waiting to lock <0x000000076b6a3e58> (a java.lang.Object)
        - locked <0x000000076b6a3e68> (a java.lang.Object)
        at com.sxq.sync.DeadLock$$Lambda$2/1078694789.run(Unknown Source)
        at java.lang.Thread.run(Thread.java:750)
"A":
        at com.sxq.sync.DeadLock.lambda$main$0(DeadLock.java:21)
        - waiting to lock <0x000000076b6a3e68> (a java.lang.Object)
        - locked <0x000000076b6a3e58> (a java.lang.Object)
        at com.sxq.sync.DeadLock$$Lambda$1/1324119927.run(Unknown Source)
        at java.lang.Thread.run(Thread.java:750)

Found 1 deadlock.
```



# 7、Callable接口

目前我们学习了有两种创建线程的方法一种是通过创建Thread类，另一种是通过使用Runnable创建线程。但是，Runnable缺少的一项功能是，当线程终止时（即run ( )完成时），我们无法使线程返回结果。为了支持此功能，Java中提供了Callable接口。

Callable接口的特点如下(重点)

- 为了实现Runnable，需要实现不返回任何内容的run ( )方法，而对于
  Callable，需要实现 在完成时返回结果的call ( )方法。
- call ( )方法可以引发异常，而run ( )则不能。
- 为实现Callable而必须重写call方法

```java
package com.sxq.callable;

import java.util.concurrent.Callable;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.FutureTask;

/**
 * @author song
 * @version 1.0
 * @date 2023/1/8 17:41
 * @description: 比较Callable和Runnable
 */
//实现Runnable接口
class MyThread1 implements Runnable{

    @Override
    public void run() {

    }
}
//实现Callable接口
class MyThread2 implements Callable{

    @Override
    public Integer call() throws Exception {
        System.out.println(Thread.currentThread().getName() + " come in callable");
        return 200;
    }
}
public class Demo1 {

    public static void main(String[] args) throws ExecutionException, InterruptedException {
        //Runnable 创建线程
//        new Thread(new MyThread1(),"AA").start();

        //Callable 创建线程需要 借助FutureTask
        //FutureTask 创建方式一
        FutureTask<Integer> futureTask1 = new FutureTask<>(new MyThread2());
        //lambda表达式 创建方式二
        FutureTask<Integer> futureTask2 = new FutureTask<>(()->{
            System.out.println(Thread.currentThread().getName() + " come in callable");
            return 1024;
        });
        //FutureTask原理 未来任务 单开一个线程去做要做的任务

        //创建一个线程
        new Thread(futureTask2,"lucy").start();
        new Thread(futureTask1,"marry").start();
//        while (!futureTask2.isDone()){
//            System.out.println("wait......");
//        }
        //调用FutureTask中的get方法
        System.out.println(futureTask2.get());
        System.out.println(futureTask1.get());

        System.out.println(Thread.currentThread().getName() + " come over");
    }
}

```



# 8、JUC强大的辅助类

### 8.1 减少计数CountDownLatch

CountDownLatch类可以设置一个计数器，然后通过countDown方法来进行减1的操作，使用await方法等待计数器不大于0，然后继续执行await方法之后的语句。

- CountDownLatch主要有两个方法，当一个或多个线程调用await方法时，这
  些线程会阻塞
- 其它线程调用countDown方法会将计数器减1(调用countDown方法的线程
  不会阻塞)。
- 当计数器的值变为0时，因await方法阻塞的线程会被唤醒，继续执行

演示

```java
/**
 * @author song
 * @version 1.0
 * @date 2023/1/10 9:25
 * @description: 演示CountDownLatch
 * 例子：6个同学陆续离开教室之后，班长锁门
 */

public class CountDownDemo {
    public static void main(String[] args) throws InterruptedException {
        //问题演示
//        for (int i = 0; i < 6; i++) {
//            new Thread(()->{
//                System.out.println(Thread.currentThread().getName() + "号同学离开教室");
//            },String.valueOf(i)).start();
//        }
//        System.out.println(Thread.currentThread().getName() + "班长锁门");
        /*
            0号同学离开教室
            3号同学离开教室
            5号同学离开教室
            2号同学离开教室
            4号同学离开教室
            main班长锁门
            1号同学离开教室
         */

        //解决方式 CountDownLatch

        //创建CountDownLatch对象，设置初始值
        CountDownLatch latch = new CountDownLatch(6);
        for (int i = 0; i < 6; i++) {
            new Thread(()->{
                System.out.println(Thread.currentThread().getName() + "号同学离开教室");
                //计数-1
                latch.countDown();
            },String.valueOf(i)).start();
        }
        //等待
        latch.await();
        System.out.println(Thread.currentThread().getName() + "班长锁门");
    }
}

```



### 8.2 循环栅栏CyclicBarrier

CyclicBarrier看英文单词可以看出大概就是循环阻塞的意思，在使用中
CyclicBarrier 的构造方法第一个参数是目标障碍数，每次执行CyclicBarrier一次障碍数会加一，如果达到了目标障碍数，才会执行cyclicBarrier.await(之后的语句。可以将CyclicBarrier理解为加1操作。 

```java
/**
 * @author song
 * @version 1.0
 * @date 2023/1/10 9:37
 * @description: CyclicBarrier演示
 * 例子: 集齐7颗龙珠召唤神龙
 */

public class CyclicBarrierDemo {
    //创建固定值
    private static final int NUMBER = 7;

    public static void main(String[] args) {
        CyclicBarrier cyclicBarrier = new CyclicBarrier(NUMBER,()->{
            System.out.println("集齐7颗龙珠就可以召唤神龙");
        });

        //集齐龙珠的过程
        for (int i = 0; i < 7; i++) {
            new Thread(()->{
                try {
                    System.out.println(Thread.currentThread().getName() + " 星龙珠");
                    //等待集齐七颗龙珠
                    cyclicBarrier.await();
                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                } catch (BrokenBarrierException e) {
                    throw new RuntimeException(e);
                }
            },String.valueOf(i)).start();
        }
    }
}
```



### 8.3 信号灯Semaphore

```java
package com.sxq.juc;

import java.util.Random;
import java.util.concurrent.Semaphore;
import java.util.concurrent.TimeUnit;

/**
 * @author song
 * @version 1.0
 * @date 2023/1/10 9:48
 * @description: SemaphoreDemo信号灯演示
 * 例子： 6辆汽车 停3个停车位
 */

public class SemaphoreDemo {

    public static void main(String[] args) {
        //创建Semaphore，设置许可证
        Semaphore semaphore = new Semaphore(3);
        
        //模拟6辆汽车
        for (int i = 0; i < 6; i++) {
            new Thread(()->{
                //抢占车位
                try {
                    semaphore.acquire();
                    System.out.println(Thread.currentThread().getName()+ " 抢到了车位");
                    //设置随机停车时间
                    TimeUnit.SECONDS.sleep(new Random().nextInt(5));
                    System.out.println(Thread.currentThread().getName()+ " ------离开了车位");

                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                }finally {
                    //释放
                    semaphore.release();
                }
            },String.valueOf(i)).start();
        }
    }
}
```

# 9、ReentrantReadWriteLock读写锁

读写锁:一个资源可以被多个读线程访问，或者可以被一个写线程访问，但是不能同时存在读写线程，读写互斥，读读共享的

读锁 ： 共享锁（都可能发生死锁）

写锁 ： 独占锁（都可能发生死锁）

```java
package com.sxq.readwrite;

import java.util.HashMap;
import java.util.Map;
import java.util.concurrent.TimeUnit;
import java.util.concurrent.locks.ReentrantReadWriteLock;

/**
 * @author song
 * @version 1.0
 * @date 2023/1/10 10:08
 * @description: 读写锁
 */
//创建资源类
class MyCache {
    //创建Map集合
    private volatile Map<String, Object> map = new HashMap<>();

    //创建读写锁
    private ReentrantReadWriteLock rwLock = new ReentrantReadWriteLock();

    //放数据
    public void put (String key, Object value) throws InterruptedException {
        //加写锁
        rwLock.writeLock().lock();
        try {
            System.out.println(Thread.currentThread().getName() + " 正在写操作 " + key);
            //等一会
            TimeUnit.MICROSECONDS.sleep(300);
            map.put(key,value);
            System.out.println(Thread.currentThread().getName() + " 写完了 " + key);
        }catch (InterruptedException e ){
            e.printStackTrace();
        }finally {
            rwLock.writeLock().unlock();
        }
    }

    //取数据
    public Object get(String key) throws InterruptedException {
        //加读锁
        rwLock.readLock().lock();
        Object result = null;
        try {
            System.out.println(Thread.currentThread().getName() + " 正在读取操作 " + key);
            //等一会
            TimeUnit.MICROSECONDS.sleep(300);
            result = map.get(key);
            System.out.println(Thread.currentThread().getName() + " 读完了 " + key);
        }catch (InterruptedException e){
            e.printStackTrace();
        }finally {
            rwLock.readLock().unlock();
        }
        return result;
    }
}

public class ReadWriteLockDemo {

    public static void main(String[] args) {
        MyCache myCache = new MyCache();
        //放
        for (int i = 1; i <= 5; i++) {
            final int num = i;
            new Thread(()->{
                try {
                    myCache.put(num+"",num+"");
                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                }
            },String.valueOf(i)).start();
        }
        //取
        for (int i = 1; i <= 5; i++) {
            final int num = i;
            new Thread(()->{
                try {
                    myCache.get(num+"");
                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                }
            },String.valueOf(i)).start();
        }
            /*
                2 正在写操作 2
                2 写完了 2
                3 正在写操作 3
                3 写完了 3
                4 正在写操作 4
                4 写完了 4
                1 正在写操作 1
                1 写完了 1
                5 正在写操作 5
                5 写完了 5
                1 正在读取操作 1
                2 正在读取操作 2
                3 正在读取操作 3
                4 正在读取操作 4
                5 正在读取操作 5
                3 读完了 3
                2 读完了 2
                5 读完了 5
                4 读完了 4
                1 读完了 1
         */
    }
}

```





# 10、BlockingQueue阻塞队列

### 10.1 阻塞队列概述

在多线程领域︰所谓阻塞，在某些情况下会挂起线程（即阻塞），一旦条件满足，被挂起的线程又会自动被唤起
为什么需要BlockingQueue?

> 好处是我们不需要关心什么时候需要阻塞线程，什么时候需要唤醒线程，因为这一切BlockingQueue都给你一手包办了。
> 在concurrent包发布以前，在多线程环境下，我们每个程序员都必须去自己控制这些细节，尤其还要兼顾效率和线程安全，而这会给我们的程序带来不小的复杂度。



### 10.2 阻塞队列的架构

队列(先进先出)

- 当队列是空的，从队列中获取元素的操作将会被阻塞
- 当队列是满的，从队列中添加元素的操作将会被阻塞
- 试图从空的队列中获取元素的线程将会被阻塞，直到其他线程往空的队列插入新的元素·
- 试图向已满的队列中添加新元素的线程将会被阻塞，直到其他线程从队列中移除一个或多个元素或者完全清空，使队列变得空闲起来并后续新增

### 10.3 阻塞队列分类

#### 10.3.1 ArrayBlockingQueue(常用)

基于数组的阻塞队列实现，在ArrayBlockingQueue内部，维护了一个定长数组，以便缓存队列中的数据对象，这是一个常用的阻塞队列，除了一个定长数组外，ArrayBlockingQueue内部还保存着两个整形变量，分别标识着队列的头部和尾部在数组中的位置。

**总结：数组结构组成的有界阻塞队列**

#### 10.3.2 LinkedBlockingQueue

总结：有链表结构组成的有界阻塞队列（但默认值大小为integer.MAX-VALUE）

#### 10.3.3 DelayQueue

DelayQueue中的元素只有当其指定的延迟时间到了，才能够从队列中获取到该元素。DelayQueue是一个没有大小限制的队列，因此往队列中插入数据的操作(生产者)永远不会被阻塞，而只有获取数据的操作（消费者)才会被阻塞。

**总结:使用优先级队列实现的延迟无界阻塞队列。**

#### 10.3.4 

基于优先级的阻塞队列(优先级的判断通过构造函数传入的Compator对象来决定），但需要注意的是PriorityBlockingQueue并不会阻塞数据生产者，而只会在没有可消费的数据时，阻塞数据的消费者。
因此使用的时候要特别注意，生产者生产数据的速度绝对不能快于消费者消费数据的速度，否则时间一长，会最终耗尽所有的可用堆内存空间。
在实现 PriorityBlockingQueue时，内部控制线程同步的锁采用的是公平锁。

**总结:支持优先级排序的无界阻塞队列。**

#### 10.3.5 SynchronousQueue

**不存储元素的阻塞队列，也即单个元素的队列。**

#### 10.3.6 LinkedTransferQueue

LinkedTransferQueue是一个由链表结构组成的无界阻塞TransferQueue队列。相对于其他阻塞队列，LinkedTransferQueue多了 tryTransfer和transfer方法。
LinkedTransferQueue采用一种预占模式。意思就是消费者线程取元素时，如果队列不为空，则直接取走数据，若队列为空，那就生成一个节点(节点元素为null )入队，然后消费者线程被等待在这个节点上，后面生产者线程入队时发现有一个元素为null的节点，生产者线程就不入队了，直接就将元素填充到该节点，并唤醒该节点等待的线程，被唤醒的消费者线程取走元素，从调用的方法返回。
**总结:由链表组成的无界阻塞队列**

#### 10.3.7 LinkedBlockingDeque

LinkedBlockingDeque是一个由链表结构组成的双向阻塞队列，即可以从队列的两端插入和移除元素。
对于一些指定的操作，在插入或者获取队列元素时如果队列状态不允许该操作可能会阻塞住该线程直到队列状态变更为允许操作，这里的阻塞一般有两种情况

**总结:由链表组成的双向阻塞队列**

### 10.4 阻塞队列核心方法

| 方法类型 | 抛出异常  | 特殊值   | 阻塞    | 超时               |
| -------- | --------- | -------- | ------- | ------------------ |
| 插入     | add(e)    | offer(e) | put(e)  | offer(e,time,unit) |
| 移除     | remove(e) | poll(e)  | take(e) | poll(time,unit)    |
| 检查     | element() | peek()   | 不可用  | 不可用             |

| 抛出异常 | 当阻塞队列满时，再往队列里add插入元素会抛IllegalStateException:Queue full<br>当阻塞队列空时，再往队列里remove移除元素会抛NoSuchElementException |
| -------- | ------------------------------------------------------------ |
| 特殊值   | 插入方法，成功ture失败false <br>移除方法，成功返回出队列的元素，队列里没有就返回null |
| 一直阻塞 | 当阻塞队列满时，生产者线程继续往队列里put元素，队列会一直阻塞生产者线程直到put数据or响应中断艮出<br/>当阻塞队列空时，消费者线程试图从队列里take元素，队列会一直阻塞消费者线程直到队列可用 |
| 超时退出 | 当阻塞队列满时，队列会阻塞生产者线程一定时间，超过限时后生产者线程会退出 |

```java
    public static void main(String[] args) throws InterruptedException {
        //创建一个阻塞队列
        BlockingQueue<String> blockingQueue = new ArrayBlockingQueue<>(3);

        /**
         * 第一组
         */
        System.out.println(blockingQueue.add("a"));
        System.out.println(blockingQueue.add("b"));
        System.out.println(blockingQueue.add("c"));
        System.out.println(blockingQueue.element());
//        System.out.println(blockingQueue.add("d"));
        
//        true
//        true
//        true
//        a
//        Exception in thread "main" java.lang.IllegalStateException: Queue full
//        at java.util.AbstractQueue.add(AbstractQueue.java:98)
//        at java.util.concurrent.ArrayBlockingQueue.add(ArrayBlockingQueue.java:312)
//        at com.sxq.queue.BlockingQueueDemo.main(BlockingQueueDemo.java:22)
         
        System.out.println(blockingQueue.remove());
        System.out.println(blockingQueue.remove());
        System.out.println(blockingQueue.remove());
        System.out.println(blockingQueue.remove());
        
//        a
//        b
//        c
//        Exception in thread "main" java.util.NoSuchElementException
//            at java.util.AbstractQueue.remove(AbstractQueue.java:117)
//            at com.sxq.queue.BlockingQueueDemo.main(BlockingQueueDemo.java:37)
          /**
         * 第二组
         */
        System.out.println(blockingQueue.offer("a"));
        System.out.println(blockingQueue.offer("b"));
        System.out.println(blockingQueue.offer("c"));
        System.out.println(blockingQueue.offer("www"));
        
        //true
        //true
        //true
        //false
         

        System.out.println(blockingQueue.poll());
        System.out.println(blockingQueue.poll());
        System.out.println(blockingQueue.poll());
        System.out.println(blockingQueue.poll());
        
       // a
       // b
       // c
       // null
         /**
         * 第三组
         */
        blockingQueue.put("a");
        blockingQueue.put("b");
        blockingQueue.put("c");
		//blockingQueue.put("d");
        System.out.println(blockingQueue.take());
        System.out.println(blockingQueue.take());
        System.out.println(blockingQueue.take());
        System.out.println(blockingQueue.take());
        
          /**
         * 第四组
         */
        System.out.println(blockingQueue.offer("a"));
        System.out.println(blockingQueue.offer("b"));
        System.out.println(blockingQueue.offer("c"));
        System.out.println(blockingQueue.offer("d", 3L,TimeUnit.SECONDS ));
        //	true
        //	true
        //	true
        //	false
```



# 11、ThreadPool线程池

### 11.1 线程池概述

线程池（英语:thread pool) :一种线程使用模式。线程过多会带来调度开销，进而影响缓存局部性和整体性能。而线程池维护着多个线程，等待着监督管理者分配可并发执行的任务。这避免了在处理短时间任务时创建与销毁线程的代价。线程池不仅能够保证内核的充分利用，还能防止过分调度。

线程池的优势︰

>  线程池做的工作只要是控制运行的线程数量，处理过程中将任务放入队列，然后在线程创建后启动这些任务，如果线程数量超过了最大数量，超出数量的线程排队等候，等其他线程执行完毕，再从队列中取出任务来执行。

它的主要特点为:

- 降低资源消耗:通过重复利用已创建的线程降低线程创建和销毁造成的销耗。
- 提高响应速度:当任务到达时，任务可以不需要等待线程创建就能立即执行。
- 提高线程的可管理性:线程是稀缺资源，如果无限制的创建，不仅会销耗系统资源，还会降低系统的稳定性，使用线程池可以进行统一的分配，调优和监控。
- Java 中的线程池是通过Executor框架实现的，该框架中用到了 Executor，Executors ,ExecutorService ，ThreadPoolExecutor这几个类

### 11.2 线程池架构

 Executor，Executors ,ExecutorService ，ThreadPoolExecutor

### 11.3 线程池使用方式

Executors.newFixedThreadPool(int) 	一池N线程

Executors.newSingleThreadExecutor()	一个任务一个任务执行，一池一线程

Executors.newCachedThreadPool()	线程池根据需求创建线程，可扩容，遇强则强

#### 11.3.1 newFixedThreadPool

作用∶创建一个可重用固定线程数的线程池，以共享的无界队列方式来运行这些线程。在任意点，在大多数线程会处于处理任务的活动状态。如果在所有线程处于活动状态时提交附加任务，则在有可用线程之前，附加任务将在队列中等待。如果在关闭前的执行期间由于失败而导致任何线程终止，那么一个新线程将代替它执行后续的任务（如果需要)。在某个线程被显式地关闭之前，池中的线程将一直存在。

特征:,

- 线程池中的线程处于一定的量，可以很好的控制线程的并发量
- 线程可以重复被使用，在显示关闭之前，都将一直存在
- 超出一定量的线程被提交时候需在队列中等待

#### 11.3.2 newSingleThreadExecutor

#### 11.3.3 newCachedThreadPool

#### 11.3.4 创建方式

```java
 public static void main(String[] args) {
        //一池5线程
        ExecutorService threadPool1 = Executors.newFixedThreadPool(5); //5个窗口
        //一池一线程
        ExecutorService threadPool2 = Executors.newSingleThreadExecutor(); //一个窗口
        //一池可扩容的线程
        ExecutorService threadPool3 = Executors.newCachedThreadPool();

        //10个顾客
        try {
            for (int i = 1; i <= 10; i++) {
                //执行
                threadPool3.execute(() -> {
                    System.out.println(Thread.currentThread().getName() + " 办理业务");
                });
            }
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            //关闭
            threadPool3.shutdown();
        }
    }
```



### 11.4 线程池底层原理

Executors.newFixedThreadPool

```java
public static ExecutorService newFixedThreadPool(int nThreads) {
    return new ThreadPoolExecutor(nThreads, nThreads,
                                  0L, TimeUnit.MILLISECONDS,
                                  new LinkedBlockingQueue<Runnable>());
}
```

Executors.newSingleThreadExecutor

```java
public static ExecutorService newSingleThreadExecutor() {
    return new FinalizableDelegatedExecutorService
        (new ThreadPoolExecutor(1, 1,
                                0L, TimeUnit.MILLISECONDS,
                                new LinkedBlockingQueue<Runnable>()));
}
```

Executors.newCachedThreadPool

```java
public static ExecutorService newCachedThreadPool() {
    return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                  60L, TimeUnit.SECONDS,
                                  new SynchronousQueue<Runnable>());
}
```

**都是通过ThreadPoolExecutor来创建的**

### 11.5 线程池的七个参数

```java
int corePoolSize,	常驻线程数量（核心）
int maximumPoolSize,	最大线程数量
long keepAliveTime,		线程存活时间
TimeUnit unit,			线程存活时间单位
BlockingQueue<Runnable> workQueue,	阻塞队列
ThreadFactory threadFactory,	线程工厂
RejectedExecutionHandler handler	拒绝策略
```

### 11.6 线程池的底层工作流程

- 主线程调用execute() 方法时才创建线程
- 一开始线程池中的线程数为0，所以会先创建出一个线程，让它去执行一个对应的任务；
- 执行完任务后，这个线程是不会死掉的，它会尝试从一个无界的LinkedBlockingQueue里获取新的任务，如果没有新的任务，就会阻塞住，等待新任务的到来；
- 只要线程池中的线程数量小于corePoolSize，都会直接创建新线程来执行这个任务，执行完了就尝试从无界队列里获取任务，直到线程池中有corePoolSize个线程为止；
- 线程池中线程数达到corePoolSize后，对于新提交的任务，不会再去创建新的线程执行，而是放到队列中去，线程会争抢获取任务执行，如果同一时间所有的线程都在执行任务，那么无界队列中的线程就会越来越多。
- 当阻塞队列中也满了，线程池会创建新线程（达到最大线程数）来处理新的任务
- 如果线程数达到最大，阻塞队列满了，来新的任务就会触发拒绝策略

拒绝策略

- AbortPolicy(默认)∶直接抛出RejectedExecutionExecption异常阻止系统正常运行
- CallerRunsPolicy :"调用者运行"—种调节机制，该策略既不会抛弃任务，也不会抛出异常，而是将某些任务回退到调用者，从而降低新任务的流量。
- DiscardoldestPolicy :抛弃队列中等待最久的任务，然后把当前任务加人队列中尝试再次提交当前任务。
- DiscardPolicy :该策略默默地丢弃无去处理的任务，不予任何处理也不抛出异常。如果允许任务丢失，这是最好的一种策略。

### 11.7 自定义线程池

```java
package com.sxq.pool;

import java.util.concurrent.ArrayBlockingQueue;
import java.util.concurrent.Executors;
import java.util.concurrent.ThreadPoolExecutor;
import java.util.concurrent.TimeUnit;

/**
 * @author song
 * @version 1.0
 * @date 2023/1/10 15:54
 * @description: 自定义线程池
 */

public class CustomThreadPool {
    public static void main(String[] args) {
        ThreadPoolExecutor threadPool = new ThreadPoolExecutor(
                2,
                5,
                3L,
                TimeUnit.SECONDS,
                new ArrayBlockingQueue<>(3),
                Executors.defaultThreadFactory(),
                new ThreadPoolExecutor.AbortPolicy()
        );
        //10个顾客
        try {
            for (int i = 1; i <= 10; i++) {
                //执行
                threadPool.execute(() -> {
                    System.out.println(Thread.currentThread().getName() + " 办理业务");
                });
            }
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            //关闭
            threadPool.shutdown();
        }
    }
}
```

# 12、Fork/Join分支合并框架

### 12.1 Fork/Join 简介

Fork/Join它可以将一个大的任务拆分成多个子任务进行并行处理，最后将子任务结果合并成最后的计算结果，并进行输出。Fork/Join框架要完成两件事情：

> Fork:把一个复杂任务进行分拆，大事化小
> Join:把分拆任务的结果进行合并

```java
package com.sxq.forkjoin;

import java.util.concurrent.ExecutionException;
import java.util.concurrent.ForkJoinPool;
import java.util.concurrent.ForkJoinTask;
import java.util.concurrent.RecursiveTask;

/**
 * @author song
 * @version 1.0
 * @date 2023/1/10 16:14
 * @description: TODO
 */
class MyTask extends RecursiveTask<Integer>{

    //拆分插值不能超过十
    private static final Integer VALUE = 10;
    private int begin;
    private int end;
    private int result;

    //创建一个有参构造
    public MyTask(int begin,int end){
        this.begin = begin;
        this.end = end;
    }

    //拆分和合并
    @Override
    protected Integer compute() {
        //判断相加的两个数差值是否大于十
        if((end - begin) < VALUE){
            //相加
            for(int i = begin; i <= end; i++){
                result = result + i;
            }
        }else {
            //进一步拆分
            //获取数据中间值
            int middle = (begin + end)/2;
            //拆分左边
            MyTask task01 = new MyTask(begin,middle);
            //拆分右边
            MyTask task02 = new MyTask(middle+1,end);
            task01.fork();
            task02.fork();
            result = task01.join() + task02.join();
        }
        return result;
    }
}
public class ForkJoinDemo {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        //创建MyTask对象
        MyTask task = new MyTask(0, 100);
        //创建分支合并池对象
        ForkJoinPool forkJoinPool = new ForkJoinPool();
        ForkJoinTask<Integer> forkJoinTask = forkJoinPool.submit(task);
        //获取最终合并之后的结果
        Integer integer = forkJoinTask.get();
        System.out.println(integer);
        //关闭分支合并池
        forkJoinPool.shutdown();
    }
}

```



# 13、CompletableFuture异步回调

同步：执行某个任务后阻塞，一直等待结果返回

异步：执行某个任务后阻塞，先做其他事，等待任务完成通知再执行。

```java
package com.sxq.completable;

import java.util.concurrent.CompletableFuture;
import java.util.concurrent.ExecutionException;

/**
 * @author song
 * @version 1.0
 * @date 2023/1/10 16:32
 * @description: 异步回调
 */

public class CompletableFutureDemo {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        //异步调用没有返回值
        CompletableFuture<Void> completableFuture1 = CompletableFuture.runAsync(()->{
            System.out.println(Thread.currentThread().getName()+ " completableFuture1");
        });
        completableFuture1.get();

        //异步调用有返回值
        CompletableFuture<Integer> completableFuture2 = CompletableFuture.supplyAsync(()->{
            System.out.println(Thread.currentThread().getName()+ " completableFuture2");
            //模拟异常
            int i = 1/0;
            return 1024;
        });
        completableFuture2.whenComplete((t,u)->{
            System.out.println("------t "+t);
            System.out.println("------u "+u); //异常信息
        }).get();

    }
}

```

