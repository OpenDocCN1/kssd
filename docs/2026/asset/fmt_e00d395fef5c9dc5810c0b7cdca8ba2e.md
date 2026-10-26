
调用的是`openConnectDialog()`，这个在java端的`aggressor/bridges/AggressorBridges.java`中进行判断，如果点击了这个，则调用这个文件中的代码。`AggressorBridges.java`先是注册，再是响应：

注册：

![图片描述](upload/attach/202310/951654\_B4GH9Y2RKV8A9CW.webp)

响应：

![图片描述](upload/attach/202310/951654\_XBN49BS7VDBEM4Q.webp)

最后执行的代码为：

```java
(new ConnectDialog(this.window)).show();
```

这个代码是在`aggressor/dialogs/ConnectDialog.java`中的类。实际上aggressor界面上的那些按钮的触发都是这个套路，因此我们后续如果不想用纯粹的cna脚本，就可以用cna脚本触发，函数写死在java端，好处就是把一些不改动的cna脚本写死在里面，不用拷贝给队友时还需要额外附带脚本。cna脚本写死并加载的地方在`aggressor/AggressorClient.java`中：

![图片描述](upload/attach/202310/951654\_MFHB2XY6UH9YQ6Z.webp)

# aggressor 提取监听器信息

这个没什么绕的地方，主要就是分析代码，我直接贴提取方式了：

```java
String lname = DialogUtils.string(var2, "listener"); // 监听器名称，通过 lname 可获得 lhost 和 lport
String lhost = ListenerUtils.getListener(this.client, lname).getCallbackHosts(); // 监听器IP
String lport = String.valueOf(ListenerUtils.getListener(this.client, lname).getPort()); // 监听器端口
```

我之所以弄这个，是之前想将免杀模块放在本地来进行操作，后来这个方案被我废弃了，打算将免杀流程放在teamserver端进行操作，再回传至aggressor。

# aggressor 发送信号

发送信号我是分析的钓鱼模块，aggressor交互都在`aggressor/dialogs`里面，发送信号使用的是`TeamQueue`类，CS的通信主要用的就是这个类：

```java
protected TeamQueue conn = null;
this.conn.call("SendSign.test", CommonUtils.args(var1, var2, var3, checksum));
```

其中`"SendSign.test"`代表的是发送数据的标签，服务端需要根据这个标签进行判断，后面的几个都是传递的参数。我自己实现的调用逻辑为：

`dialogAction() -> send() -> final_send()`

```java
public void dialogAction(ActionEvent var1, Map var2) {
String lname = DialogUtils.string(var2, "listener"); // 监听器名称，通过 lname 可获得 lhost 和 lport
String lhost = ListenerUtils.getListener(this.client, lname).getCallbackHosts(); // 监听器IP
String lport = String.valueOf(ListenerUtils.getListener(this.client, lname).getPort()); // 监听器端口
String[] stringArray = {lname,lhost,lport};
this.send(var1, var2, stringArray); // var1 与 var2 固定，var3 传递传给 final_send 的参数
}
public void send(ActionEvent var1, Map var2,String[] var3) {
this.final_send(var3[0],var3[1],var3[2]);
}
private void final_send(String var1,String var2,String var3) {
this.conn.call("SendSign.test", CommonUtils.args(var1, var2, var3, checksum));
System.out.println("发送的checksum为："+ checksum);
}
```

# server 接收信号

与`dialog.java`成对存在的就是server目录中的文件，我这自己新建了一个类，具体接收信号的函数为：

```java
public void call(Request var1, ManageUser var2) { // 实际运行的函数
String var4;
if (var1.is("SendSign.test", 4)) { // 前三个参数跟编译相关，第四个为 checksum 校验值
synchronized(this) {
int result = genCrossC2((String) var1.arg(0), (String) var1.arg(1), (String) var1.arg(2)); // 打印输入的参数，需要将 object 强制转为 string
CommonUtils.print_info("result: "+ result);
if (result == 0){
this.resources.broadcast("genCrossC2", (String) var1.arg(3), true);
System.out.println("send checksum: "+ (String) var1.arg(3));
}
}
}
}
```

