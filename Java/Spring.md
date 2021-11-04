# Spring简介

Spring是一个轻量级的**IoC**和**AOP**容器框架。是为Java应用程序提供基础性服务的一套框架，目的是用于简化企业应用程序的开发，它使得开发者只需要关心业务需求。

* **低侵入式设计，代码的污染极低**
* DI机制将对象之间的依赖关系交由框架处理，减低组件的耦合性
* 提供了AOP技术，支持将一些通用任务，如安全、事务、日志、权限等进行集中式管理，从而提供更好的复用

* 对主流的应用框架提供了集成支持

## 涉及到的设计模式

* 工厂模式：BeanFactory是简单工厂模式的体现，用来创建bean的实例
* 单例模式：Bean的作用范围默认为单例
* 代理模式：Spring的AOP是基于JDK和CGLIB的动态代理
* 模板方法模式：`org.mybatis.spring.SqlSessionTemplate`等`*Template`类
* 观察者模式：Spring 的**<u>事件驱动模型</u>**，其中`ApplicationEventPublisher`是事件发布者，`ApplicationListener`是观察者
* 适配器模式：Spring AOP的**<u>通知(Advice)</u>**；Spring MVC的**处理适配器(HandlerAdapter)**

# Bean

## 声明对应类为Bean的注解

- `@Component` ：通用的注解，可标注任意类为 `Spring` 组件
- `@Repository` : 对应持久层即 Dao 层
- `@Service` : 对应服务层
- `@Controller` : 对应 Spring MVC 控制层

## 作用域

### 类型

* singleton: 将单个 bean 定义范围限定为**每个 Spring IoC 容器**的单个对象实例。默认作用域。
* prototype: 将单个 bean 定义范围限定为**任意数量的对象实例**。
* reuqest:  将单个 bean 定义范围限定为**单个 HTTP 请求**。
* session:  将单个 bean 定义范围限定为**单个 HTTP Seesion**。
* global session: 将单个 bean 定义范围限定为**全局的HTTP Seesion**。

### 配置方式

* xml 方式：在`<bean>`中设置`scope`属性

```xml
<bean id="..." class="..." scope="singleton"></bean>
```

* 注解方式：使用@Scope声明bean的作用域。

```java
@Bean
@Scope(value = ConfigurableBeanFactory.SCOPE_PROTOTYPE)
public Person personPrototype() {
    return new Person();
}
```

## 管理方式

1. 配置文件xml中，用`< bean id="" class=""/>`这种bean标签形式

2. 通过注解的形式，如@Bean,@ Component,@ Service,@Dao,@ Repository等标签

3. 实现 `Factorybean`接口，将bean托管给IOC容器

## 生命周期

1. 实例化：IoC容器调用`createBean`方法实例化bean对象

2. 设置属性：实例化后的对象被封装在`BeanWrapper`对象中，然后Spring根据`BeanDefinition`中的信息以及通过`BeanWrapper`提供的设置属性的接口完成**<u>依赖注入</u>**

3. 处理Aware接口：如果Bean实现了`BeanNameAware`、`BeanFactoryAware`、`ApplicationContextAware`等`*Aware`接口,，则调用对应的`set*`方法

4. 前置处理：如果bean实现了`BeanPostProcessor`接口，调用`postProcessBeforeInitialization(Object obj, String s)`方法

5. 初始化：依次调用`InitializingBean`接口的`afterPropertiesSet()`方法的和`init-method`属性指定的方法

6. 后置处理：如果bean实现了`BeanPostProcessor`接口，调用`postProcessAfterInitialization(Object obj, String s)`方法

   ---

7. 销毁bean：依次调用`DisposableBean`接口的 `destroy()` 方法和`destroy-method`属性指定的方法

