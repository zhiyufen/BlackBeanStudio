## Espresso

可使用 Espresso 来编写简洁、美观且可靠的 Android 界面测试。

### 1. 概念

核心 API 小巧、可预测且易于学习，但仍可进行自定义。Espresso 测试会清楚地说明预期、交互和断言，让您不受样板内容、自定义基础架构或杂乱的实现细节干扰。

Espresso 测试运行速度极快！当它在静止状态下对应用界面进行操纵和断言时，让您无需等待、同步、休眠和轮询。

##### 同步功能

每次测试调用 [`onView()`](https://developer.android.com/reference/androidx/test/espresso/Espresso?hl=zh-cn#onView(org.hamcrest.Matcher)) 时，Espresso 都会等待执行相应的界面操作或断言，直到满足以下同步条件：

- 消息队列为空。
- 没有当前正在执行任务的 `AsyncTask` 实例。
- 开发者定义的所有[空闲资源](https://developer.android.com/training/testing/espresso/idling-resource?hl=zh-cn)都处于空闲状态。

通过执行这些检查，Espresso 大大提高了在任何给定时间只能发生一项界面操作或断言的可能性。此功能可给您带来更可靠的测试结果。

##### 软件包

- espresso-core - 包含核心和基本的 View 匹配器、操作和断言。请参阅基础知识和测试方案。
- espresso-web - 包含 WebView 支持的资源。
- espresso-idling-resource - Espresso 与后台作业同步的机制。
- espresso-contrib - 外部贡献，包含 DatePicker、RecyclerView 和 Drawer 操作、无障碍功能检查以及 CountingIdlingResource。
- espresso-intents - 用于对封闭测试的 intent 进行验证和打桩的扩展。
- espresso-remote - Espresso 的多进程功能的位置。

### 2. 基本知识

使用 Espresso API 时，我们建议测试创建者从用户与应用交互时可能会执行哪些操作（即，找到界面元素并与之交互）的角度进行思考。

同时，该框架阻止直接访问应用的 Activity 和视图，因为保留这些对象并在界面线程外对其执行操作是导致测试不稳定的主要根源。

#### API 组件

Espresso 的主要组件包括：

- **Espresso** - 用于与视图交互（通过 `onView()` 和 `onData()`）的入口点。此外，还公开不一定与任何视图相关联的 API，如 `pressBack()`。
- **ViewMatchers** - 实现 `Matcher` 接口的对象的集合。您可以将其中一个或多个对象传递给 `onView()` 方法，以在当前视图层次结构中找到某个视图。
- **ViewActions** - 可以传递给 `ViewInteraction.perform()` 方法的 `ViewAction` 对象的集合，例如 `click()`。
- **ViewAssertions** - 可以通过 `ViewInteraction.check()` 方法传递的 `ViewAssertion` 对象的集合。在大多数情况下，您将使用 matches 断言，它使用视图匹配器断言当前选定视图的状态。

示例：

```kotlin
    // withId(R.id.my_view) is a ViewMatcher
    // click() is a ViewAction
    // matches(isDisplayed()) is a ViewAssertion
    onView(withId(R.id.my_view))
        .perform(click())
        .check(matches(isDisplayed()))
```

#### 查找视图

在绝大多数情况下，`onView()` 方法采用 hamcrest 匹配器；

Espresso 允许您使用现有的 `ViewMatcher` 对象或您自己的自定义对象来缩小视图范围，从而彻底解决了这一问题。

按 `R.id` 查找视图就像调用 `onView()` 一样简单：

```kotlin
    onView(withId(R.id.my_view))    
```

有时，会在多个视图之间共享 `R.id` 值。发生这种情况时，如果您尝试使用某个特定的 `R.id`，系统会抛出异常，如 `AmbiguousViewMatcherException`。异常消息会为您提供当前视图层次结构的文本表示形式，您可以从中搜索并查找与非唯一 `R.id` 匹配的视图：

```kotlin
    onView(allOf(withId(R.id.my_view), withText("Hello!")))    
```

您也可以选择不反转任何匹配器：

```kotlin
    onView(allOf(withId(R.id.my_view), not(withText("Unwanted"))))    
```

### 注意事项

- 在运行状况良好的应用中，用户可与之交互的所有视图都应包含描述性文本或具有内容描述。如需了解详情，请参阅[改进应用的无障碍功能](https://developer.android.com/guide/topics/ui/accessibility/apps?hl=zh-cn)。如果您无法使用 `withText()` 或 `withContentDescription()` 缩小搜索范围，考虑将其视为无障碍功能错误。
- 使用描述内容最少的匹配器找到您要查找的一个视图。不要过度指定，因为这样会强制框架执行不必要的工作。例如，如果某个视图可由其文本唯一标识，则您无需指定该视图也可从 `TextView` 分配。对于许多视图来说，指定视图的 `R.id` 应该就足够了。
- 如果目标视图在 `AdapterView` 内（如 `ListView`、`GridView` 或 `Spinner`），则 `onView()` 方法可能不起作用。在这些情况下，您应改用 `onData()`。