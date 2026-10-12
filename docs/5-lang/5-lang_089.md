
# Xenocode fox

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-171.htm`](https://www.pediy.com/kssd/pediy08/pediy8-171.htm)

**【文章标题】**: 静态分析破 `Xenocode` 2006 14 天评估版  

**【文章作者】**: `dreaman`  

**【作者邮箱】**: `dreaman_163@163.com`  

**【作者主页】**: `http://dreaman.haolinju.net`  

**【下载地址】**: 看雪论坛《工具下载》有提到下载地址  

**【加壳方式】**: 无壳  

**【操作平台】**: `.net` 2.0  

**【软件介绍】**: 一个类似 `Reflector` 的软件，不过不是免费的。  

**【作者声明】**: 请参照看雪破文的惯例声明  

---

### 详细过程

今天想起来要用一下 `Xenocode`，结果运行时被告知 14 天试用时间已到，非常感谢评估，请付费云云。

有点不太爽，就研究了一下，结果发现超级简单！

#### 1、查找程序入口点

首先用 `reflector` 打开看了看，一堆 `x07168889` 之类的类名与成员，试了一下动态调试，调试器一装

载 `fox.exe` 就会直接运行，弹出过期信息，点确定后直接退出。

于是拿出 `ildasm` 来，反出的 IL 文件来，用 `UltraEdit` 打开这个文本文件，搜索

`System.Windows.Forms.Application::Run`

（猜想 `windows form` 程序应该会用这个函数作主循环 [D3D 的好象不用这个，代之 `DoEvents`]）

找到的地方上下文如下：

```
.class private abstract auto ansi sealed beforefieldinit Xenocode.Fox.Gui.x867eb3246b182488
    extends [mscorlib]System.Object
{
    .field public static int32 x7ee7e0aa39016337
    .method private hidebysig static void  xc447809891322395() cil managed
    {
      .entrypoint
      .custom instance void [mscorlib]System.STAThreadAttribute::.ctor() = ( 01 00 00 00 )
      // 代码大小       30 (0x1e)
      .maxstack  8
      IL_0000:  call       bool Xenocode.Fox.Gui.x867eb3246b182488::_xc7c43f12e732db09()
      IL_0005:  brfalse.s  IL_0008
      IL_0007:  ret
      IL_0008:  call       void [System.Windows.Forms]System.Windows.Forms.Application::EnableVisualStyles()
      IL_000d:  ldc.i4.0
      IL_000e:  call       void [System.Windows.Forms]System.Windows.Forms.Application::SetCompatibleTextRenderingDefault(bool)
      IL_0013:  newobj     instance void Xenocode.Fox.Gui.x3e4e23fadc83a77e::.ctor()
      IL_0018:  call       void [System.Windows.Forms]System.Windows.Forms.Application::Run(class 
[System.Windows.Forms]System.Windows.Forms.Form)
      IL_001d:  ret
    } // end of method x867eb3246b182488::xc447809891322395
```

由此知此调用出现在类 `Xenocode.Fox.Gui.x867eb3246b182488` 中，换 `reflector` 继续。

#### 2、回到 reflector

在名空间 `Xenocode.Fox.Gui` 下查到类 `x867eb3246b182488` (别费劲记这个名啊，就找以 `2488` 结尾的，呵呵)

，选 `c#` 语言，瞧瞧

```
internal static class x867eb3246b182488
{
        // Methods
        static x867eb3246b182488();
        public static bool _xc7c43f12e732db09();
        [STAThread]
        private static void xc447809891322395();
        // Fields
        public static int x7ee7e0aa39016337;
}
```

嗯，很标准的 `windows form` 启动类。

#### 3、现在该看看主函数了

```
[STAThread]
private static void xc447809891322395()
{
        if (!x867eb3246b182488._xc7c43f12e732db09())//关键判断
        {
              Application.EnableVisualStyles();
              Application.SetCompatibleTextRenderingDefault(false);
              Application.Run(new x3e4e23fadc83a77e());
        }
}
```

`[STAThread]` 这个属性应该也可作为程序启动函数的一个标记（多线程的另说了）！

可以看到启动循环前有个判断，呵呵，这就是关键了。

#### 4、看看程序正常运行前的这个函数

