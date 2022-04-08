## [Android笔记]动画

[TOC]

### 1. 动画之分类：

在Android动画中,有两种动画：View Animation(视图动画)和Property Animator(属性动画)；
	View Animation包括Tween Animation（补间动画）和Frame Animation(逐帧动画)；
	Property Animator包括ValueAnimator和ObjectAnimation；
	
直观上，他们有如下三点不同： 
1、引入时间不同：View Animation是API Level 1就引入的。Property Animation是API Level 11引入的，即Android 3.0才开始有Property Animation相关的API。 
2、所在包名不同：View Animation在包android.view.animation中。而Property Animation API在包 android.animation中。 
3、动画类的命名不同：View Animation中动画类取名都叫XXXXAnimation,而在Property Animator中动画类的取名则叫XXXXAnimator

深度上：
1，View Animation仅能对指定的控件做动画，而Property Animator是通过改变控件某一属性值来做动画的。
2，补间动画虽能对控件做动画，但并没有改变控件内部的属性值。而Property Animator则是恰恰相反，Property Animator是通过改变控件内部的属性值来达到动画效果的。

### 2. View Animation 视图动画： 

##### 代码动态生成动画及插值器

Animation类是所有动画（scale、alpha、translate、rotate）的基类.而它所具有的属性对应的方法如下：
android:duration -------------> setDuration(long)	 动画持续时间，以毫秒为单位 
android:fillAfter ------------> setFillAfter(boolean)	如果设置为true，控件动画结束时，将保持动画最后时的状态
android:fillBefore -----------> setFillBefore(boolean)	如果设置为true,控件动画结束时，还原到开始动画前的状态
android:fillEnabled ----------> setFillEnabled(boolean)	与android:fillBefore 效果相同，都是在动画结束时，将控件还原到初始化状态
android:repeatCount ----------> setRepeatCount(int)	重复次数
android:repeatMode -----------> setRepeatMode(int)	重复类型，有reverse和restart两个值，取值为RESTART或 REVERSE，必须与repeatCount一起使用才能看到效果。因为这里的意义是重复的类型，即回放时的动作。
android:interpolator --------->  setInterpolator(Interpolator) 设定插值器，其实就是指定的动作效果，比如弹跳效果等

上面所述的 xml 动画标签 对应的动画类如下：
scale ---------> ScaleAnimation
	构造方法如下：

```java
ScaleAnimation(Context context, AttributeSet attrs)  //从XML文件加载动画，基本用不到
ScaleAnimation(float fromX, float toX, float fromY, float toY)
ScaleAnimation(float fromX, float toX, float fromY, float toY, float pivotX, float pivotY)
ScaleAnimation(float fromX, float toX, float fromY, float toY, int pivotXType, float pivotXValue, int pivotYType, float pivotYValue)
```

alpha ---------> AlphaAnimation
	构造方法如下:

```java
AlphaAnimation(Context context, AttributeSet attrs) // 同样，从本地XML加载动画，基本不用
AlphaAnimation(float fromAlpha, float toAlpha)
```

rotate -------->  RotateAnimation
	构造方法如下:

```java
RotateAnimation(Context context, AttributeSet attrs)　　//从本地XML文档加载动画，同样，基本不用
RotateAnimation(float fromDegrees, float toDegrees)
RotateAnimation(float fromDegrees, float toDegrees, float pivotX, float pivotY)	
RotateAnimation(float fromDegrees, float toDegrees, int pivotXType, float pivotXValue, int pivotYType, float pivotYValue)
```

translate -----> TranslateAnimation
	构造方法如下:

```java
TranslateAnimation(Context context, AttributeSet attrs)  //同样，基本不用
TranslateAnimation(float fromXDelta, float toXDelta, float fromYDelta, float toYDelta)
TranslateAnimation(int fromXType, float fromXValue, int toXType, float toXValue, int fromYType, float fromYValue, int toYType, float toYValue)
```

注：标签属性android:pivotX中有三种取值，数，百分数，百分数p，对应着构造函数的pivotXType,

它的取值有三个，Animation.ABSOLUTE、Animation.RELATIVE_TO_SELF和Animation.RELATIVE_TO_PARENT；

对于数值、百分数、百分数p，譬如50表示以当前View左上角坐标加50px为初始点、50%表示以当前View的左上角加上当前View宽高的50%做为初始点、50%p表示以当前View的左上角加上父控件宽高的50%做为初始点; 



作者：whd_Alive
链接：https://www.jianshu.com/p/26a801a755a0
来源：简书
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。



set -----------> AnimationSet
	构造方法如下:

```java
AnimationSet(Context context, AttributeSet attrs)  //同样，基本不用
AnimationSet(boolean shareInterpolator)  //shareInterpolator取值true或false，取true时，指在AnimationSet中定义一个插值器（interpolater），它下面的所有动画共同。如果设为false，则表示它下面的动画自己定义各自的插值器。
```

​	重要方法：

```java
public void addAnimation (Animation a) //增加动画的函数。
```

注：shareInterpolator取值true或false，取true时，指在AnimationSet中定义一个插值器（interpolater），它下面的所有动画共同。如果设为false，则表示它下面的动画自己定义各自的插值器。

我们可直接在xml的res/anim/文件夹位置创建相关视图动画：

语法：

```xml
<?xml version="1.0" encoding="utf-8"?>
<set xmlns:android="http://schemas.android.com/apk/res/android"
    android:interpolator="@[package:]anim/interpolator_resource"
    android:shareInterpolator=["true" | "false"] >
    <alpha
        android:fromAlpha="float"
        android:toAlpha="float" />
    <scale
        android:fromXScale="float"
        android:toXScale="float"
        android:fromYScale="float"
        android:toYScale="float"
        android:pivotX="float"
        android:pivotY="float" />
    <translate
        android:fromXDelta="float"
        android:toXDelta="float"
        android:fromYDelta="float"
        android:toYDelta="float" />
    <rotate
        android:fromDegrees="float"
        android:toDegrees="float"
        android:pivotX="float"
        android:pivotY="float" />
    <set>
        ...
    </set>
</set>
```



### 3.Property Animator(属性动画)：

#### Property Animator之ValueAnimator 基础使用：
​	ValueAnimator的功能：ValueAnimator对指定值区间做动画运算，我们通过对运算过程做监听来自己操作控件。	
​	ValueAnimator只负责对指定的数字区间进行动画运算,我们需要对运算过程进行监听，然后自己对控件做动画操作。

例子:

```java

//创建 ValueAnimator 实例
ValueAnimator animator = ValueAnimator.ofInt(0,400);  
animator.setDuration(1000);  
//添加监听
animator.addUpdateListener(new ValueAnimator.AnimatorUpdateListener() {  
    @Override  
    public void onAnimationUpdate(ValueAnimator animation) {  
        int curValue = (int)animation.getAnimatedValue();  
        tv.layout(curValue,curValue,curValue+tv.getWidth(),curValue+tv.getHeight()); //根据动画比率来重新布局View位置与大小
    }  
}); 
animator.start();  
```

##### ValueAnimator的常用方法：

###### 1. ofInt与ofFloat：

```java
public static ValueAnimator ofInt(int... values)  
public static ValueAnimator ofFloat(float... values) 
```

参数类型都是可变参数长参数，所以我们可以传入任何数量的值；传进去的值列表，就表示动画时的变化范围；

比如ofInt(2,90,45)就表示从数值2变化到数字90再变化到数字45；
ofInt需要传入Int类型的参数，而ofFloat则表示需要传入Float类型的参数。

注： ValueAnimator.getAnimatedValue() 返回对象为 Object对象，需要进行强转换。如Float curValueFloat = (Float)animation.getAnimatedValue();

###### 2. 常用函数：
```java
//设置动画时长，单位是毫秒
ValueAnimator setDuration(long duration)
//获取ValueAnimator在运动时，当前运动点的值,返回的值是Object,根据我们原来设置数据类型进行强转换
Object getAnimatedValue();
//开始动画 
void start()  
//设置循环次数,设置为0表示不循环，设置为INFINITE表示无限循环
void setRepeatCount(int value)
//设置循环模式 value取值有RESTART，REVERSE
void setRepeatMode(int value)
//取消动画
void cancel()  
```


###### 3.  2个监听器
​	在ValueAnimator共有2个监听器:


```java
 //监听器一：监听动画变化时的实时值 
public static interface AnimatorUpdateListener {  
	void onAnimationUpdate(ValueAnimator animation);  
}  
//添加方法为
public void addUpdateListener(AnimatorUpdateListener listener)  
//移除AnimatorUpdateListener  
void removeUpdateListener(AnimatorUpdateListener listener);  
void removeAllUpdateListeners();  
```

```java
	//监听器二：监听动画变化时四个状态 
	public static interface AnimatorListener {  
		void onAnimationStart(Animator animation);  
		void onAnimationEnd(Animator animation);  
		void onAnimationCancel(Animator animation);  
		void onAnimationRepeat(Animator animation);  
	}  
	//添加AnimatorListener的方法
	addListener(AnimatorListener listener) ；
```



	
```java
//移除AnimatorListener 
void removeListener(AnimatorListener listener);  
void removeAllListeners(); 

//例子：
private ValueAnimator doAnimatorListener(){  
	ValueAnimator animator = ValueAnimator.ofInt(0,400);  
  
	animator.addUpdateListener(new ValueAnimator.AnimatorUpdateListener() {  
		@Override  
		public void onAnimationUpdate(ValueAnimator animation) {  
			int curValue = (int)animation.getAnimatedValue();  
			tv.layout(tv.getLeft(),curValue,tv.getRight(),curValue+tv.getHeight());  
		}  
	});  
	animator.addListener(new Animator.AnimatorListener() {  
		@Override  
		public void onAnimationStart(Animator animation) {  
			Log.d("qijian","animation start");  
		}  
  
		@Override  
		public void onAnimationEnd(Animator animation) {  
			Log.d("qijian","animation end");  
		}  
  
		@Override  
		public void onAnimationCancel(Animator animation) {  
			Log.d("qijian","animation cancel");  
		}  
  
		@Override  
		public void onAnimationRepeat(Animator animation) {  
			Log.d("qijian","animation repeat");  
		}  
	});
	animator.setRepeatMode(ValueAnimator.REVERSE);  
	animator.setRepeatCount(ValueAnimator.INFINITE);  
	animator.setDuration(1000);  
	animator.start();  
	return animator;  
}
```

