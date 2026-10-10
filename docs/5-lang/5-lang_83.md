

# 再堕落一次:破解慧源通用进销存(.net)

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-97.htm`](https://www.pediy.com/kssd/pediy07/pediy7-97.htm)

名称：慧源通用进销存  
版本：单机版 1.13  
编译：VB.net  
网址：www.hysb.net  
保护方式：模糊器(Obfuscator)  
  
圈内人常说喜欢爆破的人比较堕落，我不是圈内人，所以不在乎，这不，又堕落了一次。  
首先运行一下，弹出如下窗口，显示未注册版本还能使用几次。  
用 Reflector 打开慧源数霸.EXE，发现经过了模糊，满眼都是毫无意义的 b、h、au 等等。（图 2）  
![](img/b3690760c57381ee1938b877f0403629.jpg)   
点击“注册”，弹出窗口中要输入用户名和密码。  
  
这是我第一次面对模糊过的.NET 程序，只能摸索自已的思路。现在首先要做的就是从茫茫代码中找出判断注册码的算法所在。用 ILDASM 将慧源数霸的 EXE 反汇编，可以得到很多资源文件。我们用 Resorucer 打开这些资源，发现了突破口，因为几乎每个资源中都有一张图片，表示出当前窗口。还记得我们刚才的注册窗口中的“软件注册”的图片吗，找的就是它。  
  
找遍所有的资源，都没有发现这张图片，难道注册码判断不在主文件中？我完全有可能。于是将所有的 DLL 反汇编，终于，在 HYSB.DLL 中被发现。原来注册窗口的主代码就是 HYSB.DLL-->am 中的 private void b(object A\_0, EventArgs A\_1)和 private string a(string A\_0, string A\_1, string A\_2)。  
   
而注册窗口中的序列号刚是在.ctor()中实现的。代码如下：  

>  
> 
> ```
> 代码:
> 
> * * *
> 
> public am()
> {
>       int num1;
>       int num2;
>       string text1;
>       string text2;
>       this.n = false;
>       this.p = new au();
>       this.a();
>       string text3 = @"C:\\";
>       am.GetVolumeInformation(ref text3, ref text1, 0xff, ref this.q, ref num1, ref num2, ref text2, 0x100);
>       this.r = StringType.FromInteger(this.q);
>       this.r = Strings.Mid(this.r, this.r.Length - 4);
>       this.l().Text = this.r;
> }
> 
> * * *
> 
> 
> ```

  
   
简单看了一下，算法本身并不难，可是我试着写了注册机，就是算不出来（功力还不够啊），再加上满眼的模糊过的代码，看的头晕。算了，还是爆破吧。软件是每次运行时检测注册，而这里的代码只是将注册信息写入注册表，肯定不能在这里爆破，我们得寻找软件运行时判断注册的初始代码。这不难，已经知道了是根据注册表来的，那就以 Registry.CurrentUser.OpenSubKey 在代码中寻找。  
  
于是用 ILDASM 将 MYSB.DLL 反汇编，存储成 MYSB.il，然后在其中搜寻刚才提到的关键词，很快可以找到关键比较处在 MYSB.DLL-->au-->d 中。其中，我们又找到了熟悉的代码 private string a(string A\_0, string A\_1, string A\_2)，这和刚才的注册窗口中的判断代码是同样的，看其返回值，是 string，应该是某代码调用它，然后对返回的 string 进行比较，所以爆破因选在比较 string 处而不是 a 中。  
  
想一想，比较应该是返回 bool 值，这里成员并不多，一个一个浏览，最终发现了比较代码在 public bool h()中。代码如下：  

>  
> 
> ```
> 代码:
> 
> * * *
> 
> public bool h()
> {
>       bool flag1;
>       try
>       {
>             string text1;
>             string text2;
>             string text3;
>             RegistryKey key1;
>             if (this.d())
>             {
>                   this.b();
>                   text2 = @"Software\\HYSB\\Soft\\" + this.a;
>                   key1 = Registry.CurrentUser.OpenSubKey(text2);
>                   if (key1 == null)
>                   {
>                         string text5 = @"Software\\Microsoft\\Windows\\" + this.a;
>                         RegistryKey key2 = Registry.CurrentUser.OpenSubKey(text5);
>                         if (key2 != null)
>                         {
>                               string text6 = StringType.FromObject(key2.GetValue("Code"));
>                               string text7 = StringType.FromObject(key2.GetValue("NoCode"));
>                               key1 = Registry.CurrentUser.CreateSubKey(text2);
>                               key1 = Registry.CurrentUser.OpenSubKey(text2, true);
>                               key1.SetValue("Code", this.c(text6));
>                               key1.SetValue("NoCode", this.c(text7));
>                               key1.Close();
>                               key2.Close();
>                         }
>                   }
>                   key1 = Registry.CurrentUser.OpenSubKey(text2);
>                   if ((key1 != null) && (key1.GetValue("Code") != null))
>                   {
>                         text1 = this.a((byte\[\]) key1.GetValue("Code"));
>                         text3 = this.a((byte\[\]) key1.GetValue("NoCode"));
>                         goto Label\_0152;
>                   }
>             }
>             return false;
>       Label\_0152:
>             key1.Close();
>             text2 = @"AdmiSoft\\" + this.a;
>             key1 = Registry.CurrentUser.OpenSubKey(text2);
>             if (key1 == null)
>             {
>                   return false;
>             }
>             string text4 = this.a((byte\[\]) key1.GetValue("UnitName"));
>             if (StringType.StrCmp(this.a(text4, text3, this.a), text1, false) == 0)
>             {
>                   return true;
>             }
>             if (StringType.StrCmp(this.a(text4, text3), text1, false) == 0)
>             {
>                   Interaction.MsgBox("\\u672c\\u8f6f\\u4ef6\\u5df2\\u7ecf\\u66f4\\u65b0\\u6ce8\\u518c\\u53f7\\u3002\\u5982\\u679c\\u60a8\\u662f\\u4e00\\u4e2a\\u5df2\\u7ecf\\u6ce8\\u518c\\u7684\\u6b63\\u7248\\u7528\\u6237\\uff0c\\u8bf7\\u5411\\u8f6f\\u4ef6\\u5f00\\u53d1\\u5546\\u7d22\\u53d6\\u65b0\\u7684\\u6ce8\\u518c\\u53f7\\u3002", MsgBoxStyle.Information, this.i());
>                   return true;
>             }
>             flag1 = false;
>       }
>       catch (Exception exception2)
>       {
>             ProjectData.SetProjectError(exception2);
>             Exception exception1 = exception2;
>             this.c(exception1);
>             ProjectData.ClearProjectError();
>       }
>       return flag1;
> }
> 
> * * *
> 
> 
> ```

  
  
看一看代码，就是从注册表中读取数值，然后比较。（其时这个软件保护很一般，将注册表的 AmdiSoft 键值删除，那使用次数就清 0，又可以再用 60 次。）为了保险，其时是方便，将其中所有的返回 flase 的地方全部改为返回 true。上次是用 UltraEdit 在文件中直接修改的，这次在代码中修改。  
  
打开反汇编得到的 MYSB.il，找到 9619 行，这里应该是  

>  
> 
> ```
> 代码:
> 
> * * *
> 
>   .method /\*06000085\*/ public instance bool 
>           h() cil managed
>   // SIG: 20 00 02
>   {
>     // 方法在 RVA 0x6550 处开始
>     // 代码大小       533 (0x215)
> 
> * * *
> 
> 
> ```

  
然后将 h()方法中以下五处的 ldc.i4.0 修改，分别为 IL\_000a、IL\_00f6、IL\_0145、IL\_017d、IL\_01f2，修改为 ldc.i4.1。保存。其实这五处代码都如下：  

>  
> 
> ```
> 代码:
> 
> * * *
> 
>       IL\_01f2:  /\* 16   |                  \*/ ldc.i4.0
>       IL\_01f3:  /\* 0A   |                  \*/ stloc.0
>       IL\_01f4:  /\* DE   | 1D               \*/ leave.s    IL\_0213
> 
> * * *
> 
> 
> ```

  
由于 IL 语言以栈为基本操作对象，这三句代码意议很明确，给第一个局部变量传递 0 值，那个变量是啥子呢？向上看看就可以找到.locals /\*11000034\*/ init (bool V\_0，这对应于 Reflector 反编译的 bool flag1。  
  
到这里，修改就完毕了。运行命令 ilasm.exe /DLL /RESOURCE=MYSB.res MYSB.il，不一会生成了.DLL 文件。用它覆盖原来的 MYSB.DLL，运行，呵呵，已经不再显示未注册窗口了。btw:好像该系列的软件保护方式都一样。 

 4dd466d0e1e0cc123afbe504da2462ca