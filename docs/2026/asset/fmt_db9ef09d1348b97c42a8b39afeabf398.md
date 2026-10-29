
Service 是一种应用组件，它可以在后台执行长时间运行的操作，而无需提供用户界面。其他应用组件可以启动 Service，并且即使用户切换到其他应用，Service 仍将继续在后台运行。此外，组件可以通过绑定到 Service 来与其进行交互，甚至进行进程间通信 `IPC`。举例来说，Service 可以在后台处理网络请求、播放音乐、执行文件 I/O 操作或与内容提供者进行交互。

Service 有三种不同的类型，分别是前台服务、后台服务、应用组件绑定到服务。以下是三种不同类型的详解：

### 前台服务

前台服务执行一些用户能注意到的操作。例如，音频应用会使用前台服务来播放音频曲目。前台服务必须显示通知，通知就是在应用程序的界面之外显示的消息，旨在向用户提供提醒、来自他人的通信信息或应用中的其他实时信息。即使用户停止与应用的交互，前台服务仍会继续运行。

### 后台服务

后台服务不会执行用户能直接注意到的操作。例如，如果应用使用某个服务来压缩其存储空间，则此服务通常是后台服务。

### 应用组件绑定到服务

当应用组件通过调用 `bindService()` 方法绑定到服务时，服务即进入绑定状态。绑定服务提供了一个客户端-服务器接口，使得应用组件（客户端）可以与服务（服务器）进行交互、发送请求、接收结果，甚至可以通过进程间通信 `IPC` 跨进程执行这些操作。只有当服务与另一个应用组件绑定时，绑定服务才会运行。多个组件可以同时绑定到同一个服务，但只有当全部组件取消绑定后，该服务才会被销毁。

## 二、Service 基础知识

创建服务时，必须创建一个继承自 `Service` 类的子类或使用现有的 `Service` 子类。在实现中，开发者需要重写一些回调方法来处理服务的生命周期和将组件绑定到服务。`Service` 类提供了四个核心回调方法：`onStartCommand()`、`onBind()`、`onCreate()`、`onDestroy()`。以下为 Service 应重写的最重要的回调方法：

```
onStartCommand()
```

当另一个组件（如 `Activity`）请求启动服务时，系统会通过调用 `startService()` 来调用此回调方法。执行此回调方法时，服务即会启动并可在后台无限期运行。如果要实现此回调方法，则在服务工作完成后，开发者需要负责通过调用 `stopSelf()` 或 `stopService()` 来停止服务。（如果只是想提供绑定，则无需实现此方法。）

```
onBind()
```

当另一个组件想要与服务绑定时，系统会通过调用 `bindService()` 来调用此方法。在此方法的实现中，您必须通过返回 `IBinder` 提供一个接口，以供客户端用来与服务进行通信。请务必实现此方法；但是，如果开发者并不希望允许绑定，则应该会返回 `null`。

```
onCreate()
```

首次创建服务时，系统会调用 `onCreate()` 回调方法来执行一次性设置程序。在调用 `onStartCommand()` 或 `onBind()` 之前，系统会先调用 `onCreate()` 回调方法。如果服务已在运行，则不会再次调用 `onCreate()` 回调方法。

```
onDestroy()
```

当不再使用服务且准备将其销毁时，系统会调用 `onDestroy()` 回调方法。在 `onDestroy()` 回调方法中，服务应该通过实现此方法来清理任何资源，如线程、注册的侦听器、接收器等。这是服务接收的最后一个调用，表示服务即将被销毁。

Service 最重要的四个回调方法已然介绍完了，你可以发现 Service 的生命周期相较于 Activity 的生命周期简单许多，我们总结一下 Service 的生命周期：

如果组件通过调用 `startService()` 启动服务（这会引起对 `onStartCommand()` 的调用），则服务会一直运行，直到其使用 `stopSelf()` 自行停止运行，或由其他组件通过调用 `stopService()` 将其停止为止。

