## ConcurrentHashMap

### 为什么hashMap线程不安全

* JDK1.7及以前hashMap的链表结构采用头插法，这在扩容的时候会导致链表死循环或数据丢失
* JDK1.8没有上述问题，但是当两个同时put的元素的key算出来的hash相同时，会数据覆盖(不会拖一个链表出来)导致数据丢失

### put操作

![未命名文件](https://wtjmarkdownpic.oss-cn-shanghai.aliyuncs.com/%E6%9C%AA%E5%91%BD%E5%90%8D%E6%96%87%E4%BB%B6.png)

`sizeCtl`

默认为0

当为-1时表示正在初始化

当在扩容时表示（正在扩容的线程 + 1)再乘以-1。即2个线程扩容表时，值为-3

当初始化完成后，表示将要扩容的阈值。例如当size为100时，负载因子0.75，则`sizeCtl`的值为75

## ThreadLocal

获取当前线程的`ThreadLocalMap`对象，`ThreadLocalMap`里有个`Entry`数组.

更据`threadLocal`对象hash取余定位具体的数组位置。

`Entry`数组本身只有一个value对象用于存值，继承`WeakReference`，再继承`Reference`。将ThreadLocal对象复制给其中的`referent`

### 为什么是weakReference

> 只具有弱引用的对象拥有更短暂的生命周期。在垃圾回收器线程扫描它所管辖的内存区域的过程中，一旦发现了只具有弱引用的对象，不管当前内存空间足够与否，都会回收它的内存。

**包装了weakReference不代表就只有弱引用！！**

`WeakReference` 对`referent` 的引用关系才是弱引用，但是这时候可以有其他类同时引用referent所引用的对象的，这时候这个对象就有强引用了

归根结底的原因是ThreadLocal实现的方式本质靠的是Thread，但是Thread在正常情况下是不会被回收的因此被它强引用的对象也都不会回收

A引用链条：

```
Thread -> ThreadLocal.ThreadLocalMap -> Entry[] -> Enrty -> key（threadLocal对象）和value
```

B引用链条：

```
 yourClass -> threadLocal
```

正常使用时，会有两个引用链，其中B是强引用。当B被回收时，理论上我们是想要threadLocal一起被回收的。但是如果threadLocal不是弱引用，那A引用链就无法回收了。因此通过设置其为弱引用，来保证Thread这条引用链不影响垃圾回收！





