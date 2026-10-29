
# [原创]android app漏洞入门学习小记-自定义权限和content provider

## 自定义权限

我们称为服务程序

调用我们的成为第三方程序

android自定义权限是第三方程序，调用服务程序组件的时候，服务程序，赋予第三方程序的权限。

```
<permission
android:name="com.sl.permission.aidl"
android:description="@string/service\_permission"
android:permissionGroup="com.sl.permissions"
android:protectionLevel="signature" />
```

### Permission 属性：

- `name` : 权限名称,第三方，写明声明的这个权限

- `description` : 权限描述

- `permissionGroup` : 指定权限属于的权限组，别的程序申请的时候需要写的

- `protectionLevel` : 权限保护级别

- Normal

这是最低风险的权限，如果应用声明了此权限，也不会提示安装应用的用户授权（例如，如果声明了定位权限，则应用到定位功能时，会明确提示用户，是否授予定位权限，但是`protectionLevel`为`normal`的不会明确提示，直接默认授予），系统直接默认该应用有此权限；

- dangerous

这种级别的权限风险更高，拥有此权限可能会访问用户私人数据或者控制设备，给用户带来负面影响，这种类型的权限一般不会默认授权。

- signature

这种权限级别，只有当发请求的应用和接收此请求的应用使用同一签名文件，并且声明了该权限才会授权，并且是默认授权，不会提示用户授权

- signatureOrSystem

这种权限级别是系统授权的系统应用或者相同签名的应用，一般避免使用该级别，因为 `signature` 已经能满足大部分需求。

### 服务程序组件声明自定义权限

```
<provider
android:authorities="com.example.student"
android:name=".provider.MyContentProvider"
android:grantUriPermissions="true"
android:permission="com.sl.permission.aidl"
android:exported="true"/>
```

### 第三方程序请求权限

`<uses-permission android:name="com.sl.permission.aidl" />`

### 自定义权限安全相关问题

`android:permission`：统一提供程序范围读取/写入权限。

`android:readPermission`：提供程序范围读取权限。

`android:writePermission`：提供程序范围写入权限。

而权限，只有四个值，或者不写权限

- Normal 和不写权限

没有声明权限，第三方程序，也可以不写，但是如果服务程序写了权限，第三方程序，必须写，否则为报错，使用这个组件权限不足的错误。

- dangerous

这个权限没试过，估计权限很高，很危险，但是，我估计，基本不会遇到这种权限的app

- signature 和 signatureOrSystem

`signature`：只有签名相同的程序才能使用，估计除非签名文件被盗，否则，应该都是自家在用，跟不开放没啥区别，

`signatureOrSystem`：比签名多了一个使用者，就是拥有系统签名的app，可能可以靠某个系统app间接调用过去，我们去测试的app，估计都是第三发发行的应用，我估计应该没人用这个。

总结：

只要`export = true`，外部程序直接可以访问，寻找漏洞。否则无法攻击

自定义的权限`Normal`，攻击程序就能进行攻击，否则无法攻击。

这个，其实没那么重要，因为，我们可以分析目标app，进行目标app的权限申请，如果有签名验证的话，是无法申请的，只能通过别的方式调用，也没办法处理。这个可能写的不太细致。有兴趣的可以自己查一查。

## ContentProvider

### Google Doc中对ContentProvider的大致概述。

内容提供者将一些特定的应用程序数据供给其它应用程序使用。数据可以存储于文件系统、SQLite数据库或其它方式。内容提供者继承于`ContentProvider` 基类，为其它应用程序取用和存储它管理的数据实现了一套标准方法。然而，应用程序并不直接调用这些方法，而是使用一个 `ContentResolver` 对象，调用它的方法作为替代。`ContentResolver`可以与任意内容提供者进行会话，与其合作来对所有相关交互通讯进行管理。

### 怎么使用ContentProvider

首先我们需要知道三个类

- `ContentProvider`（内容提供者）

- `ContentResolver`（内容解析者）

- `ContentObserver`（内容观察者）

假如我们现在有个应用A 提供了数据 ，应用B要操作应用A的数据，那么

- 应用A使用`ContentProvider`去共享自己数据

- 应用B使用`ContentResolver`去操作应用A的数据，通过`ContentObserver`去监听应用A的数据变化