如果组件通过调用 `bindService()` 来创建服务，且未调用 `onStartCommand()`，则服务只会在该组件与其绑定时运行。当该服务与其所有组件取消绑定后，系统便会将其销毁。

只有在内存过低且必须回收系统资源以供拥有用户焦点的 `Activity` 使用时，Android 系统才会停止服务。如果将服务绑定到拥有用户焦点的 `Activity`，则它其不太可能会终止；如果将服务声明为在前台运行，则其几乎永远不会终止。如果服务已启动并长时间运行，则系统逐渐降低其在后台任务列表中的位置，而服务被终止的概率也会大幅提升。如果服务是启动服务，则开发者必须将其设计为能够妥善处理系统执行的重启。如果系统终止服务，则其会在资源可用时立即重启服务，但这还取决于开发者从 `onStartCommand()` 返回的值。`onStartCommand()` 返回的值一个有五个，分别是：

<a href="elink@30eK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1k6i4k6W2L8r3!0H3k6i4u0Q4x3X3g2S2L8X3c8J5L8$3W2V1i4K6u0W2k6$3!0G2k6$3I4W2i4K6u0W2j5$3&6Q4x3V1k6J5k6h3k6W2M7X3g2F1j5$3g2Q4x3V1k6S2L8X3c8J5L8$3W2V1i4K6u0r3j5i4m8H3i4K6u0r3f1$3g2J5N6X3W2U0k6g2)9J5x3#2y4f1b7g2u0f1i4K6g2X3b7@1!0z5g2p5W2z5g2f1q4f1d9f1!0z5i4K6g2X3e0f1q4e0d9H3`.`.">START_CONTINUATION_MASK</a>

<a href="elink@2e4K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1k6i4k6W2L8r3!0H3k6i4u0Q4x3X3g2S2L8X3c8J5L8$3W2V1i4K6u0W2k6$3!0G2k6$3I4W2i4K6u0W2j5$3&6Q4x3V1k6J5k6h3k6W2M7X3g2F1j5$3g2Q4x3V1k6S2L8X3c8J5L8$3W2V1i4K6u0r3j5i4m8H3i4K6u0r3f1$3g2J5N6X3W2U0k6g2)9J5x3#2y4f1b7g2u0f1i4K6g2X3f1#2c8u0b7@1E0k6i4K6g2X3b7@1!0y4f1p5q4f1d9f1u0u0e0p5W2f1h3b7`.`.">START_STICKY_COMPATIBILITY</a>

<a href="elink@5a5K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1k6i4k6W2L8r3!0H3k6i4u0Q4x3X3g2S2L8X3c8J5L8$3W2V1i4K6u0W2k6$3!0G2k6$3I4W2i4K6u0W2j5$3&6Q4x3V1k6J5k6h3k6W2M7X3g2F1j5$3g2Q4x3V1k6S2L8X3c8J5L8$3W2V1i4K6u0r3j5i4m8H3i4K6u0r3f1$3g2J5N6X3W2U0k6g2)9J5x3#2y4f1b7g2u0f1i4K6g2X3f1#2c8u0b7@1E0k6">START_STICKY</a>

<a href="elink@5b0K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1k6i4k6W2L8r3!0H3k6i4u0Q4x3X3g2S2L8X3c8J5L8$3W2V1i4K6u0W2k6$3!0G2k6$3I4W2i4K6u0W2j5$3&6Q4x3V1k6J5k6h3k6W2M7X3g2F1j5$3g2Q4x3V1k6S2L8X3c8J5L8$3W2V1i4K6u0r3j5i4m8H3i4K6u0r3f1$3g2J5N6X3W2U0k6g2)9J5x3#2y4f1b7g2u0f1i4K6g2X3e0V1!0f1i4K6g2X3f1#2c8u0b7@1E0k6">START_NOT_STICKY</a>

