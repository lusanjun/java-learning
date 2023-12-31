## Synchronized和锁

### 1. Synchronized

#### 1.1 Synchronized作用域

##### 1.1.1 对象锁

```java
//例1,修饰一个代码块
public class Test{
    public void t1(){
        synchronized(this){}
    }
}

//例2，修饰一个方法
public class Test{
    public synchronized void t2(){}  
}
```

锁的都是 this 当前对象，代码块锁比方法锁更灵活，代码块外的代码仍可访问。

锁住的是同一个对象，一个线程在访问对象的同步代码块时，另一个访问对象的同步代码块的线程会被阻塞；

##### 1.1.2 类锁

```java
//例3，修饰一个类
public class Test{
    public void t3(){
        synchronized(Test.class){}
    }  
}   

//例4，修饰一个静态方法
public class Test{
    public static synchronized void t4(){}  
} 
```

锁的是当前类的 class。类锁和对象锁互不干扰。

类锁相当于一个院子的大门锁，对象锁相当于院子内的各个房间锁。

#### 1.2 实现可见性过程

1. 获得互斥锁
2. 清空本地内存
3. 从主内存拷贝变量的最新副本到本地内存
4. 执行代码
5. 将更改后的共享变量的值刷新到主内存
6. 释放互斥锁

#### 1.3 实现原理

##### 1.3.1 对象头

Class Metadata Address：类型指针，是对象指向它的类元数据的指针，虚拟机通过这个指针来确定这个对象是哪个类的实例

Mark Word：标记字段，用于存储对象自身的运行时数据。默认存储对象的hashCode、分代年龄、锁类型、锁标志位等信息。

