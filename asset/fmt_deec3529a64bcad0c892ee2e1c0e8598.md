
```
00402795      F7D7                    NOT EDI                       ;  unpackme.00401952  
00402797      C1C7 10               ROL EDI, 10  
0040279A      83E8 06               SUB EAX, 6  
0040279D      C700 FF150000         MOV DWORD PTR [EAX], 15FF       //写入 CALL NEAR DWORD PTR  
004027A3      8978 02               MOV DWORD PTR [EAX+2], EDI      //写入地址  
004027A6      83C6 08               ADD ESI, 8  
004027A9    ^ EB C7                JMP SHORT unpackme.00402772  
004027AB      90                    NOP  
004027AC      90                    NOP  
004027AD      90                    NOP  
004027AE      8B7E 04               MOV EDI, DWORD PTR [ESI+4]      //是 jmp 就跳到这里来  
004027B1      2BF8                  SUB EDI, EAX                    ;  unpackme.00401950  
004027B3      F7D7                  NOT EDI                         ;  unpackme.00401952  
004027B5      C1C7 10               ROL EDI, 10  
004027B8      83E8 06               SUB EAX, 6  
004027BB      C700 FF250000         MOV DWORD PTR [EAX], 25FF       //写入 JMP NEAR DWORD PTR  
004027C1      8978 02               MOV DWORD PTR [EAX+2], EDI      //写入地址  
004027C4      83C6 08               ADD ESI, 8  
004027C7    ^ EB A9                JMP SHORT unpackme.00402772  
004027C9      61                    POPAD  
004027CA      90                    NOP  
```

```
60 BE F3 3E 38 00 8B 06 83 3E 00 74 51 90 90 90 90 33 D2 B9 02 00 00 00 F7 E1 D1 E8 08 D2 75 22  
90 90 90 90 8B 7E 04 2B F8 F7 D7 C1 C7 10 83 E8 06 C7 00 FF 15 00 00 89 78 02 83 C6 08 EB C7 90  
90 90 8B 7E 04 2B F8 F7 D7 C1 C7 10 83 E8 06 C7 00 FF 25 00 00 89 78 02 83 C6 08 EB A9 61 90 00  
```

重新载入程序，跟到`0038383F`时(执行被偷的字节的地址)，找到`0040276C`写入上面的代码，转到`0040276C`执行还原代码后，跳到刚才取得的原函数的`OEP`

把

```
00401950    FFD7                  CALL NEAR EDI                     ; unpackme.00401952  
00401952    58                    POP EAX                           ; 0012FFF0  
```

恢复为：

```
00401950 >/$  55                  PUSH EBP  
00401951  |.  8BEC                MOV EBP, ESP  
```

在把运行`OEP`转到`00401950`，`dump it`，用`imprREC`：

填入

`OEP＝00001950`

`RAV＝2000`，大小＝`2000`   （`RAV`可以随便找程序里的`CALL`看看就知道了）

`getimport`，`level3 trace`，无效的`cut`，`fixdump`，`ok`

Special thx:

`eaton`,`ama`,`catking668`,[www.pediy.com](http://www.pediy.com),`www.crackslatinos.hispadominio.net`,`luocong's scc and you!`

Ps:但要记住，虽然我没提到你，我还是记得你的，感谢着你的。。。

`4dd466d0e1e0cc123afbe504da2462ca`