<a href="elink@14fK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1k6i4k6W2L8r3!0H3k6i4u0Q4x3X3g2S2L8X3c8J5L8$3W2V1i4K6u0W2k6$3!0G2k6$3I4W2i4K6u0W2j5$3&6Q4x3V1k6J5k6h3k6W2M7X3g2F1j5$3g2Q4x3V1k6S2L8X3c8J5L8$3W2V1i4K6u0r3j5i4m8H3i4K6u0r3f1$3g2J5N6X3W2U0k6g2)9J5x3#2y4f1b7g2u0f1i4K6g2X3f1V1g2p5c8f1I4u0g2V1g2d9i4K6g2X3d9f1&6f1c8f1&6f1">START_REDELIVER_INTENT</a>

以上就是 `onStartCommand()` 的返回值，如有兴趣可以自行了解，我这里就不细说了。

服务和 Activity 以及其他组件一样必须要在应用的清单文件中声明所有服务。如果要声明服务，那需要在 `<application>` 元素中添加 `<service>` 元素用于声明服务组件，下面是示例：

```
<manifest ... >
...
<application ... >
<service android:name=".ExampleService" />
...
</application>
</manifest>
```

以上是基本示例，可以在 `<service>` 元素中加入其他属性，以定义一些特性，如启动服务及其运行时所在进程需要的权限。但请记住 `android:name` 属性是唯一必需的属性，用于指定服务的类名，此类名不能被改变，以避免因依赖显式 Intent 来启动或绑定服务而破坏代码的风险。以下是声明 service 的语法：

```
<service android:description="string resource"
android:directBootAware=["true" | "false"]
android:enabled=["true" | "false"]
android:exported=["true" | "false"]
android:foregroundServiceType=["camera" | "connectedDevice" |
"dataSync" | "location" | "mediaPlayback" |
"mediaProjection" | "microphone" | "phoneCall"]
android:icon="drawable resource"
android:isolatedProcess=["true" | "false"]
android:label="string resource"
android:name="string"
android:permission="string"
android:process="string" >
...
</service>
```

这里是大致了解一下语法，如需详细了解，有兴趣的可以跳转至 <a href="elink@048K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1k6i4k6W2L8r3!0H3k6i4u0Q4x3X3g2S2L8X3c8J5L8$3W2V1i4K6u0W2k6$3!0G2k6$3I4W2i4K6u0W2j5$3&6Q4x3V1k6Y4N6h3W2V1k6g2)9J5c8Y4c8G2M7r3W2U0M7#2)9J5c8X3#2S2L8X3W2X3k6i4y4@1i4K6u0r3M7$3g2J5N6X3W2U0k6g2)9J5k6r3g2D9k6h3#2W2L8Y4c8Q4x3@1k6Z5L8q4)9K6c8s2A6Z5i4K6u0V1j5$3^5`.">Android 开发者 | Android Developers (google.cn)</a> 自行观看。

## 三、创建启动 Service

另一个组件通过调用 `startService()` 启动服务，这样服务就会调用 `onStartCommand()` 方法。服务被另一个组件启动后，服务的生命周期即独立于启动它的组件，即使系统已销毁启动服务的组件，该服务还是可以在后台无限期地运行。想要关闭服务，可以在其工作完成时通过调用 `stopSelf()` 来自行停止运行，或者由另一个组件通过调用 `stopService()` 来将其停止。

应用组件（如 `Activity`）可以通过调用 `startService()` 方法并传递 `Intent` 对象（指定服务并包含待使用服务的所有数据）来启动服务。服务会在 `onStartCommand()` 方法接收此 `Intent`。

例如，假设某 `Activity` 需要将一些数据保存到在线数据库中。该 `Activity` 可以启动一个协同服务，并通过向 `startService()` 传递一个 `Intent`，为该服务提供要保存的数据。服务会通过 `onStartCommand()` 接收 `Intent`，连接到互联网并执行数据库事务。事务完成后，服务将自行停止并销毁。

