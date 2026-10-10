

# SoftICE 与 TRW2000 安装安装与配制 

> 原文：[`www.pediy.com/kssd/tutorial/chap3-1.htm`](https://www.pediy.com/kssd/tutorial/chap3-1.htm)

**SoftICE 与 TRW2000 安装安装与配制** 

>  

SOFTICE 有几个平台的版本，DOS,WINDOWS 3.0,Windows 95/98,WINDOWS NT,等。 由于现在最普及的操作系统是 Windows 95/98、Windows NT、Windows Millennium、Windows2000 因此就讲讲 SOFTICE 在这几个平台安装时的一些注意事项。

一、SOFTICE for win9x 安装与配制   
  
　　1、显卡安装  
　　2、鼠标安装  
　　3、Autoexec.bat 和 config.sys 配制  
　　4、Symbol Loader  
　　5、winice.dat 配制  
二、SOFTICE for Windows Millennium  
三、SOFTICE for NT/2K 安装与配制  
四、TRW2000 安装  

　**SOFTICE 的安装与配制**

**一、SOFTICE for win9x 安装与配制**

**㈠、SOFTICE 安装　**

1、SOFTICE 目前最新版本是 4.05，如你的系统是 win9x,就请下载 for win9x 版本的 SOFTICE，建议下载 SOFTICE 的最新版本，这样稳定性好些。运行 setup.exe 开始安装，出现（图一）：

![](img/4f4c0fdc6fffda23b219b0a4dc9f96eb.jpg)

（图一）

2、然后点击下一步，输入安装序列号（图二）（序列号一般在安装软件的 readme.txt 或其它说明文件里）

![](img/f7d9a8fa46d114c21b9d2f53ef631aee.jpg)

（图二）

3、然后下面几个画面是要求选定路径和安装组件，不久你会来到显卡配制对话框（图三）：

![](img/53142bfeeaa250280e1a87038035309b.jpg)

（图三）

在显卡配制里两种方法：  
  
①这种配制使 SOFTICE 激活状态时类似 DOS 全屏状态一样（也就是字符模式状态），在显卡列表选择你的显卡类型，Universal Vidoe Driver 和 Use monochrome card/monitor 这两项不要选，然后点击**Test**按钮，在测试过程中你能看到各种颜色的字符，说明显卡测试通过，就可点击下一步了。  
  
② 这种配制使 SOFTICE 在激活状态下类似 windows 应用程序的一个窗口那样，这样在调试时可避免显示器不停地在图形和字符模式转换，对提高显示器寿命大有好处。配制时，显卡列表一栏忽略，不用配制，只要把 Universal Vidoe Driver 这一项选上，然后 test，如跳出如（图四）对话框，测试通过。（本人强烈推荐这种方式）

![](img/e456879424ac01f6278e0a6d16fa7201.jpg)  
  
（图四） 

4、再下一步是鼠标的配制（图五）：

![](img/ef479594b111bb52ffb0c144c3d7abad.jpg)

（图五）

现在鼠标常见的一般是串行口或 ps/2 接口，你跟据自己的鼠标接口类型或位置选上合适的就可。如碰到鼠标在 SOFTICE 调试画面不能用或一用死机，可能是没选好正确的选项，你可在 SOFTICE 菜单里的，运行 mosue setup 这个菜单项重新配制鼠标。

5、最后一步是询问以何种方式装载 SOFTICE 的主文件 winice.exe（图六）：

![](img/58681dfa67d21821dacbab9b0df62c6d.jpg)

（图六）

首先要了解 SOFTICE for 9x 版本是如何装载的，在 SOFTICE 的安装目录下有 winice.exe 这个文件，windows 启动到纯 DOS 环境下，运行 winice.exe 这个文件，将装载 SOFTICE。  
安装时默认将 C:\\PROGRA~1\\NUMEGA\\SOFTIC~1\\WINICE.EXE 这一行放时进 Autoexec.bat（自动批处理文件），这样 windows 以后每次都运行 Autoexec.bat 这个文件，自动装载 SOFTICE。另外，你可根据自身需要配制启动模式，具体参考第 8 步。

6、然后安装程序将复制文件到硬盘里，来到最后一个对话框（图七）：

![](img/17ac193f56a66156e03c422e61542771.jpg)

（图七）

最后一项是电子注册，注册成功后，以后可在线升级 SOFTICE 版本，这里选最后一项 Regiter later.

7、至此安装完成，重新启动 windows，微机先到 DOS 下，自动或手动运行相应批处理文件，运行其内的 winice.exe 文件，装载 windows。

8、启动菜单配制：

通过配制启动菜单，启动时根据自己的需要选择是否装载 SOFTICE。

**AUTOEXEC.BAT**配制样例： 

| @ECHO OFF  
goto %config%:SICE  
C:\\PROGRA~1\\NUMEGA\\SOFTIC~1\\WINICE.EXE  
goto common:NORM  
goto common:common |
| --- |

**CONFIG.SYS**配制样例：  

| \[MENU\]  
MENUITEM NORM,Windows 9x  
MENUITEM SICE,Windows 9x with SoftICE  
MENUDEFAULT NORM,2  
  
\[NORM\]  
  
\[SICE\]  
  
\[common\]  
 |
| --- |

 

**㈡、SOFTICE 配制**

1、**Symbol Loader 的使用**

![](img/24ed6a651861d92b1c6ac8c8f4cd8d92.jpg)  
（图八） 

你在开始 SOFTICE 的菜单里有一项 ymbol Loader 快捷方式，运行后，在其菜单***EDIT***→***SOFTICE Initialization Settings***选项，打开后如图（图八）。这里你就可配制 SOFTICE 了。

①**General 选项**

在 Initialization string 里，你可填上需要 SOFTICE 一启动自动运行的命令。如：

`WD 2; WC 14; FAULTS OFF; IXHERE OFF; IYHERE OFF; set font 2;lines 40;x;（各行以分号分开）`

②**Exports 选项**

在这里可添加相关的 DLL 文件，以便在 SOFTICE 下拦截这些 DLL 的函数。特别是破解 VB 程序时，定要将 VB 运行库装载进去。

③**Keyboard Mappings 选项**

这里配制各功能热键。如：F5="^x;"用 F5 键代替命令 X.

④**Macro Definitions 选项**

宏定义，你可定制各种命令宏，以方便平时的操作。  
如：s7878="S 30:0 L ffffffff '78787878' " 用命令 s7878 代替一串命令：S 30:0 L ffffffff '78787878'

⑤**Remote Debugging**

利用网络远程调试配制。

***注：以上所有配制好后的参数，都保存在 winice.dat 文件里。***

2、**winice.dat 配制**

在 Windows 9x 下 SoftICE 配制除了用上面的方法外，也可通过文件 winice.dat 来实现的。Soft-ICE 在启动的时候通过它装入一些 DLL/EXE 的信息。  
你可在 SOFTICE 安装目录下发现 winice.dat，可用任何文本编辑软件打开它（如记事本）。如我机子里的 winice.dat：(你可参考我的来修改你的 winice.dat）

* * *

| 

```


;注意分号后是描述语言，不被执行。
```

 
PHYSMB=128;<=这个值是你的物理内存大小  
SYM=1024  
HST=256;<=历史缓冲区为 256K  
TRA=8  
MACROS=32;<=宏操作的最大个数，此处是 32 个  
DRAWSIZE=2048;<= 我的显卡内存是 2MB ，此处值是你显卡内存大小 INIT=" wd 2;wc 20;FAULTS OFF; IXHERE OFF; IYHERE OFF; set font 2;lines 40;code on;x;";<=初始化，此处默认的是 800＊600 分分辨率   
;如你是全屏请换上：lines 57F1="h;"  
F2="^wr;"  
F3="^src;"   
F4="^rs;"  
F5="^x;"  
F6="^ec;"  
F7="^here;"  
F8="^t;"  
F9="^bpx;"  
F10="^p;"  
F11="^G @SS:ESP;"  
F12="^p ret;"  
SF3="^format;"  
CF8="^XT;"  
CF9="TRACE OFF;"  
CF10="^XP;"  
CF11="SHOW B;"  
CF12="TRACE B;"  
AF1="^wr;"  
AF2="^wd;"  
AF3="^S 0 L FFFFFFFF 8B,CA,F3,A6,74,01,9F,92,8D,5E,08;";<=VB3 特征字符串   
AF4="^s 0 l ffffffff 56,57,8B,7C,24,10,8B,74,24,0C,8B,4C,24,14,33,C0,F3,66,A7;" ;<=VB4 特征字符串  
AF5="^s 0 l ffffffff FF,75,E0,E8,85,EF,FF,FF,DC,1D,28,10,40,00,DF,E0,9E,75,03;" ;<=VB5 特征字符串  
　　　　　　　　　　　　　　　　　　　　　　　　　　　  
AF8="^XT R;"  
AF11="^dd dataaddr->0;"  
AF12="^dd dataaddr->4;"  
CF1="altscr off; lines 60; wc 32; wd 8;"  
CF2="^wr;^wd;^wc;";<=以下是宏操作命令：  
MACRO s7878="S 30:0 L ffffffff '78787878' "  
MACRO sname="S 0 L FFFFFFFF 'toye' "  
MACRO swide="s 0 l FFFFFFFF '7','8,'7','8,'7','8,'7','8,'7','8,'7','8,'7','8','7','8' "  
MACRO reg="bpx regqueryvalueexa if \*(esp->8)>='Soft' do "d(esp->14)" "  
MACRO bpxpe="bpx loadlibrarya do "dd esp->4" "  
MACRO bpxgeta="bpx GetDlgItemTextA; bpx getwindowtexta; bpx getdlgitemint; bpx getdlgitemtext;"; \*\*\*\*\* Examples of sym files that can be included if you have the SDK \*\*\*\*\*  
; Change the path to the appropriate drive and directory  
;LOAD=c:\\windows\\system\\user.exe  
;LOAD=c:\\windows\\system\\gdi.exe  
;LOAD=c:\\windows\\system\\krnl386.exe  
;LOAD=c:\\windows\\system\\mmsystem.dll  
;LOAD=c:\\windows\\system\\win386.exe; Exports - change the path to the appropriate drive and directoryEXP=c:\\windows\\system\\advapi32.dll ;<=这四行前不要加分号,否则不被装载，SOFTICE 可能什么也拦不到 :   
EXP=c:\\windows\\system\\kernel32.dll  
EXP=c:\\windows\\system\\user32.dll  
exp=c:\\windows\\system\\gdi32.dll   
exp=c:\\windows\\system\\comctl32.dll ;  
; 如你要破解 VB 程序，下面的 VB 运行库将要装载，SOFTICE 默认值是没有这几行，你需手动加上。;EXP=c:\\windows\\system\\msvbvm60.dll;<= Visual Basic 6 具体参考第十五课 VB 破解  
EXP=c:\\windows\\system\\msvbvm50.dll ;<= Visual Basic 5 注意在这五个 DLL 中最好不要同时装载 2 个以上  
; EXP=c:\\windows\\system\\vb40032.dll;<= Visual Basic 4（32bit)  
; EXP=c:\\windows\\system\\vb40016.dll;<=Visual Basic 4(16-bit)较少见  
; EXP=c:\\windows\\system\\vbrun300.dll;<=Visual Basic 3  
;EXP=c:\\windows\\system\\vga.drv;  
;EXP=c:\\windows\\system\\vga.3gr  
;EXP=c:\\windows\\system\\sound.drv  
;EXP=c:\\windows\\system\\mouse.drv  
;EXP=c:\\windows\\system\\netware.drv  
;EXP=c:\\windows\\system\\system.drv  
;EXP=c:\\windows\\system\\keyboard.drv  
;EXP=c:\\windows\\system\\toolhelp.dll  
;EXP=c:\\windows\\system\\shell.dll  
;EXP=c:\\windows\\system\\commdlg.dll  
;EXP=c:\\windows\\system\\olesvr.dll  
;EXP=c:\\windows\\system\\olecli.dll  
;EXP=c:\\windows\\system\\mmsystem.dll  
;EXP=c:\\windows\\system\\winoldap.mod  
;EXP=c:\\windows\\progman.exe   
;EXP=c:\\windows\\drwatson.exe   
; \*\*\*\*\* Examples of export symbols that can be included for Windows 95 \*\*\*\*\*  
; Change the path to the appropriate drive and directory  
EXP=c:\\windows\\system\\kernel32.dll  
EXP=c:\\windows\\system\\user32.dll  
EXP=c:\\windows\\system\\gdi32.dll  
EXP=c:\\windows\\system\\comdlg32.dll  
EXP=c:\\windows\\system\\shell32.dll  
EXP=c:\\windows\\system\\advapi32.dll  
EXP=c:\\windows\\system\\shell232.dll  
EXP=c:\\windows\\system\\comctl32.dll  
;EXP=c:\\windows\\system\\crtdll.dll  
;EXP=c:\\windows\\system\\version.dll  
EXP=c:\\windows\\system\\netlib32.dll  
;EXP=c:\\windows\\system\\msshrui.dll  
EXP=c:\\windows\\system\\msnet32.dll  
EXP=c:\\windows\\system\\mspwl32.dll  
;EXP=c:\\windows\\system\\mpr.dll  
 |
| --- |

　启动 windows 装载 SOFTICE 后，咦！怎么没反应，没调试画面！哈哈，别着急，按 CTRL＋D 看看，再按一下回到 windows 下，或按 F5 也能回来。此时调试窗口象 windows 开的一窗口，如是象全屏 DOS 一样窗口，那就是安装显卡时，参数没选好，此时按上文修正即可。下面的命令是调整 SOFTICE 窗口状态：

 　　set font n(n=1,2,3)设置字体;本人建议 set font 2(在 800\*600 条件下）  
　　set origin x,y(x,y)锁定窗口;  
　　lines n n=(25-128)设置显示行数；本人建议 lines 40  
　　Ctrl+Alt+光标键 移动窗口；  
　　Ctrl+Alt+home 重设窗口位置原点（0,0）；  
　　Ctrl+L 刷新。

如你以默认 winice.dat 启动 SOFTICE，有可能需用 WD 打开数据窗口;用 SET FONT 2 设置字体等重复工作。你可在 winice.dat 文件内设置自动执行命令操作，方法是在 INIT 这一行，各命令用分号分开，如：

INIT=" WD 2; WC 14; FAULTS OFF; IXHERE OFF; IYHERE OFF; set font 2;lines 40;x;"这样配制后界面类似 TRW2000。（这些是在 800\*600 条件下的情况，如你不是此分辩率可调整 set font n;lines n）

**二、SOFTICE for Windows Millennium**

　由于 Windows Millennium 没有了 DOS 平台，因此不能用常用的方法来安装 SOFTICE,在此平台上可以运行 SOFTICE for WIN9x 版本和 TRW2000 版本。要实现 SOFTICE 在 Windows Millennium 平台的安装，需要工具**Winice Loader**，安装过程如下: 

1 正常安装 Windows Millennium,如在 C:\\WIN98ME；  
2 正常安装 Softice 405 Build 334；  
3 复制 WINICE.EXE、WINICE.DAT、SIWVID.386 三个文件到 C:\\WIN98ME 下；  
4 从**Winice Loader**内解压 LOADER.EXE,复制这个文件到 C:\\WINME98\\SYSTEM\\VMM32\\ 下.   
5 重新启动系统。

**三、SOFTICE for NT/2K 安装与配制**

1、SOFTICE for NT/2k 的安装与 for 9x 版本差不多，所不同的是在第五步：装载 SOFTICE 方式选择，如图（图九）：

![](img/9f4f05c5748a4da6b010b1f95ed69f1d.jpg)

（图九）

你可根据需要选择不同的装载方式，注：如你选择了 Manual 方式，要装载 SOFTICE，需要来 SOFTICE 的菜单里运行选项：START SOFTICE 快捷方式来装载 SOFTICE。 

2、在 NT 下，配制 SOFTICE 是用 SOFTICE Loader（从你的开始菜单选），选择 Edit/SoftICE，一般的选项是初始化，这里你可参考手册了解不同的开关选项的详细描述。如：

CODE ON; FAULTS OFF; I3HERE OFF; WD 3; WF; X; 

其它两个重要的选项是 Symbols & Exports。如果你拥有自己系统的 SDK（软件开发工具包），你可用 SOFTICE 装载并调试它。那些没有 SDK 应该用 exports 选项从%WINNT%/System32 目录下增加下面的 DLL 文件。 

advapi32.dll, comctl32.dll, comdlg32.dll, gdi32.dll, kernel32.dll, msvbvm(50/60).dll (如果需要), msvcrt.dll (如果需要), ole32.dll, oleaut32.dll, shell32.dll, user32.dll, version.dll. 

 

**四、TRW2000 的安装与配制 **

**㈠、TRW2000 安装**

TRW 安装简单多了，没 SOFTICE 那样复杂，但目前 TRW2000 不支持 windows NT。它发布版本是一个 ZIP 压缩包，才 200 多 K。只要将其解压缩到一个目录下，然后运 TRW2000.EXE 即可（图十） , 无须安装或者重启计算机。

![](img/b0ebab788e7a1a864ff278e7f9b1b974.jpg)

（图十）

 激活方式同 SOFTICE 不一样：

 1. Ctrl + M 特权级 0 级的热键，能够立即中断 Win9x。 相当于 Soft-ICE 的热键 Ctrl+D. 

2\. Ctrl + N 特权级 3 级的热键。 在绝大多数时候，我们并不需要在 0 级上中断。 Ctrl + N 可以中断 Windows 的特权级 3 级的前台线程。 这应该是我们最常用的。 

其它指令同 SOFTICE 兼容，也就说你在上一节学得东西完全可用在 TRW2000 上。

但是 TRW2000 有许多更新的思想，具体看后面几课介绍及范例。

另外，TRW2000 可支持 plug-ins，也可装载 dll 文件，在 1.15 版本以上，在安装目录下有一 DLL 目录，如你特别需要的 dll 复制到此目录，即可装载，如破解 VB 时，就需要将 VB dll 复制此目录，具体参考后面的 VB 破解。

其它的请读其 Readme.

**㈡、TRW2000 的配制**

TRW2000 的配制是通过其安装目录下的

TRW2000.ini 来实现的，你可按自己的需要配制它（一般按默认即可）。

| ; TRW2000 Initialize file; Please modify it as your habit . ;rem PLUGS=C:\\PLUGS\\HELLO.SYS  
;No, you don't put this line. Now we have changed plug-ins load-method.  
;Please copy your SYS to <TRW2000 Directory>\\SYS, TRW2000 will load it  
;automatic.; Now, we don't support keyword LINES=, please use command LINES instead.  
INIT="lines 35;wr 3;wd 4;wc 16"F1=^HELP ; Command length CAN'T be longer than 15 characters !  
; This command length is 5 charcaters .  
F3=^SRC  
F4=^RS  
F5=^X  
F6=^EC  
F7=^HERE  
F8=^T  
F9=^BPX  
F10=^P  
F12=^PRET;HOTKEY=320D ;Ctrl+M  
;R3HOTKEY=310E ;Ctrl+NGRAPHICS=ON ;Use graphics mode driverVESA=OFFVGA=ONINTELLIMOUSE=ON ;If your mouse is a intellmouse, set this to ONWINMOUSE=ON ;If you found your mouse is not work properly, try it to ON. But we do NOT recommend this, because it maybe decrease stabilization.;HST=256 ;History buffer size, default = 256kSYMBUFFER=1024 CAPITAL=ONWONDER=ONTESTMODE=OFF |
| --- |

 4dd466d0e1e0cc123afbe504da2462ca