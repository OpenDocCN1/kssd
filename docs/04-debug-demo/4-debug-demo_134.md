
# 从 IDA 中提取代码做汇编注册机之 C/C++ 篇

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-516.htm`](https://www.pediy.com/kssd/pediy08/pediy8-516.htm)

继上篇那个 `VB` 后，我也想做个 `C/C++` 类的例子，正好看到 `FreeSoul` 写的 `FScrackme2`，所以写了这篇文章。

`FScrackme2` 应该是在 `lunix` 下取了 `GNU` 中的 `GCC.exe` 和 `G++.exe` 来编译的一个 `C` 程序! 所以用 `PEID` 来看语言类型是 `MingWin32 GCC 3.x`。也就是 `GCC` 编译的 `C` 程序。

先来分析程序，找到真正的注册码并不难，`OD` 中的算法部分：

```
00401AE7   |.   8945 9C       MOV [LOCAL.25],EAX                    ; |

00401AEA   |.   837D A0 03    CMP [LOCAL.24],3                      ; |用户名长度必须大于 3 位

00401AEE   |.   0F8E C2000000 JLE <FScrackm.loc_401BB6>             ; |

00401AF4   |.   837D 9C 1A    CMP [LOCAL.25],1A                     ; |注册码长度必须大于 27 位

00401AF8   |.   0F8E B8000000 JLE <FScrackm.loc_401BB6>             ; |

...

00401B62   |.   8B45 9C       MOV EAX,[LOCAL.25]

00401B65   |.   894424 0C     MOV DWORD PTR SS:[ESP+C],EAX          ;  注册码长度

00401B69   |.   8B45 A0       MOV EAX,[LOCAL.24]

00401B6C   |.   894424 08     MOV DWORD PTR SS:[ESP+8],EAX          ;  用户名长度

00401B70   |.   8B45 A4       MOV EAX,[LOCAL.23]

00401B73   |.   894424 04     MOV DWORD PTR SS:[ESP+4],EAX          ;  注册码 ASCII

00401B77   |.   8B45 EC       MOV EAX,[LOCAL.5]

00401B7A   |.   890424        MOV DWORD PTR SS:[ESP],EAX            ;  用户名 ASCII

00401B7D   |.   E8 1C010000   CALL <FScrackm.sub_401C9E>           ;  主要算法

00401B82   |.   85C0          TEST EAX,EAX

00401B84   |.   75 18         JNZ SHORT <FScrackm.loc_401B9E>      ;  判断返回值是否为 0

\================================================================================================
```

`CALL` `<FScrackm.sub_401C9E>` 主要算法：

```
00401D33  >  > \8B45 0C       MOV EAX,DWORD PTR SS:[EBP+C]        

00401D36    .  8038 4E       CMP BYTE PTR DS:[EAX],4E             ;  对比第一位注册码是否为“N”

00401D39    .  74 0F         JE SHORT <FScrackm.loc_401D4A>

00401D3B    .  C785 4CFFFFFF>MOV DWORD PTR SS:[EBP-B4],1

00401D45    .  E9 7E070000   JMP <FScrackm.loc_4024C8>

00401D4A  >  >  8B45 0C       MOV EAX,DWORD PTR SS:[EBP+C]        

00401D4D    .  40            INC EAX

00401D4E    .  8038 61       CMP BYTE PTR DS:[EAX],61             ;  对比第二位注册码是否为“a”

00401D51    .  74 07         JE SHORT <FScrackm.loc_401D5A>

00401D53    .  C745 94 00000>MOV DWORD PTR SS:[EBP-6C],0

00401D5A  >  >  8B45 0C       MOV EAX,DWORD PTR SS:[EBP+C]        

00401D5D    .  83C0 02       ADD EAX,2

00401D60    .  8038 52       CMP BYTE PTR DS:[EAX],52             ;  对比第三位注册码是否为“R”

00401D63    .  74 0F         JE SHORT <FScrackm.loc_401D74>

