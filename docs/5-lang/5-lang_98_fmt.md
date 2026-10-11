
# 程序轻松驾考 2004 V1.0

> 原文：[`www.pediy.com/kssd/pediy06/pediy6611.htm`](https://www.pediy.com/kssd/pediy06/pediy6611.htm)

**【破解作者】** `blue_devil_bomb[DFCG][BCG]`  
**【作者邮箱】** `ninesunnine@sina.com`  
**【作者主页】** `www.chinadfcg.com`  
**【使用工具】** `ILDASM`, `HEXWORKSHOP`  
**【破解平台】** `Win2000`  
**【软件名称】** 轻松驾考 2004 1.0  
**【下载地址】** 中国共享软件注册中心  

**【软件简介】** 关于本软件

## 1.1 系统功能

《轻松驾考 2004》主要目的是帮助您熟悉机动车驾驶证理论考试的有关试题，使您在较短的时间内有针对性地了解和掌握驾照理论考试的有关内容。系统还提供最新交通法规、现行交通标志、顺序练习、模拟测试等功能。我们相信，通过使用本系统将对您顺利通过驾照理论考试有相当的帮助。

## 1.2 系统特点

本系统采用公安部 2004 年 7 月份公布的 04 版科目一题库，并按照所学车型和试题种类进行了详细的分类，提高了练习的针对性。

## 1.3 系统要求

本系统采用 `VB.net` 编写，客户端必须安装 `DotNetFX1.1` 和 `MDAC2.7` 以上版本，最好安装声卡，系统的配置也不能太低。

**【软件大小】** `160kb`  
**【加壳方式】** 未  
**【破解声明】** 我是一只小菜鸟，偶得一点心得，愿与大家分享：）

---

**【破解内容】**

前几天一位朋友要考驾照，便找到该软件，由于需要注册。无奈之下我破解了它。可由于该软件为 `VB.NET` 编写的程序，本人还没有接触过，便在看雪学院上找到了相关的文章，在此还要感谢！

由了前人破解的经验，我们只需要依葫芦画瓢即可，由于 `.NET` 方面破解文章较少，小弟不才，不敢一人独享，放上来与大家共享。

该程序未注册启动时显示为如下图：

背景知识介绍：首先，我们遇到的是一个全新的东西，`.NET` 框架，这个东西是什么呢，他怎么工作的，如果你用 `WIN32DASM` 反汇编一下这个程序，你会看到莫名其妙的代码，因为它不再是我们传统意义上的汇编代码了，它是 `.NET` 的核心也就是 `MSIL` 语言，`IL` 全称是 `INTERMEDIATE LANGUAGE`，也就是一种中间语言，类似 `PCODE`，但这回所有的编程语言包括 `VB`, `VC`, `C#` 全都编译成 `MSIL` 然后再利用 `.NET` 框架的 `JIT` 实时转译成本地机的可执行代码，从而实现了平台无关性，于是我们要破解这类程序就要找到一种类似 `PCODE` 反编译器的东西，很幸运的是微软大叔为我们提供了全套超强绿色套装：）对我们最有用的就是 `ILDASM`（这些东西在 `.NET FRAMEWORK SDK` 中提供）

好了基本的东西知道了一点，现在开始就要了解这种语言了，当然微软的 `MSDN` 有所有你想知道的东西，所以我这里就不罗索了，你可以去看看他的教程，我就来讲讲具体的破解，这才是我等感兴趣的：）

首先用 `ILDASM` 把源程序载入，很快我们得到了反编译的代码，你看到的是一种树状的结构图：

很清晰的结构，完全采用类的结构，通过名字我们判断 `frm_reg` 这个类就是关键所在,在这个类中找到 `btn_reg_click` 双击进入如下：

```csharp
.method private instance void btn_reg_Click(object sender,
                                                class [mscorlib]System.EventArgs e) cil managed
{
  // 代码大小       513 (0x201)
  .maxstack  6
  .locals init ([0] string Reg,
                 [1] string regcode,
                 [2] string RegStr,
                 [3] class [ADODB]ADODB.Connection cn,
                 [4] class [ADODB]ADODB.Recordset rs,
                 [5] string sql,
                 [6] class [mscorlib]System.Exception err,
                 [7] string[] _Vb_t_array_0)
  IL_0000:  nop
  IL_0001:  ldarg.0
  IL_0002:  callvirt   instance class [System.Windows.Forms]System.Windows.Forms.TextBox DSES.frm_reg::get_txt_chr()
  IL_0007:  callvirt   instance string [System.Windows.Forms]System.Windows.Forms.TextBox::get_Text()
  IL_000c:  stloc.0
  IL_000d:  ldloc.0
  IL_000e:  call       int32 [Microsoft.VisualBasic]Microsoft.VisualBasic.Strings::Len(string)
  IL_0013:  ldc.i4.s   29
  IL_0015:  beq.s      IL_0030
  IL_0017:  ldstr      bytearray (79 72 81 5F 01 78 19 95 EF 8B 0C FF E0 65 D5 6C   // yr._.x.......e.l
                              8C 5B 10 62 E8 6C 8C 51 01 FF )                   // .[.b.l.Q..
  IL_001c:  ldstr      bytearray (19 95 EF 8B )
  IL_0021:  ldc.i4.0
  IL_0022:  ldc.i4.s   48
  IL_0024:  call       valuetype [System.Windows.Forms]System.Windows.Forms.DialogResult [System.Windows.Forms]System.Windows.Forms.MessageBox::Show(string,
                                                                                  string,
                                                                                  valuetype [System.Windows.Forms]System.Windows.Forms.MessageBoxButtons,
                                                                                  valuetype [System.Windows.Forms]System.Windows.Forms.MessageBoxIcon)
  IL_0029:  pop
  IL_002a:  nop
  IL_002b:  br         IL_01ff
  IL_0030:  nop
  IL_0031:  ldc.i4.5
  IL_0032:  newarr     [mscorlib]System.String
  IL_0037:  stloc.s    _Vb_t_array_0
  IL_0039:  ldloc.s    _Vb_t_array_0
  IL_003b:  ldc.i4.0
  IL_003c:  ldloc.0
  IL_003d:  ldc.i4.1
  IL_003e:  ldc.i4.5
  IL_003f:  call       string [Microsoft.VisualBasic]Microsoft.VisualBasic.Strings::Mid(string,
                                                                                        int32,
                                                                                        int32)
  IL_0044:  call       int32 [Microsoft.VisualBasic]Microsoft.VisualBasic.CompilerServices.IntegerType::FromString(string)
  IL_0049:  ldc.i4.1
  IL_004a:  sub.ovf
  IL_004b:  call       char [Microsoft.VisualBasic]Microsoft.VisualBasic.Strings::ChrW(int32)
  IL_0050:  call       string [Microsoft.VisualBasic]Microsoft.VisualBasic.CompilerServices.StringType::FromChar(char)
  IL_0055:  stelem.ref
  IL_0056:  nop
  IL_0057:  ldloc.s    _Vb_t_array_0
  IL_0059:  ldc.i4.1
  IL_005a:  ldloc.0
  IL_005b:  ldc.i4.7
  IL_005c:  ldc.i4.5
  IL_005d:  call       string [Microsoft.VisualBasic]Microsoft.VisualBasic.Strings::Mid(string,
                                                                                        int32,
                                                                                        int32)
  IL_0062:  call       int32 [Microsoft.VisualBasic]Microsoft.VisualBasic.CompilerServices.IntegerType::FromString(string)
  IL_0067:  ldc.i4.2
  IL_0068:  sub.ovf
  IL_0069:  call       char [Microsoft.VisualBasic]Microsoft.VisualBasic.Strings::ChrW(int32)
  IL_006e:  call       string [Microsoft.VisualBasic]Microsoft.VisualBasic.CompilerServices.StringType::FromChar(char)
  IL_0073:  stelem.ref
  IL_0074:  nop
  IL_0075:  ldloc.s    _Vb_t_array_0
  IL_0077:  ldc.i4.2
  IL_0078:  ldloc.0
  IL_0079:  ldc.i4.s   12
  IL_007b:  ldc.i4.5
  IL_007c:  call       string [Microsoft.VisualBasic]Microsoft.VisualBasic.Strings::Mid(string,
                                                                                        int32,
                                                                                        int32)
  IL_0081:  call       int32 [Microsoft.VisualBasic]Microsoft.VisualBasic.CompilerServices.IntegerType::FromString(string)
  IL_0086:  ldc.i4.3
  IL_0087:  sub.ovf
  IL_0088:  call       char [Microsoft.VisualBasic]Microsoft.VisualBasic.Strings::ChrW(int32)
  IL_008d:  call       string [Microsoft.VisualBasic]Microsoft.VisualBasic.CompilerServices.StringType::FromChar(char)
  IL_0092:  stelem.ref
  IL_0093:  nop
  IL_0094:  ldloc.s    _Vb_t_array_0
  IL_0096:  ldc.i4.3
  IL_0097:  ldloc.0
  IL_0098:  ldc.i4.s   19
  IL_009a:  ldc.i4.5
  IL_009b:  call       string [Microsoft.VisualBasic]Microsoft.VisualBasic.Strings::Mid(string,
                                                                                        int32,
                                                                                        int32)
  IL_00a0:  call       int32 [Microsoft.VisualBasic]Microsoft.VisualBasic.CompilerServices.IntegerType::FromString(string)
  IL_00a5:  ldc.i4.4
  IL_00a6:  sub.ovf
  IL_00a7:  call       char [Microsoft.VisualBasic]Microsoft.VisualBasic.Strings::ChrW(int32)
  IL_00ac:  call       string [Microsoft.VisualBasic]Microsoft.VisualBasic.CompilerServices.StringType::FromChar(char)
  IL_00b1:  stelem.ref
  IL_00b2:  nop
  IL_00b3:  ldloc.s    _Vb_t_array_0
  IL_00b5:  ldc.i4.4
  IL_00b6:  ldloc.0
  IL_00b7:  ldc.i4.s   25
  IL_00b9:  ldc.i4.5
  IL_00ba:  call       string [Microsoft.VisualBasic]Microsoft.VisualBasic.Strings::Mid(string,
                                                                                        int32,
                                                                                        int32)
}
```



```
IL_00bf:   call       int32 [Microsoft.VisualBasic]Microsoft.VisualBasic.CompilerServices.IntegerType::FromString(string)  
IL_00c4:   ldc.i4.5  
IL_00c5:   sub.ovf  
IL_00c6:   call       char [Microsoft.VisualBasic]Microsoft.VisualBasic.Strings::ChrW(int32)  
IL_00cb:   call       string [Microsoft.VisualBasic]Microsoft.VisualBasic.CompilerServices.StringType::FromChar(char)  
IL_00d0:   stelem.ref  
IL_00d1:   nop  
IL_00d2:   ldloc.s    _Vb_t_array_0  
IL_00d4:   call       string [mscorlib]System.String::Concat(string[])  
IL_00d9:   stloc.2  
IL_00da:   ldarg.0  
IL_00db:   ldloc.2  
IL_00dc:   callvirt   instance string DSES.frm_reg::CFS(string)  
IL_00e1:   stloc.1  
IL_00e2:   ldarg.0  
IL_00e3:   callvirt   instance class [System.Windows.Forms]System.Windows.Forms.TextBox DSES.frm_reg::get_txt_reg()  
IL_00e8:   callvirt   instance string [System.Windows.Forms]System.Windows.Forms.TextBox::get_Text()  
IL_00ed:   ldloc.1  
IL_00ee:   ldc.i4.0  
IL_00ef:   call       int32 [Microsoft.VisualBasic]Microsoft.VisualBasic.CompilerServices.StringType::StrCmp(string,  
                                                           string,  
IL_00f4:   /* 16   |                  */ ldc.i4.0  
IL_00f5:   /* 40   | F0000000         */ bne.un     IL_01ea  
IL_00fa:   /* 72   | (70)0006F4       */ ldstr      bytearray (6D 60 9C 55 0C FF E8 6C 8C 51 10 62 9F 52 01 FF   // m`.U...l.Q.b.R..  
                                                F7 8B 00 90 FA 51 6F 8F F6 4E 0E 54 CD 91 B0 65   // .....Qo..N.T...e  
                                                D0 8F 4C 88 6F 8F F6 4E E5 67 0B 77 E8 6C 8C 51   // ..L.o..N.g.w.l.Q  
                                                2F 66 26 54 10 62 9F 52 01 FF )                   // /f&T.b.R..  
