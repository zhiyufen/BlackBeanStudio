##  RecyclerView 相关笔记

#### 1. 添加分割线

添加一个实现水平分割线的Demo:

```java
class MyItemDecoration extends RecyclerView.ItemDecoration {
        private Drawable mDrawable;

        public MyItemDecoration(Drawable drawable) {
            mDrawable = drawable;
        }

        @Override
        public void onDraw(Canvas c, RecyclerView parent, State state) {
            int childCount = parent.getChildCount();
            for (int i = 0; i < childCount; i++) {
                View child = parent.getChildAt(i);
                int left = child.getLeft();
                int top = child.getBottom();
                int right = child.getRight();
                int bottom = top + mDrawable.getIntrinsicHeight();
                mDrawable.setBounds(left, top, right, bottom);
                mDrawable.draw(c);
            }
        }

        @Override
        public void getItemOffsets(Rect outRect, View view, RecyclerView parent, State state) {
            L.d(TAG,"getItemOffsets: view = " + view);
            L.d(TAG,"getItemOffsets: parent = " + parent);
            outRect.bottom = mDrawable.getIntrinsicHeight();
        }
    }
```

Drawable:

```xml
<?xml version="1.0" encoding="utf-8"?>

<shape xmlns:android="http://schemas.android.com/apk/res/android"
    android:shape="rectangle">
    <size android:height="0.5dp" />
    <solid android:color="#EEEEEE" />
</shape>
```



#### 2. 添加下拉/上拉加载

平常来说下拉用SwipeRefreshLayout就足够了，网上的可参考实现：https://juejin.im/entry/58953b45128fe100654f32e6；

我们来自己来实现上拉加载，要求如下：

1. 飞扔动作到底部时， 需要直接显示“加载中”的Item；
2. 慢拖动到现有数据最后一个时（还没显示完时），需要后台加载更多数据， 这样有时可以让用户无感下加载后台数据(技术实现，直接在后面添加“加载中”的Item，但由于当前数据还没拖动完，因此不显示“加载中”的Item)， 再拖动到底部时，需要显示“加载中”的Item；
3. 无网络时，飞扔动作到底部或者慢动作到底部时，显示Toast: "无网络网络"；

实现原理： 在RecyclerView监听下拉到底部时，开始加载更多数据时，会添加一个 Footer 到 Adapter里，让其显示；加载完后，让其删除Footer view; 很简单吧。

##### 2.1 Footer的xml

footer_view_layout.xml:

```xml
<?xml version="1.0" encoding="utf-8"?>

<android.support.constraint.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:paddingTop="12dp"
    android:paddingBottom="12dp"
    android:background="#fafafa"
    app:layout_constraintTop_toTopOf="parent"
    app:layout_constraintBottom_toBottomOf="parent"
    app:layout_constraintStart_toStartOf="parent"
    app:layout_constraintEnd_toEndOf="parent">

    <ProgressBar
        android:id="@+id/load_more_progress"
        android:layout_width="24dp"
        android:layout_height="24dp"
        android:indeterminate="true"
        app:layout_constraintHorizontal_chainStyle="packed"
        app:layout_constraintTop_toTopOf="parent"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintEnd_toStartOf="@id/load_more_tv"
        />
    <TextView
        android:id="@+id/load_more_tv"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:textSize="14dp"
        android:textColor="#252525"
        android:text="Loading"
        android:layout_marginStart="8dp"
        app:layout_constraintTop_toTopOf="parent"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintStart_toEndOf="@id/load_more_progress"
        app:layout_constraintEnd_toEndOf="parent"/>
</android.support.constraint.ConstraintLayout>
```

##### 2.2 Adapter实现