00401D65    .  C785 4CFFFFFF>MOV DWORD PTR SS:[EBP-B4],1

00401D6F    .  E9 54070000   JMP <FScrackm.loc_4024C8>

00401D74  >  >  8B45 0C       MOV EAX,DWORD PTR SS:[EBP+C]        

00401D77    .  83C0 03       ADD EAX,3

00401D7A    .  8038 46       CMP BYTE PTR DS:[EAX],46             ;  对比第四位注册码是否为“F”

00401D7D    .  74 0F         JE SHORT <FScrackm.loc_401D8E>

00401D7F    .  C785 4CFFFFFF>MOV DWORD PTR SS:[EBP-B4],1

00401D89    .  E9 3A070000   JMP <FScrackm.loc_4024C8>

...

00401E8B    .  83C0 04       ADD EAX,4

00401E8E    .  0FB600        MOVZX EAX,BYTE PTR DS:[EAX]

00401E91    .  3A45 87       CMP AL,BYTE PTR SS:[EBP-79]          ;  对比第 5 位开始 5 位注册码

00401E94    .  74 0F         JE SHORT <FScrackm.loc_401EA5>

...

00401EEA    .  83C1 09       ADD ECX,9

00401EF6    .  0FB601        MOVZX EAX,BYTE PTR DS:[ECX]

00401EF9    .  3A02          CMP AL,BYTE PTR DS:[EDX]             ;  对比第 10 位注册码

00401EFB    .  74 0F         JE SHORT <FScrackm.loc_401F0C>

...

00401F4A    .  83C2 0A       ADD EDX,0A

00401F4D    .  8B85 70FFFFFF MOV EAX,DWORD PTR SS:[EBP-90]       ;  对比第 11 位注册码...

00401F8B    .  E8 30420000   CALL <FScrackm.isalpha>             ; 第 12 位后 6 位注册码，必须为字母

..

00401FB5    .  E8 F6410000   CALL <FScrackm.isupper>             ; 第 12 位后 6 位注册码，必须大写

...

0040205F    .  83C0 0A       ADD EAX,0A                           ;  第 10 位后

00402075    .  0FBEC0        MOVSX EAX,AL

00402078    .  39C1          CMP ECX,EAX                          ;  对比第 11 位开始 3 位的注册码

0040207A    .  74 0F         JE SHORT <FScrackm.loc_40208B>

...

004020B1    .  0FBEC0        MOVSX EAX,AL

004020B4    .  39C1          CMP ECX,EAX                          ;  对比第 17 位开始倒数 3 位的注册码

004020B6    .  74 44         JE SHORT <FScrackm.loc_4020FC>

...

00402125    .  83C0 11       ADD EAX,11                           ; 第 17 位后

00402128    .  0FBE00        MOVSX EAX,BYTE PTR DS:[EAX]         

0040212B    .  890424        MOV DWORD PTR SS:[ESP],EAX          

0040212E    .  E8 6D400000   CALL <FScrackm.isdigit>             ; \第 18 位开始的 6 位必须为数字

00402133    .  85C0          TEST EAX,EAX

00402135    .  75 0F         JNZ SHORT <FScrackm.loc_402146>

...

0040220D    .  0FB601        MOVZX EAX,BYTE PTR DS:[ECX]

00402210    .  3A02          CMP AL,BYTE PTR DS:[EDX]             ;  对比第 18 位后的 6 位注册码

00402212    .  74 0F         JE SHORT <FScrackm.loc_402223>

...

00402230    .  83C0 17       ADD EAX,17

00402233    .  8038 2D       CMP BYTE PTR DS:[EAX],2D             ;  对比第 24 位注册码，必须为“-”号

00402236    .  74 07         JE SHORT <FScrackm.loc_40223F>

...

004022C7    .  83C1 18       ADD ECX,18

004022CA    .  8B85 58FFFFFF MOV EAX,DWORD PTR SS:[EBP-A8]

