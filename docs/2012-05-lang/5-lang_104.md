
# 极速 IP 搜索 II

> 原文：[`www.pediy.com/kssd/pediy05/pediy50366.htm`](https://www.pediy.com/kssd/pediy05/pediy50366.htm)

## 概述

- **软件制作平台**: VS C#.NET
- **破解人**: DarkNess0ut[FCG]
- **目的**: 了解 VS.NET 软件的破解, 制作注册机
- **声明**: 本文章只做研究使用!!

由于我答应了法语助手的作者，不再发布它的破解补丁文件:)，所以本来要写 VB.NET 的文章改成了这个 C# 的文章，反正菩提兄写了详细的 VB.NET 的破解了:)

## 过程

1. 找到 `ILDASM.exe` (好像在 vs.net 的第三张光盘吧)，打开!
2. 拖入 `SpeedII` 主程序，开始反汇编...OK
3. 找到注册相关的地方 (`regB_Click` 或 启动监测处)，打开!

拷贝的代码如下:

```
由于不做爆破,所以就不显示字节代码了,主要是算法分析(爆破也简单,一两个字节就行了)

.method private hidebysig instance void  regB_Click(object sender, 
                                                        class [mscorlib]System.EventArgs e) cil managed 
{ 
  // Code size      331 (0x14b) 
  .maxstack  5      //堆栈 5 个 
  .locals init (char[] V_0, 
               char[] V_1, 
               int32 V_2,     0 
               int32 V_3,     1 
               int32 V_4,     2 
               string V_5, 
               int32 V_6, 
               bool V_7, 
               class [mscorlib]System.IO.StreamWriter V_8, 
               class [mscorlib]System.Exception V_9) 
  .try 
  { 
    IL_0000:  ldc.i4.s  20    //压入 20 
    IL_0002:  newarr    [mscorlib]System.Char    //申明数组 
    IL_0007:  stloc.0    //char[]=new str0[20] 
    IL_0008:  ldc.i4.s  20    //压入 20 
    IL_000a:  newarr    [mscorlib]System.Char    //申明数组 
    IL_000f:  stloc.1    //char[]=new str1[20] 
    IL_0010:  ldarg.0       Arg0 
    IL_0011:  ldfld      class [System.Windows.Forms]System.Windows.Forms.TextBox SpeedIP_II.Reg::userName    //很明显,取用户名 
    IL_0016:  callvirt  instance string [System.Windows.Forms]System.Windows.Forms.Control::get_Text() 
    IL_001b:  callvirt  instance char[] [mscorlib]System.String::ToCharArray() 
    IL_0020:  stloc.0    //str0=username.tochararray(); 将用户名数组=>str0 

&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&//开始第一组计算 
    IL_0021:  ldc.i4.0       //push 0 我按照字符解释为 load (下同) 
    IL_0022:  stloc.2    //set loc.2=0 
    IL_0023:  br.s      IL_0040 jump 40 
    IL_0025:  ldloc.1       //load str1 
    IL_0026:  ldloc.2       //load loc.2 
>>>IL_0027:  ldloc.0       //load str0 
    IL_0028:  ldloc.2       //load loc.2 
    IL_0029:  ldelem.u2       //[] 
             //load str0[loc.2] 
  /IL_002a:  ldloc.2       /load loc.2 
  |IL_002b:  ldloc.0       // load str0 
/ |IL_002c:  ldlen       // len(str0) 
| |IL_002d:  conv.i4   =======//i4=len(str0) 
| \IL_002e:  add       //add 
| /IL_002f:  ldloc.0   =========// load loc.0 
  |IL_0030:  ldlen       // len(loc.0) 
| \IL_0031:  conv.i4   =========//i4=len(loc.0) 
\  IL_0032:  mul       // * 
    IL_0033:  add       // + 
    IL_0034:  ldc.i4.s  26    // load 26 
    IL_0036:  rem       // rem???这是什么 
    IL_0037:  ldc.i4.s  65    // load 65 
>>>IL_0039:  add       // add 
    IL_003a:  conv.u2       //??? 
    IL_003b:  stelem.i2       //[] 
    IL_003c:  ldloc.2       //load loc.2 
    IL_003d:  ldc.i4.1       //1 
    IL_003e:  add       //I++ 
    IL_003f:  stloc.2       //set loc.2= 
    IL_0040:  ldloc.2       //load loc.2 
    IL_0041:  ldloc.0    //load loc.0 
    IL_0042:  ldlen   //求长度 
    IL_0043:  conv.i4       //i4=len(name) 
    IL_0044:  blt.s      IL_0025    //if loc.2<i4 then jump 0025 

&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&& 
```

