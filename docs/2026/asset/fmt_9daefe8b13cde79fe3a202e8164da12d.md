
```
006E5966   push dword ptr ds:[6D4EDF]
006E596C   pop dword ptr ds:[6D4F03]                     ; KERNEL32.77E887E7
006E5972   push dword ptr ds:[6D4F03]
006E5978   mov eax,dword ptr ss:[esp] //stolen code第7行,mov eax,4cbff8的变形
006E597B   pop dword ptr ds:[6D4EFF]                     ; KERNEL32.77E887E7
006E5981   nop
006E5982   nop
```

以上代码要一步一步按F7跟踪，注意执行前后各寄存器值的变化以及堆浅`esp`的变化，以便确认是附值还是`push`，从而得出正确的stolen code，并作好纪录。到`006e5981`后按F9继续运行。出现`int3`异常中断：

```
006E2CFE   int3
006E2CFF   nop
006E2D00   pop dword ptr fs:[0]                          ; 0012FFE0
006E2D06   add esp,4
006E2D09   pushad
006E2D0A   call ACProtec.006E2D0F
006E2D0F   pop esi                                       ; 0012FFE0
006E2D10   sub esi,6
006E2D13   mov ecx,5B
006E2D18   sub esi,ecx
006E2D1A   mov edx,8C98BB7
006E2D1F   shr ecx,2
006E2D22   sub ecx,2
006E2D25   cmp ecx,0
006E2D28   jl short ACProtec.006E2D44
006E2D2A   mov eax,dword ptr ds:[esi+ecx*4]
006E2D2D   mov ebx,dword ptr ds:[esi+ecx*4+4]
006E2D31   xor eax,ebx
006E2D33   ror eax,12
006E2D36   add eax,edx
006E2D38   sub edx,7D69989C
006E2D3E   mov dword ptr ds:[esi+ecx*4],eax
006E2D41   dec ecx
006E2D42   jmp short ACProtec.006E2D25
006E2D44   popad
006E2D45   popad
006E2D46   retn //在此处按F2,然后按shift-F9，停在此处再按F2取消，F7走1步
```

此时再次察看`ESP=12FF90`，去转存窗口，Ctrl+G:`0012FF90` 到达`0012FF90`内存处，在`0012FF90`处的4个字节上，下硬件访问－>`DWord`断点。F9运行，断下。

由于执行以下代码包含许多`call`，会影响程序代码和数据的变化，因此在这儿用LordPE完全DUMP进程。

接着前面继续分析stolen code。以下仍然是F7单步跟踪，遇到`call`时再F8。

