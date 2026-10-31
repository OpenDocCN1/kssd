
# [原创]review assistant破解心得

review assistant是一个能集成在visual studio的代码评审工具，目前最新版本是1.2.30，网址为：

d67K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3c8W2N6X3q4J5N6q4)9J5k6h3y4G2L8g2)9J5c8Y4u0W2N6X3W2W2N6#2)9J5k6r3q4K6M7$3W2K6N6r3q4F1N6q4)9J5c8R3`.`.

花了两天分析破解，基本破解成功，将这两天的破解思路汇报一下。

1. 通过查看review assistant的目录结构和文件命名规则，判断它为C#程序，决定选用`Reflector`和`reflexil`进行破解；

2. 初步猜测其会在`Devart.ReviewAssistant.Common.dll`实现注册、加密等逻辑，于是通过`Reflector`打开查看，发现一个命名空间`Devart.ReviewAssistant.Common.License`，其中包含`LicenseInfo`类，分析`LicenseInfo`类，发现其中包含它的各种注册等代码；

3. `LicenseInfo`有一个方法`DaysBeforeExpired`，内容如下：

   ```
   public int DaysBeforeExpired
   {
       get
       {
           TimeSpan span = (TimeSpan) (this.EndDate.Date - DateTime.Now.Date);
           return (span.Days + 1);
       }
   }
   ```

   以为破解会非常简单，只要将`+1`改为`+1000`便可完成破解，但改完后发现没有效果。

4. 猜测别的程序或动态库会使用`LicenseInfo`，于是将所有程序载入`Reflector`，按`F3`搜索`LicenseInfo`，发现`ReviewAssistantServer.exe`有一个类`#=q9VXlN7R77KJQsIcXNDOyoQZ5uVA_SQytMl0gr9McnSnn4v$Kwf$S1rtOrCtwbGbTM5JUujvNgx2UmXGkSjsIKQ==`（已被混淆）创建了`LicenseInfo`，如下所示：

   ```
   public OperationResult GetLicenseInformation(LoginInfo #=qZG2sgtPtY$hvFevuYpDgzQ==, out byte[][] #=qM970WgKWmT1RKDhOAzW4BA==)
   {
       DateTime now = DateTime.Now;
       #=qM970WgKWmT1RKDhOAzW4BA== = null;
       List<byte[]> list = new List<byte[]>();
       try
       {
           foreach (LicenseInfo info in #=qzJZguenem3lycvMKsZrLYMlx5ni1fzNqKrhQfvGoGXow9pKX3W9ndyn3WLIUAMiX.#=q_egRSvuus951dsxJkBD1ng==().#=qDfpT$TbjCumxO6pwEVpvBg==())
           {
               info.Signature = null;
               list.Add(info.ToBytes());
           }
       }
       catch (Exception exception)
       {
           return this.#=qNvV_s1I$7V9ukfZAO0fQtP0eEgV966_EzLkM$xWQFug=(null, #=q$BFJDxTd1oZkxfbLBg3DYMJIdM1MO0Sa0t8Q8Z$3KdE=.#=qgfjJC0lJg6Mz314CRmsMEw==(-277034973), exception, now);
       }
       #=qM970WgKWmT1RKDhOAzW4BA== = list.ToArray();
       return new OperationResult(OperationResultStatus.Ok, now);
   }
   ```

   可以看出，类`#=qzJZguenem3lycvMKsZrLYMlx5ni1fzNqKrhQfvGoGXow9pKX3W9ndyn3WLIUAMiX`需要重点关注；

5. 分析查看类`#=qzJZguenem3lycvMKsZrLYMlx5ni1fzNqKrhQfvGoGXow9pKX3W9ndyn3WLIUAMiX`，发现其确实与注册有关；

6. 类`#=qzJZguenem3lycvMKsZrLYMlx5ni1fzNqKrhQfvGoGXow9pKX3W9ndyn3WLIUAMiX`初始化时，创建了两个License，如下所示：

   ```
   static #=qzJZguenem3lycvMKsZrLYMlx5ni1fzNqKrhQfvGoGXow9pKX3W9ndyn3WLIUAMiX()
   {
       LicenseInfo info = new LicenseInfo {
           UserName = #=qKcATx1QrcSRw846XymY4aA==(),
           Company = #=qgxYhyF6W0XXcoV7jXR53gvHFwt$klSOUQTkuEzLby8w=(),
           Type = LicenseType.Trial,
           UserCount = 0x7fffffff,
           EndDate = DateTime.Now.Date.AddDays((double) (#=q2cBrNNFo14ZAXwSnSHPPQHRh2V1kmI0H21MoTOkrIAXJ4rhkj78wx4SNJb0Y4Unh.#=qgds4YRA1s7lnD2tqMclPwA==() - 1))
       };
       #=qCnV2kgLhXvPXd38e$TkKPg== = info;
       LicenseInfo info2 = new LicenseInfo {
           UserName = #=qKcATx1QrcSRw846XymY4aA==(),
           Company = #=qgxYhyF6W0XXcoV7jXR53gvHFwt$klSOUQTkuEzLby8w=(),
           Type = LicenseType.Free,
           UserCount = 3,
           EndDate = DateTime.MaxValue
       };
       #=q76MDXyntEgMQZR6lSfq8PAMGGRM0iOx6gtw68Z7sxQM= = info2;
   }
   ```

7. 试着将第一个License的超期日期由`-1`改为`+1000`，修改之后能够工作，各种功能正常，但5-10分钟后系统退出，不再可用；

8. 猜测系统在启动后，会`Sleep`5-10分钟后进行各种检查，于是搜索`Sleep`，发现多处有`Sleep`5-10分钟的代码，如下所示：

   ```
   Random random = new Random();
   Thread.Sleep(random.Next(0x493e0, 0x927c0));
   ......
   ```

   其中`0x493e0`十进制为300000，`0x927c0`十进制为600000。

9. 系统初始能够正常运行，过了5-10分钟之后便不能运行，应该是`Sleep`5-10分钟之后开始干坏事，于是搜索所有的`0x493e0`和`0x927c0`，如果其与`Sleep`配对的地方直接短路返回，不让它干坏事，如下所示：

   ```
   Random random = new Random();
   Thread.Sleep(random.Next(0x493e0, 0x927c0));
   ret
   ```

10. 这样修改之后，系统能正常工作。

11. 其它心得要点：

    A、`Devart.ReviewAssistant.Common.dll`确实为其包含了注册相关功能，它检查系统中是否有License文件并读出，刚开始我曾经想利用它直接产生注册文件，但它的注册文件采用RSA加密且包含签名信息，在`ReviewAssistantServer.exe`中能找到它的公钥，但无法得到私钥，此方法行不通；

    B、`ReviewAssistantServer.exe`中采用了签名检查，能检查`ReviewAssistantServer.exe`是否被修改，如果被修改，则5至10分钟之后开始干坏事；

    C、review assistant采用了Eaz混淆工具的最新版本，`de4dot`对它不能完成反混淆；

    D、`ReviewAssistantServer.exe`随机插入了5个防破解处理类，这些类中在后台启动了很多线程。

总结：

相对于.NET平台下的代码很容易被破解。

欢迎各位朋友一起交流心得，我的邮箱：linkman2002#sina.com QQ:2998143609

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

[#调试逆向](https://bbs.kanxue.com/forum-4-1-1.htm)
