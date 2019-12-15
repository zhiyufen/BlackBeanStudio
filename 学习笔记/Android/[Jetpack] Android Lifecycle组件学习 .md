## 背景
略(自行百度一下)
 
## 配置
Android： 
```xml
ependencies {
    def lifecycle_version = "1.1.1"

    // ViewModel and LiveData
    implementation "android.arch.lifecycle:extensions:$lifecycle_version"
    // alternatively - just ViewModel
    implementation "android.arch.lifecycle:viewmodel:$lifecycle_version" // use -ktx for Kotlin
    // alternatively - just LiveData
    implementation "android.arch.lifecycle:livedata:$lifecycle_version"
    // alternatively - Lifecycles only (no ViewModel or LiveData).
    //     Support library depends on this lightweight import
    implementation "android.arch.lifecycle:runtime:$lifecycle_version"

    annotationProcessor "android.arch.lifecycle:compiler:$lifecycle_version" // use kapt for Kotlin
    // alternately - if using Java8, use the following instead of compiler
    implementation "android.arch.lifecycle:common-java8:$lifecycle_version"

    // optional - ReactiveStreams support for LiveData
    implementation "android.arch.lifecycle:reactivestreams:$lifecycle_version"

    // optional - Test helpers for LiveData
    testImplementation "android.arch.core:core-testing:$lifecycle_version"
}
```

androidX： 
```xml
dependencies {
    def lifecycle_version = "2.0.0"

    // ViewModel and LiveData
    implementation "androidx.lifecycle:lifecycle-extensions:$lifecycle_version"
    // alternatively - just ViewModel
    implementation "androidx.lifecycle:lifecycle-viewmodel:$lifecycle_version" // For Kotlin use lifecycle-viewmodel-ktx
    // alternatively - just LiveData
    implementation "androidx.lifecycle:lifecycle-livedata:$lifecycle_version"
    // alternatively - Lifecycles only (no ViewModel or LiveData). Some UI
    //     AndroidX libraries use this lightweight import for Lifecycle
    implementation "androidx.lifecycle:lifecycle-runtime:$lifecycle_version"

    annotationProcessor "androidx.lifecycle:lifecycle-compiler:$lifecycle_version" // For Kotlin use kapt instead of annotationProcessor
    // alternately - if using Java8, use the following instead of lifecycle-compiler
    implementation "androidx.lifecycle:lifecycle-common-java8:$lifecycle_version"

    // optional - ReactiveStreams support for LiveData
    implementation "androidx.lifecycle:lifecycle-reactivestreams:$lifecycle_version" // For Kotlin use lifecycle-reactivestreams-ktx

    // optional - Test helpers for LiveData
    testImplementation "androidx.arch.core:core-testing:$lifecycle_version"
}
```
更新配置请参考官方版本发布： https://developer.android.com/jetpack/androidx/releases/lifecycle；

Lifecycle 被包含在 support library 26.1.0 及之后的依赖包中，如果我们的项目依赖的支持库版本在 26.1.0及以上，那么不需要额外导入 Lifecycle 库; 

如果支持库版本小于 26.1.0 ，就需要单独导入 Lifecycle 库 ：
```xml
implementation "android.arch.lifecycle:runtime:1.1.1"
```



注：从android9.0 ，API28开始, android的support库将会进行改进, V7: 28.0.0将会是support库的终结版本。未来新的特性和改进都会进入Androidx包；更详细请参考https://blog.csdn.net/csdn_aiyang/article/details/80859771；

## Lifecycle组件基本使用

Lifecycle是一个包含组件生命周期状态（如activity或fragment）信息的类，并允许其他对象观察该状态。
Lifecycle主要使用两个枚举来跟踪其相关组件的生命周期状态：Event, State; 

Event: 表示来自于framework和Lifecycle类的分派的生命周期事件，映射来自Activity和Fragment的回调事件；
State: 表示Lifecycle对象的当前状态；

