
# 有关 VB 程序 P-CODE 代码逆向工程入门浅说

> 原文：[`www.pediy.com/kssd/pediy06/pediy6272.htm`](https://www.pediy.com/kssd/pediy06/pediy6272.htm)

对于破解初学者来说，可能在调试 VB（P-CODE）编写的程序时感到难于分析和跟踪，特别是查找和定位“注册码比较”关键点，以及修补代码等。小弟对 VB 程序内部构造同样也是一塌糊涂，对破解更是一知半解，和各位初学者一样，只是感兴趣而已。偶尔尝试调试，跟踪个 VB 程序玩玩，没什么目的。闲着没事，咱就随便扯点个人浅显的经验，仅供和我一样的菜鸟同道参考，让各位高手见笑了。

**特别声明：**本文仅仅服务于对 VB P-CODE 程序破解感兴趣的初学者，绝非专业学术研究，不提供任何有关研究分析的科学方法和步骤。

**注：**如果您仅仅对 `OLLYDBG` 调试 VB P-CODE 感兴趣，请直接参考第四部分。

## 1. 缘起

有天，您正在网上溜达，忽然发现一个感兴趣的共享小程序，想拿来用用，可还没怎么试，它就跟您较劲儿，老是让您注册，烦不烦？咱能受这个限制吗？… 既然玩保护，那咱们就帮作者测试测试保护的强度，呵呵，考验咱们的时候又到了。。。3 下 5/2, 经过您的火眼金睛一看，原来是个 VB（P-CODE）捏把的东东。怎么看出是 P-CODE? 方法很多，比如拿 `WKTVBDE` 要是能 `LOAD`，我想它就是了，什么?不知道 `WKTVBDE` 是什么？那用 `Exdec` 反一下，什么，这个也没听说过？那咱们先别往下继续了，您最好先了解一下有关的预备知识，也可以在本论坛搜索相关 `vb p-code` 破解案例分析。擅自推荐您参考“FLY”大侠的破解实例。

对手出现了，怎么办？要是我，可能准备如下工具：

*   `VBExplorer` (静态分析 P－CODE 的好东西，又免费，建议作者继续完善，坚持不收费政策 50 年不变 ^&^ )
*   `WKTVBDE` (VB P-CODE 动态跟踪调试器，在下一般用它来动态修改代码，判断分析的效果。要想用它真正搞清楚程序的细节，小弟还没有发现好办法，但作为粗略的分析工具，查看程序分支判断，循环次数，它还是蛮好用的。对于分析 VB P-CODE，它有如一个大砂轮，先用它磨一磨，精细加工，那是下一个将介绍的工具)
*   `Ollydbg` (很不错的调试器，我用 `1.09D`。好在它的兼容性和便捷性，在我的 `WIN2003 sever` 上表现很稳定，跟踪，调试 VB P-CODE 再合适不过了，只可惜不是咱们中国人的产品。在本短文稍后，咱们主要就讨论如何用它跟踪分析 VB P-CODE，因为这个步骤也算蛮重要嘛)

还有其它一些可以选择的工具，就看您的个人爱好了，比如：

*   `Exdec` vb p-code 代码反编译器，不错的工具
*   `Vbparser 1.2` 同样是 vb p-code 反编译器，不过我用的版本不能反 VB 5, 有些 vb 6 的也不行，不知道是怎么回事。

还有什么好工具希望您告诉我。

## 2. 鬼穷三技

聊斋上说，小鬼儿也没什么可怕的，也就有 3 招，你知道了，对付它就心里有了底，有了战胜它的信心。话说 VB P-CODE 程序，咱们这些新手外行如何搞定它呢？咱们也来个 3 招克敌:

a. **静态分析**：粗略搞清楚目标程序的大致流程，分支，凭分析结果和编程，破解的经验，判断和假设目标程序的设计思路
b. **代码还原**：根据 p-code 伪代码，大致可以还原出 VB 源程序代码（针对关键和重要的部分，对于破解，就是还原判断有关的和比较注册码前后部分的 VB 代码）
c. **动态跟踪**：应用源码级调试器（例如 `olldbg`）跟踪程序的运行细节，发现重要数据，甚至注册码等信息

空口无凭，咱们还是通过一个简单案例来演示一下整个的 VB P-CODE 逆向过程，如何？

## 3. 循序渐进

为了演示方便，咱们先写一个简单的 VB 注册小程序，源代码如下(程序未经任何优化，仅用于演示)：

```vb
Private Sub Text1_Change()

  Dim i As Byte

  For i = 1 To Len(Text1.Text)

  If Mid(Text1.Text, i, 1) > "9" Or Mid(Text1.Text, i, 1) < "0" Then

   Text2.Text = "Please enter 0-9, try again"

   GoTo final_end

  End If

  Next

  If Len(Text1.Text) >= 3 Then

  get_1 = CInt(Mid(Text1.Text, 1, 1))

  get_2 = CInt(Mid(Text1.Text, 2, 1))

  get_3 = CInt(Mid(Text1.Text, 3, 1))

  get_4 = get_2 + get_3

  If get_1 = get_4 Then

  Text2.Text = "good"

  Else

  Text2.Text = "bad"

  End If

  End If

final_end:

End Sub
```

程序仅仅根据用户输入的数字判断是否符合内定的规则，如果是，显示：`good`,错误，则提示：`bad`。程序很简单，您自己看一下就明白了，咱们就不再详细解释了，不耽误您的时间了。

这段小程序对应的 P-CODE 代码如下（使用 `VBExplorer` 获得，相关程序可以从本论坛得到，或联系作者）：

```asm
\[Text1.Change\]

:00401B9C  F401                LitI2_Byte          ;Push 01

:00401B9E  FC0D                CUI1I2              ;

:00401BA0  047AFF              FLdRfVar            ;Push LOCAL_0086

:00401BA3  0470FF              FLdRfVar            ;Push LOCAL_0090

:00401BA6  21                  FLdPrThis           ;\[SR\]=\[stack2\]

:00401BA7  0F0003              VCallAd             ;Return the control index 02

:00401BAA  1974FF              FStAdFunc           ;

For i = 1 To Len(Text1.Text)

:00401BAD  0874FF              FLdPr               ;\[SR\]=\[LOCAL_008C\]

\*\*\*\*\*\*\*\*\*\*\*Reference To:\[propget\]TextBox.Text

:00401BB0  0DA0000000          VCallHresult        ;Call ptr_004016B8

:00401BB5  6C70FF              ILdRf               ;Push DWORD \[LOCAL_0090\]

:00401BB8  4A                  FnLenStr     ；取得输入串的长度，作为循环的参数

:00401BB9  FC0E                CUI1I4              ;

:00401BBB  2F70FF              FFree1Str           ;SysFreeString \[LOCAL_0090\];

:00401BBE  1A74FF              FFree1Ad            ;Push \[LOCAL_008C\];

:00401BC1  FE626CFFD900        ForUI1      ; 循环开始

:00401BC7  0470FF              FLdRfVar            ;Push LOCAL_0090

:00401BCA  21                  FLdPrThis           ;\[SR\]=\[stack2\]

:00401BCB  0F0003              VCallAd             ;Return the control index 02

:00401BCE  1974FF              FStAdFunc           ;

:00401BD1  0874FF              FLdPr               ;\[SR\]=\[LOCAL_008C\]

\*\*\*\*\*\*\*\*\*\*\*Reference To:\[propget\]TextBox.Text  ；出现这个提示，说明程序中参照了输入框内容

:00401BD4  0DA0000000          VCallHresult        ;Call ptr_004016B8

If Mid(Text1.Text, i, 1) > "9" Or Mid(Text1.Text, i, 1) < "0" Then

:00401BD9  0404FF              FLdRfVar            ;Push LOCAL_00FC

:00401BDC  21                  FLdPrThis           ;\[SR\]=\[stack2\]

:00401BDD  0F0003              VCallAd             ;Return the control index 02

:00401BE0  1908FF              FStAdFunc           ;

:00401BE3  0808FF              FLdPr               ;\[SR\]=\[LOCAL_00F8\]

\*\*\*\*\*\*\*\*\*\*\*Reference To:\[propget\]TextBox.Text

:00401BE6  0DA0000000          VCallHresult        ;Call ptr_004016B8

:00401BEB  283CFF0100          LitVarI2     ;Push 数字 1

:00401BF0  FCE07AFF            FLdUI1              ;

:00401BF4  E7                  CI4UI1      ;Push 变量 i

:00401BF5  3E70FF              FLdZeroAd           ;

:00401BF8  465CFF              CVarStr             ;

:00401BFB  042CFF              FLdRfVar            ;

\*\*\*\*\*\*\*\*\*\*Reference To->msvbvm60.rtcMidCharVar       ；调用了 Mid 函数，一般相关的参数在调用前已经入栈，注意上边两个 PUSH 指令，参数入栈顺序是从右到左，对于 MID，既是压入 1，i。意思是取从第 i 个开始的字符，总共取 1 个。

:00401BFE  0A01001000          ImpAdCallFPR4       ;Call ptr_00401020;

:00401C03  042CFF              FLdRfVar            ;Push LOCAL_00D4

\*\*\*\*\*\*Possible String Ref To->"9"    ；表示”9”这个文本字符被引用

:00401C06  3A1CFF0200          LitVarStr           ;PushVarString ptr_004016CC

:00401C0B  5D                  HardType            ;

:00401C0C  FB700CFF            GtVar               ;

:00401C10  28D4FE0100          LitVarI2 ;Push 数字 1

:00401C15  FCE07AFF            FLdUI1              ;

:00401C19  E7                  CI4UI1   ;Push 变量 i

:00401C1A  3E04FF              FLdZeroAd           ;

:00401C1D  46F4FE              CVarStr             ;

:00401C20  04C4FE              FLdRfVar            ;

\*\*\*\*\*\*\*\*\*\*Reference To->msvbvm60.rtcMidCharVar

:00401C23  0A01001000          ImpAdCallFPR4       ;Call ptr_00401020;

:00401C28  04C4FE              FLdRfVar            ;Push LOCAL_013C

\*\*\*\*\*\*Possible String Ref To->"0"

:00401C2B  3AB4FE0300          LitVarStr ; 表示”0”文本字符串被引用

:00401C30  5D                  HardType            ;

:00401C31  FB63A4FE            LtVar               ;

:00401C35  FB1F94FE            OrVar     ; or 另一个条件

:00401C39  FF1B                CBoolVarNull        ;vbaBoolVarNull

:00401C3B  29040074FF08FF      FFreeAd             ;

:00401C42  360C005CFF3CFF2C    FFreeVar            ;Free 000C/2 variants

:00401C51  1CD000              BranchF   ；比较指令，

\*\*\*\*\*\*Possible String Ref To->"Please enter 0-9, try again"

Text2.Text = "Please enter 0-9, try again"        ；如果输入字符无效，提示重新输入

:00401C54  1B0400              LitStr              ;Push ptr_004016DC

:00401C57  21                  FLdPrThis           ;\[SR\]=\[stack2\]

:00401C58  0F0803              VCallAd             ;Return the control index 04

:00401C5B  1974FF              FStAdFunc           ;

:00401C5E  0874FF              FLdPr               ;\[SR\]=\[LOCAL_008C\]

\*\*\*\*\*\*\*\*\*\*\*Reference To:\[propput\]TextBox.Text

:00401C61  0DA4000000          VCallHresult        ;Call ptr_004016B8

:00401C66  1A74FF              FFree1Ad            ;

GoTo final_end

:00401C69  1E1002              Branch              ;ESI=00401DAC

:00401C6C  047AFF              FLdRfVar            ;Push LOCAL_0086

Next

:00401C6F  FE786CFF2B00        NextUI1      ; 循环部分中止

:00401C75  0470FF              FLdRfVar            ;Push LOCAL_0090

:00401C78  21                  FLdPrThis           ;\[SR\]=\[stack2\]
```



### 代码段 1：字符串长度检查

```
:00401C79  0F0003             VCallAd            ;Return the control index 02   
:00401C7C  1974FF             FStAdFunc          ;   
If Len(Text1.Text) >= 3 Then   
:00401C7F  0874FF             FLdPr              ;[SR]=[LOCAL_008C]   
***********Reference To:[propget]TextBox.Text       
:00401C82  0DA0000000         VCallHresult       ;Call ptr_004016B8   
:00401C87  6C70FF             ILdRf              ;Push DWORD [LOCAL_0090]   
:00401C8A  4A                 FnLenStr     ; 输入字符串长度   
:00401C8B  F503000000         LitI4        ；常数 3 用于比较   
:00401C90  E0                 GeI4           ;   
:00401C91  2F70FF             FFree1Str          ;SysFreeString [LOCAL_0090];   
:00401C94  1A74FF             FFree1Ad           ;Push [LOCAL_008C];   
:00401C97  1C1002             BranchF      ；比较判断   
```

### 变量 `get_1` 赋值：获取第一个字符

`get_1` = `CInt(Mid(Text1.Text, 1, 1))`

```
:00401C9A  0470FF             FLdRfVar           ;Push LOCAL_0090   
:00401C9D  21                 FLdPrThis          ;[SR]=[stack2]   
:00401C9E  0F0003             VCallAd            ;Return the control index 02   
:00401CA1  1974FF             FStAdFunc          ;   
:00401CA4  0874FF             FLdPr              ;[SR]=[LOCAL_008C]   
***********Reference To:[propget]TextBox.Text   
:00401CA7  0DA0000000         VCallHresult       ;   
:00401CAC  283CFF0100         LitVarI2        ; Push 1 参数，用于 MID 函数 取几个字符   
:00401CB1  F501000000         LitI4          ；Push 1 参数，用于 MID 函数   
                                              ；表示 从哪个字符位置开始取   
:00401CB6  3E70FF             FLdZeroAd          ;Push DWORD [LOCAL_0090];   
:00401CB9  465CFF             CVarStr            ;   
:00401CBC  042CFF             FLdRfVar           ;Push LOCAL_00D4   
***********Reference To->msvbvm60.rtcMidCharVar      ；mid 函数调用   
                      |   
:00401CBF  0A01001000         ImpAdCallFPR4      ;Call ptr_00401020; check stack 0010; Push EAX   
:00401CC4  042CFF             FLdRfVar           ;Push LOCAL_00D4   
:00401CC7  FC45               FnCIntVar          ;vbaI2ErrVar   
:00401CC9  441CFF             CVarI2             ;   
:00401CCC  FCF684FE           FStVar             ;   
:00401CD0  1A74FF             FFree1Ad           ;Push [LOCAL_008C]; Call   
:00401CD3  3608005CFF3CFF2C   FFreeVar           ;Free 0008/2 variants   
```

### 变量 `get_2` 赋值：获取第二个字符

`get_2` = `CInt(Mid(Text1.Text, 2, 1))`

```
:00401CDE  0470FF             FLdRfVar           ;Push LOCAL_0090   
:00401CE1  21                 FLdPrThis          ;[SR]=[stack2]   
:00401CE2  0F0003             VCallAd            ;Return the control index 02   
:00401CE5  1974FF             FStAdFunc          ;   
:00401CE8  0874FF             FLdPr              ;[SR]=[LOCAL_008C]   
***********Reference To:[propget]TextBox.Text   
:00401CEB  0DA0000000         VCallHresult       ;Call ptr_004016B8   
:00401CF0  283CFF0100         LitVarI2           ;PushVarInteger 0001   
:00401CF5  F502000000         LitI4              ;Push 00000002   
:00401CFA  3E70FF             FLdZeroAd          ;Push DWORD [LOCAL_0090];   
:00401CFD  465CFF             CVarStr            ;   
:00401D00  042CFF             FLdRfVar           ;Push LOCAL_00D4   
***********Reference To->msvbvm60.rtcMidCharVar   
:00401D03  0A01001000         ImpAdCallFPR4      ;Call ptr_00401020;   
:00401D08  042CFF             FLdRfVar           ;Push LOCAL_00D4   
:00401D0B  FC45               FnCIntVar          ;vbaI2ErrVar   
:00401D0D  441CFF             CVarI2             ;   
:00401D10  FCF674FE           FStVar             ;   
:00401D14  1A74FF             FFree1Ad           ;Push [LOCAL_008C]; Call   
:00401D17  3608005CFF3CFF2C   FFreeVar           ;Free 0008/2 variants   
```

### 变量 `get_3` 赋值：获取第三个字符

`get_3` = `CInt(Mid(Text1.Text, 3, 1))`

```
:00401D22  0470FF             FLdRfVar           ;Push LOCAL_0090   
:00401D25  21                 FLdPrThis          ;[SR]=[stack2]   
:00401D26  0F0003             VCallAd            ;Return the control index 02   
:00401D29  1974FF             FStAdFunc          ;   
:00401D2C  0874FF             FLdPr              ;[SR]=[LOCAL_008C]   
***********Reference To:[propget]TextBox.Text   
:00401D2F  0DA0000000         VCallHresult       ;Call ptr_004016B8   
:00401D34  283CFF0100         LitVarI2           ;PushVarInteger 0001   
:00401D39  F503000000         LitI4              ;Push 00000003   
:00401D3E  3E70FF             FLdZeroAd          ;Push DWORD [LOCAL_0090];   
:00401D41  465CFF             CVarStr            ;   
:00401D44  042CFF             FLdRfVar           ;Push LOCAL_00D4   
***********Reference To->msvbvm60.rtcMidCharVar   
:00401D47  0A01001000         ImpAdCallFPR4      ;Call ptr_00401020; check stack 0010; Push EAX   
:00401D4C  042CFF             FLdRfVar           ;Push LOCAL_00D4   
:00401D4F  FC45               FnCIntVar          ;vbaI2ErrVar   
:00401D51  441CFF             CVarI2             ;   
:00401D54  FCF664FE           FStVar             ;   
:00401D58  1A74FF             FFree1Ad           ;Push [LOCAL_008C]; Call   
:00401D5B  3608005CFF3CFF2C   FFreeVar           ;Free 0008/2 variants   
```

### 变量 `get_4` 赋值：计算和后两位之和

`get_4` = `get_2` + `get_3`

```
:00401D66  0474FE             FLdRfVar       ;Push 变量 1 用于相加   
:00401D69  0464FE             FLdRfVar       ;Push 变量 2 用于相加   
:00401D6C  FB945CFF           AddVar           ;   
:00401D70  FCF654FE           FStVar           ;   
```

### 条件判断：验证输入

若 `get_1` = `get_4`，则执行后续逻辑。

```
:00401D74  0484FE             FLdRfVar       ;Push 变量 1 用于比较   
:00401D77  0454FE             FLdRfVar       ;Push 变量 2 用于比较   
:00401D7A  FB33               EqVarBool        ;   
:00401D7C  1CFB01             BranchF       ;比较判断结果   
******Possible String Ref To->"good"   
```

### 分支 1：输入正确

`Text2.Text` = `"good"`

```
:00401D7F  1B0500             LitStr           ;Push ptr_00401718   
:00401D82  21                 FLdPrThis          ;[SR]=[stack2]   
:00401D83  0F0803             VCallAd            ;Return the control index 04   
:00401D86  1974FF             FStAdFunc          ;   
:00401D89  0874FF             FLdPr              ;[SR]=[LOCAL_008C]   
***********Reference To:[propput]TextBox.Text   
:00401D8C  0DA4000000         VCallHresult       ;Call ptr_004016B8   
:00401D91  1A74FF             FFree1Ad           ;Push [LOCAL_008C]; Call   
Else   
```

### 分支 2：输入错误

`Text2.Text` = `"bad"`

```
:00401D94  1E1002             Branch           ;ESI=00401DAC   
******Possible String Ref To->"bad"   
:00401D97  1B0600             LitStr           ;Push ptr_00401728   
:00401D9A  21                 FLdPrThis          ;[SR]=[stack2]   
:00401D9B  0F0803             VCallAd            ;Return the control index 04   
:00401D9E  1974FF             FStAdFunc          ;   
:00401DA1  0874FF             FLdPr              ;[SR]=[LOCAL_008C]   
***********Reference To:[propput]TextBox.Text   
:00401DA4  0DA4000000         VCallHresult       ;Call ptr_004016B8   
:00401DA9  1A74FF             FFree1Ad           ;Push [LOCAL_008C]; Call   
```

### 程序退出

```
:00401DAC  13                 ExitProcHresult    ;   
:00401DAD  0000               LargeBos           ;IDE beginning of line   
```

> 注意：以上伪码中提示的 `VB` 源代码行，可能并不是精确的定位，仅是凭经验加入的，仅供参考。由于对 `P-CODE` 以及 `VBExplorer` 都不甚了解，许多伪代码的含义，以及其在 `VBExplorer` 中给出的注释，都不能明了个中的含义，还望了解者指点，多谢。

> 不论如何，根据 `P-CODE` 代码，我们可以大致分析出粗略的程序流程，变量的使用和定位并不一定精确，许多还要凭您的破解经验和编程经验。但是，这个粗略的分析可以帮助我们模拟出大致的 `VB` 源代码。



在反向出粗略的 VB 源代码的过程中，您要不断将您写出的 VB 代码编译后 P-CODE 代码与源程序反编译后的 P-code 代码进行比对，尽量使其一致，至少看上去相似。这个过程需要您花些时间了，您要假设，如果你就是作者，你会如何写，特别是变量的运用，类型的定义，编程的习惯以及经验等。这个过程在不断的进行中，会使你逐渐积累更多有益的经验，不知不觉中，你就会有一种“感觉”，以前，那个台湾写破解教程的老兄说这个叫“触机”，大概对应的英文词是“sense”。总之，您就是知道了，似乎一下就明白了，原来做这个程序的老兄是这样的，是这个意思。好了，下一步就是验证我们的思路是否真的就是对的了。

### 4．着法宝

通常，我们可以用 `WKTVBDE` 跟踪程序 P-CODE，看看它在关键部分的循环，分支走向，在某些关节点（P-CODE 伪代码，或者是某个 VB 函数调用上）设断点，这样，基本上可以将我们模拟出来的 VB 源代码中大的框架构造搞清楚，设定好基本的分支条件。那么，到底在程序的细节实现上，哪里使用了哪些变量，关键的比较点，比较的变量值是怎样计算得到的，还有什么隐藏的花招，以及一些细节的东西？为了快速获得有益的信息，我们还是应该通过 `OLLDBG` 这样的实时调试器去观察分析。

在应用 `OLLDBG` 调试 VB P-CODE 代码时，我们应该注意几点：

一般 VB 中的使用的局部变量位置在内存中是不确定的，这次调试和上次调试的同一变量位置基本是不同的，是动态产生的，所以，不要象调试其他 C 编译或汇编程序那样，下一个固定的内存断点去发现某些变量的变化（仅仅是个人意见，不一定对）。

VB P-CODE 编译的程序运行时，在主要流程中，通常 `ESI` 寄存器指向相应的 VB p-code 伪代码。 所以，调试时，最好同时打开相应程序的 P-CODE 伪代码（例如运行 `VBExplorer`,并打开相应的程序。为了便于分析，可以是同样程序的拷贝副本），以供参考，查看程序的实际流程。

通常，vb 程序运行在虚拟机（`MSVBVM60.DLL`）中 ，因此，当您使用 `OLLDBG` 打开一个 vb p-code 程序后，应该在菜单中选择“view”选项，然后再进入选择“Executable modules”, 并在其中的“MSVBVM60”行上，点击鼠标右键，然后选“view name”。如此，可以直接查看 VB 运行库中的函数名，查找有关的函数名，在常用的函数名上点击，并设置断点。

例如：在 `rtcMidCharVar` 上设置断点：

在我得机器上是：`7352B403 > 55    PUSH EBP`

例如：设置条件断点 `esi = = 401BEF` ；ESI 指向的 P-CODE 代码，应该是你在分析目标程序后，认为重要的调用了 MID 函数的位置。参考上面的演示程序，我们假定要查看两个 MID 函数被调用的位置，这两个位置可能是：

语句 `If Mid(Text1.Text, i, 1) > "9" Or Mid(Text1.Text, i, 1) < "0"` 中的两个调用了 MID 的地方（p-code 代码的相对位置）：`00401BFE` 和 `00401C23`（VBExplorer 中看到的）

当程序中断在您设置的断点后，你可以逐步跟踪程序了，为了把握住程序的流向，您要时刻关注 `ESI` 值的变化，它通常在 `40xxxx` 范围内，它明确地指向 VB P-CODE 伪代码的流向，你不用担心不知道自己所在的位置。你可以随时切换到 `VBExplorer` 中，看看程序到底跑到哪里去了。

在您调试的过程中，您会发 `CALL AX`，应该单步跟进去，它通常会带你返回那个正被调试的 VB 程序主域（或叫“领空”，学台湾人的说法），一般附近是一些 `JMP` 长跳转，这些是程序中需要调用函数，应该给予重视，可以尝试在相关函数上下断点。

当您看到 `call bx` 时，通常程序要调用真正的函数执行功能，你应该跟进去，一般在执行 P-CODE 比较伪指令的过程中，你会发现一个 `call bx`，一定要跟进去，通常会它引导你看到真正的注册码以及你输入的假注册码进行比较。通过观察堆栈的显示，你就可以看到他们，两个可爱的字符串(通常出现在 OLLYDBG 的调试窗口的右下方)。

跟踪 VB 程序时，通常会看到如下的代码：

```
7353F7B7   3BFC            CMP EDI,ESP
7353F7B9   0F85 D67D0000   JNZ MSVBVM60.73547595
7353F7BF   33C0            XOR EAX,EAX
7353F7C1   8A06            MOV AL,BYTE PTR DS:[ESI]
7353F7C3   46              INC ESI
7353F7C4   FF2485 58FA5373 JMP DWORD PTR DS:[EAX*4+7353FA58]
7353F7CB   0FB70E          MOVZX ECX,WORD PTR DS:[ESI]
7353F7CE   0FB77E 02       MOVZX EDI,WORD PTR DS:[ESI+2]
7353F7D2   83C6 04         ADD ESI,4
7353F7D5   03FC            ADD EDI,ESP
7353F7D7   8B55 AC         MOV EDX,DWORD PTR SS:[EBP-54]
7353F7DA   8B048A          MOV EAX,DWORD PTR DS:[EDX+ECX*4]
7353F7DD   0BC0            OR EAX,EAX
7353F7DF   90              NOP
7353F7E0   0F84 027F0000   JE MSVBVM60.735476E8
7353F7E6   803D B4115573 00 CMP BYTE PTR DS:[735511B4],0
7353F7ED   0F85 037F0000   JNZ MSVBVM60.735476F6
7353F7F3   FFD0            CALL EAX
7353F7F5   3BFC            CMP EDI,ESP
7353F7F7   0F85 987D0000   JNZ MSVBVM60.73547595
7353F7FD   50              PUSH EAX
7353F7FE   33C0            XOR EAX,EAX
7353F800   8A06            MOV AL,BYTE PTR DS:[ESI]
7353F802   46              INC ESI
7353F803   FF2485 58FA5373 JMP DWORD PTR DS:[EAX*4+7353FA58]
7353F80A   0FB706          MOVZX EAX,WORD PTR DS:[ESI]
```

其中：

```
7353F7BF   33C0            XOR EAX,EAX
7353F7C1   8A06            MOV AL,BYTE PTR DS:[ESI]
7353F7C3   46              INC ESI
7353F7C4   FF2485 58FA5373 JMP DWORD PTR DS:[EAX*4+7353FA58]
```

既是伪代码的执行模板。`JMP` 指令转入伪指令的执行，不断按下“F8”，盯着 `esi` 寄存器的变化，参照 `vbexplorer` 中提示的 p-code，你就不会迷失方向（通常是这样。当然，如果你按错了键，那就是你自己的事了，别怪我没提醒你）

### 大结局

关于最后的修改 VB P-CODE 代码，暴力破解法，据我所知一般只要能将重要的转移指令改变就可以了。无非是在您选好的比较指令的位置将 `BranchF`（伪代码 `1c`）`BranchT`(`1d`) 或者 `Branch`(`1e`)改来改去。别的好招我还不会呢，等着咱们继续切磋。

有空的朋友，还可以分析程序中的算法，只要花功夫，没什么看不懂的。除非编程序的那位憋着劲和咱们作对，不怕，咱不用他的软件了还不行？我是菜鸟，我怕谁？

够了，说了一堆有用没用的废话，您自己看着办 8，您也听累了，我手也酸了。总之，vb p-code 毕竟是由高级语言转换来的，应该不难分析和跟踪，只是特点不同，掌握了基本的要点，用 `OLLDBG` 跟踪没什么难的。现在又有 `WKTVBDE` 和 `vbexplorer` 助阵，一边捏着 P-CODE,一边瞄着 `ESI`, 祭起 OLLYDBG，一手按着 F8，您说，咱还怕谁呀？

什么逆向工程，什么 VB,  V JQK，谁是破解高手？不就是花时间嘛。

唉，所以高手都不来发言了，所以，您和我，还是菜鸟。苦恼中。。。。

要是那位高手作个 `OLLDBG` 的 VB P-CODE 插件，一遇到 VB p-code 中 `JMP DWORD PTR DS:[EAX*4+&*%$@$^]` 就显示相关的 P-CODE 指令就更方便了，尤其期待中。。。。

`4dd466d0e1e0cc123afbe504da2462ca`
