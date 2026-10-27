
# [原创]以静制动的算法分析之一、二

## 【文章标题】: 以静制动的算法分析之一

【文章作者】: 请哥慢捂

【作者邮箱】: none

【软件名称】: 批量更名XX(R\*nam\*Wiz) V2.2 Build 0801

【软件大小】: 841 KB

【下载地址】: 自己搜索下载

【加壳方式】: ASPack 2.12 -> Alexey Solodovnikov

【保护方式】: 用户名 + 注册码

【编写语言】: Borland Delphi 4.0 - 5.0

【使用工具】: Darkde4 + OD110

【操作平台】: Win9x/NT/2000/XP/2003

【软件介绍】: 批量更名专家是一款优秀的批量文件改名工具。。。

【作者声明】: 只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教!

--------------------------------------------------------------------------------

## 【详细过程】

首先脱壳，ASPack的脱壳机很方便，既然是Delphi的程序，可以用Dede来指导调试工作。

反编译出来以后看见有个`TInputRegForm`，很明显应该是输入注册码的地方，这里有个`Button1Click`，反汇编之：

```
004A936D   E8C6F0FFFF             call    004A8438
004A9372   3C01                   cmp     al, $01
004A9374   751D                   jnz     004A9393
004A9376   6A00                   push    $00
* Possible String Reference to: '批量更名专家 V2.2'
|
004A9378   B910954A00             mov     ecx, $004A9510
* Possible String Reference to: '批量更名专家 V2.2您已经是注册用户，
|                                谢谢您的支持！'
```

根据提示知道前面的函数里面会判断用户是否注册了，返回1就是注册过了，当然要跟进去看看：

里面是一大段Delphi的`TRegistry`类的函数，由于没有关键的算法，所以简单的说一下处理过程：

读取`HKEY_LOCAL_MACHINE\Software\zigsoft\renamewiz\`下面的`Setup`值，

如果是10表示注册了，否则设置成5。

还是回到`Button1Click`的处理过程中，往下看：

```
* Reference to control edtUserID : TEdit
004A9396   8B83D8020000           mov     eax, [ebx+$02D8]
* Reference to: controls.TControl.GetText(TControl):TCaption;读取用户名
004A939C   E8BF87F8FF             call    00431B60
004A93A1   8B45F0                 mov     eax, [ebp-$10]
004A93A4   8D55FC                 lea     edx, [ebp-$04]
...
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

`HKEY_LOCAL_MACHINE\SOFTWARE\zigsoft\renamewiz` `RWCode`两个键值

```
* Possible String Reference to: '批量更名专家 V2.2'
004A94B8   B910954A00             mov     ecx, $004A9510
* Possible String Reference to: '非常感谢您的注册,请重新运行程序来验证注册码, 谢谢！'
```

然后就是毫不犹豫的重启验证。

对付重启验证，还是先看看Dede能给出什么提示，多半都会在dpr初始化或者`FormCreate`里面发现，

果然在dpr代码里面看到了端倪：