```
public static bool _xc7c43f12e732db09()
{
        if (x71a0073930f50f3f.xb30f5e1eb4806151("XFTRL" + x77fa6322561797a0.x9c1ceef9a932f141, new TimeSpan
(x867eb3246b182488.x7ee7e0aa39016337, 0, 0, 0, 0)))
        {
              MessageBox.Show(MessageTable.TrialExpired, x77fa6322561797a0.x6886d5a1867d55cb);
              return true;
        }
        return false;
}
```

还真是暴露的可以啊：）

看看这个：

```
new TimeSpan(x867eb3246b182488.x7ee7e0aa39016337, 0, 0, 0, 0)
```

第一个参数是什么呢？点一下来到这里：

```
public static int x7ee7e0aa39016337;
```

在 `reflector` 与上面这个 field 对应的树叶上右键=>`Analyzer`, 展开右边显示的 `Analyzer` 窗口中的

树:

```
Xenocode.Fox.Gui.x867eb3246b182488.x7ee7e0aa39016337 : Int32
  Used By
    Xenocode.Fox.Gui.x867eb3246b182488..cctor()
    Xenocode.Fox.Gui.x867eb3246b182488._xc7c43f12e732db09() : Boolean
```

看到在两个地方引用了这个 field, 一个是构造函数，一个是刚才我们看到的判断函数，看一下构造

里做了什么：

```
static x867eb3246b182488()
{
        x867eb3246b182488.x7ee7e0aa39016337 = 14;
}
```

`14` ？？？！！！不是 14 天评估时间吗？！

#### 5、继续看看怎么判断时间过期的

来到 `x71a0073930f50f3f.xb30f5e1eb4806151`

```
public static bool xb30f5e1eb4806151(string x2d32ac7a8f1f8f5b, TimeSpan x5906905c888d3d98)
{
        bool flag1;
        try
        {
              uint num1;
              DateTime time1;
              TimeSpan span1;
              uint num2;
              RegistryKey key1 = Registry.LocalMachine.CreateSubKey(@"Software\\Oak Vale Networks\\" + 
x2d32ac7a8f1f8f5b);//用到了参数 1 作注册表键名，从前面知参数 1 为 XFTRL 开头的串，回去看一下知道这个串是"XFTRL1.0"
              if ((((uint) flag1) - num1) >= 0)//用两个局部变量作减法，迷惑人的代码，一定成立,到 0074 了
              {
                    goto Label_0074;
//后面有一段是逆指令流，所以请直接跳到 0074 处，然后从后往前看。
              }
              goto Label_005D;
        Label_0037:
              flag1 = true;//唉，过期了
              if (num2 < 0)//又来了个迷惑人的，这个东西根本就是 0，不会跳的
              {
                    goto Label_007D;
              }
              return flag1;//报告，过期了，告诉他“让他掏钱买吧”
        Label_0052:
              if (span1 >= x5906905c888d3d98)//差 >= TimeSpan(14,0,0,0,0)
              {
                    goto Label_0037;//再往回跳，这便是过期之后的事了
              }
              return false;//报告，还没有过期！
        Label_005D:
              span1 = (TimeSpan) (DateTime.UtcNow - time1);//计算当前时间与记录时间之差
              goto Label_0052;//又往回跳，感觉跟逆指令流似的了！！！
        Label_0074:
              if (key1.GetValue(null) == null)//读注册表中的默认值，没有数据就去 00BB 了
              {
                    goto Label_00BB;
              }
        Label_007D:
              num1 = uint.Parse((string) key1.GetValue(null));//转成整数
              time1 = DateTime.FromOADate(((double) num1) / 10);//除 10 成 OLE 日期数据，转成.net 的日期结构
              if ((num1 - ((uint) flag1)) >= 0)//又减了一次没用过的局部变量，继续迷惑人
              {
                    goto Label_005D;//往回跳了
              }
              goto Label_0074;//跳回去了，如果 num1<0 可就死循环了，不过从前面知，这是不可能的。
        Label_00BB:
              num2 = (uint) (DateTime.UtcNow.ToOADate() * 10);//当前时间转 OLE 格式并乘 10
              key1.SetValue(null, num2.ToString());//写到默认值，前面是没有默认值时跳来的，
所以这个分支是初始记录分支，第一次安装时把当时的时间记在注册表
              return false;
              if (num1 <= uint.MaxValue)
              {
                    goto Label_007D;
              }
              goto Label_005D;
        }
        catch (Exception)
        {
              flag1 = true;//如果出现异常，也认为是过期，假设代码被改错了，也算作过期了！
        }
        return flag1;
}
```

#### 6、OK，到此已经非常清楚是如何过期的了

