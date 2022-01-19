java 



# Java基础

### jvm中的内存区域

#### jvm中有五大内存区域，分别是程序计数器，java栈，本地方法栈，堆，方法区

1.程序计数器是线程私有的，如果是底层方法，那么计数器为空

2.java栈，也是虚拟机栈，描述的是方法执行的内存模型，每个方法被执行的时候都会创建一个栈帧用于存储局部变量表，操作栈，动态链接，方法出口等信息，每一个方法被调用的过程就是对应的一个栈帧在虚拟机中从入栈到出栈的过程。平常说的栈是存储局部变量表的部分，局部变量表所需要的内存空间在编译器内完成分配，当进入一个方法时，这个方法在栈中需要分配多大的局部变量空间是完全确定的，在方法运行期间不会改变局部变量表大小，

Java虚拟机栈中可能出现两种类型的异常：

1）线程请求的栈深度大于虚拟机允许的栈深度，将抛出StackOverflowError

2)虚拟机占空间可以动态扩展，当动态扩展时无法申请到足够的空间时，抛出OutOfMemory异常

3.本地方法栈，

与虚拟机栈发挥的作用相似，区别时虚拟机栈执行的是Java方法，，也就是字节码，而本地方法栈使用到的native方法服务，可能底层调用的是c或者c++，jdk安装目录可以看到很多用c编写的文件，可能就是native方法调用的c代码

4.堆

堆是Java虚拟机管理内存最大的一块内存区域，因为堆存放的对象是线程共享的，所以多线程的时候也需要同步机制

所有对象实例及数组都要在对上分配内存，

堆是所有线程共享的，他的目的是存放对象实例，同时它也是GC所管理的主要区域，因此常被称为GC堆，又因为现在收集器常用分代算法，Java堆中还可以细分为新生代和老生代，

根据虚拟机规范，Java堆可以存在物理上不连续的内存空间，就像磁盘空间只要是连续的即可，他的内存大小可以设定为固定大小，也可以扩展，当前主流的虚拟机如HotPot都能扩展实现（通过设置-Xmx和-Xms），如果堆中没有内存完成实例的分配，而且对无法扩展将报OOM错误（OutOfMemaryError）

5.方法区，

方法区同堆一样，是所有线程共享的区域，方法区用于存储已被虚拟机加载的类信息，常量，静态变量等。



### **垃圾回收与算法**

1.触发条件:程序调用System.gc时可以触发，系统自身来决定GC 触发的时机

#### 2.如何回收：

1）标记-清除算法，

标记：垃圾收集器会将所有堆中的对象都扫描一遍，将需要回收的对象所在的内存和不需要回收对象所在的内存作标记，以此确定回收的对象，

清楚：垃圾收集器会清除掉上一步标记出来的那些需要回收的对象区域

问题：标记清楚后会产生大量的不连续内存碎片，碎片太多会导致以后在程序运行过程中需要分配较大的对象时，无法找到连续的足够内存而不得不提前触发另一次垃圾收集动作



2）复制算法

复制算法将内存按容量大小划分为相等的两块，每次只是用其中的一块，当这一块使用完了，就将还存活的对象复制到另一块上面，然后再把已使用过的内存空间一次清理掉，这样使得每次都是对半区域的内存回收，内存分配时就不用考虑到内存碎片等复杂情况，只要移动堆顶指针，按顺序分配内存即可，实现简单，内存高效，缺点就是会减少可使用内存



3）标记整理算法

由于简单的标记清楚会存在碎片问题，所以出现了压缩清楚的算法，就是先清除需要回收的对象，然后再对内存进行压缩操作，将内存分成可用和不可用两部分

4）分代收集算法，

是上面算法的综合，Java堆分为新生代和老年代，这样就可以根据各个年代的特点采用适当的算法，在新生代中，每次收集时都会有大量的对象死去，只有少量存活，那就选用复制算法，只需要付出少量存活对象的复制成本就可以完成收集，而老年代因为对象存活率较高，没有额外的空间对他进行分配担保，就必须使用“标记-清除”或者“标记-整理”算法来回收

垃圾收集器分为串行收集器，并行收集器，并发收集器

Serial(用于新生代，采用复制算法，)（串行收集器）

Serial(用于老年代，采用标记整理算法)（串行收集器）

Parnew(用于新生代，采用复制算法)（并行收集器）

Parallel old(用于老年代，采用标记整理算法)（并行收集器）

Parallel Scavenge(用于新生代，采用复制算法)（并行收集器）

CMS(用于老年代，采用标记清除算法)（并发收集器）

Gl(jdk7以后的版本推出，维持高回收率，减少停顿，Java9 默认GC算法是Gl,且把CMS标记为废弃)

JVM分为两种模式

client模式和server模式，默认是client模式，server模式启动较慢，但长时间运行的话，运行会越来越快

Java的垃圾回收某个对象的时候，都要依据引用的概念

在不同的垃圾回收算法中，对引用的判断方式有所不同：

引用计数法：为每个对象添加一个引用计数器，每当有一个引用指向它时，计数器就加1，当引用失效时，计数器就减1，当计数器为0时，则认为该对象可以被回收（目前Java中已经弃用这种方式了）

可达性分析算法：从一个被称为 GC Roots的对象开始向下搜索，如果一个对象到GC Roots没有任何引用链相连时，则说明此对象不可用





jvm运行时内存有哪些

### **四种引用类型**

#### 强引用(Strong Reference):Java中默认声明 的引用

```java
Object obj=new Object();//只要obj还是指向Object对象，Object对象就不会被回收
obj=null;//手动置空
```

只要强引用存在，垃圾回收器将永远不会回收被引用的对象，哪怕内存不足，jvm会直接抛出OutOfMemoryError异常，不会去回收，如果想中断强引用与对象之间的联系，可以显示的将对象赋值为null，这样一来，jvm可以适时的回收对象了。

软引用(Soft Refrence) ：用来描述一些非必须但有用的对象，在内存足够的时候，软引用对象不会被回收，只有内存不足时，系统则会回收软引用对象，如果回收了软引用对象之后仍然没有足够的内存，才会抛出内存溢出异常。这种特性通常用来实现缓存技术，如网页缓存，图片缓存等，用SoftRefrence类表示软引用。

```java
 private static void testSoftReference() {
        for (int i = 0; i < 10; i++) {
            byte[] buff=new byte[1024*1024];
            SoftReference<byte[]> sr=new SoftReference<>(buff);
            list.add(sr);
        }
        System.gc();//主动通知垃圾回收
        for (int i = 0; i < list.size(); i++) {
            Object obj=((SoftReference)list.get(i)).get();
            System.out.println(obj);
        }
    }
```

弱引用(WeakRefrence)：引用强度比软引用更弱一些，无论内存是否足够，只要jvm进行垃圾回收，那些被弱引用关联的对象都会被回收，用WeakReference表示弱引用

虚引用(PhantomRefrence) ：最弱的引用关系，如果一个对象仅持有弱引用，那么他跟没有引用一样，随时可能会被回收，用PhantomReference类表示。在源码中，这个类只有一个构造方法和一个ge()方法，而他的get()方法只返回一个null，也就是说无法通过虚引用来获取对象，所以与软引用、弱引用不同，虚引用必须要和ReferenceQueue引用队列一起使用

引用队列(RefrenceQueue)：引用队列可以与软引用、弱引用和虚引用一起使用，当垃圾回收器回收一个对象时，如果发现他还有引用，那么就会在回收对象之前，把这个引用加入到与之关联的引用队列中去，程序可以通过判断引用队列中是否已经加入了引用，来判断被引用的对象是否将要被垃圾回收，这样就可以在被回收之前采取一些必要的措施。



### **IO和NIO的区别**

NIO即New IO，NIO和IO有相同的作用和目的，但是实现方式不同，NIO主要用到的是块，所以NIO的效率比IO高很多，在Java API中提供了两套NIO，一套是针对标准输入输出NIO，另一套就是网络编程NIO

| IO     | NIO      |
| ------ | -------- |
| 面向流 | 面向缓冲 |
| 阻塞IO | 非阻塞IO |
| 无     | 选择器   |

1.面向流与面向缓冲

Java IO与NIO最大的区别是面向流与面向缓冲，IO面向流意味着每次从流中读取一个或多个字节，直至读取所有字节，他们没有被缓存存在任何地方，此外他不能前后移动流中的数据，如果需要前后移动从流中读取的数据，需要将他缓存在缓存区，NIO的缓冲导向方法略有不同，数据读取到一个他稍后处理的缓冲区，需要时可在缓冲区中前后移动，增加了数据处理的灵活性，但是还需要检查是否该缓冲区中包含所有需要处理的数据，而且需确保当更多的数据读入到缓冲区时，不要覆盖缓冲区未处理的数据。

2.阻塞与非阻塞IO

Java IO的各种流是阻塞的，这意味着当一个线程调用read()或者write()时，该线程被阻塞，知道有一些数据被读取，或数据完全写入，该线程在此期间不能在干任何事了，Java NIO是非阻塞模式，使一个线程在某通道发送请求读取数据，但是他仅能得到目前可用的数据，如果目前没有可用数据时，就什么都不会读取，而不是保持线程阻塞，所以直至数据变得可以读取之前，该线程可以继续做其他的事情，非阻塞写也是如此，一个线程请求写入数据到某通道，但不需要等待他完全写入，这个线程同时可以去做其他的事情，线程通常将非阻塞IO的空闲时间用于在其他通道上执行IO操作，所以一个单独的线程现在可以管理多个输入和输出通道(channal)

3.选择器(Selcetors)

Java NIO的选择器允许一个单独的线程监视多个输入通道，你可以注册多个通道使用一个选择器，然后使用一个单独的线程选择通道，这些通道里已经有可以处理的输入，或者选择已准备写入的通道，这种选择机制，可以使一个单独的线程管理多个通道



### java的类加载机制

1.jvm将类的加载过程分三个步骤：装载(Load)、链接(Link)、初始化(Initialize)，链接又分为三个步骤：

1）装载：查找并加载类的二进制数据

2）链接：

​				验证：确保被加载类的正确性

​				准备：为类的静态变量分配内存，并将其初始化为默认值

​				解析：把类的符号引用转换为直接引用

3）初始化：为类的静态变量赋予正确的初始值

2.类的初始化：

类什么时候被初始化：

 1）创建类的实例，即new一个对象

2）访问某个类或者接口的静态变量，或者对该静态变量进行赋值

3）调用类的静态方法

4）反射(Class.forName(“com.y.load”))

5）初始化一个子类（会首先初始化一个子类的父类）

6）jvm启动时标明的启动类，即文件名和类名相同的那个类

3.类的初始化步骤：

1） 如果这个类没有被加载或者链接，那先进行加载和链接

2）假如这个类存在直接父类，并且这个二类还没有初始化（在类加载器中，一个类只能被初始化一次），那就初始化直接的父类（不适用于接口）

3）假如类中存在初始化语句（如static变量或者static块）那就依次执行这些初始化语句

4.类的加载

类的加载指的是将类的.class文件中二进制数据读入内存中，将其放在运行时数据区的方法内，然后在堆区创建一个这个类的java.lang.Class对象，用来封装类在方法区类的对象：

