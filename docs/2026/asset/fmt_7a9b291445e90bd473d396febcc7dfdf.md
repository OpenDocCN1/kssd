

# [分享] Frida 辅助分析 OLLVM

## 字符串加密

先从程序的入口点看看：

![图片描述](upload/attach/202211/939330\_V4HFSRR3FYVU6YY.png)

通过点击 `onclick` 按钮，随机生成一个字符串，然后传入 `sign1` 方法。这个 `sign1` 方法是一个 `native` 层的方法，我们用 `objection` 来验证一下：

![图片描述](upload/attach/202211/939330\_VNXJYBJGNJDQ9ZT.png)

我们可以发现确实是这么回事。

然后用 `objection` 看一下 `so` 层的内存信息：

![图片描述](upload/attach/202211/939330\_2CWVZA7P5WKJAZH.png)

发现了 `libhello-jni.so`。

![图片描述](upload/attach/202211/939330\_5QWU48GVC7KVJSX.png)

![图片描述](upload/attach/202211/939330\_DN5EACZNYWQ2MUX.png)

那么我们就可以分析 `so` 层了，拖入 `IDA` 看看：

![图片描述](upload/attach/202211/939330\_6FUNNFVQTC94KD5.png)

在 `export` 导入函数中没有发现 `sign1` 函数，那么我们从 `init_array` 字段看看：

![图片描述](upload/attach/202211/939330\_CAXP7TWYTU6WAQY.png)

很明显，我们发现了 OLLVM 字符串混淆的关键字：以 `.datadiv` 开头的字段。

我们选择第二个跟进去分析：

![图片描述](upload/attach/202211/939330\_SZDXEN72CYP4GNF.png)

发现确实是一堆字符串的异或运算。我们首先在 `010editor` 中还原一下看看。

先看一下它的值 `byte_6EA6DBB118`：

![图片描述](upload/attach/202211/939330\_APMUGCC44D94X4C.png)

是 `F9 D2`，然后和 `D2` 异或之后的结果是这样的：

![图片描述](upload/attach/202211/939330\_8UMR5M23CGM7MGU.png)

然后继续看看接下来几个字符串异或的结果：

`byte_6EA6DBB11C` -> `byte_6EA6DBB122`

![图片描述](upload/attach/202211/939330\_Q8Q7A4R6RWWBGZM.png)

![图片描述](upload/attach/202211/939330\_RBA4239HUQD89Y8.png)

这个是异或之后的结果：

![图片描述](upload/attach/202211/939330\_DTMQMXAH5QS7D5H.png)

这里我们能确定它是一个 OLLVM 字符串混淆的加密，但分析不出来算法。然后我们再看一下 `JNI_OnLoad` 函数，这里展示一下我修复之后的代码：

![图片描述](upload/attach/202211/939330\_U2FFSHS9EFXJRCP.png)

接下来我们看一下 `RegisterNatives` 第三个参数是什么。一般动态注册的函数都在这里面：`v7 = *&off_33D60`，在上面有个赋值：`v7 = *&off_33D60`。

跟进去看看这个字符串：

![图片描述](upload/attach/202211/939330\_TW6G6HW59KEASES.png)

继续跟进去看看：

![图片描述](upload/attach/202211/939330\_QDBYDNQ8S4KHBE5.png)

发现也是被加密的，所以我们用 `Frida` 看看它到底是什么值。

附上 `JS` 代码：

```javascript
function hook_native(){
    // OLLVM 默认的字符串混淆，静态的时候没法看见字符串
    // 执行起来之后，先调用 .init_array 里面的函数来解密字符串
    // 解密完之后，内存中的字符串就是明文状态了。
    // 首先找到函数的基址
    var base_hello_jni = Module.findBaseAddress("libhello-jni.so");
    // 这里是基址指针
    if(base_hello_jni){
        var addr_37070 = base_hello_jni.add(0x37070);
        // 利用指针加偏移的方法找到函数地址
        console.log("addr_37070:",ptr(addr_37070).readCString());
        var addr_37080 = base_hello_jni.add(0x37080);
        // 利用指针加偏移的方法找到函数地址
        console.log("addr_37080:",ptr(addr_37080).readCString());
    }
}
```

