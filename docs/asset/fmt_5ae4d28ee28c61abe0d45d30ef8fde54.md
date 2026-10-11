
```
0069093B    81E3 FFFFF0F   and ebx,0FFFFFFF  
00690941    53              push ebx  
00690942    FFB5 17FC4000   push dword ptr ss:[ebp+40FC17]  
00690948    FF95 8CE24100   call dword ptr ss:[ebp+41E28C]  
0069094E    3B9D 1FFC4000   cmp ebx,dword ptr ss:[ebp+40FC1F]  
00690954    7C 0F           jl short 00690965  
00690956    90              nop  
00690957    90              nop  
00690958    90              nop  
00690959    90              nop  
0069095A    60              pushad  
0069095B    2BC0            sub eax,eax  
0069095D    8803            mov byte ptr ds:[ebx],al  

//用完之后清空函数名   NOP 掉 ③！ ★

0069095F    43              inc ebx  
00690960    3803            cmp byte ptr ds:[ebx],al  
00690962    75 F9           jnz short 0069095D  
00690964    61              popad  
00690965    0BC0            or eax,eax  
00690967    0F84 15FFFFFF   je 00690882  
0069096D    3B85 9CE24100   cmp eax,dword ptr ss:[ebp+41E29C]  

//是否是 MessageBoxA ？EMbedded Protector 专用 APT 接口

00690973    74 20           je short 00690995  
00690975    90              nop  
00690976    90              nop  
00690977    90              nop  
00690978    90              nop  
00690979    3B85 9D014100   cmp eax,dword ptr ss:[ebp+41019D]  

//是否是 RegisterHotKey ？

0069097F    74 09           je short 0069098A  
00690981    90              nop  
00690982    90              nop  
00690983    90              nop  
00690984    90              nop  
00690985    EB 14           jmp short 0069099B  
00690987    90              nop  
00690988    90              nop  
00690989    90              nop  
0069098A    8D85 0A024100   lea eax,dword ptr ss:[ebp+41020A]  
00690990    EB 09           jmp short 0069099B  
00690992    90              nop  
00690993    90              nop  
00690994    90              nop  
00690995    8D85 24024100   lea eax,dword ptr ss:[ebp+410224]  
0069099B    56              push esi  
0069099C    FFB5 17FC4000   push dword ptr ss:[ebp+40FC17]  
006909A2    5E              pop esi  
006909A3    39B5 FA234000   cmp dword ptr ss:[ebp+4023FA],esi  

//比较是否是 Kernel32.DLL 基址

006909A9    74 15           je short 006909C0  
006909AB    90              nop  
006909AC    90              nop  
006909AD    90              nop  
006909AE    90              nop  
006909AF    39B5 FE234000   cmp dword ptr ss:[ebp+4023FE],esi  

//比较是否是 User32.DLL 基址

006909B5    74 09           je short 006909C0  
006909B7    90              nop  
006909B8    90              nop  
006909B9    90              nop  
006909BA    90              nop  
006909BB    EB 63           jmp short 00690A20  
006909BD    90              nop  
006909BE    90              nop  
006909BF    90              nop  
006909C0    80BD D2594100 0>cmp byte ptr ss:[ebp+4159D2],0  
006909C7    74 57           je short 00690A20  

//Magic Jump！ 如果用 ImportREC 修复输入表，则可以修改这里为：jmp 00690A20

006909C9    90              nop  
006909CA    90              nop  
006909CB    90              nop  
006909CC    90              nop  
006909CD    EB 07           jmp short 006909D6  

//下面就是加密了

006909CF    90              nop  
006909D0    90              nop  
006909D1    90              nop  
006909D2    0100            add dword ptr ds:[eax],eax  
006909D4    0000            add byte ptr ds:[eax],al  
006909D6    8BB5 E4FC4000   mov esi,dword ptr ss:[ebp+40FCE4]  
006909DC    83C6 0D         add esi,0D  
006909DF    81EE EA1B4000   sub esi,401BEA  
006909E5    2BF5            sub esi,ebp  
006909E7    83FE 00         cmp esi,0  
006909EA    7F 34           jg short 00690A20  
006909EC    90              nop  
006909ED    90              nop  
006909EE    90              nop  
006909EF    90              nop  
006909F0    8BB5 E4FC4000   mov esi,dword ptr ss:[ebp+40FCE4]  
006909F6    53              push ebx  
006909F7    50              push eax  
006909F8    E8 A3B2FFFF     call 0068BCA0  
006909FD    8BD8            mov ebx,eax  
006909FF    58              pop eax  
00690A00    33C3            xor eax,ebx  

//函数加密只是简单异或

00690A02    C606 68         mov byte ptr ds:[esi],68  
00690A05    8946 01         mov dword ptr ds:[esi+1],eax  
00690A08    C746 05 8134240>mov dword ptr ds:[esi+5],243481  
00690A0F    895E 08         mov dword ptr ds:[esi+8],ebx  
00690A12    C646 0C C3      mov byte ptr ds:[esi+C],0C3  
00690A16    5B              pop ebx  
00690A17    8BC6            mov eax,esi  
00690A19    8385 E4FC4000 0>add dword ptr ss:[ebp+40FCE4],0D  
00690A20    5E              pop esi  
00690A21    60              pushad  
00690A22    8BD0            mov edx,eax  
00690A24    2BBD 1FFC4000   sub edi,dword ptr ss:[ebp+40FC1F]  
00690A2A    8BC7            mov eax,edi  
00690A2C    B9 01010000     mov ecx,101  
00690A31    8DBD D3F04000   lea edi,dword ptr ss:[ebp+40F0D3]  
00690A37    F2:AF           repne scas dword ptr es:[edi]  
00690A39    0BC9            or ecx,ecx  
00690A3B    74 13           je short 00690A50  
00690A3D    90              nop  
00690A3E    90              nop  
00690A3F    90              nop  
00690A40    90              nop  
00690A41    81E9 01010000   sub ecx,101  
00690A47    F7D1            not ecx  
00690A49    89948D D3EC4000 mov dword ptr ss:[ebp+ecx*4+40ECD3],edx  

//注意[ebp+ecx*4+40ECD3]从 00689CD3 开始到 0068A0D3 结束  ★  Size=400

00690A49 处就是处理“API 定向到一个地址”的地方！其实这里和 0058117C 至 0058159C 之间的函数是一样的（去除里面的 00000000）。修复代码见最后一节。

00690A50    61              popad  
00690A51    8907            mov dword ptr ds:[edi],eax  

//API 函数的系统地址（或者加密地址）填充到 IAT 中    NOP 掉！④ ★

00690A53    8385 1BFC4000 0>add dword ptr ss:[ebp+40FC1B],4  
00690A5A    E9 6EFEFFFF     jmp 006908CD  
00690A5F    83C6 14         add esi,14  
00690A62    8B95 1FFC4000   mov edx,dword ptr ss:[ebp+40FC1F]  
00690A68    E9 D0FDFFFF     jmp 0069083D  

//循环处理

00690A6D    8DBD D3F04000   lea edi,dword ptr ss:[ebp+40F0D3]  

//修改上面 4 处后直接 F4 到这里   输入表处理完毕了

00690A73    33C0            xor eax,eax  
00690A75    B9 00010000     mov ecx,100  
00690A7A    F3:AB           rep stos dword ptr es:[edi]  
00690A7C    60              pushad  
00690A7D    E8 00000000     call 00690A82
```

