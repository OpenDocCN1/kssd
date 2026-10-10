

# Absolute Time Corrector V2.1 build 137

> 原文：[`www.pediy.com/kssd/pediy05/pediy50283.htm`](https://www.pediy.com/kssd/pediy05/pediy50283.htm)

首先声明，严格说来这算不上破解，只是提供了一种对付日期和使用次数   
限制和 NAG screen 保护方式的一种简单的方法，高手可以不看。   
  
巧破 Absolute Time Corrector V2.1 build 137   
\===============================================   
Sunrix, 2002-9-20   
  
XP 支持与 Internet 时间服务器同步来校正计算机系统时间，可是 XP 这个功能不   
支持代理服务器。所以上网找了个第三方的时间同步程序：Absolute Time Corrector   
它支持 Socks 代理。这是个共享软件，在网上没有找到 crack,可能是不太出名，   
用的人不多吧：)。少不得自力更生了。   
  
软件：Absolute Time Corrector V2.1 build 137   
保护方式：Nag screen + 使用次数限制   
工具：Regmon for NT/2000/XP   
  
一般来说，软件把已经使用次数都数据存在注册表中，所以用 Regmon 来追踪 ATC   
对注册表的读。浏览 Regmon 的观察结果，很轻易的发现了下面很特别的地方：   
  
CreateKey    HKCR\\.key    SUCCESS    Key: 0xE28B12A8       
QueryKey    HKCR\\.key    SUCCESS    Name: \\REGISTRY\\MACHINE\\SOFTWARE\\Classes\\.key       
CreateKey    HKCR\\CLSID\\{B22B19A8-C7DC-76B6-5781-646D9C08DF94}    SUCCESS    Key: 0xE18BD958       
QueryKey    HKCR\\CLSID\\{B22B19A8-C7DC-76B6-5781-646D9C08DF94}    SUCCESS    Name: \\REGISTRY\\MACHINE\\SOFTWARE\\Classes\\CLSID\\{B22B19A8-C7DC-76B6-5781-646D9C08DF94}       
QueryValue    HKCR\\CLSID\\{B22B19A8-C7DC-76B6-5781-646D9C08DF94}\\(Default)    SUCCESS    "S/Y="       
QueryKey    HKCR\\CLSID\\{B22B19A8-C7DC-76B6-5781-646D9C08DF94}    SUCCESS    Name: \\REGISTRY\\MACHINE\\SOFTWARE\\Classes\\CLSID\\{B22B19A8-C7DC-76B6-5781-646D9C08DF94}       
QueryValue    HKCR\\CLSID\\{B22B19A8-C7DC-76B6-5781-646D9C08DF94}\\(Default)    SUCCESS    "S/Y="       
QueryKey    HKCR\\CLSID\\{B22B19A8-C7DC-76B6-5781-646D9C08DF94}    SUCCESS    Name: \\REGISTRY\\MACHINE\\SOFTWARE\\Classes\\CLSID\\{B22B19A8-C7DC-76B6-5781-646D9C08DF94}       
QueryValue    HKCR\\CLSID\\{B22B19A8-C7DC-76B6-5781-646D9C08DF94}\\(Default)    SUCCESS    "S/Y="       
QueryKey    HKCR\\CLSID\\{B22B19A8-C7DC-76B6-5781-646D9C08DF94}    SUCCESS    Name: \\REGISTRY\\MACHINE\\SOFTWARE\\Classes\\CLSID\\{B22B19A8-C7DC-76B6-5781-646D9C08DF94}       
QueryValue    HKCR\\CLSID\\{B22B19A8-C7DC-76B6-5781-646D9C08DF94}\\(Default)    SUCCESS    "S/Y="       
CreateKey    HKCR\\CLSID\\{B22B19A8-C7DC-76B6-5781-646D9C08DF94}    SUCCESS    Key: 0xE28B12A8       
QueryKey    HKCR\\CLSID\\{B22B19A8-C7DC-76B6-5781-646D9C08DF94}    SUCCESS    Name: \\REGISTRY\\MACHINE\\SOFTWARE\\Classes\\CLSID\\{B22B19A8-C7DC-76B6-5781-646D9C08DF94}       
QueryValue    HKCR\\CLSID\\{B22B19A8-C7DC-76B6-5781-646D9C08DF94}\\(Default)    SUCCESS    "S/Y="       
QueryKey    HKCR\\CLSID\\{B22B19A8-C7DC-76B6-5781-646D9C08DF94}    SUCCESS    Name: \\REGISTRY\\MACHINE\\SOFTWARE\\Classes\\CLSID\\{B22B19A8-C7DC-76B6-5781-646D9C08DF94}       
QueryValue    HKCR\\CLSID\\{B22B19A8-C7DC-76B6-5781-646D9C08DF94}\\(Default)    SUCCESS    "S/Y="       
QueryKey    HKCR\\CLSID\\{B22B19A8-C7DC-76B6-5781-646D9C08DF94}    SUCCESS    Name: \\REGISTRY\\MACHINE\\SOFTWARE\\Classes\\CLSID\\{B22B19A8-C7DC-76B6-5781-646D9C08DF94}       
QueryValue    HKCR\\CLSID\\{B22B19A8-C7DC-76B6-5781-646D9C08DF94}\\(Default)    SUCCESS    "S/Y="       
QueryKey    HKCR\\CLSID\\{B22B19A8-C7DC-76B6-5781-646D9C08DF94}    SUCCESS    Name: \\REGISTRY\\MACHINE\\SOFTWARE\\Classes\\CLSID\\{B22B19A8-C7DC-76B6-5781-646D9C08DF94}       
QueryValue    HKCR\\CLSID\\{B22B19A8-C7DC-76B6-5781-646D9C08DF94}\\(Default)    SUCCESS    "S/Y="       
QueryKey    HKCR\\CLSID\\{B22B19A8-C7DC-76B6-5781-646D9C08DF94}    SUCCESS    Name: \\REGISTRY\\MACHINE\\SOFTWARE\\Classes\\CLSID\\{B22B19A8-C7DC-76B6-5781-646D9C08DF94}       
CreateKey    HKCU\\Software\\Flexiblesoft\\ATC\\InstallDate    SUCCESS    Key: 0xE1565B20       
QueryValue    HKCU\\Software\\Flexiblesoft\\ATC\\InstallDate\\InstallDate    SUCCESS    00 00 00 00 E0 50 E2 40        
  
