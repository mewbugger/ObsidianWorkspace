#### 对Spring IoC的理解
IoC是一种设计思想，而不是一个具体的技术实现。IoC的思想就是将原本在程序中手动创建对象的控制权，交由Spring框架来管理。

**IoC（Inversion of Control：控制反转）**：
- 控制：指的是对象创建（实例化、管理）的权利
- 反转：控制权交给外部环境（Spring框架，IoC容器）

**IoC解释**：
IoC就是将对象之间的依赖关系交给容器来管理，而不是在代码中直接创建和管理这些对象。这种做法能够简化应用程序的开发，并使应用程序摆脱复杂的依赖关系。
**比如**，假设有一个电子商务系统，其中有一个购物车功能。在传统的开发方式中，购物车对象可能需要依赖于商品对象、用户对象等其他对象。在每次使用购物车功能时，都需要手动创建这些依赖对象，管理它们之间的关系，这样会导致代码复杂、难以维护。
而通过IoC容器，我们可以将这些依赖关系的管理交给容器来处理。我们只需要在配置文件或者使用注解的方式中声明需要创建的对象以及它们之间的依赖关系，然后在需要使用这些对象的地方，通过容器来获取它们。容器会根据配置信息自动创建对象，并将依赖关系注入到对象中。
以Spring框架为例，我们可以使用Spring的IoC容器来管理对象之间的依赖关系。通过配置文件（XML）或者注解方式，我们可以声明Bean（对象），并描述它们之间的依赖关系。Spring IoC容器负责创建这些Bean，并在需要时将它们注入到其他Bean中。
**举个例子**，假设我们有一个UserService类，它依赖于一个UserDao类来实现对用户数据的持久化操作。在传统的方式中，我们需要手动创建UserService和UserDao对象，并将UserDao对象传递给UserService。而在使用IoC容器的方式中，我们只需在配置文件中声明UserService和UserDao，并描述它们之间的依赖关系，容器会自动创建并管理它们的生命周期。这样就实现了依赖关系的解耦和管理。

#### Spring Bean是什么
Bean指代的是那些被IoC容器所管理的对象

我们需要告诉 IoC 容器帮助我们管理哪些对象，这个是通过配置元数据来定义的。配置元数据可以是 XML 文件、注解或者 Java 配置类。

#### 将一个类声明为Bean的注解
- `@Component`：通用的注解，可标注任意类为Spring组件。如果一个Bean不知道属于哪一层，可以使用`@Component`注解标注
- `@Repository`：对应持久层即Dao层，主要用于数据库相关操作。
- `@Service`：对应服务层，主要涉及一些复杂的逻辑，需要用到Dao层。
- `@Controller`：对应Spring MVC控制层，主要用于接受用户请求并调用`Service`层返回数据给前端页面。

#### @Component和@Bean的区别
- `@Component`注解作用于类，而@Bean注解作用于方法
- `@Component`通常是通过类路径扫描来自动侦测以及自动装配到Spring容器中个。`@Bean`注解通常是我们在标有该注解的方法中定义产生这个bean，`@Bean`告诉了Spring这是某个类的实例，当需用用它的时候还给我。
- `@Bean` 注解比 `@Component` 注解的自定义性更强，而且很多地方我们只能通过 `@Bean` 注解来注册 bean。比如当我们引用第三方库中的类需要装配到 `Spring`容器时，则只能通过 `@Bean`来实现

#### 注入Bean的注解
`@Autowired`和`@Resource`

#### @Autowired和@Resource的区别
`Autowired` 属于 Spring 内置的注解，默认的注入方式为`byType`（根据类型进行匹配），也就是说会**优先根据接口类型去匹配并注入** Bean （接口的实现类）。

**这就会产生问题**：当一个接口存在多个实现类的话，`byType`这种方式就无法正确注入对象，因为Spring会同时找到多个满足条件的选择。这种情况下，注入方式会变为 `byName`（根据名称进行匹配），这个名称通常就是类名（首字母小写）。

`@Resource`属于 JDK 提供的注解，默认注入方式为 `byName`。如果无法通过名称匹配到对应的 Bean 的话，注入方式会变为`byType`。

