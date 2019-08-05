#[Jetpack]LiveData组件学习

## 简介
LiveData是Android框架组件中的组成部分，对于数据

特性及使用如下：
1. LiveData是一个具有生命周期感知特性的可观察的数据保持类，使用LiveData保存数据时，在每次订阅或数据更新时会自动回调设置的观察者从而更新数据，真正的实现了数据驱动的效果；

2. LiveData的创建基本会在ViewModel中，从而使数据在界面销毁时继续保持；

3. LiveData 认为观察者的生命周期处于STARTED状态或RESUMED状态下时，表示观察者处于活跃状态，并且LiveData只通知活跃的观察者更新数据；

4. 注册一个实现该LifecycleOwner 接口的对象配对的观察者，当相应Lifecycle对象的状态改变为DESTROYED时移除观察者；

## 基本使用

1. 在ViewModel中创建LiveData实例， 请注意由于ViewModel的生命周期比View还长， 因此ViewModel不应该持有任何View引用；
```java
public class MyViewModel extends AndroidViewModel {
    public static final String TAG = "JetPack/MyViewModel";
    public MutableLiveData<String> mMutableLiveData;

    public MyViewModel(@NonNull Application application) {
        super(application);
        //LiveData Test
        mMutableLiveData = new MutableLiveData<>();
        mMutableLiveData.postValue("xxx1");
    }

    @Override
    protected void onCleared() {
        Log.d(TAG, "onCleared...");
    }
}
```
2. 在实现LifecycleOwner的实例中，创建ViewModel及对LiveData进行订阅：

```java
public class JetPackActivity extends AppCompatActivity {
    public static final String TAG = "JetPackActivity";
    private MyViewModel mViewModel;
    private TextView mTextView;

    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        Log.d(TAG, "onCreate...");
        super.onCreate(savedInstanceState);
        setContentView(R.layout.jetpack_main_layout);

        //Lifecycle Test
        getLifecycle().addObserver(new ActivityLifecycleObserver());

        //ViewModel Test
        mViewModel = ViewModelProviders.of(JetPackActivity.this).get(MyViewModel.class);
        Log.d(TAG, "mViewModel = " + mViewModel.toString());

        findViewById(R.id.jetpack_change_text_button).setOnClickListener(view -> {
            mViewModel.mMutableLiveData.postValue("xxx2");
        });

        mViewModel.mMutableLiveData.observe(this, new Observer<String>() {
            @Override
            public void onChanged(String s) {
                Log.d(TAG, "The content is change to: " + s);
            }
        });
    }
}
```
LiveData.observe()方法中，第一个参数为实现LifecycleOwner的实例， 第二个为当数据变化时的通知回调；
使用observe()方法，只有在Lifecycle生命周期为处于STARTED状态或RESUMED状态下时，才能监听到数据的变化；如果需要不管什么状态下都监听到的话，就需要使用observeForever(observer)方法，但该需要不会主动释放订阅，需要调用removeObserver方法进行释放；

这样基本使用的学习就结束了，用法很简洁，下面我们一边学习源码，一边看一下其中的注意事项；

## 源码学习

我们先看看LiveData.observe()方法的源码：
```java
    @MainThread
    public void observe(@NonNull LifecycleOwner owner, @NonNull Observer<? super T> observer) {
        assertMainThread("observe");
        if (owner.getLifecycle().getCurrentState() == DESTROYED) {
            // ignore
            return;
        }
        LifecycleBoundObserver wrapper = new LifecycleBoundObserver(owner, observer);
        ObserverWrapper existing = mObservers.putIfAbsent(observer, wrapper);
        if (existing != null && !existing.isAttachedTo(owner)) {
            throw new IllegalArgumentException("Cannot add the same observer"
                    + " with different lifecycles");
        }
        if (existing != null) {
            return;
        }
        owner.getLifecycle().addObserver(wrapper);
    }
```
从上面的源码，我们知道几个信息：

1. LiveData.observe()必须在Main线程上调用，否则会直接抛出异常；

2. LiveData会组合Lifecycle(LifecycleBoundObserver)使用，进行管理数据及订阅者的生命周期；

3. LiveData会把订阅者的Observer与LifecycleObserver放入mObservers数据中：
```java
    private SafeIterableMap<Observer<? super T>, ObserverWrapper> mObservers =
            new SafeIterableMap<>();
```

我们来学习一下LifecycleBoundObserver；

LifecycleBoundObserver是LiveData的内部类，继承于ObserverWrapper并实现GenericLifecycleObserver接口；

LifecycleBoundObserver的作用的就是根据Lifecyele的生命周期的变化，从而决定当前订阅者的活跃状态及是否去掉订阅者；

```java
        @Override
        boolean shouldBeActive() {
			//设置Lifecycle的生命周期状态至少是STARTED或以上才是有效的
            return mOwner.getLifecycle().getCurrentState().isAtLeast(STARTED);
        }

        @Override
        public void onStateChanged(LifecycleOwner source, Lifecycle.Event event) {
			//ifecycle的生命周期状态是DESTROYED进行注销订阅
            if (mOwner.getLifecycle().getCurrentState() == DESTROYED) {
                removeObserver(mObserver);
                return;
            }
			//更新该订阅者的LiveData活跃状态
            activeStateChanged(shouldBeActive());
        }

```

