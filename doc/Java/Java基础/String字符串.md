## String字符串



### 1. String基础

#### 1.1 创建字符串

```java
 String s1 = "abc";
 String s2 = new String(); s2 = "abc";
 String s3 = new String("abc");
```

#### 1.2 数据结构

在 JDK8 中，String 内部使用 char 数组存储数据。

```java
 public final class String
     implements java.io.Serializable, Comparable<String>, CharSequence {
     /** The value is used for character storage. */
     private final char value[];
 }
```

在 JDK9 之后，String 类的实现改用 byte 数组存储字符串，同时使用 coder 来标识使用了哪种编码。

```java
 public final class String
     implements java.io.Serializable, Comparable<String>, CharSequence {
     /** The value is used for character storage. */
     private final byte[] value;
 
     /** The identifier of the encoding used to encode the bytes in {@code value}. */
     private final byte coder;
 }
```

改变的原因：

之前一直是最小单位是一个 char，用到两个 byte，但是 java8 是基于 latin1 的，而这个 latin1 编码可以用一个 byte 标识，所以当你数据明明可以用到一个 byte 的时候，却用到了一个最小单位 char 两个byte，就多出了一个 byte 的空间。所以 java9 在这一方面进行了更新，现在的 java9 是基于 ISO/latin1/Utf-16  ,latin1 和 ISO 用一个 byte 标识，UTF-16 用两个 byte 标识，java9 会自动识别用哪个编码，当数据用到 1byte，就会使用 ISO 或者 latin1 ，当空间数据满足 2byte 的时候，自动使用 utf-16，节省了很多空间。

#### 1.3 字符串的不可变性

String 的对象一旦被创建，则不能修改，是不可变的。内部是 final 修饰。

所谓的修改其实是创建了新的对象，所指向的内存空间不变。

不变的好处：

- 可以缓存hash值：因为 String 的 hash 值经常被使用，例如 String 用做 HashMap 的 key。不可变的特性可以使得 hash 值也不可变，因此只需要进行一次计算。
- String Pool 的需要：如果一个 String 对象已经被创建过了，那么就会从 String Pool 中取得引用。只有 String 是不可变的，才可能使用 String Pool。
- 安全性：String 经常作为参数，String 不可变性可以保证参数不可变。例如在作为网络连接参数的情况下如果 String 是可变的，那么在网络连接过程中，String 被改变，改变 String 的那一方以为现在连接的是其它主机，而实际情况却不一定是。
- 线程安全：String 不可变性天生具备线程安全，可以在多个线程中安全地使用。

### 2. StringBuilder,StringBuffer

#### 2.1 区别

可变性：String 不可变；StringBuffer 和 StringBuilder 可变。

线程安全：String 不可变，因此是线程安全的；StringBuilder 不是线程安全的；StringBuffer 是线程安全的，内部使用  synchronized  进行同步。

#### 2.2 + 运算符

```java
 String a = "a"; a = a + a;
```

字符串用 + 号拼接，实际是把字符串封装成 StringBuilder，调用 append 方法后再 toString 返回。

大量字符串拼接时，建议直接使用 StringBuilder。

#### 2.3 字符串常量池

字符串常量池（String Pool）保存着所有字符串字面量（literal strings），这些字面量在编译时期就确定。不仅如此，还可以使用 String 的 intern() 方法在运行过程将字符串添加到 String Pool 中。

当一个字符串调用 intern() 方法时，如果 String Pool 中已经存在一个字符串和该字符串值相等（使用 equals() 方法进行确定），那么就会返回 String Pool 中字符串的引用；否则，就会在 String Pool 中添加一个新的字符串，并返回这个新字符串的引用。

“+”符号，会被编译器处理成 StringBuilder.append() 方法。在运行期间，链接字符串的计算都是通过创建 StringBuilder 对象，调用 append() 方法来完成的，而且是每一个链接字符串的表达式都要创建一个 StringBuilder 对象。因此对于循环中反复执行字符串链接时，应该考虑直接使用StringBuilder 来代替 + 链接，避免重复创建 StringBuilder 的性能开销。

下面示例中，s1 和 s2 采用 new String() 的方式新建了两个不同字符串，而 s3 和 s4 是通过 s1.intern() 方法取得同一个字符串引用。intern() 首先把 s1 引用的字符串放到 String Pool 中，然后返回这个字符串引用。因此 s3 和 s4 引用的是同一个字符串。