```
* Reference to: forms.TApplication.Initialize(TApplication);Delphi的应用程序初始化
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

首先进去`004A80AC`里面，没有涉及到注册信息的那两个键值，还好可以忽略了。

然后进去`004A8580`里面，偷笑一下，也没有关键的代码，忽略。

最后来到`004A8438`里面，这下子郁闷了，仍然没有计算的代码，不太可能啊，肯定是前面有忽视了什么。

回头来看，第一个和第二个之间的那个函数`004A8CC8`进去，居然在这个里面：

```
* Reference to: registry.TRegistry.Create(TRegistry;boolean);overload;
004A8CEB   E8201EFDFF             call    0047AB10
...
004A8D01   BA02000080             mov     edx, $80000002        /*HKEY_LOCAL_MACHINE\*/
004A8D06   8B45F4                 mov     eax, [ebp-$0C]
* Reference to: registry.TRegistry.SetRootKey(TRegistry;HKEY);
004A8D09   E8A21EFDFF             call    0047ABB0
...
* Possible String Reference to: 'RWUser' － 读取用户名
004A8D32   BAB4904A00             mov     edx, $004A90B4
004A8D37   8B45F4                 mov     eax, [ebp-$0C]
* Reference to: registry.TRegistry.ReadString(TRegistry;AnsiString):AnsiString;
004A8D3A   E89D20FDFF             call    0047ADDC
...
* Possible String Reference to: 'RWCode' － 读取注册码
004A8D70   BAC4904A00             mov     edx, $004A90C4
004A8D75   8B45F4                 mov     eax, [ebp-$0C]
* Reference to: registry.TRegistry.ReadString(TRegistry;AnsiString):AnsiString;
004A8D78   E85F20FDFF             call    0047ADDC
...
004A8DBA   8D55EC                 lea     edx, [ebp-$14]
004A8DBD   8B45FC                 mov     eax, [ebp-$04] { RWUser;  }
004A8DC0   E8DFFBFFFF             call    004A89A4
004A8DC5   8B45EC                 mov     eax, [ebp-$14] { EncUser;  }
004A8DC8   8B55F8                 mov     edx, [ebp-$08] { RWCode;  }
* Reference to: system.@LStrCmp;
004A8DCB   E820B2F5FF             call    00403FF0
```

经过`004A89A4`以后会有一个比较，这个函数肯定是不能错过的，启动OD开始调试了。

用OD载入原版文件，按键次数＝F7×14＋Enter×1＋F4×1＋F7×10，就到入口点了，中断在004A89A4，运行：

```
...
计算用户名的长度：‘qgmw’＝4，记做n
004A89FB  |.  8B45 F4                MOV     EAX, DWORD PTR [EBP-C]
004A89FE  |.  E8 DDB4F5FF            CALL    00403EE0
...
接下来一段循环，EBX为循环变量，记做i，它循环从4到1：
004A8A17  |> /8BC3                   /MOV     EAX, EBX
004A8A19  |. |25 01000080            |AND     EAX, 80000001
004A8A1E  |. |79 05                  |JNS     SHORT 004A8A25
004A8A20  |. |48                     |DEC     EAX
004A8A21  |. |83C8 FE                |OR      EAX, FFFFFFFE
004A8A24  |. |40                     |INC     EAX
004A8A25  |> |85C0                   |TEST    EAX, EAX
004A8A27  |. |75 2E                  |JNZ     SHORT 004A8A57              ；按i的奇偶分，奇数跳下面处理
004A8A29  |. |8B45 F4                |MOV     EAX, DWORD PTR [EBP-C]      ；偶数部分
004A8A2C  |. |0FB64418 FF            |MOVZX   EAX, BYTE PTR [EAX+EBX-1]   ；RWUser[i]
004A8A31  |. |8BD6                   |MOV     EDX, ESI
004A8A33  |. |2BD3                   |SUB     EDX, EBX                    ；j = n - i
004A8A35  |. |8B4D F4                |MOV     ECX, DWORD PTR [EBP-C]
004A8A38  |. |0FB65411 FF            |MOVZX   EDX, BYTE PTR [ECX+EDX-1]   ；RWUser[j-1]
004A8A3D  |. |F7EA                   |IMUL    EDX                         ；EAX \* RWUser[j-1]
004A8A3F  |. |83E0 03                |AND     EAX, 3                      ；EAX and 3，即(RWUser[i]\*RWUser[n-i-1]) and 3
004A8A42  |. |8D55 E8                |LEA     EDX, DWORD PTR [EBP-18]
004A8A45  |. |E8 1E04F6FF            |CALL    00408E68                    ；Delphi：IntToStr
004A8A4A  |. |8B55 E8                |MOV     EDX, DWORD PTR [EBP-18]
004A8A4D  |. |8B45 EC                |MOV     EAX, DWORD PTR [EBP-14]
004A8A50  |. |8B08                   |MOV     ECX, DWORD PTR [EAX]
004A8A52  |. |FF51 34                |CALL    NEAR DWORD PTR [ECX+34]     ；保存到动态数组
004A8A55  |. |EB 57                  |JMP     SHORT 004A8AAE              ；继续循环
004A8A57  |> |8BC3                   |MOV     EAX, EBX                    ；奇数部分
004A8A59  |. |B9 03000000            |MOV     ECX, 3
004A8A5E  |. |99                     |CDQ
004A8A5F  |. |F7F9                   |IDIV    ECX
004A8A61  |. |85D2                   |TEST    EDX, EDX
004A8A63  |. |75 2B                  |JNZ     SHORT 004A8A90              ；如果i mod 3 <> 0跳到下面处理
004A8A65  |. |8B45 F4                |MOV     EAX, DWORD PTR [EBP-C]      ；i mod 3==0部分
004A8A68  |. |0FB64418 FF            |MOVZX   EAX, BYTE PTR [EAX+EBX-1]   ；RWUser[i]
004A8A6D  |. |8BD6                   |MOV     EDX, ESI
004A8A6F  |. |2BD3                   |SUB     EDX, EBX
```