事件和状态流程图：
![Image](https://developer.android.com/images/topic/libraries/architecture/lifecycle-states.svg)

从图中也可以看到Events和State对应的枚举字符；

#### 简单使用

当你已经使用AppCompatActivity或者Fragment时，不需要额外实现LifecycleOwner接口，因为这两个类已经实现LifecycleOwner接口了；

简单使用步骤： 

1, 添加一个类实现LifecycleObserver接口；
```java
public class ActivityLifecycleObserver implements LifecycleObserver {
    public static final String TAG = "ActivityLifecycle";

    @OnLifecycleEvent(Lifecycle.Event.ON_CREATE)
    public void onCreate(LifecycleOwner lifecycleOwner) {
        Log.d(TAG, "onCreate: " + lifecycleOwner.getLifecycle().getCurrentState());
    }
    @OnLifecycleEvent(Lifecycle.Event.ON_START)
    public void onStart(LifecycleOwner lifecycleOwner) {
        Log.d(TAG, "onStart: " + lifecycleOwner.getLifecycle().getCurrentState());
    }

    @OnLifecycleEvent(Lifecycle.Event.ON_RESUME)
    public void onResume(LifecycleOwner lifecycleOwner) {
        Log.d(TAG, "onResume: " + lifecycleOwner.getLifecycle().getCurrentState());
    }


    @OnLifecycleEvent(Lifecycle.Event.ON_PAUSE)
    public void onPause(LifecycleOwner lifecycleOwner) {
        Log.d(TAG, "onPause: " + lifecycleOwner.getLifecycle().getCurrentState());
    }

    @OnLifecycleEvent(Lifecycle.Event.ON_STOP)
    public void onStop(LifecycleOwner lifecycleOwner) {
        Log.d(TAG, "onStop: " + lifecycleOwner.getLifecycle().getCurrentState());
    }

    @OnLifecycleEvent(Lifecycle.Event.ON_DESTROY)
    public void onDestroy(LifecycleOwner lifecycleOwner) {
        Log.d(TAG, "onDestroy: " + lifecycleOwner.getLifecycle().getCurrentState());
    }
}
```

还有一种方式就是直接实现DefaultLifecycleObserver接口， 然后重写里面生命周期方法；
推荐使用第二种方式，性能会更好，至于理由，我们下面看源码再解释；

2, 在Activity中注册该监听对象；

```java
public class JetPackActivity extends AppCompatActivity {

    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        getLifecycle().addObserver(new ActivityLifecycleObserver());
    }
}
```
这样我们ActivityLifecycleObserver就可以监听到Activity的生命周期的回调了；是不是很简洁方便呢。


#### 进阶使用
当我们使用场景是普通Activity或者需要使用这个Lifecycle来完善一些其它功能的生命周期管理时；我们就可以自己来实现LifecycleOwner接口了；
在之前我们需要了解几个接口：

##### Lifecycle接口
```java
public interface LifecycleOwner {
    /**
     * Returns the Lifecycle of the provider.
     *
     * @return The lifecycle of the provider.
     */
    @NonNull
    Lifecycle getLifecycle();
}

```

该接口抽象出Lifecycle的所有权，并允许编写与其共同工作的组件。任何自定义应用程序类都可以实现LifecycleOwner接口。

在普通的Activity上实现一个自己的LifecycleOwner， 我们使用已经实现Lifecycle接口的LifecycleRegistry类；

```java
public class MyLifecycleActivity extends Activity implements LifecycleOwner {
    private LifecycleRegistry mLifecycleRegistry;

    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        mLifecycleRegistry = new LifecycleRegistry(this);
        mLifecycleRegistry.markState(Lifecycle.State.CREATED);
    }

    @Override
    protected void onResume() {
        super.onResume();
        mLifecycleRegistry.markState(Lifecycle.State.RESUMED);
    }

    @Override
    protected void onPause() {
        super.onPause();
        Log.d("MyLifecycleActivity", "onPause: " + mLifecycleRegistry.getCurrentState());
    }

    @Override
    protected void onStop() {
        super.onStop();
        mLifecycleRegistry.markState(Lifecycle.State.CREATED);
    }

    @Override
    protected void onDestroy() {
        super.onDestroy();
        mLifecycleRegistry.markState(Lifecycle.State.DESTROYED);
    }

    @NonNull
    @Override
    public Lifecycle getLifecycle() {
        return mLifecycleRegistry;
    }
}
```
但从上面监听的log来看，使用LifecycleRegistry的话， 更改markState()时， 不完成按Activity生命周期来的话， 状态有点不正确；

```xml
2019-07-25 19:20:29.653 12652-12652/blackbean.rxjavademo D/ActivityLifecycle: onCreate: CREATED
2019-07-25 19:20:29.679 12652-12652/blackbean.rxjavademo D/ActivityLifecycle: onStart: RESUMED
2019-07-25 19:20:29.679 12652-12652/blackbean.rxjavademo D/ActivityLifecycle: onResume: RESUMED
2019-07-25 19:20:31.891 12652-12652/blackbean.rxjavademo D/MyLifecycleActivity: onPause: RESUMED 
//在MyLifecycleActivity中onPause()中，并没有更新状态为CREATED，但这里已经更新为onStop更新的状态CREATED了，另外按理也不应该调用ActivityLifecycle.onPause()；
2019-07-25 19:20:32.489 12652-12652/blackbean.rxjavademo D/ActivityLifecycle: onPause: CREATED 
2019-07-25 19:20:32.490 12652-12652/blackbean.rxjavademo D/ActivityLifecycle: onStop: CREATED
2019-07-25 19:20:32.491 12652-12652/blackbean.rxjavademo D/ActivityLifecycle: onDestroy: DESTROYED
```
注： 如果你需要管理整个应用程序的生命周期的，请查阅 ProcessLifecycleOwner；

带着上面的疑问，我们来学习一下源码：

## Lifecycle组件源码实现

我们参考大神的类图：https://blog.csdn.net/zhuzp_blog/article/details/78871374
![Image1](https://img-blog.csdnimg.cn/20190117102651379.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NkX3podXpoaXBlbmc=,size_16,color_FFFFFF,t_70)

我们以JetPackActivity为例，看看是如何工作的；
注： AndroidX的源码是有些区别的， 比如AndroidX的androidx.fragment.app.Fragment是实现LifecycleOwner接口的，但android.app.Fragment并没有；
```java
public class JetPackActivity extends AppCompatActivity {

    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        getLifecycle().addObserver(new ActivityLifecycleObserver());
    }
}
```

1， 首先AppCompatActivity的父类ComponentActivity实现了LifecycleOwner接口；
注： AppCompatActivity 继承于 FragmentActivity 继承于 ComponentActivity 继承于Activity；

```java
public class ComponentActivity extends Activity implements LifecycleOwner, Component {
    private LifecycleRegistry mLifecycleRegistry = new LifecycleRegistry(this);
	
	//在创建SupportActivity时， 会专门创建一个ReportFragment来处理生命周期相关事件；
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        ReportFragment.injectIfNeededIn(this);
    }
	
	//更新Lifecycle状态
    protected void onSaveInstanceState(Bundle outState) {
        this.mLifecycleRegistry.markState(State.CREATED);
        super.onSaveInstanceState(outState);
    }
	 
	 
	public Lifecycle getLifecycle() {
        return this.mLifecycleRegistry;
    }

}

@RestrictTo(RestrictTo.Scope.LIBRARY_GROUP)
public class ReportFragment extends Fragment {
    private static final String REPORT_FRAGMENT_TAG = "android.arch.lifecycle"
            + ".LifecycleDispatcher.report_fragment_tag";

	//创建一个ReportFragment(无界面)并加入FragmentManager中；
    public static void injectIfNeededIn(Activity activity) {
        // ProcessLifecycleOwner should always correctly work and some activities may not extend
        // FragmentActivity from support lib, so we use framework fragments for activities
        android.app.FragmentManager manager = activity.getFragmentManager();
        if (manager.findFragmentByTag(REPORT_FRAGMENT_TAG) == null) {
            manager.beginTransaction().add(new ReportFragment(), REPORT_FRAGMENT_TAG).commit();
            // Hopefully, we are the first to make a transaction.
            manager.executePendingTransactions();
        }
    }
}
```
注： 这里的Fragment是android.app.Fragment，不是AndroidX的；

从上面看， Lifecycle的接口最终实现是LifecycleRegistry类， ComponentActivity也会创建一个无界面的ReportFragment；ReportFragment会把对应的生命周期发送给LifecycleRegistry进行处理；

```java
package android.arch.lifecycle;


public class ReportFragment extends Fragment {
    private static final String REPORT_FRAGMENT_TAG = "android.arch.lifecycle"
            + ".LifecycleDispatcher.report_fragment_tag";

    public static void injectIfNeededIn(Activity activity) {
        // ProcessLifecycleOwner should always correctly work and some activities may not extend
        // FragmentActivity from support lib, so we use framework fragments for activities
        android.app.FragmentManager manager = activity.getFragmentManager();
        if (manager.findFragmentByTag(REPORT_FRAGMENT_TAG) == null) {
            manager.beginTransaction().add(new ReportFragment(), REPORT_FRAGMENT_TAG).commit();
            // Hopefully, we are the first to make a transaction.
            manager.executePendingTransactions();
        }
    }

    static ReportFragment get(Activity activity) {
        return (ReportFragment) activity.getFragmentManager().findFragmentByTag(
                REPORT_FRAGMENT_TAG);
    }

    @Override
    public void onActivityCreated(Bundle savedInstanceState) {
        super.onActivityCreated(savedInstanceState);
        dispatchCreate(mProcessListener);
        dispatch(Lifecycle.Event.ON_CREATE);
    }

    @Override
    public void onStart() {
        super.onStart();
		//关于ActivityInitializationListener已暂时省略贴代码，因为暂时不确定有什么用，但名字是专门用于监听Activity初始化时的事件的
        dispatchStart(mProcessListener);
        dispatch(Lifecycle.Event.ON_START);
    }

    @Override
    public void onResume() {
        super.onResume();
        dispatchResume(mProcessListener);
        dispatch(Lifecycle.Event.ON_RESUME);
    }

    @Override
    public void onPause() {
        super.onPause();
        dispatch(Lifecycle.Event.ON_PAUSE);
    }

    @Override
    public void onStop() {
        super.onStop();
        dispatch(Lifecycle.Event.ON_STOP);
    }

    @Override
    public void onDestroy() {
        super.onDestroy();
        dispatch(Lifecycle.Event.ON_DESTROY);
        // just want to be sure that we won't leak reference to an activity
        mProcessListener = null;
    }

	//最后事件都会分发LifecycleRegistry进行处理；
    private void dispatch(Lifecycle.Event event) {
        Activity activity = getActivity();
		//LifecycleRegistryOwner已弃用，直接于AppCompatActivity来实现LifecycleOwner接口；
        if (activity instanceof LifecycleRegistryOwner) {
            ((LifecycleRegistryOwner) activity).getLifecycle().handleLifecycleEvent(event);
            return;
        }

        if (activity instanceof LifecycleOwner) {
            Lifecycle lifecycle = ((LifecycleOwner) activity).getLifecycle();
            if (lifecycle instanceof LifecycleRegistry) {
                ((LifecycleRegistry) lifecycle).handleLifecycleEvent(event);
            }
        }
    }
}

```
可以看到ReportFragment分别重写了Fragment的生命周期变化的方法， 但调用时间还有由fragment来决定，我们来看看：

```java
public class Fragment implements ComponentCallbacks2, OnCreateContextMenuListener {
	//省略其他
    void performActivityCreated(Bundle savedInstanceState) {
        onActivityCreated(savedInstanceState);
        if (mChildFragmentManager != null) {
            mChildFragmentManager.dispatchActivityCreated();
        }
    }

    void performStart() {
        //...
        onStart();
        //...
        if (mChildFragmentManager != null) {
            mChildFragmentManager.dispatchStart();
        }
    }

    void performResume() {
        onResume();
        if (mChildFragmentManager != null) {
            mChildFragmentManager.dispatchResume();
            mChildFragmentManager.execPendingActions();
        }
    }

	//需要注意的，从Pause开始，会先处理mChildFragmentManager再处理fragment本身的onPause;
    void performPause() {
        if (mChildFragmentManager != null) {
            mChildFragmentManager.dispatchPause();
        }       
        onPause();
    }

    void performStop() {
        if (mChildFragmentManager != null) {
            mChildFragmentManager.dispatchStop();
        }

        onStop();
    }

    void performDestroy() {
        if (mChildFragmentManager != null) {
            mChildFragmentManager.dispatchDestroy();
        }

        onDestroy();
    }
}

```

至于Activity怎么传递事件到fragment的，我们这里就不提了；

下面学习LifecycleRegistry类是怎么处理事件：handleLifecycleEvent()；

```java
public class LifecycleRegistry extends Lifecycle {

    private static final String LOG_TAG = "LifecycleRegistry";

    //保存所有观察者对象
    private FastSafeIterableMap<LifecycleObserver, ObserverWithState> mObserverMap =
            new FastSafeIterableMap<>();
    //当前最新状态
    private State mState;
	
    //根据事件更新所有观察者状态
    public void handleLifecycleEvent(@NonNull Lifecycle.Event event) {
        //根据当前事件获取对应的状态；
        State next = getStateAfter(event);
        moveToState(next);
    }
	
	//根据事件
    private void moveToState(State next) {
	    //与当前状态相同，不需要处理
        if (mState == next) {
            return;
        }
		//更新当前状态
        mState = next;
		/*
		 * mAddingObserverCounter在添加新观察者时(addObserver())过程中，会+1，等调用sync()后，会置0；
		 * mHandlingEvent在当前方法里调用sync会置true; 
		 * 这两个变量的目的保存sync只有一个在进行中。sync()进行中，有状态更新的话，会根据当前mNewEventOccurred为 true时
		 * sync会停止当前进行处理，重新从头开始；
		 */
        if (mHandlingEvent || mAddingObserverCounter != 0) {
            mNewEventOccurred = true;
            // we will figure out what to do on upper level.
            return;
        }
        mHandlingEvent = true;
        sync();
        mHandlingEvent = false;
    }

    private boolean isSynced() {
        if (mObserverMap.size() == 0) {
            return true;
        }
		//获取mObserverMap里第一个观察者和最后一个的观察者的状态
        State eldestObserverState = mObserverMap.eldest().getValue().mState;
        State newestObserverState = mObserverMap.newest().getValue().mState;
		//判断是最新状态和mObserverMap里的最新、最旧的状态是否一致，一致返回true; 
		//eldestObserverState == newestObserverState为false, 说明前面的sync更新到一半，就停止了，从而不一致；
		//
        return eldestObserverState == newestObserverState && mState == newestObserverState;
    }

    // happens only on the top of stack (never in reentrance),
    // so it doesn't have to take in account parents
    private void sync() {
        LifecycleOwner lifecycleOwner = mLifecycleOwner.get();
        if (lifecycleOwner == null) {
            Log.w(LOG_TAG, "LifecycleOwner is garbage collected, you shouldn't try dispatch "
                    + "new events from it.");
            return;
        }
        while (!isSynced()) {
            mNewEventOccurred = false;
            // no need to check eldest for nullability, because isSynced does it for us.
			//当前最新状态比第一个观察者的状态要小，那么需要把mObserverMap从尾到头进行更新;
            if (mState.compareTo(mObserverMap.eldest().getValue().mState) < 0) {
                backwardPass(lifecycleOwner);
            }
			
            Entry<LifecycleObserver, ObserverWithState> newest = mObserverMap.newest();
			//注：在运行backwardPass()时，有可能mNewEventOccurred已经置为true,所以这里需要检查；如果有更新
			////当前最新状态比最后一个观察者的状态要大，那么需要把mObserverMap从头到尾进行更新；
            if (!mNewEventOccurred && newest != null
                    && mState.compareTo(newest.getValue().mState) > 0) {
                forwardPass(lifecycleOwner);
            }
        }
        mNewEventOccurred = false;
    }
```
简单来讲， 整个处理流程就是对所有观察者（从头到尾或从尾到头）一一进行回调(forwardPass ()或backwardPass())，稍后再讲具体调用；

这里有个小难点在于该类并没有添加synchronized等相关同步机制，那它是分别通过mNewEventOccurred， mHandlingEvent，mAddingObserverCounter三个变量来保存及判断从而保证线程安全；

如果正在添加观察者(mAddingObserverCounter为true)或 在通知观察者中(mHandlingEvent为true)， 则直接设置当前状态为最新的,同时设置mNewEventOccurred为true就可以了，因为在添加完观察者后，它会调用sync来通知观察者新状态 或者在通知观察者过程中，有新状态设置时，会根据当前mNewEventOccurred为 true时；sync会停止当前进行处理，重新从头开始；

另外还有一点需要注意是，在通知所有观察者，我们看到新状态比第一个状态小时，需要从尾到头；新状态比最后一个大时，从头到尾；

比如：

当前状态是 1： 1 --> 1 --> 1 --> 1

然后来了新状态2，2比最后一个要大， 那么从头到尾更新：

2 --> 1 --> 1 --> 1
2 --> 2 --> 1 --> 1

这时候又来一个状态3，3比最后一个要大， 那么从头到尾更新：

3 --> 2 --> 1 --> 1 //可以看到后面有两个观察者是直接跳转该更新的

刚更新了一个3，又来一个1；1比第一个小， 那边从尾到头进行更新；而后面2个状态1， 就不需要再更新；

这双头龙思想的实现目的就是为了在状态快速变化时， 还没来得及更新的观察者直接更新最新状态，从而避免状态在升降时，观察者进行多余的操作，直接可一步到位；
不过对于Fragment的生命周期事件分发时序来讲，一般不存在这种多线程导致状态快速变化；

在学习backwardPass()是怎么从尾到头进行通知观察者的前，我们需要先学习一个ObserverWithState类；
```java
    static class ObserverWithState {
		//当前状态
        State mState;
        GenericLifecycleObserver mLifecycleObserver;

        ObserverWithState(LifecycleObserver observer, State initialState) {
            //
            mLifecycleObserver = Lifecycling.getCallback(observer);
            mState = initialState;
        }

        void dispatchEvent(LifecycleOwner owner, Event event) {
            State newState = getStateAfter(event);
			//没看懂这里为什么取最小；
            mState = min(mState, newState);
			//通知状态变化；
            mLifecycleObserver.onStateChanged(owner, event);
            mState = newState;
        }
    }
```
其中GenericLifecycleObserver是继承于LifecycleObserver接口, 在这里我们暂时只需要知道当状态变化时， 可onStateChanged来通知观察者，最终能调用我们添加的观察者类里对应状态的方法， 我们先往下看：

```java
    private void backwardPass(LifecycleOwner lifecycleOwner) {
        //mObserverMap为所有的观察者的集合，后面再讲怎么添加观察者到mObserverMap的源码；
        Iterator<Entry<LifecycleObserver, ObserverWithState>> descendingIterator =
                mObserverMap.descendingIterator();
        //找到下一个观察者(状态要小于当前最新状态并且刚刚没有更新最新状态)
        while (descendingIterator.hasNext() && !mNewEventOccurred) {
            Entry<LifecycleObserver, ObserverWithState> entry = descendingIterator.next();
            ObserverWithState observer = entry.getValue();
            while ((observer.mState.compareTo(mState) > 0 && !mNewEventOccurred
                    && mObserverMap.contains(entry.getKey()))) {
                //获取观察者当前状态的降级状态(因为当前最新状态是小于它)，这样一步步进行降级调用；
                //比如观察者状态为3，当前最新状态为1 ， 先降级到2， 再循环降级到1为至；
                Event event = downEvent(observer.mState);
				//调用前，把观察者当前状态添加进mParentStates列表中；后面添加观察者时会有用；
                pushParentState(getStateAfter(event));
                observer.dispatchEvent(lifecycleOwner, event);
                //调用后，从mParentStates删除观察者状态；
                popParentState();
            }
        }
    }
```
从上面可以看到，无论是把状态升级还是降级，都是一步步调用的，从而保证一些观察者操作不会丢失；forwardPass方法一样, 只是调用顺序不一样;

这样也能解释到我们在Activity中直接使用LifecycleRegistry时,在changeState时, 有些状态没修改到它,但一样会调用,因为它是一级级来的,因此如果直接使用LifecycleRegistry时需要注意一点;

下面我们学习一个LifecycleRegistry是怎么添加观察者的?
```java
    /**
     * Custom list that keeps observers and can handle removals / additions during traversal.
     *
     * Invariant: at any moment of time for observer1 & observer2:
     * if addition_order(observer1) < addition_order(observer2), then
     * state(observer1) >= state(observer2),
     */
	//其中FastSafeIterableMap 继承于SafeIterableMap是一个可在遍历过程中进行移除或增加成员项的Iterable<Map.Entry<K, V>>， 线程不安全；
    //关于SafeIterableMap，可参考https://blog.csdn.net/c6E5UlI1N/article/details/79608996；
    private FastSafeIterableMap<LifecycleObserver, ObserverWithState> mObserverMap =
            new FastSafeIterableMap<>();
	
    @Override
    public void addObserver(@NonNull LifecycleObserver observer) {
        //创建ObserverWithState对象给新观察者并添加到mObserverMap中；
        State initialState = mState == DESTROYED ? DESTROYED : INITIALIZED;
        ObserverWithState statefulObserver = new ObserverWithState(observer, initialState);
        ObserverWithState previous = mObserverMap.putIfAbsent(observer, statefulObserver);
		//之前已经添加过了；
        if (previous != null) {
            return;
        }
        LifecycleOwner lifecycleOwner = mLifecycleOwner.get();
        if (lifecycleOwner == null) {
            // it is null we should be destroyed. Fallback quickly
            return;
        }
		//是否正在添加观察者中或进行sync方法中；
        boolean isReentrance = mAddingObserverCounter != 0 || mHandlingEvent;
        //计算新观察者需要更新到哪个目标状态：取min(min(观察者当前状态，mObserverMap链表的前一个状态)，mParentStates列表中最新的状态)；
        State targetState = calculateTargetState(observer);
        mAddingObserverCounter++;
        //把新观察者同步到目标状态值；
        while ((statefulObserver.mState.compareTo(targetState) < 0
                && mObserverMap.contains(observer))) {
            pushParentState(statefulObserver.mState);
            statefulObserver.dispatchEvent(lifecycleOwner, upEvent(statefulObserver.mState));
            popParentState();
            // mState / subling may have been changed recalculate
            targetState = calculateTargetState(observer);
        }
		//如果添加时，其它地方已经在调用sync时，不再调用；
        if (!isReentrance) {
            // we do sync only on the top level.
            sync();
        }
        mAddingObserverCounter--;
    }
```
到这里,处理流程和添加观察者源码感觉已经读完了,但我们是不是忽略什么细节? 

从上面看, 状态变化时, 会调用观察者的onStateChanged()方法; 但我们添加观察者时,是实现LifecycleObserver接口,该接口并没有onStateChanged()方法,那又是怎么一回事？


```java
public interface LifecycleObserver {

}
```

我们再详细看一下流程，在添加观察者时，会创建ObserverWithState, 有状态变化时，会先调用ObserverWithState.onStateChanged(); 

```java
    ObserverWithState statefulObserver = new ObserverWithState(observer, initialState);
    ObserverWithState previous = mObserverMap.putIfAbsent(observer, statefulObserver);
```

似乎秘密在ObserverWithState的构造方法中：
```java
        GenericLifecycleObserver mLifecycleObserver;

        ObserverWithState(LifecycleObserver observer, State initialState) {
            mLifecycleObserver = Lifecycling.getCallback(observer);
            mState = initialState;
        }
```
可以看到由Lifecycling.getCallback(observer)来返回一个GenericLifecycleObserver的接口实例，我们再往下看：
```java
    @NonNull
    static GenericLifecycleObserver getCallback(Object object) {
        //如果你写的观察者类是实现FullLifecycleObserver接口的, 直接返回FullLifecycleObserverAdapter实例;
		//FullLifecycleObserverAdapter实例会实现GenericLifecycleObserver接口，并且根据状态来调用FullLifecycleObserver接口方法；
        if (object instanceof FullLifecycleObserver) {
            return new FullLifecycleObserverAdapter((FullLifecycleObserver) object);
        }
        //如果你写的观察者类是实现GenericLifecycleObserver接口的, 直接返回实例;
        if (object instanceof GenericLifecycleObserver) {
            return (GenericLifecycleObserver) object;
        }
        //接下来就是实现LifecycleObserver接口的观察者了，会有点麻烦；
		
        //获取观察者类的Class; 
        final Class<?> klass = object.getClass();
        //返回观察者类的类型
        int type = getObserverConstructorType(klass);
		//针对GENERATED_CALLBACK创建GenericLifecycleObserver类型；
        if (type == GENERATED_CALLBACK) {
            List<Constructor<? extends GeneratedAdapter>> constructors =
                    sClassToAdapters.get(klass);
            if (constructors.size() == 1) {
                GeneratedAdapter generatedAdapter = createGeneratedAdapter(
                        constructors.get(0), object);
                return new SingleGeneratedAdapterObserver(generatedAdapter);
            }
            GeneratedAdapter[] adapters = new GeneratedAdapter[constructors.size()];
            for (int i = 0; i < constructors.size(); i++) {
                adapters[i] = createGeneratedAdapter(constructors.get(i), object);
            }
            return new CompositeGeneratedAdaptersObserver(adapters);
        }
		//针对REFLECTIVE_CALLBACK创建GenericLifecycleObserver类型；
        return new ReflectiveGenericLifecycleObserver(object);
    }
```

观察者实现LifecycleObserver接口的类在Lifecycle分两种类型：

1. GENERATED_CALLBACK：表示该用户实现的LifecycleObserver接口的类已经有注解处理器自动生成了相应的实现GenericLifecyleObserver接口的类：一般为 xxx__LifecycleAdapter; 

2. REFLECTIVE_CALLBACK：如果没有提前使用注解处理器的话，那么这类型表示要使用反射机制来生成相应的实现GenericLifecyleObserver接口的类；


在这里我们就知道为什么要推荐使用 DefaultLifecycleObserver 接口，而不是直接实现LifecycleObserver接口了，因为直接使用 DefaultLifecycleObserver 接口或者使用GenericLifecycleObserver接口的话， 整个调用的性能会提高很多；

1. 能使用DefaultLifecycleObserver就用；

2，使用注解的话， 最好build.gradle里同时配置注解处理器；


对于getObserverConstructorType()方法，可查看下面的流程图，自行查看源码：
![Image](https://github.com/zhiyufen/BlackBeanStudio/blob/master/%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/Jetpack_Lifecycle_FlowchartDiagra.svg)

大概需要明确几点，在该方法内：

针对GENERATED_CALLBACK；会把由注解处理器自动某该Class生成的类的构造方法对象放在sClassToAdapters中；

针对REFLECTIVE_CALLBACK；会把观察者类的的方法对象封装在CallbackInfo中，并保存在ClassesInfoCache类的，以便后面事件发生时进行反射调用对应的事件方法；

针对 GENERATED_CALLBACK 类型的，SingleGeneratedAdapterObserver或CompositeGeneratedAdaptersObserver实例；
```java
@RestrictTo(RestrictTo.Scope.LIBRARY_GROUP)
public class SingleGeneratedAdapterObserver implements GenericLifecycleObserver {

    private final GeneratedAdapter mGeneratedAdapter;

    SingleGeneratedAdapterObserver(GeneratedAdapter generatedAdapter) {
        mGeneratedAdapter = generatedAdapter;
    }

    @Override
    public void onStateChanged(LifecycleOwner source, Lifecycle.Event event) {
        mGeneratedAdapter.callMethods(source, event, false, null);
        mGeneratedAdapter.callMethods(source, event, true, null);//
    }
}
```
可以看到SingleGeneratedAdapterObserver只是GeneratedAdapter的代理类而已；


对于 REFLECTIVE_CALLBACK 类型的， 会返回ReflectiveGenericLifecycleObserver实例，：

```java
	class ReflectiveGenericLifecycleObserver implements GenericLifecycleObserver {
		private final Object mWrapped;
		private final CallbackInfo mInfo;

		ReflectiveGenericLifecycleObserver(Object wrapped) {
			mWrapped = wrapped;
			//在getObserverConstructorType()方法中，针对该Class创建对应的CallbackInfo； 
			mInfo = ClassesInfoCache.sInstance.getInfo(mWrapped.getClass());
		}

		@Override
		public void onStateChanged(LifecycleOwner source, Event event) {
			//使用反射机制进行调用相应的方法；
			mInfo.invokeCallbacks(source, event, mWrapped);
		}
	}
```
当事件过来时，是调用CallbackInfo.invokeCallbacks进行调用的：
```java
//CallbackInfo类
        @SuppressWarnings("ConstantConditions")
        void invokeCallbacks(LifecycleOwner source, Lifecycle.Event event, Object target) {
			//获取对应事件的方法对象；
            invokeMethodsForEvent(mEventToHandlers.get(event), source, event, target);
            invokeMethodsForEvent(mEventToHandlers.get(Lifecycle.Event.ON_ANY), source, event,
                    target);
        }

        private static void invokeMethodsForEvent(List<MethodReference> handlers,
                LifecycleOwner source, Lifecycle.Event event, Object mWrapped) {
            if (handlers != null) {
                for (int i = handlers.size() - 1; i >= 0; i--) {
                    handlers.get(i).invokeCallback(source, event, mWrapped);
                }
            }
        }

//MethodReference类
        void invokeCallback(LifecycleOwner source, Lifecycle.Event event, Object target) {
            //noinspection TryWithIdenticalCatches
			//分参数类型进行反射调用
            try {
                switch (mCallType) {
                    case CALL_TYPE_NO_ARG:
                        mMethod.invoke(target);
                        break;
                    case CALL_TYPE_PROVIDER:
                        mMethod.invoke(target, source);
                        break;
                    case CALL_TYPE_PROVIDER_WITH_EVENT:
                        mMethod.invoke(target, source, event);
                        break;
                }
            } catch (InvocationTargetException e) {
                throw new RuntimeException("Failed to call observer method", e.getCause());
            } catch (IllegalAccessException e) {
                throw new RuntimeException(e);
            }
        }
```
在这里的，我们知道当我们使用@OnLifecycleEvent注解的方法时， 其中的参数是可以根据需求进行增删的；
到这里，我们基本把Lifecycle的源码看完了，当然有些没看，比如它的注解处理器是怎么处理注解来生成GeneratedAdapter接口对象等。
