
008F09A4 02 00 4C 00 89 69 EB 25 02 00 4D 00 D6 68 87 83 L 榉?M 棖>

008F09B4 80 00 00 00 04 00 00 00 80 00 00 00 0A 00 00 00 ?..?...

008F09C4 02 00 48 00 25 3C 50 91 02 00 56 00 28 A7 07 B5 H?酐 V??

008F09D4 02 00 48 00 5A 56 8D 4C 02 00 47 00 4B AB 0D 19 H 噚?G??

008F09E4 02 00 47 00 6D 2F CD E2 02 00 47 00 BA 5B 49 4E G?G 宺>

008F09F4 02 00 47 00 ED 52 B7 AF 02 00 47 00 8C 68 8D A0 G 勭?G 梌>

008F0A04 08 00 00 00 03 00 00 00 08 00 00 00 00 00 00 00 .....

008F0A14 40 00 00 00 03 00 00 00 02 00 54 00 11 04 59 C8 @..TБ?

`02`：普通函数标志，用 `008F73D3 call 008F75BC` 函数解密

`08`：特殊函数标志，用 `008F742F call 008F76D9` 函数恢复

`80`：特殊函数标志

对于 `80` 这个标志，用“`008F7498 call [ebx+54]`”这个函数也处理，但得到的结果是错误，猜测作者在 `Obsidium1.3` 里新增了一种处理函数方法。

在这，我们还是用以前的方法，将几处代码修改好：

```
008F75E4             test [esi],8
008F75EC             jnz 008F7685
008F75F8 66:F706 0200 test word ptr [esi], 2
008F7602 /75 47      jnz short 008F764B
008F760E 33C0        xor eax, eax //设断，让程序执行，查看数据面板窗口
008F7613 66:C706 0400 mov word ptr [esi], 4
008F761E 6A 01       push 1
008F7620 50         push eax
008F7621 FF76 04    push dword ptr [esi+4]
008F7624 50         push eax
008F7625 FF75 18    push dword ptr [ebp+18]
008F7628 FF53 54    call [ebx+54]
008F762F 85C0       test eax, eax
008F7635           je 008F7685
008F7641 8907       mov [edi], eax
008F7646 EB 3D      jmp short 008F7685
008F7650 66:C706 0400 mov word ptr [esi], 4
008F7659 0FB756 02  movzx edx, word ptr [esi+2]
008F7661 6A 01      push 1
008F7663 52         push edx
008F7664 6A 00      push 0
008F7666 FF76 04    push dword ptr [esi+4]
008F7669 FF75 18    push dword ptr [ebp+18]
008F766C FF53 54    call [ebx+54]
008F7674 85C0       test eax, eax
008F7679 74 0A      je 008F7685
```

`008F760E` 中断后，查看数据面板窗口：

```
00955000 7C80C729 kernel32.lstrcpyA
00955004 7C81EE79 kernel32.lstrcmpA
00955008 7C838CB9 kernel32.GetStringTypeA
0095500C 7C80CEC4 kernel32.LCMapStringW
00955010 7C832E2B kernel32.LCMapStringA
00955014 7C809CAD kernel32.MultiByteToWideChar
00955018 00B60048                   //这个是 80 标志的函数
```

……

为了了解代码是如何处理 `80` 标志的函数，我们跟进 `00955018` 这个 `IAT` 地址所指的 `00B60048` 函数。按 `Ctrl` ＋ `G` 打开地址框，输入 `0B60048`（注意：地址首位如果是字母，别忘加个 `0`）：

```
00B60048 60       pushad //在这按右键新建 EIP 源，将当前 EIP 指针指到此处
00B60049 66:BF B55B mov di, 5BB5
00B6004D B2 80    mov dl, 80
00B6004F - E9 E350D9FF jmp 008F5137
```

除去花指令后的代码：

```
008F52C7 90       nop
008F52C8 90       nop
008F52C9 90       nop
008F52CA 90       nop
008F52CB 90       nop
008F52CC 0FB6D2   movzx edx, dl
008F52CF 90       nop
008F52D0 90       nop
008F52D1 90       nop
008F52D2 0FB7C7   movzx eax, di
```

……

```
008F5399 8D34C6   lea esi, [esi+eax*8] //来到这里，查看 ESI
008F53A0 0FB706   movzx eax, word ptr [esi]
008F53AC 83F8 04  cmp eax, 4
008F53B5 0F84 78010000 je 008F5533
008F53C4 83F8 01  cmp eax, 1
008F53CA 0F84 11010000 je 008F54E1
008F53DA 3D 80000000 cmp eax, 80
008F53E3 /0F84 AE070000 je 008F5B97//这里会跳
```

到 `008F5209` 这句查看 `ESI`：

