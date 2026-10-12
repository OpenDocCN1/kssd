
# 批量更名专家 V2.2 Build 0801

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-247.htm`](https://www.pediy.com/kssd/pediy08/pediy8-247.htm)

【文章标题】: 以静制动的算法分析之一

【文章作者】: 请哥慢捂

【作者邮箱】: none

【软件名称】: 批量更名专家 V2.2 Build 0801

【软件大小】: 841 KB

【下载地址】: 自己搜索下载

【加壳方式】: `ASPack` 2.12 -> Alexey Solodovnikov

【保护方式】: 用户名 + 注册码

【编写语言】: `Borland` `Delphi` 4.0 - 5.0

【使用工具】: `Darkde4` + `OD110`

【操作平台】: `Win9x`/`NT`/`2000`/`XP`/`2003`

【软件介绍】: 批量更名专家是一款优秀的批量文件改名工具。。。

【作者声明】: 只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教!

--------------------------------------------------------------------------------

## 详细过程

首先脱壳，`ASPack` 的脱壳机很方便，既然是 `Delphi` 的程序，可以用 `Dede` 来指导调试工作。

反编译出来以后看见有个 `TInputRegForm`，很明显应该是输入注册码的地方，这里有个 `Button1Click`，反汇编之：

```
004A936D   E8C6F0FFFF             call    004A8438
004A9372   3C01                   cmp     al, $01
004A9374   751D                   jnz     004A9393
004A9376   6A00                   push    $00
* Possible String Reference to: '批量更名专家 V2.2'
|
004A9378   B910954A00             mov     ecx, $004A9510
* Possible String Reference to: '批量更名专家 V2.2 您已经是注册用户，
|                                  谢谢您的支持！'
```

根据提示知道前面的函数里面会判断用户是否注册了，返回 `1` 就是注册过了，当然要跟进去看看：

里面是一大段 `Delphi` 的 `TRegistry` 类的函数，由于没有关键的算法，所以简单的说一下处理过程：

读取 `HKEY_LOCAL_MACHINE\Software\zigsoft\renamewiz\` 下面的 `Setup` 值，

如果是 `10` 表示注册了，否则设置成 `5`。

还是回到 `Button1Click` 的处理过程中，往下看：

```
* Reference to control edtUserID : TEdit
004A9396   8B83D8020000           mov     eax, [ebx+$02D8]
* Reference to: controls.TControl.GetText(TControl):TCaption;读取用户名
004A939C   E8BF87F8FF             call    00431B60
004A93A1   8B45F0                 mov     eax, [ebp-$10]
004A93A4   8D55FC                 lea     edx, [ebp-$04]
。。。。。。

* Reference to control edtUserCode : TEdit
004A93AF   8B83DC020000           mov     eax, [ebx+$02DC]
* Reference to: controls.TControl.GetText(TControl):TCaption;读取注册码
004A93B5   E8A687F8FF             call    00431B60
004A93BA   8B45EC                 mov     eax, [ebp-$14]
004A93BD   8D55F8                 lea     edx, [ebp-$08]
* Reference to: sysutils.Trim(AnsiString):AnsiString;
004A93C0   E8EFF8F5FF             call    00408CB4
004A93C5   837DFC00               cmp     dword ptr [ebp-$04], +$00
004A93C9   7406                   jz      004A93D1
004A93CB   837DF800               cmp     dword ptr [ebp-$08], +$00
004A93CF   750F                   jnz     004A93E0
```

如果有一项没有填写就出错

```
* Possible String Reference to: '请输入作者发送给您的注册码认证
004A93D1   B860954A00             mov     eax, $004A9560
```

都填写好了就保存到注册表里

`HKEY_LOCAL_MACHINE\SOFTWARE\zigsoft\renamewiz` `RWUser`

`HKEY_LOCAL_MACHINE\SOFTWARE\zigsoft\renamewiz` `RWCode` 两个键值

```
* Possible String Reference to: '批量更名专家 V2.2'
004A94B8   B910954A00             mov     ecx, $004A9510
* Possible String Reference to: '非常感谢您的注册,请重新运行程序来验证注册码, 谢谢！'
```

然后就是毫不犹豫的重启验证。

对付重启验证，还是先看看 `Dede` 能给出什么提示，多半都会在 `dpr` 初始化或者 `FormCreate` 里面发现，

果然在 `dpr` 代码里面看到了端倪：

```
* Reference to: forms.TApplication.Initialize(TApplication);Delphi 的应用程序初始化
004B4F23   E8C0B1F9FF             call    004500E8
```

一个神秘的判断

```
004B4F28   E87F31FFFF             call    004A80AC
004B4F2D   84C0                   test    al, al
004B4F2F   E8943DFFFF             call    004A8CC8
```

又一个神秘的判断

```
004B4F34   E84736FFFF             call    004A8580
004B4F39   84C0                   test    al, al
004B4F3B   7405                   jz      004B4F42
004B4F3D   E80E3CFFFF             call    004A8B50
```

第三个判断

```
004B4F42   E8F134FFFF             call    004A8438
004B4F47   84C0                   test    al, al
004B4F49   7563                   jnz     004B4FAE
```

首先进去 `004A80AC` 里面，没有涉及到注册信息的那两个键值，还好可以忽略了。

然后进去 `004A8580` 里面，偷笑一下，也没有关键的代码，忽略。

最后来到 `004A8438` 里面，这下子郁闷了，仍然没有计算的代码，不太可能啊，肯定是前面有忽视了什么。

回头来看，第一个和第二个之间的那个函数 `004A8CC8` 进去，居然在这个里面：

```
* Reference to: registry.TRegistry.Create(TRegistry;boolean);overload;
004A8CEB   E8201EFDFF             call    0047AB10
。。。。。。
004A8D01   BA02000080             mov     edx, $80000002        /*HKEY_LOCAL_MACHINE*/
004A8D06   8B45F4                 mov     eax, [ebp-$0C]
* Reference to: registry.TRegistry.SetRootKey(TRegistry;HKEY);
004A8D09   E8A21EFDFF             call    0047ABB0
。。。。。。

