
# 有关VB程序P-CODE代码逆向工程入门浅说

有关VB程序P-CODE代码逆向工程入门浅说

对于破解初学者来说，可能在调试VB（P-CODE）编写的程序时感到难于分析和跟踪，特别是查找和定位“注册码比较”关键点，以及修补代码等。小弟对VB程序内部构造同样也是一塌糊涂，对破解更是一知半解，和各位初学者一样，只是感兴趣而已。偶尔尝试调试，跟踪个VB程序玩玩，没什么目的。闲着没事，咱就随便扯点个人浅显的经验，仅供和我一样的菜鸟同道参考，让各位高手见笑了。

特别声明：本文仅仅服务于对VB P-CODE程序破解感兴趣的初学者，绝非专业学术研究，不提供任何有关研究分析的科学方法和步骤。

注：如果您仅仅对OLLYDBG调试VB P-CODE感兴趣，请直接参考第4部分。

## 1．缘起

有天，您正在网上溜达，忽然发现一个感兴趣的共享小程序，想拿来用用，可还没怎么试，它就跟您较劲儿，老是让您注册，烦不烦？咱能受这个限制吗？… 既然玩保护，那咱们就帮作者测试测试保护的强度，呵呵，考验咱们的时候又到了。。。3下5/2, 经过您的火眼金睛一看，原来是个VB（P-CODE）捏把的东东。怎么看出是P-CODE? 方法很多，比如拿WKTVBDE要是能LOAD，我想它就是了，什么?不知道WKTVBDE是什么？那用Exdec反一下，什么，这个也没听说过？那咱们先别往下继续了，您最好先了解一下有关的预备知识，也可以在本论坛搜索相关vb p-code破解案例分析。擅自推荐您参考“FLY”大侠的破解实例。

对手出现了，怎么办？要是我，可能准备如下工具：

- VBExplorer (静态分析P－CODE的好东西，又免费，建议作者继续完善，坚持不收费政策50年不变 ^&^ )
- WKTVBDE (VB P-CODE动态跟踪调试器，在下一般用它来动态修改代码，判断分析的效果。要想用它真正搞清楚程序的细节，小弟还没有发现好办法，但作为粗略的分析工具，查看程序分支判断，循环次数，它还是蛮好用的。对于分析VB P-CODE，它有如一个大砂轮，先用它磨一磨，精细加工，那是下一个将介绍的工具)
- Ollydbg(很不错的调试器，我用1.09D。好在它的兼容性和便捷性，在我的WIN2003 sever上表现很稳定，跟踪，调试VB P-CODE再合适不过了，只可惜不是咱们中国人的产品。在本短文稍后，咱们主要就讨论如何用它跟踪分析VB P-CODE，因为这个步骤也算蛮重要嘛)

还有其它一些可以选择的工具，就看您的个人爱好了，比如：

- Exdec vb p-code代码反编译器，不错的工具
- Vbparser 1.2 同样是 vb p-code反编译器，不过我用的版本不能反VB 5, 有些vb 6的也不行，不知道是怎么回事。

还有什么好工具希望您告诉我。

## 2．鬼穷三技

聊斋上说，小鬼儿也没什么可怕的，也就有3招，你知道了，对付它就心里有了底，有了战胜它的信心。话说VB P-CODE程序，咱们这些新手外行如何搞定它呢？咱们也来个3招克敌:

a.静态分析：粗略搞清楚目标程序的大致流程，分支，凭分析结果和编程，破解的经验，判断和假设目标程序的设计思路

b.代码还原：根据p-code伪代码，大致可以还原出VB源程序代码（针对关键和重要的部分，对于破解，就是还原判断有关的和比较注册码前后部分的VB代码）

c.动态跟踪：应用源码级调试器（例如olldbg）跟踪程序的运行细节，发现重要数据，甚至注册码等信息

空口无凭，咱们还是通过一个简单案例来演示一下整个的VB P-CODE逆向过程，如何？

## 3．循序渐进

为了演示方便，咱们先写一个简单的VB注册小程序，源代码如下(程序未经任何优化，仅用于演示)：

```
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

程序仅仅根据用户输入的数字判断是否符合内定的规则，如果是，显示：good,错误，则提示：bad。程序很简单，您自己看一下就明白了，咱们就不再详细解释了，不耽误您的时间了。

这段小程序对应的P-CODE代码如下（使用VBExplorer获得，相关程序可以从本论坛得到，或联系作者）：

```
[Text1.Change]

:00401B9C  F401                LitI2_Byte           ;Push 01

:00401B9E  FC0D                CUI1I2               ;

:00401BA0  047AFF              FLdRfVar             ;Push LOCAL_0086

:00401BA3  0470FF              FLdRfVar             ;Push LOCAL_0090

:00401BA6  21                  FLdPrThis            ;[SR]=[stack2]

:00401BA7  0F0003              VCallAd              ;Return the control index 02

:00401BAA  1974FF              FStAdFunc            ;

For i = 1 To Len(Text1.Text)

:00401BAD  0874FF              FLdPr                ;[SR]=[LOCAL_008C]

**********Reference To:[propget]TextBox.Text

:00401BB0  0DA0000000          VCallHresult         ;Call ptr_004016B8

:00401BB5  6C70FF              ILdRf                ;Push DWORD [LOCAL_0090]

:00401BB8  4A                  FnLenStr    ；取得输入串的长度，作为循环的参数

:00401BB9  FC0E                CUI1I4               ;

:00401BBB  2F70FF              FFree1Str            ;SysFreeString [LOCAL_0090];

