
下面还会跟据三个密钥生成一个15 byte的表:

---

```asm
004029BD   .  8B5C2>mov ebx,dword ptr ss:[esp+20]         ;  密钥生成完毕,到表的生成
004029C1   .  DDD8  fstp st                               ;  st0出栈
004029C3   >  33C0  xor eax,eax                           ;  清零,第一段表开始
004029C5   >  8BC8  mov ecx,eax                           ;  前5位表,累加值存入ecx
004029C7   .  8B742>mov esi,dword ptr ss:[esp+14]         ;  取浮点运算结果HEX放到esi中
004029CB   .  0FAFC>imul ecx,eax                          ;  循环相乘
004029CE   .  0FAFC>imul ecx,eax                          ;  再乘一次
004029D1   .  8D4C3>lea ecx,dword ptr ds:[ecx+esi+1F]     ;  esi=Key1,累加+0x1F
004029D5   .  81E1 >and ecx,8000007F                      ;  与运算and 0x800007F
004029DB   .  79 05 jns short 038版本?004029E2            ;  跳了
004029DD   .  49    dec ecx
004029DE   .  83C9 >or ecx,FFFFFF80
004029E1   .  41    inc ecx
004029E2   >  884C0>mov byte ptr ss:[esp+eax+2C],cl       ;  保存到表的第一段中★
004029E6   .  40    inc eax                               ;  eax递增
004029E7   .  83F8 >cmp eax,5                             ;  少于5?
004029EA   .^ 7C D9 jl short 038版本?004029C5             ;  小于5则继续循环
004029EC   .  B8 05>mov eax,5                             ;  第二段表开始
004029F1   >  8BC8  mov ecx,eax                           ;  以下同上
004029F3   .  0FAFC>imul ecx,eax
004029F6   .  0FAFC>imul ecx,eax
004029F9   .  8D4C2>lea ecx,dword ptr ds:[ecx+ebp+1F]     ;  ebp=Key2,其余同上
004029FD   .  81E1 >and ecx,8000007F
00402A03   .  79 05 jns short 038版本?00402A0A            ;  跳了
00402A05   .  49    dec ecx
00402A06   .  83C9 >or ecx,FFFFFF80
00402A09   .  41    inc ecx
00402A0A   >  884C0>mov byte ptr ss:[esp+eax+2C],cl       ;  保存到表的第二段中★
00402A0E   .  40    inc eax
00402A0F   .  83F8 >cmp eax,0A                            ;  到10了没?
00402A12   .^ 7C DD jl short 038版本?004029F1             ;  未完继续
00402A14   .  B8 0A>mov eax,0A                            ;  第三段表开始
00402A19   >  8BC8  mov ecx,eax                           ;  以下同上
00402A1B   .  0FAFC>imul ecx,eax
00402A1E   .  0FAFC>imul ecx,eax
00402A21   .  8D4C1>lea ecx,dword ptr ds:[ecx+edx+1F]     ;  edx=Key3,其余同上
00402A25   .  81E1 >and ecx,8000007F
00402A2B   .  79 05 jns short 038版本?00402A32            ;  跳了
00402A2D   .  49    dec ecx
00402A2E   .  83C9 >or ecx,FFFFFF80
00402A31   .  41    inc ecx
00402A32   >  884C0>mov byte ptr ss:[esp+eax+2C],cl       ;  保存到表的第三段中★
00402A36   .  40    inc eax
00402A37   .  83F8 >cmp eax,0F                            ;  到15了没?
00402A3A   .^ 7C DD jl short 038版本?00402A19             ;  未完继续
```

上面分三次生成了一个15 byte的表,我这里表的数据如下

```text
0012F56C  12 13 1A 2D 52 24 7F 7E 27 00 1E 69 76 4B 6E     -R$~'.ivKn
```

即:

表1

`0x12` `0x13` `0x1A` `0x2D` `0x52`

表2

`0x24` `0x7F` `0x7E` `0x27` `0x00`

表3

`0x1E` `0x69` `0x76` `0x4B` `0x6E`

---

```asm
00402A3C   .  33F6  xor esi,esi                           ;  表已生成完毕,开始计算注册码
00402A3E   .  C6442>mov byte ptr ss:[esp+3B],0
00402A43   .  33C9  xor ecx,ecx
00402A45   >  8A443>mov al,byte ptr ss:[esp+esi+2C]       ;  取表中第N位的低8位到al
00402A49   .  3C 30 cmp al,30                             ;  '0'
00402A4B   .  7C 04 jl short 038版本?00402A51
00402A4D   .  3C 39 cmp al,39                             ;  '9'
00402A4F   .  7E 29 jle short 038版本?00402A7A            ;  非可显字符则跳
00402A51   >  3C 41 cmp al,41                             ;  'a'
00402A53   .  7C 04 jl short 038版本?00402A59
00402A55   .  3C 5A cmp al,5A                             ;  'z'
00402A57   .  7E 21 jle short 038版本?00402A7A            ;  非可显字符则跳
00402A59   >  3C 61 cmp al,61                             ;  'A'
00402A5B   .  7C 04 jl short 038版本?00402A61
00402A5D   .  3C 7A cmp al,7A                             ;  'Z'
00402A5F   .  7E 19 jle short 038版本?00402A7A            ;  非可显字符则跳
00402A61   >  0FBED>movsx edx,al                          ;  表中第N位放到edx
00402A64   .  8D440>lea eax,dword ptr ds:[edx+ecx+1F]     ;  edx=表第N位ASCII,相加结果放到eax中
00402A68   .  25 7F>and eax,8000007F                      ;  与运算and 0x800007F
00402A6D   .  79 05 jns short 038版本?00402A74            ;  跳了
00402A6F   .  48    dec eax
00402A70   .  83C8 >or eax,FFFFFF80
00402A73   .  40    inc eax
00402A74   >  88443>mov byte ptr ss:[esp+esi+2C],al       ;  保存注册码第N位
00402A78   .^ EB CB jmp short 038版本?00402A45            ;  继续下一个
00402A7A   >  83C1 >add ecx,7                             ;  非可显字符ASCII+0x7
00402A7D   .  46    inc esi                               ;  记数器加1
00402A7E   .  83F9 >cmp ecx,69                            ;  0x7\*0x0F=0x69,是否循环完了15次?
00402A81   .^ 7C C2 jl short 038版本?00402A45             ;  未完继续,完了向下
```

