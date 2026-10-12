
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
