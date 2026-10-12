
# 阿达宠物园 V2.10 先脱后爆

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-636.htm`](https://www.pediy.com/kssd/pediy07/pediy7-636.htm)

**【软件名称】** 阿达宠物园 `V2.10` 先脱后爆  
**【下载地址】** 未知  
**【软件大小】** 未知  
**【破解难度】** `Normal`  
**【保护方式】** `ACprotect`  
**【软件语言】** `Microsoft Visual Basic 5.0 - 6.0`  
**【破解目的】** 初尝脱壳，望赐教

--------------------------------------------------------------------------------

## 过程

最近初尝脱加密保护的壳，顺便下了个阿达连连看 `4.2` 自带的阿达宠物园 `V2.10`。直接运行有提示版本过期，下载新版程序不能运行。提示标题是“`ADA PET`”，确定后程序退出。

用 `Peid` 查壳，为 `ACprotect`。

`OD` 异常设置不忽略内存异常，全自动隐藏 `OD` 插件开启，载入程序。

```
0045D000 a>   60            pushad
0045D001     E8 01000000     call adapet.0045D007
0045D006     E8 83042406     call 0669D48E
0045D00B     C3            retn
```

`F9` 运行到

```
0046EB1F     CD 01          int 1                 中断
0046EB21     40            inc eax
0046EB22     40            inc eax
0046EB23     0BC0         or eax,eax
0046EB25     75 05        jnz short adapet.0046EB2C
0046EB27     90            nop
0046EB28     90            nop
0046EB29     90            nop
0046EB2A     90            nop
0046EB2B     61            popad
0046EB2C     33C0         xor eax,eax
0046EB2E     64:8F00      pop dword ptr fs:[eax]
0046EB31     58            pop eax
0046EB32     60            pushad
0046EB33     E8 00000000   call adapet.0046EB38
0046EB38     5E            pop esi
0046EB39     83EE 06      sub esi,6
0046EB3C     B9 57000000   mov ecx,57
0046EB41     29CE         sub esi,ecx
0046EB43     BA E9A9421B   mov edx,1B42A9E9
0046EB48     C1E9 02      shr ecx,2
0046EB4B     83E9 02      sub ecx,2
0046EB4E     83F9 00      cmp ecx,0
0046EB51     7C 1A        jl short adapet.0046EB6D
0046EB53     8B048E       mov eax,dword ptr ds:[esi+ecx*4]
0046EB56     8B5C8E 04    mov ebx,dword ptr ds:[esi+ecx*4+4]
0046EB5A     03C3         add eax,ebx
0046EB5C     C1C0 14      rol eax,14
0046EB5F     03C2         add eax,edx
0046EB61     81EA 8617B30F sub edx,0FB31786
0046EB67     89048E       mov dword ptr ds:[esi+ecx*4],eax
0046EB6A     49            dec ecx
0046EB6B    ^ EB E1        jmp short adapet.0046EB4E
0046EB6D     61            popad
0046EB6E     61            popad
0046EB6F     C3            retn
```

`Alt+M` 打开内存镜像

内存镜像，项目 `22`

```
 地址=00401000           //下访问断点，然后 shift+F9
 大小=00055000
 Owner=adapet   00400000
 区段=.text
 包含=code
 类型=Imag 01001002
 访问=R
 初始访问=RWE
