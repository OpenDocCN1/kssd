

# Visual Studio 2008 搭建 DDK（已经成功）

> 原文：[`www.pediy.com/kssd/pediy10/102163.html`](https://www.pediy.com/kssd/pediy10/102163.html)

第一步：安装程序  
  
安装 DDK,VS2008，DriverStudio 3.2 和 ddkwizard  
  
   
第二步：配置环境变量  
  
1.拷贝 DDKBUILD.bat 和 DDKBUILD.cmd 到 DDK 安装根目录,如我的是 C:\\WinDDK.  
  
2.将上述文件（DDKBUILD.bat 和 DDKBUILD.cmd ）的路径添加到系统的 path 变量(右击我的电脑,系统属性/高级/环境变量/系统变量/path 变量双击修改值)。  
  
3.根据你所安装的 DDK 的不同版本添加不同的变量到系统变量中。  
右击我的电脑,系统属性/高级/环境变量/系统变量/新建：  
2000 DDK 则变量名为: W2KBASE  
XP DDK 则变量名为: WXPBASE  
2003 sp1 DDK 则变量名为: WNETBASE  
这里我的是 WXPBASE(值为 C:\\WinDDK\\2006  
  
4.启动 vs2008 打开菜单中的工具选项，弹出选项对话框。选择左边的项目和解决方案 VC++目录，在右边的显示以下内容的目录下拉列表中选择可执行文件，在下面列表框中添加上面 DDKBuild.bat 文件所在的路径(这里是 C:\\WinDDK)，同样设置包含文件和库文件  
  
设置可执行文件（C:\\WinDDK）  
  
设置包含文件(C:\\WINDDK\\2006\\inc\\wnet C:\\WINDDK\\2006\\inc\\ddk\\wnet ）  
  
设置库文件（C:\\WINDDK\\2006\\lib\\wnet\\i386）  
  
   
  
第三步：建立驱动开发文件  
  
1.开始->所有程序->Compuware DriverStudio->Develop->DDK Build Settings  
  
选择第三项（Progame to start）,打开路径：C:\\Program Files\\Microsoft Visual Studio 9.0\\Common7\\IDE  
  
文件名：devenv.exe，点击 Launch Progame  
  
2.VC2008 在新建项目里面增加了有 DDK Project 的选项，新建 EmptyDriver ，添加代码，选择 WXP CHECK 或者 WXP FREE 编译通过。  
  
   
  
感想：辛苦很多天了，终于把 VS 2008 搭建 DDK 平台成功了。VS 2008 缺少快捷的搭建方法啊。  
  
过几天写一个专属自己 EasySYS 程序来，快速搭建适合自己平台。谢谢。  
  
   
  
（1）Visual Studio 2008 Team Suite 简体中文版下载地址：（90 天试用版）  
http://download.microsoft.com/download/d/8/9/d89c9839-ac45-4a6c-b25f-2f60b190e356/VS2008TeamSuiteENU90DayTrialX1429235.iso  
  
（2）ddkwizard  
  
http://www.pudn.com/downloads139/sourcecode/others/detail600300.html  
  
（3）DriverStudio 3.2  
  
http://soft.77169.com/HTML/36077.html  
  
（4）DriverStudio 3.2 注册机  
  
http://download.csdn.net/down/822465/mydreams  
  
（5）ddkbuild.cmd ddkbuild.bat  
  
http://ddkwizard.assarbad.net/downloads/ddkbuild\_bat.zip  
  
http://ddkwizard.assarbad.net/downloads/ddkbuild\_cmd.zip   
  
（6）DDK FOR XP  
  
http://210.21.117.86/down/ocx/winxpddk.zip  
  
（7）DAEMON 虚拟光驱-》用来装 VS2008 的 ISO 文件  
  
http://dl.pconline.com.cn/html\_2/1/121/id=1051&pn=0.html  
  
   
  
把 VS2008 九十天试用版本升级成正式版的办法  
  
法一：  
把 Setupsetup.sdb 文件中的\[Product Key\]，由 T2CRQGDKBVW7KJR8C6CKXMW3D 修改为 PYHYPWXB3BB2CCMV9DX9VDY8T。  
法二：  
安装好后，添加或删除程序，删除 VS2008，输入 cd-key：PYHYP-WXB3B-B2CCM-V9DX9-VDY8T 

DriverStudio3.2 在 XPSP3 及以上平台上好像不大好用，很多时候断不下来，你知道你有没有注意到。  
我现在是 Windows7+VMware  
在 VMware 里搭测试及 DEBUG 环境。然后在 WIN7 上远程 WINDBG。  
感觉这样好点。而且 VMware 支持这种调试功能。就是在虚拟机里有点慢，不过对调试来说就无所谓了。