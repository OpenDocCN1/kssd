

# 破解 WebCombo.NET Version 2.0

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-343.htm`](https://www.pediy.com/kssd/pediy07/pediy7-343.htm)

【软件名称】：WebCombo.NET Version 2.0  
【软件类别】：国外软件  
【整理时间】：2005-08-09  
【下载地址】：http://ftp.evget.com/product/InterSoft/WebCombo2TrialSetup.zip  
【保护方式】：NAG + 试用时间 30 天  
【保护软件】：Dotfuscator  
【编译语言】：ASP.NET  
【调试环境】：WinXP、.NET Framework 1.1、Reflector、ildasm、sn、ultraedit  
【破解日期】：2005-08-08  
【破解目的】：研究.NET 程序的破解流程  
【作者声明】：初学 Crack，只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教！  
  
【破解过程】：  
相信用过 Reflector 的朋友都感受到 Reflector 的强大，先用 Reflector 打开 ISNet.WebUI.WebCombo.dll，在庞大的类文件里面查找我们感兴趣的信息。可以看到里面部分 internal class 的类名已经被混淆器混淆，幸运的是还没有经过反 Reflector 的处理，依然可以直接静态分析代码。  
  
首先我们从经过混淆的部分开始看，按照逻辑来推测，既然作者没有全部代码都进行混淆，那么混淆的部分必然是存在一些秘密在里面。很快我们可以找到 internal class a 的 a()方法（注意，这个不是构造函数，构造函数是.ctor，并且如果有同名而不同参数的函数，不要按照惯性思维认为是同一个功能的不同参数版本。那些都是混淆器干的好事而已）  
  
public void a(){  
      string text1 = Environment.GetFolderPath(Environment.SpecialFolder.System);  
      string text2 = text1 + @"\\c4184ef0d326354b.data";//放一个文件到 windows\\system32  
      FileStream stream1 = File.Open(text2, FileMode.Create, FileAccess.Write);  
      DateTime time1 = DateTime.Today;  
      this.a = time1;  
      this.b = time1;//记录当前日期  
      this.c = time1.AddDays(30);//这里的 30 就是试用 30 天了，因此这个是过期日期  
      this.e = (TimeSpan) (this.c - this.b);//这个就是剩下使用天数  
      this.d = false;  
      string text3 = "";  
      string\[\] textArray1 = new string\[7\] { this.a.ToShortDateString(), ",", this.b.ToShortDateString(), ",", this.c.ToString(), ",", this.d.ToString() } ;  
      text3 = string.Concat(textArray1);  
      byte\[\] buffer1 = a.a(text3);  
      DESCryptoServiceProvider provider1 = new DESCryptoServiceProvider();  
      provider1.Key = this.g;  
      provider1.IV = this.g;  
      ICryptoTransform transform1 = provider1.CreateEncryptor();      CryptoStream stream2 = new CryptoStream(stream1, transform1, CryptoStreamMode.Write);  
      stream2.Write(buffer1, 0, buffer1.Length);//把上面几个参数经过 DES 变换后写到文件中  
      stream2.Flush();  
      stream2.Close();  
      stream2 = null;  
}  
  
显然这里用到了密码学的加密变换，看起来是在第一次运行的时候执行的。这软件可真黑，在 system32 里面增加垃圾文件，去里面找找，果然找到一个 data 文件。既然有写文件，必定有读文件，因此我们继续找，来到这  
public void b()  
{  
      string text1 = Environment.GetFolderPath(Environment.SpecialFolder.System);  
      string text2 = text1 + @"\\c4184ef0d326354b.data";  
      if (!File.Exists(text2))//如果文件不存在  
      {  
            this.a();//调用上面的 a()初始化过期时间等信息  
      }  
      else  
      {  
            FileStream stream1 = File.Open(text2, FileMode.Open, FileAccess.Read);  
            DESCryptoServiceProvider provider1 = new DESCryptoServiceProvider();  
            provider1.IV = this.g;  
            provider1.Key = this.g;  
            ICryptoTransform transform1 = provider1.CreateDecryptor();  
            CryptoStream stream2 = new CryptoStream(stream1, transform1, CryptoStreamMode.Read);  
            string text3 = new StreamReader(stream2, new UnicodeEncoding()).ReadToEnd();  
            char\[\] chArray1 = new char\[1\] { ',' } ;  
            string\[\] textArray1 = text3.Split(chArray1);  
            stream2.Close();  
            transform1.Dispose();  
            provider1.Clear();  
            stream1.Close();  
            this.a = DateTime.Parse(textArray1.GetValue(0).ToString());  
            this.b = DateTime.Parse(textArray1.GetValue(1).ToString());  
            this.c = DateTime.Parse(textArray1.GetValue(2).ToString());  
            DateTime time1 = DateTime.Today;  
            if (time1 < this.b)//如果比第一次运行时间早  
            {  
                  this.d = true;//判断为非法  
            }  
            else if (time1 == this.b)//试用的第一天  
            {  
                  if (this.b > this.c)//好像没什么可能第一天就过期吧  
                  {  
                        this.d = true;  
                  }  
            }  
            else if (time1 > this.b)  
            {  
                  this.b = time1;  
                  if (this.b > this.c)//比试用结束日期晚  
                  {  
                        this.d = true;  
                  }  
                  this.c();//跟进去  
            }  
            if (!this.d)//未过期  
            {  
                  this.e = (TimeSpan) (this.c - this.b);//重新计算剩余过期时间  
            }  
      }  
}  
  
public void c()//不注释了，就是把更新后的资料重新保存到文件中  
{  
      string text1 = Environment.GetFolderPath(Environment.SpecialFolder.System);  
      string text2 = text1 + @"\\c4184ef0d326354b.data";  
      FileStream stream1 = File.Open(text2, FileMode.Open, FileAccess.Write);  
      string text3 = "";  
      string\[\] textArray1 = new string\[7\] { this.a.ToShortDateString(), ",", this.b.ToShortDateString(), ",", this.c.ToString(), ",", this.d.ToString() } ;  
      text3 = string.Concat(textArray1);  
      byte\[\] buffer1 = a.a(text3);  
      DESCryptoServiceProvider provider1 = new DESCryptoServiceProvider();  
      provider1.Key = this.g;  
      provider1.IV = this.g;  
      ICryptoTransform transform1 = provider1.CreateEncryptor();  
      CryptoStream stream2 = new CryptoStream(stream1, transform1, CryptoStreamMode.Write);  
      stream2.Write(buffer1, 0, buffer1.Length);  
      stream2.Close();  
      stream2 = null;  
}  
  
到这里，我们可以确定出这个类是用来判断过期与否的，可是并没有注册验证的过程。于是打开 ICryptoTransform 的 callee graph，找一下这个 dll 里面还有没其他使用到密码转换的过程，结果发现就只有这三个函数 L  
  
换一个思路，去找 class a 的 callee，找到 class d 里面的函数  
private void b(object A\_0, EventArgs A\_1)  
{  
      this.f.Links\[0\].LinkData = "http://www.intersoftpt.com";  
      if (this.l == ISNet.WebUI.WebCombo.WebCombo.a.c)//合法 license  
      {  
            RegistryKey key1 = Registry.LocalMachine.OpenSubKey(@"Software\\Intersoft Solutions");  
            if (key1 != null)  
            {  
                  try  
                  {  
                        RegistryKey key2 = key1.OpenSubKey(@"WebCombo.NET\\2.0.2500");  
                        this.i.Text = key2.GetValue("User") as string;  
                        this.j.Text = key2.GetValue("Organization") as string;  
                        this.k.Text = key2.GetValue("SID") as string;  
                  }  
                  catch  
                  {  
                        this.i.Text = "Unknown user.";  
                        this.j.Text = "Unknown organization.";  
                        this.k.Text = "Unknown license.";  
                  }  
            }  
      }  
      else if (this.l == ISNet.WebUI.WebCombo.WebCombo.a.e)//试用 license  
      {  
            this.o = new a(this.m);  
            this.o.b();  
            this.i.Text = "Trial User.";  
            this.j.Text = "Trial Version.";  
            this.m.u = this.o.d;  
            if (this.o.d)  
            {  
                  this.k.Text = "Trial period has expired.";  
                  this.n.Visible = true;  
            }  
            else  
            {  
                  this.k.Text = "This trial product will expire in " + this.o.e.Days + " day(s).";  
            }  
      }  
      else  
      {  
            this.i.Text = "Unlicensed User.";  
            this.j.Text = "";  
            this.k.Text = "No License Key found.";  
      }  
}  
  
看来合法的 license 应该满足  
1.  this.l == ISNet.WebUI.WebCombo.WebCombo.a.c  
2.  HKLM\\Software\\Intersoft Solutions\\WebCombo.NET\\2.0.2500 有注册信息  
再看看 I,j,k 的定义  
      private Label i;  
      private Label j;  
      private Label k;  
是 asp.net 的标签组件，由此推测该类只是读取注册表来显示注册信息的。具体在注册表内有什么关系不大  
再看看 l 的值是哪里来的，在.ctor 看到  
public d(){  
      this.h = null;  
      this.l = ISNet.WebUI.WebCombo.WebCombo.a.b;  
      this.m = null;  
      this.o = null;  
      this.a();  
}  
找 this.l 的 callee，查到 ISNet.WebUI.WebCombo.WebCombo.b()  
                  if (this.a1 == ISNet.WebUI.WebCombo.WebCombo.a.e)  
                  {  
                        d d1 = new d();  
                        d1.m = this;  
                        d1.l = this.a1;  
                        d1.ShowDialog();  
                        d1.Dispose();  
                  }  
  
转到 ISNet.WebUI.WebCombo.WebCombo.a  
internal enum a  
{  
      // Fields  
      public int a;,  
      b = -1,//非法  
      c = 1,//合法 license  
      d = 2,//这个其实是 RuntimeOnly，在其他地方分析得出  
      e = 3//试用 license  
}  
  
看.ctor  
public WebCombo()  
{  
  …  
      this.a0 = "";  
      this.a1 = ISNet.WebUI.WebCombo.WebCombo.a.e;  
this.a("WebCombo.NET", "2.0.2500");  
…  
}  
  
明白了，条件只有一个，只要 ISNet.WebUI.WebCombo.WebCombo.a1 为 1，合法，否则为试用状态。初始化条件为试用状态。  
  
再查找 a1 的 callee，居然没有向它赋值的……因此推测这个字段是编译的时候固化在软件中的，改成 ISNet.WebUI.WebCombo.WebCombo.a.e，编译出来就是试用版，改成 ISNet.WebUI.WebCombo.WebCombo.a.c，就是正式版（真龌蹉-\_-）  
  
看来只能爆破解决。接下来问题是怎么修改 dll 文件，把这个初始值改成 ISNet.WebUI.WebCombo.WebCombo.a.c。先去找个破解的 ildasm（不要用 vs.net 自带那个，那个遇到被保护的 dll 就打不开了），转到 ISNet.WebUI.WebCombo.WebCombo..ctor  
  
  IL\_0194:  ldstr      ""  
  IL\_0199:  stfld      string ISNet.WebUI.WebCombo.WebCombo::az  
  IL\_019e:  ldarg.0  
  IL\_019f:  ldstr      ""  
  IL\_01a4:  stfld      string ISNet.WebUI.WebCombo.WebCombo::a0  
  IL\_01a9:  ldarg.0  
  IL\_01aa:  ldc.i4.3  //这里改成 ldc.i4.1 就成了  
  IL\_01ab:  stfld      valuetype ISNet.WebUI.WebCombo.WebCombo/a ISNet.WebUI.WebCombo.WebCombo::a1  
  IL\_01b0:  ldarg.0  
  IL\_01b1:  call       instance void class \[System.Web\]System.Web.UI.WebControls.WebControl::.ctor()  
  IL\_01b6:  ldarg.0  
  IL\_01b7:  ldstr      "WebCombo.NET"  
  IL\_01bc:  ldstr      "2.0.2500"  
  IL\_01c1:  call       instance void class ISNet.WebUI.WebCombo.WebCombo::a(string,  
                                                                            string)  
  IL\_01c6:  ldarg.0  
  
不用怕不懂 IL 的字节码，对照 Reflector 的结果你总可以半猜吧。找到位置了，转存为 il 文件，用 ultraedit 打开，找到这段代码（200 多 k 的 dll 居然转成 2M 的文件，直接查找吧），修改为  
IL\_01aa:  ldc.i4.1  
  
然后回到文件头，把 publickey 删掉，因为我们已经修改过 il 了，所以原来的签名肯定是无效了，我们要重新签名  
  
在 cmd 下输入  
sn –k MySign.sgn  //产生自己的公钥和私钥  
ilasm /dll /resource:dump.res /key:MySign.sgn dump.il /quiet  
提示已经成功生成了，把 dump.dll 改名覆盖回去，呵呵，这下可以爽了。别忘了要 rebuild 所有引用到它的 project 才可以正常运行哦（开始我就是忘记这一点，还以为有什么暗桩，最后居然是在它自己网站的 FAQ 上面找到答案……） 

 4dd466d0e1e0cc123afbe504da2462ca