* Possible String Reference to: 'RWUser' － 读取用户名
004A8D32   BAB4904A00             mov     edx, $004A90B4
004A8D37   8B45F4                 mov     eax, [ebp-$0C]
* Reference to: registry.TRegistry.ReadString(TRegistry;AnsiString):AnsiString;
004A8D3A   E89D20FDFF             call    0047ADDC
。。。。。。

* Possible String Reference to: 'RWCode' － 读取注册码
004A8D70   BAC4904A00             mov     edx, $004A90C4
004A8D75   8B45F4                 mov     eax, [ebp-$0C]
* Reference to: registry.TRegistry.ReadString(TRegistry;AnsiString):AnsiString;
004A8D78   E85F20FDFF             call    0047ADDC
。。。。。。

004A8DBA   8D55EC                 lea     edx, [ebp-$14]
004A8DBD   8B45FC                 mov     eax, [ebp-$04] { RWUser;  }
004A8DC0   E8DFFBFFFF             call    004A89A4
004A8DC5   8B45EC                 mov     eax, [ebp-$14] { EncUser;  }
004A8DC8   8B55F8                 mov     edx, [ebp-$08] { RWCode;  }
* Reference to: system.@LStrCmp;
004A8DCB   E820B2F5FF             call    00403FF0
```

经过 `004A89A4` 以后会有一个比较，这个函数肯定是不能错过的，启动 `OD` 开始调试了。

用 `OD` 载入原版文件，按键次数＝`F7`×14＋`Enter`×1＋`F4`×1＋`F7`×10，就到入口点了，中断在 `004A89A4`，运行：

```
。。。。。。
```

计算用户名的长度：`qgmw`＝4，记做 `n`

```
004A89FB  |.  8B45 F4               MOV     EAX, DWORD PTR [EBP-C]
004A89FE  |.  E8 DDB4F5FF           CALL    00403EE0
。。。。。。
```

接下来一段循环，`EBX` 为循环变量，记做 `i`，它循环从 `4` 到 `1`：

```
004A8A17 |> /8BC3                  /MOV     EAX, EBX
004A8A19 |. |25 01000080           |AND     EAX, 80000001
004A8A1E |. |79 05                |JNS     SHORT 004A8A25
004A8A20 |. |48                   |DEC     EAX
004A8A21 |. |83C8 FE              |OR      EAX, FFFFFFFE
004A8A24 |. |40                   |INC     EAX
004A8A25 |> |85C0                 |TEST    EAX, EAX
004A8A27 |. |75 2E                |JNZ     SHORT 004A8A57               ；按 i 的奇偶分，奇数跳下面处理
004A8A29 |. |8B45 F4              |MOV     EAX, DWORD PTR [EBP-C]     ；偶数部分
004A8A2C |. |0FB64418 FF          |MOVZX   EAX, BYTE PTR [EAX+EBX-1]   ；RWUser[i]
004A8A31 |. |8BD6                 |MOV     EDX, ESI
004A8A33 |. |2BD3                 |SUB     EDX, EBX                   ；j = n - i
004A8A35 |. |8B4D F4              |MOV     ECX, DWORD PTR [EBP-C]
004A8A38 |. |0FB65411 FF          |MOVZX   EDX, BYTE PTR [ECX+EDX-1]   ；RWUser[j-1]
004A8A3D |. |F7EA                 |IMUL    EDX                        ；EAX * RWUser[j-1]
004A8A3F |. |83E0 03              |AND     EAX, 3                     ；EAX and 3，即(RWUser[i]*RWUser[n-i-1]) and 3
004A8A42 |. |8D55 E8              |LEA     EDX, DWORD PTR [EBP-18]
004A8A45 |. |E8 1E04F6FF          |CALL    00408E68                   ；Delphi：IntToStr
004A8A4A |. |8B55 E8              |MOV     EDX, DWORD PTR [EBP-18]
004A8A4D |. |8B45 EC              |MOV     EAX, DWORD PTR [EBP-14]
004A8A50 |. |8B08                 |MOV     ECX, DWORD PTR [EAX]
004A8A52 |. |FF51 34              |CALL    NEAR DWORD PTR [ECX+34]    ；保存到动态数组
004A8A55 |. |EB 57                |JMP     SHORT 004A8AAE             ；继续循环
004A8A57 |> |8BC3                 |MOV     EAX, EBX                   ；奇数部分
004A8A59 |. |B9 03000000          |MOV     ECX, 3
004A8A5E |. |99                   |CDQ
```



```
  004A8A5F   .|F7F9                 |IDIV    ECX
  004A8A61   .|85D2                 |TEST    EDX, EDX
  004A8A63   .|75 2B                |JNZ     SHORT 004A8A90         ；如果 i mod 3 <> 0 跳到下面处理
  004A8A65   .|8B45 F4              |MOV     EAX, DWORD PTR [EBP-C]     ；i mod 3==0 部分
  004A8A68   .|0FB64418 FF          |MOVZX   EAX, BYTE PTR [EAX+EBX-1]    ；RWUser[i]
  004A8A6D   .|8BD6                 |MOV     EDX, ESI
  004A8A6F   .|2BD3                 |SUB     EDX, EBX
  004A8A71   .|8B4D F4              |MOV     ECX, DWORD PTR [EBP-C]
  004A8A74   .|0FB65411 FF          |MOVZX   EDX, BYTE PTR [ECX+EDX-1]    ；RWUser[n-i-1]
  004A8A79   .|03C2                 |ADD     EAX, EDX                 ；RWUser[i] + RWUser[n-i-1]
  004A8A7B   .|8D55 E4              |LEA     EDX, DWORD PTR [EBP-1C]
  004A8A7E   .|E8 E503F6FF          |CALL    00408E68                 ；Delphi：IntToStr
  004A8A83   .|8B55 E4              |MOV     EDX, DWORD PTR [EBP-1C]
  004A8A86   .|8B45 EC              |MOV     EAX, DWORD PTR [EBP-14]
  004A8A89   .|8B08                 |MOV     ECX, DWORD PTR [EAX]
  004A8A8B   .|FF51 34              |CALL    NEAR DWORD PTR [ECX+34]    ；保存到动态数组
  004A8A8E   .|EB 1E                |JMP     SHORT 004A8AAE         ；继续循环
  004A8A90   >|8B45 F4              |MOV     EAX, DWORD PTR [EBP-C]     ；i mod 3<>0 部分
  004A8A93   .|0FB64418 FF          |MOVZX   EAX, BYTE PTR [EAX+EBX-1]    ；RWUser[i]
  004A8A98   .|83C0 05              |ADD     EAX, 5                 ；RWUser[i] + 5
  004A8A9B   .|8D55 E0              |LEA     EDX, DWORD PTR [EBP-20]
  004A8A9E   .|E8 C503F6FF          |CALL    00408E68                 ；Delphi：IntToStr
  004A8AA3   .|8B55 E0              |MOV     EDX, DWORD PTR [EBP-20]
  004A8AA6   .|8B45 EC              |MOV     EAX, DWORD PTR [EBP-14]
  004A8AA9   .|8B08                 |MOV     ECX, DWORD PTR [EAX]
  004A8AAB   .|FF51 34              |CALL    NEAR DWORD PTR [ECX+34]    ；保存到动态数组
  004A8AAE   >|4B                   |DEC     EBX
  004A8AAF   .|85DB                 |TEST    EBX, EBX
  004A8AB1   .^\\0F8F 60FFFFFF       \\JG      004A8A17
