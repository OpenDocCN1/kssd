
# Delphi 制作注册机

> 原文：[`www.pediy.com/kssd/pediy05/pediy50654.htm`](https://www.pediy.com/kssd/pediy05/pediy50654.htm)

For Delphi，让你的注册机变小一些

请抛弃 VCL，如果你嫌自己用 Delphi 编译出来的注册机个头儿过大的话...

事实上这种事情确实在发生，就在今天，就在刚才，在偶还没吃饭的时候就看到一个网友在这样抱怨...

的确如此，KeyGen 的个头儿与它的界面一样让人失望，简单的 form 再来上两个 `Edit`，三个 `Button`。普普通通的界面有着 360K 这样的大体积，而上过壳后那 150K 的体积同样不能让人满意...

会造成这样的结果，与 VCL 是脱不了关系的，事实上你用 Delphi 编译一个什么也不做的程序也会有 351K，否则你以为为何用 Delphi 写程序是一件相当轻松的事情？

不过这样沉重的代价付到我们可爱的 KeyGen 上就有点儿太过残忍了，很难想像某天要给一个大小 200K 左右的软件写注册机时会是怎样一个情景...
![img/ef77583bd97289c54af1fb39f1d25f7c.jpg](img/ef77583bd97289c54af1fb39f1d25f7c.jpg)

来吧，让我们想点儿办法，目前有一些控件可以很好的完成这个工作，用了它们后你编译出来的可执行文件的个头儿要比平常小很多，不过我并不打算介绍它们
![img/1ddd2a0a311dc555b0cb6f154f64693a.jpg](img/1ddd2a0a311dc555b0cb6f154f64693a.jpg)

这并不是什么太高深的东西，相信也不会很难
![img/ef77583bd97289c54af1fb39f1d25f7c.jpg](img/ef77583bd97289c54af1fb39f1d25f7c.jpg)

扔掉了 VCL，我们能使用的东西就只有 Pascal 语言与 API 函数了，但实际上这就已经足够了:)

这之前先要找个软件做例子，大大大大大大大前天看到一个杂志在海吹敏思硬盘卫士，就 Down 了一个下来，后来才发现它对我没任何用处。呵呵，好老的一个软件了，一年多没更新了吧，估计注册机漫天飞的都有
![img/ef77583bd97289c54af1fb39f1d25f7c.jpg](img/ef77583bd97289c54af1fb39f1d25f7c.jpg)

先用 `TRW2000` 来摆平它，启动软件，我的机器码是 `yjeyj239416`，随便输入一个注册码，然后用 `hmemcpy` 做断点......

在 `004a4165` 处的时候，我们会发现程序会把正确的注册码装入 `edx` 中，比如这我边儿的是 `3887361024`，不过我们对它不感兴趣，在它之上的的 `004a4160` 处，才是我们真正感兴趣的地方，正确的注册码就是这个 `CALL` 来计算的。

按 `F8` 跟进去,来到 `004a3a08` 处:

```asm
0167:004a3a08  push     ebp  
0167:004a3a09  mov      ebp,esp  
0167:004a3a0b  push     byte +00  
0167:004a3a0d  push     byte +00  
0167:004a3a0f  push     byte +00  
0167:004a3a11  push     ebx  
0167:004a3a12  push     esi  
0167:004a3a13  mov      esi,ecx  
0167:004a3a15  mov      ebx,edx  
0167:004a3a17  mov      [ebp-04],eax  
0167:004a3a1a  mov      eax,[ebp-04]  
0167:004a3a1d  call     00404050  
0167:004a3a22  xor      eax,eax  
0167:004a3a24  push     ebp  
0167:004a3a25  push     dword 004a3a6e  
0167:004a3a2a  push     dword [fs:eax]               
0167:004a3a2d  mov      [fs:eax],esp                 <--挂 SEH，不关咱的事儿
0167:004a3a30  lea      edx,[ebp-0c]  
0167:004a3a33  mov      eax,[ebp-04]  
0167:004a3a36  call     004085a8                      <--将机器码转换为大写字符  
0167:004a3a3b  mov      eax,[ebp-0c]  
0167:004a3a3e  lea      ecx,[ebp-08]  
0167:004a3a41  movzx    edx,bx                        <--edx 装入 Dh，即十进制数 13  
0167:004a3a44  call     004a3958                      <--完成计算正确注册码的第一步  
0167:004a3a49  mov      edx,esi  
0167:004a3a4b  mov      eax,[ebp-08]  
0167:004a3a4e  call     004a3bdc                      <-完成计算正确注册码的第二步  
0167:004a3a53  xor      eax,eax  
0167:004a3a55  pop      edx  
0167:004a3a56  pop      ecx  
0167:004a3a57  pop      ecx  
0167:004a3a58  mov      [fs:eax],edx  
0167:004a3a5b  push     dword 004a3a75  
0167:004a3a60  lea      eax,[ebp-0c]  
0167:004a3a63  mov      edx,03  
0167:004a3a68  call     00403c40  
0167:004a3a6d  ret   
0167:004a3a6e  jmp      00403630  
0167:004a3a73  jmp      short 004a3a60  
0167:004a3a75  pop      esi  
0167:004a3a76  pop      ebx  
0167:004a3a77  mov      esp,ebp  
0167:004a3a79  pop      ebp  
0167:004a3a7a  ret
```

