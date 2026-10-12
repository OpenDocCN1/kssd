
# 呼吸动画秀 2.7 

> 原文：[`www.pediy.com/kssd/pediy06/pediy6275.htm`](https://www.pediy.com/kssd/pediy06/pediy6275.htm)

我看了一下呼吸动画秀 2.7   

`peid` 报告使用了 `blowfish` 算法，心里还挺高兴，才看过，正好练练算法，可是最终程序没有使用，不知为何，莫非是留待以后使用还是用了什么需要注册的控件。   

里面的预注册号码没有用，是随机生成的，保存在目录下   

`regokgif.pin` 文件里。   

注册按钮按下响应：   

```
CODE:004EEAD8 push ebp
CODE:004EEAD9 mov ebp, esp
CODE:004EEADB mov ecx, 7
CODE:004EEAE0
CODE:004EEAE0 loc_4EEAE0: ; CODE XREF: CODE:004EEAE5j
CODE:004EEAE0 push 0
CODE:004EEAE2 push 0
CODE:004EEAE4 dec ecx
CODE:004EEAE5 jnz short loc_4EEAE0
CODE:004EEAE7 push ebx
CODE:004EEAE8 mov ebx, eax
CODE:004EEAEA xor eax, eax
CODE:004EEAEC push ebp
CODE:004EEAED push offset loc_4EEDF8
CODE:004EEAF2 push dword ptr fs:[eax]
CODE:004EEAF5 mov fs:[eax], esp
CODE:004EEAF8 lea edx, [ebp-0Ch]
CODE:004EEAFB mov eax, [ebx+300h] ; 读取输入的注册码
CODE:004EEB01 call @TControl@GetText$qqrv ; TControl::GetText(void)
CODE:004EEB06 mov eax, [ebp-0Ch]
CODE:004EEB09 call @System@_16823 ; System::_16823
CODE:004EEB0E cmp eax, 14h
CODE:004EEB11 jnz loc_4EED91
CODE:004EEB17 lea edx, [ebp-10h]
CODE:004EEB1A mov eax, [ebx+2F8h] ; 读取用户名
CODE:004EEB20 call @TControl@GetText$qqrv ; TControl::GetText(void)
CODE:004EEB25 mov eax, [ebp-10h]
CODE:004EEB28 call @System@_16823 ; System::_16823
CODE:004EEB2D test eax, eax
CODE:004EEB2F jle loc_4EED91
CODE:004EEB35 lea eax, [ebp-4]
CODE:004EEB38 push eax
CODE:004EEB39 lea edx, [ebp-18h]
CODE:004EEB3C mov eax, [ebx+2F8h]
CODE:004EEB42 call @TControl@GetText$qqrv ; TControl::GetText(void)
CODE:004EEB47 mov eax, [ebp-18h]
CODE:004EEB4A lea edx, [ebp-14h]
CODE:004EEB4D call @Trim
CODE:004EEB52 lea eax, [ebp-14h]
CODE:004EEB55 mov edx, offset dword_4EEE0C
CODE:004EEB5A call @System@@LStrCat$qqrv ; System::__linkproc__ LStrCat(void)
CODE:004EEB5F mov eax, [ebp-14h]
CODE:004EEB62 mov ecx, 8
CODE:004EEB67 mov edx, 1
CODE:004EEB6C call @System@@LStrCopy$qqrv ; System::__linkproc__ LStrCopy(void)
CODE:004EEB71 lea eax, [ebp-8]
CODE:004EEB74 push eax
CODE:004EEB75 lea edx, [ebp-1Ch]
CODE:004EEB78 mov eax, [ebx+300h]
CODE:004EEB7E call @TControl@GetText$qqrv ; TControl::GetText(void)
CODE:004EEB83 mov eax, [ebp-1Ch]
CODE:004EEB86 push eax
CODE:004EEB87 mov eax, ds:dword_509F8C
CODE:004EEB8C mov eax, [eax]
CODE:004EEB8E mov ecx, [eax+708h]
CODE:004EEB94 lea eax, [ebp-20h]
CODE:004EEB97 mov edx, [ebp-4]
CODE:004EEB9A call @System@@LStrCat3$qqrv ; System::__linkproc__ LStrCat3(void)
CODE:004EEB9F mov edx, [ebp-20h] ; 用户名+空格 +"8806" 补足 12 位
CODE:004EEBA2 xor ecx, ecx
CODE:004EEBA4 pop eax ; 用户输入的注册码
CODE:004EEBA5 call sub_4B1DC0 ; 关键 CALL，对输入用户名及注册码进行变换
CODE:004EEBAA mov eax, ds:dword_509F8C
CODE:004EEBAF mov eax, [eax]
CODE:004EEBB1 add eax, 72Ch
CODE:004EEBB6 mov edx, [ebp-8]
CODE:004EEBB9 call @System@@LStrAsg$qqrv ; System::__linkproc__ LStrAsg(void)
CODE:004EEBBE lea eax, [ebp-24h]
CODE:004EEBC1 push eax
CODE:004EEBC2 mov ecx, 6
CODE:004EEBC7 mov edx, 1
CODE:004EEBCC mov eax, [ebp-8]
CODE:004EEBCF call @System@@LStrCopy$qqrv ; System::__linkproc__ LStrCopy(void)
CODE:004EEBD4 mov eax, [ebp-24h]
CODE:004EEBD7 mov edx, ds:dword_509F8C
CODE:004EEBDD mov edx, [edx]
CODE:004EEBDF mov edx, [edx+70Ch]
CODE:004EEBE5 call @System@@LStrCmp$qqrv ; System::__linkproc__ LStrCmp(void) // 前 6 字节为 machinecode_1 为程序根据 asm 指令 cpuid 经过运算得来
CODE:004EEBEA jnz loc_4EED91
CODE:004EEBF0 lea eax, [ebp-28h]
CODE:004EEBF3 push eax
CODE:004EEBF4 mov ecx, 3
CODE:004EEBF9 mov edx, 7
CODE:004EEBFE mov eax, [ebp-8]
CODE:004EEC01 call @System@@LStrCopy$qqrv ; System::__linkproc__ LStrCopy(void)
CODE:004EEC06 mov eax, [ebp-28h]
CODE:004EEC09 mov edx, offset dword_4EEE20
CODE:004EEC0E call @System@@LStrCmp$qqrv ; System::__linkproc__ LStrCmp(void) // 后 3 字节为"gif",程序固定值
CODE:004EEC13 jnz loc_4EED91
...
// 关键 CALL sub_004B1DC0
CODE:004B1DC0 push ebp
CODE:004B1DC1 mov ebp, esp
CODE:004B1DC3 add esp, 0FFFFFFC4h
CODE:004B1DC6 push ebx
CODE:004B1DC7 push esi
CODE:004B1DC8 push edi
CODE:004B1DC9 xor ebx, ebx
CODE:004B1DCB mov [ebp+var_3C], ebx
CODE:004B1DCE mov [ebp+var_34], ebx
CODE:004B1DD1 mov [ebp+var_38], ebx
CODE:004B1DD4 mov [ebp+var_2C], ebx
CODE:004B1DD7 mov [ebp+var_30], ebx
CODE:004B1DDA mov [ebp+var_28], ebx
CODE:004B1DDD mov [ebp+var_10], ebx
CODE:004B1DE0 mov ebx, ecx
CODE:004B1DE2 mov [ebp+var_8], edx
CODE:004B1DE5 mov [ebp+var_4], eax
CODE:004B1DE8 mov eax, [ebp+var_4]
CODE:004B1DEB call @System@@LStrAddRef$qqrv ; System::__linkproc__ LStrAddRef(void)
CODE:004B1DF0 mov eax, [ebp+var_8]
CODE:004B1DF3 call @System@@LStrAddRef$qqrv ; System::__linkproc__ LStrAddRef(void)
CODE:004B1DF8 xor eax, eax
CODE:004B1DFA push ebp
CODE:004B1DFB push offset loc_4B1FFF
CODE:004B1E00 push dword ptr fs:[eax]
CODE:004B1E03 mov fs:[eax], esp
CODE:004B1E06 xor eax, eax
CODE:004B1E08 push ebp
CODE:004B1E09 push offset loc_4B1FBD
CODE:004B1E0E push dword ptr fs:[eax]
CODE:004B1E11 mov fs:[eax], esp
CODE:004B1E14 mov eax, [ebp+var_8]
```



```asm
CODE:004B1E17                 call    @System@_16823  ; System::_16823   

CODE:004B1E1C                 mov     [ebp+var_C], eax   

CODE:004B1E1F                 cmp     [ebp+var_C], 0   

CODE:004B1E23                 jnz     short loc_4B1E32   

CODE:004B1E25                 lea     eax, [ebp+var_8]   

CODE:004B1E28                 mov     edx, offset dword_4B2018   

CODE:004B1E2D                 call    @System@@LStrLAsg$qqrv ; System::__linkproc__ LStrLAsg(void)   

CODE:004B1E32                 

CODE:004B1E32 loc_4B1E32:                      ; CODE XREF: sub_4B1DC0+63j   

CODE:004B1E32                 xor     esi, esi   

CODE:004B1E34                 mov     edi, 100h   

CODE:004B1E39                 test    bl, bl   

CODE:004B1E3B                 jz      loc_4B1EE5       //若为 1 是生成序列号    
                                                        //  为 0 是变换用户名及用户输入的注册码    

CODE:004B1E41                 call    @System@Randomize$qqrv ; System::Randomize(void)   

CODE:004B1E46                 mov     eax, edi          // 序列号第一字节随机生成    

CODE:004B1E48                 call    @System@@RandInt$qqrv ; System::__linkproc__ RandInt(void)   

CODE:004B1E4D                 mov     edi, eax   

CODE:004B1E4F                 lea     eax, [ebp+var_10]   

CODE:004B1E52                 push    eax   

CODE:004B1E53                 mov     [ebp+var_24], edi   

CODE:004B1E56                 mov     [ebp+var_20], 0   

CODE:004B1E5A                 lea     edx, [ebp+var_24]   

CODE:004B1E5D                 xor     ecx, ecx   

CODE:004B1E5F                 mov     eax, offset dword_4B2030   

CODE:004B1E64                 call    @Sysutils@Format$qqrx17System@AnsiStringpx14System@TVarRecxi ; Sysutils::Format(System::AnsiString,System::TVarRec *,int)   

CODE:004B1E69                 mov     eax, [ebp+var_4]   

CODE:004B1E6C                 call    @System@_16823  ; System::_16823   

CODE:004B1E71                 test    eax, eax   

CODE:004B1E73                 jle     loc_4B1FA8   

CODE:004B1E79                 mov     [ebp+var_1C], eax   

CODE:004B1E7C                 mov     [ebp+var_14], 1   

CODE:004B1E83                 

CODE:004B1E83 loc_4B1E83:                      ; CODE XREF: sub_4B1DC0+11Ej   

CODE:004B1E83                 mov     eax, [ebp+var_4]   

CODE:004B1E86                 mov     edx, [ebp+var_14]   

CODE:004B1E89                 movzx   eax, byte ptr [eax+edx-1]   

CODE:004B1E8E                 add     eax, edi   

CODE:004B1E90                 mov     ecx, 0FFh   

CODE:004B1E95                 cdq   

CODE:004B1E96                 idiv    ecx   

CODE:004B1E98                 mov     ebx, edx      // 序列号生成简单算法    
                                                        //  (序列号[n] + MachinCode_1[n])%0xFF = 序列号[n+1]    

CODE:004B1E9A                 cmp     esi, [ebp+var_C]   

CODE:004B1E9D                 jge     short loc_4B1EA2   

CODE:004B1E9F                 inc     esi   

CODE:004B1EA0                 jmp     short loc_4B1EA7   

CODE:004B1EA2 ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?   

CODE:004B1EA2                 

CODE:004B1EA2 loc_4B1EA2:                      ; CODE XREF: sub_4B1DC0+DDj   

CODE:004B1EA2                 mov     esi, 1   

CODE:004B1EA7                 

CODE:004B1EA7 loc_4B1EA7:                      ; CODE XREF: sub_4B1DC0+E0j   

CODE:004B1EA7                 mov     eax, [ebp+var_8]   

CODE:004B1EAA                 movzx   eax, byte ptr [eax+esi-1]   

CODE:004B1EAF                 xor     ebx, eax        //  序列号[n＋1] = 序列号[n＋1] ^ 种子字符串"http://www.okayle.com"[n]    

CODE:004B1EB1                 lea     eax, [ebp+var_28]   

CODE:004B1EB4                 push    eax   

CODE:004B1EB5                 mov     [ebp+var_24], ebx   

CODE:004B1EB8                 mov     [ebp+var_20], 0   

CODE:004B1EBC                 lea     edx, [ebp+var_24]   

CODE:004B1EBF                 xor     ecx, ecx   

CODE:004B1EC1                 mov     eax, offset dword_4B2030   

CODE:004B1EC6                 call    @Sysutils@Format$qqrx17System@AnsiStringpx14System@TVarRecxi ; Sysutils::Format(System::AnsiString,System::TVarRec *,int)   

CODE:004B1ECB                 mov     edx, [ebp+var_28]   

CODE:004B1ECE                 lea     eax, [ebp+var_10]   

CODE:004B1ED1                 call    @System@@LStrCat$qqrv ; System::__linkproc__ LStrCat(void)   

CODE:004B1ED6                 mov     edi, ebx   

CODE:004B1ED8                 inc     [ebp+var_14]   

CODE:004B1EDB                 dec     [ebp+var_1C]   

CODE:004B1EDE                 jnz     short loc_4B1E83   

CODE:004B1EE0                 jmp     loc_4B1FA8   

CODE:004B1EE5 ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?   

CODE:004B1EE5                 

CODE:004B1EE5 loc_4B1EE5:                      ; CODE XREF: sub_4B1DC0+7Bj   

CODE:004B1EE5                 lea     eax, [ebp+var_30]   

CODE:004B1EE8                 push    eax   

CODE:004B1EE9                 mov     ecx, 2   

CODE:004B1EEE                 mov     edx, 1   

CODE:004B1EF3                 mov     eax, [ebp+var_4]   

CODE:004B1EF6                 call    @System@@LStrCopy$qqrv ; System::__linkproc__ LStrCopy(void)   

CODE:004B1EFB                 mov     ecx, [ebp+var_30]   

CODE:004B1EFE                 lea     eax, [ebp+var_2C]   

CODE:004B1F01                 mov     edx, offset dword_4B2040 ; "$"   

CODE:004B1F06                 call    @System@@LStrCat3$qqrv ; System::__linkproc__ LStrCat3(void)   

CODE:004B1F0B                 mov     eax, [ebp+var_2C] ; 取前 2 字节的字符串化为 16 进制，如"6F"-->6Fh   

CODE:004B1F0B                                                        ;   

CODE:004B1F0E                 call    @Sysutils@StrToInt$qqrx17System@AnsiString ; Sysutils::StrToInt(System::AnsiString)   

CODE:004B1F13                 mov     edi, eax   

CODE:004B1F15                 mov     [ebp+var_14], 3   

CODE:004B1F1C                 

CODE:004B1F1C loc_4B1F1C:                      ; CODE XREF: sub_4B1DC0+1E2j   

CODE:004B1F1C                 lea     eax, [ebp+var_38]   

CODE:004B1F1F                 push    eax   

CODE:004B1F20                 mov     ecx, 2   

CODE:004B1F25                 mov     edx, [ebp+var_14]   

CODE:004B1F28                 mov     eax, [ebp+var_4]   

CODE:004B1F2B                 call    @System@@LStrCopy$qqrv ; System::__linkproc__ LStrCopy(void)   

CODE:004B1F30                 mov     ecx, [ebp+var_38] ; 存放后两字节   

CODE:004B1F33                 lea     eax, [ebp+var_34]   

CODE:004B1F36                 mov     edx, offset dword_4B2040 ; "$"   

CODE:004B1F3B                 call    @System@@LStrCat3$qqrv ; System::__linkproc__ LStrCat3(void)   

CODE:004B1F40                 mov     eax, [ebp+var_34] ; 后两字节-->HEX   

CODE:004B1F43                 call    @Sysutils@StrToInt$qqrx17System@AnsiString ; Sysutils::StrToInt(System::AnsiString)   

CODE:004B1F48                 mov     ebx, eax   

CODE:004B1F4A                 cmp     esi, [ebp+var_C]   

CODE:004B1F4D                 jge     short loc_4B1F52   

CODE:004B1F4F                 inc     esi   

CODE:004B1F50                 jmp     short loc_4B1F57   

CODE:004B1F52 ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?   

CODE:004B1F52                 

CODE:004B1F52 loc_4B1F52:                      ; CODE XREF: sub_4B1DC0+18Dj   

CODE:004B1F52                 mov     esi, 1   

CODE:004B1F57                 

CODE:004B1F57 loc_4B1F57:                      ; CODE XREF: sub_4B1DC0+190j   

CODE:004B1F57                 mov     eax, [ebp+var_8]   

CODE:004B1F5A                 movzx   eax, byte ptr [eax+esi-1] ; 取用户名字符串中 1 字节   

CODE:004B1F5F                 xor     eax, ebx        ; 简单运算   

CODE:004B1F61                 mov     [ebp+var_18], eax   

CODE:004B1F64                 cmp     edi, [ebp+var_18] ; 运算过程类似于：
```



## 汇编代码片段

```
CODE:004B1F64                     ; ABS(RegCode[n]^UserName[n-1] - RegCode[n-1]) = Out[n]

CODE:004B1F67             jl      short loc_4B1F78

CODE:004B1F69             mov     eax, [ebp+var_18]

CODE:004B1F6C             add     eax, 0FFh

CODE:004B1F71             sub     eax, edi

CODE:004B1F73             mov     [ebp+var_18], eax

CODE:004B1F76             jmp     short loc_4B1F7B

CODE:004B1F78 ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?

CODE:004B1F78 

CODE:004B1F78 loc_4B1F78:                     ; CODE XREF: sub_4B1DC0+1A7j

CODE:004B1F78             sub     [ebp+var_18], edi

CODE:004B1F7B 

CODE:004B1F7B loc_4B1F7B:                     ; CODE XREF: sub_4B1DC0+1B6j

CODE:004B1F7B             lea     eax, [ebp+var_3C]

CODE:004B1F7E             mov     edx, [ebp+var_18]

CODE:004B1F81             call    unknown_libname_25

CODE:004B1F86             mov     edx, [ebp+var_3C]

CODE:004B1F89             lea     eax, [ebp+var_10]

CODE:004B1F8C             call    @System@@LStrCat$qqrv ; System::__linkproc__ LStrCat(void)

CODE:004B1F91             mov     edi, ebx

CODE:004B1F93             add     [ebp+var_14], 2

CODE:004B1F97             mov     eax, [ebp+var_4]

CODE:004B1F9A             call    @System@_16823  ; System::_16823

CODE:004B1F9F             cmp     eax, [ebp+var_14]

CODE:004B1FA2             jg      loc_4B1F1C

CODE:004B1FA8 

CODE:004B1FA8 loc_4B1FA8:                     ; CODE XREF: sub_4B1DC0+B3j

CODE:004B1FA8                                 ; sub_4B1DC0+120j

CODE:004B1FA8             mov     eax, [ebp+arg_0]

CODE:004B1FAB             mov     edx, [ebp+var_10]

CODE:004B1FAE             call    @System@@LStrAsg$qqrv ; System::__linkproc__ LStrAsg(void)

CODE:004B1FB3             xor     eax, eax

CODE:004B1FB5             pop     edx

CODE:004B1FB6             pop     ecx

CODE:004B1FB7             pop     ecx

CODE:004B1FB8             mov     fs:[eax], edx

CODE:004B1FBB             jmp     short loc_4B1FCF

CODE:004B1FBD ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?

CODE:004B1FBD 

CODE:004B1FBD loc_4B1FBD:                     ; DATA XREF: sub_4B1DC0+49o

CODE:004B1FBD             jmp     @System@@HandleAnyException$qqrv ; System::__linkproc__ HandleAnyException(void)

CODE:004B1FC2 ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?

CODE:004B1FC2             mov     eax, [ebp+8]

CODE:004B1FC5             call    @System@@LStrClr$qqrr17System@AnsiString ; System::__linkproc__ LStrClr(System::AnsiString &)

CODE:004B1FCA             call    @@DoneExcept$qqrv ; __linkproc__ DoneExcept(void)

CODE:004B1FCF 

CODE:004B1FCF loc_4B1FCF:                     ; CODE XREF: sub_4B1DC0+1FBj

CODE:004B1FCF             xor     eax, eax

CODE:004B1FD1             pop     edx

CODE:004B1FD2             pop     ecx

CODE:004B1FD3             pop     ecx

CODE:004B1FD4             mov     fs:[eax], edx

CODE:004B1FD7             push    offset loc_4B2006

CODE:004B1FDC 

CODE:004B1FDC loc_4B1FDC:                     ; CODE XREF: CODE:004B2004j

CODE:004B1FDC             lea     eax, [ebp+var_3C]

CODE:004B1FDF             mov     edx, 6

CODE:004B1FE4             call    @System@@LStrArrayClr$qqrv ; System::__linkproc__ LStrArrayClr(void)

CODE:004B1FE9             lea     eax, [ebp+var_10]

CODE:004B1FEC             call    @System@@LStrClr$qqrr17System@AnsiString ; System::__linkproc__ LStrClr(System::AnsiString &)

CODE:004B1FF1             lea     eax, [ebp+var_8]

CODE:004B1FF4             mov     edx, 2

CODE:004B1FF9             call    @System@@LStrArrayClr$qqrv ; System::__linkproc__ LStrArrayClr(void)

CODE:004B1FFE             retn
```

## 运算逻辑说明

运算过程类似于：

先补足

`用户名 = 用户名 + n 个空格 + machinecode_2`，中间加空格补足到 12 字节，`machinecode_2` 为程序根据 `asm` 指令 `cpuid` 经过运算得来。

`(RegCode[n]^UserName[n-1] - RegCode[n-1]) = Out[n]`

中间减法为若不够减，就给加上 `0xFF` 再减。

最后 `Out[]` 须等于 `machinecode_1 + "gif"`，同样 `machinecode_1` 也是程序根据 `asm` 指令 `cpuid` 经过运算得来。

另 `sub_5060D4` 生成预注册码序列号。

随手写了个注册机。`vc6` 下编译通过。

## 注册机代码示例

```c
void CAaDlg::OnButton1() 

{ 

  char PreRegCode[30] = "EA4B0-8480A-0614C-E6FD3-370D6"; 

  char* UserName = "test1";      //用户名,长度小于 8    

  char TempPreRegCode[24]={0};  // 保存去掉"-"及最后一组中的第一位固定值 3 的值    

  char* pp =TempPreRegCode; 

  unsigned char PreRegCode_1[7]; 

  unsigned char PreRegCode_2[5]; 

  int i = 0, temp2 = 0; 

  // 去掉"-"及最后一组中的第一位固定值 3 

  while(PreRegCode[i] !=0) 

  { 

    if (i == 24 || PreRegCode[i]=='-')    // PreRegCode[24] = 3    

    { 

      i++; 

      continue;     

    } 

    *(pp++) = PreRegCode[i];   

    i++; 

  } 

  pp=NULL; 

  // 前 14 字节，为 PreRegCode_1，来反推 MachinCode_1 

  for(i=0;i<7;i++) 

  { 

    sscanf(&TempPreRegCode[i*2], "%02x", &temp2); 

    PreRegCode_1[i] = temp2; 

  } 

  // 后 5 字节，为 PreRegCode_2，，来反推 MachinCode_2 

  for(i=0;i<5;i++) 

  { 

    sscanf(&TempPreRegCode[14 + i*2], "%02x", &temp2); 

    PreRegCode_2[i] = temp2; 

  } 

  char* szEDX = "http://www.okayle.com";  //程序种子数，固定值   

  unsigned char MachinCode_1[6], MachinCode_2[4]; 

  //由预注册码得来最后比较值得前 6 字节 

  for(i =0 ; i<6 ; i++) 

  { 

    int temp1 = PreRegCode_1[i+1]^szEDX[i]; 

    if(temp1  > PreRegCode_1[i]) 

      MachinCode_1[i] = temp1 - PreRegCode_1[i]; 

    else 

      MachinCode_1[i] = temp1 + 0xFF - PreRegCode_1[i];   

  }   

  //由预注册码得来填充到用户名后 4 字节   

  for(i =0 ; i<4 ; i++) 

  { 

    int temp1 = PreRegCode_2[i+1]^szEDX[i]; 

    if(temp1  > PreRegCode_2[i]) 

      MachinCode_2[i] = temp1 - PreRegCode_2[i]; 

    else 

      MachinCode_2[i] = temp1 + 0xFF - PreRegCode_2[i];   

  } 

  char Out[9];      // 最后计算出的结果要等于这个字符串    

  for(i=0;i<6;i++)   

    Out[i]=MachinCode_1[i]; 

  Out[6] = 'g';  Out[7] = 'i';  Out[8] = 'f'; 

  char UserName_2[16] = {0} ;  // 保存处理后的用户名    

  strcpy(UserName_2, UserName);      //用户名,长度小于 8    

  strcat(UserName_2, "        ");  //后面填充 8 个空格   

  UserName_2[8] = MachinCode_2[0]; 

  UserName_2[9] = MachinCode_2[1]; 

  UserName_2[10] = MachinCode_2[2]; 

  UserName_2[11] = MachinCode_2[3]; //只用前 12 位 

  unsigned char RegCode[10];     

  RegCode[0] = 0x30; //任意给 

  for(i=0; i<10 ; i++) 

  { 

    RegCode[i+1] = (Out[i]+RegCode[i])^UserName_2[i]; 

    if((RegCode[i+1]^UserName_2[i])<RegCode[i]) 

      RegCode[i+1] = (Out[i]+RegCode[i]-0xFF)^UserName_2[i];   

  } 

  char szRegCode[30]; 

  memset(szRegCode, 0x0, sizeof(szRegCode)); 

  for(i=0; i<10 ; i++) 

  { 

    wsprintf(&szRegCode[i*2], "%02X",RegCode[i]); 

  } 

  MessageBox(szRegCode, "Kengen",MB_OK); 

}
```

`4dd466d0e1e0cc123afbe504da2462ca`
