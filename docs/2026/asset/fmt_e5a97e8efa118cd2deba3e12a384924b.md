
```text
0040D1F7        E8 0E000000       call NOTEPAD.0040D20A
0040D1FC        8985 A0274000     mov dword ptr ss:[ebp+4027A0],eax
0040D202        8D85 A01F4000     lea eax,dword ptr ss:[ebp+401FA0]
0040D208        50                push eax
0040D209        C3                retn
                                  ====>返回到 0040D213
0040D213        F785 6C254000 100>test dword ptr ss:[ebp+40256C],10
0040D21D        74 37             je short NOTEPAD.0040D256
0040D21F        64:FF35 30000000  push dword ptr fs:[30]
0040D226        58                pop eax
0040D227        85C0              test eax,eax
0040D229        78 0F             js short NOTEPAD.0040D23A
                                  ====>跳
0040D23A        6A 00             push 0
0040D23C        FF95 1A274000     call dword ptr ss:[ebp+40271A]
0040D242        85D2              test edx,edx
0040D244        79 10             jns short NOTEPAD.0040D256
0040D246        837A 08 FF        cmp dword ptr ds:[edx+8],-1
0040D24A        75 0A             jnz short NOTEPAD.0040D256
0040D24C        8B52 04           mov edx,dword ptr ds:[edx+4]
0040D24F        C742 50 00100000  mov dword ptr ds:[edx+50],1000
0040D256        8BBD 64254000     mov edi,dword ptr ss:[ebp+402564]
0040D25C        037F 3C           add edi,dword ptr ds:[edi+3C]
0040D25F        8BB5 64254000     mov esi,dword ptr ss:[ebp+402564]
0040D265        8B4F 54           mov ecx,dword ptr ds:[edi+54]
0040D268        8D85 D2274000     lea eax,dword ptr ss:[ebp+4027D2]
0040D26E        50                push eax
0040D26F        6A 04             push 4
0040D271        51                push ecx
0040D272        FFB5 64254000     push dword ptr ss:[ebp+402564]
0040D278        FF95 2D274000     call dword ptr ss:[ebp+40272D]
0040D27E        F785 6C254000 080>test dword ptr ss:[ebp+40256C],8
0040D288        0F84 A7000000     je NOTEPAD.0040D335
0040D28E        68 04010000       push 104
0040D293        8DBD D2274000     lea edi,dword ptr ss:[ebp+4027D2]
0040D299        57                push edi
0040D29A        6A 00             push 0
0040D29C        FF95 44274000     call dword ptr ss:[ebp+402744]
0040D2A2        6A 00             push 0
0040D2A4        68 80000000       push 80
0040D2A9        6A 03             push 3
0040D2AB        6A 00             push 0
0040D2AD        6A 01             push 1
0040D2AF        68 00000080       push 80000000
0040D2B4        57                push edi
0040D2B5        FF95 54274000     call dword ptr ss:[ebp+402754]
0040D2BB        83F8 FF           cmp eax,-1
                                  ====>把这里的EAX返回值改成FFFFFFFF
0040D2BE        75 04             jnz short NOTEPAD.0040D2C4
                                  ====>这里不跳！这样会快点！    :-)
0040D2C0        33C0              xor eax,eax
0040D2C2        EB 71             jmp short NOTEPAD.0040D335
                                  ====>跳
0040D335        8B85 64254000     mov eax,dword ptr ss:[ebp+402564] ; NOTEPAD.00400000
0040D33B        BB 01000000       mov ebx,1
0040D340        E8 08000000       call NOTEPAD.0040D34D
                                  ====>这个CALL用F8带过
0040D345        8D85 A3214000     lea eax,dword ptr ss:[ebp+4021A3]
0040D34B        50                push eax
0040D34C        C3                retn
                                  ====>返回到 0040D416
0040D416        8B9D 64254000     mov ebx,dword ptr ss:[ebp+402564]  ; NOTEPAD.00400000
                                  ====>[ebp+402564]=00400000
0040D41C        039D 68254000     add ebx,dword ptr ss:[ebp+402568]
                                  ====>EBX=00400000 + 000010CC=004010CC 这就是OEP值  :-)
0040D422        C1CB 07           ror ebx,7
                                  ====>把这里改一下：JMP EBX，让其直接跳到OEP去！
0040D422      - FFE3              jmp ebx ; NOTEPAD.004010CC        //改后的变化
                                  ====>跳到OEP
```

---

