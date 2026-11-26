[//]: # (title: Schedule and manage background tasks in Compose Multiplatform)

Background-task APIs let your app schedule work to run once or repeatedly, including when users aren't actively
interacting with the app.
Common use cases include synchronizing data, uploading files, clearing caches, and processing data.

Each platform provides its own mechanisms for scheduling background work:

* Android provides [WorkManager](https://developer.android.com/develop/background-work/background-tasks/persistent/getting-started),
  as well as specialized APIs for exact alarms and foreground services.
* iOS provides the [Background Tasks framework](https://developer.apple.com/documentation/backgroundtasks) for
  refresh and processing work.
* Desktop apps can use JVM APIs such as
  [`ScheduledExecutorService`](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/concurrent/ScheduledExecutorService.html)
  for work in the app process, or integrate with operating-system services when work needs to continue after the
  app exits.
* Web apps can use
  [Web Workers](https://developer.mozilla.org/en-US/docs/Web/API/Web_Workers_API),
  [Service Workers](https://developer.mozilla.org/en-US/docs/Web/API/Service_Worker_API), or browser APIs such as
  [Background Sync](https://developer.mozilla.org/en-US/docs/Web/API/Background_Synchronization_API).
  Availability and execution behavior depend on the browser.

Compose Multiplatform doesn't currently provide a common background-task API across these platforms. For desktop
and web apps, use the corresponding platform APIs described above.

On mobile platforms, the operating system determines when background work can run based on factors such as
battery state, network availability, and system resource limits. Therefore, treat the scheduled time as the earliest possible
start time rather than an exact execution time.

To use one background-task API across Android and iOS, you can use the
[KMP WorkManager](https://github.com/brewkits/kmpworkmanager) community library.

## Use KMP WorkManager on Android and iOS

KMP WorkManager provides a shared API on top of Android WorkManager and iOS `BGTaskScheduler`. It lets you manage
background work without implementing the scheduling logic separately on each platform.

For now, the library supports Android and iOS only; it doesn't support desktop or web targets.
Supported platform versions, setup requirements, and APIs can change, so
check the library's [installation and platform setup guide](https://github.com/brewkits/kmpworkmanager/blob/main/docs/platform-setup.md)
before using it.

The following example uses the shared API to schedule a registered `SyncWorker` to run when a network connection is
available:

```kotlin
val scheduler = KmpWorkManager.getInstance().backgroundTaskScheduler

val result = scheduler.enqueue(
    id = "sync-pending-changes",
    trigger = TaskTrigger.OneTime(initialDelayMs = 0),
    workerClassName = "SyncWorker",
    constraints = Constraints(
        requiresNetwork = true,
    ),
    policy = ExistingPolicy.KEEP,
)

if (result == ScheduleResult.ACCEPTED) {
    println("Sync scheduled")
}
```

The task `id` identifies the work. With `ExistingPolicy.KEEP`, reusing the same `id` prevents the scheduler from
adding another instance of the task.
You can also use the `id` to cancel the task:

```kotlin
scheduler.cancel("sync-pending-changes")
```

KMP WorkManager also supports periodic work, execution constraints, retries, task state observation, task chains,
and group cancellation. For worker registration and scheduling options, see the library's
[quick start](https://github.com/brewkits/kmpworkmanager/blob/main/docs/quickstart.md).

Both Android and iOS impose execution limits. See KMP WorkManager's
[constraints and triggers guide](https://github.com/brewkits/kmpworkmanager/blob/main/docs/constraints-triggers.md)
for platform differences, the [iOS best practices](https://github.com/brewkits/kmpworkmanager/blob/main/docs/ios-best-practices.md),
and the [troubleshooting guide](https://github.com/brewkits/kmpworkmanager/blob/main/docs/TROUBLESHOOTING.md).

## What's next

* Learn how to [add dependencies on multiplatform libraries](multiplatform-add-dependencies.md).
* Learn about the [Compose lifecycle](compose-lifecycle.md).
