## AndroidStudio的内存分析器（Memory Profiler）使用笔记

[TOC]

官网介绍使用：https://developer.android.com/studio/profile/memory-profiler?hl=zh-cn

#### 内存计算方式

![img](https://developer.android.com/static/studio/images/profile/memory-profiler-counts_2x.png?hl=zh-cn)

**图 2.** 内存分析器顶部的内存计数图例

内存计数中的类别如下：

- **Java**：从 Java 或 Kotlin 代码分配的对象的内存。

- **Native**：从 C 或 C++ 代码分配的对象的内存。

  即使您的应用中不使用 C++，您也可能会看到此处使用了一些原生内存，因为即使您编写的代码采用 Java 或 Kotlin 语言，Android 框架仍使用原生内存代表您处理各种任务，如处理图像资源和其他图形。

- **Graphics**：图形缓冲区队列为向屏幕显示像素（包括 GL 表面、GL 纹理等等）所使用的内存。（请注意，这是与 CPU 共享的内存，不是 GPU 专用内存。）

- **Stack**：您的应用中的原生堆栈和 Java 堆栈使用的内存。这通常与您的应用运行多少线程有关。

- **Code**：您的应用用于处理代码和资源（如 dex 字节码、经过优化或编译的 dex 代码、.so 库和字体）的内存。

- **Others**：您的应用使用的系统不确定如何分类的内存。

- **Allocated**：您的应用分配的 Java/Kotlin 对象数。此数字没有计入 C 或 C++ 中分配的对象。

#### 查看内存分配情况

内存分配情况图表为您显示内存中每个 Java 对象和 JNI 引用的分配方式。具体而言，内存分析器可为您显示有关对象分配情况的以下信息：

- 分配了哪些类型的对象以及它们使用多少空间。
- 每个分配的堆栈轨迹，包括在哪个线程中。
- 对象在何时被取消分配（仅当使用搭载 Android 8.0 或更高版本的设备时）。

![img](https://developer.android.com/static/studio/images/profile/memory-profiler-allocations-detail_2x.png?hl=zh-cn)

您可以使用已分配对象列表上方的两个菜单选择需检查的堆以及如何组织数据。

**从左侧的菜单中，选择需检查的堆：**

- **default heap**：当系统未指定堆时。
- **image heap**：系统启动映像，包含启动期间预加载的类。此处的分配确保绝不会移动或消失。
- **zygote heap**：写时复制堆，其中的应用进程是从 Android 系统中派生的。
- **app heap**：您的应用在其中分配内存的主堆。
- **JNI heap**：显示 Java 原生接口 (JNI) 引用被分配和释放到什么位置的堆。

**从右侧的菜单中，选择如何安排分配：**

- **Arrange by class**：根据类名称对所有分配进行分组。这是默认值。
- **Arrange by package**：根据软件包名称对所有分配进行分组。
- **Arrange by callstack**：将所有分配分组到其对应的调用堆栈。

**挑选显现哪些类**

- **Show all classes**：展示一切Class类（包含体系类），这是默认值。
- **Show activity/fragment Leaks**：展示走漏的activity/fragment。
- **Show project class**：展示项目的Class类。

在最右侧的搜索框，这儿能够查找你要查找的类，比方你怀疑某个目标存在内存走漏，能够直接查找这个类。

**在`Leaks`点击这儿能够直接显现asp供给给咱们的内存走漏状况**

**在下方的详细框里， 这儿显现当时设备内存运用具体状况**

- **Allocations**：当时内存中类目标个数
- **Native Size**：此目标类型运用的原生内存总量（以字节为单位）。只要在运用 Android 7.0 及更高版别时，才会看到此列
  您会在此处看到选用 Java 分配的某些目标的内存，由于 Android 对某些结构类（如 Bitmap）运用原生内存。
- **Shallow Size**：此目标自身占有的内存（以字节为单位）。
- **Retained Size**：此目标引证链上的一切目标的总内存运用（以字节为单位）
- **Depth**：从任意 GC 根到选定实例的最短跳数

#### 查看全局 JNI 引用

Java 原生接口 (JNI) 是一个允许 Java 代码和原生代码相互调用的框架。

JNI 引用由原生代码进行管理，因此原生代码使用的 Java 对象可能会保持活动状态过长时间。如果丢弃了 JNI 引用而未先明确将其删除，Java 堆上的某些对象可能会变得无法访问。此外，还可能会达到全局 JNI 引用限制。

如需排查此类问题，请使用内存分析器中的 **JNI heap** 视图浏览所有全局 JNI 引用，并按 Java 类型和原生调用堆栈对其进行过滤。借助此信息，您可以了解创建和删除全局 JNI 引用的时间和位置。

在您的应用运行时，选择您要检查的一部分时间轴，然后从类列表上方的下拉菜单中选择 **JNI heap**。 您随后可以像往常一样检查堆中的对象，还可以双击 **Allocation Call Stack** 标签页中的对象，以查看在代码中将 JNI 引用分配和释放到了什么位置，如图 4 所示。

![img](https://developer.android.com/static/studio/images/memory-profiler-jni-heap_2x.png?hl=zh-cn)

#### 原生内存分析器

原生内存分析器会跟踪特定时间段内采用原生代码表示的对象的分配/解除分配情况，并提供以下信息：

- **Allocations**：在选定时间段内通过 `malloc()` 或 `new` 运算符分配的对象数。
- **Deallocations**：在选定时间段内通过 `free()` 或 `delete` 运算符解除分配的对象数。
- **Allocations Size**：在选定时间段内所有分配的总大小（以字节为单位）。
- **Deallocations Size**：在选定时间段内所有已释放内存的总大小（以字节为单位）。
- **Total Count**：**Allocations** 列中的值减去 **Deallocations** 列中的值所得的结果。
- **Remaining Size**：**Allocations Size** 列中的值减去 **Deallocations Size** 列中的值所得的结果。


![原生内存分析器](https://developer.android.com/static/studio/images/profile/native_memory_profiler.png?hl=zh-cn)

如需在搭载 Android 10 及更高版本的设备上录制原生分配情况，请选择 **Record native allocations**，然后选择 **Record**。录制会持续到您点击 **Stop** 图标后，之后内存分析器界面会转换到显示原生录制的单独屏幕。

![“Record native allocations”按钮](https://developer.android.com/static/studio/images/profile/record_native_allocations.png?hl=zh-cn)

在 Android 9 及更低版本中，**Record native allocations** 选项不可用。

默认情况下，原生内存分析器使用的采样大小为 32 个字节：每次分配 32 个字节的内存时，系统都会截取内存的快照。示例规模越小，快照越频繁，从而产生更准确的内存用量数据。示例规模越大，数据的准确性越差，但占用的系统资源越少，因此录制时的性能就越高。

> **注意**：原生内存分析器提供的内存数据不同于 [Java 堆的内存分析器](https://developer.android.com/studio/profile/memory-profiler?hl=zh-cn#record-allocations)提供的数据。 原生内存分析器只会跟踪通过 C/C++ 分配器（包括原生 JNI 对象）进行的分配，而不会对 Java 堆上的对象进行性能剖析。
>
> 原生内存分析器基于性能分析工具的 Perfetto 堆栈中的 `heapprofd` 构建。如需详细了解原生内存分析器的内部信息，请参阅 [`heapprofd` 文档](https://perfetto.dev/docs/data-sources/native-heap-profiler)。

#### 捕获堆转储

堆转储显示在您捕获堆转储时您的应用中哪些对象正在使用内存。特别是在长时间的用户会话后，堆转储会显示您认为不应再位于内存中却仍在内存中的对象，从而帮助识别内存泄漏。

捕获堆转储后，您可以查看以下信息：

- 您的应用分配了哪些类型的对象，以及每种对象有多少。
- 每个对象当前使用多少内存。
- 在代码中的什么位置保持着对每个对象的引用。
- 对象所分配到的调用堆栈。（目前，对于 Android 7.1 及更低版本，只有在记录分配期间捕获堆转储时，才会显示调用堆栈的堆转储。）

如需捕获堆转储，请点击 **Capture heap dump**，然后选择 **Record**。在转储堆期间，Java 内存量可能会暂时增加。 这很正常，因为堆转储与您的应用发生在同一进程中，并需要一些内存以收集数据。

在分析器捕获堆转储后，内存分析器界面将转换到显示堆转储的单独屏幕。

![img](https://developer.android.com/static/studio/images/profile/profiler-heap-dump-display.png?hl=zh-cn)

**图 5.** 查看堆转储。

如果您需要使转储创建时间更加确切，可以通过调用 `dumpHprofData()` 在应用代码的关键点创建堆转储。

在类列表中，您可以查看以下信息：

- **Allocations**：堆中的分配数。

- **Native Size**：此对象类型使用的原生内存总量（以字节为单位）。只有在使用 Android 7.0 及更高版本时，才会看到此列。

  您会在此处看到采用 Java 分配的某些对象的内存，因为 Android 对某些框架类（如 `Bitmap`）使用原生内存。

- **Shallow Size**：此对象类型使用的 Java 内存总量（以字节为单位）。

- **Retained Size**：为此类的所有实例而保留的内存总大小（以字节为单位）。

您可以使用已分配对象列表上方的两个菜单选择要检查的堆转储以及如何组织数据。

从左侧的菜单中，选择需检查的堆：

- **default heap**：当系统未指定堆时。
- **app heap**：您的应用在其中分配内存的主堆。
- **image heap**：系统启动映像，包含启动期间预加载的类。此处的分配确保绝不会移动或消失。
- **zygote heap**：写时复制堆，其中的应用进程是从 Android 系统中派生的。

从右侧的菜单中，选择如何安排分配：

- **Arrange by class**：根据类名称对所有分配进行分组。这是默认值。
- **Arrange by package**：根据软件包名称对所有分配进行分组。
- **Arrange by callstack**：将所有分配分组到其对应的调用堆栈。只有在记录分配期间捕获堆转储时，此选项才有效。即便如此，堆中也很可能有在您开始记录之前分配的对象，所以会先显示这些分配，直接按类名称列出它们。

默认情况下，此列表按 **Retained Size** 列排序。如需按其他列中的值排序，请点击该列的标题。

点击一个类名称可在右侧打开 **Instance View** 窗口（如图 6 所示）。列出的每个实例都包含以下信息：

- **Depth**：从任意 GC 根到选定实例的最短跳数。
- **Native Size**：原生内存中此实例的大小。 只有在使用 Android 7.0 及更高版本时，才会看到此列。
- **Shallow Size**：Java 内存中此实例的大小。
- **Retained Size**：此实例所支配内存的大小（根据[支配项树](https://en.wikipedia.org/wiki/Dominator_(graph_theory))）。

**注意**：默认情况下，堆转储不会向您显示每个已分配对象的堆栈轨迹。如需获取堆栈轨迹，在点击 **Capture heap dump** 之前，您必须先开始[记录内存分配](https://developer.android.com/studio/profile/memory-profiler?hl=zh-cn#record-allocations)。然后，您可以在 **Instance View** 中选择一个实例，并查看 **References** 标签页旁边的 **Call Stack** 标签页，如图 6 所示。不过，在您开始记录分配之前，可能已分配一些对象，因此不会显示这些对象的调用堆栈。 包含调用堆栈的实例在图标 ![img](https://developer.android.com/static/studio/images/profile/memory-profiler-icon-stack.png?hl=zh-cn) 上用一个“堆栈”标志表示。 （遗憾的是，由于堆栈轨迹需要您执行分配记录，因此您目前无法在 Android 8.0 上查看堆转储的堆栈轨迹。）

![img](https://developer.android.com/static/studio/images/profile/memory-profiler-dump-stacktrace_2x.png?hl=zh-cn)

**图 6.** 在时间轴上标示捕获堆转储所需的持续时间

#####  怎么检查内存走漏的引证链

点击或许走漏的类型，然后点击具体列表的“References”，并把“Show nearest GC root only”勾上。
就能够显现当时目标走漏的引证链。这儿看到是由于MyApp中的viewList目标引证了MainActivity中的context导致。

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/21ca523384154e0aa9871fa00dc088f5~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.image?)

##### **怎么检查一段时刻内的内存运用状况：**

在Memory界面单击，然后就会显现一段区间，开发者能够依据需求挑选开端和完毕时刻。

![asp差异.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9509c59225554d59948349cc2cfa0b62~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.image?)

不止上面这些功用，ASP还能够对办法路径上的目标创立个数和巨细进行**可视化调查**，那这个功用就很强壮了

![asp visual.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9a39f72b64d342a88a9d64c57309539a~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.image?)



#### 将堆转储另存为 HPROF 文件

捕获堆转储后，只有在内存分析器正在运行时，才能在该分析器中查看数据。当您退出分析会话时，会丢失堆转储。因此，如果您要保存堆转储以供日后查看，请将其导出到 HPROF 文件。在 Android Studio 3.1 及更低版本中，**Export capture to file** 按钮 ![img](https://developer.android.com/static/studio/images/buttons/profiler-export-hprof.png?hl=zh-cn) 位于时间轴下方工具栏的左侧；在 Android Studio 3.2 及更高版本中，**Sessions** 窗格中每个 **Heap Dump** 条目的右侧都有一个 **Export Heap Dump** 按钮。在随即显示的 **Export As** 对话框中，使用 `.hprof` 文件扩展名保存文件。

如需使用其他 HPROF 分析器（如 [jhat](https://docs.oracle.com/javase/8/docs/technotes/tools/unix/jhat.html)），您需要将 HPROF 文件从 Android 格式转换为 Java SE HPROF 格式。 您可以使用 `android_sdk/platform-tools/` 目录中提供的 `hprof-conv` 工具执行此操作。运行包含两个参数（即原始 HPROF 文件和转换后 HPROF 文件的写入位置）的 `hprof-conv` 命令。例如：

```
hprof-conv heap-original.hprof heap-converted.hprof
```

#### 导入堆转储文件

如需导入一个 HPROF (`.hprof`) 文件，请点击 **Sessions** 窗格中的 **Start a new profiling session** 图标 ![img](https://developer.android.com/static/studio/images/buttons/ic_plus.png?hl=zh-cn)，选择 **Load from file**，然后从文件浏览器中选择该文件。

您还可以通过将 HPROF 文件从文件浏览器拖动到编辑器窗口导入该文件。

#### 对内存进行性能剖析的技巧

使用内存分析器时，您应对应用代码施加压力并尝试强制内存泄漏。在应用中引发内存泄漏的一种方式是，先让其运行一段时间，然后再检查堆。泄漏在堆中可能逐渐汇聚到分配顶部。不过，泄漏越小，为了发现泄漏而需要运行应用的时间就越长。

您还可以通过以下某种方式触发内存泄漏：

- 在不同的 activity 状态下，先将设备从竖屏旋转为横屏，再将其旋转回来，这样反复旋转多次。旋转设备经常会使应用泄漏 `Activity`、`Context` 或 `View` 对象，因为系统会重新创建 `Activity`，而如果您的应用在其他地方保持对这些对象其中一个的引用，系统将无法对其进行垃圾回收。
- 在不同的 Activity 状态下，在您的应用与其他应用之间切换（导航到主屏幕，然后返回到您的应用）。