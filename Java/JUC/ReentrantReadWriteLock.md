`ReentrantReadWriteLock`底层是基于`ReentrantLock`和`AbstractQueuedSynchronizer`来实现的，所以，`ReentrantReadWriteLock`的数据结构也依托于AQS的数据结构。
#### 源码分析
##### 继承关系
``` java
public class ReentrantReadWriteLock implements ReadWriteLock, java.io.Serializable {}
```
**说明**：可以看到，`ReentrantReadWriteLock`**实现**了`ReadWriteLock`接口，`ReadWriteLock`接口定义了**获取读锁和写锁的规范**，具体需要实现类去实现；同时其还实现了`Serializable`接口，表示可以进行序列化，在源代码中可以看到`ReentrantReadWriteLock`实现了自己的序列化逻辑。
##### 类的内部类
ReentrantReadWriteLock有五个内部类，五个内部类之间相互关联
![](../../img/Pasted%20image%2020240324201613.png)
**说明**：`Sync`**继承自**`AQS`、`NonfairSync`**继承自**`Sync`，`FairSync`**继承自**`Sync`；`ReadLock`**实现**了`Lock`接口、`WriteLock`**实现**了`Lock`接口
##### 内部类-Sync类
- 类的继承关系
``` java
abstract static class Sync extends AbstractQueuedSynchronizer {}
```
**说明**：`Sync`抽象类**继承自**`AQS`抽象类，`Sync`类**提供**了对`ReentrantReadWriteLock`的支持。
- 类的内部类
`Sync`类内部**存在两个内部类**，分别为`HoldCounter`和`ThreadLocalHoldCounter`，其中`HoldCounter`主要与读锁配套使用，其中，`HoldCounter`源码如下。
``` java
// 计数器
static final class HoldCounter {
    // 计数
    int count = 0;
    // Use id, not reference, to avoid garbage retention
    // 获取当前线程的TID属性的值
    final long tid = getThreadId(Thread.currentThread());
}
```
**说明**：`HoldCounter`主要有**两个属性**，`count`和`tid`，其中`count`表示**某个读线程重入的次数**，`tid`表示**该线程的tid字段的值**，该字段可以**用来唯一标识一个线程**。
`ThreadLocalHoldCounter`的源码如下：
``` java
// 本地线程计数器
static final class ThreadLocalHoldCounter
    extends ThreadLocal<HoldCounter> {
    // 重写初始化方法，在没有进行set的情况下，获取的都是该HoldCounter值
    public HoldCounter initialValue() {
        return new HoldCounter();
    }
}
```
**说明**：`ThreadLocalHoldCounter`**重写**了`ThreadLocal`的`initialValue`方法，`ThreadLocal`类**可以将线程与对象相关联**。在没有进行`set`的情况下，`get`到的均是`initialValue`方法里面生成的那个`HolderCounter`对象。

