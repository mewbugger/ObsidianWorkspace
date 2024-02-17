#### Spring支持两种方式的事务管理
- **编程式事务**：在代码中硬编码（在分布式系统中推荐使用）：通过`TransactionTemplate`或者`TransactionManager`手动管理事务，事务范围过大会出现事务未提交导致超时，因此事务要比锁的粒度更小。
- 声明式事务：在XML配置文件中配置或者直接基于注解（单体应用或者简单业务系统推荐使用）：实际是通过AOP实现（基于`@Transactional`的全注释方式使用最多）
#### Spring事务中的事务传播行为
**事务传播行为是为了解决业务层方法之间互相调用的事务问题。**
当事务方法被另一个事务方法调用时，必须指定事务应该如何传播。例如：事务可能继续在现有事务中运行，也可能开启一个新事物，并在自己的事务中运行。
**正确**的事务传播行为可能的值如下:
1. **`TransactionDefinition.PROPAGATION_REQUIRED`**
使用最多的一个事务传播行为，我们平时经常使用的`@Transactional`注解默认使用就是这个事务传播行为。如果当前存在事务，则加入该事务；如果当前没有事务，则创建一个新的事务。
2. **`TransactionDefinition.PROPAGATION_REQUIRES_NEW`**
创建一个新的事务，如果当前存在事务，则把当前事务挂起。也就是说不管外部方法是否开启事务，`Propagation.REQUIRES_NEW`修饰的内部方法会新开启自己的事务，且开启的事务相互独立，互补干扰。
3. **`TransactionDefinition.PROPAGATION_NESTED`**
如果当前存在事务，则创建一个事务作为当前事务的嵌套事务来运行；如果当前没有事务，则该取值等价于`TransactionDefinition.PROPAGATION_REQUIRED`
4. **`TransactionDefinition.PROPAGATION_MANDATORY`**
如果当前存在事务，则加入该事务；如果当前没有事务，则抛出异常。（mandatory：强制性）
**错误**的配置以下3中事务传播行为，事务将不会发生回滚：
- **`TransactionDefinition.PROPAGATION_SUPPORTS`**：如果当前存在事务，则加入该事务；如果当前没有事务，则以非事务的方式继续运行。
- **`TransactionDefinition.PROPAGATION_NOT_SUPPORTED`**：以非事务方式运行，如果当前存在事务，则把当前事务挂起。
- **`TransactionDefinition.PROPAGATION_NEVER`**：以非事务方式运行，如果当前存在事务，则抛出异常。
#### 事务的隔离级别 （[隔离级别](../../数据库/基础和原理/隔离级别.md)）
- **`TransactionDefinition.ISOLATION_DEFAULT`**：使用后端数据库默认的隔离级别，`MySQL`默认采用的`REPEATABLE_READ`隔离级别，`Oracle`默认采用的`READ_COMITTED`隔离级别
- **`TransactionDefinition.ISOLATION_READ_UNCOMMITTED`**：最低的隔离级别，使用这个隔离级别很少，因为它允许读取尚未提交的数据变更，**可能会导致脏读、幻读或不可重复读。**[[../../数据库/基础和原理/并发一致性问题.md|并发一致性问题]]
- **`TransactionDefinition.ISOLATION_REPEATABLE_READ`**：对同一字段的多次读取结果都是一致的，除非数据是被本身事务自己所修改，**可以阻止脏读和不可重复读**，**但幻读仍有可能发生**。
- **`TransactionDefinition.ISOLATION_SERIALIZABLE`**：最高隔离级别，完全服从ACID的隔离级别。所有的事务依次逐个执行，这样的事务之间就完全不可能产生干扰，也就是说，**该级别可以防止脏读、不可重复读以及幻读**。但是这将严重影响程序的性能。通常情况下也不会用到该级别。