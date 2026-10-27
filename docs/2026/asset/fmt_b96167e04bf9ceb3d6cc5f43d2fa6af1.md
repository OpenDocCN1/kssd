
```
     IL_00f0:  call       float64 class [mscorlib]System.Math::Round(float64)
     IL_00f5:  conv.ovf.i4
     IL_00f6:  stloc.s    V_5　　　　保存转换结果（1）
     IL_00f8:  ldloc.s    V_4
     IL_00fa:  callvirt   instance void class [mscorlib]System.IO.StreamReader::Close()
     IL_00ff:  ldloc.s    V_5
     IL_0101:  ldc.i4     0xc85e7　　　　十进制820711
     IL_0106:  beq.s      IL_014d　　　　相等的话就跳
     IL_0108:  ldloc.s    V_5
     IL_010a:  box        class [mscorlib]System.Int32
     IL_010f:  ldarg.0　　　以下修改注册表
     IL_0110:  ldfld      class [CommonDll]x k::ap
     IL_0115:  ldstr      "TimesLeft"
     IL_011a:  ldc.i4.s   50
     IL_011c:  box        class [mscorlib]System.Int32
     IL_0121:  callvirt   instance object class [CommonDll]x::a(string, object)
     IL_0126:  ldc.i4.1
     IL_0127:  call       int32 class [Microsoft.VisualBasic]Microsoft.VisualBasic.CompilerServices.ObjectType::ObjTst(object, object, bool)
     IL_012c:  ldc.i4.0
     IL_012d:  ble.s      IL_014d
     IL_012f:  ldarg.0
     IL_0130:  ldfld      class [CommonDll]x k::ap
     IL_0135:  ldstr      "TimesLeft"
     IL_013a:  ldc.i4.s   50
     IL_013c:  box        class [mscorlib]System.Int32
     IL_0141:  callvirt   instance object class [CommonDll]x::a(string, object)
     IL_0146:  call       int32 class [Microsoft.VisualBasic]Microsoft.VisualBasic.CompilerServices.IntegerType::FromObject(object)
     IL_014b:  stloc.s    V_5
     IL_014d:  ldarg.0
     IL_014e:  ldfld      class [CommonDll]x k::ap
     IL_0153:  callvirt   instance string class [CommonDll]x::c()另一个同名注册信息文件
     IL_0158:  call       bool class [mscorlib]System.IO.file::Exists(string)　存在？
     IL_015d:  brfalse.s  IL_019b　　　不存在则跳走，下面再读文件并转换
     IL_015f:  ldarg.0
     IL_0160:  ldfld      class [CommonDll]x k::ap
     IL_0165:  callvirt   instance string class [CommonDll]x::c()
     IL_016a:  call       class [mscorlib]System.IO.StreamReader class [mscorlib]System.IO.file::OpenText(string)
     IL_016f:  stloc.s    V_4
     IL_0171:  call       class [mscorlib]System.Text.Encoding class [mscorlib]System.Text.Encoding::get_Unicode()
     IL_0176:  ldloc.s    V_4
     IL_0178:  callvirt   instance string class [mscorlib]System.IO.StreamReader::ReadLine()
     IL_017d:  call       unsigned int8[] class [mscorlib]System.Convert::FromBase64String(string)
     IL_0182:  callvirt   instance string class [mscorlib]System.Text.Encoding::GetString(unsigned int8[])
     IL_0187:  call       float64 class [Microsoft.VisualBasic]Microsoft.VisualBasic.Conversion::Val(string)
     IL_018c:  call       float64 class [mscorlib]System.Math::Round(float64)
     IL_0191:  conv.ovf.i4
     IL_0192:  stloc.s    V_6　　　保存转换结果（2）
     IL_0194:  ldloc.s    V_4
     IL_0196:  callvirt   instance void class [mscorlib]System.IO.StreamReader::Close()
     IL_019b:  ldloc.s    V_6
     IL_019d:  ldloc.s    V_5
     IL_019f:  bge.s      IL_01a5　　比较转换结果（1）（2），取其中较小一个
     IL_01a1:  ldloc.s    V_6
     IL_01a3:  stloc.s    V_5
     IL_01a5:  ldloc.s    V_5
     IL_01a7:  ldc.i4.0
     IL_01a8:  bgt.s      IL_01d4　　　较小一个转换结果如果大于0，就跳
     IL_01aa:  ldarg.0　　　　　　　以下修改注册表后退出程序
     IL_01ab:  ldfld      class [CommonDll]x k::ap
     IL_01b0:  ldstr      "TimesLeft"
     IL_01b5:  ldc.i4.0
     IL_01b6:  call       string class [Microsoft.VisualBasic]Microsoft.VisualBasic.CompilerServices.StringType::FromInteger(int32)
     IL_01bb:  callvirt   instance void class [CommonDll]x::a(string, string)
     IL_01c0:  newobj     instance void class [CommonDll]r::.ctor()
     IL_01c5:  stloc.s    V_7
     IL_01c7:  ldloc.s    V_7
     IL_01c9:  callvirt   instance valuetype [System.Windows.forms]System.Windows.forms.DialogResult class [System.Windows.forms]System.Windows.forms.form::ShowDialog()
     IL_01ce:  pop
     IL_01cf:  call       void class [Microsoft.VisualBasic]Microsoft.VisualBasic.CompilerServices.ProjectData::EndApp()
     IL_01d4:  ldloc.s    V_5
     IL_01d6:  ldc.i4     0xc85e7
     IL_01db:  beq        IL_0266　　　是否等于820711，等于则跳
     IL_01e0:  ldloc.s    V_5
     IL_01e2:  ldc.i4.1
     IL_01e3:  sub.ovf　　　　　　　　　否则使用次数减1
     IL_01e4:  stloc.s    V_5
     IL_01e6:  ldstr      "..\\application.config"　　　以下修改注册信息文件
     IL_01eb:  ldc.i4.4
     IL_01ec:  ldc.i4.2
     IL_01ed:  ldc.i4.1
     IL_01ee:  newobj     instance void class [mscorlib]System.IO.FileStream::.ctor(string, valuetype [mscorlib]System.IO.FileMode, valuetype [mscorlib]System.IO.FileAccess, valuetype [mscorlib]System.IO.FileShare)
     IL_01f3:  call       class [mscorlib]System.Text.Encoding class [mscorlib]System.Text.Encoding::get_UTF8()
     IL_01f8:  newobj     instance void class [mscorlib]System.IO.StreamWriter::.ctor(class [mscorlib]System.IO.Stream, class [mscorlib]System.Text.Encoding)
     IL_01fd:  stloc.s    V_8
     IL_01ff:  ldloc.s    V_8
     IL_0201:  call       class [mscorlib]System.Text.Encoding class [mscorlib]System.Text.Encoding::get_Unicode()
     IL_0206:  ldloca.s   V_5
     IL_0208:  call       instance string class [mscorlib]System.Int32::ToString()
     IL_020d:  callvirt   instance unsigned int8[] class [mscorlib]System.Text.Encoding::GetBytes(string)
     IL_0212:  call       string class [mscorlib]System.Convert::ToBase64String(unsigned int8[])
     IL_0217:  callvirt   instance void class [mscorlib]System.IO.TextWriter::WriteLine(string)
     IL_021c:  ldloc.s    V_8
     IL_021e:  callvirt   instance void class [mscorlib]System.IO.StreamWriter::Close()
     IL_0223:  ldarg.0
     IL_0224:  ldfld      class [CommonDll]x k::ap
     IL_0229:  callvirt   instance string class [CommonDll]x::c()
     IL_022e:  ldc.i4.4
     IL_022f:  ldc.i4.2
     IL_0230:  ldc.i4.1
     IL_0231:  newobj     instance void class [mscorlib]System.IO.FileStream::.ctor(string, valuetype [mscorlib]System.IO.FileMode, valuetype [mscorlib]System.IO.FileAccess, valuetype [mscorlib]System.IO.FileShare)
     IL_0236:  call       class [mscorlib]System.Text.Encoding class [mscorlib]System.Text.Encoding::get_UTF8()
     IL_023b:  newobj     instance void class [mscorlib]System.IO.StreamWriter::.ctor(class [mscorlib]System.IO.Stream, class [mscorlib]System.Text.Encoding)
     IL_0240:  stloc.s    V_8
     IL_0242:  ldloc.s    V_8
     IL_0244:  call       class [mscorlib]System.Text.Encoding class [mscorlib]System.Text.Encoding::get_Unicode()
     IL_0249:  ldloca.s   V_5
     IL_024b:  call       instance string class [mscorlib]System.Int32::ToString()
     IL_0250:  callvirt   instance unsigned int8[] class [mscorlib]System.Text.Encoding::GetBytes(string)
     IL_0255:  call       string class [mscorlib]System.Convert::ToBase64String(unsigned int8[])
     IL_025a:  callvirt   instance void class [mscorlib]System.IO.TextWriter::WriteLine(string)
     IL_025f:  ldloc.s    V_8
     IL_0261:  callvirt   instance void class [mscorlib]System.IO.StreamWriter::Close()
     IL_0266:  leave.s    IL_0295　　　　　　　跳出
   }  // end .try
   catch [mscorlib]System.Exception 　　　　　　　　程序异常处理
   {
     IL_0268:  dup
     IL_0269:  call       void class [Microsoft.VisualBasic]Microsoft.VisualBasic.CompilerServices.ProjectData::SetProjectError(class [mscorlib]System.Exception)
     IL_026e:  stloc.s    V_9
     IL_0270:  ldloc.s    V_9
     IL_0272:  ldc.i4.0
     IL_0273:  ldnull
     IL_0274:  call       valuetype [Microsoft.VisualBasic]Microsoft.VisualBasic.MsgBoxResult class [Microsoft.VisualBasic]Microsoft.VisualBasic.Interaction::MsgBox(object,
```
