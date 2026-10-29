
在这个函数中有很多复杂且奇怪的函数，并且也出现了函数指针和 `1000` 多的数字，因此可以猜想这里主要做的事情是准备 Java 层环境，之后调用 Java 函数，将伪代码中 `JNIEnv*` 参数类型标注出来。刷新伪代码后能够很明显看到先 `new` 了一个 Java 层的 `bytearray`，然后 `findclass`，通过 `GetStaticMethodID` 得到函数，最后在 `sub_3ABBC` 中调用这个函数。

![](https://bbs.kanxue.com/upload/attach/202304/831526_79JJPP5SJ5XF6ZV.jpg)

这里我首先对 `findclass` 进行 `hook`，发现类名是空的，静态函数名称能得到是 `encrypt`，基本可以确定无误。用 `jadx` 打开 `dex` 发现并没有这个函数，因此怀疑是通过 `RegisterNatives` 动态注册的。使用 `frida` 对 `RegisterNatives` 进行 `hook`，发现了一条记录：

|  |  |
| --- | --- |
| 1 | `[RegisterNatives] java_class: com.tencent.games.sec2023.Sec2023Application name: initialize sig: ()I fnPtr: 0x7cfe0d5abc fnOffset: 0x7cfe0d5abc libsec2023.so!0xfabc callee: 0x7cfe0d5ed0 libsec2023.so!JNI_OnLoad+0x3b0` |

但是经过排查，很明显不是上面的 `encrypt` 函数。思考一下，除了动态注册以外，有可能是动态加载了 `dex`，于是使用 `frida-dexdump` 将内存中的 `dex` dump 下来。果然在某个 `dex` 中发现了该函数。

![](https://bbs.kanxue.com/upload/attach/202304/831526_C8YED6W979TA64Z.jpg)

但是其控制流也被混淆了，好在函数本身并不复杂，因此可以将伪代码直接复制新建一个 Java 工程，测试功能完好之后对其进行调试分析，将函数逻辑梳理一下，得到其逻辑代码如下：

```cpp
char* encrypt(char* var0) {
    // 输入 109, -108, -54, -28
    // 输出 -6, 23, -40, 16
    // 输入 109, -102, -10, 14
    // 输出 46, 23, -52, 119
    int var5 = 0;
    int var6 = 0;

    var6 = (var0[3] & 0xff) + (var0[0] << 24 & -0x1000000) +
           (var0[1] << 16 & 0xff0000) + (var0[2] << 8 & 0xff00);
    var5 = ((unsigned int)var6 >> 7) | (var6 & 0x7f) << 25;

    char *var1 = new char[4];

    var1[0] = (unsigned char)(var5 >> 24 & 255);
    var1[1] = (unsigned char)(var5 >> 16 & 255);
    var1[2] = (unsigned char)(var5 >> 8 & 255);
    var1[3] = (unsigned char)(var5 & 255);

    unsigned char data[] = {50, -51, -1, -104, 25, -78, 124, -102};
    for (int i = 0; i < 4; i++) {
        var1[i] ^= data[i % 8];
        var1[i] = (unsigned char)(var1[i] + i);
    }

    return var1;
}
```

经过分析也能得出对应的逆算法如下：

```cpp
char* decrypt(char* var0) {
    unsigned char data[] = {50, -51, -1, -104, 25, -78, 124, -102};

    for (int i = 0; i < 4; i++) {
        var0[i] = (unsigned char)(var0[i] - i);
        var0[i] ^= data[i % 8];
    }

    int var5 = (var0[3] & 0xff) + (var0[2] << 8 & 0xff00) +
               (var0[1] << 16 & 0xff0000) +
               (var0[0] << 24 & -0x1000000);
    var5 = ((unsigned int)var5 << 7) | ((unsigned int)var5 >> 25);

    char* var1 = new char[4];
    var1[0] = (unsigned char)(var5 >> 24 & 255);
    var1[1] = (unsigned char)(var5 >> 16 & 255);
    var1[2] = (unsigned char)(var5 >> 8 & 255);
    var1[3] = (unsigned char)(var5 & 255);

    return var1;
}
```

使用 `frida hook` 得到的结果与上面算法计算得到的结果一致。

![](https://bbs.kanxue.com/upload/attach/202304/831526_2WYMZ6VQR62BDKX.jpg)

梳理完这个函数，那么算法在 `libsec2023.so` 的部分就已经结束了。按照动态流程分析，执行完 `sub_31164` 后会跳回到 `libil2cpp.so + 0x465AB4` 中。从 Java 层这个 `encrypt` 返回的结果与 `0x465AB4` 所拦截的结果是一致的。所以 `libsec2023.so` 部分到此结束。

### `libil2cpp.so ==> sub_465AB4`

这个函数是从 `libsec2023.so` 返回所进入的函数。首先根据伪代码观察其流程，主要做的事情是初始化了一些数组和对象，将上一个 `encrypt` 的结果传入 `sub_46AD44` 函数中进行加密，然后在一个 `do_while` 循环中进行第二次加密。最后如果 `v28` 等于 `0` 并且 `v27` 与 `token` 的值相等，就开启 mod 成功。这里手动 `patch` 判断语句可以实现作弊功能，所以整体思路是对的。

![](https://bbs.kanxue.com/upload/attach/202304/831526_GMSHCWNK9UY485B.jpg)

### `libil2cpp.so ==> sub_46AD44`

首先来分析第一个 `sub_46AD44` 函数。这个函数非常复杂，主要是重要的字段名信息还有函数名信息都被混淆掉了。

![](https://bbs.kanxue.com/upload/attach/202304/831526_3WQJW93M9PHR2QU.jpg)

整个类中做了什么事情？首先会初始化一些参数，最重要的是某个迭代值（类内偏移 `0x2c`）、`action` 词典（类内偏移 `0x38`）、码表（类内偏移 `0x10`）。该类在初始化的时候会将 `0x1`-`0x16` 的值与某个操作函数的对应关系注册在 `action` 词典中，然后在一个大循环中随着迭代值的变化在码表中选择不同的 `action`，类似于状态机的实现。下面这个 `sub_46AD44` 函数内的循环即是主要逻辑。

![](https://bbs.kanxue.com/upload/attach/202304/831526_SFR9EFYEGHJ4E6W.jpg)

最重要的事情是，无论输入值是多少，该状态机的码表和操作序列都是一样的。因此可以 dump 码表和操作序列，dump 出来的码表和操作序列如下：

|  |  |
| --- | --- |
