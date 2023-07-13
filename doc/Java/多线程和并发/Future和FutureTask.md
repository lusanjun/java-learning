## Future和FutureTask

### 1.  Future

#### 1.1 Future接口

异步编程接口，主要用来以异步的形式运行其他任务，并返回结果。

向线程池提交一个任务，提交时线程池立即返回一个空的Future对象。当任务一完成，线程池把结果填充到之前的Future中，通过这个对象获知任务的状态和结果，也可以用来干涉任务的执行。

#### 1.2 主要方法

##### 1.2.1 get

get方法的行为取决于Callable任务的状态

- 任务正常完成：get方法会立刻返回结果
- 任务尚未完成：任务还没有开始或进行中，get将阻塞并直到任务完成。
- 任务执行过程中抛出Exception：get方法会抛出ExecutionException，这里抛出异常，是call()执行时产生的那个异常
- 任务被取消：get方法会抛出CancellationException
- 任务超时：get方法有一个重写方法，是传入一个延迟时间的，如果时间到了还没有获得结果，get方法会抛出TimeoutException

##### 1.2.2 get(timeout, unit)

如果call()在规定时间内完成任务，那么就会正常获取到返回值，而如果在指定时间内没有计算出结果，则会抛出TimeoutException

##### 1.2.3 cancel

- 如果这个任务还没有开始执行，任务会被正常取消，未来也不会被执行，返回true
- 如果任务已经完成或已经取消，则cancel()方法会执行失败，方法返回false
- 如果这个任务已经开始，这个取消方法将不会直接取消该任务，而是会根据参数mayInterruptIfRunningg来做判断。如果是true,就会发出中断信号给这个任务

##### 1.2.4 idDone

判断线程是否执行完，执行完并不代表执行成功。

##### 1.2.5 isCancelled

判断是否被取消。

### 2. FutureTask

FutrueTask实现了RunnableFuture接口，RunnableFuture 继承自 Runnable 和 Future接口。

把Callable实例当作参数，生成 FutureTask对象，然后把这个对象当作一个 Runnable对象，用线程池去执行这个Runnable对象。最后通过FutureTask获取执行的结果。

