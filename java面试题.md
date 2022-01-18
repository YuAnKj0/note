java 



### **jvm中的内存区域**

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



### 序列化



### cookie和session分别是干什么的，转发和重定向的区别，



### jsp 的四种作用域和九种内置对象，



### spring的特点，核心组件有哪些，常用注解



### ioc的原理



### springbean的作用周期，作用域



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



### rabbit MQ的几种工作模式，怎么保证消息的可靠性，怎么保证消息不被重复消费，消息的有序性，



### kafka的设计理念



### 数据量过大数据库存不下，hbase的核心概念是什么，有哪些核心的架构，写逻辑



### mangoDB 适合在什么场景下使用



### 设计模式：



### 负载均衡做过哪些，有哪些策略与算法



### lvs,keep live还有nginx的负载均衡有什么区别



### 索引是怎么构建的，如何失效，三范式是什么，数据库事务，存储过程，触发器，



### 慢sql怎么定位，怎么解决，多少量级的数据可以考虑分库分表，依据是什么



### 一致性算法：paxos算法，



### 算法:冒泡，二分，



### 最短路径算法如何算的



### 数据结构有哪些



### 排序二叉树，红黑树，



### 加密算法，rsa



### 分布式缓存中缓存如何雪崩，缓存穿透如何解决，缓存预热是什么，缓存如何更新，如何降级，