004022D0    .  0FBE10        MOVSX EDX,BYTE PTR DS:[EAX]

004022D3    .  0FB601        MOVZX EAX,BYTE PTR DS:[ECX]

004022D6    .  3A842A 78FFFF>CMP AL,BYTE PTR DS:[EDX+EBP-88]      ;  对比第 25 位注册码

004022DD    .  74 0F         JE SHORT <FScrackm.loc_4022EE>

...

00402474    .  83C2 19       ADD EDX,19

00402477    .  8B85 50FFFFFF MOV EAX,DWORD PTR SS:[EBP-B0]

0040247D    .  3802          CMP BYTE PTR DS:[EDX],AL             ;  对比第 26 位注册码

0040247F    .  74 07         JE SHORT <FScrackm.loc_402488>

00402481    .  C745 94 00000>MOV DWORD PTR SS:[EBP-6C],0

00402488  >  >  8B45 0C       MOV EAX,DWORD PTR SS:[EBP+C]        

0040248B    .  83C0 1A       ADD EAX,1A

0040248E    .  8038 44       CMP BYTE PTR DS:[EAX],44             ;  对比第 27 位注册码是否为“D”

00402491    .  74 07         JE SHORT <FScrackm.loc_40249A>

...

0040249A  >  > \837D 14 1B    CMP DWORD PTR SS:[EBP+14],1B          ;  对比注册码长度是否为 27 位

0040249E    .  74 0C         JE SHORT <FScrackm.loc_4024AC>

\==================================================================================================
```

我们主要是讨论写注册机，所以算法具体不去讨论。我们现在所知道的是，用户名要大于三位，注册码必须为 27 位，而且我们知道了每个注册码比较的地方。`CALL 00401C9E` 里面是具体的算法，里面还有很多子函数，想提取这样一个含有很多子程序的代码到注册机里在 `OD` 中好象没有好的方法，只有手动提，而在 `IDA` 中就有办法，这得益于最早由 `dreaman` 写的提取函数的脚本，我称它为 `GetCall.IDC`（`http://bbs.pediy.com/showthread.php?s=&threadid=23231`），在 `IDA` 中，只要将光标定位于一个函数的起始代码，运行脚本后，它会截取所有该函数的代码，包括里面所有的子函数代码，并在文件目录保存汇编代码为一个 `ASM` 文件。真是太方便了。让我们行动。

`IDA` 反汇编 `FScrackme2`，“`G`”到 `401C9E`，菜单“文件”-“`IDC` 文件”，运行脚本。`FScrackme2` 目录中已经生成了 `FScrackme2Part.asm`。但是在这之前，对于这个软件，需要在 `IDA` 中动动手脚。因为软件是 `GCC` 写的，`IDA` 的脚本 `gcc32rtf.sig` 自动把 `GCC` 的库函数识别出来了：

```
.text:00401DCF          call    __Znaj
```

这本来是个好事，我也想找 `GCC` 的运行库，然后将它转换成 `MASM` 的库文件，这样，这些 `CALL` 就可以直接在 `MASM` 中引用了，写注册机就非常方便了。但是找不到这样的运行库。而 `GetCall.IDC` 脚本只要碰到被 `IDA` 识别出来的函数，它就不进去拷里面的子函数，所以，解决方法只有一个，将 `IDA` 中的 `SIG` 文件夹下的 `gcc32rtf.sig` 文件删除，再让 `IDA` 反汇编 `FScrackme2`，再拷 `CALL 401C9E` 里的代码。这样拷出来的子函数就比较全了。（当然我也把所有的 `GCC` 的库函数提取出来做个备份，谁需要的话跟我联系。）

现在就已经把代码全部拷出到 `FScrackme2Part.asm` 了，我们不管三七二十一先把这段汇编代码拷到一个汇编注册机模板里再说，就是上次那个模板，当然这样在 `RADASM` 中运行，错误会非常多，有错就改嘛，中国人的好作风。