运行 `LordPE` 完全 Dump 出这个进程，修正 `dumped.exe` 的 `Import Table RVA=00181000`

### 三、OEP Stolen Code

`Alt+M` 打开内存查看窗口，在 `401000` 第二区段上设置内存访问断点，`Shift+F9` 运行

```
004067F4    53              push ebx  

//中断在这里，取消断点

004067F5    8BD8            mov ebx,eax  
004067F7    33C0            xor eax,eax  
004067F9    A3 9CB05700     mov dword ptr ds:[57B09C],eax  
004067FE    6A 00           push 0  
00406800    E8 25542800     call 0068BC2A  
00406805    A3 68065800     mov dword ptr ds:[580668],eax  
0040680A    A1 68065800     mov eax,dword ptr ds:[580668]  
0040680F    A3 A8B05700     mov dword ptr ds:[57B0A8],eax  
00406814    33C0            xor eax,eax  
00406816    A3 ACB05700     mov dword ptr ds:[57B0AC],eax  
0040681B    33C0            xor eax,eax  
0040681D    A3 B0B05700     mov dword ptr ds:[57B0B0],eax  
00406822    E8 C1FFFFFF     call 004067E8  
00406827    BA A4B05700     mov edx,57B0A4  
0040682C    8BC3            mov eax,ebx  
0040682E    E8 9DD7FFFF     call 00403FD0  
00406833    5B              pop ebx  
00406834    C3              retn
```

很明显，这是 `Delphi` 入口后的第一个 `CALL` 代码，`OEP` 处的代码已经被 `Stolen` 了

此时寄存器和堆栈情况如下：

```
EAX 0057A00C GoodName.0057A00C  
ECX 0013FFB0  
EDX 7C92EB94 ntdll.KiFastSystemCallRet  
EBX 7FFDF000  
ESP 0013FFA4 ASCII "jvi"  
EBP 0013FFC0  
ESI 00000020  
EDI 00791000  
EIP 004067F4 GoodName.004067F4  

0013FFA4    0069766A  返回到 GoodName.0069766A 来自 GoodName.004067F4  
0013FFA8    00000020  
0013FFAC    7FFDF000  
0013FFB0    7C92EB94  ntdll.KiFastSystemCallRet  
0013FFB4    00000020  
0013FFB8    00000020  
0013FFBC    00791000
```