![img](https://images2018.cnblogs.com/blog/991670/201806/991670-20180628155146481-1545095755.png)



![img](https://images2018.cnblogs.com/blog/991670/201806/991670-20180628155204199-1045249479.png)

类加载的最终产品是位于堆区的Class对象，Class对象封装了类在方法区内的数据结构，并且向程序员提供了访问方法区内数据结构的接口

加载类的方式有一下几种：

1）通过本地系统直接加载

2）通过网络下载.class文件

3）从zip，jar等归档文件中加载.class文件

4）从专有数据库中提取.class文件

5）将Java源文件动态编译为.class文件

JVM的类的加载是通过CLassLoader及其子类完成的，

1）BootStrap ClassLoader：引导类加载器，负责加载%JAVA_HOME%中jre/lib/rt.jar里所有的class，由C#实现，不是ClassLoader子类

2）Extension ClassLoader：拓展类加载器，负责加载Java平台中扩展功能的一些jar包，包括$JAVA_HOME中jre/lib/ext/*.jar或者-Djava.ext.dirs指定目录下的包

3）App ClassLoader：系统类加载器，负责加载classpath中指定的jar包及目录中的class

4）Custom ClassLoader：自定义类加载器，如tomcat，jboss都会根据j2ee规范自行实现ClassLoader，加载过程中会先检查类是否已经被加载，检查顺序自底向上，从Custom ClassLoader到Bootstrap ClassLoader逐层检查，只要某个class Loader已加载就视为已加载此类，保证此类在所有Class Loader只加载一次，而加载的顺序写是自顶向下，也就是由上层来逐层尝试加载此类



### 双亲委派机制

Java虚拟机对class文采用的是按需加载的方式，也就是说当需要使用该类时才会将他的class文件加载到内存生成class对象，而且加载某个class文件时，Java默认采用的是双亲委派机制，即把请求交由父类处理

工作原理：

1）如果一个类加载器收到了类加载请求，他并不会自己先加载，而是把这个请求委托给父类的加载器去执行，

2）如果父类加载器还存在其父类加载器，则进一步向上委托，依次递归，请求最终到达顶层的引导类加载器，

3）如果父类加载器可以完成类加载任务，就成功返回，倘若父类加载器无法完成加载任务，子加载器就会尝试自己加载，这就是双亲委派机制

4）父类加载器一层一层的往下分配任务，如果子类加载器能够加载，则加载此类，如果将加载任务分配至系统类加载器也无法加载此类，则抛出异常。

![双亲委派机制图示](https://img-blog.csdnimg.cn/20201018150330316.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjgzNTIzMA==,size_16,color_FFFFFF,t_70#pic_center)

双亲委派机制可以：

1）避免类的重复加载

2）保护程序安全，防止核心API被篡改

- 自定义类：java.lang.String(没用)
- 自定义类：java.lang.SkuStr(报错，阻止创建java.lang开头的类)



### tomcat如何打破了双亲委派机制

![img](https://img2020.cnblogs.com/blog/1187916/202007/1187916-20200701044305758-214059128.png)

橙色部门还是和原来一样, 采用双亲委派机制. 而黄色部分是tomcat第一部分自定义的类加载器, 这部分主要是加载tomcat包中的类, 这一部分依然采用的是双亲委派机制, 而绿色部分是tomcat第二部分自定义类加载器, 正事这一部分, 打破了类的双亲委派机制



#### 1. tomcat第一部分自定义类加载器(黄色部分)

这部分类加载器, 在tomcat7及以前是tomcat自定义的三个类加载器, 分别加载不同文件加载的jar包. 而到了tomcat8及以后, tomcat将这三个文件夹合并了, 合并成了一个lib包. 也就是我们现在看到的lib包

- commonClassLoader: tomcat最基本的类加载器, 加载路径中的class可以被tomcat容器本身和各个webapp访问;
- catalinaClassLoader: tomcat容器中私有的类加载器, 加载路径中的class对于webapp不可见
- sharedClassLoader: 各个webapps共享的类加载器, 加载路径中的class对于所有的webapp都可见, 但是对于tomcat容器不可见.

这一部分类加载器, 依然采用的是双亲委派机制, 原因是, 他只有一份. 如果有重复, 那么也是以这一份为准. 

#### 2.tomcat第二部分自定义类加载器(绿色部分)

 绿色部分是java项目在打war包的时候, tomcat自动生成的类加载器, 也就是说 , 每一个项目打成一个war包, tomcat都会自动生成一个类加载器, 专门用来加载这个war包. 而这个类加载器打破了双亲委派机制. 我们可以想象一下, 加入这个webapp类加载器没有打破双亲委派机制会怎么样?

如果没有打破, 他就会委托父类加载器去加载, 一旦加载到了, 子类加载器就没有机会在加载了. 那么, spring4和spring5的项目想共存, 那是不可能的了. 

所以, 这一部分他打破了双亲委派机制

 这样一来, webapp类加载器不需要在让上级去加载, 他自己就可以加载对应war里的class文件. 当然了, 其他的项目文件, 还是要委托上级加载的. 

### 集合

java 集合类存放在java.util包中，是用来存放对象的容器。

注意：

1. 集合只能存放对象，比如存一个int类型的数据1，其实是自动转换成Integer类型存储的，Java中每一种基本类型都有其对应的引用类型
2. 集合存放的是多个对象啊的引用，对象本身还是放在堆中
3. 集合可以存放不同类型，不限数量的数据类型

#### 集合框架图：

![Java集合关系图](C:/Users/16143/Desktop/Java%E9%9B%86%E5%90%88%E5%85%B3%E7%B3%BB%E5%9B%BE.jpg)

除了map系列的集合外，左边的集合都实现了Iterator接口，这是一个用于遍历集合的接口，主要hashNext(),next(),remove(),三种方法，他的一个子接口ListIterator在他的基础上又添了三个方法，分别是add(),previous(),hasPrevious(),也就是说如果实现Iterator接口，那么在遍历集合元素的时候，只能往后遍历，被遍历后的元素不会再次遍历，通常无序集合实现的是这个接口，比如HashSet;而那些元素有序的集合，一般都实现的是ListIterator，实现这个接口的集合可以双向遍历，既可以通过next()访问下一个元素，也可以通过previous()访问前一个元素，比如ArrayList。



1. Iterator迭代器，这是Java集合的顶层接口(不包括map系列的集合，Map是map集合的顶层接口)

Object next()：返回迭代器刚越过的元素的引用，返回值是Object，需要强制转换为需要的类型。

boolNext()：判断容器内是否还有可供访问的元素

void remove()：删除迭代器刚越过的元素

所以除了map系列的集合，我们都可以通过迭代器进行遍历集合

注意：

在源码中看到，在集合的顶层接口，比如Collection接口，可以看到他继承的是Iterable

```java
public Interface Iterable<T>{
    Iterator<T> iterator();
}
```

Iterator和Iterable的区别：

在Iterable中封装了Iterator接口，只要实现了Iterable接口，就可以使用Iterator迭代器了

```java
//产生一个List集合，典型的实现为ArrayList
List list=new ArrayList();
//添加三个元素
list.add("qwe");
list.add("qwed");
list.add("fvd");
//构造list的迭代器
Iterator iterator= list.iterator();
while (iterator.hasNext())
{
    Object obj=iterator.next();
    System.out.println(obj);
}
```

2. Collection：List接口和Set接口的父接口

```java
public interface Collection<E> extends Iterable<E>{}
```

Collection集合使用：

```java
//将ArrayList作为Collection的实现类
Collection collection=new ArrayList();
//添加元素

collection.add("gfdc");
collection.add("efdfs");

//删除指定元素
collection.remove("gfdc");

//删除指定元素
Collection c=new ArrayList();
c.add("qwe");
c.removeAll(c);

//检测是否存在某个元素
c.contains("qwe");

//判断是否为空
c.isEmpty();

//利用增强for循环遍历集合
for (Object o : collection) {
    System.out.println(o);
}

//利用迭代器
Iterator iterator= collection.iterator();
while (iterator.hasNext()) {
    Object object=iterator.next();
    System.out.println(object);
}
```

3. #### List：有序，可重复的集合

```
public interface List<E> extends Collection<E>{}
```

1） List接口的三个典型实现：

```java
List list1 = new ArrayList();
List list2 = new LinkedList();
List list3 = new Vector();
```

ArrayList底层数据结构是数组，查询快，增删慢，线程不安全，效率高

Vector底层数据结构是数组，查询快，增删慢，线程安全，效率低，几乎已经淘汰

LinkedList底层数据结构是链表，查询慢，增删快，线程不安全，效率高

2） List接口遍历还可以使用普通的for循环进行遍历，指定位置添加元素，替换元素等

```java
//产生一个List集合，典型的实现为ArrayList
List list=new ArrayList();
//添加三个元素
list.add("qwe");
list.add("qwed");
list.add("fvd");
//构造list的迭代器
Iterator iterator= list.iterator();
while (iterator.hasNext())
{
    Object obj=iterator.next();
    System.out.println(obj);
}

//向指定地方添加元素
list.add(2,0);
//在指定地方替换元素
list.set(2,1);

//获取指定地方的索引
int i=list.indexOf(1);
System.out.println("索引为："+i);

//遍历，普通for循环
for (int j = 0; j < list.size(); j++) {
    System.out.println(list.get(j));
}
```

4. #### Set：典型实现是HashSet()是一个无序的不可重复的集合

1）Set set=new HashSet();底层原理是：数组+链表+红黑树，底层是HashMap

HashSet不能保证元素的顺序，不可重复，线程不是安全的 ，集合 元素可以是null。

底层其实是个数组，存在的意义是加快查询速度，在一般的数组中，元素在数组中的位置是随机的，元素的取值和位置之间不存在确定关系 ，因此，在数组中查找值和一些特定的元素时，需要把查找值和一系列的元素相比较，此时的查找效率依赖于查找过程中的比较次数，而HashSet底层数组的索引和值有确定的关系，index=hash(value)，那么只要调用这个公式，就能快速找到元素或者索引。

对于HashSet：如果两个对象通过eques()方法返回TRUE，这两个对象的hashcode值应该相同，

1.当HashSet集合中存入一个元素时，HashSet会调用该对象的hashCode()方法来得到该对象的hashcode值，然后根据hashcode值来确定该对象在HashSet中的位置。

1.1 如果hashcode值不同，	那直接将元素存到hashCode()指定的位置，

1.2 如果hashCode值相同，那么会继续判断该元素和集合对象的equals()作比较，

 1.2.1 hashcode相同，equals为true，则视为同一个对象，不保存在hashSet中，

 1.2.2 hashcode相同，equals为false，则存储在之前对象同槽位的链表上，这非常麻烦，所以我们应该保证：如果两个对象通过equals方法返回true，这两个对象的hashcode值也应该相同，

注意： 每个存储到哈希表中国的对象，都得提供hashCode()和equals()方法的实现，用来判断是否是同一个对象，对于hashSet集合，我们要保证如果两个对象通过equals()方法返回true，这两个对象的 hashcode值也应该相同，

2. ）Set linkedHashSet = new LinkedHashSet();

不可以重复，有序　　因为底层采用 链表 和 哈希表的算法。链表保证元素的添加顺序，哈希表保证元素的唯一性

3) Set treeSet = new TreeSet();

   TreeSet:有序；不可重复，底层使用 红黑树算法，擅长于范围查询。

     * 如果使用 TreeSet() 无参数的构造器创建一个 TreeSet 对象, 则要求放入其中的元素的类必须实现 Comparable 接口所以, 在其中不能放入 null 元素

     * 必须放入同样类的对象**.(默认会进行排序) 否则可能会发生类型转换异常.我们可以使用泛型来进行限制

       ```java
       Set treeSet = new TreeSet();
           treeSet.add(1); //添加一个 Integer 类型的数据
           treeSet.add("a");  //添加一个 String 类型的数据
           System.out.println(treeSet);  `//会报类型转换异常的错误
       ```

自动排序：添加自定义对象的时候，必须要实现 Comparable 接口，并要覆盖 compareTo(Object obj) 方法来自定义比较规则

　　　　如果 this > obj,返回正数 1

　　　　如果 this < obj,返回负数 -1

　　　　如果 this = obj,返回 0 ，则认为这两个对象相等

   *  两个对象通过 Comparable 接口 compareTo(Object obj) 方法的返回值来比较大小, 并进行升序排列



#### Map:Map：key-value 的键值对，key 不允许重复，value可以

1、严格来说 Map 并不是一个集合，而是两个集合之间 的映射关系。

2、这两个集合没每一条数据通过映射关系，我们可以看成是一条数据。即 Entry(key,value）。Map 可以看成是由多个 Entry 组成。

3、因为 Map 集合即没有实现于 Collection 接口，也没有实现 Iterable 接口，所以不能对 Map 集合进行 for-each 遍历。

Map的常用实现类：https://blog.csdn.net/qq877728715/article/details/103029866



<img src="https://images2015.cnblogs.com/blog/1120165/201705/1120165-20170509224035660-1080038371.png" alt="img" style="zoom: 150%;" />



#### Map 和 Set 集合的关系

　　　　1、都有几个类型的集合。HashMap 和 HashSet ，都采 哈希表算法；TreeMap 和 TreeSet 都采用 红-黑树算法；LinkedHashMap 和 LinkedHashSet 都采用 哈希表算法和红-黑树算法。

　　　　2、分析 Set 的底层源码，我们可以看到，Set 集合 就是 由 Map 集合的 Key 组成。



### Map当中的ConcurrentHashMap是如何实现线程安全的

1. HashTable之所以效率低下是因为其实现使用了synchronized关键字对于put等操作进行加锁，而synchronized关键字加锁是对整个对象进行加锁，也就是说在进行put等修改哈希表的操作时，锁住了整个哈希表，从而使得其表现的效率低下，因此在jdk1,5~1，7版本，Java使用了分段锁机制实现ConcurrentHashMap
2. ConcurrentHashMap在对象中保存了一个Segment数组，默认长度为16，即将整个哈希表分为多个分段，而每个Segment元素，即每个分段类似于一个hashtable，这样在执行put操作时，首先根据

### 创建线程的四种方式

#### 1. 继承Thread类创建线程

```java
public class Demo extends Thread{ //继承Thread
        String name;
        Demo(String name){
            this.name=name;
        }
        //复写其中的run方法
        @Override
        public void run(){
            for (int i = 0; i <= 20; i++) {
                System.out.println("name="+name+",i="+i);
            }
        }
        class ThreadDemo{
            public static void main(String[] args) {
                //创建两个线程任务
                Demo d1=new Demo("ewre");
                Demo d2=new Demo("wqerrwreewr");
//                d1.run();
//                d2.run(); //主线程调用run方法，并没有开启两个线程
                d2.start(); //开启一个线程
                d1.start(); //主线程在调用run方法
            }

        }
```

原理：建立单独的路径，让部分代码实现同时执行。需要重写run方法

#### 2. 实现Runnable接口

2.1 定义类实现Runnable接口

2.2 覆盖接口中的run方法

2.3 创建Thread类对象

2.4 将Runnable 接口的子类对象作为参数传递给Thread类的构造函数

2.5 调用Thread类的start方法开启线程

```java
class Demo implements  Runnable{

    private String name;
    Demo(String name){
        this.name=name;
    }

    @Override
    public void run() {
        for (int i = 0; i <=20; i++) {
            System.out.println("name="+name+"_______"+Thread.currentThread().getName()+"___________"+i);
        }
    }
}
public class ThreadDemo2 {
    public static void main(String[] args) {
        Demo d=new Demo("Demo");
        Thread t1=new Thread(d);
        Thread t2=new Thread(d);
        t1.start();
        t2.start();
        System.out.println(Thread.currentThread().getName()+"-------------------->");
        System.out.println("Helle World");
    }
```

原理：避免了Thread类单继承的局限性，覆盖Runnable接口的run方法，将线程任务代码定义到run方法中，创建Thread类的对象，只有创建Thread类的对象才可以创建线程，线程任务已经被封装到Runnable接口的run方法中，而这个run方法所属于Runnable接口的子类对象，所以将这个子类对象作为参数传递给Thread类的构造函数，这样，线程创建时就可以明确要运行的线程的任务

好处：实现Runnable的方式，更加的符合面向对象，线程分为两部分，一部分是线程任务，一部分是线程对象
继承Thread类，线程对象和线程任务耦合再一起，创建线程任务时又创建了线程对象，

实现Runnable接口：将线程任务单独分离出来封装成对象，类型就是Runnable接口类型，Runnable接口对线程任务和线程对象进行解耦



多线程内存图解：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200803232509132.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ1OTAxNzQx,size_16,color_FFFFFF,t_70#pic_center)

Thread.currentThread()：获取当前线程对象

Thread.currentThread().getName()：获取当前线程对象的名称

多线程的异常只会影响异常所属的那个线程

线程安全问题产生的原因：a.线程任务中操作共享的数据    b.线程任务操作共享数据的代码有多条（共享资源弊端隐患）

解决共享的资源：让一个线程在执行线程任务时将操作多条共享数据的代码执行完，在执行过程中，不让其他线程参与运算，Java中提供了synchronized代码块标识其作为一个同步代码块。

##### synchronized关键字

synchronized使用时，需要一个对象作为标记，当任何线程进入synchronized这个标记的代码块时，首先会判断线程有没有使用synchronized标记对象，若有线程使用这个对象，那么这个线程就在synchronized外面等着，直到获取到这个标记对象后，这个线程才能执行同步代码块

```java
class Ticket implements Runnable
{
	//1、描述票的数量。
	private int tickets = 100;
	//2、售票的动作，这个动作需要被多线程执行，那就是线程任务代码。需要定义run方法中。
	//定义同步代码块的标记对象。相当与锁的功能
	private Object obj = new Object();
	public void run()
	{
		//线程任务中通常都有循环结构。
		while(true)
		{
			//使用同步代码块解决线程安全问题。
			synchronized(obj)
			{
				if(tickets>0)
				{	//由于run方法是复写接口中的，run方法没有抛出异常， 此时这里只能捕获异常，而不能抛出
					//让线程在此冻结10毫秒
					try{
						Thread.sleep(10);
						}catch(InterruptedException e){/*异常处理代码*/}
					System.out.println(Thread.currentThread().getName()+"....."+tickets--);//打印线程名称。
				}
			}
		}
	}
}
class ThreadDemo3 
{
	public static void main(String[] args) 
	{
		//1,创建Runnable接口的子类对象。
		Ticket t = new Ticket();

		//2,创建四个线程对象。并将Runnable接口的子类对象作为参数传递给Thread的构造函数。
		Thread t1 = new Thread(t);
		Thread t2 = new Thread(t);
		Thread t3 = new Thread(t);
		Thread t4 = new Thread(t);

		//3,开启四个线程。
		t1.start();
		t2.start();
		t3.start();
		t4.start();
	}
}

