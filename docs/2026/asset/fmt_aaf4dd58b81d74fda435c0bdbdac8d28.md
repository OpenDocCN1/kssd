
`android:exported`

`provider`是否可供其他应用使用：
`true`：`providerr`可供其他应用使用。任何应用均可使用`provider`的`URI`来访问它，但需依据为`provider`指定的权限进行访问。
`false`：`provider`不可供其他应用使用。设置`android:exported="false"`可仅限您的应用访问`provider`。只有与`provider`具有相同的用户 ID (`UID`)的应用或者通过`android:grantUriPermissions`元素被临时授予对`provider`的访问权限的应用才能访问它。
由于此属性是在 API 级别 17 中引入的，因此所有搭载 API 级别 16 及更低级别的设备的行为方式就像将此属性设为 `"true"` 一样。对于搭载 API 级别 17 及更高级别的设备，如果您将`android:targetSdkVersion`设为 17 或更高版本，则默认值为 `"false"`。
您可以设置`android:exported="false"`并且仍然限制对`provider`的访问，方法是使用`permission`属性来设置相应权限。

这里也说了，`provider export = false ,grantUriPermissions= true`，授权给别的应用，是可以访问的，能绕过 `export = false` 。

但是实际情况是，在`android:grantUriPermissions=true` ，`export = false`的情况下，我无法访问`provider`，并且报错，就是`export=false`

```
java.lang.RuntimeException: Unable to start activity ComponentInfo{com.example.exploitapplication/com.example.exploitapplication.MainActivity}: java.lang.SecurityException: Permission Denial: opening provider androidx.core.content.FileProvider from ProcessRecord{ccc307 31798:com.example.exploitapplication/u0a247} (pid=31798, uid=10247) that is not exported from UID 10248
```

我也设置了权限传递

```
intent.addFlags(Intent.FLAG_GRANT_WRITE_URI_PERMISSION | Intent.FLAG_GRANT_READ_URI_PERMISSION);
```

并且查了不少的资料，开发的一些demo，国内外的provider漏洞利用，都说没有错，但是我就是不好使，我开始觉得，可能他就是有问题的，但也可能这是个低版本问题，毕竟我手里，只有android 8 和 9。

### grantUriPermissions = true 无法生效的解决方案

可能是对自己测试结果的不自信吧，于是到网上找了别人写好的demo试了一试，额，还真过去了。

```
Intent intent = new Intent(MediaStore.ACTION_IMAGE_CAPTURE);
intent.putExtra(MediaStore.EXTRA_OUTPUT, uriForFile);
intent.setFlags(Intent.FLAG_GRANT_WRITE_URI_PERMISSION | Intent.FLAG_GRANT_READ_URI_PERMISSION);
startActivityForResult(intent, 200);
@Override
protected void onActivityResult(int requestCode, int resultCode, Intent data) {
super.onActivityResult(requestCode, resultCode, data);
if (requestCode == 200) {
Log.e("Rzx",mPicPath.toString());
mImageView.setImageURI(Uri.fromFile(mPicPath));
}
}
```

他用的是`fileprovider`，开始我以为是`fileprovider`的问题，于是，我也用了这个，发现还是不行。

于是我也调用了相机程序，发现，相机程序是可以做到的，于是我又去逆向了系统的相机程序，代码很多，但是感觉他也没做什么多余的。

后来网上查到这篇资料

<mark class="encrypted">735K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2M7$3W2Q4x3X3c8&6k6h3g2Q4x3X3g2U0L8$3#2Q4x3V1j5J5x3o6t1H3i4K6u0r3x3o6S2Q4x3V1j5I4z5g2)9J5c8W2)9J5y4f1f1&6i4K6t1#2z5o6N6Q4x3U0f1^5c8q4)9J5y4f1f1#2i4K6t1#2b7f1c8Q4x3U0g2m8y4V1q4F1k6s2u0G2K9h3c8Q4x3U0g2q4y4q4)9J5y4f1t1&6i4K6t1#2z5p5u0r3K9h3I4W2f1s2u0G2N6X3W2V1k6i4u0Q4x3V1j5`.</mark>

他说了`action`的问题，但是我是主动设置的。

`intent.setFlags(Intent.FLAG_GRANT_WRITE_URI_PERMISSION | Intent.FLAG_GRANT_READ_URI_PERMISSION);`

于是我抱着试一下的心态，测试了一下，还真过去了

#### 第一步，修改action

```
intent = new Intent(MediaStore.ACTION_IMAGE_CAPTURE);
intent.setClassName("com.example.exploitapplication","com.example.exploitapplication.MainActivity");
intent.putExtra(MediaStore.EXTRA_OUTPUT, uriForFile);
intent.addFlags(Intent.FLAG_GRANT_WRITE_URI_PERMISSION | Intent.FLAG_GRANT_READ_URI_PERMISSION);
startActivityForResult(intent, 200);
```

#### 第二部，给程序授予相机权限，否则会报错，Intent发不出去

接收代码

```
public void grantUriPermissions_fileprovider(Intent intent){
try {
Uri uriForFile = intent.getParcelableExtra(MediaStore.EXTRA_OUTPUT);
InputStream is = getContentResolver().openInputStream(uriForFile);
byte[] buffer = new byte[1024];
int byteCount;
while ((byteCount = is.read(buffer)) != -1) {
Log.e("Rzx",new String(buffer));
}
is.close();
} catch (FileNotFoundException e) {
e.printStackTrace();
} catch (IOException e) {
e.printStackTrace();
}
}
```

### grantUriPermissions = true 无法生效的解决方案（二）

在无意中，我还发现第二种方案，不过可能有些鸡肋。

