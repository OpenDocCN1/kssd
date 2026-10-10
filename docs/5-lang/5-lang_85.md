

# 破解\*\*\*\*\*\*的 license 验证原理

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-784.htm`](https://www.pediy.com/kssd/pediy07/pediy7-784.htm)

反编译工具：.NET Reflector  
  
前言：  
.NET 的 IL 给我们反编译提供了巨大方便，Reflector 更是.NET 领域的最伟大的发明，反编译出来的程序几乎和源文件一摸一样，任何初学者只需要稍微懂得 C#语言就可以分析注册码算法。因此，.NET 几乎是没有秘密的，.NET 的软件作者，如果想保护自己的成果的话，记住用混淆器啊:D  
  
本文主要面向初学者（我也是其中一个，呵呵），注重演示分析类代码的过程，比较适合新手练手之用。  
本文只作科研和学习用途。  
  
过程：  
首先找找有那些可疑文件，分别用.NET Reflector 打开看看。最后找到 bin/Web\*\*\*\*\*.dll，发 Lealsoft.\*\*\*\*\*.Common 里面有几个出现 License 字眼的类名，于是我们就仔细往里面看看  
  
public class \*\*\*\*\*\*\*\*License : ServerLicense  
{  
      // Methods  
      public \*\*\*\*\*\*\*\*License(Type type, string key, long startTicks, long ticks, string username, long userIndex, long clientCount);  
  
      // Properties  
      public long ClientCount { get; }  
      public bool IsExpired { get; }  
      public long StartTicks { get; }  
      public long Ticks { get; }  
      public long UserIndex { get; }  
      public string Username { get; }  
  
      // Fields  
      private long \_clientCount;  
      private long \_startTicks;  
      private long \_ticks;  
      private long \_userIndex;  
      private string \_username;  
}  
  
看来只是一个用于保存资料的类，我们需要的 License 认证算法不在里面，不过我们由此也知道一个 License 包含有哪些信息。  
  
public class ServerLicenseProvider : LicenseProvider  
{  
      // Methods  
      static ServerLicenseProvider();  
      public ServerLicenseProvider();  
      protected virtual ServerLicense CreateEmptyLicense(Type type);  
      protected virtual ServerLicense CreateLicense(Type type, string key);  
      public override License GetLicense(LicenseContext context, Type type, object instance, bool allowExceptions);  
      protected virtual string GetLicenseData(Type type);  
      protected virtual Stream GetLicenseDataStream(Type type);  
      protected virtual bool ValidateLicense(ServerLicense license, out string errorMessage);  
      protected virtual bool ValidateLicenseData(Type type, string licenseData);  
  
      // Fields  
      private static readonly ServerLicenseCollector LicenseCollector;  
  
      // Nested Types  
      private sealed class ServerLicenseCollector  
      {  
            // Methods  
            public ServerLicenseCollector();  
            public void AddLicense(Type objectType, ServerLicense license);  
            public ServerLicense GetLicense(Type objectType);  
            public void RemoveLicense(Type objectType);  
  
            // Fields  
            private IDictionary \_collectedLicenses;  
      }  
}  
  
看起来有点像了，于是我们找 ValidateLicense 的实现来看看  
protected virtual bool ValidateLicense(ServerLicense license, out string errorMessage)  
{  
      errorMessage = null;  
      \*\*\*\*\*\*\*\*License license1 = (\*\*\*\*\*\*\*\*License) license;  
      if (license1.IsExpired)  
      {  
            errorMessage = "The License has expired.";  
            return false;  
      }  
      return true;  
}  
  
很失望，居然只是按照 IsExpired 字段来判断，跟具体算法依然无关。那 IsExpired 是怎么判断的呢？我们找\*\*\*\*\*\*\*\*License.IsExpired 属性：  
public bool IsExpired  
{  
      get  
      {  
            if (DateTime.Today.Ticks <= this.\_ticks)  
            {  
                  return (DateTime.Now.Ticks < this.\_startTicks);  
            }  
            return true;  
      }  
}  
   
原来是看当前时间的 Ticks 是否在 License 指定的时间范围内。ok，我们继续看另外一个 Validate 的函数  
  
protected virtual bool ValidateLicenseData(Type type, string licenseData)  
{  
      bool flag1 = false;  
      char\[\] chArray1 = new char\[1\] { ':' } ;  
      string\[\] textArray1 = licenseData.Split(chArray1);  
      if (textArray1.Length == 6)  
      {  
            return (string.Compare("\*\*\*\*\*\*\*\*1Licensed", textArray1\[0\], true, CultureInfo.InvariantCulture) == 0);  
      }  
      return flag1;  
}  
  
看起来是验证 licenseData 的格式，它是用":"分隔各字段的，并且第一个字段需要为\*\*\*\*\*\*\*\*1Licensed  
  
protected virtual Stream GetLicenseDataStream(Type type)  
{  
      string text3 = type.Assembly.GetName().Name;  
      type.Assembly.GetName().Version.ToString();  
      string text1 = "~/licenses/\*\*\*\*\*\*\*\*.lic";  
      string text2 = null;  
      try  
      {  
            text2 = HttpContext.Current.Server.MapPath(text1);  
            if (!File.Exists(text2))  
            {  
                  text2 = null;  
            }  
      }  
      catch  
      {  
      }  
      if (text2 != null)  
      {  
            return new FileStream(text2, FileMode.Open, FileAccess.Read, FileShare.Read);  
      }  
      return null;  
}  
  
恩，看到没有，里面找到 license 的保存位置了。我们生成的 license 也应该放在这里  
  
protected virtual ServerLicense CreateLicense(Type type, string key)  
{  
      char\[\] chArray1 = new char\[1\] { ':' } ;  
      string\[\] textArray1 = key.Split(chArray1);  
      return new \*\*\*\*\*\*\*\*License(type, key, long.Parse(textArray1\[1\], CultureInfo.InvariantCulture), long.Parse(textArray1\[2\], CultureInfo.InvariantCulture), textArray1\[3\], long.Parse(textArray1\[4\], CultureInfo.InvariantCulture), long.Parse(textArray1\[5\], CultureInfo.InvariantCulture));  
}  
   
从这里可以看到 license 的保存格式，原来合法的 license 只是简单用":"把几个 license 参数分隔开。对照\*\*\*\*\*\*\*\*License 的构造函数，我们就明白各字段的含义了。也可以构造出一个伪 license。我随便构造一个，  
string data = "\*\*\*\*\*\*\*\*1Licensed:632454807859509835:735608407859509835:henry:0:10000";  
注意一下那两个 ticks 字段，需要把当前的 ticks 时间包含起来才行，否则 IsExpired 会判断为 true 的  
  
我们还没有找到验证的核心代码，继续找……什么？找完了 ServerLicenseProvider 了？  
看来我们要的东西不在这里，那会在哪里呢？  
看看 Lealsoft.\*\*\*\*\*\*\*\*.Common 命名空间，似乎还有个 EncryptedLicenseProvider 的类，向里面看看  
  
protected override Stream GetLicenseDataStream(Type type)  
{  
      Stream stream1 = base.GetLicenseDataStream(type);  
      if (stream1 == null)  
      {  
            return null;  
      }  
      DESCryptoServiceProvider provider1 = new DESCryptoServiceProvider();  
      provider1.Key = EncryptedLicenseProvider.encryptionKeyBytes;  
      provider1.IV = EncryptedLicenseProvider.encryptionKeyBytes;  
      ICryptoTransform transform1 = provider1.CreateDecryptor();  
      return new CryptoStream(stream1, transform1, CryptoStreamMode.Read);  
}  
  
像是算法了吧。没错了，这就是解密算法了，这么短？确实是啊，你没有看到它调用了 base.GetLicenseDataStream 吗？而它的基类就是我们刚刚分析过的 ServerLicenseProvider。  
好，我们下一步搞清楚 Key 和 IV 的值。去看看构造函数：  
  
static EncryptedLicenseProvider()  
{  
      EncryptedLicenseProvider.encryptionKeyBytes = new byte\[8\] { 0x35, 70, 0x42, 50, 0x38, 0x31, 70, 0x36 } ;  
}  
  
yeah，private key 也找到了，于是我们就可以实现 keygen 了（其实是 license generator）。就是实现 DES 加密而已。不熟.NET 框架？查 MSDN 去吧:D  
  
\=============== keygen ===============  
      byte\[\] encryptionBytes = new byte\[8\] { 0x35, 70, 0x42, 50, 0x38, 0x31, 70, 0x36 } ;  
      DESCryptoServiceProvider provider = new DESCryptoServiceProvider();  
      provider.Key = encryptionBytes;  
      provider.IV = encryptionBytes;  
      ICryptoTransform transform1 = provider.CreateEncryptor();  
  
      string data = "\*\*\*\*\*\*\*\*1Licensed:632454807859509835:735608407859509835:henry:0:10000";  
      //Console.WriteLine(System.DateTime.Now.Ticks);  //看看现在的 ticks  
      byte\[\] message = Encoding.UTF8.GetBytes(data);  
      FileStream fs = new FileStream("c:\\\\\*\*\*\*\*\*\*\*.lic", FileMode.Create,  
        FileAccess.Write, FileShare.Write);  
      CryptoStream cs = new CryptoStream(fs, transform1, CryptoStreamMode.Write);  
      cs.Write(message, 0, message.Length);  
      cs.Flush();  
      cs.Close();  
      fs.Close();  
\=============================================== 

 4dd466d0e1e0cc123afbe504da2462ca