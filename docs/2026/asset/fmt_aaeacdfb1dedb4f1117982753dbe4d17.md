
```asm
FF15D0104000            Call dword ptr [004010D0]
:004FD220
DC9DC8FDFFFF            fcomp qword ptr [ebp+FFFFFDC8]
         ====>真假码比较！
 ====>ST=13572468.000000000000
                       ====>SS[0074E81C]=13328314.00000000
:004FD226
DFE0                    fstsw
ax
:004FD228 F6C440
 test ah, 40
:004FD22B 7407
       je 004FD234
                       ====>跳则OVER！
:004FD22D
B801000000              mov eax, 00000001
         ====>置1则OK！
:004FD232
EB02                    jmp
004FD236
\* Referenced
by a (U)nconditional or (C)onditional Jump at Address:
|:004FD22B(C)
|
:004FD234
33C0                    xor
eax, eax
             ====>清0则OVER！
\*
Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004FD232(U)
|
:004FD236
F7D8                    neg
eax
:004FD238 8D4DD4
 lea ecx, dword ptr [ebp-2C]
:004FD23B 668BF0
         mov si, ax
\*
Reference To: MSVBVM60.\_\_vbaFreeStr, Ord:0000h
 |
:004FD23E FF1524124000            Call
dword ptr [00401224]
:004FD244 8D4DD0
     lea ecx, dword ptr [ebp-30]
\*
Reference To: MSVBVM60.\_\_vbaFreeObj, Ord:0000h
 |
:004FD247 FF1528124000            Call
dword ptr [00401228]
\*
Reference To: MSVBVM60.\_\_vbaFreeVarList, Ord:0000h
 |
:004FD24D 8B1D34104000            mov
ebx, dword ptr [00401034]
:004FD253 8D55A8
       lea edx, dword ptr [ebp-58]
:004FD256 8D45B8
               lea eax, dword ptr [ebp-48]
:004FD259
52                      push
edx
:004FD25A 50
   push eax
:004FD25B 6A02
       push 00000002
:004FD25D FFD3
             call ebx
:004FD25F 83C40C
               add esp, 0000000C
:004FD262
6685F6                  test si,
si
:004FD265 0F8455030000            je 004FD5C0
         ====>跳则OVER！
:004FD26B
8D7734                  lea esi,
dword ptr [edi+34]
       ====>下面保存注册信息！
\* Possible StringData Ref from
Code Obj ->"sselect \* from Regsn"
 |
:004FD26E BAF0134200              mov
edx, 004213F0
:004FD273 8BCE
     mov ecx, esi
……
……省 略…… ……
\* Possible
StringData Ref from Code Obj ->"SSn"
 |
:004FD389 B91C144200              mov
ecx, 0042141C
:004FD38E 898DD0FEFFFF            mov
dword ptr [ebp+FFFFFED0], ecx
:004FD394 8B30
         mov esi, dword ptr [eax]
:004FD396 8913
                 mov dword ptr [ebx],
edx
:004FD398 8B95CCFEFFFF            mov edx,
dword ptr [ebp+FFFFFECC]
:004FD39E 50
         push eax
:004FD39F 8985FCFDFFFF
       mov dword ptr [ebp+FFFFFDFC], eax
:004FD3A5 895304
                 mov dword ptr [ebx+04],
edx
:004FD3A8 894B08
 mov dword ptr [ebx+08], ecx
:004FD3AB 8B8DD4FEFFFF
     mov ecx, dword ptr [ebp+FFFFFED4]
:004FD3B1 894B0C
               mov dword ptr [ebx+0C],
ecx
:004FD3B4 FF5628
 call [esi+28]
:004FD3B7 85C0
       test eax, eax
:004FD3B9 DBE2
             fclex
:004FD3BB 7D15
                 jge 004FD3D2
:004FD3BD
8B95FCFDFFFF            mov edx, dword ptr [ebp+FFFFFDFC]
:004FD3C3
6A28                    push
00000028
:004FD3C5 685C0B4200              push
00420B5C
:004FD3CA 52
     push edx
:004FD3CB 50
           push eax
……
……省 略…… ……
\* Reference
To: MSVBVM60.rtcMsgBox, Ord:0253h
                     |
:004FD597
FF1598104000            Call dword ptr [00401098]
         ====>呵呵，胜利女神！
……
……省 略…… ……
\* Reference
To: MSVBVM60.rtcMsgBox, Ord:0253h
                     |
:004FD635
FF1598104000            Call dword ptr [00401098]
         ====>BAD BOY！
```

---

### 算法总结

1. 依次取信息码字符，如果是数字则直接保留，是字母则转化为所对应的ASCII码的10进制值。信息码`5KA089DJ`变为`575650896874`。
2. 取`575650896874`的后8位：`50896874`，并转化成16进制值`03089FEA`。
3. `(03089FEA % 001747E9) * B + 006F780F = 00CB5FBA(H) = 13328314(D)` 就是注册码了！

---

### KeyMake之{70th}内存注册机

中断地址：`004FD202`
中断次数：`1`
第一字节：`89`
指令长度：`6`
寄存器方式：`EDX`（10进制）

---

### 注册信息保存

同目录下的`AcctSet.mdb`中：`RegSn`

---

### 整理

信息码：`5KA089DJ`
注册码：`13328314`

---

```text
,     \_/
        /| \_.-~/
     \\_     ,        青春都一饷
    ( /~   /              \~-._\
|\
       `\\  \_/
   \   ~\ )          忍把浮名
\_-~~~-.)  )\_\_/;;,.          \\_  //'
 /'\_,\   --~   \ ~~~-  ,;;\\_\_\_(  (.-~~~-.
     换了破解轻狂
 `~ \_( ,\_..--\ (     ,;'' /
 ~--   /.\_`\
  /~~//'   /' `~\
) /--..\_, )\_  `~
  "  `~"  "
 `"      /~'`\    `\\~~\
                      "
    "   "~'  ""
               Cracked By 巢水工作坊――fly [OCN][FCG]
             2003-04-25  2:16
```
