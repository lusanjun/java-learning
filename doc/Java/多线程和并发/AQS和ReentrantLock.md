## AQS和ReentrantLock

### 1. AQS

#### 1.1 什么是AQS

AQS，AbstractQueuedSynchronizer，抽象队列同步器。本身只是一个抽象类，AQS中也用到了 volatile 和CAS。通过维护一个共享资源状态（volatile int state）和一个先进先出（FIFO）的线程等待队列来实现一个多线程访问共享资源的同步框架。是其他同步组件（ReentrantLock、Semaphore）的基础。

#### 1.2 结构

![AQS](https://gitee.com/lusanjun/blog-img/raw/master/img/AQS.png)

```java
//AQS内部维护这一个双向链表，AQS主要属性
public abstract class AbstractQueuedSynchronizer extends AbstractOwnableSynchronizer implements java.io.Serializable {
	private transient volatile Node head;//头节点指针
	private transient volatile Node tail;//尾节点指针
	private volatile int state;//同步状态，0无锁；大于0，有锁，state的值代表重入次数。
	//AQS链表节点结构
	static final class Node {
		static final Node SHARED = new Node();//共享模式
		static final Node EXCLUSIVE = null;//独占模式
        /**
        *等待状态：取消。表明线程已取消争抢锁并从队列中删除。
        *取消动作：获取锁超时或者被其他线程中断。
        */
        static final int CANCELLED = 1;
        /**
        *等待状态：通知。表明线程为竞争锁的候选者。
        *只要持有锁的线程释放锁，会通知该线程。
        */
        static final int SIGNAL = -1;
        /**
        *等待状态：条件等待
        *表明线程当前线程在condition队列中。
        */
        static final int CONDITION = -2;
        /**
        *等待状态：传播。
        *用于将唤醒后继线程传递下去，这个状态的引入是为了完善和增强共享锁的唤醒机
        制。
        *在一个节点成为头节点之前，是不会跃迁为此状态的
        */
        static final int PROPAGATE = -3;
        volatile int waitStatus;
        volatile Node prev;//直接前驱节点指针
        volatile Node next;//直接后继节点指针
        volatile Thread thread;//线程
        Node nextWaiter;//condition队列中的后继节点
    }
}    
```

AQS是一个FIFO的双向队列，内部通过节点head和tail记录队首和队尾元素，队列元素的类型为Node。

- thread 变量用来存放进入AQS队列里面的线程，Node节点内部：
  - prev 记录当前节点的前驱节点
  - next 记录当前节点的后继节点
- shared 用来标记该线程是获取共享资源时被阻塞挂起后放入AQS队列的
- exclusive 用来标记线程是获取独占资源时被挂起后放入AQS队列的
- waitStatus 记录当前线程等待状态，可以为
  - cancelled 线程被取消
  - signal 线程需要被唤醒
  - condition 线程在condition条件队列里面等待
  - propagate 释放共享资源时需要通知其他节点

AQS中维持了一个单一的状态信息state，对于ReentrantLock的实现来说，state可以用来表示当前线程获取锁的可重入次数。

AQS继承自AbstractOwnableSynchronizer，其中的exclusiveOwnerThread变量表示当前共享资源的持有线程。

#### 1.3 原理

AQS有两个节点head和tail分别是头节点和尾节点，默认为null。AQS中的内部静态类Node为链表节点，AQS会在线程获取锁失败后，线程会被阻塞并被封装成Node加入到AQS队列中，当获取锁的线程释放锁后，会从AQS队列中唤醒一个线程（节点）。

##### 1.3.1 线程抢夺锁失败

1. AQS的head、tail分别代表同步队列头节点和尾节点指针，默认为null

   ![AQS](https://gitee.com/lusanjun/blog-img/raw/master/img/AQS1.png)

2. 当第一个线程抢夺锁失败，同步队列会先初始化，随后线程会被封装成Node节点追加到AQS队列中。假设当前独占锁的线程为ThreadA，抢占锁失败的线程为ThreadB。

   1）同步队列初始化，首先会在队列中添加一个空Node，代表当前获取锁成功的线程，AQS的head和tail同时指向这个节点。

   ![AQS](https://gitee.com/lusanjun/blog-img/raw/master/img/AQS2.png)

   2）接下来将ThreadB封装成Node节点，追加到AQS队列，设置新节点的prev指向AQS队尾节点。将队尾节点的next指向新节点。最后将AQS尾节点指针指向新节点。

   ![AQS](https://gitee.com/lusanjun/blog-img/raw/master/img/AQS3.png)

3. 当下一个线程抢夺锁失败时，重复上面步骤。将新线程封装成Node，追加到AQS队列。

   ![AQS](https://gitee.com/lusanjun/blog-img/raw/master/img/AQS4.png)

##### 1.3.2 线程被唤醒时

ReentrantLock唤醒阻塞线程时，会按照FIFO的原则从AQS中head头部开始唤醒首个节点中的线程。

head节点表示当前获取锁成功的线程ThreadA节点。

当ThreadA释放锁时，它会唤醒后继节点线程ThreadB，ThreadB开始尝试获得锁，如果ThreadB获得锁成功，会将自己设置为AQS的头节点。ThreadB获取锁成功后，AQS会发生以下变化：

1. head指针指向ThreadB节点
2. 将原来头节点的next指向null，从AQS队列中删除
3. 将ThreadB节点的prev指向Null，设置节点的thread=null

![AQS](https://gitee.com/lusanjun/blog-img/raw/master/img/AQS5.png)

### 2. ReentrantLock

#### 2.1 获取锁

##### 2.1.1 ReentrantLock.lock()

```java
public void lock(){
	sync.lock();
}
```

ReentrantLock获取锁调用了lock方法，内部调用了 `sync.lock()`

sync是Sync类的一个实例，Sync类实际上是ReentrantLock的抽象静态内部类，它集成了AQS来实现重入锁的具体业务逻辑。AQS是一个同步队列，实现了线程的阻塞和唤醒，没有实现具体的业务功能。在不同的同步场景中，需要用户继承AQS来实现对应的功能。

Sync有两个实现类公平锁FairSync和非公平锁NoFairSync。重入锁实例化时，根据参数fair为属性sync创建对应锁的实例。以公平锁为例，调用`sync.lock`事实上调用的是FairSync的lock方法。

##### 2.1.2 FairSync.lock()

```java
final void lock(){
    acquire(1);
}
```

执行了方法 `acquire(1)`，acquire为AQ中的final方法，用于竞争锁

##### 2.1.3 AQS.acquire(1)

```java
public final void acquire(int arg) {
	if (!tryAcquire(arg) && acquireQueued(addWaiter(Node.EXCLUSIVE),arg))
		selfInterrupt();
}
```

先尝试抢占锁，抢占成功，直接返回；

抢占失败，将线程封装成Node节点追加到AQS队列中并使线程阻塞等待。

1. 首先会执行`tryAcquire(1)`尝试抢占锁，成功返回true，失败返回false。抢占成功了，就不会执行下面的代码。
2. 抢占锁失败后，执行`addWaiter(Node.EXCLUSIVE)`将新线程封装成Node节点追加到AQS队列
3. 调用`accquireQueued`将线程阻塞

##### 2.1.4 FairSync.tryAcquire(1)

尝试获取锁：获取成功返回true；获取失败返回false。

获取当前锁的状态，如果为无锁状态，当前线程会执行CAS操作尝试获取锁；若当前线程是重入获取锁，只需增加锁的重入次数。

##### 2.1.5 AQS.addWaiter(Node.EXCLUSIVE)

线程抢占锁失败后，执行`addWaiter(Node.EXCLUSIVE)`将线程封装成Node节点追加到AQS队列。

Node定义了两种类型：

- Node.SHARED：共享模式，共享锁，可以被多个线程同时持有，如读写锁的读锁；
- Node.EXCLUSIVE：独占模式，独占锁/排他锁，自己独占资源，独占排他锁只能由一个线程持有

##### 2.1.6 AQS.acquireQueued(newNode,1)

将线程阻塞

1. 若同步队列中，当前节点为队列第一个线程，则有资格竞争锁，再次尝试获得锁
   - 尝试获得锁成功，移除链表head节点，并将当前线程节点设为head节点
   - 尝试获得锁失败，判断是否需要阻塞当前线程
2. 若发生异常，取消当前线程获得锁的资格

#### 2.2 释放锁

##### 2.2.1 ReentrantLock.unlock

```java
public void unlock(){
	sync.release(1);
}
```

ReentrantLock释放锁调用了unlock方法，内部调用了 `sync.release(1)`，release方法为AQS类的final方法。

##### 2.2.2 AQS.release(1)

```java
public final boolean release(int arg) {
	if (tryRelease(arg)) {//释放锁。若释放后锁状态为无锁状态，需唤醒后继线程
		Node h = head;//同步队列头节点
		if (h != null && h.waitStatus != 0)//若head不为null,说明链表中有节点。其状态不为0，唤醒后继线程。
			unparkSuccessor(h);
		return true;
	} 
    return false;
}
```

首先执行`tryRelease(1)`，tryRelease方法为ReentrantLock中Sync类的final方法，用于释放锁

##### 2.2.3 Sync.tryRelease(1)

```java
/**
* 释放锁返回值：true释放成功；false释放失败
*/
protected final boolean tryRelease(int releases) {
	int c = getState() - releases;//重入次数减去1
	//如果当前线程不是锁的独占线程，抛出异常
	if (Thread.currentThread() != getExclusiveOwnerThread())
		throw new IllegalMonitorStateException();
	boolean free = false;
	if (c == 0) {
	//如果线程将锁完全释放，将锁初始化未无锁状态
		free = true;
		setExclusiveOwnerThread(null);
	} 
    setState(c);//修改锁重入次数
	return free;
}
```

1. 判断当前线程是否为锁持有者，若不是持有者，不能释放锁，直接抛出异常。

2. 若当前线程是锁的持有者，将重入次数减1，并判断当前线程是否完全释放了锁。

   - 若重入次数为0，则当前新线程完全释放了锁，将锁拥有线程设置为null，并将锁状态置为无锁状态(state=0)，返回true。
   - 若重入次数>0,则当前新线程仍然持有锁，设置重入次数=重入次数-1，返回false。
3. 返回true说明，当前锁被释放，需要唤醒同步队列中的一个线程，执行`unparkSuccessor`唤醒同步队列中节点线程。


##### 2.2.4 AQS.unparkSuccessor

唤醒后继线程，查找需要唤醒的节点。正常情况下，它应该是下一个节点，但是如果下一个节点为null或者它的waitStatus为取消时，则需要从同步队列tail节点向前遍历，查找到队列中首个不是取消状态的节点。

##### 2.2.5 LockSupport.unpark(s.thread)

会唤醒挂起的线程，使被阻塞的线程继续执行。

#### 2.3 公平非公平

公平锁：按照请求锁的顺序去获得锁，竞争失败后，都会到AQS队列队尾排队，不允许中途插队。

非公平锁：可以插队。

实现原理的差别：

1. lock方法差别：

   FairSync.lock：公平锁获取锁

   ```java
   final void lock(){
   	acquire(1);
   }
   ```

   NoFairSync.lock：非公平锁获取锁，lock方法中新线程会先通过CAS操作`compareAndSetState(0,1)`，尝试获得锁。

   ```java
   final void lock() {
   	if (compareAndSetState(0, 1))//新线程，第一次插队
   		setExclusiveOwnerThread(Thread.currentThread());
   	else
   		acquire(1);
   }
   ```

   

2. tryAcquire差异

   公平锁获取锁，先判断同步队列中是否有线程在等待，若有线程在等待，当前线程只能进入同步队列等待。

   ```java
   if (!hasQueuedPredecessors() && compareAndSetState(0, acquires)) {
   	setExclusiveOwnerThread(current);
   	return true;
   }
   ```

   非公平锁，不管AQS是否有线程在等待，都会先通过CAS抢占锁。

   ```java
//非公平锁，入队前，二次插队
   if (compareAndSetState(0, acquires)) {
   	setExclusiveOwnerThread(current);
   	return true;
   }
   ```

#### 2.4 ReentrantReadWriteLock

可重入锁ReentrantLock是互斥锁，互斥锁在同一时刻仅有一个线程可以进行访问，但是大多数场景下，读服务多于写服务。读服务不存在数据竞争问题，如果一个线程在读时禁止其他线程读，会导致性能降低。

读写锁维护着一对锁，一个读锁和一个写锁。读写分离，使得性能提升。

主要特性：

- 公平性：支持公平和不公平锁；
- 重入性：支持重入。
- 锁降级：写锁能够降级为读锁，但读锁不能升级为写锁。遵循获取写锁、获取读锁在释放写锁的次序。