```java
public class MyAdapter extends RecyclerView.Adapter<RecyclerView.ViewHolder> {
    private static final String TAG = "MyAdapter";
    private static final int FOOTER_TYPE = 0;
    private MyModel mModel;
    private List<ItemData> mContents;

    private boolean isShowFooterView;

    public MyAdapter(@NonNull MyModel model) {
        mModel = model;
    }

    public void setContents(@NonNull List<BaseItemData> contents) {
        mContents = new CopyOnWriteArrayList<>(contents);
        notifyDataSetChanged();
    }

    @NonNull
    @Override
    public RecyclerView.ViewHolder onCreateViewHolder(@NonNull ViewGroup viewGroup, int viewType) {
        if (viewType == FOOTER_TYPE) {
            LayoutInflater inflater = LayoutInflater.from(viewGroup.getContext());
            View v = inflater.inflate(R.layout.footer_view_layout, viewGroup, false);
            return new FooterViewHolder(v);
        } else { //
            return InfoFlowCPManager.getInstance().onCreateViewHolder(viewGroup, viewType);
        }
    }

    @Override
    public void onBindViewHolder(@NonNull RecyclerView.ViewHolder viewHolder, int position) {
        L.d(TAG, "onBindViewHolder: position: " + position);
        if (getItemViewType(position) != FOOTER_TYPE) {
            InfoFlowCPManager.getInstance().onBindViewHolder(viewHolder, mContents.get(position), mModel);
        }
    }

    @Override
    public int getItemCount() {
        if (isShowFooterView) {
            return mContents.size() + 1;
        }
        return mContents.size();
    }

    @Override
    public int getItemViewType(int position) {
        if (position == getItemCount() - 1 && isShowFooterView) {
            return FOOTER_TYPE;
        } else { // 注意普通的viewType与Footer区分开
            return mContents.get(position).getItemType();
        }
    }
    // 显示 Footer 
    void showFooterView() {
        isShowFooterView = true;
        notifyItemRangeChanged(getItemCount(), getItemCount() + 1);
    }
    // 隐藏 Footer
    void hideFooterView() {
        isShowFooterView = false;
        notifyDataSetChanged();
    }

    static class FooterViewHolder extends RecyclerView.ViewHolder {
        ProgressBar mLoadMoreProgress;
        TextView mLoadMoreTv;

        public FooterViewHolder(View itemView) {
            super(itemView);
            mLoadMoreProgress = itemView.findViewById(R.id.load_more_progress);
            mLoadMoreTv = itemView.findViewById(R.id.load_more_tv);
        }
    }
}
```

##### 2.3 RecyclerView

```java
//其它代码略
addOnScrollListener(new OnScrollListener() {
        @Override
        public void onScrollStateChanged(RecyclerView recyclerView, int newState) {
            if (recyclerView == null || recyclerView.getLayoutManager() == null) {
                return;
            }
            // 实现下拉到底部时进行加载
            if (SCROLL_STATE_IDLE == newState && !mIsLoading) {
                RecyclerView.LayoutManager layoutManager = recyclerView.getLayoutManager();
                LinearLayoutManager lm = (LinearLayoutManager) layoutManager;

                int lastCompletePosition = lm.findLastCompletelyVisibleItemPosition();
                // 实现要求3：只有当前数据中最后一个显示完全时，再显示"无网络连接"，
                if (lastCompletePosition == layoutManager.getItemCount() - 1
                    && mLoadMoreListener != null && !Utils.isNetworkAvailable()) {
                    L.d(TAG, "No network and show toast. ");
                    Toast.makeText(getContext(),
                        "Loading", Toast.LENGTH_LONG)
                        .show();
                    return;
                }
                int lastVisiblePosition = lm.findLastVisibleItemPosition();
                // 实现要求2： 当前数据最后一个显示出来时，进行加载更多数据
                if (lastVisiblePosition == layoutManager.getItemCount() - 1
                    && Utils.isNetworkAvailable()) {
                    if (mLoadMoreListener != null) {
                        mIsLoading = true;
                        L.d(TAG, "Load more data....");
                        RecyclerView.Adapter adapter = getAdapter();
                        if (adapter instanceof MyAdapter) {
                            ((MyAdapter) adapter).showFooterView();
                            // 实现要求3： 飞扔动作，最后一个数据显示，加载中的Item同时显示；
                            if (lastCompletePosition == layoutManager.getItemCount() - 2) {
                                smoothScrollToPosition(layoutManager.getItemCount()-1);
                            }
                        }
                        mLoadMoreListener.onLoadMoreData(() -> {
                            L.d(TAG, "load more finish.");
                            mIsLoading = false;
                            if (adapter instanceof MyAdapter) {
                                ((MyAdapter) adapter).hideFooterView();
                            }
                        });
                    }
                }
            }
        }

        @Override
        public void onScrolled(RecyclerView recyclerView, int dx, int dy) {
            super.onScrolled(recyclerView, dx, dy);
        }
    });
```

在实现完上面的3个要求，整个上拉加载在用户体验上，已经非常友好了。 





#### 3, LinearSmoothScroller, 可自定义实现滑动速度：

