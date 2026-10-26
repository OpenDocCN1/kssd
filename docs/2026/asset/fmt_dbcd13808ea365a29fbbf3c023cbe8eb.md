
# 解码流程分析

```asm
006AF136     74 0A                      je short DrvStudy.006AF142

006AF138     EB 00                      jmp short DrvStudy.006AF13A

006AF13A     3C E9                      cmp al,0E9

006AF13C     74 04                      je short DrvStudy.006AF142

006AF13E     43                         inc ebx

006AF13F     49                         dec ecx

006AF140   ^ EB EB                      jmp short DrvStudy.006AF12D

006AF142     8B06                       mov eax,dword ptr ds:[esi]         ; 在这里设置一个断点
```

这里找`opcode`为`e8`或`e9`的地方, `F9`运行后断下, `ebx=1FE`, `eax=[esi]`的内容, 紧接着会比较`esi`指向的地方

```asm
006AF144    /EB 00                      jmp short DrvStudy.006AF146

006AF146    \803E 36                    cmp byte ptr ds:[esi],36           ; 比较第2个字节的opcode是否为36

006AF149   ^ 75 F3                      jnz short DrvStudy.006AF13E        ; 是就不跳, 这里为36不会跳

006AF14B     24 00                      and al,0

006AF14D     C1C0 18                    rol eax,18                         ; 解码

006AF150     2BC3                       sub eax,ebx                        ; 解码

006AF152     8906                       mov dword ptr ds:[esi],eax         ; 解码后的代码存入esi指向的地方

006AF154     83C3 05                    add ebx,5

006AF157     83C6 04                    add esi,4

006AF15A     83E9 05                    sub ecx,5

006AF15D   ^ EB CE                      jmp short DrvStudy.006AF12D        ; 向上跳
```

可以判断出上面的这段代码是解码的, `C30000`处的代码会解码, 在`006AF15D`的下一个指令设好断点, `F9`运行, 断下

```asm
006AF15F     5B                         pop ebx                            ; 断下, 解码完毕

006AF160     5E                         pop esi

006AF161     59                         pop ecx

006AF162     58                         pop eax

006AF163     EB 08                      jmp short DrvStudy.006AF16D        ; 跳

006AF16D     8BC8                       mov ecx,eax                        ; 来到这里

006AF16F     8B3E                       mov edi,dword ptr ds:[esi]         ; 代码段偏移

006AF171     03BD 22040000              add edi,dword ptr ss:[ebp+422]     ; 基地址+偏移, 指向代码段

006AF177     8BB5 52010000              mov esi,dword ptr ss:[ebp+152]     ; 解码后的正确代码地址

006AF17D     C1F9 02                    sar ecx,2

006AF180     F3:A5                      rep movs dword ptr es:[edi],dword >; 填充正确的代码

006AF182     8BC8                       mov ecx,eax

006AF184     83E1 03                    and ecx,3

006AF187     F3:A4                      rep movs byte ptr es:[edi],byte pt>; 填充正确的代码

006AF189     5E                         pop esi

006AF18A     68 00800000                push 8000

006AF18F     6A 00                      push 0

006AF191     FFB5 52010000              push dword ptr ss:[ebp+152]

006AF197     FF95 51050000              call dword ptr ss:[ebp+551]        ; VirtualFree

006AF19D     83C6 08                    add esi,8

006AF1A0     833E 00                    cmp dword ptr ds:[esi],0           ; 解码完成?没完成就跳

006AF1A3   ^ 0F85 1EFFFFFF              jnz DrvStudy.006AF0C7              ; 又跳到上面
```

因此可以确定这一大段`006AF0C7 - 006AF1A3`都是解码的代码, 在`006AF1A3`处的下1条指令设置一个断点, `F9`运行, 断下

```asm
006AF1A9     68 00800000                push 8000                          ; 断下

006AF1AE     6A 00                      push 0

006AF1B0     FFB5 56010000              push dword ptr ss:[ebp+156]

006AF1B6     FF95 51050000              call dword ptr ss:[ebp+551]        ; VirtualFree
```

释放内存了

```asm
006AF1BC     8B9D 31050000              mov ebx,dword ptr ss:[ebp+531]     ; ebx=0

006AF1C2     0BDB                       or ebx,ebx

006AF1C4     74 08                      je short DrvStudy.006AF1CE         ; ebx==0,跳

006AF1C6     8B03                       mov eax,dword ptr ds:[ebx]

006AF1C8     8785 35050000              xchg dword ptr ss:[ebp+535],eax

006AF1CE     8B95 22040000              mov edx,dword ptr ss:[ebp+422]     ; 跳到这里, ebx=0x400000, 基地址

006AF1D4     8B85 2D050000              mov eax,dword ptr ss:[ebp+52D]     ; eax=0x400000

006AF1DA     2BD0                       sub edx,eax

006AF1DC     74 79                      je short DrvStudy.006AF257         ; ecx==0, 跳
```