```



1.同步的好处和弊端：

好处：解决线程安全问题

弊端：降低了程序的性能，每个线程都需要判断锁机制，会增加程序运行负担，增加CPU压力

２.同步的前提：

当线程任务代码会被多个线程执行时，这时需要加同步，但是加同步时，**一定要保证多个线程使用的是同一把锁**

3.同步代码块使用的锁可以是任意对象的，因为synchronized对象可以我们自己指定

4.同步函数锁，函数是需要调用对象去使用的，则同步函数中使用的锁就是当前调用这个方法的对象，this锁

5.同步代码块和同步函数锁的区别

同步函数使用的锁是固定的this ,当线程任务只需要一个同步时，完全可以使用同步函数

同步代码块使用的锁可以是任意对象，当线程中需要多个同步时，必须通过锁来区分，这是必须使用同步代码块，

多线程单例懒汉式的并发问题

```java
class Single
{
	private static  Single s = null;
	private Single(){}
	/*
	并发访问会有安全隐患，所以加入同步机制解决安全问题。
	但是，同步的出现降低了效率。
	可以通过双重判断的方式，解决效率问题，减少判断锁的次数。
	*/
	public static Single getInstance()
	{
		if(s==null)
		{
			synchronized(Single.class)//-->B线程，等着A解锁才让进去
			{
				if(s==null) s = new Single();
			}
		}
		return s;
	}
}

```

懒汉模式是延迟加载的实例，面对多线程访问时，需要进行同步代码块，为了增加效率，又要使用双重判断

如果没有第一个if ,那么多线程访问时，每个线程都需要去判断锁，而双重判断模式下，更多机会只需要判断if条件，相比较判断锁更有效率

死锁：

当线程任务中出现了多个同步（多个锁）时，如果同步中嵌套了其他的同步，这是容易引发死锁

##### 死锁经典代码：

```java
class Text implements Runabble{
	private boolean flag ;
	Test(boolean flag)
	{
		this.flag = flag;
	}
	public void run()
	{
		if(flag)
		{
			synchronized(MyLock.LOCKA)
			{
				System.out.println(Thread.currentThread().getName()+"...if...MyLock.LOCKA");
				synchronized(MyLock.LOCKA)
				{
					System.out.println(Thread.currentThread().getName()+"...if...MyLock.LOCKB");
				}
			}
		}
		else
		{
			synchronized(MyLock.LOCKB)
			{
				System.out.println(Thread.currentThread().getName()+"...if...MyLock.LOCKB");
				synchronized(MyLock.LOCKA)
				{
					System.out.println(Thread.currentThread().getName()+"...if...MyLock.LOCKA");
				}
			}
		}

	}
}
//单独描述锁对象
class MyLock
{
	public static final MyLock LOCKA = new MyLock();
	public static final MyLock LOCKB = new MyLock();
}
class DeadThread 
{
	public static void main(String[] args) 
	{
		Test t1 = new Test(true);
		Test t2 = new Test(false);
		Thread t11 = new Thread(t1);
		Thread t22 = new Thread(t2);
		t11.start();
		t22.start();
	}
}

```

##### 死锁产生的条件和解决方案

1)竞争不可抢占性资源

如上述例子中，LockA想去打开LockB，LockB又想去打开LockA，但是LockA和LockB都是不可抢占的，这是发生死锁。

2)竞争可消耗资源引起死锁

进程间通信，如果顺序不当，会产生死锁，比如p1发消息m1给p2，p1接收p3的消息m3，p2接收p1的m1，发m2给p3，p3，以此类推，如果进程之间是先发信息的那么可以完成通信，但是如果是先接收信息就会产生死锁。

3)进程推进顺序不当

进程在运行过程中，请求和释放资源的顺序不当，也同样会导致产生进程死锁。
　　
死锁产生的必要条件：

产生死锁必须同时满足以下四个条件，只要其中任一条件不成立，死锁就不会发生。

1、互斥条件：进程要求对所分配的资源（如打印机）进行排他性控制，即在一段时间内某 资源仅为一个进程所占有。此时若有其他进程请求该资源，则请求进程只能等待。

2、不剥夺条件：进程所获得的资源在未使用完毕之前，不能被其他进程强行夺走，即只能 由获得该资源的进程自己来释放（只能是主动释放)。

3、请求和保持条件：进程已经保持了至少一个资源，但又提出了新的资源请求，而该资源 已被其他进程占有，此时请求进程被阻塞，但对自己已获得的资源保持不放。

4、循环等待条件

死锁解决办法：

1、加锁顺序（线程按照一定的顺序加锁）

2、加锁时限（线程尝试获取锁的时候加上一定的时限，超过时限则放弃对该锁的请求，并释放自己所占的锁）

3、死锁检测

​	步骤一：每个进程，每个资源制定唯一编号

​	步骤二：设定资源分配表，记录进程与所占资源之间的关系

​	步骤三：设置进程等待表，记录进程与要申请资源之间的关系

##### 生产者和消费者

处理同一资源，而处理的方式不同，生产者和消费者同时执行，需要多线程，但是执行的任务不同，处理的资源却是相同的：线程间的通信

思路:

​	1、描述处理的资源

​	2、描述生产者，具备着生产的任务

​	3、描述消费者，具备着消费的任务

等待唤醒机制

为解决连续生产不消费，或者同一个商品多次消费，引入该机制

生产者生产了商品后应该告诉消费者来消费，这时的生产者应该处于等待状态，消费者消费了商品后，应该告诉生产者，这时消费者处于等待状态

wait()：会让线程处于等待状态，其实就是将线程放在了线程池中，

notify()：唤醒线程池中任意一个等待的线程

notifyAll()：唤醒线程池中所有等待线程

​		这些方法必须使用在同步中，因为必须要标识wait，notify等方法，所属的锁。同一个锁上的notify，只能唤醒该锁上的被wait的线程。

为什么这些方法定义在Object类中

因为这些方法必须标识所属的锁，而锁可以是任意对象，任意对象调用的方法必然是Object类中的方法。

```java
package com.note.refrence;
import java.util.Properties;

/**
 * @author Ykj
 * @ClassName ThreadDemo3
 * @Discription
 * @date 2022/1/17 9:05
 */

//描述资源，属性：商品名和编号    行为：对商品名进行赋值，获取商品
class Resource{
    private String name;
    private int count=1;
    private boolean flag=false;

    //设置对外提供的设置商品的方法
    public synchronized void set(String name){
        while (flag){
            try {
                wait();

            }catch (InterruptedException e){}
        }
        this.name=name+count;
        count++;
        System.out.println(Thread.currentThread().getName()+"............生产者......."+this.name);
        //将标记改为true
        flag=true;
        //唤醒消费者
        this.notifyAll();
    }

    public synchronized void get(){
        while (!flag){
            try {
                wait();

            }catch (InterruptedException e){}
        }
        System.out.println(Thread.currentThread().getName()+"............消费者......."+this.name);
        //将标记改为true
        flag=false;
        //唤醒消费者
        this.notifyAll();
    }

}
//描述生产者
class Producer implements Runnable{
    private Resource r;
    Producer(Resource r){
        this.r=r;
    }

    @Override
    public void run() {
        while (true){
            r.set("面包");
        }
    }
}
//描述消费者
class Consumer implements Runnable{

    private Resource r;
    Consumer(Resource r){
        this.r=r;
    };
    //生产者生产商品的任务
    @Override
    public void run() {

        while (true){
            r.get();
        }
    }
}
public class ThreadDemo3 {
    public static void main(String[] args) {
        //创建资源对象
        Resource resource=new Resource();
        //创建生产者对象
        Producer producer=new Producer(resource);
        //创建消费者对象
        Consumer consumer=new Consumer(resource);
        //创建线程对象
        Thread producerThread=new Thread(producer);
        Thread consumerThread=new Thread(consumer);
        //开启线程
        producerThread.start();
        consumerThread.start();
        System.out.println("hello world");
    }
}
```

注意：多生产多消费，必须循环while条件，但是使用了while会死锁，不能唤醒对方，为了唤醒，没有对应的方法，只能唤醒所有

#### 3.  使用Callable和Future创建线程

和Runnable接口不一样，Callable接口提供了一个call方法作为线程执行体，call方法比run方法要强大，其表现在

1、call方法可以有返回值

2、call方法可以申明抛出异常

Java5提供了Future接口来代表Callable接口里call方法的返回值，并且为Future接口提供了实现类FutureTask，这个类既实现了Future接口，也实现了Runnable接口，因此可以作为Thread类的target。

##### 创建并启动有返回值的线程的步骤如下：

1）创建Callable接口的实现类，并实现call方法，然后创建该实现类的实例（从Java8开始就是用Lambda表达式创建callable对象）

2）使用FutureTask类来包装Callable对象，该FutureTask对象封装了Callable对象的call()方法的返回值。

3）使用FutureTask对象作为Thread对象的target创建并启动线程（因为FutureTask实现了Runnable接口）

4）调用FutureTask对象的get方法获得子线程执行结束后的返回值

```java
public class MyThread implements Callable<String>{//Callable是一个泛型接口
 
	@Override
	public String call() throws Exception {//返回的类型就是传递过来的V类型
		for(int i=0;i<10;i++){
			System.out.println(Thread.currentThread().getName()+" : "+i);
		}
		
		return "Hello Tom";
	}
	public static void main(String[] args) throws Exception {
		MyThread myThread=new MyThread();
		FutureTask<String> futureTask=new FutureTask<>(myThread);
		Thread t1=new Thread(futureTask,"线程1");
		Thread t2=new Thread(futureTask,"线程2");
		Thread t3=new Thread(futureTask,"线程3");
		t1.start();
		t2.start();
		t3.start();
		System.out.println(futureTask.get());
		
	}
}

```

FutureTask 适用于耗时的计算，主线程可以在完成自己的任务后，再去获取结果，FutureTask可以保证即使调用了多次run方法，他都会执行一次Runnable或者Callable任务，或者通过cancel取消FutureTask的执行等

线程池：

线程有五种状态：

NEW状态(初始化状态)：实例化一个Thread对象出来（未执行start()方法前）Thread的状态为NEW

RUNNABLE(可运行)状态：调用线程的start()方法，此时线程进入RUNNABLE状态，该状态的线程位于可运行线程池中，等待被操作系统调度，获取CPU的使用权，当获得CPU的时间片时，线程进入运行状态，执行程序代码

BLOCKED(阻塞)状态：当线程等待获取monitor锁（synchronized）时，线程就会进入BLOCKED状态，等待获取monitor锁，线程的状态时BLOCKED，等待获取Lock锁(LockSupport.park())，线程的状态是WAITING。

TIMED_WAITING(超时等待)状态：当执行

```java
Thread.sleep(time)
Thread.join(time)
Object.wait(time)
LockSupport.parkNanos(time)
LockSupport.partUntil(time)
```

等操作时，线程进入TIMED_WAITING状态

当执行超时时间到

```java
Thread.join() 线程执行完
Object.notify()
notifyAll()
LockSupport.unpark()
```

线程被中断操作时，线程会从TIMED_WAITING状态转到RUNNABLE状态

WAITING(等待)状态：当执行Object.wait()、Thread.join()、LockSupport.unpark()，线程被中断操作时，线程会从RUNNABLE状态进入到WAITING状态，当执行Object.notify()/notifyAll()、Thread.join()程序执行完、LockSupport.unpark()、线程被中断等操作时，线程会从WAITING状态进入RUNNABLE状态。

TERMINATED(终止)状态：当线程执行完毕、Thread.stop()、内存溢出时，线程就会进入TERMINATED状态，线程一旦死亡，就无法复活，在一个死亡的线程上调用start方法，会抛出java.lang.IllegalThreadStateException异常

分别对应操作系统进程（线程）的五种状态：新建（new Thread）、就绪（runnable）、运行（running）、终止（dead）、阻塞（blocked）

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210401230053496.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ1OTAxNzQx,size_16,color_FFFFFF,t_70)

![img](https://img-blog.csdn.net/20180601075651365?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzMyOTM5Njc5/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

Thread类中有个内部枚举类描述这六种状态

```java
public enum State {
        NEW,
        RUNNABLE,
        BLOCKED,
        WAITING,
        TIMED_WAITING,
        TERMINATED;
    }