#### Bean的作用域
Bean的作用域通常有下面几种：
- **singleton**：IoC容器中只有唯一的bean实例。Spring中的bean默认都是单利的，是对单利设计模式的应用。
- **prototype**：每次获取都会创建一个新的bean实例。即，连续`getBean()`两次，得到的是不同的Bean实例。
- **request（仅Web应用可用）**：每一次HTTP请求都会产生一个新的bean（请求bean），该bean仅在当前HTTP request内有效。
- **session（仅Web应用可用**）：每一次来自新session的HTTP请求都会产生一个新的bean（会话bean），仅在当前HTTP session内有效。
- **application/global-session** （仅 Web 应用可用）：每个 Web 应用在启动时创建一个 Bean（应用 Bean），该 bean 仅在当前应用启动时间内有效。
- **websocket** （仅 Web 应用可用）：每一次 WebSocket 会话产生一个新的 bean。

#### IoC体系结构设计
IoC容器的整体功能，如下图所示：
![](../../img/Pasted%20image%2020240325210910.png)
##### BeanFactory和BeanRegistry：IoC容器功能规范和Bean的注册
Spring Bean的创建是典型的工厂模式，这一系列的Bean工厂，也即IoC容器为开发者管理对象间的依赖关系提供了很多便利和基础服务，在Spring中有许多IoC容器的实现供用户选择和使用，这是IoC容器的基础；在顶层的结构设计主要围绕着`BeanFactory`和`xxxRegistry`进行：
- `BeanFatory`：工厂模式定义了IoC容器的基本功能规范
- `BeanRegistry`：向IoC容器手工注册`BeanDefinition`对象的方法
![](../../img/Pasted%20image%2020240325211326.png)
##### BeanDefinition：各种Bean对象及其相互的关系
Bean对象存在依赖嵌套等关系，所以设计者设计了BeanDefinition，它用来对Bean对象及关系定义；我们在理解时只需要抓住如下三个要点：
- `BeanDefinition`定义了各种Bean对象及其相互的关系
- `BeanDefinitionReader`是BeanDefinition的解析器
- `BeanDefinitionHolder`是BeanDefinition的包装类，用来存储BeanDefinition，name以及aliases等
##### ApplicationContext：IoC接口设计和实现
IoC容器的接口类是`ApplicationContext`，很显然它必然继承BeanFactory对Bean规范（最基本的ioc容器的实现）进行定义。而`ApplicationContext`表示的是应用的上下文，**除了**对Bean的管理外，还至少应该包含了
- **访问资源**： 对不同方式的Bean配置（即资源）进行加载。(实现ResourcePatternResolver接口)
- **国际化**: 支持信息源，可以实现国际化。（实现MessageSource接口）
- **应用事件**: 支持应用事件。(实现ApplicationEventPublisher接口)

#### IoC容器初始化流程的基本步骤
![](../../img/Pasted%20image%2020240325205255.png)
- 初始化的入口在容器实现中的`refresh()`调用来完成
- 对bean定义载入IoC容器使用的是`loadBeanDefinition`其中的大致过程如下：
	- 通过`ResourceLoader`来完成资源文件位置的定位，`DefaultResourceLoader`是默认的实现，同时上下文本身就给出了`ResourceLoader`的实现，可以从类路径，文件系统，URL等方式来定位资源位置。如果是`XmlBeanFactory`作为IoC容器，那么需要为它指定bean定义的资源，也就是说bean定义文件通过抽象成`Resource`来被IoC容器处理。
	- 通过`BeanDefinitionReader`来完成定义信息的解析和Bean信息的注册，往往使用的是`XmlBeanDefinitionReader`来解析bean的xml定义信息，这些信息在Spring中使用`BeanDefinition`对象来表示，这个名字可以让我们想到`loadBeanDefinition`，`RegisterBeanDefinition`这些相关的方法，都是为处理`BeanDefinitin`服务的。
	- 容器解析得到`BeanDefinitin`以后，需要把它在IoC容器中注册，这由IoC实现`BeanDefinitionRegistry`接口来实现。注册过程就是在IoC容器内部维护一个HashMap来保存得到的`BeanDefinition`的过程。这个HashMap是IoC容器持有bean信息的场所，以后对bean的操作都是围绕这个HashMap来实现的。