执行一下看看：

![图片描述](upload/attach/202211/939330\_JSSDVHFTSPC97AD.png)

发现就是我们要分析的函数，但是之前是被 OLLVM 加密的。

接下来我们分析一下注册的这个函数：

![图片描述](upload/attach/202211/939330\_XVQMTDP2YWFDDCZ.png)

发现它里面也是有一堆字符串。刚刚分析的过程中知道了这个 `so` 使用了 OLLVM 加密，所以我们继续看看这些字符串里面是什么值：

![图片描述](upload/attach/202211/939330\_7B9QX4F84ZBBT8E.png)

用 `Frida` 来 `hook` 一下：

这里有一种更好的 `JS` 脚本（这个 `JS` 脚本可以查看任意地址的字符串的值）：

```javascript
function print_string(addr){
    var base_hello_jni = Module.findBaseAddress("libhello-jni.so");
    // 这里是基址指针
    var addr_str = base_hello_jni.add(addr);
    // 利用指针加偏移的方法找到函数地址
    console.log("addr:",addr," ",ptr(addr_str).readCString());
}
```

执行结果：

![图片描述](upload/attach/202211/939330\_3SXQMWQ6ATF59PM.png)

我们接下来先不分析 `sign` 算法，最后再分析。我们接下来看看如果修改一下 OLLVM 源码会发生什么：`init_array` 的 OLLVM 字符串混淆关键字段 `.datadiv` 也是可以修改的：

![图片描述](upload/attach/202211/939330\_9MSSTBDH79RV682.png)

然后我们跟进去分析，发现也是字符串的异或：

![图片描述](upload/attach/202211/939330\_ZASYR9MN6YHR7Y3.png)

前面的字符串异或我们刚刚已经介绍完了，接下来的这个我们怎么处理呢？

![图片描述](upload/attach/202211/939330\_W7Q8HDZPM4N7E2Y.png)

`stru_37010[0] = veorq_s8(stru_37010[0], v0);`

我们首先看一下 `stru_37010` 的值：

![图片描述](upload/attach/202211/939330\_NVGJRZE2JSJTYKK.png)

然后看一下 `v0` 的值：

![图片描述](upload/attach/202211/939330\_NRMH8QPXJ5E4VJ4.png)

发现 `v0 = 0xC6`。

这里我们还是用 `010editor` 看看，发现是这样的字符串：

![图片描述](upload/attach/202211/939330\_5VDBFJUBADZA464.png)

然后我们继续看一下 `JNI_OnLoad` 函数：

![图片描述](upload/attach/202211/939330\_X659TVSJQCEWHBQ.png)

根据之前的分析，我们还是要确定一下这个的值：

`v7 = *(_OWORD *)&off_33D60`

跟进去看看：

![图片描述](upload/attach/202211/939330\_PQX2MKNDF82NM2C.png)

然后找到字符串的地址：

![图片描述](upload/attach/202211/939330\_5DSYXTPCT9E2FTP.png)

用刚刚的通杀字符串的 JS 脚本看一下：

![图片描述](upload/attach/202211/939330\_AVK5C6VSTM4RAFS.png)

发现是我们要分析的 `sign1` 函数。

然后我们再修改一下 OLLVM 源码，继续混淆字符串。

还是先看 `init_array` 字段：

![图片描述](upload/attach/202211/939330\_KM4R7FRHYQKWKNE.png)

跟进去看看函数：

![图片描述](upload/attach/202211/939330\_8AJEH6WP6FTMG2K.png)

我们在这里并看不出来解密字符串。

接下来看看 `JNI_OnLoad` 函数吧。

我们还是先找 `RegisterNatives` 函数：

![图片描述](upload/attach/202211/939330\_Z3DXSWDMU9BS463.png)

然后定位到它的第三个参数。

我们向上定位分析，发现并不像之前那样好分析了（直接用查看字符串的 JS 脚本不行了）。

其实也能分析，先说一下吧，就是向上回溯分析。先讲一下这个麻烦的办法，一会儿再说一种通用的方法。

我们还是先看它的第三个参数：

