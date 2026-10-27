
# 汇编修改记录

## //循环处理输入表

```
00463C55    83C1 04         add ecx,4
00463C58    83C2 04         add edx,4
00463C5B    E9 22FFFFFF     jmp 00463B82
00463C60    83C6 0C         add esi,0C
00463C63    E9 96FCFFFF     jmp 004638FE
```

```
//循环处理输入表
00463C68    33C0            xor eax,eax
00463C6A    40              inc eax
00463C6B    83F8 01         cmp eax,1
00463C6E    74 02           je short 00463C72
00463C70    61              popad
00463C71    C3              retn
```

――――――――――――――――――――――――

```
00463921    56              push esi
00463922    57              push edi
00463923    8BF0            mov esi,eax
00463925    8BF8            mov edi,eax
00463927    AC              lods byte ptr ds:[esi]
00463928    C0C8 04         ror al,4
0046392B    AA              stos byte ptr es:[edi]
0046392C    3E:803F 00      cmp byte ptr ds:[edi],0
00463930    75 F5           jnz short 00463927
```

```
//解码出DLL名、函数名
00463932    5F              pop edi
00463933    5E              pop esi
00463934    C3              retn
```

――――――――――――――――――――――――

```
004641C2    EB 05           jmp short 004641C9
004641C4    3E:C600 00      mov byte ptr ds:[eax],0
```

```
//修改①：  NOP掉 ★  对使用过的DLL名和函数名清0
004641C8    40              inc eax
004641C9    3E:8038 00      cmp byte ptr ds:[eax],0
004641CD    75 F5           jnz short 004641C4
004641CF    C3              retn
```

――――――――――――――――――――――――

修改完以上3处后直接F4至`00463C68`处，运行`LordPE`修正`ImageSize`后完全Dump这个进程

如果要使用`ImportREC`修复输入表，则不需要修改`00463BE0`处，仅修改`00463C40`处就行了

下面来手工修复输入表，看看`004642E2`处保存的表：

```
004642E2   08 9D 05 00 B0 9C 05 00 AC 96 05 00 54 9D 05 00
004642F2   4C 97 05 00 48 91 05 00 F4 9E 05 00 0C 9A 05 00
00464302   08 94 05 00 22 9F 05 00 14 98 05 00 10 92 05 00
00464312   68 A1 05 00 48 98 05 00 44 92 05 00 FA A5 05 00
00464322   F8 9A 05 00 F4 94 05 00 50 A7 05 00 88 97 05 00
00464332   84 91 05 00 82 A7 05 00 7C 9C 05 00 78 96 05 00
00464342   8E A8 05 00 E0 96 05 00 DC 90 05 00 DE A8 05 00
00464352   BC 9A 05 00 B8 94 05 00 00 00 00 00 00 00 00 00
```

每3个`Dword`对应一个`DLL`，`[ESI]`=`NameRVA`、`[ESI+4]`=`FirstThunk`、`[ESI+8]`=`OriginaFirstThunk`

在`dump.eXe`中找块空地，`005AC50`处吧，根据`IID`数组的结构来重组一下`004642E2`表中数据

```
0005AC50   AC 96 05 00 00 00 00 00  00 00 00 00 08 9D 05 00
0005AC60   B0 9C 05 00 48 91 05 00  00 00 00 00 00 00 00 00
0005AC70   54 9D 05 00 4C 97 05 00  08 94 05 00 00 00 00 00
0005AC80   00 00 00 00 F4 9E 05 00  0C 9A 05 00 10 92 05 00
0005AC90   00 00 00 00 00 00 00 00  22 9F 05 00 14 98 05 00
0005ACA0   44 92 05 00 00 00 00 00  00 00 00 00 68 A1 05 00
0005ACB0   48 98 05 00 F4 94 05 00  00 00 00 00 00 00 00 00
0005ACC0   FA A5 05 00 F8 9A 05 00  84 91 05 00 00 00 00 00
0005ACD0   00 00 00 00 50 A7 05 00  88 97 05 00 78 96 05 00
0005ACE0   00 00 00 00 00 00 00 00  82 A7 05 00 7C 9C 05 00
0005ACF0   DC 90 05 00 00 00 00 00  00 00 00 00 8E A8 05 00
0005AD00   E0 96 05 00 B8 94 05 00  00 00 00 00 00 00 00 00
0005AD10   DE A8 05 00 BC 9A 05 00  00 00 00 00 00 00 00 00
```

