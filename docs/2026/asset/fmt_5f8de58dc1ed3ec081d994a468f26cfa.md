
```text
0012EC68  /0012ECA0

0012EC6C  |5D175334     返回到 5D175334 来自 kernel32.GetModuleHandleA

0012EC70  |5D175380     ASCII "kernel32.dll"

0012ED28  /0012ED44

0012ED2C  |77F45BB0     返回到 SHLWAPI.77F45BB0 来自 kernel32.GetModuleHandleA

0012ED30  |77F44FF4     ASCII "KERNEL32.DLL"

0012F53C  /0012F5A4

0012F540  |005A07ED     返回到 IDTOOLS.005A07ED 来自 kernel32.GetModuleHandleA

0012F544  |00000000

0012BF1C  /0012ED70

0012BF20  |00D2C807     返回到 00D2C807 来自 kernel32.GetModuleHandleA

0012BF24  |00D3D6C8     ASCII "kernel32.dll"

0012BF28  |00D3E67C     ASCII "VirtualAlloc"

0012BF1C  /0012ED70

0012BF20  |00D2C824     返回到 00D2C824 来自 kernel32.GetModuleHandleA

0012BF24  |00D3D6C8     ASCII "kernel32.dll"

0012BF28  |00D3E670     ASCII "VirtualFree"

0012BC94  /0012BF20

0012BC98  |00D1799B     返回到 00D1799B 来自 kernel32.GetModuleHandleA

0012BC9C  |0012BDD4     ASCII "kernel32.dll"      //此时返回
```

```text
00D1799B    8B0D E011D400      mov ecx,dword ptr ds:[D411E0] //返回到这
00D179A1    89040E             mov dword ptr ds:[esi+ecx],eax
00D179A4    A1 E011D400        mov eax,dword ptr ds:[D411E0]
00D179A9    393C06             cmp dword ptr ds:[esi+eax],edi
00D179AC    75 16              jnz short 00D179C4
00D179AE    8D85 B4FEFFFF      lea eax,dword ptr ss:[ebp-14C]
00D179B4    50                 push eax
00D179B5    FF15 CC80D300      call dword ptr ds:[D380CC]        ; kernel32.LoadLibraryA
00D179BB    8B0D E011D400      mov ecx,dword ptr ds:[D411E0]
00D179C1    89040E             mov dword ptr ds:[esi+ecx],eax
00D179C4    A1 E011D400        mov eax,dword ptr ds:[D411E0]
00D179C9    393C06             cmp dword ptr ds:[esi+eax],edi
00D179CC    0F84 AD000000      je 00D17A7F   //Magic Jump，修改成jmp
00D179D2    33C9               xor ecx,ecx
```

修改Magic Jump，再直接按`F9`中断。

用`Imprec1.6f`选择进程`4A4`，填入OEP地址`15AC`，填`RAV=F2120`，`SIZE=1000`。

不要按自动搜索IAT，直接按获取输入表，再按显示无效地址，剪掉修复抓取文件就OK了。

---

## 5、Armadillo 1.xx - 2.xx&单&标准&IAT修复&跨平台问题的简单处理====>目标程序“沙漠杀手-2005-0404私服版本”

---

### 方法：2次断点法，即是：

```text
bp GetModuleHandleA/he GetModuleHandleA
bp GetCurrentThreadId
```

---

OD，载入。。。

```text
00991660 S>/$  55              push ebp  //入口
00991661   |.  8BEC            mov ebp,esp
00991663   |.  6A FF           push -1
00991665   |.  68 90CA9A00     push SFsmss.009ACA90
```

---

下断`he GetModuleHandleA`，`F9`！注意看堆栈！

```text
7C80B529 k>  8BFF              mov edi,edi     ; ntdll.7C930738  //停在这里
7C80B52B     55                push ebp
7C80B52C     8BEC              mov ebp,esp
7C80B52E     837D 08 00        cmp dword ptr ss:[ebp+8],0
```

---

每一次`F9`运行时候，所出现的堆栈情况！

