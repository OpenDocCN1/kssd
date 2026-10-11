
# 破解 WebUI.NET Framework 验证原理

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-363.htm`](https://www.pediy.com/kssd/pediy07/pediy7-363.htm)

【软件名称】：`WebGrid.NET` `Version` `4.0`

【软件类别】：国外软件

【整理时间】：`2005-08-17`

【下载地址】：`http://www.evget.com/view/viewProductInfo.asp?productId=89&tabIndex=0`

【保护方式】：`License`

【保护软件】：`XenoCode`

【编译语言】：`ASP.NET`

【调试环境】：`WinXP`、`.NET` `Framework` `1.1`、`Reflector`、`ildasm`、`sn`、`ultraedit`

【破解日期】：`2005-08-17`

【破解目的】：研究 `.NET` 程序的破解流程

【作者声明】：初学 `Crack`，只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教！

听说 `WebGrid.NET` `4.0` 采用了另外的加密技术，于是下载下来想见识一下，结果非常失望。除了把验证部分转移到了 `Framework` 的通用框架内以外，没有发现有什么改进的，加密水平还后退回 `WebCombo.NET` 的水平去了。

先是利用流程反混淆技术，把一堆函数反混淆出来，过程没有什么新意，参看以前我写的《揭开 `.NET` 程序保护的秘密》和《破解 `WebCombo.NET` `Version` `2.0`》，用 `ildasm` 反编译成 `il` 后修改转跳流程，然后用 `ilasm` 重新编译回去即可，唯一遇到一点点麻烦的是 `ilasm` 居然不认那个混淆后的 `resource`。这也没关系，我们只是要分析验证原理，暂时不把 `resource` 编进去就是了。虽然是用 `XenoCode` 混淆的，可是连变量名字都没有处理过，非常容易就找到这两个函数

```csharp
<ISNet.WebUI.ISNetControl>

private void CheckLicense()
{
    string text2 = "";
    string text3 = "";
    RegistryKey key3 = this.GetIntersoftRegKey(); // HKLM\Software\Intersoft Solutions
    text2 = this.Product.ProductName; // 具体的软件产品会重载此 field
    text3 = this.Product.VersionNumber; // 具体的软件产品会重载此 field
    if (key3 != null)
    {
        RegistryKey key1 = key3.OpenSubKey(text2); // 从注册表读键值
        if (key1 != null)
        {
            RegistryKey key2 = key1.OpenSubKey(text3);
            if (key2 != null)
            {
                string text1 = key2.GetValue("SID") as string; // 注册码
                if ((text1 != "") && (text1 != null))
                {
                    if (this.MatchSN(this.Product.dk, text1)) // 关键，如果爆破只要 nop 掉这里即可
                    {
                        this.LicenseType = LicType.Full;
                    }
                    else if (this.MatchSN(this.Product.dr, text1))
                    {
                        this.LicenseType = LicType.RuntimeOnly;
                    }
                    else
                    {
                        text1 = this.ReadWebConfigValue("RuntimeLicenseKey");
                        if ((text1 != null) && this.MatchSN(this.Product.dr, text1))
                        {
                            this.LicenseType = LicType.RuntimeOnly;
                        }
                    }
                }
            }
        }
    }
}

private bool MatchSN(byte[] ?, string ?) // 比较序列号的核心 call
{
    byte[] buffer1 = Encoding.ASCII.GetBytes(?);
    bool flag1 = false;
    bool flag2 = false;
    bool flag3 = false;
    int num1 = 0;
    byte[] buffer2 = ?;
    int num4 = 0;

Label_001B:
    if (num4 < buffer2.Length) // 实际是循环：for(num4=0;num4<buffer2.Length;num4++)
    {
        byte num2 = buffer2[num4]; // 读入 num2 和 num3 进行比较
        byte num3 = 0;
        try
        {
            num3 = (byte) buffer1.GetValue(num1);
        }
        catch
        {
            flag3 = true;
            goto Label_014B;
        }
        if (num2 != 0xbc) // 下面一大段是核心判断，后面解释
        {
            if (num2 == 190)
            {
                if ((num3 >= 0x41) && (num3 <= 0x4b))
                {
                    goto Label_0054;
                }
                flag3 = true;
            }
            else
            {
                if (num2 == 0xb8)
                {
                    flag2 = true;
                    goto Label_005A;
                }
                if (num2 != 0x4a)
                {
                    if (num2 != 0x7e)
                    {
                        if (num2 != 90)
                        {
                            if (num2 == 0x42)
                            {
                                if ((num3 >= 0x30) && (num3 <= 0x35))
                                {
                                    goto Label_0054;
                                }
                                flag3 = true;
                            }
                            else
                            {
                                if (num2 != 0x80)
                                {
                                    if (!flag2)
                                    {
                                        flag3 = true;
                                        goto Label_014B;
                                    }
                                    if (num3 != (num2 / 2))
                                    {
                                        flag3 = true;
                                        goto Label_014B;
                                    }
                                    flag2 = false;
                                    goto Label_0054;
                                }
                                if ((num3 >= 0x36) && (num3 <= 0x39))
                                {
                                    goto Label_0054;
                                }
                                flag3 = true;
                            }
                        }
                        else
                        {
                            if (num3 == (num2 / 2))
                            {
                                goto Label_0054;
                            }
                            flag3 = true;
                        }
                    }
                    else
                    {
                        if ((num3 >= 0x41) && (num3 <= 90))
                        {
                            goto Label_0054;
                        }
                        flag3 = true;
                    }
                }
                else
                {
                    if ((num3 >= 0x30) && (num3 <= 0x39))
                    {
                        goto Label_0054;
                    }
                    flag3 = true;
                }
            }
        }
        else
        {
            if ((num3 >= 0x4c) && (num3 <= 90))
            {
                goto Label_0054;
            }
            flag3 = true;
        }
    }
    goto Label_014B;

Label_0054:
    num1++;

Label_005A: // 实际是循环：for(num4=0;num4<buffer2.Length;num4++)
    num4++;
```