有三个可疑的地方：   
1、HKCR\\.key   
2、HKCR\\CLSID\\{B22B19A8-C7DC-76B6-5781-646D9C08DF94}   
3、HKCU\\Software\\Flexiblesoft\\ATC\\InstallDate   
  
打开注册表编辑器，试着把下面三个 SubKey 删掉：   
1、HKCR\\.key   
2、HKCR\\CLSID\\{B22B19A8-C7DC-76B6-5781-646D9C08DF94}   
3、HKCU\\Software\\Flexiblesoft   
  
再重新运行 ATC，哈哈，试用次数又回到了 50 次!   
用 VBScript 写一个脚本来清除上面的 Key，未注册版本启动时   
要弹出 Nag Screen，非常 boring，正好也可利用 VBScript 来   
模拟按键，跳过 Nag screen。   
  
on error resume next   
  
set WSHShell = CreateObject("WScript.Shell")   
WSHShell.RegDelete "HKCR\\.key\\"   
  
' regdelete 不能删除有 subkey 的 key，所以先把 subkey 删掉   
WSHShell.RegDelete "HKCR\\CLSID\\{B22B19A8-C7DC-76B6-5781-646D9C08DF94}\\InProcServer32\\"   
WSHShell.RegDelete "HKCR\\CLSID\\{B22B19A8-C7DC-76B6-5781-646D9C08DF94}\\ProgID\\"   
WSHShell.RegDelete "HKCR\\CLSID\\{B22B19A8-C7DC-76B6-5781-646D9C08DF94}\\"   
  
WSHShell.RegDelete "HKCU\\Software\\Flexiblesoft\\ATC\\InstallDate\\"   
WSHShell.RegDelete "HKCU\\Software\\Flexiblesoft\\ATC\\"   
WSHShell.RegDelete "HKCU\\Software\\Flexiblesoft\\"   
  
WSHShell.Run "atc.exe"    '将本脚本与 atc.exe 放在同一目录下   
WScript.Sleep 1000    '等待 Nag screen 弹出并接收到键盘输入焦点   
WSHShell.SendKeys "{TAB}"    '切换到 Evaluating 按钮   
WSHShell.SendKeys "{ENTER}"    '模拟回车键，跳过 Nag screen   
  
set WSHShell = nothing   
  
这样破过后，ATC 基本上能无限制使用了，不过，总归感觉歉完美。Cracker   
的终极目标是写出注册机，不是么：)。大概跟了跟程序，好象是出品该软件   
的公司专门为他们写的一系列程序做的压缩和软件保护，程序跟外壳配合的，   
好难，太费时间，算了。希望哪位大虾可以试试，呵呵。  4dd466d0e1e0cc123afbe504da2462ca