`004a3a44` 处的所谓计算注册码的第一步就是指先对机器码进行一些处理，我们按 `F8` 跟进去:

```asm
0167:004a3958  push     ebp  
0167:004a3959  mov      ebp,esp  
0167:004a395b  add      esp,byte -14  
0167:004a395e  push     ebx  
0167:004a395f  push     esi  
0167:004a3960  push     edi  
0167:004a3961  xor      ebx,ebx  
0167:004a3963  mov      [ebp-14],ebx  
0167:004a3966  mov      [ebp-0c],ecx  
0167:004a3969  mov      [ebp-08],edx  
0167:004a396c  mov      [ebp-04],eax  
0167:004a396f  mov      eax,[ebp-04]  
0167:004a3972  call     00404050  
0167:004a3977  xor      eax,eax  
0167:004a3979  push     ebp  
0167:004a397a  push     dword 004a39f9  
0167:004a397f  push     dword [fs:eax]  
0167:004a3982  mov      [fs:eax],esp  
0167:004a3985  mov      eax,[ebp-0c]  
0167:004a3988  call     00403c1c  
0167:004a398d  mov      eax,[ebp-08]  
0167:004a3990  add      eax,[ebp-08]  
0167:004a3993  mov      [ebp-10],eax  
0167:004a3996  mov      eax,[ebp-04]  
0167:004a3999  call     00403e9c                
0167:004a399e  mov      edi,eax                  <--eax 中此时装的是机器码的位数  
0167:004a39a0  test     edi,edi                  <--测试 edi  
0167:004a39a2  jng      004a39db                 <--为 0 就跳到 004a39db 处  
0167:004a39a4  mov      esi,01                   <--esi 置 1，用于后边儿的循环  
0167:004a39a9  mov      eax,[ebp-04]             <--机器码的地址装入 eax 中，此时的机器码已转换为大写字符  
0167:004a39ac  xor      ebx,ebx                  <--ebx 清 0  
0167:004a39ae  mov      bl,[eax+esi-01]          <--得到机器码  
0167:004a39b2  test     esi,01                   <--test esi,01 可以理解为测试 esi 中装的值是否为双数
0167:004a39b8  jnz      004a39bf                 <--不为双数就跳到 004a39bf 处  
0167:004a39ba  add      ebx,[ebp-08]             <--ebp-08 处装的是 Dh,也就是那个 13，用其加上 ebx 中的机器码的 ASCII 码  
0167:004a39bd  jmp      short 004a39c2           <--无条件跳转到 004a39bd 处  
0167:004a39bf  add      ebx,[ebp-10]             <--ebp-10 处装的是 1Ah,即十进制数 26,用其加上 ebx 中装的机器码的 ASCII 码  
0167:004a39c2  lea      eax,[ebp-14]            
0167:004a39c5  mov      edx,ebx  
0167:004a39c7  call     00403dc4  
0167:004a39cc  mov      edx,[ebp-14]  
0167:004a39cf  mov      eax,[ebp-0c]  
0167:004a39d2  call     00408594                 <--将得到的 ASCII 码与 26 或 13 相加后的结果保存起来  
0167:004a39d7  inc      esi                      <--esi 加上 1  
0167:004a39d8  dec      edi                      <--edi 减去 1  
0167:004a39d9  jnz      004a39a9                 <--edi 不为零，也就是机器码还没有全计算完，就跳回 004a39a9 处继续  
0167:004a39db  xor      eax,eax  
0167:004a39dd  pop      edx  
0167:004a39de  pop      ecx  
0167:004a39df  pop      ecx  
0167:004a39e0  mov      [fs:eax],edx  
0167:004a39e3  push     dword 004a3a00  
0167:004a39e8  lea      eax,[ebp-14]  
0167:004a39eb  call     00403c1c  
0167:004a39f0  lea      eax,[ebp-04]  
0167:004a39f3  call     00403c1c  
0167:004a39f8  ret   
0167:004a39f9  jmp      00403630  
0167:004a39fe  jmp      short 004a39e8  
0167:004a3a00  pop      edi  
0167:004a3a01  pop      esi  
0167:004a3a02  pop      ebx  
0167:004a3a03  mov      esp,ebp  
0167:004a3a05  pop      ebp  
0167:004a3a06  ret
```

