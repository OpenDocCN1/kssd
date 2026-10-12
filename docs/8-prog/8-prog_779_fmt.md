
# 一个键位映射程序,r3 控制+r0 驱动,算是研究了 10 来天驱动的成果吧

> 原文：[`www.pediy.com/kssd/pediy11/108494.html`](https://www.pediy.com/kssd/pediy11/108494.html)

这是基于那个很有名的 `CTRL2CAP` 和寒江独钓里面的例子写出来的。

添加了驱动的 `MajorFunction[IRP_MJ_DEVICE_CONTROL]` 和改进了一下原来的 `unload` 函数,最后不用按个键也可以卸载。

映射是用传统的挂接过滤设备的方式来实现的。

`ring3` 部分用 C# 写的,调用 `win32api`,建立服务,启动,`DeviceIoControl` 通信,关闭等等。

将记录下要映射的源键和目标键的扫描码发到驱动。

首先要把服务安装和启动,基本的流程是：

`OpenSCManager` >> `CreateService` or `OpenService` >> `StartService`,到这里就启动好了。

然后要 `DeviceIoControl`,基本流程~

`CreateFile` ~ `DeviceIoControl` 这样就可以通信 `r0` 了。

但是 `CreateFile` 可能会遇到一些权限问题,因为是自己写的驱动,可以在 `CreateFile` 第二个参数写 `0`,不需要要任何权限...反正没有权限也可以 `DeviceIoControl`。

最后记得要关掉从 `CreateFile` 拿来的句柄,不然没办法卸载和接受新的 `IO` 控制。

于是于是,

经过无数次 `BOSD` 后她们终于跟操作系统和睦相处了![](img/6821a6175013046370981b6035d43ab4.jpg "使眼色")

在 32 位 XP 和 win7 测试通过~

目前可以支持大概 4 块键盘(过段时间更新一下应该可以去掉这个限制),和最多映射 9 对键,当然这只是为了 `r3` 程序的布局好看而已..你可以随便添加的/

`ring3` `ring0` 源码和 binary 都在附件。

在这里估计 C# 很小众吧..而且我是用 VS2010 建的 C# 项目(更加没人用!)...

把 C# 关键代码段贴出来,源码不下也罢-w-

```csharp
/********************************************************************************
 * 模块描述:KCC 安装驱动和 DeviceIoControl 通信
 * 作者:月
 ********************************************************************************/
//下面开始使用 Service 系列 API 安装驱动
//首先需要 OpenSCManager 拿到 hSCManager
IntPtr hSCManager = OpenSCManager(null, null, 0xF003F); //SC_MANAGER_ALL_ACCESS (0xF003F)
if (hSCManager == IntPtr.Zero)
{
    MessageBox.Show("OpenSCManager 失败,Error Code:" + Marshal.GetLastWin32Error().ToString());
    return false;
}
//建立一个服务
IntPtr hService;
RegistryKey serviceKey = Registry.LocalMachine;
try
{
    //在 XP 和 win7 下 OpenSubKey 对指定键值不存在的反应会不一样,XP 会异常而 win7 不会,故当键值不存在时手动抛出一个异常
    //如果键值不存说明服务没有创建,进 catch 块建立服务
    if (serviceKey.OpenSubKey("SYSTEM", true).OpenSubKey("CurrentControlSet", true).OpenSubKey("services", true).OpenSubKey("kbdFilter", true) == null)
    {
        throw new Exception();
    }
}
catch
{
    hService = CreateService(
        hSCManager,
        "kbdFilter",
        "kbdFilter",
        0x0002,
        0x00000001,
        0x00000003,
        0x00000001,
        System.AppDomain.CurrentDomain.BaseDirectory + "kbdFilter.sys",        // path to service's binary
        null,                     // no load ordering group
        0,                    // no tag identifier
        null,                     // no dependencies
        null,                     // LocalSystem account
        null);                 // no password
    if (hService == IntPtr.Zero && Marshal.GetLastWin32Error().ToString() != "1073")
    {
        MessageBox.Show("CreateService 失败,Error Code:" + Marshal.GetLastWin32Error().ToString());
        return false;
    }
}
//打开服务
hService = OpenService(
    hSCManager,       // SCM database
    "kbdFilter",          // name of service
    0xF003F);   // full access
if (hService == IntPtr.Zero)
{
    MessageBox.Show("OpenService 失败,Error Code:" + Marshal.GetLastWin32Error().ToString());
    return false;
}
//为了应对程序被移动路径的情况
//判断注册表中本服务的 ImagePath 字段是否和当前 KCC 程序路径相符,不相符则用用当前路径重写
//如 c:\aaa.sys 这样的路径让 CreateService 在注册表注册服务,实际在注册表中保存的 ImagePath 是\\??\\c:\aaa.sys 故和当前路径比对时,当前路径之前要加上\\??\\
if (serviceKey.OpenSubKey("SYSTEM").OpenSubKey("CurrentControlSet").OpenSubKey("services").OpenSubKey("kbdFilter").GetValue("ImagePath").ToString() != "\\\\??\\\\" + System.AppDomain.CurrentDomain.BaseDirectory + "kbdFilter.sys")
{
    serviceKey.OpenSubKey("SYSTEM", true).OpenSubKey("CurrentControlSet", true).OpenSubKey("services", true).OpenSubKey("kbdFilter", true).SetValue("ImagePath", "\\\\??\\\\" + System.AppDomain.CurrentDomain.BaseDirectory + "kbdFilter.sys", RegistryValueKind.ExpandString);
}
//启动服务,到此完成驱动的加载
if (!StartService(hService, 0, string.Empty))
{
    MessageBox.Show("StartService 失败,Error Code:" + Marshal.GetLastWin32Error().ToString());
    return false;
}
//下面开始 Ring3 和 Ring0 通信的部分
for (int i = 0; ; i++)
{
    //在驱动中,建立的过滤设备名字是 kbdFilter0~kbdFilter1~....~kbdFilter99~kbdFilterX,这样循环增加的形式
    //这里使用"\\\\.\kbdFilter1""\\\\.\kbdFilter2"...."\\\\.\kbdFilter99"去尝试 CreateFile 取得设备句柄
    //失败说明已经取得了所有过滤设备的句柄,退出循环
    string kbdFilterName = "\\\\.\\kbdFilter" + i.ToString();
    System.IntPtr hDevice = CreateFile(kbdFilterName,
                                        0,  //不需要权限,否则会 getlasterror:5
                                        0,
                                        IntPtr.Zero,
                                        3,
                                        4,
                                        IntPtr.Zero
                                        );
    if (hDevice.ToInt32() == -1)
    {
        break;
    }
    int Len = 0;
    IntPtr Buffer = Marshal.AllocHGlobal(union.Length * Marshal.SizeOf(union[0]));
    Marshal.Copy(union, 0, Buffer, union.Length);
    int ret = DeviceIoControl(hDevice,
          0xa01,
          Buffer,
          union.Length * Marshal.SizeOf(union[0]),
          IntPtr.Zero,
          0,
          ref Len,
          IntPtr.Zero);
    Marshal.FreeHGlobal(Buffer);
    if (ret == 0 && Marshal.GetLastWin32Error().ToString() != "1")
    {
        // Free the unmanaged memory.
        MessageBox.Show("DeviceIoControl 错误,Error Code:" + Marshal.GetLastWin32Error().ToString());
        return false;
    }
    //最后句柄是不能忘记关的
    CloseHandle(hDevice);
}

/********************************************************************************
 * 模块描述:KCC 卸载驱动
 * 作者:月
 ********************************************************************************/
//下面开始使用 Service 系列 API 卸载驱动
//首先需要 OpenSCManager 拿到 hSCManager
IntPtr hSCManager = OpenSCManager(null, null, 0xF003F); //SC_MANAGER_ALL_ACCESS (0xF003F)
if (hSCManager == IntPtr.Zero)
{
    MessageBox.Show("OpenSCManager 失败,Error Code:" + Marshal.GetLastWin32Error().ToString());
    return false;
}
//拿到服务句柄
IntPtr hService = OpenService(
    hSCManager,       // SCM database
    "kbdFilter",          // name of service
    0xF003F);   // full access
if (hService == IntPtr.Zero)
{
    MessageBox.Show("OpenService 失败,Error Code:" + Marshal.GetLastWin32Error().ToString());
}



```
        return false;

    }

    SERVICE_STATUS service_starus = new SERVICE_STATUS();

    //停止设备
    if (!ControlService(hService, 0x00000001, ref service_starus))
    {
        MessageBox.Show("ControlService(停止服务)失败,Error Code:" + Marshal.GetLastWin32Error().ToString());

        return false;

    }

    return true;

****************************************************************/
```

### 上传的附件

| ![](img/0623e01110d741fd20f66acbc8ec6f9d.jpg) || [\[R0source\]kbdfilter.rar](108494/772558/40135.rar) |

| ![](img/0623e01110d741fd20f66acbc8ec6f9d.jpg) || [\[R0binary\]kbdFilter.rar](108494/772558/40134.rar) |

| ![](img/0623e01110d741fd20f66acbc8ec6f9d.jpg) || [\[R3binary\]KbdfilterControlCenter.rar](108494/772558/40136.rar) |

| ![](img/0623e01110d741fd20f66acbc8ec6f9d.jpg) || [\[R3source\]KbdfilterControlCenter.rar](108494/772558/40137.rar) |