:00401BBE  1A74FF              FFree1Ad             ;Push [LOCAL_008C];

:00401BC1  FE626CFFD900        ForUI1      ; 循环开始

:00401BC7  0470FF              FLdRfVar             ;Push LOCAL_0090

:00401BCA  21                  FLdPrThis            ;[SR]=[stack2]

:00401BCB  0F0003              VCallAd              ;Return the control index 02

:00401BCE  1974FF              FStAdFunc            ;

:00401BD1  0874FF              FLdPr                ;[SR]=[LOCAL_008C]

**********Reference To:[propget]TextBox.Text  ；出现这个提示，说明程序中参照了输入框内容

:00401BD4  0DA0000000          VCallHresult         ;Call ptr_004016B8

If Mid(Text1.Text, i, 1) > "9" Or Mid(Text1.Text, i, 1) < "0" Then

:00401BD9  0404FF              FLdRfVar             ;Push LOCAL_00FC

:00401BDC  21                  FLdPrThis            ;[SR]=[stack2]

:00401BDD  0F0003              VCallAd              ;Return the control index 02

:00401BE0  1908FF              FStAdFunc            ;

:00401BE3  0808FF              FLdPr                ;[SR]=[LOCAL_00F8]

**********Reference To:[propget]TextBox.Text

:00401BE6  0DA0000000          VCallHresult         ;Call ptr_004016B8

:00401BEB  283CFF0100          LitVarI2    ;Push 数字 1

:00401BF0  FCE07AFF            FLdUI1               ;

:00401BF4  E7                  CI4UI1      ;Push 变量 i

:00401BF5  3E70FF              FLdZeroAd            ;

:00401BF8  465CFF              CVarStr              ;

:00401BFC  042CFF              FLdRfVar             ;

**********Reference To->msvbvm60.rtcMidCharVar       ；调用了Mid 函数，一般相关的参数在调用前已经入栈，注意上边两个PUSH指令，参数入栈顺序是从右到左，对于MID，既是压入1，i。意思是取从第i个开始的字符，总共取1个。

:00401BFE  0A01001000          ImpAdCallFPR4        ;Call ptr_00401020;

:00401C03  042CFF              FLdRfVar             ;Push LOCAL_00D4

******Possible String Ref To->"9"    ；表示”9”这个文本字符被引用

:00401C06  3A1CFF0200          LitVarStr            ;PushVarString ptr_004016CC

:00401C0B  5D                  HardType             ;

:00401C0C  FB700CFF            GtVar                ;

:00401C10  28D4FE0100          LitVarI2 ;Push 数字 1

:00401C15  FCE07AFF            FLdUI1               ;

:00401C19  E7                  CI4UI1   ;Push 变量 i

:00401C1A  3E04FF              FLdZeroAd            ;

:00401C1D  46F4FE              CVarStr              ;

:00401C20  04C4FE              FLdRfVar             ;

**********Reference To->msvbvm60.rtcMidCharVar

:00401C23  0A01001000          ImpAdCallFPR4        ;Call ptr_00401020;

:00401C28  04C4FE              FLdRfVar             ;Push LOCAL_013C

******Possible String Ref To->"0"

:00401C2B  3AB4FE0300          LitVarStr ; 表示”0”文本字符串被引用

:00401C30  5D                  HardType             ;

:00401C31  FB63A4FE            LtVar                ;

:00401C35  FB1F94FE            OrVar     ; or 另一个条件

:00401C39  FF1B                CBoolVarNull         ;vbaBoolVarNull

:00401C3B  29040074FF08FF      FFreeAd              ;

:00401C42  360C005CFF3CFF2C    FFreeVar             ;Free 000C/2 variants

:00401C51  1CD000              BranchF   ；比较指令，

******Possible String Ref To->"Please enter 0-9, try again"

Text2.Text = "Please enter 0-9, try again"        ；如果输入字符无效，提示重新输入

:00401C54  1B0400              LitStr               ;Push ptr_004016DC

:00401C57  21                  FLdPrThis            ;[SR]=[stack2]

:00401C58  0F0803              VCallAd              ;Return the control index 04

:00401C5B  1974FF              FStAdFunc            ;

:00401C5E  0874FF              FLdPr                ;[SR]=[LOCAL_008C]

**********Reference To:[propput]TextBox.Text

:00401C61  0DA4000000          VCallHresult         ;Call ptr_004016B8

:00401C66  1A74FF              FFree1Ad             ;

GoTo final_end

:00401C69  1E1002              Branch               ;ESI=00401DAC

:00401C6C  047AFF              FLdRfVar             ;Push LOCAL_0086

Next

:00401C6F  FE786CFF2B00        NextUI1       ; 循环部分中止

:00401C75  0470FF              FLdRfVar             ;Push LOCAL_0090

:00401C78  21                  FLdPrThis            ;[SR]=[stack2]

:00401C79  0F0003              VCallAd              ;Return the control index 02

:00401C7C  1974FF              FStAdFunc            ;

If Len(Text1.Text) >= 3 Then

:00401C7F  0874FF              FLdPr                ;[SR]=[LOCAL_008C]

**********Reference To:[propget]TextBox.Text

:00401C82  0DA0000000          VCallHresult         ;Call ptr_004016B8

:00401C87  6C70FF              ILdRf                ;Push DWORD [LOCAL_0090]

:00401C8A  4A                  FnLenStr     ; 输入字符串长度

:00401C8B  F503000000          LitI4        ；常数3用于比较

:00401C90  E0                  GeI4                 ;
```