4, 其它函数： 

```java
//延时多久时间开始，单位是毫秒
public void setStartDelay(long startDelay) 
//完全克隆一个ValueAnimator实例，包括它所有的设置以及所有对监听器代码的处理
public ValueAnimator clone()
```



### ValueAnimator高级进阶：
http://blog.csdn.net/harvic880925/article/details/50546884

1， 插值器:Interpolator： 也就是定义动画区间的值是如何变化的类。
	在 ValueAnimator 上使用和Animation一样，同样 android 也为我们定义不少动画加速器：
AccelerateDecelerateInterpolator, AccelerateInterpolator, AnticipateInterpolator, AnticipateOvershootInterpolator, BaseInterpolator, BounceInterpolator, CycleInterpolator, DecelerateInterpolator, FastOutLinearInInterpolator, FastOutSlowInInterpolator, Interpolator, LinearInterpolator, LinearOutSlowInInterpolator, OvershootInterpolator, PathInterpolator

```java
//例子
animator.setInterpolator(new BounceInterpolator()); 
```

可自定义 Interpolator： 
1， 实现 Interpolator接口， 
2， 实现其接口方法： float getInterpolation(float input); 参数input:input参数是一个float类型，它取值范围是0到1，表示当前动画的进度，取0时表示动画刚开始，取1时表示动画结束，取0.5时表示动画中间的位置，其它类推。 
返回值：表示当前实际想要显示的进度。取值可以超过1也可以小于0，超过1表示已经超过目标值，小于0表示小于开始位置。

例子：


	public class MyInterploator implements TimeInterpolator {  
		@Override  
		public float getInterpolation(float input) {  
			return 1-input;  
		}  
	} 
	getInterpolation返回的值为 当前动画的小数数值进度： 0 ~ 1.0f;

2, Evaluator: 相当于转换器，把 InterPolator 接口方法返回的 小数进度 换成 对应的数值的位置。
	当前的值 = 100 + （400 - 100）* 显示进度

	可自定义 Evaluator： 
	1， 实现 TypeEvaluator<T> 接口，及其接口方法：evaluate(float fraction, T startValue, T endValue)
	其中参数：
	feaction: 当前动画的数值进度
	startValue： 动画刚开始时值：
	endValue: 动画结束时的值。
	整个接口方法大概按下面返回结果或者自定义
	result = startValue + fraction * (endValue - startValue),
	
	public class MyEvaluator implements TypeEvaluator<Integer> {  
		@Override  
		public Integer evaluate(float fraction, Integer startValue, Integer endValue) {  
			int startInt = startValue;  
			return (int)(200+startInt + fraction * (endValue - startInt));  
		}  
	} 
	Andorid 系统提供的 Evaluatore有： ArgbEvaluator, FloatArrayEvaluator, FloatEvaluator, IntArrayEvaluator, IntEvaluator, PointFEvaluator, RectEvaluator
	
	其中ArgbEvaluator是用来做颜色值过渡转换的。
	其核心代码： 
	public class ArgbEvaluator implements TypeEvaluator {  
		public Object evaluate(float fraction, Object startValue, Object endValue) {  
			int startInt = (Integer) startValue;  
			int startA = (startInt >> 24);  
			int startR = (startInt >> 16) & 0xff;  
			int startG = (startInt >> 8) & 0xff;  
			int startB = startInt & 0xff;  
	  
			int endInt = (Integer) endValue;  
			int endA = (endInt >> 24);  
			int endR = (endInt >> 16) & 0xff;  
			int endG = (endInt >> 8) & 0xff;  
			int endB = endInt & 0xff;  
	  
			return (int)((startA + (int)(fraction * (endA - startA))) << 24) |  
					(int)((startR + (int)(fraction * (endR - startR))) << 16) |  
					(int)((startG + (int)(fraction * (endG - startG))) << 8) |  
					(int)((startB + (int)(fraction * (endB - startB))));  
		}  
	}  
	
	通过 1，2来看：我们可以通过重写 ImterPolator 改变数值进度来改变数值位置，也可以通过改变Evaluator中进度所对应的数值来改变数值位置.
	
	整体来看：
	    ofInt(0, 400) --------------> 插值器 ------------------> Evaluator -----------------> 监听器返回
	( 定义动画数字区间)      (返回当前数字进度如0.2)    (根据数字进度计算当前值)    在(AnimatorUpdateListener)中返回

3， ofObject()：

	从上面讲了ofInt()和ofFloat()来定义动画，但要操作其它对象，就需要使用 ofObject(); 可以传入任何类型的变量：
	
	public static ValueAnimator ofObject(TypeEvaluator evaluator, Object... values);  
	参数：
	evaluator：自定义的Evaluator，values：是可变长参数，Object类型的
	
	Evaluator的作用是根据当前动画的显示进度，计算出当前进度下把对应的值。Object对象是我们自定的，从进度到值的转换过程也必须由我们来做，不然系统哪知道你要转成个什么。 
	
	实现步骤：
	
	1， 构造ValueAnimator动画对象：
		注意其 ofObject方法的参数： 自定义的Evaluator 和 需要实现动画值的对象。
	
		ValueAnimator mAni = ValueAnimator.ofObject(new NumbaerEvalutor(), new Number(0), new Number(100000));
		
	2， 添加监听。
	
		mAni.addUpdateListener(new AnimatorUpdateListener() {			
			@Override
			public void onAnimationUpdate(ValueAnimator arg0) {
				// TODO Auto-generated method stub
				Number num = (Number)arg0.getAnimatedValue(); //注意取出来里的对象转换
				int text = num.getNum();
		        tv.setText(String.valueOf(text));  
			}
		});
	
	3， 添加插值器：
		mAni.setInterpolator(new AccelerateDecelerateInterpolator());
		
	4， 实现自定义的Evaluator：
	
		public class NumbaerEvalutor implements TypeEvaluator<Number>{
			@Override
			public Number evaluate(float arg0, Number arg1, Number arg2) {
				int start = arg1.getNum();
				int  end = arg2.getNum();
				return new Number((int)(start+ arg0 * (end - start)));
			}
	    }
整体代码：
package com.example.animationdamo;

import android.animation.TypeEvaluator;
import android.animation.ValueAnimator;
import android.animation.ValueAnimator.AnimatorUpdateListener;
import android.app.Activity;
import android.os.Bundle;
import android.view.View;
import android.view.animation.AccelerateDecelerateInterpolator;
import android.widget.Button;
import android.widget.TextView;

public class ObjectValueAnimator extends Activity {

	TextView tv;
	private Button btnStart,btnCancel;  
	private ValueAnimator repeatAnimator; 
	
	@Override
	protected void onCreate(Bundle savedInstanceState) {
		// TODO Auto-generated method stub
		super.onCreate(savedInstanceState);
	    setContentView(R.layout.activity_main);  
	    
	    btnStart = (Button) findViewById(R.id.btn);  
	    btnCancel = (Button)findViewById(R.id.btn_cancel);  
	    tv = (TextView) findViewById(R.id.tv);
	    
	    btnStart.setOnClickListener(new View.OnClickListener() {  
	        @Override  
	        public void onClick(View v) {  
	            repeatAnimator = doRepeatAnim();  
	        }  
	    });  
	  
	    btnCancel.setOnClickListener(new View.OnClickListener() {  
	        @Override  
	        public void onClick(View v) {  
	            repeatAnimator.cancel();  
	        }  
	    });  
	}
	
	   private ValueAnimator doRepeatAnim(){
		   ValueAnimator mAni = ValueAnimator.ofObject(new NumbaerEvalutor(), new Number(0), new Number(100000));
		   mAni.addUpdateListener(new AnimatorUpdateListener() {
			@Override
			public void onAnimationUpdate(ValueAnimator arg0) {
				// TODO Auto-generated method stub
				Number num = (Number)arg0.getAnimatedValue();  
				int text = num.getNum();
		        tv.setText(String.valueOf(text));  
			}
		   });
		   mAni.setDuration(10000);
		   mAni.setInterpolator(new AccelerateDecelerateInterpolator());
		   mAni.start();
		   return mAni;
		}  
	
	   public class Number{
		   int num;
		   public Number(int num) {
			   this.num = num;
		   }
		   public int getNum() {
			   return num;
		   }
	   }
	   
	   public class NumbaerEvalutor implements TypeEvaluator<Number>{
			@Override
			public Number evaluate(float arg0, Number arg1, Number arg2) {
				int start = arg1.getNum();
				int  end = arg2.getNum();
				return new Number((int)(start+ arg0 * (end - start)));
			}
	   }
}


​	
Property Animator 之 ObjectAnimator：

1,  ObjectAnimator 基本使用： ValueAnimator有个缺点，就是只能对数值对动画计算。我们要想对哪个控件操作，需要监听动画过程，在监听中对控件操作。这样使用起来相比补间动画而言就相对比较麻烦。为了能让动画直接与对应控件相关联，以使我们从监听动画过程中解放出来，谷歌的开发人员在ValueAnimator的基础上，又派生了一个类ObjectAnimator; 
由于ObjectAnimator是派生自ValueAnimator的，所以ValueAnimator中所能使用的方法，在ObjectAnimator中都可以正常使用。 但ObjectAnimator也重写了几个方法，比如ofInt(),ofFloat()等。我们先看看利用ObjectAnimator重写的ofFloat方法如何实现一个动画：（改变透明度）

1), public static ObjectAnimator ofFloat(Object target, String propertyName, float... values)
	第一个参数用于指定这个动画要操作的是哪个控件
	第二个参数用于指定这个动画要操作这个控件的哪个属性: 第2点详细讲解
	第三个参数是可变长参数，这个就跟ValueAnimator中的可变长参数的意义一样了，就是指这个属性值是从哪变到哪。

	public static ObjectAnimator ofFloat(Object target, String propertyName, float... values)  
	public static ObjectAnimator ofInt(Object target, String propertyName, int... values)  
	public static ObjectAnimator ofObject(Object target, String propertyName,TypeEvaluator evaluator, Object... values) 