其中比较重要的就是`var1.is("SendSign.test", 4)`，这个与aggressor传来的数据是相对应的，4为四个参数。

# server 进行广播

广播的代码为：

```java
this.resources.broadcast("genCrossC2", (String) var1.arg(3), true);
```

这个true的含义貌似是aggressor重连后还会接收到这个信息，我改为false也没什么变化，广播的数据为`Map`类型。

# aggressor 接收广播

接收广播部分有个巨坑，如果直接使用`TeamQueue`提供的`setSubscriber`回调函数来接收广播，则会导致CS的UI不会刷新。存在问题的代码为：

```java
conn.setSubscriber(new Callback() {
@Override
public void result(String call, Object content) { //
if ("genCrossC2".equals(call)) {
System.out.println("接收到 genCrossC2 广播！");
if (content instanceof String) {
String contentString = (String) content;
if ("OK".equals(contentString)) {
System.out.println("广播的内容是 'OK'！");
}
}
}
}
});
```

这个代码虽然可以正确接收，但是UI不刷新肯定是存在问题的，后来换了一个方案：

```java
this.conn.setSubscriber(this.data);
String response = data.getDataSafe("genCrossC2").toString();
if(checksum.equals(response)){
System.out.println("接收到服务端广播的内容是：" + checksum);
break;
}
```

在使用这个时，一定要注意是否`data`中存在这个键值对，如果不存在就会空指针异常，因此我在`dialogAction`中去检索`data`数据前先使用`this.data.put("key","value");`进行了置空操作，问题就解决了。

# 总结

以上是基于下面的模型进行讲解的，掌握这个模型一系列操作方式后CS的可玩性还是非常高的。

![图片描述](upload/attach/202310/951654\_3X5Q7PZXKY7VT9D.webp)

本人也是刚刚学习java开发，文中可能存在诸多问题，欢迎大家指出，最后祝大家国庆快乐~

# 前言

本文为CS 4.4 二开笔记系列第二篇，主要是针对aggressor客户端进行功能分析与二开。感觉现在的查到的文章较多在于beacon的协议的分析和针对特征的修改，在aggressor的分析还是比较少的，而aggressor又是使用者进行一系列操作的入口，只掌握sleep来编写cna脚本是远远不够的，因此本文将着重讨论针对aggressor功能的分析与扩充，文中可能会对部分步骤进行省略，主要是提供足够的思路来分析aggressor的源码和增强。

# Event Log 显示连接的 teamserver 的 IP

之前在实战中遇到的问题是经常忘了连接的是哪台服务器的teamserver，后来分析了一下cscat 4.5的源码，他的设置方式为通过aggressor进行设置，因为我在开发时没有打算在aggressor处设置配置文件，因此使用了不同的方式实现显示，先看一下效果：

