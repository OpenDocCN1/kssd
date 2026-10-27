
# 关于 VB P-CODE 的一些总结（10千字）

## 前言

到网上查 PCODE 的资料，到了一个讲 Fight Against Crack 的网站上，那个作者讲了许多阴毒的招数后，还特别说明，如果是 VB 程序，最好把它做成 P-CODE，这会大大增加破解的难度。事实真的如此吗？WKT 的一个大虾却说：We are going to show that protecting a VB application is a very difficult task. The last Microsoft's invention on VB, the 'p-code', it's a delicious bite for reverse engineers. I'll show that a 'p-compiled' application may be easier to crack that a conventional compiled one.

不知是不是我孤陋寡闻，总觉得现在关于 VB P-CODE 的资料少之又少，不管国内国外只能找到各位老大们为数不多的几篇教程（很多也是说“我猜想”之类拿不准的话），《加密与解密》上关于这个也是一笔带过（大概是为续集着想^_^），可能是大虾觉得这个太简单不屑于讲，或是哪里已经有非常系统全面的介绍。总之以前我是一见是 VB 就害怕，再见是 PCODE 就投降，这大概就是人家用 P-CODE 对付破解的原因。这几天终于下决心自己写了几个程序试验，再结合各位老大的文章，总结出一点东西，非常不全面，希望大家多多指教，多多补充。

## 工具

“工欲善其事，必先利其器。”那句古话好像是这么说的。我们找的工具有：`WKTVBDebug`（动态破解用，相当于 PCODE 里的 `SOFTICE`）、`EXDEC`（静态分析，相当于 `W32DASM`）、`SmartCheck`（可以作辅助用）。

## P-CODE 助记符

要想破解 PCODE 程序，关键要理解里面的“助记符”（它又不同于汇编语言的“mnemonics”，我不知道该怎么表示了）的作用，PCODE 的助记符乍看上去很乱，好像比汇编还难，其实它们都是由几部分组成的。比如 `CVarStr` 就是由三部分组成（详见下文）。VB PCODE 中常见的“助记符”如下面所示：（只总结出了一点，恳请各位补充）

### 表示数据类型的

