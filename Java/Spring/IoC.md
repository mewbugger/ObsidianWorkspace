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
