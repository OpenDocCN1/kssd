
为了进行比较，以下示例代码展示了 `Service` 类的实现，该类执行的工作与上述使用 `IntentService` 的示例完全相同。换言之，对于每个启动请求，其均使用工作线程来执行作业，且每次仅处理一个请求。

```java
public class HelloService extends Service {
    private Looper serviceLooper;
    private ServiceHandler serviceHandler;

    // 从线程接收消息的Handler
    private final class ServiceHandler extends Handler {
        public ServiceHandler(Looper looper) {
            super(looper);
        }

        @Override
        public void handleMessage(Message msg) {
            // 通常在这里会执行一些工作，比如下载文件。
            // 这个示例中，我们只是睡眠5秒钟。
            try {
                Thread.sleep(5000);
            } catch (InterruptedException e) {
                // 恢复中断状态。
                Thread.currentThread().interrupt();
            }

            // 使用startId停止服务，以确保我们不会在处理其他任务时停止服务
            stopSelf(msg.arg1);
        }
    }

    @Override
    public void onCreate() {
        // 启动运行服务的线程。注意，我们创建了一个单独的线程，因为服务通常在进程的主线程中运行，
        // 我们不希望阻塞主线程。我们还将其设置为后台优先级，以防止CPU密集型工作影响我们的UI。
        HandlerThread thread = new HandlerThread(
            "ServiceStartArguments",
            Process.THREAD_PRIORITY_BACKGROUND
        );
        thread.start();

        // 获取HandlerThread的Looper，并将其用于我们的Handler
        serviceLooper = thread.getLooper();
        serviceHandler = new ServiceHandler(serviceLooper);
    }

    @Override
    public int onStartCommand(Intent intent, int flags, int startId) {
        Toast.makeText(this, "服务启动中", Toast.LENGTH_SHORT).show();

        // 对于每个启动请求，发送一条消息来启动一个任务，并传递startId，
        // 这样我们在完成任务后就知道要停止哪个请求
        Message msg = serviceHandler.obtainMessage();
        msg.arg1 = startId;
        serviceHandler.sendMessage(msg);

        // 如果我们被杀死，返回之后重新启动
        return START_STICKY;
    }

    @Override
    public IBinder onBind(Intent intent) {
        // 我们不提供绑定，所以返回null
        return null;
    }

    @Override
    public void onDestroy() {
        Toast.makeText(this, "服务已完成", Toast.LENGTH_SHORT).show();
    }
}
```

对比可以发现，相较于使用 `IntentService`，此示例需要执行更多工作、编写更繁多的代码。

但是，由于 `onStartCommand()` 的每个调用均有开发者自己处理，因此开发者可以同时执行多个请求。此示例并未这样做，但想要同时执行多个请求，则可以为每个请求创建新线程，然后立即运行这些线程（而非等待上一个请求完成）。

请注意，`onStartCommand()` 方法必须返回一个整数值。这个整数值用于描述系统在终止服务的情况下如何继续运行服务。`IntentService` 的默认实现会处理这种情况，但开发者可以进行修改。从 `onStartCommand()` 返回的值必须是以下常量之一：

- `START_NOT_STICKY`

  如果系统在 `onStartCommand()` 返回后终止服务，则系统*不会*重新创建服务，除非有待处理的挂起 Intent。这是最安全的选项，可以避免在不需要运行服务的情况下重启服务，并且应用可以轻松地重新启动所有未完成的任务。

- `START_STICKY`

  如果系统在 `onStartCommand()` 返回后终止服务，则系统会重新创建服务并调用 `onStartCommand()`，但不会重新传递最后一个 Intent。相反，系统会调用 `onStartCommand()` 并传递一个空的 Intent，除非有挂起的 Intent 需要启动服务。这种情况下，系统会传递这些 Intent。这个常量适用于不执行命令，但需要无限期运行并等待任务的媒体播放器（或类似服务）。

