#### Android杂记

#### 1. 系统类学习

- AsyncLayoutInflater: 一个可以在异步线程中加载解析layout，一般用户layout比较大，加载完毕后会在UI线程中调用OnInflateFinishedListener；



#### 2. View

- View可能通过ViewOutlineProvider 来裁剪View(Android L):

  比如剪裁圆角：
  
  ```java
  view.setOutlineProvider(new ViewOutlineProvider() {
      @Override
      public void getOutline(View view, Outline outline) {
          outline.setRoundRect(0, 0, view.getWidth(), view.getHeight(), 30);
      }
  });
view.setClipToOutline(true);
  ```
  
  


### 3. Android事件分发机制详解

https://www.jianshu.com/p/38015afcdb58