![img](https://upload-images.jianshu.io/upload_images/6393906-e32ad6e42a0e96d8.png?imageMogr2/auto-orient/strip|imageView2/2/w/1057/format/webp)

## 线程安全问题

Spring中的singleton bean 不是线程安全的，如果要处理并发问题，可以尝试的方法：

* 将bean的作用域改成 `prototype`或者避免定义可变的成员变量
* 使用**ThreadLocal**，将**<u>可变的成员变量</u>**封装进ThreadLocal对象中

## `@Component` 和 `@Bean` 的区别

1. 作用目标：`@Component` 注解作用于类；`@Bean`注解作用于配置类（被`@Configuration`）的方法。
2. 原理：`@Component`通常是通过**<u>类路径扫描</u>**来自动侦测及自动装配bean到Spring容器中；`@Bean` 注解直接在作用的方法中产生这个 bean。
3. 适用性：`@Bean` 注解比 `@Component` 注解的适用范围强，一些情况下只能通过 `@Bean` 注解来注册 bean，如将第三方库中的类声明为bean时。

# IoC

IoC（Inverse of Control:控制反转）：将原本在程序中<u>**手动创建对象的控制权**</u>和<u>**对象之间的相互依赖关系**</u>交给IoC容器来管理。 IoC 容器是 Spring 用来实现 IoC 的载体， IoC 容器本质是个Map，用于存放各种对象实例。

* **对象与对象之间松散耦合，利于功能的复用**
* 简化应用开发，增加了项目的可维护性且降低开发难度

## 实现方式

在spring中，IoC通过DI（Dependency Injection，依赖注入）实现。

* 基于setter方法注入
* 基于构造器参数注入
* 基于注解的注入

# AOP

AOP（Aspect-oriented programming，面向切面的程序设计）：作为面向对象的一种补充，用于将与业务无关，但却对多个对象产生影响的**公共行为和逻辑**，抽取并封装为一个可重用的模块，即切面（Aspect）。

* 减少系统中的重复代码
* 降低模块间的耦合度
* 提高了系统的可维护性
* 可用于权限认证、日志、事务处理

## AOP 概念

* 切面（Aspect）：被抽取的公共模块，可能会横切多个对象

* 连接点（Join point）：程序执行的某个特定位置（如某个方法调用前、调用后，方法抛出异常后）。
* 通知（Advice）：切面在特定连接点采取的动作
* 切入点（Pointcut）：由**切入点表达式**匹配，通知将在与切入点匹配的任何连接点处运行

* 目标对象（Target Object）：被一个或者多个切面所通知的对象
* 织入（Weaving）：**在运行时**将切面和目标对象链接起来创建新的代理对象的过程

### 通知类型

* 前置通知（Before advice）：在连接点之前执行的通知，但其不能阻止连接点的执行（除非它抛出一个异常）。
* 后置通知（After returning advice）：在连接点正常完成后执行的通知。
* 环绕通知（Around Advice）：包围一个连接点（join point）的通知，可以在方法调用前后都执行的通知。
* 异常返回通知（After throwing Advice）：在方法抛出异常退出时执行的通知。
* 返回通知(After (finally) Advice)：不管连接点退出的方式（正常或异常返回）都将执行的通知。

## 原理

本质是通过**JDK动态代理**和**CGLIB动态代理**实现的。

不管是前置增强、后置增强还是环绕增强，底层都是通过`MethodInterceptor`，即前置增强和后置增强本质仍然是环绕增强，通过将`Advice`转换成`MethodInterceptor`去实现。

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/432bfb2b2c894f85bdbe2b4036345e32~tplv-k3u1fbpfcp-watermark.awebp)

CGLIB动态代理使用场景：

* 目标对象没有实现接口
* `proxy-target-class`的值为true
* 使用了优化策略

## 拦截器步骤

1. 设置切面类和通知。
2. 设置切入点和连接点。

# 事务传播机制

事务传播机制：定义事务如何相互关联，即<u>**解决业务层方法之间互相调用的事务问题**</u>。

类型：

0. **REQUIRED**：支持当前事务，如果没有事务就新建事务
1. **SUPPORTS**：支持当前事务，如果没有事务就以非事务的方式运行
2. **MANDATORY**：支持当前事务，如果没有事务就抛出异常
3. **REQUIRES_NEW**：新建一个事务，如果当前存在事务，则挂起该事务
4. **NOT_SUPPORTED**：不支持事务，如果当前存在事务就将该事务挂起
5. **NEVER**：不支持事务，如果有事务就抛出异常
6. **NESTED**：如果当前事务存在，则在嵌套事务中执行，否则新建一个事务