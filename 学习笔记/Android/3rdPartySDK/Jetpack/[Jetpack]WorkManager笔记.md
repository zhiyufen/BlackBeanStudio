## WorkManager笔记

[TOC]

https://developer.android.com/topic/libraries/architecture/workmanager?hl=zh-cn

### 1.0 相关概念

[WorkManager](https://developer.android.com/reference/androidx/work/WorkManager?hl=zh-cn) 是适合用于持久性工作的推荐解决方案。如果工作始终要通过应用重启和系统重新启动来调度，便是永久性的工作。由于大多数后台处理操作都是通过持久性工作完成的，因此 WorkManager 是适用于后台处理操作的主要推荐 API。

WorkManager 可处理三种类型的永久性工作：

- **立即执行**：必须立即开始且很快就完成的任务，可以加急。
- **长时间运行**：运行时间可能较长（有可能超过 10 分钟）的任务。
- **可延期执行**：延期开始并且可以定期运行的预定任务。

特性：

-  具备更为简单且一致的 API ；
- 使用[工作约束](https://developer.android.com/topic/libraries/architecture/workmanager/how-to/define-work?hl=zh-cn#work-constraints)明确定义工作运行的最佳条件；
- 强大的调度：WorkManager 允许您使用灵活的调度窗口[调度工作](https://developer.android.com/topic/libraries/architecture/workmanager/how-to/define-work?hl=zh-cn)，以运行[一次性](https://developer.android.com/reference/androidx/work/OneTimeWorkRequest?hl=zh-cn)或[重复](https://developer.android.com/reference/androidx/work/PeriodicWorkRequest?hl=zh-cn)工作。您还可以对工作进行标记或命名，以便调度唯一的、可替换的工作以及监控或取消工作组。
  已调度的工作存储在内部托管的 SQLite 数据库中，由 WorkManager 负责确保该工作持续进行，并在设备重新启动后重新调度。
- 加急工作：使用 WorkManager 调度需在后台立即执行的工作。您应该使用[加急工作](https://developer.android.com/topic/libraries/architecture/workmanager/how-to/define-work?hl=zh-cn#expedited)来处理对用户来说很重要且会在几分钟内完成的任务。
- 灵活的重试政策：有时工作会失败。WorkManager 提供了[灵活的重试政策](https://developer.android.com/topic/libraries/architecture/workmanager/how-to/define-work?hl=zh-cn#retries_backoff)，包括可配置的[指数退避政策](https://developer.android.com/reference/androidx/work/BackoffPolicy?hl=zh-cn)。
- 工作链：对于复杂的相关工作，您可以使用直观的接口[将各个工作任务串联起来](https://developer.android.com/topic/libraries/architecture/workmanager/how-to/chain-work?hl=zh-cn)，这样您便可以控制哪些部分依序运行，哪些部分并行运行。对于每项工作任务，您可以[定义工作的输入和输出数据](https://developer.android.com/topic/libraries/architecture/workmanager/how-to/define-work?hl=zh-cn#input_output)。将工作串联在一起时，WorkManager 会自动将输出数据从一个工作任务传递给下一个工作任务。
- 内置线程互操作性：WorkManager 无缝集成 Coroutines 和 RxJava，让您可以插入自己的异步 API，非常灵活。

此外，WorkManager 遵循[低电耗模式](https://developer.android.com/training/monitoring-device-state/doze-standby?hl=zh-cn)等省电功能和最佳做法，因此您在这方面无需担心。

WorkManager 适用于需要**可靠运行**的工作，即使用户导航离开屏幕、退出应用或重启设备也不影响工作的执行。例如：

- 向后端服务发送日志或分析数据。
- 定期将应用数据与服务器同步。

WorkManager 不适用于那些可在应用进程结束时安全终止的进程内后台工作。它也并非对所有需要立即执行的工作都适用的通用解决方案。

虽然协程是适合某些用例的推荐解决方案，但您不应将其用于持久性工作。请务必注意，协程是一个并发框架，而 WorkManager 是一个持久性工作库。同样，AlarmManager 仅适合用于时钟或日历。

| **API**          | **推荐使用场景**           | **与 WorkManager 的关系**                                    |
| :--------------- | :------------------------- | :----------------------------------------------------------- |
| **Coroutines**   | 所有不需要持久的异步工作。 | 协程是在 Kotlin 中退出主线程的标准方式。不过，它们在应用关闭后会释放内存。对于持久性工作，请使用 WorkManager。 |
| **AlarmManager** | 仅限闹钟。                 | 与 WorkManager 不同，AlarmManager 会使设备从低电耗模式中唤醒。因此，它在电源和资源管理方面来讲并不高效。AlarmManager 仅适合用于精确闹钟或通知（例如日历活动）场景，而不适用于后台工作。 |

WorkManager API 是一个适合用来替换先前的 Android 后台调度 API（包括 [FirebaseJobDispatcher](https://developer.android.com/topic/libraries/architecture/workmanager/migrating-fb?hl=zh-cn)、[GcmNetworkManager](https://developer.android.com/topic/libraries/architecture/workmanager/migrating-gcm?hl=zh-cn) 和 [JobScheduler](https://developer.android.com/reference/android/app/job/JobScheduler?hl=zh-cn)）的推荐组件。

### 2.0 基本使用

#### 依赖

```groovy
dependencies {
    val work_version = "2.7.1"

    // (Java only)
    implementation("androidx.work:work-runtime:$work_version")

    // Kotlin + coroutines
    implementation("androidx.work:work-runtime-ktx:$work_version")

    // optional - RxJava2 support
    implementation("androidx.work:work-rxjava2:$work_version")

    // optional - GCMNetworkManager support
    implementation("androidx.work:work-gcm:$work_version")

    // optional - Test helpers
    androidTestImplementation("androidx.work:work-testing:$work_version")

    // optional - Multiprocess support
    implementation "androidx.work:work-multiprocess:$work_version"
}
```

> **注意**：您随时都可以在 [WorkManager 版本页面](https://developer.android.com/jetpack/androidx/releases/work?hl=zh-cn)上找到最新版本的 WorkManager，包括 Beta 版、Alpha 版和候选版本。

#### Worker

工作使用 `Worker` 类定义，相对于Runnable接口；而doWork方法相关于run(); 

但doWork()方法需要返回`Result` 会通知 WorkManager 服务工作是否成功，以及工作失败时是否应重试工作。

- `Result.success()`：工作成功完成。
- `Result.failure()`：工作失败。
- `Result.retry()`：工作失败，应根据其[重试政策](https://developer.android.com/topic/libraries/architecture/workmanager/how-to/define-work#retries_backoff)在其他时间尝试。

```kotlin
class UploadWorker(appContext: Context, workerParams: WorkerParameters):
       Worker(appContext, workerParams) {
   override fun doWork(): Result {

       // Do the work here--in this case, upload the images.
       uploadImages()

       // Indicate whether the work finished successfully with the Result
       return Result.success()
   }
}
```

#### WorkRequest

Worker定义了做什么任务，而`WorkRequest`（及其子类）则定义工作运行方式和时间。

一般来说，除了自定义，可使用系统提供的：

1. [OneTimeWorkRequest](https://developer.android.com/reference/androidx/work/OneTimeWorkRequest)： 适用于调度非重复性工作(一次性)；
2. [PeriodicWorkRequest](https://developer.android.com/reference/androidx/work/PeriodicWorkRequest)： 适用于调度以一定间隔重复执行的工作；

```kotlin
val uploadWorkRequest: WorkRequest = OneTimeWorkRequestBuilder<UploadWorker>().build()
```

#### WorkManager

定义好Worker和WorkReuqest后，需要使用 `enqueue()` 方法将 `WorkRequest` 提交到 `WorkManager`。

```kotlin
WorkManager
    .getInstance(myContext)
    .enqueue(uploadWorkRequest)
```

执行工作器的确切时间取决于 `WorkRequest` 中使用的约束和系统优化方式。WorkManager 经过设计，能够在满足这些约束的情况下提供最佳行为。

### 3.0 深入WorkRequest

如何定义和自定义 `WorkRequest` 对象来处理常见用例:

- 调度一次性工作和重复性工作
- 设置工作约束条件，例如要求连接到 Wi-Fi 网络或正在充电
- 确保至少延迟一定时间再执行工作
- 设置重试和退避策略
- 将输入数据传递给工作
- 使用标记将相关工作分组在一起

WorkRequest 对象包含 WorkManager 调度和运行工作所需的所有信息。其中包括运行工作必须满足的约束、调度信息（例如延迟或重复间隔）、重试配置，并且可能包含输入数据（如果工作需要）。

`WorkRequest` 本身是抽象基类，可用于创建 `OneTimeWorkRequest` 和 `PeriodicWorkRequest` 请求；

#### 调度一次性工作

对于无需额外配置的简单工作，请使用静态方法 `from`：

```
val myWorkRequest = OneTimeWorkRequest.from(MyWork::class.java)
```

对于更复杂的工作，可以使用构建器：

```kotlin
val uploadWorkRequest: WorkRequest =
   OneTimeWorkRequestBuilder<MyWork>()
       // Additional configuration
       .build()
```

#### 调度加急工作

WorkManager 2.7.0 引入了加急工作的概念。这使 WorkManager 能够执行重要工作，同时使系统能够更好地控制对资源的访问权限。

加急工作具有以下特征：

- **重要性**：加急工作适用于对用户很重要或由用户启动的任务。
- **速度**：加急工作最适合那些立即启动并在几分钟内完成的简短任务。
- **配额**：限制前台执行时间的系统级配额决定了加急作业是否可以启动。
- **电源管理**：[电源管理限制](https://developer.android.com/topic/performance/power/power-details)（如省电模式和低电耗模式）不太可能影响加急工作。
- **延迟时间**：系统立即执行加急工作，前提是系统的当前工作负载允许执行此操作。这意味着这些工作对延迟时间较为敏感，不能安排到以后执行。

> **配额**: 系统必须先为加急作业分配应用执行时间，然后才能运行作业。执行时间并非无限制，而是受配额限制。如果您的应用使用其执行时间并达到分配的配额，在配额刷新之前，您无法再执行加急工作。这样，Android 可以更有效地在应用之间平衡资源。
>
> 每个应用均有自己的前台执行时间配额。可用的执行时间取决于[待机模式存储分区](https://developer.android.com/topic/performance/appstandby)和进程的重要性。您可以确定在执行配额不允许立即运行加急作业时会出现什么情况；
>
> **当您的应用在前台运行时，配额不会限制加急工作的执行。仅在应用在后台运行时或当应用移至后台时，执行时间配额才适用。**因此，您应在后台加快要继续的工作。当应用在前台运行时，您可以继续使用 `setForeground()`。

#### 执行加急工作

从 WorkManager 2.7 开始，您的应用可以调用 `setExpedited()` 来声明 `WorkRequest` 应该使用加急作业；

```kotlin
val request = OneTimeWorkRequestBuilder()
    .setExpedited(OutOfQuotaPolicy.RUN_AS_NON_EXPEDITED_WORK_REQUEST)
    .build()

WorkManager.getInstance(context)
    .enqueue(request)
```

上面工作，如果配额允许，它将立即开始在后台运行；

而OutOfQuotaPolicy定义配额政策：

- `OutOfQuotaPolicy.RUN_AS_NON_EXPEDITED_WORK_REQUEST`: 当该应用程序没有工作配额时，这会导致该工作作为普通工作请求运行。
- `OutOfQuotaPolicy.DROP_WORK_REQUEST`: 当该应用程序没有工作配额时，这会导致该工作请求取消，不运行工作。

##### 向后兼容性和前台服务

为了保持加急作业的向后兼容性，WorkManager 可能会在 Android 12 之前版本的平台上运行前台服务。前台服务可以向用户显示通知。

在 Android 12 之前，工作器中的 `getForegroundInfoAsync()` 和 `getForegroundInfo()` 方法可让 WorkManager 在您调用 `setExpedited()` 时显示通知。

如果您想要请求任务作为加急作业运行，则所有的 ListenableWorker 都必须实现 `getForegroundInfo` 方法。

如果未能实现对应的 `getForegroundInfo` 方法，那么在旧版平台上调用 `setExpedited` 时，可能会导致运行时崩溃。

以 Android 12 或更高版本为目标平台时，前台服务仍然可通过对应的 `setForeground` 方法使用。

`setForeground()` 可能会在 Android 12 上抛出运行时异常，并且在[启动受到限制](https://developer.android.com/guide/components/foreground-services#background-start-restrictions)时可能会抛出异常。

##### 工作器

工作器不知道自身所执行的工作是否已加急。不过，在某些版本的 Android 上，如果 `WorkRequest` 被加急，工作器可以显示通知。

为此，WorkManager 提供了 `getForegroundInfoAsync()` 方法，您必须实现该方法，让 WorkManager 在必要时显示通知，以便启动 `ForegroundService`。

##### CoroutineWorker

如果您使用 `CoroutineWorker`，则必须实现 `getForegroundInfo()`。然后，在 `doWork()` 内将其传递给 `setForeground()`。这样做会在 Android 12 之前的版本中创建通知。

```kotlin
class ExpeditedWorker(appContext: Context, workerParams: WorkerParameters):
   CoroutineWorker(appContext, workerParams) {

   override suspend fun getForegroundInfo(): ForegroundInfo {
       return ForegroundInfo(
           NOTIFICATION_ID, createNotification()
       )
   }

   override suspend fun doWork(): Result {
       TODO()
   }

    private fun createNotification() : Notification {
       TODO()
    }

}
```

> 您应该将 `setForeground()` 封装在 `try/catch` 块中，以捕获可能出现的 `IllegalStateException`。如果您的应用此时无法在前台运行，便可能会发生这类异常。在 Android 12 及更高版本中，您可以使用更详细的 `ForegroundServiceStartNotAllowedException`。

[WorkManager代码例子](https://github.com/android/architecture-components-samples/blob/android-s/WorkManagerSample/lib/src/main/java/com/example/background/ImageOperations.kt)

#### 调度定期工作

使用 `PeriodicWorkRequest` 创建定期执行的 `WorkRequest` 对象的方法如下：

```kotlin
val saveRequest =
       PeriodicWorkRequestBuilder<SaveImageToFileWorker>(1, TimeUnit.HOURS)
       // Additional configuration
       .build()
```

> **注意**：可以定义的最短重复间隔是 15 分钟（与 [JobScheduler API](https://developer.android.com/reference/android/app/job/JobScheduler) 相同）。

##### 灵活的运行间隔

某些工作的性质致使其对运行时间敏感，您可以将 `PeriodicWorkRequest` 配置为在每个时间间隔的**灵活时间段**内运行；

定义具有灵活时间段的定期工作，请在创建 `PeriodicWorkRequest` 时传递 `flexInterval` 以及 `repeatInterval`。灵活时间段从 `repeatInterval - flexInterval` 开始，一直到间隔结束。

以下是可在每小时的最后 15 分钟内运行的定期工作的示例：

```kotlin
val myUploadWork = PeriodicWorkRequestBuilder<SaveImageToFileWorker>(
       1, TimeUnit.HOURS, // repeatInterval (the period cycle)
       15, TimeUnit.MINUTES) // flexInterval
    .build()
```

![](https://developer.android.com/images/topic/libraries/architecture/workmanager/how-to/definework-flex-period.png)

重复间隔必须大于或等于 `PeriodicWorkRequest.MIN_PERIODIC_INTERVAL_MILLIS`，而灵活间隔必须大于或等于 `PeriodicWorkRequest.MIN_PERIODIC_FLEX_MILLIS`。

##### 约束对定期工作的影响

您可以对定期工作设置[约束](https://developer.android.com/topic/libraries/architecture/workmanager/how-to/define-work#work-constraints)。例如，您可以为工作请求添加约束，以便工作仅在用户设备充电时运行。在这种情况下，除非满足约束条件，否则即使过了定义的重复间隔，`PeriodicWorkRequest` 也不会运行。这可能会导致工作在某次运行时出现延迟，甚至会因在相应间隔内未满足条件而被跳过。

#### 工作约束

[约束](https://developer.android.com/reference/androidx/work/Constraints)可确保将工作延迟到满足最佳条件时运行。以下约束适用于 WorkManager。

| 类型                 | 说明                                                         |
| -------------------- | ------------------------------------------------------------ |
| **NetworkType**      | 约束运行工作所需的[网络类型](https://developer.android.com/reference/androidx/work/NetworkType)。例如 Wi-Fi (`UNMETERED`)。 |
| **BatteryNotLow**    | 如果设置为 true，那么当设备处于“电量不足模式”时，工作不会运行。 |
| **RequiresCharging** | 如果设置为 true，那么工作只能在设备充电时运行。              |
| **DeviceIdle**       | 如果设置为 true，则要求用户的设备必须处于空闲状态，才能运行工作。在运行批量操作时，此约束会非常有用；若是不用此约束，批量操作可能会降低用户设备上正在积极运行的其他应用的性能。 |
| **StorageNotLow**    | 如果设置为 true，那么当用户设备上的存储空间不足时，工作不会运行。 |

如需创建一组约束并将其与某项工作相关联，请使用一个 `Contraints.Builder()` 创建 `Constraints` 实例，并将该实例分配给 `WorkRequest.Builder()`。

```kotlin
val constraints = Constraints.Builder()
   .setRequiredNetworkType(NetworkType.UNMETERED)
   .setRequiresCharging(true)
   .build()

val myWorkRequest: WorkRequest =
   OneTimeWorkRequestBuilder<MyWork>()
       .setConstraints(constraints)
       .build()
```

如果指定了多个约束，工作将仅在满足所有约束时才会运行。

如果在工作运行时不再满足某个约束，WorkManager 将停止工作器。系统将在满足所有约束后重试工作。

#### 延迟工作

如果工作没有约束，或者当工作加入队列时所有约束都得到了满足，那么系统可能会选择立即运行该工作。如果您不希望工作立即运行，可以将工作指定为在经过一段最短初始延迟时间后再启动。

```kotlin
val myWorkRequest = OneTimeWorkRequestBuilder<MyWork>()
   .setInitialDelay(10, TimeUnit.MINUTES)
   .build()
```

对于PeriodicWorkRequest也可以设置，但只有首次运行时会延迟。

#### 重试和退避政策

如果您需要让 WorkManager 重试工作，可以从工作器返回 `Result.retry()`。然后，系统将根据[退避延迟时间](https://developer.android.com/reference/androidx/work/WorkRequest#DEFAULT_BACKOFF_DELAY_MILLIS)和[退避政策](https://developer.android.com/reference/androidx/work/BackoffPolicy)重新调度工作。

- 退避延迟时间指定了首次尝试后重试工作前的最短等待时间。此值不能超过 10 秒（或 [MIN_BACKOFF_MILLIS](https://developer.android.com/reference/androidx/work/WorkRequest#MIN_BACKOFF_MILLIS)）。
- 退避政策定义了在后续重试过程中，退避延迟时间随时间以怎样的方式增长。WorkManager 支持 2 个退避政策，即 `LINEAR` 和 `EXPONENTIAL`。
  1. `LINEAR` : 表示以线性来增长，比如避延迟时间为10秒，尝试后继续返回 `Result.retry()`，那么接下来会在 20 秒、30 秒、40 秒后重试，以此类推。
  2. `EXPONENTIAL`： 表示以指数来增长，比如避延迟时间为10秒，尝试后继续返回 `Result.retry()`那么重试时长序列将接近 20、40、80 秒，以此类推。

每个工作请求都有退避政策和退避延迟时间。默认政策是 `EXPONENTIAL`，延迟时间为 10 秒，但您可以在工作请求配置中替换此设置。

```kotlin
val myWorkRequest = OneTimeWorkRequestBuilder<MyWork>()
   .setBackoffCriteria(
       BackoffPolicy.LINEAR,
       OneTimeWorkRequest.MIN_BACKOFF_MILLIS,
       TimeUnit.MILLISECONDS)
   .build()
```

在本示例中，最短退避延迟时间设置为允许的最小值，即 10 秒。由于政策为 `LINEAR`，每次尝试重试时，重试间隔都会增加约 10 秒。

#### 标志工作

每个工作请求都有一个[唯一标识符](https://developer.android.com/reference/androidx/work/WorkRequest#getId())，该标识符可用于在以后标识该工作，以便[取消](https://developer.android.com/topic/libraries/architecture/workmanager/how-to/managing-work#cancelling)工作或[观察其进度](https://developer.android.com/topic/libraries/architecture/workmanager/how-to/states-and-observation#observing)。

如果有一组在逻辑上相关的工作，对这些工作项进行标记可能也会很有帮助。通过标记，您一起处理一组工作请求。

`WorkManager.cancelAllWorkByTag(String)` 会取消带有特定标记的所有工作请求，`WorkManager.getWorkInfosByTag(String)` 会返回一个 WorkInfo 对象列表，该列表可用于确定当前工作状态。

```kotlin
val myWorkRequest = OneTimeWorkRequestBuilder<MyWork>()
   .addTag("cleanup")
   .build()
```

最后，可以向单个工作请求添加多个标记。这些标记在内部以一组字符串的形式进行存储。您可以使用 [WorkInfo.getTags()](https://developer.android.com/reference/androidx/work/WorkInfo#getTags()) 获取与 `WorkRequest` 关联的标记集。

从 `Worker` 类中，您可以通过 [ListenableWorker.getTags()](https://developer.android.com/reference/androidx/work/ListenableWorker#getTags()) 检索其标记集。

#### 分配输入数据

您的工作可能需要输入数据才能正常运行， 在创建WorkRequest时， 使用setInputData方法输入值以键值对的形式存储在 `Data` 对象中。WorkManager 会在执行工作时将输入 `Data` 传递给工作。`Worker` 类可通过调用 `Worker.getInputData()` 访问输入参数。

```kotlin
// Define the Worker requiring input
class UploadWork(appContext: Context, workerParams: WorkerParameters)
   : Worker(appContext, workerParams) {

   override fun doWork(): Result {
       val imageUriInput =
           inputData.getString("IMAGE_URI") ?: return Result.failure()

       uploadFile(imageUriInput)
       return Result.success()
   }
   ...
}

// Create a WorkRequest for your Worker and sending it input
val myUploadWork = OneTimeWorkRequestBuilder<UploadWork>()
	//该方法将接受您使用 Data.Builder 创建的 Data 对象
   .setInputData(workDataOf(
       "IMAGE_URI" to "http://..."
   ))
   .build()
```

如需输出返回值，任务应当将其包含在 [`Result`](https://developer.android.com/reference/androidx/work/ListenableWorker.Result) 中，例如返回 [`Result.success(Data)`](https://developer.android.com/reference/androidx/work/ListenableWorker.Result#success(androidx.work.Data))。您可以通过观察任务的 [`WorkInfo`](https://developer.android.com/reference/androidx/work/WorkInfo) 获得输出内容。

```kotlin
WorkManager.getInstance(myContext).getWorkInfoByIdLiveData(myUploadWork.id)
        .observe(this, Observer { info ->
            if (info != null && info.state.isFinished) {
                val myResult = info.outputData.getInt(KEY_RESULT,
                      myDefaultValue)
                // ... do something with the result ...
            }
        })
```

### 4.0 深入工作

#### 工作状态

工作在其整个生命周期内经历了一系列 [`State`](https://developer.android.com/reference/androidx/work/WorkInfo.State) 更改。

对于 [`one-time`](https://developer.android.com/topic/libraries/architecture/workmanager/how-to/define-work#schedule_one-time_work) 工作请求，工作的初始状态为 [`ENQUEUED`](https://developer.android.com/reference/androidx/work/WorkInfo.State#ENQUEUED)：
![](https://developer.android.com/images/topic/libraries/architecture/workmanager/how-to/one-time-work-flow.png)

`SUCCEEDED`、`FAILED` 和 `CANCELLED` 均表示此工作的终止状态。如果您的工作处于上述任何状态，[`WorkInfo.State.isFinished()`](https://developer.android.com/reference/androidx/work/WorkInfo.State#isFinished()) 都将返回 true。

成功和失败状态仅适用于一次性工作和[链式工作](https://developer.android.com/topic/libraries/architecture/workmanager/how-to/chain-work)。[定期工作](https://developer.android.com/topic/libraries/architecture/workmanager/how-to/define-work#schedule_periodic_work)只有一个终止状态 `CANCELLED`。这是因为定期工作永远不会结束。每次运行后，无论结果如何，系统都会重新对其进行调度。

![image](https://developer.android.com/images/topic/libraries/architecture/workmanager/how-to/periodic-work-states.png)

还有一种 `BLOCKED`状态，此状态适用于一系列已编排的工作，或者说工作链。[链接工作](https://developer.android.com/topic/libraries/architecture/workmanager/how-to/chain-work)中介绍了工作链及其状态图。

#### 管理工作

在将工作加入队列时请小心谨慎，以避免重复。例如，应用可能会每 24 小时尝试将其日志上传到后端服务。如果不谨慎，即使作业只需运行一次，您最终也可能会多次将同一作业加入队列。为了实现此目标，您可以将工作调度为[唯一工作](https://developer.android.com/topic/libraries/architecture/workmanager/how-to/managing-work#unique-work)。

##### 唯一工作

唯一工作是一个很实用的概念，可确保同一时刻只有一个具有特定名称的工作实例。与 ID 不同的是，唯一名称是人类可读的，由开发者指定，而不是由 WorkManager 自动生成。与[标记](https://developer.android.com/topic/libraries/architecture/workmanager/how-to/define-work#tag)不同，唯一名称仅与一个工作实例相关联。

唯一工作既可用于一次性工作，也可用于定期工作。您可以通过调用以下方法之一创建唯一工作序列，具体取决于您是调度重复工作还是一次性工作。

- [`WorkManager.enqueueUniqueWork()`](https://developer.android.com/reference/androidx/work/WorkManager#enqueueUniqueWork(java.lang.String, androidx.work.ExistingWorkPolicy, androidx.work.OneTimeWorkRequest))（用于一次性工作）
- [`WorkManager.enqueueUniquePeriodicWork()`](https://developer.android.com/reference/androidx/work/WorkManager#enqueueUniquePeriodicWork(java.lang.String, androidx.work.ExistingPeriodicWorkPolicy, androidx.work.PeriodicWorkRequest))（用于定期工作）

这两种方法都接受 3 个参数：

- uniqueWorkName - 用于唯一标识工作请求的 `String`。
- existingWorkPolicy - 此 `enum` 可告知 WorkManager：如果已有使用该名称且尚未完成的唯一工作链，应执行什么操作。如需了解详情，请参阅[冲突解决政策](https://developer.android.com/topic/libraries/architecture/workmanager/how-to/managing-work#conflict-resolution)。
- work - 要调度的 `WorkRequest`。

```kotlin
val sendLogsWorkRequest =
       PeriodicWorkRequestBuilder<SendLogsWorker>(24, TimeUnit.HOURS)
           .setConstraints(Constraints.Builder()
               .setRequiresCharging(true)
               .build()
            )
           .build()
WorkManager.getInstance(this).enqueueUniquePeriodicWork(
           "sendLogs",
           ExistingPeriodicWorkPolicy.KEEP,
           sendLogsWorkRequest
)
```

上述代码中， 通过唯一工作，如果添加的工作已经处于队列中的情况运行时，默认情况下， 系统会保留现有的工作，并且不会添加新的工作；

##### 冲突解决政策

调度唯一工作时，您必须告知 WorkManager 在发生冲突时要执行的操作。您可以通过在将工作加入队列时传递一个枚举来实现此目的。

对于一次性工作，您需要提供一个 [`ExistingWorkPolicy`](https://developer.android.com/reference/androidx/work/ExistingWorkPolicy)，它支持用于处理冲突的 4 个选项。

- [`REPLACE`](https://developer.android.com/reference/androidx/work/ExistingWorkPolicy#REPLACE)：用新工作替换现有工作。此选项将取消现有工作。
- [`KEEP`](https://developer.android.com/reference/androidx/work/ExistingWorkPolicy#KEEP)：保留现有工作，并忽略新工作。
- [`APPEND`](https://developer.android.com/reference/androidx/work/ExistingWorkPolicy#APPEND)：将新工作附加到现有工作的末尾。此政策将导致您的新工作[链接](https://developer.android.com/topic/libraries/architecture/workmanager/how-to/chain-work)到现有工作，在现有工作完成后运行。

现有工作将成为新工作的先决条件。如果现有工作变为 `CANCELLED` 或 `FAILED` 状态，新工作也会变为 `CANCELLED` 或 `FAILED`。如果您希望无论现有工作的状态如何都运行新工作，请改用 `APPEND_OR_REPLACE`。

- [`APPEND_OR_REPLACE`](https://developer.android.com/reference/androidx/work/ExistingWorkPolicy#APPEND) 函数类似于 `APPEND`，不过它并不依赖于**先决条件**工作状态。即使现有工作变为 `CANCELLED` 或 `FAILED` 状态，新工作仍会运行。

对于定期工作，您需要提供一个 [`ExistingPeriodicWorkPolicy`](https://developer.android.com/reference/androidx/work/ExistingPeriodicWorkPolicy)，它支持 `REPLACE` 和 `KEEP` 这两个选项。这些选项的功能与其对应的 ExistingWorkPolicy 功能相同。

##### 观察您的工作

在将工作加入队列后，您可以随时按其 `name`、`id` 或与其关联的 `tag` 在 WorkManager 中进行查询，以检查其状态。

```kotlin
// by id
workManager.getWorkInfoById(syncWorker.id) // ListenableFuture<WorkInfo>

// by name
workManager.getWorkInfosForUniqueWork("sync") // ListenableFuture<List<WorkInfo>>

// by tag
workManager.getWorkInfosByTag("syncTag") // ListenableFuture<List<WorkInfo>>
```

该查询会返回 [`WorkInfo`](https://developer.android.com/reference/androidx/work/WorkInfo) 对象的 [`ListenableFuture`](https://guava.dev/releases/23.1-android/api/docs/com/google/common/util/concurrent/ListenableFuture.html)，该值包含工作的 [`id`](https://developer.android.com/reference/androidx/work/WorkInfo#getId())、其标记、其当前的 [`State`](https://developer.android.com/reference/androidx/work/WorkInfo.State) 以及通过 [`Result.success(outputData)`](https://developer.android.com/reference/androidx/work/ListenableWorker.Result#success(androidx.work.Data)) 设置的任何输出数据。

利用每个方法的 [`LiveData`](https://developer.android.com/topic/libraries/architecture/livedata) 变种，您可以通过注册监听器来观察 `WorkInfo` 的变化。例如，如果您想要在某项工作成功完成后向用户显示消息，您可以进行如下设置：

```kotlin
workManager.getWorkInfoByIdLiveData(syncWorker.id)
               .observe(viewLifecycleOwner) { workInfo ->
   if(workInfo?.state == WorkInfo.State.SUCCEEDED) {
       Snackbar.make(requireView(),
      R.string.work_completed, Snackbar.LENGTH_SHORT)
           .show()
   }
}
```

WorkManager 2.4.0 及更高版本支持使用 [`WorkQuery`](https://developer.android.com/reference/androidx/work/WorkQuery) 对象对已加入队列的作业进行复杂查询。WorkQuery 支持按工作的标记、状态和唯一工作名称的组合进行查询。

```kotlin
val workQuery = WorkQuery.Builder
       .fromTags(listOf("syncTag"))
       .addStates(listOf(WorkInfo.State.FAILED, WorkInfo.State.CANCELLED))
       .addUniqueWorkNames(listOf("preProcess", "sync")
    )
   .build()

val workInfos: ListenableFuture<List<WorkInfo>> = workManager.getWorkInfos(workQuery)
```

`WorkQuery` 中的每个组件（标记、状态或名称）与其他组件都是 `AND` 逻辑关系。组件中的每个值都是 `OR` 逻辑关系。

`WorkQuery` 也适用于等效的 LiveData 方法 [`getWorkInfosLiveData()`](https://developer.android.com/reference/androidx/work/WorkManager#getWorkInfosLiveData(androidx.work.WorkQuery))。

#####  取消和停止工作

如果您不再需要运行先前加入队列的工作，则可以要求将其取消。您可以按工作的 `name`、`id` 或与其关联的 `tag` 取消工作。

```kotlin
// by id
workManager.cancelWorkById(syncWorker.id)

// by name
workManager.cancelUniqueWork("sync")

// by tag
workManager.cancelAllWorkByTag("syncTag")
```

WorkManager 会在后台检查工作的 [`State`](https://developer.android.com/reference/androidx/work/WorkInfo.State)。如果工作已经[完成](https://developer.android.com/reference/androidx/work/WorkInfo.State#isFinished())，系统不会执行任何操作。否则，工作的状态会更改为 [`CANCELLED`](https://developer.android.com/reference/androidx/work/WorkInfo.State#CANCELLED)，之后就不会运行这个工作。任何[依赖于此工作](https://developer.android.com/topic/libraries/architecture/workmanager/how-to/chain-work)的 [`WorkRequest`](https://developer.android.com/reference/androidx/work/WorkRequest) 作业也将变为 `CANCELLED`。

目前，[`RUNNING`](https://developer.android.com/reference/androidx/work/WorkInfo.State#RUNNING) 可收到对 [`ListenableWorker.onStopped()`](https://developer.android.com/reference/androidx/work/ListenableWorker#onStopped()) 的调用。如需执行任何清理操作，请替换此方法。如需了解详情，请参阅[停止正在运行的工作器](https://developer.android.com/topic/libraries/architecture/workmanager/how-to/managing-work#stop-worker)。

##### 停止正在运行的工作器

正在运行的 `Worker` 可能会由于以下几种原因而停止运行：

- 您明确要求取消它（例如，通过调用 `WorkManager.cancelWorkById(UUID)` 取消）。
- 如果是[唯一工作](https://developer.android.com/topic/libraries/architecture/workmanager/how-to/managing-work#unique-work)，您明确地将 [`ExistingWorkPolicy`](https://developer.android.com/reference/androidx/work/ExistingWorkPolicy) 为 [`REPLACE`](https://developer.android.com/reference/androidx/work/ExistingWorkPolicy#REPLACE) 的新 `WorkRequest` 加入到了队列中。旧的 `WorkRequest` 会立即被视为已取消。
- 您的工作约束条件已不再满足。
- 系统出于某种原因指示您的应用停止工作。如果超过 10 分钟的执行期限，可能会发生这种情况。该工作会调度为在稍后重试。

在这些情况下，您的工作器会停止。

您应该合作地取消正在进行的任何工作，并释放您的工作器保留的所有资源。例如，此时应该关闭所打开的数据库和文件句柄。有两种机制可让您了解工作器何时停止。

- onStopped() 回调：
  		在您的工作器停止后，WorkManager 会立即调用 [`ListenableWorker.onStopped()`](https://developer.android.com/reference/androidx/work/ListenableWorker#onStopped())。替换此方法可关闭您可能保留的所有资源。
- isStopped() 属性：
  	您可以调用 [`ListenableWorker.isStopped()`](https://developer.android.com/reference/androidx/work/ListenableWorker#isStopped()) 方法以检查工作器是否已停止。如果您在工作器执行长时间运行的操作或重复操作，您应经常检查此属性，并尽快将其用作停止工作的信号。

**注意**：WorkManager 会忽略已收到 onStop 信号的工作器所设置的 [`Result`](https://developer.android.com/reference/androidx/work/ListenableWorker.Result)，因为工作器已被视为停止。

#### 观察工作器的中间进度

WorkManager `2.3.0-alpha01` 为设置和观察工作器的中间进度添加了一流的支持。如果应用在前台运行时，工作器保持运行状态，那么也可以使用返回 [`WorkInfo`](https://developer.android.com/reference/androidx/work/WorkInfo) 的 [`LiveData`](https://developer.android.com/reference/androidx/lifecycle/LiveData) 的 API 向用户显示此信息。

[`ListenableWorker`](https://developer.android.com/reference/androidx/work/ListenableWorker) 现在支持 [`setProgressAsync()`](https://developer.android.com/reference/androidx/work/ListenableWorker#setProgressAsync(androidx.work.Data)) API，此类 API 允许保留中间进度。借助这些 API，开发者能够设置可通过界面观察到的中间进度。进度由 [`Data`](https://developer.android.com/reference/androidx/work/Data) 类型表示，这是一个可序列化的属性容器（类似于 [`input` 和 `output`](https://developer.android.com/topic/libraries/architecture/workmanager/advanced#params)，并且受到相同的限制）。

只有在 `ListenableWorker` 运行时才能观察到和更新进度信息。如果尝试在 `ListenableWorker` 完成执行后在其中设置进度，则将会被忽略。您还可以使用 [`getWorkInfoBy…()` 或 `getWorkInfoBy…LiveData()`](https://developer.android.com/reference/androidx/work/WorkManager#getWorkInfoById(java.util.UUID)) 方法来观察进度信息。这两个方法会返回 [`WorkInfo`](https://developer.android.com/reference/androidx/work/WorkInfo) 的实例，后者有一个返回 `Data` 的新 [`getProgress()`](https://developer.android.com/reference/androidx/work/WorkInfo#getProgress()) 方法。

##### 更新进度

对于使用 [`ListenableWorker`](https://developer.android.com/reference/androidx/work/ListenableWorker) 或 [`Worker`](https://developer.android.com/reference/androidx/work/Worker) 的 Java 开发者，[`setProgressAsync()`](https://developer.android.com/reference/androidx/work/ListenableWorker#setProgressAsync(androidx.work.Data)) API 会返回 `ListenableFuture`；更新进度是异步过程，因为更新过程涉及将进度信息存储在数据库中。在 Kotlin 中，您可以使用 [`CoroutineWorker`](https://developer.android.com/reference/kotlin/androidx/work/CoroutineWorker) 对象的 [`setProgress()`](https://developer.android.com/reference/kotlin/androidx/work/CoroutineWorker#setprogress) 扩展函数来更新进度信息。

Java:

```java
import android.content.Context;
import androidx.annotation.NonNull;
import androidx.work.Data;
import androidx.work.Worker;
import androidx.work.WorkerParameters;

public class ProgressWorker extends Worker {

    private static final String PROGRESS = "PROGRESS";
    private static final long DELAY = 1000L;

    public ProgressWorker(
        @NonNull Context context,
        @NonNull WorkerParameters parameters) {
        super(context, parameters);
        // Set initial progress to 0
        setProgressAsync(new Data.Builder().putInt(PROGRESS, 0).build());
    }

    @NonNull
    @Override
    public Result doWork() {
        try {
            // Doing work.
            Thread.sleep(DELAY);
        } catch (InterruptedException exception) {
            // ... handle exception
        }
        // Set progress to 100 after you are done doing your work.
        setProgressAsync(new Data.Builder().putInt(PROGRESS, 100).build());
        return Result.success();
    }
}
```

Kotlin:

```kotlin
import android.content.Context
import androidx.work.CoroutineWorker
import androidx.work.Data
import androidx.work.WorkerParameters
import kotlinx.coroutines.delay

class ProgressWorker(context: Context, parameters: WorkerParameters) :
    CoroutineWorker(context, parameters) {

    companion object {
        const val Progress = "Progress"
        private const val delayDuration = 1L
    }

    override suspend fun doWork(): Result {
        val firstUpdate = workDataOf(Progress to 0)
        val lastUpdate = workDataOf(Progress to 100)
        setProgress(firstUpdate)
        delay(delayDuration)
        setProgress(lastUpdate)
        return Result.success()
    }
}
```

##### 观察进度

观察进度信息也很简单。您可以使用 [`getWorkInfoBy…()` 或 `getWorkInfoBy…LiveData()`](https://developer.android.com/reference/androidx/work/WorkManager#getWorkInfoById(java.util.UUID)) 方法，并引用 [`WorkInfo`](https://developer.android.com/reference/androidx/work/WorkInfo)。

```kotlin
WorkManager.getInstance(applicationContext)
    // requestId is the WorkRequest id
    .getWorkInfoByIdLiveData(requestId)
    .observe(observer, Observer { workInfo: WorkInfo? ->
            if (workInfo != null) {
                val progress = workInfo.progress
                val value = progress.getInt(Progress, 0)
                // Do something with progress information
            }
    })
```

#### 链接工作

您可以使用 WorkManager 创建工作链并将其加入队列。工作链用于指定多个依存任务并定义这些任务的运行顺序。当您需要以特定顺序运行多个任务时，此功能尤其有用。

1. 创建工作链： 使用 [`WorkManager.beginWith(OneTimeWorkRequest)`](https://developer.android.com/reference/androidx/work/WorkManager#beginWith(androidx.work.OneTimeWorkRequest)) 或 [`WorkManager.beginWith(List)`](https://developer.android.com/reference/androidx/work/WorkManager#beginWith(java.util.List))，这会返回 [`WorkContinuation`](https://developer.android.com/reference/androidx/work/WorkContinuation) 实例；
2. 使用 `WorkContinuation` 通过 [`then(OneTimeWorkRequest)`](https://developer.android.com/reference/androidx/work/WorkContinuation#then(androidx.work.OneTimeWorkRequest)) 或 [`then(List)`](https://developer.android.com/reference/androidx/work/WorkContinuation#then(java.util.List)) 添加 `OneTimeWorkRequest` 依赖实例。
   每次调用 `WorkContinuation.then(...)` 都会返回一个新的 `WorkContinuation` 实例。如果添加了 `OneTimeWorkRequest` 实例的 `List`，这些请求可能会**并行运行**。
3. 最后，您可以使用 [`WorkContinuation.enqueue()`](https://developer.android.com/reference/androidx/work/WorkContinuation#enqueue()) 方法对 `WorkContinuation` 工作链执行 `enqueue()` 操作；

```kotlin
WorkManager.getInstance(myContext)
   // Candidates to run in parallel
   .beginWith(listOf(plantName1, plantName2, plantName3))
   // Dependent work (only runs after all previous work in chain)
   .then(cache)
   .then(upload)
   // Call enqueue to kick things off
   .enqueue()
```

在本例中，有 3 个不同的工作器作业配置为运行（可能并行运行）。然后这些工作器的结果将联接起来，并传递给正在缓存的工作器作业。最后，该作业的输出将传递到上传工作器，由上传工作器将结果上传到远程服务器。

##### 输入合并器

当您链接 `OneTimeWorkRequest` 实例时，父级工作请求的输出将作为子级的输入传入。因此，在上面的示例中，`plantName1`、`plantName2` 和 `plantName3` 的输出将作为 `cache` 请求的输入传入。

为了管理来自多个父级工作请求的输入，WorkManager 使用 [`InputMerger`](https://developer.android.com/reference/androidx/work/InputMerger)。

WorkManager 提供两种不同类型的 `InputMerger`：

- [`OverwritingInputMerger`](https://developer.android.com/reference/androidx/work/OverwritingInputMerger) 会尝试将所有输入中的所有键添加到输出中。如果发生冲突，它会覆盖先前设置的键。
- [`ArrayCreatingInputMerger`](https://developer.android.com/reference/androidx/work/ArrayCreatingInputMerger) 会尝试合并输入，并在必要时创建数组。

如果您有更具体的用例，则可以创建 `InputMerger` 的子类来编写自己的用例。

###### OverwritingInputMerger

`OverwritingInputMerger` 是默认的合并方法。如果合并过程中存在键冲突，键的最新值将覆盖生成的输出数据中的所有先前版本。

例如，如果每种植物的输入都有一个与其各自变量名称（`"plantName1"`、`"plantName2"` 和 `"plantName3"`）匹配的键，传递给 `cache` 工作器的数据将具有三个键值对。

![此图显示了三个作业将不同的输出传递给链中的下一个作业。由于这三个输出都具有不同的键，因此下一个作业将收到三个键值对。](https://developer.android.com/images/topic/libraries/architecture/workmanager/how-to/chaining-overwriting-merger-example.png)

如果存在冲突，那么最后一个工作器将在争用中“取胜”，其值将传递给 `cache`。

![此图显示了三个作业将输出传递给链中的下一个作业。在本例中，其中有两个作业使用同一个键生成输出。因此，下一个作业将收到两个键值对，其中一个存在冲突的输出会被丢弃。](https://developer.android.com/images/topic/libraries/architecture/workmanager/how-to/chaining-overwriting-merger-conflict.png)

由于工作请求是并行运行的，因此无法保证其运行顺序。在上面的示例中，`plantName1` 可以保留值 `"tulip"` 或 `"elm"`，具体取决于最后写入的是哪个值。如果有可能存在键冲突，并且您需要在合并器中保留所有输出数据，那么 `ArrayCreatingInputMerger` 可能是更好的选择。

###### ArrayCreatingInputMerger

对于上面的示例，假设我们要保留所有植物名称工作器的输出，则应使用 `ArrayCreatingInputMerger`。

```kotlin

val cache: OneTimeWorkRequest = OneTimeWorkRequestBuilder<PlantWorker>()
   .setInputMerger(ArrayCreatingInputMerger::class)
   .setConstraints(constraints)
   .build()
```

`ArrayCreatingInputMerger` 将每个键与数组配对。如果每个键都是唯一的，您会得到一系列一元数组。

![此图显示了三个作业将不同的输出传递给链中的下一个作业。有三个数组传递到下一个作业，每个输出键对应一个数组。每个数组都有一个成员。](https://developer.android.com/images/topic/libraries/architecture/workmanager/how-to/chaining-array-merger-example.png)

如果存在任何键冲突，那么所有对应的值会分组到一个数组中。

![此图显示了三个作业将输出传递给链中的下一个作业。在本例中，其中有两个作业使用同一个键生成输出。有两个数组传递到下一个作业，每个键对应一个数组。其中一个数组有两个成员，因为有两个输出具有该键。](https://developer.android.com/images/topic/libraries/architecture/workmanager/how-to/chaining-array-merger-conflict.png)

##### 链接和工作状态

只要工作成功完成（即，返回 `Result.success()`），`OneTimeWorkRequest` 链便会按顺序执行。运行时，工作请求可能会失败或被取消，这会对依存工作请求产生下游影响。

当第一个 `OneTimeWorkRequest` 被加入工作请求链队列时，所有后续工作请求会被屏蔽，直到第一个工作请求的工作完成为止。

![此图显示一个作业链。第一个作业已加入队列；所有后续的作业都会被屏蔽，直到第一个作业完成为止。](https://developer.android.com/images/topic/libraries/architecture/workmanager/how-to/chaining-enqueued-all-blocked.png)

在加入队列且满足所有工作约束后，第一个工作请求开始运行。如果工作在根 `OneTimeWorkRequest` 或 `List` 中成功完成（即返回 `Result.success()`），系统会将下一组依存工作请求加入队列。

![此图显示一个作业链。第一个作业已成功完成，其两个直接后继作业已加入队列。剩余作业会被屏蔽，直到其之前的作业完成为止。](https://developer.android.com/images/topic/libraries/architecture/workmanager/how-to/chaining-enqueued-in-progress.png)

只要每个工作请求都成功完成，工作请求链中的剩余工作请求就会遵循相同的运行模式，直到链中的所有工作都完成为止。这是最简单的用例，通常也是首选用例，但处理错误状态同样重要。

如果在工作器处理工作请求时出现错误，您可以根据[您定义的退避政策](https://developer.android.com/topic/libraries/architecture/workmanager/how-to/define-work#retry_and_backoff_policy)来重试该请求。重试请求链中的某个请求意味着，系统将使用提供给该请求的输入数据仅对该请求进行重试。并行运行的所有其他作业均不会受到影响。

![此图显示一个作业链。其中一个作业失败，但已定义退避政策。该作业将在一段适当的时间过后重新运行，链中跟随其后的作业会被屏蔽，直到该作业成功运行为止。](https://developer.android.com/images/topic/libraries/architecture/workmanager/how-to/chaining-enqueued-retry.png)

如需详细了解如何定义自定义重试策略，请参阅[重试和退避政策](https://developer.android.com/topic/libraries/architecture/workmanager/how-to/define-work#retry_and_backoff_policy)。

如果该重试政策未定义或已用尽，或者您以其他方式已达到 `OneTimeWorkRequest` 返回 `Result.failure()` 的某种状态，该工作请求和所有依存工作请求都会被标记为 `FAILED.`

![此图显示一个作业链。一个作业失败，无法重试。因此，链中在该作业后面的所有作业也都将失败。](https://developer.android.com/images/topic/libraries/architecture/workmanager/how-to/chaining-enqueued-failed.png)

`OneTimeWorkRequest` 被取消时遵循相同的逻辑。任何依存工作请求也会被标记为 `CANCELLED`，并且无法执行其工作。

![此图显示一个作业链。一个作业已被取消。因此，链中在该作业后面的所有作业也都会被取消。](https://developer.android.com/images/topic/libraries/architecture/workmanager/how-to/chaining-enqueued-cancelled.png)

请注意，如果要向已失败或已取消工作请求的链附加更多工作请求，新附加的工作请求也会分别标记为 `FAILED` 或 `CANCELLED`。如果您想扩展现有链的工作，请参阅 [ExistingWorkPolicy](https://developer.android.com/reference/androidx/work/ExistingWorkPolicy) 中的 `APPEND_OR_REPLACE`。

创建工作请求链时，依存工作请求应定义重试政策，以确保始终及时完成工作。失败的工作请求可能会导致链不完整和/或出现意外状态。

### 5.0 测试/调试 Worker

WorkManager 提供了用于测试 [`Worker`](https://developer.android.com/reference/kotlin/androidx/work/Worker)、[`ListenableWorker`](https://developer.android.com/reference/androidx/work/ListenableWorker) 和 `ListenableWorker` 变体（[`CoroutineWorker`](https://developer.android.com/reference/kotlin/androidx/work/CoroutineWorker) 和 [`RxWorker`](https://developer.android.com/reference/androidx/work/RxWorker)）的 API。

> 在版本 2.1.0 以前，如需测试工作器，您需要使用 [`WorkManagerTestInitHelper`](https://developer.android.com/reference/androidx/work/testing/WorkManagerTestInitHelper) 来初始化 WorkManager

#### 测试工作器

需要测试的类如下：

```kotlin
class SleepWorker(context: Context, parameters: WorkerParameters) :
    Worker(context, parameters) {

    override fun doWork(): Result {
        // Sleep on a background thread.
        Thread.sleep(1000)
        return Result.success()
    }
}
```

如需测试这个 `Worker`，您可以使用 [`TestWorkerBuilder`](https://developer.android.com/reference/androidx/work/testing/TestWorkerBuilder)。此构建器有助于构建可用于测试业务逻辑的 `Worker` 实例。

```kotlin
// Kotlin code uses the TestWorkerBuilder extension to build
// the Worker
@RunWith(AndroidJUnit4::class)
class SleepWorkerTest {
    private lateinit var context: Context
    private lateinit var executor: Executor

    @Before
    fun setUp() {
        context = ApplicationProvider.getApplicationContext()
        executor = Executors.newSingleThreadExecutor()
    }

    @Test
    fun testSleepWorker() {
        val worker = TestWorkerBuilder<SleepWorker>(
            context = context,
            executor = executor
        ).build()

        val result = worker.doWork()
        assertThat(result, `is`(Result.success()))
    }
}

```

`TestWorkerBuilder` 也可用于设置标记（例如 `inputData` 或 `runAttemptCount`），以便您单独验证工作器状态。例如，`SleepWorker` 将休眠时长当作输入数据，而不是工作器中定义的常量数据：

```kotlin
	@Test
    fun testSleepWorker() {
        val worker = TestWorkerBuilder<SleepWorker>(
            context = context,
            executor = executor,
            inputData = workDataOf("SLEEP_DURATION" to 1000L)
        ).build()

        val result = worker.doWork()
        assertThat(result, `is`(Result.success()))
    }
```

了解 `TestWorkerBuilder` API，请参阅 [`TestListenableWorkerBuilder`](https://developer.android.com/reference/androidx/work/testing/TestListenableWorkerBuilder)（`TestWorkerBuilder` 的父类）的参考页面。

#### 测试 ListenableWorker 及其变体

如需测试 [`ListenableWorker`](https://developer.android.com/reference/androidx/work/ListenableWorker) 或其变体（[`CoroutineWorker`](https://developer.android.com/reference/kotlin/androidx/work/CoroutineWorker) 和 [`RxWorker`](https://developer.android.com/reference/androidx/work/RxWorker)），请使用 [`TestListenableWorkerBuilder`](https://developer.android.com/reference/androidx/work/testing/TestListenableWorkerBuilder)。`TestWorkerBuilder` 和 [`TestListenableWorkerBuilder`](https://developer.android.com/reference/androidx/work/testing/TestListenableWorkerBuilder) 的主要区别在于，`TestWorkerBuilder` 允许指定用来运行 `Worker` 的后台 `Executor`，而 `TestListenableWorkerBuilder` 依赖 `ListenableWorker` 实现的线程逻辑。

```kotlin
class SleepWorker(context: Context, parameters: WorkerParameters) :
    CoroutineWorker(context, parameters) {
    override suspend fun doWork(): Result {
        delay(1000L) // milliseconds
        return Result.success()
    }
}
```

为了测试 `SleepWorker`，我们首先使用 `TestListenableWorkerBuilder` 创建一个工作器实例，然后在协程中调用其 `doWork` 函数。

```kotlin
@RunWith(AndroidJUnit4::class)
class SleepWorkerTest {
    private lateinit var context: Context

    @Before
    fun setUp() {
        context = ApplicationProvider.getApplicationContext()
    }

    @Test
    fun testSleepWorker() {
        val worker = TestListenableWorkerBuilder<SleepWorker>(context).build()
        runBlocking {
            val result = worker.doWork()
            assertThat(result, `is`(Result.success()))
        }
    }
}
```

`runBlocking` 可用作测试的协程构建器，使任何异步执行的代码都转为并行运行。

#### 使用 WrokManager 进行集成测试

##### 构造测试

[WorkManager](https://developer.android.com/topic/libraries/architecture/workmanager?hl=zh-cn) 提供了一个 `work-testing` 工件，可以协助进行工作器测试。

```groovy
dependencies {
    def work_version = "2.5.0"

    ...

    // optional - Test helpers
    androidTestImplementation "androidx.work:work-testing:$work_version"
}
```

`work-testing` 为测试模式提供了一种特殊的 WorkManager 实现，它通过使用 [`WorkManagerTestInitHelper`](https://developer.android.com/reference/androidx/work/testing/WorkManagerTestInitHelper?hl=zh-cn) 来初始化。

`work-testing` 工件还提供了 [`SynchronousExecutor`](https://developer.android.com/reference/androidx/work/testing/SynchronousExecutor?hl=zh-cn)，让您可以更轻松地以同步方式编写测试，而无需处理多个线程、锁定或锁存。

```kotlin
//定义Worker
class EchoWorker(context: Context, parameters: WorkerParameters)
   : Worker(context, parameters) {
   override fun doWork(): Result {
       return when(inputData.size()) {
           0 -> Result.failure()
           else -> Result.success(inputData)
       }
   }
}

//初始化测试模式中的WorkManager 
@RunWith(AndroidJUnit4::class)
class BasicInstrumentationTest {
    @Before
    fun setup() {
        val context = InstrumentationRegistry.getTargetContext()
        val config = Configuration.Builder()
            .setMinimumLoggingLevel(Log.DEBUG)
            .setExecutor(SynchronousExecutor())
            .build()

        // Initialize WorkManager for instrumentation tests.
        WorkManagerTestInitHelper.initializeTestWorkManager(context, config)
    }
}

@Test
@Throws(Exception::class)
fun testSimpleEchoWorker() {
    // Define input data
    val input = workDataOf(KEY_1 to 1, KEY_2 to 2)

    // Create request
    val request = OneTimeWorkRequestBuilder<EchoWorker>()
        .setInputData(input)
        .build()

    val workManager = WorkManager.getInstance(applicationContext)
    // Enqueue and wait for result. This also runs the Worker synchronously
    // because we are using a SynchronousExecutor.
    workManager.enqueue(request).result.get()
    // Get WorkInfo and outputData
    val workInfo = workManager.getWorkInfoById(request.id).get()
    val outputData = workInfo.outputData

    // Assert
    assertThat(workInfo.state, `is`(WorkInfo.State.SUCCEEDED))
    assertThat(outputData, `is`(input))
}

@Test
@Throws(Exception::class)
fun testEchoWorkerNoInput() {
   // Create request
   val request = OneTimeWorkRequestBuilder<EchoWorker>()
       .build()

   val workManager = WorkManager.getInstance(applicationContext)
   // Enqueue and wait for result. This also runs the Worker synchronously
   // because we are using a SynchronousExecutor.
   workManager.enqueue(request).result.get()
   // Get WorkInfo
   val workInfo = workManager.getWorkInfoById(request.id).get()

   // Assert
   assertThat(workInfo.state, `is`(WorkInfo.State.FAILED))
}
```

##### 模拟约束、延迟和定期工作

`WorkManagerTestInitHelper` 为您提供了 [`TestDriver`](https://developer.android.com/reference/androidx/work/testing/TestDriver?hl=zh-cn) 的一个实例，可用于模拟初始延迟、满足 `ListenableWorker` 实例约束的条件，以及 `PeriodicWorkRequest` 实例的间隔。

工作器可以具有初始延迟。如需测试具有 `initialDelay` 的 `EchoWorker`，而不必在测试中等待 `initialDelay`，您可以使用 `TestDriver` 通过 `setInitialDelayMet` 将工作请求的初始延迟标记为已满足条件。

```kotlin
@Test
@Throws(Exception::class)
fun testWithInitialDelay() {
    // Define input data
    val input = workDataOf(KEY_1 to 1, KEY_2 to 2)

    // Create request
    val request = OneTimeWorkRequestBuilder<EchoWorker>()
        .setInputData(input)
        .setInitialDelay(10, TimeUnit.SECONDS)
        .build()

    val workManager = WorkManager.getInstance(getApplicationContext())
    // Get the TestDriver
    val testDriver = WorkManagerTestInitHelper.getTestDriver()
    // Enqueue
    workManager.enqueue(request).result.get()
    // Tells the WorkManager test framework that initial delays are now met.
    testDriver.setInitialDelayMet(request.id)
    // Get WorkInfo and outputData
    val workInfo = workManager.getWorkInfoById(request.id).get()
    val outputData = workInfo.outputData

    // Assert
    assertThat(workInfo.state, `is`(WorkInfo.State.SUCCEEDED))
    assertThat(outputData, `is`(input))
}
```

`TestDriver` 也可用于利用 `setAllConstraintsMet` 将约束标记为已满足条件。以下示例展示了如何测试具有约束的 `Worker`。

```kotlin

@Test
@Throws(Exception::class)
fun testWithConstraints() {
    // Define input data
    val input = workDataOf(KEY_1 to 1, KEY_2 to 2)

    val constraints = Constraints.Builder()
        .setRequiredNetworkType(NetworkType.CONNECTED)
        .build()

    // Create request
    val request = OneTimeWorkRequestBuilder<EchoWorker>()
        .setInputData(input)
        .setConstraints(constraints)
        .build()

    val workManager = WorkManager.getInstance(myContext)
    val testDriver = WorkManagerTestInitHelper.getTestDriver()
    // Enqueue
    workManager.enqueue(request).result.get()
    // Tells the testing framework that all constraints are met.
    testDriver.setAllConstraintsMet(request.id)
    // Get WorkInfo and outputData
    val workInfo = workManager.getWorkInfoById(request.id).get()
    val outputData = workInfo.outputData

    // Assert
    assertThat(workInfo.state, `is`(WorkInfo.State.SUCCEEDED))
    assertThat(outputData, `is`(input))
}
```

`TestDriver` 也公开 `setPeriodDelayMet`，可用于指明某一时间间隔已经完成。以下是使用的 `setPeriodDelayMet` 的示例。

```kotlin
@Test
@Throws(Exception::class)
fun testPeriodicWork() {
    // Define input data
    val input = workDataOf(KEY_1 to 1, KEY_2 to 2)

    // Create request
    val request = PeriodicWorkRequestBuilder<EchoWorker>(15, MINUTES)
        .setInputData(input)
        .build()

    val workManager = WorkManager.getInstance(myContext)
    val testDriver = WorkManagerTestInitHelper.getTestDriver()
    // Enqueue and wait for result.
    workManager.enqueue(request).result.get()
    // Tells the testing framework the period delay is met
    testDriver.setPeriodDelayMet(request.id)
    // Get WorkInfo and outputData
    val workInfo = workManager.getWorkInfoById(request.id).get()

    // Assert
    assertThat(workInfo.state, `is`(WorkInfo.State.ENQUEUED))
}
```

#### 调试 WorkManager

##### 启用日志记录

如需确定工作器未正确运行的原因，查看详细的 WorkManager 日志很有帮助。如需启用日志记录功能，您需要使用[自定义初始化](https://developer.android.com/topic/libraries/architecture/workmanager/advanced/custom-configuration?hl=zh-cn)。首先，通过创建应用了清单合并规则 **`remove`** 的新 WorkManager 提供程序来停用 `AndroidManifest.xml` 中的默认 `WorkManagerInitializer`：

```xml
<provider
    android:name="androidx.work.impl.WorkManagerInitializer"
    android:authorities="${applicationId}.workmanager-init"
    tools:node="remove"/>
```

现在，默认 WorkManager 初始化程序已停用，您可以使用[按需初始化](https://developer.android.com/topic/libraries/architecture/workmanager/advanced/custom-configuration?hl=zh-cn#implement-configuration-provider)。为此，`android.app.Application` 类必须提供 `androidx.work.Configuration.Provider` 的实现。

```kotlin
class MyApplication() : Application(), Configuration.Provider {
    override fun getWorkManagerConfiguration() =
        Configuration.Builder()
            .setMinimumLoggingLevel(android.util.Log.DEBUG)
            .build()
}
```

在您定义自定义 WorkManager 配置后，WorkManager 会在您调用 [`WorkManager.getInstance(Context)`](https://developer.android.com/reference/androidx/work/WorkManager?hl=zh-cn#getInstance(android.content.Context)) 时进行初始化，而不是在应用启动时自动初始化。如需了解详情，包括对 2.1.0 之前的 WorkManager 版本提供的支持，请参阅[自定义 WorkManager 配置和初始化](https://developer.android.com/topic/libraries/architecture/workmanager/advanced/custom-configuration?hl=zh-cn)。

启用调试日志记录后，系统会开始显示更多包含日志标记前缀 `WM-` 的日志。

##### 使用 adb shell dumpsys jobscheduler

搭载 API 级别 23 或更高版本时，您可以运行命令 `adb shell dumpsys jobscheduler` 来查看已归因于您的软件包的作业列表。

类似于：

```
JOB #u0a172/4: 6412553 com.google.android.youtube/androidx.work.impl.background.systemjob.SystemJobService
  u0a172 tag=*job*/com.google.android.youtube/androidx.work.impl.background.systemjob.SystemJobService
  Source: uid=u0a172 user=0 pkg=com.google.android.youtube
  JobInfo:
    Service: com.google.android.youtube/androidx.work.impl.background.systemjob.SystemJobService
    Requires: charging=false batteryNotLow=false deviceIdle=false
    Extras: mParcelledData.dataSize=180
    Network type: NetworkRequest [ NONE id=0, [ Capabilities: NOT_METERED&INTERNET&NOT_RESTRICTED&TRUSTED&VALIDATED Uid: 10172] ]
    Minimum latency: +1h29m59s687ms
    Backoff: policy=1 initial=+30s0ms
    Has early constraint
  Required constraints: TIMING_DELAY CONNECTIVITY [0x90000000]
  Satisfied constraints: DEVICE_NOT_DOZING BACKGROUND_NOT_RESTRICTED WITHIN_QUOTA [0x3400000]
  Unsatisfied constraints: TIMING_DELAY CONNECTIVITY [0x90000000]
  Tracking: CONNECTIVITY TIME QUOTA
  Implicit constraints:
    readyNotDozing: true
    readyNotRestrictedInBg: true
  Standby bucket: RARE
  Base heartbeat: 0
  Enqueue time: -51m29s853ms
  Run time: earliest=+38m29s834ms, latest=none, original latest=none
  Last run heartbeat: 0
  Ready: false (job=false user=true !pending=true !active=true !backingup=true comp=true)
```

使用 WorkManager 时，负责管理工作器执行的组件为 `SystemJobService`（搭载 API 级别 23 或更高版本）。您应该查找归因于您的软件包名称和 `androidx.work.impl.background.systemjob.SystemJobService` 的作业实例。

对于每个作业，命令的输出都会列出**必需**、**满足**和**不满足**约束条件。您应该检查是否完全满足工作器的约束条件。

您可以利用它检查最近是否调用了 `SystemJobService`。输出还包括最近执行的作业的作业历史记录：

```
Job history:
     -1h35m26s440ms   START: #u0a107/9008 com.google.android.youtube/androidx.work.impl.background.systemjob.SystemJobService
     -1h35m26s362ms  STOP-P: #u0a107/9008 com.google.android.youtube/androidx.work.impl.background.systemjob.SystemJobService app called jobFinished
```

##### 从 WorkManager 2.4.0 及更高版本请求诊断信息

在应用的调试 build 中，您可以使用以下命令从 WorkManager 2.4.0 及更高版本请求诊断信息：

```shell
adb shell am broadcast -a "androidx.work.diagnostics.REQUEST_DIAGNOSTICS" -p "<your_app_package_name>"
```

这提供了以下方面的信息：

- 在过去 24 小时内完成的工作请求。
- 目前正在运行的工作请求。
- 预定运行的工作请求。

### 6.0 高级概念

#### 自定义 WorkManager 配置和初始化

默认情况下，当您的应用启动时，WorkManager 使用适合大多数应用的合理选项自动进行配置。如果您需要进一步控制 WorkManager 管理和调度工作的方式，可以通过自行初始化 WorkManager 来自定义 WorkManager 配置。

为 WorkManager 提供自定义初始化的最灵活方式是使用 WorkManager 2.1.0 及更高版本中提供的[按需初始化](https://developer.android.com/topic/libraries/architecture/workmanager/advanced/custom-configuration?hl=zh-cn#on-demand)。

##### 按需初始化

通过按需初始化，您可以仅在需要 WorkManager 时创建该组件，而不必每次应用启动时都创建。这样做可将 WorkManager 从关键启动路径中移出，从而提高应用启动性能。如需使用按需初始化，请执行以下操作：

- 移除默认初始化程序：
  如需提供自己的配置，必须先移除默认初始化程序。为此，请使用合并规则 `tools:node="remove"` 更新 [`AndroidManifest.xml`](https://developer.android.com/guide/topics/manifest/manifest-intro?hl=zh-cn)。
  从 WorkManager 2.6 开始，[应用启动](https://developer.android.com/topic/libraries/app-startup?hl=zh-cn)功能便已在 WorkManager 内部使用。如需提供自定义初始化程序，您需要移除 `androidx.startup` 节点。


  如果您不在应用中使用应用启动功能，则可以将其彻底移除：

  ```groovy
   <!-- If you want to disable android.startup completely. -->
   <provider
      android:name="androidx.startup.InitializationProvider"
      android:authorities="${applicationId}.androidx-startup"
      tools:node="remove">
   </provider>
  ```

  否则，仅移除 `WorkManagerInitializer` 节点即可：

  ```groovy
   <provider
      android:name="androidx.startup.InitializationProvider"
      android:authorities="${applicationId}.androidx-startup"
      android:exported="false"
      tools:node="merge">
      <!-- If you are using androidx.startup to initialize other components -->
      <meta-data
          android:name="androidx.work.WorkManagerInitializer"
          android:value="androidx.startup"
          tools:node="remove" />
   </provider>
  ```

  如果您使用的 WorkManager 是 2.6 之前的版本，请改为移除 `workmanager-init`：

  ```groovy
  <provider
      android:name="androidx.work.impl.WorkManagerInitializer"
      android:authorities="${applicationId}.workmanager-init"
      tools:node="remove" />
  ```

- 实现 Configuration.Provider
  让您的 `Application` 类实现 [`Configuration.Provider`](https://developer.android.com/reference/androidx/work/Configuration.Provider?hl=zh-cn) 接口，并提供您自己的 [`Configuration.Provider.getWorkManagerConfiguration`()](https://developer.android.com/reference/androidx/work/Configuration.Provider?hl=zh-cn#getWorkManagerConfiguration()) 实现。当您需要使用 WorkManager 时，请务必调用方法 [`WorkManager.getInstance(Context)`](https://developer.android.com/reference/androidx/work/WorkManager?hl=zh-cn#getInstance(android.content.Context))。WorkManager 会调用应用的自定义 `getWorkManagerConfiguration()` 方法来发现其 `Configuration`。（您无需自行调用 `WorkManager.initialize()`。）

  > 如果您在初始化 WorkManager 之前调用已弃用的无参数 `WorkManager.getInstance()` 方法，该方法将抛出异常。即使您不自定义 WorkManager，您也应始终使用 `WorkManager.getInstance(Context)` 方法。

  自定义 `getWorkManagerConfiguration()` 实现:

  ```kotlin
  //您必须移除默认的初始化程序，自定义的 getWorkManagerConfiguration() 实现才能生效。
  class MyApplication() : Application(), Configuration.Provider {
       override fun getWorkManagerConfiguration() =
             Configuration.Builder()
                  .setMinimumLoggingLevel(android.util.Log.INFO)
                  .build()
  }
  ```

  其它: [WorkManager 2.1.0 之前版本的自定义初始化](https://developer.android.com/topic/libraries/architecture/workmanager/advanced/custom-configuration?hl=zh-cn#pre-2.1.0)

#### WorkManager的线程处理

为了更高级的使用，我们还需要了解WorkManager 中的线程处理和并发机制。

WorkManager 提供了四种不同类型的工作基元：

- [`Worker`](https://developer.android.com/reference/androidx/work/Worker?hl=zh-cn) 是最简单的实现，我们已在前面几节进行了介绍。WorkManager 会在后台线程中自动运行该基元（您可以将它替换掉）。
- [`CoroutineWorker`](https://developer.android.com/reference/kotlin/androidx/work/CoroutineWorker?hl=zh-cn) 是为 Kotlin 用户建议的实现。`CoroutineWorker` 实例公开了后台工作的一个挂起函数。默认情况下，这些实例运行默认的 `Dispatcher`，但您可以进行自定义。
- [`RxWorker`](https://developer.android.com/reference/androidx/work/RxWorker?hl=zh-cn) 是为 RxJava 用户建议的实现。如果您有很多现有异步代码是用 RxJava 建模的，则应使用 RxWorker。与所有 RxJava 概念一样，您可以自由选择所需的线程处理策略。
- [`ListenableWorker`](https://developer.android.com/reference/androidx/work/ListenableWorker?hl=zh-cn) 是 `Worker`、`CoroutineWorker` 和 `RxWorker` 的基类。这个类专为需要与基于回调的异步 API（例如 `FusedLocationProviderClient`）进行交互并且不使用 RxJava 的 Java 开发者而设计。

##### Worker的线程处理

当您使用 [`Worker`](https://developer.android.com/reference/androidx/work/Worker?hl=zh-cn) 时，WorkManager 会自动在后台线程中调用 [`Worker.doWork()`](https://developer.android.com/reference/androidx/work/Worker?hl=zh-cn#doWork())。该后台线程来自于 WorkManager 的 [`Configuration`](https://developer.android.com/reference/androidx/work/Configuration?hl=zh-cn) 中指定的 `Executor`。默认情况下，WorkManager 会为您设置 `Executor`，但您也可以自己进行自定义。例如，您可以在应用中共享现有的后台 Executor，也可以创建单线程 `Executor` 以确保所有后台工作都按顺序执行，甚至可以指定一个自定义 `Executor`。如需自定义 `Executor`，请确保手动初始化 WorkManager。

在手动配置 WorkManager 时，您可以按以下方式指定 `Executor`：

```kotlin
WorkManager.initialize(
    context,
    Configuration.Builder()
         // Uses a fixed thread pool of size 8 threads.
        .setExecutor(Executors.newFixedThreadPool(8))
        .build())
```

[`Worker.doWork()`](https://developer.android.com/reference/androidx/work/Worker?hl=zh-cn#doWork()) 是同步调用 - 您应以阻塞方式完成整个后台工作，并在方法退出时完成工作。如果您在 `doWork()` 中调用异步 API 并返回 [`Result`](https://developer.android.com/reference/androidx/work/ListenableWorker.Result?hl=zh-cn)，那么回调可能无法正常运行。

如果当前正在运行的 `Worker` [因任何原因而停止](https://developer.android.com/topic/libraries/architecture/workmanager/how-to/managing-work?hl=zh-cn#cancelling)，它就会收到对 [`Worker.onStopped()`](https://developer.android.com/reference/androidx/work/ListenableWorker?hl=zh-cn#onStopped()) 的调用。在必要的情况下，只需替换此方法或调用 [`Worker.isStopped()`](https://developer.android.com/reference/androidx/work/ListenableWorker?hl=zh-cn#isStopped())，即可对代码进行检查点处理并释放资源。当下面示例中的 `Worker` 被停止时，内容的下载可能才完成了一半；但即使该工作器被停止，下载也会继续。

```kotlin
class DownloadWorker(context: Context, params: WorkerParameters) : Worker(context, params) {

    override fun doWork(): ListenableWorker.Result {
        repeat(100) {
            try {
                downloadSynchronously("https://www.google.com")
            } catch (e: IOException) {
                return ListenableWorker.Result.failure()
            }
        }

        return ListenableWorker.Result.success()
    }
}
```

如需优化此行为，您可以执行以下操作：

```kotlin
class DownloadWorker(context: Context, params: WorkerParameters) : Worker(context, params) {

    override fun doWork(): ListenableWorker.Result {
        repeat(100) {
            if (isStopped) {
                break
            }

            try {
                downloadSynchronously("https://www.google.com")
            } catch (e: IOException) {
                return ListenableWorker.Result.failure()
            }

        }

        return ListenableWorker.Result.success()
    }
}
```

`Worker` 停止后，从 `Worker.doWork()` 返回什么已不重要；`Result` 将被忽略。

##### CoroutineWorker的线程处理

对于 Kotlin 用户，WorkManager 为[协程](https://kotlinlang.org/docs/reference/coroutines-overview.html)提供了一流的支持。如要开始使用，请将 [`work-runtime-ktx` 包含到您的 gradle 文件中](https://developer.android.com/jetpack/androidx/releases/work?hl=zh-cn#declaring_dependencies)。不要扩展 [`Worker`](https://developer.android.com/reference/androidx/work/Worker?hl=zh-cn)，而应扩展 [`CoroutineWorker`](https://developer.android.com/reference/kotlin/androidx/work/CoroutineWorker?hl=zh-cn)，后者包含 `doWork()` 的挂起版本。例如，如果要构建一个简单的 `CoroutineWorker` 来执行某些网络操作，您需要执行以下操作：

```kotlin
class CoroutineDownloadWorker(
    context: Context,
    params: WorkerParameters
) : CoroutineWorker(context, params) {

    override suspend fun doWork(): Result {
        val data = downloadSynchronously("https://www.google.com")
        saveData(data)
        return Result.success()
    }
}
```

[`CoroutineWorker.doWork()`](https://developer.android.com/reference/kotlin/androidx/work/CoroutineWorker?hl=zh-cn#doWork()) 是一个“挂起”函数。此代码不同于 `Worker`，不会在 [`Configuration`](https://developer.android.com/reference/androidx/work/Configuration?hl=zh-cn) 中指定的 `Executor` 中运行，而是默认为 `Dispatchers.Default`。您可以提供自己的 `CoroutineContext` 来自定义这个行为。在上面的示例中，您可能希望在 `Dispatchers.IO` 上完成此操作，如下所示：

```kotlin
class CoroutineDownloadWorker(
    context: Context,
    params: WorkerParameters
) : CoroutineWorker(context, params) {

    override suspend fun doWork(): Result {
        withContext(Dispatchers.IO) {
            val data = downloadSynchronously("https://www.google.com")
            saveData(data)
            return Result.success()
        }
    }
}
```

`CoroutineWorker` 通过取消协程并传播取消信号来自动处理停工情况。您无需执行任何特殊操作来处理[停工情况](https://developer.android.com/topic/libraries/architecture/workmanager/how-to/managing-work?hl=zh-cn#cancelling)。

您还可以使用 [`RemoteCoroutineWorker`](https://developer.android.com/reference/kotlin/androidx/work/multiprocess/RemoteCoroutineWorker?hl=zh-cn)（`ListenableWorker` 的实现）将工作器绑定到特定进程。

`RemoteCoroutineWorker` 会使用您在构建工作请求时于输入数据中提供的两个额外参数绑定到特定进程：`ARGUMENT_CLASS_NAME` 和 `ARGUMENT_PACKAGE_NAME`。

以下示例演示了如何构建绑定到特定进程的工作请求：

```kotlin
val PACKAGE_NAME = "com.example.background.multiprocess"

val serviceName = RemoteWorkerService::class.java.name
val componentName = ComponentName(PACKAGE_NAME, serviceName)

val data: Data = Data.Builder()
   .putString(ARGUMENT_PACKAGE_NAME, componentName.packageName)
   .putString(ARGUMENT_CLASS_NAME, componentName.className)
   .build()

return OneTimeWorkRequest.Builder(ExampleRemoteCoroutineWorker::class.java)
   .setInputData(data)
   .build()
```

另外对于每个 `RemoteWorkerService`，您还需要在 `AndroidManifest.xml` 文件中添加服务定义：

```xml
<manifest ... >
    <service
            android:name="androidx.work.multiprocess.RemoteWorkerService"
            android:exported="false"
            android:process=":worker1" />

        <service
            android:name=".RemoteWorkerService2"
            android:exported="false"
            android:process=":worker2" />
    ...
</manifest>
```

##### RxWorker的线程处理

我们在 WorkManager 与 RxJava 之间提供互操作性。如需开始使用这种互操作性，除了在您的 gradle 文件中包含 [`work-runtime` 之外，还应包含 `work-rxjava3` 依赖项](https://developer.android.com/jetpack/androidx/releases/work?hl=zh-cn#declaring_dependencies)。而且还有一个支持 rxjava2 的 `work-rxjava2` 依赖项，您可以根据情况使用。

然后，您应该扩展 `RxWorker`，而不是扩展 `Worker`。最后替换 [`RxWorker.createWork()`](https://developer.android.com/reference/androidx/work/RxWorker?hl=zh-cn#createWork()) 方法以返回 `Single`，用于表示代码执行的 [`Result`](https://developer.android.com/reference/androidx/work/ListenableWorker.Result?hl=zh-cn)，如下所示：

```kotlin
class RxDownloadWorker(
        context: Context,
        params: WorkerParameters
) : RxWorker(context, params) {
    override fun createWork(): Single<Result> {
        return Observable.range(0, 100)
                .flatMap { download("https://www.example.com") }
                .toList()
                .map { Result.success() }
    }
}
```

`RxWorker.createWork()` 在主线程上调用，但默认情况下会在后台线程上订阅返回值。您可以替换 [`RxWorker.getBackgroundScheduler()`](https://developer.android.com/reference/androidx/work/RxWorker?hl=zh-cn#getBackgroundScheduler()) 来更改订阅线程。

当 `RxWorker` 为 `onStopped()` 时，系统会处理订阅，因此您无需以任何特殊方式处理[停工情况](https://developer.android.com/topic/libraries/architecture/workmanager/how-to/managing-work?hl=zh-cn#cancelling)。

##### ListenableWorker的线程处理

在某些情况下，您可能需要提供自定义线程处理策略。例如，您可能需要处理基于回调的异步操作。在这种情况下，不能只依靠 `Worker` 来完成操作，因为它无法以阻塞方式完成这项工作。

`ListenableWorker` 是最基本的工作器 API；[`Worker`](https://developer.android.com/reference/androidx/work/Worker?hl=zh-cn)、[`CoroutineWorker`](https://developer.android.com/reference/kotlin/androidx/work/CoroutineWorker?hl=zh-cn) 和 [`RxWorker`](https://developer.android.com/reference/androidx/work/RxWorker?hl=zh-cn) 都是从这个类衍生而来的。`ListenableWorker` 只会发出信号以表明应该开始和停止工作，而线程处理则完全交您决定。开始工作信号在主线程上调用，因此请务必手动转到您选择的后台线程。

抽象方法 [`ListenableWorker.startWork()`](https://developer.android.com/reference/androidx/work/ListenableWorker?hl=zh-cn#startWork()) 会返回一个将使用操作的 [`Result`](https://developer.android.com/reference/androidx/work/ListenableWorker.Result?hl=zh-cn) 设置的 `ListenableFuture`。`ListenableFuture` 是一个轻量级接口：它是一个 `Future`，用于提供附加监听器和传播异常的功能。在 `startWork` 方法中，应该返回 `ListenableFuture`，完成操作后，您需要使用操作的 `Result` 设置这个返回结果。您可以通过以下两种方式之一创建 `ListenableFuture` 实例：

1. 如果您使用的是 Guava，请使用 `ListeningExecutorService`。

2. 否则，请将 [`councurrent-futures`](https://developer.android.com/jetpack/androidx/releases/concurrent?hl=zh-cn#declaring_dependencies) 包含到您的 gradle 文件中并使用 [`CallbackToFutureAdapter`](https://developer.android.com/reference/androidx/concurrent/futures/CallbackToFutureAdapter?hl=zh-cn)。

   ```groovy
   dependencies {
       implementation("androidx.concurrent:concurrent-futures:1.1.0")
   
       // Kotlin
       implementation("androidx.concurrent:concurrent-futures-ktx:1.1.0")
   }
   ```

如果您希望基于异步回调执行某些工作，则应以类似如下的方式执行：

```kotlin
class CallbackWorker(
        context: Context,
        params: WorkerParameters
) : ListenableWorker(context, params) {
    override fun startWork(): ListenableFuture<Result> {
        return CallbackToFutureAdapter.getFuture { completer ->
            val callback = object : Callback {
                var successes = 0

                override fun onFailure(call: Call, e: IOException) {
                    completer.setException(e)
                }

                override fun onResponse(call: Call, response: Response) {
                    successes++
                    if (successes == 100) {
                        completer.set(Result.success())
                    }
                }
            }

            repeat(100) {
                downloadAsynchronously("https://example.com", callback)
            }

            callback
        }
    }
}
```

如果您的工作[停止](https://developer.android.com/topic/libraries/architecture/workmanager/how-to/managing-work?hl=zh-cn#cancelling)会发生什么？如果预计工作会停止，则始终会取消 `ListenableWorker` 的 `ListenableFuture`。通过使用 `CallbackToFutureAdapter`，您只需添加一个取消监听器即可，如下所示：

```kotlin
class CallbackWorker(
        context: Context,
        params: WorkerParameters
) : ListenableWorker(context, params) {
    override fun startWork(): ListenableFuture<Result> {
        return CallbackToFutureAdapter.getFuture { completer ->
            val callback = object : Callback {
                var successes = 0

                override fun onFailure(call: Call, e: IOException) {
                    completer.setException(e)
                }

                override fun onResponse(call: Call, response: Response) {
                    ++successes
                    if (successes == 100) {
                        completer.set(Result.success())
                    }
                }
            }

 completer.addCancellationListener(cancelDownloadsRunnable, executor)

            repeat(100) {
                downloadAsynchronously("https://example.com", callback)
            }

            callback
        }
    }
}
```

您还可以使用 [`RemoteListenableWorker`](https://developer.android.com/reference/kotlin/androidx/work/multiprocess/RemoteListenableWorker?hl=zh-cn)（`ListenableWorker` 的实现）将工作器绑定到特定进程。`RemoteListenableWorker` 会使用您在构建工作请求时于输入数据中提供的两个额外参数绑定到特定进程：`ARGUMENT_CLASS_NAME` 和 `ARGUMENT_PACKAGE_NAME`。

```kotlin
val PACKAGE_NAME = "com.example.background.multiprocess"

val serviceName = RemoteWorkerService::class.java.name
val componentName = ComponentName(PACKAGE_NAME, serviceName)

val data: Data = Data.Builder()
   .putString(ARGUMENT_PACKAGE_NAME, componentName.packageName)
   .putString(ARGUMENT_CLASS_NAME, componentName.className)
   .build()

return OneTimeWorkRequest.Builder(ExampleRemoteListenableWorker::class.java)
   .setInputData(data)
   .build()
```

```
<manifest ... >
    <service
            android:name="androidx.work.multiprocess.RemoteWorkerService"
            android:exported="false"
            android:process=":worker1" />

        <service
            android:name=".RemoteWorkerService2"
            android:exported="false"
            android:process=":worker2" />
    ...
</manifest>
```

#### 支持长时间运行的Worker

WorkManager `2.3.0-alpha02` 增加了对长时间运行的工作器的内置支持。 在这种情况下，WorkManager 可以向操作系统提供一个信号，指示在此项工作执行期间应尽可能让进程保持活跃状态。这些工作器可以运行超过 10 分钟。这一新功能的示例用例包括批量上传或下载（不可分块）、在本地进行的机器学习模型处理，或者对应用的用户很重要的任务。

在后台，WorkManager 会代表您管理和运行前台服务以执行 [`WorkRequest`](https://developer.android.com/reference/androidx/work/WorkRequest?hl=zh-cn)，同时还会显示可配置的通知。

**[`ListenableWorker`](https://developer.android.com/reference/androidx/work/ListenableWorker?hl=zh-cn) 现在支持 [`setForegroundAsync()`](https://developer.android.com/reference/androidx/work/ListenableWorker?hl=zh-cn#setProgressAsync(androidx.work.Data)) API，而 [`CoroutineWorker`](https://developer.android.com/reference/kotlin/androidx/work/CoroutineWorker?hl=zh-cn) 则支持挂起 [`setForeground()`](https://developer.android.com/reference/kotlin/androidx/work/CoroutineWorker?hl=zh-cn#setforeground) API。这些 API 允许开发者指定此 `WorkRequest` 是“重要的”（从用户的角度来看）或“长时间运行的”任务。**

从 `2.3.0-alpha03` 开始，WorkManager 还允许您创建 [`PendingIntent`](https://developer.android.com/reference/android/app/PendingIntent?hl=zh-cn)，此 Intent 可用于取消工作器，而不必使用 [`createCancelPendingIntent()`](https://developer.android.com/reference/androidx/work/WorkManager?hl=zh-cn#createCancelPendingIntent(java.util.UUID)) API 注册新的 Android 组件。此方法与 `setForegroundAsync()` 或 `setForeground()` API 一起使用时特别有用，可用于添加一个取消 `Worker` 的通知操作。

##### 创建和管理长时间运行的工作器

Kotlin 开发者应使用 [`CoroutineWorker`](https://developer.android.com/reference/kotlin/androidx/work/CoroutineWorker?hl=zh-cn)。您可以不使用 `setForegroundAsync()`，而使用该方法的挂起版本 [`setForeground()`](https://developer.android.com/reference/kotlin/androidx/work/CoroutineWorker?hl=zh-cn#setforeground)。

```kotlin
class DownloadWorker(context: Context, parameters: WorkerParameters) :
   CoroutineWorker(context, parameters) {

   private val notificationManager =
       context.getSystemService(Context.NOTIFICATION_SERVICE) as
               NotificationManager

   override suspend fun doWork(): Result {
       val inputUrl = inputData.getString(KEY_INPUT_URL)
                      ?: return Result.failure()
       val outputFile = inputData.getString(KEY_OUTPUT_FILE_NAME)
                      ?: return Result.failure()
       // Mark the Worker as important
       val progress = "Starting Download"
       setForeground(createForegroundInfo(progress))
       download(inputUrl, outputFile)
       return Result.success()
   }

   private fun download(inputUrl: String, outputFile: String) {
       // Downloads a file and updates bytes read
       // Calls setForegroundInfo() periodically when it needs to update
       // the ongoing Notification
   }
   // Creates an instance of ForegroundInfo which can be used to update the
   // ongoing notification.
   private fun createForegroundInfo(progress: String): ForegroundInfo {
       val id = applicationContext.getString(R.string.notification_channel_id)
       val title = applicationContext.getString(R.string.notification_title)
       val cancel = applicationContext.getString(R.string.cancel_download)
       // This PendingIntent can be used to cancel the worker
       val intent = WorkManager.getInstance(applicationContext)
               .createCancelPendingIntent(getId())

       // Create a Notification channel if necessary
       if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
           createChannel()
       }

       val notification = NotificationCompat.Builder(applicationContext, id)
           .setContentTitle(title)
           .setTicker(title)
           .setContentText(progress)
           .setSmallIcon(R.drawable.ic_work_notification)
           .setOngoing(true)
           // Add the cancel action to the notification which can
           // be used to cancel the worker
           .addAction(android.R.drawable.ic_delete, cancel, intent)
           .build()

       return ForegroundInfo(notification)
   }

   @RequiresApi(Build.VERSION_CODES.O)
   private fun createChannel() {
       // Create a Notification channel
   }

   companion object {
       const val KEY_INPUT_URL = "KEY_INPUT_URL"
       const val KEY_OUTPUT_FILE_NAME = "KEY_OUTPUT_FILE_NAME"
   }
}
```

使用 `ListenableWorker` 或 `Worker` 的开发者可以调用 [`setForegroundAsync()`](https://developer.android.com/reference/androidx/work/ListenableWorker?hl=zh-cn#setForegroundAsync(androidx.work.ForegroundInfo)) API，该 API 会返回 `ListenableFuture`。您还可以调用 `setForegroundAsync()` 以更新持续运行的 `Notification`。

```java
public class DownloadWorker extends Worker {
   private static final String KEY_INPUT_URL = "KEY_INPUT_URL";
   private static final String KEY_OUTPUT_FILE_NAME = "KEY_OUTPUT_FILE_NAME";

   private NotificationManager notificationManager;

   public DownloadWorker(
       @NonNull Context context,
       @NonNull WorkerParameters parameters) {
           super(context, parameters);
           notificationManager = (NotificationManager)
               context.getSystemService(NOTIFICATION_SERVICE);
   }

   @NonNull
   @Override
   public Result doWork() {
       Data inputData = getInputData();
       String inputUrl = inputData.getString(KEY_INPUT_URL);
       String outputFile = inputData.getString(KEY_OUTPUT_FILE_NAME);
       // Mark the Worker as important
       String progress = "Starting Download";
       setForegroundAsync(createForegroundInfo(progress));
       download(inputUrl, outputFile);
       return Result.success();
   }

   private void download(String inputUrl, String outputFile) {
       // Downloads a file and updates bytes read
       // Calls setForegroundAsync(createForegroundInfo(myProgress))
       // periodically when it needs to update the ongoing Notification.
   }

   @NonNull
   private ForegroundInfo createForegroundInfo(@NonNull String progress) {
       // Build a notification using bytesRead and contentLength

       Context context = getApplicationContext();
       String id = context.getString(R.string.notification_channel_id);
       String title = context.getString(R.string.notification_title);
       String cancel = context.getString(R.string.cancel_download);
       // This PendingIntent can be used to cancel the worker
       PendingIntent intent = WorkManager.getInstance(context)
               .createCancelPendingIntent(getId());

       if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
           createChannel();
       }

       Notification notification = new NotificationCompat.Builder(context, id)
               .setContentTitle(title)
               .setTicker(title)
               .setSmallIcon(R.drawable.ic_work_notification)
               .setOngoing(true)
               // Add the cancel action to the notification which can
               // be used to cancel the worker
               .addAction(android.R.drawable.ic_delete, cancel, intent)
               .build();

       return new ForegroundInfo(notification);
   }

   @RequiresApi(Build.VERSION_CODES.O)
   private void createChannel() {
       // Create a Notification channel
   }
}
```

如果您的应用以 Android 10（API 级别 29）或更高版本为目标平台，且包含需要位置信息访问权限的长时间运行的工作器，请指明该工作器使用 `location` 的[前台服务类型](https://developer.android.com/guide/components/foreground-services?hl=zh-cn#types)。此外，如果您的应用以 Android 11（API 级别 30）或更高版本为目标平台，且包含需要访问相机或麦克风的长时间运行的工作器，请分别声明 `camera` 或 `microphone` 前台服务类型。

如需添加这些前台服务类型，请完成以下各部分中介绍的步骤。

- 在应用清单中声明前台服务类型:
  在应用的清单中声明工作器的前台服务类型。在以下示例中，工作器需要位置信息和麦克风访问权限：AndroidManifest.xml

  ```xml
  <service
     android:name="androidx.work.impl.foreground.SystemForegroundService"
     android:foregroundServiceType="location|microphone"
     tools:node="merge" />
  ```

  > [清单合并工具](https://developer.android.com/studio/build/manage-manifests?hl=zh-cn#merge-manifests)将上述代码段中的 `` 元素声明与 WorkManager 通过其自身清单中的 `SystemForegroundService` 定义的声明合并在一起。

  

- 在运行时指定前台服务类型：
  当您调用 `setForeground()` 或 `setForegroundAsync()` 时，请指定前台服务类型 [`FOREGROUND_SERVICE_TYPE_LOCATION`](https://developer.android.com/reference/android/content/pm/ServiceInfo?hl=zh-cn#FOREGROUND_SERVICE_TYPE_LOCATION)、[`FOREGROUND_SERVICE_TYPE_CAMERA`](https://developer.android.com/reference/android/content/pm/ServiceInfo?hl=zh-cn#FOREGROUND_SERVICE_TYPE_CAMERA) 或 [`FOREGROUND_SERVICE_TYPE_MICROPHONE`](https://developer.android.com/reference/android/content/pm/ServiceInfo?hl=zh-cn#FOREGROUND_SERVICE_TYPE_MICROPHONE)，如以下代码段所示。

  > 在运行时，您可以选择限制长时间运行的工作器对数据的访问，方法是传递您已在清单文件中声明的一部分前台服务类型。您可以在[前台服务类型说明](https://developer.android.com/guide/components/foreground-services?hl=zh-cn#types)中查看一些示例。

  

  ```kotlin
  private fun createForegroundInfo(progress: String): ForegroundInfo {
     // ...
     return ForegroundInfo(NOTIFICATION_ID, notification,
             FOREGROUND_SERVICE_TYPE_LOCATION or
  FOREGROUND_SERVICE_TYPE_MICROPHONE) }
  
  ```

  

