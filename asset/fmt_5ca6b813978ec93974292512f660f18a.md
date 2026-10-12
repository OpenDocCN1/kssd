
# 快捷打开文件夹 V3.0 破解过程

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-111.htm`](https://www.pediy.com/kssd/pediy07/pediy7-111.htm)

**日期**：2005 年 3 月 3 日  
**破解人**：萝卜 `[pcg]`

---

### 【软件信息】
- **软件名称**：快捷打开文件夹
- **软件版本**：`3.0`
- **软件大小**：`511 KB`
- **软件简介**：一款不错的文件夹快速浏览工具
- **软件限制**：未注册只能使用 `30` 次
- **破解声明**：初学 `Crack`，只是感兴趣，没有其它目的。失误之处敬请诸位大侠赐教！
- **破解工具**：`OLLYDBG 1.1 fly 汉化版`

---

### 【破解过程】

先用 `PEID 0.92 汉化增强版` 查壳，无壳，`Borland C++`。

用 `od` 打开目标程序 `Tbfof.exe`，然后查找字符，可以找到“注册失败”字样，双击来到 `cpu` 窗口，向上翻看可以找到以下代码：

```assembly
00406DEA   CALL TBFOF.0042862C
00406DEF   CMP DWORD PTR SS:[EBP-8],0             ;  比较用户名是否为 0
00406DF3   JE SHORT TBFOF.00406DFA               ;  如果为 0，跳走了，当然不能是 0 了
00406DF5   MOV EAX,DWORD PTR SS:[EBP-8]          ;  走到这里，把用户名赋值给 eax
00406DF8   JMP SHORT TBFOF.00406DFF              ;  跳下面的 call
00406DFA   MOV EAX,TBFOF.0045E507
00406DFF   CALL TBFOF.00447C50                    ;  这个 call 用来计算用户名的位数，存到 eax
00406E04   MOV ESI,EAX                            ;  用户名位数赋值给 esi
00406E06   DEC DWORD PTR SS:[EBP-28]
00406E09   LEA EAX,DWORD PTR SS:[EBP-8]
00406E0C   MOV EDX,2
00406E11   CALL TBFOF.0045ADF8
00406E16   TEST ESI,ESI                           ;  比较用户名位数是否为零，我们当然不为 0 向下跳
00406E18   JNZ SHORT TBFOF.00406E4F               ;  跳
00406E1A   MOV EAX,DWORD PTR DS:[4634EC]
00406E1F   PUSH 0
00406E21   MOV ECX,TBFOF.0045E519
00406E26   MOV EDX,TBFOF.0045E508
00406E2B   MOV EAX,DWORD PTR DS:[EAX]
00406E2D   CALL TBFOF.0045ACB0
00406E32   MOV EDX,DWORD PTR SS:[EBP-48]
00406E35   MOV EAX,DWORD PTR DS:[EDX+1F4]
00406E3B   MOV EDX,DWORD PTR DS:[EAX]
00406E3D   CALL DWORD PTR DS:[EDX+7C]
00406E40   MOV ECX,DWORD PTR SS:[EBP-44]
00406E43   MOV DWORD PTR FS:[0],ECX
00406E4A   JMP TBFOF.0040701F
00406E4F   XOR EDX,EDX
00406E51   LEA EAX,DWORD PTR SS:[EBP-9C]          ;  把用户名赋值 eax
00406E57   CMP ESI,EDX                            ;  esi 为用户名的长度
00406E59   JLE SHORT TBFOF.00406E67               ;  小于 0 则跳
00406E5B   /MOVSX ECX,BYTE PTR DS:[EAX]           ;  把用户名第 i 个字符的 asc 赋值给 ecx
00406E5E   |ADD DWORD PTR SS:[EBP-4C],ECX         ;  上面的结果+到 ebp-4c
00406E61   |INC EDX                               ;  计数器+1
00406E62   |INC EAX                               ;  eax+1
00406E63   |CMP ESI,EDX                           ;  比较 edx 是否大于用户名 3
00406E65   \JG SHORT TBFOF.00406E5B               ;  跳回去继续取用户名的 i+1 位
00406E67   ADD DWORD PTR SS:[EBP-4C],7            ;  结果+7
00406E6B   /IMUL EBX,DWORD PTR SS:[EBP-4C]        ;  ebx*上面的结果（ebx 初始值＝1）
00406E6F   |CMP EBX,989680                       ;  比较 ebx 和 989680
00406E75   \JL SHORT TBFOF.00406E6B               ;  小于就跳回去再乘用户名
00406E77   CMP EBX,5F5E100                        ;  比较上面的结果，是否小于等于 5f5e100
00406E7D   JLE SHORT TBFOF.00406E93               ;  满足条件就向下跳
00406E7F   /MOV ECX,2                             ;  否则到这里 ecx=2.
00406E84   |MOV EAX,EBX                           ;  把上面计算的结果赋值给 eax
00406E86   |CDQ                                   ;  字符扩展
00406E87   |IDIV ECX                              ;  eax 除以 ecx
00406E89   |MOV EBX,EAX                           ;  取商
00406E8B   |CMP EBX,5F5E100                       ;  比较 ebx 是否大于 5f5e100
00406E91   \JG SHORT TBFOF.00406E7F               ;  大于则跳回去重新计算
00406E93   XOR EAX,EAX                            ;  否则到这里，初始化 eax(当然下面要用到了)
00406E95   LEA EDX,DWORD PTR SS:[EBP-10]
00406E98   MOV DWORD PTR SS:[EBP-10],EAX
00406E9B   MOV EAX,EBX                            ;  把计算结果赋值给 eax,下面的 call 要用
00406E9D   INC DWORD PTR SS:[EBP-28]
00406EA0   CALL TBFOF.00447914
00406EA5   LEA EDX,DWORD PTR SS:[EBP-10]
00406EA8   XOR ECX,ECX
00406EAA   PUSH EDX
00406EAB   LEA EDX,DWORD PTR SS:[EBP-C]
00406EAE   MOV WORD PTR SS:[EBP-34],20
00406EB4   MOV DWORD PTR SS:[EBP-C],ECX
00406EB7   INC DWORD PTR SS:[EBP-28]
00406EBA   MOV EAX,DWORD PTR SS:[EBP-48]
00406EBD   MOV EAX,DWORD PTR DS:[EAX+200]
00406EC3   CALL TBFOF.0042862C                    ;  这个 call 把注册码转换成 10 进制
00406EC8   LEA EAX,DWORD PTR SS:[EBP-C]
00406ECB   POP EDX
00406ECC   CALL TBFOF.0045AEC8                    ;  关键 call （真码和假码比较地方）
00406ED1   PUSH EAX
00406ED2   DEC DWORD PTR SS:[EBP-28]
00406ED5   LEA EAX,DWORD PTR SS:[EBP-C]
00406ED8   MOV EDX,2
00406EDD   CALL TBFOF.0045ADF8
00406EE2   DEC DWORD PTR SS:[EBP-28]              ; |
00406EE5   LEA EAX,DWORD PTR SS:[EBP-10]          ; |
00406EE8   MOV EDX,2                              ; |
00406EED   CALL TBFOF.0045ADF8                    ; \\TBFOF.0045ADF8
00406EF2   POP ECX
00406EF3   TEST CL,CL
00406EF5   JE TBFOF.00406FDB                      ;  (关键比较，跳到错误地方)比较结果改成 nop，但是打开软件还是显示未注册。
```

上面的 `call` 调用此处：

```assembly
0045AEC8   PUSH EBP
0045AEC9   MOV EBP,ESP
0045AECB   PUSH EBX
0045AECC   MOV EAX,DWORD PTR DS:[EAX]             ;  出现假码
0045AECE   MOV EDX,DWORD PTR DS:[EDX]             ;  出现真码（可以做内存注册机）
0045AED0   CALL TBFOF.0044F2A8
0045AED5   SETE AL
0045AED8   AND EAX,1
0045AEDB   POP EBX
0045AEDC   POP EBP
0045AEDD   RETN                                   ; 返回调用 call
```

---

### 【Crack 总结】

算法简单，希望大家不要笑啊。

用剑英兄的 `key make 1.73` 做了个注册机，为了大家学习 `key make` 贴出源码：

```asm
.const
.data
szHomePage db "http://www.365hz.net",0
szEmail    db "mailto:ljyljx@163.com",0
szErrMess  db "输入的序列号不正确！",0
szBuff db 50 dup(0) ;保存转换后十进制字符。
szFormat db "%d",0  ;输出十进制字符格式

.code
mov ebx,eax
push eax
invoke lstrlen,ebx
mov esi,eax
mov ebx,1h
xor EDX,EDX
XOR EDI,EDI
pop eax
n1:
CMP ESI,EDX
JLE n2      
MOVSX ECX,BYTE PTR [EAX]
ADD EDI,ECX
INC EDX
INC EAX
CMP ESI,EDX          
JG n1
n2:
ADD edi,7h
n3:
IMUL EBX,edi
CMP EBX,989680h
JL n3
CMP EBX,5F5E100h
JLE n5
n4:
MOV ECX,2h
MOV EAX,EBX
CDQ
IDIV ECX
MOV EBX,EAX
CMP EBX,5F5E100h
JG n4
n5:
invoke wsprintf,addr szBuff,addr szFormat,ebx
LEA EAX,szBuff
```
