
# 喜欢杀狗的朋友请进，我恨狗！！！！

小弟恨狗，见到狗就非要把它给宰了不可，呵呵，有点变态对不？

好了，看一下我们怎么样把这狗给去掉吧,这个软件是韩国出的喷绘软件，效果比蒙泰5.0要好（他奶奶的，中国的软件什么时候才达到国际水准啊）。

工具：`TRW2000`

目标：`art.exe`

运行`TRW2000`，下断点`bpio 378`，然后运行`art.exe`，BOOM！！！被拦下来的，按几下`F12`，返回到主程式，如下：

```
:0045FCA3 8D85E0FBFFFF            lea eax, dword ptr [ebp+FFFFFBE0]
:0045FCA9 50                      push eax

* Reference To: SP32W.RNBOproInitialize, Ord:0000h
              |
:0045FCAA E87DAA0100              Call 0047A72C
:0045FCAF 668945E4                mov word ptr [ebp-1C], ax  <======返回到这里
:0045FCB3 8B4DE4                  mov ecx, dword ptr [ebp-1C]
:0045FCB6 81E1FFFF0000            and ecx, 0000FFFF
:0045FCBC 85C9                    test ecx, ecx
:0045FCBE 7430                    je 0045FCF0
:0045FCC0 C78598FBFFFF00000000    mov dword ptr [ebp+FFFFFB98], 00000000
:0045FCCA C645FC00                mov [ebp-04], 00
:0045FCCE 8D4DE8                  lea ecx, dword ptr [ebp-18]
:0045FCD1 E8C61B0300              call 0049189C
:0045FCD6 C745FCFFFFFFFF          mov [ebp-04], FFFFFFFF
:0045FCDD 8D4DF0                  lea ecx, dword ptr [ebp-10]
:0045FCE0 E8B71B0300              call 0049189C
:0045FCE5 8B8598FBFFFF            mov eax, dword ptr [ebp+FFFFFB98]
:0045FCEB E920010000              jmp 0045FE10
```

接下来一直按`F10`，直到下面的代码处：

```
:0045FDD8 E8BF1A0300              call 0049189C
:0045FDDD 8B8590FBFFFF            mov eax, dword ptr [ebp+FFFFFB90]  <======注意，放标志了，如果这里返回0，则没有狗，把它改为：B8 01 00 00 00 90，就把狗宰了，呵呵
:0045FDE3 EB2B                    jmp 0045FE10

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0045FDA6(C)
|
:0045FDE5 C7858CFBFFFF01000000    mov dword ptr [ebp+FFFFFB8C], 00000001
:0045FDEF C645FC00                mov [ebp-04], 00
:0045FDF3 8D4DE8                  lea ecx, dword ptr [ebp-18]
:0045FDF6 E8A11A0300              call 0049189C
:0045FDFB C745FCFFFFFFFF          mov [ebp-04], FFFFFFFF
:0045FE02 8D4DF0                  lea ecx, dword ptr [ebp-10]
:0045FE05 E8921A0300              call 0049189C
:0045FE0A 8B858CFBFFFF            mov eax, dword ptr [ebp+FFFFFB8C]

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:0045FC9E(U), :0045FCEB(U), :0045FD54(U), :0045FDE3(U)
|
:0045FE10 8B4DF4                  mov ecx, dword ptr [ebp-0C]
:0045FE13 64890D00000000          mov dword ptr fs:[00000000], ecx
:0045FE1A 5F                      pop edi
:0045FE1B 5E                      pop esi
:0045FE1C 8BE5                    mov esp, ebp
:0045FE1E 5D                      pop ebp
:0045FE1F C3                      ret
```

小弟第一次写破解，好累啊，小弟下一编文章是破蒙泰5.0的`ROCKEY-4`狗，请期待啦。呵呵
