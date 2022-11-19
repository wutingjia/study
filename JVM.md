# JVM



## 垃圾回收器

### G1垃圾回收器

G1(Garbage-First)是一款面向服务端应用的垃圾收集器,主要针对配备**多核CPU**及大容量内存的机器,以**极高概率满足GC停顿时间**的同时,还兼具**高吞吐量**的性能特征。

#### 内存划分

把内存分为2048个region，每块1~32M不等且都为2的N次幂（可设置为一样），他们再物理上**不连续**。

region分为Eden、Survivor、Old、Humongous四类。前三类的含义与其他垃圾回收器中的含义相同，Humongous主要存放大对象，当对象大小大于region的一半就使用Humongous进行存储。可以多个Humongous连续存储超大对象。

#### 垃圾回收过程

总体有两个阶段`young-gc`和`concurrent marking cycle`

1、yong-gc

此阶段会STW。顾名思义只会进行yongGC。

使用标记复制算法，将**多块**Eden region和survivor region中存活的对象使用**多线程**复制到**新的Survivor**或者**Old区**，然后清理原来的区域。这里的新的区域将会选择尽可能使内存连续的区域，块数也会比之前少，通过这样的方式完全内存碎片整理的过程。



二、concurrent marking cycle

如果yongGC后堆的总体使用量在40%以上就会进入`Space-Reclamation`阶段。

1. Initial Mark （STW）：下一次yonggc触发时，**顺道**进行。标记GCroot直接可达对象，然后标记剩下的Survivor region
2. Root Region Scanning：扫描上一步Survivor region(也就是Root Region)所引用的Old Region对象并标记。必须在下一次GC前完成否则会中断
3. Concurrent Marking ：递归的标记整堆上所有可达对象，还会扫描Remember Set，如果这时候发现整块可回收会直接进行回收。
4. Remark（STW）:：重新标记Concurrent Marking阶段产生的浮动垃圾，基于Snapshot-at-the-Beginning (SATB)算法
5. Cleaup（STW）：清理各个region，同时更新Rset。其中Old region中的对象据允许的收集时间,**优先回收价值最大的Region**

> GCRoot包括：虚拟机栈中引用的对象、本地方法栈引用的对象、类静态属性引用的对像、常量引用的对象
>
> Remember Set：每一块region都有一个对应的Remember Set记录了**引用了该region里对象的对象**
