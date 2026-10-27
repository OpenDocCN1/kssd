
# [原创]对某.net OA软件的破解记录

特别说明：本文仅做学习研究使用，有一定参考价值，是一些破解的心得和思路，完全是个人对程序的研究，因此也未说明破解的是哪个软件，不涉及商业行为，版权问题是另外一方面的事情了，本人对其他事务不负责。

公司最近买了某公司的一个.net OA软件进行二次开发，但由于其的许可证限制了同时可连接的用户数以及可使用的时间（截止时间到后变为演示版，演示版仅能2个用户），我们的客户一般比较多，每个人一个账号，在测试和二次开发方面不太方便，于是尝试自己破解一下。

先用Reflector查看了一下其DLL文件，发现使用了XenoCode进行混淆，但程度不是太深，基本上还不影响阅读。

根据观察，发现其所有页面都基于一个Page基类WebPage，其中有一个关键函数`Loginon()`，用于系统登录，其中判断能否登录，是根据一个`login_result`的结果来判断的。

`Login_result`是一个enum：

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

这是登录状态，那么肯定有返回这些状态的函数，通过查找，找到类`SecurityMgr`中包含有一个函数：

```csharp
private SecurityMgr.LOGON_RESULT x5913b537a9ca4090(string x6b237db413aed7db, string xb45a741896260675, Page x9c79b5ad7b769b12)
```

用来返回登录状态。

找到这个函数，反编译，查看代码，代码中的字符串都已经被XenoCode加密过，直接是看不出来的，好在我们可以利用反射来解密这些字符串的值（这些网上都有说明）。

转换后，我们可以看到，其对用户数的判断是在对数据库中的用户表进行top筛选来限制的：`select top "+license1.LicenseNumber+ "username from users`。其中`license1.LicenseNumber`即是限制用户数的数目。

这个从哪里来呢？往上看，可以看到：

```csharp
License license1 = new License(x9c79b5ad7b769b12.Application);
```

这样，我们就清楚了，所有的license信息是在此处读取的。

找到这个类，发现`license1.LicenseNumber`这个属性对应的是`x957eec0002850847.x01b6751913efbd51`：

```csharp
public int LicenseNumber
{
    get
    {
        return this.x957eec0002850847.x01b6751913efbd51;
    }
}
```

那么`x957eec0002850847.x01b6751913efbd51`元素在哪里有被使用过呢？

通过Reflector查看Analyzer功能，我们可以发现，其在构造函数`public License(HttpApplicationState app)`和`private void x1052178a2aa3affc(HttpApplicationState xbc9689dd53140596)`中被使用。

对其进行分析，对字符串解码后，再去除花指令，对其混淆过的流程进行重新编排，我们可以看到构造函数中对这个属性的操作仅仅是初始化，那么赋值的地方在哪里呢？

往下看我们可以看到，当`app["License"]`为null时，会去调用：

```csharp
this.x822eb47bc415e83b(app);
this.x1052178a2aa3affc(app);
```

用于取得license信息。

很容易的，我们知道`x957eec0002850847.x01b6751913efbd51`这个属性一定在`this.x1052178a2aa3affc(app)`函数中被改写以取到相应的用户限制数。

来到该函数中逆向依次搜索`x01b6751913efbd51》text7》text5》text4`，得到这些关键语句：

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

知道，这个属性是从`text4`中来的，而`text4`是由`this.x4c262adfb75e23fa.x3f7a1096840d5b39`经过一系列转换而来。

而`x4c262adfb75e23fa.x3f7a1096840d5b39`的值我们通过分析再结合前文，可以很自然的想到是从`x822eb47bc415e83b(app)`函数运行后得到。`x822eb47bc415e83b(app)`比较简单，很容易就知道其作用是`ReadLicenseFromDB`。

从中我们找到对应关系，License下的一个包含4个元素的结构，用于记录从数据库中取到的信息，对比下程序，我们很容易找出其对应关系：

```csharp
private struct x29e2d415f271a930
{
    public string xf67f2f73d67cc6b6; //Key
    public string x3f7a1096840d5b39; //License
    public string x00af6e04034127cd; //Data
    public DateTime xd00ef6aee32c8aff; //CreateTime
}
```

到此，我们就已经知道，用户数信息是根据数据库中的license字段信息经过一系列换算得到。

那么我们结合`private void x1052178a2aa3affc(HttpApplicationState xbc9689dd53140596)`函数分析：

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
}
```