- 当应用A的数据发送改变时，通知`ContentObserver`去告诉应用B数据变化了及时更新

这就是通信的大致流程，在了解更加详细的流程之前，我们还需要知道几个概念

### ContentProvider中的URI

ContentProvider中的URI是有固定格式的，例如：

![在这里插入图片描述](upload/attach/202107/767217\_GJTPPJ26Q79RY75.jpg)

`Authority`：授权信息，以区别不同的`ContentProvider`

`path`：表名，以区分`ContentProvider`中不同的数据表

`id`：id号，用于区分表中的不同数据

- `getAuthority()` :获取Uri中Authority部分

- `getPath()` :获取Uri中path部分

`ContentResolver`通过`Authority`寻找`ContentProvider`，

### UriMatch

`UriMatch`主要为了区配URI，比如应用A提供了数据，但是并不是说有的应用都可以操作这些数据，只有提供了满足应用A的URI，才能访问A的数据，而`UriMatch`就是做这个区配工作的，他有如下方法

- `public UriMatcher(int code)` ：它的作用就是创建一个`UriMatch`对象

- `public void addURI(String authority,String path, int code)`：它的作用是在`ContentProvider`添加一个用于匹配的Uri，当匹配成功时返回`code`。Uri可以是精确的字符串，Uri中带有`\*`表示可匹配任意text，`#`表示只能匹配数字。

- `public int match(Uri uri)` ：这里的`Uri`就是传过来的要进行验证，匹配的`Uri`假如传过来的是：`content://com.example.test/student/#`, `content://com.example.test/student/10`可以匹配成功，这里的10可以使任意的数字。

### ContentProvider真实的样子

开始笔记的时候，我才发现，虽然我了解`ContentProvider`的部分原理，但是真的没有完整的用过她，更没有在某一个app中，实现并使用过她，甚至demo都是看博客，说说干嘛，而且我唯一分析的大型关于`ContentProvider`的app竟然就只有`virturalApp`,但是他的使用。废话到此为止。

`ContentProvider`，网上的大致都是打开数据库，进行操作数据的接口，或者`FileProvider`文件共享，给我整懵逼了，我仔细总结了一下，并亲自写了demo,总结如下：

- `ContentProvider`作为一个系统组件，他唯一的提供的只有，远程调用（rpc,是的，你没看错）

- 数据操作和文件贡献，都是在他提供的远程调用的框架上，我们需要写的代码。

```
//远程调用contentResolver对应的方法
switch (v.getId()) {
case R.id.zeng:
contentResolver.insert(URI, contentValues);
break;
case R.id.shan:
contentResolver.delete(URI, null, null);
break;
case R.id.gai:
contentResolver.update(URI, contentValues, null, null);
break;
case R.id.cha:
contentResolver.query(URI, null, null, null, null);
break;
case R.id.openfile:
try {
contentResolver.openInputStream(URI);
} catch (FileNotFoundException e) {
e.printStackTrace();
}
break;
case R.id.call:
Bundle bundle = new Bundle();
contentResolver.call(URI,"ABC","DEF",bundle);
break;
}
```

上面的函数会直接的调用对应URI的provider 的方法，

`openInputStream`，会调用`openAssetFile`

`call` 会调用`call`

还会把参数传递过去，至于实现什么功能，就是你自己的事情了

```
public @Nullable AssetFileDescriptor openAssetFile(@NonNull Uri uri, @NonNull String mode)
throws FileNotFoundException {
ParcelFileDescriptor fd = openFile(uri, mode);
return fd != null ? new AssetFileDescriptor(fd, 0, -1) : null;
}
```

`openAssetFile` ,调用`openFile`，我们可以直接重写`openFile`，也可以，重写`openAssetFile`（我开始的时候重写的`openAssetFile`，有点傻，后来发现要解决的问题有点多，不太会用，发现写`openFile`比较方便）

```
@Override
public ParcelFileDescriptor openFile(Uri uri, String mode)
throws FileNotFoundException {
Log.e("Rzx","provider openfile");
Log.e("Rzx",uri.toString()+uri.getPath());
//由于是应用内的不同进程，故这里把文件放到data/data/包名 目录下
File testFile =new File(getContext().getCacheDir(),"test.html");
// ParcelFileDescriptor.MODE\_READ\_ONLY：只可读
// ParcelFileDescriptor.MODE\_WRITE\_ONLY：只可写
// ParcelFileDescriptor.MODE\_READ\_WRITE：可读可写
return ParcelFileDescriptor.open(testFile,
ParcelFileDescriptor.MODE\_READ\_WRITE);
}
```

