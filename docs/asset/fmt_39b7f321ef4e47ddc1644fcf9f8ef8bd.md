
### 代码部分

```
matrix:='8x3p5BeabcdfghijklmnoqrstuvwyzACDEFGHIJKLMNOPQRSTUVWXYZ1246790';   

machinecode:=Form1.FlatEdit1.Text;   

key:= Form1.FlatEdit2.Text;   

lm:=strlen(pchar(machinecode));   

lk:=strlen(pchar(key));   

i:=1;   

resultstr:='';   

while i<=lm do   

    begin   

      if i<=lk then t:=i   

      else t:=i-lk\*((i-1) div lk);   

      m:=pos(key\[t\],matrix)-pos(machinecode\[i\],matrix);   

      if m<=0 then   

        resultpos:=m+63   

      else   

        resultpos:=m+1;   

      resultstr:=resultstr+copy(matrix,resultpos,1);   

      i:=i+1;   

    end;   

Form1.FlatEdit3.Text:=resultstr;   

end;   
```

### 分析与调试思路

算法完成了，但是我原先的程序里是错位的，也就是算出的注册码都比正确的前移了一个，我一直没找到原因，如果有个能动态调试的工具可以让我看到推栈里的一个值，我就可以纠正这个错误，但是没有（有个 `cordbg` 不会用，也不知道能不能达到目的：），所以我有了个想法，点注册的时候你会发现他弹出一个错误提示框，能不能利用他把正确的注册码弹出来，我不就可以知道错在哪了吗（如果我没出错，就不会有下面的 `diy msil` 了：）

### 第二部分：`diy msil`

我们现在要找到那个注册按钮事件，这样我们才能加入自己的处理，来看看在哪里：就在 `frmRegistry::BtnRegistry_Click`，

