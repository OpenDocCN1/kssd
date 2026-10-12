
# FCG 注册机 SDK

> 原文：[`www.pediy.com/kssd/pediy06/pediy6828.htm`](https://www.pediy.com/kssd/pediy06/pediy6828.htm)

**【题目】** 用“破解调试”的方法修改注册机（SDK）功能——献给自由的 FCG 和所有 Cracker

因为我们 FCG 的“家”总是居无定所，只好发表在看雪这里了。

**【作者】** laoqian\[FCG\]

**【作者邮箱】** lao-qian@163.com

**【作者主页】** FCGchina.126.com

**【使用工具】** Delphi 7 Mini-Edition，Microsoft Visual C++ 6.0，ollydbg 1.09 ，W32DASM10，UltraEdit8.0

**【平台】** Win98se/2000/XP(有小 bug)

**【软件名称】** FCG 注册机 SDK

**【下载地址】** FCGchina.126.com

**【破解声明】** 我是一只小小鸟，偶得一点心得，愿与大家分享：）

------------------------------------------------------------------

**【内容】**

本人在做社区游戏伴侣 V1.11（联众，边锋游戏牌类记牌器，注册后全部功能可用，破文详见看雪论坛精华）的注册机时，采用了本组织 FCG 的注册机模板（FCG SDK 模板 1.5.1.126 for Delphi，you_known\[FCG\]开发）。在编制过程中，由于社区游戏伴侣 V1.11 需分别注册联众，边锋用户，而此模板只提供一个注册窗口，当时匆忙中只好写了 2 个注册机发布。某天无聊又无事，又很懒于破解软件（也主要因为是让那几个 CCG 的 crackme 搞得我头晕对 CRACk 几乎丧失信心），突发念头，把以前的 2 个注册机合并为一。

仔细看了看 you_known\[FCG\]注册机模板（Delphi 版 SDK），研究发现一些有趣的东东，不敢敝帚自珍，遂写下这篇拙文。

首先看到下面 you_known\[FCG\]的声明了吗？

```
///////////////////////////////////////////
  名称: FCG SDK 模板 1.5.1.126 for Delphi
  作者: Mystery(you_known[FCG])
  描述:
           没有使用 VCL，没有使用类，纯 API 写成，移植简单，对资源稍加修改
        即可轻易转为 Visual C,MASM,Borland C 等源程序。本源代码仅供 FCG
        成员内部使用，作为 FCG 成员，你可以自由修改此代码(不要移除本说
        明内容)，若你修改了其中的不足，请寄我一份(you_known@163.com)。
        严禁非法传播，或者提供给组织外部的人使用！！
        仅以此献给自由的[FCG]。
        版权: Copyright(C)2000-2003 Mystery Studio.
  ///////////////////////////////////////////        
```

`$$$$因此我不能发布全部代码！只发布我添加的相关代码。!!!!看不明白处，请见谅。`

或者你努力加入 FCG，你即可获得全部源代码！

愿自由的\[FCG\]发展壮大！！！自由万岁！！！

`************************`

说实话，在接触 Crack 以前，我基本不会编程！中学、大学学过一些 basic 和 fortrun，考完试，就还给老师了。工作后，也只是用计算机而已。而 10 年之后喜欢了 Crack，不知不觉就耳闻目睹，居然能看懂程序了，当然是关于破解的程序！因喜欢 Crack，而进而编程序，大有人在吧？^\_^

但这次是我第一次真正编程，虽然是在别人的基础上。此前，我甚至不会用 Delphi、Visual C+等，这次下来，基本通了一遍。感谢 Crack，此后我可以说我会编程了！！！

不说了，再说有灌水之嫌了。

`************************`

废话少说，言归正传：

`************************`

为什么叫 用“破解调试”的方法修改注册机功能？

因为我说过此前我不会编程，对 API 函数编程更是一窍不通！而这次我是用调试（类似设断点）的方法。

用过 Delphi7 的都知道，它有一项 Debug 功能，其中功能键和 ollydbg 差不多，`F9` 运行，`F8` 单步跟踪，`F5` 设断，...，其中还有 View CPU 调试窗口，虽不如 OD 全，但什么 `eax`，`edx` 等都在，甚至还有汇编窗口！这对于我来说，足够了，因为我有 FCG SDK 模板源代码！

你会说，为什么这么费劲呢？因为这个注册机体积小啊！编译后才几十 `kB`，你用自己编的试试，少说几百 `kB`！

### 第一步，首先来分析资源

SDK 压缩包中的说明： `res15.rc` 资源源文件，只能手工修改。

好，可我不会手工修改！怎么办？我要添加很多东东啊！

没办法，只好先编译再说！把资源源文件编译成 `res15.res`。

我记得某位大侠说过编译后的 res 文件已经与语言无关了，这时我们用 Visual C 就可以打开 `res15.res` 看看！列位看官说了，为什么不用 VC6 直接打开 `res15.rc`？这里的 `res15.rc` 是用 Delphi 处理的，VC6 不认，而 Delphi 却没有打开 rc 文件的功能，明白了？

打开后，你可以随便修改了，就这么简单！添加，删除，改风格...不要说你不会用 VC6，我也是第一次用！

但是重要的一点，不要随便改那些单元属性的 `ID`，添加更改的都要记住那些 `ID`，VC 很方便得到 `ID`。这是后话了。

### 第二步，来看看 `.dpr` 文件（Delphi 的源码）了，这是首先静态分析，和破解一样。

用 Delphi7 打开 `.dpr` 文件，看到如下（见后）：

我为了写方便，就不分别先后了，只是说说我的思路。以下代码是我修改过的。

一. 看到“全局变量、常量数据声明”，你不难理解，这是为了注册机编制方便。

二. 重点看到“资源常量定义”那里，发现什么，这些就是我上面说的各个单元的 `ID`！其中著名增加的是我添加的，就是我在第一步修改资源时得到的 `ID`！

三. 如果我们只是到这里，我们的工作几乎就可以完成了：

1. 我们添加 2 个注册码 `Edit` 框，改几个按钮值。
2. 增加几个变量，常量，再把里面的几个 `case of` 选项改改。
3. 添加注册机的子程序。
4. 编译完工。

这是地球人都会的！^\_^

### 第三步. 本文核心，动态调试！

到了这里，这里注册机基本已经做好了。

但是我要给他添加功能：

1. 输入用户名同时，可实时显示注册码。（如果实现好像很方便，也很...）
2. 增加几项超级链接效果。（只是表面文章而已）
3. 直接写入注册表，完成注册。（也是为了方便）

这个可没有源代码了，要自己写。重点是第一项难办，第二、三项好办。

#### 解决 1. 输入用户名，可实时显示注册码。

仔细观察，我们可以在 `//主对话框过程函数//` 发现臭名昭著的 window 消息函数 `WM_COMMAND`，它是什么地球人都知道！而关键就在那里。

我们在这个 `WM_COMMAND` 那里下面的“`case wParam of`”那行 `F5` 设断点。`F9` 运行，断点拦住了，但是 `F8` 按了好久，还没有进入程序。忽然想起来，这个 `WM_COMMAND` 消息函数是谁都用的，关掉断点。`F9` 进入注册机。

重新在那里 `F5` 设断点，点击任一按钮，断下。

这时你看一下 `wParam` 的值，就是那些按钮 `ID`！怎样看？Delphi7 提供的，鼠标在“`wParam`”上面停留。

明白了，就是这样简单，那按钮 `ID` 我们都看到了，点哪个执行那个 `ID` 选项！

此刻，添加 `case wParam of` 的选项，选项是 `ID` 为用户名的输入框（`Edit` 属性），...

这时我点击注册用户名的输入框（`Edit` 属性），马上中断，可是它却不执行我们添加的 `case of` 的选项！！也就是说，反复调试，查看 `wParam` 的值也不是那 `Edit` 框的 `ID`，而是乱七八糟的数！

我们进入 `Debug` 功能 `View CPU` 调试窗口，看看，发现 `eax` 的值明明是我们的 `ID`（显示 16 进制）啊？但是 `F8` 跟踪下去，就是不执行我们添加的 `ID` 项，检查程序也没错。

什么原因呢？找啊找，找到一个好朋友——

打开看雪的书第一版，找到 `WM_COMMAND` 函数的解释（手头只有这书），看到了 `Hiword(wParam)`，`Loword(wParam)`的说明，试试改为添加 `case Loword(wParam) of` 选项，OK！

我们点击注册用户名的输入框（`Edit` 属性），这次中断了！我找到你了

添加我们的代码，再说后面的那些都是多余了！

#### 解决 2. 增加几项超级链接效果

超级链接效果 SDK 里有子程序，在子程序里但是他是设的常量（如网站地址等），我要增加几项，常量就不行了。

为了利用他的子程序，当然，我也不会编新的！我们还是用上面的思路。

这次是 `Text` 属性框，设断，点击马上中断，可是无论在 `case Loword(wParam) of` 还是 `case wParam of` 里面，都没有执行我们的 `ID`！我不明白了。

只好把 `Text` 属性改成 `Edit` 属性，这样，就中断了，后面同上理。

自己看看我的更改代码，你会理解我的思路。

#### 解决 3. 直接写入注册表，完成注册。

我们利用它原来的注册按钮改造，添加自己的子程序，完成，不用调试断点了，代码自己看吧。

btw：如果不是写入注册表，而是写在其他地方，或者是 `keyfile` 等，你都可以如法炮制自己添加。

`////////////////////////////////////////////////////////////////////\*)`

```delphi
{$J+}

program Keygen;

uses Windows, Messages, ShellAPI;

{$R res15.res}

var
  ////////////////////
  //全局变量数据声明// ;不要修改全局变量！^_^明白了可以改！^_^
  ////////////////////
        szName:array[0..54] of char;   //输入用户名
        szNamebak:array[0..54] of char;   //输入用户名 新增
        szCode:array[0..54] of char;   //输出注册码
        namelz,namebf,snlz,snbf: PAnsiChar;   //输出用户名，注册码，写入注册表
        Reg_Nag:integer;    //联众边锋标志 ，加入标志是因为有 2 个注册机，写 2 个注册表！
        S_link: PAnsiChar;    //超链变量，增加！！
        flAgree:BOOL =       FALSE; //用户接受协议标志
        略....
        h_Icon: HICON;                 //实例句柄

//*lines 60***********以下可以自己修改******************************

const
  ////////////////
  //常量数据声明//
  ////////////////
  (*颜色设定*)
clBackground =       $000000 //背景颜色 
略....
  (*主对话框*)
              略....
        szBy ='http://www.ptcn.com';//软件开发者
        szCracker ='laoqian[FCG]';//解密人
        szCrackeremail ='mailto:lao-qian@163.com'; //解密人 email
        略....
        szLink ='http://FCGchina.126.com';//主页地址
        略....
  (*协议对话框*)
        szLicenseCaption='最终用户许可协议'; //协议对话框标题
        略....
  (*关于对话框*)
        szAboutCaption ='关于'; //关于对话框标题
        略....
      '〖模板修改〗'#10'by laoqian[FCG]'#10#10+
'版权所有(C)中国自由破解联盟';

//======================================================================
//这是社区游戏伴侣 V1.2 注册机源码，我以前已经发表过，只说说改正的.

//联众部分注册机程序，
//------------------------------------------------------------
procedure Calculatelz(A:PAnsiChar);
var
s1,s2,s3,s4 : string;
m,n,i,inc     :integer;
c1 : char;
begin
     s1:=A;
     m:=length(s1);
     n:=0;
     for i:=1 to m do
       begin
         c1:=s1[i];
         inc:= ord(c1);
         n:=n+inc*(i+122)+i;
       end;
     n:=n mod 10000;
     str(n,s2);
      case length(s2) of  //这里说明若不足 4 位前面补 0
       1: s2:='000'+s2;
       2: s2:='00'+s2;
       3: s2:='0'+s2;   
      end;               //
     s3:=s1+s2;
     m:=length(s3);
     n:=1246;
     for i:=1 to m do
       begin
         c1:=s3[i];
         inc:= ord(c1);
         n:=n+(i-1)*m+ inc*(m+i-1)+(i-1)*(i-1);
       end;
     n:=n mod 10000;
     if n<1000 then n:=n+6000;
     str(n,s4);
  lstrcpy(szCode,PAnsiChar(s4+S2));
end;

//--------------------------------------------------------

//////////////////////////////////////////////////////////////////////
//边锋部分注册机程序
procedure Calculatebf(A:PAnsiChar);
var
s1,s2,s3,s4 : string;
m,n,i,inc     :integer;
c1 : char;
begin
     s1:=A;
     m:=length(s1);
     n:=0;
     for i:=1 to m do
       begin
         c1:=s1[i];
         inc:= ord(c1);
         n:=n+inc*(i+255);
       end;
     n:=n mod 10000;
     str(n,s2);
      case length(s2) of      //这里说明若不足 4 位前面补 0
       1: s2:='000'+s2;
       2: s2:='00'+s2;
       3: s2:='0'+s2;        //   
      end;
     s3:=s1+s2;
     m:=length(s3);
     n:=3210;
```



```
for i:=1 to m do  
  begin  
    c1:=s3[i];  
    inc:= ord(c1);  
    n:=n+(i-1)*m+ inc*(m+i-1);  
  end;  
n:=n mod 10000;  
if n<10000 then n:=n+80000;  
str(n,s4);  

lstrcpy(szCode,PAnsiChar(s4+S2));  
end;  

//--------------------------------------------------------  

//////////////////////////////////////////////////////////////////////  
//写入注册表程序,这是我增加的直接写入注册表的代码  

procedure RegCreate(A,B:PAnsiChar;Reg_Nag:integer);  

var  
  cbDataB:integer;  
  hkey_reg:HKEY;  
begin  
   cbDataB:=length(B)+1;  //cbData 参数为保存键值数据的长度，好像 W2K，XP 有用。  
                  //键值数据类型为 reg_sz 时，键值数据字符串长度+1  
  case Reg_Nag of  
    1:  
      begin  
      //创建子键  
       RegCreateKey(HKEY_CURRENT_USER,'Software\zgsq\lzUser',hkey_reg);  
       //写入键值数据  
       RegSetValueEx(hkey_reg,A,0,reg_sz,B,cbDataB)  
       // 关闭  
       RegCloseKey(hkey_reg);  
      end;  
    2:  
      begin  
        RegCreateKey(HKEY_CURRENT_USER,'Software\zgsq\bfUser',hkey_reg);  
        RegSetValueEx(hkey_reg,A,0,reg_sz,B,cbDataB);  
        RegCloseKey(hkey_reg);  
      end;  

  end;  

end;  
```

`//*lines 130*******以上可以自己修改*************************************************************************//`

```
const  
  //////////////////  
  //资源常量定义// ;不要修改！  ^_^明白了可以改！^_^  
  //////////////////  
        MAINICON             =           100   
        略......  
        IDD_MAINDLG          =           2000 
        MAIN_DO1             =          2001     //增加  
        MAIN_EXIT            =           2002 
         
        MAIN_SOFT            =           2005 
        MAIN_BY              =           2006 
        MAIN_CRACKER         =           2007 
        MAIN_DATE            =           2008 
        MAIN_CODE1           =           2009     //增加  
        MAIN_NAME1           =           2010     //增加  
        MAIN_LINK            =           2011 
        MAIN_COPY            =           2012 
        MAIN_CODE2           =           2013     //增加  
        MAIN_NAME2           =           2014     //增加  
        MAIN_DO2             =           2015     //增加  
         略......  
//////////////////////////////////////////////////////////////  
//计算字符串行数函数  

 略......  
//////////////////////////////////////////////////////////////  
//绘制标题栏函数  

 略......  
//////////////////////////////////////////////////////////////  
//绘制按钮函数  

 略......  
//////////////////////////////////////////////////////////////////  
//动态显示窗体函数  

 略......  
//////////////////////////////////////////////////////////////////  
//超链接过程函数  

function LinkProc(h_Wnd: HWND; Msg, wParam, lParam: DWORD): LRESULT; stdcall;  

var  
  l: POINTER;  

begin  
  Result := 1;  
   case Msg of  
     WM_LBUTTONUP:  
       //鼠标左键点击后打开浏览器，定位到主页地址  
       ShellExecute(0, nil, S_Link, nil, nil, 0);     //原来 S_Link 是常量，我们改为变量  
    WM_NCHITTEST:  
      //将 WM_NCHITTEST 返回 TRUE 可以接收鼠标动作，实现按下功能 !  
      Result := 1;  
    WM_SETCURSOR:  
      //更改鼠标移动时的样式  
      SetCursor(h_Cur);  
    else  
      begin  
        l :=pointer(GetWindowLong(h_Wnd, GWL_USERDATA));  
        CallWindowProc(l, h_Wnd, Msg, wParam, lParam);  
      end;  

  end;  

end;  

//////////////////////////////////////////////////////////////////  
//字幕滚动过程函数  

 略......  
//////////////////////////////////////////////////////////////////  
//关于对话框过程函数  

略......  
//////////////////////////////////////////////////////////////////  
//主对话框过程函数  

function MainProc(hDlg: HWND; Msg, wParam, lParam: DWORD): LRESULT; stdcall;  

const  
  rcCaption: TRECT = ();  

var  
  略....  
  h_Link,h_by,h_CRACKER: HWND;   //增加  
  略....  
  byUser,CRACKERUser: integer;  //增加  

begin  
  Result := 0;  
  case Msg of  
    WM_INITDIALOG:  
      begin  
        //计算标题栏矩形区域大小  
        略....  
        SetDlgItemText(hDlg, MAIN_LINK, szLink);//显示主页链接  
        SetDlgItemText(hDlg, MAIN_DATE, szDate);//显示发布日期  
        SetDlgItemText(hDlg, MAIN_SOFT, szSoft);//显示软件名  
        SetDlgItemText(hDlg, MAIN_BY, szBy);//显示软件开发者  
        SetDlgItemText(hDlg, MAIN_CRACKER, szCracker);//显示解密人  
        略....  
        
        //建立并设定超链接字体  
        h_LinkFont := CreateFont(略....);  
      //显示 FCG 主页超链  
        h_Link := GetDlgItem(hDlg, MAIN_LINK);  
        SendMessage(h_Link, WM_SETFONT, h_LinkFont, 0);  
         
        //为其设定单独的消息处理过程，实现超链接效果  
        lUser := SetWindowLong(h_Link, GWL_WNDPROC, integer(@LinkProc));  
        SetWindowLong(h_Link, GWL_USERDATA, lUser);  
        AnimateShow(hDlg);  
      //显示软件开发者超链  
        h_by := GetDlgItem(hDlg, MAIN_by);  
        SendMessage(h_by, WM_SETFONT, h_LinkFont, 0);  
         
        //为其设定单独的消息处理过程，实现超链接效果  
        byUser := SetWindowLong(h_by, GWL_WNDPROC, integer(@LinkProc));  
        SetWindowLong(h_by, GWL_USERDATA, byUser);  
        AnimateShow(hDlg);  
      //显示解密人超链  
        h_CRACKER := GetDlgItem(hDlg, MAIN_CRACKER);  
        SendMessage(h_CRACKER, WM_SETFONT, h_LinkFont, 0);  
         
        //为其设定单独的消息处理过程，实现超链接效果  
        CRACKERUser := SetWindowLong(h_CRACKER, GWL_WNDPROC, integer(@LinkProc));  
        SetWindowLong(h_CRACKER, GWL_USERDATA, CRACKERUser);  
        AnimateShow(hDlg);  
      end;  
    WM_LBUTTONDOWN:  
      begin  
        //响应鼠标左键按下消息，若在标题栏内则使窗体移动  
        略....  
      end;  
    WM_PAINT:  
      begin  
        //响应绘制消息，绘制标题栏  
        略....  
      end;  
    WM_COMMAND:       //著名的 window 消息函数  
      begin  
        case wParam of     //我们可以在这里下断点！！！！  
          MAIN_DO1:      //声明的 ID（常量）选项，以下很多同  
            begin  
              //产生写注册码按钮按下  
              //获取输入名  
              GetDlgItemText(hDlg, MAIN_NAME1, szName, 55);  
              
              namelz:=szName;  
              
              if (length(namelz)>0) and  (length(namelz)<54) then  
                begin  
                  //计算注册码  
                  Calculatelz(szName);  
                  snlz:=szCode;  
                  Reg_Nag:=1;   //计算注册码联众标志，加入标志是因为有 2 个注册机，写 2 个注册表！  
                  if szName<>szNamebak  then  
                  //写注册表  
                    RegCreate(namelz,snlz,Reg_Nag);  
                end;  
            end;  
          MAIN_DO2:  
            begin  
              //产生写注册码按钮按下  
              //获取输入名  
              GetDlgItemText(hDlg, MAIN_NAME2, szName, 55);  
              
              namebf:=szName;  
              
              if (length(namebf)>0) and  (length(namebf)<54) then  
                begin  
                   //计算注册码  
                   Calculatebf(szName);  
                   namebf:=szName;  
                   snbf:=szCode;  
                   Reg_Nag:=2;    //计算注册码边锋标志  
                   if szName<>szNamebak  then  
                   //写注册表  
                    RegCreate(namebf,snbf,Reg_Nag);  
                end;  
            end;  

////////////////////////////////////////  
//原来模板的是这样的：  

//          MAIN_DO:  
//            begin  
//              //产生注册码按钮按下  
//              //获取输入名  
//              GetDlgItemText(hDlg, MAIN_NAME, szName, 255);  
//             //计算注册码子程序  
//              Calculate(szName);  
//              //显示注册码  
//              SetDlgItemText(hDlg, MAIN_CODE, szCode);  
 //           end;  
 ////////////////////////////////////////////////////          
          MAIN_about :  
            begin  
              //显示关于对话框  
              略....  
            end;  
          MAIN_EXIT:  
            EndDialog(hDlg, 0);  
          MAIN_CLOSE:  
            EndDialog(hDlg, 0);  

        end;  

///////////  
//重点添加//  
//  下面  //  
///////////          
        case Loword(wParam) of    //我们的断点在这里，哈哈，不容易啊！  
```



```pascal
MAIN_NAME1:     //我添加的声明的 ID（常量）选项，以下同
    begin
      //获取联众输入名框内动作
      GetDlgItemText(hDlg, MAIN_NAME1, szName, 55);
      namelz := szName;
      if (length(namelz) > 0) and (length(namelz) < 54) then //设定上限，防止爆机！
      begin
        if szName <> szNamebak then
          //你看 you_known[FCG]的天才构思，设定 szName，szCode 为数组变量为我们提供了巨大方便！！
          //这样我们实现实时注册码显示，简直几乎易如反掌！
          //不只这一个地方！我不得不佩服他的才能，不愧是 FCG 第一编程高手！
        begin
          //计算
          Calculatelz(szName);
          //显示注册码
          SetDlgItemText(hDlg, MAIN_CODE1, szCode);
        end;
      end
      //提示信息
      else SetDlgItemText(hDlg, MAIN_CODE1, '请在上面输入联众用户 ID，小于 55 位！');
    end;

MAIN_NAME2:
    begin
      //获取边锋输入名框内动作
      GetDlgItemText(hDlg, MAIN_NAME2, szName, 55);
      namebf := szName;
      if (length(namebf) > 0) and (length(namebf) < 54) then
      begin
        if szName <> szNamebak then
        begin
          //计算
          Calculatebf(szName);
          //显示注册码
          SetDlgItemText(hDlg, MAIN_CODE2, szCode);
        end
      end
      //提示信息
      else SetDlgItemText(hDlg, MAIN_CODE2, '请在上面输入边锋用户 ID，小于 55 位！');
    end;

MAIN_link:       //链接的 EDIT 框的 ID
    S_link := szLink;   //点击哪里，就赋值对应网址！

MAIN_By:
    S_link := szBy;

MAIN_Cracker:
    S_link := szCrackeremail;

///////////
// 上面   //
//重点添加//
/////////

    end;
    Result := 0;
end;

WM_DRAWITEM:
  begin
    略....
  end;

///////////////////////////////////////////////////
//响应绘制窗体内容消息
略....
else
  Result := 0;

end;
end;

//////////////////////////////////////////////////////////////////
//协议对话框过程函数
略......

//////////////////////////////////////////////////////////////////
//程序入口函数
begin
  略....
  //显示协议对话框
  DialogBox(h_Inst, LPCTSTR(IDD_LICENSEDLG), 0, @LicenseProc);
  if (flAgree) then //如果用户同意协议则显示主对话框
    DialogBox(h_Inst, LPCTSTR(IDD_MAINDLG), 0, @MainProc); // 主程序
  略....
  //退出程序
  ExitProcess(0);
end.
```

---

### 【\* \* 总结】

感谢 `you_known[FCG]` 提供了这么好的一个程序模板，让我们受益匪浅。

你看 `you_known[FCG]` 的天才构思，设定 `szName`，`szCode` 为数组变量为我们提供了巨大方便！！这样我们实现实时注册码显示，简直几乎易如反掌！还有超链接过程函数，堪称巧妙的构思，至于我不得不隐去的那些函数更是妙不可言，不只这一个地方！我不得不佩服他的才能，不愧是 FCG 第一编程高手！

如果你理解了我上面的，再加上你对 API 编程的了解，你已经可以自己编一个简单的实时显示注册码的注册机！只是界面原始一些罢了。如果不是写入注册表，而是写在其他地方，或者是 keyfile，你都可以如法炮制自己添加。你还能移植到 `MASM`，`VC` 等上，其实 `ASM` 代码已经在 `delphi` 里可以看到，只是我不会编程，如果你完成了或者有好的想法，给我寄一份（`lao-qian@163.com`）。

起初是看到国外的一些注册机可以输入用户名同时，可实时显示注册码，觉得很酷，就想知道他是怎样实现的，但大都是 `ASM` 的源码，功力太浅看不懂，因此这次钻研了一番，主要得益于 `you_known[FCG]` 的无私奉献。

完工之后，还有一些体会或许对大家有益：这些对于破解也有新的思路，比如有些软件注册是在你输入用户名同时，实时计算注册码，并对比结果，他的原理应该大同小异，一般都会调用 `window` 消息函数 `WM_COMMAND`，那我们找断点岂不是可以在这里呢？而对于编制 `CRackme` 或软件注册是否就更有用处呢？你可以改造一番试试。

### 【\* \* 闲话】

我玩 `Crack` 也 3 年了，起初只是在看雪论坛看看，今天看看，自己居然也在里面有了那么 8、9 篇文章入选了精华，也认识了不少朋友，最重要是学到了不少东西。由于日后工作将繁忙，又最近特别是渴望买车，更是抓紧挣钱，对 `Crack` 渐渐会接触的少了，只是偶尔有朋友要求帮忙才破几个小东东。这很可能是我最后一篇破文了，也是我给 `FCG` 和大家的礼物，当然我还会关注 `Crack`，因为我们热爱 `Free`！

有时，我也特别想加入 `CCG`，`BCG` 等，除了功力、时间不够，还有就是觉得：`FCG` 虽然组织松散，“家”也居无定所，但 `Free`！因为热爱 `Free` 的人，好像都很友善乐于助人！^_^，因此在 `FCG` 里很快乐，希望它的宗旨继续发扬光大！

不过，如果 `CCG`，`BCG` 的老大们看了这篇文章，能破格批准我加入，我也会很欣然的，哈哈，...

最后，感谢看雪为我们提供这个地方，我以有此同济师弟为荣！！！

btw：只是那些框框颜色不好，不知道怎样改。还有注册机在 `XP` 里的显示有些小问题无法解决。

看来只有请 `you_known` 解决了。

---

### 【版权声明】 本文纯属技术交流, 转载请注明作者并保持文章的完整, 谢谢!

```text
                    \\\|///
                   \\ - - //
                   (  @ @  )
 +-----------------------o00o-(_)-o00o--------------------+
 |                                                          |
 |        献给我喜欢游戏的海,我相信海永远是蓝的            |
 |                  laoqian[FCG]                          |
 +--------------------------------0ooo--------------------+
                    ooo0  (   )
                  (   )   ) /
                    \ (   (\_/
                     \\\_)
```