![图片描述](https://bbs.kanxue.com/upload/attach/202310/951654_33RG3YR3CXBKFKW.webp)

此处涉及到cna脚本的编写及aggressor的源码，cna脚本的编写可以参考下面的官方链接及狼组的中文文档，本文中不再赘述：

[cs插件开发 - 先知社区](https://bbs.kanxue.com/elink@934K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6^5P5W2)9J5k6h3q4D9K9i4W2#2L8W2)9J5k6h3y4G2L8g2)9J5c8Y4c8Q4x3V1j5I4x3e0b7H3y4l9`.`.)

[Aggressor-Script | 狼组安全团队公开知识库](https://bbs.kanxue.com/elink@0afK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4K9h3E0A6i4K6u0W2N6$3N6H3M7$3g2U0i4K6u0W2L8%4u0Y4i4K6u0r3K9$3&6G2N6$3I4W2k6r3N6W2i4K6u0r3K9h3&6@1M7X3q4F1k6i4c8Q4x3V1k6m8k6$3N6J5k6i4y4K6L8%4u0Q4x3X3c8K6j5%4u0A6M7s2c8Q4x3X3g2Z5N6r3#2D9)

[d9dK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Z5M7%4c8W2j5$3S2V1L8$3y4K6i4K6u0W2K9r3g2D9M7s2y4&6M7%4c8W2L8i4y4Q4x3X3g2U0L8$3#2Q4x3V1k6E0j5h3&6#2j5h3I4K6i4K6u0r3j5$3!0T1j5h3I4@1M7%4c8J5K9h3E0W2i4K6u0r3j5%4g2J5M7X3g2F1N6q4)9J5c8Y4g2K6k6i4u0Y4N6h3W2V1k6g2)9J5c8X3y4G2L8Y4c8W2L8Y4c8Q4x3V1k6@1L8%4m8A6j5%4y4Q4y4h3k6S2k6$3N6J5k6i4y4K6L8%4u0Q4x3X3c8K6j5%4u0A6M7s2c8K6i4K6u0r3j5i4y4Q4y4h3k6U0L8$3u0S2L8s2c8Q4x3X3c8K6N6s2u0A6K9$3g2Q4x3X3g2Z5N6r3@1`.)

cna脚本会在点击时检索脚本中是否存在相应的函数，如果不存在则进行函数提交，交给java端进行判断及相应，在这里我修改了默认的`default.cna`脚本，新增了`serverip()`函数来进行显示：

![图片描述](https://bbs.kanxue.com/upload/attach/202310/951654_T7RVVE6E478X7VV.webp)

这个函数的实现并不在cna脚本中，因此在触发时脚本会向aggressor发送`serverip`这个字符串，然后aggressor会判断是否等于这个字符串来进行响应。aggressor的代码与下面两部分有关：

1. 注册

2. 响应

注册在`DataBridge.java`的`scriptLoaded()`函数中，需要先注册这个函数的键值：

![图片描述](https://bbs.kanxue.com/upload/attach/202310/951654_4MVE5CF8EYPRWW4.webp)

响应在`DataBridge.java`的`evaluate()`函数中，当接收到这个键值传来信号时则做出响应：

![图片描述](https://bbs.kanxue.com/upload/attach/202310/951654_GUP94Q3KK9G8GRN.webp)

`aggressorRemoteIp`这个字符串是我在`connect.java`中设置的`public static`变量，当我们在connect界面连接teamserver时则将对应的host传递给event log，最后达到了我们想要的效果：

![图片描述](https://bbs.kanxue.com/upload/attach/202310/951654_DH27W2939MMAHDM.webp)

# 钉钉上线提醒

钉钉提醒网上常用的方案为调用`agscript`来执行python操作，再进行提醒。我感觉这样过于麻烦，因此直接将此功能集成至server端中，只需要在server端的配置文件中填入server端的配置文件，就可以默认启动，类似

![图片描述](https://bbs.kanxue.com/upload/attach/202310/951654_XZXPBDFB9HWK33V.webp)

server端日志显示：

![图片描述](https://bbs.kanxue.com/upload/attach/202310/951654_WVFQQUY3M8H6WFS.webp)

创建方法如下：

新建一个类：

![图片描述](https://bbs.kanxue.com/upload/attach/202310/951654_W536P3D9J5N6B7V.webp)

接着在`server/Beacons.java`中调用这个类，我这里是通过配置文件进行的设置，其他方式大家可以自己研究：

![图片描述](https://bbs.kanxue.com/upload/attach/202310/951654_6R2GR9ZPVXUEEZP.webp)

上线效果就是这样：

![图片描述](https://bbs.kanxue.com/upload/attach/202310/951654_H2KTZ7K6SFW26KT.webp)

# 第一次上线时间显示

这个功能需要涉及到数据库的保存，这里使用的是jdbc，效果如下：

![图片描述](https://bbs.kanxue.com/upload/attach/202310/951654_P75VEHQVFN9JCWP.webp)

在新增这个功能时，发现了cscat 4.5的一个bug，就是无法正确的显示`last`，后来发现主要原因在于`server/Beacons.java`的`checkin`函数循环导致无法正确刷新map。目前更改后的`checkin`代码如下：

|  |  |

| --- | --- |
