
valuetype [Microsoft.VisualBasic]Microsoft.VisualBasic.MsgBoxstyle,

object)

`IL_0279`:  `pop`

`IL_027a`:  `newobj`     instance void class

[CommonDll]r::.ctor()

`IL_027f`:  `stloc.s`    `V_10`

`IL_0281`:  `ldloc.s`    `V_10`

`IL_0283`:  `callvirt`   instance valuetype

[System.Windows.forms]System.Windows.forms.DialogResult class [System.Windows.forms]System.Windows.forms.form::ShowDialog()

`IL_0288`:  `pop`

`IL_0289`:  `call`       void class [Microsoft.VisualBasic]Microsoft.VisualBasic.CompilerServices.ProjectData::EndApp()

`IL_028e`:  `call`       void class [Microsoft.VisualBasic]Microsoft.VisualBasic.CompilerServices.ProjectData::ClearProjectError()

`IL_0293`:  `leave.s`    `IL_0295`

}  // end handler

`IL_0295`:  `ldarg.0`

`IL_0296`:  `ldfld`      class [CommonDll]x k::ap　　取注册表值

`IL_029b`:  `ldstr`      "TimesLeft"

`IL_02a0`:  `ldstr`      ""

`IL_02a5`:  `callvirt`   instance object class [CommonDll]x::a(string,

                  object)

`IL_02aa`:  `ldstr`      "820711"

`IL_02af`:  `ldc.i4.1`

`IL_02b0`:  `call`       int32 class [Microsoft.VisualBasic]Microsoft.VisualBasic.CompilerServices.ObjectType::ObjTst(object,

       object,

       bool)

`IL_02b5`:  `ldc.i4.0`

`IL_02b6`:  `bne.un`     `IL_040e`　　判断使用剩余次数是否为820711，不等则跳

.try

