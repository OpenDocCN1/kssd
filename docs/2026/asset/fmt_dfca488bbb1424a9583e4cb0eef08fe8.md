
这时候用 `il2cppdumper` 选择 dump 下来的二进制文件,填上 `so` 在内存中的基址,直接 dump 下来了,甚至不需要手动找 `CodeRegistration` 和 `MetadataRegistration`, `global-metadata.dat` 文件也没有被修改,如果被修改了也可以从内存中 dump 下来

|  |  |

| --- | --- |

| 1  2  3  4  5  6  7  8  9  10  11  12  13  14  15  16  17  18 | `Initializing metadata...`  `Metadata Version:` `29`  `Initializing il2cpp` `file``...`  `Applying relocations...`  `Il2Cpp Version:` `29`  `Detected this may be a dump` `file``.`  `Input` `il2cpp dump address` `or` `input` `0` `to force` `continue``:`  `7ad74c3000`  `Searching...`  `CodeRegistration :` `7ad85254d0`  `MetadataRegistration :` `7ad856ff38`  `Dumping...`  `Done!`  `Generate struct...`  `Done!`  `Generate dummy dll...`  `Done!`  `Press` `any` `key to exit...` |

到此拿到了

|  |  |

| --- | --- |

| 1  2  3  4  5 | `DummyDll`  `dump.cs`  `il2cpp.h`  `script.json`  `stringliteral.json` |

但是还没结束,dump的文件没有导入导出函数的符号,而 `il2cpp` 符号的脚本又是针对于 dump 文件的,所以可以顺手修复一下 dump 的文件头之类的,具体如下操作

首先可以看见 `SECTION_HEADER` 处一片空白,暂且不管