```
Intent intent = new Intent();
intent.setClassName("com.example.exploitapplication","com.example.exploitapplication.MainActivity");
List<ResolveInfo> resInfoList = getPackageManager().queryIntentActivities(intent, PackageManager.MATCH_DEFAULT_ONLY);
for (ResolveInfo resolveInfo : resInfoList) {
String packageName = resolveInfo.activityInfo.packageName;
grantUriPermission(packageName, uriForFile, Intent.FLAG_GRANT_WRITE_URI_PERMISSION | Intent.FLAG_GRANT_READ_URI_PERMISSION);
}
intent.putExtra(MediaStore.EXTRA_OUTPUT, uriForFile);
intent.addFlags(Intent.FLAG_GRANT_WRITE_URI_PERMISSION | Intent.FLAG_GRANT_READ_URI_PERMISSION);
startActivityForResult(intent, 200);
```

### 总结

手机类型不全，不清楚android7是否会没有这样的问题，系统源码方面么有深入挖掘，第一种方法需要应用本身有相机权限，第二种方法，应用本身需要给目标应用主动授权。这两种方法都可以在应用层实现，不知道有没有办法，hook java方法什么的，达到这两种方法的目的。

## 自定义权限

我们称为服务程序

调用我们的成为第三方程序

`android`自定义权限是第三方程序，调用服务程序组件的时候，服务程序，赋予第三方程序的权限。

|  |  |

| --- | --- |

| 1  2  3  4  5 | `<permission`  `android:name`=`"com.sl.permission.aidl"`  `android:description`=`"@string/service_permission"`  `android:permissionGroup`=`"com.sl.permissions"`  `android:protectionLevel`=`"signature"` `/`>` |

### Permission 属性：

- `name` : 权限名称,第三方，写明声明的这个权限

- `description` : 权限描述

- `permissionGroup` : 指定权限属于的权限组，别的程序申请的时候需要写的

- `protectionLevel` : 权限保护级别

  - `Normal`

    这是最低风险的权限，如果应用声明了此权限，也不会提示安装应用的用户授权（例如，如果声明了定位权限，则应用到定位功能时，会明确提示用户，是否授予定位权限，但是`protectionLevel`为`normal`的不会明确提示，直接默认授予），系统直接默认该应用有此权限；

  - `dangerous`

    这种级别的权限风险更高，拥有此权限可能会访问用户私人数据或者控制设备，给用户带来负面影响，这种类型的权限一般不会默认授权。

  - `signature`

    这种权限级别，只有当发请求的应用和接收此请求的应用使用同一签名文件，并且声明了该权限才会授权，并且是默认授权，不会提示用户授权

  - `signatureOrSystem`

    这种权限级别是系统授权的系统应用或者相同签名的应用，一般避免使用该级别，因为 `signature` 已经能满足大部分需求。

### 服务程序组件声明自定义权限

|  |  |

| --- | --- |

| 1  2  3  4  5  6  7  8 | `<provider`  `android:authorities`=`"com.example.student"`  `android:name`=`.provider.MyContentProvider`  `android:grantUriPermissions`=`"true"`    `android:permission`=`"com.sl.permission.aidl"`    `android:exported`=`"true"`/>` |

### 第三方程序请求权限

|  |  |

| --- | --- |

| 1 | `<uses-permission android:name`=`"com.sl.permission.aidl"` />` |

### 自定义权限安全相关问题

`android:permission`：统一提供程序范围读取/写入权限。

`android:readPermission`：提供程序范围读取权限。

`android:writePermission`：提供程序范围写入权限。

而权限，只有四个值，或者不写权限

- `Normal` 和不写权限

  没有声明权限，第三方程序，也可以不写，但是如果服务程序写了权限，第三方程序，必须写，否则为报错，使用这个组件权限不足的错误。

- `dangerous`

  这个权限没试过，估计权限很高，很危险，但是，我估计，基本不会遇到这种权限的app

- `signature` 和 `signatureOrSystem`

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

![在这里插入图片描述](https://bbs.kanxue.com/upload/attach/202107/767217_GJTPPJ26Q79RY75.jpg)

**Authority**：授权信息，以区别不同的ContentProvider

**path**：表名，以区分ContentProvider中不同的数据表

**id**：id号，用于区分表中的不同数据

- `getAuthority()`:获取Uri中Authority部分

- `getPath()`:获取Uri中path部分

`ContentResolver`通过`Authority`寻找`ContentProvider`，

### UriMatch

`UriMatch`主要为了区配URI，比如应用A提供了数据，但是并不是说有的应用都可以操作这些数据，只有提供了满足应用A的URI，才能访问A的数据，而`UriMatch`就是做这个区配工作的，他有如下方法

- `public UriMatcher(int code)` ：它的作用就是创建一个`UriMatch`对象

- `public void addURI(String authority,String path, int code)`：它的作用是在`ContentProvider`添加一个用于匹配的Uri，当匹配成功时返回code。Uri可以是精确的字符串，Uri中带有`*`表示可匹配任意text，`#`表示只能匹配数字。

- `public int match(Uri uri)` ：这里的Uri就是传过来的要进行验证，匹配的Uri假如传过来的是：`content://com.example.test/student/#`, `content://com.example.test/student/10`可以匹配成功，这里的10可以使任意的数字。

### ContentProvider真实的样子

开始笔记的时候，我才发现，虽然我了解`ContentProvider`的部分原理，但是真的没有完整的用过她，更没有在某一个app中，实现并使用过她，甚至demo都是看博客，说说干嘛，而且我唯一分析的大型关于`ContentProvider`的app竟然就只有`virturalApp`，但是他的使用。废话到此为止。
