
# [原创]提供一个C#编写的基类源码（用于操作WINDOWNS系统的服务）

可以对系统服务 检查 增加 删除 启动 停止 重启

``` 
/// <summary>
/// Windows服务类
/// </summary>
public class ServiceUl
{
/// <summary>
/// 检查服务存在的存在性
/// </summary>
/// <param name="NameService">服务名</param>
/// <returns>存在返回 true,否则返回 false;</returns>
public static bool isServiceIsExisted(string NameService)
{
ServiceController[] services = ServiceController.GetServices();
foreach (ServiceController s in services)
if (s.NameService.ToLower() == NameService.ToLower())
return true;
return false;
}

    /// <summary>
    /// 安装Windows服务
    /// </summary>
    /// <param name="stateSaver">集合</param>
    /// <param name="filepath">程序文件路径</param>
    public static void InstallmyService(IDictionary stateSaver, string filepath)
    {
        try
        {
            AssemblyInstaller AssemblyInstaller1 = new AssemblyInstaller();
            AssemblyInstaller1.UseNewContext = true;
            AssemblyInstaller1.Path = filepath;
            AssemblyInstaller1.Install(stateSaver);
            AssemblyInstaller1.Commit(stateSaver);
            AssemblyInstaller1.Dispose();
        }
        catch (Exception ex)
        {
            throw new SysException(ex.Message, ex);
        }
    }

    /// <summary>
    /// 卸载Windows服务
    /// </summary>
    /// <param name="filepath">程序文件路径</param>
    public static void UnInstallmyService(string filepath)
    {
        try
        {
            AssemblyInstaller AssemblyInstaller1 = new AssemblyInstaller();
            AssemblyInstaller1.UseNewContext = true;
            AssemblyInstaller1.Path = filepath;
            AssemblyInstaller1.Uninstall(null);
            AssemblyInstaller1.Dispose();
        }
        catch (Exception ex)
        {
            throw new SysException(ex.Message, ex);
        }
    }

    /// <summary>
    /// 检查Windows服务是否在运行
    /// </summary>
    /// <param name="name">程序的服务名</param>
    public static bool IsRunning(string name)
    {
        bool IsRun = false;
        try
        {
            if (!ServiceIsExisted(name)) return false;
            var sc = new ServiceController(name);
            if (sc.Status == ServiceControllerStatus.StartPending || sc.Status == ServiceControllerStatus.Running)
            {
                IsRun = true;
            }
            sc.Close();
        }
        catch
        {
            IsRun = false;
        }
        return IsRun;
    }

    /// <summary>
    /// 启动Windows服务
    /// </summary>
    /// <param name="name">程序的服务名</param>
    /// <returns>启动成功返回 true,否则返回 false;</returns>
    public static bool StarmyService(string name)
    {
        try
        {
            var sc = new ServiceController(name);
            if (sc.Status == ServiceControllerStatus.Stopped || sc.Status == ServiceControllerStatus.StopPending)
            {
                sc.Start();
                sc.WaitForStatus(ServiceControllerStatus.Running, new TimeSpan(0, 0, 10));
            }
            else
            {

            }
            sc.Close();
            return true;
        }
        catch (Exception ex)
        {
            throw new SysException(ex.Message, ex);
        }
    }

    /// <summary>
    /// 停止Windows服务
    /// </summary>
    /// <param name="name">程序的服务名</param>
    /// <returns>停止成功返回 true,否则返回 false;</returns>
    public static bool StopmyService(string name)
    {
        try
        {
            var sc = new ServiceController(name);
            if (sc.Status == ServiceControllerStatus.Running || sc.Status == ServiceControllerStatus.StartPending)
            {
                sc.Stop();
                sc.WaitForStatus(ServiceControllerStatus.Stopped, new TimeSpan(0, 0, 10));
            }
            else
            {

            }
            sc.Close();
            return true;
        }
        catch (Exception ex)
        {
            throw new SysException(ex.Message, ex);
        }
    }

    /// <summary>
    /// 重启Windows服务
    /// </summary>
    /// <param name="name">程序的服务名</param>
    /// <returns>重启成功返回 true,否则返回 false;</returns>
    public static bool RefreshmyService(string name)
    {
        return StopmyService(name) && StarmyService(name);
    }
}
```

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)
