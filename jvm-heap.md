# JVM-heap

## 堆的核心概述

​	一个JVM实例只存在一个堆内存，堆也是Java内存管理的核心区域

​	Java堆区在JVM启动的时候被创建，其空间大小也就确定了。是JVM管理的最大一块内存空间。（堆内存的大小是可以调节的） 

参数设置：-Xms10m -Xmx20m

​	《Java虚拟机规范》规定：堆可以处理物理上不连续的内存空间，但在逻辑上它应该被视为连续的。

《Java虚拟机规范》中对Java堆的描述是：所有的对象实例以及数组都应当在运行时分配在堆上。

数组和对象可能永远不会存储在栈上，因为栈帧中保存引用，这个引用的执行对象或数组在堆中的位置

在方法结束后，堆中的对象不会马上被移除，仅仅是在垃圾回收的时候才会被移除

堆是GC执行垃圾回收的重点区域

​	所有的线程共享Java堆，在这里还可以划分线程私有的缓冲区（Thread Local Allocation Buffer，TLAB）

### 内存细分

现代垃圾收集器大部分都基于分代收集理论设计，堆空间细分为：

​	java7之前逻辑上分为：新生区+养老区+永久区

- Yong Generation Space	新生区/新生代/年轻代  	New/Young

  ​		 	又被划分为Eden区和Survivor区	

- Tenure Generation Space	养老区 /老年区/老年代     Old/Tenure

-  Permanment  Space	永久区/永久代               Perm  

java8及之后逻辑上分为：新生区+养老区+元空间

- Yong Generation Space	新生区/新生代/年轻代  	New/Young

  ​		 		又被划分为Eden区和Survivor区

- Tenure Generation Space	养老区 /老年区/老年代   Old/Tenure

- Meta Space	元空间                       Meta

  -XX: +PrintGCDetails  打印堆空间细分空间名称

## 设置堆内存大小与OOM

参数设置(新生代+老年代)：-Xms10m(起始内存，等价于-XX:InitialHeapSize，默认为物理电脑可用内存的 1/64) -Xmx20m(最大内存，等价于-XX:MaxHeapSize，默认为 1/4)。

开发中建议将初始堆内存和最大的堆内存设置成相同的值，避免内存波动（GC后调整大小）影响性能。

​	-- 如何查查设置的参数 ： 1 jps / jstat -gc 进程id

​												2 -XX: +PrintGCDetails

一旦堆中的内存大小超过“—Xmx”所指定的最大内存时，将会抛出OutOfMemberyError



## 年轻代与老年代

存储在JVM中的对象，分为两类：

- 一类是生命周期较短的瞬时对象，这类对象的创建和消亡都非常迅速

- 另一类对象的生命周期却非常长，在某些极端的情况下还能够与JVM的生命周期保持一致

  JVM堆区进一步细分的话，可以划分为年轻代（YoungGen）和老年代(OldGen)，其中年轻代又可以划分为Eden空间和survivor0空间和survivor1空间（也叫from区和to区）

参数设置(一般不会调)：

- ​	默认	 -XX:NewRatio=2，表示新生代占1，老年代占2，新生代占整个堆的1/3

- ​	可以设置为 	-XX:NewRatio=4，表示新生代占1，老年代占4，新生代占整个堆的1/5

- ​	默认	-XX:SurvivorRatio=8，表示新生代中Eden占8，From占1，To占1

- ​    -xx:-UseAdaptiveSizePolicy 关闭自适应的内存分配策略（暂时用不到）

  在hotspot中，Eden空间和另外两个survivor空间的缺省所占的比例是8：1：1，可以通过参数调节，如：-XX:SurvivorRatio=8

  几乎所有的对象都是在Eden中被new出来的（太大放不下）

  绝大部分的Java对象的销毁都在新生代进行了

  - ​	IBM公司研究发现，新生代中80%对象都是”朝生夕死“

  可以使用参数选项”-Xmn“设置新生代最大内存大小

  - ​	这个参数一般使用默认值就可以

