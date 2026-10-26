
# [原创]初步分析AxProtector V6.40.226.200Dotnet加壳与脱壳

已测环境 :

`XP SP3`

原版 `OD`

`OllyICE v1.10`[汉化第二版]，`LordPE`(`LordPE Deluxe by youda`)，
`CFF Explorer II`，`Windbg:6.11.0001.404 X86`，`.net Reflector V6.6.0.30`，`WinHex 15.1`，`IL`字节码解码工具`V 2.0`。

`WinDbg`

`AxProtector`

`AxProtector` 是一款德国瑞步的加密狗`dotNet`加壳器。产品介绍去自己看`252K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4N6A6j5Y4g2Q4x3X3g2#2M7#2)9J5c8Y4m8J5L8$3c8#2j5%4c8K6i4K6u0r3j5i4S2H3M7X3!0@1k6h3y4@1L8%4u0Q4x3X3g2Z5N6r3#2D9`

![](https://bbs.kanxue.com/upload/attach/201206/3120_rsdxhkc6qvcyz6s.jpg)

先加壳 加壳选项如图1－2：选上`AES`加密 基本反调试选项

![](https://bbs.kanxue.com/upload/attach/201206/3120_93mjd9vj9i6babc.jpg)

----------ok,

我们看看 加壳前的区段如图所示，加壳后程序的区段如图所示。可以看出加壳后的程序增加了一个`.sdata`区段。

![](https://bbs.kanxue.com/upload/attach/201206/3120_tr6cbkwt074c5h6.png)

![](https://bbs.kanxue.com/upload/attach/201206/3120_e77giftqaw0xeex.png)

输入表比较，加壳前后的输入表没有变化，请看下图

![](https://bbs.kanxue.com/upload/attach/201206/3120_jewer8b2t0zojwm.png)

先静态反编译一下，看看它对我们的小程序做了什么手脚。

采用`Reflector`载入`dot_NET_bounce`后，可以发现加壳后的程序的增加了威步相关的类，程序原来的类的名称和字段名称相关信息没有改变，如图所示。

![](https://bbs.kanxue.com/upload/attach/201206/3120_6t2ifwj7dbhc4gk.png)

加壳后程序的方法中采用`switch`加入了流程混淆，只能采用`IL`代码查看，以`bounce_loop`为例该方法的`IL`代码如下。从中可以看出用的是动态调用和委托来执行原来的代码。

``` 
.method public hidebysig virtual instance void

        bounce_loop(object ob,

                    class [mscorlib]System.EventArgs ev) cil managed

{
  // Code size       246 (0xf6)
  .maxstack  12
  .locals init (int32 pinned V_0,   class [mscorlib]System.Type[] pinned V_1)
  IL_0000:  ldc.i4.4
  IL_0001:  br         IL_0019
  IL_0006:  ldnull
  IL_0007: stsfld     class [mscorlib]System.Type[] com.wibu.xpm.AxpNet.QTvJOYTWFEBSHZO::voX0cCgoco
  IL_000c:  ldc.i4.0
  IL_000d:  br         IL_0019
  IL_0012:  ldnull
  IL_0013:  stsfld     class [mscorlib]System.Type[] com.wibu.xpm.AxpNet.QTvJOYTWFEBSHZO::kuFup3Ww5UH
  IL_0018:  ldc.i4.1
  IL_0019:  switch     ( IL_0012,    IL_002f,   IL_0047)
  IL_002a:  br         IL_0006
  IL_002f:  br.s       IL_0038
  IL_0031:  ldc.i4.s   20
  IL_0033:  call       void [mscorlib]System.Threading.Thread::Sleep(int32)
  IL_0038:  volatile.
  IL_003a: ldsfld     bool modreq([mscorlib]System.Runtime.CompilerServices.IsVolatile) com.wibu.xpm.AxpNet.QTvJOYTWFEBSHZO::GOnQQaxbvpt0
  IL_003f:  brfalse.s  IL_0031
  IL_0041:  ldc.i4.2
  IL_0042:  br         IL_0019
  IL_0047:  ldc.i4     0x2
  IL_004c:  ldc.i4     0x244
  IL_0051:  newarr     [mscorlib]System.Byte
  IL_0056:  dup
  IL_0057: ldtoken    field valuetype $ArrayType$580 com.wibu.xpm.AxpNet.QTvJOYTWFEBSHZO::m2
  IL_005c: call       void [mscorlib]System.Runtime.CompilerServices.RuntimeHelpers::InitializeArray(class [mscorlib]System.Array,
                                                                                                          valuetype [mscorlib]System.RuntimeFieldHandle)
  IL_0061:  ldtoken    [mscorlib]System.Void
  IL_0066:  call       class [mscorlib]System.Type [mscorlib]System.Type::GetTypeFromHandle(valuetype [mscorlib]System.RuntimeTypeHandle)
  IL_006b:  ldc.i4.s   3
  IL_006d:  newarr     [mscorlib]System.Type
  IL_0072:  stloc.1
  IL_0073:  ldloc.1
  IL_0074:  ldc.i4.0
  IL_0075:  ldtoken    Aladdin.BouncingBall.Sample
  IL_007a:  call       class [mscorlib]System.Type [mscorlib]System.Type::GetTypeFromHandle(valuetype [mscorlib]System.RuntimeTypeHandle)
  IL_007f:  stelem.ref
  IL_0080:  ldloc.1
  IL_0081:  ldc.i4.s   1
  IL_0083:  ldtoken    [mscorlib]System.Object
  IL_0088:  call       class [mscorlib]System.Type [mscorlib]System.Type::GetTypeFromHandle(valuetype [mscorlib]System.RuntimeTypeHandle)
  IL_008d:  stelem.ref
  IL_008e:  ldloc.1
  IL_008f:  ldc.i4.s   2
  IL_0091:  ldtoken    [mscorlib]System.EventArgs
  IL_0096:  call       class [mscorlib]System.Type [mscorlib]System.Type::GetTypeFromHandle(valuetype [mscorlib]System.RuntimeTypeHandle)
  IL_009b:  stelem.ref
  IL_009c:  ldloc.1
  IL_009d:  ldtoken    Aladdin.BouncingBall.Sample
  IL_00a2:  call       class [mscorlib]System.Type [mscorlib]System.Type::GetTypeFromHandle(valuetype [mscorlib]System.RuntimeTypeHandle)
  IL_00a7:  ldnull
  IL_00a8:  ldsfld     class [mscorlib]System.Reflection.MethodBase[] com.wibu.xpm.AxpNet.QTvJOYTWFEBSHZO::BaseMethods
  IL_00ad:  ldc.i4     0x2
  IL_00b2:  ldelem.ref
  IL_00b3:  ldnull
  IL_00b4:  ceq
  IL_00b6:  brfalse.s  IL_00be
  IL_00b8:  pop
  IL_00b9:  call       class [mscorlib]System.Reflection.MethodBase [mscorlib]System.Reflection.MethodBase::GetCurrentMethod()
  IL_00be:  call       class [mscorlib]System.Reflection.Emit.DynamicMethod com.wibu.xpm.AxpNet.QTvJOYTWFEBSHZO::GetMethod(int32,
                                                                                                                               uint8[],
                                                                                                                               class [mscorlib]System.Type,
                                                                                                                               class [mscorlib]System.Type[],
                                                                                                                               class [mscorlib]System.Type,
                                                                                                                               class [mscorlib]System.Reflection.MethodBase)
  IL_00c3:  ldtoken    class com.wibu.xpm.AxpNet.QTvJOYTWFEBSHZO/da00<object,class [mscorlib]System.EventArgs>
  IL_00c8:  call       class [mscorlib]System.Type [mscorlib]System.Type::GetTypeFromHandle(valuetype [mscorlib]System.RuntimeTypeHandle)
  IL_00cd:  ldarg.0
  IL_00ce:  callvirt   instance class [mscorlib]System.Delegate [mscorlib]System.Reflection.Emit.DynamicMethod::CreateDelegate(class [mscorlib]System.Type,
                                                                                                                                   object)
  IL_00d3:  castclass  class com.wibu.xpm.AxpNet.QTvJOYTWFEBSHZO/da00<object,class [mscorlib]System.EventArgs>
  IL_00d8:  ldarg.1
  IL_00d9:  ldarg.2
  IL_00da:  ldsfld     int32 com.wibu.xpm.AxpNet.QTvJOYTWFEBSHZO::okVUZmPoInkRot
  IL_00df:  stloc.0
  IL_00e0:  ldc.i4     0x2
  IL_00e5:  stsfld     int32 com.wibu.xpm.AxpNet.QTvJOYTWFEBSHZO::okVUZmPoInkRot
  IL_00ea:  callvirt   instance void class com.wibu.xpm.AxpNet.QTvJOYTWFEBSHZO/da00<object,class [mscorlib]System.EventArgs>::Invoke(!0,!1)
  IL_00ef:  ldloc.0
  IL_00f0:  stsfld     int32 com.wibu.xpm.AxpNet.QTvJOYTWFEBSHZO::okVUZmPoInkRot
  IL_00f5:  ret
} // end of method Sample::bounce_loop
```

解密加密数据方法的`C#`代码如下所示，从中可以看出使用`AES`算法进行解密。

``` 
internal static byte[] Iw5cqEEEffEbCUV(byte[] Encrypted, byte[] Key)

{
    try
    {
        RijndaelManaged managed = new RijndaelManaged();
        managed .Mode = CipherMode.CBC;
        managed .IV = IV;
       managed .Key =Key ;
        MemoryStream stream = new MemoryStream();
        CryptoStream stream2 = new CryptoStream(, .CreateDecryptor(), CryptoStreamMode.Write);
        stream2.Write(, 0, .Length);
        stream2.FlushFinalBlock();
        return stream2.ToArray();
    }
    catch
    {
        return new byte[1];
    }
}
```

用`OllyICE`寻找动态生成的方法

运行动态方法，需要对动态生成的方法进行编译，因此可以在`CompileMethod`方法上下断点。

找到`CompileMethod`函数的地址

运行加壳后的程序，用`OllyICE`进行附加，查看`mscorjit.dll`模块的输出函数`getJit`函数，如所示。

地址`790A60A0`指向虚函数表，该虚函数表的第一个指针即为`CompileMethod`函数的地址，本例该函数的地址为`7906114F`，如图所示。
