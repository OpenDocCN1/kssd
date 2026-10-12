
# MP3 WMA Cutter.2.0 破解算法分析

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-228.htm`](https://www.pediy.com/kssd/pediy09/pediy09-228.htm)

> 【文章标题】: MP3 WMA Cutter.2.0 破解分析  
> 【文章作者】: `xss517`  
> 【作者邮箱】: `xss5172002@yahoo.com.cn`  
> 【作者 QQ 号】: `251496329`  
> 【软件名称】: MP3 WMA Cutter.2.0  
> 【下载地址】: 自己搜索下载  
> 【保护方式】: 无壳  
> 【编写语言】: Borland Delphi 6.0 - 7.0  
> 【作者声明】: 只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教!

--------------------------------------------------------------------------------

【详细过程】

查壳发现是 Borland Delphi 6.0 - 7.0

`DEDE 3.5` 分析，找到过程里面发现有 `reg` 单元模块名

```assembly
004AE234   55                      push    ebp
004AE235   8BEC                    mov     ebp, esp
004AE237   33C9                    xor     ecx, ecx
004AE239   51                      push    ecx
004AE23A   51                      push    ecx
004AE23B   51                      push    ecx
004AE23C   51                      push    ecx
004AE23D   51                      push    ecx
004AE23E   51                      push    ecx
004AE23F   51                      push    ecx
004AE240   53                      push    ebx
004AE241   56                      push    esi
004AE242   57                      push    edi
004AE243   8945FC                  mov     [ebp-$04], eax
004AE246   33C0                    xor     eax, eax
004AE248   55                      push    ebp

* Possible String Reference to: '?W?胨_^[[?
|
004AE249   685DE44A00             push    $004AE45D

***** TRY
|
004AE24E   64FF30                 push    dword ptr fs:[eax]
004AE251   648920                 mov     fs:[eax], esp
004AE254   B301                   mov     bl, $01
004AE256   FF0538EE4C00           inc     dword ptr [$004CEE38]
004AE25C   833D38EE4C0003         cmp     dword ptr [$004CEE38], +$03    连续输入 3 次错误的注册码后会出错
004AE263   7E1D                   jle     004AE282
004AE265   6A00                   push    $00
004AE267   668B0D6CE44A00         mov     cx, word ptr [$004AE46C]
004AE26E   B202                   mov     dl, $02

* Possible String Reference to: 'Invalid register code! Please retry
|                              !'
|
004AE270   B878E44A00             mov     eax, $004AE478

|
004AE275   E8727DF8FF             call    00435FEC
004AE27A   8B45FC                 mov     eax, [ebp-$04]

* Reference to: Forms.TCustomForm.Close(TCustomForm);
|
004AE27D   E8F26EFDFF             call    00485174
004AE282   8D55F0                 lea     edx, [ebp-$10]
004AE285   8B45FC                 mov     eax, [ebp-$04]

* Reference to control Tfm_register.edt_name : TEdit
|
004AE288   8B8010030000           mov     eax, [eax+$0310]

* Reference to: Controls.TControl.GetText(TControl):TCaption;       内部字符串的操作
|
004AE28E   E811A7FBFF             call    004689A4
004AE293   8B45F0                 mov     eax, [ebp-$10]
004AE296   8D55F8                 lea     edx, [ebp-$08]

* Reference to: SysUtils.TrimLeft(AnsiString):AnsiString;overload;
|
004AE299   E856A6F5FF             call    004088F4
004AE29E   8D55EC                 lea     edx, [ebp-$14]
004AE2A1   8B45F8                 mov     eax, [ebp-$08]

* Reference to: SysUtils.TrimRight(AnsiString):AnsiString;overload;
|
004AE2A4   E87FA6F5FF             call    00408928
004AE2A9   8B55EC                 mov     edx, [ebp-$14]
004AE2AC   8D45F8                 lea     eax, [ebp-$08]

