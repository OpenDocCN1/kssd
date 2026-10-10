

# 【原创】VS2005 + WDK7.0+DDKWizard 配置驱动开发环境

> 原文：[`www.pediy.com/kssd/pediy11/109854.html`](https://www.pediy.com/kssd/pediy11/109854.html)

首先，依次安装 VS2005，WDK，DDKWizard。DDKWizard 下载地址 http://ddkwizard.assarbad.net/，分别下载 ddkwizard\_setup.exe，ddkbuild\_cmd.zip，和 ddkbuild\_bat.zip。  
  
如果 WDK 的安装路径是 D:\\WinDDK\\7600.16385.0，则复制 ddkbuild.cmd 和 ddkbuild.bat 到目录 D:\\WINDDK\\下。  
  
然后，启动 VS2005 选择菜单“工具”->“选项”->“项目和解决方案”->“VC++目录”，“可执行文件”添加目录：D:\\WINDDK；“库文件”添加目录 D:\\WinDDK\\7600.16385.0\\lib\\wxp；“包含文件”添加目录 D:\\WinDDK\\7600.16385.0\\inc\\ddk。  
  
最后还要添加系统环境变量 WXPBASE = D:\\WinDDK\\7600.16385.0，配置完成。  
  
启动 VS 新建项目，在 Visual C++项目下便会有个 DDK Project 选项，在右侧的模板里选择“Driver”即生成一个驱动项目。在 DDKWizard 中选择 windows xp dkk，把下面的复选框的选项都去掉，点击“Finish”即可。F7 编译即生成 sys 驱动文件……  
  
不过 DDKWizard 生成的模板不太合我胃口，Driver 模板太复杂太乱，EmptyDriver 又只生成空的项目。  
  
于是 DIY 了一个 SimpleDriver，具体做法如下：  
  
在 VS 的安装目录下找到存放项目模板的文件，如我的是在：  
  
D:\\Program Files\\Microsoft Visual Studio 8\\VC\\vcprojects\\DDK Project  
  
打开文件 DDKWizard.vsdir，新增一句：  
  
SimpleDriver.vsz| |Simple driver|1|Creates a Simple driver project using DDKBUILD and the specified DDK.| |6777| |SimpleDriver  
  
保存。  
  
创建一个图标文件：SimpleDriver.ico 和一个 SimpleDriver.vsz 文件，并设置 SimpleDriver.vsz 内容为：  
  
VSWIZARD 7.0  
Wizard=VsWizard.VsWizardEngine.8.0  
Param="WIZARD\_NAME = SimpleDriver"  
Param="ABSOLUTE\_PATH = D:\\Program Files\\DDKWizard\\Wizards\\SimpleDriver"  
Param="FALLBACK\_LCID = 1033"  
  
其中 SimpleDriver 是向导名字，D:\\Program Files\\DDKWizard\\Wizards\\SimpleDriver 是存储模板的目录。  
  
在 D:\\Program Files\\DDKWizard\\Wizards 目录下，复制一份 Driver 目录文件并改名为 SimpleDriver 作为我们的模板文件。进入目录 D:\\Program Files\\DDKWizard\\Wizards\\SimpleDriver\\Templates\\1033，  
  
清除 PROJECT\_NAME.h 文件的正文内容，只留下版权注释部分和下面一行的// $Id$；编辑 PROJECT\_NAME.c 的内容为：  
  
///////////////////////////////////////////////////////////////////////////////  
///  
/// Copyright (c) \[!output PROJECT\_YEAR\] - \[!output DDKWIZ\_USERVARIABLE\_COMPANY\]  
///  
/// Original filename: \[!output PROJECT\_FILENAME\]  
/// Project          : \[!output PROJECT\_NAME\]  
/// Date of creation : \[!output PROJECT\_DATE\]  
/// Author(s)        : softsing  
///  
/// Purpose          : <description>  
///  
/// Revisions:  
/// 0000 \[\[!output PROJECT\_DATE\]\] Initial revision.  
///  
///////////////////////////////////////////////////////////////////////////////  
  
#include <ntddk.h>  
//#include "\[!output PROJECT\_NAME\].h"  
  
VOID DriverUnload(IN PDRIVER\_OBJECT DriverObject)  
{  
        //Add your codes here...  
}  
  
NTSTATUS DriverEntry(IN OUT PDRIVER\_OBJECT DriverObject, IN PUNICODE\_STRING RegistryPath)  
{  
        DriverObject->DriverUnload = DriverUnload;  
        //Add your codes here...  
        return STATUS\_SUCCESS;  
}  
  
以便生成最简单项目文件。  
  
由于是 XP 环境，因此每次创建项目都要重新选择 XP DDK 环境，未免显得多余。可以修改目录  
  
D:\\Program Files\\DDKWizard\\Wizards\\SimpleDriver\\HTML\\1033 下的 default.htm 文件，修改代码为：  
  
<select size="4" id="DDK\_CHOICE\_LISTBOX" multiple="multiple" style="width:70%;">  
          <option value="0">Windows 2000 DDK</option>  
          <option value="1" selected="selected">Windows XP DDK</option>  
          <option value="2">Windows 2003 DDK</option>  
          <option value="3">Longhorn/Vista WDK</option>  
</select>  
  
保存后，那么以后新建项目默认选项就是“Windows XP DDK”了。当然自定义设置还很多，你可以根据自己的需要来定制。