初看上面的代码觉得很乱，和菩提那篇介绍的不很一样，而且没有很多的提示，觉得很难理解

呵呵，后来我才发现了原因，解决的方法比较简单，

这个和那些写的很乱的代码很像，如果对上述代码做个下小小的变化

内容就显而易见了.

### 变化如下

```
IL_0021:  ldc.i4.0       //push 0 我按照字符 ld 解释为 load (下同) 
    IL_0022:  stloc.2    //set loc.2=0 
    IL_0023:  br.s      IL_0040 jump 40 
    IL_0025:  ldloc.1       //load str1 
    IL_0026:  ldloc.2       //load loc.2 
  { 
    { 
        { 
        IL_0027:  ldloc.0       //load str0 
        IL_0028:  ldloc.2       //load loc.2 
        IL_0029:  ldelem.u2       //[] 
                     //load str0[loc.2] arg0 
        } 
        { 
          { 
              IL_002a:  ldloc.2       /load loc.2 arg1    
              { 
                IL_002b:  ldloc.0       // load str0     //三行看作一个 arg 
                IL_002c:  ldlen       // len(str0) 
                IL_002d:  conv.i4   =======//i4=len(str0) arg2 
              }    
              IL_002e:  add       //add 运算 
              //也就是 arg1+arg2=>arg1+len(str0)===> arg3 
          }   
          { 
              IL_002f:  ldloc.0   =========// load loc.0 
                IL_0030:  ldlen       // len(loc.0) 
                IL_0031:  conv.i4   =========//i4=len(loc.0) 
          }   
                     arg4=len(str0) 
              IL_0032:  mul       // * 运算 fsarg3*arg4=>arg5 
        } 
        IL_0033:  add       // + 
    } 
    IL_0034:  ldc.i4.s  26    // load 26 
    IL_0036:  rem       // rem???这是什么 这个就是取模运算啦 
             //当初还以为是注释语句,哈哈,汇编代码也要注释?呵呵 
    IL_0037:  ldc.i4.s  65    // load 65 
    IL_0039:  add       // add 
} 
    IL_003a:  conv.u2       //??? 哦这个就是将上述的结果 conv 一下 
    IL_003b:  stelem.i2       //[] set elem i2? 
    结合上面的语句不难看出 
    //IL_0025:  ldloc.1       //load str1 
    //IL_0026:  ldloc.2       //load loc.2 
    //RESULT 
    //setelem.i2 
    //哦,str1[loc.2]=RESULT 
    IL_003c:  ldloc.2       //load loc.2 
    IL_003d:  ldc.i4.1       //1 
    IL_003e:  add       //I++ 
    IL_003f:  stloc.2       //set loc.2=loc.2+1 
    //很明显,循环索引值嘛 
    IL_0040:  ldloc.2       //load loc.2 
    IL_0041:  ldloc.0    //load loc.0 
    IL_0042:  ldlen   //求长度 
    IL_0043:  conv.i4       //i4=len(name) 
    IL_0044:  blt.s      IL_0025    //if loc.2<i4 then jump 0025 
    分析,上述过程之所以看起来很复杂的原因是因为使用了套嵌,也就是编程使用了(),哈哈 
所以上述的过程的算法为,VB 表示 
str0=name 
for i=0 to len(str0)-1 
    str1[lndex]=((str0[index]+(index+len(str0))*len(str0)) mod 26) +65 
next 
作用是将注册码限制到[A-Z]之间 
看懂了这一段的话,下面的两段代码简直和看源代码一样啦 
后面不作详细解释,我将各段用空行隔开了,只要记住 
数据操作总是这样的形式 
    push arg1 
    push arg2 
    operate 
找到两个参数,和一个运算就行了,简单的很, 
也就是所说的堆栈操作了,没有寄存器,看起来很别扭:) 
就是套嵌看起来很复杂 
要是可以做到和 vs.net 的代码操作一样,可以将代码收缩的话,哈哈! 
```

\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

