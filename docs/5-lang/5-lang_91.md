
# 对某.net OA 软件的破解记录

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-311.htm`](https://www.pediy.com/kssd/pediy08/pediy8-311.htm)

特别说明：本文仅做学习研究使用，有一定参考价值，是一些破解的心得和思路，完全是个人对程序的研究，因此也未说明破解的是哪个软件，不涉及商业行为，版权问题是另外一方面的事情了，本人对其他事务不负责

公司最近买了某公司的一个.net OA 软件进行二次开发，但由于其的许可证限制了同时可连接的用户数以及可使用的时间（截止时间到后变为演示版，演示版仅能 2 个用户），我们的客户一般比较多，每个人一个账号，在测试和二次开发方面不太方便，于是尝试自己破解一下。

先用 `Reflector` 查看了一下其 DLL 文件，发现使用了 `XenoCode` 进行混淆，但程度不是太深，基本上还不影响阅读。

根据观察，发现其所有页面都基于一个 `Page` 基类 `WebPage`，其中有一个关键函数 `Loginon()`，用于系统登录，其中判断能否登录，是根据一个 `login_result` 的结果来判断的。

`Login_result` 是一个 `enum`

```csharp
public enum LOGON_RESULT
{
    // Fields
    Fail = 0,
    Fail_Lic_Limit = -1,
    Success = 1,
    Success_Pwd_Expire = 3,
    Success_Pwd_Force_Change = 4,
    Success_Pwd_Length = 2
}
```

这是登录状态，那么肯定有返回这些状态的函数，通过查找，找到类 `SecurityMgr` 中包含有一个函数

```csharp
private SecurityMgr.LOGON_RESULT x5913b537a9ca4090(string x6b237db413aed7db, string xb45a741896260675, Page x9c79b5ad7b769b12)
```

用来返回登录状态。找到这个函数，反编译，查看代码，代码中的字符串都已经被 `XenoCode` 加密过，直接是看不出来的，好在我们可以利用反射来解密这些字符串的值（这些网上都有说明）。

转换后，我们可以看到，其对用户数的判断是在对数据库中的用户表进行 `top` 筛选来限制的：`select top " + license1.LicenseNumber + " username from users`

其中 `license1.LicenseNumber` 即是限制用户数的数目。这个从哪里来呢？往上看，可以看到：

```csharp
License license1 = new License(x9c79b5ad7b769b12.Application);
```

这样，我们就清楚了，所有的 `license` 信息是在此处读取的。找到这个类，发现 `license1.LicenseNumber` 这个属性对应的是 `x957eec0002850847.x01b6751913efbd51`

```csharp
public int LicenseNumber
{
    get
    {
        return this.x957eec0002850847.x01b6751913efbd51;
    }
}
```

那么 `x957eec0002850847.x01b6751913efbd51` 元素在哪里有被使用过呢？

通过 `Reflector` 查看 `Analyzer` 功能，我们可以发现，其在构造函数 `public License(HttpApplicationState app)` 和 `private void x1052178a2aa3affc(HttpApplicationState xbc9689dd53140596)` 中被使用。

对其进行分析，对字符串解码后，再去除花指令，对其混淆过的流程进行重新编排，我们可以看到构造函数中对这个属性的操作仅仅是初始化，那么赋值的地方在哪里呢？

往下看我们可以看到，当 `app["License"]` 为 `null` 时，会去调用：

```csharp
this.x822eb47bc415e83b(app);
this.x1052178a2aa3affc(app);
```

用于取得 `license` 信息。很容易的，我们知道 `x957eec0002850847.x01b6751913efbd51` 这个属性一定在 `this.x1052178a2aa3affc(app)` 函数中被改写以取到相应的用户限制数。

来到该函数中逆向依次搜索 `x01b6751913efbd51》text7》text5》text4`，得到这些关键语句：

```csharp
text1 = Encoding.Unicode.GetString(Convert.FromBase64String(this.x4c262adfb75e23fa.x3f7a1096840d5b39));
text2 = text1.Substring(0, 0x40);
string text3 = text1.Substring(0x40);
text4 = this.xcc381ffa3ede662f(text3);
string[] textArray1 = text4.Split(new char[] { ';' });
text5 = textArray2[num1];
text6 = text5.Trim().Substring(0, 3);
text7 = text5.Trim().Substring(3);
```

知道，这个属性是从 `text4` 中来的，而 `text4` 是由 `this.x4c262adfb75e23fa.x3f7a1096840d5b39` 经过一系列转换而来。

而 `x4c262adfb75e23fa.x3f7a1096840d5b39` 的值我们通过分析再结合前文，可以很自然的想到是从 `x822eb47bc415e83b(app)` 函数运行后得到。`x822eb47bc415e83b(app)` 比较简单，很容易就知道其作用是 `ReadLicenseFromDB`。

从中我们找到对应关系：`License` 下的一个包含 4 个元素的结构，用于记录从数据库中取到的信息，对比下程序，我们很容易找出其对应关系。

```csharp
private struct x29e2d415f271a930
{
    public string xf67f2f73d67cc6b6; // Key
    public string x3f7a1096840d5b39; // License
    public string x00af6e04034127cd; // Data
    public DateTime xd00ef6aee32c8aff; // CreateTime
}
```

到此，我们就已经知道，用户数信息是根据数据库中的 `license` 字段信息经过一系列换算得到。

那么我们结合 `private void x1052178a2aa3affc(HttpApplicationState xbc9689dd53140596)` 函数分析

```csharp
private void x1052178a2aa3affc(HttpApplicationState xbc9689dd53140596)
{
    try
    {
        string text1;
        string text2;
        string text4;
        string text5;
        string text6;
        string text7;
        string text8;
        string[] textArray2;
        int num1;
        string text9;
        DateTime time1;

        if (this.x4c262adfb75e23fa.x3f7a1096840d5b39 != null)
        {
            goto Label_079D;
        }
        return;

    Label_0015:
        xbc9689dd53140596[string.Intern(x1110bdd110cdcea4._d574bb1a8f3e9cbc("jccadejakdabjdhbpdobbefcadmcbbddddkdicbeocieoooe", 0x7db601dd))] = string.Intern(x1110bdd110cdcea4._d574bb1a8f3e9cbc("elodfjhegcdcglcdldmc", 0x4270c1b9)) + this.x957eec0002850847.xf98c0407c36b694d.ToString(string.Intern(x1110bdd110cdcea4._d574bb1a8f3e9cbc("lgocigfdfgmdcgdedbkeadbfncifkapfodggldng", 0x59062df2))) + string.Intern(x1110bdd110cdcea4._d574bb1a8f3e9cbc("hjdgmobe", 0x5594d3d0));

        if (-2 != 0)
        {
            return;
        }

    Label_008A:
        if (time1.AddDays(2) > this.x957eec0002850847.xf98c0407c36b694d)
        {
            goto Label_0015;
        }
        return;

    Label_00B1:
        if (DateTime.Now > this.x957eec0002850847.xf98c0407c36b694d)
        {
            goto Label_01CA;
        }

        if (((uint) num1) < 0)
        {
            goto Label_0499;
        }

        time1 = DateTime.Now;

        if ((((uint) num1) | 3) != 0)
        {
            goto Label_008A;
        }

        goto Label_0015;

    Label_0106:
        this.x957eec0002850847.x01b6751913efbd51 = 2;
        xbc9689dd53140596[string.Intern(x1110bdd110cdcea4._d574bb1a8f3e9cbc("dhpknigleinldiemjilmlicnkhjnlfaonhhochooihfpidmp", 0x1ac1af27))] = string.Intern(x1110bdd110cdcea4._d574bb1a8f3e9cbc("fpddgnmdgoccphkb", 0x75ddb6fa)) + this.x957eec0002850847.xf98c0407c36b694d.ToString(string.Intern(x1110bdd110cdcea4._d574bb1a8f3e9cbc("mflhjfcigfjidfajeahjbcojobfklplkpcdlmckl", 0x68787ae3))) + string.Intern(x1110bdd110cdcea4._d574bb1a8f3e9cbc("kofhpdef", 0x2cede623));

        return;

    Label_0187:
        xbc9689dd53140596[string.Intern(x1110bdd110cdcea4._d574bb1a8f3e9cbc("mfhdghodngfemgmechdfehkfdgbgeeigggpglfghbgnhacei", 0x2cb63710))] = string.Intern(x1110bdd110cdcea4._d574bb1a8f3e9cbc("gjbjcgokdehljockbknkedmmihef", 0x5a852a6a)) + this.x957eec0002850847.x1bf37c739ee04ce8;

        goto Label_00B1;

    Label_01CA:
        this.x957eec0002850847.x63d2aa540697615b = string.Intern(x1110bdd110cdcea4._d574bb1a8f3e9cbc("bdej", 0x45d793ed));

        goto Label_023D;

    Label_01ED:
        if (this.x957eec0002850847.x5d8a72c9b50ea3f1 != string.Intern(x1110bdd110cdcea4._d574bb1a8f3e9cbc("jleogllonlcpaljpnkaaelhahkoaekfblkmbojdcljkcckbdfjidcjpdjjgeminejief", 0x1f82e489)))
        {
            goto Label_031B;
        }
        goto Label_0231;

    Label_0218:
        if (text8 != this.x957eec0002850847.x5d8a72c9b50ea3f1)
        {
            goto Label_0288;
        }
        goto Label_0187;

    Label_0231:
        if (4 == 0)
        {
            goto Label_0218;
        }
        goto Label_0187;

    Label_023D:
        if (((uint) num1) >= 0)
        {
            goto Label_0453;
        }
        goto Label_01ED;

        if ((((uint) num1) - ((uint) num1)) > uint.MaxValue)
        {
            goto Label_065A;
        }

        if (8 == 0)
        {
            goto Label_07B8;
        }
        goto Label_0328;

    Label_0288:
        this.x957eec0002850847.x63d2aa540697615b = string.Intern(x1110bdd110cdcea4._d574bb1a8f3e9cbc("pmek", 0x6a1ca48b));
        this.x957eec0002850847.x01b6751913efbd51 = 2;
        LogService.Write(string.Intern(x1110bdd110cdcea4._d574bb1a8f3e9cbc("nbfjhdmjocdknckkddblfdilecplgpfmfanmpbengblnfbcolbjonbapmahpemnpeoeaoolaapcbhcihmiehpjfhcpjjkpnk", 0x40c094d1)));
        xbc9689dd53140596[string.Intern(x1110bdd110cdcea4._d574bb1a8f3e9cbc("gajbacachbhcgbocmbfdobmdnadeoojeabbffaiflapfkmfg", 0x374c18ba))] = string.Intern(x1110bdd110cdcea4._d574bb1a8f3e9cbc("bilecgefagceaooejijegpaemjngjpcgblng", 0x41cdce86));

        if (((uint) num1) < 0)
        {
            goto Label_0757;
        }
        return;

    Label_031B:
        text8 = this.x1083e6509fcf5d08();
        goto Label_0218;

    Label_0328:
        if (0 == 0)
        {
            goto Label_0187;
        }
    }
}
```



```
        }  

        goto Label_0453;  

    Label_0333:  

        if (1 == 0)  

        {  

            goto Label_04CC;  

        }  

        goto Label_01ED;  

    Label_0344:  

        if ((text9 = text6) != null)  

        {  

            goto Label_060C;  

        }  

    Label_034E:  

        num1++;  

    Label_0354:  

        if (num1 < textArray2.Length)  

        {  

            goto Label_0701;  

        }  

        if ((((uint) num1) & 0) == 0)  

        {  

            goto Label_0333;  

        }  

    Label_0373:  

        if (15 == 0)  

        {  

            goto Label_03D1;  

        }  

        goto Label_0394;  

    Label_037C:  

        if ((((uint) num1) - ((uint) num1)) < 0)  

        {  

            goto Label_0373;  

        }  

    Label_0394:  

        this.x957eec0002850847.x5d8a72c9b50ea3f1 = text7;  

        goto Label_034E;  

    Label_03A3:  

        this.x957eec0002850847.x1b3806a16cc4a0b2 = text7;  

        goto Label_034E;  

    Label_03B7:  

        if (text9 == string.Intern(x1110bdd110cdcea4._d574bb1a8f3e9cbc("dnmnjmdoflko", 0x256dc7e)))  

        {  

            goto Label_0404;  

        }  

        goto Label_03D8;  

    Label_03D1:  

        if (-2147483648 == 0)  

        {  

            goto Label_03B7;  

        }  

    Label_03D8:  

        if (text9 != string.Intern(x1110bdd110cdcea4._d574bb1a8f3e9cbc("jiegdjlgnhch", 0x6d846446)))  

        {  

            goto Label_05D6;  

        }  

        goto Label_037C;  

    Label_0404:  

        this.x957eec0002850847.x01b6751913efbd51 = int.Parse(text7);  

        if ((((uint) num1) - ((uint) num1)) > uint.MaxValue)  

        {  

            goto Label_0520;  

        }  

        if (((uint) num1) > uint.MaxValue)  

        {  

            goto Label_0354;  

        }  

        if (0 != 0)  

        {  

            goto Label_0708;  

        }  

        goto Label_034E;  

    Label_0453:  

        if ((((uint) num1) | 0xff) != 0)  

        {  

            goto Label_0106;  

        }  

    Label_046B:  

        this.x957eec0002850847.x63d2aa540697615b = text7;  

        goto Label_034E;  

    Label_047D:  

        this.x957eec0002850847.xf98c0407c36b694d = DateTime.Parse(text7);  

        goto Label_034E;  

    Label_0499:  

        if ((((uint) num1) | 2) == 0)  

        {  

            goto Label_0508;  

        }  

    Label_04B1:  

        if (text9 != string.Intern(x1110bdd110cdcea4._d574bb1a8f3e9cbc("koimmopmnmgn", 0x751bc896)))  

        {  

            goto Label_0635;  

        }  

    Label_04CC:  

        if ((((uint) num1) | 0x7fffffff) == 0)  

        {  

            return;  

        }  

        goto Label_046B;  

    Label_04EE:  

        if (text9 == string.Intern(x1110bdd110cdcea4._d574bb1a8f3e9cbc("gihkoiokkhfl", 0x5a6a743)))  

        {  

            goto Label_0538;  

        }  

        goto Label_0499;  

    Label_0508:  

        if ((((uint) num1) & 0) != 0)  

        {  

            goto Label_04EE;  

        }  

        goto Label_04B1;  

    Label_0520:  

        if ((((uint) num1) - ((uint) num1)) < 0)  

        {  

            goto Label_04EE;  

        }  

    Label_0538:  

        this.x957eec0002850847.x1bf37c739ee04ce8 = text7;  

        goto Label_034E;  

    Label_0553:  

        if (text9 == string.Intern(x1110bdd110cdcea4._d574bb1a8f3e9cbc("hphnpoonlnfo", 0x76c2d7a4)))  

        {  

            this.x957eec0002850847.xb644c3d4b9723516 = text7;  

            if ((((uint) num1) + ((uint) num1)) < 0)  

            {  

                goto Label_0499;  

            }  

            if (((uint) num1) > uint.MaxValue)  

            {  

                goto Label_03D1;  

            }  

            goto Label_034E;  

        }  

        if (0xff != 0)  

        {  

            goto Label_05FD;  

        }  

        if ((((uint) num1) + ((uint) num1)) <= uint.MaxValue)  

        {  

            goto Label_03D8;  

        }  

    Label_05D6:  

        if (text9 == string.Intern(x1110bdd110cdcea4._d574bb1a8f3e9cbc("gghclgocaffd", 0x7afc2719)))  

        {  

            goto Label_03A3;  

        }  

        goto Label_034E;  

    Label_05FD:  

        if (3 != 0)  

        {  

            goto Label_04EE;  

        }  

        goto Label_04B1;  

    Label_060C:  

        text9 = string.IsInterned(text9);  

        if ((((uint) num1) + ((uint) num1)) >= 0)  

        {  

            goto Label_0553;  

        }  

        goto Label_04EE;  

    Label_0635:  

        if (text9 == string.Intern(x1110bdd110cdcea4._d574bb1a8f3e9cbc("bmbcbnicdlpc", 0xaab217c)))  

        {  

            goto Label_047D;  

        }  

        goto Label_074D;  

    Label_065A:  

        text7 = text5.Trim().Substring(3);  

        goto Label_0344;  

    Label_0701:  

        text5 = textArray2[num1];  

    Label_0708:  

        text6 = text5.Trim().Substring(0, 3);  

        if (15 == 0)  

        {  

            goto Label_046B;  

        }  

        goto Label_065A;  

    Label_072C:;  

        string[] textArray1 = text4.Split(new char[] { ';' });  

        textArray2 = textArray1;  

        num1 = 0;  

        goto Label_0354;  

    Label_074D:  

        if (0 == 0)  

        {  

            goto Label_0779;  

        }  

        return;  

    Label_0757:  

        if (!this.xb12591d958637077(text4, text2))  

        {  

            return;  

        }  

        if ((((uint) num1) | 3) != 0)  

        {  

            goto Label_072C;  

        }  

    Label_0779:  

        if ((((uint) num1) + ((uint) num1)) <= uint.MaxValue)  

        {  

            goto Label_03B7;  

        }  

        goto Label_04B1;  

    Label_079D:  

        text1 = Encoding.Unicode.GetString(Convert.FromBase64String(this.x4c262adfb75e23fa.x3f7a1096840d5b39));  

    Label_07B8:  

        text2 = text1.Substring(0, 0x40);  

        string text3 = text1.Substring(0x40);  

        text4 = this.xcc381ffa3ede662f(text3);  

        goto Label_0757;  

    }  

    catch (Exception exception1)  

    {  

        LogService.Write(string.Intern(x1110bdd110cdcea4._d574bb1a8f3e9cbc("eebfofifffpfefggkfngmfehlelhnbciecjiceajndhjgeojidfkgdmkkbdledkllcbmkcimadpmcdgnbcnn", 0x6ca750f8)));  

        LogService.Write(exception1.Message);  

        return;  

    }  

}
```

可以得到这样的结果

`License` 先经过 `Convert.FromBase64String` 函数转换为 `byte[]`

然后再把得到的 `byte[]` 通过 `Encoding.Unicode.GetString` 函数转换为 unicode 字符串

然后取前 64 位存入字符串 `a`，剩余的存入字符串 `b`

对 `b` 使用对称算法密钥解密

```
ICryptoTransform transform1 = new DESCryptoServiceProvider().CreateDecryptor(buffer1, buffer2);
    byte[] buffer3 = Encoding.Unicode.GetBytes(b);
    byte[] buffer4 = transform1.TransformFinalBlock(buffer3, 0, buffer3.Length);
