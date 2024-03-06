#### 概念
MVC是一种设计模式，Spring MVC是一款很优秀的MVC框架。Spring MVC可以帮助我们进行更简洁的Web层开发，并且它天生与Spring框架集成。Spring MVC 下我们一般把后端项目分为 Service 层（处理业务）、Dao 层（数据库操作）、Entity 层（实体类）、Controller 层(控制层，返回数据给前台页面)。

#### Spring MVC的核心组件
- **`DispatcherServlet`**：**核心的中央处理器**，负责接受请求，分发，并给予客户端响应。
- **`HandlerMapping`**：**处理器映射器**，根据URL去匹配查找能处理的`Handler`，并会将请求涉及到的拦截器和`Handler`一起封装。
- **`HandlerAdapter`**：**处理器适配器**，根据`HandlerMapping`找到的`Handler`，适配执行对应的`Handler`。
-  **`Handler`**：**请求处理器**，处理实际请求的处理器。
- **`ViewResolver`**：**视图解析器**，根据`Handler`返回的逻辑视图/视图，解析并渲染真正的视图，并传递给`DispatcherServlet`响应客户端。

##### DispatcherServlet
**工作流程**：
1. **接收请求**：当客户端发送一个 `HTTP` 请求到应用程序时，请求首先到达 `DispatcherServlet`。
2. **请求映射**：`DispatcherServlet` 使用 `HandlerMapping`（处理程序映射器）来确定哪个控制器（Controller）应该处理这个请求。
3. **处理器链**：`HandlerMapping`将返回一个`HandlerExecutionChain`对象，包含了要执行的处理器（`Controller`）以及拦截器（`Interceptors`）链。处理器表示真正执行业务逻辑的组件，拦截器用于在处理器执行前后进行预处理和后处理。
4. **调用处理器适配器**：找到合适的处理器后，`DispatcherServlet` 会调用 `HandlerAdapter`（处理器适配器）来调用处理器。
5. **执行处理器**：HandlerAdapter 会调用具体的处理（Controller）来处理请求。
6. **返回 ModelAndView**：处理器执行完成后，会返回一个`ModelAndView` 对象，该对象包含了模型数据和视图名。
7. **视图解析**：`DispatcherServlet` 会将 `ModelAndView` 对象传给 `ViewResolver`（视图解析器）来解析得到具体的视图。
8. **渲染视图**：`DispatcherServlet` 会根据 `ViewResolver` 解析得到的视图来渲染视图，即将模型数据填充到视图中。
9. **响应用户**：最后，`DispatcherServlet` 会将渲染后的视图返回给用户。

##### HandlerMapping
`HandlerMapping` 的作用是根据请求的 URL，将请求映射到相应的处理器(Controller)上。它负责确定哪个处理器将处理特定的请求，以及哪些拦截器将应用于处理器。`HandlerMapping` 主要用于决定请求如何被处理，确定请求处理链。（**`RequestMapping`**）
##### HandlerAdapter
`HandlerAdapter` 的作用是将处理器（Controller）适配成标准的 Spring MVC Controller 接口。由于 Spring MVC 中的处理器可以是任意的 Java 对象，而不一定要实现特定的接口，因此需要适配器将处理器包装成标准的接口，以便 `DispatcherServlet` 能够调用它们。
下面是一个普通的Java类：
``` java
public class UserController {
    public String getUserDetail(Model model) {
        // 处理用户详情逻辑，将用户信息放入模型中
        model.addAttribute("username", "John");
        return "userDetailPage"; // 返回逻辑视图名
    }
}
```
想要实现调用接口功能，需要实现`SpringMVC`设定的标准`Controller`接口。
例如下面：
``` java
import org.springframework.web.servlet.ModelAndView;
import org.springframework.web.servlet.mvc.Controller;
public class UserControllerAdapter implements Controller {
    private UserController userController;
    public UserControllerAdapter(UserController userController) {
        this.userController = userController;
    }
    @Override
    public ModelAndView handleRequest(javax.servlet.http.HttpServletRequest httpServletRequest, javax.servlet.http.HttpServletResponse httpServletResponse) throws Exception {
        ModelAndView modelAndView = new ModelAndView();
        String viewName = userController.getUserDetail(modelAndView.getModel());
        modelAndView.setViewName(viewName);
        return modelAndView;
    }
}

```
实际开发中则是使用`@RestController`或`@Controller`注解来实现上述功能。
#### Spring MVC工作原理
![](../../img/Pasted%20image%2020240219221621.png)
**流程说明**：
1. 客户端（浏览器）发送请求， `DispatcherServlet`拦截请求。
2. `DispatcherServlet` 根据请求信息调用 `HandlerMapping` 。`HandlerMapping` 根据 URL 去匹配查找能处理的 `Handler`（也就是我们平常说的 `Controller` 控制器） ，并会将请求涉及到的拦截器和 `Handler` 一起封装。
3.  `DispatcherServlet` 调用 `HandlerAdapter`适配器执行 `Handler` 。
4.  `Handler` 完成对用户请求的处理后，会返回一个 `ModelAndView` 对象给`DispatcherServlet`，`ModelAndView` 顾名思义，包含了数据模型以及相应的视图的信息。`Model` 是返回的数据对象，`View` 是个逻辑上的 `View`。
5.  `ViewResolver` 会根据逻辑 `View` 查找实际的 `View`。
6.  `DispaterServlet` 把返回的 `Model` 传给 `View`（视图渲染）。
7.  把 `View` 返回给请求者（浏览器）
#### 工作原理流程补充
![](../../img/Pasted%20image%2020240306145850.png)
**Filter(ServletFilter)**（比Interceptor优先执行）：
进入Servlet前可以有preFilter, Servlet处理之后还可有postFilter