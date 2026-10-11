
# MP3 RM Converter V1.3 注册算法分析

> 原文：[`www.pediy.com/kssd/pediy10/74104.html`](https://www.pediy.com/kssd/pediy10/74104.html)

**【文章标题】**: MP3 RM Converter V1.3 注册算法分析

**【文章作者】**: `xuecrack`

**【作者邮箱】**: `243261920@163.com`

**【作者主页】**: `http://hi.csdn.net/haart`

**【作者 QQ 号】**: `243261920`

**【软件名称】**: MP3 RM Converter V1.3

**【下载地址】**: `http://www.audiotoolsfactory.com`（Homepage）

**【加壳方式】**: 未加壳

**【编写语言】**: Borland Delphi 6.0 - 7.0

**【使用工具】**: `OD`，`PEID`

**【操作平台】**: WinXP Sp3

**【软件介绍】**: 一个不错的音频转换软件，支持常见的音频格式相互转换

**【作者声明】**: 只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教! 本文仅供学习参考!

---

## 详细过程

上次写了一篇破文，最后没能分析出注册算法，有点不爽。OK，今天这一篇破文把软件的注册算法给分析出来！

首先把软件扔给 `PEID` 老兄，他嗲嗲的告诉我没加壳，接着试注册：

```
USER:PEDIY
CODE:123456789
```

弹出错误提示框，内容为“`invalid register code! please retry!`”。接下来用 `OD` 的超级字符串查找插件，查找 `UNICODE`。查找“`code`”找到：

> 超级字串参考 ＋        , 条目 462
>
> 地址=`0049A9B5`
>
> 反汇编=`MOV EAX,MP3_RM_C.0049AB44`
>
> 文本字串=`invalid register code! please retry!`

双击该行进入反汇编窗口。定位在 `0049A9B5` 这一行。

```
0049A9AA  |.   6A 00            PUSH 0                               ; /Arg1 = 00000000
0049A9AC  |.   66:8B0D 38AB4> MOV CX,WORD PTR DS:[49AB38]             ; |
0049A9B3  |.   B2 02           MOV DL,2                              ; |
0049A9B5  |.   B8 44AB4900    MOV EAX,MP3_RM_C.0049AB44              ; |invalid register code! please retry!
0049A9BA  |.   E8 9D7CF9FF    CALL MP3_RM_C.0043265C                 ; \MP3_RM_C.0043265C
```

向上找到段首下断，再次输入用户名，假码注册。程序被断下来，`F8` 单步跟踪，一些对注册没影响的 `CALL` 略过，跟踪过程记录如下。

```
0049A928  /.   55             PUSH EBP
0049A929  |.   8BEC           MOV EBP,ESP
0049A92B  |.   33C9           XOR ECX,ECX
0049A92D  |.   51             PUSH ECX
0049A92E  |.   51             PUSH ECX
0049A92F  |.   51             PUSH ECX
0049A930  |.   51             PUSH ECX
0049A931  |.   51             PUSH ECX
0049A932  |.   51             PUSH ECX
0049A933  |.   53             PUSH EBX
0049A934  |.   56             PUSH ESI
0049A935  |.   57             PUSH EDI
0049A936  |.   8BF8           MOV EDI,EAX
0049A938  |.   33C0           XOR EAX,EAX
0049A93A  |.   55             PUSH EBP
0049A93B  |.   68 2AAB4900    PUSH MP3_RM_C.0049AB2A
0049A940  |.   64:FF30        PUSH DWORD PTR FS:[EAX]
0049A943  |.   64:8920        MOV DWORD PTR FS:[EAX],ESP
0049A946  |.   C605 0C2D4A00> MOV BYTE PTR DS:[4A2D0C],1
0049A94D  |.   8D55 F4        LEA EDX,[LOCAL.3]
0049A950  |.   8B87 1C030000  MOV EAX,DWORD PTR DS:[EDI+31C]
0049A956  |.   E8 3132FCFF    CALL MP3_RM_C.0045DB8C                 ;  取用户名
0049A95B  |.   8B45 F4        MOV EAX,[LOCAL.3]                      ;  把用户名放到 EAX
0049A95E  |.   8D55 FC        LEA EDX,[LOCAL.1]
0049A961  |.   E8 3EDDF6FF    CALL MP3_RM_C.004086A4
0049A966  |.   8D55 F0        LEA EDX,[LOCAL.4]
0049A969  |.   8B45 FC        MOV EAX,[LOCAL.1]
0049A96C  |.   E8 67DDF6FF    CALL MP3_RM_C.004086D8
0049A971  |.   8B55 F0        MOV EDX,[LOCAL.4]
0049A974  |.   8D45 FC        LEA EAX,[LOCAL.1]
0049A977  |.   E8 9898F6FF    CALL MP3_RM_C.00404214
0049A97C  |.   BB 15000000    MOV EBX,15
0049A981  |.   BE F8174A00    MOV ESI,MP3_RM_C.004A17F8
0049A986  |>   8B45 FC        /MOV EAX,[LOCAL.1]                      ;  用户名地址放到 EAX
0049A989  |.   8B16           |MOV EDX,DWORD PTR DS:[ESI]              ;  DS:[004A17F8]=0049A6E4 (MP3_RM_C.0049A6E4), ASCII "VS88T6-Vs86"
0049A98B  |.   E8 F89BF6FF    |CALL MP3_RM_C.00404588                ;  用户名与程序“固有注册名”比较，判断是否相等
0049A990  |.   75 09          |JNZ SHORT MP3_RM_C.0049A99B
0049A992  |.   C605 0C2D4A00> |MOV BYTE PTR DS:[4A2D0C],0             ;  把 DS:[004A2D0C]置 0
0049A999  |.   EB 06          |JMP SHORT MP3_RM_C.0049A9A1
0049A99B  |>   83C6 04        |ADD ESI,4
0049A99E  |.   4B             |DEC EBX
0049A99F  |.^  75 E5          \JNZ SHORT MP3_RM_C.0049A986
0049A9A1  |>   803D 0C2D4A00> CMP BYTE PTR DS:[4A2D0C],0              ;  判断 DS:[004A2D0C]是否为 0，不为 0 就失败
0049A9A8  |.   74 1A          JE SHORT MP3_RM_C.0049A9C4              ;  必须跳，不跳就失败
0049A9AA  |.   6A 00          PUSH 0                               ; /Arg1 = 00000000
0049A9AC  |.   66:8B0D 38AB4> MOV CX,WORD PTR DS:[49AB38]             ; |
0049A9B3  |.   B2 02         MOV DL,2                              ; |
0049A9B5  |.   B8 44AB4900    MOV EAX,MP3_RM_C.0049AB44              ; |invalid register code! please retry!
0049A9BA  |.   E8 9D7CF9FF    CALL MP3_RM_C.0043265C                 ; \MP3_RM_C.0043265C
```

跟踪到 `0049A9A8` 这一行，程序没有跳，接下来弹出注册失败信息框。由上述记录分析：

```
0049A9A1  |>   803D 0C2D4A00> CMP BYTE PTR DS:[4A2D0C],0
```

这一行的判断很重要，`DS:[4A2D0C]` 必须为 `0`，再次注册断下时会发现 `DS:[4A2D0C]` 的值是为 `1` 的，也就是说 `DS:[4A2D0C]` 初始化为 `1`，从记录再往上看：

```
0049A992  |.   C605 0C2D4A00> |MOV BYTE PTR DS:[4A2D0C],0
```

这一行代码正是把 `DS:[4A2D0C]` 置 `0`，因此这一行必须执行。那么：

```
0049A990  |.   75 09          |JNZ SHORT MP3_RM_C.0049A99B
```

一定不能跳。`0049A980` 的上一行是比较输入的用户名与程序“固有用户名”的，二者不相等就跳。这里解释一下程序“固有注册名”，通过分析，我发现这软件的用户名必须与软件本身的一些字符串相等，否则就会跳向失败。用十六进制编辑软件 `WINHEX` 打开 `MP3 RM Converter.exe`，搜索可以找到十几个类似 `"VS88T6-Vs86"` 这样的字符串，程序运行时把他们载入内存。下面代码的作用就是循环读取程序“固有注册名”，比较我们输入的注册名“`PEDIY`”是否与之相等。若循环完没有与之相等的则失败！

```
0049A989  |.   8B16           |MOV EDX,DWORD PTR DS:[ESI]              ;  DS:[004A17F8]=0049A6E4 (MP3_RM_C.0049A6E4), ASCII "VS88T6-Vs86"
0049A98B  |.   E8 F89BF6FF    |CALL MP3_RM_C.00404588                ;  判断用户名与程序“固有注册”名比较
0049A990  |.   75 09          |JNZ SHORT MP3_RM_C.0049A99B
0049A992  |.   C605 0C2D4A00> |MOV BYTE PTR DS:[4A2D0C],0             ;  把 DS:[004A2D0C]置 0
0049A999  |.   EB 06          |JMP SHORT MP3_RM_C.0049A9A1
0049A99B  |>   83C6 04        |ADD ESI,4
0049A99E  |.   4B             |DEC EBX
0049A99F  |.^  75 E5          \JNZ SHORT MP3_RM_C.0049A986
```

重新注册，输入用户名：`VS88T6-Vs86`，注册码：`123456789`，程序断下来，直接 `F4` 运行到：

```
0049A9A8  |.  /74 1A          JE SHORT MP3_RM_C.0049A9C4
```

这回跳了，继续往下分析。

```
0049A9A8  |.  /74 1A          JE SHORT MP3_RM_C.0049A9C4               ;  必须跳，不跳就失败
0049A9AA  |.  |6A 00          PUSH 0                               ; /Arg1 = 00000000
0049A9AC  |.  |66:8B0D 38AB4> MOV CX,WORD PTR DS:[49AB38]             ; |
0049A9B3  |.  |B2 02         MOV DL,2                              ; |
0049A9B5  |.  |B8 44AB4900    MOV EAX,MP3_RM_C.0049AB44              ; |invalid register code! please retry!
0049A9BA  |.  |E8 9D7CF9FF    CALL MP3_RM_C.0043265C                 ; \MP3_RM_C.0043265C
0049A9BF  |.  |E9 2B010000    JMP MP3_RM_C.0049AAEF
0049A9C4  |>  \8D55 EC        LEA EDX,[LOCAL.5]
0049A9C7  |.   8B87 20030000  MOV EAX,DWORD PTR DS:[EDI+320]
0049A9CD  |.   E8 BA31FCFF    CALL MP3_RM_C.0045DB8C
0049A9D2  |.   8B45 EC        MOV EAX,[LOCAL.5]                      ;  注册码放到 EAX
0049A9D5  |.   8D55 F8        LEA EDX,[LOCAL.2]
0049A9D8  |.   E8 C7DCF6FF    CALL MP3_RM_C.004086A4
0049A9DD  |.   8D55 E8        LEA EDX,[LOCAL.6]
0049A9E0  |.   8B45 F8        MOV EAX,[LOCAL.2]
0049A9E3  |.   E8 F0DCF6FF    CALL MP3_RM_C.004086D8
0049A9E8  |.   8B55 E8        MOV EDX,[LOCAL.6]
0049A9EB  |.   8D45 F8        LEA EAX,[LOCAL.2]
0049A9EE  |.   E8 2198F6FF    CALL MP3_RM_C.00404214
0049A9F3  |.   837D FC 00     CMP [LOCAL.1],0
0049A9F7  |.   0F84 F2000000  JE MP3_RM_C.0049AAEF
0049A9FD  |.   837D F8 00     CMP [LOCAL.2],0                        ;  判断注册码是否为空，为空就跳
0049AA01  |.   0F84 E8000000  JE MP3_RM_C.0049AAEF
0049AA07  |.   8B45 F8        MOV EAX,[LOCAL.2]
0049AA0A  |.   E8 2D9AF6FF    CALL MP3_RM_C.0040443C                 ;  计算注册码长度
0049AA0F  |.   85C0           TEST EAX,EAX
0049AA11  |.   7E 35          JLE SHORT MP3_RM_C.0049AA48
```
