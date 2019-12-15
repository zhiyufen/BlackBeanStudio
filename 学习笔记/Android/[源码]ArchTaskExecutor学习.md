# [源码]ArchTaskExecutor学习

## 简介
该笔记在学习Jetpack组件源码时，遇到的一些系统工具，有些虽然不能调用(只能内部使用)；
但学习一下人家的写法，还是大有好处的，因此在这里进行学习及做笔记；


### ArchTaskExecutor学习
ArchTaskExecutor位于androidx.arch.core.executor包的工具类，是一个能把Runnable放进UI线程或者子线程的线程工具类；

用法如下：
```java
//放进UI线程运行
ArchTaskExecutor.getInstance().postToMainThread(mPostValueRunnable);
ArchTaskExecutor.getInstance().executeOnMainThread(mPostValueRunnable);
	
//放进子线程运行
ArchTaskExecutor.getInstance().executeOnDiskIO(mPostValueRunnable);
```

除了上面这种，还可以通过静态方法来获取对应特性的线程来运行：
```java
//放进UI线程运行
getMainThreadExecutor().execute(mPostValueRunnable);
	
//放进子线程运行
getIOThreadExecutor().execute(mPostValueRunnable);
```

两种用法在本质上是相同的，只是用法不一样而已；

我们先看看TaskExecutor接口，因为ArchTaskExecutor是实现它的；
```java
@RestrictTo(RestrictTo.Scope.LIBRARY_GROUP)
public abstract class TaskExecutor {
    /**
     * Executes the given task in the disk IO thread pool.
     *
     * @param runnable The runnable to run in the disk IO thread pool.
     */
    public abstract void executeOnDiskIO(@NonNull Runnable runnable);

    /**
     * Posts the given task to the main thread.
     *
     * @param runnable The runnable to run on the main thread.
     */
    public abstract void postToMainThread(@NonNull Runnable runnable);

    /**
     * Executes the given task on the main thread.
     * <p>
     * If the current thread is a main thread, immediately runs the given runnable.
     *
     * @param runnable The runnable to run on the main thread.
     */
    public void executeOnMainThread(@NonNull Runnable runnable) {
        if (isMainThread()) {
            runnable.run();
        } else {
            postToMainThread(runnable);
        }
    }

    /**
     * Returns true if the current thread is the main thread, false otherwise.
     *
     * @return true if we are on the main thread, false otherwise.
     */
    public abstract boolean isMainThread();
}
```
该接口定义线程工具应具有的特性: 放进UI线程运行/放进子线程运行/是否UI线程等；

虽然ArchTaskExecutor实现TaskExecutor接口，但只是一个对外代理类而已，实际真正实现TaskExecutor接口是DefaultTaskExecutor类；

```java

    @NonNull
    private static final Executor sMainThreadExecutor = new Executor() {
        @Override
        public void execute(Runnable command) {
            getInstance().postToMainThread(command);
        }
    };

    @NonNull
    private static final Executor sIOThreadExecutor = new Executor() {
        @Override
        public void execute(Runnable command) {
            getInstance().executeOnDiskIO(command);
        }
    };

    private ArchTaskExecutor() {
        mDefaultTaskExecutor = new DefaultTaskExecutor();
        mDelegate = mDefaultTaskExecutor;
    }

    /**
     * Returns an instance of the task executor.
     *
     * @return The singleton ArchTaskExecutor.
     */
    @NonNull
    public static ArchTaskExecutor getInstance() {
        if (sInstance != null) {
            return sInstance;
        }
        synchronized (ArchTaskExecutor.class) {
            if (sInstance == null) {
                sInstance = new ArchTaskExecutor();
            }
        }
        return sInstance;
    }
```
可以看到ArchTaskExecutor是单例的，同时静态变量sMainThreadExecutor， sIOThreadExecutor的exceute方法也是直接使用到getInstance()；

不过不太明白，为什么要同时用两个用法？也许为了让使用者能更灵活的使用；

先不管，我们来学习直接的实现DefaultTaskExecutor类；

DefaultTaskExecutor会创建一个拥有两个线程的线程池，以便提供在子线程的运行环境：
```java
    private final ExecutorService mDiskIO = Executors.newFixedThreadPool(2, new ThreadFactory() {
        private static final String THREAD_NAME_STEM = "arch_disk_io_%d";
        //原子操作，适合在高并发的场景下；
        private final AtomicInteger mThreadId = new AtomicInteger(0);

        @Override
        public Thread newThread(Runnable r) {
            Thread t = new Thread(r);
            t.setName(String.format(THREAD_NAME_STEM, mThreadId.getAndIncrement()));
            return t;
        }
    });

    @Override
    public void executeOnDiskIO(Runnable runnable) {
        mDiskIO.execute(runnable);
    }

```

对于UI线程的， 则创建一个持有MainLooper的Handler来处理：
```java
    private final Object mLock = new Object();
    
    @Nullable
    private volatile Handler mMainHandler;

    @Override
    public void postToMainThread(Runnable runnable) {
        if (mMainHandler == null) {
            synchronized (mLock) {
                if (mMainHandler == null) {
                    mMainHandler = new Handler(Looper.getMainLooper());
                }
            }
        }
        //noinspection ConstantConditions
        mMainHandler.post(runnable);
    }

    @Override
    public boolean isMainThread() {
        return Looper.getMainLooper().getThread() == Thread.currentThread();
    }
```

整个源码来说，都是很简洁易懂的；

至少为什么不直接使用DefaultTaskExecutor类，这个应该为了以后方便更改内部实现，比如实现不再使用DefaultTaskExecutor作为实现了而更改其它类，那么对于使用ArchTaskExecutor的用户来说，并不需要更改任何东西；