## 图解对象分配过程

为新对象分配内存是一件非常严谨和复杂的任务，jvm的设计者们不仅需要考虑内存如何分配和在哪儿分配等问题，并且由于内存分配算法与内存回收算法密切相关，所以还需要考虑GC执行之后是否会在内存空间中产生内存碎片

new的对象先放在伊甸园区。此区有大小限制

当伊甸园区空间填满时，程序又需要创建对象，JVM的垃圾回收器将对伊甸园区进行垃圾回收（Minor GC）,将伊甸园区中的不再被其他对象所引用的对象进行销毁，再加载新的对象放到伊甸园区

然后伊甸园区的剩余对象移动到幸存者0区

如果再次触发垃圾回收，此时上次幸存下来的方到幸存者0区，如果没有回收，就会放到幸存者1区

如果再次经历垃圾回收，此时会重新放回幸存者0区，接着再去幸存者1区

啥时候能去养老区呢？可以设置次数。默认是15次。 可以设置参数：

​		-XX:MaxTenuringThreshold=<N>进行设置‘

在养老区，相对悠闲。当养老区内存不足时，再次触发GC:Major GC进行养老区的内存清理

若养老区执行了Major GC之后依然无法对象的保存，就会发生OOM异常

总结：

​	针对幸存者s0,s1区总结：复制之后有交换，谁空谁是to

​	关于垃圾回收：频繁在新生区收集，甚少在养老区收集，几乎不在永久区/元空间收集

常用的调优工具：

- JDK命令行

- Eclipse： Memory Analyzer Tool

- Jconsole

- VisualVM

- Jprufuiler

- Java Flight Recorder

- GcViewer

- Gc Easy

## minor GC/ Major GC/ Full GC

JVM在进行GC时，并非每次都对上面三个内存区域一起回收的，大部分时候回收的都是指新生代

针对hotspot VM的实现，它里面的GC按照回收区域又分为两大种类型：一种是部分收集（Parial GC）,一种是整堆收集（Full GC）

​	部分收集：不是完全收集整个Java堆的垃圾收集。其中又分为：

- 新生代收集（Minor GC/Young GC）

- 老年代收集（Major GC/OldGC）

  ​				目前只有CMS GC会有单独收集老年代的行为，注意，很多时候Major GC会和Full GC混淆使用，需要具体分辨是老年代回收还是整堆回收

- 混合回收（Mixed GC）:收集整个新生代以及部分老年代的垃圾收集

  ​					目前只有G1 GC会有这种行为

​	整堆收集（Full GC）:收集整个java堆和方法区的垃圾收集

#### 最简单的分代式GC策略的触发条件

##### 年轻代GC(Minor GC)触发机制：

- 当年轻代空间不足时，就会触发Minor GC，这里的年轻代指的是Eden代满，survivor满不会触发Minor GC，（每次Minor GC会清理年轻代的内存）

- 因为Java对象大多都具备朝生息死的特性，Minor GC很频繁，速度也比较快，这一定义清晰又易理解

- Minor GC会引发STW,暂停其他用户的线程，等垃圾回收结束时，用户线程才会恢复运行

##### 老年代GC(Major GC/Full GC)触发机制：

- 指发生在老年代的GC,对象从老年代消失了

- 出现了Major GC，经常会伴随至少一次的Minor GC(但非绝对的，在’Parallel Scavenge 收集器的收集策略里就有直接进行Major GC的策略选择过程)
- Major GC的速度一般比Minor GC慢10倍以上，STW时间更长
- 如果Major GC后，内存还不足，就OOM

##### Full GC触发机制：

- 调用System.gc().系统建议执行Full GC,但是不必然执行

- 老年代空间不足

- 方法区空间不足

- 通过Minor GC 进入老年代的平均大小大于老年代的可用内存