- `I2`：`Integer`，占一个字节的整数（汇编里的 `BYTE`）
- `I4`：`Integer`，占两个字节的整数（汇编里的 `WORD`）
- `I8`：`Integer`，占四个字节的整数（汇编里的 `DWORD`）
- `UI4`：`Unsigned Integer`，无符号整数
- `UI8`：`Unsigned Integer`，无符号整数
- `R4`：`Real`，单精度实数（`Single`）
- `R8`：`Real`，双精度实数（`Double`）
- `Str`：`String`，字符串类型
- `Var`：`Variant`，变量类型。这就是 BASIC 特殊的地方，它允许用户在使用变量前不进行声明，这种不声明的变量就用这种类型存储，它可以包括数字、字串等各种类型。我看 M$ 的这个玩意儿没给用户带来方便，只能让一些初学编程的菜鸟思维混乱，让咱们破解时也非常郁闷:(。它的存储方式非常奇怪，比方说你看到一个 `VARIANT` 类型的数据被放到内存里了，你跟过去找，结果什么也找不到。看雪书上说应该 `D *(EAX+8)`，原来它真正的数据往后挪了 8 个字节，真不知在搞什么.....BTW:如果是一个数值类型的数据，它的地址向后移 8 个字节即为真正的数值，如果是一个字符串型的数据，它的地址向后移 8 个字节即为指向一个 UNICODE 字串的指针。

### 表示堆栈操作的

PCODE 没有寄存器，全部通过堆栈传送数据，因此非常重要。

- `St`：Store，把当前栈顶的数据放在内存里
- `Ld`：Load，把内存某处的数据压入堆栈
- `Lit`：Literal，把一个“立即数”压入堆栈

### 其它重要的

- `C`：Convert，数据转换。如 `CI4I2` 即把 `BYTE` 扩充为 `WORD`（`I2` -> `I4`）
- `Eq`：Equal，判断是否相等，并把结果（0 或 1）入栈
- `Lt`：判断是否小于
- `Gt`：判断是否大于
- `Len`：得到字串长度

### 跳转指令

- `Branch`：无条件跳转
- `BranchT`：栈顶数据为真则跳
- `BranchF`：栈顶数据为假则跳

### 一些算术运算

- `Add`、`Sub` 等等应该都比较好认吧。

### VB 控件前缀

从一篇介绍 PCODE 的文章里抄来一些，不知有没有用：

| Prefix | Control |
|---|---|
| `cbo` | Combo box |
| `chk` | Check box |
| `cmd` | Command Button |
| `dir` | Directory box |
| `drv` | Drive list box |
| `fil` | File list box |
| `fra` | Frame |
| `frm` | Form |
| `grd` | Grid |
| `hsb` | Horizontal scrollbar |
| `img` | Image |
| `lbl` | Label |
| `lin` | Line |
| `lst` | List box |
| `mnu` | Menu |
| `ole` | OLE client |
| `opt` | Option button |
| `pic` | Picture Box |
| `shp` | Shape |
| `tmr` | Timer |
| `txt` | Text box |
| `vsb` | Vertical scrollbar |

### 一些不太清楚的猜想

还有一些不太清楚的，都是我的猜想，希望大虾解释：

- `Call`：调用过程
- `Free`：释放内存空间
- `Rf`：局部变量????
- `Pr`：????
- `Ad`：是不是 `Address`？？
- `HardType`：是干什么的？

这些组合在一起就成了多种多样的指令，很有趣吧。

## P-CODE 的堆栈

还有一个要特别强调的是 PCODE 的堆栈，PCODE 几乎所有的指令都要对堆栈进行操作，有许多指令都是针对栈顶的一个或两个数据进行操作，因此在动态调试 PCODE 时要十分注意右边显示的堆栈区，并经常查看内存，这样才能理解指令的意义。

## 实践

下面来实践一下，运行起尘封已久的 VB，在 FORM 上放一个 `TEXT1`，一个 `BUTTON1`，双击 `Button1`，在下面输入：

```
Private Sub Command1_Click()
    st1 = Text1.Text
    st2 = ""
    m = Len(Text1.Text)
    For i = 1 To m
        st2 = st2 + Mid$(Text1.Text, m - i + 1, 1)
    Next i
    MsgBox st2, vbOKOnly, "CRACK"
End Sub
```

呵呵，很简单是不是。按一下按钮就把 TEXT 里的文本反过来显示在消息框里。

下面来“生成工程”，注意一定要在“选项”里选择生成 P-CODE 文件。然后用 `Exdec` 分析一下：

```
Proc: 401a90

4019B0: 04 FLdRfVar                local_008C          好像是一个指向TEXT的指针
4019B3: 21 FLdPrThis                                   先给一个下马威，前几句全不太明白！
4019B4: 0f VCallAd                 text                用WKTVBDebug过这一句时能看到Form1.text1
4019B7: 19 FStAdFunc               local_0088          猜想应该是取得句柄之类的事情
4019BA: 08 FLdPr                   local_0088
4019BD: 0d VCallHresult            get__ipropTEXTEDIT  调用，从字面上可以看出是GetText
4019C2: 3e FLdZeroAd               local_008C          好像压入一个指向上面文本的指针，不太清楚,反正上面这个过程很经典啦，几乎从文本框读数都是这样
4019C5: 46 CVarStr                 local_00AC          把上面得到的字串转为Var格式
4019C8: Lead1/f6 FStVar                                再把这个VAR数据入栈 st1
4019CC: 1a FFree1Ad                local_0088          释放前面的空间
4019CF: 3a LitVarStr:              ( local_00CC )      压入一个立即数:空字串st2=""
4019D4: Lead2/00 FStVarCopy
4019D8: 04 FLdRfVar                local_008C
4019DB: 21 FLdPrThis
4019DC: 0f VCallAd                 text
4019DF: 19 FStAdFunc               local_0088
4019E2: 08 FLdPr                   local_0088
4019E5: 0d VCallHresult            get__ipropTEXTEDIT  和上面相同,得到字串
4019EA: 6c ILdRf                   local_008C          压入字串
4019ED: 4a FnLenStr                                    得到字串的长度m
4019EE: Lead2/69 CVarI4            local_00CC          转为VAR类型
4019F2: Lead1/f6 FStVar                                VAR类型的长度入栈
4019F6: 2f FFree1Str               local_008C          释放内存空间
4019F9: 1a FFree1Ad                local_0088
4019FC: 28 LitVarI2:               ( local_00FC ) 0x1  (1) 压入一个立即数0x1
401A01: 04 FLdRfVar                local_00EC          local_00EC是循环变量i
401A04: 04 FLdRfVar                local_00DC          这个是上面得到的长度m
401A07: Lead3/68 ForVar:           (when done) 401A67  FOR i=1 to m 开始循环
401A0D: 04 FLdRfVar                local_008C
401A10: 21 FLdPrThis
401A11: 0f VCallAd                 text
401A14: 19 FStAdFunc               local_0088
401A17: 08 FLdPr                   local_0088
401A1A: 0d VCallHresult            get__ipropTEXTEDIT  和上面相同的过程,得到字串
401A1F: 04 FLdRfVar                local_00BC          把local_BC压入,这实际上是st2
401A22: 28 LitVarI2:               ( local_013C ) 0x1  (1) 压一个0x1,CALL的参数
401A27: 04 FLdRfVar                local_00DC          字串长度m
401A2A: 04 FLdRfVar                local_00EC          循环变量i
401A2D: Lead0/9c SubVar                                相减 m-i
401A31: 28 LitVarI2:               ( local_00CC ) 0x1  (1) 再压入一个0x1
401A36: Lead0/94 AddVar            local_012C          再加1, m-i+1,CALL的参数
401A3A: Lead1/22 CI4Var                                转成整数型
401A3C: 6c ILdRf                   local_008C          压入,作为下面CALL的参数
401A3F: 0b ImpAdCallI2                                 这是rtcMidCharBStr,源码中的Mid$()
401A44: 46 CVarStr                 local_014C          把取得的字符转成Var型
401A47: Lead0/94 AddVar            local_015C          把新取得的字符和上面的401A1F处的st2连起来
401A4B: Lead1/f6 FStVar
401A4F: 2f FFree1Str               local_008C
401A52: 1a FFree1Ad                local_0088          释放
401A55: 36 FFreeVar
401A5E: 04 FLdRfVar                local_00EC          设好循环变量
401A61: Lead3/7e NextStepVar:      (continue) 401A0D   NEXT i,循环变量+1,直到结束
401A67: 27 LitVar_Missing                               VB里面那些带[]的可选参数，如果不加设定
401A6A: 27 LitVar_Missing                               就会变成这种Missing或NULL的形式
```
