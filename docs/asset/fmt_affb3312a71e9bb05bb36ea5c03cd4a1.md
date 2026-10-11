
0040135D ; \*\*\*\*\*\*\*\*\*\*\*\*\*\* S U B R O U T I N E \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

0040135D

0040135D ; Attributes: bp-based frame

0040135D

0040135D VJz                 proc near

0040135D

0040135D var\_RegEip      = dword ptr -14h

0040135D var\_RegDl       = byte ptr -10h

0040135D var\_7           = dword ptr -7

0040135D var\_ZFlag       = byte ptr -2

0040135D arg\_lpVMCode     = dword ptr  10h

0040135D

0040135D                 inc     [ebp+var\_RegEip]

00401360                 mov     eax, [ebp+var\_RegEip]

00401363                 movzx   eax, byte ptr [eax]

00401366                 mov     [ebp+var\_RegDl], al

00401369                 shl     eax, 8

0040136C                 mov     [ebp+var\_7], eax

0040136F                 inc     [ebp+var\_RegEip]

00401372                 mov     eax, [ebp+var\_RegEip]

00401375                 movzx   eax, byte ptr [eax]

00401378                 mov     [ebp+var\_RegDl], al

0040137B                 add     [ebp+var\_7], eax

0040137E                 mov     eax, [ebp+var\_7]

00401381                 cmp     [ebp+var\_ZFlag], 1  ; 标志位测试

00401385                 jnz     short loc\_40138D

00401387                 add     eax, [ebp+arg\_lpVMCode]

0040138A                 mov     [ebp+var\_RegEip], eax

0040138D

0040138D loc\_40138D:

0040138D                 jmp     InterpretingEntry

0040138D VJz             endp

12. VJnz，条件跳转指令，当 `var_ZFlag=0` 时触发，偏移量为操作码后面紧跟的一个 word（相对于伪代码区基址）

00401392 ; \*\*\*\*\*\*\*\*\*\*\*\*\*\* S U B R O U T I N E \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

00401392

00401392 ; Attributes: bp-based frame

00401392

00401392 VJnz            proc near

00401392

00401392 var\_RegEip      = dword ptr -14h

00401392 var\_RegDl       = byte ptr -10h

00401392 var\_7           = dword ptr -7

00401392 var\_ZFlag       = byte ptr -2

00401392 arg\_lpVMCode     = dword ptr  10h

00401392

00401392                 inc     [ebp+var\_RegEip]

00401395                 mov     eax, [ebp+var\_RegEip]

00401398                 movzx   eax, byte ptr [eax]

0040139B                 mov     [ebp+var\_RegDl], al

0040139E                 shl     eax, 8

004013A1                 mov     [ebp+var\_7], eax

004013A4                 inc     [ebp+var\_RegEip]

004013A7                 mov     eax, [ebp+var\_RegEip]

004013AA                 movzx   eax, byte ptr [eax]

004013AD                 mov     [ebp+var\_RegDl], al

004013B0                 add     [ebp+var\_7], eax

004013B3                 mov     eax, [ebp+var\_7]

004013B6                 cmp     [ebp+var\_ZFlag], 0  ; 标志位测试

004013BA                 jnz     short loc\_4013C2

004013BC                 add     eax, [ebp+arg\_lpVMCode]

004013BF                 mov     [ebp+var\_RegEip], eax

004013C2

004013C2 loc\_4013C2:

004013C2                 jmp     InterpretingEntry

004013C2 VJnz            endp

13. VJs，条件跳转指令，当 `var_SFlag=1` 时触发，偏移量为操作码后面紧跟的一个 word（相对于伪代码区基址）

004013C7 ; \*\*\*\*\*\*\*\*\*\*\*\*\*\* S U B R O U T I N E \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

004013C7

004013C7 ; Attributes: bp-based frame

004013C7

004013C7 VJs             proc near

004013C7

004013C7 var\_RegEip      = dword ptr -14h

004013C7 var\_RegDl       = byte ptr -10h

004013C7 var\_7           = dword ptr -7

004013C7 var\_SFlag       = byte ptr -3

