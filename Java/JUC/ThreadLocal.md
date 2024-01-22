#### 简介
ThreadLocal是一个将在多线程中为每一个线程创建单独变量副本的类；当使用ThreadLocal来维护变量时，ThreadLocal会为每个线程创建单独的变量副本（**TLS: thread local storage 线程局部存储**），避免因多线程操作共享变量而导致的数据不一致的情况。
#### 理解
提到ThreadLocal的时候，可以想到数据库连接管理，这里以数据访问为例子来帮助理解ThreadLocal
下述代码是数据库管理类在单线程使用下的情况;
``` java
class ConnectionManager { 
	private static Connection connect = null;
	public static Connection openConnection() { 
		if (connect == null) { 
			connect = DriverManager.getConnection(); 
		} 
		return connect; 
	} 
	public static void closeConnection() { 
		if (connect != null) connect.close(); 
	} 
}

```
上述代码在单线程的情况下是安全的，但是如果是多线程的话就会有问题
上述代码中的connect变量由于是静态的，所以只会实例化一次（[3.类的初始化](../JVM/类加载机制.md#3.类的初始化)），是一个共享资源，所有线程都会访问这个connect变量。那么当多线程环境下，会导致线程安全问题。
**可能产生的线程安全问题**：
- `openConnection`方法在检查`connect`是否为`null`时，没有进行任何同步。这意味着如果两个线程几乎同时使用调用`openConnection`，且`connect`为`null`，它们都可能尝试创建连接。这可能导致两个线程分别创建不同的连接实例，但由于`connect`变量是静态的，只能实例化一次，最终只有一个实例会被赋值给`connect`，另一个实例可能永远不会被关闭，导致资源泄漏。
- `openConnection`和`closeConnection`方法中对`connect`的检查和赋值操作不是原子的，也不是同步。在多线程环境中，这可能会导致不一致的状态和意外的行为。例如：当一个线程正在使用`connect`变量操作数据库的时候，另一个线程调用`closeConnection`关闭连接。
为了解决上述的线程安全问题，第一个反应就是**考虑互斥同步**，将这段代码的两个方法进行同步处理，并且在调用`connect`的地方需要进行同步处理，比如用`Synchronized`或者`ReentrantLock`互斥锁
**再抛出一个问题**：
- 这里到底需不需要将`connect`变量进行共享
实际上是不需要的，如果每个线程中都有一个`connect`变量，各线程之间对`connect`变量的访问实际上是没有依赖关系的，即一个线程不需要关心其他线程是否对这个`connect`进行了修改的。
修改后的代码如下：
``` java
class ConnectionManager { 
	private Connection connect = null; 
	public Connection openConnection() { 
		if (connect == null) { 
			connect = DriverManager.getConnection(); 
		} 
		return connect; 
	} 
	public void closeConnection() { 
		if (connect != null) connect.close(); 
	} 
} 
class Dao { 
	public void insert() { 
		ConnectionManager connectionManager = new ConnectionManager(); 
		Connection connection = connectionManager.openConnection(); 
		// 使用connection进行操作 connectionManager.closeConnection(); 
	} 
}

```
在`insert`方法中，每次调用`insert`时都会创建一个新的`ConnectionManager`实例和一个新的数据库连接。由于这个连接时局部于`insert`方法的。因此每个线程都会有自己的`ConnectManager`实例和数据库连接。
**这就意味着**：
- **资源隔离**：每个线程使用自己的`ConnectManager`实例，因此它们各自维护自己的数据库连接。避免了不同线程之间共享同一个连接实例，从而消除了线程安全问题。
因此，在这种情况下，每个线程都在操作自己的资源（数据库连接），并且这些资源在方法的作用域内被创建和销毁，所以不存在线程安全问题。

但是，很显然，这种每次请求都创建新连接的方法是很低效的，此时就需要`ThreadLocal`的特性**TLS: thread local storage 线程局部存储**。
ThreadLocal在每个线程中对该变量会创建一个副本，即每个线程内部都会有一个该变量，且在线程内部任何地方都可以使用，线程之间不会影响，这样一来就不存在线程安全问题，也不会验证影响效率和资源。
改进后的代码如下：
``` java
import java.sql.Connection;  
import java.sql.DriverManager;  
import java.sql.SQLException;  
  
public class ConnectionManager {  
  
    private static final ThreadLocal<Connection> dbConnectionLocal = new ThreadLocal<Connection>() {  
        @Override  
        protected Connection initialValue() {  
            try {  
                return DriverManager.getConnection("", "", "");  
            } catch (SQLException e) {  
                e.printStackTrace();  
            }  
            return null;  
        }  
    };  
  
    public Connection getConnection() {  
        return dbConnectionLocal.get();  
    }  
}
```
如果我们希望通过某个类想状态（例如用户ID、事务ID）与线程关联起来，那么通常在这个类中定义`private static`类型的`ThreadLocal`实例。
**注意**：虽然ThreadLocal解决了上述问题，但是每个线程都创建副本，这需要考虑对资源的消耗。