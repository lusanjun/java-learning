## 原子类和CAS

### 1.Atomic包

JUC的atomic包中，提供了一系列用法简单、性能高效、线程安全的更新一个变量的类，都是原子类。

原子类保证共享变量操作的原子性、可见性，可以解决volatile原子性的缺陷。

原子类 AtomicInteger 主要API

```java
get() //直接返回值
getAndAdd(int) //增加指定的数据，返回变化前的数据
getAndDecrement() //减少1，返回减少前的数据
getAndIncrement() //增加1，返回增加前的数据
getAndSet(int) //设置指定的数据，返回设置前的数据
addAndGet(int) //增加指定的数据后返回增加后的数据
decrementAndGet() //减少1，返回减少后的值
incrementAndGet() //增加1，返回增加后的值
lazySet(int) //仅仅当get时才会set
compareAndSet(int, int)//尝试新增后对比，若增加成功则返回true否则返回false
```

atomic包中主要的原子类

- 基本类型：

  - AtomicInteger：整形原子类
  - AtomicLong：长整型原子类
  - AtomicBoolean：布尔型原子类
- 引用类型：

  - AtomicReference：引用类型原子类
  - AtomicStampedReference：原子更新引用类型里的字段原子类
  - AtomicMarkableReference ：原子更新带有标记位的引用类型
- 数组类型：
  - AtomicIntegerArray：整形数组原子类
  - AtomicLongArray：长整型数组原子类
  - AtomicReferenceArray：引用类型数组原子类
- 对象的属性修改类型：
  - AtomicIntegerFiledUpdater：原子更新整型字段的更新器
  - AtomicLongFiledUpdater：原子更新长整型字段的更新器
  - AtomicReferenceFiledUpdater：原子更新引用类型字段的更新器

### 2. CAS

#### 2.1 CAS

compare and swap 比较并交换。

CAS本质是一个方法调用了一行CPU原子指令

```java
执行方法：CAS(V,E,N)
```

- V：要读写的内存地址
- E：进行比较的值（预期值）
- N：拟写入的新值

当且仅当内存地址V中的值等于预期值E时，将内存V中的值改为N，否则会进行自旋操作，不断的重试。

#### 2.2 原理

CAS操作，都是通过unsafe类实现的，unsafe方法都是native方法。

以compareAndSwapInt方法为例。

```java
public final native boolean compareAndSwapInt(Object o, long offset, int expected, int x);
```

- Object o：需要改变的对象
- long offset：内存的偏移量，待更新的原值的准确内存地址
- int expecte：预期值
- int x：拟替换的新值

unsafe类源码上，Unsafe_CompareAndSwapInt 函数实现是使用了`cmpxchg`指令，这是一条汇编指令。

#### 2.3 缺陷

- 循环时间不可控：如果CAS一直不成功，CAS自旋就是死循环，会给CPU造成负担。
- 只能保证一个共享变量原子操作
- ABA问题：如果原来的值是A，然后变成B，又变成了A，那么在CAS检查时会发现没有改变。解决方法是加上版本号，每个变量绑定一个版本号，每次改变时加1，即 A - B - A，变成了 1A - 2B - 3A 。