
```java
if (Java.available) {
    Java.perform(function () {
        var ay_class = Java.use("com.tencent.mm.plugin.sns.model.ay");
        var desc = "To be, or not to be, that is a question.";
        var ayInstance = ay_class.$new(2);
        ayInstance.adk(desc);
        ayInstance.commit();
    });
}
```

文字动态的内容是：`To be, or not to be, that is a question.`。果不其然，脚本运行后，文字动态发表成功了。

![](upload/attach/202003/815293_Z57PK642S4T8Q6K.jpg)

经过对`com.tencent.mm.plugin.sns.ui.ae`的`a`方法的分析，我们可以知道，`a`方法主要传入一些发朋友圈动态所需要的通用的数据，比如动态是否私密，动态的文字描述，艾特的人，定位信息等，这些信息在其他类型的朋友圈动态中也会用得到。我们还知道发文字动态只需要文字描述就能发表成功。

### 0x3 带图片的朋友圈动态分析

带图片的的动态和文字动态差不多，只是多加了图片的参数，我们在分析此类动态时多关注图片在哪传入即可。带图片的动态对应的类是`com.tencent.mm.plugin.sns.ui.ai`，有了上面的经验，我们直接去看它的`a`方法（`ai`类有许多`a`方法，注意这里说的`a`方法参数和`com.tencent.mm.plugin.sns.ui.z`接口里的`a`方法参数一致）。

在`a`方法的开头，看到利用迭代器去遍历一个列表，遍历过程中组装`com.tencent.mm.plugin.sns.data.j`类的数据，然后把`j`类放入链表`linkedList2`中：

![](upload/attach/202003/815293_EFUYEPYFMK2FB3C.jpg)

在组装数据的时候，我们看到`j`类构造时传入一个字符串和数字，而这个字符串对应`j`类的`path`字段，这可能就是图片的路径：

![](upload/attach/202003/815293_F5MYAERN8DBETQN.jpg)

那么我们猜测`j`类就是存储朋友圈的动态图片信息的类，上面提到`j`类被放入链表`linkedList2`中，那么来看`linkedList2`被哪里引用了：

![](upload/attach/202003/815293_JHN6JR3M3U4VB8E.jpg)

看到醒目的字符串：`commit pic size`，这应该是日志要打印的字符串，现在基本上可以确定`j`类就是存储要发表的图片的信息的类了，那么`linkedList2`就是存储所有将要发表的图片信息，继续往下寻找`linkedList2`还被哪里引用了：

![](upload/attach/202003/815293_9MJ5CWS96FDDXRD.jpg)

可以看到`linkedList2`传入两个地方，一处传入`a`方法：

![](upload/attach/202003/815293_DBGTR9ABJYH8JQ9.jpg)

另一处传入`com.tencent.mm.plugin.sns.ui.ai$a`类构造函数：

![](upload/attach/202003/815293_MUJ9CT7R44FU3AS.jpg)

`linkedList2`传入`a`类后，又赋值给成员变量`tPF`，这个`tPF`成员变量只在`a`类的`dU`方法中被引用：

![](upload/attach/202003/815293_WAYNECVWHATUN59.jpg)

而`dU`方法在哪里调用呢？在`a`类的父类：`com.tencent.mm.plugin.sns.model.h`中，我们看到`dU`方法在`u`方法被调用：

![](upload/attach/202003/815293_RXXZ8P7Z69NAPFR.jpg)

而`u`方法在`ai`类的`a`方法中调用（可以回看前面的图）。分析到这，上面的`linkedList2`传出去之后都终有所属了，即最终都传入了`com.tencent.mm.plugin.sns.model.ay`类`ey`方法。知道图片往哪传了，就写段`frida`代码调用试试吧。

```javascript
if (Java.available) {
    Java.perform(function () {
        var ay_class = Java.use("com.tencent.mm.plugin.sns.model.ay");
        var j_class = Java.use("com.tencent.mm.plugin.sns.data.j");
        var desc = "To be, or not to be, that is a question.";
        var linkedList_class = Java.use("java.util.LinkedList");
        var linkedListInstance = linkedList_class.$new();
        var ayInstance = ay_class.$new(1);
        var jInstance1 = j_class.$new("/storage/emulated/0/test1.jpg", 2);
        var jInstance2 = j_class.$new("/storage/emulated/0/test2.jpg", 2);
        var jInstance3 = j_class.$new("/storage/emulated/0/test3.jpg", 2);

        linkedListInstance.add(jInstance1);
        linkedListInstance.add(jInstance2);
        linkedListInstance.add(jInstance3);
        ayInstance.ey(linkedListInstance);
        ayInstance.adk(desc);
        ayInstance.commit();
    });
}
```