- `START_REDELIVER_INTENT`

  如果系统在 `onStartCommand()` 返回后终止服务，则系统会重新创建服务并调用 `onStartCommand()`，并且会传递给服务最后一个 Intent。所有挂起的 Intent 会依次传递。这个常量适用于需要立即恢复执行的任务（例如下载文件）的服务。

## 四、启动服务

我们了解完了如何通过扩展 `Service` 类和 `IntentService` 类来创建启动服务，接下来我们该了解一下要如何启动服务了。

### 绑定Service

调用者可以通过 `bindService()` 方法来绑定 `Service`，该方法需要传入一个 `Intent` 对象和 `ServiceConnection` 对象。`Intent` 对象用于指定要绑定的 `Service`，`ServiceConnection` 对象用于监听 `Service` 的连接状态。

当调用 `bindService()` 方法时，如果 `Service` 已经启动，则会调用 `Service` 的 `onBind()` 方法，返回一个 `IBinder` 对象。调用者可以通过该对象与 `Service` 进行通信。如果 `Service` 未启动，则会先启动 `Service`，再调用 `onBind()` 方法。

### 解绑Service

调用者可以通过 `unbindService()` 方法来解绑 `Service`，该方法需要传入一个 `ServiceConnection` 对象。当调用 `unbindService()` 方法时，会调用 `ServiceConnection` 对象的 `onServiceDisconnected()` 方法，表示 `Service` 已经断开连接。

如果 `Service` 没有与其他组件绑定，则会调用 `Service` 的 `onUnbind()` 方法，表示 `Service` 已经解绑。如果 `Service` 没有其他工作要做，则可以在 `onUnbind()` 方法中停止 `Service`，释放资源。

需要注意的是，当调用者与 `Service` 绑定后，需要及时解绑，否则会导致 `Service` 一直存在，浪费系统资源。在解绑 `Service` 时，需要确保所有与 `Service` 绑定的组件都已经解绑，否则 `Service` 不会被销毁。

## 五、停止服务

启动的 `Service` 需要自行管理自己的生命周期。也就是说，除非必须释放内存资源，否则系统不会主动停止或销毁 `Service`。`Service` 在 `onStartCommand()` 方法返回后仍然会继续运行。要停止 `Service` 的运行，可以通过调用 `stopSelf()` 方法自行停止，或者由另一个组件通过调用 `stopService()` 方法来停止。

一旦调用了 `stopSelf()` 或 `stopService()` 来停止 `Service`，系统会尽快销毁 `Service`。

如果 `Service` 同时处理多个 `onStartCommand()` 请求，那么在处理完一个启动请求后就立即停止 `Service` 是不合适的，因为可能会收到新的启动请求（在第一个请求结束时停止 `Service` 会终止第二个请求）。为了避免这个问题，可以使用 `stopSelf(int)` 方法，确保停止请求始终基于最近的启动请求。也就是说，在调用 `stopSelf(int)` 方法时，需要传递与停止请求 ID 相对应的启动请求 ID（即传递给 `onStartCommand()` 的 `startId`）。如果 `Service` 在能够调用 `stopSelf(int)` 之前收到新的启动请求，则 ID 不匹配，`Service` 也不会停止。

需要注意的是，为了避免浪费系统资源和耗电，务必确保在任务完成后停止 `Service`。如果需要，其他组件可以通过调用 `stopService()` 来停止 `Service`。即使 `Service` 启用了绑定，如果 `Service` 收到 `onStartCommand()` 的调用，仍然需要手动停止 `Service`。

## 六、创建绑定服务

绑定服务允许应用组件通过调用 `bindService()` 方法与其进行绑定，从而创建一个长期的连接。通常情况下，这种服务不允许组件通过调用 `startService()` 方法来启动它。

如果需要与 `Activity` 和其他应用组件进行交互，或者需要通过进程间通信（IPC）向其他应用公开某些应用功能，那么就应该创建绑定服务。

