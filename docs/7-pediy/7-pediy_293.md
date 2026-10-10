

# 屏录专家 DIY

> 原文：[`www.pediy.com/kssd/pediy12/140103.html`](https://www.pediy.com/kssd/pediy12/140103.html)

【软件大小】: 2.34M  
 【下载地址】: 自己搜索下载  
 【加壳方式】: 无壳  
 【保护方式】: 注册码  
 【编写语言】: Borland C++ 1999  
 【使用工具】: PEID,OD,LordPE  
 【软件介绍】: 录制电脑屏幕、声音  
 【作者声明】: 只谈技术,共同提高,欢迎批评指正  
  
 在使用屏幕录像专家过程中，发现录制的视频默认的的文件名是“录像 1、2、3”，而我需要的是按录像开始时间命名的文件， 遂对“屏录专家”进行了改造。  
  
一、查壳：用 PEID 查了一下屏录专家的主文件，无壳，省了点事。  
  
二、去自校验：随便改了一下文件的资源，运行后提示文件被破坏，这显然是文件有校验，这里不给出详细去校验的过程，只给出怎么修改。自校验修改针对屏幕录像专家 V2011 Build0726 版本。  
  
![](img/7f0e608024b6add2ec0d5697a45fae3e.jpg)  
三、改造思路：为使视频文件用开始录制的时间命名，只要在开始录制时，能在如图红圈所示的地方自动写入当前电脑日期时间即可，为减小劳动量，我用 delphi xe 写了一个 dll，在合适的地方调用 dll 即可。  
![](img/ce5fcd07252776101c6f9d2b7003058a.jpg)  
四、dll 的编写：  
具体代码如下：  
library SetTime;  
  
uses  
ShareMem, SysUtils, Windows, Messages, Classes;  
function GetHwnd: THandle;  
const  
A\_szClassName: array \[0 .. 5\] of PChar = ('TMainForm', 'TPageControl', 'TTabSheet', 'TPageControl', 'TTabSheet', 'TEdit');  
A\_szWinName: array \[0 .. 5\] of PChar = ('屏幕录像专家 V2011', '', '录像模式               ', '', '基本设置', '');  
var  
i: Integer;  
hLastWin: THandle;  
t: TDateTime;  
begin  
  
hLastWin := FindWindow(A\_szClassName\[0\], A\_szWinName\[0\]); // 先取顶级父窗口句柄  
  
for i := 1 to 5 do  
   begin  
      hLastWin := FindWindowEx(hLastWin, THandle(nil), A\_szClassName\[i\], A\_szWinName\[i\]);  
   end;  
hLastWin := GetWindow(hLastWin, 2);  
Result := hLastWin;  
end;  
  
procedure MySetTime;stdcall;  
var  
txtHwnd: THandle;  
t: TDateTime;  
StrT: String;  
begin  
txtHwnd := GetHwnd;  
t := Now;  
StrT := DateTimeToStr(t);  
StrT := StringReplace(StrT, ':', '时', \[\]);//替换不能做文件名的几个字符串（下同）  
StrT := StringReplace(StrT, ':', '分', \[\]);  
  
StrT := StringReplace(StrT, '/', '-', \[rfReplaceAll, rfIgnoreCase\]);  
StrT := StringReplace(StrT, '\\', '-', \[rfReplaceAll, rfIgnoreCase\]);  
StrT := StringReplace(StrT, '\*', '-', \[rfReplaceAll, rfIgnoreCase\]);  
StrT := StringReplace(StrT, '|', '-', \[rfReplaceAll, rfIgnoreCase\]);  
StrT := StringReplace(StrT, '?', '-', \[rfReplaceAll, rfIgnoreCase\]);  
StrT := StringReplace(StrT, '<', '-', \[rfReplaceAll, rfIgnoreCase\]);  
StrT := StringReplace(StrT, '>', '-', \[rfReplaceAll, rfIgnoreCase\]);  
StrT := StringReplace(StrT, '"', '-', \[rfReplaceAll, rfIgnoreCase\]);  
StrT := StringReplace(StrT, ' ', '(', \[\]);  
StrT := StrT + '秒)';  
SendMessage(txtHwnd, WM\_SETTEXT, 0, LParam(StrT));  
end;  
  
exports  
MySetTime;  
{$R \*.res}  
  
begin  
  
end.  
五、主文件输入表用添加自己的 dll：  
我用的是 LordPE（用 PEditor 1.7 会出错，不知为何），如下图所示，很详细，目的就是让新手看完本文后可以独立完成改造。  
![](img/36205724e05d6d95be079506db8a52b8.jpg)  
六、在文件中调用 dll 中的函数（过程）：  
     完成以上所有工作以后，就可以用 OD 来调试了，用 OD 加载目标文件运行后，在如图所示的文件名的文本框中填入不能用来做文件名的字符串（我填的是“<>”），点击录像后提示“文件已经存在，换一个文件名。”这不是已存在的问题，是 Windows 不允许“<>”作为文件名。  
![](img/f3a9a9d6d97db512dfba16f65e9f9832.jpg)  
     根据提示，很容易来到下面的位置：  
004089F3 . FF8D E0FCFFFF dec dword ptr ss:\[ebp-320\]  
004089F9   E8 22DC1600 call 00576620----------------------// 改为 call \[686018\]，这是自已 dll 中的函数地址，在用 LordPE 添加函数引用时可以看到。  
004089FE   59 pop ecx-------------------------------------// NOP  
004089FF   84C9 test cl,cl--------------------------------// NOP  
00408A01   0F84 13010000 je 00408B1A  
00408A07 . 66:C785 D4FCFFFF 70>mov word ptr ss:\[ebp-32C\],170  
00408A10 . 8D85 38FFFFFF lea eax,dword ptr ss:\[ebp-C8\]  
00408A16 . E8 7D8FFFFF call 00401998  
00408A1B . 50 push eax  
00408A1C . FF85 E0FCFFFF inc dword ptr ss:\[ebp-320\]  
00408A22 . BA 843C5900 mov edx,00593C84 ; 已经存在，请换一个文件名  
00408A27 . 8D85 3CFFFFFF lea eax,dword ptr ss:\[ebp-C4\]  
00408A2D . E8 E6D81600 call 00576318  
00408A32 . FF85 E0FCFFFF inc dword ptr ss:\[ebp-320\]  
00408A38 . 8D95 3CFFFFFF lea edx,dword ptr ss:\[ebp-C4\]  
00408A3E . 52 push edx  
00408A3F . 6A 00 push 0  
00408A41 . 6A 00 push 0  
完成改造后，我们测试一下，生成的视频文件果然是开始录像的时间，达到我的目的。  
附件中是用到的 dll 下和本文。  
plzjDiy.rar