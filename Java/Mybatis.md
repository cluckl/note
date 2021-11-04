## Mybatis 

### 简介

一个ORM类型的半自动框架，执行了对JDBC的封装，是一个持久层框架，它可以通过XML文件或者注解来配置和映射原始类型、接口和 Java POJO（Plain Old Java Objects，普通老式 Java 对象）为数据库中的记录。

### 优缺点

* 优点：
  * 解除sql与程序代码的耦合
  * 基于SQL语句编译，相当灵活；
  * 兼容各种主流数据库；
  * 支持动态SQL；
  * 强大的语句映射，与JDBC相比，减少了大部分的代码量；
  * 能够与Spring很好的集成，提供映射标签。
* 缺点：
  * SQL语句依赖于数据库，导致数据库移植性较差；
  * 编写SQL语句时工作量很大，尤其是字段多、关联表多时.

### 适用场合

MyBatis专注于SQL本身，是一个足够灵活的DAO层解决方案，对性能要求很高，或者需求变化较多的项目。

### 组成

#### Mybatis配置文件

MyBatis 的配置文件包含了会深深影响 MyBatis 行为的设置和属性信息。

配置文件的顶层结构：

configuration（配置）

- properties（属性）
  - 可以在resource/url 属性中指定的配置文件中配置属性，也可以在 properties 元素的子元素中设置
  - resource/url 属性中指定的配置文件的优先级高于 properties 元素中指定的属性
- settings（设置）
  - cacheEnabled：全局性地开启或关闭所有映射器配置文件中已配置的二级缓存，默认开启。
  - logImpl：指定 MyBatis 所用日志的具体实现方式，未指定时将自动查找。
- typeAliases（类型别名）
  - 为 Java 类型设置一个缩写名字。 它仅用于 XML 配置，意在降低冗余的**全限定类名**书写。
- 类型处理器（typeHandlers）
  - 将预处理语句（PreparedStatement）中的参数或从结果集中取出的值以合适的方式转换成 Java 类型
- 对象工厂（objectFactory）
  - 在每次 MyBatis 创建结果对象的新实例时完成实例化工作。
- 插件（plugins）
- environments（环境配置）：
  - environment（环境变量）
    - transactionManager（事务管理器）
      - 有两种类型的事务管理器（也就是 type="[JDBC|MANAGED]"）
      - **JDBC**：直接使用了 JDBC 的提交和回滚设施。
      - MANAGED – 这个配置几乎没做什么。它从不提交或回滚一个连接。
    - dataSource（数据源）
      - 有三种内建的数据源类型（也就是 type="[UNPOOLED|POOLED|JNDI]"）
      - UNPOOLED:  这个数据源的实现会每次请求时打开和关闭连接。
      - **POOLED**: 利用“池”的概念将 JDBC 连接对象组织起来，避免了创建新的连接实例时所花费的时间。
- databaseIdProvider（数据库厂商标识）
- mappers（映射器）
  - 告诉 MyBatis 到哪里去找映射文件

##### 一对一和一对多关联查询

一对一的关联查询：使用association标签，其属性如下：

* property:对象属性的名称.
* **javaType:对象属性的类型.**
* column:所对应的外键字段名称.
* select:使用另一个查询封装的结果

一对多的关联查询：使用collcetion标签，其属性如下：

* property:指的是集合属性的值.
* **ofType:指的是集合中元素的类型.**
* column:所对应的外键字段名称.
* select:使用另一个查询封装的结果.