2)，ObjectAnimator做动画时是通过指定属性所对应的set方法来改变的。比如上面指定的改变rotation的属性值，ObjectAnimator在做动画时就会到指定控件（TextView）中去找对应的setRotation()方法来改变控件中对应的值。

	对于 View来说， Set方法如下：
		//1、透明度：alpha  
		public void setAlpha(float alpha)  
		  
		//2、旋转度数：rotation、rotationX、rotationY  
		public void setRotation(float rotation)    //表示围绕Z旋转,rotation表示旋转度数 
		public void setRotationX(float rotationX)  //表示围绕X轴旋转，rotationX表示旋转度数 
		public void setRotationY(float rotationY)  //表示围绕Y轴旋转，rotationY表示旋转度数
		  
		//3、平移：translationX、translationY  
		public void setTranslationX(float translationX) //表示在X轴上的平移距离,以当前控件为原点，向右为正方向，参数translationX表示移动的距离。 
		public void setTranslationY(float translationY) //表示在Y轴上的平移距离，以当前控件为原点，向下为正方向，参数translationY表示移动的距离。 
		  
		//缩放：scaleX、scaleY  
		public void setScaleX(float scaleX)  //在X轴上缩放，scaleX表示缩放倍数
		public void setScaleY(float scaleY)  //在Y轴上缩放，scaleY表示缩放倍数


	a) 改变透明度动画：
		ObjectAnimator mAni = ObjectAnimator.ofFloat(tv, "alpha", 1, 0, 1);
		mAni.setDuration(2000);
		mAni.start();
	
	b) 改变旋转角度动画：
	  围绕 Z 轴旋转：
		ObjectAnimator animator = ObjectAnimator.ofFloat(tv,"rotation",0,180,0);  
		
	  围绕 X 轴旋转：
		ObjectAnimator animator = ObjectAnimator.ofFloat(tv,"rotationX",0,270,0);  
		
	  围绕 Y 轴旋转：
		ObjectAnimator animator = ObjectAnimator.ofFloat(tv,"rotationY",0,180,0);   
	
	c) 改变位移的动画：
	  X 轴上的移动动画：
		ObjectAnimator animator = ObjectAnimator.ofFloat(tv, "translationX", 0, 200, -200,0);  
	  Y 轴上的移动动画
		ObjectAnimator animator = ObjectAnimator.ofFloat(tv, "translationY", 0, 200, -100,0);  
	d) 改变缩放的动画：
	  X 轴上的缩放动画:
		ObjectAnimator animator = ObjectAnimator.ofFloat(tv, "scaleX", 0, 3, 1); 
	  Y 轴上的缩放动画:

(添加图： D:\Notepad++\Android 基本知识相关文档\Image\手机屏幕坐标.PNG)	  
	  
使用总结： 
	要使用ObjectAnimator来构造对画，要操作的控件中，必须存在对应的属性的set方法。
	setter 方法的命名必须以骆驼拼写法命名，即set后每个单词首字母大写，其余字母小写，即类似于setPropertyName所对应的属性为propertyName。	
	
	
	
	
2， ObjectAnimator动画原理：	
	
(添加图： D:\Notepad++\Android 基本知识相关文档\Image\ValueAnimator VS ObjectAnimator 动画流程图.PNG)	 
	ObjectAnimator的动画流程中，也是首先通过加速器产生当前进度的百分比，然后再经过Evaluator生成对应百分比所对应的数字值。这两步与ValueAnimator是完全一样的，唯一不同的是最后一步，在ValueAnimator中，我们要通过添加监听器来监听当前数字值。而在ObjectAnimator中，则是先根据属性值拼装成对应的set函数的名字，比如这里的scaleY的拼装方法就是将属性的第一个字母强制大写后，与set拼接，所以就是setScaleY。然后通过反射找到对应控件的setScaleY(float scaleY)函数，将当前数字值做为setScaleY(float scale)的参数将其传入。 
	这里在找到控件的set函数以后，是通过反射来调用这个函数的，有关反射的使用大家可以参考《夯实JAVA基本之二 —— 反射（1）：基本类周边信息获取》 
	这就是ObjectAnimator的流程，最后一步总结起来就是调用对应属性的set方法，将动画当前数字值做为参数传进去。

	注：
	a) 拼接set函数的方法: 首先是强制将属性的第一个字母大写，然后与set拼接，就是对应的set函数的名字。如函数名命名为setScalePointX(float )那我们在写属性时可以写成”scalePointX”或者写成“ScalePointX”都是可以的
	b) 确定函数的参数类型: ObjectAnimator的动画中产生的数值类型也是与构造时的类型一样的,也就是如果构造时使用的是ofFloat函数，所以中间值的类型应该是Float类型的，所以在最后一步拼装出来的set函数应该是setScaleY(float xxx)的样式，否则报错误。
	如：05-18 14:36:07.591: W/PropertyValuesHolder(13466): Method setPointRadius() with type float not found on target class class com.example.animationdamo.MyPointView
	c) 调用set函数以后怎么办？ObjectAnimator只负责把动画过程中的数值传到对应属性的set函数中就结束了. set函数中的对控件的操作还是需要我们自己来写的。
	
	总结： ObjectAnimator的动画设置流程:ObjectAnimator需要指定操作的控件对象，在开始动画时，到控件类中去寻找设置属性所对应的set函数，然后把动画中间值做为参数传给这个set函数并执行它。

3，自定义ObjectAnimator属性Damo:

	保存圆形信息类——Point:
	public class Point {  
		private int mRadius;  
	  
		public Point(int radius){  
			mRadius = radius;  
		}  
	  
		public int getRadius() {  
			return mRadius;  
		}  
	  
		public void setRadius(int radius) {  
			mRadius = radius;  
		}  
	}  
	
	自定义控件——MyPointView:
	public class MyPointView extends View {  
		private Point mPoint = new Point(100);  
	  
		public MyPointView(Context context, AttributeSet attrs) {  
			super(context, attrs);  
		}  
	  
		@Override  
		protected void onDraw(Canvas canvas) {  
			if (mPoint != null){  
				Paint paint = new Paint();  
				paint.setAntiAlias(true);  
				paint.setColor(Color.RED);  
				paint.setStyle(Paint.Style.FILL);  
				canvas.drawCircle(300,300,mPoint.getRadius(),paint);  
			}  
			super.onDraw(canvas);  
		}  
	  
		void setPointRadius(int radius){  
			mPoint.setRadius(radius);  
			invalidate();  
		}  
	  
	}
	
	使用MyPointView: 
	main.xml
	
		<com.example.BlogObjectAnimator1.MyPointView  
				android:id="@+id/pointview"  
				android:layout_width="match_parent"  
				android:layout_height="match_parent"  
				android:layout_below="@id/tv"/> 
	
	使用MyActivity:	
	
	@Override
	protected void onCreate(Bundle savedInstanceState) {
		// TODO Auto-generated method stub
		super.onCreate(savedInstanceState);
	    setContentView(R.layout.activity_main);  
	    
	    btnStart = (Button) findViewById(R.id.btn);  
	    btnCancel = (Button)findViewById(R.id.btn_cancel);  
	    tv = (TextView) findViewById(R.id.tv);
	    mPointView = (MyPointView)findViewById(R.id.pointview);	
	    
	    btnStart.setOnClickListener(new View.OnClickListener() {  
	        @Override  
	        public void onClick(View v) {  
	            repeatAnimator = doRepeatAnim();  
	        }  
	    });  
	  
	    btnCancel.setOnClickListener(new View.OnClickListener() {  
	        @Override  
	        public void onClick(View v) {  
	            repeatAnimator.cancel();  
	        }  
	    });  
	}
	
	   private ValueAnimator doRepeatAnim(){
		   ObjectAnimator mAni = ObjectAnimator.ofInt(mPointView, "pointRadius", 0, 1000, 200);
		   mAni.setDuration(2000);
		   mAni.start();
		   return mAni;
		}  

4， 使用注意事项：
	1) ofFloat()/ofInt()/ofObject() 的第三个参数为可变长参数， 如果我们只定义一个值的话。首先会发出警告(Method getXXX() with type int not found on target class calss com.example....), 但同时会用系统默认值做为动画初始值。如果通过自定义控制 XXXView 设置了 getXX() 函数，那么将会以 getXX() 函数的返回值做为初始值。
	总结：当且仅当动画的只有一个过渡值时，系统才会调用对应属性的get函数来得到动画的初始值。
	
5, 常用函数：
	1) 使用ArgbEvaluator的话，构造ObjectAnimator时必须使用ofInt() ; 
	//TextView 
	public void setBackgroundColor(int color); 
	//Damo
	ObjectAnimator animator = ObjectAnimator.ofInt(tv, "BackgroundColor", 0xffff00ff, 0xffffff00, 0xffff00ff);  
	animator.setDuration(8000);  
	animator.setEvaluator(new ArgbEvaluator());  
	animator.start(); 


Property Animator 之 PropertyValuesHolder与Keyframe

Property Animator 之 PropertyValuesHolder

1, ValueAnimator、ObjectAnimator除了通过 ofInt(),ofFloat(),ofObject() 创建实例外，还通过下面的方法进行创建：
	/** 
	 * valueAnimator的 
	 */  
	public static ValueAnimator ofPropertyValuesHolder(PropertyValuesHolder... values)   
	/** 
	 * ObjectAnimator的 
	 */  
	public static ObjectAnimator ofPropertyValuesHolder(Object target,PropertyValuesHolder... values)  	
	//target：指需要执行动画的控件	//values：是一个可变长参数，可以传进去多个PropertyValuesHolder实例，由于每个PropertyValuesHolder实例都会针对一个属性做动画，所以如果传进去多个PropertyValuesHolder实例，将会对控件的多个属性同时做动画操作。
2， PropertyValuesHolder的概述

	PropertyValuesHolder这个类的意义就是，它其中保存了动画过程中所需要操作的属性和对应的值。通过ofFloat(Object target, String propertyName, float… values)构造的动画，ofFloat()的内部实现其实就是将传进来的参数封装成PropertyValuesHolder实例来保存动画状态。在封装成PropertyValuesHolder实例以后，后期的各种操作也是以PropertyValuesHolder为主的。 
	
	public static ObjectAnimator ofPropertyValuesHolder(Object target,PropertyValuesHolder... values) 
	
	1) 创建实例： 
		public static PropertyValuesHolder ofFloat(String propertyName, float... values)  
		public static PropertyValuesHolder ofInt(String propertyName, int... values)   
		public static PropertyValuesHolder ofObject(String propertyName, TypeEvaluator evaluator,Object... values)  
		public static PropertyValuesHolder ofKeyframe(String propertyName, Keyframe... values)  

