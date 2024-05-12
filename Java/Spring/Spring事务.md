#### Spring支持两种方式的事务管理
- **编程式事务**：在代码中硬编码（在分布式系统中推荐使用）：通过`TransactionTemplate`或者`TransactionManager`手动管理事务，事务范围过大会出现事务未提交导致超时，因此事务要比锁的粒度更小。
``` java
@Service
public class MyService {
    @Autowired
    private TransactionTemplate transactionTemplate;
    public void performComplexBusinessLogic() {
        transactionTemplate.execute(new TransactionCallbackWithoutResult() {
            @Override
            protected void doInTransactionWithoutResult(TransactionStatus status) {
                try {
                    // 业务逻辑操作1
                    // 例如：updateOperation1();
                    // 业务逻辑操作2
                    // 例如：updateOperation2();
                    // 此处可以根据需要调用更多的数据库操作
                } catch (Exception e) {
                    status.setRollbackOnly();  // 发生异常，回滚事务
                }
            }
        });
    }
}
```
- **声明式事务**：在XML配置文件中配置或者直接基于注解（单体应用或者简单业务系统推荐使用）：实际是通过AOP实现（基于`@Transactional`的全注释方式使用最多）
	- **优点**：使用了AOP实现，本质就是在目标方法执行前后进行拦截，对代码没有侵入性，方法内只需要写业务逻辑就可以
	- **缺点**：
		- 最小的粒度是作用在方法上的
		- 开发者没有注意到一个方法是被事务嵌套的，可能会在方法中加入一些如**RPC远程调用、消息发送、缓存更新、文件写入等操作。** 会有如下两个问题：
			1. 这些操作自身是无法回滚的，这就会导致数据的不一致。可能RPC调用成功了，但是本地事务回滚了，可是RPC调用无法回滚。
			2. 在事务中有远程调用，就会拉长整个事务。那么就会导致本事务的数据库连接一直被占用，那么如果类似操作过多，就会导致数据库连接池耗尽。
``` java
@Service
public class UserService {
    @Autowired
    private JdbcTemplate jdbcTemplate;
    @Transactional
    public void createUser(String username, String email) {
        jdbcTemplate.update("INSERT INTO users(username, email) VALUES (?, ?)", username, email);
        // 假设这里有一个可能引发异常的操作，以演示事务回滚
        if (email.contains("@example.com")) {
            throw new RuntimeException("Cannot use example.com emails");
        }
    }
}
```

#### Spring事务中的事务传播行为
**事务传播行为是为了解决业务层方法之间互相调用的事务问题。**
当事务方法被另一个事务方法调用时，必须指定事务应该如何传播。例如：事务可能继续在现有事务中运行，也可能开启一个新事物，并在自己的事务中运行。
**正确**的事务传播行为可能的值如下:
1. **`TransactionDefinition.PROPAGATION_REQUIRED`**
使用最多的一个事务传播行为，我们平时经常使用的`@Transactional`注解默认使用就是这个事务传播行为。**如果当前存在事务，则加入该事务；如果当前没有事务，则创建一个新的事务。**
2. **`TransactionDefinition.PROPAGATION_REQUIRES_NEW`**
**创建一个新的事务，如果当前存在事务，则把当前事务挂起。**也就是说不管外部方法是否开启事务，`Propagation.REQUIRES_NEW`修饰的内部方法会新开启自己的事务，且开启的事务相互独立，互不干扰。
3. **`TransactionDefinition.PROPAGATION_NESTED`**
**如果当前存在事务，则创建一个事务作为当前事务的嵌套事务来运行**；如果当前没有事务，则该取值等价于`TransactionDefinition.PROPAGATION_REQUIRED`
4. **`TransactionDefinition.PROPAGATION_MANDATORY`**
**如果当前存在事务，则加入该事务；如果当前没有事务，则抛出异常。**（mandatory：强制性）
**错误**的配置以下3中事务传播行为，事务将不会发生回滚：
- **`TransactionDefinition.PROPAGATION_SUPPORTS`**：如果当前存在事务，则加入该事务；如果当前没有事务，则以非事务的方式继续运行。
- **`TransactionDefinition.PROPAGATION_NOT_SUPPORTED`**：以非事务方式运行，如果当前存在事务，则把当前事务挂起。
- **`TransactionDefinition.PROPAGATION_NEVER`**：以非事务方式运行，如果当前存在事务，则抛出异常。
#### Spring事务失效可能的原因
1. @Transactional应用在给**非public修饰的方法**上：private方法，只会在当前对象中的其他方法中调用，这种情况是用**this调用，并不会走到代理对象**，而 **@Transactional是基于动态代理实现的**，所以代理会失效。
2. **同一个类中方法调用**，导致@Transactional失效：**和private是一回事**，没办法走到代理，所以事务会失效
3. **final、static**方法：AOP通过创建代理对象实现，**无法对final方法进行子类化和覆盖**。**static方法是直接属于这个类的，并不是对象**，所以无法被AOP。
4. 在**多线程情况下是无法生效**的，因为@Transactional的事务管理**使用的是ThreadLocal机制来存储事务上下文**，而ThreadLocal变量是**线程隔离**的，即每个线程都有自己的事务上下文副本。
#### 事务的隔离级别 （[隔离级别](../../数据库/基础和原理/隔离级别.md)）
- **`TransactionDefinition.ISOLATION_DEFAULT`**：使用后端数据库默认的隔离级别，`MySQL`默认采用的`REPEATABLE_READ`隔离级别，`Oracle`默认采用的`READ_COMITTED`隔离级别
- **`TransactionDefinition.ISOLATION_READ_UNCOMMITTED`**：最低的隔离级别，使用这个隔离级别很少，因为它**允许读取尚未提交的数据变更**，**可能会导致脏读、幻读或不可重复读。**[[../../数据库/基础和原理/并发一致性问题（T1，T2指的是事务）|并发一致性问题（T1，T2指的是事务）]]
- **`TransactionDefinition.ISOLATION_REPEATABLE_READ`**：对**同一字段的多次读取结果都是一致的（除非数据是被本身事务自己所修改）**，**可以阻止脏读和不可重复读**，**但幻读仍有可能发生**。
- **`TransactionDefinition.ISOLATION_SERIALIZABLE`**：最高隔离级别，完全服从ACID的隔离级别。所有的事务依次逐个执行，这样的事务之间就完全不可能产生干扰，也就是说，**该级别可以防止脏读、不可重复读以及幻读**。但是这将严重影响程序的性能。通常情况下也不会用到该级别。