- 类的属性
``` java
abstract static class Sync extends AbstractQueuedSynchronizer {
    // 版本序列号
    private static final long serialVersionUID = 6317671515068378041L;        
    // 高16位为读锁，低16位为写锁
    static final int SHARED_SHIFT   = 16;
    // 读锁单位
    static final int SHARED_UNIT    = (1 << SHARED_SHIFT);
    // 读锁最大数量
    static final int MAX_COUNT      = (1 << SHARED_SHIFT) - 1;
    // 写锁最大数量
    static final int EXCLUSIVE_MASK = (1 << SHARED_SHIFT) - 1;
    // 本地线程计数器
    private transient ThreadLocalHoldCounter readHolds;
    // 缓存的计数器
    private transient HoldCounter cachedHoldCounter;
    // 第一个读线程
    private transient Thread firstReader = null;
    // 第一个读线程的计数
    private transient int firstReaderHoldCount;
}
```
**说明**：该属性中包括了**读锁**、**写锁线程的最大量**。**本地线程计数器**等
- 类的构造函数
``` java
// 构造函数
Sync() {
    // 本地线程计数器
    readHolds = new ThreadLocalHoldCounter();
    // 设置AQS的状态
    setState(getState()); // ensures visibility of readHolds
}
```
**说明**：在Sync的构造函数中设置了**本地线程计数器**和AQS的状态state
##### 内部类 - Sync核心函数分析
对ReentrantReadWriteLock对象的操作绝大多数都转发至Sync对象进行处理，下面对Sync类中的重点函数进行分析
- `sharedCount`函数
表示占有读锁的线程数量
``` java
static int sharedCount(int c)    { return c >>> SHARED_SHIFT; }
```
**说明**：直接将state**右移16位**，就可以得到读锁的线程数量，应为state的**高16位表示读锁**，对应的**低16位表示写锁数量**。
- `exclusiveCount`函数
表示占有写锁的线程数量
``` java
static int exclusiveCount(int c) { return c & EXCLUSIVE_MASK; }
```
**说明**：直接将状态state和(2^16 - 1)做与运算，这样就可以得到高16位都是0，低16位则是写锁的数量。写锁数量由state的低十六位表示。
- `tryRelease`函数
``` java
/*
* Note that tryRelease and tryAcquire can be called by
* Conditions. So it is possible that their arguments contain
* both read and write holds that are all released during a
* condition wait and re-established in tryAcquire.
*/

protected final boolean tryRelease(int releases) {
    // 判断是否伪独占线程
    if (!isHeldExclusively())
        throw new IllegalMonitorStateException();
    // 计算释放资源后的写锁的数量
    int nextc = getState() - releases;
    boolean free = exclusiveCount(nextc) == 0; // 是否释放成功
    if (free)
        setExclusiveOwnerThread(null); // 设置独占线程为空
    setState(nextc); // 设置状态
    return free;
}
```
![](../../img/Pasted%20image%2020240324204500.png)
- `tryAcquire`函数
``` java
protected final boolean tryAcquire(int acquires) {
    /*
        * Walkthrough:
        * 1. If read count nonzero or write count nonzero
        *    and owner is a different thread, fail.
        * 2. If count would saturate, fail. (This can only
        *    happen if count is already nonzero.)
        * 3. Otherwise, this thread is eligible for lock if
        *    it is either a reentrant acquire or
        *    queue policy allows it. If so, update state
        *    and set owner.
        */
    // 获取当前线程
    Thread current = Thread.currentThread();
    // 获取状态
    int c = getState();
    // 写线程数量
    int w = exclusiveCount(c);
    if (c != 0) { // 状态不为0
        // (Note: if c != 0 and w == 0 then shared count != 0)
        if (w == 0 || current != getExclusiveOwnerThread()) // 写线程数量为0或者当前线程没有占有独占资源
            return false;
        if (w + exclusiveCount(acquires) > MAX_COUNT) // 判断是否超过最高写线程数量
            throw new Error("Maximum lock count exceeded");
        // Reentrant acquire
        // 设置AQS状态
        setState(c + acquires);
        return true;
    }
    if (writerShouldBlock() ||
        !compareAndSetState(c, c + acquires)) // 写线程是否应该被阻塞
        return false;
    // 设置独占线程
    setExclusiveOwnerThread(current);
    return true;
}
```
![](../../img/Pasted%20image%2020240324204533.png)
- `tryReleaseShared`函数 **读锁线程释放锁**
``` java
protected final boolean tryReleaseShared(int unused) {
    // 获取当前线程
    Thread current = Thread.currentThread();
    if (firstReader == current) { // 当前线程为第一个读线程
        // assert firstReaderHoldCount > 0;
        if (firstReaderHoldCount == 1) // 读线程占用的资源数为1
            firstReader = null;
        else // 减少占用的资源
            firstReaderHoldCount--;
    } else { // 当前线程不为第一个读线程
        // 获取缓存的计数器
        HoldCounter rh = cachedHoldCounter;
        if (rh == null || rh.tid != getThreadId(current)) // 计数器为空或者计数器的tid不为当前正在运行的线程的tid
            // 获取当前线程对应的计数器
            rh = readHolds.get();
        // 获取计数
        int count = rh.count;
        if (count <= 1) { // 计数小于等于1
            // 移除
            readHolds.remove();
            if (count <= 0) // 计数小于等于0，抛出异常
                throw unmatchedUnlockException();
        }
        // 减少计数
        --rh.count;
    }
    for (;;) { // 无限循环
        // 获取状态
        int c = getState();
        // 获取状态
        int nextc = c - SHARED_UNIT;
        if (compareAndSetState(c, nextc)) // 比较并进行设置
            // Releasing the read lock has no effect on readers,
            // but it may allow waiting writers to proceed if
            // both read and write locks are now free.
            return nextc == 0;
    }
}
```
![](../../img/Pasted%20image%2020240324204956.png)
- `tryAcquireShared`函数 **读锁线程获取锁**
``` java
private IllegalMonitorStateException unmatchedUnlockException() {
    return new IllegalMonitorStateException(
        "attempt to unlock read lock, not locked by current thread");
}

// 共享模式下获取资源
protected final int tryAcquireShared(int unused) {
    /*
        * Walkthrough:
        * 1. If write lock held by another thread, fail.
        * 2. Otherwise, this thread is eligible for
        *    lock wrt state, so ask if it should block
        *    because of queue policy. If not, try
        *    to grant by CASing state and updating count.
        *    Note that step does not check for reentrant
        *    acquires, which is postponed to full version
        *    to avoid having to check hold count in
        *    the more typical non-reentrant case.
        * 3. If step 2 fails either because thread
        *    apparently not eligible or CAS fails or count
        *    saturated, chain to version with full retry loop.
        */
    // 获取当前线程
    Thread current = Thread.currentThread();
    // 获取状态
    int c = getState();
    if (exclusiveCount(c) != 0 &&
        getExclusiveOwnerThread() != current) // 写线程数不为0并且占有资源的不是当前线程
        return -1;
    // 读锁数量
    int r = sharedCount(c);
    if (!readerShouldBlock() &&
        r < MAX_COUNT &&
        compareAndSetState(c, c + SHARED_UNIT)) { // 读线程是否应该被阻塞、并且小于最大值、并且比较设置成功
        if (r == 0) { // 读锁数量为0
            // 设置第一个读线程
            firstReader = current;
            // 读线程占用的资源数为1
            firstReaderHoldCount = 1;
        } else if (firstReader == current) { // 当前线程为第一个读线程
            // 占用资源数加1
            firstReaderHoldCount++;
        } else { // 读锁数量不为0并且不为当前线程
            // 获取计数器
            HoldCounter rh = cachedHoldCounter;
            if (rh == null || rh.tid != getThreadId(current)) // 计数器为空或者计数器的tid不为当前正在运行的线程的tid
                // 获取当前线程对应的计数器
                cachedHoldCounter = rh = readHolds.get();
            else if (rh.count == 0) // 计数为0
                // 设置
                readHolds.set(rh);
            rh.count++;
        }
        return 1;
    }
    return fullTryAcquireShared(current);
}
```
![](../../img/Pasted%20image%2020240324210125.png)
##### 类的属性
``` java
public class ReentrantReadWriteLock
        implements ReadWriteLock, java.io.Serializable {
    // 版本序列号    
    private static final long serialVersionUID = -6992448646407690164L;    
    // 读锁
    private final ReentrantReadWriteLock.ReadLock readerLock;
    // 写锁
    private final ReentrantReadWriteLock.WriteLock writerLock;
    // 同步队列
    final Sync sync;
    
    private static final sun.misc.Unsafe UNSAFE;
    // 线程ID的偏移地址
    private static final long TID_OFFSET;
    static {
        try {
            UNSAFE = sun.misc.Unsafe.getUnsafe();
            Class<?> tk = Thread.class;
            // 获取线程的tid字段的内存地址
            TID_OFFSET = UNSAFE.objectFieldOffset
                (tk.getDeclaredField("tid"));
        } catch (Exception e) {
            throw new Error(e);
        }
    }
}
```
**说明**：可以看到`ReentrantReadWriteLock`属性包括了一个`ReentrantReadWriteLock.ReadLock`对象，表示**读锁**；一个`ReentrantReadWriteLock.WriteLock`对象，表示**写锁**；一个`Sync`对象，表示**同步队列**。
##### 类的构造函数
-  `ReentrantReadWriteLock()`型构造函数
``` java
public ReentrantReadWriteLock() {
    this(false);
}
```
**说明**：此构造函数会调用**另外一个有参构造函数**
-  `ReentrantReadWriteLock(boolean)`型构造函数
``` java
public ReentrantReadWriteLock(boolean fair) {
    // 公平策略或者是非公平策略
    sync = fair ? new FairSync() : new NonfairSync();
    // 读锁
    readerLock = new ReadLock(this);
    // 写锁
    writerLock = new WriteLock(this);
}
```
**说明**：可以指定设置公平策略或者非公平策略，并且该构造函数中生成了读锁和写锁两个对象