―――――――――――――――――――――――――――――――――

## 十、时间检验

```
00463C72    F785 A4A34200 0>test dword ptr ss:[ebp+42A3A4],2
00463C7C    74 14           je short 00463C92
00463C7E    8BBD 34A04200   mov edi,dword ptr ss:[ebp+42A034]
00463C84    037F 3C         add edi,dword ptr ds:[edi+3C]
00463C87    8B32            mov esi,dword ptr ds:[edx]
00463C89    8B4F 54         mov ecx,dword ptr ds:[edi+54]
00463C8C    C606 00         mov byte ptr ds:[esi],0
00463C8F    46              inc esi
00463C90    E2 FA           loopd short 00463C8C
00463C92    8D85 629B4200   lea eax,dword ptr ss:[ebp+429B62]
00463C98    50              push eax; 00463D7C
00463C99    C3              retn
00463D7C    BA C3724200     mov edx,4272C3
00463D81    FF5415 00       call dword ptr ss:[ebp+edx]; kernel32.GetTickCount
00463D85    8B8D BCA04200   mov ecx,dword ptr ss:[ebp+42A0BC]
```

//`[ebp+42A0BC]`＝`00C7E130`    还记得`00461EAB`取的时间不？

```
00463D8B    2BC1            sub eax,ecx
```

//`EAX`= `0209F1AD-00C7E130`= `0142107D`

```
00463D8D    3D E02E0000     cmp eax,2EE0
00463D92    78 08           js short 00463D9C
```

//“磨蹭”了这么久肯定超时啦，修改标志位`S=1`使这里跳转

```
00463D9C    8D85 809A4200   lea eax,dword ptr ss:[ebp+429A80]
00463DA2    50              push eax; 00463C9A
00463DA3    C3              retn
```

―――――――――――――――――――――――――――――――――

## 十一、醒来吧：解锁键盘、鼠标和任务栏

```
00463C9A    F785 48A04200 0>test dword ptr ss:[ebp+42A048],8
```

//判断是否是`Win2K/XP`系统平台

```
00463CA4    75 0B           jnz short 00463CB1
00463CA6    6A 00           push 0
00463CA8    BB DF724200     mov ebx,4272DF
00463CAD    FF541D 00       call dword ptr ss:[ebp+ebx]; User32.BlockInput
```

//恢复键盘、鼠标锁定了，输入表处理完了，不能还让别人不动鼠标吧

```
00463CB1    BB 67724200     mov ebx,427267
00463CB6    FF541D 00       call dword ptr ss:[ebp+ebx]
00463CBA    8B9D 60A04200   mov ebx,dword ptr ss:[ebp+42A060]; kernel32.GetCurrentProcess
00463CC0    53              push ebx
00463CC1    50              push eax
00463CC2    BB 93724200     mov ebx,427293
00463CC7    FF541D 00       call dword ptr ss:[ebp+ebx]; kernel32.SetPriorityClass
```

//`NORMAL_PRIORITY_CLASS`     当前进程的优先级别也没必要那么高了

```
00463CCB    F785 A4A34200 8>test dword ptr ss:[ebp+42A3A4],80
00463CD5    75 40           jnz short 00463D17
00463CD7    8B85 40A04200   mov eax,dword ptr ss:[ebp+42A040]
00463CDD    8B9D 3CA04200   mov ebx,dword ptr ss:[ebp+42A03C]
00463CE3    3BC3            cmp eax,ebx
```

//还比较资源浏览器(`Explorer.exe`)的进程ID和父进程的ID