```

循环结束后，根据用户名计算生成了一个动态的字符串数组，下面继续处理

```
  004A8AC9   8D4D DC                /LEA     ECX, DWORD PTR [EBP-24]
  004A8ACC   8BD3                   |MOV     EDX, EBX
  004A8ACE   8B45 EC                |MOV     EAX, DWORD PTR [EBP-14]
  004A8AD1   8B38                   |MOV     EDI, DWORD PTR [EAX]
  004A8AD3   FF57 0C                |CALL    NEAR DWORD PTR [EDI+C]     ；取出动态数组中的一个
  004A8AD6   8B55 DC                |MOV     EDX, DWORD PTR [EBP-24]
  004A8AD9   8D45 F0                |LEA     EAX, DWORD PTR [EBP-10]
  004A8ADC   8B4D F0                |MOV     ECX, DWORD PTR [EBP-10]
  004A8ADF   E8 48B4F5FF            |CALL    00403F2C                 ；system.@LStrCat3 字符串连接
  004A8AE4   43                     |INC     EBX
  004A8AE5   4E                     |DEC     ESI
  004A8AE6  ^ 75 E1                \\JNZ     SHORT 004A8AC9
```

全部连接成一个字符串后返回，然后注册码将和这个字符串进行比较，如果相同的话：

```
  * Possible String Reference to: 'Setup'
  004A8E44   BAD4904A00             mov     edx, $004A90D4
  004A8E49   B90A000000             mov     ecx, $0000000A              ；成功的标记
  004A8E4E   8B45F4                 mov     eax, [ebp-$0C]
  * Reference to: registry.TRegistry.WriteInteger(TRegistry;AnsiString;Integer);
  004A8E51   E8FE1FFDFF             call    0047AE54
```

将`HKEY_LOCAL_MACHINE\\Software\\zigsoft\\renamewiz\\`下面的`Setup`值设置成`10`，前面的分析知道这个就是成功的标志。

但是接下来的代码让人觉得疑惑：

```
  004A8DD6   33C0                   XOR     EAX, EAX
  004A8DD8   8945 F0                MOV     DWORD PTR [EBP-10], EAX
  004A8DDB   817D F0 E8030000       CMP     DWORD PTR [EBP-10], 3E8
  004A8DE2   0F84 AB000000          JE      004A8E93
  ...
  004A8F3E   FF45 F0                INC     DWORD PTR [EBP-10]
  004A8F41   817D F0 E9030000       CMP     DWORD PTR [EBP-10], 3E9
  004A8F48  ^ 0F85 8DFEFFFF         JNZ     004A8DDB