* Reference to: System.@LStrLAsg(void;void;void;void);
|
004AE2AF   E82C60F5FF             call    004042E0
004AE2B4   BF15000000             mov     edi, $00000015            内建注册名是十四位，形如 前面七位-后面六位
004AE2B9   BED8CB4C00             mov     esi, $004CCBD8            这个软件是让你注册只能用固定的注册名，
004AE2BE   8B45F8                 mov     eax, [ebp-$08]
004AE2C1   8B16                   mov     edx, [esi]

* Reference to: System.@LStrCmp;
|
004AE2C3   E88C63F5FF             call    00404654
004AE2C8   7504                   jnz     004AE2CE
004AE2CA   33DB                   xor     ebx, ebx
004AE2CC   EB06                   jmp     004AE2D4
004AE2CE   83C604                 add     esi, +$04
004AE2D1   4F                     dec     edi
004AE2D2   75EA                   jnz     004AE2BE
004AE2D4   84DB                   test    bl, bl
004AE2D6   741A                   jz      004AE2F2            这里我 jmp 掉，可以任意注册名注册
004AE2D8   6A00                   push    $00
004AE2DA   668B0D6CE44A00         mov     cx, word ptr [$004AE46C]
004AE2E1   B202                   mov     dl, $02

* Possible String Reference to: 'Invalid register code! Please retry
|                              !'
|
004AE2E3   B878E44A00             mov     eax, $004AE478

|
004AE2E8   E8FF7CF8FF             call    00435FEC
004AE2ED   E930010000             jmp     004AE422
004AE2F2   8D55E8                 lea     edx, [ebp-$18]
004AE2F5   8B45FC                 mov     eax, [ebp-$04]

* Reference to control Tfm_register.edt_code : TEdit
|
004AE2F8   8B8014030000           mov     eax, [eax+$0314]

* Reference to: Controls.TControl.GetText(TControl):TCaption;
|
004AE2FE   E8A1A6FBFF             call    004689A4
004AE303   8B45E8                 mov     eax, [ebp-$18]
004AE306   8D55F4                 lea     edx, [ebp-$0C]

* Reference to: SysUtils.TrimLeft(AnsiString):AnsiString;overload;
|
004AE309   E8E6A5F5FF             call    004088F4
004AE30E   8D55E4                 lea     edx, [ebp-$1C]
004AE311   8B45F4                 mov     eax, [ebp-$0C]

* Reference to: SysUtils.TrimRight(AnsiString):AnsiString;overload;
|
004AE314   E80FA6F5FF             call    00408928
004AE319   8B55E4                 mov     edx, [ebp-$1C]
004AE31C   8D45F4                 lea     eax, [ebp-$0C]

* Reference to: System.@LStrLAsg(void;void;void;void);
|
004AE31F   E8BC5FF5FF             call    004042E0
004AE324   837DF800               cmp     dword ptr [ebp-$08], +$00          检查用户名为空否
004AE328   0F84F4000000           jz      004AE422
004AE32E   837DF400               cmp     dword ptr [ebp-$0C], +$00          检查注册码是否为空
004AE332   0F84EA000000           jz      004AE422
004AE338   8B45F4                 mov     eax, [ebp-$0C]

