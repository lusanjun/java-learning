## JUC

### 1. JUC

JUC是java中`java.util.concurrent`包的简称，主要包含三个部分

- `atomic`包：支持原子操作类包
- `locks`包：锁相关的包
- 直接接口和类：并发工具容器阻塞队列等相关代码

### 2. atomic

提供一系列原子类，通过CAS支持原子操作。

### 3. locks

提供java中的锁，如 ReentrantLock，通过AQS来实现。

- AbstractOwnableSynchronizer
- AbstractQueuedLongSynchronizer
- AbstractQueuedSynchronizer
- Condition
- Lock
- LockSupport
- ReadWriteLock
- ReentrantLock
- ReentrantReadWriteLock
- StampedLock