```

java中涉及到线程池的相关类在jdk1.5开始的java.util.concurrent包中，涉及到的几个核心类包括Executor、Executors、ExecutorService、ThreadPoolExecutor、FutureTask、Callable、Runnable等

代码比较(https://blog.csdn.net/qq_45901741/article/details/113752400?spm=1001.2014.3001.5501)

```java
public class PuTongXianCheng {
    public static void main(String[] args) throws Exception {
       	//计时
        Long start = System.currentTimeMillis();
		//生成随机数
        Random random = new Random();
        List list = new ArrayList();

        for (int i = 0;i <= 100000;i++){
        	
            Thread thread = new Thread(){
            	//创建线程
                @Override
                public void run() {
                    list.add(random.nextInt());
                }
            };
            //线程运行
            thread.start();
            thread.join();
        }
        System.out.println("时间:"+(System.currentTimeMillis() - start));
        System.out.println("list = "+list.size());
    }
}
```

代码运行时间为

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210401230859152.JPG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ1OTAxNzQx,size_16,color_FFFFFF,t_70#pic_center)



```java
public static void main(String[] args) throws Exception {
        Long start = System.currentTimeMillis();

        Random random = new Random();
        List list = new ArrayList();

        //创建线程池
        ExecutorService service = Executors.newSingleThreadExecutor();
        for (int i = 0;i <= 100000;i++){
            service.execute(
                    new Thread(){
                        @Override
                        public void run() {
                            list.add(random.nextInt());
                        }
                    }
            );
        }
        service.shutdown();
        service.awaitTermination(1, TimeUnit.DAYS);

        System.out.println("时间:"+(System.currentTimeMillis() - start));
        System.out.println("list = "+list.size());
    }

```

运行时间为：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210401230958863.JPG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ1OTAxNzQx,size_16,color_FFFFFF,t_70#pic_center)

线程池的创建：

线程池可以自动创建，也可以手动创建

1、自动创建体现在Executors工具类中，常见的可以i创建newFixedThreadPool、newCachedThreadPool、newSingleThreadExecutor

```java
public static ExecutorService newFixedThreadPool(int var0) {
        return new ThreadPoolExecutor(var0, var0, 0L, TimeUnit.MILLISECONDS, new LinkedBlockingQueue());
  }
	
  public static ExecutorService newSingleThreadExecutor() {
        return new Executors.FinalizableDelegatedExecutorService(new ThreadPoolExecutor(1, 1, 0L, TimeUnit.MILLISECONDS, new LinkedBlockingQueue()));
  }
 
  public static ExecutorService newCachedThreadPool() {
        return new ThreadPoolExecutor(0, 2147483647, 60L, TimeUnit.SECONDS, new SynchronousQueue());
  }

```

2、手动创建主要使用ThreadPoolExecutor类，体现在可以灵活设置线程的各个参数，体现在代码中即ThreadPoolExecutor类构造器上各个实参的不同

```java
public ThreadPoolExecutor(int corePoolSize,
                           int maximumPoolSize,
                           long keepAliveTime,
                           TimeUnit unit,
                           BlockingQueue<Runnable> workQueue,
                           ThreadFactory threadFactory,
                          RejectedExecutionHandler handler) 

```

3、ThreadPoolExecutor重要参数：

**`corePoolSize`**：核心线程数，也就是线程池中常驻的线程数，线程池初始化是没有线程的，当任务来临时才开始创建线程去执行任务

**`maximumPoolSize`**：最大线程数在核心线程数的基础上可能会额外增加一些非核心线程，需要注意的是只有当workQueue队列填满时才会创建多于corePoolSize的线程

**`keepAliveTime`**：非核心线程的空闲时间超过keepAliveTime就会被自动终止回收掉，注意当corePoolSize=maxPoolSize时，keepAliveTime参数也就不起作用了（因为不存在核心线程）

**`unit`**：keepAliveTime的时间单位

**`workQueue`**：用于保存任务的队列，可以为无界，有界、同步移交三种队列类型之一，当池子里的工作线程数大于corePoolSize时，这时新进来的队列就会被放在队列中

**`threadFactory`**：创建线程的工厂类，默认使用Executors.defaultThreadFactory()，也可以使用guava库的ThreadFactoryBuilder来创建

**`handler`**：线程池无法继续接收任务(队列已满且线程数达到	)时的饱和策略，取值有AbortPolicy、CallerRunsPolicy、DiscardOldestPolicy、DiscardPolicy

#### 4.线程池中线程的创建流程

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210401234204561.JPG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ1OTAxNzQx,size_16,color_FFFFFF,t_70#pic_center)

##### 线程池的拒绝策略

前提：所有的拒绝策略都实现了接口RejectedExecutionHandler

```java
public interface RejectedExecutionHandler {

    /**
     * @param r the runnable task requested to be executed
     * @param executor the executor attempting to execute this task
     * @throws RejectedExecutionException if there is no remedy
     */
    void rejectedExecution(Runnable r, ThreadPoolExecutor executor);
}

```

这个接口只有一个rejectedExecution方法，r 为待执行任务，executor 为线程池

当线程池的任务队列已满并且线程池中线程的数量达到maximumPoolSize时，如果还有任务到来时就会采取拒绝策略，通常有以下四种策略

**ThreadPoolExecutor.AbortPolicy：**丢弃任务并抛出RejectedExecutionException异常，也是线程池的默认拒绝策略

```java
private static final RejectedExecutionHandler defaultHandler = new AbortPolicy();
```

如果是比较关键的业务，推荐使用此拒绝策略，这样子在系统不能承载更大的并发量的时候，能够及时的通过异常发现。但是会中断调用者的处理过程，所以除非有明确需求，一般不推荐。

**ThreadPoolExecutor.DiscardPolicy：**丢弃任务，但是不抛出异常，如果线程队列已满，则后续提交的任务都会被丢弃，且是静默丢弃。
使用此策略，可能会使我们无法发现系统的异常状态。建议是一些无关紧要的业务采用此策略。

**ThreadPoolExecutor.DiscardOldestPolicy**：丢弃队列最前面的任务，然后重新提交被拒绝的任务。
此拒绝策略，是一种喜新厌旧的拒绝策略。是否要采用此种拒绝策略，还得根据实际业务是否允许丢弃老任务来认真衡量。

**ThreadPoolExecutor.CallerRunsPolicy**：由调用线程（提交任务的线程）处理该任务

##### 四种创建线程的方法对比：

实现Runnable和Callable接口的方法基本相同，不过是后者的执行有返回值，后者线程执行体run()方法无返回值，并且如果使用FutureTask类的话只执行一次Callable任务

这张方式与继承Thread类的方法之间的区别如下：

1、线程只是实现Runnable或者Callable接口，还可以继承其他类

2、这个方式下，多个线程可以共享一个target对象，非常适合多线程处理同一资源的场景

3、继承Thread类只需要this关键字就能获取当前线程，不需要使用Thread.currentThread()方法

4、继承Thread类的线程类不能继承其他父类（Java单继承决定）

5、前三种的线程如果创建关闭频繁会影响系统资源和性能，而使用线程池可以在不使用线程时将线程放回线程池，用的时候再从线程池提取，项目开发中只要用线程池创建多个线程

6、实现接口的创建方式必须实现方法（run()/call()）

### 如何终止线程：

1. 使用退出标志，使线程正常退出，也就是当run方法完成后线程终止。
2. 使用stop方法强行终止，但是不推荐这个方法，因为stop和suspend及resume一样都是过期作废的方法。
3. 使用interrupt方法中断线程。

##### 1. 停止不了的线程

interrupt()方法的使用效果并不像for+break语句那样，马上就停止循环。调用interrupt方法是在当前线程中打了一个停止标志，并不是真的停止线程。

```java
public class MyThread extends Thread {
    public void run(){
        super.run();
        for(int i=0; i<500000; i++){
            System.out.println("i="+(i+1));
        }
    }
}