```java
public class SmoothScrollLayoutManager extends LinearLayoutManager {

        public SmoothScrollLayoutManager(Context context) {
            super(context);
        }

        @Override
        public void smoothScrollToPosition(RecyclerView recyclerView,
                                           RecyclerView.State state, final int position) {

            LinearSmoothScroller smoothScroller =
                    new LinearSmoothScroller(recyclerView.getContext()) {
                        // 返回：滑过1px时经历的时间(ms)。
                        @Override
                        protected float calculateSpeedPerPixel(DisplayMetrics displayMetrics) {
                            return 150f / displayMetrics.densityDpi;
                        }
                    };

            smoothScroller.setTargetPosition(position);
            startSmoothScroll(smoothScroller);
        }
    }
```



#### 4. 自定义实现RecyclerView.ItemDecoration，

可自定义ItemView上下左右的边距：https://juejin.im/entry/596587ea51882568cb6b49de 或者 绘制分割线。

##### getItemOffsets方法：

```
    /**
     * @param outRect 用于规定分割线的范围
     * @param view    进行分割线操作的子view
     * @param parent  父view
     * @param state   (这里暂时不使用)
     */
    @Override
    public void getItemOffsets(Rect outRect, View view, RecyclerView parent,    RecyclerView.State state) {
        super.getItemOffsets(outRect, view, parent, state);
    }
```

水平分割线：
    outRect.set(0, 0, 0, mDivider.getIntrinsicHeight());我们应该底部预留分割线的高度。
垂直分割线：
    outRect.set(0, 0, mDivider.getIntrinsicWidth(), 0);我们应该右侧预留分割线的宽度。

##### onDraw方法：

在上面Rect(Item)的范围内画图；

##### onDrawOver方法：

这个方法是相对与整个视图的，不受Item的限制，比如我们可以在视图的顶部加一个悬浮的层等等。

##### Demo：

```java
// 水平分割线
    class MyItemDecoration extends RecyclerView.ItemDecoration {
        private Drawable mDrawable;

        public InfoFlowItemDecoration(Drawable drawable) {
            mDrawable = drawable;
        }

        //子视图上设置绘制范围，并绘制内容
        @Override
        public void onDraw(Canvas c, RecyclerView parent, State state) {
            int childCount = parent.getChildCount();
            for (int i = 0; i < childCount; i++) {
                View child = parent.getChildAt(i);
                int left = child.getLeft();
                int top = child.getBottom();
                int right = child.getRight();
                int bottom = top + mDrawable.getIntrinsicHeight();
                mDrawable.setBounds(left, top, right, bottom);
                mDrawable.draw(c);
            }
        }
        //设置ItemView的内嵌偏移长度（inset）
        @Override
        public void getItemOffsets(Rect outRect, View view, RecyclerView parent, State state) {
            outRect.bottom = mDrawable.getIntrinsicHeight();
        }
    }
```

分割线xml：

```xml
<?xml version="1.0" encoding="utf-8"?>
<shape xmlns:android="http://schemas.android.com/apk/res/android"
    android:shape="rectangle">
    <size android:height="0.5dp" />
    <solid android:color="@color/item_decoration_color" />
</shape>
```



#### 5，滚动事件： 

RecyclerView嵌套同方向的RecyclerView时，滑动事件无法传递给内部的RecyclerView; 

解决滑动冲突原理其实就是：当我们想让内层滑动时在onTouchEvent中设置getParent().requestDisallowInterceptTouchEvent(true);这样控制父View不拦截滑动事件，进而让内层获得焦点滑动。当内层RecyclerView滑动到底部了我们要让外层滑动，设置etParent().requestDisallowInterceptTouchEvent(false);并返回false 不消耗事件。

实现Demo: 

1. 内部RecyclerView还没显示完全时，不处理事件；
2. 滑动到顶部时，不处理事件，从而与事件交给上一个RecyclerView处理；

