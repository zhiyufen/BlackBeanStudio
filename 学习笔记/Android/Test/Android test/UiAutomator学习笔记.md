## UiAutomator学习笔记

[TOC]

### 1. 概念和配置

`UI Automator`是一个UI测试框架，适用于跨系统和已安装应用程序的跨应用程序功能UI测试。

`UI Automator`测试框架是一个基于`instrumentation`的API，与`AndroidJUnitRunner`测试运行程序一起工作。它非常适合编写黑盒子的自动测试，其中测试代码不依赖于目标应用程序的内部实现细节。

UiAutomator提供了以下两种工具来支持UI自动化测试：

- uiautomatorviewer：用来分析UI控件的图形界面工具，位于SDK目录下的tools文件夹中。
- uiautomator：一个java库，提供执行自动化测试的各种API。

```groovy
    dependencies {
        ...
        androidTestImplementation 'androidx.test.uiautomator:uiautomator:2.2.0'
    }
```

### 2. 基础知识

#### 获取设备状态

`UI Automator`框架提交`UiDevice`类对设备进行获取相应信息和执行操作：

- 修改手机方向， 点击电源上键，点击返回键，Home键，菜单键，打开通知栏，获取当前屏幕的截图等；

```java
 Context context = InstrumentationRegistry.getInstrumentation().getTargetContext();

// Initialize UiDevice instance
UiDevice device = UiDevice.getInstance(InstrumentationRegistry.getInstrumentation());

// 点击Home键
uiDevice.pressHome();
// 点击返回键
uiDevice.pressBack();
```

#### 常用API

`UI Automator` `API`允许您编写健壮的测试，而无需了解目标应用程序的实现细节。

- `UiObject2`
  代表一个UI控件，通过uiDevice的findObject(UiSelector)方法获取，获取到`UiObject2`实例后，就可以对UI控件进行相关的操作，比如点击、长按等：

  ```java
  device = UiDevice.getInstance(InstrumentationRegistry.getInstrumentation())
  device.pressHome()
  
  val gmail: UiObject2 = device.findObject(By.text("Gmail"))
  // Perform a click and wait until the app is opened.
  val opened: Boolean = gmail.clickAndWait(Until.newWindow(), 3000)
  assertThat(opened).isTrue()
  ```

- `BySelector`
  用于获取某些符合条件的UI控件对象，可以通过资源id、描述等熟悉获取：

  ```java
  // 通过资源id获取
  new UiSelector().resourceId("com.yang.designsupportdemo:id/CollapsingToolbarLayout");
  
  // 通过描述文件获取
  new UiSelector().description("Navigate up")
  
  // 通过className获取
  new UiSelector().className("android.support.v7.widget.RecyclerView")
  
  // 通过By工具类快速拿(推荐使用这个)
  By.text("Gmail")
...
  ```
  
- `By`: 能以简洁的方式构造BySelector；

  ```kotlin
  val okButton: UiObject2 = device.findObject(
      By.text("OK").clazz("android.widget.Button")
  )
  ```

  

- `Configurator`： 用于设置运行UI Automator测试的关键参数

- `UiCollection`
  代表UI控件集合，相当于ViewGroup，比如界面中有多个CheckBox时，可以通过类名获取到当前界面下的所有CheckBox，然后通过控件id获取指定的CheckBox对象：

  ```java
  // 获取指定的CheckBox对象
  UiCollection uiCollection = new UiCollection(new UiSelector().className("类名"));
  UiObject checkBox = uiCollection.getChild(new UiSelector().resourceId(""));
  ```

- `UiScrollable`
  代表可滚动的控件，比如打开设置的关于手机选项：

  ```java
  // 滑动列表到最后，点击About phone选项
  UiScrollable settings = new UiScrollable(new UiSelector().className("android.support.v7.widget.RecyclerView"));
  UiObject about = settings.getChildByText(new UiSelector().className("android.widget.LinearLayout"), "About phone");
  about.click();
  ```

  

