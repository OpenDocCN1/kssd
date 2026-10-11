
# 破解 DotNetBar 7.2.0.6 for Windows Forms 控件

> 原文：[`www.pediy.com/kssd/pediy10/61111.html`](https://www.pediy.com/kssd/pediy10/61111.html)

## 基本信息

- 【文章标题】: 破解 DotNetBar 7.2.0.6 for Windows Forms 控件
- 【文章作者】: YuSheng
- 【作者邮箱】: 
- 【软件名称】: DotNetBar 7.2.0.6 For Windows Forms
- 【软件大小】: 2.859M
- 【下载地址】: `http://www.devcomponents.com/`
- 【加壳方式】: 无
- 【保护方式】: 混淆
- 【编写语言】: `C#`
- 【使用工具】: `Reflector`, `XenocodeFox2007`, `UltraEdit32`
- 【操作平台】: `Dot Net`
- 【软件介绍】: `.net` 平台上的界面控件包。
- 【作者声明】: 只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教!

## 详细过程

破解 DotNetBar 7.2.0.6 for Windows Forms 控件

以前总来看雪学院，总是潜水学习，写过一点破解记录，也总是在低水平徘徊，不敢拿来献丑。后来学做别的，离开这里很久了，最近在编程时看到 DotNetBar 控件，就拿来练手。水平有限，高手笑略。

DotNetBar 是 `DotNet` 平台上的界面控件包，极大地丰富了 `VS2003`、`2005` 的程序界面，包含最新的 `Office 2007` 的样式的控件组，`Vista` 样式的 `BubbleBar`。最近出了 `v7.2.0.6` 全功能试用版,试破之。

**试用限制**：试用时间：`33` 天；某些组件启动时会有试用提示框，几秒钟后激活“`OK`”按钮，点击后方可继续；某些控件标题栏会有“`Trail`”字样。

**破解目标**：解除各试用限制。

用 `Reflector` 载入，（这真是个好工具啊），首先找它提示试用的字符“`Trail`”，找到了三个地方：

- `DevComponents.DotNetBar.Bar.OnPaint(PaintEventArgs) : Void`
- `DevComponents.DotNetBar.ItemControl.PaintControl(ItemPaintArgs) : Void`
- `DevComponents.DotNetBar.RibbonStrip.PaintControlBackground(ItemPaintArgs) : Void`

进入 `DevComponents.DotNetBar.RibbonStrip.PaintControlBackground(ItemPaintArgs) : Void` 看到

```
if (1Z.0RB())
{
    pa.Graphics.Clear(Color.White);
    59.1HQ(pa.Graphics, "Trial Version Expired", this.Font, Color.FromArgb(0x80, Color.Black), base.ClientRectangle, eTextFormat.VerticalCenter | eTextFormat.HorizontalCenter);
}
else
{
    59.1HQ(pa.Graphics, "Trial Version", this.Font, Color.FromArgb(0x80, Color.Black), base.ClientRectangle, eTextFormat.Right);
}
```

即在标题上绘制 `"Trial Version Expired"` 和 `"Trial Version"` 字符串的地方。其他两处也引用了同样的判断过程。

先切进 `PaintControlBackground` 方法，在 `Xenocode Fox 2007` 中看到地址 `Method Body Address: 0x0011B754`。

转为 `IL`：

```
L_011a: call bool DevComponents.DotNetBar.1Z::0RB()
L_011f: brfalse.s L_015e
L_0121: ldarg.1 
L_0122: ldfld class [System.Drawing]System.Drawing.Graphics DevComponents.DotNetBar.ItemPaintArgs::Graphics
L_0127: call valuetype [System.Drawing]System.Drawing.Color [System.Drawing]System.Drawing.Color::get_White()
L_012c: callvirt instance void [System.Drawing]System.Drawing.Graphics::Clear(valuetype [System.Drawing]System.Drawing.Color)
L_0131: ldarg.1 
L_0132: ldfld class [System.Drawing]System.Drawing.Graphics DevComponents.DotNetBar.ItemPaintArgs::Graphics
L_0137: ldstr "Trial Version Expired"
L_013c: ldarg.0 
L_013d: callvirt instance class [System.Drawing]System.Drawing.Font [System.Windows.Forms]System.Windows.Forms.Control::get_Font()
L_0142: ldc.i4 0x80
L_0147: call valuetype [System.Drawing]System.Drawing.Color [System.Drawing]System.Drawing.Color::get_Black()
L_014c: call valuetype [System.Drawing]System.Drawing.Color [System.Drawing]System.Drawing.Color::FromArgb(int32, valuetype [System.Drawing]System.Drawing.Color)
L_0151: ldarg.0 
L_0152: call instance valuetype [System.Drawing]System.Drawing.Rectangle [System.Windows.Forms]System.Windows.Forms.Control::get_ClientRectangle()
L_0157: ldc.i4.5 
L_0158: call void DevComponents.DotNetBar.59::1HQ(class [System.Drawing]System.Drawing.Graphics, string, class [System.Drawing]System.Drawing.Font, valuetype [System.Drawing]System.Drawing.Color, valuetype [System.Drawing]System.Drawing.Rectangle, valuetype DevComponents.DotNetBar.eTextFormat)
L_015d: ret 
L_015e: ldarg.1 
L_015f: ldfld class [System.Drawing]System.Drawing.Graphics DevComponents.DotNetBar.ItemPaintArgs::Graphics
L_0164: ldstr "Trial Version"
L_0169: ldarg.0 
L_016a: callvirt instance class [System.Drawing]System.Drawing.Font [System.Windows.Forms]System.Windows.Forms.Control::get_Font()
L_016f: ldc.i4 0x80
L_0174: call valuetype [System.Drawing]System.Drawing.Color [System.Drawing]System.Drawing.Color::get_Black()
L_0179: call valuetype [System.Drawing]System.Drawing.Color [System.Drawing]System.Drawing.Color::FromArgb(int32, valuetype [System.Drawing]System.Drawing.Color)
L_017e: ldarg.0 
L_017f: call instance valuetype [System.Drawing]System.Drawing.Rectangle [System.Windows.Forms]System.Windows.Forms.Control::get_ClientRectangle()
L_0184: ldc.i4.2 
L_0185: call void DevComponents.DotNetBar.59::1HQ(class [System.Drawing]System.Drawing.Graphics, string, class [System.Drawing]System.Drawing.Font, valuetype [System.Drawing]System.Drawing.Color, valuetype [System.Drawing]System.Drawing.Rectangle, valuetype DevComponents.DotNetBar.eTextFormat)
L_018a: ret 
```

用 `UltraEdit` 打开，定位到 `11B8BE`，此处和计算所得并不相同，向后推移了 `12` 个字节，不知是不是程序增加了一些偏移。好在用 `Reflector` 可以看到代码的十六进制形式，向后推，很快就可以到对应的位置。

将 `15E` 到 `18A` 前一个字节共 `44` 字节的内容换为 `00`。试用版的提示字符就不再存在了。

跟进 `1Z.0RB()`，是如下代码：

```csharp
internal static bool 0RB()
{
    Color control = SystemColors.Control;
    control.ToArgb();
    control.ToArgb();
    if (!42G.IsEmpty)
    {
        return !(42G == Color.Black);
    }
    try
    {
        RegistryKey classesRoot = Registry.ClassesRoot;
        try
        {
            classesRoot = classesRoot.CreateSubKey(@"CLSID\{BC469E31-CD88-442c-8BB2-A1EBD6B44AB1}\InprocServer32");
        }
        catch (UnauthorizedAccessException)
        {
            classesRoot = classesRoot.OpenSubKey(@"CLSID\{BC469E31-CD88-442c-8BB2-A1EBD6B44AB1}\InprocServer32");
        }
        try
        {
            if ((classesRoot.GetValue("") == null) || (classesRoot.GetValue("").ToString() == ""))
            {
                classesRoot.SetValue("", DateTime.Today.ToOADate().ToString());
            }
            else
            {
                if (classesRoot.GetValue("").ToString() == "windows3.dll")
                {
                    42G = Color.White;
                    classesRoot.Close();
                    classesRoot = null;
                    return true;
                }
                DateTime time = DateTime.FromOADate(double.Parse(classesRoot.GetValue("").ToString()));
                if (DateTime.Today.Subtract(time).TotalDays > 21.0)          // 判断是否过期
                {
                    42G = Color.White;
                    classesRoot.SetValue("", "windows4.dll");
                    classesRoot.Close();
                    classesRoot = null;
                    return true;
                }
                if (DateTime.Today.Subtract(time).TotalDays < 0.0)            // 判断是否过期
                {
                    42G = Color.White;
                    classesRoot.SetValue("", "windows3.dll");
                    classesRoot.Close();
                    classesRoot = null;
                    return true;
                }
            }
        }
        finally
        {
            if (classesRoot != null)
            {
                classesRoot.Close();
            }
            42H = true;
        }
    }
    catch
    {
    }
    42G = Color.Black;
    return false;
}
```

返回 `false` 可以运行，`42G` 的颜色为 `Color.Black`，返回 `true` 就是前后过期了，且颜色为 `Color.White`，有三个这样的地方。

切换到 `IL`，如下：
