
# supercleaner 2.0 (1)

> 原文：[`www.pediy.com/kssd/pediy03/forum734.htm`](https://www.pediy.com/kssd/pediy03/forum734.htm)

`supercleaner` 2.0 超酷的系统清洁工具破解过程！

来到注册处输入用户名：`crack`（任意）

注册码：`12346567890`（任意）

下断：

```
bpx hmemcpy
```

`f5` 回到注册处，按 `ok`

程序被断：

```
* Possible Reference to Dialog: DialogID_0065, CONTROL_ID:03FC, ""

:00409231 68FC030000                  push 000003FC
:00409236 56                          push esi
:00409237 FFD7                        call edi
:00409239 8D842408010000             lea eax, dword ptr [esp+00000108] *假的注册码
:00409240 8D4C2408                   lea ecx, dword ptr [esp+08]       *假的用户名
:00409244 50                          push eax *压入堆栈保存
:00409245 51                          push ecx *压入堆栈保存
:00409246 E8F5060000                 call 00409940 *一定是比较是否正确的子程序，f8 进入
:0040924B 83C408                      add esp, 00000008
:0040924E 84C0                        test al, al
:00409250 7447                        je 00409299
:00409252 8D942408010000             lea edx, dword ptr [esp+00000108]
:00409259 8D442408                   lea eax, dword ptr [esp+08]
:0040925D 52                          push edx
:0040925E 50                          push eax
```

来到：

```
* Referenced by a CALL at Addresses:
|:00409246  , :004098BE

:00409940 81EC00010000                sub esp, 00000100
:00409946 A070A74100                  mov al, byte ptr [0041A770]
:0040994B 53                          push ebx
:0040994C 57                          push edi
:0040994D 88442408                    mov byte ptr [esp+08], al
:00409951 B93F000000                  mov ecx, 0000003F
:00409956 33C0                        xor eax, eax
:00409958 8D7C2409                    lea edi, dword ptr [esp+09]
:0040995C 8B94240C010000             mov edx, dword ptr [esp+0000010C]
:00409963 F3                          repz
:00409964 AB                          stosd
:00409965 66AB                        stosw
:00409967 8D4C2408                    lea ecx, dword ptr [esp+08]
:0040996B 32DB                        xor bl, bl
:0040996D 51                          push ecx
:0040996E 52                          push edx
:0040996F AA                          stosb
:00409970 E8AB000000                 call 00409A20  *这里是算码过程，进入
:00409975 8B8C2418010000             mov ecx, dword ptr [esp+00000118]*出来以后应该是正确的注册码了，用 d eax 看看？呵呵！
:0040997C 83C408                      add esp, 00000008
:0040997F 8D442408                    lea eax, dword ptr [esp+08]
:00409983 50                          push eax
:00409984 51                          push ecx

* Referenced by a CALL at Address:
|:00409970

:00409A20 81EC00010000                sub esp, 00000100
:00409A26 A070A74100                  mov al, byte ptr [0041A770]
:00409A2B 53                          push ebx
:00409A2C 55                          push ebp
:00409A2D 56                          push esi
:00409A2E 57                          push edi
:00409A2F 88442410                    mov byte ptr [esp+10], al
:00409A33 B93F000000                  mov ecx, 0000003F
:00409A38 33C0                        xor eax, eax
:00409A3A 8D7C2411                    lea edi, dword ptr [esp+11]
:00409A3E F3                          repz
:00409A3F AB                          stosd
:00409A40 66AB                        stosw
:00409A42 AA                          stosb
:00409A43 8BBC2414010000             mov edi, dword ptr [esp+00000114]
:00409A4A 57                          push edi
```

我的用户名：`crack`

注册码：`706-26832-576-7224`

# SuperCleaner V1.91

帮助用户清洗他们的计算机硬盘内不必要的文件的程序

类型：共享软件，注册费$20.00，试用 30 天  平台：Win 95/98/NT/2000

目的：找到注册码并分析算法

作者：`2000yeah`

```
:00408289 8D842408010000               lea eax, dword ptr [esp+00000108] ----fake code
:00408290 8D4C2408                     lea ecx, dword ptr [esp+08] ----fake name
:00408294 50                           push eax
:00408295 51                           push ecx
:00408296 E8A5020000                   call 00408540 -------------****** 进入
:0040829B 83C408                       add esp, 00000008
:0040829E 85C0                         test eax, eax
:004082A0 7447                         je 004082E9
:004082A2 8D942408010000              lea edx, dword ptr [esp+00000108]
:004082A9 8D442408                     lea eax, dword ptr [esp+08]
:004082AD 52                           push edx
:004082AE 50                           push eax

* Possible StringData Ref from Data Obj ->"Software\SuperCleaner\Registration" -----save reg code

:004082AF 6850714100                   push 00417150
:004082B4 6801000080                   push 80000001
:004082B9 E8D2020000                   call 00408590

* Possible StringData Ref from Data Obj ->"Software\SuperCleaner\Registration" ----save name

:004082BE 6850714100                   push 00417150
:004082C3 6801000080                   push 80000001
:004082C8 E863010000                   call 00408430
:004082CD 83C418                       add esp, 00000018

* Referenced by a CALL at Addresses:
|:00408296  , :0040850E

:00408540 8B4C2404                      mov ecx, dword ptr [esp+04]
:00408544 81EC00010000                  sub esp, 00000100
:0040854A 8D442400                      lea eax, dword ptr [esp]
:0040854E 56                           push esi
:0040854F 50                           push eax
:00408550 51                           push ecx
:00408551 33F6                         xor esi, esi
:00408553 E8B8000000                   call 00408610    -------------进入此 CALL
:00408558 8B842414010000               mov eax, dword ptr [esp+00000114]
:0040855F 83C408                       add esp, 00000008
:00408562 8D542404                      lea edx, dword ptr [esp+04]
:00408566 52                           push edx  －－－－－－－－－－－－'real code '
:00408567 50                           push eax  －－－－－－－－－－－－'fake code'

* Reference To: KERNEL32.lstrcmpA, Ord:0329h  －－－－－－－－－－－

:00408568 FF15C0504100                 Call dword ptr [004150C0]
:0040856E 85C0                         test eax, eax      －－－－－－－－－－－－

* Possible Reference to String Resource ID=00001: "Registered to: "

:00408570 B801000000                   mov eax, 00000001
:00408575 7402                         je 00408579
:00408577 8BC6                         mov eax, esi

----------------------------------------------------00408553 : call 00408610 内--------------------------------------

* Referenced by a CALL at Address:
|:00408553

:00408610 81EC00010000                  sub esp, 00000100
:00408616 A09CA74100                    mov al, byte ptr [0041A79C]
:0040861B 53                           push ebx
:0040861C 55                           push ebp
:0040861D 56                           push esi
:0040861E 57                           push edi
:0040861F 88442410                      mov byte ptr [esp+10], al
:00408623 B93F000000                    mov ecx, 0000003F
:00408628 33C0                         xor eax, eax
:0040862A 8D7C2411                      lea edi, dword ptr [esp+11]
:0040862E F3                           repz
:0040862F AB                           stosd
:00408630 66AB                         stosw
:00408632 AA                           stosb
:00408633 8BBC2414010000               mov edi, dword ptr [esp+00000114]
:0040863A 57                           push edi

* Reference To: KERNEL32.lstrlenA, Ord:0335h

:0040863B FF1578514100                 Call dword ptr [00415178]
:00408641 8BF0                         mov esi, eax
:00408643 33C9                         xor ecx, ecx
:00408645 33C0                         xor eax, eax
:00408647 85F6                         test esi, esi
```



# 汇编代码分析

以下代码段展示了注册码计算的循环逻辑及字符串拼接过程。

## 第一段循环：计算部分和

```assembly
:00408649 7613                jbe 0040865E   
```

- 引用自无条件或条件跳转地址：
  ```
  :0040865C(C)
  ```

```assembly
:00408651 0FBE1C38            movsx ebx, byte ptr [eax+edi]
; 依次将 NAME 运算

:00408655 03DA                add ebx, edx
; 此时：EDX=26H

:00408657 03CB                add ecx, ebx

:00408659 40                  inc eax

:0040865A 3BC6                cmp eax, esi

:0040865C 72F3                jb 00408651
; 无条件或条件跳转引用：
; :00408649(C)
```

## 数据加载与格式化输出

```assembly
:0040865E 8B9C2418010000      mov ebx, dword ptr [esp+00000118]

:00408665 51                  push ecx
; ecx=23fh=575d

; 可能的字符串数据引用 -> "%u-" 格式

:00408666 6894774100          push 00417794

:0040866B 53                  push ebx

; 引用至: USER32.wsprintfA, Ord:02B3h

:0040866C FF15F4514100        Call dword ptr [004151F4]

:00408672 83C40C              add esp, 0000000C

:00408675 33C9                xor ecx, ecx

:00408677 33C0                xor eax, eax

:00408679 85F6                test esi, esi

:0040867B 7614                jbe 00408691
```

## 第二段循环：乘法运算

```assembly
:0040867D 8B155C794100        mov edx, dword ptr [0041795C]
; 引用自无条件或条件跳转地址：
; :0040868F(C)

:00408683 0FBE2C38            movsx ebp, byte ptr [eax+edi]
; 依次将 NAME 运算

:00408687 0FAFEA              imul ebp, edx
; 此时：EDX=34H

:0040868A 03CD                add ecx, ebp

:0040868C 40                  inc eax

:0040868D 3BC6                cmp eax, esi

:0040868F 72F2                jb 00408683
; 引用自无条件或条件跳转地址：
; :0040867B(C)
```

## 拼接与后续处理

```assembly
:00408691 51                  push ecx
; ecx=55ech=21996d

:00408692 8D4C2414            lea ecx, dword ptr [esp+14]

; 可能的字符串数据引用 -> "%u-" 格式

:00408696 6894774100          push 00417794

:0040869B 51                  push ecx

; 引用至: USER32.wsprintfA, Ord:02B3h

:0040869C FF15F4514100        Call dword ptr [004151F4]

:004086A2 83C40C              add esp, 0000000C

:004086A5 8D542410            lea edx, dword ptr [esp+10]

:004086A9 52                  push edx

:004086AA 53                  push ebx

; 引用至: KERNEL32.lstrcatA, Ord:0326h

:004086AB FF15A0504100        Call dword ptr [004150A0]

:004086B1 33C9                xor ecx, ecx

:004086B3 33C0                xor eax, eax

:004086B5 85F6                test esi, esi

:004086B7 7613                jbe 004086CC
```

## 第三段循环：加法运算

```assembly
:004086B9 8B1560794100        mov edx, dword ptr [00417960]
; 引用自无条件或条件跳转地址：
; :004086CA(C)

:004086BF 0FBE2C38            movsx ebp, byte ptr [eax+edi]
; 依次将 NAME 运算

:004086C3 03EA                add ebp, edx
; 此时：EDX=CH

:004086C5 03CD                add ecx, ebp

:004086C7 40                  inc eax

:004086C8 3BC6                cmp eax, esi

:004086CA 72F3                jb 004086BF
; 引用自无条件或条件跳转地址：
; :004086B7(C)
```

## 再次拼接

```assembly
:004086CC 51                  push ecx
; ecx=1d7h=471d

:004086CD 8D442414            lea eax, dword ptr [esp+14]

; 可能的字符串数据引用 -> "%u-" 格式

:004086D1 6894774100          push 00417794

:004086D6 50                  push eax

; 引用至: USER32.wsprintfA, Ord:02B3h

:004086D7 FF15F4514100        Call dword ptr [004151F4]
; 将注册码合并在一起

:004086DD 83C40C              add esp, 0000000C

:004086E0 8D4C2410            lea ecx, dword ptr [esp+10]

:004086E4 51                  push ecx

:004086E5 53                  push ebx
; ebx=575-21996-

; 引用至: KERNEL32.lstrcatA, Ord:0326h

:004086E6 FF15A0504100        Call dword ptr [004150A0]

:004086EC 33C9                xor ecx, ecx

:004086EE 33C0                xor eax, eax

:004086F0 85F6                test esi, esi

:004086F2 7614                jbe 00408708
```

## 第四段循环：乘法运算

```assembly
:004086F4 8B1564794100        mov edx, dword ptr [00417964]
; 引用自无条件或条件跳转地址：
; :00408706(C)

:004086FA 0FBE2C38            movsx ebp, byte ptr [eax+edi]
; 依次将 NAME 运算

:004086FE 0FAFEA              imul ebp, edx
; 此时:EDX=EH

:00408701 03CD                add ecx, ebp

:00408703 40                  inc eax

:00408704 3BC6                cmp eax, esi

:00408706 72F2                jb 004086FA
; 引用自无条件或条件跳转地址：
; :004086F2(C)
```

## 最终拼接与函数退出

```assembly
:00408708 51                  push ecx
; ecx=1722h=5922d

:00408709 8D542414            lea edx, dword ptr [esp+14]

; 可能的字符串数据引用 -> "%u" 格式，最后一位

:0040870D 6890774100          push 00417790

:00408712 52                  push edx

; 引用至: USER32.wsprintfA, Ord:02B3h

:00408713 FF15F4514100        Call dword ptr [004151F4]
; 将注册码合并在一起

:00408719 83C40C              add esp, 0000000C

:0040871C 8D442410            lea eax, dword ptr [esp+10]

:00408720 50                  push eax

:00408721 53                  push ebx
; ebx=575-21996-471-

; 引用至: KERNEL32.lstrcatA, Ord:0326h

:00408722 FF15A0504100        Call dword ptr [004150A0]
; 过了此 CALL，就看到“全家福”了

:00408728 5F                  pop edi

:00408729 5E                  pop esi

:0040872A 5D                  pop ebp

:0040872B 5B                  pop ebx

:0040872C 81C400010000        add esp, 00000100

:00408732 C3                  ret
```

## 算法概述

此注册码共分 4 段。每段 `EDX` 的值分别等于 `"26h"`、`"34h"`、`"ch"`、`"eh"`。

关键循环逻辑示例（第一段）：

```assembly
:00408651 0FBE1C38            movsx ebx, byte ptr [eax+edi]
; 依次将 NAME 运算

:00408655 03DA                add ebx, edx
; 此时：EDX=26H //ebx=121+26

:00408657 03CB                add ecx, ebx
; //ecx=ebx+ecx

:00408659 40                  inc eax
; //eax=eax+1

:0040865A 3BC6                cmp eax, esi
; //if eax<esi

:0040865C 72F3                jb 00408651
; 依次循环
```

其他每段的循环雷同，不再一一说明。

## 结果验证

- `name`: `yeah`
- `code`: `575-21996-471-5922`

## 注册信息存储位置

注册信息添加在以下注册表路径中：

`HKEY_CURRENT_USER\Software\SuperCleaner\Registration`

相关数据：`4dd466d0e1e0cc123afbe504da2462ca`
