
# 键盘乐器算法分析

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-181.htm`](https://www.pediy.com/kssd/pediy08/pediy8-181.htm)

**【破文标题】** 键盘乐器分析  
**【破文作者】** 無幻刹那  
**【作者邮箱】** lovecy616927@yahoo.com.cn  
**【所属组织】** 無業游民  
**【软件名称】** 键盘乐器  
**【下载地址】** [http://kmxuyi.vip.myrice.com/chinese/](http://kmxuyi.vip.myrice.com/chinese/)  
**【破解工具】** `w32asm` , `od`  
**【保护方式】** 注册码保护方式  
**【软件限制】** 试用次数  
**【破解难度】** 简单  

----------------------------------------------------

### 软件介绍

键盘乐器是一个利用电脑键盘演奏乐器的软件。操作非常简单。用它你可以演奏包括钢琴、小提琴、吉他、萨克斯、古筝在内的一百多种乐器和音效！键盘乐器具有录制功能、调式的选择功能，键盘布局合理。

利用键盘乐器的录制功能，你能将自己的得意之作录制下来，保存为文件，和朋友一起分享。

调式的选择使你可以根据乐谱选择各种调式，如 `C` 大调，`F` 大调，`c` 小调等，操作更加简便。

合理的键盘布局让你在弹奏乐器时手感舒适，无形中还能提高你的指法，当你成为键盘乐器高手时，说不定已炼成“弹指神通”。^_^

----------------------------------------------------

### 破解声明

我是一只小菜鸟，偶得一点心得，愿与大家分享：）

----------------------------------------------------

### 【破解分析】

用 `W32ASM` 找关键字，很快来到下面这里，下面是我找的关键算法部分

```
00406882   .  68 02010000         PUSH 102                    ; |ControlID = 102 (258.)
00406887   .  53                  PUSH EBX                    ; |hWnd
00406888   .  FF15 DCE14000       CALL DWORD PTR DS:[<&USER32.GetDlgItemIn>; \取得我们输入的假注册码的 16 进制，放入 eax 中
0040688E   .  8B35 74334100       MOV ESI,DWORD PTR DS:[413374]     ;  取得我们的机器码的 16 进制 2BFC631 存入 esi
00406894   .  B9 20000000         MOV ECX,20                  ;  ecx 加 20
00406899   .  83E6 1F             AND ESI,1F                  ;  16 进制机器码和 1f 进行与运算=00000011
0040689C   .  8BD0                MOV EDX,EAX                 ;  eax 传 给 edx 其中 eax 是我们输入的注册码的 16 进制
0040689E   .  2BCE                SUB ECX,ESI                 ;  ecx 减去 esi，ecx=0f
004068A0   .  BD 15D30800         MOV EBP,8D315               ;  把 8D315 的值放到 ebp=8D315
004068A5   .  D3E8                SHR EAX,CL                  ;  逻辑右移 cl 位，输入注册码=932c，cl=0f=ecx 的低八位
004068A7   .  8BCE                MOV ECX,ESI                 ;  把 esi 的值放入 ecx 中，esi=11
004068A9   .  BE 0F000000         MOV ESI,0F                  ;  esi=0f
004068AE   .  D3E2                SHL EDX,CL                  ;  输入的注册码在逻辑左移动 cl 位，edx=5A40000  111
004068B0   .  0BC2                OR EAX,EDX                  ;  把 2 者在或运算 1111111111111
004068B2   .  2D 78DAEC00         SUB EAX,0ECDA78             ;  eax 减去 0ECDA78 V111111111
004068B7   .  99                  CDQ                         ;  edx 清零
004068B8   .  83E2 03             AND EDX,3                   ;  edx 和 3 与运算
004068BB   .  03C2                ADD EAX,EDX                 ;  edx 加 eax，edx=3
004068BD   .  8BC8                MOV ECX,EAX                 ;  把 eax 的值赋给 ecx，也即我们输入计算的数放在 ecx 中
004068BF   .  8BC7                MOV EAX,EDI                 ;  把 edi 的值赋给 eax，edi 存的是我们机器码的 16 进制
004068C1   .  99                  CDQ                         ;  edx 清零
004068C2   .  F7FE                IDIV ESI                    ;  eax 除 esi，商在 eax 余数在 edx 中
004068C4   .  8BC7                MOV EAX,EDI                 ;  继续把机器码放在 eax 中
004068C6   .  C1F9 02             SAR ECX,2                   ;  把我们输入得到的数逻辑右移 2 位
004068C9   .  81F1 FE0F0F0F       XOR ECX,0F0F0FFE            ;  把 ecx 和 0f0F0FFFE
004068CF   .  8BF2                MOV ESI,EDX                 ;  EDX 的值传给 ESI，esi=edx=5
004068D1   .  99                  CDQ                         ;  edx 清零
004068D2   .  69F6 60A49A04       IMUL ESI,ESI,49AA460        ;  49AA460*5
004068D8   .  F7FD                IDIV EBP                    ;  eax 除 ebp 商在 eax，余数在 edx
004068DA   .  2BF2                SUB ESI,EDX                 ;  esi=esi-edx
004068DC   .  03F7                ADD ESI,EDI                 ;  把机器码加上 esi
004068DE   .  81F6 6F3D6400       XOR ESI,643D6F
004068E4   .  F7D6                NOT ESI
004068E6   .  81E6 FFFFFF0F       AND ESI,0FFFFFFF
004068EC   .  3BF1                CMP ESI,ECX                 ;  esi 和 ecx 相等吗，相等就注册成功，不等就注册失败
004068EE   .  75 35             JNZ SHORT JPYQ.00406925       ;  爆破点
004068F0   .  6A 40             PUSH 40                       ; /Style = MB_OK|MB_ICONASTERISK|MB_APPLMODAL
004068F2   .  68 0C1A4100       PUSH JPYQ.00411A0C           ; |Title = "欢迎使用！"
004068F7   .  68 E4194100       PUSH JPYQ.004119E4           ; |Text = "注册成功！谢谢您对国产共享软件的支持。"
004068FC   .  53                  PUSH EBX                    ; |hOwner
004068FD   .  C705 70334100 010000>MOV DWORD PTR DS:[413370],1      ; |
00406907   .  FF15 BCE14000       CALL DWORD PTR DS:[<&USER32.MessageBoxA>>; \MessageBoxA
0040690D   >  6A 00             PUSH 0                       ; /Result = 0; Case 2 of switch 0040684A
0040690F   .  53                  PUSH EBX                    ; |hWnd
00406910   .  FF15 ECE14000       CALL DWORD PTR DS:[<&USER32.EndDialog>]  \EndDialog
00406916   .  5F                  POP EDI
00406917   .  5E                  POP ESI
00406918   .  5D                  POP EBP
00406919   .  B8 01000000         MOV EAX,1
0040691E   .  5B                  POP EBX
0040691F   .  83C4 08             ADD ESP,8
00406922   .  C2 1000             RETN 10
00406925   >  6A 40             PUSH 40                       ; /Style = MB_OK|MB_ICONASTERISK|MB_APPLMODAL
00406927   .  68 D4194100       PUSH JPYQ.004119D4           ; |Title = "请检查注册码"
0040692C   .  68 B8194100       PUSH JPYQ.004119B8           ; |Text = "注册失败！请检查注册码。"
00406931   .  53                  PUSH EBX                    ; |hOwner
00406932   .  FF15 BCE14000       CALL DWORD PTR DS:[<&USER32.MessageBoxA>>; \MessageBoxA
00406938   .  5F                  POP EDI
```

----------------------------------------------------

### 【总结】

总的算法是由机器码得到一个生成数，然后再由我们输入的数生成一个数，如果这两个数相等就注册成功。由机器码得到数的运算过程是：(我们设机器码是 `A`)

1. `A/0F` = 商 `B` 余数 `C`
2. 余数 `C*49AA460=D`
3. `A/8D315` = 商 `E` 余数 `F`
4. `Z=D-F+A`
5. `Z=Z XOR 643D6F`
6. `Z=NOT Z`
7. `Z= AND 0FFFFFFFF`

这个就是我们由机器码得到的一个数 `Z`。

我们输入的数它的运算过程如下：

设我们输入的数为 `H`

1. `H` 逻辑右移 `0f` 位得到一个数为 `I`
2. `H` 逻辑左移 `0F` 位得到一个数为 `J`
3. `I=I OR J`
4. `I=I-0ECDA78`
5. `I=I SAR 2`
6. `I=I XOR 0F0F0FFE` (这个就是由我们输入的数生成的数)

最后 `I=Z` 就注册成功 (我们可以逆向就可以得到注册码了，很累了，就到这里了)

初次写算法表达可能不好，错误之处在所难免，恳请指正。

----------------------------------------------------

`4dd466d0e1e0cc123afbe504da2462ca`