```

这是一个次数为`1000`的循环，其过程只是重复的写入注册成功的标志，有这个必要么？

果然在循环结束的地方，它又将标志设置成了`5`，即注册失败了！意味着前面被骗了，所看到的并非是真正的计算过程。

那么真正的算法会在哪里呢。。。。。。待续

---

【经验总结】

在貌似验证算法的地方投入大量精力，也许不太值得，但是对于程序代码分析的思路是有帮助的，同时对于程序开发人员的揣摩也是有帮助的，所谓知己知彼嘛。

---

【版权声明】: 本文原创于看雪技术论坛, 转载请注明作者并保持文章的完整, 谢谢!

2006 年 07 月 27 日 14:38:10

【文章标题】: 以静制动的算法分析之二

【文章作者】: 请哥慢捂

【作者邮箱】: none

【软件名称】: ????XX(R\*nam\*Wiz) V2.2 Build 0801

【软件大小】: 841 KB

【下载地址】: http://www.skycn.com/soft/7412.html

【加壳方式】: `ASPack 2.12` -> `Alexey Solodovnikov`

【保护方式】: 用户名 + 注册码

【编写语言】: `Borland Delphi 4.0 - 5.0`

【使用工具】: `Darkde4` + `OD110` + `RSATool2v17`

【操作平台】: `Win9x/NT/2000/XP/2003`

【软件介绍】: 批量更名专家是一款优秀的批量文件改名工具。。。

【作者声明】: 只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教!

---

【详细过程】

接上篇，由于看到一处比较的地方便以为找到了关键算法，所以被骗在了假的算法处，那么如果比较不成功可能就不一样了：

```
  004A8DC5   8B45EC                 mov     eax, [ebp-$14] { EncUser;  }
  004A8DC8   8B55F8                 mov     edx, [ebp-$08] { RWCode;  }
  * Reference to: system.@LStrCmp;假的比较处
  004A8DCB   E820B2F5FF             call    00403FF0
  004A8DD0   0F8578010000           jnz     004A8F4E
```

暂且大胆跳下去把：

首先是判断用户名和注册码是否为空

```
  004A8F4E   837DFC00               cmp     dword ptr [ebp-$04], +$00 { RWUser;  }
  004A8F52   0F84F3000000           jz      004A904B
  004A8F58   837DF800               cmp     dword ptr [ebp-$08], +$00 { RWCode;  }
  004A8F5C   0F84E9000000           jz      004A904B
  004A8F62   8B45FC                 mov     eax, [ebp-$04] { RWUser;  }
  * Reference to: system.@LStrLen:Integer;
  004A8F65   E876AFF5FF             call    00403EE0
  004A8F6A   83F807                 cmp     eax, +$07
  004A8F6D   0F8ED8000000           jle     004A904B
  004A8F73   8B45FC                 mov     eax, [ebp-$04] { RWUser;  }
  * Reference to: system.@LStrLen:Integer;
  004A8F76   E865AFF5FF             call    00403EE0
  004A8F7B   83F80A                 cmp     eax, +$0A
  004A8F7E   0F8DC7000000           jnl     004A904B
```

然后检查用户名长度，限制在(7, 10)之间，即[8, 9]（看来我的用户名要重复了，用 qgmwqgmw 把）

```
  004A8F84   8D55E8                 lea     edx, [ebp-$18]
  004A8F87   8B45FC                 mov     eax, [ebp-$04] { RWUser;  }
  004A8F8A   E895F8FFFF             call    004A8824
  004A8F8F   8B45E8                 mov     eax, [ebp-$18]
  004A8F92   8B55F8                 mov     edx, [ebp-$08] { RWCode;  }
  * Reference to: system.@LStrCmp;
  004A8F95   E856B0F5FF             call    00403FF0
```

又一处比较出现了，这次小心点，先看看后面是怎么处理的：

```
  * Possible String Reference to: 'Setup'
  004A8FFC   BAD4904A00             mov     edx, $004A90D4
  004A9001   B90A000000             mov     ecx, $0000000A
  004A9006   8B45F4                 mov     eax, [ebp-$0C]
  * Reference to: registry.TRegistry.WriteInteger(TRegistry;AnsiString;Integer);
  004A9009   E8461EFDFF             call    0047AE54
```

设置成功标志了，并且没有后续的机关，那应该就是这里了，嘻嘻，再 OD 之：

```
  004A8886   8D45 D8                LEA     EAX, DWORD PTR [EBP-28]
  004A8889   8B55 FC                MOV     EDX, DWORD PTR [EBP-4]            ；用户名
  004A888C   E8 67B4F5FF            CALL    00403CF8
  004A8891   8D55 F4                LEA     EDX, DWORD PTR [EBP-C]
```



```
  004A8894    B8 68894A00               MOV     EAX, 004A8968                    ; ASCII "1023654741601"
  004A8899    E8 CAC9FFFF               CALL    004A5268                        ；转换函数 1，跟进
  004A889E    8D45 F4                   LEA     EAX, DWORD PTR [EBP-C]
  004A88A1    E8 3EF4FFFF               CALL    004A7CE4                        ；查找下一个素数，得到"EE5693C6A7"
  004A88A6    8D55 EC                   LEA     EDX, DWORD PTR [EBP-14]
  004A88A9    B8 80894A00               MOV     EAX, 004A8980                   ; ASCII "AEFGhdoiaFA"
  004A88AE    E8 DDDBFFFF               CALL    004A6490                        ；转换函数 2，跟进
  004A88B3    8D45 EC                   LEA     EAX, DWORD PTR [EBP-14]
  004A88B6    E8 29F4FFFF               CALL    004A7CE4                        ；查找下一个素数，得到"4145464768646F696146EB"
  004A88BB    8D4D DC                   LEA     ECX, DWORD PTR [EBP-24]
  004A88BE    8D55 EC                   LEA     EDX, DWORD PTR [EBP-14]         ；q="4145464768646F696146EB"
  004A88C1    8D45 F4                   LEA     EAX, DWORD PTR [EBP-C]          ；p="EE5693C6A7"
  004A88C4    E8 8BD5FFFF               CALL    004A5E54                        ；n=p*q，大数乘法
  004A88C9    8D55 E4                   LEA     EDX, DWORD PTR [EBP-1C]
  004A88CC    B8 94894A00               MOV     EAX, 004A8994                   ; ASCII "834249076537"
  004A88D1    E8 92C9FFFF               CALL    004A5268                        ；调用转换函数 1
  004A88D6    8D45 D8                   LEA     EAX, DWORD PTR [EBP-28]
  004A88D9    50                        PUSH    EAX
  004A88DA    8D4D DC                   LEA     ECX, DWORD PTR [EBP-24]         ；n 的大数组
  004A88DD    8D55 E4                   LEA     EDX, DWORD PTR [EBP-1C]         ；e 的大数组
  004A88E0    8B45 D8                   MOV     EAX, DWORD PTR [EBP-28]         ；用户名
  004A88E3    E8 1CF5FFFF               CALL    004A7E04                        ；转换函数 3，跟进