要创建绑定服务，开发者是需要实现 `onBind()` 回调方法的，实现 `onBind()` 回调方法会返回一个 `IBinder` 对象，以定义与服务进行通信的接口。然后，其他应用组件可以通过调用 `bindService()` 方法来获取该接口，并开始调用与服务相关的方法。绑定服务只会为绑定到它的应用组件提供服务，所以如果没有组件与该服务绑定，系统就会销毁该服务。与启动服务不同，开发者是不需要以相同的方式停止绑定服务。

要创建绑定服务，开发者就必须定义一个接口，指定客户端如何与服务进行通信。这个接口必须是 `IBinder` 的实现，并且服务必须从 `onBind()` 回调方法返回该接口。客户端收到 `IBinder` 后，就可以开始通过该接口与服务进行交互。

多个客户端可以同时绑定到同一个服务。当与服务的交互完成后，客户端可以通过调用 `unbindService()` 方法来解除绑定。如果没有绑定到服务的客户端，系统就会销毁该服务。

实现绑定服务有多种方法，比启动服务更复杂。因此，如果你对此有兴趣建议你可以查阅相关文档，详细了解如何实现绑定服务。

## 七、在前台运行服务

前台服务是用户主动意识到的一种服务，系统不会在内存不足时终止它。前台服务必须在状态栏中显示通知，并将其放在正在运行的标题下方。这意味着除非停止服务或将其从前台移除，否则无法清除通知。

请注意，应该限制应用使用前台服务的频率。

只有当应用执行的任务需要用户查看（即使该任务未直接与应用交互）时，才应该使用前台服务。因此，前台服务的通知必须具有 `PRIORITY_LOW` 或更高的优先级，以确保用户了解应用正在进行的任务。如果某个操作不是特别重要，开发者希望使用最低优先级的通知，那么前台服务可能就不适合，相反，开发者可以考虑使用后台任务处理。

每个运行服务的应用都会增加系统的负担，并消耗系统资源。如果应用尝试使用低优先级的通知隐藏其服务，可能会降低用户正在主动交互的应用的性能。因此，如果某个应用尝试运行拥有最低优先级通知的服务，则系统会在抽屉式通知栏的底部调用出该应用的行为。

比如，如果一个音乐播放器应用通过服务播放音乐，应将其设置为在前台运行，这样用户会明确意识到音乐播放器正在运行。状态栏中的通知可以显示正在播放的歌曲，并且允许用户通过启动 `Activity` 与音乐播放器进行交互。同样，如果应用允许用户追踪其位置，需要通过前台服务来追踪用户的位置。

请注意，在开发时如果应用面向 Android 9（API 级别 28）或更高版本，并且使用前台服务，必须请求 `"FOREGROUND_SERVICE"` 权限。这是一种普通权限，系统会自动为请求此权限的应用授予它。如果面向 API 级别 28 或更高版本的应用尝试创建前台服务但未请求 `"FOREGROUND_SERVICE"` 权限，系统会抛出 `SecurityException` 异常。

要请求服务在前台运行，可以调用 `startForeground()` 方法。此方法接受两个参数：唯一标识通知的整型数和用于状态栏的 `Notification` 对象。此通知必须具有 `PRIORITY_LOW` 或更高的优先级。

要从前台移除服务，可以调用 `stopForeground()` 方法。此方法接受一个布尔值，指示是否同时移除状态栏通知。调用此方法不会停止服务，但是如果在服务仍然在前台运行时停止它，通知也会被移除。

以下是一个示例，展示了如何使用 Kotlin 语言创建前台服务和通知：

```java
Intent notificationIntent = new Intent(this, ExampleActivity.class);
PendingIntent pendingIntent =
    PendingIntent.getActivity(this, 0, notificationIntent, 0);
Notification notification =
    new Notification.Builder(this, CHANNEL_DEFAULT_IMPORTANCE)
        .setContentTitle(getText(R.string.notification_title))
        .setContentText(getText(R.string.notification_message))
        .setSmallIcon(R.drawable.icon)
        .setContentIntent(pendingIntent)
        .setTicker(getText(R.string.ticker_text))
        .build();
startForeground(ONGOING_NOTIFICATION_ID, notification);
```