##### 例子

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<!-- MyBatis的主配置文件 -->
<configuration>
    <properties resource="mysql.properties"/>

    <settings>
        <setting name="logImpl" value="LOG4J"/>
    </settings>

    <typeAliases>
        <typeAlias type="com.julan.entity.Student" alias="Student"/>
        <typeAlias type="com.julan.entity.Teacher" alias="Teacher"/>
    </typeAliases>

    <!-- 配置环境 -->
    <environments default="mysql">
        <!-- 配置MySQL环境 -->
        <environment id="mysql">
            <!-- 配置事务类型 -->
            <transactionManager type="JDBC"/>
            <!-- 配置数据源/连接池 -->
            <dataSource type="POOLED">
                <!-- 配置连接数据库的4个基本信息  -->
                <property name="driver" value="${driver}"/>
                <property name="url" value="${url}"/>
                <property name="username" value="${username}"/>
                <property name="password" value="${password}"/>
            </dataSource>
        </environment>
    </environments>

    <!-- 指定映射配置文件的位置 -->
    <mappers>
        <mapper resource="com/julan/dao/StudentDao.xml"/>
        <mapper resource="com/julan/dao/TeacherDao.xml"/>
    </mappers>
</configuration>
```

#### Mapper映射文件

SQL 映射文件的几个顶级元素（按照应被定义的顺序列出）：

* `select|insert|updae|delete`：对应数据库的查询操作和DML操作。
* <u>`resultMap` ：描述如何从数据库结果集中加载对象，是最复杂也是最强大的元素。</u>

- `sql` ：定义可重用的 SQL 代码片段，以便在其它语句中使用。
- `cache` – 该命名空间的缓存配置。
- `cache-ref` – 引用其它命名空间的缓存配置。

##### 字符串替换

一般使用用 `#{}` 参数（首选）或者用 ${}` 参数进行字符串替换：

* #{}：是预编译处理。
  * Mybatis在处理#{}时，会将sql中的#{}替换为?号，调用PreparedStatement的set方法来赋值。
  * 可以防止SQL注入，提高系统的安全性。
* \${}：直接插入一个不转义的字符串，即将${}替换成变量的值，属于**<u>静态文本替换</u>**。

#####  在获取查询结果中，实体类中的属性名和表中的字段名不一致的解决方式 

* 在查询的sql语句中定义字段名的别名，让字段名的别名和实体类的属性名一致
* 通过`resultMap` 来映射字段名和实体类属性名的一一对应关系

##### 不同的Xml映射文件，id是否可以重复？

如果配置了namespace，那么id可以重复；如果没有配置namespace，那么id不能重复。

但是除非使用旧版本，否则XML映射文件必须配置namespace，因此id可以重复。

##### Xml映射文件对应DAO接口的工作原理

Dao接口即 `Mapper`接口，本质使用了**<u>JDK动态代理</u>**

1. 接口的全限名，就是映射文件的namespace的值，接口的方法名，就是映射文件中Mapper的Statement的id值，接口方法内的参数，就是传递sql的参数。
2. 在 MyBatis 中，每一个`<select>`、`<insert>`、`<update>`、`<delete>`标签，都会被解析为一个`MappedStatement`对象。当调用接口方法时，**<u>接口的全限名+方法名拼接成的字符串</u>**作为 key 值，可唯一定位一个`MappedStatement`

## 接口绑定

接口绑定：定义接口，然后把接口里面的方法和sql语句绑定，直接调用接口方法即可。
有两种实现方式：

1. 注解绑定：在接口的方法上面加上@select，@update等注解，里面包含sql语句
2. XML绑定：通过xml里面写sql语句来绑定

<u>当sql语句比较简单的时候，用注解绑定，当sql语句比较复杂的时候，用xml绑定，一般使用xml绑定的比较多。</u>

### SqlSessionFactory

SqlSessionFactory：MyBatis的应用的核心，单个数据库映射关系经过编译后的内存映像，也是创建`SqlSession`的工厂。

获取方式：通过`SqlSessionFactoryBuilder`获得，而`SqlSessionFactoryBuilder`则可以从XML配置文件或一个预先定制的`Configuration`的实例构建出`SqlSessionFactory`的实例。

### SqlSession

* 执行持久化操作的对象，可以通过SqlSession实例来直接执行已映射的SQL语句
* 包含了面向数据库执行SQL命令所需的所有方法
  