上面的代码在发送文本动态代码的基础上初始化三个`j`类，分别传入三个本地图片路径，再将三个类实例添加到链表，再将链表传入`ay`类的`ey`方法，最后调用`ay`类的`commit`方法将动态发送出去，代码运行，发现带图片的朋友圈动态发表成功：

![](upload/attach/202003/815293_QTS5PNQBQJT66QC.jpg)

### 0x0 前言

最近一直在研究Windows逆向的东西，想着快要把Android给遗忘了。所以就想利用工作之余来研究Android相关的技术，来保持对Android热情。调用微信代码来发送朋友圈动态一直是自己想实现的东西，研究了一下，果然实现了，遂写下本文当作记录。本文主要分析发送纯文字朋友圈动态和发送图片朋友圈动态。

### 0x1 朋友圈动态类型分析

本文用到的工具如下：

- PC
- 一台可以调试微信进程的Android手机
- 微信7.0.11
- `ddms`（用于跟踪调用过程）
- `uiautomatorviewer`（用于定位控件id）
- `jadx`（用于对微信apk静态分析）
- `frida`（用于hook微信，获得相关信息，发布朋友圈）

在分析代码之前，首先要定位到与之相近的地方，我们首先想到的肯定是发朋友圈那个界面，如何查看发朋友圈的界面是哪个Activity呢？很简单，在手机上打开发送朋友圈的界面。

