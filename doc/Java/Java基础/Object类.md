## Object类



Object 类是 Java 中所有类的祖先，Object类没有定义属性，只有所有类都可以调用的方法。

### 1. clone 方法

```java
 protected native Object clone() throws CloneNotSupportedException;
```

保护方法，native 方法，创建并返回此对象的一个副本。

clone 方法实现的是浅拷贝，只拷贝当前对象，并且在堆中分配新的空间，放这个复制的对象，但是对象里有其他类的子对象，就不会拷贝到新对象中。

浅拷贝和深拷贝：

- 浅拷贝会创建一个新对象，如果属性是基本类型，拷贝的就是基本类型的值；如果属性是内存地址（引用类型），拷贝的就是内存地址 ，因此如果其中一个对象改变了这个地址，就会影响到另一个对象。
- 深拷贝会拷贝所有的属性,并拷贝属性指向的动态分配的内存。当对象和它所引用的对象一起拷贝时即发生深拷贝。深拷贝相比于浅拷贝速度较慢并且花销较大。

### 2. getClass 方法

```java
 public final native Class<?> getClass();
```

返回此 Object 对象的类对象，效果与 Object.class 相同。

### 3. equals 方法

```java
  public boolean equals(Object obj) {
         return (this == obj);
     }
```

== 对于基础类型，比较的是数值。引用类型比较的是对象指向内存的地址。

equals 表示的是对象的内容相同。通常需要自定义对象的 equals 方法。

### 4. hashCode 方法

```java
 public native int hashCode();
```

返回一个整型数值，表示该对象的哈希码值。

hashCode 约定：

- Java 程序运行期间，对同一对象多次调用 hashCode 方法，返回值是相同的。
- 如果两个对象相等，这两个对象的 hashCode 返回值也必须相等。
- 两个对象调用 hashCode 方法返回值相等，这两个对象不一定相等。

重写 equals 方法，必须也重写  hashCode 方法。因为 equals 相等的两个对象，hashCode 值也一定相等。但 hashCode 值相等的对象 equals 不一定相等。

### 5. toString 方法

```java
 public String toString() {
         return getClass().getName() + "@" + Integer.toHexString(hashCode());
 }
```

返回该对象的字符串表示。

getClass 返回对象的类对象，getName 以 String 形式返回类对象的名称。toHexString 以16进制无符号整数形式返回此哈希码的字符串表示形式。

同一类型但不相等的两个对象分别调用 toString 方法返回的结果可能相同。

### 6. 线程方法

wait：让当前线程等待。

notify：唤醒一个正在等待的线程。

notifyAll：唤醒所有正在等待的线程。

### 7. finalize 方法

```java
 protected void finalize() throws Throwable { }
```

finalize 方法主要与 Java 垃圾回收机制有关.