
# Resource Explorer

> 原文：[`www.pediy.com/kssd/pediy02/forum150.htm`](https://www.pediy.com/kssd/pediy02/forum150.htm)

[`http://batry.hypermart.net/DFMV_1e.htm`](http://batry.hypermart.net/DFMV_1e.htm)  
[`http://www.abgc.homecreatures.com/September1999/xOtix/vcl_30.zip`](http://www.abgc.homecreatures.com/September1999/xOtix/vcl_30.zip)

**Registration Key:** `35005259-dr0`

**标题:** 用了这个注册码，虽然显示已注册，但是打开一个 `delphi` 程序后，选择 `save resource`，它仍然会提示只有注册用户才能使用此功能。

**内容:**

下面的这个比较是有用的，虽然不影响它在 `About` 对话框里显示 `Registered to:XXXX`，但是影响 `Save Resource`。

```
015F:0040F140  MOV       EDX,[EBP-0C]  //正确
015F:0040F143  POP       EAX           //错误
015F:0040F144  CALL      004011B0
015F:0040F149  SETZ      BL
```

以 `ResXplor3.exe` 为例，其它类似。

输入注册码之后用 `hmemcpy` 作断点，中断后按 12 次 `F12`，就到这里，是在 `vcl30.dpl` 中。执行 `call` 之前敲 `d edx` 可以看见自己输入的注册码，于是用 `BPR edx edx+len-1` 设断点监视假注册码（`len` 为注册码长度）。`BPR` 断点是很好用的，可惜在 `NT` 上好象没有。

```
015F:4137BEBB  MOV       EDX,[EBP-28]  //假注册码
015F:4137BEBE  MOV       EAX,[EBP-08]
015F:4137BEC1  CALL      41304754
015F:4137BEC6  MOV       BYTE PTR [EBP-09],01
015F:4137BECA  XOR       EAX,EAX
```

然后按 `F5` 执行，就到这里停下来。还是在 `DLL` 中。在执行 `REPNZ SCASB` 那条指令的时候看一下 `AL` 寄存器中的值为 `2D`，这说明它试图在你输入的假注册码中匹配横杠字符（减号），由此可知注册码是含有横杠的。实际上下面的代码就对应 `C` 语言中的 `strstr( )` 函数，即检查一个串是否为另一个串的子串。

```
015F:41304CA9  MOV       AL,[ESI]
015F:41304CAB  INC       ESI
015F:41304CAC  SUB       ECX,EDX
015F:41304CAE  JLE       41304CC4
015F:41304CB0  REPNZ SCASB
015F:41304CB2  JNZ       41304CC4
015F:41304CB4  MOV       EBX,ECX
015F:41304CB6  PUSH      ESI
015F:41304CB7  PUSH      EDI
015F:41304CB8  MOV       ECX,EDX
015F:41304CBA  REPZ CMPSB
015F:41304CBC  POP       EDI
015F:41304CBD  POP       ESI
015F:41304CBE  JZ        41304CCC
015F:41304CC0  MOV       ECX,EBX
015F:41304CC2  JMP       41304CB0
015F:41304CC4  POP       EDX
015F:41304CC5  XOR       EAX,EAX
015F:41304CC7  JMP       41304CD1
015F:41304CC9  XOR       EAX,EAX
015F:41304CCB  RET
015F:41304CCC  POP       EDX
015F:41304CCD  MOV       EAX,EDI
015F:41304CCF  SUB       EAX,EDX
015F:41304CD1  POP       EDI
015F:41304CD2  POP       ESI
015F:41304CD3  POP       EBX
015F:41304CD4  RET
```

在假注册码中加入一个横杠字符，重复上面的步骤，执行完上面的 `RET` 指令后，按 `F10` 或者直接向下翻屏就看见一段比较：

```
015F:0040EF5A  MOV       EAX,[EBP-10]
015F:0040EF5D  LEA       ECX,[EBP-08]
015F:0040EF60  MOV       EDX,00000003  //3 个字符
015F:0040EF65  CALL      0040EE10
015F:0040EF6A  MOV       EAX,[EBP-0C]
015F:0040EF6D  MOV       AL,[EAX]
015F:0040EF6F  MOV       EDX,[EBP-08]
015F:0040EF72  CMP       AL,[EDX]      //比较注册码的首字符
015F:0040EF74  JNZ       0040EFA0
015F:0040EF76  MOV       EAX,[EBP-0C]
015F:0040EF79  MOV       AL,[EBX+EAX-01]
015F:0040EF7D  MOV       EDX,[EBP-08]
015F:0040EF80  CMP       AL,[EDX+02] //比较横杠前的那个字符
015F:0040EF83  JNZ       0040EFA0
015F:0040EF85  MOV       EAX,[EBP-0C]
015F:0040EF88  MOV       AL,[EBX+EAX-02]
015F:0040EF8C  MOV       EDX,[EBP-08]
015F:0040EF8F  CMP       AL,[EDX+01]  //比较横杠前的倒数第 2 个字符
015F:0040EF92  JNZ       0040EFA0
015F:0040EF94  MOV       EDI,00000001  //good guy
015F:0040EF99  JMP       0040EFA0
015F:0040EF9B  MOV       EDI,00000001
015F:0040EFA0  DEC       EDI          //bag guy
015F:0040EFA1  SETZ      BL
015F:0040EFA4  JMP       0040EFA8
```

至此我们找到了注册码中的 3 位。继续按 `F10`（或者按一下 `F12`），又看见一段判断。这个判断也不复杂，写个小程序穷举一下或者在心里推算一下就可以确定注册码的 2～5 位应该是什么（但是不唯一）。

```
015F:0040F05A  MOV       EDX,00000002
015F:0040F05F  CALL      00401C28    //把注册码的 2～5 转成一个整数
015F:0040F064  MOV       ECX,EAX
015F:0040F066  CMP       ECX,00001389 //根据该数和 0x1389 的大小分成两种情况
015F:0040F06C  JGE       0040F080
015F:0040F06E  LEA       EAX,[ECX+29] //小于 0x1389 则加上 0x29
015F:0040F071  MOV       ECX,00000007
015F:0040F076  CDQ
015F:0040F077  IDIV      ECX
015F:0040F079  TEST      EDX,EDX
015F:0040F07B  SETZ      BL          //和必须是 7 的倍数才行
015F:0040F07E  JMP       0040F096
015F:0040F080  MOV       EAX,ECX
015F:0040F082  SUB       EAX,27      //大于等于 0x1389 则减去 0x27
015F:0040F085  MOV       ECX,0000000D
015F:0040F08A  CDQ
015F:0040F08B  IDIV      ECX
015F:0040F08D  TEST      EDX,EDX
015F:0040F08F  SETZ      BL         //差必须是 0x0D 的倍数才行
015F:0040F092  JMP       0040F096
```

继续按 `F10`，看见另一个判断，这是在判断注册码的第 6 位。

```
015F:0040F140  MOV       EDX,[EBP-0C]  //正确
015F:0040F143  POP       EAX           //错误
015F:0040F144  CALL      004011B0
015F:0040F149  SETZ      BL
```

至于横杠后面为何是名字，等注册成功后看一下它的 `About` 对话框就明白了，因为那里显示的是 `Registered to: XXXX`。 `4dd466d0e1e0cc123afbe504da2462ca`
