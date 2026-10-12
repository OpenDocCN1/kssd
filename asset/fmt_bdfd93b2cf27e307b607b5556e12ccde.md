
以下内容为汇编代码反汇编片段，展示了解压、解码及输入表（IAT）处理过程：

```asm
003D0757    8D85 00104000       LEA EAX,DWORD PTR SS:[EBP+401000]        ; EP 入 EAX
003D075D    B9 85130000         MOV ECX,1385                             ; 要解压的大小
003D0762    8D95 B41B4000       LEA EDX,DWORD PTR SS:[EBP+401BB4]        ; 下一段要解压的起始地址 438bb4
003D0768    FFD2                CALL EDX                                   ; 这个 CALL 就是计算 CRC 的值
...
003D080F    8DBD 31254000       LEA EDI,DWORD PTR SS:[EBP+402531]        ; 这里又开始解压下一面的代码
003D0815    03BD B5234000       ADD EDI,DWORD PTR SS:[EBP+4023B5]
003D081B    33D2                XOR EDX,EDX
003D081D    B9 00010000         MOV ECX,100
003D0822    F7F1                DIV ECX
003D0824    8BDA                MOV EBX,EDX
003D0826    F7F1                DIV ECX
003D0828    03DA                ADD EBX,EDX
003D082A    F7F1                DIV ECX
003D082C    03DA                ADD EBX,EDX
003D082E    F7F1                DIV ECX
003D0830    03DA                ADD EBX,EDX
003D0832    59                  POP ECX
003D0833    8BC3                MOV EAX,EBX
003D0835    8B8D B9234000       MOV ECX,DWORD PTR SS:[EBP+4023B9]
003D083B    3007                XOR BYTE PTR DS:[EDI],AL                 ; 这次从 4399e8 处开始解码
003D083D    47                  INC EDI
003D083E    49                  DEC ECX
003D083F    75 FA               JNZ SHORT 003D083B                       ; 没解压完跳回去继续
...
003D08E6    E8 B70A0000         CALL 003D13A2                              ; 又清除断点
003D08EB    8B85 A9234000       MOV EAX,DWORD PTR SS:[EBP+4023A9]        ; 把上面申请的地址空间 920000 入 EAX
003D08F1    8BF0                MOV ESI,EAX
003D08F3    50                  PUSH EAX
003D08F4    8D9D 31254000       LEA EBX,DWORD PTR SS:[EBP+402531]
003D08FA    039D B5234000       ADD EBX,DWORD PTR SS:[EBP+4023B5]        ; 004399E8
...
003D09A5    53                  PUSH EBX
003D09A6    8D95 08114000       LEA EDX,DWORD PTR SS:[EBP+401108]
003D09AC    FFD2                CALL EDX                                   ; ApLib 解码
003D09AE    83C4 08             ADD ESP,8
003D09B1    8BBD A9234000       MOV EDI,DWORD PTR SS:[EBP+4023A9]        ; 申请的空间 920000 入 EDI
003D09B7    803F C3             CMP BYTE PTR DS:[EDI],0C3                ; 判断是否开始处理相关的 DLL(也就是就是否第一次处理相应的 DLL 中的 API)
003D09BA    0F85 CC000000       JNZ 003D0A8C                               ; 如果不是则跳
003D09C0    83C7 02             ADD EDI,2
003D09C3    57                  PUSH EDI                                   ; /Push "User32.dll"
003D09C4    FF95 C8244000       CALL DWORD PTR SS:[EBP+4024C8]            ; \GetModuleHandleA 判断 DLL 有没有载入
003D09CA    85C0                TEST EAX,EAX
003D09CC    0F85 AC000000       JNZ 003D0A7E                               ; 如果已经加载则跳
...
003D0A77    57                  PUSH EDI                                   ; 如果没有载入则加载 user32.dll
003D0A78    FF95 CC244000       CALL DWORD PTR SS:[EBP+4024CC]            ; LoadLibraryA
003D0A7E    8985 7C244000       MOV DWORD PTR SS:[EBP+40247C],EAX        ; hModule(77D1000.USER32.DLL)入[EBP+40247C]处
003D0A84    33DB                XOR EBX,EBX
003D0A86    8A5F FF             MOV BL,BYTE PTR DS:[EDI-1]
003D0A89    03FB                ADD EDI,EBX
003D0A8B    47                  INC EDI
003D0A8C    803F C4             CMP BYTE PTR DS:[EDI],0C4                ; 对输入表的几种不同情况进行处理??
003D0A8F    0F84 98010000       JE 003D0C2D                                ; 第一次这里没有跳,主程序没有跳。
003D0A95    47                  INC EDI
003D0A96    57                  PUSH EDI                                   ; /PUSH "TranslateMessage"
003D0A97    FFB5 7C244000       PUSH DWORD PTR SS:[EBP+40247C]            ; |hModule = 77D10000 (USER32)
003D0A9D    FF95 C4244000       CALL DWORD PTR SS:[EBP+4024C4]            ; \GetProcAddress
...
003D0B4A    8A5F FF             MOV BL,BYTE PTR DS:[EDI-1]               ; 加地址
003D0B4D    03FB                ADD EDI,EBX
003D0B4F    47                  INC EDI
003D0B50    8A1F                MOV BL,BYTE PTR DS:[EDI]
003D0B52    47                  INC EDI
003D0B53    E9 B5000000         JMP 003D0C0D
...
003D0C0D    80FB 00             CMP BL,0                                   ; 如果 BL 大于 0 就跳
003D0C10    0F87 42FFFFFF       JA 003D0B58
...
003D0B58    50                  PUSH EAX                                   ; PUSH API(77D18bce)
003D0B59    8B0F                MOV ECX,DWORD PTR DS:[EDI]               ; 把要写入 iat 的地址入 ECX
003D0B5B    E8 62090000         CALL 003D14C2                              ; 这看到这里我就猜它里面应该填充 API，进去看看
003D14C2    60                  PUSHAD                                     ; 进入后到这里
003D14C3    8BF0                MOV ESI,EAX                                ; API 地址入 esi
003D14C5    8B85 A5234000       MOV EAX,DWORD PTR SS:[EBP+4023A5]        ; EAX=003E0000
...
003D1570    0385 B1234000       ADD EAX,DWORD PTR SS:[EBP+4023B1]        ; [EBP+4023B1]处保存放后的值，比如第一没写就是 0，第二的话就是 4 了
003D1576    894424 1C           MOV DWORD PTR SS:[ESP+1C],EAX            ; 这里的用意就是把 API 的部分放到申请的空间里，真正的 IAT 调用申请的地址
003D157A    E8 00000000         CALL 003D157F
003D157F    5B                  POP EBX
003D1580    81C3 61020000       ADD EBX,261                                ; 3d157F+261=3D17E0 处临时保存 API 的地址
003D1586    8933                MOV DWORD PTR DS:[EBX],ESI                 ; API 临时保存到 3D17E0 处
003D1588    8BF8                MOV EDI,EAX                                ; EDI=3E0000
...
003D162F    8BF3                MOV ESI,EBX                                ; 再把 API 存放的地址放到 ESI 中
003D1631    81EE 38000000       SUB ESI,38
003D1637    B9 4A000000         MOV ECX,4A
003D163C    FC                  CLD
003D163D    F3:A4               REP MOVS BYTE PTR ES:[EDI],BYTE PTR DS:[ESI] ; 这里把作者自己搞的花指令先填在我们去 API 的代码前
003D163F    8B85 A5234000       MOV EAX,DWORD PTR SS:[EBP+4023A5]
003D1645    0385 B1234000       ADD EAX,DWORD PTR SS:[EBP+4023B1]          ; EAX=3E0000
...
003D16F0    8BD8                MOV EBX,EAX                                ; 再把 003E0000 的地址放到 ebx 上去
003D16F2    83C3 04             ADD EBX,4
003D179A    8918                MOV DWORD PTR DS:[EAX],EBX
003D179C    8185 B1234000 4A00>ADD DWORD PTR SS:[EBP+4023B1],4A            ; 这也就是说代码指定大小为 4A
003D17A6    61                  POPAD
003D17A7    C3                  RETN                                         ; 返回上去
...
003D0C05    8901                MOV DWORD PTR DS:[ECX],EAX                   ; 把加密后的地址放到 IAT 中,这个值是壳计算生成的.
003D0C07    83C7 04             ADD EDI,4                                    ; 继续指向下一个要处理的 API
003D0C0A    FECB                DEC BL
003D0C0C    58                  POP EAX
003D0C0D    80FB 00             CMP BL,0                                     ; 如果 BL 大于 0 就跳
003D0C10    0F87 42FFFFFF       JA 003D0B58
003D0C16    803F C3             CMP BYTE PTR DS:[EDI],0C3                    ; 到这里跟过就会发现壳是通过两个 C3 来判断有没有处理完当前的 DLL
003D0C19    0F84 A1FDFFFF       JE 003D09C0                                  ; 判断当前 DLL 是否已经处理完,处理则跳去下一个 DLL 的 api 处理
003D0C1F    833F 00             CMP DWORD PTR DS:[EDI],0
003D0C22    0F84 DE000000       JE 003D0D06                                  ; 判断有没有处理完全部的 API，如果处理完就跳
003D0C28    E9 8AFDFFFF         JMP 003D09B7                                 ; 没处理完则回去继续
```

好了，处理完输入表，到这里：

```asm
003D0D06    E8 97060000         CALL 003D13A2                              ; 处理完 IAT 就到这里,清除断点，不用跟进
003D0D0B    8D9D 31254000       LEA EBX,DWORD PTR SS:[EBP+402531]          ; EBX=439531
...
003D0DB6    039D B5234000       ADD EBX,DWORD PTR SS:[EBP+4023B5]
003D0DBC    039D B9234000       ADD EBX,DWORD PTR SS:[EBP+4023B9]          ; 00439C6D
...
003D0E67    8B8D 95234000       MOV ECX,DWORD PTR SS:[EBP+402395]          ; ECX=A1E
...
```
