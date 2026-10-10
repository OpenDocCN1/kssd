

# \].Net 加混淆器的分析

> 原文：[`www.pediy.com/kssd/pediy06/pediy6233.htm`](https://www.pediy.com/kssd/pediy06/pediy6233.htm)

先贴一段代码,是一个简单的运算.然后用混淆器进行业务逻辑的混淆...   
基本上没有起到作用,代码如下.   
        Dim a, b, c, d As String   
        d = 50   
        c = d   
        b = c   
        a = b   
        b = 20   
        d = 30   
        Dim s As Decimal   
        s = a + b + c \* d   
        MsgBox(s)   
\--------------------------------   
对比混淆后   
\--------------------------------   
private void a(object A\_0, EventArgs A\_1)   
{ string text1;   
string text2;   
string text3;   
string text4;   
decimal num1;   
text4 = StringType.FromInteger(50);   
text3 = text4;   
text2 = text3;   
text1 = text2;   
text2 = StringType.FromInteger(20);   
text4 = StringType.FromInteger(30);   
num1 = new decimal((DoubleType.FromString(string.Concat(text1, text2)) + (DoubleType.FromString(text3) \* DoubleType.FromString(text4))));   
Interaction.MsgBox(num1, 0, null);   
    
}    
\-------------------------- 

  

下面开始慢慢混淆了,看看...   
源代码为:   
    Private Sub Button1\_Click(ByVal sender As System.Object, ByVal e As System.EventArgs) Handles Button1.Click   
        Dim a, b, c, d As Integer   
        d = 50   
        c = d   
        b = c   
        a = b   
        b = 20   
        d = 30   
        Dim s As Decimal   
        Swap(d, c)   
        s = add(a, add(b, c)) \* c   
        MsgBox(s)   
    End Sub   
  
    Function add(ByVal a As Integer, ByVal b As Integer) As Integer   
        Return a + b   
    End Function   
  
    Function Swap(ByRef a As Integer, ByRef b As Integer)   
        Dim aa As Integer   
        aa = a   
        a = b   
        b = aa   
    End Function   
\--------------------------------------------   
之后为   
\--------------------------------------------   
private void a(object A\_0, EventArgs A\_1)   
{ int num1;   
int num2;   
int num3;   
int num4;   
decimal num5;   
num4 = 50;   
num3 = num4;   
num2 = num3;   
num1 = num2;   
num2 = 20;   
num4 = 30;   
this.a(&(num4), &(num3));   
num5 = new decimal((this.a(num1, this.a(num2, num3)) \* num3));   
Interaction.MsgBox(num5, 0, null);   
    
}    
  
public object a(ref int A\_0, ref int A\_1)   
{ int num1;   
object obj1;   
num1 = A\_0;   
A\_0 = A\_1;   
A\_1 = num1;   
return obj1;    
}    
  
public int a(int A\_0, int A\_1)   
{ int num1;   
return (A\_0 + A\_1);    
}    
\-----------------------------   
读一读?是不是有些困难了些,当然,我是把代码都直接粘出来了,你读起来还方便点,但是...如果从几十个或是上面个....   
比方说,有意的制作一个大的 Class,再把 Class 中混淆...哇.头痛的事来了. 

  

算了,不继续了,没时间去写很复杂的...反正大家兴一反三吧.   
决窍就是调用...功能写在函数,写成类,使用对象.类与类之间可能有关系...当然,还是结构逻辑第一...不要为了故意混淆搞到自己都看不懂...或看起来麻烦.   
  
有些事情可以在项目完成的那段时间再做   
  
有不正确之处,请指出...我对这个研究也很肤浅,如果有什么想法也可以提出...说不定咱们自己还可以搞个什么比这个更好的东东...  
 4dd466d0e1e0cc123afbe504da2462ca