```

利用 `WIN2000` 自带的字符映射表我们查出，这段字串的意思为：恭喜，注册成功！请退出软件重新启动.....

我们在 `IL_00f5` 处看到 `bne.un   IL_01ea` 即不等的话跳转到 `IL_01ea` 处，而 `IL_01ea` 处的字串的意思为：“注册码错误，无法完成注册！”我们只需将此 `NOP` 掉，即什么都不做就成功了。

用 `HEXWORKSHOP` 打开该程序，查找 `HEX` 值 `1640f000000072f4060070` 改为：`16400000000072f4060070` 即可。

至此我们可以接过董存瑞手中的炸药包，轰轰轰，从此整个世界清静了。

```
IL_00ff:   ldstr      bytearray (6F 8F F6 4E E8 6C 8C 51 )                       // o..N.l.Q  
IL_0104:   ldc.i4.0  
IL_0105:   ldc.i4.s   64  
IL_0107:   call       valuetype [System.Windows.Forms]System.Windows.Forms.DialogResult [System.Windows.Forms]System.Windows.Forms.MessageBox::Show(string,  
                                                                                                                             string,  
                                                                                                                            valuetype [System.Windows.Forms]System.Windows.Forms.MessageBoxButtons,  
                                                                                                                            valuetype [System.Windows.Forms]System.Windows.Forms.MessageBoxIcon)  
