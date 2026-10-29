
我们已经学会了如何接收广播，接下来我们需要了解开发者在开发安卓APP时如何发送广播。广播是一种事件通知机制，开发者可以通过发送广播来通知其他应用程序或组件发生了某个事件。发送广播时，开发者会创建一个`Intent`对象，并指定一个特定的`Action`标识符，表示要发送的广播类型。其他应用程序或组件可以通过注册对应的广播接收器，并指定相同的`Action`标识符来接收该广播。当某个事件触发时，安卓系统会将该广播发送给所有注册了对应`Action`的广播接收器，接收器可以根据接收到的广播类型进行相应的处理。广播的内容可以包含额外的数据，可以通过`Intent`的`putExtra`方法来添加。

### Android 发送广播的三种方式

Android 为应用提供三种方式来发送广播，以下是安卓开发者官方文档关于发送广播的阐述：

- `sendOrderedBroadcast(Intent, String)` 方法一次向一个接收器发送广播。当接收器逐个顺序执行时，接收器可以向下传递结果，也可以完全中止广播，使其不再传递给其他接收器。接收器的运行顺序可以通过匹配的 `intent-filter` 的 `android:priority` 属性来控制(`android:priority` 属性表示等级，值是-1000到1000，默认是0)；具有相同优先级的接收器将按随机顺序运行。

- `sendBroadcast(Intent)` 方法会按随机的顺序向所有接收器发送广播。这称为常规广播。这种方法效率更高，但也意味着接收器无法从其他接收器读取结果，无法传递从广播中收到的数据，也无法中止广播。

- `LocalBroadcastManager.sendBroadcast` 方法会将广播发送给与发送器位于同一应用中的接收器。如果您不需要跨应用发送广播，请使用本地广播。这种实现方法的效率更高（无需进行进程间通信），而且您无需担心其他应用在收发您的广播时带来的任何安全问题。

以下代码段展示了如何通过创建 `Intent` 并调用 `sendBroadcast(Intent)` 来发送广播。

```
Intent intent = new Intent();
intent.setAction("com.example.broadcast.MY_NOTIFICATION");
intent.putExtra("data","Notice me senpai!");
sendBroadcast(intent);
```

广播消息封装在 `Intent` 对象中。`Intent` 的操作字符串必须提供应用的 Java 软件包名称语法，并唯一标识广播事件。您可以使用 `putExtra(String, Bundle)` 向 `intent` 附加其他信息。您也可以对 `intent` 调用 `setPackage(String)`，将广播限定到同一组织中的一组应用。

**注意**：虽然 `intent` 既用于发送广播，也用于通过 `startActivity(Intent)` 启动 `Activity`，但这两种操作是完全无关的。广播接收器无法查看或捕获用于启动 `Activity` 的 `intent`；同样，当您广播 `intent` 时，也无法找到或启动 `Activity`。

以上就是安卓开发者官方文档对发送广播的阐述，这里讲的还是算通俗易懂的。然而除了发送和接收广播，开发者还可以通过权限限制广播，下面就讲讲开发者是怎么通过权限限制广播的：

开发者可以通过权限将广播限定到拥有特定权限的一组应用。开发者可以对广播的发送器或接收器施加限制。那么接下来就讲讲开发者是如何带权限的发送广播和带权限的接收广播的。

### 带权限发送广播

开发者想要带权限的发送广播，在调用 `sendBroadcast(Intent, String)` 或 `sendOrderedBroadcast(Intent, String, BroadcastReceiver, Handler, int, String, Bundle)` 时，可以指定权限参数。当开发者带权限的发送广播后，如果广播接收器希望接收此广播，则必须通过其应用清单文件中的标记请求该权限（如果存在危险，则会被授予该权限）。换句话说，通过指定权限参数，开发者可以确保只有具有相应权限的接收器才能接收到该广播。例如，以下代码会带权限的发送广播：

```
sendBroadcast(new Intent("com.example.NOTIFY"),
Manifest.permission.SEND_SMS);
```

要接收此广播，接收方应用必须请求如下权限：

```
<uses-permission android:name="android.permission.SEND_SMS"/>
```

开发者不仅可以指定现有的系统权限（如 `SEND_SMS`），也可以使用 `<permission>` 元素定义自定义权限，自定义权限格式为“`包名.自定义的权限名`”，权限名一般都是大写，跟 `action` 的格式是一样的。

还有一点要注意：自定义权限将在安装应用时注册。定义自定义权限的应用必须在使用自定义权限的应用之前安装。

### 带权限接收广播

现在知道了开发者是如何带权限的发送广播的，那开发者又是如何带权限的接收广播的就是下面所要讲的内容。

如果开发者在注册广播接收器时指定了权限参数（通过 `registerReceiver(BroadcastReceiver, IntentFilter, String, Handler)` 或清单中的 `<receiver>` 标记指定），则发送广播的一方必须通过其应用清单文件中的 `<uses-permission>` 标记请求该权限（如果存在危险，则会被授予该权限），才能向该接收器发送 `Intent`。