3, 	PropertyValuesHolder 之 ofFloat()/ofInt(): 

	其构造方法：
	public static PropertyValuesHolder ofFloat(String propertyName, float... values)  
	public static PropertyValuesHolder ofInt(String propertyName, int... values)   	//propertyName：表示ObjectAnimator需要操作的属性名。即ObjectAnimator需要通过反射查找对应属性的setProperty()函数的那个property.		//values：属性所对应的参数，同样是可变长参数，可以指定多个，还记得我们在ObjectAnimator中讲过，如果只指定了一个，那么ObjectAnimator会通过查找getProperty()方法来获得初始值。不理解的同学请参看
	
	Damo: 颜色和角度动画一起进行
		//View
		myTextView = (TextView) findViewById(R.id.my_text_view);
		//动画设置
		//旋转角度的Holder，属性值是Rotation，对应的是View类中SetRotation(float rotation)函数
		PropertyValuesHolder rotationHolder = PropertyValuesHolder.ofFloat("Rotation", 0f,60f, -60f, 40f, -40f, 20f, -20f, 10f, -10f, 0f);
	    //背景颜色，属性是BackgroundColor，对应的是View类中的setBackgroundColor(int color)函数
	    PropertyValuesHolder colorHolder = PropertyValuesHolder.ofInt("BackgroundColor", 0xffffffff, 0xffff00ff, 0xffffff00, 0xffffffff);
	    ObjectAnimator animator = ObjectAnimator.ofPropertyValuesHolder(myTextView, rotationHolder, colorHolder);
	    animator.setDuration(3000);
	    animator.setInterpolator(new AccelerateInterpolator());
	    animator.start();

4， PropertyValuesHolder之ofObject():
	其构造方法：
	public static PropertyValuesHolder ofObject(String propertyName, TypeEvaluator evaluator,Object... values)  
	//propertyName:ObjectAnimator动画操作的属性名;	
	//evaluator: Evaluator实例Evaluator是将当前动画进度计算出当前值的类，可以使用系统自带的IntEvaluator、FloatEvaluator也可以自定义
	//values：可变长参数，表示操作动画属性的值 
	
	Damo:
	
	//自定义 Evalutor
	public class CharEvaluator implements TypeEvaluator<Character> {  
		@Override  
		public Character evaluate(float fraction, Character startValue, Character endValue) {  
			int startInt  = (int)startValue;  
			int endInt = (int)endValue;  
			int curInt = (int)(startInt + fraction *(endInt - startInt));  
			char result = (char)curInt;  
			return result;  
		}  
	}
	//自定义View (Object) 
	public class MyTextView extends TextView {
	
		public MyTextView(Context context, AttributeSet attrs) {  
			super(context, attrs);  
		}  
		public void setCharText(Character character){  
			setText(String.valueOf(character));  
		}  
	}
	
	//获取自定义对象
	myTextView = (MyTextView) findViewById(R.id.my_text_view);
	//动画设置
	PropertyValuesHolder charHolder = PropertyValuesHolder.ofObject("CharText", new CharEvaluator(), new Character('A'), new Character('Z'), new Character('A'));
	PropertyValuesHolder rotationHolder = PropertyValuesHolder.ofFloat("Rotation", 0f,60f, -60f, 40f, -40f, 20f, -20f, 10f, -10f, 0f);PropertyValuesHolder colorHolder = PropertyValuesHolder.ofInt("BackgroundColor", 0xffffffff, 0xffff00ff, 0xffffff00, 0xffffffff);
	ObjectAnimator animator = ObjectAnimator.ofPropertyValuesHolder(myTextView, charHolder, rotationHolder, colorHolder);
	animator.setDuration(3000);
	animator.setInterpolator(new AccelerateInterpolator());
	animator.start();

5， PropertyValuesHolder之其它函数：
	/** 
	 * 设置动画的Evaluator 
	 */  
	public void setEvaluator(TypeEvaluator evaluator)  
	/** 
	 * 用于设置ofFloat所对应的动画值列表,对应PropertyValuesHolder.ofFloat(),下面类同
	 */  
	public void setFloatValues(float... values)  
	/** 
	 * 用于设置ofInt所对应的动画值列表 
	 */  
	public void setIntValues(int... values)  
	/** 
	 * 用于设置ofKeyframe所对应的动画值列表 
	 */  
	public void setKeyframes(Keyframe... values)  
	/** 
	 * 用于设置ofObject所对应的动画值列表 
	 */  
	public void setObjectValues(Object... values)  
	/** 
	 * 设置动画属性名，用于设置PropertyValuesHolder所需要操作的动画属性名
	 */  
	public void setPropertyName(String propertyName) 

Property Animator 之 Keyframe

1， 知识点：
	Keyframe： 关键帧
	 
	分析flash动画的制作原理：
		关键帧这个概念是从动画里学来的，视频里，一秒要播放24帧图片，对于制作flash动画， 不可能每帧再画出来，比如我们要让一个球在30秒时间内，从（0,0）点运动到（300，200）点，那flash是怎么来做的呢，在flash中，我们只需要定义两个关键帧，在动画开始时定义一个，把球的位置放在(0,0)点；在30秒后，再定义一个关键帧，把球的位置放在（300，200）点。在动画 开始时，球初始在是（0，0）点，30秒时间内就adobe flash就会自动填充，把球平滑移动到第二个关键帧的位置（300，200）点；
	
	从上面得出：一个关键帧必须包含两个原素，第一时间点，第二位置。即这个关键帧是表示的是某个物体在哪个时间点应该在哪个位置上。 


2, 构造方法：
	public static Keyframe ofFloat(float fraction) //调用KeyFrame.setValue()方法动态设置其值
	public static Keyframe ofFloat(float fraction, float value)
	public static Keyframe ofInt(float fraction) 
	public static Keyframe ofInt(float fraction, int value)
	public static Keyframe ofObject(float fraction)  
	public static Keyframe ofObject(float fraction, Object value)
	//fraction：表示当前的显示进度，即从加速器中getInterpolation()函数的返回值； 0f ~ 1f(动画开始 ~ 动画结束)
	//value：表示当前应该在的位置 


3, 使用Damo: 
	1) 生成Keyframe对象; 2) 利用PropertyValuesHolder.ofKeyframe()生成PropertyValuesHolder对象; 3) ObjectAnimator.ofPropertyValuesHolder()生成对应的Animator
	
	Damo:
	
	//定义11个keyFrame
	Keyframe frame0 = Keyframe.ofFloat(0f, 0);
	Keyframe frame1 = Keyframe.ofFloat(0.1f, -20f);  
	Keyframe frame2 = Keyframe.ofFloat(0.2f, 20f);  
	Keyframe frame3 = Keyframe.ofFloat(0.3f, -20f);  
	Keyframe frame4 = Keyframe.ofFloat(0.4f, 20f);  
	Keyframe frame5 = Keyframe.ofFloat(0.5f, -20f);  
	Keyframe frame6 = Keyframe.ofFloat(0.6f, 20f);  
	Keyframe frame7 = Keyframe.ofFloat(0.7f, -20f);  
	Keyframe frame8 = Keyframe.ofFloat(0.8f, 20f);  
	Keyframe frame9 = Keyframe.ofFloat(0.9f, -20f);  
	Keyframe frame10 = Keyframe.ofFloat(1, 0);//恢复到原样
	PropertyValuesHolder frameHolder = PropertyValuesHolder.ofKeyframe("rotation",frame0,frame1,frame2,frame3,frame4,frame5,frame6,frame7,frame8,frame9,frame10);          
	Animator animator = ObjectAnimator.ofPropertyValuesHolder(mImage,frameHolder);  
	animator.setDuration(1000);  
	animator.start();  

4, KeyFrame的常用方法：
	/** 
	 * 设置fraction参数，即Keyframe所对应的进度 
	 */  
	public void setFraction(float fraction)   
	/** 
	 * 设置当前Keyframe所对应的值 
	 */  
	public void setValue(Object value)  
	/** 
	 * 设置Keyframe动作期间所对应的插值器 
	 */  
	public void setInterpolator(TimeInterpolator interpolator)  
	
	对于Interpolator的作用，是应用于2 Frame之间的动画，比如：
	Keyframe frame0 = Keyframe.ofFloat(0f, 0);  
	Keyframe frame1 = Keyframe.ofFloat(0.1f, -20f);  
	frame1.setInterpolator(new BounceInterpolator());  
	Keyframe frame2 = Keyframe.ofFloat(1f, 20f);  
	frame2.setInterpolator(new LinearInterpolator());  

5， KeyFrame 之 ofOBject:
	构造方法：
	public static Keyframe ofObject(float fraction)  
	public static Keyframe ofObject(float fraction, Object value) 
	
	Damo1: 
	
	Keyframe frame0 = Keyframe.ofObject(0f, new Character('A'));  
	Keyframe frame1 = Keyframe.ofObject(0.1f, new Character('L'));  
	Keyframe frame2 = Keyframe.ofObject(1,new Character('Z'));  
	  
	PropertyValuesHolder frameHolder = PropertyValuesHolder.ofKeyframe("CharText",frame0,frame1,frame2);  
	frameHolder.setEvaluator(new CharEvaluator()); //利用关键帧创建PropertyValuesHolder后，一定要记得设置自定义的Evaluator:
	ObjectAnimator animator = ObjectAnimator.ofPropertyValuesHolder(myTextView,frameHolder);  
	animator.setDuration(3000);  
	animator.start();  
	
	//使用ofObject来做动画的时候，都必须调用frameHolder.setEvaluator显示设置Evaluator

注：
	1) 如果去掉第0帧，将以第一个关键帧为起始位置
	2) 如果去掉结束帧，将以最后一个关键帧为结束位置
	3) 使用Keyframe来构建动画，至少要有两个或两个以上帧
	


