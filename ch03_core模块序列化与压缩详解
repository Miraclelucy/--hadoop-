# 一、为什么要序列化
在面向对象程序设计中，类是个很重要的概念。所谓“类”，可以将它想像成建筑图纸，而对象就是根据图纸盖的大楼。类，规定了对象的一切。根据建筑图纸造房子，盖出来的就是大楼，等同于将类进行实例化，得到的就是对象。
一开始，在源代码里，类的定义是明确的，但对象的行为有些地方是明确的，有些地方是不明确的。对象里不明确地方，是因为对象在运行的时候，需要处理无法预测的事情，诸如用户点了下屏幕，用户点了下按钮，输入点东西，或者需要从网络发送接收数据之类的。后来，引入了泛型的概念之后，类也开始不明确了，如果使用了泛型，直到程序运行的时候，才知道究竟是哪种对象需要处理。
对象可以很复杂，也可以跟时序相关。一般来说，“活的”对象只生存在内存里，关机断电就没有了。一般来说，“活的”对象只能由本地的进程使用，不能被发送到网络上的另外一台计算机。想象一下，对象怎么出现的，一般是new出来的，new出来的对象在内存里面，另外的计算机怎么可能使用我这台机器上的对象呢？如果要的话，序列化就是你必须要使用的东西。
序列化，可以存储“活的”对象，可以将“活的”对象发送到远程计算机。把“活的”对象序列化，就是把“活的”对象转化成一串字节，而“反序列化”，就是从一串字节里解析出“活的”对象。于是，如果想把“活的”对象存储到文件，存储这串字节即可，如果想把“活的”对象发送到远程主机，发送这串字节即可，需要对象的时候，做一下反序列化，就能将对象“复活”了。

通常来说有三个用途：（分布式系统中主要使用持久化和通信的功能）
  - 持久化：对象可以被存储到磁盘上
  - 通信：对象可以通过网络进行传输
  - 拷贝、克隆：可以通过将某一对象序列化到内存的缓冲区，然后通过反序列化生成该对象的一个深拷贝（破解单例模式的一种方法）

# 二、java序列化机制和hadoop序列化机制比较
## 1、Java序列化机制
Java中要实现序列化，只需要实现Serializable即可（说是实现，其实不需要实现任何成员方法）。

举例：
```java
public class MyObject  implements Serializable{
    private static final long serialVersionUID = -5809782578272943999L;
    public String name ;
    private int age;
    public String sex;
    public String getName() {
        return name;
    }
    public void setName(String name) {
        this.name = name;
    }
    public int getAge() {
        return age;
    }
    public void setAge(int age) {
        this.age = age;
    }
    public String getSex() {
        return sex;
    }
    public void setSex(String sex) {
        this.sex = sex;
    }
}
```
测试序列化和反序列化代码：
```java
public class MySerializableTest {
 
    public static void main(String[] args) throws FileNotFoundException, IOException, ClassNotFoundException {
        SerializeObject();
        MyObject object = Deserialize();
        System.out.println("name:" + object.getName() + "\tage:" + object.getAge() + "\tsex:" + object.getSex());
    }
 
    private static MyObject Deserialize() throws FileNotFoundException, IOException, ClassNotFoundException {
        ObjectInputStream oi = new ObjectInputStream(new FileInputStream(new File("/usr/local/hadoop-0.20.2/serialize.txt")));
        MyObject object = (MyObject) oi.readObject();
        System.out.println("反序列化成功");
        return object;
    }
 
    private static void SerializeObject() throws FileNotFoundException, IOException {
 
        MyObject object = new MyObject();
        object.setName("Jackie");
        object.setAge(25);
        object.setSex("male");
        ObjectOutputStream oo = new ObjectOutputStream(new FileOutputStream(new File("/usr/local/hadoop-0.20.2/serialize.txt")));
        oo.writeObject(object);
        System.out.println("序列化成功");
        oo.close();
         
    }
}
```
由此可见，如果想对某个对象进行序列化的操作，只需要在OutputStream对象上创建一个输入流 ObjectOutputStream 对象，然后调用 writeObject（）。在序列化过程中，对象的类、类签名、类的所有非暂态和非静态成员变量的值，以及它所有的父类都会被写入。

进行反序列化的操作，只需要调用 ObjectInputStream 的 readObject() ，并向下转型，就可以得到正确结果。
优点：实现简便，对于循环引用和重复引用的情况也能处理，允许一定程度上的类成员改变。支持加密，验证。
缺点：序列化后的对象占用空间过大，数据膨胀。反序列会不断创建新的对象。同一个类的对象的序列化结果只输出一份元数据（描述类关系），导致了文件不能分割。

## 2、Hadoop序列化机制
hadoop在节点间的内部通讯使用的是RPC，RPC协议把消息翻译成二进制字节流发送到远程节点，远程节点再通过反序列化把二进制流转成原始的信息。RPC的序列化需要实现以下几点：
  - 压缩，可以起到压缩的效果，占用的宽带资源要小。
  - 快速，内部进程为分布式系统构建了高速链路，因此在序列化和反序列化间必须是快速的，不能让传输速度成为瓶颈。
  - 可扩展的，新的服务端为新的客户端增加了一个参数，老客户端照样可以使用。
  - 兼容性好，可以支持多个语言的客户端
为了支持以上特性，hadoop中引用了Writable接口。和说明性Serializable接口不一样，它要求实现两个方法。
```java
package org.apache.hadoop.io;
public interface Writable {
  void write(DataOutput out) throws IOException;
  void readFields(DataInput in) throws IOException;
}
```
hadoop源码本身自带很多test类，	大家可以多多尝试一下。


# 三、典型的Writable类详解


# 四、Writable家族