请注意，传递给 `startForeground()` 方法的整型 ID 不能为 0。

如要从前台移除服务，可以调用 `stopForeground()`。此方法采用布尔值，指示是否需同时移除状态栏通知。此方法不会停止服务。但是，如果开发者在服务仍运行于前台时将其停止，则通知也会随之移除。

## 八、管理服务的生命周期

服务的生命周期比 `Activity` 的生命周期要简单得多。服务可以在后台运行而不被用户察觉，因此比起 `Activity`，服务更加重要的是密切关注服务的创建和销毁。

服务的生命周期可以遵循以下两种路径之一：

1. **启动服务**：服务在其他组件调用 `startService()` 方法时被创建，并且会一直运行下去，直到调用 `stopSelf()` 方法停止自己的运行，或者其他组件调用 `stopService()` 方法停止该服务。一旦服务停止，系统会销毁它。
2. **绑定服务**：服务在其他组件（客户端）调用 `bindService()` 方法时被创建。客户端通过 `IBinder` 接口与服务进行通信。客户端可以通过调用 `unbindService()` 方法关闭连接。多个客户端可以绑定到同一个服务，当所有绑定都取消后，系统会销毁该服务。（服务不需要自己停止运行）

这两种路径并不是完全独立的。开发者是可以绑定到已经通过 `startService()` 方法启动的服务。例如，开发者可以使用 `Intent` 来调用 `startService()` 方法启动后台音乐服务。然后，当用户需要对播放器进行控制或获取当前播放歌曲的信息时，`Activity` 可以通过调用 `bindService()` 方法绑定到该服务。在所有客户端取消绑定之前，`stopService()` 或 `stopSelf()` 方法实际上不会停止服务。

与 `Activity` 类似，服务也有生命周期回调方法，开发者可以通过实现这些方法来监控服务状态的变化并执行相应的工作。以下是一个展示了每个生命周期方法的服务示例：

```java
public class ExampleService extends Service {
    int startMode; // 表示服务被杀死时的行为方式
    IBinder binder; // 用于与绑定客户端通信的接口
    boolean allowRebind; // 表示是否应该使用onRebind方法

    @Override
    public void onCreate() {
        // 服务被创建
    }

    @Override
    public int onStartCommand(Intent intent, int flags, int startId) {
        // 服务正在启动，由于调用了startService()方法
        return startMode;
    }

    @Override
    public IBinder onBind(Intent intent) {
        // 有客户端通过bindService()方法绑定到服务
        return binder;
    }

    @Override
    public boolean onUnbind(Intent intent) {
        // 所有客户端都通过unbindService()方法取消了绑定
        return allowRebind;
    }

    @Override
    public void onRebind(Intent intent) {
        // 一个客户端通过bindService()方法重新绑定到服务，之前已经调用了onUnbind()方法
    }

    @Override
    public void onDestroy() {
        // 服务不再被使用，正在被销毁
    }
}
```

请注意，与 `Activity` 的生命周期回调方法不同，开发者是不需要调用这些回调方法的超类实现。

以下为官方给出的服务生命周期图示：

![](upload/attach/202311/926486_JYBPP4AHZH9FNVM.webp)

图左显示的是使用 `startService()` 创建的服务的生命周期，图右显示的是使用 `bindService()` 创建的服务的生命周期。

上图展示服务的典型回调方法。尽管该图分开介绍通过 `startService()` 创建的服务和通过 `bindService()` 创建的服务，但还请记住，不管服务是通过 `startService()` 还是 `bindService()` 启动的，都可以允许客户端与其进行绑定。这意味着，即使最初是通过客户端调用 `startService()` 来启动服务的，该服务仍然可以接收到客户端调用 `bindService()` 时的 `onBind()` 回调。换句话说，无论服务是通过哪种方式启动的，都可以允许客户端进行绑定操作。

通过实现服务生命周期的这些回调方法，开发者可以监控服务生命周期的以下两种嵌套循环：