activeStateChanged()方法是ObserverWrapper父类的：

```java
        void activeStateChanged(boolean newActive) {
            if (newActive == mActive) {
                return;
            }
            // immediately set active state, so we'd never dispatch anything to inactive
            // owner
            mActive = newActive;
			//mActiveCount为LiveData里的活跃状态下的数量
            boolean wasInactive = LiveData.this.mActiveCount == 0;
            LiveData.this.mActiveCount += mActive ? 1 : -1;
            if (wasInactive && mActive) {
                onActive();//LiveData第一次转跃状态时，目前该方法为空
            }
            if (LiveData.this.mActiveCount == 0 && !mActive) {
                onInactive();//LiveData里所有订阅转为不跃状态时，目前该方法为空
            }
            if (mActive) {
				//通知值更新(这也就是我们前面说当进行调用oberser方法时，也会进行回调值的变化的)
                dispatchingValue(this);
            }
        }
```
如果我们需要继承LiveData来实例自己的功能时， onActive()和onInactive()也许就能用上了；

接下来我们再来看看MutableLiveData类，MutableLiveData类其实只是LiveData的代理而已， 它提供了两个方法和我们来设置数据的值，我们通过这两个方法来学习LiveData是如何在数据变化时通知订阅者的；

```java
    @Override
    public void postValue(T value) {
        super.postValue(value);
    }

    @Override
    public void setValue(T value) {
        super.setValue(value);
    }
```

1. setValue()方法必须在 主线程 进行调用，
2. postValue()方法更适合在子线程中进行调用,它会把设置值和调用回调放在主

看看源码
```java
    @MainThread
    protected void setValue(T value) {
		//必须UI线程
        assertMainThread("setValue");
		//每次更新数据都会增加版本；
        mVersion++;
        mData = value;
		//进行通知所有订阅者
        dispatchingValue(null);
    }

    @SuppressWarnings("WeakerAccess") /* synthetic access */
    void dispatchingValue(@Nullable ObserverWrapper initiator) {
        if (mDispatchingValue) {
			//如果当前正在通知订阅者中的话，置原来的数据为无效；
            mDispatchInvalidated = true;
            return;
        }
        mDispatchingValue = true;
        do {
            mDispatchInvalidated = false;
            if (initiator != null) {
                considerNotify(initiator);
                initiator = null;
            } else {
                for (Iterator<Map.Entry<Observer<? super T>, ObserverWrapper>> iterator =
                        mObservers.iteratorWithAdditions(); iterator.hasNext(); ) {
					//进行通知
                    considerNotify(iterator.next().getValue());
                    if (mDispatchInvalidated) {//有新数据更新了，退出当前通知
                        break;
                    }
                }
            }
        } while (mDispatchInvalidated);//有新数据，重新进行一一通知；
        mDispatchingValue = false;
    }
	
	private void considerNotify(ObserverWrapper observer) {
        if (!observer.mActive) {
            return;
        }
        // Check latest state b4 dispatch. Maybe it changed state but we didn't get the event yet.
        //
        // we still first check observer.active to keep it as the entrance for events. So even if
        // the observer moved to an active state, if we've not received that event, we better not
        // notify for a more predictable notification order.
		
		//是否为活跃状态
        if (!observer.shouldBeActive()) {
            observer.activeStateChanged(false);
            return;
        }
		//ObserverWrapper会持有当前已经更新的数据的版本，如果当前进行通知数据版本过低，不通知；
        if (observer.mLastVersion >= mVersion) {
            return;
        }
        observer.mLastVersion = mVersion;
        //noinspection unchecked
        observer.mObserver.onChanged((T) mData);
    }
```

整个更新逻辑很简单， 至于PostValue()的区别， 只不过创建一个Runnable，然后让其运行的UI线程中；
```
    protected void postValue(T value) {
        boolean postTask;
        synchronized (mDataLock) {
            postTask = mPendingData == NOT_SET;
            mPendingData = value;
        }
        if (!postTask) {
            return;
        }
        ArchTaskExecutor.getInstance().postToMainThread(mPostValueRunnable);
    }
	
	private final Runnable mPostValueRunnable = new Runnable() {
        @Override
        public void run() {
            Object newValue;
            synchronized (mDataLock) {
                newValue = mPendingData;
                mPendingData = NOT_SET;
            }
            //noinspection unchecked
			//在UI线程中可直接调用setValue();
            setValue((T) newValue);
        }
    };
```

DefaultTaskExecutor的postToMainThread()方法；

```java
    @Override
    public void postToMainThread(Runnable runnable) {
        if (mMainHandler == null) {
            synchronized (mLock) {
                if (mMainHandler == null) {
					//获取UI线程的MainLooper；
                    mMainHandler = new Handler(Looper.getMainLooper());
                }
            }
        }
        //noinspection ConstantConditions
        mMainHandler.post(runnable);
    }
```

到这里LiveData的学习就结束了，具体运行需要后面慢慢摸了；



参考文章：
> [Android官方架构组件LiveData: 观察者模式领域二三事](https://juejin.im/post/5c25753af265da61561f5335)

> [Android Jetpack架构组件之 LiveData（使用、源码篇）](https://blog.csdn.net/Alexwll/article/details/82996003)