因为是 `C` 语言类的，所以不可避免的用到 `C` 运行库 `msvcrt.dll`。所以看到汇编代码中很多对 `msvcrt` 的调用，比如：

```
.text:00401F8B          call    isalpha

.text:00401D14          call    isalnum
```

先解决这个问题。有了上篇的文章，这已经不是问题了，用 `Dll2inc` 将 `msvcrt.dll` 转换为 `MASM` 的 `msvcrt.lib` 和 `msvcrt.inc` 文件，然后在汇编代码中调用这两个文件。这样就可以直接在汇编中调用 `msvcrt` 的库函数了。



# 使用 RADASM 和 IDA 修复与修改代码

然后继续让 `RADASM` 找出错误，一步一步来解决。一般的方法是将 `RADASM` 中的错误提示地址复制出来，在 `IDA` 中 `G` 到该地址，看数据是属于哪个段的：
- 如果是 `.text` 可执行代码段的子函数，就用 `GetCall` 脚本复制出来粘贴到汇编代码区。
- 如果是 `.bss`、`.rdata`、`.data` 等数据段，就拷贝到汇编代码的 `.DATA` 区。

`LINUX` 中还会出现 `.stab`、`.stabstr`、`.comment`、`.note`、`.shstrtab`、`.symtab`、`.strtab`，不过这些都没用。我想在下篇结束篇专门写如何从 `IDA` 中拷代码，结合具体实例讲的详细些。在这里，拷好的代码就看附件中的注册机代码。

## 仿制与注册机思路

拷好并修改好后，这俨然是用汇编代码仿制 `FScrackme2`，但是文件大小比原来小了 N 倍，执行速度也快了不少。

做成个仿制 `FScrackme2` 还不行，我们是要做它的注册机。

很多 `crackme` 都可以通过用户名算出个结果，然后将注册码也通过运算得到这个结果，那就可以通过注册码算法的逆运算推出真实的注册码。

但是这个 `crackme` 是个正向的，它就是通过运算得到一个个注册码，然后将输入的注册码也一一对比。

对于这种 `crackme` 写注册机时我提供一种解决思路，就是制造一个“伪注册码”，然后将程序里算出的一个个真实的注册码重新存到一个地方，这个地方的字符串就是真的注册码了。

## 具体实现步骤

### 1. 定义伪注册码

首先说伪注册码，只要符合条件的任何字符都行。我这里为了减少弯路，根据软件的要求：第 12 到 17 位必须为大写字母，第 18 位和第 24 位必须为数字，我提供个名称为 `TEMP` 的伪注册码：`12345678901ABCDEF123456789D`。

```asm
.data
Temp            db '12345678901ABCDEF123456789D', 0
```

我将一个个真实的注册码存到 `SerialBuffer` 中：

```asm
invoke SetDlgItemText, hDlg, IDC_SERIAL, addr SerialBuffer
```

### 2. 修改对比与跳转逻辑

其次要将所有对比注册码的地方都改为移动真实的注册码到 `SerialBuffer`，并且将那些对比发生错误跳转到退出的地方都“爆掉”，相当于先爆破再复制代码：

```asm
cmp  byte ptr [eax],  4Eh                     ; 对比第一位注册码是否为“N”
mov    [SerialBuffer+0], 4Eh                  ; 将注册码第一位“N”写回
jmp  short loc_401D4A                          ; 避免退出，原来是 je 401D4A
```

再次要注意原来程序中循环取输入的注册码进行对比的地方，我们要将它改为循环存储到 `SerialBuffer` 中，这就需要个累计参数，可以利用原程序中循环的时候对比是否循环完的那个参数，用那个非常爽：