6, 电话响铃效果:
	/** 
	 * 左右震动效果 
	 */  
	Keyframe frame0 = Keyframe.ofFloat(0f, 0);  
	Keyframe frame1 = Keyframe.ofFloat(0.1f, -20f);  
	Keyframe frame2 = Keyframe.ofFloat(0.2f, 20f);  
	Keyframe frame3 = Keyframe.ofFloat(0.3f, -20f);  
	Keyframe frame4 = Keyframe.ofFloat(0.4f, 20f);  
	Keyframe frame5 = Keyframe.ofFloat(0.5f, -20f);  
	Keyframe frame6 = Keyframe.ofFloat(0.6f, 20f);  
	Keyframe frame7 = Keyframe.ofFloat(0.7f, -20f);  
	Keyframe frame8 = Keyframe.ofFloat(0.8f, 20f);  
	Keyframe frame9 = Keyframe.ofFloat(0.9f, -20f);  
	Keyframe frame10 = Keyframe.ofFloat(1, 0);  
	PropertyValuesHolder frameHolder1 = PropertyValuesHolder.ofKeyframe("rotation", frame0, frame1, frame2, frame3, frame4,frame5, frame6, frame7, frame8, frame9, frame10);  
	  
	  
	/** 
	 * scaleX放大1.1倍 
	 */  
	Keyframe scaleXframe0 = Keyframe.ofFloat(0f, 1);  
	Keyframe scaleXframe1 = Keyframe.ofFloat(0.1f, 1.1f);   
	Keyframe scaleXframe9 = Keyframe.ofFloat(0.9f, 1.1f);  
	Keyframe scaleXframe10 = Keyframe.ofFloat(1, 1);  
	PropertyValuesHolder frameHolder2 = PropertyValuesHolder.ofKeyframe("ScaleX",scaleXframe0,scaleXframe1,scaleXframe9,scaleXframe10);  


​	  
​	/** 
​	 * scaleY放大1.1倍 
​	 */  
​	Keyframe scaleYframe0 = Keyframe.ofFloat(0f, 1);  
​	Keyframe scaleYframe1 = Keyframe.ofFloat(0.1f, 1.1f);   
​	Keyframe scaleYframe9 = Keyframe.ofFloat(0.9f, 1.1f);  
​	Keyframe scaleYframe10 = Keyframe.ofFloat(1, 1);  
​	PropertyValuesHolder frameHolder3 = PropertyValuesHolder.ofKeyframe("ScaleY",scaleYframe0,scaleYframe1,scaleYframe2,scaleYframe9,scaleYframe10);  
​	  
	/** 
	 * 构建动画 
	 */  
	Animator animator = ObjectAnimator.ofPropertyValuesHolder(mImage, frameHolder1,frameHolder2,frameHolder3);  
	animator.setDuration(1000);  
	animator.start(); 

7， 总结：
	借助Keyframe，不需要使用AnimatorSet，也能实现多个动画同时播放。这也是ObjectAnimator中唯一一个能实现多动画同时播放的方法，其它的ObjectAnimator.ofInt,ObjectAnimator.ofFloat,ObjectAnimator.ofObject都只能实现针对一个属性动画的操作！ 


Property Animator 之 联合动画的代码实现(AnimatorSet)

前言： ValueAnimator和ObjectAnimator都只能单单实现一个动画，那如果我们想要使用一个组合动画，比如边放大，边移动，边改变alpha值，要怎么办。对于这种组合型的动画，谷歌给我们提供了一个类AnimatorSet;

一， AnimatorSet——playSequentially,playTogether

1, playSequentially: 逐个播放动画：一个动画结束后，播放下一个动画 

	构造方法：
	public void playSequentially(Animator... items); 
	public void playSequentially(List<Animator> items);
	
	Damo:
	public class AnimatorSetActivity extends Activity {
		private Button mButton;
		private TextView mTv1, mTv2;
		@Override
		protected void onCreate(Bundle savedInstanceState) {
			super.onCreate(savedInstanceState);
			setContentView(R.layout.animator_set_main);
			mButton = (Button) findViewById(R.id.btn);
			mTv1 = (TextView) findViewById(R.id.tv_1);
			mTv2 = (TextView) findViewById(R.id.tv_2);
			mButton.setOnClickListener(new View.OnClickListener() {
				@Override
				public void onClick(View v) {
					doPlaySequentiallyAnimator();
				}
			});
		}
	
		private void doPlaySequentiallyAnimator(){
			ObjectAnimator tv1BgAnimator = ObjectAnimator.ofInt(mTv1, "BackgroundColor",  0xffff00ff, 0xffffff00, 0xffff00ff);
			ObjectAnimator tv1TranslateY = ObjectAnimator.ofFloat(mTv1, "translationY", 0, 300, 0);
			ObjectAnimator tv2TranslateY = ObjectAnimator.ofFloat(mTv2, "translationY", 0, 400, 0);
			
			AnimatorSet animatorSet = new AnimatorSet();
			//ArrayList<Animator> list = new ArrayList<Animator>();
			//list.add(tv1BgAnimator);
			//list.add(tv1TranslateY);
			//list.add(tv2TranslateY);
			//animatorSet.playSequentially(list);
			animatorSet.playSequentially(tv1BgAnimator,tv1TranslateY,tv2TranslateY);
			animatorSet.setDuration(1000);
			animatorSet.start();
		}
	}

2， playTogether： 所有动画一起播放
	
	构造方法：
	public void playTogether(Animator... items);
	public void playTogether(Collection<Animator> items);
	
	Damo: 只需要将上面的 animatorSet.playSequentially(tv1BgAnimator,tv1TranslateY,tv2TranslateY);  -->   animatorSet.playTogether(tv1BgAnimator,tv1TranslateY,tv2TranslateY); 其它类同

3,  如何实现无限循环动画:

	ObjectAnimator tv1BgAnimator = ObjectAnimator.ofInt(mTv1, "BackgroundColor",  0xffff00ff, 0xffffff00, 0xffff00ff);
	tv1BgAnimator.setRepeatCount(ValueAnimator.INFINITE);
	ObjectAnimator tv1TranslateY = ObjectAnimator.ofFloat(mTv1, "translationY", 0, 400, 0);
	tv1TranslateY.setRepeatCount(ValueAnimator.INFINITE);
	ObjectAnimator tv2TranslateY = ObjectAnimator.ofFloat(mTv2, "translationY", 0, 400, 0);		
	tv2TranslateY.setRepeatCount(ValueAnimator.INFINITE);
	
	AnimatorSet animatorSet = new AnimatorSet();
	animatorSet.playTogether(tv1BgAnimator,tv1TranslateY,tv2TranslateY);
	animatorSet.setDuration(2000);
	animatorSet.start();

总结：
	1) playTogether和playSequentially在激活动画后，控件的动画情况与它们无关，他们只负责定时激活控件动画。 
	2) playSequentially只有上一个控件做完动画以后，才会激活下一个控件的动画，如果上一控件的动画是无限循环，那下一个控件就别再指望能做动画了。
	3) playTogether和playSequentially只是负责指定什么时候开始动画，不干涉动画自己的运行过程。换言之：playTogether和playSequentially只是赛马场上的每个赛道的门，门打开以后，赛道上的那匹马怎么跑跟它没什么关系。

二、自由设置动画顺序——AnimatorSet.Builder


1， AnimatorSet.Bundler函数：
	AnimatorSet.Builder是通过animatorSet.play(tv1BgAnimator)生成的，这是生成AnimatorSet.Builder对象的唯一途径！
	
	//调用AnimatorSet中的play方法是获取AnimatorSet.Builder对象的唯一途径
	//表示要播放哪个动画
	public Builder play(Animator anim)
	
	AnimatorSet.Builder函数：
		//anim和前面动画一起执行
		public Builder with(Animator anim)
		//先执行这个anim动画再执行前面动画
		public Builder before(Animator anim)
		//执行前面的动画后才执行该anim动画
		public Builder after(Animator anim)
		//延迟n毫秒之后执行动画
		public Builder after(long delay)
	
	方式1: 使用builder对象逐个添加动画
		AnimatorSet.Builder builder = animatorSet.play(tv1TranslateY);
		builder.with(tv2TranslateY);
		builder.after(tv1BgAnimator);
	
	方式2: 串行方式
		animatorSet.play(tv1TranslateY).with(tv2TranslateY).after(tv1BgAnimator);
		
	目前好像只能进行 前中后三个动画组合，

2, 实现代码：
		ObjectAnimator tv1BgAnimator = ObjectAnimator.ofInt(mTv1, "BackgroundColor",  0xffff00ff, 0xffffff00, 0xffff00ff);
	    ObjectAnimator tv1TranslateY = ObjectAnimator.ofFloat(mTv1, "translationY", 0, 300, 0);
	    ObjectAnimator tv2TranslateY = ObjectAnimator.ofFloat(mTv2, "translationY", 0, 400, 0);
	    
	    AnimatorSet animatorSet = new AnimatorSet();
	    AnimatorSet.Builder builder = animatorSet.play(tv1BgAnimator);
	    builder.with(tv2TranslateY);
	    builder.before(tv1TranslateY);
	    animatorSet.setDuration(2000);
	    animatorSet.start();



三、AnimatorSet监听器：

1， 在AnimatorSet中也可以添加监听器，对应的监听器为：
	public static interface AnimatorListener {
		/**
		 * 当AnimatorSet开始时调用
		 */
		void onAnimationStart(Animator animation);

		/**
		 * 当AnimatorSet结束时调用
		 */
		void onAnimationEnd(Animator animation);
	
		/**
		 * 当AnimatorSet被取消时调用
		 */
		void onAnimationCancel(Animator animation);
	
		/**
		 * 当AnimatorSet重复时调用，由于AnimatorSet没有设置repeat的函数，所以这个方法永远不会被调用
		 */
		void onAnimationRepeat(Animator animation);
	}
	
	添加方法为：
	public void addListener(AnimatorListener listener);
	
	监听的是AnimatorSet的过程，只有当AnimatorSet的状态发生变化时，才会被调用。
	AnimatorSet的监听： 
	1、AnimatorSet的监听函数也只是用来监听AnimatorSet的状态的，与其中的动画无关； 
	2、AnimatorSet中没有设置循环的函数，所以AnimatorSet监听器中永远无法运行到onAnimationRepeat()中！ 

