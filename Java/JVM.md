---
title: "JVM"
date: 2021-03-19T00:13:24+08:00
draft: false
toc: true
comment: true
images:
tags:
  - Java
---



[TOC]

# 概述

**JVM**(Java Virtual Machine): Java虚拟机，充当运行Java应用程序的==运行时引擎(run-time engine)==，具有平台无关性和语言无关性。

![JVM Model](https://i2.wp.com/techvidvan.com/tutorials/wp-content/uploads/sites/2/2020/06/JVM-Model.jpg?resize=711%2C776&ssl=1)


## JVM的组成部分

* **类加载器(Class Loader)**
* **运行时数据区(Run-Time Data Areas)**
* **执行引擎(Execution Engine)**
  * **解释器(Interpreter)**：在**运行时**逐行解释字节码，然后执行，是Java速度慢的主要原因之一。
  * **即时编译器(Just-In-Time Compiler, JIT)**：为运行比较频繁的代码(Hot Spot Code)编译整个字节码并将其更改为本地代码，因此每当解释器看到重复的方法调用时，JIT都会为该部分提供直接的本地代码，因此不需要重新解释，从而提高了解释器的效率。
  * **垃圾收集器(Garbage Collector, GC)**： 销毁无用的对象。
* **Java本机接口(Java Native Interface, JNI)**：与本机方法库交互的接口，并提供执行所需的本机库。
* **本机方法库(Native Method Library)**：执行引擎所需的本机库（C，C ++）的集合。

## JVM与JDK和JRE的关系

![](https://i0.wp.com/techvidvan.com/tutorials/wp-content/uploads/sites/2/2020/06/Difference-Between-JDK-JRE-JVM.jpg?resize=643%2C395&ssl=1)

* JRE：Java Runtime Environment，包括JVM和Java类库。
* JDK：Java Development Kit，包括JRE和开发工具（如编译器，调试器等）。

# 运行时数据区

**运行时数据区(Run-Time Data Areas)**：定义程序执行期间使用的各种运行时数据区域，是<u>内存分配</u>和<u>垃圾回收</u>的场所。
![image-20210511124055870](https://raw.githubusercontent.com/cluckl/Pinnned-repo/master/img/20210511124055.png)

## PC寄存器

**PC寄存器(PC Registers**)：每个线程都有一个PC寄存器。线程在执行非本地方法时，PC寄存器指向**线程当前执行的字节码指令的地址**（如果执行的是一个本地方法，PC寄存器的值为未定义）。

> PC寄存器是唯一一个不会出现`OutOfMemoryError `的内存区域。

## Java虚拟机栈

**Java虚拟机栈(Java Virtual Machine Stacks)**：每个线程都拥有一个私有的虚拟机栈，用来表示**方法的运行过程**。

栈的元素称为**栈帧(stack frame)**，每个栈帧内容：

* **局部变量数组(Local Variable Array )**：保存方法的所有参数和局部变量，通过索引访问数据。
* **操作数栈(Operand Stack )**:存储中间计算的结果，通过栈的push和pop访问数据。
* **帧数据(Frame Data)**: 包含当前类的常量池中数据的引用和返回值等消息。

![Java-Stack](https://media.geeksforgeeks.org/wp-content/uploads/JVM.jpg)

Java虚拟机在Java虚拟机栈上仅执行Push 和 Pop两个操作：

* 当线程调用一个方法时，JVM会创建一个新栈帧并将其push进 Java栈中；
* 方法返回后（正常return或抛出异常），JVM会pop出这个栈帧。

## 本地方法栈

**本地方法栈(Native Method Stacks)**：存储本地方法的信息，每个线程都有一个本地方法栈，栈帧的变化过程和[虚拟机栈](JVM内存空间.md#JVM内存空间结构#虚拟机栈)类似，也会抛出 	`StackOverFlowError` 和`OutOfMemoryError `异常。

>由于很多 Native 方法都是用 C 语言实现的，所以它通常又叫 C 栈。

## 堆

**堆(Heap Area)**：Java虚拟机有一个**所有线程共享**的堆，用于分配大部分<u>类实例</u>和<u>数组</u>的内存。

* 在虚拟机启动时创建，在虚拟机退出时销毁。
* 垃圾回收的主要场所。
* 如果计算需要的堆多于自动存储管理系统可以提供的堆，则Java虚拟机将抛出一个 `OutOfMemoryError`。

> 在JVM开启逃逸分析的情况下，JIT分析出对象没有逃逸出方法区，则可能把该对象进行标量替换(将该对象替换出多个成员变量)分配在栈上。

## 方法区

**方法区(Method Area)**：Java虚拟机有一个所有**线程共享**的区域，用来存储每个类的结构，包括<u>**运行时常量池**</u>、字段、方法等信息。该方法区域类似于常规语言的编译代码的存储区域，或者<u>类似于操作系统过程中的“文本”段</u>。

* 逻辑上属于堆的一部分，但可以选择不进行垃圾回收
* 如果无法提供方法区域中的内存来满足分配请求，则Java虚拟机将抛出一个`OutOfMemoryError`。

### 运行时常量池

**运行时常量池(Run-Time Constant Pool)**: 方法区的一部分，表示 `.class` 文件中的常量池数据，包含多种常量，范围从<u>编译时</u>已知的数字文字<u>运行时</u>解析的方法和字段引用。功能类似于常规编程语言的符号表。

* 可以在运行时动态增加内容。

* 当Java虚拟机创建类或接口时，将为该类或接口构造运行时常量池。
* 如果运行时常量池的构造所需的内存超过Java虚拟机的方法区域中可用的内存，则Java虚拟机将抛出`OutOfMemoryError`。

### 永久代和元空间

永久代和元空间是HotSpot的方法区不同版本的<u>实现方式</u>。

* 永久代(PermGen)：在JDK1.7及其之前的版本使用，容易抛出`java.lang.OutOfMemoryError: PremGen space`

* 元空间(Metaspace)：在JDK1.8后的版本使用，使用**本地内存**，==不容易出现内存泄漏==。

## 直接内存

直接内存(Direct Memory)：<u>不属于运行时数据区的一部分</u>，也不是Java虚拟机规范中定义的内存区域，但被频繁使用。

* 可能导致`OutOfMemeryError`。
* 不受Java堆大小的限制，但是会受系统内存的限制。

> NIO(New Input/Output) 类直接使用 Native 函数库直接分配堆外内存，然后使用一个存储在 Java 堆中的 `DirectByteBuffer` 对象引用这块内存。
>
> 这样就能在一些场景中显著提高性能，因为避免了在 Java 堆和 Native 堆之间来回复制数据。

## 字符串常量池

字符串常量池(string constant pool):  将字符串存储在该池子中，可以避免具有相同字面量的字符串被创建，节省内存空间。

* 内容：在JDK6中存放**<u>字符串对象</u>**；JDK7开始存放的是字符串对象的<u>**引用**</u>。

* 位置：在JDK6中，作为运行时常量池的一部分存放在**<u>方法区</u>**；在JDK7中，存放在**<u>堆</u>**中；JDK8开始，存放在<u>**本地内存**</u>中。

  ![](https://raw.githubusercontent.com/cluckl/Pinnned-repo/master/img/20210511122731.png)

# 类加载机制

类从被加载到JVM内存开始，到卸载出内存为止，它的整个生命周期包括以下 7 个阶段：

![image-20210513194225397](https://raw.githubusercontent.com/cluckl/Pinnned-repo/master/img/20210513194225.png)

类加载过程主要包括3个阶段：

- **Loading(加载)**
- **Linking(链接)**
- **Initialization(初始化)**

## 加载

1. 通过全类名读取对应的*.class*文件，生成该类的**<u>二进制字节流</u>**
2. 将字节流所代表的静态结构转化为**<u>方法区</u>**的运行时数据结构
3. 在**堆**中创建一个代表该类的 `java.lang.Class` 对象，通过该对象可以获取该类的名称、父类名称、方法和变量等信息

注意：<u>数组类本身不通过类加载器创建，而是由 **Java 虚拟机**直接创建，再由类加载器创建数组中的元素类。</u>

## 链接

Linking(链接)可以分成三个阶段：

* **验证(Verification)**：确保加载的类符合 JVM 规范和安全，包括文件格式验证、元数据验证、字节码验证和符号引用验证。如果验证失败，将抛出运行时异常`java.lang.VerifyError`。

* **准备(Preparation)**: 在方法区中为<u>**类变量**</u>分配内存并设置为对应数据类型的默认值。

* **解析(Resolution)**: 将常量池内的<u>**符号引用**</u>替换为<u>**直接引用**</u>。

  * **符号引用(symbolic references)**：描述目标的符号，可以是任何字面量
  * **直接引用(direct references)**：直接指向目标的指针、相对偏移量或一个间接定位到目标的句柄
  
  > 为了支持Java的运行时绑定，解析阶段也可能是在初始化阶段后才开始。

## 初始化

初始化(Initialization)：调用类构造器 `<clinit>()` 方法，为所有的**类变量**赋值。

## 类加载器类型

JVM主要预定义了三种类型的类加载器：

* **启动类加载器（Bootstrap ClassLoader）**：负责加载存放在启动路径(`<JAVA_HOME>\jre\lib`)  中的核心 java API 类。由本地语言(如C/C ++等)实现，因此不需要被加载。

  ![](https://raw.githubusercontent.com/cluckl/Pinnned-repo/master/img/20210510222753.png)

* **扩展类加载器（Extension ClassLoader）**： 启动类加载器的子类，负责加载在**扩展路径**(`<JAVA_HOME>\jre\lib\ext `)或`java.ext.dirs`系统变量指定的类，开发者可以直接使用扩展类加载器。

  ![](https://raw.githubusercontent.com/cluckl/Pinnned-repo/master/img/20210510222829.png)

* **系统/应用程序类加载器（System/Application ClassLoader）**：扩展类加载器的子类，负责加载**系统类路径（`CLASSPATH`）**上所指定的类，开发者可以直接使用这个类加载器。

  ![](https://raw.githubusercontent.com/cluckl/Pinnned-repo/master/img/20210510225417.png)
  

除了上述三种类加载器，用户可以用Java编写**用户自定义类加载器(CustomClassLoader)**，加载指定路径上的类。用户自定义类加载器必须直接或间接继承`java.lang.ClassLoader`。

可以使用类对象的`getClassLoader()`获取加载该类的加载器。

## 双亲委派机制

**Delegation Hierarchy Algorithm**(双亲委派机制)：当某个类加载器需要加载某个`.class`文件时，它首先把这个任务**委托(delegate)**给它的上级类加载器，递归这个操作，如果上级的类加载器没有加载，自己才会去加载这个类，如果找不到可以加载的类加载器，则抛出运行时异常`java.lang.ClassNotFoundException`。

![](https://raw.githubusercontent.com/cluckl/Pinnned-repo/master/img/20210510231132.png)

作用：

* 防止内存中出现**多份同样的字节码**，即避免相同类的重复加载
* 保证Java核心API的**安全**。

> 如何改写并加载一个系统中的类？
> 自己定义一个类加载器并将其放在一个特殊的目录下，防止被系统的加载器加载。
> 注意：这个类虽然和系统中的某个类全称相同，但不是同一个类：任意一个类，都由加载它的类加载器和这个类本身一同确立其在 Java 虚拟机中的唯一性。

# 垃圾回收

GC(Garbage Collector)主要负责**堆**和方法区的分配和回收。可以主动使用GC进行垃圾回收(JVM不一定会执行)：

* ` System.gc();`
* `Runtime.getRuntime().gc();`

## 引用类型

JDK 1.2之后，存在四种类型的引用：

* 强引用：new出来的对象，即使抛出`OutOfMemoryError`也不会被回收。

  ```java
  Object obj = new Object();
  ```

* 软引用：使用`SoftReference` 类实现软引用。只有JVM 认为内存不足时，才会去回收软引用指向的对象。

  ```java
  Object obj = new Object();
  SoftReference<Object> sf = new SoftReference<Object>(obj);
  obj = null;  
  ```

* 弱引用：使用`WeakReference` 类实现弱引用。下一次GC时会被回收。

  ```java
  Object obj = new Object();
  WeakReference<Object> wf = new WeakReference<Object>(obj);
  obj = null;
  ```

* 虚引用：使用 `PhantomReference` 类实现虚引用，**主要用于跟踪对象被垃圾回收的活动**。

  ```java
  Object obj = new Object();
  PhantomReference<Object> pf = new PhantomReference<Object>(obj, null);
  obj = null;
  ```

## 回收条件

### 引用计数法

为对象添加一个引用计数器：

* 当对象增加一个引用时计数器加 1
* 引用失效时计数器减 1

引用计数为 0 的对象可被回收。

<u>弊端：很难解决对象之间循环引用的问题</u>。

### 可达性分析法
以 GC Roots 为起始点进行搜索，可达的对象都是存活的，不可达的对象可被回收。
GC Roots 是指：

* **Java 虚拟机栈**或**本地方法栈**引用的对象
* 方法区中**常量**或**类变量**引用的对象

由于**GC Roots 不包含堆中对象引用的对象**，可以解决循环引用的问题。

### 对象重生

在对象被GC回收之前，**如果GC没有调用过该对象的`finalize()`方法**，则会执行该方法。如果在执行该对象的`finalize()`方法的过程中，该对象会被重新引用，不会被GC回收，即重生。

### 方法区的回收

方法区的回收主要针对<u>**不被引用的常量**</u>和**<u>无用的类</u>**，并且即使满足条件也不一定会被回收。

* 无用的类
  * 该<u>**类的所有实例**</u>被回收
  * 加载该类的<u>**ClassLoder**</u>被回收
  * 对应的<u>`java.lang.Class`</u>对象没有被引用

## 回收策略

### 标记清除算法

1. **标记**：检查每个对象，标记不需要被回收的对象。
2. **清除**：将没有被标记的对象全部清除同时清除标记。

* 效率不高，且会产生大量不连续内存碎片
* 妨碍较大对象的分配，甚至提前触发垃圾回收动作

### 标记整理算法

1. **标记**：和[标记清除算法](JVM.md#垃圾回收#标记清除算法)相同。
2. **整理**：将所有存活的对象都向一端移动，然后直接清理掉端边界以外的内存。

- 不会产生内存碎片
- 需要移动大量对象，处理效率比较低

### 复制算法

将可用内存划分为容量相等的两块轮流使用。一块需要垃圾回收时，就将存活对象复制到另一块并清除该块的所有内存。
特点：复制算法不会产生内存碎片但是会**浪费内存空间**。

### 分代收集算法

对象存活周期将内存划分为几块，不同块采用适当的收集算法。

* 新生代：复制算法
* 老年代：标记-清除算法、标记-整理算法

## HotSpot垃圾收集器

![img](https://cs-notes-1256109796.cos.ap-guangzhou.myqcloud.com/c625baa0-dde6-449e-93df-c3a67f2f430f.jpg)

### Serial 收集器

Serial：只开启一条 GC 线程进行垃圾回收，并且在垃圾收集过程中停止一切用户线程(Stop The World)。

* 适合客户端使用：一般客户端应用所需内存较小，不会创建太多对象，而且堆内存不大，因此垃圾收集器回收时间短，即使在这段时间停止一切用户线程，也不会感觉明显卡顿。
* 简单高效：只使用一条 GC 线程，<u>避免线程切换的开销</u>。

### **ParNew 收集器** 

**ParNew** ： Serial 的多线程版本。由多条 GC 线程并行地进行垃圾清理。但清理过程依然需要Stop The World。

* **追求“低停顿时间”**, 与 Serial 唯一区别是**使用多线程进行垃圾收集**。
* 在多 CPU 环境下性能比 Serial 会有一定程度的提升；但线程切换需要额外的开销，因此在单 CPU 环境中表现不如 Serial。

###  Parallel Scavenge 收集器

**Parallel Scavenge**：**追求 CPU 吞吐量**，能够在较短时间内完成指定任务，因此适合没有交互的后台计算。使用复制算法。

> 缩短停顿时间是以牺牲吞吐量和新生代空间来换取的：新生代空间变小，垃圾回收变得频繁，导致吞吐量下降。

### Serial Old 收集器

**Serial Old**: Serial的老年代版本，和Serial的区别：

* Serial Old 工作在老年代，使用**标记整理**算法；Serial 工作在新生代，使用**复制**算法。

### Parallel Old 收集器

**Parallel Old**： Parallel Scavenge 的老年代版本，使用标记-整理算法。

### CMS 收集器

**CMS(Concurrent Mark Sweep，并发标记清除)**：以**最短回收停顿时间**为目标的收集器（追求低停顿），能够在垃圾收集时的大部分时间内让用户线程和 GC 线程并发执行，因此在垃圾收集过程中用户也不会感到明显的卡顿，使用标记-清除算法。

#### 工作过程

![img](https://cs-notes-1256109796.cos.ap-guangzhou.myqcloud.com/62e77997-6957-4b68-8d12-bfd609bb2c68.jpg)

1. **初始标记**：Stop The World，使用一条初始标记线程对所有<u>与 GC Roots 直接关联的对象</u>进行标记。
2. **并发标记**：使用标记线程与用户线程并发执行。此过程进行**可达性分析**，标记出所有废弃对象。速度很慢。
3. **重新标记**：Stop The World，使用多条标记线程并发执行，将在并发标记过程被标记的对象中又被重新引用的对象重新标记(remark)，**<u>不能处理浮动垃圾</u>**（用户线程在和垃圾收集器并发执行过程中产生的垃圾）。
4. **并发清除**：使用一条 GC 线程清除刚才标记的对象，与用户线程并发执行。

<u>并发标记与并发清除过程耗时最长，且可以与用户线程一起工作，因此，总体上说，CMS 收集器的内存回收过程是与用户线程一起并发执行的。</u>

#### CMS 的缺点

* 无法处理浮动垃圾，导致频繁 Full GC
* 使用“标记-清除”算法产生碎片空间
* 吞吐量低

### G1 收集器

**G1(Garbage First)**：将堆划分为一块块独立的 Region，不区分新生代和老年代。当要进行垃圾收集时，首先估计每个 Region 中垃圾的数量，每次都从<u>**垃圾回收价值**</u>（由过去的垃圾回收时间以及回收所获得的空间决定）最大的 Region 开始回收，因此可以获得最大的回收效率。

* 回收策略：从整体上看， G1 是基于“**<u>标记-整理</u>**”算法实现的收集器，从局部（两个 Region 之间）上看是基于“**<u>复制</u>**”算法实现的，这意味着运行期间不会产生内存空间碎片。
* Remembered Set：每个 Region 都有一个 Remembered Set，用来记录该 Region 对象的引用对象所在的 Region，可以在执行可达性分析的时候**<u>避免全堆扫描</u>**。
* 可预测的停顿：能让使用者明确指定在一个长度为 M 毫秒的时间片段内，消耗在 GC 上的时间不得超过 N 毫秒。

![Types of Garbage Collector in Java](https://static.javatpoint.com/core/images/types-of-garbage-collector-in-java5.png)

#### 工作过程

如果不计算维护 Remembered Set 的操作，G1 收集器的运作可划分为以下几个步骤：

1. **初始标记**：Stop The World，使用一条初始标记线程对所有与 GC Roots 直接关联的对象进行标记。

2. **并发标记**：使用一条标记线程与用户线程并发执行。此过程进行可达性分析，速度很慢。

3. **最终标记**：Stop The World，使用多条标记线程并发执行。

4. **筛选回收**： Stop The World，使用多条筛选回收线程并发执行，回收废弃对象。

   	> 此阶段其实也可以做到与用户程序一起并发执行，但是因为只回收一部分 Region，时间是用户可控制的，而且停顿用户线程将大幅度提高收集效率。

# 内存分配

![img](https://miro.medium.com/max/875/0*NzhXk29lJrjsTJ5p)

## Minor GC 和 Full GC

- Minor/Young GC：回收新生代，因为新生代对象存活时间很短。因此 Minor GC **<u>执行频率高，执行速度较快</u>**。
- Full/Major GC：回收老年代和新生代，老年代对象其存活时间长。因此 Full GC **<u>执行频率低，执行速度较慢</u>**。

## 内存分配规则

堆的内存分配规则不固定，但有几条最普遍的规则。

* **对象优先在Eden区分配**：大多数情况下，对象在新生代 Eden 区分配，当 Eden 区空间不够时，发起 Minor GC。
* **大对象直接进入老年代**：大对象是需要大量连续内存空间的 Java 对象，如很长的字符串或数据。一个大对象能够存入 Eden 区的概率比较小，发生分配担保的概率比较大，而分配担保需要涉及大量的复制，就会造成效率低下。
* **长期存活的对象将进入老年代**：每个对象存在一个年龄计数器，对象在 Eden 区出生如果经过 Minor GC 依然存活，则进入将Survivor区，对象每经过一次Minor GC 年龄就加1，当年龄达到<u>**年龄阈值**</u>时将被移动到老年代中。
* **动态对象年龄判定**：如果在 Survivor 区中<u>**具有相同年龄的所有对象的空间总和**</u>大于 Survivor 区空间的一半，则<u>**大于或等于该年龄**</u>的对象直接进入老年代（若该年龄小于年龄阈值则更新年龄阈值为该年龄的值）。

## 空间分配担保

* 担保要求：在发生 Minor GC 之前，JVM会先检查老年代**最大可用的<u>连续</u>空间**是否大于**新生代所有对象的空间总和**，如果这个条件成立，那么 Minor GC 可以确保是安全的。
  只要老年代的<u>**连续空间**</u>大于<u>**新生代对象总大小**</u>或者**<u>历次晋升的平均大小</u>**，就会进行 [Minor GC](JVM垃圾回收.md#Minor%20GC%20和%20Full%20GC)，否则将进行 [Full GC](JVM垃圾回收#Minor%20GC%20和%20Full%20GC)。
* 担保过程：当新生代内存不足时，将新生代的存活对象移到老年代，而新生代释放出来的空间给最新的分配给对象。

## 内存分配方式

为对象从Java堆中分配出一块大小确定的内存空间时，分配方式主要有两种：

* **指针碰撞**
  * 将被使用的内存全部整合到一边，空闲的内存放在另一边，用一个分界值指针隔开。
  * 当需要分配内存时，根据对象的内存大小沿着空闲内存的方向移动该指针
  * 适用于没有内存碎片的场合。
* **空闲列表**
  * 维护一个列表，记录内存块是否可用。
  * 当需要分配内存时，从该列表中取出一块足够大的内存块存储该对象实例，并更新列表信息。
  * 适用于存在内存碎片的场合。

## 内存分配并发问题

在多线程编程中，内存分配会产生并发问题。可以使用TLAB解决：

**TLAB(Thread Local Allocation Buffer)**:  在 Eden 区中，每个线程有一块预先分配的内存。在对象分配内存时，优先在 TLAB 分配，当对象大于 TLAB 中的剩余内存时，更新TLAB后重新分配或者直接在Eden区分配对象。

# 对象的创建和使用

## 对象的创建过程

1. **类加载检查**：检查是否能根据指令的参数在常量池中定位到这个类的符号引用，并且检查这个符号引用代表的类是否执行了相应的类加载过程，如果没有则执行。

2. **内存分配**：为对象分配内存，对象在内存中的布局包括<u>对象头（Header）</u>、<u>实例数据（Instance Data）</u>和<u>对齐填充（Padding）</u>。

3. **初始化零值**：将对象的**实例数据**部分初始化为零值，保证对象的**实例数据**在不赋初始值的情况下可以直接使用。

4. **设置对象头**：对象头由三部分组成：

   1. **Mark Word（标记字段）**：存储对象自身的运行时数据， 如HashCode、`GC`分代年龄、锁状态标志、线程持有的锁、偏向线程ID、偏向时间戳等等。

   2. **Klass Word（类型指针）**：指向对象的类元数据，通过这个指针确定该对象是哪个类的实例。

   3. **数组长度**：表示数组的长度，数组对象特有。

5. **执行`<init>` 方法**：执行相应构造方法的字节码。

## 对象的访问方式

两种主流的访问方式：

* **直接指针**：reference中直接存储对象的地址，该地址包含<u>**对象实例数据**</u>和<u>**对象类型数据的指针**</u>。

  ![image-20211002133819204](https://raw.githubusercontent.com/cluckl/Pinnned-repo/master/img/20211002133820.png)

* **使用句柄**：在Java堆中划分出一块内存来作为句柄池，reference中存储对象的句柄地址，而句柄中包含对象实例数据的指针**</u>和**<u>对象类型数据的指针</u>**。

  ![image-20211002133802269](https://raw.githubusercontent.com/cluckl/Pinnned-repo/master/img/20211002133804.png)

  

# 参考

* [How JVM Works - JVM Architecture? - GeeksforGeeks](https://www.google.com/url?sa=i&url=https%3A%2F%2Fwww.geeksforgeeks.org%2Fjvm-works-jvm-architecture%2F&psig=AOvVaw24Qqny3ShzBHWD3N3aaiV2&ust=1620710116183000&source=images&cd=vfe&ved=0CA0QjhxqFwoTCPD1wumtvvACFQAAAAAdAAAAABBD)
* [JVM - Java Virtual Machine Working and Architecture - TechVidvan](https://www.google.com/url?sa=i&url=https%3A%2F%2Ftechvidvan.com%2Ftutorials%2Fjava-virtual-machine%2F&psig=AOvVaw24Qqny3ShzBHWD3N3aaiV2&ust=1620710116183000&source=images&cd=vfe&ved=0CA0QjhxqFwoTCPD1wumtvvACFQAAAAAdAAAAABBL)
* [doocs.github.io/jvm](https://doocs.github.io/jvm)
* [关于Java类加载双亲委派机制的思考（附一道面试题） - Alexia ](https://www.cnblogs.com/lanxuezaipiao/p/4138511.html)
* [Custom ClassLoader in Java Example | JVM (Part - 4) | Codez Up](https://codezup.com/custom-classloader-in-java-example-jvm-architecture/)
* [Chapter 2. The Structure of the Java Virtual Machine](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-2.html)
* [Java Virtual Machine (JVM) Stack Area - GeeksforGeek](https://www.geeksforgeeks.org/java-virtual-machine-jvm-stack-area/)
* [JVM | What is Java Virtual Machine & its Architecture](https://www.guru99.com/java-virtual-machine-jvm.html)
* [Java方法区、永久代、元空间、常量池详解](https://blog.csdn.net/u011635492/article/details/81046174)
* [What is string constant pool in Java? - tutorialspoint](https://www.tutorialspoint.com/What-is-string-constant-pool-in-Java)
* [JVM运行时数据区- 华为云](https://www.huaweicloud.com/articles/92a2b8455ed00ac4579768a432630ddb.html)
* [CS-Note -Java 虚拟机](http://www.cyc2018.xyz/Java/Java%20%E8%99%9A%E6%8B%9F%E6%9C%BA.html)
* [java基础：String — 字符串常量池与intern(二） - 掘金](https://juejin.cn/post/6844903741032759310)
* [求你了，别再说Java对象都是在堆内存上分配空间的了](https://www.cnblogs.com/hollischuang/p/12501950.html)
* [TLAB？深入了解JVM是如何用它优化内存分配的 - 开发者头条](https://toutiao.io/posts/0ebr4w4/preview)