- 由Eden区/survivor space 0(From Space)/区向survivor space 1(To Space)区复制时，对象大小大于 To Space 可用内存，则把该对象转存到老年代，且老年代的可用内存小于该对象大小

  说明：Full GC 是项目中调优尽量避免的，这样暂时时间会短一些。

## 堆的分代思想

优化GC性能

![image-20201120115005324](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20201120115005324.png)

## 内存分配策略（对象提升Promotion规则）

针对不同年龄段的对象分配原则如下所示

- 优先分配到Eden区
- 大对象直接分配到老年代
- 长期存活的对象分配到老年代
- 动态对象年龄判断：如果survivor区中的相同年龄的所有对象大小总和大于survivor区总空间一半，年龄大于等于该年龄的对象可以直接进入老年代，无需等待MaxTenuringThreshold中的年龄要求
- 空间分配担保 ： -XX:HandlePromotionFailure

## 为对象分配内存：TLAB

### 什么是TLAB

- 从内存模型而不是垃圾收集的角度，对Eden区域继续进行划分，JVM为每个线程分了一个独有的私有缓冲区，它包含在Eden内
- 多线程同时分配内存时，使用TLAB可以避免一系列的非线程安全问题，同时还能够提升内存分配的吞吐量，因此我们可以将这种内存分配方式称为**快速分配策略**
- 所有基于OpenJDK衍生出来的JVM都提供了TLAB的设计

### 说明：

- ​	尽管不是所有的对象实例都能在TLAB中成功分配内存，但JVM确实是将TLAB作为内存分配的首选
- ​	在程序中，开发人员可以通过”-XX:UseTLAB“设置是否开启TLAB空间
- ​	默认情况下，TLAB空间内存非常小，仅占整个Eden区的1%，当然我们可以通过选项”-XX:TLABWasteTargetPercent“设置TLAB空间所占Eden空间的百分比大小
- ​	一旦对象在TLAB空间分配内存失败时，JVM就会尝试着通过使用加锁机制确保数据操作的原子性，从而直接在Eden空间中分配内存。
- ![image-20201120132641431](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20201120132641431.png)

## 小结：堆空间参数设置

官网：

```
* 测试堆空间常用的jvm参数：
* -XX:+PrintFlagsInitial : 查看所有的参数的默认初始值
* -XX:+PrintFlagsFinal  ：查看所有的参数的最终值（可能会存在修改，不再是初始值）
*      具体查看某个参数的指令： jps：查看当前运行中的进程
*                             jinfo -flag SurvivorRatio 进程id
*
* -Xms：初始堆空间内存 （默认为物理内存的1/64）
* -Xmx：最大堆空间内存（默认为物理内存的1/4）
* -Xmn：设置新生代的大小。(初始值及最大值)
* -XX:NewRatio：配置新生代与老年代在堆结构的占比
* -XX:SurvivorRatio：设置新生代中Eden和S0/S1空间的比例
* -XX:MaxTenuringThreshold：设置新生代垃圾的最大年龄
* -XX:+PrintGCDetails：输出详细的GC处理日志
* 打印gc简要信息：① -XX:+PrintGC   ② -verbose:gc
* -XX:HandlePromotionFailure：是否设置空间分配担保
```

在发生Minor GC 之前，虚拟机会检查老年代最大可用的连续空间是否大于新生代所有对象的总空间”

​			如果大于，则此次Minor GC是安全的

​			如果小于，则虚拟机会查看-XX:HandkePromotionFailure设置值是否为允许担保失败

​					如果XX:HandkePromotionFailure=true,那么会继续检查老年代最大可用连续空间是否大于历次晋升到老年代的对象平均大小。如果大于，则尝试进行一次Minor GC，但这次Minor GC依然是有，风险。反之则改为进行一个Full GC

​					如果XX:HandkePromotionFailure=false,则改为进行一个Full GC

​	在JDK6 update24之后XX:HandkePromotionFailure参数不会再影响虚拟机的空间担保策略，改变为**只要老年代的连续空间大于新生代的对象总大小**或者**历次晋升平均大小**就会进行Minor GC，反之进行Full GC.