![](https://bbs.kanxue.com/upload/attach/202003/815293_2HN4TK7SRS2ENFR.jpg)

把手机连接电脑，打开USB调试，在PC的`cmd`窗口中执行以下命令：

```bash
adb shell dumpsys activity top
```

![](https://bbs.kanxue.com/upload/attach/202003/815293_HBXHDZJMRUBF3N9.jpg)

可以看到发朋友圈的那个Activity就是`com.tencent.mm.plugin.sns.ui.SnsUploadUI`。

虽然找到了Activity，但还是不能高兴太早，想要通过Activity知道哪部分是发朋友的代码还是比较费力的。于是我们就想到从“发表”按钮入手，找出发表朋友圈的相关代码。点击“发表”按钮会发生什么？发表是一个动态的行为，我们可以通过跟踪点击“发表”按钮时的调用过程，来找到有用的信息。跟踪调用过程，可以使用`ddms`工具来完成。

打开`ddms`，选中微信进程，在手机中打开发表朋友圈界面，然后在`ddms`中点击下图圈出的图标开始跟踪：

![](https://bbs.kanxue.com/upload/attach/202003/815293_PNPAC3BHM43QCY6.jpg)

将朋友圈动态发出，再点一次上图圈出的图标停止跟踪。`ddms`会生成跟踪结果，对于跟踪结果，怎么找到按钮事件相关的信息呢，学过Android的朋友就会想到`onClick`方法，那我们就在`ddms`的搜索结果中搜索这个名称：

![](https://bbs.kanxue.com/upload/attach/202003/815293_Q9ZSADEV5CWNJ2E.jpg)

成功的定位到了`onClick`的位置，但是比起这条`onClick`结果，更加令人引人注目的是它的上一条结果，因为它包含了我们刚才找到的Activity的类名：

![](https://bbs.kanxue.com/upload/attach/202003/815293_5ESNJJKA7ZYMPFH.jpg)

知道这个方法被调用，我们去看看`com.tencent.mm.plugin.sns.ui.SnsUploadUI`类里的`OnMemuItemClick`究竟是什么。

用`jadx`打开微信apk，定位到`com.tencent.mm.plugin.sns.ui.SnsUploadUI`类，在类中搜索`onMemuItemClick`，结果不多，看起来比较像的就是这个`onMemuItemClick`了：

![](https://bbs.kanxue.com/upload/attach/202003/815293_XXMSZD2QXG8QVKD.jpg)

在`onMemuItemClick`方法中看到了：

```java
String unused2 = SnsUploadUI.this.desc = SnsUploadUI.this.tQN.getText().toString();
```

这行代码有什么特别的呢，在我看来，有两个特别的地方：

- `desc`是描述（description）的英文单词的缩写。
- `this.desc`被赋予`this.tQN.getText().toString()`。

我们发朋友圈动态时候，是要写动态的描述的，所以这个`desc`可能就是发朋友圈动态的描述，如果是描述，我们就可以根据这个描述，顺藤摸瓜找到发朋友圈动态的地方。而且`this.desc`的值又来自于`this.tQN.getText().toString()`，即`this.tQN`很有可能就是我们填写动态描述的文本框。

我们来看看`this.tQN`赋值的地方，它在`onCreate`方法被赋值：

![](https://bbs.kanxue.com/upload/attach/202003/815293_ARA9K8XDKRUNJ3A.jpg)

可以知道它的id是`d41`，那么`d41`是哪个控件？打开`uiautomatorviewer`，对发朋友圈界面截图分析，点击截图中的文本框，`uiautomatorviewer`右侧跳转到了相应的位置，果然，`d41`就是发动态时填写描述文本框的id。

![](https://bbs.kanxue.com/upload/attach/202003/815293_X4H6FTZNGYVU27Y.jpg)

好了，现在知道`this.desc`就是发表朋友圈动态时的描述，跟上他应该就可以找到发朋友圈动态的地方。继续往`onMemuItemClick`方法下部分析，可以看到`this.desc`被传入了`SnsUploadUI.this.tQO.a`方法：

![](https://bbs.kanxue.com/upload/attach/202003/815293_F2BRJB5M46BT2NJ.jpg)

`SnsUploadUI.this.tQO.a`方法定义在接口`com.tencent.mm.plugin.sns.ui.z`中：

![](https://bbs.kanxue.com/upload/attach/202003/815293_WUG4JGVRPQACPY9.jpg)

![](https://bbs.kanxue.com/upload/attach/202003/815293_DYAX26624QMD98Z.jpg)

知道它定义在哪个接口并不能解决问题，毕竟接口没有实质性代码，要找还得找接口的的实现类，在`com.tencent.mm.plugin.sns.ui.SnsUploadUI`类中寻找`this.tQO`在哪里会被赋值。最终，我们在`com.tencent.mm.plugin.sns.ui.SnsUploadUI`类的`ag`方法中看到了许多给`this.tQO`赋值的地方：

![](https://bbs.kanxue.com/upload/attach/202003/815293_58VZK58KEUE5KCW.jpg)

由此，可见`this.tQO`被赋予什么值是根据`this.tMY`来决定的，`this.tMY`是一个int类型的数据，那我们hook `com.tencent.mm.plugin.sns.ui.SnsUploadUI`类的`ag`方法就可以知道`this.tMY`是什么值。在这里，我用`frida`来hook，`frida`的JavaScript部分代码如下：

```javascript
var SnsUploadUI = Java.use('com.tencent.mm.plugin.sns.ui.SnsUploadUI');

var ag = SnsUploadUI.ag.overload("android.os.Bundle");

// get sns type
ag.implementation = function (bundle) {
    var ret = ag.call(this, bundle);

    send("sns type = " + this.tMY.value);

    return ret;
};
```

hook之后，每当我们在手机上打开发布朋友圈动态的界面，`ag`方法被调用，控制台就会输出相应的数字。经过我的测试，这个数字是发表朋友圈动态的类型。朋友圈类型和其对应类如下：

- 0 带图片的动态，对应：`com.tencent.mm.plugin.sns.ui.ai`
- 9 纯文字动态，对应：`com.tencent.mm.plugin.sns.ui.ae`

这些类都直接或间接的实现了上面讲到的`com.tencent.mm.plugin.sns.ui.z`接口。这样一来，就知道`this.tQO`会根据朋友圈的动态类型进行初始化，那么，上面的`SnsUploadUI.this.tQO.a`方法很有可能就是发朋友圈动态的方法。接下来，我们根据不同的朋友圈动态所对应的类来分别分析。

### 0x2 文字动态分析

文字动态分析起来应该比图片动态来说简单一些，我们就先来分析它。上面讲到，这类动态对应类是`com.tencent.mm.plugin.sns.ui.ae`，这个类里我们主要看`a`方法，在看`a`方法之前，先看它传入什么参数，为了看清楚，这就要回看上文`onMenuItemClick`方法调用`a`方法的地方：