```
00463CE5    74 30           je short 00463D17
00463D17    8B85 54A04200   mov eax,dword ptr ss:[ebp+42A054]
00463D1D    50              push eax
00463D1E    6A F0           push -10
00463D20    8B85 50A04200   mov eax,dword ptr ss:[ebp+42A050]
00463D26    50              push eax
00463D27    BB E7724200     mov ebx,4272E7
00463D2C    FF541D 00       call dword ptr ss:[ebp+ebx]; User32.SetWindowLongA
```

//任务栏也该有反应了，其实我的任务栏一直是正常的
//`0012EAC8`   `00463D30`  /CALL 到 `SetWindowLongA` 来自 yP.`00463D2C`
//`0012EACC`   `000200E4`  |hWnd = `000200E4` (class=`'Shell_TrayWnd'`)
//`0012EAD0`   `FFFFFFF0`  |Index = `GWL_STYLE`
//`0012EAD4`   `96000000`  \NewValue = `96000000`

```
00463D30    8B85 5CA04200   mov eax,dword ptr ss:[ebp+42A05C]
00463D36    50              push eax
00463D37    6A F0           push -10
00463D39    8B85 58A04200   mov eax,dword ptr ss:[ebp+42A058]
00463D3F    50              push eax
00463D40    BB E7724200     mov ebx,4272E7
00463D45    FF541D 00       call dword ptr ss:[ebp+ebx]; User32.SetWindowLongA
```

―――――――――――――――――――――――――――――――――

## 十二、再一再而不可再三再四：再次的检验

现在取消所有断点、恢复以前修改的所有代码，下面还有检验和解码。

```
00463D49    8D85 2F734200   lea eax,dword ptr ss:[ebp+42732F]
```

//`[ebp+42732F]`=`00461549`     `ModuleEntryPoint`

```
00463D4F    B9 E39B4200     mov ecx,429BE3
00463D54    81E9 2F734200   sub ecx,42732F
```

//`ECX`= `429BE3-42732F`= `000028B4`

```
00463D5A    EB 01           jmp short 00463D5D
00463D5D    51              push ecx
00463D5E    50              push eax
00463D5F    E8 6AE8FFFF     call 004625CE
```

//计算壳`EP`处长度`28B4`代码段的校验值＝`000FC7C5`

```
00463D64    83C4 08         add esp,8
00463D67    EB 01           jmp short 00463D6A
00463D6A    8B9D A8A34200   mov ebx,dword ptr ss:[ebp+42A3A8]
```

//`[ebp+42A3A8]`=`[004645C2]`=`000FC7C5`  还记得`0046178A`取得的检验值不？

```
00463D70    33C3            xor eax,ebx
```

//检验   恢复了修改的代码

```
00463D72    74 30           je short 00463DA4
```

//不等不跳就`Over`了。所以如果检验错误的话，则要修改标志位`Z=1`使这里跳转

―――――――――――――――――――――――――――――――――

## 十三、解出463DFD处下一步运行的代码

```
00463DA4    8DBD E39B4200   lea edi,dword ptr ss:[ebp+429BE3]
```

//`[ebp+429BE3]`=`00463DFD`

```
00463DAA    8BF7            mov esi,edi
00463DAC    B9 A39F4200     mov ecx,429FA3
00463DB1    81E9 E39B4200   sub ecx,429BE3
```

//`ECX`= `429FA3-429BE3`= `000003C0`

```
00463DB7    EB 04           jmp short 00463DBD
00463DBD    AC              lods byte ptr ds:[esi]
00463DBE    04 3C           add al,3C
00463DC0    EB 01           jmp short 00463DC3
00463DC3    2C 73           sub al,73
00463DC5    FEC8            dec al
00463DC7    EB 01           jmp short 00463DCA
00463DCA    EB 01           jmp short 00463DCD
00463DCD    90              nop
00463DCE    EB 01           jmp short 00463DD1
00463DD1    EB 01           jmp short 00463DD4
00463DD4    F8              clc
```
