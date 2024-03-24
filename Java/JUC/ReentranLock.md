#### 源码分析
##### 类的继承关系
ReentrantLock**实现了**Lock接口，Lock接口中定义了lock与unlock相关操作，并且还存在newCondition方法，表示生成一个条件。
``` java
public class ReentrantLock implements Lock, java.io.Serializable
```
##### 类的内部类
ReentrantLock总共有**三个内部类**，并且三个内部类是紧密相关的，下面先看三个类的关系。
![](../../img/Pasted%20image%2020240324185558.png)
**说明**：`ReentrantLock`类内部总共存在`Sync`、`NonfairSync`、`FairSync`三个类，`NonfairSync`与`FairSync`类继承自`Sync`类，`Sync`类继承自`AbstractQueuedSynchronicer`抽象类。

`Sync`类：
源码如下：
``` java
abstract static class Sync extends AbstractQueuedSynchronizer {
    // 序列号
    private static final long serialVersionUID = -5179523762034025860L;
    
    // 获取锁
    abstract void lock();
    
    // 非公平方式获取
    final boolean nonfairTryAcquire(int acquires) {
        // 当前线程
        final Thread current = Thread.currentThread();
        // 获取状态
        int c = getState();
        if (c == 0) { // 表示没有线程正在竞争该锁
            if (compareAndSetState(0, acquires)) { // 比较并设置状态成功，状态0表示锁没有被占用
                // 设置当前线程独占
                setExclusiveOwnerThread(current); 
                return true; // 成功
            }
        }
        else if (current == getExclusiveOwnerThread()) { // 当前线程拥有该锁
            int nextc = c + acquires; // 增加重入次数
            if (nextc < 0) // overflow
                throw new Error("Maximum lock count exceeded");
            // 设置状态
            setState(nextc); 
            // 成功
            return true; 
        }
        // 失败
        return false;
    }
    
    // 试图在共享模式下获取对象状态，此方法应该查询是否允许它在共享模式下获取对象状态，如果允许，则获取它
    protected final boolean tryRelease(int releases) {
        int c = getState() - releases;
        if (Thread.currentThread() != getExclusiveOwnerThread()) // 当前线程不为独占线程
            throw new IllegalMonitorStateException(); // 抛出异常
        // 释放标识
        boolean free = false; 
        if (c == 0) {
            free = true;
            // 已经释放，清空独占
            setExclusiveOwnerThread(null); 
        }
        // 设置标识
        setState(c); 
        return free; 
    }
    
    // 判断资源是否被当前线程占有
    protected final boolean isHeldExclusively() {
        // While we must in general read state before owner,
        // we don't need to do so to check if current thread is owner
        return getExclusiveOwnerThread() == Thread.currentThread();
    }

    // 新生一个条件
    final ConditionObject newCondition() {
        return new ConditionObject();
    }

    // Methods relayed from outer class
    // 返回资源的占用线程
    final Thread getOwner() {        
        return getState() == 0 ? null : getExclusiveOwnerThread();
    }
    // 返回状态
    final int getHoldCount() {            
        return isHeldExclusively() ? getState() : 0;
    }

    // 资源是否被占用
    final boolean isLocked() {        
        return getState() != 0;
    }

    /**
        * Reconstitutes the instance from a stream (that is, deserializes it).
        */
    // 自定义反序列化逻辑
    private void readObject(java.io.ObjectInputStream s)
        throws java.io.IOException, ClassNotFoundException {
        s.defaultReadObject();
        setState(0); // reset to unlocked state
    }
}　　
```
`NonfairSync`类：
NonfairSync类**继承**了Sync类，表示采用**非公平策略**获取锁，其实现了Sync类中抽象的lock方法，源码如下：
``` java
// 非公平锁
static final class NonfairSync extends Sync {
    // 版本号
    private static final long serialVersionUID = 7316153563782823691L;

    // 获得锁
    final void lock() {
        if (compareAndSetState(0, 1)) // 比较并设置状态成功，状态0表示锁没有被占用
            // 把当前线程设置独占了锁
            setExclusiveOwnerThread(Thread.currentThread());
        else // 锁已经被占用，或者set失败
            // 以独占模式获取对象，忽略中断
            acquire(1); 
    }

    protected final boolean tryAcquire(int acquires) {
        return nonfairTryAcquire(acquires);
    }
}
```
**说明**：从`lock`方法的源码可知，每一次都尝试获取锁，而并**不会按照公平等待的原则**进行等待，让等待时间最久的线程获得锁。

`FairSync`类
FairSync类也**继承了Sync类**，表示**采用公平策略**获取锁，其实现了Sync类中的抽象lock方法，源码如下：
``` java
// 公平锁
static final class FairSync extends Sync {
    // 版本序列化
    private static final long serialVersionUID = -3000897897090466540L;

    final void lock() {
        // 以独占模式获取对象，忽略中断
        acquire(1);
    }

    /**
        * Fair version of tryAcquire.  Don't grant access unless
        * recursive call or no waiters or is first.
        */
    // 尝试公平获取锁
    protected final boolean tryAcquire(int acquires) {
        // 获取当前线程
        final Thread current = Thread.currentThread();
        // 获取状态
        int c = getState();
        if (c == 0) { // 状态为0
            if (!hasQueuedPredecessors() &&
                compareAndSetState(0, acquires)) { // 不存在已经等待更久的线程并且比较并且设置状态成功
                // 设置当前线程独占
                setExclusiveOwnerThread(current);
                return true;
            }
        }
        else if (current == getExclusiveOwnerThread()) { // 状态不为0，即资源已经被线程占据
            // 下一个状态
            int nextc = c + acquires;
            if (nextc < 0) // 超过了int的表示范围
                throw new Error("Maximum lock count exceeded");
            // 设置状态
            setState(nextc);
            return true;
        }
        return false;
    }
}
```
**说明**：跟踪lock方法的源码可知，资源空闲时，总会先判断sync队列（同步队列）是否有等待时间更长的线程，如果存在，则将线程加入到等待队列的尾部，实现了公平获取原则。
##### 类的属性
ReentrantLock类的sync非常重要，对ReentrantLock类的操作大部分都直接转化为对Sync和AbstractQueuedSynchronizer类的操作。
``` java
public class ReentrantLock implements Lock, java.io.Serializable {
    // 序列号
    private static final long serialVersionUID = 7373984872572414699L;    
    // 同步队列
    private final Sync sync;
}
```
##### 类的构造函数
- ReentrantLock()型构造函数
**默认**采用的是**非公平策略**获取锁
``` java
public ReentrantLock() {
    // 默认非公平策略
    sync = new NonfairSync();
}
```
- ReentrantLock(boolean)型构造函数
可以传递参数确定采用公平策略或是非公平策略，参数为true表示公平策略，否则，采用非公平策略
``` java
public ReentrantLock(boolean fair) {
    sync = fair ? new FairSync() : new NonfairSync();
}
```