## 堆是分配对象的唯一选择吗？

在《深入理解Java虚拟机》中关于Java堆内存有这样一段描述：

随着JIT编译器的发展与逃逸分析技术逐渐成熟，栈上分配/标量替换优化技术将会导致一些微妙的变化，所有的对象都分配到堆上也渐渐变得不那么绝对了

在Java虚拟机中，对象是在Java堆中分配内存的，这是一个普遍的常识。但是有一种特殊情况，那就是如果经过**逃逸分析**（Escape Analysis）后发现，一个对象并没有逃逸出方法的话，那么就可能被优化成**栈上分配**。这样就无需在堆上分配内存，也无需进行垃圾回收。这也是常见的堆外存储技术

此外，基于OpenJDK深度定制的TaoBaoVM,GCIH(GC invisible heap)技术实现off-heap,将生命周期较长的对象从heap中移至heap外，并且GC不能管理CCIH内部的java对象，以此达到降低GC的回收频率和提升GC回收效率的目的。

### 逃逸分析概述

逃逸分析的基本行为就是分析对象动态作用域：

- 当一个对象在方法中被定义后，对象只在方法内部使用，则认为发生逃逸
- 当一个对象在方法中被定义后，它被外部方法所引用，则认为发生逃逸

#### 参数设置：

在JDK6update24版本之后，hotspot中默认开启逃逸分析

如果使用较早版本，开发测试人员可以

​	选项“-XX:+DoEscapeAnalysis”显示开启逃逸分析

​	选项“-XX:+PrintEscapeAnalysis”查看逃逸分析的筛选结果

#### 代码优化：

- ​	栈上分配：

  ​		逃逸分析后无逃逸现象的，无需在堆上分配内存，也无需进行垃圾回收。这也是常见的堆外存储技术。

- ​	同步省略（锁消除）

  ​		如果一个对象被发现只能从一个线程（线程同步的代价是相当高的->降低并发性和性能）被访问到，那么对于这个对象操作可以不考虑同步

  ​		在动态编译同步块的时候，JIT编译器可以借助逃逸分析来**判断同步代码块使用的锁对象是否能够被一个线程访问而没有被发布到其他线程**，如果没有，那么JIT编译器在编译这个同步代码块的时候就会取消对这部分代码的同步，这样就能够大大提高并发性和性能，这个取消同步的过程就叫同步省略，也叫**锁消除**

- ​	分离对象或标量替换

  ​		有的对象可能不需要作为一个连续的内存结构存在也可以被访问到，那么对象的部分（或全部）可以不存储在内存，而是存储在CPU寄存器中。

  ​		参数设置：-XX:+EliminateAllocations 开启标量替换（默认）

![image-20201120152841546](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20201120152841546.png)

#### 小结

逃逸分析并不成熟，根本原因是无法保证逃逸分析的性能消耗一定高于其他消耗，虽然经过逃逸分析可以做标量替换/栈上分配/锁消除，但是逃逸分析自身也是需要进行一系列复杂的分析的，这其实也是一个相对耗时的过程



## 本章小结

年轻代是对象的诞生/成长/消亡的区域，一个对象在这里产生/应用。最后被垃圾回收器收集/结束生命

老年代房子和长生命周期的对象，通常都是从Survivor区域筛选拷贝过来的Java对象。当然，也有特殊情况，我们知道普通的对象会被分配在TLAB上，如果对象较大，JVM会试图直接分配在Eden的其他位置上：如果对象较大，完全无法在新生代找到足够长的连续空闲空间，JVM就会直接分派到老年代

当GC只发生在年轻代中，回收年轻代对象的行为被称为Minor GC,当GC发生在老年代称Major GC 或者Full GC ，一般的Minor GC的发生频率要比Major GC 高很多，即老年代中垃圾回收发生的频率将大大低于年轻代。