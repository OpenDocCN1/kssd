
``` 
var offset = 0x000428BC;
var nativelibModule = Process.findModuleByName("libnative-lib.so");
var func_addr = nativelibModule.base.add(offset).add(1);
Interceptor.attach(func_addr,{
onEnter(args){
this.arg1 = args[1];
this.arg0 = args[0];
console.log("hookMD5 onEnter arg0===> ",ptr(args[0]).readUtf8String());
// console.log("onEnter arg1===> ",hexdump(ptr(args[1])));
},
onLeave(retval){
// console.log("onLeave arg0===> ",hexdump(ptr(this.arg0)));
// ArrayBuffer
var arrayBuffer = ptr(this.arg1).readByteArray(16);
console.log("hookMD5 result->", arrayBuffer);
// var buffer = Java.array('byte', bytes);
// var ByteString = Java.use("com.android.okhttp.okio.ByteString");
// console.log("result->", bytes);
// console.log("onLeave arg1===> ",);
}
})
```

![图片描述](https://bbs.kanxue.com/upload/attach/202009/715334_DVJRQ6CVDKQ9G8X.png)

图片左边是`hook`的结果，右边是`CyberChef`的加密结果，`hook`多次后发现，`sub_428bc`函数确实是`md5` `hash`函数，第一个参数是输入，第二个参数是用于存储`md5`加密后的`byte`数组的地址。

![图片描述](https://bbs.kanxue.com/upload/attach/202009/715334_HSU6EXMWN7U62JX.png)

而`mytest`函数中`sub_8748`使用`FindCrypt`插件会发现是一个`base64`加密函数，`hook`再次确认，是`base64`加密，函数的第一个参数是我们`md5`加密后的值，第二个参数是固定的`16`.

![图片描述](https://bbs.kanxue.com/upload/attach/202009/715334_7QJ9SF7KBFAVE4C.png)

在`hook`的过程中会发现，`mytest`函数第二个基本块，也就是下面这张图中。也调用了`sub_8748`函数且生成的结果唯一。其值为`4QrcOUm6Wau+VuBX8g+IPg==`

![图片描述](https://bbs.kanxue.com/upload/attach/202009/715334_UWVEN7SDTK8XDXR.png)

直接逆推对应的`md5`哈希值为`e10adc3949ba59abbe56e057f20f883e`

一解`emmmm`, `flag`是`123456`

![图片描述](https://bbs.kanxue.com/upload/attach/202009/715334_CUYSNVSQZ6UQCAV.png)

验证发现是对的。。。

# 小结

在我做这两个练习题的过程中，主要使用静态的代码逆向去进行大概的逻辑分析，使用`frida`的`hook`和主动调用去进行动态验证，压根没有什么`IDA`进行动态调试，所以最后还是喊一句`frida`牛逼！

另外我在做第一题时，想研究一下如何做到静态`jeb`看的字符串和动态使用`Objection`查看的字符串不同的这个技术，可是我竟然在`so`文件中没有找到操作对应位置的`stringID`的地方，懵了，希望知道的大佬不吝赐教2333，最后一句，寒冰师傅牛逼！

附件附上

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

上传的附件：

- [1.zip](javascript:void(0);)

  （8.32MB，46次下载）
