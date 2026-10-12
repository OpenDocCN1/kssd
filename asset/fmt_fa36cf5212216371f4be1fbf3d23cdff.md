
# \[.NET\]\[爆破\]我的 Microsoft Math 3.0 激活之路

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-181.htm`](https://www.pediy.com/kssd/pediy09/pediy09-181.htm)

> \[标 题\]

> \[原创\]\[.NET\]\[爆破\]我的 Microsoft Math 3.0 激活之路

> \[作 者\]

> 快雪时晴，2007 年 6 月 16 日

> \[目 标\]

> 在 CNBeta 看到一则新闻：
>
> Microsoft Math 3.0 试用版发布啦！
>
> `CB_Freshman` 发布于 2007-06-14 06:54:32|3185 次阅读 字体：大 小 打印预览
>
> 来源：AppBeta
>
> 微软推出的 `Math` 工具提供了强大的数学工具，尤其适合学生和教师，可以帮助他们逐步解方程，更好的理解代数学、几何学、物理、化学和微积分等。
>
> （该软件为收费软件，30 天试用时间）
>
> `Math` 的界面左侧被设计成一个计算器模型，右侧则是主要的显示区域。主要功能有：
>
> 1. 图形化计算器：具有广泛的图形和解方程能力，具有制作 2D 和增强的 3D 彩色图形功能，有助于人们可视化解决问题并理解概念。
> 2. 逐步解方程：从基本的数学问题到微积分，可以解决许多数学问题。
> 3. 公式和方程库：具有 100 多个常用方程和公式。
> 4. 解三角形。
> 5. 单位换算。
> 6. 新：支持 `Tablet` 和 `Ultra-Mobile PC` 的数字墨水技术，可以通过手写解决许多 `Math` 可以识别的问题。
>
> 提示：`Math` 是收费软件，这个是 30 天试用版。
>
> 每次程序运行都会提示，30 天后必须输入 25 位产品序列 ID。
>
> 下载地址：`http://msft-dnl.digitalrivercontent.net/msoffice/pub/X13-66853/X13-66853.exe`
>
> oh-yeah，就是它！

> \[工 具\]

> `PEID`、`OllyICE`、`010Editor`、`Reflector`、`ILDasm`、
> `ILAsm`、`SN.exe`、`SNRemove`、`SNReplace.exe`、`StrongName Patcher`、
> `Abel_Load231`、`DUP2.15final`
> `PEBrowseDbg_pro`

> \[平 台\]

> `EN-WINXPSP2+MUI`
> `.Net Framwork v2.0.50727`

> \[第一部分\] 探究关键代码

> 主程序 `MATHAPP.EXE` 为.NET 程序，当然首先采用静态分析方法了（动态调试还不熟悉，工具和参考资料都远不及 WIN32 PE）。
>
> 微软的东西就是好，不加壳，没有应用代码混淆，很快就找到关键地方。

```
\===============================================Reflector 逆向情况========================================================
private static void Main(string[] args);

Declaring Type: Microsoft.MicrosoftMath.Application.AppMain
Assembly: MathApp, Version=3.0.1184.1020

[STAThread]
private static void Main(string[] args)
{
    Application.SetUnhandledExceptionMode(UnhandledExceptionMode.ThrowException);
    AppDomain.CurrentDomain.UnhandledException += new UnhandledExceptionEventHandler(AppMain.OnUnhandledException);
    ResManager.InitializeMinimum();
    SingleInstance.AppName = (ResManager.SKU == null) ? "EncCalc" : ("EncCalc-" + ResManager.SKU);
    if (SingleInstance.CheckSingleInstance(args, false, true) == SingleInstanceState.AlreadyRunning)
    {
        return;
    }
    AppResManager.Initialize();
    AppDrawingInfo.Initialize();
    uint dwGraceTime = 0;
    AppResManager.Activated = NativeMethods.CheckLicenseStatus(ResManager.SKU, out dwGraceTime) != 0;
    if (!CheckEula())
    {
        return;
    }
    CheckSQM();
    if (!AppResManager.VerifyMSCertificate(Path.Combine(ResManager.AppDirectory, "MathRichEditNative.dll")))
    {
        AppResManager.ExitApp();
    }
    if (!AppResManager.Activated)
    {
        string pszProdKey = null;
        switch (NativeMethods.ShowActivationWizard(IntPtr.Zero, pszProdKey, 0, dwGraceTime, ResManager.SKU))
        {
            case 0:
            case 0xc000000c:
            case 0xf000000e:
                AppResManager.Activated = NativeMethods.CheckLicenseStatus(ResManager.SKU, out dwGraceTime) != 0;
                if ((dwGraceTime == 0) && !AppResManager.Activated)
                {
                    AppResManager.ExitApp();
                }
                goto Label_00FB;
        }
        if (dwGraceTime == 0)
        {
            AppResManager.ExitApp();
        }
    }
Label_00FB:
    Application.ApplicationExit += new EventHandler(AppMain.OnApplicationExit);
    Application.Idle += new EventHandler(AppMain.OnApplicationIdle);
    Application.EnableVisualStyles();
    MainForm mainForm = new MainForm();
    mainForm.SetCommandArgs(args);
    Application.Run(mainForm);
}
```

