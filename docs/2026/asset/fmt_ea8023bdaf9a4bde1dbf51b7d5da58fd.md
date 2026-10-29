
# [原创]脱壳成长之路(2)-2代壳进阶(1)

- 样本：很老的x度加固
- 分析工具：ida、jeb、010、apktool
- NativeHook：基于Substrate的Nativehook
- Android系统：4.4

本次脱壳，并没有完全像上一篇一样，一句一句的分析，而是直接从整体看，去想办法脱壳。不过为了碰一下反调试，还是先进入了`JNIONLOAD`层。下面是记录

## 进入`JNIONLOAD`

![](https://bbs.kanxue.com/upload/attach/202002/762912_5RAD25A56RZHNNK.jpg)

首先看到Java层有第一个反调试。这个太老了，所以网络上资料非常多。我直接采用hook方法去过掉这个校验。

```
int new_debug() {
    return 0;
}
```

```
if (old_debug == NULL) {
    elfHook("libdvm.so", "_Z25dvmDbgIsDebuggerConnectedv", (void *) new_debug,
            (void **) &old_debug);
}
```

然后我又增加了一些常用的过反调试检测的手段，比如下面这段等等.....

```
char *new_fgets(char *buf, int bufsize, FILE *stream) {
    char *trace = "TracerPid:\t1";
    if (strstr(buf, trace) != NULL) {
        strcpy(buf, "TracerPid:\t0");
    }
    return old_fgets(buf, bufsize, stream);
}
```

继续下一步，使用NDK hook 卡死`Jnionload`函数。

```
jint new_JNI_OnLoad(JavaVM *vm, void *reserved) {
    log("%s", "jni on load");
    while (!debug) {
    }
    log("%s", "in jni onload");
    debug = true;
    return old_JNI_OnLoad(vm, reserved);
}
```

然后启动App，并尝试用IDA附加，发现有下面两个进程。

![](https://bbs.kanxue.com/upload/attach/202002/762912_A3EDM9RWK4U3999.png)

使用`ps`命令，查看哪个进程是受精卵孵化的。

![](https://bbs.kanxue.com/upload/attach/202002/762912_PAUA75KFDWP5H7Q.png)

然后附加3622进程，报错。

![](https://bbs.kanxue.com/upload/attach/202002/762912_HYGGHFAD3RH868F.png)

我们搞老壳有个好处就是，网上能参考的资料比较多。通过一番查找，发现这个错误很可能是孵化进程后，再`ptrace`自己，然后造成我们不能附加。

那么我解决的手段很暴力，直接干掉子进程，然后再附加。幸运的是，成功了。

附加成功后，修改寄存器中`debug`数值，强制跳转到app 原`jnionload`函数，这里一定要注意`T`指令和`A`指令转换。

然后成功进入`JNIONLOAD`。
