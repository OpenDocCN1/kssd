
# VB.NET WinXp Manager V4.8.8.0 算法

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-100.htm`](https://www.pediy.com/kssd/pediy07/pediy7-100.htm)

## WinXp Manager V4.8.8.0 算法

首先感谢 `tankaiha` 给的 `Reflector` 下载地址，没有 `Reflector` 这个超级好的工具，

用 `ildasm` 还不知道要研究到何年何月呢，

`peid` `V0.92` 检测：Microsoft Visual C# / Basic .NET

它是用 Microsoft Visual Basic .NET 编写的，这可以从版本信息里看出来

首先当然是把 `WinXP Manager.exe` 拖到 `Reflector` 里，双击 `WinXP Manager/WinXP Manager.exe/WinXP_Manager`，就可以看到 `WinXP_Manager` 名称空间里的各个类了

我想作者的编程风格很好，要不然我怎么一眼就看到了 `frmRegister` 类了呢^-^

双击 `frmRegister` 类，又会看见一大堆方法、属性、字段等东西，

找到了 `BtnRegistry_Click`，就点它，......恐怖!

`Reflector` 竟然把它的源代码清清楚楚的显示在我们面前，`Reflector` 可以用 IL, C#, VB.NET, Delphi 四种风格显示代码，

这个软件没用用混淆器，这么好的软件，可惜

```vb.net
private void BtnRegistry_Click(object sender, EventArgs e)
{
    if (this.WriteRegisterCode(this.txtSN.Text, this.txtUserName.Text))      //这里调用 WriteRegisterCode()方法写注册码
    {
        MessageBox.Show("\u8bf7\u60a8\u91cd\u65b0\u542f\u52a8\u672c\u8f6f\u4ef6.", "\u5b8c\u6210", MessageBoxButtons.OK, MessageBoxIcon.Asterisk);                      //这里是用 Unicode 表示的"请您重新启动本软件"
        Application.Exit();
    }
    else
    {
        MessageBox.Show("\u6ce8\u518c\u8f6f\u4ef6\u5931\u8d25, \u8bf7\u8054\u7cfb gayxq@tom.com \u6216 gayxq@163.com", "\u9519\u8bef", MessageBoxButtons.OK, MessageBoxIcon.Hand);
    }
}
```

点 `WriteRegisterCode()` 方法，跳到这个方法上来，

它是把注册码写入注册表的，没有运算

它是重启效验，所以我进入 `load/main()` 里:

```vb.net
[STAThread]
public static void Main()
{
    //.....略
    Encrypt.MyClsIsRegister register1 = new Encrypt.MyClsIsRegister();    
    register1.Registered();                                              //用这个 Registered()方法验证
    register1 = null;
    if (publicVarFun.IsRegistered)
    {
        Encrypt.MyClsRegister.DelExprie();
        frmSplash splash1 = new frmSplash();
        splash1.Show();
    }
    else
    {
        frmOverdue overdue1 = new frmOverdue();
        overdue1.Show();
    }
    Application.Run();
}
```

到 `Registered()` 方法里看它怎么验证的

```vb.net
public void Registered()
{
    string text1 = Encrypt.MyClsRegister.GetRegCodeValue();                //这个方法读取注册表中的注册码
    string text2 = Encrypt.MyClsRegister.GetRegCodeWord();                 //这个方法读取用户名
    if ((StringType.StrCmp(text1, "", false) == 0) | (StringType.StrCmp(text2, "", false) == 0))  //为空就 OVER
    {
        publicVarFun.IsRegistered = false;
    }
    else if (StringType.StrCmp(text1, this.FromHDSNGetRegisterCode(text2), false) != 0)        //注册码和运算结果比较,经典的明码比较
    {
        publicVarFun.IsRegistered = false;
    }
    else
    {
        publicVarFun.IsRegistered = true;
    }
}
```

追到 `FromHDSNGetRegisterCode` 方法里

```vb.net
private string FromHDSNGetRegisterCode(string strCodeWord)
{
    Encrypt.MyClsEncrypt encrypt1 = new Encrypt.MyClsEncrypt(strCodeWord);       // 这个有参数构造器初始化了要用到的数据
    return encrypt1.Encrypt(Encrypt.MyClsRegister.GetPySerialNum());             /*用自己写的 Encrypt.MyClsEncrypt 类里的 Encrypt 方法计算,用 GetPySerialNum()方法获得硬盘 ID*/
}
```

到这个构造器里看看

```vb.net
public MyClsEncrypt(string strCodeWord)
{
    this.strCryptMatrix = new string[0x62];                                      /*先初始化 strCryptMatrix 的长度,按照作者的目的,长度为 62 就可以了,不知道为什么长度 0x62*/
    this.CodeWord = this.EncryptCode("Chinese" + strCodeWord + publicVarFun.ainfo.Version);  /*下面看 EncryptCode()方法,输入参数是 ("Chinese"+用户名+版本) 的一个字符串*/
    this.Matrix = "8x3p5BeabcdfghijklmnoqrstuvwyzACDEFGHIJKLMNOPQRSTUVWXYZ1246790";                           //Matrix 长度=62      
    int num1 = 1;
    this.LAM = Strings.Len(this.Matrix);                                       
    this.strCryptMatrix[1] = this.Matrix;
    int num3 = this.LAM;
    for (int num2 = 2; num2 <= num3; num2++)
    {
        this.mov1 = Strings.Left(this.strCryptMatrix[num1], 1);
        this.mov2 = Strings.Right(this.strCryptMatrix[num1], this.LAM - 1);
        this.strCryptMatrix[num2] = this.mov2 + this.mov1;
        num1++;
    }                                             //把 Matrix 变换 62 次后全放在 strCryptMatrix 里
}
```

## 版本信息

```vb.net
public string Version
{
    get
    {
        string[] textArray1 = new string[6] { "V", StringType.FromInteger(this.myType.Assembly.GetName().Version.Major), ".", StringType.FromInteger(this.myType.Assembly.GetName().Version.Minor), ".", StringType.FromInteger(this.myType.Assembly.GetName().Version.Build) } ;
        return string.Concat(textArray1);
    }
}
```

分别是主版本 4, 次版本 8, Build 应该是内部版本号 8

进入 `EncryptCode()` 方法:

```vb.net
private string EncryptCode(string strEncrypted)
{
    string text1;
    MD5CryptoServiceProvider provider1 = new MD5CryptoServiceProvider();           //用现成的 MD5 类
    UTF8Encoding encoding1 = new UTF8Encoding();
    byte[] buffer1 = provider1.ComputeHash(encoding1.GetBytes(strEncrypted));      //MD5 加密
    byte[] buffer2 = buffer1;
    for (int num2 = 0; num2 < buffer2.Length; num2++)
    {
        byte num1 = buffer2[num2];
        text1 = text1 + StringType.FromByte(num1);                                 //转换为字符串
    }
    return StringType.FromInteger(Math.Abs(text1.GetHashCode()));                  /*Math.Abs()取正值,GetHashCode()方法,说是得到希哈代码,在.NET 里几乎每个实例都有这个方法*/
}
```

追到 `Encrypt` 方法里，算法就在这里:

```vb.net
public string Encrypt(string strEncrypted)                 //硬盘 ID
{
    string text2;
    try
    {
        string text3 = strEncrypted;
        this.LS2E = strEncrypted.Length;                   //硬盘 ID 长度
        this.LCW = this.CodeWord.Length;
        this.EncryptedLetter = "";
        this.EncryptedString = "";
        int num2 = 1;
        int num5 = this.LS2E - 1;
        for (int num1 = 1; num1 <= num5; num1++)
        {
            string text1 = Strings.Mid(text3, num1, 1);        //依次取 ID 字符
            this.MP = Strings.InStr(1, this.Matrix, text1, CompareMethod.Binary);  //字符在 Matrix 里的位置
            this.CWL = Strings.Mid(this.CodeWord, num2, 1);    //依次取 CodeWord 的字符
            int num4 = this.LAM;                               
            for (int num3 = 1; num3 <= num4; num3++)
            {
                if (StringType.StrCmp(Strings.Mid(this.strCryptMatrix[num3], this.MP, 1), this.CWL, false) == 0) //ID 字符在 Matrix 中的位置就是 strCryptMatrix[num3]的相应位置字符
                {
                    this.EncryptedLetter = Strings.Left(this.strCryptMatrix[num3], 1);   //最左端字符
                    this.EncryptedString = this.EncryptedString + this.EncryptedLetter;  //连接上面的字符
                    break;
                }
            }
            num2++;
            if (num2 > this.LCW)
            {
                num2 = 1;
            }
        }
        text2 = Strings.Trim(this.EncryptedString).ToUpper();
    }
    catch (Exception exception1)
    {
        ProjectData.SetProjectError(exception1);
        ProjectData.ClearProjectError();
    }
    return text2;
}
```

## 算法总结

得到 `MD5("Chinese"+用户名+版本信息)` 散列，转换为字符串，叫 `CodeWord` 吧，

重新组合这个字符串，例如字符串"12345"放在 `strCryptMatrix[]` 里

```
strCryptMatrix[0] = "12345"
strCryptMatrix[1] = "23451"
strCryptMatrix[2] = "34512"
strCryptMatrix[3] = "45123"
```

准备下次用
