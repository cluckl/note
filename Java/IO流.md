## I/0分类

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191014111930276.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NTQzNTA4,size_16,color_FFFFFF,t_70)

* 按照流的流向分，可以分为输入流和输出流
* 按照操作单元划分，可以划分为字节流（继承 InputStream和OutputStream）和字符流（继承 InputSteamReader和OutputStreamWriter）
* 按照流的角色划分为节点流和处理流。

### 字符流和字节流的区别

* 读取过程：字节流读取的时候，读到一个字节就返回一个字节；字符流基于字节流读到一个或多个字节时，先**<u>查找指定的编码表</u>**，再将查到的字符返回。
* 适用对象：字节流可以处理包括图片、音频、视频文件在内的**<u>所有类型数据</u>**；字符流只能处理**<u>纯文本</u>**数据。

### 节点流和处理流的区别

- 节点流：作为装饰者模式中的<u>**被装饰对象**</u>，可以从某节点读数据或向某节点写数据的流。
- 处理流：作为装饰者模式中的<u>**装饰器**</u>，连接和封装已存在的流，实现更为丰富的流数据处理，处理流的构造方法必须传入其他的流对象参数。

## 编码和解码

编码：将字符转换为字节。

解码：将字节重新组合成字符。

### 编码

**InputStreamReader**类：创建`InputStreamReader`实例装饰`FileInputStream`实例：

```java
InputStreamReader inputStreamReader = new InputStreamReader(new FileInputStream(fileName), "编码格式");
```

### 解码

**OutputStreamWriter**类：创建`OutputStreamWriter`实例装饰`FileOutputStream`实例：

```java
OutputStreamWriter outputStreamWriter = new OutputStreamWriter(new FileOutputStream(fileName) , "编码格式");
```



## 序列化

序列化：将一个**对象**转换成**字节序列**，方便存储和传输。

<u>序列化不会转化静态变量的内容，因为序列化只是保存对象的状态，静态变量属于类的状态。</u>

### 序列化步骤

<u>前置条件：对序列化的类实现 `Serializable` 接口。</u>

> 它只是一个标准，没有任何方法需要实现，但如果不去实现该接口在序列化时会抛出异常。

* 序列化步骤：用`ObjectOutputStream`装饰`FileOutputStream`实例，然后调用其`writeObject(String)`方法
* 反序列化步骤：用`ObjectInputStream`装饰`FileInputStream`实例，然后调用其`readObject()`方法并将返回对象强制转换成对应的类对象。

> `ObjectOutputStream`和`ObjectInputStream`把对象写入数据源或者从数据源读出来

```java
public static void main(String[] args) throws IOException, ClassNotFoundException {

    A a = new A();
    String objectFile = "dir/file_name";

    ObjectOutputStream objectOutputStream = new ObjectOutputStream(new FileOutputStream(objectFile));
    objectOutputStream.writeObject(a);
    objectOutputStream.close();

    ObjectInputStream objectInputStream = new ObjectInputStream(new FileInputStream(objectFile));
    A a2 = (A) objectInputStream.readObject();
    objectInputStream.close();
    System.out.println(a2);
}

private static class A implements Serializable {
    //...
}
```

### transient

被`transient` 关键字修饰的属性不会被序列化。

## NIO

### channel

Channel（通道） ：对原 I/O 包中的流的模拟，可以通过它**<u>同时读取和写入数据</u>**。

类型：

- FileChannel：从文件中读写数据；
- DatagramChannel：通过 UDP 读写网络中数据；
- SocketChannel：通过 TCP 读写网络中数据；
- ServerSocketChannel：可以监听新进来的 TCP 连接，对每一个新进来的连接都会创建一个 `SocketChannel`。

### buffer

buffer：用于存放数据，本质是一个<u>**数组**</u>。

* 介于通道和数据接收方/提供方之间
* 提供对数据的**结构化访问**
* 可以**跟踪系统的读/写进程**

### selector

 selector（选择器）：selector 通过**轮询**的方式监听<u>多个通道 Channel 上的事件</u>，从而让一个使用它的线程处理多个事件，减少创建和切换线程的开销，适用于IO 密集型的应用。

前提条件：需要将监听的通道 channel  配置为非阻塞：

```java
ServerSocketChannel ssChannel = ServerSocketChannel.open();
ssChannel.configureBlocking(false);
```

>  只有套接字 Channel 才能配置为非阻塞，而 FileChannel 不能，同时为 FileChannel 配置为非阻塞也没有意义。

![img](https://cs-notes-1256109796.cos.ap-guangzhou.myqcloud.com/093f9e57-429c-413a-83ee-c689ba596cef.png)

### NIO与IO区别

- NIO 是非阻塞的
- NIO 面向块，I/O 面向流

## 参考

* [Java I/O CS -Notes](http://www.cyc2018.xyz/Java/Java%20IO.html#)