四、通用函数逐个设置与AnimatorSet设置的区别

1， 设置状态函数
	//设置单次动画时长
	public AnimatorSet setDuration(long duration);
	//设置加速器
	public void setInterpolator(TimeInterpolator interpolator)
	//设置ObjectAnimator动画目标控件
	public void setTarget(Object target)

	AnimatorSet中设置与在单个ObjectAnimator中设置有什么区别呢？
	在AnimatorSet中设置以后，会覆盖单个ObjectAnimator中的设置；即如果AnimatorSet中没有设置，那么就以ObjectAnimator中的设置为准。如果AnimatorSet中设置以后，ObjectAnimator中的设置就会无效。


2, setTarget(Object target)示例

	//设置ObjectAnimator动画目标控件
	public void setTarget(Object target)
	
	要通过AnimatorSet的setTartget函数设置了目标控件，那么单个动画中的目标控件都以AnimatorSet设置的为准 

3， AnimatorSet之setStartDelay(long startDelay)

	//设置延时开始动画时长
	public void setStartDelay(long startDelay)
	
	注意：setStartDelay函数不会覆盖单个动画的延时，而且仅针对性的延长AnimatorSet的激活时间，单个动画的所设置的setStartDelay仍对单个动画起作用。
	
	a) AnimatorSet的延时是仅针对性的延长AnimatorSet激活时间的，对单个动画的延时设置没有影响。
	b) AnimatorSet真正激活延时 = AnimatorSet.startDelay+第一个动画.startDelay
	c) 在AnimatorSet激活之后，第一个动画绝对是会开始运行的，后面的动画则根据自己是否延时自行处理

### Android动画 之 联合动画的XML实现与使用示例

一、联合动画的XML实现

在xml中对应animator总共有三个标签，分别是
	<animator />:对应ValueAnimator
	<objectAnimator />:对应ObjectAnimator
	<set />:对应AnimatorSet
	
1, animator
	1) animator所有字段及意义
		<animator
			android:duration="int"
			android:valueFrom="float | int | color"
			android:valueTo="float | int | color"
			android:startOffset="int"
			android:repeatCount="int"
			android:repeatMode=["repeat" | "reverse"]
			android:valueType=["intType" | "floatType"]
			android:interpolator=["@android:interpolator/XXX"]/>
	意义：		
		android:duration:每次动画播放的时长
		android:valueFrom:初始动化值；取值范围为float,int和color，如果取值为float对应的值样式应该为89.0，取值为Int时，对应的值样式为：89;当取值为clolor时，对应的值样式为 #333333;
		android:valueTo：动画结束值；取值范围同样是float,int和color这三种类型的值；
		android:startOffset：动画激活延时；对应代码中的startDelay(long delay)函数；
		android:repeatCount：动画重复次数
		android:repeatMode：动画重复模式，取值为repeat和reverse；repeat表示正序重播，reverse表示倒序重播
		android:valueType：表示参数值类型，取值为intType和floatType；与android:valueFrom、android:valueTo相对应。如果这里的取值为intType，那么android:valueFrom、android:valueTo的值也就要对应的是int类型的数值。如果这里的数值是floatType，那么android:valueFrom、android:valueTo的值也要对应的设置为float类型的值。非常注意的是，如果android:valueFrom、android:valueTo的值设置为color类型的值，那么不需要设置这个参数；
		android:interpolator:设置加速器；有关系统加速器所对应的xml值对照表如下： 
	
	2) 将xml加载到程序中
		ValueAnimator valueAnimator = (ValueAnimator) AnimatorInflater.loadAnimator(getApplicationContext(),R.animator.animator);
		valueAnimator.start();
	通过loadAnimator将animator动画的xml文件，加载进来，根据类型进行强转。
	
	3) Damo
	a) res/animator文件夹下生成一个动画的xml文件：
		<?xml version="1.0" encoding="utf-8"?>
			<animator xmlns:android="http://schemas.android.com/apk/res/android" 
				android:valueFrom="0"
				android:valueTo="1000"
				android:duration="2000"
				android:valueType="intType"
				android:interpolator="@android:anim/bounce_interpolator">
		</animator>
	b) 加载到程序中：
	    mButton.setOnClickListener(new View.OnClickListener() {
			@Override
			public void onClick(View v) {
	        	ValueAnimator valueAnimator = (ValueAnimator) AnimatorInflater.loadAnimator(getApplicationContext(),R.animator.value_animator);
	            valueAnimator.addUpdateListener(new ValueAnimator.AnimatorUpdateListener() {
	            	@Override
	            	public void onAnimationUpdate(ValueAnimator animation) {
	            	    int offset = (int)animation.getAnimatedValue();
	            	    mTv1.layout( offset,offset,mTv1.getWidth()+offset,mTv1.getHeight() + offset);
	            	}
	            });
	            valueAnimator.start();
	        }
	    });

2， objectAnimator

	1) 字段意义及使用方法
		<objectAnimator
		android:propertyName="string"
		android:duration="int"
		android:valueFrom="float | int | color"
		android:valueTo="float | int | color"
		android:startOffset="int"
		android:repeatCount="int"
		android:repeatMode=["repeat" | "reverse"]
		android:valueType=["intType" | "floatType"]
		android:interpolator=["@android:interpolator/XXX"]/>
		
	意义： 
		- android:propertyName：对应属性名，即ObjectAnimator所需要操作的属性名。 
		其它字段的意义与animator的意义与取值是一样的，下面再重新列举一下。 
		- android:duration:每次动画播放的时长 
		- android:valueFrom:初始动化值；取值范围为float,int和color； 
		- android:valueTo：动画结束值；取值范围同样是float,int和color这三种类型的值； 
		- android:startOffset：动画激活延时；对应代码中的startDelay(long delay)函数； 
		- android:repeatCount：动画重复次数 
		- android:repeatMode：动画重复模式，取值为repeat和reverse；repeat表示正序重播，reverse表示倒序重播 
		- android:valueType：表示参数值类型，取值为intType和floatType；与android:valueFrom、android:valueTo相对应。如果这里的取值为intType，那么android:valueFrom、android:valueTo的值也就要对应的是int类型的数值。如果这里的数值是floatType，那么android:valueFrom、android:valueTo的值也要对应的设置为float类型的值。非常注意的是，如果android:valueFrom、android:valueTo的值设置为color类型的值，那么不需要设置这个参数； 
		- android:interpolator:设置加速器；
		
	2) 
	
	3) Damo: 
	a) 动画的xml:
		<?xml version="1.0" encoding="utf-8"?>
		<objectAnimator xmlns:android="http://schemas.android.com/apk/res/android"
			android:propertyName="TranslationY"
			android:duration="2000"
			android:valueFrom="0.0"
			android:valueTo="1000"
			android:interpolator="@android:anim/accelerate_interpolator"
			android:valueType="floatType"
			android:repeatCount="1"
			android:repeatMode="reverse"
			android:startOffset="1000">
		</objectAnimator>
	b) Activity代码：
		mButton = (Button) findViewById(R.id.btn);
	    mTv1 = (TextView) findViewById(R.id.tv_1);
	    mButton.setOnClickListener(new View.OnClickListener() {
	        @Override
	        public void onClick(View v) {
	        	ObjectAnimator animator = (ObjectAnimator) AnimatorInflater.loadAnimator(getApplicationContext(),R.animator.object_animator);
	        	animator.setTarget(mTv1);
	        	animator.start();
	        }
	    });

3， Set
	1) 字段意义及使用方法
		<set
			android:ordering=["together" | "sequentially"]>
		android:ordering：表示动画开始顺序。together表示同时开始动画，sequentially表示逐个开始动画；
		加载方式：
		AnimatorSet set = (AnimatorSet) AnimatorInflater.loadAnimator(MyActivity.this,
			R.animator.set_animator);
		set.setTarget(mTv1);
		set.start();
	
	2) Damo:
	
	a) 动画的xml:
		<?xml version="1.0" encoding="utf-8"?>
		<set xmlns:android="http://schemas.android.com/apk/res/android"
			android:ordering="together">
			<objectAnimator
					android:propertyName="x"
					android:duration="500"
					android:valueFrom="0"
					android:valueTo="400"
					android:valueType="floatType"/>
			<objectAnimator
					android:propertyName="y"
					android:duration="500"
					android:valueFrom="0"
					android:valueTo="300"
					android:valueType="floatType"/>
		</set>
	b) Activity代码：
		protected void onCreate(Bundle savedInstanceState) {
			// TODO Auto-generated method stub
			super.onCreate(savedInstanceState);			
			setContentView(R.layout.animator_set_main);
			mButton = (Button) findViewById(R.id.btn);
			mTv1 = (TextView) findViewById(R.id.tv_1);
			mButton.setOnClickListener(new View.OnClickListener() {
				@Override
				public void onClick(View v) {
					AnimatorSet set = (AnimatorSet) AnimatorInflater.loadAnimator(getApplicationContext(),
							R.animator.set_animator);
					set.setTarget(mTv1);
					set.start();
				}
			});
		}