这个 `CALL` 的作用呢，就是这样的：将机器码中的每个字符的 ASCII 码，加上 26 或 13（单位，如第 1、3、5 位加上 26，双位，如第 2、4、6 位加上 13），并将之保存起来，以便后面的应用。与我的机器对应的是:`sW_fd?MFN>P`

好的，我们继续，等到返回后，接着走两步，我们就会到达 `004a3a4e` 处，在这个 `CALL` 里，会完成计算注册码的最后一步，一起去看看吧:

```asm
0167:004a3bdc  push     ebp  
0167:004a3bdd  mov      ebp,esp  
0167:004a3bdf  add      esp,byte -10  
0167:004a3be2  push     ebx  
0167:004a3be3  mov      [ebp-08],edx  
0167:004a3be6  mov      [ebp-04],eax  
0167:004a3be9  mov      eax,[ebp-04]  
0167:004a3bec  call     00404050  
0167:004a3bf1  xor      eax,eax  
0167:004a3bf3  push     ebp  
0167:004a3bf4  push     dword 004a3c62  
0167:004a3bf9  push     dword [fs:eax]  
0167:004a3bfc  mov      [fs:eax],esp  
0167:004a3bff  mov      eax,[ebp-08]  
0167:004a3c02  call     00403c1c  
0167:004a3c07  mov      ebx,1e61                 <--ebx 装入 1e61h，即十进值数 7777  
0167:004a3c0c  mov      eax,[ebp-04]  
0167:004a3c0f  call     00403e9c  
0167:004a3c14  mov      edx,eax  
0167:004a3c16  test     edx,edx  
0167:004a3c18  jna      004a3c32  
0167:004a3c1a  mov      eax,01                   <--eax 中装入 1，用于下边儿的循环  
0167:004a3c1f  mov      ecx,[ebp-04]             <--ebp-04 中装的正是字符串 sW_fd?MFN>P 的地址  
0167:004a3c22  movzx    ecx,byte [ecx+eax-01]    <--ecx 中装入本轮用于计算的字符  
0167:004a3c27  add      ecx,eax                  <--ecx 加上 eax 的值  
0167:004a3c29  imul     ecx,ebx                  <--乘以 ebx，第一次的时候 ebx 中的值为前边装入的 7777  
0167:004a3c2c  mov      ebx,ecx                  <--结果装入 ebx 中  
0167:004a3c2e  inc      eax                      <--eax 加上 1，用于得到下一个字符  
0167:004a3c2f  dec      edx                      <--edx 减去 1  
0167:004a3c30  jnz      004a3c1f                 <--不为零就跳回去继续  
0167:004a3c32  mov      eax,[ebp-08]  
0167:004a3c35  push     eax  
0167:004a3c36  mov      [ebp-10],ebx
```



# 逆向分析与注册机实现

## 反汇编代码片段

```
0167:004a3c39  mov      byte [ebp-0c],00
0167:004a3c3d  lea      edx,[ebp-10]
0167:004a3c40  xor      ecx,ecx
0167:004a3c42  mov      eax,004a3c78
0167:004a3c47  call     004098ec
0167:004a3c4c  xor      eax,eax
0167:004a3c4e  pop      edx
0167:004a3c4f  pop      ecx
0167:004a3c50  pop      ecx
0167:004a3c51  mov      [fs:eax],edx
0167:004a3c54  push     dword 004a3c69
0167:004a3c59  lea      eax,[ebp-04]
0167:004a3c5c  call     00403c1c
0167:004a3c61  ret      
0167:004a3c62  jmp      00403630
0167:004a3c67  jmp      short 004a3c59
0167:004a3c69  pop      ebx
0167:004a3c6a  mov      esp,ebp
0167:004a3c6c  pop      ebp
0167:004a3c6d  ret     
```