![](https://bbs.kanxue.com/upload/attach/202304/888558_ACY267TPH65CV9F.png)

将 `program_table` 的每一个的 `p_offset` 改成 `p_vaddr`,以及 `p_filesz` 改成 `p_memsz`

![](https://bbs.kanxue.com/upload/attach/202304/888558_GNPEWBB983W8RAN.png)

值得注意的是第十一个表,这个表的结尾地址,正是之前 `SECTION_HEADER` 的开头处,也就是说 `SECTION_HEADER` 本来应该是移动到 `0x13BC000+63352 = 0x13CB778`,虽然这个地方是空的,但是暂且不管,把这个地址填到 `SECTION_HEADER` 处再说

![](https://bbs.kanxue.com/upload/attach/202304/888558_48QRZCBTZF2B9AT.png)

修改完按 `program_table` 按一下 `F5` 就会重新分析了,也可以看到一片空白的 `section_header`

![](https://bbs.kanxue.com/upload/attach/202304/888558_6KSU92M8FPM3N88.png)

直接十六进制复制粘贴,即 `Ctrl+shift+C` `Ctrl+shift+V`,再 `F5` 刷新

可以看见字符还是对应不上,这其实是由 `elf_header` 中的 `e_shtrndx` 指定的,`e_shtrndx` 为`26`,意思是第 26 个指定了字符串的位置

![](https://bbs.kanxue.com/upload/attach/202304/888558_ZPXY7AFTTTZM4HB.png)

索引一下就可以发现问题所在,是因为这个地方被重定位了

![](https://bbs.kanxue.com/upload/attach/202304/888558_3VMHJ8ZHD988V2J.png)

而在第 26 个表中 `s_addr` 为 `0`,说明这个 Section 是不会存在于运行的文件内的,然而往上翻发现了有趣的事情

![](https://bbs.kanxue.com/upload/attach/202304/888558_5DU87YM8DS5VMC6.png)

没错,第 25 个表的 `s_offset` 与第 26 个的一样,但是他有 `s_addr`,所以可以确定第 26 个表的 `s_addr` 与第 25 个的一样,顺手把第 25个 `s_offset` 覆盖成 `s_addr`

不幸的是又出现了空白,不过没关系,直接二进制复制粘贴

![](https://bbs.kanxue.com/upload/attach/202304/888558_6XKH89DECZKTGXF.png)

除此之外,把其他的所有 section 中的 `s_offset` 覆盖成 `s_addr`,最后再按一次 `F5` 重载模板,就可以看见心心念念的符号了

![](https://bbs.kanxue.com/upload/attach/202304/888558_6WSZTCY7FDG32CH.png)

最后记得保存,不然白忙活一场

将 dump 文件载入 `ida` 之后,最好是 `Rebase` 一下,因为是运行态文件,可能有些内存值已经被重定位,因此 `Rebase` 之后可能得到更多符号, `Rebase` 的值就是 dump 文件的载入地址,文件名上就有,比如我的是 `0x7ad74c3000`

![](https://bbs.kanxue.com/upload/attach/202304/888558_SQKYAN674MMQP38.png)

等待 `ida` 初轮分析完之后,加载两次 `il2cppdumper` 的 `ida_with_struct_py3.py` 脚本,选择 `script.json` 和 `il2cpp.h`, 第二次的 `json` 选择 `stringliteral.json`,一阵分析之后,基于 `il2cpp` 能拿到的符号就全拿到了

## Anti-antidebug

经过尝试,发现直接挂 `frida` 服务上游戏,会直接弹窗,但是后开 `frida` 服务不会弹

后来发现 hook `libsec2023.so` 会弹窗,但是有一定延时才会弹出来,猜测调用 `sleep`, 尝试 hook 一下 `sleep` 发现真的不弹了,

包括 `ida` 附加的时候也不会闪退了

hook 的 `frida` 脚本如下,,既然是复现那就我自己来看

|  |  |

| --- | --- |

| 1  2  3  4  5  6  7  8  9  10  11  12  13  14  15  16  17  18 | `/``/``frida` `-``U` `-``f com.com.sec2023.rocketmouse.mouse` `-``l .``/``desktop``/``antidebug.js`  `function AntiDebug()`  `{`  `var sleep_addr` `=` `Module.findExportByName(null,` `"sleep"``);`  `var sleep` `=` `new NativeFunction(sleep_addr,` `"void"``,[``"int"``]);`  `Interceptor.attach(sleep,`  `{`  `onEnter: function (args)`  `{`  `args[``0``]` `=` `ptr(``1000000``);`  `console.log(``"hooked sleep"``);`  `console.log(``'sleep called from:\n'` `+` `Thread.backtrace(this.context, Backtracer.ACCURATE).``map``(DebugSymbol.fromAddress).join(``'\n'``)` `+` `'\n'``);`  `},`  `onLeave: function (returnValue)`  `{`  `}`  `})`  `}AntiDebug();` |

用 `frida` 上号直接可以看见打印 `log`

|  |  |

| --- | --- |

| 1  2  3  4  5  6  7  8  9  10  11  12  13  14  15  16  17  18  19  20  21  22  23  24  25  26  27  28  29  30 | `hooked sleep`  `hooked sleep`  `hooked sleep`  `sleep called` `from``:`  `0x7b3c2bc908` `libsec2023.so!``0x36908`  `0x7b3c2bc7f8` `libsec2023.so!``0x367f8`  `0x7b3c2be708` `libsec2023.so!``0x38708`  `0x7b3c2be5a0` `libsec2023.so!``0x385a0`  `0x7b3c2bc278` `libsec2023.so!``0x36278`  `0x7b3c297f44` `libsec2023.so!``0x11f44`  `0x7bd4bd350c` `libc.so!_ZL15__pthread_startPv``+``0x44`  `0x7bd4b73b10` `libc.so!__start_thread``+``0x44`    `sleep called` `from``:`  `0x7b3c2bc794` `libsec2023.so!``0x36794`  `0x7b3c297f44` `libsec2023.so!``0x11f44`  `0x7bd4bd350c` `libc.so!_ZL15__pthread_startPv``+``0x44`  `0x7bd4b73b10` `libc.so!__start_thread``+``0x44`    `sleep called` `from``:`  `0x7b3c2a80a8` `libsec2023.so!``0x220a8`  `0x7b3c297f44` `libsec2023.so!``0x11f44`  `0x7bd4bd350c` `libc.so!_ZL15__pthread_startPv``+``0x44`  `0x7bd4b73b10` `libc.so!__start_thread``+``0x44`    `hooked sleep`  `sleep called` `from``:`  `0x7bd3e16b24` `libEGL.so!``0x13b24`  `0x7bd4bd350c` `libc.so!_ZL15__pthread_startPv``+``0x44`  `0x7bd4b73b10` `libc.so!__start_thread``+``0x44` |

最后一个是弹出了一个什么权限我拒绝之后 hook 到的,应该是没用的,那么主要就是分析前面的 `libsec2023.so` 的函数

`0x11f44` 是函数 `0x11f24` 的一部分,这个函数看上去像是某种初始化,说明重点不在这,重点在下一步 call 的函数

![](https://bbs.kanxue.com/upload/attach/202304/888558_9JWVWJY4JGVC6K4.png)

`0x220a8` 是函数 `0x22080` 的一部分,这个 `sleep` 在函数的头部,但是这整个函数充斥着混淆,太难了不看

![](https://bbs.kanxue.com/upload/attach/202304/888558_SX6HY5SAKJEGDJ3.png)

`0x36794` 是函数 `0x36784` 的一部分

![](https://bbs.kanxue.com/upload/attach/202304/888558_AT8NP6XY69QQEDS.png)

![](https://bbs.kanxue.com/upload/attach/202304/888558_BXXZ4XHWPTS7B6Q.png)

![](https://bbs.kanxue.com/upload/attach/202304/888558_53SS6CBP7ZFCNA2.png)

看上去好像没什么异常的地方

`0x36908` 是 `0x36818` 的一部分,而这个函数...发现了很熟悉的操作

![](https://bbs.kanxue.com/upload/attach/202304/888558_W2EAZP5T6MSANTB.png)

以我上一次做这个的经验,这里是一个字符串解密,解密函数 `0x36f6c`,修复去混淆之后重建函数,逻辑如下

![](https://bbs.kanxue.com/upload/attach/202304/888558_86WRR3UM59GM3PW.png)

解密之后

![](https://bbs.kanxue.com/upload/attach/202304/888558_KR2P5XZJTA7QNDF.png)

`0x36818` 剩下的逻辑也没啥好看的,其实是看不懂,用插件确实能找到 `CRC` 常数,不过却没有引用,算了算了,能跑就不管了,反反调试就到这吧

另外提一嘴,关于 `ida` 调试的事情

首先一定要新开 `ida`,不要用分析了 `so` 的 `ida` 附加,否则退出调试的时候会卡死,之前的分析就炸了,推荐是动静结合来分析

还有一件很重要的事情是关闭一些异常处理

`Debugger Options...->Edit exceptions->`

`SIGPWR` `SIGXCPU` 右键 edit,去掉挂起程序的勾,并勾上通过引用,report 选 log 或者 Silent 都行

![](https://bbs.kanxue.com/upload/attach/202304/888558_VYC6TRF46GM4FGK.png)

这一点可能是因为 `unity` 的问题或者是 Android 的问题,调着调着就会弹这俩个异常,如果挂起程序,程序就会崩溃,程序崩溃又会导致 `ida` 崩溃,而设置完之后就不会出现这个情况了

还有一件事,程序暂停调试的时候不要点击屏幕,否则会出现和前面异常挂起一样的问题

# 开始分析

## getflag

按照说明,游戏内获得 `1000` 金币的时候,会出现 `flag`

那么肯定需要有个地方来判断金币数量,金币一般是用 `coin` 表示,在 `dump.cs` 中可以找到

![](https://bbs.kanxue.com/upload/attach/202304/888558_8JXUKGAUZ98ZK5U.png)

同时发现 `CollectCoin` 函数

|  |  |

| --- | --- |

| 1  2 | `/``/` `RVA:` `0x4652E4` `Offset:` `0x4652E4` `VA:` `0x7AD79282E4`  `private void CollectCoin(Collider2D coinCollider) { }` |

在有符号的 dump 中可以直接跳转到这个函数,也很容易发现相关逻辑

![](https://bbs.kanxue.com/upload/attach/202304/888558_FV6KNDC2X67789W.png)

用 `frida` 改一下字节,比较`1000`改成比较`0`,这样随便捡一个金币就可以显示 `flag` 了

```
function main()
{
var g_libil2cpp_addr = Module.findBaseAddress("libil2cpp.so");
var coin_cmp = g_libil2cpp_addr.add(0x4653cc);
Memory.protect(coin_cmp, 4, "rwx");
Memory.writeInt(coin_cmp, 0x7100001f);//cmp w0,0
Memory.protect(coin_cmp, 4, "rx");
}main();
```

效果如下

![](https://bbs.kanxue.com/upload/attach/202304/888558_Q7HDAVJV6P3V9KH.png)

## 注册机

这游戏自带外挂,但是必须输入 `token` 相对应的密码才能够使用,所以所谓注册机就是关于这个密码的逆向

注册机界面如下

![](https://bbs.kanxue.com/upload/attach/202304/888558_CRAZKKFKNVFK868.png)

本来没有什么下手点,但是从 `dump.cs` `coin` 往下随便翻翻,看到了 `SmallKeyboard` 相关的函数

甚至专门混淆了函数名,简直是此地无银三百两

|  |  |

| --- | --- |
