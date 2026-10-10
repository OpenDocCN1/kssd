

# 文件夹加密超级大师软件的脆弱加密分析

> 原文：[`www.pediy.com/kssd/pediy11/109958.html`](https://www.pediy.com/kssd/pediy11/109958.html)

文件夹超级加密大师（版本:15, 9, 3, 0）是一种常用的加密软件，很多人在使用，但是常常看到有人因为丢失密码，而再找破解方法，因为如果找该软件的公司破解的话，它让你交 50 注册费，所以很多人因此而苦恼，现在我们来分析这个软件，首先看看这个软件的磁盘保护功能，磁盘保护有三种：初级、中级、高级。由于没注册所以只能使用初级和中级，  
初级的保护原理如下：  
HKEY\_LOCAL\_MACHINE\\SOFTWARE\\Microsoft\\Windows\\CurrentVersion\\policies\\Explorer 下新建 DWORD 值 NoDrives（其中隐藏的值与磁盘号按照以下规则：二进制类型,A 对应 1, B 对应 2, C 对应 4,D 对应 8，以此类推。如果你要隐藏 C 盘、D 盘的话就是:4+8=12）,同时还在 HKEY\_LOCAL\_MACHINE\\SOFTWARE\\Microsoft\\Windows\\CurrentVersion\\policies\\Explorer 下新建 DWORD 值 NoViewOnDrive，防止你打开磁盘，与上面 NoDrives 的设置值一样。你要不让浏览 C 盘、D 盘的话，你就设置 NoViewOnDrive 的值为 4+8=12 即可。  
中级的保护原理如下：  
通过删除磁盘的驱动号来隐藏磁盘，你通过控制面板-〉管理工具-〉计算机管理-〉磁盘管理，对删除磁盘的驱动号重新添加驱动号来重新显示磁盘。  
禁止使用 USB 设备的原理：  
HKEY\_LOCAL\_MACHINE\\SYSTEM\\CurrentControlSet\\Services\\USBSTOR  
里的 DWORD 值的名称为 Start 的 DWORD 值改为 4，就是让 USB 的驱动程序开机运行时不自动加载，从而达到禁止使用 U 盘的目的。  
只读使用 U 盘、移动硬盘的原理：  
该软件在注册表里的如下路径里：  
HKEY\_LOCAL\_MACHINE＼SYSTEM＼Current＼ControlSet＼Control 添加一个 StorageDevicePolicies 的项，然后再增加 DWORD 值的名称为 WriteProtect，将这个 DWORD 值设为 1。  
   文件夹超级加密大师的一个重要功能就是对文件夹可以进行加密，  
它的加密方式分为：快速、隐藏等几种加密方式，我在这里专门分析快速和隐藏这两种加密方式。当你安装好这个软件后，你会发现你计算机的每个磁盘下都有回收站（RECYCLER 这个文件夹），这个文件夹里保存着你用这个软件加密的文件夹。这就是加密某文件夹后的效果，如下图：  
   
图 2  
这个文件夹是无法打开、删除、复制等等，甚至使用冰刃、文件夹嗅探器等工具也无法识别和操作，因为这个文件夹含有特殊的字符所造成的，加密后的文件夹它包含三层，第一层特殊文件夹名如图 1，第二层文件夹名是(W-S-FSE-750715-771104-3000.)，第三层文件名就是 com1.{21EC2020-3AEA-1069-A2DD-08002B30309D}。我们需要一层层把它“抽丝剥茧”，这里注意第一层文件夹名是利用 Unicode 字符命名的。主要代码如下：  
function JieMi(Sourcedir,DestDir:widestring):boolean;  
var  
i:integer;  
TempDir,Reserved,LReserved: widestring;  
runpath:string;  
begin  
result:=false;  
//去掉第一层文件夹  
for i:=0 to 12 do  
sourcedir:= sourcedir + widechar($007F);  
sourcedir:=sourcedir+widechar($00E3)+widechar($002E)+widechar($002E)+widechar($005C);  
//去掉文件夹系统属性  
SetFileAttributesW(pwidechar(sourcedir),FILE\_ATTRIBUTE\_NORMAL);  
if not MoveFileW(Pwidechar(sourcedir),pwidechar(DestDir)) then  
exit;  
//去掉第二层文件夹  
TempDir :=DestDir+ 'W-S-FSE-750715-771104-3000..\\';  
Reserved:=DriverPath+'TEMP1\\';  
if not MoveFileW(Pwidechar(TempDir),pwidechar(Reserved)) then  
Exit;  
//去掉第三层文件夹  
TempDir:=Reserved+ 'com1.{21EC2020-3AEA-1069-A2DD-08002B30309D}\\';  
LReserved:=DriverPath+'解密文件夹\\';  
if not MoveFileW(Pwidechar(TempDir),pwidechar(LReserved)) then  
exit;  
SetFileAttributesW(pwidechar(LReserved+'Local\\'), FILE\_ATTRIBUTE\_NORMAL);  
SetFileAttributesW(pwidechar(LReserved+'Hide\\'), FILE\_ATTRIBUTE\_NORMAL);  
RmDir(string(DestDir));  
RmDir(string(Reserved));  
runpath:='Explorer  /n,/e,'+ LReserved;  
winexec(pchar(runpath),SW\_SHOW);  
esult:=True;  
end;  
下面通过该程序去解密文件夹超级加密大师 V15.9.3.0 这个软件加密的文件夹，其解密后的效果如下图：  
   
图 3  
解密后会自动打开解密的文件夹，其中 Hide 文件夹里保存着用隐藏方式加密的文件夹，而 Local 文件夹里保存着闪电方式加密的文件夹。每个解密的文件夹里会含有一个文件名为 FSEEncFolder.info 的文件，这就是存放加密该文件夹的密码的文件。其他的加密方式： 全面、金钻、移动。因为这几种方式该软件使用了加密算法把文件夹里的文件进行加密，如果想要解密出来的话，需要分析该软件所使用的算法或者解密出来加密密码才能对加密的文件夹进行解密。这里就不多说那，很多快速加密的软件大多都是使用我所分析的这种“加密“方式来加密文件，其中所不同的是加密后的文件夹的命名不同罢了，有兴趣的读者可以根据我所提供的思路和源代码来破解这类所谓的“高强度”加密软件。  
  
本文已发表在黑防 2010 第 3 期上。

上传的附件
| ![](img/0623e01110d741fd20f66acbc8ec6f9d.jpg) || 文件夹加密超级大师破解.rar  
 |