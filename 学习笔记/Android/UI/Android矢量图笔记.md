## Android矢量图笔记

[TOC]

### 简介

`VectorDrawable` 是一种矢量图形，在 XML 文件中定义为一组点、线条和曲线及其相关颜色信息。使用矢量可绘制对象的主要优势在于图片可缩放。您可以在不降低显示质量的情况下缩放图片，也就是说，可以针对不同的屏幕密度调整同一文件的大小，而不会降低图片质量。这不仅能缩减 APK 文件大小，还能减少开发者维护工作。您还可以对动画使用矢量图片，具体方法是针对各种显示屏分辨率使用多个 XML 文件，而不是多张图片。

> Android 5.0（API 级别 21）是第一个使用 `VectorDrawable` 和 `AnimatedVectorDrawable` 正式支持矢量可绘制对象的版本，但您可以使用提供 `VectorDrawableCompat` 和 `AnimatedVectorDrawableCompat` 类的 Android 支持库支持较低版本。

- SVG: 即Scalable Vector Graphics 矢量图;
- Vector: 在Android中指的是Vector Drawable，也就是Android中的矢量图

### Vector 语法

Android以一种简化的方式对SVG进行了兼容，这种方式就是通过使用它的Path标签，通过Path标签，几乎可以实现SVG中的其它所有标签，虽然可能会复杂一点，但这些东西都是可以通过工具来完成的，

Path指令解析如下所示：

1. 支持的指令：

- M = moveto(M X,Y) ：将画笔移动到指定的坐标位置
- L = lineto(L X,Y) ：画直线到指定的坐标位置
- H = horizontal lineto(H X)：画水平线到指定的X坐标位置
- V = vertical lineto(V Y)：画垂直线到指定的Y坐标位置
- C = curveto(C X1,Y1,X2,Y2,ENDX,ENDY)：三次贝赛曲线
- S = smooth curveto(S X2,Y2,ENDX,ENDY)
- Q = quadratic Belzier curve(Q X,Y,ENDX,ENDY)：二次贝赛曲线
- T = smooth quadratic Belzier curveto(T ENDX,ENDY)：映射
- A = elliptical Arc(A RX,RY,XROTATION,FLAG1,FLAG2,X,Y)：弧线
- Z = closepath()：关闭路径

1. 使用原则:

- 坐标轴为以(0,0)为中心，X轴水平向右，Y轴水平向下
- 所有指令大小写均可。大写绝对定位，参照全局坐标系；小写相对定位，参照父容器坐标系
- 指令和数据间的空格可以省略
- 同一指令出现多次可以只用一个

> 注意，'M'处理时，只是移动了画笔， 没有画任何东西。 它也可以在后面给出上同时绘制不连续线。

###  VectorDrawable 类

`VectorDrawable` 定义静态可绘制对象。与 SVG 格式类似，每个矢量图形定义为树状层次结构，由 `path` 和 `group` 对象构成。每个 `path` 都包含对象轮廓的几何图形，而 `group` 包含转换的详细信息。所有路径都是按照其在 XML 文件中显示的顺序绘制的。

