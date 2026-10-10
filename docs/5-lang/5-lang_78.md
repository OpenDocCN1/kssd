

# 破解 DotNetBar 7.2.0.6 for Windows Forms 控件

> 原文：[`www.pediy.com/kssd/pediy10/61111.html`](https://www.pediy.com/kssd/pediy10/61111.html)

【文章标题】: 破解 DotNetBar 7.2.0.6 for Windows Forms 控件  
【文章作者】: YuSheng  
【作者邮箱】:   
【软件名称】: DotNetBar 7.2.0.6 For Windows Forms  
【软件大小】: 2.859M  
【下载地址】: http://www.devcomponents.com/  
【加壳方式】: 无  
【保护方式】: 混淆  
【编写语言】: C#  
【使用工具】: Reflector XenocodeFox2007 UltraEdit32  
【操作平台】: Dot Net  
【软件介绍】: .net 平台上的界面控件包。  
【作者声明】: 只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教!  
\--------------------------------------------------------------------------------  
【详细过程】  
  破解 DotNetBar 7.2.0.6 for Windows Forms 控件  
    
      以前总来看雪学院，总是潜水学习，写过一点破解记录，也总是在低水平徘徊，不敢拿来献丑。后  
  
来学做别的，离开这里很久了，最近在编程时看到 DotNetBar 控件，就拿来练手。水平有限，高手笑略。  
    
      DotNetBar 是 DotNet 平台上的界面控件包，极大地丰富了 VS2003、2005 的程序界面，包含最新的  
  
Office 2007 的样式的控件组，Vista 样式的 BubbleBar。最近出了 v7.2.0.6 全功能试用版,试破之。  
    
      试用限制：试用时间：33 天；某些组件启动时会有试用提示框，几秒钟后激活“OK”按钮，点击后  
  
方可继续；某些控件标题栏会有“Trail”字样。  
    
      破解目标：解除各试用限制。  
    
      用 Reflector 载入，（这真是个好工具啊），首先找它提示试用的字符“Trail”，找到了三个地方  
  
，  
    
  DevComponents.DotNetBar.Bar.OnPaint(PaintEventArgs) : Void  
  DevComponents.DotNetBar.ItemControl.PaintControl(ItemPaintArgs) : Void  
  DevComponents.DotNetBar.RibbonStrip.PaintControlBackground(ItemPaintArgs) : Void  
    
  进入 DevComponents.DotNetBar.RibbonStrip.PaintControlBackground(ItemPaintArgs) : Void  
  看到  
      if (1Z.0RB())  
      {  
          pa.Graphics.Clear(Color.White);  
          59.1HQ(pa.Graphics, "Trial Version Expired", this.Font, Color.FromArgb(0x80,   
  
Color.Black), base.ClientRectangle, eTextFormat.VerticalCenter |   
  
eTextFormat.HorizontalCenter);  
      }  
      else  
      {  
          59.1HQ(pa.Graphics, "Trial Version", this.Font, Color.FromArgb(0x80, Color.Black),   
  
base.ClientRectangle, eTextFormat.Right);  
      }  
  即在标题上绘制"Trial Version Expired"和"Trial Version"字符串的地方。其他两处也引用了同样的  
  
判断过程。  
    
  先切进 PaintControlBackground 方法，在 Xenocode Fox 2007 中看到地址 Method Body Address:   
  
0x0011B754  
  转为 IL：  
      L\_011a: call bool DevComponents.DotNetBar.1Z::0RB()  
      L\_011f: brfalse.s L\_015e  
      L\_0121: ldarg.1   
      L\_0122: ldfld class \[System.Drawing\]System.Drawing.Graphics   
  
DevComponents.DotNetBar.ItemPaintArgs::Graphics  
      L\_0127: call valuetype \[System.Drawing\]System.Drawing.Color \[System.Drawing\]  
  
System.Drawing.Color::get\_White()  
      L\_012c: callvirt instance void \[System.Drawing\]System.Drawing.Graphics::Clear  
  
(valuetype \[System.Drawing\]System.Drawing.Color)  
      L\_0131: ldarg.1   
      L\_0132: ldfld class \[System.Drawing\]System.Drawing.Graphics   
  
DevComponents.DotNetBar.ItemPaintArgs::Graphics  
      L\_0137: ldstr "Trial Version Expired"  
      L\_013c: ldarg.0   
      L\_013d: callvirt instance class \[System.Drawing\]System.Drawing.Font   
  
\[System.Windows.Forms\]System.Windows.Forms.Control::get\_Font()  
      L\_0142: ldc.i4 0x80  
      L\_0147: call valuetype \[System.Drawing\]System.Drawing.Color \[System.Drawing\]  
  
System.Drawing.Color::get\_Black()  
      L\_014c: call valuetype \[System.Drawing\]System.Drawing.Color \[System.Drawing\]  
  
System.Drawing.Color::FromArgb(int32, valuetype \[System.Drawing\]System.Drawing.Color)  
      L\_0151: ldarg.0   
      L\_0152: call instance valuetype \[System.Drawing\]System.Drawing.Rectangle   
  
\[System.Windows.Forms\]System.Windows.Forms.Control::get\_ClientRectangle()  
      L\_0157: ldc.i4.5   
      L\_0158: call void DevComponents.DotNetBar.59::1HQ(class \[System.Drawing\]  
  
System.Drawing.Graphics, string, class \[System.Drawing\]System.Drawing.Font, valuetype   
  
\[System.Drawing\]System.Drawing.Color, valuetype \[System.Drawing\]System.Drawing.Rectangle,   
  
valuetype DevComponents.DotNetBar.eTextFormat)  
      L\_015d: ret   
      L\_015e: ldarg.1   
      L\_015f: ldfld class \[System.Drawing\]System.Drawing.Graphics   
  
DevComponents.DotNetBar.ItemPaintArgs::Graphics  
      L\_0164: ldstr "Trial Version"  
      L\_0169: ldarg.0   
      L\_016a: callvirt instance class \[System.Drawing\]System.Drawing.Font   
  
\[System.Windows.Forms\]System.Windows.Forms.Control::get\_Font()  
      L\_016f: ldc.i4 0x80  
      L\_0174: call valuetype \[System.Drawing\]System.Drawing.Color \[System.Drawing\]  
  
System.Drawing.Color::get\_Black()  
      L\_0179: call valuetype \[System.Drawing\]System.Drawing.Color \[System.Drawing\]  
  
System.Drawing.Color::FromArgb(int32, valuetype \[System.Drawing\]System.Drawing.Color)  
      L\_017e: ldarg.0   
      L\_017f: call instance valuetype \[System.Drawing\]System.Drawing.Rectangle   
  
\[System.Windows.Forms\]System.Windows.Forms.Control::get\_ClientRectangle()  
      L\_0184: ldc.i4.2   
      L\_0185: call void DevComponents.DotNetBar.59::1HQ(class \[System.Drawing\]  
  
System.Drawing.Graphics, string, class \[System.Drawing\]System.Drawing.Font, valuetype   
  
\[System.Drawing\]System.Drawing.Color, valuetype \[System.Drawing\]System.Drawing.Rectangle,   
  
valuetype DevComponents.DotNetBar.eTextFormat)  
      L\_018a: ret   
    
  用 UltraEdit 打开，定位到 11B8BE，此处和计算所得并不相同，向后推移了 12 个字节，不知是不是程序  
  
增加了一些偏移。好在用 Reflector 可以看到代码的十六进制形式，向后推，很快就可以到对应的位置。  
  将 15E 到 18A 前一个字节共 44 字节的内容换为 00。试用版的提示字符就不再存在了。  
    
  跟进 1Z.0RB（），是如下代码：  
    
  internal static bool 0RB()  
  {  
      Color control = SystemColors.Control;  
      control.ToArgb();  
      control.ToArgb();  
      if (!42G.IsEmpty)  
      {  
          return !(42G == Color.Black);  
      }  
      try  
      {  
          RegistryKey classesRoot = Registry.ClassesRoot;  
          try  
          {  
              classesRoot = classesRoot.CreateSubKey(@"CLSID\\{BC469E31-CD88-442c-8BB2-  
  
A1EBD6B44AB1}\\InprocServer32");  
          }  
          catch (UnauthorizedAccessException)  
          {  
              classesRoot = classesRoot.OpenSubKey(@"CLSID\\{BC469E31-CD88-442c-8BB2-  
  
A1EBD6B44AB1}\\InprocServer32");  
          }  
          try  
          {  
              if ((classesRoot.GetValue("") == null) || (classesRoot.GetValue("").ToString()   
  
\== ""))  
              {  
                  classesRoot.SetValue("", DateTime.Today.ToOADate().ToString());  
              }  
              else  
              {  
                  if (classesRoot.GetValue("").ToString() == "windows3.dll")  
                  {  
                      42G = Color.White;  
                      classesRoot.Close();  
                      classesRoot = null;  
                      return true;  
                  }  
                  DateTime time = DateTime.FromOADate(double.Parse(classesRoot.GetValue  
  
("").ToString()));  
                  if (DateTime.Today.Subtract(time).TotalDays > 21.0)           // 判断是否  
  
过期  
                  {  
                      42G = Color.White;  
                      classesRoot.SetValue("", "windows4.dll");  
                      classesRoot.Close();  
                      classesRoot = null;  
                      return true;  
                  }  
                  if (DateTime.Today.Subtract(time).TotalDays < 0.0)            // 判断是否  
  
过期  
                  {  
                      42G = Color.White;  
                      classesRoot.SetValue("", "windows3.dll");  
                      classesRoot.Close();  
                      classesRoot = null;  
                      return true;  
                  }  
              }  
          }  
          finally  
          {  
              if (classesRoot != null)  
              {  
                  classesRoot.Close();  
              }  
              42H = true;  
          }  
      }  
      catch  
      {  
      }  
      42G = Color.Black;  
      return false;  
  }  
    
   返回 false 可以运行，42G 的颜色为 Color.Black，返回 true 就是前后过期了，且颜色为 Color.White，  
  
有三个这样的地方。  
  切换到 IL，如下：  
  .method assembly hidebysig static bool 0RB() cil managed  
  {  
      .maxstack 3  
      .locals init (  
          \[0\] valuetype \[System.Drawing\]System.Drawing.Color color,  
          \[1\] valuetype \[System.Drawing\]System.Drawing.Color color2,  
          \[2\] class \[mscorlib\]Microsoft.Win32.RegistryKey key,  
          \[3\] valuetype \[mscorlib\]System.DateTime time,  
          \[4\] bool flag,  
          \[5\] valuetype \[mscorlib\]System.DateTime time2,  
          \[6\] float64 num,  
          \[7\] valuetype \[mscorlib\]System.DateTime time3,  
          \[8\] valuetype \[mscorlib\]System.TimeSpan span,  
          \[9\] valuetype \[mscorlib\]System.DateTime time4,  
          \[10\] valuetype \[mscorlib\]System.TimeSpan span2)  
      L\_0000: call valuetype \[System.Drawing\]System.Drawing.Color \[System.Drawing\]  
  
System.Drawing.SystemColors::get\_Control()  
      L\_0005: stloc.0   
      L\_0006: ldloc.0   
      L\_0007: stloc.1   
      L\_0008: ldloca.s color2  
      L\_000a: call instance int32 \[System.Drawing\]System.Drawing.Color::ToArgb()  
      L\_000f: ldloca.s color  
      L\_0011: call instance int32 \[System.Drawing\]System.Drawing.Color::ToArgb()  
      L\_0016: pop   
      L\_0017: pop   
      L\_0018: ldsflda valuetype \[System.Drawing\]System.Drawing.Color   
  
DevComponents.DotNetBar.1Z::42G  
      L\_001d: call instance bool \[System.Drawing\]System.Drawing.Color::get\_IsEmpty()  
      L\_0022: brtrue.s L\_0039  
      L\_0024: ldsfld valuetype \[System.Drawing\]System.Drawing.Color   
  
DevComponents.DotNetBar.1Z::42G  
      L\_0029: call valuetype \[System.Drawing\]System.Drawing.Color \[System.Drawing\]  
  
System.Drawing.Color::get\_Black()  
      L\_002e: call bool \[System.Drawing\]System.Drawing.Color::op\_Equality(valuetype   
  
\[System.Drawing\]System.Drawing.Color, valuetype \[System.Drawing\]System.Drawing.Color)  
      L\_0033: brtrue.s L\_0037  
      L\_0035: ldc.i4.1   
      L\_0036: ret   
      L\_0037: ldc.i4.0   
      L\_0038: ret   
      L\_0039: ldsfld class \[mscorlib\]Microsoft.Win32.RegistryKey \[mscorlib\]  
  
Microsoft.Win32.Registry::ClassesRoot  
      L\_003e: stloc.2   
      L\_003f: ldloc.2   
      L\_0040: ldstr "CLSID\\\\{BC469E31-CD88-442c-8BB2-A1EBD6B44AB1}\\\\InprocServer32"  
      L\_0045: callvirt instance class \[mscorlib\]Microsoft.Win32.RegistryKey \[mscorlib\]  
  
Microsoft.Win32.RegistryKey::CreateSubKey(string)  
      L\_004a: stloc.2   
      L\_004b: leave.s L\_005c  
      L\_004d: pop   
      L\_004e: ldloc.2   
      L\_004f: ldstr "CLSID\\\\{BC469E31-CD88-442c-8BB2-A1EBD6B44AB1}\\\\InprocServer32"  
      L\_0054: callvirt instance class \[mscorlib\]Microsoft.Win32.RegistryKey \[mscorlib\]  
  
Microsoft.Win32.RegistryKey::OpenSubKey(string)  
      L\_0059: stloc.2   
      L\_005a: leave.s L\_005c  
      L\_005c: ldloc.2   
      L\_005d: ldstr ""  
      L\_0062: callvirt instance object \[mscorlib\]Microsoft.Win32.RegistryKey::GetValue  
  
(string)  
      L\_0067: brfalse.s L\_0085  
      L\_0069: ldloc.2   
      L\_006a: ldstr ""  
      L\_006f: callvirt instance object \[mscorlib\]Microsoft.Win32.RegistryKey::GetValue  
  
(string)  
      L\_0074: callvirt instance string \[mscorlib\]System.Object::ToString()  
      L\_0079: ldstr ""  
      L\_007e: call bool \[mscorlib\]System.String::op\_Equality(string, string)  
      L\_0083: brfalse.s L\_00ac  
      L\_0085: ldloc.2   
      L\_0086: ldstr ""  
      L\_008b: call valuetype \[mscorlib\]System.DateTime \[mscorlib\]System.DateTime::get\_Today  
  
()  
      L\_0090: stloc.s time2  
      L\_0092: ldloca.s time2  
      L\_0094: call instance float64 \[mscorlib\]System.DateTime::ToOADate()  
      L\_0099: stloc.s num  
      L\_009b: ldloca.s num  
      L\_009d: call instance string \[mscorlib\]System.Double::ToString()  
      L\_00a2: callvirt instance void \[mscorlib\]Microsoft.Win32.RegistryKey::SetValue(string,   
  
object)  
      L\_00a7: br L\_0191  
      L\_00ac: ldloc.2   
      L\_00ad: ldstr ""  
      L\_00b2: callvirt instance object \[mscorlib\]Microsoft.Win32.RegistryKey::GetValue  
  
(string)  
      L\_00b7: callvirt instance string \[mscorlib\]System.Object::ToString()  
      L\_00bc: ldstr "windows3.dll"  
      L\_00c1: call bool \[mscorlib\]System.String::op\_Equality(string, string)  
      L\_00c6: brfalse.s L\_00e2  
      L\_00c8: call valuetype \[System.Drawing\]System.Drawing.Color \[System.Drawing\]  
  
System.Drawing.Color::get\_White()      // 0xDB16C  
      L\_00cd: stsfld valuetype \[System.Drawing\]System.Drawing.Color   
  
DevComponents.DotNetBar.1Z::42G   // 设 42G 为 White 色   0xDB171  
      L\_00d2: ldloc.2   
      L\_00d3: callvirt instance void \[mscorlib\]Microsoft.Win32.RegistryKey::Close()  
      L\_00d8: ldnull   
      L\_00d9: stloc.2   
      L\_00da: ldc.i4.1       // 返回 true  0xDB17E  
      L\_00db: stloc.s flag  
      L\_00dd: leave L\_01b4  
      L\_00e2: ldloc.2   
      L\_00e3: ldstr ""  
      L\_00e8: callvirt instance object \[mscorlib\]Microsoft.Win32.RegistryKey::GetValue  
  
(string)  
      L\_00ed: callvirt instance string \[mscorlib\]System.Object::ToString()  
      L\_00f2: call float64 \[mscorlib\]System.Double::Parse(string)  
      L\_00f7: call valuetype \[mscorlib\]System.DateTime \[mscorlib\]  
  
System.DateTime::FromOADate(float64)  
      L\_00fc: stloc.3   
      L\_00fd: call valuetype \[mscorlib\]System.DateTime \[mscorlib\]System.DateTime::get\_Today  
  
()  
      L\_0102: stloc.s time3  
      L\_0104: ldloca.s time3  
      L\_0106: ldloc.3   
      L\_0107: call instance valuetype \[mscorlib\]System.TimeSpan \[mscorlib\]  
  
System.DateTime::Subtract(valuetype \[mscorlib\]System.DateTime)  
      L\_010c: stloc.s span  
      L\_010e: ldloca.s span  
      L\_0110: call instance float64 \[mscorlib\]System.TimeSpan::get\_TotalDays()  
      L\_0115: ldc.r8 21  
      L\_011e: ble.un.s L\_0147  
      L\_0120: call valuetype \[System.Drawing\]System.Drawing.Color \[System.Drawing\]  
  
System.Drawing.Color::get\_White()       //0xDB1C4  
      L\_0125: stsfld valuetype \[System.Drawing\]System.Drawing.Color   
  
DevComponents.DotNetBar.1Z::42G    // 设 42G 为 White 色  0xDB1C9  
      L\_012a: ldloc.2   
      L\_012b: ldstr ""  
      L\_0130: ldstr "windows4.dll"  
      L\_0135: callvirt instance void \[mscorlib\]Microsoft.Win32.RegistryKey::SetValue(string,   
  
object)  
      L\_013a: ldloc.2   
      L\_013b: callvirt instance void \[mscorlib\]Microsoft.Win32.RegistryKey::Close()  
      L\_0140: ldnull   
      L\_0141: stloc.2   
      L\_0142: ldc.i4.1      //返回 true  0xDB1E6  
      L\_0143: stloc.s flag  
      L\_0145: leave.s L\_01b4  
      L\_0147: call valuetype \[mscorlib\]System.DateTime \[mscorlib\]System.DateTime::get\_Today  
  
()  
      L\_014c: stloc.s time4  
      L\_014e: ldloca.s time4  
      L\_0150: ldloc.3   
      L\_0151: call instance valuetype \[mscorlib\]System.TimeSpan \[mscorlib\]  
  
System.DateTime::Subtract(valuetype \[mscorlib\]System.DateTime)  
      L\_0156: stloc.s span2  
      L\_0158: ldloca.s span2  
      L\_015a: call instance float64 \[mscorlib\]System.TimeSpan::get\_TotalDays()  
      L\_015f: ldc.r8 0  
      L\_0168: bge.un.s L\_0191  
      L\_016a: call valuetype \[System.Drawing\]System.Drawing.Color \[System.Drawing\]  
  
System.Drawing.Color::get\_White()        // 0xDB20E  
      L\_016f: stsfld valuetype \[System.Drawing\]System.Drawing.Color   
  
DevComponents.DotNetBar.1Z::42G      // 设 42G 为 White 色  0xDB213  
      L\_0174: ldloc.2   
      L\_0175: ldstr ""  
      L\_017a: ldstr "windows3.dll"  
      L\_017f: callvirt instance void \[mscorlib\]Microsoft.Win32.RegistryKey::SetValue(string,   
  
object)  
      L\_0184: ldloc.2   
      L\_0185: callvirt instance void \[mscorlib\]Microsoft.Win32.RegistryKey::Close()  
      L\_018a: ldnull   
      L\_018b: stloc.2   
      L\_018c: ldc.i4.1     // 返回 true  0xDB230  
      L\_018d: stloc.s flag  
      L\_018f: leave.s L\_01b4  
      L\_0191: leave.s L\_01a3  
      L\_0193: ldloc.2   
      L\_0194: brfalse.s L\_019c  
      L\_0196: ldloc.2   
      L\_0197: callvirt instance void \[mscorlib\]Microsoft.Win32.RegistryKey::Close()  
      L\_019c: ldc.i4.1   
      L\_019d: stsfld bool DevComponents.DotNetBar.1Z::42H  
      L\_01a2: endfinally   
      L\_01a3: leave.s L\_01a8  
      L\_01a5: pop   
      L\_01a6: leave.s L\_01a8  
      L\_01a8: call valuetype \[System.Drawing\]System.Drawing.Color \[System.Drawing\]  
  
System.Drawing.Color::get\_Black()    // 0xDB24C  
      L\_01ad: stsfld valuetype \[System.Drawing\]System.Drawing.Color   
  
DevComponents.DotNetBar.1Z::42G    // 设 42G 为 Black 色   0xDB251  
      L\_01b2: ldc.i4.0     // 返回 false；0xDB256  
      L\_01b3: ret   
      L\_01b4: ldloc.s flag  
      L\_01b6: ret   
      .try L\_003f to L\_004d catch \[mscorlib\]System.UnauthorizedAccessException handler   
  
L\_004d to L\_005c  
      .try L\_005c to L\_0193 finally handler L\_0193 to L\_01a3  
      .try L\_0039 to L\_01a5 catch object handler L\_01a5 to L\_01a8  
  }  
    
  在 Xenocode Fox 2007 中看到此段代码地址为：// Method Body Address: 0x000DB0A4  
    
  用 UltraEdit 打开，依次定位到各地址，再向后找 12 字节，修改前三次取色的 call 字节“28 99 00 00   
  
0A”为“28 F1 02 00 0A”，三个返回“true”的字节“17”为“16”。保存。  
  替换原文件，更改系统日期，发现时间限制已经去除。  
    
  根据启动对话框中的文字，查到启动对话框是“2T”，引用的地方如下：  
  DevComponents.DotNetBar.2T  
    Depends On  
    Used By  
      DevComponents.DotNetBar.DevCoLicenseProvider.GetLicense(LicenseContext,   
  
Type, Object, Boolean) : License  
      DevComponents.DotNetBar.DotNetBarManager..ctor(IContainer)  
      DevComponents.DotNetBar.DotNetBarManager.FU(PopupItem, EventArgs) : Void  
      DevComponents.DotNetBar.ExplorerBar..ctor()  
      DevComponents.DotNetBar.RibbonControl.OnHandleCreated(EventArgs) : Void  
      DevComponents.DotNetBar.SideBar..ctor()  
      DevComponents.DotNetBar.Wizard.OnHandleCreated(EventArgs) : Void  
    Exposed By  
    Instantiated By  
    
    
    
  展开 DevComponents.DotNetBar.DevCoLicenseProvider.GetLicense(LicenseContext, Type, Object,   
  
Boolean) : License  
  此处在 Xenocode 中看不到，实际上还有许多的地方均显示为“Invalid string heap index”，估计是  
  
程序采取了一些混淆处理，好在一个一个展开，也用不了太长时间，就可以看到“GetLicense”这句了。  
  此处代码如下：  
  public override License GetLicense(LicenseContext context, Type type, object instance,   
  
bool allowExceptions)  
  {  
      0Z z = null;  
      if (context != null)  
      {  
          z = new 0Z("");  
          if (context.UsageMode == LicenseUsageMode.Runtime)  
          {  
              new 2T().ShowDialog();   //此句置 0  
          }  
          else  
          {  
              context.SetSavedLicenseKey(type, z.LicenseKey);  
          }  
      }  
      if ((z == null) && (context != null))  
      {  
          new 2T().ShowDialog();      //此句置 0  
          z = new 0Z("");  
      }  
      return z;  
  }  
    
  IL 格式如下：  
  .method public hidebysig virtual instance \[System\]System.ComponentModel.License   
  
GetLicense(\[System\]System.ComponentModel.LicenseContext context, \[mscorlib\]System.Type   
  
Invalid string heap index, object Invalid string heap index, bool allowExceptions) cil   
  
managed  
  {  
       // Method Body Address: 0x00086620  
       // Code Size: 84 byte(s)  
       .maxstack 3  
       .locals init (  
            DevComponents.DotNetBar.0Z z1,  
            DevComponents.DotNetBar.2T t1,  
            DevComponents.DotNetBar.2T t2)  
       L\_0000: ldnull   
       L\_0001: stloc.0   
       L\_0002: ldarg.1   
       L\_0003: brfalse.s L\_0034  
       L\_0005: ldstr ""  
       L\_000a: newobj instance void DevComponents.DotNetBar.0Z::.ctor(string)  
       L\_000f: stloc.0   
       L\_0010: ldarg.1   
       L\_0011: callvirt instance \[System\]System.ComponentModel.LicenseUsageMode \[System\]  
  
System.ComponentModel.LicenseContext::get\_UsageMode()  
       L\_0016: brtrue.s L\_0027  
       L\_0018: newobj instance void DevComponents.DotNetBar.2T::.ctor()          //  此处置 0  
  
共 13 字节  
       L\_001d: stloc.1   
       L\_001e: ldloc.1   
       L\_001f: callvirt instance \[System.Windows.Forms\]System.Windows.Forms.DialogResult   
  
\[System.Windows.Forms\]System.Windows.Forms.Form::ShowDialog()  
       L\_0024: pop                                                               // 至此  
       L\_0025: br.s L\_0034  
       L\_0027: ldarg.1   
       L\_0028: ldarg.2   
       L\_0029: ldloc.0   
       L\_002a: callvirt instance string \[System\]  
  
System.ComponentModel.License::get\_LicenseKey()  
       L\_002f: callvirt instance void \[System\]  
  
System.ComponentModel.LicenseContext::SetSavedLicenseKey(\[mscorlib\]System.Type, string)  
       L\_0034: ldloc.0   
       L\_0035: brtrue.s L\_0052  
       L\_0037: ldarg.1   
       L\_0038: brfalse.s L\_0052  
       L\_003a: newobj instance void DevComponents.DotNetBar.2T::.ctor()            //此处置 0  
  
共 13 字节  
       L\_003f: stloc.2   
       L\_0040: ldloc.2   
       L\_0041: callvirt instance \[System.Windows.Forms\]System.Windows.Forms.DialogResult   
  
\[System.Windows.Forms\]System.Windows.Forms.Form::ShowDialog()  
       L\_0046: pop                                                                // 至此  
       L\_0047: ldstr ""  
       L\_004c: newobj instance void DevComponents.DotNetBar.0Z::.ctor(string)  
       L\_0051: stloc.0   
       L\_0052: ldloc.0   
       L\_0053: ret   
  }  
     
  展开 DevComponents.DotNetBar.DotNetBarManager..ctor(IContainer)  
  // Method Body Address: 0x000966D4  
  替换    2T t = new 2T();  
      t.ShowDialog();  
      t.Dispose();  
  IL 格式如下：  
      L\_0099: newobj instance void DevComponents.DotNetBar.2T::.ctor()                 //此  
  
处开始置 0 共 19 字节  
      L\_009e: stloc.0   
      L\_009f: ldloc.0   
      L\_00a0: callvirt instance valuetype \[System.Windows.Forms\]  
  
System.Windows.Forms.DialogResult \[System.Windows.Forms\]  
  
System.Windows.Forms.Form::ShowDialog()  
      L\_00a5: pop   
      L\_00a6: ldloc.0   
      L\_00a7: callvirt instance void \[System\]System.ComponentModel.Component::Dispose()  
  去掉 DotNetBarManager 控件的启动框。  
     
  展开 DevComponents.DotNetBar.DotNetBarManager.FU(PopupItem, EventArgs) : Void  
  // Method Body Address: 0x0009DFE4  
  IL 格式如下：  
      L\_0033: newobj instance void DevComponents.DotNetBar.2T::.ctor()             // 此处开  
  
始置 0 共 13 字节  
      L\_0038: stloc.0   
      L\_0039: ldloc.0   
      L\_003a: callvirt instance valuetype \[System.Windows.Forms\]  
  
System.Windows.Forms.DialogResult \[System.Windows.Forms\]  
  
System.Windows.Forms.Form::ShowDialog()  
      L\_003f: pop   
    
  展开 DevComponents.DotNetBar.RibbonControl.OnHandleCreated(EventArgs) : Void  
  // Method Body Address: 0x00116988  
  替换  
  2T t = new 2T();  
      t.ShowDialog();  
      t.Dispose();  
  IL 格式为：  
      L\_0000: newobj instance void DevComponents.DotNetBar.2T::.ctor()             // 此处开  
  
始置 0 共 19 字节  
      L\_0005: stloc.0   
      L\_0006: ldloc.0   
      L\_0007: callvirt instance valuetype \[System.Windows.Forms\]  
  
System.Windows.Forms.DialogResult \[System.Windows.Forms\]  
  
System.Windows.Forms.Form::ShowDialog()  
      L\_000c: pop   
      L\_000d: ldloc.0   
      L\_000e: callvirt instance void \[System\]System.ComponentModel.Component::Dispose()  
  去掉 Ribbon 控件的启动框。  
    
  展开 DevComponents.DotNetBar.ExplorerBar..ctor()  
  // Method Body Address: 0x000BCC3C  
  IL 格式如下：  
      L\_0116: newobj instance void DevComponents.DotNetBar.2T::.ctor()            // 此处开  
  
始置 0 共 13 字节  
      L\_011b: stloc.0   
      L\_011c: ldloc.0   
      L\_011d: callvirt instance valuetype \[System.Windows.Forms\]  
  
System.Windows.Forms.DialogResult \[System.Windows.Forms\]  
  
System.Windows.Forms.Form::ShowDialog()  
      L\_0122: pop   
    
  展开 DevComponents.DotNetBar.SideBar..ctor()  
   // Method Body Address: 0x0012A0A4  
  IL 格式如下：  
       L\_00e9: newobj instance void DevComponents.DotNetBar.2T::.ctor()            //此处开  
  
始置 0 共 13 字节  
       L\_00ee: stloc.0   
       L\_00ef: ldloc.0   
       L\_00f0: callvirt instance \[System.Windows.Forms\]System.Windows.Forms.DialogResult   
  
\[System.Windows.Forms\]System.Windows.Forms.Form::ShowDialog()  
       L\_00f5: pop   
  去掉 SideBar 控件的启动框。  
    
  展开 DevComponents.DotNetBar.Wizard.OnHandleCreated(EventArgs) : Void  
  // Method Body Address: 0x00159600  
  IL 格式如下：  
      L\_0007: newobj instance void DevComponents.DotNetBar.2T::.ctor()            // 此处开  
  
始置 0 共 19 字节  
      L\_000c: stloc.0   
      L\_000d: ldloc.0   
      L\_000e: callvirt instance valuetype \[System.Windows.Forms\]  
  
System.Windows.Forms.DialogResult \[System.Windows.Forms\]  
  
System.Windows.Forms.Form::ShowDialog()  
      L\_0013: pop   
      L\_0014: ldloc.0   
      L\_0015: callvirt instance void \[System\]System.ComponentModel.Component::Dispose()  
    
  总结：去掉 new 2T().ShowDialog();形式的框要置 0 共 13 字节。  
  去掉 2T t = new 2T();  
      t.ShowDialog();  
      t.Dispose();形式的框要置 0 共 19 字节。  
    
  修改系统日期，并逐条试用自带的 Demo，运行正常。完成。  
    
\--------------------------------------------------------------------------------  
【版权声明】: 本文原创于看雪技术论坛, 转载请注明作者并保持文章的完整, 谢谢!  
  
                                                       2008 年 03 月 12 日 11:51:53