```
    IL_0046:  ldloc.0 
    IL_0047:  ldlen 
    IL_0048:  conv.i4 
    IL_0049:  stloc.3    //set loc.3=len(str0) //Index 
    IL_004a:  br.s      IL_0061    //Jump 
    IL_004c:  ldloc.1    //str1[i] 
    IL_004d:  ldloc.3    
    IL_004e:  ldloc.0 
    IL_004f:  ldc.i4.0 
    IL_0050:  ldelem.u2 
    IL_0051:  ldloc.3 
    IL_0052:  mul 
    IL_0053:  ldloc.3 
    IL_0054:  mul 
    IL_0055:  ldc.i4.s  26 
    IL_0057:  rem 
    IL_0058:  ldc.i4.s  65 
    IL_005a:  add 
    IL_005b:  conv.u2 
    IL_005c:  stelem.i2 
    IL_005d:  ldloc.3    //i 
    IL_005e:  ldc.i4.1    //1 
    IL_005f:  add    //i=i+1 
    IL_0060:  stloc.3 
    IL_0061:  ldloc.3           //round 
    IL_0062:  ldc.i4.s  10    //?<10 
    IL_0064:  blt.s      IL_004c 
```

\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

上述运算

```vb
for Index=len(str0) to 10-1 
    str1[Index]=(str0[0]*Index*Index mod 26) +65 
next 
```

\============================================

```
    IL_0066:  ldc.i4.s  10 
    IL_0068:  stloc.s    V_4 
    IL_006a:  br.s      IL_0087 
    IL_006c:  ldloc.1       //str1 
    IL_006d:  ldloc.s    V_4 
    IL_006f:  ldloc.0 
    IL_0070:  ldloc.0 
    IL_0071:  ldlen   
```



```
IL_0072:  conv.i4
IL_0073:  ldc.i4.1
IL_0074:  sub
IL_0075:  ldelem.u2
IL_0076:  ldloc.s    V_4
IL_0078:  mul
IL_0079:  ldc.i4.s   26
IL_007b:  rem
IL_007c:  ldc.i4.s   65
IL_007e:  add
IL_007f:  conv.u2
IL_0080:  stelem.i2
IL_0081:  ldloc.s    V_4
IL_0083:  ldc.i4.1
IL_0084:  add
IL_0085:  stloc.s    V_4
IL_0087:  ldloc.s    V_4
IL_0089:  ldc.i4.s   20
IL_008b:  blt.s      IL_006c
```

==============================================

For `Index=10` to `20-1`

`str1[Index]=(str0[len(str0)-1]*Index mod 26) +65`

Next

到此,注册码的计算完成了,呵呵,还是较简单的吧

一共 20 位的注册码,这种算法是不行地:)

@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@

```
IL_008d:  ldstr      "SpeedIPII-"
IL_0092:  stloc.s    V_5
IL_0094:  ldc.i4.0
IL_0095:  stloc.s    V_6
IL_0097:  br.s      IL_00b5
IL_0099:  ldloc.s    V_5
IL_009b:  ldloc.1
IL_009c:  ldloc.s    V_6
IL_009e:  ldelema    [mscorlib]System.Char
IL_00a3:  call       instance string [mscorlib]System.Char::ToString()
IL_00a8:  call       string [mscorlib]System.String::Concat(string, string)
IL_00ad:  stloc.s    V_5 //最后将"SpeedIPII-" + 注册码=>完整的注册码
IL_00af:  ldloc.s    V_6
IL_00b1:  ldc.i4.1
IL_00b2:  add
IL_00b3:  stloc.s    V_6
IL_00b5:  ldloc.s    V_6
IL_00b7:  ldc.i4.s   20
IL_00b9:  blt.s      IL_0099    //采用循环读取每个字节的方式
```

@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@

```
IL_00bb:  ldloc.s    V_5
IL_00bd:  ldarg.0
IL_00be:  ldfld      class [System.Windows.Forms]System.Windows.Forms.TextBox SpeedIP_II.Reg::userCode    //读取输入的注册码
IL_00c3:  callvirt   instance string [System.Windows.Forms]System.Windows.Forms.Control::get_Text()
IL_00c8:  callvirt   instance bool [mscorlib]System.String::Equals(string)    //=?
IL_00cd:  stloc.s    V_7
IL_00cf:  ldloc.s    V_7
IL_00d1:  brfalse.s  IL_0122    // 如果不.则跳到 122 给你警告啦! 爆破也在这里啦
```

不过应该在启动的时候爆破,这里没有意义