```
006ED0E3   push esi
006ED0E4   mov dword ptr ss:[esp],ecx
006ED0E7   push eax                                      ; ACProtec.004CBFF8
006ED0E8   mov eax,ACProtec.00406EDC
006ED0ED   mov ecx,eax                                   ; ACProtec.004CBFF8
006ED0EF   pop eax                                       ; ACProtec.004CBFF8
006ED0F0   push ecx
006ED0F1   pop dword ptr ds:[6D4F0F]
006ED0F7   pop dword ptr ds:[6D4EFF]                     ; ACProtec.004CBFF8
006ED0FD   mov ecx,dword ptr ds:[6D4EFF]                 ; ACProtec.004CBFF8
006ED103   call dword ptr ds:[6D4F0F]  //stolen code第8行，call 406edc的变形
006ED109   mov eax,dword ptr ds:[4F258C]  //stolen code第9行,没变形
006ED10E   mov eax,dword ptr ds:[eax]     //stolen code第10行，没变形
006ED110   mov dword ptr ds:[6D4EFB],ACProtec.004622F8
006ED11A   push dword ptr ds:[6D4EFB]
006ED120   pop dword ptr ds:[6D4F0B]
006ED126   call dword ptr ds:[6D4F0B]  //stolen code第11行，call 4622f8的变形
006ED12C   mov dword ptr ds:[6D4EF7],edi                 ; ACProtec.005D16A8
006ED132   nop
006ED133   pushad
006ED134   call ACProtec.006E1460
006ED139   mov eax,dword ptr ss:[ebp+41A27A]
006ED13F   add eax,dword ptr ss:[ebp+40D2AA]
006ED145   mov dword ptr ss:[ebp+41A27A],eax             ; ACProtec.004CBFF8
006ED14B   popad
006ED14C   push dword ptr ds:[6D4EF7]
006ED152   push eax                                      ; ACProtec.004CBFF8
006ED153   mov eax,ACProtec.004F2354
006ED158   mov edi,eax                                   ; ACProtec.004CBFF8
006ED15A   pop eax                                       ; ACProtec.004CBFF8
006ED15B   mov ecx,dword ptr ds:[edi]  //stolen code第12行，mov ecx,[4f2354]的变形
006ED15D   pop dword ptr ds:[6D4EF3]
006ED163   mov edi,dword ptr ds:[6D4EF3]
006ED169   mov eax,dword ptr ds:[4F258C]  //stolen code第13行，没变形
006ED16E   mov eax,dword ptr ds:[eax]     //stolen code第14行，没变形
006ED170   mov dword ptr ds:[6D4EEF],eax                 ; ACProtec.004CBFF8
006ED176   push dword ptr ds:[6D4EEF]                    ; ACProtec.006D4F0F
006ED17C   mov dword ptr ds:[6D4EEB],ACProtec.0047FB5C
006ED186   mov eax,dword ptr ds:[6D4EEB]
006ED18C   mov edx,dword ptr ds:[eax]  //stolen code第15行，mov edx,[47fb5c]的变形
006ED18E   pop dword ptr ds:[6D4EE7]
006ED194   mov eax,dword ptr ds:[6D4EE7]
006ED19A   push ebx
006ED19B   nop
006ED19C   pushad
006ED19D   call ACProtec.006E1460
006ED1A2   mov byte ptr ss:[ebp+41A237],0E8
006ED1A9   popad
006ED1AA   mov dword ptr ss:[esp],ACProtec.00462310
006ED1B1   pop dword ptr ds:[6D4F07]
006ED1B7   call dword ptr ds:[6D4F07]  //stolen code第16行,call 462310的变形
006ED1BD   push dword ptr ds:[4F228C]                    ; ACProtec.004F45E8
006ED1C3   pop dword ptr ds:[6D4EE3]
006ED1C9   mov ecx,dword ptr ds:[6D4EE3]  //stolen code第17行,mov ecx,[4f228c]的变形
006ED1CF   mov eax,dword ptr ds:[4F258C]  //stolen code第18行,没有变形
006ED1D4   mov eax,dword ptr ds:[eax]     //stolen code第19行,没有变形
006ED1D6   push dword ptr ds:[47F7CC]                    ; ACProtec.0047F818
006ED1DC   pop edx                 //stolen code第20行,mov edx,[47f7cc]的变形
006ED1DD   push ACProtec.00462310
006ED1E2   pop dword ptr ds:[6D4F03]                     ; ACProtec.004CBFF8
006ED1E8   call dword ptr ds:[6D4F03] //stolen code第21行，call 462310的变形
006ED1EE   mov eax,dword ptr ds:[4F258C] //stolen code第22行,没有变形
006ED1F3   mov eax,dword ptr ds:[eax]    //stolen code第23行,没有的变形
006ED1F5   nop
006ED1F6   nop
006ED1F7   nop
006ED1F8   nop
006ED1F9   nop
006ED1FA   pushad
006ED1FB   call ACProtec.006E1460
006ED200   mov dword ptr ss:[ebp+41A238],25FF
006ED20A   lea eax,dword ptr ss:[ebp+41A27A]
006ED210   mov dword ptr ss:[ebp+41A23A],eax             ; ACProtec.004CBFF8
006ED216   call ACProtec.006E1460
006ED21B   lea edi,dword ptr ss:[ebp+419F20]
006ED221   lea ecx,dword ptr ss:[ebp+41A22C]
006ED227   sub ecx,edi                                   ; ACProtec.005D16A8
006ED229   shr ecx,2
006ED22C   call ACProtec.006E11FE
006ED231   stos dword ptr es:[edi]
006ED232   loopd short ACProtec.006ED22C
006ED234   popad
006ED235   jmp short ACProtec.006ED238
006ED238   jmp dword ptr ds:[6ED27A]  //飞向光明之巅！JMP 04cc263
```

## 2. 修复输入表

为了还原加密了IAT，随便在程序一个空地（偶是选择`6d4525`处），写入以下代码并执行：

```
006D4D25   pushad
006D4D26   mov esi,ACProtec.006D4010
006D4D2B   mov edi,dword ptr ds:[esi+8]
006D4D2E   xor dword ptr ds:[esi+1],edi                  ; ACProtec.005D16A8
006D4D31   add esi,0D
006D4D34   cmp byte ptr ds:[esi],68
006D4D37   je short ACProtec.006D4D2B
006D4D39   popad
```

运行ImportREC，选择这个进程。把OEP改为`00CC263`，点IT AutoSearch，发现完全不对。于是回到OLLYDBG，找到能正确显示API调用的代码，偶随便找到`4074a8`处，于是在ImportREC中把OEP修改为`0074a8`，点IT AutoSearch，点“Get Import”，用层次1即可修复全部无效函数数。FixDump！

## 3. 修复入口Stolen code

根据分析变形后stolen code得出程序的入口代码，并在`004cc210`开始处补上：

```
push ebp
mov ebp,esp
sub esp,10   (变形后为：push esi；push eax；push ebx；push esi）
mov eax,04cbff8
call 406edc
mov eax,[4f528c]
mov eax,[eax]
call 4622f8
mov ecx,[4f2354]
mov edx,[47fb5c]
mov eax,[4f258c]
mov eax,[eax]
call 462310
mov ecx,[4f228c]
mov eax,[4f258c]
mov eax,[eax]
mov edx,[47f7cc]
call 462310
mov eax,[4f258c]
mov eax,[eax]
```

## 4. 程序还有许多暗桩有待修复，这里只讨论脱壳