#### 分析界面元素

启动被测试App，打开`uiautomatorviewer.bat`工具(`<android-sdk>/tools/`)，点击左上角的`Device Screenshot`按钮捕获屏幕快照，左侧显示屏幕快照，右侧显示布局结构与控件属性，控件属性在编写测试用例时会用到。

#### 确定Activity可访问

UI Automator测试框架在实现了Android辅助功能的应用程序上表现更好。当您使用View类型的UI元素或SDK中View的子类时，您不需要实现可访问性支持，因为这些类已经为您实现了这一点。

但一些自定义View需要自己实现；如果您的应用程序包含不是来自SDK的View子类的实例，请确保通过完成以下步骤向这些元素添加可访问性功能：

- 创建一个扩展ExploreByTouchHelper的具体类。

- 通过调用setAccessibilityDelegate（），将新类的实例与特定的自定义UI元素关联起来。

更多请参考 [让自定义视图使用起来更没有障碍](https://developer.android.com/guide/topics/ui/accessibility/custom-views?hl=zh-cn)

#### 创建`UI Automator`测试类

`UI Automator`测试类如 Junit4的测试类一样，在测试类前需要使用`@RunWith(AndroidJUnit4.class)`的注解；

在UI Automator测试类中实现以下编程模型：

1. 通过调用getInstance（）方法并将Instrumentation对象作为参数传递给它，获取一个UiDevice对象以访问要测试的设备。

2. 通过调用findObject（）方法，获取一个UiObject2对象以访问设备上显示的UI组件（例如，前台的当前视图）。

3. 通过调用UiObject2方法，模拟要在该UI组件上执行的特定用户交互；例如，调用scrollUntil（）进行滚动，调用setText（）编辑文本字段。您可以根据需要重复调用步骤2和3中的API，以测试涉及多个UI组件或用户操作序列的更复杂的用户交互。

4. 在执行这些用户交互之后，请检查UI是否反映了预期的状态或行为。

#### 访问UI组件

以下代码片段显示了您的测试如何获取UiDevice的实例并模拟Home按钮按下：

```kotlin

import org.junit.Before
import androidx.test.runner.AndroidJUnit4
import androidx.test.uiautomator.UiDevice
import androidx.test.uiautomator.By
import androidx.test.uiautomator.Until
...

private const val BASIC_SAMPLE_PACKAGE = "com.example.android.testing.uiautomator.BasicSample"
private const val LAUNCH_TIMEOUT = 5000L
private const val STRING_TO_BE_TYPED = "UiAutomator"

@RunWith(AndroidJUnit4::class)
@SdkSuppress(minSdkVersion = 18) //确保该测试例子只跑在 API level 18+
class ChangeTextBehaviorTest2 {

private lateinit var device: UiDevice

@Before
fun startMainActivityFromHomeScreen() {
  // Initialize UiDevice instance
  device = UiDevice.getInstance(InstrumentationRegistry.getInstrumentation())

  // 点击`Home`键
  device.pressHome()

  // 等待并检查 Launcher App是否显示
  val launcherPackage: String = device.launcherPackageName
  assertThat(launcherPackage, notNullValue())
  device.wait(
    Until.hasObject(By.pkg(launcherPackage).depth(0)),
    LAUNCH_TIMEOUT
  )

  // 启动测试的Activity
  val context = ApplicationProvider.getApplicationContext<Context>()
  val intent = context.packageManager.getLaunchIntentForPackage(
  BASIC_SAMPLE_PACKAGE).apply {
    // Clear out any previous instances
    addFlags(Intent.FLAG_ACTIVITY_CLEAR_TASK)
  }
  context.startActivity(intent)

  // 等待并检查测试的Activity是否显示
  device.wait(
    Until.hasObject(By.pkg(BASIC_SAMPLE_PACKAGE).depth(0)),
    LAUNCH_TIMEOUT
    )
  }
}
```

使用`UiDevice`的`findObject()`来找到代表某个View的`UiObject2`对象，使用该`UiObject2`对象可以访问其属性或操行点击等操作；

```kotlin

val okButton: UiObject2 = device.findObject(
    By.text("OK").clazz("android.widget.Button")
)

// Simulate a user-click on the OK button, if found.
if (okButton != null) {
    okButton.click()
}
```

#### 指定Selector

如果要访问应用程序中的特定UI组件，请使用By类构造BySelector实例。BySelector表示对显示的UI中的特定元素的查询。

如果找到多个匹配元素，则布局层次结构中的第一个匹配元素将作为目标UiObject2返回。构造BySelector时，可以将多个特性链接在一起以优化搜索。如果找不到匹配的UI元素，则返回null。

可以使用hasChild（）或hasDescendant（）方法嵌套多个BySelector实例。例如，下面的代码示例显示了测试如何指定搜索来查找具有text属性的子UI元素的第一个ListView。

```kotlin
val listView: UiObject2 = device.findObject(
    By.clazz("android.widget.ListView")
        .hasChild(
            By.text("Apps")
        )
)
```

####  执行操作

当你获取到`UiObject2`对象时，你可以以其操作该UI组件一些操作：

- `click() `: 点击
- `drag()` : 将此对象拖动到任意坐标.
- `setText() `: 在清除字段内容后，设置可编辑字段中的文本。相反，clear（）方法清除可编辑字段中的现有文本
- `swipe() `: 向指定方向执行滑动操作。
- `scrollUntil()`: 向指定方向执行滚动操作，直到满足Condition或EventCondition为止。

同样` UI Automator`框架允许我们以Intent来启动某个Activity, 当你只对测试启动的应用程序感兴趣，而不关心启动器时，这种方法很有用。

```kotlin

fun setUp() {
...

  // Launch a simple calculator app
  val context = getInstrumentation().context
  val intent = context.packageManager.getLaunchIntentForPackage(CALC_PACKAGE).apply {
    addFlags(Intent.FLAG_ACTIVITY_CLEAR_TASK)
  }
  // Clear out any previous instances
  context.startActivity(intent)
  device.wait(Until.hasObject(By.pkg(CALC_PACKAGE).depth(0)), TIMEOUT)
}

```

#### 验证结果

`InstrumentationTestCase`继承于`TestCase`， 因为我们可使用`Junit`的`Assert`工具类来断言；

```kotlin

private const val CALC_PACKAGE = "com.myexample.calc"

fun testTwoPlusThreeEqualsFive() {
  // Enter an equation: 2 + 3 = ?
  device.findObject(By.res(CALC_PACKAGE, "two")).click()
  device.findObject(By.res(CALC_PACKAGE, "plus")).click()
  device.findObject(By.res(CALC_PACKAGE, "three")).click()
  device.findObject(By.res(CALC_PACKAGE, "equals")).click()

  // Verify the result = 5
  val result: UiObject2 = device.findObject(By.res(CALC_PACKAGE, "result"))
  assertEquals("5", result.text)
}
```

### 高级使用

#### 与System UI的交互

UI Automator可以与屏幕上的一切交互，包括应用程序之外的系统元素：

```kotlin

// Opens the System Settings.
device = UiDevice.getInstance(InstrumentationRegistry.getInstrumentation())
device.executeShellCommand("am start -a android.settings.SETTINGS")


// Opens the notification shade.
device = UiDevice.getInstance(InstrumentationRegistry.getInstrumentation())
device.openNotification()


// Opens the Quick Settings shade.
device = UiDevice.getInstance(InstrumentationRegistry.getInstrumentation())
device.openQuickSettings()


// Get the system clock.
device = UiDevice.getInstance(InstrumentationRegistry.getInstrumentation())
UiObject2 clock = device.findObject(By.res("com.android.systemui:id/clock"))
print(clock.getText())
```

#### 等待界面转换

屏幕转换可能需要时间，并且预测其持续时间是不可靠的，所以在执行操作后应该让UI Automator等待。UI Automator为此提供了多种方法：

- `UiDevice.performActionAndWait(Runnable action, EventCondition<U> condition, long timeout)`
  执行某个操作并等待新窗口出现：

  ```kotlin
  device.performActionAndWait(() -> button.click(), Until.newWindow(), timeout)
  ```

- `UiDevice.wait(Condition<Object, U> condition, long timeout)`
  一直等待直到包含着某个`UiObject2`的组件出现：

  ```kotlin
  device.wait(device.hasObject(By.text("my_text")), timeout);
  ```

- `UiObject2.wait(@NonNull Condition<Object, U> condition, long timeout)`
  一直等待直到满足某个条件出现：

  ```kotlin
  checkbox.wait(Until.checked(true), timeout);
  ```

- `UiObject2.clickAndWait(@NonNull EventCondition<U> condition, long timeout)`
  点击该组件并一直等待有新的窗口出现：

  ```kotlin
  button.clickAndWait(Until.newWindow(), timeout);
  ```

- `UiObject2.scrollUntil(@NonNull Direction direction, @NonNull Condition<Object, U> condition)`
  一直滚动该组件到直到包含着某个`UiObject2`的组件出现：

  ```kotlin
  object.scrollUntil(Direction.DOWN, Until.hasObject(By.text('new_obj')));
  ```

- `UiObject2.scrollUntil(@NonNull Direction direction, @NonNull EventCondition<U> condition)`
  一直滚动到该组件的底部：

  ```kotlin
  object.scrollUntil(Direction.DOWN, Until.scrollFinished(Direction.DOWN));
  ```

下面例子演示去关闭`免打扰`模式：

```kotlin

@Test
@SdkSuppress(minSdkVersion = 21)
@Throws(Exception::class)
fun turnOffDoNotDisturb() {
    device = UiDevice.getInstance(InstrumentationRegistry.getInstrumentation())
    device.performActionAndWait({
        try {
            device.executeShellCommand("am start -a android.settings.SETTINGS")
        } catch (e: IOException) {
            throw RuntimeException(e)
        }
    }, Until.newWindow(), 1000)
    // Check system settings has been opened.
    Assert.assertTrue(device.hasObject(By.pkg("com.android.settings")))

    // Scroll the settings to the top and find Notifications button
    var scrollableObj: UiObject2 = device.findObject(By.scrollable(true))
    scrollableObj.scrollUntil(Direction.UP, Until.scrollFinished(Direction.UP))
    val notificationsButton = scrollableObj.findObject(By.text("Notifications"))

    // Click the Notifications button and wait until a new window is opened.
    device.performActionAndWait({ notificationsButton.click() }, Until.newWindow(), 1000)
    scrollableObj = device.findObject(By.scrollable(true))
    // Scroll down until it finds a Do Not Disturb button.
    val doNotDisturb = scrollableObj.scrollUntil(
        Direction.DOWN,
        Until.findObject(By.textContains("Do Not Disturb"))
    )
    device.performActionAndWait({ doNotDisturb.click() }, Until.newWindow(), 1000)
    // Turn off the Do Not Disturb.
    val turnOnDoNotDisturb = device.findObject(By.text("Turn on now"))
    turnOnDoNotDisturb?.click()
    Assert.assertTrue(device.wait(Until.hasObject(By.text("Turn off now")), 1000))
}
```



[API文档](https://developer.android.com/reference/androidx/test/uiautomator/package-summary)

[练习例子](https://github.com/android/testing-samples/tree/main/ui/uiautomator/BasicSample)





### 其它：

```java
    @Rule
    public ActivityScenarioRule<xxxxActivity> rule = new ActivityScenarioRule<>(xxxxActivity.class);

    @Test
    public void myTest() {
        ActivityScenario<xxxxActivity> scenario = rule.getScenario();
        // Your test code goes here.
        scenario.onActivity(xxxxActivity::cancelPopup);
    }
```

