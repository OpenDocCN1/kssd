
# 某软件算法与 Delphi 注册机代码实现

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-255.htm`](https://www.pediy.com/kssd/pediy08/pediy8-255.htm)

**【文章标题】**: 某软件算法与 Delphi 注册机代码实现  
**【作者】**: testkey  
**【邮箱】**: pediytest@sina.com  
**【主页】**: http://bbs.pediy.com  
**【软件名称】**: 省略  
**【下载地址】**: 无  
**【加壳方式】**: 无  
**【保护方式】**: 随机 `ID`、密码  
**【工具】**: `OD`, `Delphi7`  
**【软件介绍】**: 无  
**【作者声明】**: 仅做分析研究，无商业用途。

--------------------------------------------------------------------------------

## 详细过程

某软件算法分析

为避免版权信息，隐去软件名称和下载地址。仅做分析算法和学习使用。

程序启动后出现 `ID`，需要输入密码才能进入程序。

我们可以从启动跟踪，或者从出错的提示入手分析，确定核心代码：

找核心的地方略过，我们直接从程序启动验证密码是否正确入手，进行算法分析：

```asm
00542AB6  |. E8 B96BECFF   call    00409674
00542ABB  |. 8B55 C0       mov     edx, [ebp-40]                 ;  ID 的值
00542ABE  |. 8B86 EC020000 mov     eax, [esi+2EC]
00542AC4  |. E8 734BEFFF   call    0043763C
00542AC9  |. A1 F8775400   mov     eax, [5477F8]
00542ACE  |. 8918          mov     [eax], ebx                    ;  计算 1，取 ID 为 10 进制数据,转换为 16 进制数据
00542AD0  |. B8 8F5F0100   mov     eax, 15F8F                    ;  初始化 EAX=$15F8F
00542AD5  |. E8 A601ECFF   call    00402C80                      ;  进行数据处理，跟进计算 2
00542ADA  |. 05 10270000   add     eax, 2710                     ;  返回结果+$2710
00542ADF  |. 8BD8          mov     ebx, eax
00542AE1  |. 03DB          add     ebx, ebx
00542AE3  |. 8D1C9B        lea     ebx, [ebx+ebx*4]
00542AE6  |. 8BC3          mov     eax, ebx                      ;  相当于 EBX 结果*$A(以上看为计算 3)
00542AE8  |. E8 E3FDFFFF   call    005428D0                      ;  对上面结果进行再处理计算 4
00542AED  |. 83C3 09       add     ebx, 9                        ;  结果+9
00542AF0  |. 2BD8          sub     ebx, eax                      ;  减去求余处理的结果 EAX(这两步为计算 5)
00542AF2  |. 8D55 BC       lea     edx, [ebp-44]
00542AF5  |. 8BC3          mov     eax, ebx                      ;  结果为正确密码
```

**计算 1 的 Delphi 代码：**

```delphi
a := StrToInt(id.Text);
```

### 2、跟进 00542AD5 `call 00402C80` 计算 2

```asm
00402C80  /$ 6915 44805400>imul    edx, [548044], 8088405       ;  EDX=0012FDD4*机器码*$8088405
00402C8A  |. 42            inc     edx                           ;  计算结果+1
00402C8B  |. 8915 44805400 mov     [548044], edx
00402C91  |. F7E2          mul     edx                           ;  EAX(15F8F)*EDX 结果的低 8 位给 EAX，高 8 位给 EDX
00402C93  |. 89D0          mov     eax, edx                      ;  EDX 给 EAX 返回数据
00402C95  \. C3            retn
```

**计算 2 的 Delphi 代码：**

```delphi
asm
  mov edx,$0012FDD4
  imul edx,a,$8088405
  inc edx
  mov eax,$15f8f
  mul edx
  mov &b,edx
end;
```

### 3、计算 3

```asm
00542ADA  |. 05 10270000   add     eax, 2710                     ;  返回结果+$2710
00542ADF  |. 8BD8          mov     ebx, eax
00542AE1  |. 03DB          add     ebx, ebx
00542AE3  |. 8D1C9B        lea     ebx, [ebx+ebx*4]
00542AE6  |. 8BC3          mov     eax, ebx                      ;  相当于 EBX 结果*$A(以上看为计算 3)
```

**计算 3 的 Delphi 代码：**

```delphi
b := (b + $2710) * $A;
```

### 4、跟进 00542AE8 `call 005428D0` 计算 4

```asm
005428D0  /$ 53            push    ebx
005428D1  |. 56            push    esi
005428D2  |. 57            push    edi
005428D3  |. 8BC8          mov     ecx, eax
005428D5  |. 33F6          xor     esi, esi
005428D7  |. BB 01000000   mov     ebx, 1
005428DC  |. 85C9          test    ecx, ecx
005428DE  |. 7E 20         jle     short 00542900
005428E0  |> 8BC1          /mov     eax, ecx                    ;  计算的结果(第 2 次开始为新得的商)
005428E2  |. BF 0A000000   |mov     edi, 0A                     ;  对$A 求余
005428E7  |. 99            |cdq
005428E8  |. F7FF          |idiv    edi                         ;  对$A 求余
005428EA  |. 0FAFD3        |imul    edx, ebx                    ;  余数*循环的次数
005428ED  |. 03F2          |add     esi, edx                    ;  余数处理结果累加
005428EF  |. 43            |inc     ebx
005428F0  |. 8BC1          |mov     eax, ecx                    ;  计算的结果
005428F2  |. B9 0A000000   |mov     ecx, 0A
005428F7  |. 99            |cdq
005428F8  |. F7F9          |idiv    ecx                         ;  对$A 整除
005428FA  |. 8BC8          |mov     ecx, eax                    ;  把商作为新的计算结果，循环计算
005428FC  |. 85C9          |test    ecx, ecx                    ;  商是否为 0，不为零则把得到的 ECX 继续处理
005428FE  |.^ 7F E0        \jg     short 005428E0
00542900  |> 8BC6          mov     eax, esi                    ;  上面累加结果
00542902  |. B9 09000000   mov     ecx, 9
00542907  |. 99            cdq
00542908  |. F7F9          idiv    ecx                         ;  对累加结果再进行处理对 9 求余
0054290A  |. 8BC2          mov     eax, edx                    ;  结果返回
0054290C  |. 5F            pop     edi
0054290D  |. 5E            pop     esi
0054290E  |. 5B            pop     ebx
0054290F  \. C3            retn
```

**计算 4 的 Delphi 代码：**

```delphi
c := 1;
e := 0;
for i := 1 to 8 do
begin
  d := (b mod $a) * i;
  e := e + d;
  b := b div $A;
end;
```

### 5、计算 5

```asm
00542AED  |. 83C3 09       add     ebx, 9                      ;  结果+9
00542AF0  |. 2BD8          sub     ebx, eax                      ;  减去求余处理的结果 EAX(这两步为计算 5)
```

**计算 5 的 Delphi 代码：**

```delphi
e := e mod 9;
f := f + 9 - e;
password1.Text := IntToStr(f);
```

--------------------------------------------------------------------------------

## 总结

算法很简单，适合新手学习。

练习 Delphi 中内嵌汇编代码。

--------------------------------------------------------------------------------

**【版权声明】**: 转载请注明文章出自 http://www.pediy.com

本代码的着色效果由 xTiNt 自动完成

`4dd466d0e1e0cc123afbe504da2462ca`