如果服务在用户与来自同一应用的 `Activity` 进行交互时执行密集型或阻止性操作，则会降低 `Activity` 性能，这是因为默认情况下，服务与服务声明所在的应用运行于同一进程，并且运行于该应用的主线程中，这样服务大量消耗主线程有限的资源，从而影响应用正在运行的任何 `Activity` 的性能，自然就会影响应用的性能了。开发者要解决这个问题，一般情况会在服务内启动新线程。

通常要创建启动服务可以通过扩展两个类来实现：

`Service`：这是适用于所有服务的基类。扩展此类时必须创建用于执行所有服务工作的新线程，因为服务默认使用应用的主线程，这会降低应用正在运行的任何 `Activity` 的性能。

`IntentService`：这是 `Service` 的子类，其使用工作线程逐一处理所有启动请求。如果应用程序不要求服务同时处理多个请求，开发者会以此类为最佳选择。实现 `onHandleIntent()`（这个方法是在工作线程上调用一个需要处理的请求），该方法会接收每个启动请求的 `Intent`，以便开发者执行后台工作。

### 扩展 IntentService 类

由于大多数启动服务无需同时处理多个请求（实际上，这种多线程情况可能很危险），因此最佳选择是利用 `IntentService` 类实现服务。

`IntentService` 类会执行以下操作：

- 创建默认的工作线程，用于在应用的主线程外执行传递给 `onStartCommand()` 的所有 `Intent`。
- 创建工作队列，用于将 `Intent` 逐一传递给 `onHandleIntent()` 实现，这样就永远不必担心多线程问题。
- 在处理完所有启动请求后停止服务，因此就永远不必调用 `stopSelf()`。
- 提供 `onBind()` 的默认实现（返回 `null`），除非服务需要提供绑定，否则不需要实现这个方法，因为默认实现返回 null。如果服务需要提供绑定，那么就会返回一个 `IBinder` 对象，客户端可以通过它调用服务。
- 提供 `onStartCommand()` 的默认实现，可将 `Intent` 依次发送到工作队列和 `onHandleIntent()` 实现。

如要完成客户端提供的工作，就需要实现 `onHandleIntent()`。不过，还需要为服务提供小型构造函数。

以下是 `IntentService` 的实现示例：

```
public class HelloIntentService extends IntentService {
/**
* 必须提供一个构造函数，并且必须调用super `IntentService(String)` 构造函数，并为工作线程指定一个名称。
*/
public HelloIntentService() {
super("HelloIntentService");
}
/**
* IntentService会在默认的工作线程中调用此方法，并传入启动服务的Intent。
* 当此方法返回时，IntentService会根据需要停止服务。
*/
@Override
protected void onHandleIntent(Intent intent) {
// 通常在这里会执行一些工作，比如下载文件。
// 这个示例中，我们只是睡眠5秒钟。
try {
Thread.sleep(5000);
} catch (InterruptedException e) {
// 恢复中断状态。
Thread.currentThread().interrupt();
}
}
}
```

只需要一个构造函数和一个 `onHandleIntent()` 实现即可。

如果还决定要重写其他回调方法（如 `onCreate()`、`onStartCommand()` 或 `onDestroy()`），需要确保调用超类（父类）实现，以便于 `IntentService` 能够妥善处理工作线程的生命周期。

例如，`onStartCommand()` 必须返回默认实现，即如何将 `Intent` 传递给 `onHandleIntent()`：

```
@Override
public int onStartCommand(Intent intent, int flags, int startId) {
Toast.makeText(this, "service starting", Toast.LENGTH_SHORT).show();
return super.onStartCommand(intent,flags,startId);
}
```

除 `onHandleIntent()` 之外，唯一一个无需从中调用超类的方法就是 `onBind()`。只有在服务允许绑定时，才需要实现该方法。

### 扩展服务类

借助 `IntentService`，是可以非常轻松地实现启动服务。但是，若要求服务执行多线程（而非通过工作队列处理启动请求），则可通过扩展 `Service` 类来处理每个 `Intent`。
