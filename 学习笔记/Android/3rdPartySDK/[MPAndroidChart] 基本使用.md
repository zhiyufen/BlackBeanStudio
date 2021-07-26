## [MPAndroidChart] 基本使用

[TOC]

#### 1. 相关教程

[MPAndroidChart 教程](https://blog.csdn.net/u014136472/article/details/50273309)

#### 2. 图表类型样例图

- LineChart (with legend, simple design)
  ![](https://img-blog.csdn.net/20151212105747921)
  ![](https://img-blog.csdn.net/20151212105802973)
- LineChart (cubic lines)
  ![](https://img-blog.csdn.net/20151212110129546)
- Combined-Chart (bar- and linechart in this case)
  ![](https://img-blog.csdn.net/20151212110150023)
- BarChart (with legend, simple design)
  ![](https://img-blog.csdn.net/20151212110222977)
- BarChart (grouped DataSets)
  ![](https://img-blog.csdn.net/20151212110237331)
- Horizontal-BarChart
  ![](https://img-blog.csdn.net/20151212110254889)
- PieChart (with selection, …)
  ![](https://img-blog.csdn.net/20151212110306926)
- ScatterChart (with squares, triangles, circles, … and more)
  ![](https://img-blog.csdn.net/20151212110318996)
- CandleStickChart (for financial data)
  ![](https://img-blog.csdn.net/20151212110340395)
- BubbleChart (area covered by bubbles indicates the value)
  ![](https://img-blog.csdn.net/20151212110350328)
- RadarChart (spider web chart)
  ![](https://img-blog.csdn.net/20151212110359688)

#### 3. 配置使用

在 `build.gradle` 添加下面的代码：

```groovy
repositories {
    maven { url "https://jitpack.io" }
}

dependencies {
    compile 'com.github.PhilJay:MPAndroidChart:v2.1.6'
}
```

或直接下载jar： https://github.com/PhilJay/MPAndroidChart/releases

#### 4. 使用

在xml里定义：

```xml
<com.github.mikephil.charting.charts.LineChart
        android:id="@+id/chart"
        android:layout_width="match_parent"
        android:layout_height="match_parent" />
```

然后在 Activity 或 Fragment 中拿到你定义的 chart：

```java
LineChart chart = (LineChart) findViewById(R.id.chart);
```

- invalidate() : 在chart中调用会使其刷新重绘
- notifyDataSetChanged() : 让chart知道它依赖的基础数据已经改变，并执行所有必要的重新计算（比如偏移量，legend，最大值，最小值 …）。在动态添加数据时需要用到。
- setLogEnabled(boolean enabled) : 设置为true将激活chart的logcat输出。但这不利于性能，如果不是必要的，应保持禁用。

##### 4.1 基本Chart风格

###### 图表背景

- `setBackgroundColor(int color)` : 设置背景颜色，将覆盖整个图表视图。 此外，背景颜色可以在布局文件 `.xml` 中进行设置；设置颜色时要ARGB完整的八位（如 `0xff00ff00`），否则可能会被视为“设置透明颜色”（如 `0xff0000`）
  ![](https://img-blog.csdn.net/20151214154812996)

###### 设置右下角描述方字方法：

- setDescription(String desc)` : 设置图表的描述文字，会显示在图表的右下角。
- `setDescriptionColor(int color)` : 设置描述文字的颜色。
  如右下角的文字：
  ![](https://img-blog.csdn.net/20151214154942077)
- `setDescriptionPosition(float x, float y)` : 自定义描述文字在屏幕上的位置（单位是像素）
- `setDescriptionTypeface(Typeface t)` : 设置描述文字的 Typeface。
- `setDescriptionTextSize(float size)` : 设置以像素为单位的描述文字，最小6f，最大16f。

###### 设置图数据为空时文字描述方法：

- `setNoDataTextDescription(String desc)` : 设置当 chart 为空时显示的描述文字。
  ![](https://img-blog.csdn.net/20151214152908563)

###### 绘图区背景相关方法：

- `setDrawGridBackground(boolean enabled)` : 如果启用，chart 绘图区后面的背景矩形将绘制。
  ![](https://img-blog.csdn.net/20151214154601010) ![](https://img-blog.csdn.net/20151214153245351)

- `setGridBackgroundColor(int color)` : 设置网格背景应与绘制的颜色。
  ![](https://img-blog.csdn.net/20151214155920149)

###### 绘图区网格相关方法：

- `setDrawBorders(boolean enabled)` : 启用/禁用绘制图表边框（chart周围的线）。
- `setBorderColor(int color)` : 设置 chart 边框线的颜色。
- `setBorderWidth(float width)` : 设置 chart 边界线的宽度，单位 dp。
  ![](https://img-blog.csdn.net/20151214160330030)
- `setMaxVisibleValueCount(int count)` : 设置最大可见绘制的 chart count 的数量。 只在 `setDrawValues()` 设置为 `true` 时有效。

###### 坐标轴

AxisBase是XAxis和YAxis的父类；

- XAxis： X轴标签设置。只使用setter方法来修改它，不要直接访问公共变量；
- YAxis： Y轴标签设置和它的条目。只使用setter方法来修改它，不要直接访问公共变量。

轴”类允许特定的Style，由以下 components/parts 组成（可以包含）：

- 轴的标签（y轴垂直绘制 或 x轴水平取向），contain 轴的描述值。
- 所谓 `axis-line` 被直接绘制在便签旁且平行。
- `grid-lines` 在水平方向，且源自每一个轴标签。
- `LimitLines` 允许呈现的特别信息，如边界或限制。

轴的设置方法：

- `setEnabled(boolean enabled)` : 设置轴启用或禁用。如果false，该轴的任何部分都不会被绘制（不绘制坐标轴/便签等）
- `setDrawGridLines(boolean enabled)` : 设置为true，则绘制网格线。
- `setDrawAxisLine(boolean enabled)` : 设置为true，则绘制该行旁边的轴线（axis-line）。
- `setDrawLabels(boolean enabled)` : 设置为true，则绘制轴的标签。

修改轴的Style方法:

- `setTextColor(int color)` : 设置轴标签的颜色。

- `setTextSize(float size)` : 设置轴标签的文字大小。

- `setTypeface(Typeface tf)` : 设置轴标签的 Typeface。

- `setGridColor(int color)` : 设置该轴的网格线颜色。

- `setGridLineWidth(float width)` : 设置该轴网格线的宽度。

- `setAxisLineColor(int color)` : 设置轴线的轴的颜色。

- `setAxisLineWidth(float width)` : 设置该轴轴行的宽度。

  ![](https://img-blog.csdn.net/20151214194142156)

- `enableGridDashedLine(float lineLength, float spaceLength, float phase)` : 启用网格线的虚线模式中得出，比如像这样“ - - - - - - ”。

  - “lineLength”控制虚线段的长度
  - “spaceLength”控制线之间的空间
  - “phase”controls the starting point.

  ![](https://img-blog.csdn.net/20151214194641313)

限制线方法：

两个轴支持 `LimitLines` 来呈现特定信息，如边界或限制线。`LimitLines` 加入到 `YAxis` 在水平方向上绘制，添加到 `XAxis` 在垂直方向绘制。 如何通过给定的轴添加和删除 `LimitLines`：

- `addLimitLine(LimitLine l)` : 给该轴添加一个新的 LimitLine 。

- `removeLimitLine(LimitLine l)` : 从该轴删除指定 LimitLine 。
  ![](https://img-blog.csdn.net/20151214201049758) ![](https://img-blog.csdn.net/20151214201224254)

- `setDrawLimitLinesBehindData(boolean enabled)` : 控制 `LimitLines` 与 `actual data` 之间的 `z-order` 。 如果设置为 true，LimitLines 绘制在 actual data 的后面，否则在其前面。 默认值：false

  ```java
          // 查看setLimitLinesBehindData()方法，true或false的效果图
          LimitLine xLimitLine = new LimitLine(2f,"is Behind");
          xLimitLine.setLineColor(Color.BLUE);
          xLimitLine.setTextColor(Color.BLUE);
          xAxis.addLimitLine(xLimitLine);
          xAxis.setDrawLimitLinesBehindData(true);
  ```

  Limit lines（`LimitLine类`） 用来为用户提供简单明了的额外信息。

  

###### X 坐标轴

`XAxis` 类是 `AxisBase` 的一个子类。
`XAxis` 类是所有与水平轴相关的 “数据和信息容器”。
每个 `Line-, Bar-, Scatter-, CandleStick- and RadarChart` 都有一个 `XAxis` 对象。 `XAxis` 对象展示了以 `ArrayList` 或 `String[] ("xVals")` 形式递交给 `ChartData` 对象的数据。

为了获得 `XAxis` 类的实例，可执行以下操作：、

```java
XAxis xAxis = chart.getXAxis();
```

- `setSpaceBetweenLabels(int characters)` : 设置标签字符间的空隙，默认characters间隔是4 。

- `setLabelsToSkip(int count)` : 设置在”绘制下一个标签”时，要忽略的标签数。

- `resetLabelsToSkip()` : 调用这个方法将使得通过 `setLabelsToSkip(...)` 的“忽略效果”失效 

- `setAvoidFirstLastClipping(boolean enabled)` : 如果设置为true，则在绘制时会避免“剪掉”在x轴上的图表或屏幕边缘的第一个和最后一个坐标轴标签项。

- `setPosition(XAxisPosition pos)` : 设置XAxis出现的位置。
  - `TOP`，`BOTTOM`，
  - `TOP_INSIDE`，`BOTTOM_INSIDE` 或 `BOTH_SIDED`。 (从左到右，从上到下，对应下图)
  
- 格式化轴的显示值

  ```java
  setValueFormatter(XAxisValueFormatter formatter) //设置自定义格式，在绘制之前动态调整x的值。 
  ```

###### Y 坐标轴

`YAxis` 是 `AxisBase` 的一个子类。

`YAxis` 类是一切与垂直轴相关的数据和信息的容器。 每个 Line-, Bar-, Scatter or CandleStickChart 都有 `left` 和 `right` 的 `YAxis` 的对象，分别在左右两边。 但是 `RadarChart` 只有一个 `YAxis` 。 缺省情况下，图表的两个轴都被启用，并且将被绘制。

通过以下方法可获得 `YAxis` 类实例 ：

```java
YAxis leftAxis = chart.getAxisLeft();
YAxis rightAxis = chart.getAxisRight();

YAxis leftAxis = chart.getAxis(AxisDependency.LEFT);

YAxis yAxis = radarChart.getYAxis(); // this method radarchart only
```

- `setStartAtZero(boolean enabled)` : 设置为 true，则无论图表显示的是哪种类型的数据，该轴最小值总是0 。

- `setAxisMaxValue(float max)` : 设置该轴的最大值。 如果设置了，这个值将不会是根据提供的数据计算出来的。

- `resetAxisMaxValue()` : 调用此方法撤销先前设置的最大值。 通过这样做，你将再次允许轴自动计算出它的最大值。

- `setAxisMinValue(float min)` : 设置该轴的自定义最小值。 如果设置了，这个值将不会是根据提供的数据计算出来的。

- `resetAxisMinValue()` : 调用此撤销先前设置的最小值。 通过这样做，你将再次允许轴自动计算它的最小值。
  ![](https://img-blog.csdn.net/20151215003245914) ![](https://img-blog.csdn.net/20151215003305229)

  ```java
   // 上面的右图是以下代码设置后的效果图
  leftAxis.setStartAtZero(false);
  leftAxis.setAxisMinValue(30);
  leftAxis.setAxisMaxValue(60);
  ```

- `setInverted(boolean enabled)` : 如果设置为true，该轴将被反转，这意味着最高值将在底部，顶部的最低值。

- `setSpaceTop(float percent)` : 设置图表中的最高值的顶部间距占最高值的值的百分比（设置的百分比 = 最高柱顶部间距/最高柱的值）。默认值是10f，即10% 。
  ![](https://img-blog.csdn.net/20151215004113457)

- `setSpaceBottom(float percent)` : Sets the bottom spacing (in percent of the total axis-range) of the lowest value in the chart in comparison to the lowest value on the axis.

- `setLabelCount(int count, boolean force)` : 设置y轴的标签数量。 请注意，这个数字是不固定 `if(force == false)`，只能是近似的。 如果 `if(force == true)`，则确切绘制指定数量的标签，但这样可能导致轴线分布不均匀。

- `setShowOnlyMinMax(boolean enabled)` : 如果启用，该轴将只显示它的最小值和最大值。 如果 `force == true` 这可能会被 `忽略/覆盖` 。

- `setPosition(YAxisLabelPosition pos)` : 设置，其中轴标签绘制的位置。 无论是 `OUTSIDE_CHART` 或 `INSIDE_CHART` 。

- `setValueFormatter(YAxisValueFormatterf)` : 设置该轴的自定义 ValueFormatter 。 该接口允许 格式化/修改 原来的标签文本，返回一个自定义的文本。

##### 4.2 手势交互

MPAndroidChart 这个库完全支持图表进行触摸和手势的交互，通过回调方法做出对应的操作。

###### 启用/ 禁止手势交互

- `setTouchEnabled(boolean enabled)` : 启用/禁用与图表的所有可能的触摸交互。
- `setDragEnabled(boolean enabled)` : 启用/禁用拖动（平移）图表。
- `setScaleEnabled(boolean enabled)` : 启用/禁用缩放图表上的两个轴。
- `setScaleXEnabled(boolean enabled)` : 启用/禁用缩放在x轴上。
- `setScaleYEnabled(boolean enabled)` : 启用/禁用缩放在y轴。
- `setPinchZoom(boolean enabled)` : 如果设置为true，捏缩放功能。 如果false，x轴和y轴可分别放大。
- `setDoubleTapToZoomEnabled(boolean enabled)` : 设置为false以禁止通过在其上双击缩放图表。
- `setHighlightPerDragEnabled(boolean enabled)` : 设置为true，允许每个图表表面拖过，当它完全缩小突出。 默认值：true
- `setHighlightPerTapEnabled(boolean enabled)` : 设置为false，以防止值由敲击姿态被突出显示。 值仍然可以通过拖动或编程方式突出显示。 默认值：true

###### 图表的 抛掷/减速

- `setDragDecelerationEnabled(boolean enabled)` : 如果设置为true，手指滑动抛掷图表后继续减速滚动。 默认值：true。
- `setDragDecelerationFrictionCoef(float coef)` : 减速的摩擦系数在[0; 1]区间，数值越高表示速度会缓慢下降，例如，如果将其设置为0，将立即停止。 1是一个无效的值，会自动转换至0.9999。

###### 高亮

- `highlightValues(Highlight[] highs)` : 高亮显示值，高亮显示的点击的位置在数据集中的值。 设置null或空数组则撤消所有高亮。
- `highlightValue(int xIndex, int dataSetIndex)` : 高亮给定xIndex在数据集的值。 设置xIndex或dataSetIndex为-1撤消所有高亮。
- `getHighlighted()` : 返回一个 `Highlight[]` 其中包含所有高亮对象的信息，xIndex和dataSetIndex。



###### 选择回调

MPAndroidChart 提供了许多用于交互回调的方法，其中 `OnChartValueSelectedListener` 在点击高亮值时回调。

```java
public interface OnChartValueSelectedListener {
    /**
    * Called when a value has been selected inside the chart.
    *
    * @param e The selected Entry.
    * @param dataSetIndex The index in the datasets array of the data object
    * the Entrys DataSet is in.
    * @param h the corresponding highlight object that contains information
    * about the highlighted position
    */
    public void onValueSelected(Entry e, int dataSetIndex, Highlight h);
    /**
    * Called when nothing has been selected or an "un-select" has been made.
    */
    public void onNothingSelected();
}
```

###### 手势回调

监听器 `OnChartGestureListener` 可以使得 chart 与手势操作进行交互。

```java
public interface OnChartGestureListener {

    /**
     * Callbacks when a touch-gesture has started on the chart (ACTION_DOWN)
     *
     * @param me
     * @param lastPerformedGesture
     */
    void onChartGestureStart(MotionEvent me, ChartTouchListener.ChartGesture lastPerformedGesture);

    /**
     * Callbacks when a touch-gesture has ended on the chart (ACTION_UP, ACTION_CANCEL)
     *
     * @param me
     * @param lastPerformedGesture
     */
    void onChartGestureEnd(MotionEvent me, ChartTouchListener.ChartGesture lastPerformedGesture);

    /**
     * Callbacks when the chart is longpressed.
     * 
     * @param me
     */
    public void onChartLongPressed(MotionEvent me);

    /**
     * Callbacks when the chart is double-tapped.
     * 
     * @param me
     */
    public void onChartDoubleTapped(MotionEvent me);

    /**
     * Callbacks when the chart is single-tapped.
     * 
     * @param me
     */
    public void onChartSingleTapped(MotionEvent me);

    /**
     * Callbacks then a fling gesture is made on the chart.
     * 
     * @param me1
     * @param me2
     * @param velocityX
     * @param velocityY
     */
    public void onChartFling(MotionEvent me1, MotionEvent me2, float velocityX, float velocityY);

   /**
     * Callbacks when the chart is scaled / zoomed via pinch zoom gesture.
     * 
     * @param me
     * @param scaleX scalefactor on the x-axis
     * @param scaleY scalefactor on the y-axis
     */
    public void onChartScale(MotionEvent me, float scaleX, float scaleY);

   /**
    * Callbacks when the chart is moved / translated via drag gesture.
    *
    * @param me
    * @param dX translation distance on the x-axis
    * @param dY translation distance on the y-axis
    */
    public void onChartTranslate(MotionEvent me, float dX, float dY);
}
```

##### 4.3 ChartData类

`ChartData` 类是所有数据类的基类，比如 `LineData`，`BarData` 等，它是用来为 `Chart` 提供数据的，通过 `setData(ChartData data){...}` 方法。


###### 设置Data的Style

以下提到的方法是在 `ChartData` 类中被实现，因此可用于所有子类：

- `setDrawValues(boolean enabled)` : 启用/禁用 绘制所有 `DataSets` 数据对象包含的数据的值文本。
- `setValueTextColor(int color)` : 设置 `DataSets` 数据对象包含的数据的值文本的颜色。
- `setValueTextSize(float size)` : 设置 `DataSets` 数据对象包含的数据的值文本的大小（单位是dp）。
- `setValueTypeface(Typeface tf)` : 设置Typeface的所有价值标签的所有DataSets这些数据对象包含。
- `setValueFormatter(ValueFormatter f)` : 为`DataSets` 数据对象包含的数据设置自定义的 `ValueFormatter` 。
- `getDataSetByIndex(int index)` : 返回目标 `DataSet` 列表中给定索引的数据对象。
- `contains(Entry entry)` : 检查此数据对象是否包含指定的Entry 。 注：这个相当影响性能，性能严峻情况下，不要过度使用。
- `contains(T dataSet)` : Returns true if this data object contains the provided DataSet , false if not.
- clearValues() : 清除所有 DataSet 对象和所有 Entries 的数据 。 不会删除所提供的 x-values ; 
- `setHighlightEnabled(boolean enabled)` : 设置为true，允许通过点击高亮突出 `ChartData` 对象和其 `DataSets` 。

###### 设置数据

想为图表添加数据，你可以通过下面这个方法：

```java
 public void setData(ChartData data) { ... }
```

基类 `ChartData` 封装了渲染过程中所需要的图表中的所有数据和信息。 对于每种类型的图表，不同的 `ChartData` 子类（例如 `LineData`）应该被用于为图表设置数据。 在构造函数中，你可以通过 `ArrayList` 作为要显示的值，一个额外 `ArrayList` 的 `String` 用来描述 x 轴上的标签。

```java
  // this is just one of many constructors
    public LineData(ArrayList<String> xVals, ArrayList<LineDataSet> sets) { ... }
```

一个 `DataSet` 对象代表一组 `entries`（数据类型 `Entry`），在图表内属于一个整体。 它在图表中被设计成 **逻辑上分离的不同组的值** 。 每种类型的图表，通过一个不同的 `DataSet` 对象（如 `LineDataSet`）来做出特定的 style 。

###### 设置颜色

ColorTemplate 类： 该类封装有 **预定义的颜色整数数组**（例如 `ColorTemplate.VORDIPLOM_COLORS`）和便利的从资源加载颜色的方法；当时自从 MPAndroidChart V1.4.0 之后，`ColorTemplate` 这个类就不再重要了。我们可以直接通过 `DataSet` 对象进行指定颜色，从而可以区分每个 `DataSet` 的 Style 。

其它方法设置颜色：

- `setColors(int [] colors, Context c)` : 设置该 DataSet 的颜色。
  您可以使用 `new int[] {R.color.red，R.color.green，...}` 使得颜色值可以重用。 在内部，颜色是使用 `getResources().getColor(...)` 来实现获取的。
- `setColors(int [] colors)` : 设置该 DataSet 的颜色。Colors are reused as soon as the number of Entries the DataSet represents is higher than the size of the colors array. Make sure that the colors are already prepared (by calling `getResources().getColor(...))` before adding them to the DataSet.
- `setColors(ArrayList colors)` : 设置该 DataSet 的颜色。Sets the colors that should be used fore this DataSet. Colors are reused as soon as the number of Entries the DataSet represents is higher than the size of the colors array. Make sure that the colors are already prepared (by calling `getResources().getColor(...))` before adding them to the DataSet.
- `setColor(int color)` : 设置该数据集 **唯一的颜色**。 在内部，实现方式类似上面的”颜色数组”，只不过”颜色数组都是同一种颜色”

```kotlin
val ySet = BarDataSet(yValues, "Data Set")
ySet.setColors(context.getColor(R.color.my_bill_color_324dff)) //未选中颜色
ySet.highLightColor = Color.rgb(254, 149, 95) // #FE965F//选中颜色
```

##### 4.4 动画

PAndroidChart 的动画机制只在Android `API 11 (Android 3.0.x)` 和以上有效。
在低于 **Android 3.0.x** 的版本中动画不会执行（但不会引起 crash）。

所有图表类型都支持动画,可以用来创建/建立图表在一个很棒的方法。三种不同的动画方法存在,动画,或者x轴和y轴分别:

- `animateX(int durationMillis)` : 水平轴的图表值动画，这意味着在指定的时间内从左到右 **建立图表**。
- `animateY(int durationMillis)` : 垂直轴的图表值动画，这意味着在指定的时间内从下到上 **建立图表**。
- `animateXY(int xDuration, int yDuration)` : 两个轴的图表值动画，从左到右，从下到上 **建立图表**。

任意一种 `animate(...)` 动画方法被调用后，无需再调用 `invalidate()` 方法。

LineChart 效果图

![animateX(8000); // 图1](https://img-blog.csdn.net/20151222170138194) ![animateY(8000); // 图2](https://img-blog.csdn.net/20151222170212425)
![animateXY(8000, 8000); // 图3](https://img-blog.csdn.net/20151222170225889) ![animateY(8000, Easing.EasingOption.EaseInElastic ); // 图4](https://img-blog.csdn.net/20151222170238137)

```java
    // 设置动画
    chart.animateX(8000); // 图1
    chart.animateY(8000); // 图2
    chart.animateXY(8000, 8000); // 图3
    chart.animateY(8000, Easing.EasingOption.EaseInElastic ); // 图4
```

BarChart 效果图

![animateX(8000); // 图1](https://img-blog.csdn.net/20151222171642638) ![animateY(8000); // 图2](https://img-blog.csdn.net/20151222171701870)
![animateXY(8000, 8000); // 图3](https://img-blog.csdn.net/20151222171719427) ![animateY(8000, Easing.EasingOption.EaseInSine); // 图4](https://img-blog.csdn.net/20151222171736337)

```java
    // 设置动画
    chart.animateX(8000); // 图1
    chart.animateY(8000); // 图2
    chart.animateXY(8000, 8000); // 图3
    chart.animateY(8000, Easing.EasingOption.EaseInSine); // 图412345
```

 PieChart 效果图

![animateX(8000); // 图1](https://img-blog.csdn.net/20151222181636046) ![animateY(8000); // 图2](https://img-blog.csdn.net/20151222181655086)
![animateXY(8000,8000); // 图3](https://img-blog.csdn.net/20151222181713391) ![animateY(8000, Easing.EasingOption.EaseOutBounce); // 图4](https://img-blog.csdn.net/20151222181805041)

```java
    // 设置动画
    chart.animateX(8000); // 图1
    chart.animateY(8000); // 图2
    chart.animateXY(8000,8000); // 图3
    chart.animateY(8000, Easing.EasingOption.EaseOutBounce); // 图4
```



#### 其它

如果想让它左右移动需要设置它的横向缩放相应的比例：

```java
Matrix m = new Matrix();  
m.postScale(scaleX, 1f);//两个参数分别是x,y轴的缩放比例。例如：将x轴的数据放大为之前的1.5倍  
mChart.getViewPortHandler().refresh(m, mChart, false);//将图表动画显示之前进行缩放  
```

视图移动到最右边：

```java
// move to last one entry to display.
barChart.moveViewTo(barChart.xChartMax, 0f, AxisDependency.RIGHT)
```

自定义MarkerView并让它显示要柱上中间位置：

```kotlin
    class MyMarkerView(context: Context)
        : MarkerView(context, R.layout.my_repayment_sum_marker_view_layout) {
        private val mContentTv: TextView by lazy { findViewById<TextView>(R.id.marker_tv) }

        override fun refreshContent(e: Entry?, highlight: Highlight?) {
            if (e == null) {
                return
            }
            mContentTv.text = "￥${e.y}"
            super.refreshContent(e, highlight)
        }

        override fun getOffset(): MPPointF {
            // set position of marker view.
            return MPPointF(-(width / 2).toFloat(), -height.toFloat())
        }
    }
```

