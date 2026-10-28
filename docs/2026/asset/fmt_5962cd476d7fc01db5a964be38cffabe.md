
进入`/path/to/xxx`目录，找到`xxx.json`。

在表达式`cvar = avar + bvar`中，加法运算符（`+`）将`avar`与`bvar`相加，得到它们的和`cvar`

在`List.of(arg0, arg1, arg2)`中，`List`接口的工厂方法`of()`接受一系列的元素，返回包含它们的只读列表。

之后我们这样调用`cmd`命令：`cmd arg0 arg1 arg2`。

```
if (condVar > someVal) {console.log("xxx")}
```

```
| 1  2  3  4  5  6  7  8  9  10  11  12  13  14  15  16  17  18  19  20  21  22  23  24  25  26  27  28  29  30  31  32  33  34 | `____`  `/` `_  |   Frida` `15.0``.``0` `-` `A world``-``class` `dynamic instrumentation toolkit`  `| (_| |`  `> _  |   Commands:`  `/``_``/` `|_|` `help`      `-``> Displays the` `help` `system`  `. . . .` `object``?` `-``> Display information about` `'object'`  `. . . .       exit``/``quit` `-``> Exit`  `. . . .`  `. . . .   More info at https:``/``/``frida.re``/``docs``/``home``/`  `` Spawning `Polaris_chs.exe`... ``  `` Spawned `Polaris_chs.exe`. Resuming main thread! ``  `[Local::Polaris_chs.exe]``-``> A01.MJO` `0xbd55010` `+``0x74` `－東京　１９２３－`  `A01.MJO` `0xbd55010` `+``0xb9` `雪が降っていた。`  `A01.MJO` `0xbd55010` `+``0xe0` `目を開けると、ゆらゆらと舞い降りる六花が見えた。`  `A01.MJO` `0xbd55010` `+``0x121` `耳からは、なにかが燃える音もした。`  `A01.MJO` `0xbd55010` `+``0x1ba` `ツバキ「……ここは？」`  `A01.MJO` `0xbd55010` `+``0x1e1` `目の前にはどこかの大きな屋敷。`  `A01.MJO` `0xbd55010` `+``0x210` `暗い夜を照らすように、煌々と燃えていた。`  `A01.MJO` `0xbd55010` `+``0x283` `主人公「気がついたか」`  `A01.MJO` `0xbd55010` `+``0x2aa` `見知らぬ男の人の声。`  `A01.MJO` `0xbd55010` `+``0x2cf` `わたしのすぐ隣に立っていた。`  `A01.MJO` `0xbd55010` `+``0x336` `主人公「お前……自分の名前がわかるか？」`  `A01.MJO` `0xbd55010` `+``0x3a0` `ツバキ「ううん、わからない……」`  `A01.MJO` `0xbd55010` `+``0x3d1` `何故だか思い出せなかった。`  `A01.MJO` `0xbd55010` `+``0x3fc` `ここが、どこなのかも分からなかった。`  `A01.MJO` `0xbd55010` `+``0x46b` `主人公「では、これを持っていけ」`  `A01.MJO` `0xbd55010` `+``0x4cd` `ツバキ「あ、はい……」`  `A01.MJO` `0xbd55010` `+``0x4f4` `そう言って、たくさんの金貨やお金をくれた。`  `A01.MJO` `0xbd55010` `+``0x52f` `他にも、何かの手紙のような物もわたしに手渡した。`  `A01.MJO` `0xbd55010` `+``0x5a7` `ツバキ「えと、あなたは？」`  `A01.MJO` `0xbd55010` `+``0x606` `主人公「通りすがりだ」`  `A01.MJO` `0xbd55010` `+``0x62d` `それだけを言うと、軽く手を上げて背を向ける男の人。`  `A01.MJO` `0xbd55010` `+``0x670` `そのまま去って行くのかと思うと……`  `A01.MJO` `0xbd55010` `+``0x69f` `一度だけ振り返り……` |
```

对照我们用[`binary_text.py`](https://bbs.kanxue.com/elink@e97K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6k6N6i4u0A6f1$3W2*7N6h3E0#2i4K6u0r3c8$3q4D9k6$3q4E0k6g2u0W2N6X3g2J5M7$3g2Q4x3V1k6T1L8r3!0T1i4K6u0r3L8h3q4K6N6r3g2J5i4K6u0r3M7$3y4J5K9i4m8@1i4K6u0r3j5X3W2F1j5i4u0&6i4K6g2X3N6r3g2^5N6q4)9J5k6i4m8&6)提取的文本， 偏移（当前位置指针-解密缓冲区基址）正好是文件中的偏移，至此这个游戏动态汉化的理论研究已经完成，之后就是用`c`和内联汇编写程序实践了。下面是我们用于翻译的文本格式，白点列用于原文，黑点列用于译文，每行是`●|num|addr|size●`的索引格式，详见[`binary_text.h`](https://bbs.kanxue.com/elink@7a9K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6k6N6i4u0A6f1$3W2*7N6h3E0#2i4K6u0r3c8$3q4D9k6$3q4E0k6g2u0W2N6X3g2J5M7$3g2Q4x3V1k6T1L8r3!0T1i4K6u0r3L8h3q4K6N6r3g2J5i4K6u0r3M7$3y4J5K9i4m8@1i4K6u0r3j5X3W2F1j5i4u0&6i4K6g2X3N6r3g2^5N6q4)9J5k6h3R3`.)

|  |  |

| --- | --- |

| 1  2  3  4  5  6  7  8  9  10  11  12  13  14  15  16  17  18  19  20  21  22  23  24  25  26  27  28  29  30  31  32  33  34  35  36  37  38  39  40  41  42  43  44  45  46  47  48  49  50  51  52  53 | `○``00001``|``000074``|``012``○ −東京　１９２３−`  `●``00001``|``000074``|``012``● −東京　１９２３−`    `○``00002``|``0000B9``|``010``○ 雪が降っていた。`  `●``00002``|``0000B9``|``010``● 雪が降っていた。`    `○``00003``|``0000E0``|``030``○ 目を開けると、ゆらゆらと舞い降りる六花が見えた。`  `●``00003``|``0000E0``|``030``● 目を開けると、ゆらゆらと舞い降りる六花が見えた。`    `○``00004``|``000121``|``022``○ 耳からは、なにかが燃える音もした。`  `●``00004``|``000121``|``022``● 耳からは、なにかが燃える音もした。`    `○``00005``|``0001AB``|``006``○ ツバキ`  `●``00005``|``0001AB``|``006``● ツバキ`    `○``00006``|``0001BA``|``010``○ 「……ここは？」`  `●``00006``|``0001BA``|``010``● 「……ここは？」`    `○``00007``|``0001E1``|``01E``○ 目の前にはどこかの大きな屋敷。`  `●``00007``|``0001E1``|``01E``● 目の前にはどこかの大きな屋敷。`    `○``00008``|``000210``|``028``○ 暗い夜を照らすように、煌々と燃えていた。`  `●``00008``|``000210``|``028``● 暗い夜を照らすように、煌々と燃えていた。`    `○``00009``|``000274``|``006``○ 主人公`  `●``00009``|``000274``|``006``● 主人公`    `○``00010``|``000283``|``010``○ 「気がついたか」`  `●``00010``|``000283``|``010``● 「気がついたか」`    `○``00011``|``0002AA``|``014``○ 見知らぬ男の人の声。`  `●``00011``|``0002AA``|``014``● 見知らぬ男の人の声。`    `○``00012``|``0002CF``|``01C``○ わたしのすぐ隣に立っていた。`  `●``00012``|``0002CF``|``01C``● わたしのすぐ隣に立っていた。`    `○``00013``|``000327``|``006``○ 主人公`  `●``00013``|``000327``|``006``● 主人公`    `○``00014``|``000336``|``022``○ 「お前……自分の名前がわかるか？」`  `●``00014``|``000336``|``022``● 「お前……自分の名前がわかるか？」`    `○``00015``|``000391``|``006``○ ツバキ`  `●``00015``|``000391``|``006``● ツバキ`    `○``00016``|``0003A0``|``01A``○ 「ううん、わからない……」`  `●``00016``|``0003A0``|``01A``● 「ううん、わからない……」`    `○``00017``|``0003D1``|``01A``○ 何故だか思い出せなかった。`  `●``00017``|``0003D1``|``01A``● 何故だか思い出せなかった。`    `○``00018``|``0003FC``|``024``○ ここが、どこなのかも分からなかった。`  `●``00018``|``0003FC``|``024``● ここが、どこなのかも分からなかった。` |

其实有时候如果我们实在找不到文本标识的偏移，也可以强行把游戏从从头到尾过一遍，把每句输出的文本提取出来。汉化的时候，再用`hashmap`或`Longest Common Subsequence` `dp`计算当前文本与文本数据库中的相似度，选取相似度最高的匹配用于替换。

## 0x4 IAT hook与Inline hook， LoadDll

以上，我们谈了谈如何进行动态汉化的相关分析，方便起见都是用的`frida`进行`hook`。但是`frida`属于测试环境，不可能要求每个人电脑上都有这个环境，而且也可能有`python`版本冲突等问题。需要用尽可能少的依赖制作汉化，因此就要结合`C`与内联汇编来写汉化程序了。在制作汉化程序之前，来科普一下汉化游戏常用的`hook`方法。

### IAT hook

即把相应函数的导入表的地址（`FirstThunk`）替换成我们的函数，实现`hook`。关于`IAT`结构和导入表相关内容，可以参考我之前写的文章[`SimpleDpack`](https://bbs.kanxue.com/elink@eb8K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2M7$3y4Z5L8X3g2W2i4K6u0W2L8h3!0W2i4K6u0r3M7r3!0K6N6s2y4Q4x3V1k6e0K9h3#2H3L8r3g2p5M7r3q4U0K9#2)9J5c8R3`.`.)。下面是`IAT hook`的代码，兼容`64`位，详见我的`github`， [`win_hook,c`](https://bbs.kanxue.com/elink@425K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6k6N6i4u0A6f1$3W2*7N6h3E0#2i4K6u0r3c8$3q4D9k6$3q4E0k6g2u0W2N6X3g2J5M7$3g2Q4x3V1k6T1L8r3!0T1i4K6u0r3L8h3q4K6N6r3g2J5i4K6u0r3M7$3y4J5K9i4m8@1i4K6u0r3N6$3W2F1k6r3!0%4M7#2)9J5c8Y4N6A6L8W2)9#2k6X3S2G2L8$3E0Q4x3X3g2U0)

|  |  |

| --- | --- |

| 1  2  3  4  5  6  7  8  9  10  11  12  13  14  15  16  17  18  19  20  21  22  23  24  25  26  27  28  29  30  31  32  33  34  35  36  37  38  39  40  41 | `BOOL` `iat_hook(LPCSTR targetDllName, PROC pfnOrg, PROC pfnNew)`  `{`  `return` `iat_hook_module(targetDllName, NULL, pfnOrg, pfnNew);`  `}`    `BOOL` `iat_hook_module(LPCSTR targetDllName, LPCSTR moduleDllName, PROC pfnOrg, PROC pfnNew)`  `{;`  `#ifdef _WIN64`  `#define VA_TYPE ULONGLONG`  `#else`  `#define VA_TYPE DWORD`  `#endif`  `DWORD dwOldProtect` `=` `0``;`  `VA_TYPE imageBase` `=` `GetModuleHandleA(moduleDllName);`  `LPBYTE pNtHeader` `=` `*``(DWORD` `*``)((LPBYTE)imageBase` `+` `0x3c``)` `+` `imageBase;`  `#ifdef _WIN64`  `VA_TYPE impDescriptorRva` `=` `*``((DWORD``*``)&pNtHeader[``0x90``]);`  `#else`  `VA_TYPE impDescriptorRva` `=` `*``((DWORD``*``)&pNtHeader[``0x80``]);`  `#endif`  `PIMAGE_IMPORT_DESCRIPTOR pImpDescriptor` `=` `(PIMAGE_IMPORT_DESCRIPTOR)(imageBase` `+` `impDescriptorRva);`  `for` `(; pImpDescriptor``-``>Name; pImpDescriptor``+``+``)` `/``/` `find the dll IMPORT_DESCRIPTOR`  `{`  `LPCSTR pDllName` `=` `(LPCSTR)(imageBase` `+` `pImpDescriptor``-``>Name);`  `if` `(!_stricmp(pDllName, targetDllName))` `/``/` `ignore case`  `{`  `PIMAGE_THUNK_DATA pFirstThunk` `=` `(PIMAGE_THUNK_DATA)(imageBase` `+` `pImpDescriptor``-``>FirstThunk);`  `for` `(; pFirstThunk``-``>u1.Function; pFirstThunk``+``+``)` `/``/` `find the iat function va`  `{`  `if` `(pFirstThunk``-``>u1.Function` `=``=` `(VA_TYPE)pfnOrg)`  `{`  `VirtualProtect((LPVOID)&pFirstThunk``-``>u1.Function,` `4``, PAGE_EXECUTE_READWRITE, &dwOldProtect);`  `pFirstThunk``-``>u1.Function` `=` `(VA_TYPE)pfnNew;`  `VirtualProtect((LPVOID)&pFirstThunk``-``>u1.Function,` `4``, dwOldProtect, &dwOldProtect);`  `return` `TRUE;`  `}`  `}`  `}`  `}`  `return` `FALSE;`  `}` |

### Inline hook

`IAThook`只适用于动态链接外部`DLL`的函数，对于`exe`内部的函数，就需要`Inline hook`了，操作如下：

1. 将`hook`点的前`6`个字节替换成`FF 24 xxxxxxxx`，或前`5`个字节替换为`E9 xxxxxxxx`，对应绝对地址和相对地址跳转，`xxxxxxxx`为`hook`我们编写的函数地址。

2. 将原函数开头处被破坏的完整指令搬到`Trampoline`（`VirtualAlloc`的一段可执行区域），后面跟一条`jmp`指令，跳转到原函数`jmp`机器码（`5`位或`6`位）替换后的下一条完整指令

3. 之后可以通过`jmp Trampoline`来返回原函数

不过我们不用再自己解析函数开头处的机器码了，直接用微软的`detours`的`Inline hook`即可。细节上和上述可能有些区别，不过原理都是一样的。`detours`用法如下：

|  |  |

| --- | --- |