IL_010c:   pop  
IL_010d:   newobj     instance void [ADODB]ADODB.ConnectionClass::.ctor()  
IL_0112:   stloc.3  
IL_0113:   ldloc.3  
IL_0114:   ldsfld     string DSES.frm_main::connstr  
IL_0119:   callvirt   instance void [ADODB]ADODB._Connection::set_ConnectionString(string)  
IL_011e:   nop  
IL_011f:   nop  
  
.try  
  
{  
  .try  
  {  
    IL_0120:   ldloc.3  
    IL_0121:   ldstr      ""  
    IL_0126:   ldstr      ""  
    IL_012b:   ldstr      ""  
    IL_0130:   ldc.i4.m1  
    IL_0131:   callvirt   instance void [ADODB]ADODB._Connection::Open(string,  
                                                                      string,  
                                                                      string,  
                                                                      int32)  
    IL_0136:   nop  
    IL_0137:   newobj     instance void [ADODB]ADODB.RecordsetClass::.ctor()  
    IL_013c:   stloc.s    rs  
    IL_013e:   ldstr      "select top 1 * from [system]"  
    IL_0143:   stloc.s    sql  
    IL_0145:   ldloc.s    rs  
    IL_0147:   ldloc.s    sql  
    IL_0149:   ldloc.3  
    IL_014a:   ldc.i4.1  
    IL_014b:   ldc.i4.3  
    IL_014c:   ldc.i4.m1  
    IL_014d:   callvirt   instance void [ADODB]ADODB._Recordset::Open(object,  
                                                                     object,  
                                                                     valuetype [ADODB]ADODB.CursorTypeEnum,  
                                                                     valuetype [ADODB]ADODB.LockTypeEnum,  
                                                                     int32)  
    IL_0152:   nop  
    IL_0153:   ldloc.s    rs  
    IL_0155:   callvirt   instance bool [ADODB]ADODB._Recordset::get_EOF()  
    IL_015a:   ldloc.s    rs  
    IL_015c:   callvirt   instance bool [ADODB]ADODB._Recordset::get_BOF()  
    IL_0161:   and  
    IL_0162:   brfalse.s  IL_017f  
    IL_0164:   ldstr      bytearray (70 65 6E 63 93 5E 19 95 EF 8B 0C FF B9 70 FB 51   // penc.^.......p.Q  
                                               6E 78 9A 5B 00 90 FA 51 01 FF F7 8B C0 68 E5 67   // nx.[...Q.....h.g  
                                               FB 7C DF 7E 4D 91 6E 7F 84 76 63 6B 6E 78 27 60   // .|.~M.n..vcknx'`  
                                               0C FF 82 59 9C 67 EE 95 98 98 9D 4F 36 71 58 5B   // ...Y.g.....O6qX[  
                                               28 57 F7 8B 0E 4E 6F 8F F6 4E 9B 4F 94 5E 46 55   // (W...No..N.O.^FU  
                                               54 80 FB 7C 02 30 )                               // T..|.0  
    IL_0169:   ldstr      bytearray (FB 7C DF 7E 19 95 EF 8B )                       // .|.~....  
    IL_016e:   ldc.i4.0  
    IL_016f:   ldc.i4.s   48  
    IL_0171:   call       valuetype [System.Windows.Forms]System.Windows.Forms.DialogResult [System.Windows.Forms]System.Windows.Forms.MessageBox::Show(string,  
                                                                                                                            string,  
                                                                                                                            valuetype [System.Windows.Forms]System.Windows.Forms.MessageBoxButtons,  
                                                                                                                            valuetype [System.Windows.Forms]System.Windows.Forms.MessageBoxIcon)  
    IL_0176:   pop  
    IL_0177:   nop  
    IL_0178:   leave      IL_01ff  
    IL_017d:   br.s       IL_01aa  
    IL_017f:   nop  
    IL_0180:   ldloc.s    rs  
    IL_0182:   callvirt   instance class [ADODB]ADODB.Fields [ADODB]ADODB._Recordset::get_Fields()  
    IL_0187:   ldstr      "serialno"  
    IL_018c:   callvirt   instance class [ADODB]ADODB.Field [ADODB]ADODB.Fields::get_Item(object)  
    IL_0191:   ldloc.1  
    IL_0192:   callvirt   instance void [ADODB]ADODB.Field::set_Value(object)  
    IL_0197:   nop  
    IL_0198:   ldloc.s    rs  
    IL_019a:   ldsfld     class [mscorlib]System.Reflection.Missing [mscorlib]System.Reflection.Missing::Value  
    IL_019f:   ldsfld     class [mscorlib]System.Reflection.Missing [mscorlib]System.Reflection.Missing::Value  
    IL_01a4:   callvirt   instance void [ADODB]ADODB._Recordset::Update(object,  
                                                                        object)  
    IL_01a9:   nop  
    IL_01aa:   nop  
```



```
      IL\_01ab:  ldloc.s    rs  

      IL\_01ad:  callvirt   instance void \[ADODB\]ADODB.\_Recordset::Close()  

      IL\_01b2:  nop  

      IL\_01b3:  ldnull  

      IL\_01b4:  stloc.s    rs  

      IL\_01b6:  leave.s    IL\_01e0  

    }  // end .try  

    catch \[mscorlib\]System.Exception  

    {  

      IL\_01b8:  dup  

      IL\_01b9:  call       void \[Microsoft.VisualBasic\]Microsoft.VisualBasic.CompilerServices.ProjectData::SetProjectError(class \[mscorlib\]System.Exception)  

      IL\_01be:  stloc.s    err  

      IL\_01c0:  nop  

      IL\_01c1:  ldloc.s    err  

      IL\_01c3:  callvirt   instance string \[mscorlib\]System.Exception::ToString()  

      IL\_01c8:  call       valuetype \[System.Windows.Forms\]System.Windows.Forms.DialogResult \[System.Windows.Forms\]System.Windows.Forms.MessageBox::Show(string)  

      IL\_01cd:  pop  

      IL\_01ce:  call       void \[Microsoft.VisualBasic\]Microsoft.VisualBasic.CompilerServices.ProjectData::ClearProjectError()  

      IL\_01d3:  leave.s    IL\_01e0  

    }  // end handler  

  }  // end .try  

  finally  

  {  

    IL\_01d5:  nop  

    IL\_01d6:  ldloc.3  

    IL\_01d7:  callvirt   instance void \[ADODB\]ADODB.\_Connection::Close()  

    IL\_01dc:  nop  

    IL\_01dd:  ldnull  

    IL\_01de:  stloc.3  

    IL\_01df:  endfinally  

  }  // end handler  

  IL\_01e0:  nop  

  IL\_01e1:  ldarg.0  

  IL\_01e2:  callvirt   instance void \[System.Windows.Forms\]System.Windows.Forms.Form::Close()  

  IL\_01e7:  nop  

  IL\_01e8:  br.s       IL\_01fe  

  IL\_01ea:  nop  

  IL\_01eb:  ldstr      bytearray (E8 6C 8C 51 01 78 19 95 EF 8B 0C FF E0 65 D5 6C   // .l.Q.x.......e.l  

                             8C 5B 10 62 E8 6C 8C 51 01 FF )                         // .\[.b.l.Q..  

  该处的字串的意思为：注册码错误，无法完成注册！  

  IL\_01f0:  ldstr      bytearray (6F 8F F6 4E E8 6C 8C 51 )                         // o..N.l.Q  

  IL\_01f5:  ldc.i4.0  

  IL\_01f6:  ldc.i4.s   48  

  IL\_01f8:  call       valuetype \[System.Windows.Forms\]System.Windows.Forms.DialogResult \[System.Windows.Forms\]System.Windows.Forms.MessageBox::Show(string,  
                             string,  
                             valuetype \[System.Windows.Forms\]System.Windows.Forms.MessageBoxButtons,  
                             valuetype \[System.Windows.Forms\]System.Windows.Forms.MessageBoxIcon)  

  IL\_01fd:  pop  

  IL\_01fe:  nop  

  IL\_01ff:  nop  

  IL\_0200:  ret  

} // end of method frm\_reg::btn\_reg\_Click  
```

--------------------------------------------------------------------------------

### 破解总结

由于本人是第一次破解 .net 程序，算法还没去看，有时间在看吧！

爆破方法为：用 `HEXWORKSHOP` 打开该程序，查找 `HEX` 值 `1640f000000072f4060070` 改为：`16400000000072f4060070` 即可。

--------------------------------------------------------------------------------

### 版权声明

本文纯属技术交流，转载请注明作者并保持文章的完整，谢谢！

`4dd466d0e1e0cc123afbe504da2462ca`
