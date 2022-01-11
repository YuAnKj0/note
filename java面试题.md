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

原理：避免了Thread类单继承的局限性，覆盖Runnable接口的run方法，将线程任务代码定义到run方法中，创建Thread类的对象，只有创建Thread类的对象才可以创建线程





#### 3.  使用Callable和Future创建线程







#### 4.  使用线程池创建(使用java.util,concurrent.Executor接口)











### 四种线程池分别是什么，线程的生命周期是什么，如何终止线程，sleep和wait的区别，start和run的区别，



### Java当中的锁机制有哪些，synchronize的作用核心和实现是什么



### 线程当中有什么方法，上下文如何切换，



### CAS AQS



### 异常有哪些，怎么分类的，





### 反射的好处是什么



### 注解，内部类，泛型，序列化



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







