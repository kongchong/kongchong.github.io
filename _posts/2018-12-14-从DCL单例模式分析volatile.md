---
layout:     post
title:      从DCL单例模式分析volatile
subtitle:   程序员进阶不能不了解的关键字
date:       2018-12-14
author:     KC
header-img: img/post-bg-debug.png
catalog: true
tags:
    - 单例模式
    - DCL
    - volatile
    - 多线程
---

> 透过现象看本质

### 前言

提到java中的设计模式，大家想到最多的可能就是单例模式，但真正能够写好它的人估计并没想象中的那么多，所以这也是很多求职者在面试过程中会遇到的坑，今天就来分享一下单例模式的正确写法，同时能够大概了解java语言中voltaile关键字的作用

### 常见写法

以懒汉式为例，流传最多的是以下这种写法

![](http://www.kcblog.cn/img/2018-12-14/1.jpg)

大概流程是

1. 私有化构造函数
2. 对外提供获取内部实例方法
3. 方法内部判断成员实例是否为空，为空的话初始化实例对象赋值给成员变量，反之直接返回实例对象

#### 存在问题

这种写法直接在方法上加同步锁是十分消耗性能的，大部分场景下并不适用，相信很多人也意识到这个问题了，于是就有了下面的改进

![](http://www.kcblog.cn/img/2018-12-14/2.jpg)

#### 存在问题

这种写法基于上一种写法进行了优化，使用了DCL写法，即双重检查。在获取对象实例的时候进行了两次非空判断，第一个判断的主要作用是为了防止当前线程进入不必要的同步操作，设想一下多线程场景下当`instance` 不为空已经初始化完成的情况下时，多个线程同时执行到16行会直接跳过当前判断执行23行代码。如果没有第一个判断条件的话每个线程都要进入同步方法内去判断是否已经完成初始化在性能角度来讲是很大的性能损耗，所以这种DCL写法在性能上讲还是不错的。但它也存在问题，问题在于第19行`instance = new SingletonModel();` 是非原子性操作。

##### 实例化一个对象的大概步骤 

1. 为对象在jvm堆中分配内存空间
2. 初始化对象
3. 将堆中对象的地址赋值给对应引用

### JAVA内存模型之指令重排序

> 在计算机中，软件技术和硬件技术有一个共同的目标：在不改变程序执行结果的前提下，尽可能的开发并行度。

java为了实现这一目标在编译时会对代码进行重新排序，从而达到更高的并行度提升程序性能。Java 在进行重排序操作时会遵守**数据依赖性**，即编译器和处理器不会改变存在数据依赖关系的两个操作的执行顺序。如下几行代码

![](http://www.kcblog.cn/img/2018-12-14/3.jpg)

这里java语言有可能会对第28行和29行的代码进行重排序，由此会导致运行顺序不一致，但对最终的运行结果不会产生影响，但30行代码不会重排序到28行代码前执行，因为30行代码的操作依赖于28行代码的结果。

### 普通DCL单例写法存在的问题

由此可以预见，`instance = new SingletonModel();` 可能会重排序为

1. 为对象在jvm堆中分配内存空间

2. 将堆中对象的地址赋值给对应引用

3. 初始化对象



   这种情况在多线程环境下会有很大的问题，当A现场按照上述顺序执行完之后，B线程判断`instance` 是不为空的所以会直接返回变量所引用的堆中的内存地址，但这时这个对象有可能还未初始化完毕，那么有没有什么好的解决办法呢？这个时候volatile的重要性就凸显出来了。

### volatile关键字的作用

##### 保证被多个线程共享变量的可见性，即一个线程修改了共享变量的值，其他线程能够立马感知到并获取到最新的值

有的小伙伴可能会疑惑了，不加volatile关键字的变量在多线程环境下修改操作的话其他线程没法及时感知到吗？答案其实是不确定的，相信了解一些计算机硬件的朋友知道cpu参数那一栏都会标有L1、L2缓存有的高端点的也会有L3缓存，这些参数其实有一个统称叫做cpu高速缓存

那么什么是cpu高速缓存呢？

> 在计算机系统中，**CPU高速缓存**是用于减少处理器访问内存所需平均时间的部件。在金字塔式存储体系中它位于自顶向下的第二层，仅次于CPU寄存器。其容量远小于内存，但速度却可以接近处理器的频率。
>
> 当处理器发出内存访问请求时，会先查看缓存内是否有请求数据。如果存在（命中），则不经访问内存直接返回该数据；如果不存在（失效），则要先把内存中的相应数据载入缓存，再将其返回处理器。
>
> 缓存之所以有效，主要是因为程序运行时对内存的访问呈现局部性（Locality）特征。这种局部性既包括空间局部性（Spatial Locality），也包括时间局部性（Temporal Locality）。有效利用这种局部性，缓存可以达到极高的命中率。
>
> 在处理器看来，缓存是一个透明部件。因此，程序员通常无法直接干预对缓存的操作。但是，**确实可以根据缓存的特点对程序代码实施特定优化，从而更好地利用缓存**。

为什么需要cpu高速缓存？

>  随着工艺的提升，最近几十年 CPU 的频率不断提升，而受制于制造工艺和成本限制，目前计算机的内存在访问速度上没有质的突破。因此，CPU 的处理速度和内存的访问速度差距越来越大，甚至可以达到上万倍。这种情况下传统的 CPU 直连内存的方式显然就会因为内存访问的等待，导致计算资源大量闲置，降低 CPU 整体吞吐量。同时又由于内存数据访问的热点集中性，在 CPU 和内存之间用较为快速而成本较高（相对于内存）的介质做一层缓存，就显得性价比极高了。

了解了上面的硬件知识后相信大家应该也能猜到了，JAVA多线程环境下每个线程都有都有其对应的本地缓存，当一个线程要对变量进行修改操作时会将变量的值从RAM主内存先加载到本地缓存中，再在本地缓存中进行修改，最后再将本地缓存中的值同步到RAM主内存中，如果某个线程对变量进行了修改但没有及时将最新的值同步到主内存中这时其他变量拿到的还是修改前的值，就会产生数据不一致问题。将共享变量修饰为volatile后，多个线程对变量进行修改操作时会直接与主内存打交道，就不会产生数据不一致的问题了



##### 禁止指令重排序

通过对可见性的理解就可以推断出volatile关键字也能禁止指令重排序，保证一定的有序性，为什么这么说呢？

1. 当程序对volatile修饰的变量进行读或者写操作时，在其前面的操作的更改肯定全部已经进行，且结果已经对后面的操作可见；在其后面的操作肯定还没有进行；
2. 在进行指令优化时，不能将在对volatile变量访问的语句放在其后面执行，也不能把volatile变量后面的语句放到其前面执行。



所以最终比较完美的DCL单例的解决方案只需要对变量修饰为volatile就可以了

![](http://www.kcblog.cn/img/2018-12-14/4.jpg)

希望对你们能有帮助(*^__^*) 