- 然后我们可以通过`BeanFactory`和`ApplicationContext`来享受到SpringIoC的服务了，使用IoC容器的时候，我们注意到除了少量粘合代码，绝大多数以正确IoC风格编写的应用程序代码完全不用关心如何到达工厂，因为容器将把这些对象与容器管理的其他对象钩在一起。基本的策略是把工厂放到已知的地方，最好是放在对预期使用的上下文有意义的地方，以及代码将实际需要访问工厂的地方。Spring本身提供了对声明式载入web应用程序用法的应用程序上下文，并将其存储在`ServletContext`中的框架实现。
#### BeanFactory中getBean的主体思路
`BeanFactory`实现`getBean`方法在`AbstractBeanFactory`中，这个方法重载都是调用`doGetBean`方法进行实现的：
``` java
public Object getBean(String name) throws BeansException {
  return doGetBean(name, null, null, false);
}
public <T> T getBean(String name, Class<T> requiredType) throws BeansException {
  return doGetBean(name, requiredType, null, false);
}
public Object getBean(String name, Object... args) throws BeansException {
  return doGetBean(name, null, args, false);
}
public <T> T getBean(String name, @Nullable Class<T> requiredType, @Nullable Object... args)
    throws BeansException {
  return doGetBean(name, requiredType, args, false);
}
```
doGetBean方法：
``` java
// 参数typeCheckOnly：bean实例是否包含一个类型检查
protected <T> T doGetBean(
			String name, @Nullable Class<T> requiredType, @Nullable Object[] args, boolean typeCheckOnly)
			throws BeansException {

  // 解析bean的真正name，如果bean是工厂类，name前缀会加&，需要去掉
  String beanName = transformedBeanName(name);
  Object beanInstance;

  // Eagerly check singleton cache for manually registered singletons.
  Object sharedInstance = getSingleton(beanName);
  if (sharedInstance != null && args == null) {
    // 无参单例从缓存中获取
    beanInstance = getObjectForBeanInstance(sharedInstance, name, beanName, null);
  }

  else {
    // 如果bean实例还在创建中，则直接抛出异常
    if (isPrototypeCurrentlyInCreation(beanName)) {
      throw new BeanCurrentlyInCreationException(beanName);
    }

    // 如果 bean definition 存在于父的bean工厂中，委派给父Bean工厂获取
    BeanFactory parentBeanFactory = getParentBeanFactory();
    if (parentBeanFactory != null && !containsBeanDefinition(beanName)) {
      // Not found -> check parent.
      String nameToLookup = originalBeanName(name);
      if (parentBeanFactory instanceof AbstractBeanFactory) {
        return ((AbstractBeanFactory) parentBeanFactory).doGetBean(
            nameToLookup, requiredType, args, typeCheckOnly);
      }
      else if (args != null) {
        // Delegation to parent with explicit args.
        return (T) parentBeanFactory.getBean(nameToLookup, args);
      }
      else if (requiredType != null) {
        // No args -> delegate to standard getBean method.
        return parentBeanFactory.getBean(nameToLookup, requiredType);
      }
      else {
        return (T) parentBeanFactory.getBean(nameToLookup);
      }
    }

    if (!typeCheckOnly) {
      // 将当前bean实例放入alreadyCreated集合里，标识这个bean准备创建了
      markBeanAsCreated(beanName);
    }

    StartupStep beanCreation = this.applicationStartup.start("spring.beans.instantiate")
        .tag("beanName", name);
    try {
      if (requiredType != null) {
        beanCreation.tag("beanType", requiredType::toString);
      }
      RootBeanDefinition mbd = getMergedLocalBeanDefinition(beanName);
      checkMergedBeanDefinition(mbd, beanName, args);

      // 确保它的依赖也被初始化了.
      String[] dependsOn = mbd.getDependsOn();
      if (dependsOn != null) {
        for (String dep : dependsOn) {
          if (isDependent(beanName, dep)) {
            throw new BeanCreationException(mbd.getResourceDescription(), beanName,
                "Circular depends-on relationship between '" + beanName + "' and '" + dep + "'");
          }
          registerDependentBean(dep, beanName);
          try {
            getBean(dep); // 初始化它依赖的Bean
          }
          catch (NoSuchBeanDefinitionException ex) {
            throw new BeanCreationException(mbd.getResourceDescription(), beanName,
                "'" + beanName + "' depends on missing bean '" + dep + "'", ex);
          }
        }
      }

      // 创建Bean实例：单例
      if (mbd.isSingleton()) {
        sharedInstance = getSingleton(beanName, () -> {
          try {
            // 真正创建bean的方法
            return createBean(beanName, mbd, args);
          }
          catch (BeansException ex) {
            // Explicitly remove instance from singleton cache: It might have been put there
            // eagerly by the creation process, to allow for circular reference resolution.
            // Also remove any beans that received a temporary reference to the bean.
            destroySingleton(beanName);
            throw ex;
          }
        });
        beanInstance = getObjectForBeanInstance(sharedInstance, name, beanName, mbd);
      }
      // 创建Bean实例：原型
      else if (mbd.isPrototype()) {
        // It's a prototype -> create a new instance.
        Object prototypeInstance = null;
        try {
          beforePrototypeCreation(beanName);
          prototypeInstance = createBean(beanName, mbd, args);
        }
        finally {
          afterPrototypeCreation(beanName);
        }
        beanInstance = getObjectForBeanInstance(prototypeInstance, name, beanName, mbd);
      }
      // 创建Bean实例：根据bean的scope创建
      else {
        String scopeName = mbd.getScope();
        if (!StringUtils.hasLength(scopeName)) {
          throw new IllegalStateException("No scope name defined for bean ´" + beanName + "'");
        }
        Scope scope = this.scopes.get(scopeName);
        if (scope == null) {
          throw new IllegalStateException("No Scope registered for scope name '" + scopeName + "'");
        }
        try {
          Object scopedInstance = scope.get(beanName, () -> {
            beforePrototypeCreation(beanName);
            try {
              return createBean(beanName, mbd, args);
            }
            finally {
              afterPrototypeCreation(beanName);
            }
          });
          beanInstance = getObjectForBeanInstance(scopedInstance, name, beanName, mbd);
        }
        catch (IllegalStateException ex) {
          throw new ScopeNotActiveException(beanName, scopeName, ex);
        }
      }
    }
    catch (BeansException ex) {
      beanCreation.tag("exception", ex.getClass().toString());
      beanCreation.tag("message", String.valueOf(ex.getMessage()));
      cleanupAfterBeanCreationFailure(beanName);
      throw ex;
    }
    finally {
      beanCreation.end();
    }
  }

  return adaptBeanInstance(name, beanInstance, requiredType);
}

```
**说明**：主要看代码中的中文注释部分即可
 - 解析bean的真正name，如果bean是工厂类，name前缀会加&，需要去掉
 - 无参单利先从缓存中尝试获取
 - 如果bean实例还在创建中，则直接抛出异常
 - 如果bean definition存在于父的bean工厂中，委派给父Bean工厂获取
 - 标记这个beanName的实例正在创建
 - 确保它的依赖也被初始化
 - 真正创建
	 - 单例时
	 - 原型时
	 - 根据bean的scope创建
