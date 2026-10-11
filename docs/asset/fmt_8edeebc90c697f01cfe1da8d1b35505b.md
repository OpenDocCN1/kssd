
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
