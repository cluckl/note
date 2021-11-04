# Maven

## 简介

Maven 是一个项目管理工具，它包含了一个**<u>项目对象模型（Project Object Model）</u>**，在配置中以 *pom.xml* 文件的形式存在。

Maven有两大核心：

- **<u>依赖管理</u>**：对 jar包的统一管理
- **<u>项目构建</u>**：对项目进行编译、测试、打包、部署、上传到服务器

## 仓库

### 仓库类型

| 仓库类型 | 说明                                                         |
| :------- | :----------------------------------------------------------- |
| 本地仓库 | 默认位置在 `..\.m2\repository`                               |
| 私服仓库 | 局域网中的服务器，访问速度较快，存放的 jar包一般是公司内部自己开发的 jar包 |
| 中央仓库 | maven官方仓库，包含了大部分的 jar                            |

### 查找路径

![img](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9tbWJpei5xcGljLmNuL21tYml6X3BuZy9HdnRER0tLNHVZbTB2aWFoZWxEaWJPUXp0a0Qwd2FpYzd4elFzcG5jcldqRDhGOVRPbjdtaWN4VFoxRUh1clRzWHluTGtBZFZhWE16NzlTcWtEZkpZWDIxakEvNjQw?x-oss-process=image/format,png)

## Maven坐标

Maven的坐标用于在Maven仓库中唯一确定一个Maven工程，使用如下三个向量确定：

- **<u>groupId</u>**：组织的名称
- **<u>artifactId</u>**：项目的模块名称
- **<u>version</u>**：模块的版本

## 依赖管理

Maven通过`<dependencies>`/`<dependency>`标签实现依赖管理

### 作用范围

在每个`<dependency>`中，通过`<scope>`标签确定当前jar包的作用范围。

![img](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9tbWJpei5xcGljLmNuL21tYml6X3BuZy9HdnRER0tLNHVZbTB2aWFoZWxEaWJPUXp0a0Qwd2FpYzd4ejF3UzAxeUk2UUd6eEZHSFpvUzZ3VW1nbk1VTndpYlpBbXlTMkJDdEprT2JMaWFCRWliSlFWaWNpYnBBLzY0MA?x-oss-process=image/format,png)

### 依赖原则

1. **<u>依赖路径最短优先原则</u>**：依赖路径短的优先于依赖路径长的。
2. **<u>声明顺序优先原则</u>**：路径相同的情况下先声明的依赖优先于后声明的依赖。
3. **<u>覆写优先原则</u>**：子 POM 内声明的依赖优先于父 POM 中声明的依赖。

### 依赖排除

1. 查找依赖冲突的jar：使用 `mvn dependency:tree` 查看依赖树或者IDEA的插件等方式查找依赖冲突的jar包；
2. 通过`<extensions>`标签排除间接引入的jar包。

## 常用命令

| 常用命令    | 中文含义 | 说明                             |
| :---------- | :------- | :------------------------------- |
| mvn clean   | 清理     | 清理已经编译完成的文件           |
| mvn compile | 编译     | 将 Java 代码编译成 *.class* 文件 |
| mvn test    | 测试     | 项目测试                         |
| mvn package | 打包     | 将项目打成 jar 包或者 war 包     |
| mvn install | 安装     | 向本地仓库安装一个 jar包         |
| mvn deploy  | 上传     | 将 jar包上传到服务器             |

## 生命周期

Maven 有三个**<u>相互独立</u>**的生命周期：

- **Clean Lifecycle**：执行清理工作
- **Default Lifecycle** 构建项目的核心部分，包括编译，测试，打包，安装，部署等等
- **Site Lifecycle**：生成并发布项目的站点文档

**<u>在一个周期中运行任何一个阶段的时候，该周期前面的所有阶段都会被运行</u>**

### Clean 生命周期

1. pre-clean：执行一些需要在 clean 之前完成的工作
2. **<u>clean</u>**：移除所有生成的文件
3. post-clean：执行一些需要在 clean 之后立刻完成的工作

### Site 生命周期

1. pre-site：执行一些需要在生成站点文档之前完成的工作
2. **<u>site</u>**：生成项目的站点文档
3. post-site：执行一些需要在生成站点文档之后完成的工作，并且为部署做准备
4. **<u>site-deploy</u>**：将生成的站点文档**<u>部署</u>**到特定的服务器上

### Default生命周期

1. validate：验证工程是否正确
2. **<u>compile</u>**：编译项目的源代码
3. **<u>test</u>**：测试已编译的源代码
4. **<u>package</u>**：把已编译的代码打包成可发布的格式（比如 jar）
5. integration-test：如有需要，将包处理和发布到一个能够进行集成测试的环境
6. verify：运行所有检查，验证包是否有效且达到质量标准
7. **<u>install</u>**：将包安装到maven本地仓库，可以被其他工程作为依赖来使用
8. deploy： 将最终包复制到远程仓库，供其他开发人员和maven项目使用

## 聚合

在多模块项目中，将多个模块聚合在一起，可以批量进行maven的相关操作。

使用方式：在总的`pom.xml`文件中使用 `<modules>`/`<module>` 标签组合。

## 参考

* [CS-Note Maven](http://www.cyc2018.xyz/%E5%85%B6%E5%AE%83/%E7%BC%96%E7%A0%81%E5%AE%9E%E8%B7%B5/%E6%9E%84%E5%BB%BA%E5%B7%A5%E5%85%B7.html#%E4%B8%80%E3%80%81%E6%9E%84%E5%BB%BA%E5%B7%A5%E5%85%B7%E7%9A%84%E4%BD%9C%E7%94%A8)
* [学Maven，这篇万余字的教程，真的够用了！ - 江南一点雨](https://www.cnblogs.com/lenve/p/12047793.html)

* [面试官问我maven package和install的区别](https://segmentfault.com/a/1190000021609439)