004013C7 arg\_lpVMCode     = dword ptr  10h

004013C7

004013C7                 inc     [ebp+var\_RegEip]

004013CA                 mov     eax, [ebp+var\_RegEip]

004013CD                 movzx   eax, byte ptr [eax]

004013D0                 mov     [ebp+var\_RegDl], al

004013D3                 shl     eax, 8

004013D6                 mov     [ebp+var\_7], eax

004013D9                 inc     [ebp+var\_RegEip]

004013DC                 mov     eax, [ebp+var\_RegEip]

004013DF                 movzx   eax, byte ptr [eax]

004013E2                 mov     [ebp+var\_RegDl], al

004013E5                 add     [ebp+var\_7], eax

004013E8                 mov     eax, [ebp+var\_7]

004013EB                 cmp     [ebp+var\_SFlag], 1  ; 标志位测试

004013EF                 jnz     short loc\_4013F7

004013F1                 add     eax, [ebp+arg\_lpVMCode]

004013F4                 mov     [ebp+var\_RegEip], eax

004013F7

004013F7 loc\_4013F7:

004013F7                 jmp     InterpretingEntry

004013F7 VJs             endp

6. VRet10，返回指令，从虚拟机解释引擎中返回

004013FC ; \*\*\*\*\*\*\*\*\*\*\*\*\*\* S U B R O U T I N E \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

004013FC

004013FC

004013FC VRet10          proc near

004013FC                 leave

004013FD                 retn    10h

004013FD VRet10          endp ; sp =  4

004013FD

在这套解释系统中，比较值得注意的是它的堆栈操作与 x86 下有所不同：

入栈 —— `add esp, 4`
`mov [esp], xxx`

出栈 —— `sub esp, 4`
`mov xxxx, [esp]`

看起来有些不可思议，不过由于该引擎同时用到了 `var_RegEax` 来传递数据，所以没有出现问题。

以上面的分析为基础，我们来反编译一下 004020CF 处的伪指令组——对序列号的处理：

004020BF lpVMData        dd 0
004020C3                dd 0
004020C7                dd 1
004020CB ; UINT uType
004020CB uType           dd 10h                 ;
004020CF lpVMCode\_4020CF db    0 ;               ; VNop
004020D0                db 1, 30h               ; VPushImm32 30h
004020D2                db  0Eh ;               ; VPushByteVArg
004020D3                db    5 ;               ; VSub
004020D4                db 2 dup(2)             ; VLoadMem32 004020C7
004020D6                db    8 ;               ; VMul
004020D7                db 2, 1                ; VLoadMem32 004020C3
004020D9                db    4 ;               ; VAdd
004020DA                db 3, 1                ; VSaveMem32 004020C3
004020DC                db 2 dup(2)            ; VLoadMem32 004020C7
004020DE                db 1, 0Ah              ; VPushImm32 0Ah
004020E0                db    8 ;               ; VMul
004020E1                db 3, 2                ; VSaveMem32 004020C7
004020E3                db 1, 30h              ; VPushImm32 30h
004020E5                db  0Eh ;               ; VPushByteVArg
004020E6                db    7 ;               ; VPushRegEax
004020E7                db 1, 0                ; VPushImm32 0
004020E9                db    5 ;               ; VSub
004020EA                db  0Dh ;               ; VPop2RegEax
004020EB                db 13h, 0, 3           ; VJs 004020D2
004020EE                db  0Dh ;               ; VPop2RegEax
004020EF                db 1, 0EBh             ; VPushImm32 0EBh
004020F1                db 2, 1                ; VLoadMem32 004020C3
004020F3                db    5 ;               ; VSub
004020F4                db 3, 1                ; VSaveMem32 004020C3
004020F6                db    6 ;               ; VRet10

这段伪代码的作用是把输入的序列号字符串转换为十进制数，然后减去 0EBh，保存至 004020C3

其他部分的虚拟机代码也可以做类似的分析，这里留给读者自己思考:D

cyclotron
2007.3.10

`4dd466d0e1e0cc123afbe504da2462ca`
