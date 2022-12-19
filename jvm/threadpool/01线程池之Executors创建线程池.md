### 线程池之 Executors 创建线程池
### 1，线程池的作用
线程池解决的核心问题就是资源管理问题。除了线程池，还有如下几种“池化”场景：
- 内存池(Memory Pooling)
- 连接池(Connection Pooling)
- 实例池(Object Pooling)

使用线程池，可以带来如下好处：
- **降低资源消耗：** 线程池中重复利用已创建的线程，降低线程创建和销毁造成的损耗；
- **提高响应速度：** 任务到达时，无需等待线程创建即可立即执行；
- **提高线程的可管理性：** 线程是稀缺资源，如果无限制的创建，不仅会消耗系统资源，还会降低系统的稳定性。使用线程池可以进行统一的分配、调优和监控。

### 2，如何创建线程池
#### 2.1 【阿里巴巴 Java 开发手册】对线程池的建议：
- **【强制】** 线程池不允许使用 `Executors` 去创建，而是通过 `ThreadPoolExecutor` 的方式，这样的处理方式让写的同学更加明确线程池的运行规则，规避资源耗尽的风险。
```java
说明：Executors 返回的线程池对象的弊端如下：

1）FixedThreadPool 和 SingleThreadPool：
	允许的请求队列长度为 Integer.MAX_VALUE， 可能会堆积大量的请求，从而导致 OOM
2）CachedThreadPool 和 ScheduledThreadPool：
	允许的创建线程数量为 Integer.MAX_VALUE，可能会创建大量的线程，从而导致 OOM
```

#### 2.2 `Executors` 创建线程池源码
```java
public class Executors {

	/**
	 * 线程数量固定的线程池
	 */
	 public static ExecutorService newFixedThreadPool(int nThreads) {
		 return new ThreadPoolExecutor(nThreads, nThreads, 
		 								0L, TimeUnit.MILLISECONDS, 
		 								new LinkedBlockingQueue<Runnable>());
	 }

	/**
	 * 创建单线程的线程池
	 */
	public static ExecutorService newSingleThreadExecutor(ThreadFactory threadFactory) {
		return new FinalizableDelegatedExecutorService
				(new ThreadPoolExecutor(1, 1, 0L, TimeUnit.MILLISECONDS,
										new LinkedBlockingQueue<Runnable>(),
										threadFactory));
	}

	/**
	 * 只要有任务，就会创建新的线程去执行。
	 * SynchronousQueue 是一个不存储元素的 BlockingQueue，
	 * 对于提交的任务，如果有空闲线程，就是用空闲线程来处理；否则新建一个线程来处理任务；
	 */
	 public static ExecutorService newCachedThreadPool() {
		 return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
		 								60L, TimeUnit.SECONDS,
										new SynchronousQueue<Runnable>());
	 }

	/**
	 * 创建需要多个后台线程执行周期任务的线程池
	 */
	 public static ScheduledExecutorService newScheduledThreadPool(int corePoolSize) {
		 return new ScheduledThreadPoolExecutor(corePoolSize);
	 }
}

// 类 ScheduledThreadPoolExecutor
public class ScheduledThreadPoolExecutor
			extends ThreadPoolExecutor
			implements ScheduledExecutorService {
	
	public ScheduledThreadPoolExecutor(int corePoolSize) {
		// 父类 ThreadPoolExecutor 的构造方法
		super(corePoolSize, Integer.MAX_VALUE, 0, NANOSECONDS,
				new DelayedWorkQueue());
	}
}
```

### 3，线程池创建示例
- **newFixedThreadPool 使用**
```java
@Slf4j
public class ThreadPoolFixedTest {
    public static void main(String[] args) {
        ExecutorService executorService = Executors.newFixedThreadPool(3);
        for (int i = 1; i < 6; i++) {
            int groupId = i;
            executorService.execute(() -> {
                for (int j = 1; j < 5; j++) {
                    try {
                        Thread.sleep(1000);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    log.info("线程名称：{} =====> 任务编号：[{}], 第 {} 次执行完成", Thread.currentThread().getName(), groupId, j);
                }
            });
        }
        executorService.shutdown();
    }
}
```

- **newSingleThreadExecutor 使用**
```java
@Slf4j
public class ThreadPoolSingleTest {

    public static void main(String[] args) {
        ExecutorService executorService = Executors.newSingleThreadExecutor();
        for (int i = 1; i < 5; i++) {
            int groupId = i;
            executorService.execute(() -> {
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                log.info("线程名称：{} =====> 任务编号：[{}], 第 {} 次执行完成", Thread.currentThread().getName(), groupId);
            });
        }
        executorService.shutdown();
    }

}
```

- **newCachedThreadPool 使用**
```java
@Slf4j
public class ThreadPoolCachedTest {

    public static void main(String[] args) {
        ExecutorService executorService = Executors.newCachedThreadPool();
        for (int i = 1; i < 5; i++) {
            int groupId = i;
            executorService.execute(() -> {
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                log.info("线程名称：{}, 完成任务编号：[{}]", Thread.currentThread().getName(), groupId);
            });
        }
        executorService.shutdown();
    }

}
```

- **newScheduledThreadPool 使用**
```java
@Slf4j
public class ThreadPoolScheduledTest {
    public static void main(String[] args) {
        ScheduledExecutorService executorService = Executors.newScheduledThreadPool(3);
        executorService.schedule(() -> {
            log.info("3秒后开始执行，仅执行一次!");
        }, 3, TimeUnit.SECONDS);

        // sheduleAtFixedRate 是以上一个任务开始的时间计时，period 时间过去后，
        // 检测上一个任务是否执行完毕，如果上一个任务执行完毕，则当前任务立即执行；
        // 如果上一个任务没有执行完毕，则需要等待上一个任务执行完毕后，立即执行

        // 3秒后开始执行,以后每2秒执行一次
        executorService.scheduleAtFixedRate(() -> {
            log.info("scheduleAtFixedRate 开始执行~");
            try {
                TimeUnit.SECONDS.sleep(5);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            log.info("scheduleAtFixedRate 执行结束~");
        }, 3, 2, TimeUnit.SECONDS);

        // sheduleWithFixedDelay: 是以上一个任务结束时开始计时，delay 时间过去后，立即执行

        // 3秒开始执行,后续延迟2秒
        executorService.scheduleWithFixedDelay(() -> {
            log.info("scheduleWithFixedDelay 开始执行~");
            try {
                TimeUnit.SECONDS.sleep(10);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            log.info("scheduleWithFixedDelay 执行结束~");
        }, 3, 2, TimeUnit.SECONDS);
    }
}
```

<br/>

**参考资料：**
- [scheduleAtFixedRate 与 sheduleWithFixedDelay 的区别](https://blog.csdn.net/u013819945/article/details/47723091)
- [线程池的介绍和使用](https://bugstack.cn/md/java/interview/2020-12-16-面经手册%20·%20第22篇《线程池的介绍和使用，以及基于jvmti设计非入侵监控》.html)