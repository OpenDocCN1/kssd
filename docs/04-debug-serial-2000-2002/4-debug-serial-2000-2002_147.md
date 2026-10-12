
# 词汇终结者

> 原文：[`www.pediy.com/kssd/pediy02/forum234.htm`](https://www.pediy.com/kssd/pediy02/forum234.htm)

首先将 `WordsendReg.exe` 考到词汇终结者安装目录下，如果不考到词汇终结者安装目录下，反汇编出来的地址是不一样的，我不知是为什么？这个版本的 `W32Dasm` 实在是太 COOL 了！然后用 `W32Dasm` `Ver8.93` 超级中文版反汇编，我用 `English` 版的 `W32Dasm` `Ver8.93` 就不能反汇编 `WordsendReg.exe`，而且 `W32Dasm` `Ver8.93` 超级中文版将程序里面的汉字都能反汇编出来。

当 `W32Dasm` 反汇编完后，点击工具栏上的 `"Strn ReF"`,在数据串项目列表后面可以看见 `"注册码错误!"` 的条目，双击该条目。

*   Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:00401C65(C)

|

```
:00401DBE 8D4C244C          lea ecx, dword ptr [esp+4C]
:00401DC2 E86A940100        call 0041B231
:00401DC7 6A00              push 00000000
:00401DC9 6A00              push 00000000
```

*   Possible StringData Ref from Data Obj -> `"注册码错误！"`

```
:00401DCB 683CC24200        push 0042C23C ---我们在这
:00401DD0 8BCB              mov ecx, ebx
:00401DD2 E8457D0100        call 00419B1C
```

可以看见 `"注册码错误！"` 的提示是从地址：`00401C65` 处过来的。

*   Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:00401BC6(C)

|

```
:00401C27 8B442448          mov eax, dword ptr [esp+48]
:00401C2B 6A00              push 00000000
:00401C2D 0517B00F00        add eax, 000FB017
:00401C32 8D4C2434          lea ecx, dword ptr [esp+34]
:00401C36 50                push eax
:00401C37 E87C950100        call 0041B1B8
:00401C3C 8B6C2468          mov ebp, dword ptr [esp+68]
:00401C40 8B542430          mov edx, dword ptr [esp+30]
:00401C44 689E960100        push 0001969E
:00401C49 55                push ebp
:00401C4A 8D4C2438          lea ecx, dword ptr [esp+38]
:00401C4E FF5234            call [edx+34]
:00401C51 8B442418          mov eax, dword ptr [esp+18]
:00401C55 8B4C2428          mov ecx, dword ptr [esp+28]
:00401C59 50                push eax
:00401C5A 51                push ecx
:00401C5B E872820000        call 00409ED2 ---注册码比较
:00401C60 83C408            add esp, 00000008
:00401C63 85C0              test eax, eax
:00401C65 0F8553010000      jne 00401DBE ---注册码不对就跳到错误显示处,
                            正确就提示注册成功！并展开字库
:00401C6B 8B542460          mov edx, dword ptr [esp+60]
:00401C6F 8B44244C          mov eax, dword ptr [esp+4C]
:00401C73 68615B0300        push 00035B61
:00401C78 52                push edx
:00401C79 8D4C2454          lea ecx, dword ptr [esp+54]
:00401C7D FF5038            call [eax+38]
:00401C80 8B54244C          mov edx, dword ptr [esp+4C]
:00401C84 689E960100        push 0001969E
:00401C89 55                push ebp
:00401C8A 8D4C2454          lea ecx, dword ptr [esp+54]
:00401C8E FF5238            call [edx+38]
:00401C91 8B44244C          mov eax, dword ptr [esp+4C]
:00401C95 6807410A00        push 000A4107
:00401C9A 57                push edi
:00401C9B 8D4C2454          lea ecx, dword ptr [esp+54]
:00401C9F FF5038            call [eax+38]
:00401CA2 8B54244C          mov edx, dword ptr [esp+4C]
:00401CA6 68A4E50600        push 0006E5A4
:00401CAB 56                push esi
:00401CAC 8D4C2454          lea ecx, dword ptr [esp+54]
:00401CB0 FF5238            call [edx+38]
:00401CB3 8B442448          mov eax, dword ptr [esp+48]
:00401CB7 8B4C2470          mov ecx, dword ptr [esp+70]
:00401CBB 8B54244C          mov edx, dword ptr [esp+4C]
:00401CBF 50                push eax
:00401CC0 51                push ecx
:00401CC1 8D4C2454          lea ecx, dword ptr [esp+54]
:00401CC5 FF5238            call [edx+38]
:00401CC8 8D4C244C          lea ecx, dword ptr [esp+4C]
:00401CCC E860950100        call 0041B231
:00401CD1 A178C54200        mov eax, dword ptr [0042C578]
:00401CD6 89442424          mov dword ptr [esp+24], eax
:00401CDA 89442440          mov dword ptr [esp+40], eax
:00401CDE 6A0D              push 0000000D
:00401CE0 6A0A              push 0000000A
:00401CE2 8D442448          lea eax, dword ptr [esp+48]
```

*   Possible StringData Ref from Data Obj -> `"%c%c"`

```
:00401CE6 6800C24200        push 0042C200
:00401CEB 50                push eax
:00401CEC C684245001000012  mov byte ptr [esp+00000150], 12
:00401CF4 E867460100        call 00416360
:00401CF9 83C410            add esp, 00000010
:00401CFC 8D8C24D4000000    lea ecx, dword ptr [esp+000000D4]
:00401D03 51                push ecx
```

*   Possible Reference to Dialog: `DialogID_0064`
*   Possible Reference to Dialog: `DialogID_7801`, `CONTROL_ID:0064`, `""`

```
:00401D04 6A64              push 00000064
```

*   Reference To: `KERNEL32.GetCurrentDirectoryA`, `Ord:00F5h`

```
:00401D06 FF1580324200      Call dword ptr [00423280]
:00401D0C 6A0D              push 0000000D
:00401D0E 8D9424D8000000    lea edx, dword ptr [esp+000000D8]
:00401D15 6A0A              push 0000000A
:00401D17 52                push edx
:00401D18 8D442430          lea eax, dword ptr [esp+30]
```

*   Possible StringData Ref from Data Obj -> `"注册成功！在当前目录(%s)下已生成名为 wordslib.z"`
    -> `"ip 的词库压缩文件，%c%c 详细用法请查看压缩文件里"`
    -> `"的 readme.txt! "`

```
:00401D1C 6894C14200        push 0042C194
:00401D21 50                push eax
:00401D22 E839460100        call 00416360
:00401D27 83C414            add esp, 00000014
:00401D2A 8D4C2440          lea ecx, dword ptr [esp+40]
:00401D2E 8D542424          lea edx, dword ptr [esp+24]
:00401D32 8D442464          lea eax, dword ptr [esp+64]
:00401D36 51                push ecx
:00401D37 52                push edx
:00401D38 50                push eax
:00401D39 E8C89C0100        call 0041BA06
```

*   Possible StringData Ref from Data Obj -> `"请妥善保存注册码和用户名！以备将来更新词库和升"`
    -> `"级文件！"`

```
:00401D3E 685CC14200        push 0042C15C
:00401D43 8D4C246C          lea ecx, dword ptr [esp+6C]
:00401D47 50                push eax
:00401D48 51                push ecx
:00401D49 C684244C01000013  mov byte ptr [esp+0000014C], 13
:00401D51 E8169D0100        call 0041BA6C
:00401D56 50                push eax
:00401D57 8D4C2428          lea ecx, dword ptr [esp+28]
:00401D5B C684244401000014  mov byte ptr [esp+00000144], 14
:00401D63 E8A89B0100        call 0041B910
:00401D68 8D4C2468          lea ecx, dword ptr [esp+68]
:00401D6C C684244001000013  mov byte ptr [esp+00000140], 13
:00401D74 E85E9A0100        call 0041B7D7
:00401D79 8D4C2464          lea ecx, dword ptr [esp+64]
:00401D7D C684244001000012  mov byte ptr [esp+00000140], 12
:00401D85 E84D9A0100        call 0041B7D7
:00401D8A 8B542424          mov edx, dword ptr [esp+24]
:00401D8E 6A30              push 00000030
:00401D90 6A00              push 00000000
:00401D92 52                push edx
:00401D93 8BCB              mov ecx, ebx
```



```
:00401D95 E8827D0100                 call 00419B1C   

:00401D9A 8D4C2440                   lea ecx, dword ptr [esp+40]   

:00401D9E C684244001000011           mov byte ptr [esp+00000140], 11   

:00401DA6 E82C9A0100                 call 0041B7D7   

:00401DAB 8D4C2424                   lea ecx, dword ptr [esp+24]   

:00401DAF C684244001000007           mov byte ptr [esp+00000140], 07   

:00401DB7 E81B9A0100                 call 0041B7D7   

:00401DBC EB23                        jmp 00401DE1   
```

在地址 `00401C5B` 处设断，下 `d eax`, `d ecx`, 你就可以看到注册码了。 `4dd466d0e1e0cc123afbe504da2462ca`