```java
 String s1 = new String("aaa");
 String s2 = new String("aaa");
 System.out.println(s1 == s2);           // false
 String s3 = s1.intern();
 String s4 = s1.intern();
 System.out.println(s3 == s4);           // true
```

如果是采用字面量的形式创建字符串，会自动地将字符串放入 String Pool 中。

```java
 String s5 = "bbb";
 String s6 = "bbb";
 System.out.println(s5 == s6);  // true
```

在 Java 7 之前，String Pool 被放在运行时常量池中，它属于永久代。而在 Java 7，String Pool 被移到堆中。

这是因为永久代的空间有限，在大量使用字符串的场景下会导致 OutOfMemoryError 错误。

#### 2.4 new String("abc")

使用这种方式一共会创建两个字符串对象（前提是 String Pool 中还没有 "abc" 字符串对象）："abc" 属于字符串字面量，因此编译时期会在 String Pool 中创建一个字符串对象，指向这个 "abc" 字符串字面量；new 会在堆中创建一个字符串对象。

```java
 public class NewStringTest {
     public static void main(String[] args) {
         String s = new String("abc");
     }
 }
```

使用 javap -verbose 进行反编译，得到以下内容：

```java
 // ...
 Constant pool:
 // ...
    #2 = Class              #18            // java/lang/String
    #3 = String             #19            // abc
 // ...
   #18 = Utf8               java/lang/String
   #19 = Utf8               abc
 // ...
 
   public static void main(java.lang.String[]);
     descriptor: ([Ljava/lang/String;)V
     flags: ACC_PUBLIC, ACC_STATIC
     Code:
       stack=3, locals=2, args_size=1
          0: new           #2                  // class java/lang/String
          3: dup
          4: ldc           #3                  // String abc
          6: invokespecial #4                  // Method java/lang/String."<init>":(Ljava/lang/String;)V
          9: astore_1
 // ...
```

在 Constant Pool 中，19 存储这字符串字面量 "abc"，3 是 String Pool 的字符串对象，它指向 #19 这个字符串字面量。在 main 方法中，0: 行使用 new #2 在堆中创建一个字符串对象，并且使用 ldc #3 将 String Pool 中的字符串对象作为 String 构造函数的参数。

以下是 String 构造函数的源码，可以看到，在将一个字符串对象作为另一个字符串对象的构造函数参数时，并不会完全复制 value 数组内容，而是都会指向同一个 value 数组。

```java
 public String(String original) {
     this.value = original.value;
     this.hash = original.hash;
 }
```

#### 2.5 StringJoiner

StringJoiner基于 StringBuilder 实现，用于实现对字符串之间通过分隔符拼接的场景。

StringJoiner 有两个构造方法，第一个构造要求依次传入分隔符、前缀和后缀。第二个构造则只要求传入分隔符即可（前缀和后缀默认为空字符串）。

```java
StringJoiner(CharSequence delimiter, CharSequence prefix, CharSequence suffix)
StringJoiner(CharSequence delimiter)
```

有些字符串拼接场景，使用 StringBuffer 或 StringBuilder 则显得比较繁琐。

比如下面的例子：

```java
List<Integer> values = Arrays.asList(1, 3, 5);
StringBuilder sb = new StringBuilder("(");

for (int i = 0; i < values.size(); i++) {
	sb.append(values.get(i));
	if (i != values.size() -1) {
		sb.append(",");
	}
}

sb.append(")");
```

而通过StringJoiner来实现拼接List的各个元素，代码看起来更加简洁。

```java
List<Integer> values = Arrays.asList(1, 3, 5);
StringJoiner sj = new StringJoiner(",", "(", ")");

for (Integer value : values) {
	sj.add(value.toString());
}
```

Collectors.joining(",")，底层是通过StringJoiner实现的。

```java
public static Collector<CharSequence, ?, String> joining(
    CharSequence delimiter,CharSequence prefix,CharSequence suffix) {
    return new CollectorImpl<>(
            () -> new StringJoiner(delimiter, prefix, suffix),
            StringJoiner::add, StringJoiner::merge,
            StringJoiner::toString, CH_NOID);
}
```

