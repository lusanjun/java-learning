## Collection

### 1.Collection体系

![image-20230708152405665](https://gitee.com/lusanjun/blog-img/raw/master/202307081524678.png)

### 2. JDK描述

集合层次结构中的根接口。一个集合代表一组对象，被称为元素。

一些集合允许重复的元素，而有些则不允许。一些是有序的，另一些是无序的。

JDK 没有提供这个接口的任何直接的实现：它提供更具体的子接口像 Set 和 List 实现。

所有通用的 Collection 实现类（通常通过其子接口间接实现Collection）应该提供两个“标准”构造函数：void（无参数） 构造函数，它创建一个空集合，以及一个带有 Collection 类型的单个参数的构造函数，它创建一个与它的参数具有相同元素的新集合。实际上，后者的构造函数允许用户复制任何集合，生成所需实现类型的等效集合。没有办法强制执行这个约定（因为接口不能包含构造函数），但是Java平台库中的所有通用Collection实现都符合。

如果这个集合不支持某个操作的话，调用这些方法可能（但不是必需）抛出 UnsupportedOperationException。

### 3. 接口方法

```java
int size();
```

返回集合中元素的数量，如果数量超过 Integer.MAX_VALUE，则返回Integer.MAX_VALUE。

```java
boolean isEmpty();
```

如果集合中没有元素，返回 true。

```java
 boolean contains(Object o);
```

如果集合中包含了指定元素，返回 true。

```java
Iterator<E> iterator();
```

返回可用于访问此Collection所包含的对象的Iterator实例。 没有定义迭代器返回元素的顺序。
 只有当Collection的实例具有定义的顺序时，才能按照该顺序返回元素。

```java
 Object[] toArray();
```

返回一个包含此集合中包含的所有元素的新数组。
 如果实现已经排序了元素，它将以与迭代器返回的顺序相同的顺序返回元素数组。
 返回的数组不反映集合的任何更改。 即使底层数据结构已经是一个数组，也创建一个新数组。

```java
 <T> T[] toArray(T[] a);
```

返回包含此集合中包含的所有元素的数组。 如果指定的数组足够大以容纳元素，则使用指定的数组
 否则将创建相同类型的数组。 如果使用指定的数组并且大于此集合，则 Collection 元素之后的数组元素将设置为 null。

```java
boolean add(E e);
```

此方法成功完成后，可确保对象包含在集合中。
 如果集合被修改，则返回 true，如果没有更改，则返回 false。

```java
boolean remove(Object o);
```

从这个集合中移除指定元素。

```java
boolean containsAll(Collection<?> c);
```

如果这个集合包含指定集合中的所有元素，返回 true。

```java
boolean addAll(Collection<? extends E> c);
```

将指定集合的所有元素添加到这个集合中。

```java
 boolean removeAll(Collection<?> c);
```

从这个集合中移除指定集合的所有元素。

```java
default boolean removeIf(Predicate<? super E> filter) {
        Objects.requireNonNull(filter);
        boolean removed = false;
        final Iterator<E> each = iterator();
        while (each.hasNext()) {
            if (filter.test(each.next())) {
                each.remove();
                removed = true;
            }
        }
        return removed;
    }
```

删除满足给定谓词的这个集合的所有元素。

```java
  boolean retainAll(Collection<?> c);
```

从这个集合中移除所有不包含在指定集合中的元素的所有元素。

```java
void clear();
```

清空集合。