```java
@Override
    public boolean dispatchTouchEvent(MotionEvent ev) {
        final int action = ev.getAction();
        switch (action) {
            case MotionEvent.SEM_ACTION_PEN_DOWN:
            case MotionEvent.ACTION_DOWN:
                mLastScrollY = ev.getY();
                getParent().requestDisallowInterceptTouchEvent(true);
                break;
            case MotionEvent.ACTION_MOVE:
                float nowY = ev.getY();
                if (isIntercept(nowY)) {
                    getParent().requestDisallowInterceptTouchEvent(false);
                    L.d(TAG, "requestDisallowInterceptTouchEvent = " + "false");
                }
                mLastScrollY = nowY;
                break;
            case MotionEvent.SEM_ACTION_PEN_CANCEL:
            case MotionEvent.ACTION_CANCEL:
            case MotionEvent.SEM_ACTION_PEN_UP:
            case MotionEvent.ACTION_UP:
                getParent().requestDisallowInterceptTouchEvent(false);
                break;
            default:
                break;
        }
        return super.dispatchTouchEvent(ev);
    }

    private boolean isIntercept(float nowY) {
        LinearLayoutManager layoutManager = (LinearLayoutManager) getLayoutManager();
        Rect rect = new Rect();
        View view = (View)getParent();
        view.getGlobalVisibleRect(rect);
        if (view.getHeight() > rect.height()) {
            L.d(TAG, "Info Flow Card is not display all, So don't handler motion event.");
            return true;
        }

        int firstVisibleItemPosition = layoutManager.findFirstVisibleItemPosition();
        if (firstVisibleItemPosition == 0) {
            if (!canScrollVertically(-1) && mLastScrollY < nowY) {
                L.d(TAG, "Info Flow Card have scroll to top, So don't handler motion event.");
                return true;
            }
        }
        return false;
    }
```

http://blog.devwiki.net/index.php/2016/06/13/RecyclerView-Scroll-Listener.html

https://www.jianshu.com/p/ce347cf991db



### 6. Item显示打点及显示时间长打点

一般我们需要对item显示时，或者item显示的时间进行打点；

首先我们选择在onScrolled里进行统计：

```java
   @Override
   public void onScrolled(RecyclerView recyclerView, int dx, int dy) {
         super.onScrolled(recyclerView, dx, dy);
           onHandleScrolled();
   }
```

```java

    private void onHandleScrolled() {
        long time = System.currentTimeMillis();
        if (InfoFlowConfig.isEnableTTCP() && mDisplayListener != null) {
            RecyclerView.LayoutManager layoutManager = getLayoutManager();
            LinearLayoutManager lm = (LinearLayoutManager) layoutManager;
            int firstVisible = lm.findFirstVisibleItemPosition();
            int lastVisible = lm.findLastVisibleItemPosition();
            int visibleItemCount = lastVisible - firstVisible;
            if (lastVisible == 0) {
                visibleItemCount = 0;
            }
            if (visibleItemCount != 0 && (lastStart != firstVisible || lastEnd != lastVisible)) {
                handleItemEnterOrExitEvent(firstVisible, lastVisible);
            }
        }
        L.d(TAG, "onHandleScrolled: Total Time: " + (System.currentTimeMillis() - time));
    }

    private void handleItemEnterOrExitEvent(int firstVisible, int lastVisible) {
        L.d(TAG, "handlerItemEnterOrExitEvent: firstVisible = " + firstVisible + ", lastVisible = " + lastVisible);
        if (mDisplayListener == null) return;
        int visibleItemCount = lastVisible - firstVisible;
        if (visibleItemCount > 0) {
            int startEnter = 0, endEnter = -1;
            int startExit = 0, endExit = -1;
            if (lastStart == -1) {
                startEnter = lastStart = firstVisible;
                endEnter = lastEnd = lastVisible;
            } else {
                if (firstVisible != lastStart) {
                    if (firstVisible > lastStart) {//向上滑动
                        startExit = lastStart;
                        endExit = firstVisible -1;
                    } else {//向下滑动
                        startEnter = firstVisible;
                        endEnter = lastStart - 1;
                    }
                    lastStart = firstVisible;
                }
                if (lastVisible != lastEnd) {
                    if (lastVisible > lastEnd) {//向上滑动
                        startEnter = lastEnd + 1;
                        endEnter = lastVisible;
                    } else {//向下滑动
                        startExit = lastVisible + 1;
                        endExit = lastEnd;

                    }
                    lastEnd = lastVisible;
                }
            }
            if (getAdapter() instanceof InfoFlowCardAdapter) {
                InfoFlowCardAdapter adapter = (InfoFlowCardAdapter) getAdapter();
                L.d(TAG, "handlerItemEnter: startEnter = " + startEnter + ", endEnter = " + endEnter);
                for (int i = startEnter; i <= endEnter; i++) {
                    L.d(TAG, "handlerItemEnter: i " + i);
                    mDisplayListener.onItemEnterDisplay(adapter.getItemData(i));
                }
                L.d(TAG, "handlerItemExit: startExit = " + startExit + ", endExit = " + endExit);
                for (int i = startExit; i <= endExit; i++) {
                    L.d(TAG, "handlerItemExit: i " + i);
                    mDisplayListener.onItemExitDisplay(adapter.getItemData(i));
                }
            }
        }
    }
```