```

### 跟进转换函数 `004A5268`(`1023654741601`)

```
  。。。。。。
```

一个循环，将待转换的字符串中非数字和负号的字符去掉

```
  004A529F    8D45 FC               /LEA     EAX, DWORD PTR [EBP-4]
  004A52A2    B9 01000000           |MOV     ECX, 1
  004A52A7    BA 01000000           |MOV     EDX, 1
  004A52AC    E8 77EEF5FF           |CALL    00404128
  004A52B1    8B45 FC                MOV     EAX, DWORD PTR [EBP-4]
  004A52B4    8A00                  |MOV     AL, BYTE PTR [EAX]
  004A52B6    2C 2D                 |SUB     AL, 2D
  004A52B8    74 11                 |JE      SHORT 004A52CB                  ；是否是负号
  004A52BA    04 FD                 |ADD     AL, 0FD
  004A52BC    2C 0A                 |SUB     AL, 0A
  004A52BE    72 0B                 |JB      SHORT 004A52CB                  ；是否是 0～9 的数字
  004A52C0    8B45 FC               |MOV     EAX, DWORD PTR [EBP-4]
  004A52C3    E8 18ECF5FF           |CALL    00403EE0
  004A52C8    48                    |DEC     EAX
  004A52C9   ^ 7F D4                \\JG     SHORT 004A529F
```

```
  。。。。。。
```

接下来的循环去掉字符串前面的‘0’

```
  004A530E    8D45 FC               /LEA     EAX, DWORD PTR [EBP-4]
  004A5311    B9 01000000           |MOV     ECX, 1
  004A5316    BA 01000000           |MOV     EDX, 1
  004A531B    E8 08EEF5FF           |CALL    00404128
  004A5320    8B45 FC                MOV     EAX, DWORD PTR [EBP-4]
  004A5323    E8 B8EBF5FF           |CALL    00403EE0
  004A5328    48                    |DEC     EAX
  004A5329    7E 25                 |JLE     SHORT 004A5350
  004A532B    8D45 E0               |LEA     EAX, DWORD PTR [EBP-20]
  004A532E    50                    |PUSH    EAX
  004A532F    B9 01000000           |MOV     ECX, 1
  004A5334    BA 01000000           |MOV     EDX, 1
  004A5339    8B45 FC               |MOV     EAX, DWORD PTR [EBP-4]
  004A533C    E8 A7EDF5FF           |CALL    004040E8
  004A5341    8B45 E0               |MOV     EAX, DWORD PTR [EBP-20]
  004A5344    BA DC544A00           |MOV     EDX, 004A54DC                   ；‘0’
  004A5349    E8 A2ECF5FF           |CALL    00403FF0
  004A534E   ^ 74 BE                \\JE     SHORT 004A530E
```

```
  。。。。。。
```

再来的循环将字符串 9 位一组的转换成数值保存起来，到这里基本上能意识到已经步进 RSA 了

```
  004A53A9    BF 01000000           MOV     EDI, 1
  004A53AE    8D45 DC               /LEA     EAX, DWORD PTR [EBP-24]
  004A53B1    50                    |PUSH    EAX
  004A53B2    8B45 FC               |MOV     EAX, DWORD PTR [EBP-4]
  004A53B5    E8 26EBF5FF           |CALL    00403EE0                      ；system.@LStrLen:Integer
  004A53BA    8BD0                  |MOV     EDX, EAX
  004A53BC    83EA 08               |SUB     EDX, 8
  004A53BF    B9 09000000           |MOV     ECX, 9
  004A53C4    8B45 FC               |MOV     EAX, DWORD PTR [EBP-4]
  004A53C7    E8 1CEDF5FF           |CALL    004040E8                      ；system.@LStrCopy
  004A53CC    8B45 DC               |MOV     EAX, DWORD PTR [EBP-24]
  004A53CF    E8 383BF6FF           |CALL    00408F0C                      ；sysutils.StrToInt
  004A53D4    99                    |CDQ
  004A53D5    52                    |PUSH    EDX
  004A53D6    50                    |PUSH    EAX
  004A53D7    8B43 04               |MOV     EAX, DWORD PTR [EBX+4]
  004A53DA    8F04F8                |POP     DWORD PTR [EAX+EDI*8]
  004A53DD    8F44F8 04             |POP     DWORD PTR [EAX+EDI*8+4]
  004A53E1    8B45 FC               |MOV     EAX, DWORD PTR [EBP-4]
  004A53E4    E8 F7EAF5FF           |CALL    00403EE0
  004A53E9    8BD0                  |MOV     EDX, EAX
  004A53EB    83EA 08               |SUB     EDX, 8
  004A53EE    8D45 FC               |LEA     EAX, DWORD PTR [EBP-4]
  004A53F1    B9 09000000           |MOV     ECX, 9
  004A53F6    E8 2DEDF5FF           |CALL    00404128                      ；system.@LStrDelete
  004A53FB    47                    |INC     EDI
  004A53FC    4E                    |DEC     ESI
  004A53FD   ^ 75 AF                \\JNZ    SHORT 004A53AE
