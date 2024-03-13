#### 简介
AQS（`AbstractQueuedSynchronizer`）是一个用来构建锁和同步器的框架，使用AQS能简单且高效地构造出应用广泛的大量的同步器，比如`ReentrantLock`，`Semaphore`，其他的诸如`ReentrantReadWriteLock`，`SynchronousQueue`，`FutureTask`等都是基于AQS的。也可以用AQS非常轻松容易地构造出符合我们自己需求的同步器。
##### 核心思想
如果被请求的共享资源空闲，则将当前请求资源的线程设置为有效的工作线程，并且将共享资源设置为锁定状态。如果被请求的共享资源**被占用**，那么就需要一套**线程阻塞等待以及被唤醒时锁分配的机制**，这个机制AQS是用**CLH队列锁**实现的，即将暂时获取不到锁的线程加入到队列中。
``` java
/*
	
*/
```