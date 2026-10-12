
# 小析 bLaCk-eye.UnPackMe.No.1

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-629.htm`](https://www.pediy.com/kssd/pediy07/pediy7-629.htm)

长夜漫漫，无心睡眠

附件下载

`OD` 载入，忽略所有异常

```
00411A78 >  60                 pushad
00411A79     E8 22000000      call UnpackMe.00411AA0
00411A7E     EB 20            jmp short UnpackMe.00411AA0
```

暴力搜索 `VirtualAlloc`

```
00411B60     813F 4C64724C    cmp dword ptr ds:[edi],4C72644C
00411B66     75 15            jnz short UnpackMe.00411B7D
00411B68     817F 04 6F61644>cmp dword ptr ds:[edi+4],4464616F
00411B6F     75 0C            jnz short UnpackMe.00411B7D
00411B71     C785 37264100 0>mov dword ptr ss:[ebp+412637],1
00411B7B     EB 2A            jmp short UnpackMe.00411BA7
00411B7D     813F 56697274    cmp dword ptr ds:[edi],74726956
00411B83     75 62            jnz short UnpackMe.00411BE7
00411B85     817F 04 75616C4>cmp dword ptr ds:[edi+4],416C6175
00411B8C     75 59            jnz short UnpackMe.00411BE7
00411B8E     817F 08 6C6C6F6>cmp dword ptr ds:[edi+8],636F6C6C
00411B95     75 50            jnz short UnpackMe.00411BE7
00411B97     807F 0C 00       cmp byte ptr ds:[edi+C],0
00411B9B     75 4A            jnz short UnpackMe.00411BE7
```

`411c83` 处理三次 `SEH` 后从这里出来，中间会改 `dr` 寄存器，不要下硬件断点

```
00411C67     64:67:8F06 0000  pop dword ptr fs:[0]             ; 0012FFE0
00411C6D     83C4 04          add esp,4
00411C70     66:BE 4746       mov si,4647
00411C74     66:BF 4D4A       mov di,4A4D
00411C78     8A85 99000000    mov al,byte ptr ss:[ebp+99]
00411C7E     E9 92000000      jmp UnpackMe.00411D15
00411D15     FEC8             dec al
00411D17     FEC8             dec al
00411D19     FEC8             dec al
00411D1B     74 28            je short UnpackMe.00411D45       ;走完 3seh 次,跳
00411D53     52               push edx
00411D54     6A 40            push 40
00411D56     68 00300000      push 3000
00411D5B     68 538A0000      push 8A53
00411D60     6A 00            push 0
00411D62     FFD0             call eax                          ;  call VirtualAlloc 我得到的是 3A0000
00411D64     8985 07264100    mov dword ptr ss:[ebp+412607],eax
00411D6A     97               xchg eax,edi
00411D6B     8DB5 D9274100    lea esi,dword ptr ss:[ebp+4127D9]
00411D71     66:813E 4D5A     cmp word ptr ds:[esi],5A4D
00411D76     75 06            jnz short UnpackMe.00411D7E
00411D78     81C6 00760000    add esi,7600
00411D7E     E8 D3010000      call UnpackMe.00411F56            ;  把壳代码 copy 到 VirtualAlloc 的空间
00411D83     5A               pop edx
00411D84     FFE7             jmp edi                            ;  跳进去执行壳代码
```

壳段(`003A0000` 起)开始一些 `SEH`，很快到这里

```
003A016B     8DB5 4B2A4100    lea esi,dword ptr ss:[ebp+412A4B]
003A0171     BF E1870000      mov edi,87E1
003A0176     E8 BA020000      call 003A0435                      ; 貌似一检验 call
003A017B     3B85 472A4100    cmp eax,dword ptr ss:[ebp+412A47]
003A0181     74 02            je short 003A0185                  ; 检验通过则跳
```

取一些 `API` 并把进入 `API` 的中间过程放在 `3B0000` 里

```
003A101E     E8 31F7FFFF      call 003A0754                      ;
003A1023     E8 23F6FFFF      call 003A064B                      ; call 3A0754 和 3A064B 是取 API 地址
003A1028     899D 10AF4100    mov dword ptr ss:[ebp+41AF10],ebx  ; 得到 GetModuleHandleA
003A102E     899D 855D4100    mov dword ptr ss:[ebp+415D85],ebx
003A1034     E8 B2F3FFFF      call 003A03EB
003A1039     FFB5 14AF4100    push dword ptr ss:[ebp+41AF14]
003A103F     E8 10F7FFFF      call 003A0754
003A1044     E8 02F6FFFF      call 003A064B
003A1049     899D 14AF4100    mov dword ptr ss:[ebp+41AF14],ebx  ; 得到 SetUnhandledExceptionFilter
003A106D     50               push eax
003A106E     FF95 14AF4100    call dword ptr ss:[ebp+41AF14]      ; SetUnhandledExceptionFilter(0x3A1080)
```

打 `UnhandledExceptionFilter` 补丁继续

```
003A111F     899D D8AE4100    mov dword ptr ss:[ebp+41AED8],ebx
003A1125     E8 C1F2FFFF      call 003A03EB
003A112A     FFB5 28AF4100    push dword ptr ss:[ebp+41AF28]
003A1130     E8 1FF6FFFF      call 003A0754
```

....到

```
003A120D     E8 39F4FFFF      call 003A064B
003A1212     899D D0AE4100    mov dword ptr ss:[ebp+41AED0],ebx
003A1218     E8 CEF1FFFF      call 003A03EB
003A121D     FFB5 04AF4100    push dword ptr ss:[ebp+41AF04]
003A1223     E8 2CF5FFFF      call 003A0754
003A1228     E8 1EF4FFFF      call 003A064B
003A122D     899D 04AF4100    mov dword ptr ss:[ebp+41AF04],ebx  ; 得到一些 CreateProcess, WriteProcessMemory, 等 API
```

然后有分支了

```
003A1233     8B8D A4AF4100    mov ecx,dword ptr ss:[ebp+41AFA4]
003A1239     8B99 0F264100    mov ebx,dword ptr ds:[ecx+41260F]
```

比较 `400003`(就是 `MZ`)后是否为 `90`

```
003A123F     807B 03 90       cmp byte ptr ds:[ebx+3],90         ; 这里可以把 400003 改成 90
003A1243     0F84 CC000000    je 003A1315                         ; 跳
```

如果不跳，后面会 `CreateProcess(自身,..,CREATE_SUSPEND)` 挂起，`VirtualProtectEx` 把 `400003` 改成可写

`WriteProcessMemory` 把 `400003` 改成 `90`，`ResumeThread` 跑，然后 `ExitProcess` 自身

改成 `90` 后继续走下去，拿了一些 `api`

因为会弹 `NAG`，所以下 `MessageBoxA` 断点

```
77D3B064 >  833D D0C3D677 0>cmp dword ptr ds:[77D6C3D0],0
77D3B06B     0F85 405B0100    jnz User32.77D50BB1
77D3B071     6A 00            push 0
77D3B073     FF7424 14        push dword ptr ss:[esp+14]
77D3B077     FF7424 14        push dword ptr ss:[esp+14]
77D3B07B     FF7424 14        push dword ptr ss:[esp+14]
77D3B07F     FF7424 14        push dword ptr ss:[esp+14]
77D3B083     E8 03000000      call User32.MessageBoxExA
77D3B088     C2 1000          retn 10                             ; 在返回处下断,头上有检测
```

断下后，返回到这儿

```
003A412E     FF95 BCAE4100    call dword ptr ss:[ebp+41AEBC]      ; call MessageBoxA 弹 NAG
003A4134     8D85 64694100    lea eax,dword ptr ss:[ebp+416964]
003A413A     50               push eax
```

再后面，对各个区段解压

```
003A451A     8B3E             mov edi,dword ptr ds:[esi]
003A451C     85FF             test edi,edi
003A451E     0F84 34020000    je 003A4758                           ;全部解压后从这里跳出来, dump 的最佳时机
003A4524     03BD 636D4100    add edi,dword ptr ss:[ebp+416D63]
003A452A     8B46 04          mov eax,dword ptr ds:[esi+4]
003A452D     8B16             mov edx,dword ptr ds:[esi]
003A452F     0385 6F6D4100    add eax,dword ptr ss:[ebp+416D6F]
003A4535     0385 576D4100    add eax,dword ptr ss:[ebp+416D57]
003A453B     57               push edi
003A453C     50               push eax
003A453D     83BD 7D694100 0>cmp dword ptr ss:[ebp+41697D],2     ; 需要解压
003A4544     75 17            jnz short 003A455D
003A4546     57               push edi
003A4547     50               push eax
003A4548     E8 5A280000      call 003A6DA7
003A454D     58               pop eax
003A454E     5F               pop edi
003A454F     50               push eax
003A4550     E8 34280000      call 003A6D89
003A4555     8946 04          mov dword ptr ds:[esi+4],eax
003A4558     83C6 08          add esi,8
003A455B    ^ EB BD           jmp short 003A451A
003A455D     E8 87270000      call 003A6CE9                       ;解压 call
003A4562     83C4 08          add esp,8
003A4565     8946 04          mov dword ptr ds:[esi+4],eax
003A4568     83C6 08          add esi,8
003A456B    ^ EB AD           jmp short 003A451A
```

然后和 `VXD` 打了一会交道，边走边解压，边 `seh` 有意义的地方不多

把 `4042D5`(执行完 `stolen oep` 返回后，第一个 `call` 进的地方) 的头 2 行，偷到 `3C0000` 处 再跳回去

```
003A5EC7     83E9 05          sub ecx,5
003A5ECA     03CB             add ecx,ebx
003A5ECC     8948 01          mov dword ptr ds:[eax+1],ecx        ;ecx 指向跳回去的地址
```

马上下面这段会破坏 `ESP` 造成异常，`OD` 过不去，好在看到出口，直接在出口处下断迎接

```
003A5F00     E8 00000000      call 003A5F05
003A5F05     5D               pop ebp
003A5F06     81ED DE864100    sub ebp,4186DE
003A5F0C     8B6424 08        mov esp,dword ptr ss:[esp+8]
003A5F10     8D85 F1864100    lea eax,dword ptr ss:[ebp+4186F1]   ; eax 指向出口
003A5F16     50               push eax                              ; 异常
003A5F17     C3               retn
```



# 汇编代码分析与执行流程

## 1. 出口处理
`003A5F18  64:8F05 0000000>pop dword ptr fs:[0] ;出口`

`003A5F1F  83C4 04  add esp,4`

`003A5F22  C685 4EAC4100 0>mov byte ptr ss:[ebp+41AC4E],0`

`003A5F29  60  pushad`

## 2. 处理 IAT
`003A61CC  96  xchg eax,esi ; esi 指向 dll 的名字, dll 名字明码`

`003A61CD  E8 DBA2FFFF  call 003A04AD ; 取 dll 名字`

`003A61D2  96  xchg eax,esi`

`003A61D3  60  pushad`

`003A61D4  8DBD 9D934100  lea edi,dword ptr ss:[ebp+41939D]`

`003A61DA  8A18  mov bl,byte ptr ds:[eax]`

`003A61DC  EB 03  jmp short 003A61E1`

### 2.1 找名字
`003A6206  803F 00  cmp byte ptr ds:[edi],0 ; 比较名字`

`003A6209  74 11  je short 003A621C ; dll 名字正确则跳`

`003A620B  803F 25  cmp byte ptr ds:[edi],25`

`003A620E  74 0C  je short 003A621C`

`003A6210  ^ EB C8  jmp short 003A61DA`

`003A6212  61  popad`

`003A6213  C685 45944100 0>mov byte ptr ss:[ebp+419445],0`

`003A621A  EB 01  jmp short 003A621D`

`003A621C  61  popad`

`003A621D  FF95 34AF4100  call dword ptr ss:[ebp+41AF34] ; LoadLibraryA dll`

`003A6223  85C0  test eax,eax`

## 3. API 名字解密
`003A63B9  53  push ebx`

`003A63BA  8D85 338A4100  lea eax,dword ptr ss:[ebp+418A33]`

`003A63C0  50  push eax`

`003A63C1  8D85 4B944100  lea eax,dword ptr ss:[ebp+41944B]`

`003A63C7  50  push eax`

`003A63C8  E8 71080000  call 003A6C3E`

`003A63CD  5B  pop ebx`

`003A63CE  3803  cmp byte ptr ds:[ebx],al`

`003A63D0  74 02  je short 003A63D4`

`003A63D2  3003  xor byte ptr ds:[ebx],al ; ebx 指向的加密的 api 名字解密`

`003A63D4  43  inc ebx`

`003A63D5  803B 00  cmp byte ptr ds:[ebx],0`

`003A63D8  ^ 75 DF  jnz short 003A63B9`

输入表 `RVA 12000, SIZE 3C8` 大部分是变形加密的, 可以写脚本还原或其它

`003A6382  85DB  test ebx,ebx`

`003A6384  0F84 7B070000  je 003A6B05 ; 一个 dll 搞定后，这里出来`

`003A638A  F7C3 00000080  test ebx,80000000`

`003A6390  74 09  je short 003A639B`

`003A6392  C685 44944100 0>mov byte ptr ss:[ebp+419444],1`

`003A6399  EB 40  jmp short 003A63DB`

`003A639B  C685 44944100 0>mov byte ptr ss:[ebp+419444],0`

## 4. IAT 搞完出来
`003A616B  85C0  test eax,eax`

`003A616D  0F84 300F0000  je 003A70A3 ; IAT 处理完后这里跳出来`

## 5. 时间检测
`003A715B  61  popad`

`003A715C  50  push eax`

`003A715D  0F31  rdtsc`

`003A715F  50  push eax`

`003A7160  0F31  rdtsc`

`003A7162  2B0424  sub eax,dword ptr ss:[esp]`

`003A7165  83C4 04  add esp,4`

`003A7168  3D FF0F0000  cmp eax,0FFF`

`003A716D  ^ 0F87 9A94FFFF  ja 003A060D ; 这里不能跳`

后面下 `VirtualFree` 或走若干步就到这了(`push 3E0000, jmp 进 VirtualFree, retn 回这里`)

`003E0000  55  push ebp ; oep 被偷走的代码`

`003E0001  8D6C24 00  lea ebp,dword ptr ss:[esp]`

`003E0005  E9 A1010000  jmp 003E01AB`

`003E000A  E9 EB000000  jmp 003E00FA`

`003E000F  E9 D6000000  jmp 003E00EA`

`003E0014  E9 70020000  jmp 003E0289`

都是 `jmp`, 不是很长

`003E0289  6A 01  push 1`

`003E028B  C70424 EC1D4000  mov dword ptr ss:[esp],401DEC`

`003E0292  - FF2424  jmp dword ptr ss:[esp] ; 回到 主代码段 401DEC 处`

## 6. 真实 oep
`00401D93 >  55  push ebp`

`00401D94  8BEC  mov ebp,esp`

`00401D96  6A FF  push -1`

`00401D98  68 50354100  push bLaCk-ey.00413550`

`00401D9D  68 1C434000  push bLaCk-ey.0040431C`

`00401DA2  64:A1 00000000  mov eax,dword ptr fs:[0]`

`00401DA8  50  push eax`

`00401DA9  64:8925 0000000>mov dword ptr fs:[0],esp`

`00401DB0  83C4 A8  add esp,-58`

`00401DB3  53  push ebx`

`00401DB4  56  push esi`

`00401DB5  57  push edi`

`00401DB6  8965 E8  mov dword ptr ss:[ebp-18],esp`

`00401DB9  FF15 A0214100  call dword ptr ds:[<&kernel32.GetVe>; kernel32.GetVersion`

`00401DBF  33D2  xor edx,edx`

`00401DC1  8AD4  mov dl,ah`

`00401DC3  8915 ECA04100  mov dword ptr ds:[41A0EC],edx`

`00401DC9  8D08  lea ecx,dword ptr ds:[eax]`

`00401DCB  81E1 FF000000  and ecx,0FF`

`00401DD1  890D E8A04100  mov dword ptr ds:[41A0E8],ecx`

`00401DD7  C1E1 08  shl ecx,8`

`00401DDA  03CA  add ecx,edx`

`00401DDC  890D E4A04100  mov dword ptr ds:[41A0E4],ecx`

`00401DE2  C1E8 10  shr eax,10`

`00401DE5  A3 E0A04100  mov dword ptr ds:[41A0E0],eax`

`00401DEA  6A 01  push 1`

`00401DEC  E8 E4240000  call bLaCk-ey.004042D5`

`00401DF1  59  pop ecx`

`00401DF2  85C0  test eax,eax`

`00401DF4  75 08  jnz short bLaCk-ey.00401DFE`

`00401DF6  6A 1C  push 1C`

`00401DF8  E8 C3000000  call bLaCk-ey.00401EC0`

`00401DFD  59  pop ecx`

`00401DFE  E8 891B0000  call bLaCk-ey.0040398C`

`00401E03  85C0  test eax,eax`

`00401E05  75 08  jnz short bLaCk-ey.00401E0F`

`00401E07  6A 10  push 10`

`00401E09  E8 B2000000  call bLaCk-ey.00401EC0`

感谢 fly 大侠英雄前辈的指导

人道渺渺，仙道茫茫

`4dd466d0e1e0cc123afbe504da2462ca`