```
008F09B4 80 00 00 00 04 00 00 00 80 00 00 00 0A 00 00 00 ?..?...
```

这个就是上面的那个数据表，`80` 就是特殊函数标志，后面的 `04` 就是与函数名有关的参数。

```
008F5BBB 8BD5     mov edx, ebp
008F5BBD 90       nop
008F5BBE 90       nop
008F5BBF 90       nop
008F5BC0 90       nop
008F5BC1 90       nop
008F5BC2 90       nop
008F5BC3 039485 9401EC00 add edx, [ebp+eax*4+EC0194]
008F5BCA 90       nop
008F5BCB 90       nop
008F5BCC 90       nop
008F5BCD 90       nop
008F5BCE FFE2     jmp edx
```

继续走，来到如下：

```
008F5D23 90       nop
008F5D24 90       nop
008F5D25 90       nop
008F5D26 8B83 C4010000 mov eax, [ebx+1C4]
008F5D2C 90       nop
008F5D2D 90       nop
008F5D2E 90       nop
008F5D2F 90       nop
008F5D30 90       nop
008F5D31 90       nop
008F5D32 ^ E9 E7FDFFFF  jmp 008F5B1E
```

查看 `ebx+1C4`，会发现一组代码的地址，原来程序是根据不同的函数参数确定函数名，你可以 `Ctrl` ＋ `G` 输入下面的地址查看代码：

```
008F0390 008F223D 008F22C7 008F23AB 008F232D
008F03A0 008F2423 008F2489 008F24EF 008F2555
008F03B0 008F2633 008F269F 008F2705 008F27E9
008F03C0 008F285B 008F25C1 008F2777 008F28C7
008F03D0 008F2159 008F21CB 008F20A1 008F20FD
008F03E0 008F1FD5 008F203B 008F1E73 008F1EF1
008F03F0 008F1F63 008F2951 008F29C3 008F2A47
008F0400 008F2AC5 008F2B2B 008F1C98 00000000
```

上面每个地址是一段代码的入口，处理某个 `API` 函数，下面要做的就是找到这些代码共同出口点，为补丁程序做准备。继续：

```
008F5B26 8038 CC  cmp byte ptr [eax], 0CC
008F5B2E 74 1B    je short 008F5B4B
008F5B38 894424 1C mov [esp+1C], eax
008F5B3C 90       nop
008F5B3D 90       nop
008F5B3E 90       nop
008F5B3F 90       nop
008F5B40 61       popad
008F5B41 90       nop
008F5B42 90       nop
008F5B43 90       nop
008F5B44 90       nop
008F5B45 FFE0     jmp eax //根据参数不同，跳到不同代码里获取函数地址
```

这次跟进 `eax` 为 `008F2423` 值，`jmp 008F2423` 地址，来到：

```
008F2423 55       push ebp
008F2424 8BEC     mov ebp, esp
008F2426 83EC 08  sub esp, 8
008F2429 53       push ebx
008F242A 56       push esi
008F242B 57       push edi
008F242C E8 00000000 call 008F2431
008F2431 5B       pop ebx
008F2432 8BF3     mov esi, ebx
008F2434 8B9B 4FF4FFFF mov ebx, [ebx-BB1]
008F243A 8975 F8  mov [ebp-8], esi
008F243D 8DBE A7070000 lea edi, [esi+7A7]
008F2443 BE 04000000 mov esi, 4
008F2448 8B07     mov eax, [edi]
008F244A 85C0     test eax, eax
008F244C 74 14    je short 008F2462
008F244E 8D55 FC  lea edx, [ebp-4]
008F2451 FF75 08  push dword ptr [ebp+8]
008F2454 52       push edx
008F2455 FFD0     call eax
008F2457 83F8 01  cmp eax, 1
008F245A 74 21    je short 008F247D
008F245C 83C7 04  add edi, 4
008F245F 4E       dec esi
008F2460 ^ 75 E6  jnz short 008F2448
008F2462 8B75 F8  mov esi, [ebp-8]
008F2465 FF75 08  push dword ptr [ebp+8]
008F2468 FFB6 57090000 push dword ptr [esi+957] //esi+957 指向的就是正确的函数名，按 F7 跟进
008F246E FF93 84000000 call [ebx+84]
008F2474 5F       pop edi
008F2475 5E       pop esi
008F2476 5B       pop ebx
008F2477 8BE5     mov esp, ebp
008F2479 5D       pop ebp
008F247A C2 0400  retn 4
```

`008F2468` 这个 `CALL` 进入会来到外壳自己的领空，其地址是 `4xxxxx`：

```
0040C633 /EB 01   jmp short 0040C636 //在这可以用 mov eax, [esp+4] 指令取得函数的地址
0040C635 |5D      pop ebp
```