* Reference to: System.@LStrLen(String):Integer;
|
004AE33B   E8C861F5FF             call    00404508
004AE340   85C0                   test    eax, eax
004AE342   7E35                   jle     004AE379                        小于或等于转移
004AE344   BA01000000             mov     edx, $00000001
004AE349   8B4DF4                 mov     ecx, [ebp-$0C]
004AE34C   0FB64C11FF             movzx   ecx, byte ptr [ecx+edx-$01]
004AE351   83F930                 cmp     ecx, +$30                 检查注册码是否为数字，0-9 之间，不是就完蛋
004AE354   7C05                   jl      004AE35B
004AE356   83F939                 cmp     ecx, +$39
004AE359   7E1A                   jle     004AE375
004AE35B   6A00                   push    $00
```



# 汇编代码分析与注册机算法

## 汇编代码段

> 004AE35D   668B0D6CE44A00         mov     cx, word ptr [$004AE46C]  

> 004AE364   B202                   mov     dl, $02  

>   
> \* Possible String Reference to: 'Invalid register code! Please retry  
> \* !'  

> 004AE366   B878E44A00             mov     eax, $004AE478  

> 004AE36B   E87C7CF8FF             call    00435FEC  

> 004AE370   E9AD000000             jmp     004AE422  

> 004AE375   42                     inc     edx  

> 004AE376   48                     dec     eax  

> 004AE377   75D0                   jnz     004AE349  

> 004AE379   33DB                   xor     ebx, ebx  

> 004AE37B   8B45F8                 mov     eax, [ebp-$08]  

> \* Reference to: System.@LStrLen(String):Integer;  

> 004AE37E   E88561F5FF             call    00404508  

> 004AE383   85C0                   test    eax, eax  

> 004AE385   7E13                   jle     004AE39A  

> 004AE387   BF01000000             mov     edi, $00000001  

> 004AE38C   8B55F8                 mov     edx, [ebp-$08]  # 用户名 asc 值逐个累加，结果放到 ebx 里面  

> 004AE38F   0FB6543AFF             movzx   edx, byte ptr [edx+edi-$01]  

> 004AE394   03DA                   add     ebx, edx  

> 004AE396   47                     inc     edi  

> 004AE397   48                     dec     eax  

> 004AE398   75F2                   jnz     004AE38C  

> 004AE39A   69C326C11B00           imul    eax, ebx, $001BC126  # eax=eax+ebx*1BC126  

> 004AE3A0   0553220000             add     eax, +$00002253  # eax=eax+2253  

> 004AE3A5   D1F8                   sar     eax, 1  # eax=eax/2  

> 004AE3A7   7903                   jns     004AE3AC  # 符号位为 "0" 时转移  

> 004AE3A9   83D000                 adc     eax, +$00  # ADC 带进位加法.  

> 004AE3AC   8BD8                   mov     ebx, eax  # 结果放到 ebx 里面  

> 004AE3AE   8B45F4                 mov     eax, [ebp-$0C]  

> \* Reference to: SysUtils.StrToInt(AnsiString):Integer;  # StrToInt 该函数用于将“字符型”转换成“整数型”。  

> 004AE3B1   E8FAA8F5FF             call    00408CB0  

> 004AE3B6   3BD8                   cmp     ebx, eax  # 比较相同否，决定注册关键  

> 004AE3B8   7553                   jnz     004AE40D  

> 004AE3BA   6A00                   push    $00  

> 004AE3BC   668B0D6CE44A00         mov     cx, word ptr [$004AE46C]  

> 004AE3C3   B202                   mov     dl, $02  

> \* Possible String Reference to: 'Congratuation! You have successfully registered!'  

> 004AE3C5   B8A8E44A00             mov     eax, $004AE4A8  

> 004AE3CA   E81D7CF8FF             call    00435FEC  

> 004AE3CF   A148D24C00             mov     eax, dword ptr [$004CD248]  

> 004AE3D4   C60001                 mov     byte ptr [eax], $01  

> 004AE3D7   A15CD34C00             mov     eax, dword ptr [$004CD35C]  

> 004AE3DC   8B00                   mov     eax, [eax]  

> 004AE3DE   33C9                   xor     ecx, ecx  

> 004AE3E0   BA04000000             mov     edx, $00000004  

> 004AE3E5   8B18                   mov     ebx, [eax]  

> 004AE3E7   FF5314                 call    dword ptr [ebx+$14]  

> 004AE3EA   8B1548D24C00           mov     edx, [$004CD248]  

> 004AE3F0   A15CD34C00             mov     eax, dword ptr [$004CD35C]  

> 004AE3F5   8B00                   mov     eax, [eax]  

> 004AE3F7   B901000000             mov     ecx, $00000001  

> \* Reference to: Classes.TStream.WriteBuffer(TStream;void;void;Longint);  

> 004AE3FC   E86B0AF7FF             call    0041EE6C  

> 004AE401   A134EE4C00             mov     eax, dword ptr [$004CEE34]  

> \* Reference to: Forms.TCustomForm.Close(TCustomForm);  

> 004AE406   E8696DFDFF             call    00485174  

> 004AE40B   EB15                   jmp     004AE422  

> 004AE40D   6A00                   push    $00  

> 004AE40F   668B0D6CE44A00         mov     cx, word ptr [$004AE46C]  

> 004AE416   B202                   mov     dl, $02  

> \* Possible String Reference to: 'Invalid register code! Please retry !'  

> 004AE418   B878E44A00             mov     eax, $004AE478  

> 004AE41D   E8CA7BF8FF             call    00435FEC  

> 004AE422   33C0                   xor     eax, eax  

> 004AE424   5A                     pop     edx  

> 004AE425   59                     pop     ecx  

> 004AE426   59                     pop     ecx  

> 004AE427   648910                 mov     fs:[eax], edx  

> \*\*\*\*\*\* FINALLY \*\*\*\*\*\*  

> 004AE42A   6864E44A00             push    $004AE464  

> 004AE42F   8D45E4                 lea     eax, [ebp-$1C]  

> \* Reference to: System.@LStrClr(void;void);  

> 004AE432   E8115EF5FF             call    00404248  

> 004AE437   8D45E8                 lea     eax, [ebp-$18]  

> \* Reference to: System.@LStrClr(void;void);  

> 004AE43A   E8095EF5FF             call    00404248  

> 004AE43F   8D45EC                 lea     eax, [ebp-$14]  

> \* Reference to: System.@LStrClr(void;void);  

> 004AE442   E8015EF5FF             call    00404248  

> 004AE447   8D45F0                 lea     eax, [ebp-$10]  

> \* Reference to: System.@LStrClr(void;void);  

> 004AE44A   E8F95DF5FF             call    00404248  

> 004AE44F   8D45F4                 lea     eax, [ebp-$0C]  

> 004AE452   BA02000000             mov     edx, $00000002  

> \* Reference to: System.@LStrArrayClr(void;void;Integer);  

> 004AE457   E8105EF5FF             call    0040426C  

> 004AE45C   C3                     ret  

> \* Reference to: System.@HandleFinally;  

> 004AE45D   E92A57F5FF             jmp     00403B8C  

> 004AE462   EBCB                   jmp     004AE42F  

> \*\*\*\*\*\* END \*\*\*\*\*\*  

> 004AE464   5F                     pop     edi  

> 004AE465   5E                     pop     esi  

> 004AE466   5B                     pop     ebx  

> 004AE467   8BE5                   mov     esp, ebp  

> 004AE469   5D                     pop     ebp  

> 004AE46A   C3                     ret  

## 结论与修改建议

先对 `004AE2D6` 的 `jz 004AE2F2` 指令改为 `jmp`，使得任意注册名可以注册。

然后用这个注册机即可。

算法逻辑：
1. 将用户名 ASCII 值逐个累加。
2. 累加结果乘以 `1BC126` (十六进制) / `1818918` (十进制)。
3. 加上 `2253` (十六进制 `2253`)。
4. 除以 2。
5. 将结果转换为字符型，再转换为整数型进行比较。

## VB 注册机代码

```vb
Option Explicit  

Private Sub Command1_Click()  
    Dim id As String  
    Dim i, ebx, eax As Long  
    ebx = 0  
    eax = 0  
    i = 1  
    id = Text1.Text  
    For i = 1 To Len(id)  
        ebx = ebx + Asc(Mid(id, i, 1))  
    Next  
    eax = (8787 + ebx * 1818918) / 2  
    Text2.Text = str(eax)  
End Sub  

Private Sub Command2_Click()  
    End  
End Sub  
```

---
【版权声明】: 本文纯属技术交流, 转载请注明作者并保持文章的完整, 谢谢!  

2007 年 08 月 13 日 PM 06:56:24