嘿嘿，“平民算法”嘛 ![](img/ef77583bd97289c54af1fb39f1d25f7c.jpg)

现在我们手里所掌握的，已足够写出注册机了。通常情况下，我们可以声明这样一个函数用于计算正确的注册码：

```pascal
function  GetKey(Name: String): String;

var

  User,Serial:String;

  i:integer;

  ASC,jia,Zhi:Cardinal;

begin

  zhi:=7777;

  User:=UpperCase(Name);

  if Length(User)=0 then Result:='请输入您的机器码...'

  else

    begin

      for i:=1 to Length(User) do

        begin

          ASC:=Ord(User[i]);

          if i mod 2 =0 then ASC:=ASC+13

          else

            ASC:=ASC+26;

          ASC:=ASC+i;

          jia:=ASC*zhi;

          zhi:=jia;

        end;

      Serial:=inttostr(zhi);

      Result:=Serial;

    end;

end;
```

然后我们就可以在 `Delphi` 中通过 `VCL` 来增大我们 `KeyGen` 的体积了，但今天我们不准备这么做 ![](img/1ddd2a0a311dc555b0cb6f154f64693a.jpg)

## 创建项目

OK，启动你的 `Delphi`，然后新建一个应用程序，然后就在代码窗口中右键 `Unit1` 单元 -> `Close Page`，接着选 `No` 来把 `Unit1.Pas` 给删除掉。

接着选菜单 `View` -> `Units`，双击 `Project1` 打开它，我们就靠它吃饭了 ![](img/1ddd2a0a311dc555b0cb6f154f64693a.jpg) 随便找个文件夹把它保存一下，比如说保存为 `KeyGen.dpr`。

然后我们可以把它的内容给全部删掉，只留下三行：

```pascal
program KeyGen;

begin

end.
```

呵呵，按 `F9` 吧，你会什么也感觉不到，但事实上这已经是一个程序了。

如果你看一下所生成的 `KeyGen.exe` 的大小，你会发现，尽管我们什么也没有写，可它已经有了 8K 大。这是因为在任何 `DELPHI` 的目标程序中，都会自动包含 `System` 单元中的代码，如果你用 `Delphi5` 的话，它会是 16K。在这小小的 8K 中，包含了支撑整座 `DELPHI` 大厦的基石，VeryVery 牢靠。

以后随着我们代码的增加，程序的大小也会从 8K 开始慢慢跟着增加 ![](img/ef77583bd97289c54af1fb39f1d25f7c.jpg)

## 注册机与资源文件代码

让我们开始吧，我把注册机与资源文件的代码贴出来：