```

将不足 9 位的也转换数值保存起来，其结果就是以 1,000,000,000 进制保存的大数。

```
  004A53FF    8B45 FC               MOV     EAX, DWORD PTR [EBP-4]
  004A5402    E8 053BF6FF           CALL    00408F0C
  004A5407    99                    CDQ
  004A5408    8B4B 04               MOV     ECX, DWORD PTR [EBX+4]
  004A540B    8B75 F8               MOV     ESI, DWORD PTR [EBP-8]
  004A540E    8904F1                MOV     DWORD PTR [ECX+ESI*8], EAX
  004A5411    8954F1 04             MOV     DWORD PTR [ECX+ESI*8+4], EDX
  004A5415    8D45 EC               LEA     EAX, DWORD PTR [EBP-14]
  004A5418    E8 43E8F5FF           CALL    00403C60
```

```
  。。。。。。
```

将转换的大数转换成 2 进制字符串

```
  004A541F    6A 00                 /PUSH    0
  004A5421    6A 02                 |PUSH    2
  004A5423    8D55 F0               |LEA     EDX, DWORD PTR [EBP-10]
  004A5426    8BC3                  |MOV     EAX, EBX
  004A5428    E8 3BFDFFFF           |CALL    004A5168                      ；大数除法，除以 2
  004A542D    FF75 F4               |PUSH    DWORD PTR [EBP-C]
  004A5430    FF75 F0               |PUSH    DWORD PTR [EBP-10]          ；余数
  004A5433    8D45 D8               |LEA     EAX, DWORD PTR [EBP-28]
  004A5436    E8 5D3AF6FF           |CALL    00408E98                      ；sysutils.StrToInt
  004A543B    8B55 D8               |MOV     EDX, DWORD PTR [EBP-28]
  004A543E    8D45 EC               |LEA     EAX, DWORD PTR [EBP-14]