注意：一旦在发送广播的应用程序的清单文件中通过 `<uses-permission>` 标记请求了某个权限，即使该权限被认为是危险的，也会被系统授予该权限。

想要带权限的接收广播可以这样做，例如，假设开发者的接收方应用具有如下所示的清单声明的接收器：

```
<receiver android:name=".MyBroadcastReceiver"
android:permission="android.permission.SEND_SMS">
<intent-filter>
<action android:name="android.intent.action.AIRPLANE_MODE"/>
</intent-filter>
</receiver>
```

或者开发者的接收方应用具有如下所示的上下文注册的接收器：

```
IntentFilter filter = new IntentFilter(Intent.ACTION_AIRPLANE_MODE_CHANGED);
registerReceiver(receiver, filter, Manifest.permission.SEND_SMS, null );
```

那么，发送方应用必须请求如下权限，才能向这些接收器发送广播：

```
<uses-permission android:name="android.permission.SEND_SMS"/>
```

以上内容都参考了安卓开发者官方文档中对广播的介绍，而关于广播接收器组件的最后就以官方文档中有关收发广播的一些安全注意事项和最佳做法作为结尾。

### 收发广播的安全注意事项和最佳做法

以下是有关收发广播的一些安全注意事项和最佳做法：

- 如果您不需要向应用以外的组件发送广播，则可以使用支持库中提供的 `LocalBroadcastManager` 来收发本地广播。`LocalBroadcastManager` 效率更高（无需进行进程间通信），并且您无需考虑其他应用在收发您的广播时带来的任何安全问题。本地广播可在您的应用中作为通用的发布/订阅事件总线，而不会产生任何系统级广播开销。

- 如果有许多应用在其清单中注册接收相同的广播，可能会导致系统启动大量应用，从而对设备性能和用户体验造成严重影响。为避免发生这种情况，请优先使用上下文注册而不是清单声明。有时，Android 系统本身会强制使用上下文注册的接收器。例如，`CONNECTIVITY_ACTION` 广播只会传送给上下文注册的接收器。

- 请勿使用隐式 `intent` 广播敏感信息。任何注册接收广播的应用都可以读取这些信息。您可以通过以下三种方式控制哪些应用可以接收您的广播：

  - 您可以在发送广播时指定权限。
  - 在 Android 4.0 及更高版本中，您可以在发送广播时使用 `setPackage(String)` 指定软件包。系统会将广播限定到与该软件包匹配的一组应用。
  - 您可以使用 `LocalBroadcastManager` 发送本地广播。

- 当您注册接收器时，任何应用都可以向您应用的接收器发送潜在的恶意广播。您可以通过以下三种方式限制您的应用可以接收的广播：

  - 您可以在注册广播接收器时指定权限。
  - 对于清单声明的接收器，您可以在清单中将 `android:exported` 属性设置为“false”。这样一来，接收器就不会接收来自应用外部的广播。
  - 您可以使用 `LocalBroadcastManager` 限制您的应用只接收本地广播。

- 广播操作的命名空间是全局性的。请确保在您自己的命名空间中编写操作名称和其他字符串，否则可能会无意中与其他应用发生冲突。

- 由于接收器的 `onReceive(Context, Intent)` 方法在主线程上运行，因此它会快速执行并返回。如果您需要执行长时间运行的工作，请谨慎生成线程或启动后台服务，因为系统可能会在 `onReceive()` 返回后终止整个进程。如需了解详情，请参阅对进程状态的影响。要执行长时间运行的工作，我们建议：

  - 在接收器的 `onReceive()` 方法中调用 `goAsync()`，并将 `BroadcastReceiver.PendingResult` 传递给后台线程。这样，在从 `onReceive()` 返回后，广播仍可保持活跃状态。不过，即使采用这种方法，系统仍希望您非常快速地完成广播（在 10 秒以内）。为避免影响主线程，它允许您将工作移到另一个线程。
  - 使用 `JobScheduler` 调度作业。如需了解详情，请参阅智能作业调度。

- 请勿从广播接收器启动 `Activity`，否则会影响用户体验，尤其是有多个接收器时。相反，可以考虑显示通知。

### Content Provider

`Content Provider` 是 Android 四大组件之一，内容提供者对中央数据存储区的访问。一般会在以下两种场景中使用内容提供者：一种是通过实现代码访问其他应用中的现有内容提供者；另一种是在应用中创建新的内容提供者，从而与其他应用共享数据。

对于内容提供者还是先从基础开始了解，我们先从宏观的角度上来看看内容提供者的执行流程是什么样的，内容提供者以一个或多个表的形式将数据呈现给外部应用，这些表与关系型数据库中的表类似。行表示内容提供者收集的某种类型数据的实例，行中的每一列表示为一个实例所收集的单个数据。内容提供者协调很多不同的 API 和组件对应用数据存储层的访问，以便实现以下功能：