```pascal
{-<KeyGen.dpr>------------------------------------Code made >> Suunb[CCG]-----.}

(**** **** ****Lovely CCG,wish it becorning more prosperous every day!**** **** ****)

{---------------------------------------------------------Data:April 26th,2003.}

program KeyGen;

uses

  Windows,

  Messages;

{---------------------------------------------------------<引入相关单元>------;}

{----Windows.pas 里面包含了常量与 API 的定义,Messages.pas 中则包含了消息的定义----;}

{---这里要说一句的是，某些单元的引入，会增加可执行文件的大小,还好这两个不会^_-}

{$R KeyGen.res}     (*<引入资源文件，资源中包含了程序中用到的对话框等等...>*)

const

  IDI_CCG       =1 ;

  IDC_EDIT_CODE       =3001 ;

  IDC_EDIT_SERIAL     =3002 ;

  IDC_BUTTON_GENERATE =3003 ;

  IDC_BUTTON_ABOUT    =3004 ;

  IDC_BUTTON_EXIT     =3005 ;

  IDC_ABOUT_BUTTON_GREETING  =  3006 ;

  szDlgName            =       'KeyGenMain';

  szAboutDlg           =       'About';

{---------------------------------------------------------<以上为常量定义>----;}

function  GetKey(S: String): String;

var

  Code,Serial:String;

  i:integer;

  ASC,Q,P:Cardinal;

  NN:array [0..512] of Char;

begin

  P:=7777;

  Code:=S;

  if Length(Code)=0 then Result:='请输入您的机器码...'

  else

    begin

      CharUpperBuff(PChar(Code),Length(S));

      for i:=1 to Length(Code) do

        begin

          ASC:=Ord(Code[i]);

          if i mod 2 =0 then ASC:=ASC+13

          else

            ASC:=ASC+26;

          ASC:=ASC+i;

          Q:=ASC*P;

          P:=Q;

        end;

      if P>2147483647 then

        begin

          if (P>2147483647) and (P<3000000000)  then

            begin

              P:=P-2000000000;

              if P<100000000 then

                wvsprintf(NN,'0%d',@P)

              else

                wvsprintf(NN,'%d',@P);

              Serial:='2'+String(NN);

            end

          else if (P>3000000000) and (P<4000000000) then

            begin

              P:=P-3000000000;

              if P<100000000 then

                wvsprintf(NN,'0%d',@P)

              else

                wvsprintf(NN,'%d',@P);

              Serial:='3'+String(NN);

            end

          else if (P>4000000000) and (P<5000000000) then

            begin

              P:=P-4000000000;

              if P<100000000 then

                wvsprintf(NN,'0%d',@P)

              else

                wvsprintf(NN,'%d',@P);

              Serial:='4'+String(NN);

            end

        end

      else

        begin

          wvsprintf(NN,'%d',@P);

          Serial:=String(NN);

        end;

      Result:=Serial;

    end;

end;

{-------------------------------<上面的这个函数就是用来计算正确的注册码的>----;}

function AboutProc (Wnd: hWnd; Msg, wParam: Word; lParam: LongInt)

:LongInt; stdcall;

var

  ea:DWORD;

  aa:WORD;

begin

  Case Msg of

    WM_CLOSE:EndDialog(Wnd,0);

    WM_COMMAND:

      begin

        aa:=wParam;

        if aa=IDC_ABOUT_BUTTON_GREETING then

          EndDialog(Wnd,0);

          {-------<Greeting 被按下，窗口关闭>}

      end;

    else

      Result:=1;

  end;

  Result:=0;

end;

{-<上面的函数用于处理“关于”对话框接收到的消息，stdcall 表示其为一个回调函数>-;}

function DlgProc (Wnd: hWnd; Msg, wParam: Word; lParam: LongInt)

:LongInt; stdcall;

var

  Code:array[0..512] of Char;

  Serial:String;

  ea:DWORD;

  aa:WORD;

begin

  Case Msg of

    WM_CLOSE:

      EndDialog(Wnd,0);

    WM_INITDIALOG:

      begin

        ea:=LoadIcon(hInstance, PChar(IDI_CCG));

        SendMessage(Wnd,WM_SETICON, ICON_SMALL, ea);

        {------------------------<设置对话框的图标>}

      end;

    WM_COMMAND:

      begin

        aa:=wParam;

        if aa=IDC_BUTTON_ABOUT then

          DialogBoxParam(hInstance,szAboutDlg,Wnd, @AboutProc, 0)

          {------------------<About 被按下，就创建“关于”对话框>}

        else if aa=IDC_BUTTON_GENERATE then

          begin

            GetDlgItemText(Wnd,IDC_EDIT_CODE,Code,512);

            Serial:=GetKey(Code);

            SetDlgItemText(Wnd,IDC_EDIT_SERIAL,PChar(Serial));

            {<Generate 被按下,调用上面声明的 GetKey 函数来计算注册码>}

          end

        else if aa=IDC_BUTTON_EXIT then

          SendMessage(Wnd,WM_CLOSE, 0, 0);

          {------------<Exit 被按下，退出>}

      end;

    else

      Result:=1;

  end;

  Result:=0;

end;

{---------------------------------<主窗口的消息循环，同样为一个回调函数>------;}

begin

  DialogBoxParam(hInstance,szDlgName,0,@DlgProc,0);

  (*程序开始，调用 DialogBoxParam 来创建程序的主对话框，hInstance 是 Delphi 提供

    给我们的全局变量，不必再去调用 GetModuleHandle 函数*)

  ExitProcess(0);    {Over,调用 ExitProcess}

end.

//------------------------------------------------------------------------------

//------------------the  Shang Dynasty (16th―11th century B.C.)-----------------

//--------------------------------------------------------------------Code  end--
```

下面再把资源文件也贴出来：

```rc
//-KeyGen.rc----------------------------------------------------------------

//编译用 rc  KeyGen.rc

//-----------------------------------------------------------by Suunb[CCG]--

#include "resource.h"

#define CCG_ICO                 1

#define Su_ICO                2

#define IDC_EDIT_CODE         3001

#define IDC_EDIT_SERIAL       3002

#define IDC_BUTTON_GENERATE   3003

#define IDC_BUTTON_ABOUT      3004

#define IDC_BUTTON_EXIT       3005

#define IDC_ABOUT_BUTTON_GREETING    3006

CCG_ICO ICON "ccg.ico"              //CCG 的图标

Su_ICO  ICON "Suunb.ico"           //自己做的一个图标 ![](img/ef77583bd97289c54af1fb39f1d25f7c.jpg)

//-------------------------------------------<以上是一些常量的定义>---------

KeyGenMain 

 DIALOGEX 10, 10, 163, 94
```