```text
0012FF38    00991728  /CALL 到 GetModuleHandleA 来自 SFsmss.00991722
0012FF3C    00000000  \pModule = NULL
0012FF40    00000000
0012CE70    5D175334  /CALL 到 GetModuleHandleA 来自 5D17532E
0012CE74    5D175380  \pModule = "kernel32.dll"
0012CE78    5D1E3AB8
0012CF30    77F45BB0  /CALL 到 GetModuleHandleA 来自 SHLWAPI.77F45BAA
0012CF34    77F44FF4  \pModule = "KERNEL32.DLL"
0012CF38    00000001
0012D744    0097C5A3  /CALL 到 GetModuleHandleA 来自 SFsmss.0097C59D
0012D748    00000000  \pModule = NULL
0012D74C    0000FFFF
00127CD0    010C0C08  /CALL 到 GetModuleHandleA 来自 010C0C02
00127CD4    010D4D68  \pModule = "kernel32.dll"
00127CD8    010D5F58  ASCII "VirtualAlloc"  //看到这个就慢行了
00127CDC    00000001
00127CD0    010C0C25  /CALL 到 GetModuleHandleA 来自 010C0C1F
00127CD4    010D4D68  \pModule = "kernel32.dll"
00127CD8    010D5F4C  ASCII "VirtualFree"  //看到这个就慢行了
00127CDC    00000001
```

这里用`F9`的话，会提示有异常，`Shift+F9`过！

```text
00127A48    010A9905  /CALL 到 GetModuleHandleA 来自 010A98FF
00127A4C    00127B84  \pModule = "kernel32.dll"
00127A50    00000000
```

此时取消断点，返回！

```text
010A9905     8B0D 40A10D01     mov ecx,dword ptr ds:[10DA140]  //返回到这
010A990B     89040E            mov dword ptr ds:[esi+ecx],eax
010A990E     A1 40A10D01       mov eax,dword ptr ds:[10DA140]
010A9913     393C06            cmp dword ptr ds:[esi+eax],edi
010A9916     75 16             jnz short 010A992E
010A9918     8D85 B4FEFFFF     lea eax,dword ptr ss:[ebp-14C]
010A991E     50                push eax
010A991F     FF15 D4F00C01     call dword ptr ds:[10CF0D4]                   ; kernel32.LoadLibraryA
010A9925     8B0D 40A10D01     mov ecx,dword ptr ds:[10DA140]
010A992B     89040E            mov dword ptr ds:[esi+ecx],eax
010A992E     A1 40A10D01       mov eax,dword ptr ds:[10DA140]
010A9933     393C06            cmp dword ptr ds:[esi+eax],edi
010A9936     0F84 AC000000     je 010A99E8  //Magic Jump！改jmp！跳得很远。。呵呵~
010A993C     33C9              xor ecx,ecx
010A993E     8B03              mov eax,dword ptr ds:[ebx]
010A9940     3938              cmp dword ptr ds:[eax],edi
010A9942     74 06             je short 010A994A
010A9944     41                inc ecx
```

此时第一断点的任务完成了~

---

下断`bp GetCurrentThreadId`，`F9`，注意观察堆栈！

```text
7C809737 k>  64:A1 18000000    mov eax,dword ptr fs:[18]  //停在这里~
7C80973D     8B40 24           mov eax,dword ptr ds:[eax+24]
7C809740     C3                retn
```

---

每一次`F9`运行时候，所出现的堆栈情况！

```text
00127308    73391E36  /CALL 到 GetCurrentThreadId 来自 73391E30
0012730C    00000001
00127310    73391C1A  返回到 73391C1A 来自 73391DE2
001272E8    7339353F  /CALL 到 GetCurrentThreadId 来自 73393539
001272EC    00000000
001272F0    734A0470
0012754C    76DB3705  /CALL 到 GetCurrentThreadId 来自 76DB36FF
00127550    76DBF014
00127554    FFFFFFFF
0012755C    76685212  /CALL 到 GetCurrentThreadId 来自 WININET.7668520C
00127560    00000000
00127564    76680000  WININET.76680000
0012D798    010C9332  /CALL 到 GetCurrentThreadId 来自 010C932C
0012D79C    0012FF2C
0012D7A0    00000000
```

此时取消断点，返回！

```text
010C9332     A3 F0E70D01       mov dword ptr ds:[10DE7F0],eax  //返回到这里~
010C9337     E8 CA08FEFF       call 010A9C06
010C933C     6A 00             push 0
010C933E     E8 E164FEFF       call 010AF824
010C9343     A1 D8E60D01       mov eax,dword ptr ds:[10DE6D8]
010C9348     59                pop ecx
010C9349     8B88 80000000     mov ecx,dword ptr ds:[eax+80]
010C934F     3348 3C           xor ecx,dword ptr ds:[eax+3C]
010C9352     3348 30           xor ecx,dword ptr ds:[eax+30]
010C9355     F6C1 40           test cl,40
010C9358     75 08             jnz short 010C9362
010C935A     6A 01             push 1
```