我把我的当前日期按上面的计算方法得到的注册表中的值

后面加了一个 `0`, 也就是 `3887410`，这个值表示过期时间为 `2964-5-1`, 不知道那时候还是不是国际劳动节

啊。好了，我们来看一下注册表

```
HKEY_LOCAL_MACHINE\SOFTWARE\Oak Vale Networks
```

其下有两个子键:

```
XCTRL4.1
XFTRL1.0
```

后面这个就是 `fox`, 前面那个呢，是装 `Xenocode` 时一并安装的 `PostBuild`，将它们的默认值都改为

`3887410` 吧，可以用好多代了啊：）

#### 7、改完后看了一下 fox.exe

发现没有签名，说明应该是可以 `crack` 的，再次拿出 `ILDasm` 来，运行，

选上“视图”菜单=》显示字节，然后打开主入口函数的反汇编：

```
.method private hidebysig static void  xc447809891322395() cil managed
// SIG: 00 00 01
{
  .entrypoint
  .custom instance void [mscorlib]System.STAThreadAttribute::.ctor() = ( 01 00 00 00 )
  // 方法在 RVA 0x112d68 处开始
  // 代码大小       30 (0x1e)
  .maxstack  8
  IL_0000:  /* 28   | (06)001A21       */ call       bool Xenocode.Fox.Gui.x867eb3246b182488::_xc7c43f12e732db09()
  IL_0005:  /* 2C   | 01               */ brfalse.s  IL_0008
  IL_0007:  /* 2A   |                  */ ret
  IL_0008:  /* 28   | (0A)00047E       */ call       void [System.Windows.Forms]System.Windows.Forms.Application::EnableVisualStyles()
  IL_000d:  /* 16   |                  */ ldc.i4.0
  IL_000e:  /* 28   | (0A)00047F       */ call       void [System.Windows.Forms]
```



`System.Windows.Forms.Application::SetCompatibleTextRenderingDefault(bool)`

```
IL_0013:  /* 73   | (06)001848       */ newobj     instance void Xenocode.Fox.Gui.x3e4e23fadc83a77e::.ctor()
IL_0018:  /* 28   | (0A)000480       */ call       void [System.Windows.Forms]
System.Windows.Forms.Application::Run(class [System.Windows.Forms]System.Windows.Forms.Form)
IL_001d:  /* 2A   |                  */ ret
} // end of method x867eb3246b182488::xc447809891322395
```

这里写明了函数开始于 `RVA 0x112d68`，我们要做的是把正常运行前的 `if()` 判断去掉，也就是

```
IL_0000:  /* 28   | (06)001A21       */ call       bool Xenocode.Fox.Gui.x867eb3246b182488::_xc7c43f12e732db09()
IL_0005:  /* 2C   | 01               */ brfalse.s  IL_0008
IL_0007:  /* 2A   |                  */ ret
```

这三行，直接 `nop` 掉，在 `.net` 中 `nop` 指令最好记了，它就是 `00`。

找出一个二进制编辑器，跳到 `0x112d68-0x1000=0x111d68` 处，用眼睛扫描一下，`28 21 1A 00 06`（`ILDasm` 里显示的操作数居然不是内存顺序，晕）。

后面接着是 `2C 01 2A`，将这些全部改为 `00`，另存为 `UnFox.exe`，运行一下，成功！

再把注册表里的数改成过期数据，比如改为 `0`，再运行一下 `UnFox.exe`，呀，又成功了！

呵呵，祝贺一下（坏了！！！我本来想用 `Xenocode` 做什么来着？？？）

--------------------------------------------------------------------------------

【经验总结】

1. 用 `ILDasm` 反汇编完整文件，然后搜索 `System.Windows.Forms.Application::Run` 来发现程序入口点（其实搜索 `.entrypoint` 属性应该更准确了，还有 `STAThread` 对单线程程序也有效）；
2. `reflector` 就是好啊就是好，别忘了它还有个 `Analyzer` 查找引用（`Used By`）很有效，另外，在 `C#` 与 `IL` 间切换，可以学习 `IL` 语言哦（鼠标停在 `IL` 指令上能看到指令解释及指令机器码，上次有兄弟已经说过了，抄袭一下：）
3. `.net` 程序没有签名的话，好象很不安全呢。

--------------------------------------------------------------------------------

【版权声明】: 本文原创于看雪论坛，转载请注明出处。

2006 年 06 月 06 日 14:30:41

`4dd466d0e1e0cc123afbe504da2462ca`
