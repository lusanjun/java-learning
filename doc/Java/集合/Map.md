## Map



### 1.map体系

![Map](https://gitee.com/lusanjun/blog-img/raw/master/img/Map.png)

### 2. HashMap

HashMap 是基于哈希表的 Map 接口的实现类。以 key-value 的形式存在。系统根据 hash 算法来确定元素的存储位置。

HashMap 实现了 Map 接口，继承于 AbstractMap，Map 接口定义了键值映射规则 ，AbstractMap 类提供了 Map 接口的骨干实现，减少了 HashMap 实现接口的工作。

HashMap 构造函数，需要初始容量（16）和加载因子（0.75）。

HashMap 是懒加载的，创建好 HashMap 对象后，并没有初始化，第一次 put 元素，才会初始化。

HashMap 支持 key 为 null。

HashMap 是非线程安全的。要求安全可以使用 ConcurrentHashMap。

#### 2.1 数据结构

![HashMap](https://gitee.com/lusanjun/blog-img/raw/master/img/HashMap.png)

在 JDK8 之前，内部包含了一个 Entry 类型的数组 table 。Entry 存储着键值对。数组的每个位置被当成一个桶，一个桶存放一个链表。极端情况下，复杂度从 O(1) 降低到 O(n) 。

在 JDK8 之后，结构为 Entry 数组+链表+红黑树。当节点树 >= 8，链表变为红黑树。极端情况下，复杂度从 O(1) 降低到 O(logn) 。

#### 2.2 增加元素

put 方法：

- 如果 HashMap 未被初始化，则初始化。
- 使用 hash 算法对 key 求 hash 值，计算下标。
- 如果 hash 值不相等，插入节点为空，直接放入Entry 数组。
- 如果 hash 值相等碰撞了：key 相同，直接覆盖原有的键值对。key 不同：
  - 如果节点是红黑树节点类型，把节点添加到树中；
  - 如果是链表，则循环查找进行尾部插入，如果节点数大于阈值，则转为红黑树。

#### 2.3 hash 算法（扰动函数）

```java
     static final int hash(Object key) {
         int h;
         return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
     }
```

直接使用 key 对象的 hashCode 值，得到一个int值，但是过大的int值，长度不合适。经过散列运算，得到一个较小的散列值，更很合适作为数组的下标索引。

h 无符号右移16位相当于高区16位移动到低区的16位，再与原 hashcode值做异或运算，可以将高低位二进制特征混合起来，就尽可能降低散列值碰撞。使用位运算，基于CPU，效率更高。

```java
 h=key.hashCode()    1111 1111 1111 1111 1111 0000 1110 1010
 h >>> 16            0000 0000 0000 0000 1111 1111 1111 1111
 
 h ^ (h >>> 16)      1111 1111 1111 1111 0000 1111 0001 0101
```

新的 hash值是为了参与 HashMap中计算数组下标 (n-1) & hash 这一步。如果不做异或运算，直接用：

```java
 h=key.hashCode()    1111 1111 1111 1111 1111 0000 1110 1010
 16-1                0000 0000 0000 0000 0000 0000 0000 1111
 
 (16-1) & hash       0000 0000 0000 0000 0000 0000 0000 1010
```

很明显丢失很多高位的特征，即使高位不参与运算，也能得到不同的 hash值，但保留高位差异碰撞几率更小。

使用异或，而不使用与运算，不使用或运算，能更好保留高区特征。使用与运算，结果更向0趋近；使用或运算，结果更向1趋近，都不能有更多差异参与计算。

JDK 7 的 hash 算法，只做了四次移位和四次异或，JDK 8 只做了一次，效率更高

```java
 // JDK 7
 static int hash(int h) {
       h ^= (h >>> 20) ^ (h >>> 12);
       return h ^ (h >>> 7) ^ (h >>> 4);
   }
```

#### 2.4 负载因子

负载因子（loadFactor ）。阈值= 负载因子 * 容量。HashMap 和 HashSet 都允许指定负载因子。当负载情况达到负载因子，容器自动扩容。默认为 0.75f ，即达到四分之三进行扩容，一般不建议修改。因为默认值是效率和容量之间相对比较平衡的。

#### 2.5 扩容

扩容（ resize 方法）发生在两个时间：

- 首次增加元素时，当 HashMap 初始化完成，put 第一个元素，会先调用 resize 方法首次扩容，默认为16。
- 容量达到阈值时，当容量达到负载因子乘以容量长度的阈值（threshold）时，进行扩容，扩充为原来容量的2倍。

```java
 newThr = oldThr << 1; // double threshold
```

#### 2.6 扩容总是2的次幂

初始化 HashMap 可以指定容量大小，但内部 tableSizeFor 方法会将容量转换成最接近其的2的次幂。

```java
 static final int tableSizeFor(int cap) {
     int n = cap - 1;
     n |= n >>> 1;
     n |= n >>> 2;
     n |= n >>> 4;
     n |= n >>> 8;
     n |= n >>> 16;
     return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
 }
```

n 是2的次幂，（n-1) 的二进制正好是都是1的形式，再和 hash 值做与运算，会让高位全部归零，只保留低位信息，方便参与 hash 运算。如果不是都是1的形式，例如 n=14，1110，和任意 h 与运算后，最后一位都是0，那么0001，0011，0111，1001，这些位置都不会存储数据，造成浪费。数据都拥挤在其他位置，提高了碰撞机会，降低效率。

### 3. HashTable

HashTable 继承自 Dictionary 类，实现了 Map 接口。Dictionary 类是各被废弃的抽象类

HashTable 容量可以为任意整数，默认容量为11，负载因子是 0.75f 。扩容后容量为原来的2倍 +1。

HashTable 是线程安全的，内部有 synchronized 关键字控制，效率较低，逐渐废弃。

HashTable采用"拉链法"实现哈希表。

#### 3.1 增加元素

put 方法：计算 key 的 hash 值，确定数组下标索引，迭代该 key 处的Entry节点链表，若该链表下存在一个同样的 key，就替换 value；若不相等，就插入该下标位置。

#### 3.2 扩容

扩容（rehash 方法），容量扩展为原容量的2倍+1。

```java
 int newCapacity = (oldCapacity << 1) + 1;
```

#### 3.3 HashMap 与 HashTable 比较

相同点：

都是存放键值对的对象（实现 Map），都是用散列 hash 算法计算元素索引。

不同点：

- 创建时间不同：HashTable 创建于 jdk1，HashMap 创建于 jdk2；
- 继承父类不同：HashTable 继承于 Dictionary 类，HashMap 继承于 AbstractMap 类；
- 对于 Null 不同：HashTable key和value都不支持 null，HashMap 可以存放 key 为 null 的键值对，hash值为0。
- 数据结构不同：HashTable 还是数组+链表，HashMap jdk8 后采用数组+链表+红黑树。
- 线程安全不同：HashTable 是线程安全的，每个方法都有 synchronized 修饰，HashMap 是不安全的。
- 初始容量和扩展容量不同：HashTable 初始容量为任意整数，默认11，扩展容量为 2倍+1。HashMap 初始容量为2的次幂，默认16，扩展容量为 2倍。

### 4. LinkedHashMap

HashMap 是无序的，迭代得到的元素顺序并不是存放的初始顺序。

LinkedHashMap 是有序的，维护了一个 Entry 的双向链表，迭代顺序即是插入顺序。

LinkedHashMap 继承于 HashMap，实现了 Map 接口。

#### 4.1 数据结构

LinkedHashMap = HashMap + 双向链表，在 HashMap的基础上，添加了before和after指针，记录链表前后顺序的元素。

```java
     static class Entry<K,V> extends HashMap.Node<K,V> {
         Entry<K,V> before, after;
         Entry(int hash, K key, V value, Node<K,V> next) {
             super(hash, key, value, next);
         }
     }
```

布尔值参数 accessOrder ，默认为 false：

- true，会把访问过的元素放在链表后面，遍历顺序是访问顺序。
- false，会按插入时的顺序来遍历。

#### 4.2 链表维护

**afterNodeRemoval**

```java
//节点删除后，维护链表，传入删除的节点  
void afterNodeRemoval(Node<K,V> e) { // unlink
    	//p 指向待删除的元素，b指向前驱，a指向后驱
        LinkedHashMap.Entry<K,V> p =
            (LinkedHashMap.Entry<K,V>)e, b = p.before, a = p.after;
    	//执行双向链表删除p节点  
    	p.before = p.after = null;
        if (b == null)
            head = a;
        else
            b.after = a;
        if (a == null)
            tail = b;
        else
            a.before = b;
    }
```

**afterNodeAccess**

```java
//在节点被访问后根据accessOrder判断是否需要调整链表顺序
void afterNodeAccess(Node<K,V> e) { // move node to last
    LinkedHashMap.Entry<K,V> last;  
    if (accessOrder && (last = tail) != e) {
        	//p指向待删除的元素，b指向前驱，a指向后驱
            LinkedHashMap.Entry<K,V> p =
                (LinkedHashMap.Entry<K,V>)e, b = p.before, a = p.after;
        	//执行双向链表删除p节点
        	p.after = null;
            if (b == null)
                head = a;
            else
                b.after = a;
            if (a != null)
                a.before = b;
            else
                last = b;
        	//将p放到末尾
            if (last == null)
                head = p;
            else {
                p.before = last;
                last.after = p;
            }
            tail = p;
            ++modCount;
        }
    }
```

**afterNodeInsertion**

```java
// 删除最早的元素
void afterNodeInsertion(boolean evict) { // possibly remove eldest
        LinkedHashMap.Entry<K,V> first;
        if (evict && (first = head) != null && removeEldestEntry(first)) {
            K key = first.key;
            removeNode(hash(key), key, null, false, true);
        }
}
// removeEldestEntry 默认返回false，一般会重写这个方法来使用
protected boolean removeEldestEntry(Map.Entry<K,V> eldest) {
        return false;
}
```

#### 4.3 LRU算法

LRU（Least Recently Used）缓存，最少最近使用，向缓存添加数据时，如果缓存已满，需要删除访问时间最早的数据。

```java
//设定最大缓存空间为3，可自定义
//LinkedHashMap 的构造函数将 accessOrder 设置为 true，开启 LRU 顺序
//覆盖 removeEldestEntry() 方法实现，在节点多于 MAX_ENTRIES 就会将最近最久未使用的数据移除。
public class LruCache<K,V> extends LinkedHashMap<K,V>{
    private static final int MAX_ENTRIES = 3;

    protected boolean removeEldestEntry(Map.Entry eldest) {
        return size() > MAX_ENTRIES;
    }

    LruCache() {
        super(MAX_ENTRIES, 0.75f, true);
    }
    public static void main(String[] args) {
        LruCache<Integer, String> cache = new LruCache<>();
        cache.put(1, "a");
        cache.put(2, "b");
        cache.put(3, "c");
        cache.get(1);
        cache.put(4, "d");
        System.out.println(cache.keySet());
    }
}
```



### 5. TreeMap

TreeMap 是一个基于 key 有序的 k-v 散列表。不是线程安全的。key 不可以存入 null。

map根据 key 的自然顺序排序，或根据创建时提供的 Comparator 排序。

如果仅用于存储数据，HashMap更好；如果需要排序或统计功能用 TreeMap。

#### 5.1 数据结构

底层是红黑树。

```java
//比较器
private final Comparator<? super K> comparator;
//根节点
private transient Entry<K,V> root;
//容器大小
private transient int size = 0;
//修改次数
private transient int modCount = 0;
```

#### 5.2 put方法

1. 获取根节点，根节点为空，产生一个根节点，将其着色为黑色，退出余下流程；
2. 获取比较器，如果传入的 Comparator 接口不为空，使用 Comparator 实现类进行比较；如果 Comparator 接口为空，将Key强转为 Comparator 接口进行比较；
3. 从根节点开始逐一依照规定的排序算法进行比较，取比较值 cmp，如果 cmp=0，表示插入的 Key 已存在；如果cmp>0，取当前节点的右子节点；如果 cmp<0，取当前节点的左子节点；
4. 排除插入的 Key 已存在的情况，第（3）步的比较一直比较到当前节点 t 的左子节点或右子节点为 null，此时 t 就是寻找到的节点，cmp>0 则准备往 t 的右子节点插入新节点，cmp<0 则准备往 t 的左子节点插入新节点；
5. new出一个新节点，默认为黑色，根据 cmp 的值向 t 的左边或者右边进行插入；
6. 插入之后进行修复，包括左旋、右旋、重新着色这些操作，让树保持平衡性；

#### 5.3 get方法

1. 当key大于当前节点，把当前节点指针指向左孩子继续循环。
2. 当key小于当前节点，把当前节点的指针指向右孩子继续循环。
3. 当key等于当前节点，则返回当前节点。

### 6. ConcurrentHashMap

ConcurrentHashMap 是一个支持高并发更新与查询的哈希表，基于 HashMap。

在保证安全的前提下，进行检索不需要锁定。与hashtable不同，该类不依赖于synchronization去保证线程操作的安全。

#### 6.1 数据结构

JDK7 中使用的是分段锁，每一个 Segment 上同时只有一个线程可以操作。冲突多了，会转化为链表。

JDK8 中同 HashMap ，转变为 Node 数组 + 链表/红黑树。

内部是 volatile 修饰的节点数组，保证可见性，禁止指令重排。

```java
transient volatile Node<K,V>[] table;

private transient volatile Node<K,V>[] nextTable;
```

#### 6.2 初始化

当数组为空时，通过对变量 sizeCtl 的值进行判断，如果小于0，说明另外的线程执行 CAS 成功，正在初始化，当前线程会主动让出 CPU。Thread.yield();如果大于0，会通过自旋和CAS操作实现数组的初始化。

#### 6.3 put方法

首先，put 方法没有 synchronized 修饰。

1. 如果 key 或 value 是空，抛出空指针异常；
2. 根据 key 计算 hash 值；
3. 判断 Node 数组有没有初始化，如果没有先通过自旋加 CAS 操作初始化数组；
4. 根据 hash 值判断数组位置是否为空，如果为空，即不存在冲突，利用 CAS 尝试写入，失败则自旋保证成功，这里不加锁；
5. 如果当前位置的 hash 值是MOVED = 1，表示正在扩容，帮助其扩容完成；
6. 如果上述条件都不满足，即 hash 值存在冲突，就先对链表的头节点或红黑树的头节点加 synchronized 锁，写入数据。
   - 链表，就遍历链表，hash 值相同，就覆盖原值，如果不同，就将元素插入到链表尾部。
   - 如果节点数量大于 TREEIFY_THRESHOLD，执行树化方法。
   - 红黑树，就按照红黑树的结构插入数据。

#### 6.4 get方法

get操作全程无锁，由于 Node 节点 val 和指针 next 都是 volatile 修饰，在多线程环境下线程 A 的修改对线程 B 是可见的。

1. 根据 key 计算 hash 值；
2. 查找指定位置，如果头节点就是要找的，直接返回 value；
3. 如果头节点 hash 值小于0，说明正在扩容或是红黑树，使用红黑树的查找方法进行查找；
4. 如果是链表节点，遍历链表进行查找。

#### 6.5 不支持key或value是null

避免歧义。如果支持 null，无法判断这个 null，是 put 时本身就是 null，还是这个 key 本身就不存在。

例如，线程1判断这个 key 不存在，是 null，但是线程2插入这个 key，value 是 null。多线程环境下，无法区分到底是哪个线程执行的结果，所以线程安全的 ConcurrentHashMap 不支持 null。