4, AnimatorSet的应用案例：
	功能： 在右下角，点击按钮，其子菜单以： 从小到大，透明度从0到1 的方式弹出，圆周的分布在按钮的周围。
	
	1) 需要用到的基本函数：
		a) JAVA中有一个Math类:
			/**
			 * 求对应弧度的正弦值
			 */
			double sin(double d)
			/**
			 * 求对应弧度的余弦值
			 */
			double cos(double d)
			/**
			 * 求对应弧度的正切值
			 */
			double tan(double d)
			
			/**
			 * Math中根据度数得到弧度值的函数
			 */
			double toRadians(double angdeg)
	
	2) 布局代码：
	animator_set_damo_menu_framelayout.xml
	<?xml version="1.0" encoding="utf-8"?>
	<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
		android:layout_width="match_parent"
		android:layout_height="match_parent"
		android:layout_marginRight="@dimen/main_margin_left"
		android:layout_marginBottom="@dimen/main_margin_left" >
		
		<Button
				android:id="@+id/menu"
				style="@style/MenuStyle"
				android:background="@drawable/menu"/>
	
		<Button
				android:id="@+id/item1"
				style="@style/MenuItemStyle"
				android:background="@drawable/circle1"
				android:visibility="gone"/>
	
		<Button
				android:id="@+id/item2"
				style="@style/MenuItemStyle"
				android:background="@drawable/circle2"
				android:visibility="gone"/>
	
		<Button
				android:id="@+id/item3"
				style="@style/MenuItemStyle"
				android:background="@drawable/circle3"
				android:visibility="gone"/>
	
		<Button
				android:id="@+id/item4"
				style="@style/MenuItemStyle"
				android:background="@drawable/circle4"
				android:visibility="gone"/>
	
		<Button
				android:id="@+id/item5"
				style="@style/MenuItemStyle"
				android:background="@drawable/circle5"
				android:visibility="gone"/>
	
	</FrameLayout>
	styly.xml文件：
	<resources>
		<style name="AppBaseTheme" parent="Theme.AppCompat.Light">
		</style>
		<!-- Application theme. -->
		<style name="AppTheme" parent="AppBaseTheme">
			<!-- All customizations that are NOT specific to a particular API-level can go here. -->
		</style>
		
		<style name="MenuStyle">
			<item name="android:layout_width">50dp</item>
			<item name="android:layout_height">50dp</item>
			<item name="android:layout_gravity">right|bottom</item>
		</style>
	
		<style name="MenuItemStyle">
			<item name="android:layout_width">45dp</item>
			<item name="android:layout_height">45dp</item>
			<item name="android:layout_gravity">right|bottom</item>
		</style>
	</resources>
	
	2) Activity代码：
	public class AnimatorSetDamoActivity extends Activity implements View.OnClickListener{
		private static final String TAG = "MainActivity";
	
		private Button mMenuButton;
		private Button mItemButton1;
		private Button mItemButton2;
		private Button mItemButton3;
		private Button mItemButton4;
		private Button mItemButton5;
	
		private boolean mIsMenuOpen = false;
		
		private int mRadius = 500;
		
		@Override
		public void onCreate(Bundle savedInstanceState) {
			super.onCreate(savedInstanceState);
			setContentView(R.layout.animator_set_damo_menu_framelayout);
			initView();
		}
		private void initView() {
			mMenuButton = (Button) findViewById(R.id.menu);
			mMenuButton.setOnClickListener(this);
	
			mItemButton1 = (Button) findViewById(R.id.item1);
			mItemButton2 = (Button) findViewById(R.id.item2);
			mItemButton3 = (Button) findViewById(R.id.item3);
			mItemButton4 = (Button) findViewById(R.id.item4);
			mItemButton5 = (Button) findViewById(R.id.item5);
			mItemButton1.setOnClickListener(this);
			mItemButton2.setOnClickListener(this);			
			mItemButton3.setOnClickListener(this);			
			mItemButton4.setOnClickListener(this);			
			mItemButton5.setOnClickListener(this);
		}
		
		@Override
		public void onClick(View v) {
			int id= v.getId();
			if (id == R.id.menu) {
				if (!mIsMenuOpen) {
					mIsMenuOpen = true;
					doAnimateOpen(mItemButton1, 0, 5, mRadius);
					doAnimateOpen(mItemButton2, 1, 5, mRadius);
					doAnimateOpen(mItemButton3, 2, 5, mRadius);
					doAnimateOpen(mItemButton4, 3, 5, mRadius);
					doAnimateOpen(mItemButton5, 4, 5, mRadius);
				} else {
					mIsMenuOpen = false;
					doAnimateClose(mItemButton1, 0, 5, mRadius);
					doAnimateClose(mItemButton2, 1, 5, mRadius);
					doAnimateClose(mItemButton3, 2, 5, mRadius);
					doAnimateClose(mItemButton4, 3, 5, mRadius);
					doAnimateClose(mItemButton5, 4, 5, mRadius);
				}
			} else {
				Toast.makeText(this, "你点击了" + v, Toast.LENGTH_SHORT).show();
			}
		}
		
		private void doAnimateOpen(View view, int index, int total, int radius) {
			if (view.getVisibility() != View.VISIBLE) {
				view.setVisibility(View.VISIBLE);
			}
			//根据角度分别算出其 X Y 方向的位移
			double degree = Math.toRadians(90)/(total - 1) * index;
			int translationX = -(int) (radius * Math.sin(degree));
			int translationY = -(int) (radius * Math.cos(degree));
	
			AnimatorSet set = new AnimatorSet();
			//包含平移、缩放和透明度动画
			set.playTogether(
					ObjectAnimator.ofFloat(view, "translationX", 0, translationX),
					ObjectAnimator.ofFloat(view, "translationY", 0, translationY),
					ObjectAnimator.ofFloat(view, "scaleX", 0f, 1f),
					ObjectAnimator.ofFloat(view, "scaleY", 0f, 1f),
					ObjectAnimator.ofFloat(view, "alpha", 0f, 1));
			//动画周期为500ms
			set.setDuration(1 * 500).start();
		}
		
		private void doAnimateClose(final View view, int index, int total,
				int radius) {
	
			double degree = Math.PI * index / ((total - 1) * 2);
			int translationX = -(int) (radius * Math.sin(degree));
			int translationY = -(int) (radius * Math.cos(degree));
			AnimatorSet set = new AnimatorSet();
			//包含平移、缩放和透明度动画
			set.playTogether(
			ObjectAnimator.ofFloat(view, "translationX", translationX, 0),
			ObjectAnimator.ofFloat(view, "translationY", translationY, 0),
			ObjectAnimator.ofFloat(view, "scaleX", 1f, 0f),
			ObjectAnimator.ofFloat(view, "scaleY", 1f, 0f),
			ObjectAnimator.ofFloat(view, "alpha", 1f, 0f));
			set.setDuration(1 * 500).start();
			//添加onAnimationEnd的监听，是为了Close动画点击后，View不可见从而不可点击
			set.addListener(new AnimatorListener(){
				@Override
				public void onAnimationCancel(Animator arg0) {
					// TODO Auto-generated method stub
				}
	
				@Override
				public void onAnimationEnd(Animator arg0) {
					// TODO Auto-generated method stub
					if (view.getVisibility() == View.VISIBLE) {
						view.setVisibility(View.INVISIBLE);
					}
				}
	
				@Override
				public void onAnimationRepeat(Animator arg0) {
					// TODO Auto-generated method stub	
				}
	
				@Override
				public void onAnimationStart(Animator arg0) {
					// TODO Auto-generated method stub				
				}			
			});		
		}
	}


Android动画之LayoutAnimation与GridLayoutAnimation

如何给容器中的控件应用统一动画。即在容器中控件出现时，不必为每个控件添加进入动画，可以在容器中为其添加统一的进入和退出动画。 

1, LayoutAnimation的xml实现——layoutAnimation标签

	1) 定义一个layoutAnimation的animation文件：如anim/layout_animation.xml
		<?xml version="1.0" encoding="utf-8"?>
		<layoutAnimation xmlns:android="http://schemas.android.com/apk/res/android"
						 android:delay="1"
						 android:animationOrder="normal"
						 android:animation="@anim/slide_in_left"/>
	2) 在ViewGroup的View控件中进行适用上面xml动画：添加Android:layoutAnimation=”@anim/layout_animation”
		<ListView
			android:id="@+id/listview"
			android:layout_width="match_parent"
			android:layout_height="match_parent"
			android:layoutAnimation="@anim/layout_animation"
	    />
	
	3) layoutAnimation各字段意义	
	delay:指每个Item的动画开始延时，取值是android:animation所指定动画时长的倍数，取值类型可以是float类型，也可以是百分数，默认是0.5;比如我们这里指定的动画是@anim/slide_in_left，而在slide_in_left.xml中指定android:duration=”1000”，即单次动画的时长是1000毫秒，而我们在这里的指定android:delay=”1”，即一个Item的动画会在上一个item动画完成后延时单次动画时长的一倍时间开始，即延时1000毫秒后开始。
	animationOrder:指viewGroup中的控件动画开始顺序，取值有normal(正序)、reverse(倒序)、random(随机)
	animation：指定每个item入场所要应用的动画。仅能指定res/aim文件夹下的animation定义的动画，不可使用animator动画。
	
	4) Damo:
	a) anim/slide_in_left.xml
		<?xml version="1.0" encoding="utf-8"?>
		<set xmlns:android="http://schemas.android.com/apk/res/android"
			android:duration="1000" >
			<translate android:fromXDelta="-50%p" android:toXDelta="0" />
			<alpha android:fromAlpha="0" android:toAlpha="1.0" />
		</set>
	b) anim/layout_animaton.xml
	  <?xml version="1.0" encoding="utf-8"?>
		<layoutAnimation xmlns:android="http://schemas.android.com/apk/res/android"
			android:delay="1"
			android:animationOrder="random"
			android:animation="@anim/slide_in_left"/>
	c) layout/activity_main.xml
		<?xml version="1.0" encoding="utf-8"?>
		<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
			xmlns:tools="http://schemas.android.com/tools"
			android:layout_width="match_parent"
			android:layout_height="match_parent"
			android:paddingBottom="@dimen/activity_vertical_margin"
			android:paddingLeft="@dimen/activity_horizontal_margin"
			android:paddingRight="@dimen/activity_horizontal_margin"
			android:paddingTop="@dimen/activity_vertical_margin"
			tools:context="com.example.samsung.myapplication.MainActivity"
			android:orientation="vertical">
	
			<Button
				android:id="@+id/addlist"
				android:layout_width="wrap_content"
				android:layout_height="wrap_content"
				android:text="刷新 ListView"/>
	
			<ListView
				android:id="@+id/listview"
				android:layout_width="match_parent"
				android:layout_height="match_parent"
				android:layoutAnimation="@anim/layout_animation"/>
		</LinearLayout>
	d) MainActivity.java
		public class MainActivity extends AppCompatActivity {
		private ListView mListView;
		private ArrayAdapter mAdapter;
		private Button mAddListBtn;
	
		@Override
		protected void onCreate(Bundle savedInstanceState) {
			super.onCreate(savedInstanceState);
			setContentView(R.layout.activity_main);
	
			mListView = (ListView) findViewById(R.id.listview);
			mAdapter = new ArrayAdapter<String>(this, android.R.layout.simple_expandable_list_item_1, getData());
			mListView.setAdapter(mAdapter);
	
			mAddListBtn = (Button)findViewById(R.id.addlist);
			mAddListBtn.setOnClickListener(new View.OnClickListener() {
				@Override
				public void onClick(View v) {
					mAdapter.addAll(getData());
				}
			});
		}
	
		private List<String> getData() {
			List<String> data = new ArrayList<String>();
			data.add("测试数据1");
			data.add("测试数据2");
			data.add("测试数据3");
			data.add("测试数据4");
			return data;
		}
	}
	由上面可得： android:layoutAnimation只在viewGroup创建的时候，才会对其中的item添加动画。在创建成功以后，再向其中添加item将不会再有动画。 


