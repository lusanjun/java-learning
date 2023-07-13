## ThreadLocal

### 1. ThreadLocal

ThreadLocal是线程本地变量类，是线程局部变量。为每一个使用该变量的线程提供一个变量值的副本，变量在每个线程中都有独立值，不会与其他线程的副本冲突。

场景：

- 解决线程安全问题：每个线程绑定一个数据库连接，避免多个线程访问同一个数据库连接：SqlSession
- 跨函数参数传递：同一个线程，跨类、跨方法传递参数时可以使用ThreadLocal，每个线程绑定一个Token/Session
- Spring使用ThreadLocal解决线程安全问题。一般只有无状态的Bean才可以在多线程中共享，Spring对一些Bean中用ThreadLocal进行处理，让它们成为线程安全的状态，有状态的Bean也可以在多线程中共享。

### 2. 底层原理

起作用的是Thread类：

```java
public class Thread implements Runnable {
    //当前线程的ThreadLocalMap，主要存储该线程自身的ThreadLocal
    ThreadLocal.ThreadLocalMap threadLocals = null;
}
```

当线程第一次调用ThreadLocal的set或get方法时，才会场景它。

set方法：

```java
public void set(T value) {
	Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null)
		map.set(this, value);
    else
        createMap(t, value);
}
```

获取一个和当前线程相关的ThreadLocalMap，然后将变量的值set到ThreadLocalMap对象中。

get方法：

```java
public T get() {
    //获得当前线程
    Thread t = Thread.currentThread();
    //每个线程 都有一个自己的ThreadLocalMap，
    //ThreadLocalMap里就保存着所有的ThreadLocal变量
    ThreadLocalMap map = getMap(t);
    if (map != null) {
        //ThreadLocalMap的key就是当前ThreadLocal对象实例，
        //多个ThreadLocal变量都是放在这个map中的
        ThreadLocalMap.Entry e = map.getEntry(this);
        if (e != null) {
            @SuppressWarnings("unchecked")
            //从map里取出来的值就是我们需要的这个ThreadLocal变量
            T result = (T)e.value;
            return result;
        }
    }
    // 如果map没有初始化，那么在这里初始化一下
    return setInitialValue();
}
```

### 3. ThreadLocalMap的key是弱引用

```java
static class Entry extends WeakReference<ThreadLocal<?>> {
    /** The value associated with this ThreadLocal. */
    Object value;
    //key就是一个弱引用
    Entry(ThreadLocal<?> k, Object v) {
        super(k);
        value = v;
    }
}
```

弱引用：被弱引用关联的对象只能生成到下一次垃圾收集发生为止。当垃圾收集器开始工作，无论内存是否足够，都会被回收。

如果是直接引用，当ThreadLocal实例对象被GC回收时，value对应的对象即使不再被使用，但由于被threadLocalMap引用而无法被GC回收，导致内存泄漏。

### 4. 内存泄漏

value的引用链条：

```
Thread -> ThreadLocalMap -> Entry -> value
```

虽然key是强引用，但只有Thread被GC回收时，value才会被回收，否则，只要线程不退出，value一直是强引用。但是要求线程退出，不太现实，大部分线程都一直存在程序的整个生命周期，value不被回收就会造成value内存泄漏。

处理方法：在ThreadLocalMap进行get、set、remove时，都进行清理。

```java
private Entry getEntry(ThreadLocal<?> key) {
    int i = key.threadLocalHashCode & (table.length - 1);
    Entry e = table[i];
    if (e != null && e.get() == key)
        //如果找到key，直接返回
        return e;
    else
        //如果找不到，就会尝试清理，如果总是访问存在的key，那么这个清理永远不会进来
        return getEntryAfterMiss(key, i, e);
}
```

ThreadLocal会检查key是否被回收，再回收value。但是如果固定访问几个一直存在的ThreadLocal，清理不会执行。

最好是不需要ThreadLocal时，主动调用remove。