
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