{　　　这是就是在commondll.dll中的判断过程

`IL_02bb`:  `newobj`     instance void class

[mscorlib]System.Security.Cryptography.RSACryptoServiceProvider::.ctor()

`IL_02c0`:  `stloc.s`    `V_15`

`IL_02c2`:  `ldloc.s`    `V_15`

`IL_02c4`:  `ldstr`      "<RSAKeyvalue><Modulus>mdmndcaDp9uHf27E+FDS5rrUfVUs"

     + "gasF83/P4RX2kzyVRyF+ugpazMc5X2sx9QsCrbeZ6VwLu4YzitMWNBuATQ==</Modulus><"

     + "Exponent>AQAB</Exponent></RSAKeyvalue>"

`IL_02c9`:  `callvirt`   instance void class [mscorlib]System.Security.Cryptography.RSA::FromXmlString(string)

`IL_02ce`:  `ldloc.s`    `V_15`

`IL_02d0`:  `newobj`     instance void class

[mscorlib]System.Security.Cryptography.RSAPKCS1SignatureDeformatter::.ctor(class

[mscorlib]System.Security.Cryptography.AsymmetricAlgorithm)

`IL_02d5`:  `stloc.s`    `V_16`

`IL_02d7`:  `ldloc.s`    `V_16`

`IL_02d9`:  `ldstr`      "SHA1"

`IL_02de`:  `callvirt`   instance void class [mscorlib]System.Security.Cryptography.RSAPKCS1SignatureDeformatter::SetHashAlgorithm(string)

`IL_02e3`:  `ldarg.0`

`IL_02e4`:  `ldfld`      class [CommonDll]x

k::ap

`IL_02e9`:  `callvirt`   instance int64 class [CommonDll]x::a()

`IL_02ee`:  `stloc.s`    `V_13`

`IL_02f0`:  `ldloc.s`    `V_13`

`IL_02f2`:  `ldc.i8`     0x1869f

`IL_02fb`:  `bge.s`      `IL_030f`

`IL_02fd`:  `ldstr`

     bytearray (E8 6C 8C 51 87 65 F6 4E 5F 63 4F 57 )

          // .l.Q.e.N_cOW

`IL_0302`:  `ldc.i4.0`

`IL_0303`:  `ldnull`

`IL_0304`:  `call`       valuetype

[Microsoft.VisualBasic]Microsoft.VisualBasic.MsgBoxResult class [Microsoft.VisualBasic]Microsoft.VisualBasic.Interaction::MsgBox(object,

           valuetype [Microsoft.VisualBasic]Microsoft.VisualBasic.MsgBoxstyle,

           object)

`IL_0309`:  `pop`

`IL_030a`:  `call`       void class [Microsoft.VisualBasic]Microsoft.VisualBasic.CompilerServices.ProjectData::EndApp()

`IL_030f`:  `ldarg.0`

`IL_0310`:  `ldfld`      class [CommonDll]x

k::ap

`IL_0315`:  `ldstr`      "SerialCode"

`IL_031a`:  `ldstr`      "a"

`IL_031f`:  `callvirt`   instance object class [CommonDll]x::a(string,

                    object)

`IL_0324`:  `call`       string class

[Microsoft.VisualBasic]Microsoft.VisualBasic.CompilerServices.StringType::FromObject(object)

`IL_0329`:  `stloc.s`    `V_14`

`IL_032b`:  `call`       class [mscorlib]System.Text.Encoding

class [mscorlib]System.Text.Encoding::get_Unicode()

`IL_0330`:  `ldloca.s`   `V_13`

`IL_0332`:  `call`       instance string

class [mscorlib]System.Int64::ToString()

`IL_0337`:  `callvirt`   instance unsigned int8[]

class [mscorlib]System.Text.Encoding::GetBytes(string)

`IL_033c`:  `stloc.s`    `V_11`

`IL_033e`:  `newobj`     instance void class

[mscorlib]System.Security.Cryptography.SHA1CryptoServiceProvider::.ctor()

`IL_0343`:  `stloc.s`    `V_17`

`IL_0345`:  `ldloc.s`    `V_17`

`IL_0347`:  `ldloc.s`    `V_11`

`IL_0349`:  `callvirt`   instance unsigned int8[]

class [mscorlib]System.Security.Cryptography.HashAlgorithm::ComputeHash(unsigned

int8[])

`IL_034e`:  `stloc.s`    `V_12`

`IL_0350`:  `ldloc.s`    `V_14`

`IL_0352`:  `call`       unsigned int8[]

class [mscorlib]System.Convert::FromBase64String(string)

`IL_0357`:  `stloc.s`    `V_18`

`IL_0359`:  `ldloc.s`    `V_16`

`IL_035b`:  `ldloc.s`    `V_12`

`IL_035d`:  `ldloc.s`    `V_18`

`IL_035f`:  `callvirt`   instance bool class [mscorlib]System.Security.Cryptography.RSAPKCS1SignatureDeformatter::VerifySignature(unsigned

int8[],

 unsigned int8[])

关键判断

`IL_0364`:  `brtrue.s`   `IL_03d0`　　　　验证通过则跳，否则下面修改注册信息并退出

`IL_0366`:  `ldarg.0`

`IL_0367`:  `ldfld`      class [CommonDll]x

k::ap

`IL_036c`:  `ldstr`      "TimesLeft"

`IL_0371`:  `ldc.i4.0`

`IL_0372`:  `call`       string class

[Microsoft.VisualBasic]Microsoft.VisualBasic.CompilerServices.StringType::FromInteger(int32)

`IL_0377`:  `callvirt`   instance void class [CommonDll]x::a(string,

                  string)

`IL_037c`:  `ldstr`      "..\\application.config"

`IL_0381`:  `ldc.i4.4`

`IL_0382`:  `ldc.i4.2`

`IL_0383`:  `ldc.i4.1`

`IL_0384`:  `newobj`     instance void class

[mscorlib]System.IO.FileStream::.ctor(string,

                  valuetype

[mscorlib]System.IO.FileMode,

                  valuetype

[mscorlib]System.IO.FileAccess,

                  valuetype

[mscorlib]System.IO.FileShare)

`IL_0389`:  `call`       class [mscorlib]System.Text.Encoding

class [mscorlib]System.Text.Encoding::get_UTF8()

`IL_038e`:  `newobj`     instance void class

[mscorlib]System.IO.StreamWriter::.ctor(class [mscorlib]System.IO.Stream,

                    class

[mscorlib]System.Text.Encoding)

`IL_0393`:  `stloc.s`    `V_20`

`IL_0395`:  `ldloc.s`    `V_20`

`IL_0397`:  `call`       class [mscorlib]System.Text.Encoding

class [mscorlib]System.Text.Encoding::get_Unicode()

`IL_039c`:  `ldstr`      "0"

`IL_03a1`:  `callvirt`   instance string string::ToString()

`IL_03a6`:  `callvirt`   instance unsigned int8[]

class [mscorlib]System.Text.Encoding::GetBytes(string)

`IL_03ab`:  `call`       string class

[mscorlib]System.Convert::ToBase64String(unsigned int8[])

`IL_03b0`:  `callvirt`   instance void class [mscorlib]System.IO.TextWriter::WriteLine(string)

`IL_03b5`:  `ldloc.s`    `V_20`

`IL_03b7`:  `callvirt`   instance void class [mscorlib]System.IO.StreamWriter::Close()

`IL_03bc`:  `newobj`     instance void class

[CommonDll]r::.ctor()

`IL_03c1`:  `stloc.s`    `V_19`

`IL_03c3`:  `ldloc.s`    `V_19`

`IL_03c5`:  `callvirt`   instance valuetype

[System.Windows.forms]System.Windows.forms.DialogResult class [System.Windows.forms]System.Windows.forms.form::ShowDialog()

`IL_03ca`:  `pop`

`IL_03cb`:  `call`       void class [Microsoft.VisualBasic]Microsoft.VisualBasic.CompilerServices.ProjectData::EndApp()

`IL_03d0`:  `leave`      `IL_056d`　　　　跳出

}  // end .try

catch [mscorlib]System.Exception

{

`IL_03d5`:  `call`       void class [Microsoft.VisualBasic]Microsoft.VisualBasic.CompilerServices.ProjectData::SetProjectError(class

[mscorlib]System.Exception)

`IL_03da`:  `ldarg.0`

`IL_03db`:  `ldfld`      class [CommonDll]x

k::ap

`IL_03e0`:  `ldstr`      "TimesLeft"

`IL_03e5`:  `ldc.i4.0`

`IL_03e6`:  `call`       string class

[Microsoft.VisualBasic]Microsoft.VisualBasic.CompilerServices.StringType::FromInteger(int32)

`IL_03eb`:  `callvirt`   instance void class [CommonDll]x::a(string,

                  string)

`IL_03f0`:  `newobj`     instance void class

[CommonDll]r::.ctor()

`IL_03f5`:  `stloc.s`    `V_21`

`IL_03f7`:  `ldloc.s`    `V_21`
