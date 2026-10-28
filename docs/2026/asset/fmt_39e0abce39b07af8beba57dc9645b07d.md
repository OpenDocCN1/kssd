
### ContentProvider

Android应用程序运行在不同的进程空间中，因此不同应用程序的数据是不能够直接访问的。为了增强程序之间的数据共享能力，Android系统虽然提供了像 `SharedPreferences` 这类简单的跨越程序边界的访问方法，但这些方法都存在一定的局限性。

`ContentProvider` 提供了应用程序之间共享数据的方法。应用程序通过 `ContentProvider` 访问数据而不需要关心数据具体的存储及访问过程，这样既提高了数据的访问效率，同时也保护了数据。

**Android系统附带的ContentProvider包括：**

- Browser：存储如浏览器的信息
- CallLog：存储通话记录等信息
- Contacts：存储联系人等信息
- MediaStore：存储媒体文件的信息
- Settings：存储设备的设置和首选项信息

**关于ContentProvider数据集**

`ContentProvider`数据集类似于数据库的数据表，每行是一条记录，每列具有相同的数据类型。

**ContentResolver 内容解析者**

应用程序使用 `ContentResolver` 对象，利用`URI`才能访问 `ContentProvider` 提供的数据集。一个`ContentProvider`可以提供多个数据集可以为多个`ContentResolver`服务。

**URI**

`Uniform Resource ldentifier` 通用资源标志符，用来定位远程或本地的可用资源。

**URI基本格式**

`content://authority/data_path/id`

- `content` 固定前缀
- `authority` 授权者名称，应保证唯一性，用于确定具体又 `ContentProvider` 提供资源
- `data_path` 数据路径，用于确定请求的数据集
- `id` 数据编号，用于匹配数据集中 `_ID` 字段的值（确定数据集中唯一的一条数据），如果需请求的数据不只限于一条，则该项省略

**URI示例**

- `content://contacts/people/` 表示全部联系人信息的URI
- `content://contacts/people/1` 表示ID=1的联系人信息的URI
- `content://com. android.contacts/contacts/` 全部联系人信息的URI（原生写法）
- `ContactsContract.Contacts.CONTENT_URI` 全部联系人信息的URI（常量写法）

由于URI通常比较长，而且容易写错，所以，在Android系统中定义了一些辅助类和常量来代替这些长字符串。

**URI和URL**

- 在Android中广泛应用URI，而不是URL。
- URL是标识资源的物理位置，相当于文件的路径
- URI则是标识资源的逻辑位置，并不提供资源的具体位置。
  - 比如说电话薄中的数据，如果用URL来标识的话：可能是一个很复杂的文件结构，而且一旦文件的存储路径改变，URL也必须得改动。
  - 但若是URI，则可以用诸如`content://contract /people`这样容易记录的逻辑地址来标识，而且并不需要关心文件的具体位置，即使文件位置改动也不需要做变化。
  - 当然这都是对于用户来说，后台程序中URI到具体位置的映射还是需要程序员来改动的。

**常见的URI**

具体的还有很多`CONTENT_URI`，可以到`ContactsContract.class`文件中去查看源码

| 常量值 | 作用 |
| --- | --- |
| ContactsContract.Contacts.CONTENT_URI | 管理联系人的Uri |
| ContactsContract.CommonDataKinds.Phone.CONTENT_URI | 管理联系人电话的Uri |
| ContactsContract.CommonDataKinds.Email.CONTENT_URI | 管理联系人邮箱的Uri |
| ContactsContract.CommonDataKinds.StructuredPostal.CONTENT_URI | 管理联系人地址的Uri |

**创建ContentProvider类**

`app` — `java` — 包名（第一个） — 右键 — New — Other — ContentProvider — 指定URI Authorities — Finish

#### 示例1

实现最简单的ContentProvider

**MainActivity.java**

```
package com.example.mytestapp;
import androidx.appcompat.app.AppCompatActivity;
import android.content.ContentResolver;
import android.database.Cursor;
import android.net.Uri;
import android.os.Bundle;
import android.widget.Toast;

public class MainActivity extends AppCompatActivity{
@Override
protected void onCreate(Bundle savedInstanceState) {
super.onCreate(savedInstanceState);
setContentView(R.layout.activity_main);

ContentResolver resolver = getContentResolver();
Uri uri = Uri.parse("content://myContentProvider");
Cursor cursor = resolver.query(uri,null,null,null,null,null);
Toast.makeText(this,"cursor：" + cursor,Toast.LENGTH_SHORT).show();
}
}
```

**MyContentProvider.java**

```
package com.example.mytestapp;

import android.content.ContentProvider;
import android.content.ContentValues;
import android.database.Cursor;
import android.net.Uri;
import android.util.Log;

public class MyContentProvider extends ContentProvider {
public MyContentProvider() {
}

@Override
public int delete(Uri uri, String selection, String[] selectionArgs) {
// Implement this to handle requests to delete one
// or more rows.
throw new UnsupportedOperationException("Not yet implemented");
}

@Override
public String getType(Uri uri) {
// TODO: Implement this to handle requests
// for the MIME type of the data
// at the given URI.
throw new UnsupportedOperationException("Not yet implemented");
}

@Override
public Uri insert(Uri uri, ContentValues values) {
// TODO: Implement this to handle requests to insert a new row.
throw new UnsupportedOperationException("Not yet implemented");
}

@Override
public boolean onCreate() {
// TODO: Implement this to initialize your content provider on startup.
return false;
}

@Override
public Cursor query(Uri uri, String[] projection, String selection,
String[] selectionArgs, String sortOrder) {
// TODO: Implement this to handle query requests
// from clients.
// throw new UnsupportedOperationException("Not yet implemented");
Log.d("lxz","MyContentProvider  query");
return null;
}

@Override
public int update(Uri uri, ContentValues values, String selection,
String[] selectionArgs) {
// TODO: Implement this to handle requests to update one
// or more rows.
throw new UnsupportedOperationException("Not yet implemented");
}
}
```

**AndroidManifest.xml**

```
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="..." xmlns:tools="...">
<application
android:allowBackup="true"
android:dataExtractionRules="@xml/data_extraction_rules"
android:fullBackupContent="@xml/backup_rules"
android:icon="@mipmap/ic_launcher"
android:label="@string/app_name"
android:supportsRtl="true"
android:theme="@style/Theme.MyTestApp"
tools:targetApi="31">
<activity
android:name=".MainActivity"
android:exported="true">
<intent-filter>
<action android:name="android.intent.action.MAIN" />
<category android:name="android.intent.category.LAUNCHER" />
</intent-filter>
</activity>

<provider
android:name=".MyContentProvider"
android:authorities="myContentProvider"
android:enabled="true"
android:exported="true">
</provider>
</application>

<uses-permission android:name="android.permission.INTERNET" />

</manifest>
```
