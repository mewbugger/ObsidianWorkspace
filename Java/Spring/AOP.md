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

#### AOP实例（日志记录）
##### 切面实例
``` java
@Aspect  // 标记一个类是切面类
@Component  
@Slf4j  
public class WebLogAop
```
##### 切点实例

``` java
/**
* 定义了切面的拦截规则， 拦截带有@RestController注解的类
*/
@Pointcut("@within(org.springframework.web.bind.annotation.RestController)")  
public void webLog() {  
}
```

##### 前置通知实例
``` java
/**  
 * @param joinPoint 切点  JoinPoint对应的是@Pointcut拦截到的方法及其响应入参，注解等
 */
@Before("webLog()")  
    public void before(JoinPoint joinPoint) throws Exception {  
        ServletRequestAttributes attributes = (ServletRequestAttributes) RequestContextHolder.getRequestAttributes();  
        HttpServletRequest request = attributes.getRequest();  
        MDC.put("requestId", String.valueOf(System.currentTimeMillis()));  
        MDC.put("requestUrl", request.getRequestURI());  
        Object[] args = joinPoint.getArgs();  
        // 获取方法上的注解  
        Signature signature = joinPoint.getSignature();  
        MethodSignature methodSignature = (MethodSignature) signature;  
		// 获取参数名称 一一对应  
        String[] parameterNames = methodSignature.getParameterNames();  
		// 获取参数类型 一一对应  
        Class[] parameterTypes = methodSignature.getParameterTypes();  
        for (int i = 0; i < args.length; i++) {  
            if (args[i] == null || args[i] instanceof HttpServletResponse || args[i] instanceof HttpServletRequest) {  
                continue;  
            }  
            try {  
                if (MultipartFile.class == parameterTypes[i]) {  
                    //说明这次传参是文件类型  
                    MultipartFile file = (MultipartFile) args[i];  
                    log.info("参数类型是文件,文件名称:{},文件大小:{}KB", file.getOriginalFilename(), file.getSize() / 1024);  
                } else {  
                    log.info("请求参数为:{}", JSON.toJSONString(args[i]));  
                }  
            } catch (Exception e) {  
                e.printStackTrace();  
                log.error("请求参数转换json失败");  
            }  
        }  
    }
```

##### 返回通知实例
``` java
/**  
 * @param ret 控制器返回对象  
 */  
@AfterReturning(returning = "ret", pointcut = "webLog()")  
public void afterReturn(JoinPoint joinPoint, Object ret) {  
    String requestId = MDC.get("requestId");  
    String interval = "";  
    if (StringUtils.isNotBlank(requestId)) {  
        Long startTime = Long.valueOf(requestId);  
        Long endTime = System.currentTimeMillis();  
        interval = "处理时间:" + (endTime - startTime) + "ms";  
    }  
    log.info(interval + "   响应数据为:" + JSON.toJSONString(ret));  
    MDC.clear();  
}
```

##### 异常通知实例
``` java
/**  
 * 异常通知  
 *  
 * @param exception  
 */  
@AfterThrowing(pointcut = "webLog()", throwing = "exception")  
public void afterThrowing(JoinPoint joinPoint, Exception exception) {  
    String requestId = MDC.get("requestId");  
    String interval = "";  
    if (StringUtils.isNotBlank(requestId)) {  
        Long startTime = Long.valueOf(requestId);  
        Long endTime = System.currentTimeMillis();  
        interval = "处理时间:" + (endTime - startTime) + "ms";  
    }  
    exception.printStackTrace();  
    log.error(interval + "  接口异常", exception);  
    MDC.clear();  
}
```
#### AOP的实现
AOP的常见实现方式有动态代理、字节码操作等方式。
SpringAOP就是基于动态代理的，**如果要代理的对象，实现了某个接口，那么SpringAOP会使用JDK Proxy**，去创建代理对象而**对于没有实现接口的对象，这时候SpringAOP会使用Cglib生成一个被代理对象的子类来作为代理。**

**Java的动态代理的最主要的用途就是应用在各种框架中。因为使用动态代理可以很方便地运行生成代理类，通过代理类可以做很多事情，比如AOP，比如过滤器、拦截器。**

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

#### Spring官方文档不推荐基于字段来实现依赖注入
**基于字段依赖注入：**
``` java
@Autowired  
private ProductService productService;
```
**原因：**
1. **可能会出现NPE问题**： 在Spring中，基于字段的依赖注入是通过反射机制在对象实例化之后进行的。这意味着，在对象的构造函数执行完毕之后，才开始进行依赖注入。如果在构造函数中、构造后的初始化方法前，或者在依赖注入之前尝试访问这些依赖的字段，这些字段可能还未被注入，从而导致NPE。
#### AOP不起作用的原因
1. **私有方法调用**：AOP通常依赖于代理机制来拦截方法调用。代理类可以实现接口或继承目标类，但代理只能拦截在其接口或父类中公开的方法。由于私有方法仅在其定义的类中可见，代理类无法直接访问私有方法，因此无法拦截和应用切面逻辑。
2. **静态方法调用**：静态方法是类级别的，直接绑定到类本身，而不是某个对象实例。因此，代理类无法将静态方法的调用委派到目标类的实例中。这使得代理无法在静态方法的执行期间插入切面逻辑。
3. **final方法调用**：final方法在子类中无法被重写。AOP的代理机制通常通过子类重写目标类的方法来实现切面逻辑的注入，但final方法阻止了这种重写行为，从而使切面无法应用于final方法。
4. **类内部自调用**：当一个类的方法在同一个类的另一个方法中被调用时，这种自调用不会通过代理对象，而是直接在对象自身上调用。因此，代理无法拦截这些内部调用，使得AOP的切面逻辑无法应用于自调用。
5. **内部类方法调用**：内部类通常持有对其外围类的引用，并且它们的实例通常直接创建在外围类的上下文中。代理无法拦截外围类与内部类之间的直接方法调用，因为这种调用是直接发生的，而不是通过代理进行的。因此，切面逻辑无法应用于内部类与外围类之间的直接调用。