```
style DS_CENTER | WS_MINIMIZEBOX | WS_VISIBLE | WS_CAPTION | WS_SYSMENU  

CAPTION "CCG KeyGen for 敏思硬盘卫士 v2.2"  

FONT 9, "Arial"  

BEGIN  

  GROUPBOX        "register",-1,5,3,151,44,BS_FLAT  

  LTEXT           "Code:",-1,10,14,23,10  

  EDITTEXT        IDC_EDIT_CODE,35,13,115,12,ES_AUTOHSCROLL,WS_EX_DLGMODALFRAME  

  LTEXT           "Serial:",-1,10,29,23,10  

  EDITTEXT        IDC_EDIT_SERIAL,35,29,115,12,ES_AUTOHSCROLL,WS_EX_DLGMODALFRAME  

  PUSHBUTTON      "gENERATE",IDC_BUTTON_GENERATE,13,54,39,11,BS_CENTER | BS_FLAT  

  PUSHBUTTON      "aBOUT",IDC_BUTTON_ABOUT,61,54,39,11,BS_CENTER | BS_FLAT  

  PUSHBUTTON      "eXIT",IDC_BUTTON_EXIT,108,54,39,11,BS_CENTER | BS_FLAT  

  LTEXT           "wish it becorning more prosperous every day!",-1,10,77,  

                  143,9  

  GROUPBOX        "Lovely CCG",-1,6,69,151,21,BS_CENTER | BS_FLAT  

END
```

`//-------------------------------<以上是程序的的主窗口对话框>---------`

About

```
 DIALOGEX 10, 10, 143, 141  

style DS_MODALFRAME | DS_CENTER | WS_POPUP | WS_VISIBLE | WS_CAPTION |   

  WS_SYSMENU  

CAPTION "The Shang Dynasty (16th - 11th century B.C.)"  

FONT 9, "Arial"  

BEGIN  

  ICON            1,-1,13,14,20,20  

  LTEXT           "Code made:",-1,25,3,41,8  

  LTEXT           "by",-1,40,13,8,8  

  LTEXT           "Date:April 26t,2003",-1,48,25,61,9  

  CONTROL         "",-1,"Static",SS_ETCHEDHORZ,4,36,132,1  

  LTEXT           "Personal Individual Greetings go to:",-1,19,41,114,8  

  LTEXT           "Sun Bird - JOJO - KANXUE",-1,11,51,85,9  

  LTEXT           "pll621 - zmworm - yyxzz",-1,46,62,73,9,SS_CENTERIMAGE  

  LTEXT           "Personal Group Greetings go to:",-1,22,87,105,8  

  LTEXT           "BCG FCG iPB DFCG DCM",-1,28,101,85,9,WS_BORDER  

  LTEXT           "Suunb[CCG]",-1,50,12,45,11,0,WS_EX_DLGMODALFRAME  

  LTEXT           "and other CCG guys...",-1,29,74,92,9,SS_CENTERIMAGE  

  ICON            2,-1,112,14,20,20  

  PUSHBUTTON      "gREETING",IDC_ABOUT_BUTTON_GREETING,50,120,41,12,BS_FLAT  

END
```

`//-------------------------------<这个是“关于”对话框>--------`

你可以用记事本将这两个文件分别保存为 `KeyGen.dpr`、`KeyGen.rc`，而后用 `rc KeyGen.rc` 来得到 `KeyGen.res`，之后将 `KeyGen.res` 与 `KeyGen.dpr` 放置同一目录下用 `Delphi` 编译 `KeyGen.dpr` 即可...

看一下编译后的 `KeyGen` 的大小吧，`20.5K`，再来个壳就只有 `10` 几 `K` 了，虽然界面并不怎么漂亮 ![](img/ef77583bd97289c54af1fb39f1d25f7c.jpg)

`OK`，就到这儿。

如果你不想放弃直接用可视控件，而又要追求 `30K` 的编译后程序，不妨试试 `delphi` 控件 `KOL`、`MCK`。

[`bonanzas.rinet.ru/`](http://bonanzas.rinet.ru/)

`4dd466d0e1e0cc123afbe504da2462ca`
