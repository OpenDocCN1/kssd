
# 网页特效梦工厂 XP 2.0

> 原文：[`www.pediy.com/kssd/pediy06/pediy6335.htm`](https://www.pediy.com/kssd/pediy06/pediy6335.htm)

**软件名称**：网页特效梦工厂 XP 2.0

**主要功能**：可以自动生成网页特效的软件，每个特效都可以有您进行参数设置，100%傻瓜性。收集了包括时间特效，文字特效，图像处理，鼠标特效，页面特效，菜单特效，在线游戏，其它特效在内的八类上百个精彩特效。

**破解工具**：`UPX`、`DeDe`、`W32Dasm`、`OllyDbg`

**破解过程**：

软件为 `Delphi` 编写，`UPX` 加壳，脱壳后，用 `W32Dasm` 反汇编，载入 `DeDe` 代码。

```text
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004CB646(C)
|
:004CB6C5 33C9                xor ecx, ecx

* Possible StringData Ref from Code Obj ->"SystemCurrentControlSetServicesClassknight"
                                                   ->"softJSBuilder"              注册表键值
                                            |
:004CB6C7 BADCB84C00          mov edx, 004CB8DC
:004CB6CC 8BC6                mov eax, esi

* Reference to: registry.TRegistry.OpenKey(TRegistry;System.AnsiString;System.Boolean):System.Boolean;
|
:004CB6CE E8C512FAFF          call 0046C998

* Possible StringData Ref from Code Obj ->"applycode"
                                      |
:004CB6D3 BA50B94C00          mov edx, 004CB950
:004CB6D8 8BC6                mov eax, esi

* Reference to: registry.TRegistry.ReadInteger(TRegistry;System.AnsiString):System.Integer;
|
:004CB6DA E81D14FAFF          call 0046CAFC                        EAX 十六进制机器码
:004CB6DF E8A0F7FFFF          call 004CAE84         计算 EAX 十六进制真码，算法，进入
:004CB6E4 8BF8                mov edi, eax

* Possible StringData Ref from Code Obj ->"registecode"
                                      |
:004CB6E6 BA64B94C00          mov edx, 004CB964
:004CB6EB 8BC6                mov eax, esi

* Reference to: registry.TRegistry.ReadInteger(TRegistry;System.AnsiString):System.Integer;
|
:004CB6ED E80A14FAFF          call 0046CAFC                        EAX 十六进制假码
:004CB6F2 3BF8                cmp edi, eax         EAX 十六进制假码 EDI 十六进制真码
:004CB6F4 7563                jne 004CB759                        出错跳转
:004CB6F6 33D2                xor edx, edx

* Reference to control Flatbar : TFlatGauge
|
:004CB6F8 8B83DC090000        mov eax, dword ptr [ebx+000009DC]
:004CB6FE E8C5EFFBFF          call 0048A6C8
:004CB703 33D2                xor edx, edx

* Reference to control KsBlendButton3 : TKsBlendButton
|
:004CB705 8B839C120000        mov eax, dword ptr [ebx+0000129C]
:004CB70B 8B08                mov ecx, dword ptr [eax]
:004CB70D FF5164              call [ecx+64]

* Possible StringData Ref from Code Obj ->"网页特效梦工厂 XP 1.5(注册版)"
                                      |
* Possible String Reference to: '网页特效梦工厂 XP 1.5(注册版)'
|
:004CB710 BA78B94C00          mov edx, 004CB978
:004CB715 A1F8504F00          mov eax, dword ptr [004F50F8]

.
.
.
.
.

* Reference to control Label106 : TLabel
|
:004CB749 8B83D8090000        mov eax, dword ptr [ebx+000009D8]
:004CB74F E82C27F7FF          call 0043DE80
:004CB754 E94F010000          jmp 004CB8A8

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004CB6F4(C)
|
* Possible StringData Ref from Code Obj ->"times"
                                      |
:004CB759 BA40B94C00          mov edx, 004CB940
:004CB75E 8BC6                mov eax, esi

* Reference to: registry.TRegistry.ReadInteger(TRegistry;System.AnsiString):System.Integer;
|
:004CB760 E89713FAFF          call 0046CAFC
:004CB765 83F828              cmp eax, 00000028     比较试用次数 h28(最大试用次数 40 次)
:004CB768 0F8EDA000000        jle 004CB848                     次数不到跳转
:004CB76E BA28000000          mov edx, 00000028

.
.
.
.
.
.

* Possible StringData Ref from Code Obj ->"软件试用次数已到,是否马上注册本软件?"
                                      |
* Possible String Reference to: '软件试用次数已到,是否马上注册本软件
|                            ?'
|
:004CB81D B8A4B94C00          mov eax, 004CB9A4
:004CB822 E8FDB8F6FF          call 00437124
:004CB827 83F806              cmp eax, 00000006
:004CB82A 757C                jne 004CB8A8
:004CB82C 6A01                push 00000001
:004CB82E 6A00                push 00000000
:004CB830 6A00                push 00000000

* Possible StringData Ref from Code Obj ->"http://www.softreg.com.cn/shareware_view.asp?i"
                                                   ->"d={F433A6B9-3F03-48A0-9D54-053D6B74E52C}"
                                            |
:004CB832 68CCB94C00          push 004CB9CC
:004CB837 6A00                push 00000000
:004CB839 8BC3                mov eax, ebx

* Reference to: controls.TWinControl.GetHandle(TWinControl):Windows.HWND;
|
:004CB83B E8E88DF7FF          call 00444628
:004CB840 50                  push eax

* Reference To: shell32.ShellExecuteA, Ord:0000h
                              |
:004CB841 E8CA69F6FF          Call 00432210
:004CB846 EB60                jmp 004CB8A8

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004CB768(C)
|
* Possible StringData Ref from Code Obj ->"times"
                                      |
:004CB848 BA40B94C00          mov edx, 004CB940
:004CB84D 8BC6                mov eax, esi

* Reference to: registry.TRegistry.ReadInteger(TRegistry;System.AnsiString):System.Integer;
|
:004CB84F E8A812FAFF          call 0046CAFC
:004CB854 83F828              cmp eax, 00000028    比较试用次数 h28(最大试用次数 40 次)
:004CB857 7F4F                jg 004CB8A8

* Possible StringData Ref from Code Obj ->"times"
                                      |
:004CB859 BA40B94C00          mov edx, 004CB940
:004CB85E 8BC6                mov eax, esi

* Reference to: registry.TRegistry.ReadInteger(TRegistry;System.AnsiString):System.Integer;
|
:004CB860 E89712FAFF          call 0046CAFC
:004CB865 8BD0                mov edx, eax

* Reference to control Flatbar : TFlatGauge
|
:004CB867 8B83DC090000        mov eax, dword ptr [ebx+000009DC]
:004CB86D E856EEFBFF          call 0048A6C8

* Possible StringData Ref from Code Obj ->"times"
                                      |
:004CB872 BA40B94C00          mov edx, 004CB940
:004CB877 8BC6                mov eax, esi

* Reference to: registry.TRegistry.ReadInteger(TRegistry;System.AnsiString):System.Integer;
|
:004CB879 E87E12FAFF          call 0046CAFC
:004CB87E 8BC8                mov ecx, eax
:004CB880 41                  inc ecx                         试用次数＋1

* Possible StringData Ref from Code Obj ->"times"
                                      |
:004CB881 BA40B94C00          mov edx, 004CB940
:004CB886 8BC6                mov eax, esi

* Reference to: registry.TRegistry.WriteInteger(TRegistry;System.AnsiString;System.Integer);
|
:004CB888 E85B12FAFF          call 0046CAE8
:004CB88D E806F5FFFF          call 004CAD98
:004CB892 8D55F0              lea edx, dword ptr [ebp-10]

* Reference to: sysutils.IntToStr(System.Integer):System.AnsiString;overload;
|
:004CB895 E8E6D7F3FF          call 00409080
:004CB89A 8B55F0              mov edx, dword ptr [ebp-10]

* Reference to control FlatEdit46 : TFlatEdit
|
:004CB89D 8B83E0090000        mov eax, dword ptr [ebx+000009E0]

* Reference to: controls.TControl.SetText(TControl;System.String);
|
:004CB8A3 E8E826F7FF          call 0043DF90

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:



|:004CB6C0(U), :004CB754(U), :004CB82A(C), :004CB846(U), :004CB857(C)

|

```
:004CB8A8 33C0                 xor eax, eax
:004CB8AA 5A                  pop edx
:004CB8AB 59                  pop ecx
:004CB8AC 59                  pop ecx
:004CB8AD 648910              mov dword ptr fs:[eax], edx
```

——————————————————————————————————————————————————

注册算法：

```
004CAE84   /$ 55             PUSH EBP
004CAE85   |. 8BEC           MOV EBP,ESP
004CAE87   |. 6A 00          PUSH 0
004CAE89   |. 53             PUSH EBX
004CAE8A   |. 56             PUSH ESI
004CAE8B   |. 8BF0           MOV ESI,EAX
004CAE8D   |. 33C0           XOR EAX,EAX
004CAE8F   |. 55             PUSH EBP
004CAE90   |. 68 52AF4C00    PUSH JSBuilde.004CAF52
004CAE95   |. 64:FF30        PUSH DWORD PTR FS:[EAX]
004CAE98   |. 64:8920        MOV DWORD PTR FS:[EAX],ESP
004CAE9B   |. BB D9D10E00    MOV EBX,0ED1D9                       有用
004CAEA0   |. 8D45 FC        LEA EAX,DWORD PTR SS:[EBP-4]         EAX 等于 12FDE4
004CAEA3   |. BA 68AF4C00    MOV EDX,JSBuilde.004CAF68            ;  ASCII "wangshuang"
EDX 等于 wangshuang
004CAEA8   |. E8 5B9CF3FF     CALL JSBuilde.00404B08
004CAEAD   |. 8B45 FC        MOV EAX,DWORD PTR SS:[EBP-4]         EAX 等于 wangshuang
004CAEB0   |. E8 739EF3FF    CALL JSBuilde.00404D28                获取字串长度入 EAX
004CAEB5   |. 85C0           TEST EAX,EAX                          字串长度是否为零
004CAEB7   |. 7E 1B          JLE SHORT JSBuilde.004CAED4
004CAEB9   |. BA 01000000    MOV EDX,1
004CAEBE   |> 8B4D FC        /MOV ECX,DWORD PTR SS:[EBP-4]        ECX＝字串
004CAEC1   |. 0FB64C11 FF    |MOVZX ECX,BYTE PTR DS:[ECX+EDX-1]
004CAEC6   |. 8D0C89         |LEA ECX,DWORD PTR DS:[ECX+ECX*4]
004CAEC9   |. 8D0C89         |LEA ECX,DWORD PTR DS:[ECX+ECX*4]
004CAECC   |. 03D9           |ADD EBX,ECX                          EBX＋ECX→EBX
004CAECE   |. 03DE           |ADD EBX,ESI                          EBX＋ESI→EBX
004CAED0   |. 42             |INC EDX                              EDX＋1
004CAED1   |. 48             |DEC EAX                              EAX－1
004CAED2   |.⁷⁵ EA          JNZ SHORT JSBuilde.004CAEBE
```

注：获取字串长度后作为循环次数，依次取字串各位的 ASCII 值（十六进制），进行运算。

例：首次运算，`EDX＝1`，`EBX＝0ED1D9`，`ESI＝机器码（十六进制）`，`EAX＝9`，`ECX＝w（ASCII 值 77）`

`77→ECX`，`ECX×19（十进制 25）＝B9F→ECX`，`ECX＋EBX＝EDD78→EBX`，`ESI＋EBX→EBX`，`EDX＋1→EDX`，`EAX－1→EAX`，至此完成 1 次循环；第 2 次 `ECX＝61（a 的 ASCII 值 61）`；依次继续，直至完成 9＋1＝A 次循环。以下计算过程相同。

```
004CAED4   |> 8D45 FC        LEA EAX,DWORD PTR SS:[EBP-4]
004CAED7   |. BA 7CAF4C00    MOV EDX,JSBuilde.004CAF7C            ;  ASCII "yaoyuan"
004CAEDC   |. E8 279CF3FF     CALL JSBuilde.00404B08
004CAEE1   |. 8B45 FC        MOV EAX,DWORD PTR SS:[EBP-4]
004CAEE4   |. E8 3F9EF3FF    CALL JSBuilde.00404D28
004CAEE9   |. 85C0           TEST EAX,EAX
004CAEEB   |. 7E 1B          JLE SHORT JSBuilde.004CAF08
004CAEED   |. BA 01000000    MOV EDX,1
004CAEF2   |> 8B4D FC        /MOV ECX,DWORD PTR SS:[EBP-4]
004CAEF5   |. 0FB64C11 FF    |MOVZX ECX,BYTE PTR DS:[ECX+EDX-1]
004CAEFA   |. 8D0C89         |LEA ECX,DWORD PTR DS:[ECX+ECX*4]
004CAEFD   |. 8D0C89         |LEA ECX,DWORD PTR DS:[ECX+ECX*4]
004CAF00   |. 03D9           |ADD EBX,ECX
004CAF02   |. 03DE           |ADD EBX,ESI
004CAF04   |. 42             |INC EDX
004CAF05   |. 48             |DEC EAX
004CAF06   |.⁷⁵ EA          JNZ SHORT JSBuilde.004CAEF2
004CAF08   |> 8D45 FC        LEA EAX,DWORD PTR SS:[EBP-4]
004CAF0B   |. BA 8CAF4C00    MOV EDX,JSBuilde.004CAF8C            ;  ASCII "JSBuilder"
004CAF10   |. E8 F39BF3FF    CALL JSBuilde.00404B08
004CAF15   |. 8B45 FC        MOV EAX,DWORD PTR SS:[EBP-4]
004CAF18   |. E8 0B9EF3FF    CALL JSBuilde.00404D28
004CAF1D   |. 85C0           TEST EAX,EAX
004CAF1F   |. 7E 1B          JLE SHORT JSBuilde.004CAF3C
004CAF21   |. BA 01000000    MOV EDX,1
004CAF26   |> 8B4D FC        /MOV ECX,DWORD PTR SS:[EBP-4]
004CAF29   |. 0FB64C11 FF    |MOVZX ECX,BYTE PTR DS:[ECX+EDX-1]
004CAF2E   |. 8D0C89         |LEA ECX,DWORD PTR DS:[ECX+ECX*4]
004CAF31   |. 8D0C89         |LEA ECX,DWORD PTR DS:[ECX+ECX*4]
004CAF34   |. 03D9           |ADD EBX,ECX
004CAF36   |. 03DE           |ADD EBX,ESI
004CAF38   |. 42             |INC EDX
004CAF39   |. 48             |DEC EAX
004CAF3A   |.⁷⁵ EA          JNZ SHORT JSBuilde.004CAF26
004CAF3C   |> 33C0          XOR EAX,EAX
004CAF3E   |. 5A             POP EDX
004CAF3F   |. 59             POP ECX
004CAF40   |. 59             POP ECX
004CAF41   |. 64:8910        MOV DWORD PTR FS:[EAX],EDX
004CAF44   |. 68 59AF4C00    PUSH JSBuilde.004CAF59
004CAF49   |> 8D45 FC        LEA EAX,DWORD PTR SS:[EBP-4]
004CAF4C   |. E8 1F9BF3FF    CALL JSBuilde.00404A70
004CAF51   . C3              RETN
```

总结如下：

注册码＝（十六进制）机器码×1A＋A9D×19＋ED1D9
或（十进制）机器码×26＋2717×25＋971225

软件在注册表中建立下列键值：

`[HKEY_LOCAL_MACHINESYSTEMControlSet001ServicesClassknightsoftJSBuilder]`

`"version"="1.1"` 版本
`"times"=dword:00000004` 使用次数
`"applycode"=dword:03b7b2c9` 机器码（十六进制）
`"registecode"=dword:60b80398` 注册码（十六进制）

`4dd466d0e1e0cc123afbe504da2462ca`
