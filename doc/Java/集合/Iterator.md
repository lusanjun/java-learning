## Iterator



### 1. Iterable

实现此接口允许对象成为“ for-each循环”语句的目标。

#### 1.1 forEach方法

对 Iterable 的每个元素执行给定的操作，直到处理完所有元素或该操作引发异常为止。除非实现类另行指定，否则操作将以迭代顺序执行（如果指定了迭代顺序）。

具体指定的操作需要自己写Consumer接口通过accept方法回调出来。

```java
 default void forEach(Consumer<? super T> action) {
        Objects.requireNonNull(action);
        for (T t : this) {
            action.accept(t);
        }
    }
```

#### 1.2 spliterator方法

在 Iterable 描述的元素上创建一个 Spliterator，默认实现从 iterable 的 Iterator 创建一个 early-binding 拼接器。 Spliter继承了iterable的迭代器的 fail-fast 属性。

spliterator 方法通过一个顺序遍历的 Iterator 对象获取一个并行遍历的 Spliterator 对象；
 关于 Spliterator：Spliterator（splitable iterator可分割迭代器）接口是 Java 为了并行遍历数据源中的元素而设计的迭代器，这个可以类比最早 Java 提供的顺序遍历迭代器 Iterator，但一个是顺序遍历，一个是并行遍历。

```java
  default Spliterator<T> spliterator() {
        return Spliterators.spliteratorUnknownSize(iterator(), 0);
    }
```

### 2. Iterator

Iterator是一个顺序遍历器。可以遍历所有元素，删除元素。

#### 2.1 forEachRemaining方法

对剩余的每个元素执行给定的操作，直到所有元素已处理完毕或该操作引发异常。

如果指定了操作，则按迭代顺序执行操作。操作引发的异常会中继给调用者。

```java
 default void forEachRemaining(Consumer<? super E> action) {
        Objects.requireNonNull(action);
        while (hasNext())
            action.accept(next());
    }
```

forEachRemaining 与 forEach 的区别

- forEachRemaining() 方法内部是通过使用迭代器 Iterator 遍历所有元素，forEach()方法内部使用的是增强for循环；
- forEach() 方法可以多次调用，forEachRemaining()方法第二次调用不会做任何操作，因为不会有下一个元素。