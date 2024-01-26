#### 简要介绍
AOP(Aspect Oriented Programming)即面向切面编程。
AOP的**目的**是将横切关注点(如日志记录、事务管理、权限控制、接口限流、接口幂等...)从核心业务逻辑中分离出来，通过动态代理、字节码操作等技术，实现代码的复用和解耦，提高代码的可维护性和可扩展性。

#### AOP关键术语
AOP之所以叫面向切面编程，是因为它的核心思想就是将横切关注点从核心业务逻辑中分离出来，形成一个个的切面(Aspect)。

**关键术语**：
- **横切关注点（cross-cutting concerns)**：多个类或对象中的**公共行为**（如日志记录、事务管理、权限控制、接口限流、接口幂等...）
- **切面（Aspect）**：对横切关注点进行封装的类，一个切面是**一个类**。切面可以定义多个通知，用来实现具体的功能。
- **连接点（JoinPoint）**：连接点是方法调用或者方法执行时的**某个特定时刻**（如方法调用、异常抛出等）。
- **通知（Advice）**：通知就是切面在某个连接点要**执行的操作**。通知有五种类型，分别是前置通知（Before）、后置通知（After）、返回通知（AfterReturning）、异常通知（AfterThrowing）和环绕通知（Around）。前四种通知都是在目标方法的前后执行，而**环绕通知可以控制目标方法的执行过程。**
- **切点（PointCut）**：一个切点是一个**表达式**，它用来**匹配哪些连接点需要被切面所增强**。切点可以通过注释、正则表达式、逻辑运算等方式来定义。比如`excution(*.com.wly.service..*(..))`匹配`com.wly.service`包以及子包下的类或接口。
- **织入（Weaving）**：织入是**将切面和目标对象连接起来的过程**，也就是将通知应用到切点匹配的连接点上。常见的织入时机有两种，分别是编译器织入和运行期织入。
**通知类型**：
- **Before（前置通知）**：目标对象的方法调用之前触发。
- **After（后置通知）**：目标对象的方法调用之后触发。
- **AfterReturning（返回通知）**：目标对象的方法调用完成，在返回结果值之后触发。
- **AfterThrowing（异常通知）**：目标对象的方法运行中抛出/触发异常后触发。**AfterRunning和AfterThrowing两者互斥**。如果方法调用成功无异常，则会有返回值；如果方法抛出异常，则不会有返回值。
- **Around（环绕通知）**：编程式控制目标对象的方法调用。环绕通知是所有通知类型中可操作范围最大的一种，因为它**可以直接拿到目标对象，以及要执行的方法，所以环绕通知可以任意的在目标对象的方法调用前后搞事，甚至不调用目标对象的方法**
#### AOP的实现
AOP的常见实现方式有动态代理、字节码操作等方式。
SpringAOP就是基于动态代理的，如果要代理的对象，实现了某个接口，那么SpringAOP会使用JDK Proxy，去创建代理对象而对于没有实现接口的对象，这时候SpringAOP会使用Cglib生成一个被代理对象的子类来作为代理。
**JDK代理代码示例：**
``` java
public interface Service {
    void operation();
}

public class ServiceImpl implements Service {
    public void operation() {
        System.out.println("Performing operation");
    }
}

public class ServiceInvocationHandler implements InvocationHandler {
    private Object target;

    public ServiceInvocationHandler(Object target) {
        this.target = target;
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        System.out.println("Before operation");
        Object result = method.invoke(target, args);
        System.out.println("After operation");
        return result;
    }
}

// 创建代理对象
Service service = new ServiceImpl();
Service proxy = (Service) Proxy.newProxyInstance(
    Service.class.getClassLoader(),
    new Class[]{Service.class},
    new ServiceInvocationHandler(service));

proxy.operation(); // 调用代理方法

```
**Cglib动态代理代码示例如下：**
``` java
public class SimpleClass {
    public void test() {
        System.out.println("Hello CGLIB");
    }
}

public class MethodInterceptorImpl implements MethodInterceptor {
    @Override
    public Object intercept(Object obj, Method method, Object[] args, MethodProxy proxy) throws Throwable {
        System.out.println("Before method");
        Object result = proxy.invokeSuper(obj, args);
        System.out.println("After method");
        return result;
    }
}

// 创建代理对象
Enhancer enhancer = new Enhancer();
enhancer.setSuperclass(SimpleClass.class);
enhancer.setCallback(new MethodInterceptorImpl());

SimpleClass proxy = (SimpleClass) enhancer.create();
proxy.test(); // 调用代理方法

```