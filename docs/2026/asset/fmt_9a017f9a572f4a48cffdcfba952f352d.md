
```java
if (this.isVip > 1) {

    Intent intent = new Intent((Context) this, (Class<?>) ChatActivity.class);

    intent.putExtra(UserCacheInfo.COLUMNNAME_USERIDIMID, this.user_id);

    intent.putExtra("userId", this.user_imid);

    startActivity(intent);

    return;

}
new CommomDialog(this, R.style.dialog, true, "成为会员才能私聊哦！", new CommomDialog.OnCloseListener() {
    /* class com.**.**.main.user.UserInfoActivity.AnonymousClass8 */

    /* JADX WARN: Type inference failed for: r0v0, types: [android.content.Context, com.**.**.main.user.UserInfoActivity] */
    @Override // com.u**.**.widget.dialog.CommomDialog.OnCloseListener
    public void onClick(Dialog dialog, boolean z) {
        if (z) {
            UserInfoActivity.this.startActivity(new Intent((Context) UserInfoActivity.this, MembersActivity.class));
            dialog.dismiss();
        }
    }
}).setTitle("温馨提示").setPositiveButton("开通会员").show();
return;
}
return;
```

这里可以看到`this.isVip`大于1的话就会提示“开通会员了呢”

### 最后

其实只要进入到`this.isVip`大于1那不就，，，，嘿嘿嘿

继续分析一下这个`isVip`是在哪里赋值呢。查找用例.........

