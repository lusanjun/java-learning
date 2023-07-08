## List



### 1. ArrayList

ArrayList 是基于数组实现的，支持随机访问，线程不安全，效率高。且实现了 RandomAccess 接口，支持快速随机访问。允许包括 null 在内的元素。增删慢，线程不安全。

默认初始容量为10，容量可自动增长。

ArrayList 继承于 AbstractList 抽象父类，实现了 List 接口、RandomAccess（可随机访问）、Cloneable（可拷贝）、Serializable（可序列化）。

#### 1.1 数据结构

底层是一个 object 数组，由 transient 修饰。

```java
 transient Object[] elementData;
```

ArrayList 底层数组不会参与序列化，而是用另外的序列化方式。

#### 1.2 增加元素

首先判断索引是否合法。

```java
 private void rangeCheckForAdd(int index) {
         if (index > size || index < 0)
             throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
     }
```

然后检查是否需要扩。

```java
 if ((s = size) == (elementData = this.elementData).length)
```

最后使用 System.arraycopy 方法完成数组的复制。

```java
 System.arraycopy(elementData, index,elementData, index + 1,s - index);
```

从指定源数组中复制一个数组。复制从指定位置开始，到目标数组的指定位置结束。

#### 1.3 删除元素

同样判断索引是否合法，把被删除元素右边的元素左移，使用 System.arraycopy 复制数组。

需要调用 System.arraycopy() 将 index+1 后面的元素都复制到 index 位置上，该操作的时间复杂度为 O(N)，ArrayList 删除元素的代价是非常高的。

```java
 public E remove(int index) {
         Objects.checkIndex(index, size);
         final Object[] es = elementData;
 
         @SuppressWarnings("unchecked") E oldValue = (E) es[index];
         fastRemove(es, index);
 
         return oldValue;
 }
 
 private void fastRemove(Object[] es, int i) {
         modCount++;
         final int newSize;
         if ((newSize = size - 1) > i)
             System.arraycopy(es, i + 1, es, i, newSize - i);
         es[size = newSize] = null;
 }
```

#### 1.4 修改元素

检查索引，进行修改。

```java
 public E set(int index, E element) {
         Objects.checkIndex(index, size);
         E oldValue = elementData(index);
         elementData[index] = element;
         return oldValue;
 }
```

#### 1.5 扩容

数组的默认大小为10。

添加元素时使用 ensureCapacityInternal 方法来保证容量足够，如果不够时，需要使用 grow 方法进行扩容，新容量的大小为 oldCapacity + (oldCapacity >> 1)，也就是旧容量的 1.5 倍。

扩容操作需要调用  Arrays.copyOf()  把原数组整个复制到新数组中，这个操作代价很高，因此最好在创建 ArrayList 对象时就指定大概的容量大小，减少扩容操作的次数。

```java
     private Object[] grow(int minCapacity) {
         int oldCapacity = elementData.length;
         if (oldCapacity > 0 || elementData != DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
             int newCapacity = ArraysSupport.newLength(oldCapacity,
                     minCapacity - oldCapacity, /* minimum growth */
                     oldCapacity >> 1           /* preferred growth */);
             return elementData = Arrays.copyOf(elementData, newCapacity);
         } else {
             return elementData = new Object[Math.max(DEFAULT_CAPACITY, minCapacity)];
         }
     }
```

大于1.5 倍可能会浪费更多的内存，1.5倍最多浪费33%。小于1.5倍，可能要频繁扩容，性能消耗严重。

#### 1.6 Fail-Fast

快速失败机制。

ArrayList 内部维护一个值 modCount ，用来记录 ArrayList 结构发生变化的次数，结构发生变化是指添加或者删除至少一个元素的所有操作，或者是调整内部数组的大小，仅仅只是设置元素的值不算结构发生变化。

在进行序列化或者迭代等操作时，需要比较操作前后 modCount 是否改变，如果改变了需要抛出 ConcurrentModificationException。

可以使用 Collections.synchronizedList()，得到一个线程安装的 ArrayList

```java
 List<String> list = new ArrayList<>();
 List<String> synList = Collections.synchronizedList(list);
```

也可以使用 concurrent 并发包下的 CopyOnWriteArrayList 类。

```java
 List<String> list = new CopyOnWriteArrayList<>();
```

删除操作，建议使用迭代器的 remove 方法而不是集合类的 remove 方法。

#### 1.7 阿里巴巴 Java 规范 ForEach

阿里巴巴 Java 规范，不要在 forEach 循环里进行元素的 remove / add 操作，remove 元素请使用 Iterator 方式，如果并发操作，需要对 Iterator 对象加锁。

原因：

ArrayList 有一个内部类 Itr，实现了 Iterator 接口，维护了一个变量 expectedModCount。

forEach 使用的是 while 和 Iterator 进行遍历。操作元素的是集合自己的 remove / add 方法，在操作时，modCount 变动了，但是 expectedModCount 没有改变。操作前检查这两个值不相等，就抛出了 ConcurrentModificationException 异常。

```java
     private void checkForComodification(final int expectedModCount) {
         if (modCount != expectedModCount) {
             throw new ConcurrentModificationException();
         }
     }
```

使用迭代器 Iterator 操作，modCount 会赋值给 expectedModCount。就不会报错。

```java
  public void remove() {
             if (lastRet < 0)
                 throw new IllegalStateException();
             checkForComodification();
 
             try {
                 ArrayList.this.remove(lastRet);
                 cursor = lastRet;
                 lastRet = -1;
                 expectedModCount = modCount;
             } catch (IndexOutOfBoundsException ex) {
                 throw new ConcurrentModificationException();
             }
         }
```

