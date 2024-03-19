#### 简介
用来创建锁和其他同步类的基本线程阻塞原语。简而言之，调用`LockSupport.park`时，当前线程将会**等待**，**直至获得许可**。当调用`LockSupport.unpark`时，必须**把等待获得许可的线程作为参数进行传递**，好让此线程继续运行。
#### 源码分析
##### 类的属性
`UNSAFE`字段表示`sun.misc.Unsafe`类，查看其源码，点击在这里，一般程序中不允许直接调用，而long型的表示实例对象相应字段在内存中的偏移地址，可以通过该偏移地址获取或者设置该字段的值。
``` java
public class LockSupport {
    // Hotspot implementation via intrinsics API
    private static final sun.misc.Unsafe UNSAFE;
    // 表示内存偏移地址
    private static final long parkBlockerOffset;
    // 表示内存偏移地址
    private static final long SEED;
    // 表示内存偏移地址
    private static final long PROBE;
    // 表示内存偏移地址
    private static final long SECONDARY;
    
    static {
        try {
            // 获取Unsafe实例
            UNSAFE = sun.misc.Unsafe.getUnsafe();
            // 线程类类型
            Class<?> tk = Thread.class;
            // 获取Thread的parkBlocker字段的内存偏移地址
            parkBlockerOffset = UNSAFE.objectFieldOffset
                (tk.getDeclaredField("parkBlocker"));
            // 获取Thread的threadLocalRandomSeed字段的内存偏移地址
            SEED = UNSAFE.objectFieldOffset
                (tk.getDeclaredField("threadLocalRandomSeed"));
            // 获取Thread的threadLocalRandomProbe字段的内存偏移地址
            PROBE = UNSAFE.objectFieldOffset
                (tk.getDeclaredField("threadLocalRandomProbe"));
            // 获取Thread的threadLocalRandomSecondarySeed字段的内存偏移地址
            SECONDARY = UNSAFE.objectFieldOffset
                (tk.getDeclaredField("threadLocalRandomSecondarySeed"));
        } catch (Exception ex) { throw new Error(ex); }
    }
}
```
##### 类的构造函数
LockSupport只有一个私有构造函数，无法被实例化
``` java
// 私有构造函数，无法被实例化
private LockSupport() {}
```
##### 核心函数分析
在分析LockSupport函数之前，先引入`sun.misc.Unsafe`类中的park和unpark函数，因为LockSupport的核心函数都是**基于Unsafe类中定义的park和unpark函数**，下面给出两个函数的定义:
``` java
public native void park(boolean isAbsolute, long time);
public native void unpark(Thread thread);
```
说明: 对两个函数的说明如下:
- **park函数，阻塞线程**，并且该线程在**下列情况发生之前**都会被**阻塞**: 
	1. 调用unpark函数，释放该线程的许可。
	2. 该线程被中断。
	3. 设置的时间到了。并且，当time为绝对时间时，isAbsolute为true，否则，isAbsolute为false。当time为0时，表示无限等待，直到unpark发生。
- **unpark函数，释放线程的许可**，即激活调用park后阻塞的线程。这个函数不是安全的，调用这个函数时要确保线程依旧存活。