public class Run {
    public static void main(String args[]){
        Thread thread = new MyThread();
        thread.start();
        try {
            Thread.sleep(2000);
            thread.interrupt();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}

```

输出结果：

```java
...
i=499994
i=499995
i=499996
i=499997
i=499998
i=499999
i=500000

```



##### 2. 判断线程是否停止状态

Thread.java类中提供了两种方法：

1. this.interrupted(): 测试当前线程是否已经中断；
2. this.isInterrupted(): 测试线程是否已经中断；

this.interrupted()方法的解释：测试当前线程是否已经中断，当前线程是指运行this.interrupted()方法的线程。

```java
public class MyThread extends Thread {
    public void run(){
        super.run();
        for(int i=0; i<500000; i++){
            i++;
//            System.out.println("i="+(i+1));
        }
    }
}

public class Run {
    public static void main(String args[]){
        Thread thread = new MyThread();
        thread.start();
        try {
            Thread.sleep(2000);
            thread.interrupt();

            System.out.println("stop 1??" + thread.interrupted());
            System.out.println("stop 2??" + thread.interrupted());
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}

```

运行结果:

```java
stop 1??false
stop 2??false
```

类Run.java中虽然是在thread对象上调用以下代码：thread.interrupt(), 后面又使用

```java
System.out.println("stop 1??" + thread.interrupted());
System.out.println("stop 2??" + thread.interrupted());  
```

来判断thread对象所代表的线程是否停止，但从控制台打印的结果来看，线程并未停止，这也证明了interrupted()方法的解释，测试当前线程是否已经中断。这个当前线程是main，它从未中断过，所以打印的结果是两个false.

```java
public class Run2 {
    public static void main(String args[]){
        Thread.currentThread().interrupt();
        System.out.println("stop 1??" + Thread.interrupted());
        System.out.println("stop 2??" + Thread.interrupted());

        System.out.println("End");
    }
}    
```

运行结果：

```java
stop 1??true
stop 2??false
End
```

方法interrupted()的确判断出当前线程是否是停止状态。但为什么第2个布尔值是false呢？ 官方帮助文档中对interrupted方法的解释：
**测试当前线程是否已经中断。线程的中断状态由该方法清除。** 换句话说，如果连续两次调用该方法，则第二次调用返回false。

下面来看一下isInterrupted()方法。

```java
public class Run3 {
    public static void main(String args[]){
        Thread thread = new MyThread();
        thread.start();
        thread.interrupt();
        System.out.println("stop 1??" + thread.isInterrupted());
        System.out.println("stop 2??" + thread.isInterrupted());
    }
}
```

运行结果：

```java
stop 1??true
stop 2??true
```

isInterrupted()并为清除状态，所以打印了两个true。

##### 3. 能停止的线程--异常法

在线程中用for语句来判断一下线程是否是停止状态，如果是停止状态，则后面的代码不再运行即可：

```java
public class MyThread extends Thread {
    public void run(){
        super.run();
        for(int i=0; i<500000; i++){
            if(this.interrupted()) {
                System.out.println("线程已经终止， for循环不再执行");
                break;
            }
            System.out.println("i="+(i+1));
        }
    }
}

public class Run {
    public static void main(String args[]){
        Thread thread = new MyThread();
        thread.start();
        try {
            Thread.sleep(2000);
            thread.interrupt();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}

```

运行结果：

```java
...
i=202053
i=202054
i=202055
i=202056
线程已经终止， for循环不再执行
```

上面的示例虽然停止了线程，但如果for语句下面还有语句，还是会继续运行的。看下面的例子：

```java
public class MyThread extends Thread {
    public void run(){
        super.run();
        for(int i=0; i<500000; i++){
            if(this.interrupted()) {
                System.out.println("线程已经终止， for循环不再执行");
                break;
            }
            System.out.println("i="+(i+1));
        }

        System.out.println("这是for循环外面的语句，也会被执行");
    }
}

```

执行结果：

```java
...
i=180136
i=180137
i=180138
i=180139
线程已经终止， for循环不再执行
这是for循环外面的语句，也会被执行
```

***此处赶时间，为后续学习使用https://www.cnblogs.com/greta/p/5624839.html***



### sleep和wait的区别，

1、这两个方法来自**不同的类**分别是，sleep来自Thread类，和wait来自Object类。

sleep是Thread的静态类方法，谁调用的谁去睡觉，即使在a线程里调用了b的sleep方法，实际上还是a去睡觉，要让b线程睡觉要在b的代码中调用sleep。
2、**最主要**是sleep方法没有**释放锁**，而wait方法释放了锁，使得其他线程可以使用同步控制块或者方法。

sleep不出让系统资源；wait是进入线程等待池等待，出让系统资源，其他线程可以占用CPU。一般wait不会加时间限制，因为如果wait线程的运行资源不够，再出来也没用，要等待其他线程调用notify/notifyAll唤醒等待池中的所有线程，才会进入就绪队列等待OS分配系统资源。sleep(milliseconds)可以用时间指定使它自动唤醒过来，如果时间不到只能调用interrupt()强行打断。

Thread.Sleep(0)的作用是“触发操作系统立刻重新进行一次CPU竞争”。
3、**使用范围：**wait，notify和notifyAll只能在同步控制方法或者同步控制块里面使用，而sleep可以在任何地方使用
  synchronized(x){
   x.notify()
   //或者wait()
  }
4、sleep必须**捕获异常**，而wait，notify和notifyAll不需要捕获异常

### start和run的区别，

1.start（）方法来启动线程，真正实现了多线程运行，这时无需等待run方法体代码执行完毕而直接继续执行下面的代码：
通过调用Thread类的start()方法来启动一个线程，这时此线程是处于就绪状态，并没有运行。然后通过此Thread类调用方法run()来完成其运行操作的，这里方法run()称为线程体，它包含了要执行的这个线程的内容，Run方法运行结束，此线程终止，而CPU再运行其它线程，

2.run（）方法当作普通方法的方式调用，程序还是要顺序执行，还是要等待run方法体执行完毕后才可继续执行下面的代码：
而如果直接用Run方法，这只是调用一个方法而已，程序中依然只有主线程--这一个线程，其程序执行路径还是只有一条，这样就没有达到写线程的目的。

3.Thread对象的run()方法在一种循环下，使线程一直运行，直到不满足条件为止，在你的main()里创建并运行了一些线程，调用Thread类的start（）方法将为线程执行特殊的初始化的过程，来配置线程，然后由线程执行机制调用run（）。如果你不调用start（）线程就不会启动。

因为线程调度机制的行为是不确定的，所以每次运行该程序都会有不同的结果，你可以把你的循环次数增多些，然后看看执行的结果，你会发现main（）的线程和Thread1是交替运行的。
4.还有就是尽管线程的调度顺序是不固定的，但是如果有很多线程被阻塞等待运行，调度程序将会让优先级高的线程先执行，而优先级低的线程执行的频率会低一些。

### Java当中的锁机制有哪些，synchronize的作用核心和实现是什么

![img](https://img-blog.csdnimg.cn/20181122101753671.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2F4aWFvYm9nZQ==,size_16,color_FFFFFF,t_70)

#### 1. 乐观锁 VS 悲观锁

对于同一个数据的并发操作，悲观锁认为自己在使用数据的时候一定有别的线程来修改数据，因此在获取数据的时候会先加锁，确保数据不会被别的线程修改。Java中，synchronized关键字和Lock的实现类都是悲观锁。而乐观锁认为自己在使用数据时不会有别的线程修改数据，所以不会添加锁，只是在更新数据的时候去判断之前有没有别的线程更新了这个数据。如果这个数据没有被更新，当前线程将自己修改的数据成功写入。如果数据已经被其他线程更新，则根据不同的实现方式执行不同的操作（例如报错或者自动重试）。



乐观锁在Java中是通过使用无锁编程来实现，最常采用的是CAS算法，Java原子类中的递增操作就通过CAS自旋实现的。

![img](https://img-blog.csdnimg.cn/20181122101819836.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2F4aWFvYm9nZQ==,size_16,color_FFFFFF,t_70)

根据从上面的概念描述我们可以发现：

- 悲观锁适合写操作多的场景，先加锁可以保证写操作时数据正确。
- 乐观锁适合读操作多的场景，不加锁的特点能够使其读操作的性能大幅提升。
- ![img](https://img-blog.csdnimg.cn/20181122101946394.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2F4aWFvYm9nZQ==,size_16,color_FFFFFF,t_70)

悲观锁基本都是在显式的锁定之后再操作同步资源，而乐观锁则直接去操作同步资源

CAS全称 Compare And Swap（比较与交换），是一种无锁算法。在不使用锁（没有线程被阻塞）的情况下实现多线程之间的变量同步。java.util.concurrent包中的原子类就是通过CAS来实现了乐观锁。

CAS算法涉及到三个操作数：

- 需要读写的内存值 V。
- 进行比较的值 A。
- 要写入的新值 B。

当且仅当 V 的值等于 A 时，CAS通过原子方式用新值B来更新V的值（“比较+更新”整体是一个原子操作），否则不会执行任何操作。一般情况下，“更新”是一个不断重试的操作。

之前提到java.util.concurrent包中的原子类，就是通过CAS来实现了乐观锁，那么我们进入原子类AtomicInteger的源码，看一下AtomicInteger的定义：

![img](https://img-blog.csdnimg.cn/20181122102030461.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2F4aWFvYm9nZQ==,size_16,color_FFFFFF,t_70)

根据定义我们可以看出各属性的作用：

- unsafe： 获取并操作内存的数据。
- valueOffset： 存储value在AtomicInteger中的偏移量。
- value： 存储AtomicInteger的int值，该属性需要借助volatile关键字保证其在线程间是可见的。

CAS虽然很高效，但是它也存在三大问题，这里也简单说一下：

1. ABA问题。CAS需要在操作值的时候检查内存值是否发生变化，没有发生变化才会更新内存值。但是如果内存值原来是A，后来变成了B，然后又变成了A，那么CAS进行检查时会发现值没有发生变化，但是实际上是有变化的。ABA问题的解决思路就是在变量前面添加版本号，每次变量更新的时候都把版本号加一，这样变化过程就从“A－B－A”变成了“1A－2B－3A”。

JDK从1.5开始提供了AtomicStampedReference类来解决ABA问题，具体操作封装在compareAndSet()中。compareAndSet()首先检查当前引用和当前标志与预期引用和预期标志是否相等，如果都相等，则以原子方式将引用值和标志的值设置为给定的更新值。

2. 循环时间长开销大。CAS操作如果长时间不成功，会导致其一直自旋，给CPU带来非常大的开销。

3. 只能保证一个共享变量的原子操作。对一个共享变量执行操作时，CAS能够保证原子操作，但是对多个共享变量操作时，CAS是无法保证操作的原子性的。

Java从1.5开始JDK提供了AtomicReference类来保证引用对象之间的原子性，可以把多个变量放在一个对象里来进行CAS操作。

#### 2. 自旋锁 VS 适应性自旋锁





#### 3. 无锁 VS 偏向锁 VS 轻量级锁 VS 重量级锁

#### 4. 公平锁 VS 非公平锁

#### 5. 可重入锁 VS 非可重入锁

#### 6. 独享锁 VS 共享锁



### 线程当中有什么方法

线程相关的方法有：wait、notify、notifyAll、sleep、join、yieId等







### 上下文如何切换，

利用了时间片轮转的方式，CPU给每个任务一定的时间，然后把当前任务的状态保存下来，在加载下一个任务的状态后，继续服务下一个任务，任务的状态保存及再加载，这个过程就叫做上下文切换时间片轮转使多个任务在一颗CPU上执行变成了可能

引起上下文切换的原因：

1、当执行的任务时间片用完以后，系统CPU正常调度下一个任务

2、当前执行任务碰到IO阻塞，调度器将此任务挂起，继续下一个任务

3、多任务抢占锁资源，当前任务没有抢到锁资源，被调度器挂起，继续下一个任务

4、用户代码挂起当前任务，让出CPU时间

5、硬件中断



### CAS 

CAS（Compare And Swap/Set） ：比较并交换，

CAS算法的过程：它包含三个参数CAS(V,E,N)，V表示要更新的变量，E表示预期值（旧的），N表示新值，当且仅当V值等于E值时，才会将V值设为N，如果V值和E值不同，则说明在其他线程做了更新，最后，CAS返回当前V的真实值。

CAS的操作是抱着乐观的态度进行的（乐观锁），他总是认为自己可以完成操作。当多个线程同时使用CAS操作一个变量时，只有一个会胜出，并成功更新，其余均会失败。失败的线程不会被挂起，仅是被告知失败，并且允许再次尝试，当然也允许失败的线程放弃操作。基于这样的原理，CAS即使没有锁，也会发现其他的线程对当前线程的干扰，并进行恰当的处理。

### AQS

AQS（AbstractQueuedSynchronizer）：抽象队列同步器，抽象的队列式的同步器。

AQS定义了一套多线程访问共享资源的同步框架，许多同步类的实现都依赖于他。比如常用的ReentrantLock/Semaphore/CountDownLatch。

AQS 定义两种资源共享方式 Exclusive 独占资源-ReentrantLock Exclusive（独占，只有一个线程能执行，如 ReentrantLock） Share 共享资源-Semaphore/CountDownLatch Share（共享，多个线程可同时执行，如 Semaphore/CountDownLatch）。 AQS 只是一个框架，具体资源的获取/释放方式交由自定义同步器去实现。



### 异常有哪些，怎么分类的，

#### Error和Exception

Error：是指Java运行时系统内部错误和资源耗尽错误，应用程序不会抛出该类对象，

Exception：有两个分支，运行时异常（RuntimeException）和检查异常（CheckedException）

RuntimeException：如NullPointerException、ClassCastException等，是那些在Java虚拟机运行期间抛出的异常类

CheckedException：IOException、SQLException，一般是外部错误，经常要求try/catch，该类异常一般包括在：

​		1、文件尾部读取数据时

​		2、打开错误格式的URL

​		3、根据给定的字符串查找class对象，但是这个字符串表示的对象不存在

异常的处理方式：不进行处理，抛出交给调用者（throw和throws）

抛出异常有三种形式：throw、throws、系统自动抛出异常

```java
public class Test01 {
    public static void main(String[] args) {
        String s="abc";
        if (s.equals("abc")){
            throw new NumberFormatException();
        }else {
            System.out.println(s);
        }
        
    }
    int div(int a,int b)throws Exception{
        return a/b;
    }
}
```

#### throw和throws的区别：

##### 位置不同：

throw常用在函数上，后面跟的是异常类，可以跟多个，而throws用在函数内，后面跟的是异常对象

##### 功能不同：

​	1、throws 用来声明异常，让调用者只知道该功能可能出现的问题，可以给出预先的处理方 式；throw 抛出具体的问题对象，执行到 throw，功能就已经结束了，跳转到调用者，并 将具体的问题对象抛给调用者。也就是说 throw 语句独立存在时，下面不要定义其他语 句，因为执行不到

​	2、throws 表示出现异常的一种可能性，并不一定会发生这些异常；throw 则是抛出了异常， 执行 throw 则一定抛出了某种异常对象

​	3、两者都是消极处理异常的方式，只是抛出或者可能抛出异常，但是不会由函数去处理异 常，真正的处理异常由函数的上层调用处理。

### 反射

在 Java 中的反射机制是指在运行状态中，对于任意一个类都能够知道这个类所有的属性和方法； 并且对于任意一个对象，都能够调用它的任意一个方法；这种动态获取信息以及动态调用对象方 法的功能成为 Java 语言的反射机制

#### 优点：

 运行期类型的判断，动态加载类，提高代码灵活度。

#### 缺点： 

性能瓶颈：反射相当于一系列解释操作，通知 JVM 要做的事情，性能 比直接的java代码要慢很多。

#### 反射机制的应用场景有哪些？

举例：①我们在使用JDBC连接数据库时使用Class.forName()通过反射加载数据库的驱动程序； ②Spring框架也用到很多反射机制， 经典的就是xml的配置模式。Spring 通过 XML 配置模式装载 Bean 的过程：1) 将程序内所有 XML 或 Properties 配置文件加载入内存中; 2)Java类里面解析xml或 properties里面的内容，得到对应实体类的字节码字符串以及相关的属性信息; 3)使用反射机制，根据这 个字符串获得某个类的Class实例; 4)动态配置实例的属

#### Java获取反射的三种方法

 1.通过new对象实现反射机制

 2.通过路径实现反射机制 

3.通过类名实现反射机制

```java
public class Student {
 private int id;
 String name;
 protected boolean sex;
 public float score;
 }
 public class Get {
 //获取反射机制三种方式
 public static void main(String[] args) throws ClassNotFoundException {
 //方式一(通过建立对象)
 Student stu = new Student();
 Class classobj1 = stu.getClass();
 System.out.println(classobj1.getName());
 //方式二（所在通过路径-相对路径）
 Class classobj2 = Class.forName("fanshe.Student");
 System.out.println(classobj2.getName());
 //方式三（通过类名）
 Class classobj3 = Student.class;
 System.out.println(classobj3.getName());
 }
```

### 注解

Annatation(注解)是一个接口，程序可以通过反射来获取指定程序中元素的 Annotation 对象，然后通过该 Annotation 对象来获取注解中的元数据信息。

### 内部类

##### 定义在类内部的静态类，就是静态内部类

1. 静态内部类可以访问外部类所有的静态变量和方法，即使是 private 的也一样。
2. 静态内部类和一般类一致，可以定义静态变量、方法，构造方法等。
3. 其它类使用静态内部类需要使用“外部类.静态内部类”方式，如下所示：Out.Inner inner =  new Out.Inner();inner.print(); 
4. Java集合类HashMap内部就有一个静态内部类Entry。Entry是HashMap存放元素的抽象， HashMap 内部维护 Entry 数组用了存放元素，但是 Entry 对使用者是透明的。像这种和外部 类关系密切的，且不依赖外部类实例的，都可以使用静态内部类。

##### 成员内部类

定义在类内部的非静态类，就是成员内部类。成员内部类不能定义静态方法和变量（final 修饰的 除外）。这是因为成员内部类是非静态的，类初始化的时候先初始化静态成员，如果允许成员内 部类定义静态变量，那么成员内部类的静态变量初始化顺序是有歧义的

```java
 public class Out {
 private static int a;
 private int b;
 public class Inner {
 public void print() {
 System.out.println(a);
 System.out.println(b);
 }
 }
}
```

##### 局部内部类（定义在方法中的类）

定义在方法中的类，就是局部类。如果一个类只在某个方法中使用，则可以考虑使用局部类。

```java
public class Out {
 private static int a;
 private int b;
 public void test(final int c) {
 final int d = 1;
 class Inner {
 public void print() {
 System.out.println(c);
 }
 }
 }
}
```

##### 匿名内部类（要继承一个父类或者实现一个接口、直接使用 new 来生成一个对象的引用)

匿名内部类我们必须要继承一个父类或者实现一个接口，当然也仅能只继承一个父类或者实现一 个接口。同时它也是没有 class 关键字，这是因为匿名内部类是直接使用 new 来生成一个对象的引 用

```java
 public abstract class Bird {
 private String name;
 public String getName() {
 return name;
 }
 public void setName(String name) {
 this.name = name;
 }
 public abstract int fly();
}
public class Test {
 public void test(Bird bird){
 System.out.println(bird.getName() + "能够飞 " + bird.fly() + "米");
 }
 public static void main(String[] args) {
 Test test = new Test();
 test.test(new Bird() {
 public int fly() {
 return 10000;
 }
 public String getName() {
 return "大雁";
 }
 });
 }
}
```



### 泛型

泛型的本 质是参数化类型，也就是说所操作的数据类型被指定为一个参数。

泛型方法（<E>)

```java
 // 泛型方法 printArray 
 public static < E > void printArray( E[] inputArray )
 { 
 for ( E element : inputArray ){ 
 System.out.printf( "%s ", element );
 }
 }
```

1. <? extends T>表示该通配符所代表的类型是 T 类型的子类。 
2. <? super T>表示该通配符所代表的类型是 T 类型的父类

泛型类（T）

泛型类的声明和非泛型类的声明类似，除了在类名后面添加了类型参数声明部分。和泛型方法一样，泛 型类的类型参数声明部分也包含一个或多个类型参数，参数间用逗号隔开。一个泛型参数，也被称为一 个类型变量，是用于指定一个泛型类型名称的标识符。因为他们接受一个或多个参数，这些类被称为参 数化的类或参数化的类型。

```java
public class Box<T> {
private T t;
public void add(T t) {
this.t = t;
}
public T get() {
return t;
}
```



类型通配符? 类 型 通 配 符 一 般 是 使 用 ? 代 替 具 体 的 类 型 参 数 。 例 如 List 在 逻 辑 上 是 List,List 等所有 List<具体类型实参>的父类。



### 序列化(创建可复用的 Java 对象)

使用 Java 对象序列化，在保存对象时，会把其状态保存为一组字节，在未来，再将这些字节组装成对 象。必须注意地是，对象序列化保存的是对象的”状态”，即它的成员变量。由此可知，对象序列化不会 关注类中的静态变量。

Serializable 实现序列化 在 Java 中，只要一个类实现了 java.io.Serializable 接口，那么它就可以被序列化。

##### 通过ObjectOutputStream 和ObjectInputStream 对对象进行序列化及反序列化

##### 在类中增加 writeObject 和 readObject 方法可以实现自定义序列化策略。

序列化并不保存静态变量序列化子父类说明 要想将父类对象也序列化，就需要让父类也实现 Serializable 接口。

Transient 关键字阻止该变量被序列化到文件中 

1. 在变量声明前加上 Transient 关键字，可以阻止该变量被序列化到文件中，在被反序列化后， transient 变量的值被设为初始值，如 int 型的是 0，对象型的是 null。
2. 服务器端给客户端发送序列化对象数据，对象中有一些数据是敏感的，比如密码字符串等，希望对 该密码字段在序列化时，进行加密，而客户端如果拥有解密的密钥，只有在客户端进行反序列化 时，才可以对密码进行读取，这样可以一定程度保证序列化对象的数据安全。



# JavaWeb

### cookie和session分别是干什么的

**Cookie通过在客户端记录信息确定用户身份**，**Session通过在服务器端记录信息确定用户身份**。

#### Cookie：

HTTP协议是无状态的协议。一旦数据交换完毕，客户端与服务器端的连接就会关闭，再次交换数据需要建立新的连接。这就意味着服务器无法从连接上跟踪会话。，**给客户端们颁发一个通行证吧，每人一个，无论谁访问都必须携带自己通行证。这样服务器就能从通行证上确认客户身份了。这就是Cookie的工作原理**。

通过**request.getCookie()获取客户端提交的所有Cookie**（以Cookie[]数组形式返回），**通过response.addCookie(Cookiecookie)向客户端设置Cookie。**

**Cookie具有不可跨域名性**

Cookie在客户端是由浏览器来管理的。

**中文属于Unicode字符，在内存中占4个字符，而英文属于ASCII字符，内存中只占2个字节**。Cookie中使用Unicode字符时需要对Unicode字符进行编码，否则会乱码。Cookie中保存中文只能编码。一般使用UTF-8编码即可。不推荐使用GBK等中文编码，因为浏览器不一定支持，而且JavaScript也不支持GBK编码。

Cookie的maxAge决定着Cookie的有效期，单位为秒（Second）。Cookie中通过getMaxAge()方法与setMaxAge(int maxAge)方法来读写maxAge属性。

如果maxAge属性为正数，则表示该Cookie会在maxAge秒之后自动失效。浏览器会将maxAge为正数的Cookie持久化，即写到对应的Cookie文件中。无论客户关闭了浏览器还是电脑，只要还在maxAge秒之前，登录网站时该Cookie仍然有效。

如果maxAge为负数，则表示该Cookie仅在本浏览器窗口以及本窗口打开的子窗口内有效，关闭窗口后该Cookie即失效。maxAge为负数的Cookie，为临时性Cookie，不会被持久化，不会被写到Cookie文件中。Cookie信息保存在浏览器内存中，因此关闭浏览器该Cookie就消失了。Cookie默认的maxAge值为–1。

如果maxAge为0，则表示删除该Cookie。Cookie机制没有提供删除Cookie的方法，因此通过设置该Cookie即时失效实现删除Cookie的效果。失效的Cookie会被浏览器从Cookie文件或者内存中删除，

response对象提供的Cookie操作方法只有一个添加操作add(Cookie cookie)。

要想修改Cookie只能使用一个同名的Cookie来覆盖原来的Cookie，达到修改的目的。删除时只需要把maxAge修改为0即可。

####  **案例：永久登录**

实现方法是**把登录信息如账号、密码等保存在Cookie中，并控制Cookie的有效期，下次访问时再验证Cookie中的登录信息即可。**

保存登录信息有多种方案。最直接的是把用户名与密码都保持到Cookie中，下次访问时检查Cookie中的用户名与密码，与数据库比较。这是**一种比较危险的选择，一般不把密码等重要信息保存到Cookie中**。

　　还有**一种方案是把密码加密后保存到Cookie中，下次访问时解密并与数据库比较**。这种方案略微安全一些。如果不希望保存密码，还可以把登录的时间戳保存到Cookie与数据库中，到时只验证用户名与登录时间戳就可以了。

```jsp
<%@ page language="java"pageEncoding="UTF-8" isErrorPage="false" %>

<%!                                                  // JSP方法

    private static final String KEY =":cookie@helloweenvsfei.com";
                                                     // 密钥 

    public final static String calcMD1(Stringss) { // MD1 加密算法

       String s = ss==null ?"" : ss;                  // 若为null返回空

       char hexDigits[] = { '0','1', '2', '3', '4', '1', '6', '7', '8', '9',
       'a', 'b', 'c', 'd', 'e', 'f' };                        // 字典

       try {

        byte[] strTemp =s.getBytes();                          // 获取字节

        MessageDigestmdTemp = MessageDigest.getInstance("MD1"); // 获取MD1

       mdTemp.update(strTemp);                                // 更新数据

        byte[] md =mdTemp.digest();                        // 加密

        int j =md.length;                                 // 加密后的长度

        char str[] = newchar[j * 2];                       // 新字符串数组

        int k =0;                                         // 计数器k

        for (int i = 0; i< j; i++) {                       // 循环输出

         byte byte0 =md[i];

         str[k++] =hexDigits[byte0 >>> 4 & 0xf];

         str[k++] =hexDigits[byte0 & 0xf];

        }

        return newString(str);                             // 加密后字符串

       } catch (Exception e){return null; }

    }

%>

<%

   request.setCharacterEncoding("UTF-8");             // 设置request编码

    response.setCharacterEncoding("UTF-8");        // 设置response编码

   

    String action =request.getParameter("action"); // 获取action参数

   

    if("login".equals(action)){                       // 如果为login动作

        String account =request.getParameter("account");                    // 获取account参数
        String password =request.getParameter("password");// 获取password参数
        int timeout = newInteger(request.getParameter("timeout"));

        String ssid =calcMD1(account + KEY); // 把账号、密钥使用MD1加密后保存

        CookieaccountCookie = new Cookie("account", account);     // 新建Cookie
       accountCookie.setMaxAge(timeout);              // 设置有效期
        Cookie ssidCookie =new Cookie("ssid", ssid);   // 新建Cookie
       ssidCookie.setMaxAge(timeout);                 // 设置有效期
       response.addCookie(accountCookie);             // 输出到客户端
       response.addCookie(ssidCookie);            // 输出到客户端

        // 重新请求本页面，参数中带有时间戳，禁止浏览器缓存页面内容

       response.sendRedirect(request.getRequestURI() + "?" + System.
        currentTimeMillis());
        return;
    }

    elseif("logout".equals(action)){                  // 如果为logout动作

        CookieaccountCookie = new Cookie("account", "");
                                                 // 新建Cookie，内容为空
       accountCookie.setMaxAge(0);                // 设置有效期为0，删除
    Cookie ssidCookie =new Cookie("ssid", ""); // 新建Cookie，内容为空
       ssidCookie.setMaxAge(0);                   // 设置有效期为0，删除
       response.addCookie(accountCookie);         // 输出到客户端
       response.addCookie(ssidCookie);         // 输出到客户端
        //重新请求本页面，参数中带有时间戳，禁止浏览器缓存页面内容
       response.sendRedirect(request.getRequestURI() + "?" + System.
        currentTimeMillis());
        return;
    }
    boolean login = false;                        // 是否登录
    String account = null;                        // 账号
    String ssid = null;                           // SSID标识
   
    if(request.getCookies() !=null){               // 如果Cookie不为空

        for(Cookie cookie :request.getCookies()){  // 遍历Cookie

           if(cookie.getName().equals("account"))  // 如果Cookie名为account
               account = cookie.getValue();       // 保存account内容
           if(cookie.getName().equals("ssid")) // 如果为SSID
               ssid = cookie.getValue();          // 保存SSID内容
        }
    }

    if(account != null && ssid !=null){    // 如果account、SSID都不为空

        login =ssid.equals(calcMD1(account + KEY));
                                      // 如果加密规则正确, 则视为已经登录
    }

%>

<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01Transitional//EN">

       <legend><%= login ? "欢迎您回来" : "请先登录"%></legend>

        <% if(login){%>

            欢迎您, ${ cookie.account.value }.   

           <a href="${ pageContext.request.requestURI }?action=logout">
            注销</a>

        <% } else {%>

        <formaction="${ pageContext.request.requestURI }?action=login"
        method="post">
           <table>
               <tr><td>账号： </td>

                   <td><input type="text"name="account" style="width:
                   200px; "></td>

               </tr>

               <tr><td>密码： </td>

                   <td><inputtype="password" name="password"></td>

               </tr>

               <tr>

                   <td>有效期： </td>

                   <td><inputtype="radio" name="timeout" value="-1"
                   checked> 关闭浏览器即失效 <br/> <input type="radio" 
                   name="timeout" value="<%= 30 *24 * 60 * 60 %>"> 30天
                   内有效 <br/><input type="radio" name="timeout" value= 
                   "<%= Integer.MAX_VALUE %>"> 永久有效 <br/> </td> </tr>

               <tr><td></td>

                   <td><input type="submit"value=" 登  录 " class= 
                   "button"></td>

               </tr>

           </table>

        </form>

        <% } %>
```

#### Session

**Session是服务器端使用的一种记录客户端状态的机制**，使用上比Cookie简单一些，相应的也**增加了服务器的存储压力**。

**Cookie机制是通过检查客户身上的“通行证”来确定客户身份的话，那么Session机制就是通过检查服务器上的“客户明细表”来确认客户身份。Session相当于程序在服务器上建立的一份客户档案，客户来访的时候只需要查询客户档案表就可以了。**

Session对应的类为javax.servlet.http.HttpSession类。每个来访者对应一个Session对象，所有该客户的状态信息都保存在这个Session对象里。**Session对象是在客户端第一次请求服务器的时候创建的**。Session也是一种key-value的属性对，通过getAttribute(Stringkey)和setAttribute(String key，Objectvalue)方法读写客户状态信息。Servlet里通过request.getSession()方法获取该客户的Session，

Servlet中必须使用request来编程式获取HttpSession对象，而JSP中内置了Session隐藏对象，可以直接使用。如果使用声明了<%@page session="false" %>，则Session隐藏对象不可用。

Session保存在服务器端。**为了获得更高的存取速度，服务器一般把Session放在内存里。每个用户都会有一个独立的Session。如果Session内容过于复杂，当大量客户访问服务器时可能会导致内存溢出。因此，Session里的信息应该尽量精简。**

　　**Session在用户第一次访问服务器的时候自动创建**。需要注意只有访问JSP、Servlet等程序时才会创建Session，只访问HTML、IMAGE等静态资源并不会创建Session。如果尚未生成Session，也可以使用request.getSession(true)强制生成Session。

**Session生成后，只要用户继续访问，服务器就会更新Session的最后访问时间，并维护该Session**。用户每访问服务器一次，无论是否读写Session，服务器都认为该用户的Session“活跃（active）”了一次。

Session需要使用Cookie作为识别标志。HTTP协议是无状态的，Session不能依据HTTP连接来判断是否为同一客户，因此服务器向客户端浏览器发送一个名为JSESSIONID的Cookie，它的值为该Session的id（也就是HttpSession.getId()的返回值）。Session依据该Cookie来识别是否为同一用户。

#### Cookie和Session的区别：

**1、cookie数据存放在客户的浏览器上，session数据放在服务器上.**

**2、cookie不是很安全，别人可以分析存放在本地的COOKIE并进行COOKIE欺骗考虑到安全应当使用session。**

**3、设置cookie时间可以使cookie过期。但是使用session-destory（），我们将会销毁会话。**

**4、session会在一定时间内保存在服务器上。当访问增多，会比较占用你服务器的性能考虑到减轻服务器性能方面，应当使用cookie。**

**5、单个cookie保存的数据不能超过4K，很多浏览器都限制一个站点最多保存20个cookie。(Session对象没有对存储的数据量的限制，其中可以保存更为复杂的数据类型)**

#### 转发和重定向的区别

##### **1、请求次数不同；**

重定向是浏览器向服务器发送一个请求并收到响应后再次向一个新地址发出请求，转发是服务器收到请求后为了完成响应跳转到一个新的地址；重定向至少请求两次，转发请求一次；



##### **2、重定向时地址栏会发生变化，而转发时地址栏不会发生变化；**

重定向可以跳转到任意URL，转发只能跳转本站点资源；

##### **3、重定向两次请求不共享数据，转发一次请求共享数据。**

##### 4、重定向是客户端行为，转发是服务器端行为；



### jsp 的四种作用域和九种内置对象，

JSP的四大作用域：page、request、session、application

九大内置对象，page、config、appliction、request、response、session、out、exception、pageContext



# Spring部分

### spring的特点，

轻量级

控制反转

通过控制反转(IOC)促进了低耦合，当应用了IOC，一个对象依赖的其他对象会通过被动的方式传递过来，而不是这个对象自己传递或者查找依赖对象





面向切面

支持面向切面的编程，并且把应用业务逻辑和系统服务分开



容器

Sping包含并且管理应用对象配置和生命周期，在这个意义上他就是个容器，你可以配置你的每个bean如何被创建，————给予一个可配置的原型，你的bean可以创建一个单独的实例或者每次需要时生成一个新的实例，以及他们是如何相互关联的





框架集合

Spring可以将简单的组件配置组合成复杂的应用，在Spring中，应用对象被声明式的组合，典型的是在一个XML文件里，SPring提供了很多基础功能（事务管理，持久化框架等），将应用逻辑的开发留给开发者



### 核心组件有哪些，

![](C:/Users/16143/Desktop/%E7%AC%94%E8%AE%B0/note-main/pic/image-20220119110916097.png)



Spring常用模块：







### 常用注解



### ioc的原理

Spring 通过一个配置文件描述 Bean 及 Bean 之间的依赖关系，利用 Java 语言的反射功能实例化 Bean 并建立 Bean 之间的依赖关系。 Spring 的 IoC 容器在完成这些底层工作的基础上，还提供 了 Bean 实例缓存、生命周期管理、 Bean 实例代理、事件发布、资源装载等高级服务

**Spring 启动时读取应用程序提供的 Bean 配置信息，并在 Spring 容器中生成一份相应的 Bean 配 置注册表，然后根据这张注册表实例化 Bean，装配好 Bean 之间的依赖关系，为上层应用提供准 备就绪的运行环境。其中 Bean 缓存池为 HashMap 实现**

![Spring IOC概论(Spring 容器高层视图)插图2](https://pic.imgdb.cn/item/61e126252ab3f51d91365bd8.png)

Java对象的生命舟周期：实例化=====》GC垃圾回收

### springbean的作用周期

- 实例化 Instantiation
- 属性赋值 Populate
- 初始化 Initialization
- 销毁 Destruction



实例化：实例化一个Bean，也就是常说的new

IOC依赖注入：按照上下文对实例化的Bean进行配置，也就是IOC注入

setBeanName实现：如果这个bean实现了BeanNameAware接口，那么会调用它实现的setBeanName(String)方法，此处传递的就是Spring配置文件中的Bean的ID的值

BeanFactoryAware实现：如果这个Bean实现了BeanFactoryAware接口，那么会调用它实现的setBeanFactory(BeanFactory)传递的是Spring工厂自身（可以用这个方式来获取其它 Bean， 只需在 Spring 配置文件中配置一个普通的 Bean 就可以）

ApplicationContextAware实现：如果这个 Bean 已经实现了 ApplicationContextAware 接口，会调用 setApplicationContext(ApplicationContext)方法，传入 Spring 上下文（同样这个方式也 可以实现步骤 4 的内容，但比 4 更好，因为 ApplicationContext 是 BeanFactory 的子接 口，有更多的实现方法）

postProcessBeforeInitialization接口实现-初始化预处理：如果这个 Bean 关联了 BeanPostProcessor 接口，将会调用 postProcessBeforeInitialization(Object obj, String s)方法，BeanPostProcessor 经常被用 作是 Bean 内容的更改，并且由于这个是在 Bean 初始化结束时调用那个的方法，也可以被应 用于内存或缓存技术。

init-method：如果 Bean 在 Spring 配置文件中配置了 init-method 属性会自动调用其配置的初始化方法。

postProcessAfterInitialization：如果这个 Bean 关联了 BeanPostProcessor 接口，将会调用 postProcessAfterInitialization(Object obj, String s)方法。 注：以上工作完成以后就可以应用这个 Bean 了，那这个 Bean 是一个 Singleton 的，所以一 般情况下我们调用同一个 id 的 Bean 会是在内容地址相同的实例，当然在 Spring 配置文件中 也可以配置非 Singleton。

Destroy 过期自动清理阶： Bean 不再需要时，会经过清理阶段，如果 Bean 实现了 DisposableBean 这个接口，会调 用那个其实现的 destroy()方法。

destroy-method 自配置清理：，如果这个 Bean 的 Spring 配置中配置了 destroy-method 属性，会自动调用其配置的 销毁方法









### 作用域



### spring aop又是什么



### spring MVC的工作流程



### springboot 的原理



### mybatis有几个缓存区



### 微服务的组件：网关，注册中心，事件调度跟踪熔断分别是哪些组件



### netty和rpc有什么联系



### 网络的七层架构和五层架构



### tcp/ip的原理



### tcp为什么需要三次握手



### http的原理和状态



### 日志框架 log4j



### zookeeper有那些角色，投票机制什么



### 消息队列 

#### 为什么使用MQ？MQ的优点

简答：

异步处理 - 相比于传统的串行、并行方式，提高了系统吞吐量。

应用解耦 - 系统间通过消息通信，不用关心其他系统的处理。 

流量削锋 - 可以通过消息队列长度控制请求量；可以缓解短时间内的高并发请 求。 

日志处理 - 解决大量日志传输。 

消息通讯 - 消息队列一般都内置了高效的通信机制，因此也可以用在纯的消息通 讯。比如实现点对点消息队列，或者聊天室等。

详答：

主要是：解耦、异步、削峰。

解耦：A 系统发送数据到 BCD 三个系统，通过接口调用发送。如果 E 系统也要 这个数据呢？那如果 C 系统现在不需要了呢？A 系统负责人几乎崩溃…A 系统 跟其它各种乱七八糟 的系统严重耦合，A 系统产生一条比较关键的数据，很多系 统都需要 A 系统将这个数据发送过来。如果使用 MQ，A 系统产生一条数据， 发送到 MQ 里面去，哪个系统需要数据 自己去 MQ 里面消费。如果新系统需要 数据，直接从 MQ 里消费即可；如果某个系统不需要这条数据了，就取消对 MQ 消息的消费即可。这样下来，A 系统压根儿不需要去考 虑要给谁发送数 据，不需要维护这个代码，也不需要考虑人家是否调用成功、失败超时等情况。 就是一个系统或者一个模块，调用了多个系统或者模块，互相之间的调用很复 杂，维护起来很麻烦。但是其实这个调用是不需要直接同步调用接口的，如果用 MQ 给它异步化解耦。

异步：A 系统接收一个请求，需要在自己本地写库，还需要在 BCD 三个系统写 库，自己本地写库要 3ms，BCD 三个系统分别写库要 300ms、450ms、 200ms。最终请求总延 时是 3 + 300 + 450 + 200 = 953ms，接近 1s，用户 感觉搞个什么东西，慢死了慢死了。用户通过浏览器发起请求。如果使用 MQ，那么 A 系统连续发送 3 条消息到 MQ 队列 中，假如耗时 5ms，A 系统从 接受一个请求到返回响应给用户，总时长是 3 + 5 = 8ms。

削峰：减少高峰时期对服务器压力。

他还支持集群化、高可用部署架构、消息高可靠支持，功能较为完善。

缺点有以下几个： 

**系统可用性降低：**本来系统运行好好的，现在你非要加入个消息队列进去，那消息队列挂了，你的 系统不是呵呵了。因此，系统可用性会降低； 

**系统复杂度提高：**加入了消息队列，要多考虑很多方面的问题，比如：一致性问题、如何保证消息 不被重复消费、如何保证消息可靠性传输等。因此，需要考虑的东西更多，复杂 性增大。 

**一致性问题：**A 系统处理完了直接返回成功了，人都以为你这个请求就成功了；但是问题是， 要是 BCD 三个系统那里，BD 两个系统写库成功了，结果 C 系统写库失败了， 咋整？你这数据就 不一致了。



### rabbit MQ的几种工作模式，怎么保证消息的可靠性，怎么保证消息不被重复消费，消息的有序性，

Rabbit MQ架构：



Message：消息，消息是不具名的，它由消息头和消息体组成。消息体是不透明的，而消息头则由一系 列的可选属性组成，这些属性包括 routing-key（路由键）、priority（相对于其他消息的优 先权）、delivery-mode（指出该消息可能需要持久性存储）等

Publisher：消息的生产者，也是一个向交换器发布消息的客户端应用程序。

Exchange：用来接收生产者发送的消息并将这些消息路由给服务器中的队列。

Binding：绑定，用于消息队列和交换器之间的关联。

Queue：消息队列，用来保存消息直到发送给消费者。它是消息的容器，也是消息的终点。一个消息 可投入一个或多个队列。消息一直在队列里面，等待消费者连接到这个队列将其取走。

Connection：网络连接，比如一个 TCP 连接

Channel：信道，多路复用连接中的一条独立的双向数据流通道。

Consumer：消息的消费者，表示一个从消息队列中取得消息的客户端应用程序。

Virtual Host：虚拟主机是共享相同的身份认证和加密 环境的独立服务器域

Broker：表示消息队列服务器实体



Exchange 类型：

Exchange 分发消息时根据类型的不同分发策略有区别，目前共四种类型：direct、fanout、 topic、headers 。headers 匹配 AMQP 消息的 header 而不是路由键，此外 headers 交换器和 direct 交换器完全一致，但性能差很多，目前几乎用不到了，所以直接看另外三种类型：









### kafka的设计理念





![image-20220119141101147](C:/Users/16143/Desktop/%E7%AC%94%E8%AE%B0/note-main/pic/image-20220119141101147.png)





Kafka、ActiveMQ.RabbitMQ、RocketMQ有什么优缺点?

|             |                           ActiveMQ                           |                          RabbitMQ                           |                           RocketMQ                           |                            Kafka                             |        zeroMQ        |
| ----------- | :----------------------------------------------------------: | :---------------------------------------------------------: | :----------------------------------------------------------: | :----------------------------------------------------------: | :------------------: |
| 单机吞吐    |                         比RabbitMQ低                         |                   2.6w/s （消息做持久化)                    |                           11.6w/s                            |                           17.3wls                            |        29w/s         |
| 开发语言    |                             Java                             |                           Erlang                            |                             Java                             |                          ScalalJava                          |          C           |
| 主要维护    |                            Apache                            |                       Mozilla/Spring                        |                           Alibaba                            |                            Apache                            |       iMatix创       |
| 成熟度      |                             成熟                             |                            成熟                             |                       开源版本不够成熟                       |                           比较成熟                           | 只有C、PHP等版本成熟 |
| 订阅形式    |                   点对点(p2p)、广播（发布                    | 提供了4种: direct,topic,Headers和fanout。fanout就是广播模式 | 基于topic/me ssageTag 以及按照消息类型、属性进行正则匹配的发布订阅模式 |       基于topic以及按照topic进行正则匹配的发布订阅模式       |     点对点(P2P)      |
| 持久化      |                        支持少量 堆积                         |                        支持少量 堆积                        |                        支持大量 堆积                         |                        支持大量 堆积                         |        不支持        |
| 顺序消息    |                            不支持                            |                           不支持                            |                             支持                             |                             支持                             |        不支持        |
| 性能稳定 性 |                              好                              |                             好                              |                             一般                             |                             较差                             |         很好         |
| 集群方式    | 支持简单 集群模 式，比 如’主备’，对 高级集群 模式 支持 不好。 |  支持简单 集群，'复 制’模 式， 对高 级集群模 式支持不 好。  | 常用 多 对’Mast erSlave’ 模 式，开源 版本需手 动切换 Slave变成 Master | 天然 的‘Lead erSlave’无 状态集 群，每台 服务器既 是Master 也是Slave |        不支持        |
| 管理界面    |                             一般                             |                            较好                             |                             一般                             |                              无                              |          无          |

中小型公司，技术实力较为一般，技术挑战不是特别高，用 RabbitMQ 是 不错的选择；大型公司，基础架构研发实力较强，用 RocketMQ 是很好的选 择。 如果是大数据领域的实时计算、日志采集等场景，用 Kafka 是业内标准的，

### 数据量过大数据库存不下，hbase的核心概念是什么，有哪些核心的架构，写逻辑



### mangoDB 适合在什么场景下使用



### 设计模式：

[23 种设计模式详解（全23种）_人生智慧的博客-CSDN博客_设计模式](https://blog.csdn.net/A1342772/article/details/91349142)

总体来说设计模式分为三大类：

创建型模式，共五种：工厂方法模式、抽象工厂模式、单例模式、建造者模式、原型模式。

结构型模式，共七种：适配器模式、装饰器模式、代理模式、外观模式、桥接模式、组合模式、享元模式。

行为型模式，共十一种：策略模式、模板方法模式、观察者模式、迭代子模式、责任链模式、命令模式、备忘录模式、状态模式、访问者模式、中介者模式、解释器模式。







### 负载均衡做过哪些，有哪些策略与算法

负载均衡 建立在现有网络结构之上，它提供了一种廉价有效透明的方法扩展网络设备和服务器的带 宽、增加吞吐量、加强网络数据处理能力、提高网络的灵活性和可用性。

##### 四层负载均衡（目标地址和端口交换）

主要通过报文中的目标地址和端口，再加上负载均衡设备设置的服务器选择方式，决定最终选择 的内部服务器。 以常见的 TCP 为例，负载均衡设备在接收到第一个来自客户端的 SYN 请求时，即通过上述方式选 择一个最佳的服务器，并对报文中目标 IP 地址进行修改(改为后端服务器 IP），直接转发给该服务 器。TCP 的连接建立，即三次握手是客户端和服务器直接建立的，负载均衡设备只是起到一个类 似路由器的转发动作。

##### 实现四层负载均衡的软件有：

F5：硬件负载均衡器，功能很好，但是成本很高。 

lvs：重量级的四层负载软件。 

nginx：轻量级的四层负载软件，带缓存功能，正则表达式较灵活。 

haproxy：模拟四层转发，较灵活。

##### 七层负载均衡（内容交换）

所谓七层负载均衡，也称为“内容交换”，也就是主要通过报文中的真正有意义的应用层内容， 再加上负载均衡设备设置的服务器选择方式，决定最终选择的内部服务器

七层应用负载的好处，是使得整个网络更智能化。例如访问一个网站的用户流量，可以通过七层 的方式，将对图片类的请求转发到特定的图片服务器并可以使用缓存技术；将对文字类的请求可 以转发到特定的文字服务器并可以使用压缩技术。

##### 实现七层负载均衡的软件有:

haproxy：天生负载均衡技能，全面支持七层代理，会话保持，标记，路径转移； 

nginx：只在 http 协议和 mail 协议上功能比较好，性能与 haproxy 差不多； 

apache：功能较差 

Mysql proxy：功能尚可。

#### 负载均衡算法/策略

##### 轮循均衡（Round Robin）

每一次来自网络的请求轮流分配给内部中的服务器，从 1 至 N 然后重新开始。此种均衡算法适合 于服务器组中的所有服务器都有相同的软硬件配置并且平均服务请求相对均衡的情况。

##### 权重轮循均衡（Weighted Round Robin）

根据服务器的不同处理能力，给每个服务器分配不同的权值，使其能够接受相应权值数的服务请 求。例如：服务器 A 的权值被设计成 1，B 的权值是 3，C 的权值是 6，则服务器 A、B、C 将分 别接受到 10%、30％、60％的服务请求。此种均衡算法能确保高性能的服务器得到更多的使用 率，避免低性能的服务器负载过重

##### 随机均衡（Random）

把来自网络的请求随机分配给内部中的多个服务器

##### 权重随机均衡（Weighted Random）

此种均衡算法类似于权重轮循算法，不过在处理请求分担时是个随机选择的过程。

##### 响应速度均衡（Response Time 探测时间

**负载均衡设备对内部各服务器发出一个探测请求（例如 Ping），然后根据内部中各服务器对探测 请求的最快响应时间来决定哪一台服务器来响应客户端的服务请求。**

##### 最少连接数均衡（Least Connection）

最少连接数均衡算法对内部中需负载的每一台服务器都有一个数据记录，记录当前该服务器正在 处理的连接数量，当有新的服务连接请求时，将把当前请求分配给连接数最少的服务器，使均衡 更加符合实际情况，负载更加均衡。此种均衡算法适合长时处理的请求服务，如 FTP。

##### 处理能力均衡（CPU、内存）

此种均衡算法将把服务请求分配给内部中处理负荷（根据服务器 CPU 型号、CPU 数量、内存大小 及当前连接数等换算而成）最轻的服务器，由于考虑到了内部服务器的处理能力及当前网络运行 状况，所以此种均衡算法相对来说更加精确，尤其适合运用到第七层（应用层）负载均衡的情况 下

##### DNS 响应均衡（Flash DNS）

在此均衡算法下，分处在不同地理位置的负载均衡设备收到同一个客户端的域名解析请求，并在 同一时间内把此域名解析成各自相对应服务器的 IP 地址并返回给客户端，则客户端将以最先收到 的域名解析 IP 地址来继续请求服务，而忽略其它的 IP 地址响应。在种均衡策略适合应用在全局负 载均衡的情况下，对本地负载均衡是没有意义的

##### 哈希算法

一致性哈希一致性 Hash，相同参数的请求总是发到同一提供者。当某一台提供者挂时，原本发往 该提供者的请求，基于虚拟节点，平摊到其它提供者，不会引起剧烈变动。

##### IP 地址散列（保证客户端服务器对应关系稳定）

通过管理发送方 IP 和目的地 IP 地址的散列，将来自同一发送方的分组(或发送至同一目的地的分 组)统一转发到相同服务器的算法。当客户端有一系列业务需要处理而必须和一个服务器反复通信 时，该算法能够以流(会话)为单位，保证来自相同客户端的通信能够一直在同一服务器中进行处 理。

##### URL 散列

通过管理客户端请求 URL 信息的散列，将发送至相同 URL 的请求转发至同一服务器的算法。

### Lvs，keep live还有nginx的负载均衡有什么区别



# 数据库部分

### 索引是怎么构建的，如何失效

索引（Index）是帮助 MySQL 高效获取数据的数据结构。常见的查询算法,顺序查找,二分查找,二 叉排序树查找,哈希散列法,分块查找,平衡多路搜索树 B 树（B-tree）

常见的索引原则有：

1、选择唯一性索引

唯一性索引的值是唯一的，可以更快速的通过该索引来确定某条记录。

2、为经常需要排序、分组和联合操作的字段建立索引

3、为常作为查询条件的字段建立索引

4．限制索引的数目： 越多的索引，会使更新表变得很浪费时间。

5、尽量使用数据量少的索引：如果索引的值很长，那么查询的速度会受到影响。

6、尽量使用前缀来索引：如果索引字段的值很长，最好使用值的前缀来索引。

7、删除不再使用或者很少使用的索引 

8 、最左前缀匹配原则，非常重要的原则。 

9、尽量选择区分度高的列作为索引

10、索引列不能参与计算，保持列“干净”：带函数的查询不参与索引。 

11、尽量的扩展索引，不要新建索引

#### 三范式是什么，

##### 第一范式(1st NF －列都是不可再分)

第一范式的目标是确保每列的原子性:如果每列都是不可再分的最小数据单元（也称为最小的原子 单元），则满足第一范式（1NF）

##### 第二范式(2nd NF－每个表只描述一件事情)

首先满足第一范式，并且表中非主键列不存在对主键的部分依赖。 第二范式要求每个表只描述一 件事情。

##### 第三范式(3rd NF－ 不存在对非主键列的传递依赖)

第三范式定义是，满足第二范式，并且表中的列不存在对非主键列的传递依赖。除了主键订单编 号外，顾客姓名依赖于非主键顾客编号。

#### 数据库事务

事务必须具备以下四个属性，简称 ACID 属性：

##### 原子性（Atomicity）

事务是一个完整的操作。事务的各步操作是不可分的（原子的）；要么都执行，要么都不执 行。

##### 一致性（Consistency）

当事务完成时，数据必须处于一致状态。

##### 隔离性（Isolation） 

对数据进行修改的所有并发事务是彼此隔离的，这表明事务必须是独立的，它不应以任何方 式依赖于或影响其他事务。 

##### 永久性（Durability）

事务完成后，它对数据库的修改被永久保持，事务日志能够保持事务的永久性。



### 存储过程，

存储过程优化思路： 

1. 尽量利用一些 sql 语句来替代一些小循环，例如聚合函数，求平均函数等。 
2. 中间结果存放于临时表，加索引。
3. 少使用游标。sql 是个集合语言，对于集合运算具有较高性能。而 cursors 是过程运算。比如 对一个 100 万行的数据进行查询。游标需要读表 100 万次，而不使用游标则只需要少量几次 读取。 
4. 事务越短越好。sqlserver 支持并发操作。如果事务过多过长，或者隔离级别过高，都会造成 并发操作的阻塞，死锁。导致查询极慢，cpu 占用率极地。 5. 
5. 使用 try-catch 处理错误异常。 
6. 查找语句尽量不要放在循环内。

### 触发器

触发器是一段能自动执行的程序，是一种特殊的存储过程，触发器和普通的存储过程的区别是： 触发器是当对某一个表进行操作时触发。诸如：update、insert、delete 这些操作的时候，系统 会自动调用执行该表上对应的触发器。SQL Server 2005 中触发器可以分为两类：DML 触发器和 DDL 触发器，其中 DDL 触发器它们会影响多种数据定义语言语句而激发，这些语句有 create、 alter、drop 语句。

##### 数据库并发策略

**并发控制一般采用三种方法，分别是乐观锁和悲观锁以及时间戳。**

时间戳就是在数据库表中单独加一列时间戳，比如“TimeStamp”，每次读出来的时候，把该字 段也读出来，当写回去的时候，把该字段加1，提交之前 ，跟数据库的该字段比较一次，如果比数 据库的值大的话，就允许保存，否则不允许保存，这种处理方法虽然不使用数据库系统提供的锁 机制，但是这种方法可以大大提高数据库处理的并发量， 以上悲观锁所说的加“锁”，其实分为几种锁，分别是：排它锁（写锁）和共享锁（读锁）。

#### 数据库锁



##### 行级锁：

行级锁是一种排他锁，防止其他事务修改此行；在使用以下语句时，Oracle 会自动应用行级锁： 

1. INSERT、UPDATE、DELETE、SELECT … FOR UPDATE [OF columns] [WAIT n | NOWAIT]; 
2. SELECT … FOR UPDATE 语句允许用户一次锁定多条记录进行更新
3. 使用 COMMIT 或 ROLLBACK 语句释放锁

##### 表级锁 

表示对当前操作的整张表加锁，它实现简单，资源消耗较少，被大部分 MySQL 引擎支持。最常使 用的 MYISAM 与 INNODB 都支持表级锁定。表级锁定分为表共享读锁（共享锁）与表独占写锁 （排他锁）。

##### 页级锁

页级锁是 MySQL 中锁定粒度介于行级锁和表级锁中间的一种锁。表级锁速度快，但冲突多，行级 冲突少，但速度慢。所以取了折衷的页级，一次锁定相邻的一组记录。BDB 支持页级锁

#### 基于 Redis 分布式锁

1. 获取锁的时候，使用 setnx（SETNX key val：当且仅当 key 不存在时，set 一个 key 为 val 的字符串，返回 1；若 key 存在，则什么都不做，返回 0）加锁，锁的 value 值为一个随机生成的 UUID，在释放锁的时候进行判断。并使用 expire 命令为锁添 加一个超时时间，超过该时间则自动释放锁
2. 获取锁的时候调用 setnx，如果返回 0，则该锁正在被别人使用，返回 1 则成功获取 锁。 还设置一个获取的超时时间，若超过这个时间则放弃获取锁。
3. 释放锁的时候，通过 UUID 判断是不是该锁，若是该锁，则执行 delete 进行锁释放。

#### 分区分表

分库分表有垂直切分和水平切分两种。

##### 垂直切分(按照功能模块)

将表按照功能模块、关系密切程度划分出来，部署到不同的库上。例如，我们会建立定义数 据库 workDB、商品数据库 payDB、用户数据库 userDB、日志数据库 logDB 等，分别用于 存储项目数据定义表、商品定义表、用户数据表、日志数据表等。

##### 水平切分(按照规则划分存储

当一个表中的数据量过大时，我们可以把该表的数据按照某种规则，例如 userID 散列，进行 划分，然后存储到多个结构相同的表，和不同的库上

![image-20220119155926315](C:/Users/16143/Desktop/%E7%AC%94%E8%AE%B0/note-main/pic/image-20220119155926315.png)







#### CAP 

CAP 原则又称 CAP 定理，指的是在一个分布式系统中， Consistency（一致性）、 Availability （可用性）、Partition tolerance（分区容错性），三者不可得兼。 一致性（C）： 1. 在分布式系统中的所有数据备份，在同一时刻是否同样的值。（等同于所有节点访问同一份 最新的数据副本） 可用性（A）： 2. 在集群中一部分节点故障后，集群整体是否还能响应客户端的读写请求。（对数据更新具备 高可用性） 分区容忍性（P）： 3. 以实际效果而言，分区相当于对通信的时限要求。系统如果不能在时限内达成数据一致性， 就意味着发生了分区的情况，必须就当前操作在 C 和 A 之间做出选择





### 慢sql怎么定位，怎么解决，多少量级的数据可以考虑分库分表，依据是什么



### 一致性算法：paxos算法，







​                                                                                                                                                                                                                                                                                                                                

### 算法:冒泡，二分，



### 最短路径算法如何算的



### 数据结构有哪些



### 排序二叉树，红黑树，



### 加密算法，rsa



### 分布式缓存中缓存如何雪崩，缓存穿透如何解决，缓存预热是什么，缓存如何更新，如何降级，







