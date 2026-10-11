
# 家庭银行家 v2b53

> 原文：[`www.pediy.com/kssd/pediy06/pediy6522.htm`](https://www.pediy.com/kssd/pediy06/pediy6522.htm)

Software : 家庭银行家 v2b53  
家庭理财软件  
无法使用 KeyMake 做内存补丁, 抵制 OllyDbg 调试(RtlRaiseException、zwQueryInformationProcess)  
[`www.homebanker.net/`](http://www.homebanker.net/)  

Tools : `pe-scan`, `W32Dasm`, `WinHex`, `OllyDbg`, `Win2000`  

Cracker : `lq7972` `[bruceyu13@sina.com]`  

蛮久冒做 PJ 了，今天温习了一下～  

用 `pe-scan` 可以脱壳，用 `W32Dasm` 反汇编，  

【1.】 用 RET 大法轻松实现注册  

查找软件在程序主窗口标题栏中的"(未注册版本，请注册，剩余天数:45)"  

```
:00746CDD 8B8000030000            mov eax, dword ptr [eax+00000300]
:00746CE3 E8B4D0F4FF              call 00693D9C; 跟进
:00746CE8 84C0                    test al, al; 这里是注册标识
:00746CEA 0F84B0000000            je 00746DA0; al = 0?
:00746CF0 8D55E8                  lea edx, dword ptr [ebp-18]
:00746CF3 A1F4FD7500              mov eax, dword ptr [0075FDF4]
:00746CF8 E8BF45CFFF              call 0043B2BC
:00746CFD FF75E8                  push [ebp-18]
* Possible StringData Ref from Data Obj ->"(注册用户: "
; ...
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00746CEA(C)
|
:00746DA0 8D55E0                  lea edx, dword ptr [ebp-20]
:00746DA3 A1F4FD7500              mov eax, dword ptr [0075FDF4]
:00746DA8 E80F45CFFF              call 0043B2BC
:00746DAD FF75E0                  push [ebp-20]
* Possible StringData Ref from Data Obj ->"(未注册版本，请注册，剩余天数:"
; ...
; ===========================================================================
; 跟进
; ret 修改大法
:00693D9C 55                      push ebp; 在这里改 "55" 为 "C3"
:00693D9D 8BEC                    mov ebp, esp
:00693D9F E848010000              call 00693EEC
:00693DA4 5D                      pop ebp
:00693DA5 C3                      ret

; 用 W32Dasm 有时不大灵光，用 WinHex 吧
; W32Dasm 中，把在光标定在 00693D9C 行，状态栏中内容如下：
Line:1362831 Pg 27257 of 35256 Code Data @:00693D9C @Offset 00293D9Ch ...
; WinHex 打开主程序文件, Alt+G, 输入 "293D9C", 改 "55" 为 "C3"
; 保存，ok
```

【2.】 寻找注册算法写注册机

```
; ...
:00736129 8B83F0020000            mov eax, dword ptr [ebx+000002F0]
:0073612F E88851D0FF              call 0043B2BC
:00736134 8B55F8                  mov edx, dword ptr [ebp-08]; 用户名 name
:00736137 33C9                    xor ecx, ecx
:00736139 8B8300030000            mov eax, dword ptr [ebx+00000300]
:0073613F E874DFF5FF              call 006940B8; 计算注册码，跟进
:00736144 84C0                    test al, al
:00736146 751A                    jne 00736162
:00736148 6A10                    push 00000010
; ...
* Possible StringData Ref from Data Obj ->"注册失败，请检查您的注册码是否输入正确。"
|
:0073614F BAC4617300              mov edx, 007361C4
; ...
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00736146(C)
|
:00736162 8B8300030000            mov eax, dword ptr [ebx+00000300]
:00736168 E82FDCF5FF              call 00693D9C
:0073616D 84C0                    test al, al
:0073616F 741F                    je 00736190
:00736171 6A40                    push 00000040
* Possible StringData Ref from Data Obj ->"家庭银行家"
|
:00736173 B9B8617300              mov ecx, 007361B8
* Possible StringData Ref from Data Obj ->"注册成功，谢谢。请退出程序后重新进入。"
; ...
:00736190 33C0                    xor eax, eax
; ...
; ==============================================================================
; 0073613F 计算注册码
; ...
:006940F5 8B45FC                  mov eax, dword ptr [ebp-04]; 用户名
:006940F8 E85F02D7FF              call 0040435C; 用户名长度 name_len
:006940FD 3B433C                  cmp eax, dword ptr [ebx+3C]; name_len > 0x19? Yes, jump
:00694100 7F19                    jg 0069411B
:00694102 8B45FC                  mov eax, dword ptr [ebp-04]
:00694105 E85202D7FF              call 0040435C
:0069410A 3B4340                  cmp eax, dword ptr [ebx+40]; name_len < 3? Yes, jump
:0069410D 7C0C                    jl 0069411B
; ...
:0069411B 33DB                    xor ebx, ebx
:0069411D EB60                    jmp 0069417F
; ...
:0069417F 33C0                    xor eax, eax
; ??????????????????????????????????????
; so, name_len > 3 && name_len < 0x19
:0069413D E866FCFFFF              call 00693DA8; 关键，跟进
:00694142 8B45F0                  mov eax, dword ptr [ebp-10]; 真注册码
:00694145 8B5508                  mov edx, dword ptr [ebp+08]; 假注册码
; ==============================================================================
; 0069413D 计算注册码
; ...
:00693DD6 8B45FC                  mov eax, dword ptr [ebp-04]; 用户名
:00693DD9 E87E05D7FF              call 0040435C; 用户名长度 name_len
:00693DDE 3B463C                  cmp eax, dword ptr [esi+3C]; name_len > 0x19?
:00693DE1 7F0D                    jg 00693DF0
:00693DE3 8B45FC                  mov eax, dword ptr [ebp-04]
:00693DE6 E87105D7FF              call 0040435C
:00693DEB 3B4640                  cmp eax, dword ptr [esi+40]; name_len >= 3?
:00693DEE 7D0C                    jge 00693DFC; Yes, jump
; ...
:00693DFC 8B45FC                  mov eax, dword ptr [ebp-04]
:00693DFF E85805D7FF              call 0040435C
:00693E04 8BD8                    mov ebx, eax; 用户名长度，计数器
:00693E06 EB31                    jmp 00693E39
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00693E4A(C)
|
:00693E08 8B45FC                  mov eax, dword ptr [ebp-04]; 用户名 name
:00693E0B 8A4418FF                mov al, byte ptr [eax+ebx-01]; 从最后一位起 name[len-i]
:00693E0F 25FF000000              and eax, 000000FF
:00693E14 33D2                    xor edx, edx
:00693E16 52                      push edx
:00693E17 50                      push eax
:00693E18 8B4658                  mov eax, dword ptr [esi+58]; 0xC7BC0D36
:00693E1B 8B565C                  mov edx, dword ptr [esi+5C]; 0x0000025C
:00693E1E E8763BD7FF              call 00407999; 关键，跟进，根据 name[len-i] 计算得到 temp00
:00693E23 52                      push edx
:00693E24 50                      push eax
:00693E25 8D45E4                  lea eax, dword ptr [ebp-1C]
:00693E28 E81F67D7FF              call 0040A54C; temp00 转为十进制数字 temp01
:00693E2D 8B55E4                  mov edx, dword ptr [ebp-1C]
:00693E30 8D45F4                  lea eax, dword ptr [ebp-0C]
:00693E33 E82C05D7FF              call 00404364
:00693E38 4B                      dec ebx; 计数器递减
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00693E06(U)
|
:00693E39 8B45FC                  mov eax, dword ptr [ebp-04]
:00693E3C E81B05D7FF              call 0040435C
:00693E41 83E806                  sub eax, 00000006
:00693E44 3BD8                    cmp ebx, eax
:00693E46 7C04                    jl 00693E4C
:00693E48 85DB                    test ebx, ebx
:00693E4A 7FBC                    jg 00693E08; 循环↑
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00693E46(C)
|
:00693E4C 8D55F8                  lea edx, dword ptr [ebp-08]
:00693E4F 8B45F4                  mov eax, dword ptr [ebp-0C]; 得到中间值 temp01，千万别当它是注册码
:00693E52 E8F925D7FF              call 00406450; 关键，跟进，利用 temp01 计算注册码 reg_code
:00693E57 8945E8                  mov dword ptr [ebp-18], eax; 注册码后 8 位
:00693E5A 8955EC                  mov dword ptr [ebp-14], edx; 注册码前 4 位
; ...
:00693E7F E83867D7FF              call 0040A5BC; 连接
:00693E84 8B07                    mov eax, dword ptr [edi]; 得到真的注册码 reg_code
; ==============================================================================
; 00693E1E 根据 name[len-i] 计算得到 temp00
; ...
:0040799D 33FF                    xor edi, edi
```



# 汇编代码分析：用户名处理与注册码计算

## 1. 用户名逆序与旋转累加

这段代码位于地址 `:0040799F` 至 `:004079F3`，主要逻辑是对用户名进行位操作（移位和累加），通常用于生成中间值 `temp00`。

```
:0040799F 8B5C2414                mov ebx, dword ptr [esp+14]; 用户名第 len-i 位 name [len-i]
:004079A3 8B4C2418                mov ecx, dword ptr [esp+18]; 0x00000000
; ...
:004079CC 8BE9                    mov ebp, ecx
:004079CE B940000000              mov ecx, 00000040
:004079D3 57                      push edi
:004079D4 33FF                    xor edi, edi
:004079D6 33F6                    xor esi, esi
; ****************************************************************************************************
:004079D8 D1E0                    shl eax, 1; eax : 0xC7BC0D36
:004079DA D1D2                    rcl edx, 1; edx : 0x0000025C
:004079DC D1D6                    rcl esi, 1
:004079DE D1D7                    rcl edi, 1
:004079E0 3BFD                    cmp edi, ebp
:004079E2 720B                    jb 004079EF
:004079E4 7704                    ja 004079EA
:004079E6 3BF3                    cmp esi, ebx; esi : temp00 的第 1、2 位
:004079E8 7205                    jb 004079EF
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004079E4(C)
|
:004079EA 2BF3                    sub esi, ebx
:004079EC 1BFD                    sbb edi, ebp
:004079EE 40                      inc eax
* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:004079E2(C), :004079E8(C)
|
:004079EF E2E7                    loop 004079D8; cx != 0? Yes,↑
; ****************************************************************************************************
:004079F1 8BC6                    mov eax, esi
:004079F3 8BD7                    mov edx, edi
```

## 2. 注册码计算循环初始化

在地址 `:0040645A` 处，程序利用中间值 `temp01` 开始计算注册码 `reg_code`。这里初始化了计数器和累加器。

```
; ==============================================================================
; 00693E52 利用 temp01 计算注册码 reg_code
; ...
:0040645A 8BF0                    mov esi, eax; 中间值 temp01
:0040645C BD01000000              mov ebp, 00000001; 计数器
:00406461 33FF                    xor edi, edi
:00406463 C744240800000000        mov [esp+08], 00000000
:0040646B C744240C00000000        mov [esp+0C], 00000000
; ...
```

## 3. 处理正负号

此段代码检查字符是否为空格、加号 `+` 或减号 `-`，以处理带符号的数值。

```
:00406481 45                      inc ebp
:00406482 807C2EFF20              cmp byte ptr [esi+ebp-01], 20; 取第 1 位与 0x20 ( ![](img/e45fac309594d4f9580d902b0ed40a73.jpg) 比较
:00406487 74F8                    je 00406481; 等于?↑
:00406489 C644241000              mov [esp+10], 00
:0040648E 8A442EFF                mov al, byte ptr [esi+ebp-01]
:00406492 3C2D                    cmp al, 2D; 取第 1 位与 0x2D (-)比较
:00406494 7508                    jne 0040649E; 不等于?↓
:00406496 C644241001              mov [esp+10], 01
:0040649B 45                      inc ebp
:0040649C EB05                    jmp 004064A3
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00406494(C)
|
:0040649E 3C2B                    cmp al, 2B; 取第 1 位与 0x2D (+)比较
:004064A0 7501                    jne 004064A3; 不等于?
:004064A2 45                      inc ebp
* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:0040649C(U), :004064A0(C)
|
```

## 4. 处理美元符与数字分支

检查字符是否为美元符 `$` 或数字 `0`，并相应地跳转或继续处理。

```
:004064A3 B301                    mov bl, 01
:004064A5 807C2EFF24              cmp byte ptr [esi+ebp-01], 24; 与 0x24 ($)比较
:004064AA 741B                    je 004064C7
:004064AC 807C2EFF30              cmp byte ptr [esi+ebp-01], 30; 与 0x30 (0)比较
:004064B1 0F85DA000000            jne 00406591; 若不等于 0 跳走 ↓
; ==============================================================================
; 等于 0，就取下一位
:004064B7 8A042E                  mov al, byte ptr [esi+ebp]; temp01[1]
:004064BA E815C8FFFF              call 00402CD4
; -----------------------------------------------------------------------------
:00402CD4 80F861                  cmp al, 61
:00402CD7 7208                    jb 00402CE1
:00402CD9 80F87A                  cmp al, 7A
:00402CDC 7703                    ja 00402CE1
:00402CDE 80E820                  sub al, 20
:00402CE1 C3                      ret
; -----------------------------------------------------------------------------
:004064BF 3C58                    cmp al, 58
:004064C1 0F85CA000000            jne 00406591 ↓ ; 跳到下面处理
; ...
; ==============================================================================
; 不等于 0，循环
; ++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
```

## 5. 数值转换与累加逻辑

这部分处理具体的数值转换逻辑，包括 ASCII 到二进制数的转换以及累加运算。

```
:00406591 8A442EFF                mov al, byte ptr [esi+ebp-01]; 中间值第 i 位 temp01 [i]
:00406595 8BD0                    mov edx, eax
:00406597 80C2D0                  add dl, D0
:0040659A 80EA0A                  sub dl, 0A
:0040659D 7362                    jnb 00406601; 如果 temp01[i]>0x0A，也就是不在 10D 以内
:0040659F 8BF8                    mov edi, eax
:004065A1 81E7FF000000            and edi, 000000FF
:004065A7 83EF30                  sub edi, 00000030; temp01[i] - 0x30，为数字本身
:004065AA 837C240C00              cmp dword ptr [esp+0C], 00000000
:004065AF 7509                    jne 004065BA; 不等于 0？↓
:004065B1 837C240800              cmp dword ptr [esp+08], 00000000
:004065B6 7249                    jb 00406601; 小于 0？↓
:004065B8 EB02                    jmp 004065BC
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004065AF(C)
|
:004065BA 7C45                    jl 00406601; 小于 0？↓
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004065B8(U)
|
:004065BC 817C240CCCCCCC0C        cmp dword ptr [esp+0C], 0CCCCCCC
:004065C4 750C                    jne 004065D2; ↓
:004065C6 817C2408CCCCCCCC        cmp dword ptr [esp+08], CCCCCCCC
:004065CE 7604                    jbe 004065D4
:004065D0 EB2F                    jmp 00406601; ↓
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004065C4(C)
|
:004065D2 7F2D                    jg 00406601; ↓
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004065CE(C)
|
:004065D4 6A00                    push 00000000
:004065D6 6A0A                    push 0000000A
:004065D8 8B442410                mov eax, dword ptr [esp+10]
:004065DC 8B542414                mov edx, dword ptr [esp+14]
:004065E0 E87B120000              call 00407860; 计算
```

## 6. 乘法辅助函数调用

调用位于 `:00407860` 的函数进行乘法运算，该函数也被其他地址引用。

```
; -----------------------------------------------------------------------------
* Referenced by a CALL at Addresses:
|:004065E0   , :0040AD33   , :0040AD4D   , :006F61FE    
|
:00407860 52                      push edx
:00407861 50                      push eax
:00407862 8B442410                mov eax, dword ptr [esp+10]
:00407866 F72424                  mul dword ptr [esp]
:00407869 8BC8                    mov ecx, eax
:0040786B 8B442404                mov eax, dword ptr [esp+04]
:0040786F F764240C                mul [esp+0C]
:00407873 03C8                    add ecx, eax
:00407875 8B0424                  mov eax, dword ptr [esp]
:00407878 F764240C                mul [esp+0C]
:0040787C 03D1                    add edx, ecx
:0040787E 59                      pop ecx
:0040787F 59                      pop ecx
:00407880 C20800                  ret 0008
```

## 7. 结果累加与保存

最后将乘法结果累加到寄存器中，并保存回栈上，生成注册码的部分位。

```
; -----------------------------------------------------------------------------
:004065E5 52                      push edx
:004065E6 50                      push eax
:004065E7 8BC7                    mov eax, edi
:004065E9 99                      cdq
:004065EA 030424                  add eax, dword ptr [esp]; 累加，就是注册码的后 8 位
:004065ED 13542404                adc edx, dword ptr [esp+04]; 前 4 位
:004065F1 83C408                  add esp, 00000008
:004065F4 89442408                mov dword ptr [esp+08], eax
:004065F8 8954240C                mov dword ptr [esp+0C], edx
```



### 汇编片段

```
:004065FC 45                 inc ebp  
:004065FD 33DB               xor ebx, ebx  
:004065FF EB90               jmp 00406591; ↑  

; ++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++  
; ...  

:00406662 8B442408           mov eax, dword ptr [esp+08]; 后 8 位  
:00406666 8B54240C           mov edx, dword ptr [esp+0C]; 前 4 位
```

### 总结

这个软件还是耍了蛮多把戏的。

### 注册算法

1. 注册用户名的长度在 `(3～25)` 间。
2. 从注册用户名最末向前取，计算出一个中间值，然后把它转为元素是十进制值的整数数组；有两个值要注意：`0xC7BC0D36`、`0x0000025C`。
3. 利用第 2 步中的结果计算真正的注册码，这个码长须为 12 位，计算结果若小了就在前面补 0，若超过了 12 位，取前面 12 个字符。

### 注册机

; 从反汇编中拷过来的，懒得(用高级语言伪指令)整理了；凑合着看吧

```
; $$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$
; keygen.asm
            .386
            .model flat, stdcall
            option casemap : none
include     windows.inc
include     user32.inc
includelib  user32.lib
include     kernel32.inc
includelib  kernel32.lib
include     masm32.inc
includelib  masm32.lib

DLG_MAIN    equ     101
ICO_MAIN    equ     102
IDC_NAME    equ     1000
IDC_CODE    equ     1001

        .data?
hInstance   dd  ?

        .data
szErrTitle  db  "ErrInfo",0
szErr       db  "NameLen:[3, 25)", 0
bCounter    dd  4
bNameLen    dd  4
szUserName  db  32 dup (0)
szRegCode   db  32 dup (0)
szTemp      db  32 dup (0)
szTemp01    db  4  dup (0)
szTemp02    db  4  dup (0)
szTemp03    db  4  dup (0)
szTemp04    db  4  dup (0)
szTemp05    db  4  dup (0)
szTemp06    db  4  dup (0)
szTemp07  db  12 dup (0)
szTemp08  db  12 dup (0)
szFmt       db  "%s%s", 0

        .code
; =====================================================================
_CalcCode       proc
                pushad
                mov     dword ptr szTemp01, eax
                mov     dword ptr szTemp02, edx
                mov     eax, dword ptr szTemp04
                mul     dword ptr szTemp01
                mov     ecx, eax
                mov     eax, dword ptr szTemp02
                mul     dword ptr szTemp03
                add     ecx, eax
                mov     eax, dword ptr szTemp01
                mul     dword ptr szTemp03
                add     edx, ecx
                mov     dword ptr szTemp03, eax
                mov     dword ptr szTemp04, edx
                popad
                ret
_CalcCode       endp
; =====================================================================
_Temp2Code      proc
                pushad
                mov     esi, eax
                mov     ebp, 1
                mov     ebx, 1
                xor     ecx, ecx
                xor     edi, edi
                mov     dword ptr szTemp05, 0
                mov     dword ptr szTemp06, 0
BEGIN :
                mov     al, byte ptr [esi+ebp\-1]
                mov     edx, eax
                add     dl, 0D0h
                sub     dl, 0Ah
                jnb     GAMEOVER
                mov     edi, eax
                and     edi, 0FFh
                sub     edi, 030h
                cmp     dword ptr szTemp06, 0
                jnz     @F
                cmp     dword ptr szTemp05, 0
                jb      GAMEOVER
                jmp     JMP_HERE
@@ :
                jl      GAMEOVER
JMP_HERE :
                cmp     dword ptr szTemp06, 0CCCCCCCh
                jnz     @F
                cmp     dword ptr szTemp05, 0CCCCCCCCh
                jbe     JBE_HERE
                jmp     GAMEOVER
@@ :
                jg      GAMEOVER
JBE_HERE :
                mov     dword ptr szTemp03, 0Ah
                mov     dword ptr szTemp04, 0
                mov     eax, dword ptr szTemp05
                mov     edx, dword ptr szTemp06
                call    _CalcCode
                mov     edx, dword ptr szTemp04
                mov     eax, edi
                cdq
                add     eax, dword ptr szTemp03
                adc     edx, dword ptr szTemp04
                mov     dword ptr szTemp05, eax
                mov     dword ptr szTemp06, edx
                inc     ebp
                xor     ebx, ebx
                jmp     BEGIN
GAMEOVER :
                popad
                ret
_Temp2Code      endp
; =====================================================================
_Name2Temp      proc
                pushad
                xor     edi, edi
                mov     ebx, dword ptr szTemp01
                mov     ecx, dword ptr szTemp02
                or      ecx, ecx
                jnz     @F
                or      edx, edx
                je      LAST
                or      ebx, ebx
                je      LAST
@@ :
                or      edx, edx
                jns     @F
                neg     edx
                neg     eax
                sbb     edx, 0
                or      edi, 1
@@ :
                or      ecx, ecx
                jns     @F
                neg     ecx
                neg     ebx
                sbb     ecx, 0
@@ :
                mov     ebp, ecx
                mov     ecx, 040h
                xor     edi, edi
                xor     esi, esi
LOOP01 :
                shl     eax, 1
                rcl     edx, 1
                rcl     esi, 1
                rcl     edi, 1
                cmp     edi, ebp
                jb      @F
                ja      JUMP01
                cmp     esi, ebx
                jb      @F
JUMP01 :
                sub     esi, ebx
                sbb     edi, ebp
                inc     eax
@@ :                 
                loop    LOOP01
                mov     eax, esi
                mov     dword ptr szTemp01, eax
                mov     edx, edi
                mov     dword ptr szTemp02, edx
                test    ebx, 1
                je      @F
                neg     edx
                neg     eax
                sbb     edx, 0
@@ :
                popad
                ret
LAST :
                div     ebx
                xchg    eax, edx
                xor     edx, edx
                jmp     @B
_Name2Temp      endp
; =====================================================================
_CalcRegCode    proc
                pushad
                xor     edx, edx
                mov     dword ptr bCounter, edx
                mov     ebx, dword ptr bNameLen
                jmp     @F
CALC_START :
                mov     eax, offset szUserName
                mov     al, byte ptr [eax+ebx\-01]
                and     eax, 000000FFh
                mov     dword ptr szTemp01, eax
                mov     dword ptr szTemp02, edx                 
                mov     eax, 0C7BC0D36h
                mov     edx, 0000025Ch
                call    _Name2Temp
                mov     eax, dword ptr szTemp01
                invoke  dwtoa, eax, addr szTemp02
                mov     eax, dword ptr szTemp02
                mov     edi, dword ptr bCounter
                mov     dword ptr [szTemp+edi], eax
                invoke  lstrlen, addr szTemp02
                add     edi, eax
                mov     dword ptr bCounter, edi
                dec     ebx
@@ :
                mov     eax, dword ptr bNameLen
                sub     eax, 00000006
                cmp     ebx, eax  
                jl      @F
                test    ebx, ebx
                jg      CALC_START
@@ :
                mov     eax, offset szTemp
                call    _Temp2Code
```



```
mov    eax, dword ptr szTemp05  
invoke  dw2hex, eax, addr szTemp07  
mov    eax, dword ptr szTemp06  
invoke  dw2hex, eax, addr szTemp08  
invoke  wsprintf, addr szTemp, addr szFmt, \  
    addr szTemp08, addr szTemp07  
invoke  lstrlen, addr szTemp  
mov    esi, eax  
xor    edi, edi  
xor    edx, edx  
mov    eax, offset szTemp  
.repeat  
    mov    dl, byte ptr [eax+edi]  
    inc    edi  
.until  edx != 030h  
sub    esi, edi  
inc    esi  
.if    esi <= 0Ch  
    invoke  rstr, addr szTemp, addr szRegCode, 0Ch  
.elseif  esi < 010h  
    mov    eax, offset szTemp  
    invoke  rstr, eax, addr szTemp, esi  
    invoke  lstr, addr szTemp, addr szRegCode, 0Bh  
.else  
    invoke  lstr, addr szTemp, addr szRegCode, 0Bh  
.endif  
popad  
ret  

_CalcRegCode    endp  

; >>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
_ProDlgMain proc    uses    ebx edi esi hWnd, wMsg, wParam, lParam  
    mov     eax, wMsg  
    .if     eax == WM_CLOSE  
        invoke  EndDialog, hWnd, NULL  
    .elseif eax == WM_INITDIALOG  
        invoke  LoadIcon, hInstance, ICO_MAIN  
        invoke  SendMessage, hWnd, WM_SETICON, ICON_BIG, eax  
    .elseif eax == WM_COMMAND  
        mov eax, wParam  
        .if ax == IDOK  
            invoke  RtlZeroMemory,addr szRegCode, 32  
            invoke  GetDlgItemText, hWnd, IDC_NAME, \  
                addr szUserName, 32  
            invoke  lstrlen, addr szUserName  
            mov     bNameLen, eax  
            .if     bNameLen >= 3 && bNameLen < 25  
                call    _CalcRegCode  
            .else  
                invoke  MessageBox, NULL, addr szErr, \  
                    addr szErrTitle, MB_OK  
            .endif  
            invoke  SetDlgItemText, hWnd, IDC_CODE, addr szRegCode  
        .elseif ax == IDCANCEL  
            invoke  EndDialog, hWnd, NULL  
        .endif  
    .else  
        mov eax, FALSE  
        ret  
    .endif  
    mov eax, TRUE  
    ret  

_ProDlgMain endp  

; >>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
start:  
    invoke  GetModuleHandle, NULL  
    mov     hInstance, eax  
    invoke  DialogBoxParam, hInstance, DLG_MAIN, \  
        NULL, offset _ProDlgMain, NULL  
    invoke  ExitProcess, NULL  
    end     start  

; $$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$
; keygen.rc  
#include <resource.h>  

#define    DLG_MAIN  101  
#define    ICO_MAIN  102  
#define   IDC_NAME  1000  
#define   IDC_CODE  1001  
#define   IDC_STATIC  -1  

IDI_MAIN  ICON    "KeyGen.ico"  

DLG_MAIN DIALOG DISCARDABLE  174, 140, 187, 79  
STYLE DS_MODALFRAME | WS_POPUP | WS_CAPTION | WS_SYSMENU  
CAPTION "家庭银行家 V 2.53 KeyGen by lq7972 [bruceyu13@sina.com]"  
FONT 10, "System"  

BEGIN  
    DEFPUSHBUTTON   "确定",IDOK,21,54,50,14  
    PUSHBUTTON      "取消",IDCANCEL,101,55,50,14  
    LTEXT           "用户名:",IDC_STATIC,7,14,27,11  
    LTEXT           "注册码:",IDC_STATIC,7,34,29,11  
    EDITTEXT        IDC_NAME,37,13,143,12,ES_AUTOHSCROLL  
    EDITTEXT        IDC_CODE,36,33,144,12,ES_AUTOHSCROLL  
END  

; $$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$
; makefile  
NAME = KeyGen  

OBJS = $(NAME).obj  

RES = $(NAME).res  

LINK_FLAG = /subsystem:windows  

ML_FLAG = /c /coff  

$(NAME).exe: $(OBJS) $(RES)  

  Link $(LINK_FLAG) $(OBJS) $(RES)  

.asm.obj:  

  ml $(ML_FLAG) $<  

.rc.res:  

  rc $<  

clean:  

  del \*.obj  

  del \*.res  

; >>>>>>>>>>>>>>>>>>>>
```

`thanks.`

`4dd466d0e1e0cc123afbe504da2462ca`