```
\====================================ILDASM 显示情况========================================================
.method private hidebysig static void  Main(string[] args) cil managed
// SIG: 00 01 01 1D 0E
{
  .entrypoint
  .custom instance void [mscorlib]System.STAThreadAttribute::.ctor() = ( 01 00 00 00 )
  // Method begins at RVA 0xb80c
  // Code size      313 (0x139)
  .maxstack  5
  .locals init (valuetype [MathControls]Microsoft.MicrosoftMath.Controls.SingleInstanceState V_0,
                uint32 V_1,
                string V_2,
                uint32 V_3,
                class Microsoft.MicrosoftMath.Application.MainForm V_4)
  IL_0000:  /* 17   |                */ ldc.i4.1
  IL_0001:  /* 28   | (0A)0001DD     */ call       void [System.Windows.Forms]System.Windows.Forms.Application::SetUnhandledExceptionMode(valuetype [System.Windows.Forms]System.Windows.Forms.UnhandledExceptionMode)
  IL_0006:  /* 28   | (0A)0001DE     */ call       class [mscorlib]System.AppDomain [mscorlib]System.AppDomain::get_CurrentDomain()
  IL_000b:  /* 14   |                */ ldnull
  IL_000c:  /* FE06 | (06)00009C     */ ldftn      void Microsoft.MicrosoftMath.Application.AppMain::OnUnhandledException(object, class [mscorlib]System.UnhandledExceptionEventArgs)
  IL_0012:  /* 73   | (0A)0001DF     */ newobj     instance void [mscorlib]System.UnhandledExceptionEventHandler::.ctor(object,native int)
  IL_0017:  /* 6F   | (0A)0001E0     */ callvirt   instance void [mscorlib]System.AppDomain::add_UnhandledException(class [mscorlib]System.UnhandledExceptionEventHandler)
  IL_001c:  /* 28   | (0A)0001E1     */ call       void [MathControls]Microsoft.MicrosoftMath.Controls.ResManager::InitializeMinimum()
  IL_0021:  /* 28   | (0A)0001E2     */ call       string [MathControls]Microsoft.MicrosoftMath.Controls.ResManager::get_SKU()
  IL_0026:  /* 2C   | 11             */ brfalse.s  IL_0039
  IL_0028:  /* 72   | (70)0006A9     */ ldstr      "EncCalc-"
  IL_002d:  /* 28   | (0A)0001E2     */ call       string [MathControls]Microsoft.MicrosoftMath.Controls.ResManager::get_SKU()
  IL_0032:  /* 28   | (0A)000091     */ call       string [mscorlib]System.String::Concat(string,string)
  IL_0037:  /* 2B   | 05             */ br.s       IL_003e
  IL_0039:  /* 72   | (70)0006BB     */ ldstr      "EncCalc"
  IL_003e:  /* 28   | (0A)0001E3     */ call       void [MathControls]Microsoft.MicrosoftMath.Controls.SingleInstance::set_AppName(string)
  IL_0043:  /* 02   |                */ ldarg.0
  IL_0044:  /* 16   |                */ ldc.i4.0
  IL_0045:  /* 17   |                */ ldc.i4.1
  IL_0046:  /* 28   | (0A)0001E4     */ call       valuetype [MathControls]Microsoft.MicrosoftMath.Controls.SingleInstanceState [MathControls]Microsoft.MicrosoftMath.Controls.SingleInstance::CheckSingleInstance(string[],bool,bool)
  IL_004b:  /* 0A   |                */ stloc.0
  IL_004c:  /* 06   |                */ ldloc.0
  IL_004d:  /* 17   |                */ ldc.i4.1
  IL_004e:  /* 33   | 01             */ bne.un.s   IL_0051
  IL_0050:  /* 2A   |                */ ret
  IL_0051:  /* 28   | (06)00027A     */ call       void Microsoft.MicrosoftMath.Application.AppResManager::Initialize()
  IL_0056:  /* 28   | (06)00009E     */ call       void Microsoft.MicrosoftMath.Application.AppDrawingInfo::Initialize()
  IL_005b:  /* 16   |                */ ldc.i4.0
  IL_005c:  /* 0B   |                */ stloc.1
  IL_005d:  /* 28   | (0A)0001E2     */ call       string [MathControls]Microsoft.MicrosoftMath.Controls.ResManager::get_SKU()
```
