
# [原创]CS 4.4 二开笔记：增强篇

## 前言

本文为CS 4.4 二开笔记系列第二篇，主要是针对aggressor客户端进行功能分析与二开。感觉现在的查到的文章较多在于beacon的协议的分析和针对特征的修改，在aggressor的分析还是比较少的，而aggressor又是使用者进行一系列操作的入口，只掌握sleep来编写cna脚本是远远不够的，因此本文将着重讨论针对aggressor功能的分析与扩充，文中可能会对部分步骤进行省略，主要是提供足够的思路来分析aggressor的源码和增强。

## Event Log 显示连接的 teamserver 的 IP

之前在实战中遇到的问题是经常忘了连接的是哪台服务器的teamserver，后来分析了一下cscat 4.5的源码，他的设置方式为通过aggressor进行设置，因为我在开发时没有打算在aggressor处设置配置文件，因此使用了不同的方式实现显示，先看一下效果：

![图片描述](upload/attach/202310/951654\_33RG3YR3CXBKFKW.webp)

此处涉及到cna脚本的编写及aggressor的源码，cna脚本的编写可以参考下面的官方链接及狼组的中文文档，本文中不再赘述：

<a href="elink@9f0K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6^5P5W2)9J5k6h3q4D9K9i4W2#2L8W2)9J5k6h3y4G2L8g2)9J5c8Y4c8Q4x3V1j5I4x3e0b7H3y4l9`.`.">cs插件开发 - 先知社区</a>

<a href="elink@961K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4K9h3E0A6i4K6u0W2N6$3N6H3M7$3g2U0i4K6u0W2L8%4u0Y4i4K6u0r3K9$3&6G2N6$3I4W2k6r3N6W2i4K6u0r3K9h3&6@1M7X3q4F1k6i4c8Q4x3V1k6m8k6$3N6J5k6i4y4K6L8%4u0Q4x3X3c8K6j5%4u0A6M7s2c8Q4x3X3g2Z5N6r3#2D9">Aggressor-Script | 狼组安全团队公开知识库</a>

<a href="elink@cc7K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Z5M7%4c8W2j5$3S2V1L8$3y4K6i4K6u0W2K9r3g2D9M7s2y4&6M7%4c8W2L8i4y4Q4x3X3g2U0L8$3#2Q4x3V1k6E0j5h3&6#2j5h3I4K6i4K6u0r3j5$3!0T1j5h3I4@1M7%4c8J5K9h3E0W2i4K6u0r3j5%4g2J5M7X3g2F1N6q4)9J5c8Y4g2K6k6i4u0Y4N6h3W2V1k6g2)9J5c8X3y4G2L8Y4c8W2L8Y4c8Q4x3V1k6@1L8%4m8A6j5%4y4Q4y4h3k6S2k6$3N6J5k6i4y4K6L8%4u0Q4x3X3c8K6j5%4u0A6M7s2c8K6i4K6u0r3j5i4y4Q4y4h3k6U0L8$3u0S2L8s2c8Q4x3X3c8K6N6s2u0A6K9$3g2Q4x3X3g2Z5N6r3@1`."><mark class="encrypted">b0bK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Z5M7%4c8W2j5$3S2V1L8$3y4K6i4K6u0W2K9r3g2D9M7s2y4&6M7%4c8W2L8i4y4Q4x3X3g2U0L8$3#2Q4x3V1k6E0j5h3&6#2j5h3I4K6i4K6u0r3j5$3!0T1j5h3I4@1M7%4c8J5K9h3E0W2i4K6u0r3j5%4g2J5M7X3g2F1N6q4)9J5c8Y4g2K6k6i4u0Y4N6h3W2V1k6g2)9J5c8X3y4G2L8Y4c8W2L8Y4c8Q4x3V1k6@1L8%4m8A6j5%4y4Q4y4h3k6S2k6$3N6J5k6i4y4K6L8%4u0Q4x3X3c8K6j5%4u0A6M7s2c8K6i4K6u0r3j5i4y4Q4y4h3k6U0L8$3u0S2L8s2c8Q4x3X3c8K6N6s2u0A6K9$3g2Q4x3X3g2Z5N6r3@1`.</mark></a>

cna脚本会在点击时检索脚本中是否存在相应的函数，如果不存在则进行函数提交，交给java端进行判断及相应，在这里我修改了默认的default.cna脚本，新增了`serverip()`函数来进行显示：

![图片描述](upload/attach/202310/951654\_T7RVVE6E478X7VV.webp)

这个函数的实现并不在cna脚本中，因此在触发时脚本会向aggressor发送`serverip`这个字符串，然后aggressor会判断是否等于这个字符串来进行响应。aggressor的代码与下面两部分有关：

1. 注册
2. 响应

注册在`DataBridge.java`的`scriptLoaded()`函数中，需要先注册这个函数的键值：

![图片描述](upload/attach/202310/951654\_4MVE5CF8EYPRWW4.webp)

响应在在`DataBridge.java`的`evaluate()`函数中，当接收到这个键值传来信号时则做出响应：

![图片描述](upload/attach/202310/951654\_GUP94Q3KK9G8GRN.webp)

`aggressorRemoteIp`这个字符串是我在`connect.java`中设置的`public static`变量，当我们在connect界面连接teamserver时则将对应的host传递给event log，最后达到了我们想要的效果：

![图片描述](upload/attach/202310/951654\_DH27W2939MMAHDM.webp)

## 钉钉上线提醒

