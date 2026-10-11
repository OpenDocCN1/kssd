
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
