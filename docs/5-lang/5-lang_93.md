

# VB.NET WinXp Manager V4.8.8.0 算法

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-100.htm`](https://www.pediy.com/kssd/pediy07/pediy7-100.htm)

WinXp Manager V4.8.8.0 算法  
首先感谢 tankaiha 给的 Reflector 下载地址,没有 Reflector 这个超级好的工具,  
用 ildasm 还不知道要研究到何年何月呢,  
peid V0.92 检测:Microsoft Visual C# / Basic .NET  
它是用 Microsoft Visual Basic .NET 编写的,这可以从版本信息里看出来  
  
首先当然是把 WinXP Manager.exe 拖到 Reflector 里,双击 WinXP Manager/WinXP Manager.exe/WinXP\_Manager,就可以看到 WinXP\_Manager 名称空间里的各个类了  
  
我想作者的编程风格很好,要不然我怎么一眼就看到了 frmRegister 类了呢^-^  
双击 frmRegister 类,又会看见一大堆方法,属性,字段等东西,  
找到了 BtnRegistry\_Click,就点它,......恐怖!  
Reflector 竟然把它的源代码清清楚楚的显示在我们面前,Reflector 可以用 IL,c#,VB.NET,Delphi 四种风格显示代码,  
这个软件没用用混淆器,这么好的软件,可惜  
private void BtnRegistry\_Click(object sender, EventArgs e)  
{  
      if (this.WriteRegisterCode(this.txtSN.Text, this.txtUserName.Text))      //这里调用 WriteRegisterCode()方法写注册码  
      {  
            MessageBox.Show("\\u8bf7\\u60a8\\u91cd\\u65b0\\u542f\\u52a8\\u672c\\u8f6f\\u4ef6.", "\\u5b8c\\u6210", MessageBoxButtons.OK, MessageBoxIcon.Asterisk);                                                      //这里是用 Unicode 表示的"请您重新启动本软件"  
            Application.Exit();  
      }  
      else  
      {  
            MessageBox.Show("\\u6ce8\\u518c\\u8f6f\\u4ef6\\u5931\\u8d25, \\u8bf7\\u8054\\u7cfb gayxq@tom.com \\u6216 gayxq@163.com", "\\u9519\\u8bef", MessageBoxButtons.OK, MessageBoxIcon.Hand);  
      }  
}  
点 WriteRegisterCode()方法,跳到这个方法上来,  
它是把注册码写入注册表的,没有运算  
它是重启效验,所以我进入 load/main()里:  
\[STAThread\]  
public static void Main()  
{  
      //.....略  
      Encrypt.MyClsIsRegister register1 = new Encrypt.MyClsIsRegister();     
      register1.Registered();                                                 //用这个 Registered()方法验证  
      register1 = null;  
      if (publicVarFun.IsRegistered)  
      {  
            Encrypt.MyClsRegister.DelExprie();  
            frmSplash splash1 = new frmSplash();  
            splash1.Show();  
      }  
      else  
      {  
            frmOverdue overdue1 = new frmOverdue();  
            overdue1.Show();  
      }  
      Application.Run();  
}  
到 Registered()方法里看它怎么验证的  
public void Registered()  
{  
      string text1 = Encrypt.MyClsRegister.GetRegCodeValue();                          //这个方法读取注册表中的注册码  
      string text2 = Encrypt.MyClsRegister.GetRegCodeWord();                           //这个方法读取用户名  
      if ((StringType.StrCmp(text1, "", false) == 0) | (StringType.StrCmp(text2, "", false) == 0))  //为空就 OVER  
      {  
            publicVarFun.IsRegistered = false;  
      }  
      else if (StringType.StrCmp(text1, this.FromHDSNGetRegisterCode(text2), false) != 0)        //注册码和运算结果比较,经典的明码比较  
      {  
            publicVarFun.IsRegistered = false;  
      }  
      else  
      {  
            publicVarFun.IsRegistered = true;  
      }  
}  
追到 FromHDSNGetRegisterCode 方法里  
private string FromHDSNGetRegisterCode(string strCodeWord)  
{  
      Encrypt.MyClsEncrypt encrypt1 = new Encrypt.MyClsEncrypt(strCodeWord);       // 这个有参数构造器初始化了要用到的数据  
      return encrypt1.Encrypt(Encrypt.MyClsRegister.GetPySerialNum());             /\*用自己写的 Encrypt.MyClsEncrypt 类里的 Encrypt 方法计算,用 GetPySerialNum()方法获得硬盘 ID\*/  
}  
到这个构造器里看看  
public MyClsEncrypt(string strCodeWord)  
{  
      this.strCryptMatrix = new string\[0x62\];                                                   /\*先初始化 strCryptMatrix 的长度,按照作者的目的,长度为 62 就可以了,不知道为什么长度 0x62\*/  
      this.CodeWord = this.EncryptCode("Chinese" + strCodeWord + publicVarFun.ainfo.Version);  /\*下面看 EncryptCode()方法,输入参数是 ("Chinese"+用户名+版本) 的一个字符串\*/  
      this.Matrix = "8x3p5BeabcdfghijklmnoqrstuvwyzACDEFGHIJKLMNOPQRSTUVWXYZ1246790";                                                  //Matrix 长度=62       
      int num1 = 1;  
      this.LAM = Strings.Len(this.Matrix);                                                       
      this.strCryptMatrix\[1\] = this.Matrix;  
      int num3 = this.LAM;  
      for (int num2 = 2; num2 <= num3; num2++)  
      {  
            this.mov1 = Strings.Left(this.strCryptMatrix\[num1\], 1);  
            this.mov2 = Strings.Right(this.strCryptMatrix\[num1\], this.LAM - 1);  
            this.strCryptMatrix\[num2\] = this.mov2 + this.mov1;  
            num1++;  
      }                                                                         //把 Matrix 变换 62 次后全放在 strCryptMatrix 里  
}  
版本信息  
public string Version  
{  
      get  
      {  
            string\[\] textArray1 = new string\[6\] { "V", StringType.FromInteger(this.myType.Assembly.GetName().Version.Major), ".", StringType.FromInteger(this.myType.Assembly.GetName().Version.Minor), ".", StringType.FromInteger(this.myType.Assembly.GetName().Version.Build) } ;  
            return string.Concat(textArray1);  
      }  
}  
分别是主版本 4,次版本 8,Build 应该是内部版本号 8  
  
  
进入 EncryptCode()方法:  
private string EncryptCode(string strEncrypted)  
{  
      string text1;  
      MD5CryptoServiceProvider provider1 = new MD5CryptoServiceProvider();           //用现成的 MD5 类  
      UTF8Encoding encoding1 = new UTF8Encoding();  
      byte\[\] buffer1 = provider1.ComputeHash(encoding1.GetBytes(strEncrypted));      //MD5 加密  
      byte\[\] buffer2 = buffer1;  
      for (int num2 = 0; num2 < buffer2.Length; num2++)  
      {  
            byte num1 = buffer2\[num2\];  
            text1 = text1 + StringType.FromByte(num1);                            //转换为字符串  
      }  
      return StringType.FromInteger(Math.Abs(text1.GetHashCode()));                    /\*Math.Abs()取正值,GetHashCode()方法,说是得到希哈代码,在.NET 里几乎每个实例都有这个方法\*/  
}  
  
  
追到 Encrypt 方法里,算法就在这里:  
public string Encrypt(string strEncrypted)                             //硬盘 ID  
{  
      string text2;  
      try  
      {  
            string text3 = strEncrypted;  
            this.LS2E = strEncrypted.Length;                          //硬盘 ID 长度  
            this.LCW = this.CodeWord.Length;  
            this.EncryptedLetter = "";  
            this.EncryptedString = "";  
            int num2 = 1;  
            int num5 = this.LS2E - 1;  
            for (int num1 = 1; num1 <= num5; num1++)  
            {  
                  string text1 = Strings.Mid(text3, num1, 1);        //依次取 ID 字符  
                  this.MP = Strings.InStr(1, this.Matrix, text1, CompareMethod.Binary);  //字符在 Matrix 里的位置  
                  this.CWL = Strings.Mid(this.CodeWord, num2, 1);    //依次取 CodeWord 的字符  
                  int num4 = this.LAM;                                
                  for (int num3 = 1; num3 <= num4; num3++)  
                  {  
                        if (StringType.StrCmp(Strings.Mid(this.strCryptMatrix\[num3\], this.MP, 1), this.CWL, false) == 0) //ID 字符在 Matrix 中的位置就是 strCryptMatrix\[num3\]的相应位置字符  
                        {  
                              this.EncryptedLetter = Strings.Left(this.strCryptMatrix\[num3\], 1);   //最左端字符  
                              this.EncryptedString = this.EncryptedString + this.EncryptedLetter;  //连接上面的字符  
                              break;  
                        }  
                  }  
                  num2++;  
                  if (num2 > this.LCW)  
                  {  
                        num2 = 1;  
                  }  
            }  
            text2 = Strings.Trim(this.EncryptedString).ToUpper();  
      }  
      catch (Exception exception1)  
      {  
            ProjectData.SetProjectError(exception1);  
            ProjectData.ClearProjectError();  
      }  
      return text2;  
}  
算法总结:  
得到 MD5("Chinese"+用户名+版本信息)散列,转换为字符串,叫 CodeWord 吧,  
重新组合这个字符串,例如字符串"12345"放在 strCryptMatrix\[\]里  
strCryptMatrix\[0\]="12345"  
strCryptMatrix\[1\]="23451"  
strCryptMatrix\[2\]="34512"  
strCryptMatrix\[3\]="45123"  
准备下次用  
这时候,记住 strEncrypted 是硬盘 ID,CodeWord 是和注册名有关的加密结果,字符串 Matrix="8x3p5BeabcdfghijklmnoqrstuvwyzACDEFGHIJKLMNOPQRSTUVWXYZ1246790"  
string 数组 strCryptMatrix\[\]放着 CodeWord 变形的结果  
LAM 是 Matrix 的长度为 62  
下面:  
依次取 ID,查找字符在 Matrix 的位置,记做 MP,  
依次取 CodeWord,记做 CWL  
如果 CWL 在 strCryptMatrix\[i\]中的位置==MP  
就在结果上连接 strCryptMatrix\[i\]的第一个字符  
循环次数=硬盘 ID 的长度-1  
去空格,大写  
到这种地步,应该说编写注册机应该很容易了,还有版本信息,  
单击 WinXP Manager,就是那个最高级的,就可以看到它的版本信息了:  
  
\[assembly: AssemblyVersion("4.8.8.0")\]  
  
但这还不是软件计算时用到的字符串,软件计算时的版本信息字符串是"V4.8.8",  
我猜它猜了两天!!!!!!!  
它的注册信息保存在 HKEY\_CURRENT\_USER\\Software\\Yanicsoft\\WinXP Manager 下  

更正你几个错误:  
首先这个软件是 c#的.  
其次这个软件构造的矩阵不是对用户名变换后够造的.而是对"8x3p5BeabcdfghijklmnoqrstuvwyzACDEFGHIJKLMNOPQRSTUVWXYZ1246790"这张表构造的.  
还有 4.9.0 的算法没有改变,为什么不直接分析 4.9.0 呢?  
  
下面是 4.9.0 的注册机的源代码,早做出来了.既然有人分析这个软件,我就直接贴出来吧:  
其中:m\_machine 是机器码,  
     m\_code    是生成的注册码,  
     m\_name    是用户名.  
void MD5(unsigned char output\[\],unsigned char input\[\],\_int64 inputlen);  
void CWinXPDlg::OnButton1()   
{  
    CString table = "8x3p5BeabcdfghijklmnoqrstuvwyzACDEFGHIJKLMNOPQRSTUVWXYZ1246790";  
    int tablelen;  
  int machinelen;  
    CString matrix\[62\];  
    CString machine;  
    CString nametable;  
    CString move1,move2;  
  CString c1,c2;  
  CString mycode;  
  unsigned char tmp\[100\];  
  unsigned char tmp1\[20\];  
  int loc = 0;  
  int cnt = 0;  
  int index;  
  int index1 = 0;  
    int i;  
  
    UpdateData();  
    nametable = "Chinese" + m\_name + "V4.9.0";  
  memset(tmp,0,100);  
    for(i=0;i<nametable.GetLength();i++)  
  {  
    char c = nametable.GetAt(i);  
    if(c >= 0 && c <= 0x7f)  
    {  
      tmp\[loc\] = c;  
      loc ++;  
    }  
    else if(c >= 0x80 && c <= 0x7ff)  
    {  
      tmp\[loc\] = (c>>6) & 0x1f | 0xc0;  
      loc ++;  
      tmp\[loc\] = (c & 0x3f) | 0x80;  
      loc ++;  
    }  
    else if(c >= 0x800 && c <= 0xffff)  
    {  
      tmp\[loc\] = (c >> 12) & 0x0f | 0xe0;  
      loc ++;  
      tmp\[loc\] = (c >> 6) & 0x3f | 0x80;  
      loc ++;  
      tmp\[loc\] =  c & 0x3f | 0x80;  
      loc ++;  
    }  
  }  
    memset(tmp1,0,20);  
    MD5(tmp1,tmp,loc);  
    memset(tmp,0,100);  
  nametable = "";  
  for(i=0;i<16;i++)  
  {  
    sprintf((char\*)tmp,"%u",tmp1\[i\]);  
    nametable = nametable + tmp;  
  }  
  
    machine   = m\_machine;  
  machinelen = machine.GetLength();  
  tablelen  = table.GetLength();  
  matrix\[0\] = table;  
  //构造那张巨大的矩阵 ^\_^  
    for(i = 1; i < tablelen; i++)  
    {  
    move1 = matrix\[cnt\].Left(1);  
    move2 = matrix\[cnt\].Right(tablelen - 1);  
        matrix\[i\] = move2 + move1;  
    cnt ++;  
  }  
      
  for(i = 0; i < machinelen -1; i ++)  
  {  
    c1 = machine.Mid(i,1); //取 machine 一个字符  
        index = table.Find(c1); //取 table 中的索引  
    if(index == -1)return;  
    c2 = nametable.Mid(index1,1); //取 nametable 中索引位置的字符  
    for(int j = 0; j < tablelen; j++)  
    {  
      if(matrix\[j\].GetAt(index) == c2.GetAt(0))  
      {//如果字符和矩阵中响应位置的字符相等,则取矩阵首字符  
        mycode = mycode + matrix\[j\].Left(1);  
        break;  
      }  
    }  
    index1 ++;  
    if(index1 == nametable.GetLength())index1 = 0;  
  }  
  
    mycode.MakeUpper();  
  m\_code = mycode;  
    UpdateData(FALSE);  
} 

只能怪小弟才疏学浅,希望大家原谅  
ForEver 大哥,不知到你是怎样判断软件版本的,请指教  
我是因为当初找不到它的具体版本,才找到了这里:  
\[assembly: AssemblyVersion("4.8.8.0")\]  
\[assembly: CLSCompliant(true)\]  
\[assembly: AssemblyTitle("WinXP\\u603b\\u7ba1")\]  
\[assembly: Guid("E3170BA9-3567-4AE3-A275-8697CA62FEA2")\]  
\[assembly: AssemblyCompany("Yanicsoft")\]  
\[assembly: Debuggable(true, true)\]  
\[assembly: AssemblyCopyright("2003-2004 \\u672c\\u8f6f\\u4ef6\\u53d7\\u6cd5\\u5f8b\\u4fdd\\u62a4,\\u4fdd\\u7559\\u6240\\u6709\\u6743\\u5229.")\]  
\[assembly: AssemblyTrademark("")\]  
\[assembly: AssemblyProduct("WinXP\\u603b\\u7ba1")\]  
\[assembly: AssemblyDescription("\\u7528\\u6765\\u5bf9WindowsXP\\u8fdb\\u884c\\u4f18\\u5316\\u53ca\\u8bbe\\u7f6e\\u7684\\u8f6f\\u4ef6")\]  
我试着把这些全贴到注册机里的 AssemblyInfo.cs 里,我用的是 C#,但不能编译,提示找不到类或名称空间,是因为这三条  
  
\[assembly: Guid("E3170BA9-3567-4AE3-A275-8697CA62FEA2")\]  
\[assembly: Debuggable(true, true)\]  
\[assembly: CLSCompliant(true)\]  
  
我在里面加上:  
using System.Runtime .InteropServices;  
using System.Diagnostics ;  
两条 using 指令,这时,还是不行,是这条  
\[assembly: CLSCompliant(true)\]  
我没办法用这条,但我到 VB.NET 里,可以看到在 C#里默认没有的上面三条,在 VB.NET 里都是默认存在的,  
还有我看它的代码,我用 Reflector 时选的是用 C#的语言风格,但里面有许多 VB 才有的如:  
Strings.Len  
Strings.Right  
等,我就认为它是 VB.NET,希望指正  
至于矩阵什么的,别怪我笨,这是我第二此听到,一直以为研究它需要高深的数学知识,誓不研究加密算法![](img/b16bec246e75299c87cf57e1fb808ac9.jpg)   
请老大给个 V4.9.0 的链接 

功夫不负有心啦，终于自己也把注册机搞出来了（用 C#写的，因为我目前只学过一点 C#。）。有朋友想要的，可以向我索取，我不能上传附件。  
楼上的朋友，那个 Right 什么的，可以自己写一个嘛。比如：  
new public static string Right(string str, int Length)  
{  
  if ((Length == 0) || (str == null))  
    {  
      return "";  
    }  
  int num1 = str.Length;  
  if (Length >= num1)  
  {  
    return str;  
  }  
  return str.Substring(num1 - Length, Length);  
} 

关于版本字符串,  
 this.strCryptMatrix = new string\[0x62\];  
 this.CodeWord = this.EncryptCode("Chinese" + strCodeWord + publicVarFun.ainfo.Version);  
接着点上面的 version,会来到下面:  
  
public string get\_Version()  
{  
      string\[\] textArray1 = new string\[6\] { "V", StringType.FromInteger(this.myType.Assembly.GetName().Version.Major), ".", StringType.FromInteger(this.myType.Assembly.GetName().Version.Minor), ".", StringType.FromInteger(this.myType.Assembly.GetName().Version.Build) } ;  
      return string.Concat(textArray1);  
}  
这就是"V4.9.0"的由来.  
  
4.9.0 的下载地址:  
http://winxpmanager.nease.net/Chinese/index.html 

抱歉。这个软件应该是 vb.net 的。我搞错了。  
软件引用了 vb 的命名空间，还有一些 vb 特有的函数。  
这两天公司的网络时断时续。回迟了。望包涵。  
关于本贴我不再跟踪了。有问题可以到 RCT 官方站点：  
bbs.xp-program.com 找我。 

 4dd466d0e1e0cc123afbe504da2462ca