
```
function dump_Dex(fun_Name, apk_Name) {
    if (fun_Name !== '') { // 判断传入的参数 fun_Name 是否为空字符串
        var hook_fun = Module.findExportByName("libart.so", fun_Name); // 在 libart.so 动态库中查找指定的导出函数地址
        Interceptor.attach(hook_fun, { // 使用 Interceptor 对导出函数进行挂钩
            onEnter: function (args) { // 进入函数之前的回调函数
                var begin = 0; // 初始化一个变量用于保存 Dex 文件的起始地址
                var dex_flag = false; // 初始化一个标志位，表示是否符合 Dex 文件的魔数
                dex_flag = DexFileVerifier(args[0]); // 调用 DexFileVerifier 函数对传入的第一个参数进行验证
                if (dex_flag === true) { // 如果验证通过
                    begin = args[0]; // 保存 Dex 文件的起始地址
                }
                if (begin === 0) { // 如果起始地址为 0
                    dex_flag = DexFileVerifier(args[1]); // 再次调用 DexFileVerifier 函数对传入的第二个参数进行验证
                    if (dex_flag === true) { // 如果验证通过
                        begin = args[1]; // 保存 Dex 文件的起始地址
                    }
                }
                if (dex_flag === true) { // 如果验证通过
                    console.log("magic : " + Memory.readUtf8String(begin)); // 打印输出 Dex 文件的魔数
                    var address = parseInt(begin, 16) + 0x20; // 计算 Dex 文件大小字段的地址
                    var dex_size = Memory.readInt(ptr(address)); // 读取 Dex 文件大小
                    console.log("dex_size :" + dex_size); // 打印输出 Dex 文件的大小
                    var dex_path = "/data/data/" + apk_Name + "/" + dex_size + ".dex"; // 构建 Dex 文件保存路径
                    var dex_file = new File(dex_path, "wb"); // 创建一个文件对象用于写入 Dex 文件
                    dex_file.write(Memory.readByteArray(begin, dex_size)); // 将 Dex 文件内容写入文件
                    dex_file.flush(); // 刷新文件缓冲区
                    dex_file.close(); // 关闭文件
                }
            },
            onLeave: function (retval) { // 函数返回之后的回调函数
            }
        });
    } else {
        console.log("Error: no hook function."); // 如果传入的参数 fun_Name 为空字符串，则打印错误消息
    }
}
```

## 3.4 调用方法

```
var fun_Name = find_hook_fun();
var apk_Name = 'com.xxx.xxx' // 给定要hook的apk

dump_Dex(fun_Name, apk_Name);
// frida -U -f com.xxx.xxx -l dumpdex.js --no-pause
```

在手机上启动`fs`执行`frida -U -f com.xxx.xxx -l dumpdex.js --no-pause` 脱壳后的dex保存在`/data/data/应用包名/目录下`

# 0x04 结论

缺点：普通加固可以脱壳,对于类抽取等加固脱出的只是个空壳，需要做指令Dump以及Patch到脱出的Dex文件中，后续如有遇到该类型apk再给大家做对应的分享。

无论是工具脱壳，还是自行脱壳，最根本的思想还是在dex加载流程中进行dump，整个流程中可脱壳时机有很多，但脱壳的时机不同，效果也会不同，我无非是站在前人的肩膀上快速的定位了较好的脱壳点。

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

最后于 2023-7-4 09:38 被行简编辑，原因：

[#逆向分析](https://bbs.kanxue.com/forum-161-1-118.htm)

[#HOOK注入](https://bbs.kanxue.com/forum-161-1-125.htm)

[#源码框架](https://bbs.kanxue.com/forum-161-1-127.htm)

[#工具脚本](https://bbs.kanxue.com/forum-161-1-128.htm)