```



```
004A5441    8B4D EC                 |MOV     ECX, DWORD PTR [EBP-14]
004A5444    E8 E3EAF5FF             |CALL    00403F2C                      ；system.@LStrCat3
004A5449    8B43 04                 MOV     EAX, DWORD PTR [EBX+4]
004A544C    8378 04 00              |CMP     DWORD PTR [EAX+4], 0
004A5450    75 CD                   |JNZ     SHORT 004A541F
004A5452    8338 01                 |CMP     DWORD PTR [EAX], 1
004A5455    75 C8                   |JNZ     SHORT 004A541F
004A5457    8B43 04                 |MOV     EAX, DWORD PTR [EBX+4]
004A545A    8378 0C 00              |CMP     DWORD PTR [EAX+C], 0
004A545E    75 BF                   |JNZ     SHORT 004A541F
004A5460    8378 08 00              |CMP     DWORD PTR [EAX+8], 0
004A5464    75 B9                   \\JNZ     SHORT 004A541F
```

。。。。。

然后每 31 位再转换回来，成为 2147483648(2³¹)进制大数

```
004A547F    8B45 EC                 MOV     EAX, DWORD PTR [EBP-14]
004A5482    E8 A90E0000             CALL    004A6330                      ；2 进制到 2147483648 进制
```

－－－－到此转换函数 1 完成了。－－－－－－－－－－－－－－－－－－－－－－－－－－－

继续跟进转换函数 2 `004A6490`('AEFGhdoiaFA')：

。。。。。。

生成 0～FF 的 2 进制 8 位字符串待用

```
004A64E0    BAFF000000              mov     edx, $000000FF
004A64E5    E842E8FFFF              call    004A4D2C
```

。。。。。。

```
004A64F8    BE 01000000             MOV     ESI, 1
004A64FD    8D45 F8                 /LEA     EAX, DWORD PTR [EBP-8]
004A6500    8B55 FC                 |MOV     EDX, DWORD PTR [EBP-4]             ；待转换的字符串
004A6503    0FB65432 FF             |MOVZX   EDX, BYTE PTR [EDX+ESI-1]          ；取第 i 位
004A6508    8B9495 F8FBFFFF         |MOV     EDX, DWORD PTR [EBP+EDX*4-408]      ；2 进制串的表
004A650F    E8 D4D9F5FF             |CALL    00403EE8                           ；system.@LStrCat
004A6514    46                      |INC     ESI
004A6515    4B                      |DEC     EBX
004A6516    75 E5                   \\JNZ     SHORT 004A64FD
```

经过上面循环，"AEFGhdoiaFA"变成了

"0100000101000101010001100100011101101000011001000110111101101001011000010100011001000001"

如果 8 位一组的话就可以还原了。

。。。。。

去掉前面的‘0’以后转换成为 2147483648(2³¹)进制大数

```
004A6545    8B45 F8                 MOV     EAX, DWORD PTR [EBP-8]
004A6548    E8 E3FDFFFF             CALL    004A6330                        ；2 进制字符串到 2147483648 进制
```

－－－－到此转换函数 2 完成了。－－－－－－－－－－－－－－－－－－－－－－－－－－－

看到第 3 个函数可以总结前面都发生了什么：

'1023654741601'：得到大数组 `p`＝EE5693C6A7（素数），

'AEFGhdoiaFA'  ：得到大数组 `q`＝4145464768646F696146EB（素数），

它们的乘积变成了大数组      `n`＝3CC47A496C8DFE566497CF57A640054D，

'834249076537' ：得到大数组 `e`＝C23D1EAF39，

如果对照 RSA 算法的描述可以知道这里已经将必须的几个数据生成了，接下来肯定是用公钥加密了。

继续跟进转换函数 3 `004A7E04`('qgmwqgmw', `n`, `e`)：

。。。。。。

```
004A7E63    8D55 E0                 LEA     EDX, DWORD PTR [EBP-20]
004A7E66    B8 64804A00             MOV     EAX, 004A8064                     ；‘0’
004A7E6B    E8 C0E4FFFF             CALL    004A6330                        ；2 进制字符串到 2147483648 进制
004A7E70    8D55 DC                 LEA     EDX, DWORD PTR [EBP-24]           ；n
004A7E73    8BC7                    MOV     EAX, EDI
004A7E75    E8 D2E3FFFF             CALL    004A624C                        ；2147483648 进制字符串到 2 进制，并且去掉前面的 0
004A7E7A    8B45 DC                 MOV     EAX, DWORD PTR [EBP-24]          ；上面的字符串
004A7E7D    E8 5EC0F5FF             CALL    00403EE0                        ；字符串长度 -> l_n
004A7E82    8BD8                    MOV     EBX, EAX
004A7E84    8D55 DC                 LEA     EDX, DWORD PTR [EBP-24]
004A7E87    8B45 FC                 MOV     EAX, DWORD PTR [EBP-4]           ；用户名
004A7E8A    E8 11D0FFFF             CALL    004A4EA0                        ；和转换函数 2 类似的过程
```

处理后'qgmwqgmw'变成"0111000101100111011011010111011101110001011001110110110101110111" -> `encName`

```
004A7E8F    8D45 DC                 LEA     EAX, DWORD PTR [EBP-24]
004A7E92    8B4D DC                 MOV     ECX, DWORD PTR [EBP-24]
004A7E95    BA 70804A00             MOV     EDX, 004A8070                   ; ASCII "111"
004A7E9A    E8 8DC0F5FF             CALL    00403F2C                        ；字符串前面加上'111'
004A7EA4    8D45 DC                 /LEA     EAX, DWORD PTR [EBP-24]
004A7EA7    8B4D DC                 |MOV     ECX, DWORD PTR [EBP-24]
004A7EAA    BA 64804A00             |MOV     EDX, 004A8064
004A7EAF    E8 78C0F5FF             |CALL    00403F2C
004A7EB4    8B45 DC                 MOV     EAX, DWORD PTR [EBP-24]
004A7EB7    E8 24C0F5FF             |CALL    00403EE0
004A7EBC    99                      |CDQ
004A7EBD    F7FE                    |IDIV    ESI
004A7EBF    85D2                    |TEST    EDX, EDX
004A7EC1    75 E1                   \\JNZ     SHORT 004A7EA4               ；循环增加前导 0，使得 encName 长度是 l_n 的倍数，便于加密时候分组
```

。。。。。。

紧接着开始分组加密了

```
004A7EE5    8D45 D4                 /LEA     EAX, DWORD PTR [EBP-2C]
004A7EE8    50                      |PUSH    EAX
004A7EE9    8BCB                    |MOV     ECX, EBX
004A7EEB    49                      |DEC     ECX
004A7EEC    BA 01000000            |MOV     EDX, 1
004A7EF1    8B45 DC                 |MOV     EAX, DWORD PTR [EBP-24]
004A7EF4    E8 EFC1F5FF             |CALL    004040E8                        ；取出一个分组字符串
004A7EF9    EB 12                   |JMP     SHORT 004A7F0D
004A7EFB    8D45 D4                 |/LEA     EAX, DWORD PTR [EBP-2C]
004A7EFE    B9 01000000             ||MOV     ECX, 1
004A7F03    BA 01000000             ||MOV     EDX, 1
004A7F08    E8 1BC2F5FF             ||CALL    00404128
004A7F0D    8D45 D0                 | LEA     EAX, DWORD PTR [EBP-30]
004A7F10    50                      ||PUSH    EAX
004A7F11    B9 01000000             ||MOV     ECX, 1
004A7F16    BA 01000000             ||MOV     EDX, 1
004A7F1B    8B45 D4                 ||MOV     EAX, DWORD PTR [EBP-2C]
004A7F1E    E8 C5C1F5FF             ||CALL    004040E8
004A7F23    8B45 D0                 ||MOV     EAX, DWORD PTR [EBP-30]
004A7F26    BA 64804A00             ||MOV     EDX, 004A8064                 ；把这个分组字符串的前导‘0’去掉
004A7F2B    E8 C0C0F5FF             ||CALL    00403FF0
004A7F30    75 0B                   ||JNZ     SHORT 004A7F3D
004A7F32    8B45 D4                 ||MOV     EAX, DWORD PTR [EBP-2C]
004A7F35    E8 A6BFF5FF             ||CALL    00403EE0
004A7F3A    48                      ||DEC     EAX
004A7F3B    7F BE                   |\\JG      SHORT 004A7EFB
004A7F3D    8D55 F0                 |LEA     EDX, DWORD PTR [EBP-10]
004A7F40    8B45 D4                 |MOV     EAX, DWORD PTR [EBP-2C]
004A7F43    E8 E8E3FFFF             |CALL    004A6330                        ；2 进制字符串到 2147483648 进制
004A7F48    8BCB                    |MOV     ECX, EBX
004A7F4A    49                      |DEC     ECX
004A7F4B    8D45 DC                 |LEA     EAX, DWORD PTR [EBP-24]
004A7F4E    BA 01000000             |MOV     EDX, 1
004A7F53    E8 D0C1F5FF             |CALL    00404128
004A7F58    8B45 D4                 |MOV     EAX, DWORD PTR [EBP-2C]
004A7F5B    BA 64804A00             |MOV     EDX, 004A8064
004A7F60    E8 8BC0F5FF             |CALL    00403FF0                        ；如果分组就是‘0’计算很简单
004A7F65    75 0D                   |JNZ     SHORT 004A7F74
004A7F67    8D55 E8                 |LEA     EDX, DWORD PTR [EBP-18]
004A7F6A    8D45 E0                 |LEA     EAX, DWORD PTR [EBP-20]
```



```text
  004A7F6D    E8 3ADCFFFF               |CALL    004A5BAC  
  004A7F72    EB 11                    |JMP     SHORT 004A7F85  
  004A7F74    8D45 E8                 |LEA     EAX, DWORD PTR [EBP-18]  
  004A7F77    50                      |PUSH    EAX  
  004A7F78    8BCF                    |MOV     ECX, EDI                      ；n  
  004A7F7A    8B55 F8                 |MOV     EDX, DWORD PTR [EBP-8]        ；e  
  004A7F7D    8D45 F0                 |LEA     EAX, DWORD PTR [EBP-10]       ；分组的大数组 u  
  004A7F80    E8 F7F2FFFF             |CALL    004A727C                      ；计算 u^e mod n -> C  
  004A7F85    8D45 F0                 |LEA     EAX, DWORD PTR [EBP-10]  
  004A7F88    E8 53D5FFFF             |CALL    004A54E0  
  004A7F8D    8D45 D4                 |LEA     EAX, DWORD PTR [EBP-2C]  
  004A7F90    E8 CBBCEF5FF            |CALL    00403C60  
  004A7F95    8D55 D4                 |LEA     EDX, DWORD PTR [EBP-2C]  
  004A7F98    8D45 E8                 |LEA     EAX, DWORD PTR [EBP-18]       ；C  
  004A7F9B    E8 ACE2FFFF             |CALL    004A624C                      ；2147483648 进制字符串到 2 进制，并且去掉前面的 0  
  004A7FA0    EB 10                   |JMP     SHORT 004A7FB2  
  004A7FA2    8D45 D4                 |/LEA     EAX, DWORD PTR [EBP-2C]  
  004A7FA5    8B4D D4                 ||MOV     ECX, DWORD PTR [EBP-2C]  
  004A7FA8    BA 64804A00             ||MOV     EDX, 004A8064  
  004A7FAD    E8 7ABFF5FF             ||CALL    00403F2C  
  004A7FB2    8B45 D4                 | MOV     EAX, DWORD PTR [EBP-2C]  
  004A7FB5    E8 26BFF5FF             ||CALL    00403EE0  
  004A7FBA    99                      ||CDQ  
  004A7FBB    F7FB                    ||IDIV    EBX  
  004A7FBD    85D2                    ||TEST    EDX, EDX  
  004A7FBF  ^ 75 E1                   |\JNZ     SHORT 004A7FA2              ；补充前导 0 到 l_n 位  
  004A7FC1    8D45 D8                 |LEA     EAX, DWORD PTR [EBP-28]  
  004A7FC4    8B55 D4                 |MOV     EDX, DWORD PTR [EBP-2C]  
  004A7FC7    E8 1CBFF5FF             |CALL    00403EE8  
  004A7FCC    8D45 E8                 |LEA     EAX, DWORD PTR [EBP-18]  
  004A7FCF    E8 0CD5FFFF             |CALL    004A54E0  
  004A7FD4    4E                      |DEC     ESI  
  004A7FD5  ^ 0F85 0AFFFFFF           \JNZ     004A7EE5  
  。。。。。。  
  004A8002    8B55 08                 MOV     EDX, DWORD PTR [EBP+8]  
  004A8005    8B45 D8                 MOV     EAX, DWORD PTR [EBP-28]  
  004A8008    E8 3FCFFFFF             CALL    004A4F4C                      ；将大数组转换成 16 进制的字节数组  
  －－－－－－到此转换函数 3 完成了。－－－－－－－－－－－－－－－－－－－－－－－－－－－  
  返回到主要的验证流程中继续：  
  004A88E8    8D55 D4                 LEA     EDX, DWORD PTR [EBP-2C]  
  004A88EB    8B45 D8                 MOV     EAX, DWORD PTR [EBP-28]       ；得到的 16 进制数组  
  004A88EE    E8 55C7FFFF             CALL    004A5048                      ；转换成字符串输出  
```

例如用户名 `qgmwqgmw` 转换成了 `079BF0FDBD5E18042DE49D9684FB227A` 返回。

这次注册码就不会再跑掉了，不用怀疑，就是它了。

对于算法的总结就是彻底的一个 `RSA` 算法，它的参数在上面已经给出，由于不需要解密，所以都不要计算 `D` 了。

## 经验总结

假算法简单，真算法复杂，耐心很重要，体力是基础，我得休息了。。。

## 版权声明

本文原创于看雪技术论坛, 转载请注明作者并保持文章的完整, 谢谢!

2006 年 07 月 27 日 20:22:38