![img](https://developer.android.com/images/guide/topics/graphics/vectorpath.png)

​												**图 1.** 矢量可绘制资源的层次结构示例

借助 [Vector Asset Studio](https://developer.android.com/studio/write/vector-asset-studio) 工具，可轻松地将矢量图形作为 XML 文件添加到项目中。

```xml
    <!-- res/drawable/battery_charging.xml -->
    <vector xmlns:android="http://schemas.android.com/apk/res/android"
        <!-- intrinsic size of the drawable -->
        android:height="24dp"
        android:width="24dp"
        <!-- size of the virtual canvas -->
        android:viewportWidth="24.0"
        android:viewportHeight="24.0">
       <group
             android:name="rotationGroup"
             android:pivotX="10.0"
             android:pivotY="10.0"
             android:rotation="15.0" >
          <path
            android:name="vect"
            android:fillColor="#FF000000"
            android:pathData="M15.67,4H14V2h-4v2H8.33C7.6,4 7,4.6 7,5.33V9h4.93L13,7v2h4V5.33C17,4.6 16.4,4 15.67,4z"
            android:fillAlpha=".3"/>
          <path
            android:name="draw"
            android:fillColor="#FF000000"
            android:pathData="M13,12.5h2L11,20v-5.5H9L11.93,9H7v11.67C7,21.4 7.6,22 8.33,22h7.33c0.74,0 1.34,-0.6 1.34,-1.33V9h-4v3.5z"/>
       </group>
    </vector>
    
```

此 XML 会渲染以下图片：

![img](https://developer.android.com/images/guide/topics/graphics/ic_battery_charging_80_black_24dp.png)

### AnimatedVectorDrawable 类

`AnimatedVectorDrawable` 会为矢量图形的属性添加动画。您可以将添加动画效果之后的矢量图形定义为三个单独的资源文件，也可以将其定义为可定义整个可绘制对象的单个 XML 文件。为了更好地理解，我们来看看这两种方法：[多个 XML 文件](https://developer.android.com/guide/topics/graphics/vector-drawable-resources#multiple-files)和[单个 XML 文件](https://developer.android.com/guide/topics/graphics/vector-drawable-resources#single-file)。

### 多个 XML 文件 

借助这种方法，您可以定义三个单独的 XML 文件：

- 一个 `VectorDrawable` XML 文件。
- 一个 `AnimatedVectorDrawable` XML 文件，用于定义目标 `VectorDrawable`、要添加动画效果的目标路径和组、属性，以及定义为 `ObjectAnimator` 或 `AnimatorSet` 对象的动画。
- 一个 Animator XML 文件。

#### 多个 XML 文件示例 

以下 XML 文件演示了矢量图形的动画。

- VectorDrawable 的 XML 文件：`vd.xml`

- ```xml
      <vector xmlns:android="http://schemas.android.com/apk/res/android"
         android:height="64dp"
         android:width="64dp"
         android:viewportHeight="600"
         android:viewportWidth="600" >
         <group
            android:name="rotationGroup"
            android:pivotX="300.0"
            android:pivotY="300.0"
            android:rotation="45.0" >
            <path
               android:name="vectorPath"
               android:fillColor="#000000"
               android:pathData="M300,70 l 0,-70 70,70 0,0 -70,70z" />
         </group>
      </vector>
  ```

- AnimatedVectorDrawable 的 XML 文件：`avd.xml`

  ```xml
      <animated-vector xmlns:android="http://schemas.android.com/apk/res/android"
         android:drawable="@drawable/vd" >
           <target
               android:name="rotationGroup"
               android:animation="@anim/rotation" />
           <target
               android:name="vectorPath"
               android:animation="@anim/path_morph" />
      </animated-vector>
  ```

  

- 用于 AnimatedVectorDrawable 的 XML 文件的 Animator XML 文件：`rotation.xml` 和 `path_morph.xml`

- ```xml
      <objectAnimator
         android:duration="6000"
         android:propertyName="rotation"
         android:valueFrom="0"
         android:valueTo="360" />
  ```

  ```xml
       <set xmlns:android="http://schemas.android.com/apk/res/android">
         <objectAnimator
            android:duration="3000"
            android:propertyName="pathData"
            android:valueFrom="M300,70 l 0,-70 70,70 0,0   -70,70z"
            android:valueTo="M300,70 l 0,-70 70,0  0,140 -70,0 z"
            android:valueType="pathType"/>
      </set>
      
  ```

### 单个 XML 文件 

借助这种方法，您可以通过 XML Bundle 格式将多个相关 XML 文件合并为单个 XML 文件。编译应用时，`aapt` 标记会创建单独的资源，并在添加动画效果之后的矢量中引用这些资源。此方法需要使用 Build Tools 24 或更高版本，且输出可向后兼容。

#### 单个 XML 文件示例 

```xml
    <animated-vector
        xmlns:android="http://schemas.android.com/apk/res/android"
        xmlns:aapt="http://schemas.android.com/aapt">
        <aapt:attr name="android:drawable">
            <vector
                android:width="24dp"
                android:height="24dp"
                android:viewportWidth="24"
                android:viewportHeight="24">
                <path
                    android:name="root"
                    android:strokeWidth="2"
                    android:strokeLineCap="square"
                    android:strokeColor="?android:colorControlNormal"
                    android:pathData="M4.8,13.4 L9,17.6 M10.4,16.2 L19.6,7" />
            </vector>
        </aapt:attr>
        <target android:name="root">
            <aapt:attr name="android:animation">
                <objectAnimator
                    android:propertyName="pathData"
                    android:valueFrom="M4.8,13.4 L9,17.6 M10.4,16.2 L19.6,7"
                    android:valueTo="M6.4,6.4 L17.6,17.6 M6.4,17.6 L17.6,6.4"
                    android:duration="300"
                    android:interpolator="@android:interpolator/fast_out_slow_in"
                    android:valueType="pathType" />
            </aapt:attr>
        </target>
    </animated-vector>
    
```

https://developer.android.com/studio/write/vector-asset-studio

