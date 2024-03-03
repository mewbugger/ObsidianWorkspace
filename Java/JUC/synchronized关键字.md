#### 基本概念
`synchronized`关键字**主要解决**的是多个线程之间访问资源的同步性，可以保证被它修饰的方法或者代码块在任意时刻只能有一个线程执行。
#### 使用
应用`sychronized`关键字时需要把握如下注意点：
- 一把锁只能同时被一个线程获取，没有获得锁的线程只能等待
- 每个实例都对应自己的一把锁(`this`)，不同实例之间互不影响；**例外**：锁对象是`*.class`以及`synchronized`修饰的是`static`方法的时候，所有对象公用同一把锁。（**原因**：所有的实力都共享同一个类对象（`*.class`），而static修饰的方法是属于类对象的（static关键字修饰的只会实例化一次，只属于类。（[3.类的初始化](../JVM/类加载机制.md#3.类的初始化)）），所以这个例外里提到的是所有对象公用同一把锁）
- `synchronized`修饰的方法，无论方法正常执行完毕还是抛出异常，都会释放锁。
##### 对象锁
包括方法锁（默认锁对象为`this`，当前实例对象）和同步代码块锁（自己指定锁对象）
**代码块形式**：手动指定锁定对象，也可以是`this`，也可以是自定义锁
示例如下：
``` java
public class SynchronizedObjectLock implements Runnable {  
	
    static SynchronizedObjectLock instance = new SynchronizedObjectLock();  
  
    @Override  
    public void run() {  
        // 同步代码块形式——锁为this,两个线程使用的锁是一样的,线程1必须要等到线程0释放了该锁后，才能执行  
        synchronized (this) {  
            System.out.println("我是线程" + Thread.currentThread().getName());  
            try {  
                Thread.sleep(3000);  
            } catch (InterruptedException e) {  
                e.printStackTrace();  
            }  
            System.out.println(Thread.currentThread().getName() + "结束");  
        }  
    }  
  
    public static void main(String[] args) {  
        Thread t1 = new Thread(instance);  
        Thread t2 = new Thread(instance);  
        t1.start();  
        t2.start();  
    }  
}

/*
	static修饰的变量，只实例化一次，只属于类，所以是共享资源
	输出内容：
		我是线程Thread-0
		Thread-0结束
		我是线程Thread-1
		Thread-1结束
*/
```
**方法锁**形式：`synchronized`修饰普通方法，锁对象默认为`this`
``` java
public class SynchronizedObjectLock implements Runnable {  
    static SynchronizedObjectLock instance = new SynchronizedObjectLock();  
  
    @Override  
    public void run() {  
        method();  
    }  
  
    public synchronized void method() {  
        System.out.println("我是线程" + Thread.currentThread().getName());  
        try {  
            Thread.sleep(3000);  
        } catch (InterruptedException e) {  
            e.printStackTrace();  
        }  
        System.out.println(Thread.currentThread().getName() + "结束");  
    }  
  
    public static void main(String[] args) {  
        Thread t1 = new Thread(instance);  
        Thread t2 = new Thread(instance);  
        t1.start();  
        t2.start();  
    }  
}

/*
	synchronized修饰普通方法的时候默认的锁对象是this，而t1线程和t2线程的this都是instance，被static修饰，两个线程的锁对象是同一个实例
	输出内容：
		我是线程Thread-0  
		Thread-0结束  
		我是线程Thread-1  
		Thread-1结束
	
*/
```
##### 类锁
指`synchronized`修饰静态的方法或指定锁对象为`Class`对象
**`synchronized`修饰静态方法**：
示例1如下：
``` java
public class SynchronizedObjectLock implements Runnable {  
    static SynchronizedObjectLock instance1 = new SynchronizedObjectLock();  
    static SynchronizedObjectLock instance2 = new SynchronizedObjectLock();  
  
    @Override  
    public void run() {  
        method();  
    }  
  
    // synchronized用在普通方法上，默认的锁就是this，当前实例  
    public synchronized void method() {  
        System.out.println("我是线程" + Thread.currentThread().getName());  
        try {  
            Thread.sleep(3000);  
        } catch (InterruptedException e) {  
            e.printStackTrace();  
        }  
        System.out.println(Thread.currentThread().getName() + "结束");  
    }  
  
    public static void main(String[] args) {  
        // t1和t2对应的this是两个不同的实例，所以代码不会串行  
        Thread t1 = new Thread(instance1);  
        Thread t2 = new Thread(instance2);  
        t1.start();  
        t2.start();  
    }  
}
/*
	synchronized修饰普通方法，锁对象默认是this，但是t1线程和t2线程的锁对象不是同一实例，所以不会串行
	输出内容：
		我是线程Thread-0  
		我是线程Thread-1  
		Thread-1结束  
		Thread-0结束
*/
```
示例2如下：
``` java
public class SynchronizedObjectLock implements Runnable {  
    static SynchronizedObjectLock instance1 = new SynchronizedObjectLock();  
    static SynchronizedObjectLock instance2 = new SynchronizedObjectLock();  
  
    @Override  
    public void run() {  
        method();  
    }  
  
    // synchronized用在静态方法上，默认的锁就是当前所在的Class类，所以无论是哪个线程访问它，需要的锁都只有一把  
    public static synchronized void method() {  
        System.out.println("我是线程" + Thread.currentThread().getName());  
        try {  
            Thread.sleep(3000);  
        } catch (InterruptedException e) {  
            e.printStackTrace();  
        }  
        System.out.println(Thread.currentThread().getName() + "结束");  
    }  
  
    public static void main(String[] args) {  
        Thread t1 = new Thread(instance1);  
        Thread t2 = new Thread(instance2);  
        t1.start();  
        t2.start();  
    }  
}
/*
	synchronized修饰静态方法，静态方法属于类对象，而类对象是所有实例共享的，所以串行执行
	输出内容：
		我是线程Thread-0  
        Thread-0结束  
        我是线程Thread-1  
        Thread-1结束
*/
```
**`synchronized`指定锁对象为`Class`对象：**
示例如下：
``` java
public class SynchronizedObjectLock implements Runnable {  
    static SynchronizedObjectLock instance1 = new SynchronizedObjectLock();  
    static SynchronizedObjectLock instance2 = new SynchronizedObjectLock();  
  
    @Override  
    public void run() {  
        // 所有线程需要的锁都是同一把  
        synchronized(SynchronizedObjectLock.class){  
            System.out.println("我是线程" + Thread.currentThread().getName());  
            try {  
                Thread.sleep(3000);  
            } catch (InterruptedException e) {  
                e.printStackTrace();  
            }  
            System.out.println(Thread.currentThread().getName() + "结束");  
        }  
    }  
  
    public static void main(String[] args) {  
        Thread t1 = new Thread(instance1);  
        Thread t2 = new Thread(instance2);  
        t1.start();  
        t2.start();  
    }  
}
/*
	Class对象是被所有实例共享的，所以synchronized的锁对象是Class对象的时候，可以串行执行
	输出内容：
		我是线程Thread-0  
        Thread-0结束  
        我是线程Thread-1  
        Thread-1结束
*/
```
#### Synchronized原理分析
#### 加锁和释放锁的原理
![](../../img/Pasted%20image%2020240303225623.png)
关注红色方框里的`monitorenter`和`monitorexit`。
`Monitorenter`和`Monitorexit`指令，会让对象在执行，使其锁计数器加1或减1。每个对象在同一时间只能被一个线程获得，一个对象在尝试获得与这个对象相关联的Monitor锁的所有权的时候，`monitorenter`指令会发生如下3种情况之一：
- monitor计数器为0，锁还未被获得，这个线程就会立刻获得然后把锁计数器加1，一旦加1，别的线程就需要等待才能获得锁。
- 如果这个monitor已经拿到了这个锁的所有权，又重入了这把锁，锁计数器会累加，随着重入的次数不断累加。
- 这把锁已经被别的线程获取，等待锁释放。
`monitorexit指令`：释放对于monitor的所有权，释放过程很简单，就是讲monitor的计数器减1，如果减完以后，计数器不是0，则代表刚才是重入进来的，当前线程还继续持有这把锁的所有权，如果计数器变成0，则代表当前线程不再拥有该monitor的所有权，即释放锁。
![](../../img/Pasted%20image%2020240303230311.png)
**对象监视器，同步队列以及执行线程状态之间的关系**
##### 可重入原理：加锁次数计数器
[可重入锁 VS 非可重入锁](锁.md#可重入锁%20VS%20非可重入锁)
代码示例如下：
``` java
public class SynchronizedDemo {
    public static void main(String[] args) {
        SynchronizedDemo demo =  new SynchronizedDemo();
        demo.method1();
    }
    private synchronized void method1() {
        System.out.println(Thread.currentThread().getId() + ": method1()");
        method2();
    }
    private synchronized void method2() {
        System.out.println(Thread.currentThread().getId()+ ": method2()");
        method3();
    }
    private synchronized void method3() {
        System.out.println(Thread.currentThread().getId()+ ": method3()");
    }
}
```
结合前文中加锁和释放锁的原理，不难理解：

- 执行monitorenter获取锁
    - （monitor计数器=0，可获取锁）
    - 执行method1()方法，monitor计数器+1 -> 1 （获取到锁）
    - 执行method2()方法，monitor计数器+1 -> 2
    - 执行method3()方法，monitor计数器+1 -> 3
- 执行monitorexit命令
    - method3()方法执行完，monitor计数器-1 -> 2
    - method2()方法执行完，monitor计数器-1 -> 1
    - method2()方法执行完，monitor计数器-1 -> 0 （释放了锁）
    - （monitor计数器=0，锁被释放了）
##### 保证可见性的原理：内存模型和happens-before规则
![](../../img/Pasted%20image%2020240303232610.png)
1. “x=42” Happens-Before 写变量 “v=true” ，这是规则 1 的内容；
2. 写变量“v=true” Happens-Before 读变量 “v=true”，这是规则 3 的内容 。
3. 再根据这个传递性规则，我们得到结果：“x=42” Happens-Before 读变量“v=true”
如果线程 B 读到了“v=true”，那么线程 A 设置的“x=42”对线程 B 是可见的。也就是说，线程 B 能看到 “x == 42”
在 Java 语言里面，Happens-Before 的语义本质上是一种可见性，A Happens-Before B 意味着 A 事件对 B 事件来说是可见的，无论 A 事件和 B 事件是否发生在同一个线程里。例如 A 事件发生在线程 1 上，B 事件发生在线程 2 上，Happens-Before 规则保证线程 2 上也能看到 A 事件的发生。