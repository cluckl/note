## 简介

**SpringMVC**：基于Java实现的**MVC架构模式**的**请求驱动**类型的轻量级Web框架。

MVC架构模式：将web层解耦成Model，View，Controller三层，简化开发，便于开发人员之间的配合。

- **Model模型**：包含应用程序的数据。
- **Controller控制器**：包含应用程序的业务逻辑。
- **View视图**：表示以特定格式提供的信息。

![Spring MVC Tutorial](https://static.javatpoint.com/sppages/images/spring-web-model-view-controller.png)



## 工作流程

1. 客户端（浏览器）发送请求，直接请求到<u>**Servlet调度器(DispatcherServlet)**</u>；
2. **<u>Servlet调度器</u>**根据请求信息调用**处理映射器(HandlerMapping)**，解析请求对应的<u>**处理器(Handler)**</u>；
3.  **处理适配器(HandlerAdapter)** 根据<u>**处理器**</u>调用<u>**控制器(Controller)**</u>；
4. <u>**控制器**</u>开始处理请求和业务逻辑，并处理完成后返回一个 **ModelAndView对象；**
5. <u>**视图解析器(ViewResolver)**</u>会根据逻辑View查找实际的View；
6. **<u>Servlet调度器</u>**把返回的<u>**Model**</u>传给<u>**View**</u>进行**<u>视图渲染</u>**之后将**<u>View</u>**返回给客户端。

![img](https://camo.githubusercontent.com/0fd35900c32b252a18ff38855ae52192f90f8884f1622bb5f92578fe68c6cb2f/68747470733a2f2f696d672d626c6f672e6373646e696d672e636e2f696d675f636f6e766572742f64653664326232313366313132323937323938663365323233626630386632382e706e67)

## 拦截器

拦截器：拦截用户请求并处理。

### 过滤器和拦截器的区别

|          | 拦截器                              | 过滤器                                               |
| -------- | ----------------------------------- | ---------------------------------------------------- |
| 实现原理 | 基于反射机制                        | 基于函数回调                                         |
| 使用范围 | 在IoC容器中，可以获取到容器中的bean | 由servlet规范指定，只能在web程序中使用，不能获取bean |
| 作用对象 | 只能拦截非静态资源的请求            | 可以拦截包括对静态资源在内的几乎所有请求             |
| 使用频率 | 被多次调用                          | 只在容器初始化时被调用一次                           |

### 拦截器实现

1. 编写拦截器类实现`HandlerInterceptor`接口，根据实际业务需求重写`preHandle`、`postHandle`或者`afterCompletion`方法

    ```java
    public interface HandlerInterceptor {
        default boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
            return true;
        }

        default void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, @Nullable ModelAndView modelAndView) throws Exception {
        }

        default void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, @Nullable Exception ex) throws Exception {
        }
    }
    ```

2. 利用 `<mvc:interceptors>`标签配置拦截器类的应用场景

   ```xml
       <mvc:interceptors>
           <mvc:interceptor>
               <!--/** 包括路径及其子路径-->
               <!--/admin/* 拦截的是/admin/add等等这种 , /admin/add/user不会被拦截-->
               <!--/admin/** 拦截的是/admin/下的所有-->
               <mvc:mapping path="/**"/>
               <mvc:exclude-mapping path=""/>  
               <!--配置拦截器bean，LoginController实现了HandlerInterceptor方法-->
               <bean class="com.julan.controller.LoginController"/>
           </mvc:interceptor>
       </mvc:interceptors>
   ```

## 控制器

### 相关注解

* @RequestMapping、@PostMapping、@PutMapping、@GetMapping、...
* @Controller
* @PathVariable

### Model

在这个对象里面调用put方法,把对象加到里面,前台就可以通过el表达式拿到。

在类上面加上`@SessionAttributes`注解,注解的value中包含的字符串就是要放入session里面的key。

### 重定向和转发

使用方法：在方法的返回字符串中加相应的前缀

* **转发（默认）**：在返回值前面加"forward:"
* **重定向**：在返回值前面加"redirect:"