```

```
004024B0     68 94264000    push adapet.00402694  中断 OEP 处 DUMP 出程序,重建输入表方式 1，程序可正常运行但和加壳的一样,有提示确定后退出。
```

```
004024B5     E8 F0FFFFFF    call adapet.004024AA          ; jmp to MSVBVM60.ThunRTMain
004024BA     0000          add byte ptr ds:[eax],al
004024BC     40            inc eax
004024BD     0000          add byte ptr ds:[eax],al
004024BF     0030          add byte ptr ds:[eax],dh
004024C1     0000          add byte ptr ds:[eax],al
004024C3     0038          add byte ptr ds:[eax],bh
004024C5     0000          add byte ptr ds:[eax],al
004024C7     0000          add byte ptr ds:[eax],al
004024C9     0000          add byte ptr ds:[eax],al
004024CB     005C38 E5     add byte ptr ds:[eax+edi-1B],bl
004024CF     EB 37        jmp short adapet.00402508
```

### 1. 破解：提示 "版本过期,下载新版程序",按"确定"后退出

`OD` 载入脱壳程序，不进行分析。在 `MSVBVM60.DLL` 中下断点 `_RTCMSGBOX` 运行后中断在

```
733AFF3F     55            push ebp   中断后修改为 RETN 见到返回到 UNADAPET.00442580  回到 UNADAPET 找到 00442580 见下面
```

```
733AFF40     8BEC          mov ebp,esp
733AFF42     83EC 30       sub esp,30
733AFF45     53            push ebx
733AFF46     8B5D 08       mov ebx,dword ptr ss:[ebp+8]
733AFF49     56            push esi
733AFF4A     57            push edi
733AFF4B     53            push ebx
733AFF4C     E8 6C5CFFFF   call MSVBVM60.733A5BBD
```

注意下面

```
00442505     83C4 0C        add esp,0C
00442508     66:85F6       test si,si
0044250B     0F84 95000000 je unadapet.004425A6
00442511     B9 0A000000    mov ecx,0A
00442516     B8 04000280    mov eax,80020004
0044251B     898D 5CFFFFFF  mov dword ptr ss:[ebp-A4],ecx
00442521     898D 6CFFFFFF  mov dword ptr ss:[ebp-94],ecx
00442527     898D 7CFFFFFF  mov dword ptr ss:[ebp-84],ecx
0044252D     8D95 4CFFFFFF  lea edx,dword ptr ss:[ebp-B4]
00442533     8D4D 8C        lea ecx,dword ptr ss:[ebp-74]
00442536     8985 64FFFFFF  mov dword ptr ss:[ebp-9C],eax
0044253C     8985 74FFFFFF  mov dword ptr ss:[ebp-8C],eax
00442542     8945 84        mov dword ptr ss:[ebp-7C],eax
00442545     C785 54FFFFFF 94A2>mov dword ptr ss:[ebp-AC],unadapet.0040A294
0044254F     C785 4CFFFFFF 0800>mov dword ptr ss:[ebp-B4],8
00442559     FF15 4C124000  call dword ptr ds:[<&MSVBVM60.__vbaVarDup>]   ; MSVBVM60.__vbaVarDup
0044255F     8D95 5CFFFFFF  lea edx,dword ptr ss:[ebp-A4]
00442565     52            push edx
00442566     8D85 6CFFFFFF  lea eax,dword ptr ss:[ebp-94]
0044256C     50            push eax
0044256D     8D8D 7CFFFFFF  lea ecx,dword ptr ss:[ebp-84]
00442573     51            push ecx
00442574     6A 00         push 0
00442576     8D55 8C      lea edx,dword ptr ss:[ebp-74]
00442579     52            push edx
0044257A     FF15 BC104000 call dword ptr ds:[<&MSVBVM60.rtcMsgBox>]     ; MSVBVM60.rtcMsgBox
00442580     8D85 5CFFFFFF lea eax,dword ptr ss:[ebp-A4]   中断后_RTCMSGBOX 的返回处 在上面 44250B 处有一处可跳过此处修改 JE 为 JNE 保存  CTRL+F2 重新运行修改后的程序无提示信息,直接运行了。但是为未注册版本，进入后正常。
```

### 2. 破解未注册

下断点 `MSVBVM60.__vbaStrMove` 中断在 `00442341` 取消断点 `F8` 见循环就跳过到下面。

注意：

```
00442D42     8BD0          mov edx,eax
00442D44     8D4D B8       lea ecx,dword ptr ss:[ebp-48]
00442D47     FFD6         call esi
00442D49     50            push eax
00442D4A     68 18A34000   push unadapet.0040A318                  ; UNICODE ".dll"
00442D4F     FFD3         call ebx
00442D51     8945 94       mov dword ptr ss:[ebp-6C],eax
00442D54     C745 8C 08000000 mov dword ptr ss:[ebp-74],8
00442D5B     E8 303B0000   call unadapet.00446890     到此 EAX=C:\WINDOWS\System32\1508693042.dll 怀疑在检查注册后的 KEY 文件
00442D60     25 FF000000   and eax,0FF
00442D65     8BF0          mov esi,eax
00442D67     83FE 64       cmp esi,64
00442D6A     72 06         jb short unadapet.00442D72
00442D6C     FF15 24114000 call dword ptr ds:[<&MSVBVM60.__vbaGenerateBo>; MSVBVM60.__vbaGenerateBoundsError
00442D72     8B55 D8       mov edx,dword ptr ss:[ebp-28]
00442D75     8B1D 78104000 mov ebx,dword ptr ds:[<&MSVBVM60.__vbaStrCat>>; MSVBVM60.__vbaStrCat
00442D7B     52            push edx
00442D7C     68 10A34000   push unadapet.0040A310
00442D81     FFD3         call ebx
00442D83     8BD0          mov edx,eax
00442D85     8D4D B0       lea ecx,dword ptr ss:[ebp-50]
00442D88     FF15 80124000 call dword ptr ds:[<&MSVBVM60.__vbaStrMove>]  ; MSVBVM60.__vbaStrMove
00442D8E     50            push eax
00442D8F     A1 50604500   mov eax,dword ptr ds:[456050]
00442D94     8B0CB0        mov ecx,dword ptr ds:[eax+esi*4]
00442D97     51            push ecx
00442D98     FFD3         call ebx                  到此 EAX=C:\WINDOWS\1508693042.dll 怀疑在检查注册后的 KEY 文件
00442D9A     8B35 80124000 mov esi,dword ptr ds:[<&MSVBVM60.__vbaStrMove>; MSVBVM60.__vbaStrMove
```



# 汇编代码分析与补丁

## 原始反汇编代码

```
00442DA0     8BD0               mov edx,eax  

00442DA2     8D4D AC            lea ecx,dword ptr ss:[ebp-54]  

00442DA5     FFD6               call esi  

00442DA7     8B15 04614500       mov edx,dword ptr ds:[456104]  

00442DAD     50                 push eax  

00442DAE     8B82 68010000       mov eax,dword ptr ds:[edx+168]  

00442DB4     50                 push eax  

00442DB5     FFD3               call ebx  

00442DB7     8BD0               mov edx,eax  

00442DB9     8D4D A8            lea ecx,dword ptr ss:[ebp-58]  

00442DBC     FFD6               call esi  

00442DBE     50                 push eax  

00442DBF     68 18A34000        push unadapet.0040A318                 ; UNICODE ".dll"  

00442DC4     FFD3               call ebx  

00442DC6     8B35 E8114000       mov esi,dword ptr ds:[<&MSVBVM60.rtcDir>] ; MSVBVM60.rtcDir  

00442DCC     6A 00              push 0  

00442DCE     8D8D 7CFFFFFF       lea ecx,dword ptr ss:[ebp-84]  

00442DD4     51                 push ecx  

00442DD5     8945 84            mov dword ptr ss:[ebp-7C],eax  

00442DD8     C785 7CFFFFFF 0800>mov dword ptr ss:[ebp-84],8  

00442DE2     FFD6               call esi  

00442DE4     8BD0               mov edx,eax  

00442DE6     8D4D A4            lea ecx,dword ptr ss:[ebp-5C]  

00442DE9     FF15 80124000      call dword ptr ds:[<&MSVBVM60.__vbaStrMove>]  ; MSVBVM60.__vbaStrMove  

00442DEF     8B1D 2C114000       mov ebx,dword ptr ds:[<&MSVBVM60.__vbaStrCmp]] ; MSVBVM60.__vbaStrCmp  

00442DF5     50                 push eax  

00442DF6     68 F8974000        push unadapet.004097F8  

00442DFB     FFD3               call ebx           此处为 CALL MSVBVM60.__vbaStrCmp   CALL 完后 EAX  EDX  ECX 均为 00000000  

00442DFD     8BD0               mov edx,eax  

00442DFF     F7DA               neg edx  

00442E01     1BD2               sbb edx,edx  

00442E03     F7DA               neg edx  

00442E05     6A 00              push 0  

00442E07     8D45 8C            lea eax,dword ptr ss:[ebp-74]  

00442E0A     F7DA               neg edx  

00442E0C     50                 push eax  

00442E0D     8995 84FEFFFF      mov dword ptr ss:[ebp-17C],edx  

00442E13     FFD6               call esi  

00442E15     8BD0               mov edx,eax  

00442E17     8D4D B4            lea ecx,dword ptr ss:[ebp-4C]  

00442E1A     FF15 80124000      call dword ptr ds:[<&MSVBVM60.__vbaStrMove>]  ; MSVBVM60.__vbaStrMove  

00442E20     50                 push eax  

00442E21     68 F8974000        push unadapet.004097F8  

00442E26     FFD3               call ebx           此处为 CALL MSVBVM60.__vbaStrCmp 且 EAX=166BBC                                 EDX=12FD48  ECX=166BBC       CALL 后 EAX  EDX  ECX 均为 0  

00442E28     8B8D 84FEFFFF      mov ecx,dword ptr ss:[ebp-17C]  

00442E2E     F7D8               neg eax  

00442E30     1BC0               sbb eax,eax  

00442E32     F7D8               neg eax  

00442E34     F7D8               neg eax  

00442E36     23C8               and ecx,eax  

00442E38     A1 80604500        mov eax,dword ptr ds:[456080]  

00442E3D     33D2               xor edx,edx  

00442E3F     83F8 0F            cmp eax,0F  

00442E42     0F95C2             setne dl  

00442E45     8D45 A4            lea eax,dword ptr ss:[ebp-5C]  

00442E48     50                 push eax  

00442E49     8D45 B0            lea eax,dword ptr ss:[ebp-50]  

00442E4C     F7DA               neg edx                 可以在此 mov ecx,dword ptr ss:[esp+4]刚好为 442E4C 和 442E4E  

00442E4E     23CA               and ecx,edx  

00442E50     66:890D 7C604500   mov word ptr ds:[45607C],cx              注意 ECX=00000000  破解在此修改 ECX=1 既可  

00442E57     8D4D A8            lea ecx,dword ptr ss:[ebp-58]  

00442E5A     51                 push ecx  

00442E5B     8D55 AC            lea edx,dword ptr ss:[ebp-54]  
```

## 修改方案

修改为：

```
00442E45     8D45 A4             lea eax,dword ptr ss:[ebp-5C]  

00442E48     50                  push eax  

00442E49     8D45 B0             lea eax,dword ptr ss:[ebp-50]  

00442E4C     8B4C24 04           mov ecx,dword ptr ss:[esp+4]       修改处（在此 ECX=1，可利用的字节数有限，查看 ESP 的值在 ESP+4 处为 1）  

00442E50     66:890D 7C604500    mov word ptr ds:[45607C],cx  

00442E57     8D4D A8             lea ecx,dword ptr ss:[ebp-58]  

00442E5A     51                  push ecx  

00442E5B     8D55 AC             lea edx,dword ptr ss:[ebp-54]  
```

## 结果与讨论

保存后运行，破解完成。`阿达宠物园` V4.00 照此也可破解。在 `4ACF50` 处。没有跨平台运行，在其他机子上 `esp+4` 处是否为 `1` 未知。 

今天是中秋佳节,希望老大别扔砖头。 

鼓励一下 

试试 `mov cx,1` 

谢谢版主，`00442E4C F7DA neg edx` 修改为 `mov cx,1` 可以。 

`4dd466d0e1e0cc123afbe504da2462ca`