```asm
mov  [ebp+var_8C], 1
…
cmp  [ebp+var_8C], 6                            ; 是否取完
….
mov  al,  [edx]                                 ; 对比第 18 位后的 6 位注册码
xor     ecx, ecx
mov     cl,  byte ptr [ebp+var_8C]             ; 循环中的对比参数
mov     [SerialBuffer+ecx+10h], al              ; 写回第 18 位后的 6 位注册码
jmp  short loc_402223                            ; 避免退出
mov  [ebp+var_B4], 1
jmp  loc_4024C8                                 ; 跳到退出
…
lea  eax, [ebp+var_8C]
inc  dword ptr [eax]                             ; 累计
```

### 3. 替换计算逻辑

最后还有一点，就是用前面已经计算出来的正确注册码来推算后面部分的注册码，这时，不能用伪注册码来推算了，因为它本身是错误的，所以推算出来的后面部分也是错的，要替换成真实的注册码，如：

```asm
mov  eax, [ebp+arg_4]  ; 注意这里，这里是将已算出的注册码第 23 位拿来运算，所以要改成：
mov  eax, offset SerialBuffer  ; 将已经算出的注册码放到 EAX，当然也包括已算出的第 23 位
mov  [esp+0C8h+var_C8], eax
call  sub_4024FE
```

### 4. 替换输入获取过程

然后我们来看看原来程序取用户名和注册码的过程，我们可以替换成取用户名和伪注册码：

```asm
00401B65  .  894424 0C    MOV DWORD PTR SS:[ESP+C],EAX         ;  注册码长度
00401B69  .  8B45 A0      MOV EAX,[LOCAL.24]
00401B6C  .  894424 08    MOV DWORD PTR SS:[ESP+8],EAX         ;  用户名长度
00401B70  .  8B45 A4      MOV EAX,[LOCAL.23]
00401B73  .  894424 04    MOV DWORD PTR SS:[ESP+4],EAX         ;  注册码 ASCII
00401B77  .  8B45 EC      MOV EAX,[LOCAL.5]
00401B7A  .  890424       MOV DWORD PTR SS:[ESP],EAX           ;  用户名 ASCII
00401B7D  .  E8 1C010000  CALL <FScrackm.sub_401C9E>          ;  主要算法
00401B82  .  85C0         TEST EAX,EAX
00401B84  .  75 18        JNZ SHORT <FScrackm.loc_401B9E>      ;  判断返回值是否为 0
```

只要改为：

```asm
invoke lstrlen, addr Temp                            ; 取伪注册码长度
mov  [esp+98h+var_8C], eax
invoke lstrlen, addr NameBuffer                       ; 取用户名长度
mov  [esp+98h+var_90], eax
mov  [esp+98h+var_94], offset Temp                   ; 装入伪注册码
mov  [esp+98h+var_98], offset NameBuffer             ; 装入用户名
call  sub_401C9E                                      ; 调用算法 CALL
test  eax, eax
jnz  loc_401B9E
```

我们还可以利用原来注册码错误的对话框的地方来提示对用户名各种限制，比如要求第一位和最后一位必须是字母等。

这样弄好后，到最后一位注册码算好，所有的真实注册码也就已经全部存到 `SerialBuffer` 中了。

如果还有错误，最好的办法就是在 `RADASM` 中用 `OD` 调试跟踪错误。

## 总结

注册机的源代码我已经放在附件里，最好结合原程序看看。注册机里我已经做了比较详细的注释了。

注册机中的全部代码都拷自 `FScrackme2` 在 `IDA` 中反汇编后的代码，也就是几乎全部是算法函数 `CALL 401C9E` 里的代码。我尽量保持“原汁原味”，虽然有些代码完全可以删除，但是直接拷贝比判断该删除哪些要好。

主要就是加入了存储真实注册码的过程，看看每个注册码的存储方法，慢慢体会。

**附件**：单击下载

**【版权声明】**: 本文原创于看雪技术论坛，转载请注明作者并保持文章的完整，谢谢！

2006 年 12 月 24 日
`4dd466d0e1e0cc123afbe504da2462ca`
