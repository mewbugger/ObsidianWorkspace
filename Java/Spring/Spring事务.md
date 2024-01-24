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