```
     goto Label_001B;  

Label_014B:  

     if (!flag3)  

     {  
         flag1 = true;  

     }  
     return flag1;  

}  
```

看起来有点眼花，实际是因为 `Reflector` 对 switch 语句的支持不太好造成的。按照里面的逻辑列一张表就清楚了：

| num2 | num3 |
| :--- | :--- |
| 190 | \[0x41,0x4b\] |
| 0xbc | \[0x4c,90\] |
| 0xb8 | `flag2` = `true`, 当前 `num3` 不变（`num2` 在这里的作用是一个额外的转义字符） |
| 0x42 | \[0x30,0x35\] |
| 90 | 45 |
| 0x7e | \[0x41,90\] |
| 0x4a | \[0x30,0x39\] |
| 0x80 | \[0x36,0x39\] |
| default | `flag2` == `true` && `num3` == `num2` / 2, `flag2` = `false` |

再看看 `Product.dk` 是在哪里初始化的，用 `Callee graph` 查到 `WebGrid.NET` 当中

```
<ISNet.WebUI.WebGrid.WebGrid>  

.method family hidebysig specialname virtual instance [ISNet.WebUI]ISNet.WebUI.ProductInfo get_Product() cil managed  

{  
     // Code Size: 292 byte(s)  

…  

     L_006b: ldc.i4.s 16  

     L_006d: newarr unsigned int8  

     L_0072: dup  

     L_0073: ldtoken ?/? ?::?  

     L_0078: call void [mscorlib]System.Runtime.CompilerServices.RuntimeHelpers::InitializeArray([mscorlib]System.Array, [mscorlib]System.RuntimeFieldHandle)  

     L_007d: stfld unsigned int8[] [ISNet.WebUI]ISNet.WebUI.ProductInfo::dk  

}  
```

找那串乱码的定义

```
.field assembly static ?/? ? = ((BC 42 4A 4A 42 5A 42 BC BE 7E 5A 7E BC BE BE BC))  
```

对照上面的表，就可以手动构造一个 `sn` 了。如 `0x4c`, `0x30`, `0x30`, `0x30`, `0x30`, `45`, `0x30`, `0x4c`, `0x41`, `0x41`, `45`, `0x41`, `0x4c`, `0x41`, `0x41`, `0x4c`，即 `N0000-0NAA-ANAAN`。`Keygen` 也不难写，甚至可以写一个 `WebUI Framework` 产品万能注册机，直接从 `exe` 里面读取这串东西，呵呵。这里就省略了。

想不明白 `WebUI Framework` 的注册码验证算法为什么不再用 `Remotesoft Protector` 了。莫非出现版权问题？不过这对偶等穷人是好事。

`4dd466d0e1e0cc123afbe504da2462ca`
