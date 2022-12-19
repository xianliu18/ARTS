### ThreadPoolExecutor 深度解读

### 1，类 ThreadPoolExecutor 继承关系
[ThreadPoolExecutor 类继承图](https://github.com/xianliu18/ARTS/tree/master/jvm/threadpool/images/ThreadPoolExecutor 类继承图.jpeg)

- 接口 `Executor`：
  - 将任务提交和任务执行进行解耦。
  - 用户只需提供 Runnable 对象，由 Executor 框架完成线程的调度和任务的执行部分；
- 接口 `ExecutorService`:
	- 管理线程池的终止：`shutdown()`
	- 增加处理异步任务的方法：`submit()`

#### 1.1 阻塞队列(BlockingQueue)
- 阻塞队列简化了生产者 - 消费者设计的实现过程。
- 阻塞队列提供了可阻塞的 `put` 和 `take` 方法，以及支持定时的 `offer(e, timeout, timeunit)` 和 `take` 方法
  - `put`：在队列末尾添加元素，若队列已满，则将阻塞直到有空间；
  - `take`：从队首取元素，若队列为空，则将阻塞直到有元素可用；
- 阻塞队列实现：

| 名称                    | 说明              |
|-----------------------|-----------------|
| ArrayBlockingQueue    | FIFO 队列         |
| LinkedBlockingQueue   | FIFO 队列         |
| PriorityBlockingQueue | 按优先级排序的队列       |
| SynchronousQueue      | 不会为队列中的元素维护存储空间 |

#### 1.2 关键参数
- `corePoolSize`：线程池核心线程数量，即使线程空闲，也不会销毁。除非设置`allowCoreThreadTimeOut`；
- `maximumPoolSize`：线程池最大线程数量；
- `keepAliveTime`：当线程数量超过`corePoolSize`后，空闲线程存活时间；
- `timeUnit`：`keepAliveTime` 的单位；
- `workQueue`：存放待执行任务的队列。当提交的任务数超过核心线程数大小后，再提交的任务就存放在工作队列，任务调度时再从队列中取出任务；
- `threadFactory`：创建线程的工厂；
- `handler`：当线程池线程数已满，并且工作队列达到饱和，新提交的任务使用拒绝策略处理。

```java
// ThreadPoolExecutor 构造参数如下
public ThreadPoolExecutor(int corePoolSize,
						  int maximumPoolSize,
						  long keepAliveTime,
						  TimeUnit unit,
						  BlockingQueue<Runnable> workQueue,
						  ThreadFactory threadFactory,
						  RejectedExecutionHandler handler) {
     // 省略
    }
```

#### 1.2.1 任务拒绝策略
| 名称                  | 说明                                           |
|---------------------|----------------------------------------------|
| AbortPolicy         | 丢弃任务并抛出`RejectedExecutionException`，默认的拒绝策略。 |
| DiscardPolicy       | 丢弃任务，但是不抛出异常。                                |
| DiscardOldestPolicy | 丢弃队列最前面的任务，然后重新提交被拒绝的任务。                     |
| CallerRunsPolicy    | 由调用线程（提交任务的线程）处理该任务，从而降低新任务的流量           |

### 2. 任务调度`execute()`
- `Executor` 基于生产者-消费者模式，提交任务的操作相当于生产者（生产待完成的工作单元），执行任务的线程则相当于消费者（执行完这些工作单元）。任务的流转分为三种情况：
  - 直接申请线程执行该任务；
  - 缓冲到队列中等待线程执行；
  - 拒绝该任务

[ThreadPoolExecutor 任务调度流程](https://github.com/xianliu18/ARTS/tree/master/jvm/threadpool/images/任务调度流程.png)

```java
// 执行任务的方法
public void execute(Runnable command) {
        if (command == null)
            throw new NullPointerException();
        
		int c = ctl.get();
		// 当前线程池中工作的线程数量小于核心线程数，直接创建新的线程执行任务
        if (workerCountOf(c) < corePoolSize) {
            if (addWorker(command, true))
                return;
            c = ctl.get();
        }
		// 若当前线程池数量达到核心线程数
		// 线程池处于运行状态，将任务添加到队列中，等待线程池调度
        if (isRunning(c) && workQueue.offer(command)) {
            int recheck = ctl.get();
            if (! isRunning(recheck) && remove(command))
                reject(command);
            else if (workerCountOf(recheck) == 0)
                addWorker(null, false);
        }
		// 当前线程池数量达到核心线程数且队列已满，则再次尝试创建新的线程
        else if (!addWorker(command, false))
			// 若执行失败，则执行拒绝策略
            reject(command);
    }
```

#### 2.1 线程池运行状态
[线程池运行状态](https://github.com/xianliu18/ARTS/tree/master/jvm/threadpool/images/线程池状态.png)

- `RUNNING`：可以接收新的任务并且处理队列中的任务；
- `SHUTDOWN`：不再接受新的任务，但是可以处理队列中的任务；
- `STOP`：不再接受新的任务，不再处理队列中的任务，并且中断正在运行中的任务；
- `TIDYING`：所有的任务均已终止，workCount 为 0，将会运行`terminated()`方法；
- `TERMINATED`：`terminated()`方法运行结束；

### 3. Worker 线程管理
- 线程池为了掌握线程的状态，维护线程的生命周期，设计了`Worker`内部类。
```java
private final class Worker extends AbstractQueuedSynchronizer implements Runnable {
	final Thread thread; // worker 持有的线程
	Runnable firstTask;  // 初始化的任务

	// 构造函数
	Worker(Runnable firstTask) {
		setState(-1); // inhibit interrupts until runWorker
		this.firstTask = firstTask;
		this.thread = getThreadFactory().newThread(this);
	}
}
```

- `Worker` 通过继承 AQS，来实现非公平锁，独占锁（不可重入）；没有使用可重入锁`ReentrantLock`，为的就是实现不可重入的特性去反应线程现在的执行状态。
  - `lock` 方法一旦获取了独占锁，表示当前线程正在执行任务中，则不应该中断线程；
  - 如果该线程现在不是独占锁状态，说明没有在处理任务，即处于空闲中，可以对该线程进行中断；
  - 线程池在执行`shutdown`或`tryTerminate`方法时，会调用`interruptIdleWorkers`方法来中断空闲线程。

```java
final void runWorker(Worker w) {
	Thread wt = Thread.currentThread();
	Runnable task = w.firstTask;
	w.firstTask = null;
	w.unlock(); // allow interrupts
	boolean completedAbruptly = true;
	try {
		while (task != null || (task = getTask()) != null) {
			// 执行任务之前，先获取独占锁
			w.lock();
			// If pool is stopping, ensure thread is interrupted;
			// if not, ensure thread is not interrupted.  This
			// requires a recheck in second case to deal with
			// shutdownNow race while clearing interrupt
			if ((runStateAtLeast(ctl.get(), STOP) ||
					(Thread.interrupted() &&
					runStateAtLeast(ctl.get(), STOP))) &&
				!wt.isInterrupted())
				wt.interrupt();
			try {
				beforeExecute(wt, task);
				Throwable thrown = null;
				try {
					task.run();
				} catch (RuntimeException x) {
					thrown = x; throw x;
				} catch (Error x) {
					thrown = x; throw x;
				} catch (Throwable x) {
					thrown = x; throw new Error(x);
				} finally {
					afterExecute(task, thrown);
				}
			} finally {
				task = null;
				w.completedTasks++;
				w.unlock();
			}
		}
		completedAbruptly = false;
	} finally {
		processWorkerExit(w, completedAbruptly);
	}
}

// 中断空闲线程
private void interruptIdleWorkers(boolean onlyOne) {
        final ReentrantLock mainLock = this.mainLock;
        mainLock.lock();
        try {
            for (Worker w : workers) {
                Thread t = w.thread;
                if (!t.isInterrupted() && w.tryLock()) {
                    try {
                        t.interrupt();
                    } catch (SecurityException ignore) {
                    } finally {
                        w.unlock();
                    }
                }
                if (onlyOne)
                    break;
            }
        } finally {
            mainLock.unlock();
        }
    }
```

### 3. 线程池的实践中的问题
- CPU 密集型：主要消耗 CPU 资源，要进行大量的计算；
- IO 密集型：涉及到网络、磁盘 IO 的任务；CPU 消耗很少，任务的大部分时间都在等待 IO 操作完成；
- 线程池参数动态化，增加线程池监控，方便掌握线程池状态；

### 4. SpringBoot 中的线程池
- `ThreadPoolTaskExecutor`：Spring 的线程池；

<br/>

**参考资料：**
- [Java 并发编程实战](https://book.douban.com/subject/10484692/)
- [Java 线程池实现原理及其在美团业务中的实践](https://tech.meituan.com/2020/04/02/java-pooling-pratice-in-meituan.html)
- [学会 Java 中的线程池](https://www.cnblogs.com/wang-meng/p/12945703.html)
- [Java 线程池七个参数详解](https://blog.csdn.net/Anenan/article/details/115603481)