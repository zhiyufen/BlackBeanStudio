#### [Android]View的坐标及获取相关位置



#### 1. View的坐标系

View 提供了如下 5 种方法获取 View 的坐标：

##### 1.1 View.getTop()、View.getLeft()、View.getBottom()、View.getRight();

![image](https://img-blog.csdn.net/20161219235525714?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxMzg3Mjg1Nw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

从上图可知，可根据View的这四个坐标来获取View的大小：

```
View的Height值 = view.getBottom() - view.getTop();
View的Width值 = view.getRight() - view.getLeft();
```

**注**： 取的坐标表示的是View原始状态时相对于父容器的坐标，对View进行平移操作并不会改变着四个方法的返回值。

##### 1.2 View.getX()、View.getY()

getX()与getY()方法获取的是View左上角相对于父容器的坐标，当View没有发生平移操作时：

```
getX() == getLeft()；
getY == getTop()
```

在我们需要获取点击事件处相对点击View和屏幕坐标时，需要通过 motionEvent 获取：

```
motionEvent event;

event.getX();       
event.getY();

event.getRawX();    
event.getRawY();
```

方式的区别参考下图：

![image1](https://user-gold-cdn.xitu.io/2019/10/22/16df0c169823b55f?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



##### 1.3 View.getTranslationX()、View.getTranslationY()

ranslationX与 translationY是View左上角相对于父容器的偏移量：translationX = getX() - getLeft()；

当View未发生平移操作时，translationX 与translationY都为0。

##### 1.4 View.getLocationOnScreen(int[] position)

获得 View 相对 屏幕 的绝对坐标; 

```
int[] location = new int[2];
view.getLocationOnScreen(location);
int x = location[0]; // view距离 屏幕左边的距离（即x轴方向）
int y = location[1]; // view距离 屏幕顶边的距离（即y轴方向）

// 注：要在view.post（Runable）里获取，即等布局变化后
```

![image2](https://user-gold-cdn.xitu.io/2019/10/22/16df0c16b80175c5?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

##### 1.5 View.getLocationInWindow(int[] position)

获取控件 相对 窗口Window 的位置

```
int[] location = new int[2];
view.getLocationInWindow(location);
int x = location[0]; // view距离window 左边的距离（即x轴方向）
int y = location[1]; // view距离window 顶边的距离（即y轴方向）

// 注：要在onWindowFocusChanged（）里获取，即等window窗口发生变化后
```

![image4](https://user-gold-cdn.xitu.io/2019/10/22/16df0c16b4c5cffb?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

##### 1.6 getLocalVisibleRect()

View可见部分 相对于 自身View位置左上角的坐标。

```
Rect localRect = new Rect();
//注：就算获取当前View的，也需要view.getLocalVisibleRect来调用；
view.getLocalVisibleRect(localRect);

localRect.getLeft();
localRect.getRight();
localRect.getTop();
localRect.getBottom();
```

![image6](https://user-gold-cdn.xitu.io/2019/10/22/16df0c16bd67453a?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

##### 1.7 getGlobalVisibleRect()

View可见部分 相对于 屏幕的坐标。

```
Rect globalRect = new Rect();
view.getGlobalVisibleRect(globalRect);

globalRect.getLeft();
globalRect.getRight();
globalRect.getTop();
globalRect.getBottom();
```

![iamge7](https://user-gold-cdn.xitu.io/2019/10/22/16df0c16bb7b8225?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

该笔记来自于

>  https://juejin.im/entry/5dae45f8f265da5ba46f6106

> https://blog.csdn.net/u013872857/article/details/53750682



#### 2. 获取当前窗口的大小

Android获取窗口可视区域大小是使用View类下的一个方法: getWindowVisibleDisplayFrame()；也就是contentParentView +actionbar的高度；

该方法执行结果与该窗口下的View无关，也就是任何View调用该方法返回的结果是一样，但需要确保该View是Attach到Windows下了。

```java
Rect rect = new Rect();
getWindow().getDecorView().getWindowVisibleDisplayFrame(rect);
```