- 与其他应用共享对应用数据的访问
- 向微件发送数据
- 使用 `SearchRecentSuggestionsProvider`，通过搜索框架返回对应用的自定义搜索建议
- 通过实现 `AbstractThreadedSyncAdapter`，将应用数据与服务器同步
- 使用 `CursorLoader` 在界面中加载数据

以下是官方给出的内容提供者与其他组件的关系：

![](upload/attach/202311/926486_YMWNRWFWA2BUMXT.webp)

如果想要访问内容提供者当中的数据，那么开发者会以客户端的形式使用应用的 `Context` 中的 `ContentResolver` 对象与内容提供者进行通信。`ContentResolver` 对象会与内容提供者的实例对象通信。内容提供者的实例对象从客户端接收数据请求、执行请求的操作并返回结果。此对象的某些方法可调用 `ContentProvider` 某个具体子类的实例中的同名方法。`ContentResolver` 方法可以提供持久性存储空间的基本增删改查功能。

一般使用界面去访问内容提供者最常用的模式是使用 `CursorLoader` 在后台运行异步查询。在开发者的设计下界面中的 `Activity` 会调用查询的 `CursorLoader`，而 `CursorLoader` 会转而使用 `ContentResolver` 获取 `ContentProvider`。这样程序就可以在用户使用界面的同时进行查询数据了。

以下是官方给出的内容提供者、其他类和存储空间之间的交互：

![](upload/attach/202311/926486_SGR627MYR8B6R2Z.webp)

开发者想要应用访问内容提供者，那么开发者通常需要在其清单文件中请求特定权限。安卓平台内置的内容提供者之一是用户字典，用于存储用户想要保存的非标准字词的拼写。这种内容提供者之一的用户字典其表中的数据可能是这样的：

| 字词 | 应用 ID | 频率 | 语言区域 | `_ID` |
| :---------- | :------ | :--- | :------- | :--- |
| mapreduce | user1 | 100 | zh_CN | 1 |
| precompiler | user14 | 200 | fr_FR | 2 |
| applet | user2 | 225 | fr_CA | 3 |
| const | user1 | 255 | pt_BR | 4 |
| int | user5 | 100 | en_UK | 5 |

在上表中，每行表示可能无法在标准字典中找到的字词实例。每列表示该字词的一些数据，如该字词首次出现时的语言区域。列标题是存储在内容提供者当中的列名称。其中 `_ID` 列是该内容提供者自动维护的“主键”列。

开发者要从用户字典内容提供者当中获取字词及其语言区域的列表就需要调用 `ContentResolver.query()`。`query()` 方法会调用用户字典内容提供者所定义的 `ContentProvider.query()` 方法。以下是调用该方法的演示：

```
// 查询用户字典并返回结果
cursor = getContentResolver().query(
UserDictionary.Words.CONTENT_URI, // words表的内容URI
projection, // 为每行返回的列
selectionClause, // 选择标准
selectionArgs, // 选择标准
sortOrder); // 返回行的排序顺序
```

下表是 `ContentProvider.query()` 方法的详细参数以及如何匹配 SQL `SELECT` 语句：

| query() 参数 | SELECT 关键字/参数 | 备注 |
| :-------------- | :---------------------------------------------------------- | :--------------------------------------------------- |
| `Uri` | `FROM *table_name*` | `Uri` 映射至内容提供者中名为 `table_name` 的表。 |
| `projection` | `*col,col,col,...*` | `projection` 是检索到的每个行所应包含的列的数组。 |
| `selection` | `WHERE *col* = *value*` | `selection` 指定选择行的条件。 |
| `selectionArgs` | （没有完全等效项，选择参数会替换选择子句中的 `?` 占位符。） | |
| `sortOrder` | `ORDER BY *col,col,...*` | `sortOrder` 指定在返回的 `Cursor` 中各行的显示顺序。 |

当开发者调用客户端方法以访问内容提供者中的表时，需要用到一个名为内容 `URI` 的东西，内容 `URI` 用来在内容提供者中标识数据。在前面的代码行中需要传递给参数 `UserDictionary.Words.CONTENT_URI` 的就是包含用户字典的“字词”表的内容 `URI`。内容 `URI` 包括整个内容提供者的符号名称（其授权）和指向表的名称（路径），当我们使用 `ContentResolver` 对象执行查询操作时，它会解析出 `URI` 的授权部分，并将该授权与已知内容提供者的系统表进行比较，然后 `ContentResolver` 可以将查询参数分派给正确的内容提供者。

`ContentProvider` 使用内容 `URI` 的路径部分选择需访问的表。通常，内容提供者会为其公开的每个表显示一条路径。比如前面那个用户字典表的完整 `URI` 是：