```
.method private instance void  BtnRegistry_Click(object sender,   
                                                class [mscorlib]System.EventArgs e) cil managed   

{   

  // Code size   247 (0xf7)   

  .maxstack  4   

  IL_0000:  nop   

  IL_0001:  ldarg.0   

  IL_0002:  callvirt  instance class [System.Windows.Forms]System.Windows.Forms.TextBox WinXP_Manager.frmRegistry::get_txtSN()   

  IL_0007:  callvirt  instance string [System.Windows.Forms]System.Windows.Forms.TextBox::get_Text()//得到你输入的注册码   

  IL_000c:  ldstr      ""   

  IL_0011:  ldc.i4.0   

  IL_0012:  call       int32 [Microsoft.VisualBasic]Microsoft.VisualBasic.CompilerServices.StringType::StrCmp(string,   

                                                                                                          string,   

                                                                                                          bool)   

  IL_0017:  ldc.i4.0   

  IL_0018:  bne.un.s  IL_0033  //比较是不是空，不是就跳，是就输出错误信息   

  IL_001a:  ldstr      bytearray (A8 60 D8 8F A1 6C 09 67 93 8F 65 51 E8 6C 8C 51  // .`...l.g..eQ.l.Q   

                                  01 78 62 54 21 00 )                           // .xbT!.   

  IL_001f:  ldstr      bytearray (19 95 EF 8B )   

  IL_0024:  ldc.i4.0   

  IL_0025:  ldc.i4.s  16   

  IL_0027:  call       valuetype [System.Windows.Forms]System.Windows.Forms.DialogResult [System.Windows.Forms]System.Windows.Forms.MessageBox::Show(string,   

                                                                                                                  string,   

                                                                                                          valuetype [System.Windows.Forms]System.Windows.Forms.MessageBoxButtons,   

                                                                                                          valuetype [System.Windows.Forms]System.Windows.Forms.MessageBoxIcon)   

  IL_002c:  pop   

  IL_002d:  nop   

  IL_002e:  br        IL_00f5   

  IL_0033:  nop   

  IL_0034:  ldarg.0   

  IL_0035:  callvirt  instance class [System.Windows.Forms]System.Windows.Forms.TextBox WinXP_Manager.frmRegistry::get_txtUserName()   

  IL_003a:  callvirt  instance string [System.Windows.Forms]System.Windows.Forms.TextBox::get_Text() //得到用户名   

  IL_003f:  ldstr      ""   

  IL_0044:  ldc.i4.0   

  IL_0045:  call       int32 [Microsoft.VisualBasic]Microsoft.VisualBasic.CompilerServices.StringType::StrCmp(string,   

                                                                                                          string,   

                                                                                                          bool)   

  IL_004a:  ldc.i4.0   

  IL_004b:  bne.un.s  IL_0066  //比较是不是空，不是就跳，是就输出错误信息   

  IL_004d:  ldstr      bytearray (F7 8B 28 75 57 5B CD 6B FB 6D 99 51 A8 60 84 76  // ..(uW[.k.m.Q.`.v   

                                  28 75 37 62 0D 54 21 00 )                       // (u7b.T!.   

  IL_0052:  ldstr      bytearray (19 95 EF 8B )   

  IL_0057:  ldc.i4.0   

  IL_0058:  ldc.i4.s  16   

  IL_005a:  call       valuetype [System.Windows.Forms]System.Windows.Forms.DialogResult [System.Windows.Forms]System.Windows.Forms.MessageBox::Show(string,   

                                                                                                                  string,   

                                                                                                          valuetype [System.Windows.Forms]System.Windows.Forms.MessageBoxButtons,   

                                                                                                          valuetype [System.Windows.Forms]System.Windows.Forms.MessageBoxIcon)   

  IL_005f:  pop   

  IL_0060:  nop   

  IL_0061:  br        IL_00f5   

  IL_0066:  nop   

  IL_0067:  ldarg.0   

  IL_0068:  callvirt  instance class [System.Windows.Forms]System.Windows.Forms.TextBox WinXP_Manager.frmRegistry::get_txtSN()   

  IL_006d:  callvirt  instance string [System.Windows.Forms]System.Windows.Forms.TextBox::get_Text()   

  IL_0072:  ldarg.0   

  IL_0073:  callvirt  instance class [System.Windows.Forms]System.Windows.Forms.TextBox WinXP_Manager.frmRegistry::get_txtUserName()   

  IL_0078:  callvirt  instance string [System.Windows.Forms]System.Windows.Forms.TextBox::get_Text()   

  IL_007d:  call       string WinXP_Manager.MyClsOther/MyClsRegister::FromHDSNGetRegisterCode(string)  //这里会得到这确的注册码然后与我们输入的作比较   

  IL_0082:  ldc.i4.0   

  IL_0083:  call       int32 [Microsoft.VisualBasic]Microsoft.VisualBasic.CompilerServices.StringType::StrCmp(string,   

                                                                                                          string,   

                                                                                                          bool)   

  IL_0088:  ldc.i4.0   

  IL_0089:  bne.un.s  IL_00c3  //不等就跳去显示出错框   

  IL_008b:  ldarg.0   

  IL_008c:  callvirt  instance class [System.Windows.Forms]System.Windows.Forms.TextBox WinXP_Manager.frmRegistry::get_txtSN()   

  IL_0091:  callvirt  instance string [System.Windows.Forms]System.Windows.Forms.TextBox::get_Text()   

  IL_0096:  ldarg.0   

  IL_0097:  callvirt  instance class [System.Windows.Forms]System.Windows.Forms.TextBox WinXP_Manager.frmRegistry::get_txtUserName()   

  IL_009c:  callvirt  instance string [System.Windows.Forms]System.Windows.Forms.TextBox::get_Text()   

  IL_00a1:  call       void WinXP_Manager.MyClsOther/MyClsRegister::WriteRegisterCode(string,   

                                                                                  string)   

  IL_00a6:  nop   

  IL_00a7:  ldstr      bytearray (A8 60 F2 5D CF 7E 10 62 9F 52 E8 6C 8C 51 21 00  // .`.\].~.b.R.l.Q!.   

                                  F7 8B CD 91 B0 65 2F 54 A8 52 2C 67 6F 8F F6 4E  // .....e/T.R,go..N   

                                  2E 00 )                                        // ..   

  IL_00ac:  ldstr      bytearray (E8 6C 8C 51 8C 5B 10 62 )                       // .l.Q.[.b   

  IL_00b1:  ldc.i4.0   

  IL_00b2:  ldc.i4.s  64   

  IL_00b4:  call       valuetype [System.Windows.Forms]System.Windows.Forms.DialogResult [System.Windows.Forms]System.Windows.Forms.MessageBox::Show(string,   

                                                                                                                  string,   
```
