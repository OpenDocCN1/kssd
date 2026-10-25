
# .NET 下爆破 Advanced System Cleaner

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-84.htm`](https://www.pediy.com/kssd/pediy07/pediy7-84.htm)

**名称**：Advanced System Cleaner  
**版本**：1.2.3  
**官方网址**：http://www.winxp-manager.com/asc/index.html  
**编译程序**：VB.NET  
**功能**：注册表、垃圾文件、隐私信息的清理

下载后安装并运行，为 20 天试用版，弹出如下 NAG 窗口，提示“未注册，还有 20 天的试用时间，你花一点点小钱就可以得到完整版”。

用 `Reflector` 打开 `asc.exe`，发现程序犯了兵家大忌：没有用混淆器。在 `AdvancedSystemCleaner.Module2` 下发现 `MyClsRegister` 类，从名称看来是和注册相关的，内容如下：

```csharp
public class MyClsRegister
{
    // Methods
    static MyClsRegister();
    public MyClsRegister();
    public static void DelExprie();
    public static string GetRegCodeValue();
    private static string GetRegisterCode(string strCodeWord, string strValue);
    public static string GetRegName();
    public static void Registered();

    // Fields
    private static RegistryKey ObjRK;
}
```

打开其中的 `Registered()` 方法，就可以找到判断注册信息的关键点了

```csharp
public static void Registered()
{
    string text2 = Module2.MyClsRegister.GetRegName();
    string text1 = Module2.MyClsRegister.GetRegCodeValue();
    if ((StringType.StrCmp(text1, "", false) == 0) &amp; (StringType.StrCmp(text2, "", false) == 0))
    {
        Module1.blnRegistered = false;
    }
    else
    {
        string text3;
        string text4;
        string text5;
        string text6;
        try
        {
            text3 = text1.Substring(0, 5);
            text4 = text1.Substring(6, 5);
            text5 = text1.Substring(12, 5);
            text6 = text1.Substring(0x12, 5);
        }
        catch (Exception exception1)
        {
            ProjectData.SetProjectError(exception1);
            Module1.blnRegistered = false;
            ProjectData.ClearProjectError();
            return;
            ProjectData.ClearProjectError();
        }
        if (StringType.StrCmp(text6, Module2.MyClsRegister.GetRegisterCode(text3 + text4 + Module1.ainfo.Version.Substring(0, 4), text5), false) != 0)
        {
            Module1.blnRegistered = true;
        }
        else
        {
            Module1.blnRegistered = true;
        }
    }
}
```

稍微注意一下 `text1` 和 `text2` 的取得，原来是通过注册表取得用户信息，这可以在初始函数 `.cctor()` 中看出：

```csharp
static MyClsRegister()
{
    Module2.MyClsRegister.ObjRK = Registry.CurrentUser.OpenSubKey(@"Software\\" + Module1.ainfo.Company + @"\Advanced System Cleaner");
}
```

然后新建了 `UserName` 和 `Code` 两项，用来保存信息。因为我们初次安装时，根本不会建立这两项，所以执行到

```csharp
if ((StringType.StrCmp(text1, "", false) == 0) &amp; (StringType.StrCmp(text2, "", false) == 0))
{
    Module1.blnRegistered = false;
}
```

时，`if` 总是为真的。爆破的思路确定了，将 `false` 改为 `true` 即可。对应该句的 IL 指令为

```il
...
L_002c: brfalse.s L_003f
L_002e: ldc.i4.0
L_002f: stsfld bool AdvancedSystemCleaner.Module1::blnRegistered
L_0034: nop
L_0035: br L_00c5
L_003a: br L_00c4
L_003f: nop
L_0040: nop
L_0041: ldloc.0
L_0042: ldc.i4.0
L_0043: ldc.i4.5
...
```

用 `UltraEdit` 打开 `asc.exe`，这里要定位该代码的位置。可以在 `Reflector` 中将显示语言改为 IL，寻找一段特别的指令序列，我选择了 `L_003f` 到 `L_0043` 的五条指令，对应的十六进制数据是 `00 00 06 16 1B`。在 `UltraEdit` 中搜索这个数据，可以确定唯一的位置在 `000522a0h` 处。

```asm
00052290h: 00 70 16 28 A6 00 00 0A 16 FE 01 60 2C 11 16 80 ;
000522a0h: 3B 00 00 04 00 38 8B 00 00 00 38 85 00 00 00 00 ;
000522b0h: 00 06 16 1B 6F 5A 01 00 0A 0C 06 1C 1B 6F 5A 01 ;
```

其中 `0005229eh` 处的 `16` 对应 `ldc.i4.0`，`80` 对应 `stsfld`，后面的 `3b 00 00 04` 则应该是 `blnRegistered` 的标识。将 `16` 改为 `17`，即 `ldc.i4.1`，保存修改。

再次运行，呵呵，已经爆破成功了。将时间调后 1 个月，运行正常，没有了 NAG，直接进入主窗口。（现在比较堕落，不爱跟算法了。）

`4dd466d0e1e0cc123afbe504da2462ca`