代码里的三个权限，设置，才是设置这个provider提供的文件读写功能，返回的文件对象，具有什么样的权限

### ContentProvider grantUriPermissions 临时权限

`grantUriPermissions = "true"`表示，表示允许权限传递，就是拥有访问权限的组件，可以把访问权限传递给别的组件，为false，权限无法传递，只能自己使用。

被攻击的provider，无法通过外部访问

```
<provider
android:authorities="com.example.student"
android:name=".provider.MyContentProvider"
android:grantUriPermissions="true"
android:exported="false"/>
```

间接传递权限的activity

```
public class grantUriPermissionsActivity extends AppCompatActivity {
protected void onCreate(Bundle savedInstanceState) {
super.onCreate(savedInstanceState);
setContentView(R.layout.activity\_test);
setResult(-1, getIntent());
finish();
}
}
```

```
Intent intent = new Intent();
intent.setData(Uri.parse("content://" + "com.example.student"+"/input"));
intent.setClassName("com.example.vulnerableapplication", "com.example.vulnerableapplication.provider.grantUriPermissionsActivity");
intent.setFlags(Intent.FLAG\_GRANT\_READ\_URI\_PERMISSION);
startActivityForResult(intent,0);
protected void onActivityResult(int requestCode, int resultCode, Intent data) {
Log.e("rzx","onActivityResult");
super.onActivityResult(requestCode, resultCode, data);
try {
String str = "content://" + "com.example.student"+"/input";
InputStream is = getContentResolver().openInputStream(Uri.parse(str));
byte[] buffer = new byte[1024];
int byteCount;
while ((byteCount = is.read(buffer)) != -1) {
Log.e("Rzx",new String(buffer));
}
is.close();
} catch (Exception e) {
throw new RuntimeException(e);
}
}
```

### ContentProvider组件安全

- `android:exported`属性

从API Level17及以后，Content Provider默认是私有的，除非显式指定，`android:exported="true"`。

provider设置为false时，只有同一个应用程序的组件或带有相同用户ID的应用程序才能启动或绑定该服务。如果为true,外部应用，就可以调用，访问使用内部提供的功能，根据实现的功能就行测试攻击，而不只是sqlite和文件访问这些功能而已。当然，可能效果没有那么好。（配合使用更佳，后面会介绍例子）

- 跨进程调用provider，如果进程死了，会唤醒目标进程，并且，在这个调用进程主动被杀以后，目标进程没有被杀。

作为第一手攻击点，启动进程，是可以的。不过，估计和广播哪个一样，没有后台弹出界面权限。助攻也可以

- provider本进程启动的时候，会在`Application oncreate`方法调用前初始化完成，调用完`oncreate`

这里，加固的时候，如果在`oncrete`函数里调用provider,要在之前完成provider手工安装（这里记不太清了，但是provider在加固的时候有坑）可以去分析他的启动时机。

- provider本身，没有权限控制，sql，文件读写功能，只提供的双端交互，后续以及产生的各种漏洞，都是开发人员在provider基础上进行功能扩展，代码检测不严格，出现的问题。

### 实际测试问题

如果你看到这里觉得前面写的没有问题的话，那么。。。。。。。。。。

好吧，先说一下，出现的问题吧

- `grantUriPermissions = true` 无法生效

先看官方描述

（<mark class="encrypted">5f5K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1k6i4k6W2L8r3!0H3k6i4u0Q4x3X3g2S2L8X3c8J5L8$3W2V1i4K6u0W2k6$3!0G2k6$3I4W2i4K6u0W2j5$3&6Q4x3V1k6Y4N6h3W2V1k6g2)9J5c8Y4c8G2M7r3W2U0M7#2)9J5c8X3#2S2L8X3W2X3k6i4y4@1i4K6u0r3M7s2u0G2N6X3W2V1k6i4u0Q4x3X3c8W2L8r3g2E0k6h3&6@1</mark> 不知道这个文档是不是老了，或者有什么问题）
