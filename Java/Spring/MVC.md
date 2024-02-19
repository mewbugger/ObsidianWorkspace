#### 概念
MVC是一种设计模式，Spring MVC是一款很优秀的MVC框架。Spring MVC可以帮助我们进行更简洁的Web层开发，并且它天生与Spring框架集成。Spring MVC 下我们一般把后端项目分为 Service 层（处理业务）、Dao 层（数据库操作）、Entity 层（实体类）、Controller 层(控制层，返回数据给前台页面)。

#### Spring MVC的核心组件
- **`DispatcherServlet`**：**核心的中央处理器**，负责接受请求，分发，并给予客户端响应。
- **`HandlerMapping`**：**处理器映射器**，根据URL去匹配查找能处理的`Handler`，并会将请求涉及到的拦截器和`Handler`一起封装。
- **`HandlerAdapter`**：**处理器适配器**，根据`HandlerMapping`找到的`Handler`，适配执行对应的`Handler`。
-  **`Handler`**：**请求处理器**，处理实际请求的处理器。
- **`ViewResolver`**：**视图解析器**，根据`Handler`返回的逻辑视图/视图，解析并渲染真正的视图，并传递给`DispatcherServlet`响应客户端。

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
