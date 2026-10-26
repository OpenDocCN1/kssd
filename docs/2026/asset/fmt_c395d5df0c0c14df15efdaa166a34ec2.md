
### 汇编代码

```
0040307F   .  8D45 8C       lea     eax, dword ptr [ebp-74]
00403082   .  52            push    edx
00403083   .  50            push    eax
00403084   .  C785 44FFFFFF>mov     dword ptr [ebp-BC], 1
0040308E   .  C785 3CFFFFFF>mov     dword ptr [ebp-C4], 2
00403098   .  FFD7          call    edi                              ;  <&MSVBVM60.\_\_vbaVarSub>
0040309A   .  8B1D 10104000 mov     ebx, dword ptr [<&MSVBVM60.\_\_vba>;  MSVBVM60.\_\_vbaVarMove
004030A0   .  8BD0          mov     edx, eax
004030A2   .  8D4D AC       lea     ecx, dword ptr [ebp-54]
004030A5   .  FFD3          call    ebx                              ;  <&MSVBVM60.\_\_vbaVarMove>
004030A7   .  6A 01         push    1
004030A9   .  FF15 DC104000 call    dword ptr [<&MSVBVM60.#570>]     ;  MSVBVM60.rtcFileLength
004030AF   .  8D8D 3CFFFFFF lea     ecx, dword ptr [ebp-C4]
004030B5   .  8985 44FFFFFF mov     dword ptr [ebp-BC], eax
004030BB   .  8D55 AC       lea     edx, dword ptr [ebp-54]
004030BE   .  51            push    ecx
004030BF   .  8D45 8C       lea     eax, dword ptr [ebp-74]
004030C2   .  52            push    edx
004030C3   .  50            push    eax
004030C4   .  C785 3CFFFFFF>mov     dword ptr [ebp-C4], 3
004030CE   .  FFD7          call    edi
004030D0   .  8BD0          mov     edx, eax
004030D2   .  8D4D DC       lea     ecx, dword ptr [ebp-24]
004030D5   .  FFD3          call    ebx
004030D7   .  0FBF7E 46     movsx   edi, word ptr [esi+46]
004030DB   .  83FF 15       cmp     edi, 15
004030DE   .  72 06         jb      short 004030E6
004030E0   .  FF15 7C104000 call    dword ptr [<&MSVBVM60.\_\_vbaGener>;  MSVBVM60.\_\_vbaGenerateBoundsError
004030E6   >  8D4D DC       lea     ecx, dword ptr [ebp-24]
004030E9   .  6A 01         push    1
004030EB   .  51            push    ecx
004030EC   .  FF15 FC104000 call    dword ptr [<&MSVBVM60.\_\_vbaI4Var>;  MSVBVM60.\_\_vbaI4Var
004030F2   .  8B55 D0       mov     edx, dword ptr [ebp-30]          ;  //指向注册码&空格前一位
004030F5   .  50            push    eax                              ;  //EAX=文件偏移位置
004030F6   .  03D7          add     edx, edi                         ;  //edi从1到20
004030F8   .  52            push    edx                              ;  //edx指向注册码的第N位
004030F9   .  6A 01         push    1                                ;  //每次Put一位
004030FB   .  FF15 2C104000 call    dword ptr [<&MSVBVM60.\_\_vbaPut4>>;  MSVBVM60.\_\_vbaPut4
00403101   .  66:8B4E 46    mov     cx, word ptr [esi+46]
00403105   .  B8 01000000   mov     eax, 1
0040310A   .  66:03C8       add     cx, ax
0040310D   .  0F80 32060000 jo      00403745
00403113   .  66:894E 46    mov     word ptr [esi+46], cx
00403117   .^ E9 44FFFFFF   jmp     00403060
```

后面还有一段是写机器码&空格的，略。

### 总结

注册码为机器码`*321+1234567890`再取前十位。程序计算完注册码之后，若注册正确，则将机器码和注册码写进`SuperPrint.ocx`尾部，机器码从`EEFC4`开始，后面接空格补齐20位，再接注册码，再接空格补齐20位。`ocx`加载的时候从自身读取保存的机器码和注册码进行比较。

--------------------------------------------------------------------------------

【版权声明】: 本文原创于看雪技术论坛, 转载请注明作者并保持文章的完整, 谢谢!

2009年07月13日 22:51:49

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

[#调试逆向](https://bbs.kanxue.com/forum-4-1-1.htm)