```

最后再转换 `Encoding.Unicode.GetString(buffer4);`

得到最终所要的字符串信息

这时候，使用“；”将字符串分割成字符串组

对每组进行判断，即可得到最终所要的信息。

这些信息，分别对应

```
private struct x9cebba6cbd2b2289
{
    public string xb644c3d4b9723516;//序列号 SN
    public string x1bf37c739ee04ce8;//公司名称 CN
    public string x63d2aa540697615b;//版本类型 TY
    public DateTime xf98c0407c36b694d;//过期时间 EX
    public int x01b6751913efbd51;//用户数 UN
    public string x5d8a72c9b50ea3f1;//CPU 编码 CP
    public string x1b3806a16cc4a0b2;//可用模块 MU
}
```

那么前面得到的前 64 位字符串 `a` 用来干什么的呢？

分析代码可知，这个是用来和 64 位后的字符串 `b` 比较，以验证签名是否正确的，查看签名验证算法函数

```
private bool xb12591d958637077(string x4a3f0a05c02f235f, string x8ebde9a98df80374)
{
    byte[] buffer2;
    RSAParameters parameters1;
    RSACryptoServiceProvider provider1;
    byte[] buffer3;
    byte[] buffer1 = Encoding.Unicode.GetBytes(x4a3f0a05c02f235f);
    if (3 != 0)
    {
        SHA1 sha1;
    Label_007B:
        sha1 = new SHA1Managed();
```



# 代码重构与分析

## 1. 代码整理

移除花指令并调整流程顺序后，代码结构如下：

```csharp
private bool xb12591d958637077(string x4a3f0a05c02f235f, string x8ebde9a98df80374)
{
    byte[] buffer2;
    RSAParameters parameters1;
    RSACryptoServiceProvider provider1;
    byte[] buffer3;
    byte[] buffer1 = Encoding.Unicode.GetBytes(x4a3f0a05c02f235f);
    SHA1 sha1;
    sha1 = new SHA1Managed();
    buffer2 = sha1.ComputeHash(buffer1);
    parameters1 = new RSAParameters();
    byte[] buffer4 = new byte[3];
    buffer4[0] = 1;
    buffer4[2] = 1;
    parameters1.Exponent = buffer4;
    parameters1.Modulus = new byte[]
    {
        // 一些字节信息，用做公钥
    };
    provider1 = new RSACryptoServiceProvider();
    provider1.ImportParameters(parameters1);
    buffer3 = Encoding.Unicode.GetBytes(x8ebde9a98df80374);
    RSAPKCS1SignatureDeformatter deformatter1 = new RSAPKCS1SignatureDeformatter(provider1);
    deformatter1.SetHashAlgorithm("SHA1");
    return deformatter1.VerifySignature(buffer2, buffer3);
}
```

在此处使用了 `RSAPKCS1SignatureDeformatter` 用于校验签名。由于该机制使用公钥验证，而在没有私钥的情况下无法反向操作。若要破解此验证，需使用爆破手段跳过校验逻辑，确保无论 `VerifySignature` 返回 `true` 还是 `false`，程序都能继续执行后续代码，从而避免因验证函数退出而导致验证失败。

## 2. 结构体分析

定义如下：

```csharp
private struct x9cebba6cbd2b2289
{
    public string xb644c3d4b9723516; // 序列号 SN
    public string x1bf37c739ee04ce8; // 公司名称 CN
    public string x63d2aa540697615b; // 版本类型 TY
    public DateTime xf98c0407c36b694d; // 过期时间 EX
    public int x01b6751913efbd51; // 用户数 UN
    public string x5d8a72c9b50ea3f1; // CPU 编码 CP
    public string x1b3806a16cc4a0b2; // 可用模块 MU
}
```

公司名称、过期时间和用户数的含义较为直观。修改这些字段可解除时间与用户数限制，因为可以根据算法生成最终的 `license`。但其他字段是否影响程序运行？是否存在其他验证点？

继续追踪代码，发现以下细节：

- **版本类型 `TY`**：仅包含三种值：
  - `R`：正式版
  - `T`：试用版
  - `D`：演示版（过期日期到达时自动切换）
  - 此时用户数限制仅为 2。
- **正式版与试用版**：目前未发现明显区别，但修改总无大碍。
- **可用模块 `MU`**：用于 `license` 类中的另一个验证函数 `public bool VerifyModule(Page p, string ModuleCode)`。该函数验证产品是否具备特定模块的使用权。这种模块化设计允许按模块销售，`license` 中包含模块信息以判断客户是否购买了对应模块。
- **CPU 编码 `CP`**：对应服务器的 `CPU ProcessorID`。若设为 `00:00:00:00:00:00`，则不做任何限制（不绑定 CPU）。若存在该值，程序会验证 `license` 中的 CPU 编码是否与服务器 CPU 编码一致，不一致则判定为未验证。建议保持默认值 `00:00:00:00:00:00`。
- **序列号 `SN`**：在代码中未见明显校验逻辑，格式未知，难以直接编写注册机。

`SN` 仅存在于 `this.x4c262adfb75e23fa.xf67f2f73d67cc6b6` 中。

`this.x4c262adfb75e23fa.xf67f2f73d67cc6b6` 除了在上述函数中被调用外，还在 `private string x579b0ad13e0e2633()` 函数中被引用。分析 `x579b0ad13e0e2633()` 可知其返回系统相关信息：

```csharp
private string x579b0ad13e0e2633()
{
    string text4;
    string text1 = this.x1083e6509fcf5d08();
    string text2 = this.x65a2df20baac0377();
    string text3 = Environment.MachineName;
    text4 = Environment.UserDomainName;
    while (true)
    {
        string text7;
        string text5 = Environment.OSVersion.ToString();
        string text6 = Environment.Version.ToString();
        text7 = "CPUSN=" + text1 + ";";
        text7 = text7 + "IP=" + text2 + ";";
        text7 = text7 + "ComputerName=" + text3 + ";";
        text7 = text7 + "DomainName=" + text4 + ";";
        text7 = text7 + "OSVer=" + text5 + ";";
        text7 = text7 + "DotNetVer=" + text6 + ";";
        text7 = text7 + "OrgName=" + this.OrgName + ";";
        object obj1 = text7;
        object[] objArray1 = new object[4];
        objArray1[0] = obj1;
        objArray1[1] = "LicNum=";
        objArray1[2] = this.LicenseNumber;
        objArray1[3] = ";";
        text7 = string.Concat(objArray1);
        text7 = text7 + "LicExpire=" + this.Expiration.ToShortDateString() + ";";
        return (text7 + "SN=" + this.x4c262adfb75e23fa.xf67f2f73d67cc6b6);
    }
}
```

该函数收集了 CPU 编码、IP、机器名、域名、操作系统版本、.NET 版本、公司名称、用户数、过期时间及 `SN`。

`x579b0ad13e0e2633()` 由 `xdea9b9654684a230` 函数调用，后者用于网络升级程序。这表明上述信息会被提交至厂商的升级服务器。

`xdea9b9654684a230` 仅在构造函数中被调用，条件为：

```csharp
License.x51ada703efb70b66.AddHours(24) < DateTime.Now
```

其中 `License.x51ada703efb70b66` 记录最近一次升级时间。因此，程序每隔 24 小时会将本机必要信息提交给升级服务器。

信息提交后，服务器端可能进行校验，以判断 `SN` 的有效性，甚至比对购买记录以识别盗版或破解版本。因此，必须屏蔽这些网络验证，即使本地验证未受影响，暴露问题也会带来隐患。

综上，破解该 OA 软件的步骤如下：

1. 去除 DLL 强签名验证，防止因签名校验失败导致程序无法运行。
2. 爆破 `license` 中的签名验证。
3. 爆破每 24 小时一次的远程网络验证。
4. 编写注册机，根据指定的公司名称、版本类型、过期时间及用户数生成 `license`。

对于序列号 `SN`、CPU 编码及可用模块等信息，通常无需修改，因为它们已被爆破屏蔽。若二次开发需识别 CPU，可在注册机中增加 CPU 编码信息（一般使用 `00:00:00:00:00:00`）。若需启用特定模块，只需将模块标识添加至可用模块信息中。

## 3. 注册机算法实现

算法逻辑较为简单，首先实现对称加密的解密函数：

```csharp
private string xcc381ffa3ede662f(string x4a3f0a05c02f235f)
{
    byte[] buffer2;
    byte[] buffer1 = new byte[] { /* 一些字节信息用作 key */ };
    buffer2 = new byte[] { /* 一些字节信息用作 key */ };
    ICryptoTransform transform1 = new DESCryptoServiceProvider().CreateDecryptor(buffer1, buffer2);
    byte[] buffer3 = Encoding.Unicode.GetBytes(x4a3f0a05c02f235f);
    byte[] buffer4 = transform1.TransformFinalBlock(buffer3, 0, buffer3.Length);
    return Encoding.Unicode.GetString(buffer4);
}
```

其对应的加密函数为：

```csharp
private string re(string text4)
{
    byte[] buffer2;
    byte[] buffer1 = new byte[] { /* 一些字节信息用作 key */ };
    buffer2 = new byte[] { /* 一些字节信息用作 key */ };
    ICryptoTransform transform1 = new DESCryptoServiceProvider().CreateEncryptor(buffer1, buffer2);
    byte[] buffer3 = Encoding.Unicode.GetBytes(text4);
    byte[] buffer4 = transform1.TransformFinalBlock(buffer3, 0, buffer3.Length);
    return Encoding.Unicode.GetString(buffer4);
}
```

接下来，按指定格式构造数据并生成最终 `license`：

```csharp
string text1 = Encoding.Unicode.GetString(Convert.FromBase64String("原始证书")); // 原始证书
string text2 = text1.Substring(0, 0x40); // 取前 64 位
string text3 = text1.Substring(0x40); // 取 64 位之后部分
string text4 = this.xcc381ffa3ede662f(text3);
string[] str = text4.Split(new char[] { ';' });
str[1] = "CN=" + this.tx_CN.Text;
str[2] = "TY=";
if (this.ra_R.Checked)
{
    str[2] += "R";
}
else if (this.ra_T.Checked)
{
    str[2] += "T";
}
else if (this.ra_D.Checked)
{
    str[2] += "D";
}
str[3] = "EX=" + Convert.ToDateTime(this.tx_EX.Text).ToString("yyyy-MM-dd");
str[4] = "UN=" + this.tx_UN.Text;
text4 = string.Join(";", str);
string text5 = this.re(text4);
```



```csharp
string text6 = Convert.ToBase64String(Encoding.Unicode.GetBytes(text2 + text5));  
this.tx_license.Text = text6;  
```

`this.tx_license.Text` 的值就是按照你设定的公司名称、版本类型、过期时间、用户数产生的新 license。

（需要说明的是，我上面对校验 `text4` 的 64 位 unicode string 就是用的原始证书的信息，也可以改成其他的信息，只要能保证其在解码时能正确在 64 位处进行分割即可，因为这段校验在后面的操作中会被爆破，所以正确与否完全没有意义，同样，`text4` 中的 `SN` 信息也是一样的，我这里都是用他原来的）

把新产生的 license 直接更新到数据库中的 license 字段即可。

当然，现在 DLL 还未爆破，这样的 license 是不正确的，使用的话，会说系统未注册。

对 DLL 的爆破，就要用到 ildasm 了，当然不是官方的，官方的解不开有混淆过的信息，这个是破解过的，网络上也有地方得到，反编译后，转存为 il 文件。

打开这个 il 文件，在其中找到：

```ilasm
.publickey = (“key 信息，很长的一段”) // @...O"+.k..\_.ol.  

.hash algorithm 0x00008004  
```

全部删除，这样就去除了这个 DLL 的强签名。

然后查找 method：

```ilasm
.method private hidebysig instance void x1052178a2aa3affc([System.Web]System.Web.HttpApplicationState xbc9689dd53140596) cil managed  
```

往下找到：

```ilasm
调用 xb12591d958637077(string, string)的地方，在其下面看到  

L_075f: brfalse.s L_0750  
```

我们可以发现这里表明，只要返回 false 就会跳到 `L_0750` 去执行，而那边就是退出校验程序。

要让其不校验这段，将 `L_0750` 改为下文中：

```ilasm
L_0761: ldloc num1  

L_0765: conv.u4  

L_0766: ldc.i4 3  

L_076b: or  

L_076c: ldc.i4.0  

L_076d: ceq  

L_076f: stloc flag1  

L_0773: ldloc flag1  

L_0777: brfalse.s L_072c  
```

正常跳转的 `L_072c` 即可，

即改为 `L_075f: brfalse.s L_072c`

这样就绕过校验，程序执行时仍旧会去校验完整性，但无论你校验结果是正确还是错误，统统跳过，直接执行下面的程序，这样就等于没有校验。

再找到：

```ilasm
.method public hidebysig specialname rtspecialname instance void .ctor([System.Web]System.Web.HttpApplicationState app) cil managed  
```

在其中找调用 `xdea9b9654684a230()` 的地方。

在他的下面有一个：

```ilasm
IL_0043: call bool [mscorlib]System.DateTime::op_LessThan(valuetype [mscorlib]System.DateTime,  
                                          valuetype [mscorlib]System.DateTime)  

IL_0048: brtrue.s IL_0023  

IL_004a: br IL_022c  
```

这段代码是用来判断时间的，符合条件即跳转，`IL_0023` 所在的地方即为调用 `xdea9b9654684a230()` 函数。

我们需要的是其不要跳转，那么修改 `IL_0023` 为 `IL_022c` 即可，这样无论时间判断的结果如何，都会继续执行下面的程序。

改动后的结果如下：

```ilasm
IL_0048: brtrue.s IL_004a  

IL_004a: br IL_022c  
```

然后是用 ilasm 工具。

执行 `ilasm /res=test.res test.il /DLL` 编译成新的 DLL 即可。

当然这时候你还可以对新产生的 DLL 加强签名，我是没有加就是了，懒-\_\_-||。

到此为止，破解完成，需要注意的是：

- 把破解后的 DLL 引入工程时，需要删除原有的引用，重新添加此 DLL 为引用后重新编译才可正常使用。
- 在更换数据库的 license 后，需要把进程中的 `w3p.exe` 中断再运行页面，才会识别出新的 license 信息。

因为该 OA 软件使用的时 application 状态来存储 license 信息，在 `w3p` 不中断的情况下，并不会重新编译页面，也就不会重新取这些信息，而导致 license 信息不更新。

4dd466d0e1e0cc123afbe504da2462ca