钉钉提醒网上常用的方案为调用agscript来执行python操作，再进行提醒。我感觉这样过于麻烦，因此直接将此功能集成至server端中，只需要在server端的配置文件中填入sever端的配置文件，就可以默认启动，类似

![图片描述](upload/attach/202310/951654\_XZXPBDFB9HWK33V.webp)

server端日志显示：

![图片描述](upload/attach/202310/951654\_WVFQQUY3M8H6WFS.webp)

创建方法如下：

新建一个类：

![图片描述](upload/attach/202310/951654\_W536P3D9J5N6B7V.webp)

接着在`server/Beacons.java`中调用这个类，我这里是通过配置文件进行的设置，其他方式大家可以自己研究：

![图片描述](upload/attach/202310/951654\_6R2GR9ZPVXUEEZP.webp)

上线效果就是这样：

![图片描述](upload/attach/202310/951654\_H2KTZ7K6SFW26KT.webp)

## 第一次上线时间显示

这个功能需要涉及到数据库的保存，这里使用的是jdbc，效果如下：

![图片描述](upload/attach/202310/951654\_P75VEHQVFN9JCWP.webp)

在新增这个功能时，发现了cscat 4.5的一个bug，就是无法正确的显示last，后来发现主要原因在于`server/Beacons.java`的`checkin`函数循环导致无法正确刷新map。目前更改后的checkin代码如下：

```java
public void checkin(ScListener request, BeaconEntry var2) {
synchronized (this) {
if (!var2.isEmpty()) {
BeaconEntry var4 = (BeaconEntry) this.privateMap.get(var2.getId());
if (var4 == null || var4.isEmpty()) {
ServerUtils.addTarget(this.resources, var2.getInternal(), var2.getComputer(), (String) null, var2.getOperatingSystem(), var2.getVersion());
ServerUtils.addSession(this.resources, var2.toMap());
if (!var2.isLinked() && request != null) {
ServerUtils.addC2Info(this.resources, request.getC2Info(var2.getId()));
}
this.resources.broadcast("eventlog", LoggedEvent.BeaconInitial(var2));
this.initial.add(var2.getId());
this.resources.process(var2);
}
}
// 2023-09-19 TOP
this.Cmp = var2.getComputer();
if (var2.isSSH() && this.Cmp.contains(SVGSyntax.OPEN_PARENTHESIS)) {
this.Cmp = var2.getComputer().replace(SVGSyntax.OPEN_PARENTHESIS, "");
this.Cmp = this.Cmp.replace(")", "");
this.Cmp = this.Cmp.replace(var2.getPid(), "");
}
String BeaconHash = this.hash(var2.getInternal(), var2.getUser(), var2.getProcess(), this.Cmp, var2.getListenerName(), var2.arch(), var2.getPid());
info BeaconInfo = new info();
BeaconInfo.BeaconId = var2.getBeaconId();
BeaconInfo.Internal = var2.getInternal();
BeaconInfo.External = var2.getExternal();
BeaconInfo.Process = var2.getProcess();
BeaconInfo.Arch = var2.arch();
BeaconInfo.Computer = this.Cmp;
BeaconInfo.User = var2.getUser();
BeaconInfo.Hash = BeaconHash;
try {
Connection conn = SqliteSave.OpenDb();
try {
HashMap<String, String> beacons = SqliteSave.CheckBeaconHash(conn, BeaconHash);
if (beacons == null) { // 如果这个机器第一次上线: beacons为null，则打开数据库连接，将BeaconInfo添加到数据库中，然后尝试从beacons中获取"StartTime"并赋值给 BeaconInfo.StartTime
Connection conn4 = SqliteSave.OpenDb();
SqliteSave.AddBeacon(conn4, BeaconInfo);
var2.start = utils.BeijingTime.formatToBeijingTime();
;
// 2023-09-18 dingTalk TOP
try {
String token = TeamServer.globalDingtalkToken;
String[] args = new String[2];
args[0] = token;
args[1] = "CobaltStrike主机上线提醒+1" + "\\n";
args[1] += "计算机名：" + var2.getComputer() + "\\n";
args[1] += "IP地址：" + var2.getExternal() + "\\n";
args[1] += "归属地：" + var2.getIpAddress() + "\\n";
args[1] += "用户名：" + var2.getUser() + "\\n";
args[1] += "进程名：" + var2.getProcess() + "\\n";
args[1] += "PID：" + var2.getPid() + "\\n";
utils.DingtalkSendMsg.send(args);
} catch (IOException e) {
throw new RuntimeException(e);
}
// 2023-09-18 END
} else {
Connection conn2 = SqliteSave.OpenDb();
HashMap<String, String> beacons2 = SqliteSave.CheckBeacon(conn2, BeaconHash, var2.getBeaconId());
if (beacons2 == null) {
Connection con3 = SqliteSave.OpenDb();
BeaconInfo.StartTime = beacons.get("StartTime");
SqliteSave.AddBeacon2(con3, BeaconInfo);
} else {
var2.start = beacons2.get("StartTime");
}
}
if (conn != null) {
conn.close();
}
this.privateMap.put(var2.getId(), var2); // <-- core code 此处实时更新
// CommonUtils.print_info("[1] var2.getId(): " + var2.getId() + "\nvar2: " + var2.getLastCheckin());
} catch (Throwable th) {
if (conn != null) {
try {
conn.close();
} catch (Throwable th2) {
th.addSuppressed(th2);
}
}
throw th;
}
} catch (SQLException e) {
throw new RuntimeException(e);
}
// 2023-09-19 END
}
}
```

数据库的实现：