下面就是恢复`IAT`了

```asm
006AF257     8B95 22040000              mov edx,dword ptr ss:[ebp+422]     ; 跳到这里, edx=0x400000, 基地址

006AF25D     8BB5 41050000              mov esi,dword ptr ss:[ebp+541]     ; esi=0

006AF263     0BF6                       or esi,esi

006AF265     74 11                      je short DrvStudy.006AF278         ; esi==0, 跳

006AF267     03F2                       add esi,edx

006AF269     AD                         lods dword ptr ds:[esi]

006AF26A     0BC0                       or eax,eax

006AF26C     74 0A                      je short DrvStudy.006AF278

006AF26E     03C2                       add eax,edx

006AF270     8BF8                       mov edi,eax

006AF272     66:AD                      lods word ptr ds:[esi]

006AF274     66:AB                      stos word ptr es:[edi]

006AF276   ^ EB F1                      jmp short DrvStudy.006AF269

006AF278     BE 00001E00                mov esi,1E0000                     ; 跳到这里, esi=1e0000

006AF27D     8B95 22040000              mov edx,dword ptr ss:[ebp+422]     ; edx=基地址

006AF283     03F2                       add esi,edx                        ; esi=51e0000

006AF285     8B46 0C                    mov eax,dword ptr ds:[esi+C]       ; 指向下一个dll文件名的RVA

006AF288     85C0                       test eax,eax                       ; 不为空就不跳

006AF28A     0F84 0A010000              je DrvStudy.006AF39A

006AF290     03C2                       add eax,edx                        ; 得到下一个dll文件名

006AF292     8BD8                       mov ebx,eax

006AF294     50                         push eax

006AF295     FF95 4D0F0000              call dword ptr ss:[ebp+F4D]        ; 取kernel32.dll的句柄

006AF29B     85C0                       test eax,eax                       ; 成功, 跳

006AF29D     75 07                      jnz short DrvStudy.006AF2A6

006AF29F     53                         push ebx

006AF2A0     FF95 510F0000              call dword ptr ss:[ebp+F51]

006AF2A6     8985 45050000              mov dword ptr ss:[ebp+545],eax     ; 跳到这里, kernel32.dll句柄存入ebp+545

006AF2AC     C785 49050000 00000000     mov dword ptr ss:[ebp+549],0       ; [ebp+549]=0

006AF2B6     8B95 22040000              mov edx,dword ptr ss:[ebp+422]     ; edx=基地址

006AF2BC     8B06                       mov eax,dword ptr ds:[esi]         ; eax=0

006AF2BE     85C0                       test eax,eax

006AF2C0     75 03                      jnz short DrvStudy.006AF2C5        ; eax==0, 不跳

006AF2C2     8B46 10                    mov eax,dword ptr ds:[esi+10]      ; IAT的FirThunk基地址

006AF2C5     03C2                       add eax,edx

006AF2C7     0385 49050000              add eax,dword ptr ss:[ebp+549]     ; 指向下1个API

006AF2CD     8B18                       mov ebx,dword ptr ds:[eax]

006AF2CF     8B7E 10                    mov edi,dword ptr ds:[esi+10]

006AF2D2     03FA                       add edi,edx

006AF2D4     03BD 49050000              add edi,dword ptr ss:[ebp+549]

006AF2DA     85DB                       test ebx,ebx

006AF2DC     0F84 A2000000              je DrvStudy.006AF384

006AF2E2     F7C3 00000080              test ebx,80000000

006AF2E8     75 04                      jnz short DrvStudy.006AF2EE

006AF2EA     03DA                       add ebx,edx

006AF2EC     43                         inc ebx                            ; 取函数名

006AF2ED     43                         inc ebx                            ; 取函数名

006AF2EE     53                         push ebx

006AF2EF     81E3 FFFFFF7F              and ebx,7FFFFFFF

006AF2F5     53                         push ebx                           ; 函数名入栈

006AF2F6     FFB5 45050000              push dword ptr ss:[ebp+545]        ; kernel32.dll句柄入栈

006AF2FC     FF95 490F0000              call dword ptr ss:[ebp+F49]        ; GetProcAddress

006AF302     85C0                       test eax,eax

006AF304     5B                         pop ebx
```
