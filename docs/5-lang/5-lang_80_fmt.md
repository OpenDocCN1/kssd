
# Windows Manager 中文版 6.0.2 版本的破解

> 原文：[`www.pediy.com/kssd/pediy10/89438.html`](https://www.pediy.com/kssd/pediy10/89438.html)

![](img/f367f7d8e05d5987a0fa571ef5fb6006.jpg)

## 1. 静态分析
首先用`Reflector`静态分析工具分析，打开到程序的入口点。

![](img/870b735d3288e4d94b09bbce05a14990.jpg)

可以发现，它的注册是调用`MyClsDetermineRegister.DetermineRegistered(....)`这个函数进行注册。

## 2. 初步尝试
如果把`publicVarFun.RemainDays==`“已注册”`改成`publicVarFun.RemainDays!=`“已注册”`发现程序可以用，但是它的辅助工具运行的时候还是提示使用期开始`15`天，如果我把他的辅助工具都这样进行感觉有点麻烦，趁现在还是个初学者那就脚踏实地的进行破解吧。

## 3. 定位注册逻辑
通过第二步我发现他的判断注册信息是放在`pcl.dll`当中，这是用`Reflector`打开`pcl.dll`发现了

![](img/222c579ac124542edcf75d036c2b97ff.jpg)

找到它的注册所调用的类。

## 4. 分析注册函数
打开看到了，那个注册函数

![](img/f367f7d8e05d5987a0fa571ef5fb6006.jpg)

剩下的就是去读函数了。

```
.method public static string DetermineRegistered([opt] string strCPUID) cil managed

{
    .param [1] = string('')
    .maxstack 8
    .locals init (
        [0] string str,
        [1] char[] chArray,
        [2] bool flag,
        [3] int32 num)
    L_0047: nop
    L_0048: ldarg.0
    L_0049: ldstr ""
    L_004e: ldc.i4.0
    L_004f: call int32 [Microsoft.VisualBasic]Microsoft.VisualBasic.CompilerServices.Operators::CompareString(string, string, bool)
    L_0054: ldc.i4.0
    L_0055: ceq
    L_0057: stloc.2
    L_0063: ldloc.2
    L_0064: brfalse.s L_00b5//判断 CPUID 是否为空，如果 CPUID 为空则跳转到 L_00b5，否则接着执行
    L_0071: br.s L_00dd
    L_007e: br.s L_00cc
    L_0080: nop
    L_0081: ldc.i4.s 15
    L_0083: call bool PCL.MyClsDetermineRegister/b::b(uint8)
    L_0088: stloc.2
    L_0097: ldc.i4.1
    L_0098: br.s L_009d
    L_009a: ldc.i4.0
    L_009b: br.s L_009d
    L_009d: brfalse.s L_009f
    L_009f: ldloc.2
    L_00a0: brfalse.s L_00f7
    L_00b0: br L_01ab
    L_00b5: nop
    L_00b6: ldarg.0
    L_00b7: stsfld string c::a
    L_00ca: br.s L_013a
    L_00cc: nop
    L_00db: br.s L_0116
    L_00dd: call string PCL.MyClsDetermineRegister::GetCPUID()
    L_00e2: stsfld string c::a
    L_00f5: br.s L_013a
    L_00f7: nop
    L_00f8: ldsfld uint8 PCL.MyClsDetermineRegister::b
    L_00fd: call string [Microsoft.VisualBasic]Microsoft.VisualBasic.CompilerServices.Conversions::ToString(uint8)
    L_0102: stloc.0
    L_0111: br L_01f0
    L_0116: nop
    L_0125: br L_01f0
    L_0138: br.s L_0116
    L_013a: nop
    L_013b: ldsfld class [mscorlib]Microsoft.Win32.RegistryKey [mscorlib]Microsoft.Win32.Registry::CurrentUser
    L_0140: ldstr "Software\\"
    L_0145: call string PCL.SoftwareInfo::get_CompanyName()
    L_014a: ldstr "\\"
    L_014f: call string PCL.SoftwareInfo::get_ProductName()
    L_0154: ldc.i4.1
    L_0155: newarr char
    L_015a: stloc.1
    L_015b: ldloc.1
    L_015c: ldc.i4.0
    L_015d: ldc.i4.s 0x2d
    L_015f: stelem.i2
    L_0160: ldloc.1
    L_0161: callvirt instance string[] [mscorlib]System.String::Split(char[])
    L_0166: ldc.i4.0
    L_0167: ldelem.ref
    L_0168: callvirt instance string [mscorlib]System.String::ToString()
    L_016d: callvirt instance string [mscorlib]System.String::Trim()
    L_0172: call string [mscorlib]System.String::Concat(string, string, string, string)
    L_0177: callvirt instance class [mscorlib]Microsoft.Win32.RegistryKey [mscorlib]Microsoft.Win32.RegistryKey::OpenSubKey(string)
    L_017c: stsfld class [mscorlib]Microsoft.Win32.RegistryKey PCL.MyClsDetermineRegister::a
    L_0181: call bool PCL.MyClsDetermineRegister::b()
    L_0186: stloc.2
    L_0195: ldloc.2
    L_0196: brfalse L_0080//关键的跳转，就是比较注册成功与否
    L_01a9: br.s L_01d4
    L_01ab: ldstr "\u60a8\u7684\u8bd5\u7528\u671f\u5df2\u8fc7\uff0c\u8bf7\u8fd0\u884c\u4e3b\u7a0b\u5e8f\u8ba2\u8d2d\u6b64\u8f6f\u4ef6\uff01"
    L_01b0: ldstr "\u8b66\u544a"
    L_01b5: ldc.i4.0
    L_01b6: ldc.i4.s 0x30
    L_01b8: call valuetype [System.Windows.Forms]System.Windows.Forms.DialogResult [System.Windows.Forms]System.Windows.Forms.MessageBox::Show(string, string, valuetype [System.Windows.Forms]System.Windows.Forms.MessageBoxButtons, valuetype [System.Windows.Forms]System.Windows.Forms.MessageBoxIcon)
    L_01bd: pop
    L_01be: ldstr "\u5df2\u8fc7\u671f"
    L_01c3: stloc.0
    L_01d2: br.s L_01f0
    L_01d4: call void PCL.MyClsDetermineRegister::a()
    L_01d9: nop
    L_01da: ldstr "\u5df2\u6ce8\u518c"//已注册
    L_01df: stloc.0
    L_01ee: br.s L_01f0
    L_01f0: ldloc.0
    L_01f1: ret
}
```
