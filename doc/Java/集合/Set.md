## Set

### 1. HashSet

元素都是唯一的，不可重复。

底层通过 HashMap 实现

```java
 public HashSet(Collection<? extends E> c) {
        map = new HashMap<>(Math.max((int) (c.size()/.75f) + 1, 16));
        addAll(c);
    }
```

#### 1.1 添加元素

```java
public boolean add(E e) {
        return map.put(e, PRESENT)==null;
    }
```

调用 map 的 put 方法，value PRESENT用来占位。

1. 计算元素的 hash 值；
2. 如果 hash 值不存在冲突，就插入 map 的节点数组位置；
3. 如果发生 hash 冲突，再通过元素的 equals 方法判断冲突元素是否相同；
   - 如果不同，则按照 map 的插入方法，放入链表或红黑树中；
   - 如果相同，则拒绝插入。

#### 1.2 如何保证唯一性

内部维护一个 HashMap，增加元素时，调用 HashMap 的 put 方法，在 put 方法中，关键的代码：

key，即 set 中的元素，key 的值或 key 的 equals 方法返回 true，put 方法终止，不会增加该重复元素。

```java
if (e.hash == hash && ((k = e.key) == key || (key ! = null && key.equals(k))))
    break;
```

### 2. TreeSet

有序集合，底层是 TreeMap，

TreeSet中的元素支持2种排序方式：自然排序 或者 根据创建TreeSet 时提供的 Comparator 进行排序。这取决于使用的构造方法。

去重方法：

根据 compareTo 方法，如果等于0，则说明元素重复。

### 3.LinkedHashSet

LinkedHashSet是一个哈希表和链表的结合，且是一个双向链表
 并且linkedHashSet是一个非线程安全的集合。如果有多个线程同时访问当前linkedhashset集合容器，并且有一个线程对当前容器中的元素做了修改，那么必须要在外部实现同步保证数据的准确性。
 LinkedHashSet 底层使用 LinkedHashMap 来保存所有元素，它继承与 HashSet，其所有的方法操作上又与 HashSet 相同。