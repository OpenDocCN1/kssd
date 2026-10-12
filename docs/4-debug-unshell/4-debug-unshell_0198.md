
# Armadillo 1.xx - 2.xx 之脱壳

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-628.htm`](https://www.pediy.com/kssd/pediy07/pediy7-628.htm)

日期：2005 年 5 月 6 日 破解人：`yijun[PYG]`

---

【加壳名称】：`Armadillo 1.xx - 2.xx`

【破解声明】：初学 `Crack`，只是感兴趣，没有其它目的。失误之处敬请诸位大侠赐教！

【破解工具】：`OD`，`PEID`，`LordPE`，`ImportREC1.6`

---

## 破解过程

`peid` 查壳知道，该软件的壳为 `Armadillo 1.xx - 2.xx` -> `SiliconRealms Toolworks`，在参考了许多大侠的破文后总算有所得 `^-^`

`OD` 载入来到这里：

```
00744740 >   55            push ebp
00744741     8BEC          mov ebp,esp
00744743     6A FF         push -1
00744745     68 A0FA7500   push chinast.0075FAA0
0074474A     68 18447400   push chinast.00744418
0074474F     64:A1 00000000 mov eax,dword ptr fs:[0]
00744755     50            push eax
00744756     64:8925 0000000>mov dword ptr fs:[0],esp
0074475D     83EC 58       sub esp,58
00744760     53            push ebx
00744761     56            push esi
00744762     57            push edi
00744763     8965 E8       mov dword ptr ss:[ebp-18],esp
00744766     FF15 78A17500 call dword ptr ds:[<&KERNEL32.GetVersion>; kernel32.GetVersion
0074476C     33D2          xor edx,edx
0074476E     8AD4          mov dl,ah
```

命令行下断点 `BP OpenMutexA,F9` 运行。

```
7C80EC1B >   8BFF          mov edi,edi  //中断在这里
7C80EC1D     55            push ebp
7C80EC1E     8BEC          mov ebp,esp
7C80EC20     51            push ecx
7C80EC21     51            push ecx
7C80EC22     837D 10 00    cmp dword ptr ss:[ebp+10],0
7C80EC26     56            push esi
7C80EC27     0F84 7A500300  je kernel32.7C843CA7
```

堆栈内容：

```
0012D7A0    00730380   /CALL 到 OpenMutexA 来自 chinast.0073037A
0012D7A4    001F0001   |Access = 1F0001
0012D7A8    00000000   |Inheritable = FALSE
0012D7AC    0012DDE0   \MutexName = "BF4::DA2A9E4090"注意 MutexName 这个地址 每个机器不同，以看到的为主。
```

`Ctrl+G` `401000`

```
00401000     0000          ADD BYTE PTR DS:[EAX],AL  //都是空地址。
00401002     0000          ADD BYTE PTR DS:[EAX],AL
00401004     0000          ADD BYTE PTR DS:[EAX],AL
00401006     0000          ADD BYTE PTR DS:[EAX],AL
00401008     0000          ADD BYTE PTR DS:[EAX],AL
0040100A     0000          ADD BYTE PTR DS:[EAX],AL
0040100C     0000          ADD BYTE PTR DS:[EAX],AL
0040100E     0000          ADD BYTE PTR DS:[EAX],AL
00401010     0000          ADD BYTE PTR DS:[EAX],AL
00401012     0000          ADD BYTE PTR DS:[EAX],AL
```

填入以下代码：

```
00401000     60            PUSHAD
00401001     9C            PUSHFD
00401002     68 DCFB1200   PUSH 12DDE0
00401007     33C0          XOR EAX,EAX
00401009     50            PUSH EAX
0040100A     50            PUSH EAX
0040100B     E8 687BA677   CALL KERNEL32.CreateMutexA
00401010     9D            POPFD
00401011     61            POPAD
00401012  - E9 75C7A677    JMP KERNEL32.OpenMutexA
```

点右键 选在此处新建 `Eip`，看到 `Eip` 变为 `401000`

`F9` 运行

```
7C80EC1B >   8BFF          mov edi,edi  //断在这里，清除断点
7C80EC1D     55            push ebp
7C80EC1E     8BEC          mov ebp,esp
7C80EC20     51            push ecx
7C80EC21     51            push ecx
7C80EC22     837D 10 00    cmp dword ptr ss:[ebp+10],0
7C80EC26     56            push esi
7C80EC27     0F84 7A500300  je kernel32.7C843CA7
7C80EC2D     64:A1 18000000 mov eax,dword ptr fs:[18]
7C80EC33     FF75 10        push dword ptr ss:[ebp+10]
7C80EC36     8DB0 F80B0000 lea esi,dword ptr ds:[eax+BF8]
7C80EC3C     8D45 F8       lea eax,dword ptr ss:[ebp-8]
7C80EC3F     50            push eax
7C80EC40     FF15 8C10807C call dword ptr ds:[<&ntdll.RtlInitAnsiSt>; ntdll.RtlInitAnsiString
7C80EC46     6A 00         push 0
```

`Ctrl+G` `401000` 来到：

```
00401000     60            pushad
00401001     9C            pushfd
00401002     68 E0DD1200   push 12DDE0          ; ASCII "CE4::DA2A9E4090"
00401007     33C0          xor eax,eax
00401009     50            push eax
0040100A     50            push eax
0040100B     E8 2FDB407C   call kernel32.CreateMutexA
00401010     9D            popfd
00401011     61            popad
00401012  - E9 04DC407C    jmp kernel32.OpenMutexA
```

撤消刚才的修改 `~~~~~~~~`

下 `bp GetModuleHandleA`，并该为硬件断点：

```
7C80B529 >   8BFF          mov edi,edi //在这里~~~~~
7C80B52B     55            push ebp
7C80B52C     8BEC          mov ebp,esp
7C80B52E     837D 08 00    cmp dword ptr ss:[ebp+8],0
7C80B532     74 18         je short kernel32.7C80B54C
7C80B534     FF75 08       push dword ptr ss:[ebp+8]
7C80B537     E8 682D0000   call kernel32.7C80E2A4
7C80B53C     85C0          test eax,eax
7C80B53E     74 08         je short kernel32.7C80B548
7C80B540     FF70 04       push dword ptr ds:[eax+4]
7C80B543     E8 F4300000   call kernel32.GetModuleHandleW
7C80B548     5D            pop ebp
7C80B549     C2 0400       retn 4
```

`F9` 运行 `N` 次，若遇见非法指令错误就 `Shift+F9` 过 `~~~~~`

在此期间仔细观察堆栈内容，直到看见：

```
00127908    00FA9AF7   /CALL 到 GetModuleHandleA 来自 00FA9AF1
0012790C    00127A4C   \pModule = "advapi32.dll"*****这个东东*****
```

再一次就看见：

```
00127B9C    00FC6AB9   /CALL 到 GetModuleHandleA 来自 00FC6AB3
00127BA0    00000000   \pModule = NULL
```

也就是到达 `magic jmp` 附近了 `~~~~~~~^-^`

清除硬件断点，`Ctrl+F9` 返回 `~~~~~~~~`

```
00FC6AB3    FF15 D400FD00   call dword ptr ds:[FD00D4]           ; kernel32.GetModuleHandleA
00FC6AB9    3985 90C4FFFF   cmp dword ptr ss:[ebp-3B70],eax//返回到这里
00FC6ABF    75 0F           jnz short 00FC6AD0
00FC6AC1    C785 8CC4FFFF 8>mov dword ptr ss:[ebp-3B74],0FD5180
00FC6ACB    E9 C4000000     jmp 00FC6B94
00FC6AD0    83A5 68C2FFFF 0>and dword ptr ss:[ebp-3D98],0
00FC6AD7    C785 64C2FFFF C>mov dword ptr ss:[ebp-3D9C],0FD57C0
00FC6AE1    EB 1C           jmp short 00FC6AFF
00FC6AE3    8B85 64C2FFFF   mov eax,dword ptr ss:[ebp-3D9C]
00FC6AE9    83C0 0C         add eax,0C
00FC6AEC    8985 64C2FFFF   mov dword ptr ss:[ebp-3D9C],eax
00FC6AF2    8B85 68C2FFFF   mov eax,dword ptr ss:[ebp-3D98]
00FC6AF8    40              inc eax
00FC6AF9    8985 68C2FFFF   mov dword ptr ss:[ebp-3D98],eax
00FC6AFF    8B85 64C2FFFF   mov eax,dword ptr ss:[ebp-3D9C]
00FC6B05    8338 00         cmp dword ptr ds:[eax],0
00FC6B08    0F84 86000000   je 00FC6B94    很大一个 magic jmp 跳转，注意，修改它为 jmp 00FC6B94 程序将异常无法继续运行，但 IAT 已经没有加密了。为了跟踪到 oep,动态修改 Z 标志吧~~~~~~~
00FC6B0E    8B85 64C2FFFF   mov eax,dword ptr ss:[ebp-3D9C]
00FC6B14    8B40 08         mov eax,dword ptr ds:[eax+8]
00FC6B17    83E0 01         and eax,1
00FC6B1A    85C0            test eax,eax
00FC6B1C    74 25           je short 00FC6B43
00FC6B1E    A1 2800FE00     mov eax,dword ptr ds:[FE0028]
00FC6B23    8B0D 2800FE00   mov ecx,dword ptr ds:[FE0028]         ; chinast.0075A310
00FC6B29    8B40 20         mov eax,dword ptr ds:[eax+20]
00FC6B2C    3341 40         xor eax,dword ptr ds:[ecx+40]
00FC6B2F    8B0D 2800FE00   mov ecx,dword ptr ds:[FE0028]         ; chinast.0075A310
00FC6B35    3341 28         xor eax,dword ptr ds:[ecx+28]
00FC6B38    25 80000000     and eax,80
00FC6B3D    85C0            test eax,eax
00FC6B3F    74 02           je short 00FC6B43
00FC6B41  ^ EB A0           jmp short 00FC6AE3
```



## 内存断点设置

对 `00A3139E 0F84 86000000 JE 00A3142A` 下内存断点。

### F9 执行后的断点位置

F9 N 次后断在这里（要不断修改标志位哟^-^）：

```
00FA13AC    8B08            mov ecx,dword ptr ds:[eax]  //断在这里,清除内存断点。  
00FA13AE    8365 08 00      and dword ptr ss:[ebp+8],0  
00FA13B2    8D50 04         lea edx,dword ptr ds:[eax+4]  
00FA13B5    894D E8         mov dword ptr ss:[ebp-18],ecx  
00FA13B8    8955 0C         mov dword ptr ss:[ebp+C],edx  
00FA13BB    C745 10 2000000>mov dword ptr ss:[ebp+10],20  
00FA13C2    8B12            mov edx,dword ptr ds:[edx]  
00FA13C4    8955 E4         mov dword ptr ss:[ebp-1C],edx  
00FA13C7    816D 08 4786C86>sub dword ptr ss:[ebp+8],61C88647  
00FA13CE    8BF2            mov esi,edx  
00FA13D0    8BFA            mov edi,edx  
00FA13D2    C1EE 05         shr esi,5  
00FA13D5    0375 FC         add esi,dword ptr ss:[ebp-4]  
00FA13D8    C1E7 04         shl edi,4  
00FA13DB    037D F0         add edi,dword ptr ss:[ebp-10]  
00FA13DE    33F7            xor esi,edi  
00FA13E0    8B7D 08         mov edi,dword ptr ss:[ebp+8]  
00FA13E3    03FA            add edi,edx  
00FA13E5    33F7            xor esi,edi  
00FA13E7    03CE            add ecx,esi  
00FA13E9    8BF1            mov esi,ecx  
00FA13EB    8BF9            mov edi,ecx  
00FA13ED    C1EE 05         shr esi,5  
```

## 内存映射与断点

### 操作命令

`ALT+M`

### 内存映射信息

内存映射，项目 23

- 地址=`00401000` `//下内存访问断点`
- 大小=`00199000` (1675264.)
- 宿主=`chinast` `00400000`
- 区段=`.text`
- 类型=`Imag` `01001002`
- 访问=`R`
- 初始访问=`RWE`

### F9 断点位置

F9 断在这里：

```
00401768   /EB 10           jmp short chinast.0040177A  
0040176A   |66:623A         bound di,dword ptr ds:[edx]  
0040176D   |43              inc ebx  
0040176E   |2B2B            sub ebp,dword ptr ds:[ebx]  
00401770   |48              dec eax  
00401771   |4F              dec edi  
00401772   |4F              dec edi  
00401773   |4B              dec ebx  
00401774   |90              nop  
00401775  -|E9 98A05900     jmp chinast.0099B812  
0040177A   \\A1 8BA05900     mov eax,dword ptr ds:[59A08B]  
0040177F    C1E0 02         shl eax,2  
00401782    A3 8FA05900     mov dword ptr ds:[59A08F],eax  
00401787    52              push edx  
00401788    6A 00           push 0  
0040178A    E8 DD701900     call chinast.0059886C                     ; jmp to kernel32.GetModuleHandleA  
0040178F    8BD0            mov edx,eax  
00401791    E8 A66C1700     call chinast.0057843C  
```

## IAT 修复

`IAT` `Imprec1.6` 修复即可~~~~

`4dd466d0e1e0cc123afbe504da2462ca`
