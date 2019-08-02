## 简介
Google在去年的 I/O 大会非常隆重地推出了一系列的 架构组件， ViewModel正是其中之一；

它主要提供了这些特性：

1, 配置更改期间自动保留其数据 (比如屏幕的横竖旋转);

2, Activity、Fragment等UI组件之间的通信;

ViewModel层的根本职责，就是负责维护UI的状态，追根究底就是维护对应的数据——毕竟，无论是MVP还是MVVM，UI的展示就是对数据的渲染。

1.定义了ViewModel的基类，并建议通过持有LiveData维护保存数据的状态；

2.ViewModel不会随着Activity的屏幕旋转而销毁，减少了维护状态的代码成本（数据的存储和读取、序列化和反序列化）；

3.在对应的作用域内，保正只生产出对应的唯一实例，多个Fragment维护相同的数据状态，极大减少了UI组件之间的数据传递的代码成本。




ViewMode的作用域：
![Image](https://user-gold-cdn.xitu.io/2019/5/28/16afebcb5f0e52c7?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

ViewModel在对应的 作用域 内保持生命周期内的 局部单例，这就引发一个更好用的特性，那就是Fragment、Activity等UI组件间的通信；　

如果Activity被重新创建了，它会收到被之前相同ViewModel实例。当所属Activity终止后，框架调用ViewModel的onCleared()方法释放对应资源：


setRetainInstance(boolean) 是Fragment中的一个方法。将这个方法设置为true就可以使当前Fragment在Activity重建时存活下来；　


## 基本使用

1. 创建自己的ViewModel对象(继承于ViewModel); 

```java
public class MyViewModel extends ViewModel {
    public static final String TAG = "JetPack/MyViewModel";
    private String mContent = null;

    @Override
    protected void onCleared() {
        mContent = null;
        Log.d(TAG, "onCleared...");
    }

    public String getContent() {
        return mContent;
    }

    public void setContent(String content) {
        this.mContent = content;
    }
}
```
注：在Activity中，MyViewModel类要重写构造方法的话，必须有一个构造方法是为无参数的， 否则会报无法创建ViewModel的异常：

```xml
java.lang.RuntimeException: Cannot create an instance of class zhiyufen.learn.jetpack.MyViewModel
```
在Android中，推荐使用AndroidViewModel来代替ViewModel，AndroidViewModel是继承于ViewModel，但它会保存一个Application对象，以便全局使用； 根本原因是ViewModelStore在创建MyViewModel实例时是通过反射机制来调用无参数构造方法创建的；

2. 在使用的Activity、Fragment中获取ViewModel, 
```java
    MyViewModel mViewModel = ViewModelProviders.of(JetPackActivity.this).get(MyViewModel.class);
```

3. 在ViewModel的生命周期内，可更改ViewModel的数据或屏幕旋转导致Activity重建，可通过步骤2取到同一个ViewModel实例；

整个ViewModel的使用很简单， 下面我们学习一下它的源码实现原理；

## 源码学习

我们看到上面使用时，是从ViewModelProviders类获取相应的ViewModel的：
```java
    MyViewModel mViewModel = ViewModelProviders.of(JetPackActivity.this).get(MyViewModel.class);
```

其中ViewModelProviders是相当ViewModel对外工具类，除了提供获取ViewModelProvider的方法外，还提供一个DefaultFactory类；

```java
    @NonNull
    @MainThread
    public static ViewModelProvider of(@NonNull Fragment fragment, @Nullable Factory factory) {
        Application application = checkApplication(checkActivity(fragment));
        //获取app全局唯一的AndroidViewModelFactory对象；
        if (factory == null) {
            factory = ViewModelProvider.AndroidViewModelFactory.getInstance(application);
        }
        //从Fragment中取得ViewModelStore
        return new ViewModelProvider(fragment.getViewModelStore(), factory);
    }
    
    @NonNull
    @MainThread
    public static ViewModelProvider of(@NonNull FragmentActivity activity,
            @Nullable Factory factory) {
        Application application = checkApplication(activity);
        if (factory == null) {
            factory = ViewModelProvider.AndroidViewModelFactory.getInstance(application);
        }
        ////从Activity中取得ViewModelStore
        return new ViewModelProvider(activity.getViewModelStore(), factory);
    }
```

从上面的源码可以看到， FragmentActivity或者Fragment都实现ViewModelStoreOwner接口，并持有一个ViewModelStore, 我们暂且不管它为什么Activity重建(旋转屏幕等)还在存活；

我们来看看它的整体实现及如何保持ViewModel唯一的： 

下面为几个关键类的功能说明：

#### Factory
Factory就负责如何通过一个实现ViewModel接口的类的Class来创建实例的方法
```xml
    ViewModelProvider.Factory接口： 定义了如何通过一个实现ViewModel接口的类的Class来创建实例的方法；
    ViewModelProvider.NewInstanceFactory： 实现Factory接口调用无参数构造方法来创建实例；
    ViewModelProvider.AndroidViewModelFactory：继承于NewInstanceFactory进行针对继承于AndroidViewModel(实现ViewModel)来额外添加了通过Class来调用可传入Application参数的构造方法来创建实例；
    ViewModelProviders.DefaultFactory： 是单纯继承于AndroidViewModelFactory的工厂类，是Android提供给用户使用或重写的默认工厂类；
```

这里源码不贴了；

#### ViewModelStore

ViewModelStore是真正存放ViewModel实例的地方，使用HashMap来存放；
```java
public class ViewModelStore {

    private final HashMap<String, ViewModel> mMap = new HashMap<>();

    final void put(String key, ViewModel viewModel) {
        ViewModel oldViewModel = mMap.put(key, viewModel);
        //把替换掉的ViewModel进行回调以便释放数据内存
        if (oldViewModel != null) {
            oldViewModel.onCleared();
        }
    }

    final ViewModel get(String key) {
        return mMap.get(key);
    }

    /**
     *  Clears internal storage and notifies ViewModels that they are no longer used.
     */
    public final void clear() {
        for (ViewModel vm : mMap.values()) {
            vm.onCleared();
        }
        mMap.clear();
    }
}
```

#### ViewModelProvider

ViewModelProvider相当于Factory和ViewModelStore的Agent类，它会持有Factory和ViewModelStore的实例，然后提供get(Class)用用户获取某ViewModel的对象；
对于get方法，代码也很简单：ViewModelStore数据中心没Class对应的ViewModel实例的话，就使用Factory来创建，然后放进ViewModelStore进行缓存复用；

```java
    @NonNull
    @MainThread
    public <T extends ViewModel> T get(@NonNull Class<T> modelClass) {
        String canonicalName = modelClass.getCanonicalName();
        if (canonicalName == null) {
            throw new IllegalArgumentException("Local and anonymous classes can not be ViewModels");
        }
        return get(DEFAULT_KEY + ":" + canonicalName, modelClass);
    }
    
    public <T extends ViewModel> T get(@NonNull String key, @NonNull Class<T> modelClass) {
        ViewModel viewModel = mViewModelStore.get(key);

        if (modelClass.isInstance(viewModel)) {
            //noinspection unchecked
            return (T) viewModel;
        } else {
            //noinspection StatementWithEmptyBody
            if (viewModel != null) {
                // TODO: log a warning.
            }
        }

        viewModel = mFactory.create(modelClass);
        mViewModelStore.put(key, viewModel);
        //noinspection unchecked
        return (T) viewModel;
    }
```

整个实现原理来说很简单，就是ViewModel会保存在ViewModelStore中，并且以Class字符串(会加一个包名前缀)为key， 不存在就用AndroidViewModelFactory来创建ViewModel实例；
这样就可以保证在它的生命周期内是唯一的；而它的唯一性，可以让我们在FragmentActivity与内部的Fragment或者Fragment与Fragment之前使用这个ViewModel进行共享数据了；

### FragmentActivity重建
那么ViewModel的另外一个特性，在Activity 屏幕旋转或者其它原因导致重建后，如何保证ViewModel不会释放或者重建呢？我们知道FragmentActivity或者Fragment都实现ViewModelStoreOwner接口，并持有一个ViewModelStore实例；

先看看FragmentActivity的ViewModelStoreOwner接口方法：
```java
    @NonNull
    @Override
    public ViewModelStore getViewModelStore() {
        if (getApplication() == null) {
            throw new IllegalStateException("Your activity is not yet attached to the "
                    + "Application instance. You can't request ViewModel before onCreate call.");
        }
        if (mViewModelStore == null) {
            NonConfigurationInstances nc =
                    (NonConfigurationInstances) getLastNonConfigurationInstance();
            if (nc != null) {
                // Restore the ViewModelStore from NonConfigurationInstances
                mViewModelStore = nc.viewModelStore;
            }
            if (mViewModelStore == null) {
                mViewModelStore = new ViewModelStore();
            }
        }
        return mViewModelStore;
    }
```

NonConfigurationInstances是FragmentActivity的实例；NonConfigurationInstances会保存在Activity的mLastNonConfigurationInstances.activity变量中；
```java
    @Nullable
    public Object getLastNonConfigurationInstance() {
        return mLastNonConfigurationInstances != null
                ? mLastNonConfigurationInstances.activity : null;
    }
```
注：mLastNonConfigurationInstances的类也命名为NonConfigurationInstances，该类和FragmentActivity中的NonConfigurationInstances是不一样的；


我们来看看整个Activity重建时，关于ViewModelStore的保存情况：

当Activity重建时会调用ActivityThread.handleRelaunchActivity() --> handleRelaunchActivityInner()方法；
```java
    private void handleRelaunchActivityInner(ActivityClientRecord r, int configChanges,
            List<ResultInfo> pendingResults, List<ReferrerIntent> pendingIntents,
            PendingTransactionActions pendingActions, boolean startsNotResumed,
            Configuration overrideConfig, String reason) {
        //省略部分代码
        
        handleDestroyActivity(r.token, false, configChanges, true, reason);

        //省略部分代码
        
        handleLaunchActivity(r, pendingActions, customIntent);
    }
```


 ActivityThread.handleDestroyActivity()时，会调用performDestroyActivity()方法中，会需要Destroy的Activity的retainNonConfigurationInstances()来保存需要保存的数据；
```java
    @Override
    public void handleDestroyActivity(IBinder token, boolean finishing, int configChanges,
            boolean getNonConfigInstance, String reason) {
        ActivityClientRecord r = mActivities.get(token);
        r.lastNonConfigurationInstances = r.activity.retainNonConfigurationInstances();
    
        return r;
    }

```

在Activity的retainNonConfigurationInstances()方法中：
```java
    NonConfigurationInstances retainNonConfigurationInstances() {
        //onRetainNonConfigurationInstance会调用FragmentActivity重写的同名方法来保存FragmentActivity的数据；
        Object activity = onRetainNonConfigurationInstance();
        HashMap<String, Object> children = onRetainNonConfigurationChildInstances();
        FragmentManagerNonConfig fragments = mFragments.retainNestedNonConfig();

        //省略部分代码
        
        NonConfigurationInstances nci = new NonConfigurationInstances();
        nci.activity = activity;
        
        //省略部分代码
        
        return nci;
    }
```

接下来看看FragmentActivity的onRetainNonConfigurationInstance()方法：
```java
    @Override
    public final Object onRetainNonConfigurationInstance() {
        Object custom = onRetainCustomNonConfigurationInstance();

        FragmentManagerNonConfig fragments = mFragments.retainNestedNonConfig();

        if (fragments == null && mViewModelStore == null && custom == null) {
            return null;
        }

        NonConfigurationInstances nci = new NonConfigurationInstances();
        nci.custom = custom;
        //保存ViewModelStore实例
        nci.viewModelStore = mViewModelStore;
        nci.fragments = fragments;
        return nci;
    }
```

从上面来看，当ActivityonDestory时， ViewModelStore最终会被层层封装到ActivityClientRecord对象中；

把当前Activity进行Destory后,会跟着调用handleLaunchActivity(r, pendingActions, customIntent) --> performLaunchActivity(r, customIntent)， 其中r就是我们前面ActivityClientRecord对象；
```java
    /**  Core implementation of activity launch. */
    private Activity performLaunchActivity(ActivityClientRecord r, Intent customIntent) {
        //省略部分代码...
        
        //创建新Activity实例
        activity = mInstrumentation.newActivity(
                    cl, component.getClassName(), r.intent);
        
        //省略部分代码...
                    
        //对新Activity实例传入ActivityClientRecord对象r; 
        activity.attach(appContext, this, getInstrumentation(), r.token,
                        r.ident, app, r.intent, r.activityInfo, title, r.parent,
                        r.embeddedID, r.lastNonConfigurationInstances, config,
                        r.referrer, r.voiceInteractor, window, r.configCallback);
    }
```

看到这时候调用Activity.attach()方法了，
```
    final void attach(Context context, ActivityThread aThread,
            Instrumentation instr, IBinder token, int ident,
            Application application, Intent intent, ActivityInfo info,
            CharSequence title, Activity parent, String id,
            NonConfigurationInstances lastNonConfigurationInstances,
            Configuration config, String referrer, IVoiceInteractor voiceInteractor,
            Window window, ActivityConfigCallback activityConfigCallback) {
        attachBaseContext(context);
//省略部分代码...
        mLastNonConfigurationInstances = lastNonConfigurationInstances;
//省略部分代码...       
    }
```
这样整个NonConfigurationInstances对象就在Activity重建中保存下来的， 这样FragmentActivity直接调用getLastNonConfigurationInstance()方法获取的ViewModelStore对象和重建前就是一样的；

### Fragment重建

先看看Fragment是怎么实现

```java
    @NonNull
    @Override
    public ViewModelStore getViewModelStore() {
        if (getContext() == null) {
            throw new IllegalStateException("Can't access ViewModels from detached fragment");
        }
        if (mViewModelStore == null) {
            mViewModelStore = new ViewModelStore();
        }
        return mViewModelStore;
    }
```

当Activity需要Ondestroy时，
```java
    @CallSuper
    public void onDestroy() {
        mCalled = true;
        FragmentActivity activity = getActivity();
        boolean isChangingConfigurations = activity != null && activity.isChangingConfigurations();
        if (mViewModelStore != null && !isChangingConfigurations) {
            mViewModelStore.clear();
        }
    }
```

可以看到如果是因为配置更改导致Activity重建的话，mViewModelStore是不会释放内存；

另外我们需要明确一点是，Fragment对象在Activity重建时，虽然里面的生命周期会调用，但对象实例是不变的，从而mViewModelStore的对象也不变；至于Fragment对象为什么不变，这里暂时略过，后面有时间再整理；

对于ViewModel的使用，Google官方建议ViewModel尽量保证 纯的业务代码，不要持有任何View层(Activity或者Fragment)或Lifecycle的引用，这样保证了ViewModel内部代码的可测试性，避免因为Context等相关的引用导致测试代码的难以编写；

到这里ViewModel就学习完了， 接下来学习LiveData;

参考大神的文章：
> [Android官方架构组件ViewModel:从前世今生到追本溯源](https://juejin.im/post/5c047fd3e51d45666017ff86)