![file](https://bbs.kanxue.com/upload/attach/202007/871827_CS5ZHB3RSHW8ZE8.jpg)

找到啦。。我们改下返回值

![file](https://bbs.kanxue.com/upload/attach/202007/871827_4KHBUMG3KTH44NR.jpg)

我们这里`frida`改一下。

献上代码

这里还会有个小问题，就是这块类没在内存加载的时候会报错，这时候点下某人的主页就好了。

```javascript
Java.perform(function () {
    var pre = Java.use("com.***.***.utils.UncleSharedPreferences");
    pre.getInt.overload('android.content.Context', 'java.lang.String').implementation = function (a1, a2) {
        return 2;
    }
})
```

看看效果。。。

![file](https://bbs.kanxue.com/upload/attach/202007/871827_P75PX7DCS6GH84P.jpg)

发过去了。。。

但是我们需要一直用的话就要一个`xposed`的插件。

这里`app`采用的`36*`加固，那我们就不能用常规的`classloader`进行`hook`，直接用`36*`壳的`classloader`进行`hook`。

```java
XposedHelpers.findAndHookMethod("com.stub.StubApp", loadPackageParam.classLoader, "attachBaseContext", Context.class, new XC_MethodHook() {
    @Override
    protected void afterHookedMethod(MethodHookParam param) throws Throwable {
        super.afterHookedMethod(param);
        Context context = (Context) param.args[0];
        ClassLoader classLoader = context.getClassLoader();
        classLoaders = classLoader;
        XposedHelpers.findAndHookMethod("com.***.***.utils.UncleSharedPreferences", classLoader, "getInt", Context.class, String.class, new XC_MethodHook() {
            @Override
            protected void afterHookedMethod(MethodHookParam param) throws Throwable {
                //设置返回值为2
                param.setResult(2);
            }
        });
        XposedHelpers.findAndHookMethod("com.***.***.utils.DateUtil", classLoader, "getDayDiff", Date.class, Date.class, new XC_MethodHook() {
            @Override
            protected void afterHookedMethod(MethodHookParam param) throws Throwable {
                param.setResult(0L);
            }
        });
    }
});
```

好了完美。。。私聊小姐姐（***）们

### 暗坑

最后发现他们有个体验会员，到期就是放弃（退出）

![file](https://bbs.kanxue.com/upload/attach/202007/871827_EAVENTQS629SD7U.jpg)

那还能咋办，继续分析

搜索大法

![file](https://bbs.kanxue.com/upload/attach/202007/871827_D4ZWFWW3NS53CRQ.jpg)

这里的`if`都是`&&`，一假则假。前边几个看着不好搞，就`dayDiff`入手，改他返回值，小于3就行了。

![file](https://bbs.kanxue.com/upload/attach/202007/871827_DWVDZ6NTBNASD7F.jpg)

代码就不放了。

### 小结

这个`vip`的破解还是很简单的那种。。。

这块其实可以通过拦截请求的方式更改数据包，将`vip`字段设置一下，没具体分析。

## 聊天IM && 半自动im机器人

根据`com.hyphenate.chat.EMMessage`可以得到这是环信的`im`聊天，那就很简单了。

两种方式实现发消息：

- 1.使用环信`sdk`，逆向该`app`的`inti`环境的一些配置信息，完成发消息。
- 2使用`app`本身的消息发送方法，用`frida`或者`xposed`主动调用。

这里由于没有`pythonSdk`，所以采用第二种。

### 分析消息发送

这里我们上`ddms`和环信的文档，分析它的调用情况。

文档如下

```java
发送文本消息
//创建一条文本消息，content为消息文字内容，toChatUsername为对方用户或者群聊的id，后文皆是如此
EMMessage message = EMMessage.createTxtSendMessage(content, toChatUsername);
//如果是群聊，设置chattype，默认是单聊
if (chatType == CHATTYPE_GROUP)
    message.setChatType(ChatType.GroupChat);
//发送消息
EMClient.getInstance().chatManager().sendMessage(message);
```

`ddms`如下

![file](https://bbs.kanxue.com/upload/attach/202007/871827_S4PX4772Y7992UQ.jpg)

打开`jadx`分析这个类代码，可以看到如下

![file](https://bbs.kanxue.com/upload/attach/202007/871827_6UFUAJ7TTVKFZ4E.jpg)

发送消息时候，创建了`EMMessage.createTxtSendMessage`，需要发送文本和对方的`id`。

下面的代码是发送文本信息的

```java
if (this.chatFragmentHelper != null) {
    this.chatFragmentHelper.onSetMessageAttributes(eMMessage);
}
if (this.chatType == 2) {
    eMMessage.setChatType(EMMessage.ChatType.GroupChat);
} else if (this.chatType == 3) {
    eMMessage.setChatType(EMMessage.ChatType.ChatRoom);
}
EaseUser userInfo = EaseUserUtils.getUserInfo(eMMessage.getFrom(), UncleSharedPreferences.getString(SZApplication.getContext(), UncleSharedPreferences.SP_UID));
eMMessage.setAttribute("avatar", userInfo.getAvatar());
eMMessage.setAttribute("gender", UncleSharedPreferences.getString(SZApplication.getContext(), UncleSharedPreferences.SP_USER_SEX));
eMMessage.setAttribute("name", userInfo.getNickname());
eMMessage.setAttribute("token", UncleSharedPreferences.getString(SZApplication.getContext(), UncleSharedPreferences.SP_UID));
if (UserCacheManager.getImidFromCache(this.toChatUsername) != null) {
    eMMessage.setAttribute("tokenTo", UserCacheManager.getImidFromCache(this.toChatUsername).getUserId());
} else {
    eMMessage.setAttribute("tokenTo", "");
}
eMMessage.setAttribute("nameTo", eMMessage.getTo());
eMMessage.setAttribute("avatarTo", UncleSharedPreferences.getString(SZApplication.getContext(), UncleSharedPreferences.SP_TO_USER_AVATAR));
EMClient.getInstance().chatManager().saveMessage(eMMessage);
```

知道上边的我们就可以用`frida`玩玩。

```javascript
var to_user_id = '1000477560';
var content = '我是一个机器人你信吗';
var uid = '1000511189';
var EMMessage = Java.use("com.hyphenate.chat.EMMessage");
var eMMessage = EMMessage.createTxtSendMessage(content, to_user_id);
eMMessage.setAttribute("avatar", "http://***/android/pic/1591284964")//头像
eMMessage.setAttribute("gender", "1")
eMMessage.setAttribute("name", "看123了看刻录机")
eMMessage.setAttribute("token", uid)//自己uid
eMMessage.setAttribute("nameTo", to_user_id)//对方imid
eMMessage.setAttribute("avatarTo", "")
eMMessage.setAttribute("tokenTo", to_user_id)
var EMClient = Java.use("com.hyphenate.chat.EMClient");
EMClient.getInstance().chatManager().saveMessage(eMMessage)
```

![file](https://bbs.kanxue.com/upload/attach/202007/871827_6ZM32EM672796WS.jpg)

发送之后，需要点进去聊天界面，才会发送过去，并且如果这个`app`没有缓存这个用户信息，就会闪退。

### 解决闪退

继续`ddms`，点击私信。

![file](https://bbs.kanxue.com/upload/attach/202007/871827_MC8S9HCHVYPWTXT.jpg)

搜索`startAc`，可以看到在`UserInfoActivity`下

![file](https://bbs.kanxue.com/upload/attach/202007/871827_YSNXTMJ3SU64G4D.jpg)

分析到如下代码，可以很清楚到看到，这里吧用户信息存起来了。然后启动聊天的`tActivity`。
