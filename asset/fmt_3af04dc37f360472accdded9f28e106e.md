
调用 `00004385 : E8(00 00 00 00 CALL DelphiIsRunning{0x201}`

后有 `test al,al`

如果 `al=1` 的话，就表明 `delphi` 正在运行，那么我们就把这一句用 `mov al,1` 代替好了（机器码 `B001`）。

在 `DCU` 文件中这个 `call DelphiIsRunning` 是要由编译器来动态连接其地址的，所以没法 `nop` 掉这个调用。

在所有的 `dcu` 文件中用同样的方法查找并替换就可以了，当然 `test al,al` 后的下一条语句有的 `DCU` 文件和这里的不同，

没关系，只改 `test al,al`。

### 2. 还有一个问题，它的 `hint` 总是要出现，告诉人们您正在用的是 `SDL` 公司的未注册版本，整个世界都要为之改变呢。

改掉它！查找 `shareware`, `copyright`, `unregister` 之类的东西，查到与 `copyright` 有关的：

`function getCopyRightStr: System.AnsiString;`

`var`

` result Result: System.AnsiString;`

` AuxStr: System.AnsiString;`

` i: System.Integer;`

`begin`

```
00000000 : 55                        PUSH EBP
00000001 : 8B EC                     MOV EBP,ESP
00000003 : 6A 00                     PUSH $00
00000005 : 6A 00                     PUSH $00
00000007 : 53                        PUSH EBX
00000008 : 56                        PUSH ESI
00000009 : 57                        PUSH EDI
0000000A : 8B F8                     MOV EDI,EAX
0000000C : 33 C0                     XOR EAX,EAX
0000000E : 55                        PUSH EBP
0000000F : 68(72 00 00 00            PUSH getCopyRightStr{0x1F2}+$00000072
00000014 : 64 FF 30                  PUSH DWORD PTR FS:\[EAX\]
00000017 : 64 89 20                  MOV DWORD PTR FS:\[EAX\],ESP
0000001A : 8D 45 FC                  LEA EAX,DWORD PTR \[EBP-4\]
0000001D : E8(00 00 00 00            CALL @LStrClr{0xE2}
00000022 : BE 87 FF FF FF            MOV ESI,$FFFFFF87
00000027 : BB(78 00 00 00            MOV EBX,ShareW{0x1F1}+$00000078
0000002C : 8D 45 F8                  LEA EAX,DWORD PTR \[EBP-8\]
0000002F : 33 D2                     XOR EDX,EDX
00000031 : 8A 13                     MOV DL,BYTE PTR \[EBX\]
00000033 : 8A 92(00 00 00 00         MOV DL,BYTE PTR \[EDXCTDecode{0x1EF}\]
00000039 : E8(00 00 00 00            CALL @LStrFromChar{0xE3}
0000003E : 8B 55 F8                  MOV EDX,DWORD PTR \[EBP-8\]
00000041 : 8D 45 FC                  LEA EAX,DWORD PTR \[EBP-4\]
00000044 : E8(00 00 00 00            CALL @LStrCat{0xE4}----------这是在拼凑那个 hint 呢
00000049 : 4B                        DEC EBX
0000004A : 46                        INC ESI
0000004B : 75 DF                     JNE -33; (0x2C)--------------在这里改！跳到 call @LStrAsg{0xE5}后面的那句
0000004D : 8B C7                     MOV EAX,EDI
0000004F : 8B 55 FC                  MOV EDX,DWORD PTR \[EBP-4\]
00000052 : E8(00 00 00 00            CALL @LStrAsg{0xE5}---------拼完了就显示，那么跳过它！
00000057 : 33 C0                     XOR EAX,EAX
00000059 : 5A                        POP EDX
0000005A : 59                        POP ECX
0000005B : 59                        POP ECX
0000005C : 64 89 10                  MOV DWORD PTR FS:\[EAX\],EDX
0000005F : 68(79 00 00 00            PUSH getCopyRightStr{0x1F2}+$00000079
00000064 : 8D 45 F8                  LEA EAX,DWORD PTR \[EBP-8\]
00000067 : BA 02 00 00 00            MOV EDX,$00000002
0000006C : E8(00 00 00 00            CALL @LStrArrayClr{0xE6}
00000071 : C3                        RET NEAR
00000072 : E9(00 00 00 00            JMP @HandleFinally{0xE7}
00000077 : EB EB                     JMP -21; (0x64)
00000079 : 5F                        POP EDI
0000007A : 5E                        POP ESI
0000007B : 5B                        POP EBX
0000007C : 59                        POP ECX
0000007D : 59                        POP ECX
0000007E : 5D                        POP EBP
0000007F : C3                        RET NEAR
```

`end;`

数一数后知道 `75 DF` 改成 `EB 0A`

这样就没有 `hint` 了，如果您想要自己的 `hint`，请用别的办法吧，可以做得到的。
