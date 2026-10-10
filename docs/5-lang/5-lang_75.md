

# .Net 环境下的程序破解(3)　　

> 原文：[`www.pediy.com/kssd/pediy06/pediy6115.htm`](https://www.pediy.com/kssd/pediy06/pediy6115.htm)

一直忙于论文，所以写到(2)就没有写下去了，最近刚好有点时间，就继续我们的《.net 环境下的程序破解》系列教程。  
现在.net 应该是个比较流行的平台了，网上已经开始大量出现.net 的控件和服务器软件，我想我们做破解的也应该紧跟时代潮流，呵呵：）  
本次主要将如何来静态分析，并且爆破一个很简单的 CrackMe。  
使用的工具是.net Framework SDK 自带的工具，ILasm、ILDasm，,这两个工具用于.net 下的汇编、反汇编，具体基本概念和工具用法参见教程(1,2)，CrackMe 在附件中。 

该 CrackMe 注册码正确试弹出对话框“Congratulations,Right Code!”，否则为"Wrong Code"。  
下面讲解如何破解该 CrackMe:  
(1)首先启动 ILDasm，导入 CrackMe.exe . 

(2)使用菜单 "File"->"Dump" (中文版的为 "文件"—>"转储"),将程序的 IL 代码导出为 crackme.il  
用文本编辑软件打开 crackme.il ,搜索"Congratulations,Right Code"，定位到这里，  
下面我用注释来解释这段 IL 的含义：

.method private hidebysig   
           instance void  button1\_Click(object sender,  
                                        class \[mscorlib\]System.EventArgse) cil managed  
   { 

     .maxstack  2  
     .locals  init (string V\_0)  
     IL\_0000:  ldarg.0  
     IL\_0001:  ldfld      class \[System.Windows.forms\]System.Windows.forms.TextBox CrackMe.form1::textBox1   
     IL\_0006:  callvirt   instance string \[System.Windows.forms\]System.Windows.forms.Control::get\_Text()   //取 textBox1 的 Text  
     IL\_000b:  stloc.0  
     IL\_000c:  ldloc.0  
     IL\_000d:  ldstr      "CCG-20030617"   
     IL\_0012:  callvirt   instance bool \[mscorlib\]System.String::Equals(string)  
     IL\_0017:  brfalse.s  IL\_002a  //这里是关键跳转，我想大家都看出来了 

     IL\_0019:  ldstr      "Congratulations,Right Code!"  
     IL\_001e:  ldstr      "CrackMe"   
     IL\_0023:  call       valuetype \[System.Windows.forms\]System.Windows.forms.DialogResult\[System.Windows.forms\]System.Windows.forms.MessageBox::Show(string,string)  //MessageBox  
     IL\_0028:  pop  
     IL\_0029:  ret                      //返回 return 

     IL\_002a:  ldstr      "Wrong Code!"   
     IL\_002f:  ldstr      "CrackMe"   
     IL\_0034:  call       valuetype \[System.Windows.forms\]System.Windows.forms.DialogResult\[System.Windows.forms\]System.Windows.forms.MessageBox::Show(string,string)  //MessageBox  
     IL\_0039:  pop  
     IL\_003a:  ret  
   } // end of method form1::button1\_Click  <---这是 ILDasm 自动生成的，告诉你这是 button1 按下去对应的 

这里我想大家就算不懂 IL，也能猜出来这段代码的含义，特别是 IL\_0017:  brfalse.s  IL\_002a，一看就是跳转语句，典型的 if .. else 判断。 当然，大家也猜出来注册码是"CCG-20030617" 了 

不过我想还是要解释一下 IL 里面的跳转语句，这是 Crack 必备知识。

BRTRUE 和 BRFALSE （(branch on false/branch on true）是常用的跳转语句，BRTRUE 和 BRFALSE 弹出堆栈最顶端的条目，然后根据该项为真（1）还是为假（0）而分别跳到指定的代码行。  
大家注意到 IL\_0017: brfalse.s IL\_002a 的上一条指令是 Call String::Equals 函数（事实上就是 strcmp），返回一个 bool 值，这个 bool 值存放在堆栈，然后 brfalse 把刚刚放进去的 bool 值从堆栈弹出来，判断如果是假，则跳转到 IL\_002a （也就是 MessageBox 提示出错的地方）。 

常用的跳转指令还有：  
BEQBranch on equal  
BGTBranch on greater than or equal  
BLEBranch on less than or equal  
BLTBranch on less than  
BNEBranch on not equal

这几条指令和 win32 平台的跳转指令都是对应的，我就不一一讲解了。

(3)爆破  
这个 CrackME 是非常简单的，一眼就可以看出注册码，然而现实是残酷的，所以我们还要学会 TNT 来使自己可以面对生活的挑战:)。  
当你还没有熟练 IL 的基本语法时，我想找到关键点并且爆破是最简单有效的办法。  
刚才这个 CrackME 我们已经找到关键点了，我们把 brfalse.s 改成 brtrue 就能轻松爆破了。  
然而，问题是你修改的是反汇编出来的 il 文件，并不是 exe，怎么办？  
这个问题.net framework sdk 里面早就准备好了解决方法，既然有 ildasm 就有 ilasm。

我们对 crackme.il 做了如下修改：  
IL\_0017: brtrue.s IL\_002a  
保存。

ilasm 是命令行方式的，我们使用命令：  
E:\\CrackWork>ilasm Crackme.il /resource=Crackme.res

马上得到下面结果：  
Copyright (C) Microsoft Corporation 1998-2002. All rights reserved.  
Assembling 'E:\\CrackWork\\Crackme.il' , n  
o listing file, to EXE --> 'E:\\CrackWork\\Crackme.EXE'  
Source file is UTF-8

Assembled method form1::.ctor  
Assembled method form1::Dispose  
Assembled method form1::InitializeComponent  
Assembled method form1::Main  
Assembled method form1::button1\_Click  
Creating PE file

Emitting members:  
Global  
Class 1 Fields: 5; Methods: 5;  
Resolving member refs: 37 -> 37 defs, 0 refs  
Writing PE file  
Operation completed successfully

马上就生成了 CrackMe.exe 文件，注意上面的参数中还要带上资源文件/resource=Crackme.res ，这个资源文件在 ildasm 进行反编译的时候自动生成，嘿嘿，大家一定又会想到，我们还可以利用这种方法来修改资源和汉化。  
运行新的 CrackMe.exe，输入任意字符串（只要不是正确的^\_^），都能成功注册。  
(CrackMe1.exe 插图)

好了，一个简单的 CrackMe 就这样搞定了，是不是觉得余味未尽啊，那不妨找一个软件来练练手：）

下次写写如何动态调试.net 下的程序，掌握了动态调试后，“调试”程序就更爽了～～ ：）

  
WinHack 2003-6-16  
QQ ：85436  
属于\[CCG\],\[OCN\],\[YCG\],\[FCG\]

  
 4dd466d0e1e0cc123afbe504da2462ca