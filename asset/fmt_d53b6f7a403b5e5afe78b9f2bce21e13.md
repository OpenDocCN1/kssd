
0046F23E     838B F0204000 02          or dword ptr ds:[ebx+4020F0],2

0046F24A     C9                      leave

0046F24B     C2 0400                 ret n 4///返回到系统代码。

77E1A990     50                      push eax///返回到这里。

77E1A991     E8 E0D40000            call 77E27E76  ; ExitThread///F8 过。

### 三、解压子进程代码

走过 `77E1A991` 时会陷入死循环。`F12` 暂停。然后到 `0046F4A5` 处新建 `EIP`。

```
0046F4A5     /EB 03                jmp short 0046F4AA///新建 EIP。

0046F4A7     |9A 37A7EB01 9AEB     call far EB9A:01EBA737

0046F4AE      039A E767EB01        add ebx,dword ptr ds:[edx+1EB67E7]

0046F4B4      9A EB039AB7 27E8     call far E827:B79A03EB

0046F4BB      0000                add byte ptr ds:[eax],al

0046F4BD      0000                add byte ptr ds:[eax],al
```

新建 `EIP` 后用 `F7` 走几步。

去花后：

```
0046F4C3     5B                      pop ebx

0046F4C9     81EB BF344000           sub ebx,4034BF

0046F4D4     EB 1C                   jmp short 0046F4F2

0046F4FA     8B93 BF1D4000           mov edx,dword ptr ds:[ebx+401DBF]

0046F503     8B72 3C                 mov esi,dword ptr ds:[edx+3C]

0046F50A     8DB432 F8000000         lea esi,dword ptr ds:[edx+esi+F8]

0046F516     8BFE                    mov edi,esi

0046F520     6BB3 76214000 28        imul esi,dword ptr ds:[ebx+402176],28

0046F52B     03F7                    add esi,edi

0046F530     EB 37                   jmp short 0046F569

0046F539     837E 0C 00              cmp dword ptr ds:[esi+C],0

0046F540     74 1D                   je short 0046F55F

0046F547     837E 08 00              cmp dword ptr ds:[esi+8],0

0046F54E     74 0F                   je short 0046F55F

0046F553     83C6 28                 add esi,28

0046F559   ^ EB C1                  jmp short 0046F51C

0046F562     83EE 28                 sub esi,28

0046F56C     83EE 28                 sub esi,28

0046F575     3BF7                    cmp esi,edi

0046F57B     0F82 63020000           jb 0046F7E4

0046F589     52                      push edx

0046F58D     8D83 BC204000           lea eax,dword ptr ds:[ebx+4020BC]

0046F598     50                      push eax

0046F59C     8B83 AC204000           mov eax,dword ptr ds:[ebx+4020AC]

0046F5A7     6A 04                   push 4

0046F5B6     0183 682B4000           add dword ptr ds:[ebx+402B68],eax

0046F5C3     8B46 08                 mov eax,dword ptr ds:[esi+8]

0046F5CF     F783 D8204000 FFFFFFFF  test dword ptr ds:[ebx+4020D8],FFFFFFFF

0046F5DF     0F84 EA010000           je 0046F7CF

0046F5EF     50                      push eax

0046F5F5     8B46 0C                 mov eax,dword ptr ds:[esi+C]

0046F5FB     03C2                    add eax,edx

0046F602     50                      push eax

00046F606      FF93 D31E4000         call near dword ptr ds:[ebx+401ED3]

0046F611     5A                      pop edx

0046F61A     FFB3 BC204000           push dword ptr ds:[ebx+4020BC]

0046F62A     8B83 7A214000           mov eax,dword ptr ds:[ebx+40217A]

0046F638     CD F7                   int 0F7///改为 xor eax,eax

0046F642     0183 CF384000           add dword ptr ds:[ebx+4038CF],eax///破坏正常代码。如果 EAX 为 0 则不影响。

0046F654     E9 26010000             jmp 0046F77F

0046F675     8B4A 3C                 mov ecx,dword ptr ds:[edx+3C]

0046F67B     8D8C0A A8000000         lea ecx,dword ptr ds:[edx+ecx+A8]

0046F687     8B01                    mov eax,dword ptr ds:[ecx]

0046F68C     83F8 00                 cmp eax,0

0046F693     74 32                   je short 0046F6C7

0046F698     0341 04                 add eax,dword ptr ds:[ecx+4]

0046F69E     3B46 0C                 cmp eax,dword ptr ds:[esi+C]

0046F6A4     72 21                   jb short 0046F6C7

0046F6AA     8B46 0C                 mov eax,dword ptr ds:[esi+C]

0046F6B0     0346 08                 add eax,dword ptr ds:[esi+8]

0046F6B8     3B01                    cmp eax,dword ptr ds:[ecx]

0046F6BD     0F87 0C010000           ja 0046F7CF

0046F6CA     8B4A 3C                 mov ecx,dword ptr ds:[edx+3C]

0046F6D1     8D8C0A 88000000         lea ecx,dword ptr ds:[edx+ecx+88]

0046F6DB     8B01                    mov eax,dword ptr ds:[ecx]

0046F6E0     83F8 00                 cmp eax,0

0046F6E6     74 35                   je short 0046F71D

0046F6EC     0341 04                 add eax,dword ptr ds:[ecx+4]

0046F6F2     3B46 0C                 cmp eax,dword ptr ds:[esi+C]

0046F6FA     72 21                   jb short 0046F71D

0046F6FF     8B46 0C                 mov eax,dword ptr ds:[esi+C]

0046F707     0346 08                 add eax,dword ptr ds:[esi+8]

0046F70D     3B01                    cmp eax,dword ptr ds:[ecx]

0046F714     0F87 B5000000           ja 0046F7CF

0046F721     8B4A 3C                 mov ecx,dword ptr ds:[edx+3C]

0046F727     8D8C0A C0000000         lea ecx,dword ptr ds:[edx+ecx+C0]

0046F733     8B01                    mov eax,dword ptr ds:[ecx]

0046F738     83F8 00                 cmp eax,0

0046F740     74 39                   je short 0046F77B

0046F747     0341 04                 add eax,dword ptr ds:[ecx+4]

0046F74D     3B46 0C                 cmp eax,dword ptr ds:[esi+C]

0046F755     72 24                   jb short 0046F77B

0046F75A     8B46 0C                 mov eax,dword ptr ds:[esi+C]

0046F762     0346 08                 add eax,dword ptr ds:[esi+8]

0046F76A     3B01                    cmp eax,dword ptr ds:[ecx]

0046F771     77 5C                   ja short 0046F7CF

0046F784     2983 2F384000           sub dword ptr ds:[ebx+40382F],eax///继续破坏。

0046F78E     8B46 08                 mov eax,dword ptr ds:[esi+8]

0046F795     2B93 C0204000           sub edx,dword ptr ds:[ebx+4020C0]

0046F79F     3B46 10                 cmp eax,dword ptr ds:[esi+10]

0046F7A6     76 0B                   jbe short 0046F7B3

0046F7AC     8B46 10                 mov eax,dword ptr ds:[esi+10]

0046F7B6     8B4E 0C                 mov ecx,dword ptr ds:[esi+C]

0046F7C1     6A 00                   push 0

0046F7C3     52                      push edx

0046F7C4     50                      push eax

0046F7C5     51                      push ecx

0046F7C6     E8 E1F2FFFF             call 0046EAAC///解压各区段代码。

0046F7D6   ^ E9 8EFDFFFF             jmp 0046F569///循环解压。

0046F806     8B93 BF1D4000           mov edx,dword ptr ds:[ebx+401DBF]///到这里就可以 DUMP 了。
```