```
IL_00d3:  ldstr      "ProgrameMessage.dll"
IL_00d8:  call       class [mscorlib]System.IO.StreamWriter [mscorlib]System.IO.File::CreateText(string)
IL_00dd:  stloc.s    V_8
IL_00df:  ldloc.s    V_8
IL_00e1:  ldarg.0
IL_00e2:  ldfld      class [System.Windows.Forms]System.Windows.Forms.TextBox SpeedIP_II.Reg::userName
IL_00e7:  callvirt   instance string [System.Windows.Forms]System.Windows.Forms.Control::get_Text()
IL_00ec:  callvirt   instance void [mscorlib]System.IO.TextWriter::WriteLine(string)
IL_00f1:  ldloc.s    V_8
IL_00f3:  ldarg.0
IL_00f4:  ldfld      class [System.Windows.Forms]System.Windows.Forms.TextBox SpeedIP_II.Reg::userCode
IL_00f9:  callvirt   instance string [System.Windows.Forms]System.Windows.Forms.Control::get_Text()
IL_00fe:  callvirt   instance void [mscorlib]System.IO.TextWriter::WriteLine(string)
IL_0103:  ldloc.s    V_8
IL_0105:  callvirt   instance void [mscorlib]System.IO.TextWriter::Close()
IL_010a:  ldstr      bytearray (E8 6C 8C 51 10 62 9F 52 01 FF 1F 61 22 8C A8 60  // .l.Q.b.R...a"..\`
                  84 76 E8 6C 8C 51 01 FF 08 FF F7 8B CD 91 B0 65  // .v.l.Q.........e
                  2F 54 A8 52 6F 8F F6 4E 09 FF )                  // /T.Ro..N..
```

//上面的这段数组,猜都能猜到是信息文字啦,那是什么呢,怎么看?

//很简单阿,用一个 Unicode=>ASCII 的显示工具就可以看到了

//正好做了一个小工具,显示看看

//"注册成功！感谢您的注册！（请重新启动软件）"

//哈哈,等的就是这个啦

```
IL_010f:  ldstr      bytearray (D0 63 3A 79 E1 4F 6F 60 )                        // .c:y.Oo\`
IL_0114:  call       valuetype [System.Windows.Forms]System.Windows.Forms.DialogResult [System.Windows.Forms]System.Windows.Forms.MessageBox::Show(string, string)
IL_0119:  pop
IL_011a:  ldarg.0
IL_011b:  call       instance void [System.Windows.Forms]System.Windows.Forms.Form::Close()
IL_0120:  br.s       IL_0132
IL_0122:  ldstr      bytearray (F9 5B 0D 4E 77 8D 01 FF A8 60 40 62 93 8F 65 51  // .[.Nw....\`@b..eQ
                  84 76 28 75 37 62 0D 54 8C 54 E8 6C 8C 51 01 78  // .v(u7b.T.T.l.Q.x
                  0D 4E 26 7B 01 FF )                                // .N&{..
IL_0127:  ldstr      bytearray (D0 63 3A 79 E1 4F 6F 60 )                        // .c:y.Oo\`
IL_012c:  call       valuetype [System.Windows.Forms]System.Windows.Forms.DialogResult [System.Windows.Forms]System.Windows.Forms.MessageBox::Show(string, string)
IL_0131:  pop
```

//"对不起！您所输入的用户名和注册码不符！"

```
IL_0132:  leave.s    IL_014a
} // end .try

catch [mscorlib]System.Exception
{
IL_0134:  stloc.s    V_9
IL_0136:  ldloc.s    V_9
IL_0138:  callvirt   instance string [mscorlib]System.Exception::get_Message()
IL_013d:  ldstr      bytearray (D0 63 3A 79 E1 4F 6F 60 )                        // .c:y.Oo\`
IL_0142:  call       valuetype [System.Windows.Forms]System.Windows.Forms.DialogResult [System.Windows.Forms]System.Windows.Forms.MessageBox::Show(string, string)
IL_0147:  pop
IL_0148:  leave.s    IL_014a
} // end handler

IL_014a:  ret
} // end of method Reg::regB_Click
```

这是继法语助手 Vb.net 后我破解的第二个.net 软件,

不过由于使用了一句语句的运算,使得结构看起来比较乱,还好搞定了!:)

哈哈,发文章了,好长时间没写了,手生了!

注册机的话,用 VBs 写就行了,反正简单,不写了!:) 4dd466d0e1e0cc123afbe504da2462ca