#### Spring如何解决循环依赖问题
##### Spring单例模式下的属性依赖
三级缓存
``` java
/** Cache of singleton objects: bean name --> bean instance */
private final Map<String, Object> singletonObjects = new ConcurrentHashMap<String, Object>(256);
 
/** Cache of early singleton objects: bean name --> bean instance */
private final Map<String, Object> earlySingletonObjects = new HashMap<String, Object>(16);

/** Cache of singleton factories: bean name --> ObjectFactory */
private final Map<String, ObjectFactory<?>> singletonFactories = new HashMap<String, ObjectFactory<?>>(16);
```
- **第一层缓存（singletonObjects）**：单例对象缓存池，已经实例化并且属性赋值，这里的对象是成熟对象
- **第二次缓存（earlySingletonObjects）**：单例对象缓存池，以及实例化但尚未属性赋值，这里的对象是半成品对象
- **第三层缓存（singletonFactories）**：单例工厂的缓存

**获取单例**
``` java
protected Object getSingleton(String beanName, boolean allowEarlyReference) {
  // Spring首先从singletonObjects（一级缓存）中尝试获取
  Object singletonObject = this.singletonObjects.get(beanName);
  // 若是获取不到而且对象在建立中，则尝试从earlySingletonObjects(二级缓存)中获取
  if (singletonObject == null && isSingletonCurrentlyInCreation(beanName)) {
    synchronized (this.singletonObjects) {
        singletonObject = this.earlySingletonObjects.get(beanName);
        if (singletonObject == null && allowEarlyReference) {
          ObjectFactory<?> singletonFactory = this.singletonFactories.get(beanName);
          if (singletonFactory != null) {
            //若是仍是获取不到而且容许从singletonFactories经过getObject获取，则经过singletonFactory.getObject()(三级缓存)获取
              singletonObject = singletonFactory.getObject();
              //若是获取到了则将singletonObject放入到earlySingletonObjects,也就是将三级缓存提高到二级缓存中
              this.earlySingletonObjects.put(beanName, singletonObject);
              this.singletonFactories.remove(beanName);
          }
        }
    }
  }
  return (singletonObject != NULL_OBJECT ? singletonObject : null);
}
```
分析`getSingleton()`的整个过程，Spring首先从一级缓存`singletonObjects`中获取。若是获取不到，而且对象正在建立中，就再从二级缓存`earlySingletonObjects`中获取。若是仍是获取不到且容许`singletonFactories`经过getObject()获取，就从`三级缓存singletonFactory.getObject()`(三级缓存)获取，若是获取到了则从三级缓存移动到了二级缓存。