此时 DUMP 出来 IAT 没被破坏。输入表地址为 `60000`。

继续：

```
0046F80F     8B72 3C                 mov esi,dword ptr ds:[edx+3C]///下面开始填充 IAT 了。

0046F815     8BB3 C4204000           mov esi,dword ptr ds:[ebx+4020C4]

0046F81E     83FE 00                 cmp esi,0

0046F824     0F84 88010000           je 0046F9B2

0046F832     8B4432 0C               mov eax,dword ptr ds:[edx+esi+C]

0046F83B     83F8 00                 cmp eax,0

0046F841     0F84 6B010000           je 0046F9B2

0046F84A     03C2                    add eax,edx

0046F84F     52                      push edx

0046F854     50                      push eax

0046F858     50                      push eax

0046F85D     FF93 F31E4000           call near dword ptr ds:[ebx+401EF3]

0046F866     E8 EDF9FFFF             call 0046F258

0046F86E     5A                      pop edx

0046F872     8BC8                    mov ecx,eax

0046F878     8B7C32 10               mov edi,dword ptr ds:[edx+esi+10]

0046F87F     03FA                    add edi,edx

0046F884     56                      push esi

0046F888     8B0432                  mov eax,dword ptr ds:[edx+esi]

0046F88E     83F8 00                 cmp eax,0

0046F894     75 0B                   jnz short 0046F8A1
```