![MarkWord](https://gitee.com/lusanjun/blog-img/raw/master/img/MarkWord.png)

##### 1.3.2 monitor 

monitor 意译为管程，直译为监视器。管程，就是管理共享变量及对共享变量操作的过程。

每个 Java 对象天生自带的一把看不见的锁。当一个 monitor 被持有后，它处于锁定状态。Mark Word 中的锁标志位为10，指针指向的就是 monitor 对象的起始地址。monitor是由 ObjectMonitor 实现的：

```c++
ObjectMonitor() {
    _header       = NULL;
    _count        = 0; // 记录个数
    _waiters      = 0,
    _recursions   = 0;
    _object       = NULL;
    _owner        = NULL;
    _WaitSet      = NULL; // 处于wait状态的线程，会被加入到_WaitSet
    _WaitSetLock  = 0 ;
    _Responsible  = NULL ;
    _succ         = NULL ;
    _cxq          = NULL ;
    FreeNext      = NULL ;
    _EntryList    = NULL ; // 处于等待锁block状态的线程，会被加入到该列表
    _SpinFreq     = 0 ;
    _SpinClock    = 0 ;
    OwnerIsThread = 0 ;
  }
```

##### 1.3.3 原理

示例代码：

```java
public class Syn {
   public void syn(){
       //同步代码块
       synchronized (this){
           System.out.println("syn");
       }
   }
   //同步方法
   public synchronized void synMethod(){
       System.out.println("synMethod");
   }
}
```

字节码文件经过反编译：

```java
public void syn();
    descriptor: ()V
    flags: ACC_PUBLIC
    Code:
      stack=2, locals=3, args_size=1
         0: aload_0
         1: dup
         2: astore_1
         3: monitorenter
         4: getstatic     #2            
         7: ldc           #3            
         9: invokevirtual #4            
        12: aload_1
        13: monitorexit
        14: goto          22
        17: astore_2
        18: aload_1
        19: monitorexit
        20: aload_2
        21: athrow
        22: return
...
      
  public synchronized void synMethod();
    descriptor: ()V
    flags: ACC_PUBLIC, ACC_SYNCHRONIZED
    Code:
      stack=2, locals=1, args_size=1
         0: getstatic     #2            
         3: ldc           #5            
         5: invokevirtual #4            
         8: return
      LineNumberTable:
        line 11: 0
        line 12: 8
      
```

从字节码文件中可以看出 synchronized 代码块是由 monitorenter 指令进入锁，monitorexit 指令退出锁，synchronized 方法是由 ACC_SYNCHRONIZED 访问标志来控制锁。

- monitorenter 指令：每个对象都有一个监视器锁（monitor），当 monitor 被占用时就会处于锁定状态，线程执行 monitorenter 指令时尝试获取 monitor 的所有权，当计数器 _count 为0时，该程序就获取到了 monitor 锁，并将 _count 计数器 +1，如果线程已经占有该 monitor ，可以重入。如果其他线程已经占用了 monitor ，则该线程会进入阻塞状态，直到计数器为0，再次尝试竞争 monitor。
- monitorexit 指令：当线程执行到 monitorexit 指令，则释放 monitor 锁，_count 计数器就 -1为0。该指令出现两次，第一次是线程正常退出释放锁，第二次为异常处理器，可以处理一起异常以能够释放锁。
- 当调用同步方法时，调用指令将检查方法的 ACC_SYNCHRONIZED 访问标志是否被设置，如果设置了，执行线程将先获取 monitor，获取成功后才能执行方法体，方法执行完后再释放 monitor 。在方法执行期间，其他任何线程都无法再获取同一个 monitor 锁。
- 重入：从互斥锁的设计上来说，当一个线程试图操作一个由其他线程持有的对象锁的共享数据时，将会处于阻塞状态，但当一个线程再次请求自己持有对象锁的共享数据时，这种情况属于重入。

缺点：jdk6 之前，synchronized 属于重量级锁，依赖于 Mutex Lock 实现，线程之间的切换需要从用户态转换到核心态，开销较大。

### 2. 锁分类

#### 2.1 上锁方式

- 隐式锁：synchronized，不需要显式加锁和解锁。
- 显式锁：JUC包中的锁，需要显式加锁和解锁。

#### 2.2 乐观/悲观锁

- 乐观锁：比较乐观，总是假设最好的情况，在获取数据时不加锁，在更新数据时去判断有没有别的线程更新了这个数据，如果没有被更新，当前线程自己修改数据；如果数据已被其他线程更新，则会根据不同情况执行不同操作。

  实现：CAS算法，关系型数据库的版本号机制

- 悲观锁：比较悲观，总是假设最坏的情况，悲观锁认为自己在使用数据时一定有别的线程来修改数据，因此在获取数据时先加锁。

  实现：JUC的锁，Synchronized

#### 2.3 可重入/不可重入锁

- 可重入锁：一个线程可以重复获取同一把锁，不会因为之前已经获取了该所未释放而阻塞。在获得一个锁之后未释放锁之前，再次获得同一把锁时，会增加获得锁的次数。释放锁时，会减少锁定次数。可以一定程度避免死锁。

  实现：ReentrantLock，Synchronized

- 不可重入锁：同一线程获得锁之后不可再次获取锁，重复获取会发生死锁。

#### 2.4 公平/非公平锁

- 公平锁：多个线程按照申请锁的顺序来获得锁

  实现：new ReentrantLock(true)

- 非公平锁：多个线程按照获取锁的顺序而不是申请锁的顺序，允许“插队”，有可能后申请的线程先获取锁。

  实现：new ReentrantLock(false)，Synchronized

#### 2.5 独享/共享锁

- 独享锁（写锁）：也叫排他锁，同一个锁同时只能被一个线程持有，如果线程A获得锁S后，其他线程只能阻塞等待释放锁，才能继续获得锁。

  实现：ReentrantLock，Synchronized

- 共享锁（读锁）：同一个锁可以被多个线程同时持有，如果线程A获得锁S后，其他线程无需等待可以同时获得锁。

  实现：ReentrantReadWriteLock的读锁

#### 2.6 其他锁

- 自旋锁：获取锁失败时，线程不会阻塞而是循环尝试获得锁，直到获得锁成功

  实现：CAS，轻量级锁

- 分段锁：容器有多把锁，每把锁用于锁定容器的某一部分数据，线程间不会存在锁竞争。

  实现：ConcurrentHashMap

- 无锁/偏向锁/轻量级锁/重量级锁：synchronized独有的四种状态，JVM为了提高Synchronized锁获取和释放效率做的优化。

#### 2.7 JUC锁与Synchronized对比

Synchronized  的不足

- 同步锁的线程阻塞存在两个缺陷：无法控制阻塞时长；阻塞不可中断，甚至造成死锁。
- 读多写少的场景，多个线程同时操作读去共享资源时，Synchronized 会阻塞其他线程读取，不可以同时进行共享读操作。

### 3. 锁优化

#### 3.1 自旋锁

线程的阻塞和唤醒需要 CPU 从用户态转为和核心态，频繁的阻塞唤醒性能压力大，而且锁状态只持续很短时间。自旋锁就是当一个线程尝试获取某个锁时，如果该锁已被其他线程占用，就一直循环检测该锁是否被释放，不让出 CPU 时间。

优点：尽可能减少线程的阻塞，自旋的消耗小于线程阻塞挂起再唤醒的操作消耗。

缺点：如果锁竞争激烈，长时间自旋占用 CPU 时间，白白浪费性能。

自旋锁在 jdk1.4 中引入，默认关闭。在 jdk1.6 中默认开启，默认自旋次数为10次，可以通过-XX:PreBlockSpin来调整。jdk1.7 后，去掉此参数，由 jvm 控制。

#### 3.2 自适应锁

自旋的次数不再固定，由前一次在同一个锁上的自旋时间及锁的拥有者的状态来决定。线程如果自旋成功了，那么下次自旋的次数就会更多。反之，很少自旋成功，就会减少自旋次数，避免CPU资源浪费。

#### 3.3 锁消除

在虚拟机即时编译时，对一些代码要求同步，但是对被检测到不可能存在共享数据竞争的锁进行消除。

判断依据是逃逸分析：

- 如果一个方法中定义的一个对象，可能被外部方法引用，称为方法逃逸
- 如果对象可能被其他外部线程访问，称为线程逃逸。比如赋值给类变量或可以在其他线程中访问的实例变量。

例如：jvm 明显检测到变量 stringBuffer 没有逃逸出 add() 方法，便可以将sb内部的锁消除。

```java
public void add(String str1,String str2){
    StringBuffer stringBuffer = new StringBuffer();
    stringBuffer.append(str1).append(str2);
}
```

#### 3.4 锁粗化

将多个连续的加锁、解锁操作连接在一起，扩展成一个范围更大的锁。

例如：stringBuffer.append 会反复加锁解锁，jvm 就会粗化锁，加到整个循环外部。

```java
public String append(String str){
    int i=0;
    StringBuffer stringBuffer = new StringBuffer();
    while(i<100){
        stringBuffer.append(str);
    }
    return stringBuffer.toString();
}
```

#### 3.5 轻量级锁

轻量级是相对于传统锁机制而言，是没有多线程竞争的情况下，减少传统锁使用系统实现互斥产生的性能消耗。

轻量级锁CAS操作之前堆栈与对象的状态：

![StackBeforeCAS](https://gitee.com/lusanjun/blog-img/raw/master/img/StackBeforeCAS.png)

加锁过程：

1. 在代码即将进入同步块时，如果同步对象没有被锁定（锁标志位为“01”状态），虚拟机首先在当前线程的栈帧中建立一个名为Lock Record的空间 ，用于存储对象目前的Mark Word的拷贝（官方称之为Displaced Mark Word）。

2. 虚拟机使用CAS操作尝试把对象的Mark Word更新为指向Lock Record的指针。

3. 如果更新成功，即代表该线程拥有了这个对象的锁，并且对象Mark Word的锁标志位变为“00”，即处于轻量级锁状态。

   ![StackAfterCAS](https://gitee.com/lusanjun/blog-img/raw/master/img/StackAfterCAS.png)

4. 如果更新失败，虚拟机会首先检查对象的Mark Word是否指向当前线程的栈帧。如果是，说明当前线程已经拥有了这个对象的锁，那直接进入同步块继续执行，否则说明这个锁对象已经被其他线程抢占了。如果出现两条以上的线程争用同一个锁，那轻量级锁就不再有效，必须膨胀为重量级锁，锁标志位为“10”，后面等待锁的线程也必须进入阻塞状态。

解锁过程：

1. 通过CAS操作把对象当前的Mark Word和线程中复制的Displaced Mark Word替换回来；
2. 如果替换成功，整个同步过程就完成了，恢复到无锁状态（01）；
3. 如果替换失败，说明有其他线程尝试过获取该锁，就要在释放锁同时，唤醒被挂起的线程。

缺点：

如果没有竞争，轻量级锁便通过CAS操作成功避免了使用互斥量的开销；但如果确实存在锁竞争，除了互斥量的开销，还有CAS操作的开销，反而比传统的重量级锁更慢。

#### 3.6 偏向锁

偏向于第一个获取锁对象的线程，这个线程在之后获取该锁不需要再做同步操作，甚至CAS都不需要。

加锁过程：

1. 当锁对象第一次被线程获取时，虚拟机将会把对象头中的标志位设置为“01”、把偏向模式设置为“1”，表示进入偏向模式。
2. 使用CAS操作把获取到这个锁的线程的ID记录再对象的Mark Word中。
3. 如果CAS操作成功，持有偏向锁的线程以后每次进入这个锁的同步块时，都不再进行任何操作。
4. 一旦出现另一个线程去尝试获取这个锁，偏向模式就结束。根据对象是否被锁定决定是否撤销偏向。撤销后标志位恢复到未锁定（01）或轻量级锁定（00）。

缺点：

如果程序中大多数锁总是被多个线程访问，偏向模式就是多余的。