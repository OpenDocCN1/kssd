
我们接着看`JNI_OnLoad`，可以发现只动态注册了一个函数，那我们有理由猜测是`test`。

![]()

在动态注册时，我们通过观察`IDA`的`.data.rel.ro`，需要知道方法结构体信息：

```c
typedef struct {
    const char* name;
    const char* signature;
    void*       fnPtr;
} JNINativeMethod;
```

结构体包含三部分，分别是：方法名、方法的签名、对应的`native`函数地址。

那么这里我们肯定重点看第三部分，因为要找到具体的解密函数，这时候我们需要在动态注册段对应上：

![](https://bbs.kanxue.com/upload/attach/202008/TGC3D6D5BSSAVWT.jpg)

**我们来验证一下：**

### 函数名

`byte_1C066`：`0x87, 0x96, 0x80, 0x87, 0xF3, 0, 0, 0, 0, 0`

![]()

长度为`4`。

> `[0] = 0x87 ^ 0xF3 = 0x74 = t`
>
> `[1] = 0x96 ^ 0xF3 = 0x65 = e`
>
> `[2] = 0x80 ^ 0xF3 = 0x73 = s`
>
> `[3] = 0x87 ^ 0xF3 = 0x74 = t`

合成得：`test`。

所以这是`test`函数名。

![](https://bbs.kanxue.com/upload/attach/202008/SHGD8VM9WT8NZW2.jpg)

### 函数签名信息

![](https://bbs.kanxue.com/upload/attach/202008/JEXKZNUBNKFFKUU.jpg)

`byte_1C070`解密后是：`(Ljava/lang/String;)Z`。

正好就是`boolean test(string content)`。

![](https://bbs.kanxue.com/upload/attach/202008/QYJW5Z3JFTAK9TK.jpg)

![](https://bbs.kanxue.com/upload/attach/202008/3JJSTDWZKKYPT5V.jpg)

### 函数注册地址

之后我们来到`OOXX`函数观看流程，发现`JUMPOUT`函数，而且进入`sub_8930`函数查看了一些逻辑之后发现流程晦涩难懂。

![](https://bbs.kanxue.com/upload/attach/202008/XEYVFKS9956H9DZ.jpg)

当我们遇到晦涩难懂的问题时，不如尝试去用用`Frida`。

## 转换思路

一直以来**想用Frida解****开**一道这种验证输入的题，奈何没有找到机会，这一次想尝试一下：

```javascript
// 新建 frida_test.js

function hookstrcmp() {
    Java.perform(function() {
        console.log("I am a Hook function");

        var strcmp = Module.findExportByName("libc.so", "strcmp"); // 这里发现无论“libnative-lib.so”还是“libc.so”都是一样的地址

        console.log("find strcmp:", strcmp);

        Interceptor.attach(strcmp, {
            onEnter: function(args) {
                // hook 住后打印 strcmp 的第一个参数和第二个参数的内容
                console.log("[*] strcmp (" + ptr(args[0]).readCString() + "," + ptr(args[1]).readCString() + ")");
            },
            onLeave: function(retval) {}
        });
    });
}
```

启动`frida`并`attach` `app`的进程：

```bash
frida -U com.kanxue.test -l frida_test.js
```

首先敲入：

```text
hookstrcmp()
```

然后再在手机端敲入“bangbang”：

```text
观察电脑端日志
```

我打印出来了整个运行过程中的`strcmp`，发现只有一处和`bangbang`进行了比较。

![](https://bbs.kanxue.com/upload/attach/202008/JJW7E6VYPPCSUNS.jpg)

毋庸置疑，`kanxuetest`即`flag`值。

## 0x03 验证

![](https://bbs.kanxue.com/upload/attach/202008/7BAR9NZUXCP5DAP.jpg)

所以，`kanxuetest`就是`flag`。

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

上传的附件：

- [2.apk](javascript:void(0;)

  （2.44MB，242次下载）