Spring**解决循环依赖**的诀窍就在于`singletonFactories`的三级缓存
这个缓存的类型是`ObjectFactory`
``` java
public interface ObjectFactory<T> {
    T getObject() throws BeansException;
}
```
在bean创建过程当中，有两处比较重要的匿名内部类实现了该接口。一处是Spring利用其建立bean的时候，另外一处就是：
``` java
addSingletonFactory(beanName, new ObjectFactory<Object>() {
   @Override   public Object getObject() throws BeansException {
      return getEarlyBeanReference(beanName, mbd, bean);
   }});
```
**此处就是解决循环依赖的关键**，这段代码发生在`createBeanInstance`以后，也就是说单例对象此时已经被建立出来的。这个对象已经被生产出来了，虽然还不完美（尚未进行初始化的第二步和第三步），可是已经能被人认出来了（根据对象引用能定位到堆中的对象），因此Spring此时将这个对象提早曝光出来让你们认识，让你们使用。

**类比**：好比“A对象setter依赖B对象，B对象setter依赖A对象”，A首先完成了初始化的第一步，而且将本身提早曝光到singletonFactories中，此时进行初始化的第二步，发现本身依赖对象B，此时就尝试去get(B)，发现B尚未被create，因此走create流程，B在初始化第一步的时候发现本身依赖了对象A，因而尝试get(A)，尝试一级缓存singletonObjects(确定没有，由于A还没初始化彻底)，尝试二级缓存earlySingletonObjects（也没有），尝试三级缓存singletonFactories，因为A经过ObjectFactory将本身提早曝光了，因此B可以经过ObjectFactory.getObject拿到A对象(半成品)，B拿到A对象后顺利完成了初始化阶段一、二、三，彻底初始化以后将本身放入到一级缓存singletonObjects中。此时返回A中，A此时能拿到B的对象顺利完成本身的初始化阶段二、三，最终A也完成了初始化，进去了一级缓存singletonObjects中，并且更加幸运的是，因为B拿到了A的对象引用，因此B如今hold住的A对象完成了初始化。
##### Spring为什么不能解决非单例属性之外的循环依赖
- **Spring为什么不能解决构造器的循环依赖**：构造器注入形成的循环依赖，也就是beanB需要在beanA的构造函数中完成初始化，beanA也需要再beanB的构造函数中完成初始化，这种情况的结果就是两个bean都不能完成初始化，循环依赖难以解决
- **Spring为什么不能解决多例的循环依赖**：这种循环依赖同样无法解决，因为Spring不会缓存prototypre作用域的bean，而spring中循环依赖的解决正是通过缓存来实现的。
- **Spring为什么不能解决多例的循环依赖**：多实例Bean是每次调用一次getBean都会执行一次构造方法并且给属性赋值，根本没有三级缓存，因此不能解决循环依赖。
##### 其他循环依赖如何解决
- **生成代理对象产生的循环依赖**
这类循环依赖问题解决方法很多，主要有：
1. 使用@Lazy注解，延迟加载
2. 使用@DependsOn注解，指定加载先后关系
3. 修改文件名称，改变循环依赖类的加载顺序
- **使用@DependsOn产生的循环依赖**
这类循环依赖问题要找到@DependsOn注解循环依赖的地方，迫使它不循环依赖就可以解决问题。
- **多例循环依赖**
这类循环依赖问题可以通过把bean改成单例的解决。
- **构造器循环依赖**
这类循环依赖问题可以通过使用@Lazy注解解决。
#### Spring中Bean的生命周期
![](../../img/Pasted%20image%2020240325234129.png)
![](../../img/Pasted%20image%2020240325234209.png)