![图片描述](upload/attach/202211/939330\_7PF2T5RQX8BKMZE.png)

`v10 = xmmword_3E1E8;`

然后继续向上回溯：

![图片描述](upload/attach/202211/939330\_CYBVBAVXSBVU6UY.png)

![图片描述](upload/attach/202211/939330\_X6XFUQW8Y683UR3.png)

这里就出现了一点小问题。我们是可以知道 `xmmword_3E1E8 = sign1`，但是它在往后的 8 个字节的长度，也就是一个指针的长度：

![图片描述](upload/attach/202211/939330\_GC3SMC87DZ877D6.png)

因为它进行了一些异或运算，我们其实可以动态调试分析出来。我们今天讲 `Frida`，就不再使用这种方法了。我们继续回溯：

![图片描述](upload/attach/202211/939330\_WU995ZYHHRV3VMS.png)

然后我们 `hook` 一下这个 `3E1BA` 看看：

![图片描述](upload/attach/202211/939330\_MQMUPDQD9UUFQ4W.png)

也是可以分析出来的。

然后 `RegisterNatives` 函数的第二个参数我们也是可以分析出来的：

![图片描述](upload/attach/202211/939330\_TAXC5WSA8RUM9FV.png)

我们 `hook` 它来看看（用刚刚的通杀字符串脚本）：

![图片描述](upload/attach/202211/939330\_R6B8JHKS2XXJ6VH.png)

这样也是可以查看的。

![图片描述](upload/attach/202211/939330\_HYKJ8UDGDZ5UMVE.png)

然后接下来我们看一种更好用的方法：直接 `hook` `RegisterNatives` 函数的参数。这样是不是我们就不用分析过程了？无论它怎么加密，我们都能得到最后的值。附上 `JS` 脚本：

```javascript
function hook_libart() {
    var module_libart = Process.findModuleByName("libart.so");
    // 枚举所有的文件来找到
    // 首先找到 so
    var symbols = module_libart.enumerateSymbols();
    // 枚举模块的符号
    var addr_RegisterNatives = null;
    // 怎么 hook RegisterNatives
    for (var i = 0; i < symbols.length; i++) {
        var name = symbols[i].name;
        if (name.indexOf("art") >= 0) {
            if ((name.indexOf("CheckJNI") == -1) && (name.indexOf("JNI") >= 0)) {
                if (name.indexOf("RegisterNatives") >= 0) { // 找到函数的名字
                    console.log("!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!")
                    console.log(name);
                    addr_RegisterNatives = symbols[i].address;
                    break;
                }
            }
        }
    }
    if (addr_RegisterNatives) {
        Interceptor.attach(addr_RegisterNatives, {
            onEnter: function (args) {
                console.log("$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$")
                console.log("addr_RegisterNatives name:", ptr(args[2]).readPointer().readCString())
                // 因为 RegisterNatives 函数的第三个参数的类型是指针类型，所以要先 ptr().readPointer() 再读取其中的值
                console.log("addr_RegisterNatives sig:", ptr(args[2]).add(Process.pointerSize).readPointer().readCString());
                // 增加一个指针的长度，也就是我们刚刚分析过程中加密的那一段字符串的值，
                // 注意一下这里不是第四个参数，因为这里是一个指针，首先指向了第三个参数的地址，然后从第三个参数的那片内存区域中增加了 8 个长度，也就是刚刚那段加密的字符串
            },
            onLeave: function (retval) {
            }
        });
    }
}
```

这个就是执行结果了：

![图片描述](upload/attach/202211/939330\_E2RSNM65YNPBQTM.png)

但是有的时候如果混淆得特别严重，我们就会找不到 `RegisterNatives` 这个函数，所以第一种方法还是要会的。接下来再说一种通过 `hook` 寄存器的值来得到结果的方法。这里我们定位 `byte_3E1BA`，然后发现了 `eor`：

![图片描述](upload/attach/202211/939330\_QCXXPSDTTSJBN7S.png)

`eor` 之后的值就是我们需要的值了，也就是这个 `w13` 的值：

![图片描述](upload/attach/202211/939330\_JHF7MQ7T59EF9WY.png)

附上 JS 脚本：