## 参数传递

* 顺序传参法：在Mapper映射文件中的`#{}`传入数字，表示Mapper接口对应方法的参数的序号。

```xml
public User selectUser(String name, int deptId);

<select id="selectUser" resultMap="UserResultMap">
    select * from user
    where user_name = #{0} and dept_id = #{1}
</select>
```

2. @Param注解传参法：Mapper接口的方法的入参用@Param注解修饰，其值表示在Mapper映射文件的`#{}`写入的值。适合参数较少的情况下使用。

```xml
public User selectUser(@Param("userName") String name, int @Param("deptId") deptId);

<select id="selectUser" resultMap="UserResultMap">
    select * from user
    where user_name = #{userName} and dept_id = #{deptId}
</select>
```

3. Map传参法： Mapper接口的方法传入一个`java.util.Map`类，Mapper映射文件的`#{}`写入map的key。适合参数较多的情况下使用。

```xml
public User selectUser(Map<String, Object> params);

<select id="selectUser" parameterType="java.util.Map" resultMap="UserResultMap">
    select * from user
    where user_name = #{userName} and dept_id = #{deptId}
</select>
```

4. Java Bean传参法：Mapper接口的方法传入一个实体类，Mapper映射文件的`#{}`写入实体类的字段名。

```xml
public User selectUser(User params);

<select id="selectUser" parameterType="com.test.User" resultMap="UserResultMap">
    select * from user
    where user_name = #{userName} and dept_id = #{deptId}
</select>
```

## 映射

### 自动映射

通过使用==resultType==标签，可以自动映射查询结果。

### 结果映射

结果映射（resultMap）：和自动映射（resultType）返回类型之一。

在 ResultSet 出现的列，如果没有设置手动映射，将被自动映射。在自动映射处理完毕后，再处理手动映射。

### resultMap和resultType比较

1. resultType跟resultMap不能同时存在，且必须存在一个。
2. 当提供的返回类型属性是resultType时，MyBatis会将Map里面的键值对取出赋给resultType所指定的对象对应的属性。所以其实MyBatis的每一个查询映射的返回类型都是ResultMap，只是<u>当提供的返回类型属性是resultType的时候，MyBatis自动地把对应的值赋给resultType所指定对象的属性</u>。
3. 当提供的返回类型是resultMap时，因为Map不能很好表示领域模型，就需要自己再进一步的把它转化为对应的对象，这常常在复杂查询中很有作用。

## 动态SQL

执行原理：根据表达式的值，完成逻辑判断并动态拼接sql。

常用元素：

- if：条件性增加SQL语句，一般 作为where 子句的一部分。
- choose (when, otherwise)：不想使用所有的条件，而只是想从多个条件中选择一个使用。
- trim (where, set)：解决因为SQL中where、set或者其他子句的边界问题而导致的语法错误。
- foreach：指定一个集合进行遍历，并且允许指定指定开头与结尾的字符串以及集合项迭代之间的分隔符。
- bind：在 OGNL 表达式以外创建一个变量，并将其绑定到当前的上下文。
- script：通过注解的方式使用动态sql。

## 缓存

![image-20210907204105191](https://raw.githubusercontent.com/cluckl/Pinnned-repo/master/img/20210907204118.png)

### 一级缓存

一级缓存也叫本地缓存

* 与数据库同一次Session期间查询到的数据会放在本地缓存中
* 以后如果需要获取相同的数据，直接从缓存中拿，没必须再去查询数据库

### 二级缓存

二级缓存也叫全局缓存，一级缓存作用域较小，所以诞生了基于 **<u>namespaces（Mapper）</u>**级別的二级缓存。

工作机制：

1. 在一个会话中查询的数据会被放在当前会话的一级缓存中；如果当前会话关闭了，一级缓存中的数据会被保存到二级缓存中
2. 新的会话查询数据时，可以从二级缓存中获取数据
   