​	
2, LayoutAnimation的代码实现——LayoutAnimationController

	1) 首先，xml中layoutAnimation标签所对应的类为LayoutAnimationController；它有两个构造函数：
		public LayoutAnimationController(Animation animation)
		public LayoutAnimationController(Animation animation, float delay)
	animation对应标签中的android:animation属性，delay对应标签中的android:delay属性。 
	LayoutAnimationController的函数如下:
		/**
		 * 设置animation动画
		 */
		public void setAnimation(Animation animation)
		/**
		 * 设置单个item开始动画延时
		 */
		public void setDelay(float delay)
		/**
		 * 设置viewGroup中控件开始动画顺序，取值为ORDER_NORMAL、ORDER_REVERSE、ORDER_RANDOM
		 */
		public void setOrder(int order)
	
	2) Damo:
		//代码设置通过加载XML动画设置文件来创建一个Animation对象；
	    Animation animation= AnimationUtils.loadAnimation(this,R.anim.slide_in_left);   //得到一个LayoutAnimationController对象；
	    LayoutAnimationController controller = new LayoutAnimationController(animation);   //设置控件显示的顺序；
	    controller.setOrder(LayoutAnimationController.ORDER_REVERSE);   //设置控件显示间隔时间；
	    controller.setDelay(0.3f);   //为ListView设置LayoutAnimationController属性；
	    mListView.setLayoutAnimation(controller);
	    mListView.startLayoutAnimation();

3, 	GridLayoutAnimation的XML实现——gridLayoutAnimation

	1) gridLayoutAnimation 标签属性：
		<?xml version="1.0" encoding="utf-8"?>
		<gridLayoutAnimation xmlns:android="http://schemas.android.com/apk/res/android"
							 android:rowDelay="75%"
							 android:columnDelay="60%"
							 android:directionPriority="none"
							 android:direction="bottom_to_top|right_to_left"
							 android:animation="@android:anim/slide_in_left"/>
	- rowDelay:每一行动画开始的延迟。与LayoutAnimation一样，可以取百分数，也可以取浮点数。取值意义为，当前android:animation所指动画时长的倍数。 
	- columnDelay：每一列动画开始的延迟。取值类型及意义与rowDelay相同。 
	- directionPriority：方向优先级。取值为row,collumn,none，意义分别为：行优先，列优先，和无优先级（同时进行）;具体意义，后面会细讲 
	- **direction：**gridview动画方向。 
	取值有四个：left_to_right：列，从左向右开始动画 
	right_to_left ：列，从右向左开始动画 
	top_to_bottom：行，从上向下开始动画 
	bottom_to_top：行，从下向上开始动画 
	这四个值之间可以通过“|”连接，从而可以取多个值。很显然left_to_right和right_to_left是互斥的，top_to_bottom和bottom_to_top是互斥的。如果不指定 direction字段，默认值为left_to_right | top_to_bottom；即从上往下，从左往右。 
	- animation: gridview内部元素所使用的动画。
	
	2) Damo: 
	a) grid_layout_animation.xml
		<?xml version="1.0" encoding="utf-8"?>
		<gridLayoutAnimation xmlns:android="http://schemas.android.com/apk/res/android"
			android:rowDelay="75%"
			android:columnDelay="60%"
			android:directionPriority="none"
			android:direction="bottom_to_top|right_to_left"
			android:animation="@anim/slide_in_left" />
	b) MainActivity.java:
		public class MainActivity extends AppCompatActivity {
		private GridAdapter mGrideAdapter;
		private List<String> mDatas = new ArrayList<>();
	
		@Override
		protected void onCreate(Bundle savedInstanceState) {
			super.onCreate(savedInstanceState);
			setContentView(R.layout.activity_main);
			
			GridView grid = (GridView) findViewById(R.id.grid);
			mDatas.addAll(getData());
			mGrideAdapter = new GridAdapter();
			grid.setAdapter(mGrideAdapter);
			
			Button addData = (Button)findViewById(R.id.add_data);
			addData.setOnClickListener(new View.OnClickListener() {
				@Override
				public void onClick(View v) {
					addData();
				}
			});
		}
	
		private List<String> getData() {
			List<String> data = new ArrayList<String>();
			for (int i = 1;i<35;i++){
				data.add("DATA "+i);
			}
			return data;
		}
		public void addData(){
			mDatas.addAll(mDatas);
			mGrideAdapter.notifyDataSetChanged();
		}
		
		public class GridAdapter extends BaseAdapter {
			public View getView(int position, View convertView, ViewGroup parent) {
				TextView i = new TextView(MainActivity.this);
				i.setText(mDatas.get(position));
				i.setLayoutParams(new GridView.LayoutParams(GridView.LayoutParams.WRAP_CONTENT, GridView.LayoutParams.WRAP_CONTENT));
				return i;
			}
			public final int getCount() {
				return mDatas.size();
			}
			public final Object getItem(int position) {
				return null;
			}
			public final long getItemId(int position) {
				return position;
			}
		}
	}
	
	由上面得：gridLayoutAnimation仅在gridview第一次创建时各个元素才会有出场动画，在创建成功以后，再向其中添加数据就不会再有动画。这一点与layoutAnimation相同。

4, GridLayoutAnimation的代码实现——GridLayoutAnimationController

	1) gridLayoutAnimation标签在代码中对应GridLayoutAnimationController类，它的构造方法如下：
		public GridLayoutAnimationController(Animation animation)
		public GridLayoutAnimationController(Animation animation, float columnDelay, float rowDelay)
		
	其中animation对应标签属性中的android:animation 
	columnDelay对应标签属性中的android:columnDelay，取值为float类型 
	rowDelay对应标签属性中的android:rowDelay，取值为float类型 
	然后是GridLayoutAnimationController的几个函数：
		/**
		 * 设置列动画开始延迟
		 */
		public void setColumnDelay(float columnDelay)
		/**
		 * 设置行动画开始延迟
		 */
		 public void setRowDelay(float rowDelay)
		 /**
		 * 设置gridview动画的入场方向。取值有：DIRECTION_BOTTOM_TO_TOP、DIRECTION_TOP_TO_BOTTOM、DIRECTION_LEFT_TO_RIGHT、DIRECTION_RIGHT_TO_LEFT
		 */
		 public void setDirection(int direction)
		 /**
		 * 动画开始优先级，取值有PRIORITY_COLUMN、PRIORITY_NONE、PRIORITY_ROW
		 */
		 public void setDirectionPriority(int directionPriority)
	2) Damo  核心代码：
		Animation animation = AnimationUtils.loadAnimation(MyActivity.this,R.anim.slide_in_left);
	    GridLayoutAnimationController controller = new GridLayoutAnimationController(animation);
	    controller.setColumnDelay(0.75f);
	    controller.setRowDelay(0.5f);
	    controller.setDirection(GridLayoutAnimationController.DIRECTION_BOTTOM_TO_TOP|GridLayoutAnimationController.DIRECTION_LEFT_TO_RIGHT);
	    controller.setDirectionPriority(GridLayoutAnimationController.PRIORITY_NONE);
	    grid.setLayoutAnimation(controller);
	    grid.startLayoutAnimation();

5, 总结：
	1) LayoutAnimationt和GridLayoutAnimation是在api1时就已经引入进来了，所以不用担心API不支持的问题 
	2) gridLayoutAnimation与layoutAnimation一样，都只是在viewGroup创建的时候，会对其中的item添加进入动画，在创建完成后，再添加数据将不会再有动画！ 
	3) LayoutAnimationt和GridLayoutAnimation仅支持Animation动画，不支持Animator动画；正是因为它们在api 1就引入进来了，而Animator是在API 11才引入的，所以它们是不可能支持Animator动画的。



### 4. Activity的进退动画

#### 4.1 当前 Activity

定义一个Style：

```xml
<style name="mystyle" parent="android:Animation">  
     <item name="@android:windowEnterAnimation">@anim/enter</item>  
     <item name="@android:windowExitAnimation">@anim/exit</item>  
</style>  
```

R.anim.enter（从左侧进入）

```
<?xml version="1.0" encoding="utf-8"?>  
<set xmlns:android="http://schemas.android.com/apk/res/android" >  
    <translate  
        android:duration="2000"  
        android:fromXDelta="-100%p"  
        android:toXDelta="0%p" />  
</set>  
```

R.anim.exit（从右侧退出）

```
<?xml version="1.0" encoding="utf-8"?>  
<set xmlns:android="http://schemas.android.com/apk/res/android" >  
    <translate  
        android:duration="2000"  
        android:fromXDelta="0%p"  
        android:toXDelta="100%p" />  
</set>  
```

然后通过window.setWindowAnimations方法指定给当前的Activity，这样，当这个Activity进入退出的时候就会分别执行windowEnterAnimation和windowExitAnimation Item指定的anim了。

```java
getWindow().setWindowAnimations(R.style.mystyle);  
```

有时候需要指定当前Activity退出及指定跳转Activity的进入时：

```java
Intent i = new Intent(MainActivity.this, MainActivity2.class);  
startActivity(i);  
overridePendingTransition(R.anim.enter, R.anim.exit);  
```

我的MainActivity会执行R.anim.exit动画，被打开的MainActivity2会执行R.anim.enter动画。

```
this.finish();  
overridePendingTransition(R.anim.enter, R.anim.exit);  
```

打开的MainActivity2里面执行的这段代码，所以MainAcivity2会执行R.anim.exit动画，而MainActivity将重新获得焦点显示出来，将执行R.anim.enter动画。