### 2. Vector

vector 实现与 ArrayList 类似，但是使用了 synchronized 进行同步，但线程安全，效率低。

#### 2.1 数据结构

底层数组不加 transient ，序列化时会全部复制。

```java
 protected Object[] elementData;
```

#### 2.2 增加元素

与 ArrayList 类似，但是使用了 synchronized 进行同步。

```java
     public synchronized void insertElementAt(E obj, int index) {
         if (index > elementCount) {
             throw new ArrayIndexOutOfBoundsException(index
                                                      + " > " + elementCount);
         }
         modCount++;
         final int s = elementCount;
         Object[] elementData = this.elementData;
         if (s == elementData.length)
             elementData = grow();
         System.arraycopy(elementData, index,
                          elementData, index + 1,
                          s - index);
         elementData[index] = obj;
         elementCount = s + 1;
     }
```

#### 2.3 删除元素

也使用了 synchronized。

```java
     public synchronized E remove(int index) {
         modCount++;
         if (index >= elementCount)
             throw new ArrayIndexOutOfBoundsException(index);
         E oldValue = elementData(index);
 
         int numMoved = elementCount - index - 1;
         if (numMoved > 0)
             System.arraycopy(elementData, index+1, elementData, index,
                              numMoved);
         elementData[--elementCount] = null; // Let gc do its work
 
         return oldValue;
     }
```

#### 2.4 扩容

Vector 的构造函数可以传入 capacityIncrement 参数，它的作用是在扩容时使容量 capacity 增长 capacityIncrement。如果这个参数的值小于等于 0，扩容时每次都令 capacity 为原来的两倍。

```java
     private Object[] grow(int minCapacity) {
         int oldCapacity = elementData.length;
         int newCapacity = ArraysSupport.newLength(oldCapacity,
                 minCapacity - oldCapacity, /* minimum growth */
                 capacityIncrement > 0 ? capacityIncrement : oldCapacity
                                            /* preferred growth */);
         return elementData = Arrays.copyOf(elementData, newCapacity);
     }
```

无参构造函数，capacityIncrement 值默认为0，默认每次扩容翻一倍。

```java
 public Vector(int initialCapacity) {
     this(initialCapacity, 0);
 }
 
 public Vector() {
     this(10);
 }
```

### 3. LinkedList

基于双向链表实现，使用Node存储链表节点信息，查询慢，增删快，线程不安全，效率高。还可做栈、队列和双向队列。

LinkedList 继承于 AbstractSequentialList ，实现了 List、Deque。

AbstractSequentialList 提供了 List 接口的骨干实现，减少了实现 List 接口的复杂度。Deque 支持两端插入和删除元素。

LinkedList 提供了3个基本属性 size、first 和 last

```java
     transient int size = 0;
     transient Node<E> first;
     transient Node<E> last;
 
     private static class Node<E> {
         E item;
         Node<E> next;
         Node<E> prev;
 
         Node(Node<E> prev, E element, Node<E> next) {
             this.item = element;
             this.next = next;
             this.prev = prev;
         }
     }
```

#### 3.1 增加元素

添加到链表开头：

```java
     private void linkFirst(E e) {
         final Node<E> f = first; //头节点
          //新建一个前驱为null，值为e，后继为原头结点的结点
         final Node<E> newNode = new Node<>(null, e, f);
         first = newNode; //更新头结点为新建结点
         if (f == null) //如果原头结点为null，说明链表为空链表
             last = newNode;
         else
             f.prev = newNode; //原头结点的前驱为新建结点
         size++;
         modCount++;
     }
```

添加到链表末尾：

```java
 void linkLast(E e) {
     final Node<E> l = last; //尾节点
     //新建一个前驱为原尾结点，值为e，后继为null的结点
     final Node<E> newNode = new Node<>(l, e, null);
     last = newNode; //更新尾结点为新建结点
     if (l == null) //尾结点为null说明链表为空链表
         first = newNode;
     else
         l.next = newNode; //原尾结点的后继为新建结点
     size++;
     modCount++;
 }
```

添加到节点 succ 前：

```java
 void linkBefore(E e, Node<E> succ) {
     // assert succ != null;
     final Node<E> pred = succ.prev; //得到succ的前驱
     //创建一个前驱为pred，值为e，后继为succ的结点
     final Node<E> newNode = new Node<>(pred, e, succ);
     succ.prev = newNode; //更新succ的前驱为新建结点
     if (pred == null) //前驱若为null，说明succ为头结点
         first = newNode;
     else
         pred.next = newNode; //更新succ前驱的后继为新建结点
     size++;
     modCount++;
 }
```

#### 3.2 删除元素

```java
 //删除第一个节点
 public E removeFirst() {
         final Node<E> f = first;
         if (f == null)
             throw new NoSuchElementException();
 }
 
 //删除最后一个节点
 public E removeLast() {
         final Node<E> l = last;
         if (l == null)
             throw new NoSuchElementException();
         return unlinkLast(l);
 }
```

#### 3.3 获取元素

查询元素，二分法，如果下标在前半部分，就开头寻找。如果在后半部分，从尾开始找。

循环次数降低一半，提高了查询性能。

```java
    Node<E> node(int index) {
        // assert isElementIndex(index);
        if (index < (size >> 1)) {
            Node<E> x = first;
            for (int i = 0; i < index; i++)
                x = x.next;
            return x;
        } else {
            Node<E> x = last;
            for (int i = size - 1; i > index; i--)
                x = x.prev;
            return x;
        }
    }
```