```text
004010CC        55                push ebp
                                  ====>在这儿用OllyDump转存调试进程
                                  ====>或者先用LordPE纠正ImageSize，然后完全脱壳！
004010CD        8BEC              mov ebp,esp
004010CF        83EC 44           sub esp,44
004010D2        56                push esi
004010D3        FF15 E4634000     call dword ptr ds:[4063E4]; KERNEL32.GetCommandLineA
```

---

重新运行加壳后的程序，运行`ImportREC`，选择这个进程。把`OEP`改为`000010CC`，点`IT AutoSearch`，点`Get Import`，手动修复几个函数。`FixDump`，正常运行！  54.5K ->64K

---

## 二、在WinXP下脱壳

用XP的记事本加壳实验。别选`API Redirection`、`Erase PE Header`，否则加壳后无法运行。

这次用的是loveboom兄弟的方法脱壳。详见《ExeStealth2.72的简单脱壳(for vb版)!!》

---

```text
01013060        60                pushad
                                  ====>进入OD后断在这！
01013061        E8 00000000       call XP-Notep.01013066
```

`F9`运行，程序会在异常处中断。

```text
010136DD        CD 68             int 68
                                  ====>第一个异常
010136DF        33DB              xor ebx,ebx
010136E1        64:8F03           pop dword ptr fs:[ebx]
010136E4        83C4 04           add esp,4
010136E7        66:81FF 9712      cmp di,1297
010136EC        74 0E             je short 010136FC
```

`Shift+F9`通过异常，2次程序运行。好了，Try Again，按1次`Shift+F9`，停下来。

```text
01013769        0000              add byte ptr ds:[eax],al
                                  ====>第2次异常在这儿！          :-)
                                  ====>看看堆栈区的第二条地址是：0101370C  F2设断
```

`Shift+F9`通过异常，程序会中断在`0101370C`处

```text
0101370C        55                push ebp
                                  ====>堆栈区的第二条地址        下断点！
0101370D        8BEC              mov ebp,esp
0101370F        57                push edi
01013710        8B45 10           mov eax,dword ptr ss:[ebp+10]
01013713        8BB8 C4000000     mov edi,dword ptr ds:[eax+C4]
01013719        FF37              push dword ptr ds:[edi]
0101371B        33FF              xor edi,edi
0101371D        64:8F07           pop dword ptr fs:[edi]
01013720        8380 C4000000 08  add dword ptr ds:[eax+C4],8
01013727        8BB8 A4000000     mov edi,dword ptr ds:[eax+A4]
0101372D        C1C7 07           rol edi,7
01013730        89B8 B8000000     mov dword ptr ds:[eax+B8],edi
01013736        B8 00000000       mov eax,0
0101373B        5F                pop edi
0101373C        C9                leave
0101373D        C3                retn
                                  ====>返回到 77F833A0
```

下面`CTRL+F9`执行到返回！

```text
77F833A0        64:8B25 00000000  mov esp,dword ptr fs:[0]
77F833A7        64:8F05 00000000  pop dword ptr fs:[0]
77F833AE        8BE5              mov esp,ebp
77F833B0        5D                pop ebp
77F833B1        C2 1400           retn 14
                                  ====>返回到 77F83372
77F83372        5F                pop edi
77F83373        5E                pop esi
77F83374        5B                pop ebx
77F83375        C2 1400           retn 14
                                  ====>返回到 77F617EE
77F617EE        F605 8A51FC77 80  test byte ptr ds:[77FC518A],80
…… …… 省 略 …… ……
77F61C9E        B0 01             mov al,1
77F61CA0        5F                pop edi
77F61CA1        5B                pop ebx
77F61CA2        5E                pop esi
77F61CA3        C9                leave
77F61CA4        C2 0800           retn 8
                                  ====>返回到 77F510A6
77F510A6        0AC0              or al,al
                                  ====>在这里 CTRL+F9 ！
```

如果运气好的话会停留到入口点的下条指令！我试了N次，并不是每次都能停到入口点！ :-(

---

```text
01006AE0        6A 70             push 70
                                  ====>这里是OEP   :-)
01006AE2        68 88180001       push 1001888
                                  ====>返回到这里！
01006AE7        E8 BC010000       call 01006CA8
01006AEC        33DB              xor ebx,ebx
01006AEE        53                push ebx
01006AEF        8B3D 4C110001     mov edi,dword ptr ds:[100114C]; kernel32.GetModuleHandleA
```

---

```text
         ,     \_/
```
