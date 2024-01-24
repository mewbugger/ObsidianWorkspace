#### 作用
##### 防重排序
在并发环境下的单利实现，通常可以采用**双重检查加锁（DCL）**的方式来实现，示例如下：
``` java
public class Singleton {  
    public static volatile Singleton singleton;  
    /**  
     * 构造函数私有，禁止外部实例化  
     */  
    private Singleton() {};  
    public static Singleton getInstance() {  
        if (singleton == null) {  
            synchronized (singleton.class) {  
                if (singleton == null) {  
                    singleton = new Singleton();  
                }  
            }  
        }  
        return singleton;  
    }  
}
```

要在变量singleton之间加上volatile关键字的原因：
实例化一个对象的三个步骤：
- 分配内存空间
- 初始化对象
- 将内存空间的地址赋值给对应的引用
由于**操作系统**可以**对指令进行重排序**，所以上面的过程也可能会变成如下过程：
- 分配内存空间
- 将内存空间的地址赋值给对应的引用
- 初始化对象
如果是这个流程，多线程环境下就可能将一个未初始化的对象引用暴露出来，从而导致不可预料的结果。因此为了**防止这个过程的重排序**，需要将变量设置为**volatile**类型的变量。
##### 实现可见性
`volatile`关键字可以实现可见性，不能实现原子性。`synchronized`([synchronized关键字](synchronized关键字.md))关键字可以实现可见性，也可以实现原子性。

可见性问题指一个线程修改了共享变量值，而另一个线程却看不到。引起可见性问题的主要原因是每个线程拥有自己的一个高速缓存区——**线程工作内存**。`volatile`关键字能有效的解决这个问题。
示例如下：
``` java
import java.util.concurrent.TimeUnit;  
  
public class TestVolatile {  
    private static boolean stop = false;  
  
    public static void main(String[] args) {  
        // Thread-A  
        new Thread("Thread A") {  
            @Override  
            public void run() {  
                while (!stop) {  
                }  
                System.out.println(Thread.currentThread() + " stopped");  
            }  
        }.start();  
  
        // Thread-main  
        try {  
            TimeUnit.SECONDS.sleep(1);  
            System.out.println(Thread.currentThread() + " after 1 seconds");  
        } catch (InterruptedException e) {  
            e.printStackTrace();  
        }  
        stop = true;  
    }  
}

/*
	输出内容：
	Thread[main,5,main] after 1 seconds
	.
	.
	.
	ThreadA一直在循环中
*/
```
由于每个线程都有自己的**线程工作内存**，所以在一秒后修改了stop的值为true，但是ThreadA所读取的stop的值来自自己的线程工作内存，仍然为false，所以一直在循环
stop变量签名加上volatile关键字则会真实stop
输出如下：
``` java
Thread[main,5,main] after 1 seconds  
Thread[Thread A,5,main] stopped
```
##### 单次读/写的原子性
`volatile`不能完全保证原子性，只能保证单次的读/写操作具有原子性。从以下两个问题来理解：
###### 问题1：`i++`为什么不能保证原子性
以下代码无法保证原子性;
``` java
public class VolatileTest01 {  
    volatile int i;  
  
    public void addI(){  
        i++;  
    }  
  
    public static void main(String[] args) throws InterruptedException {  
        final  VolatileTest01 test01 = new VolatileTest01();  
        for (int n = 0; n < 1000; n++) {  
            new Thread(new Runnable() {  
                @Override  
                public void run() {  
                    try {  
                        Thread.sleep(10);  
                    } catch (InterruptedException e) {  
                        e.printStackTrace();  
                    }  
                    test01.addI();  
                }  
            }).start();  
        }  
        Thread.sleep(10000);//等待10秒，保证上面程序执行完成  
        System.out.println(test01.i);  
    }  
}

/*
	输出内容：
	989（理论上是1000）
	每台电脑的输出内容可能不一样
*/
```
由此可见`volatile`无法保证原子性。
因为`i++`是一个复合操作，包含三步：
- 读取i的值
- 对i加1
- 将i的值写回内存
`volatile`无法保证这三个操作的原子性。

###### 问题2：共享的long和double变量为什么要用volatile
因为long和double两种数据类型的**操作可分为高32位和低32位**两部分，因此普通的long或double类型读/写可能不是原子的。