上面这段是根据之前的表生成0-9,a-z,A-Z段的注册码,以便显示出来

到这里已经可以看到注册码的原始状态,我这里显示的注册码原始状态是

```text
0012F56C  31 39 47 61 52 66 48 4E 55 78 68 69 76 4B 6E     19GaRfHNUxhivKn
```

下面继续看看还做了些什么

---

```asm
00402A83   .  8D4C2>lea ecx,dword ptr ss:[esp+2C]         ;  保存得到的15位注册码指针到ecx
00402A87   .  8D542>lea edx,dword ptr ss:[esp+10]         ;  空指针
00402A8B   .  51    push ecx                              ;  注册码入栈
00402A8C   .  68 C8>push 038版本?0041F0C8                 ;  ASCII "%s"
00402A91   .  52    push edx
00402A92   .  E8 66>call 038版本?00410CFD                 ;  计算注册码长度=0x0F
00402A97   .  8B742>mov esi,dword ptr ss:[esp+1C]         ;  注册码指针保存到esi
00402A9B   .  83C4 >add esp,0C
00402A9E   >  8B46 >mov eax,dword ptr ds:[esi-8]          ;  保存长度
00402AA1   .  33C9  xor ecx,ecx
00402AA3   .  85C0  test eax,eax
00402AA5   .  7E 2D jle short 038版本?00402AD4            ;  小于等于0x0F则over
00402AA7   >  8D79 >lea edi,dword ptr ds:[ecx+1]          ;  记数器
00402AAA   .  BD 06>mov ebp,6
00402AAF   .  8BC7  mov eax,edi
00402AB1   .  99    cdq
00402AB2   .  F7FD  idiv ebp                              ;  记数器余0x6
00402AB4   .  85D2  test edx,edx                          ;  是否0,其实这里是判断注册码第6和第12位
00402AB6   .  75 13 jnz short 038版本?00402ACB            ;  不是则跳
00402AB8   .  68 7C>push 038版本?0041F37C                 ;  是第6位或第12位了
00402ABD   .  51    push ecx
00402ABE   .  8D4C2>lea ecx,dword ptr ss:[esp+18]
00402AC2   .  E8 E3>call 038版本?004108AA                 ;  此CALL是往第6和第12位添加"-"
00402AC7   .  8B742>mov esi,dword ptr ss:[esp+10]
00402ACB   >  8B46 >mov eax,dword ptr ds:[esi-8]          ;  注册码长度
00402ACE   .  8BCF  mov ecx,edi                           ;  记数器保存到ecx
00402AD0   .  3BC8  cmp ecx,eax                           ;  是否完了15次?
00402AD2   .^ 7C D3 jl short 038版本?00402AA7             ;  未完继续
```

上面这部分是往原始注册码的第6和第12位添加"-"

添加"-"完成后,注册码就出来了,我这里的注册码是

```text
ESP 0012F540
EBP 00000006
ESI 003E6EE8 ASCII "19GaR-fHNUx-hivKn"
EDI 00000011
```

完整注册码生成完毕,下面开始比较注册码了

---

```asm
00402AD4   >  8B83 >mov eax,dword ptr ds:[ebx+18C]
00402ADA   .  8B48 >mov ecx,dword ptr ds:[eax-8]
00402ADD   .  85C9  test ecx,ecx                          ;  判断注册码长度
00402ADF   .  75 18 jnz short 038版本?00402AF9            ;  非空则继续
00402AE1   .  6A 10 push 10
00402AE3   .  68 70>push 038版本?0041F370
00402AE8   .  68 5C>push 038版本?0041F35C
00402AED   .  8BCB  mov ecx,ebx
00402AEF   .  E8 EB>call 038版本?004145DF                 ;  弹出"注册码不能为空"出错提示
00402AF4   .  E9 20>jmp 038版本?00402D19                  ;  over
00402AF9   >  56    push esi                              ;  注册码指针,这里可以作内存注册机
00402AFA   .  50    push eax
00402AFB   .  E8 A3>call 038版本?004039A3                 ;  判断注册码是否合法CALL
00402B00   .  83C4 >add esp,8
00402B03   .  85C0  test eax,eax
00402B05   .  0F85 >jnz 038版本?00402D11                  ;  暴破点,跳则over
00402B0B   .  8D83 >lea eax,dword ptr ds:[ebx+188]
00402B11   .  68 50>push 038版本?0041F350
00402B16   .  50    push eax
00402B17   .  8D442>lea eax,dword ptr ss:[esp+24]
```

## 最后小结

到此,整个注册码生成过程已经了解清楚了,可以开始写注册机了.这里用C写了个注册机,根据上面分析的结果来写的,没有经过优化,写得比较乱,见笑了:)

注册机源码如下(TC 2.0调试通过):

## 注册机

```c
#include <stdio.h>

#include <math.h>

#include <string.h>

double Call1(double inDbl)

{

/\* 求对数子函数 \*/

double tmpCallDbl;
```
