

# 注册机写作教学

> 原文：[`www.pediy.com/kssd/pediy04/kanxue156.htm`](https://www.pediy.com/kssd/pediy04/kanxue156.htm)

前些天在论坛看到有人问软件的算法是如何看懂的 ？？注册机是如何写出来的？？有感于目前看雪教学中没有这方面的内容(对于不是明码比较的注册机工具就都没有办法了）。于是这篇文章出炉了.由于我对编程方面的技术实在是那个。所以拉来了 halfbit(云观）老哥。让他帮忙写 ASM，VC 的注册机。我自己来写 VB 的。   
本文的目的是通过一个简单的 CRACKME 的破解来说一下算法的分析和注册机的写作（针对于 NAME/SERIAL 保护的）。奉献给想加入 BCG 和 CCG 的破解爱好者。我尽量写的详细点。如有不明之处请发信到 llmen@sina.com   
工具：调试器一个（我用的是 TRW2000）。编程工具一个。iceClock。   
  
技术要求：熟悉基本的汇编指令。调试器的运用。编程基础(什么 ？？你说你不会。。。。那就去找一本书了。一般看完前 4 章就可以了。由于能力原因只给出了 VB，VC，ASM 的注册机原代码。）   
  
目标：http://www.cracknow.com/crackme/figugegl.1.zip    2KB。伪装者那里的。加了自己写的壳，入口点是 0040147B。   
选目标可让我费了些工夫。一开始 halfbit 让我找个软件，可是我没找到简单点的。才想到 CRACKME，又小又简单。可是手头只有 CCG 和 BCG 的考试 CRACKME。不过为了生命安全着想我还是用别的吧。：）   
废话说完了，现在我们开始。   
  
启动 TRW2000 和 CRACKME。输入姓名：liuliu,注册码：98765,用 HMEMCPY 做断点。点下 CHECK。断下来，用 PMODULE 回到程序领空。跟踪的时候要注意寄存器 EAX，EBX，ECX，EDX，ESI，EDI 的变化。用 D 命令查看内容。   
:0040132B E80C020000              call 0040153C                                  
:00401330 89C6                    mov esi, eax                                  我们在这里，   
:00401332 83FE05                  cmp esi, 00000005                            比较姓名长度是不是小于 5 。   
这里看看 ESI 里的数值。是 6。我的姓名输入是 liuliu。长度是 6。所以是比较长度。你问为什么知道是姓名长度 ？？经验啦。你不知道是比较什么可以改变一下 NAME 或 SERIAL 的长度看看。      
:00401335 7D04                    jge 0040133B                                  不小就跳   
:00401337 31C0                    xor eax, eax   
:00401339 EB44                    jmp 0040137F                                  小于就挂。   
:0040133B 6A14                    push 00000014   
:0040133D 8D45D6                  lea eax, dword ptr \[ebp\-2A\]   
:00401340 50                      push eax                
:00401341 6A66                    push 00000066   
:00401343 53                      push ebx   
:00401344 E8F3010000              call 0040153C                                这个是取 SERIAL 的长度。   
:00401349 09C0                    or eax, eax                                  看看 EAX 是不是 0，也就是是不是注册码没有输入。   
:0040134B 7432                    je 0040137F                                  是 0 就挂   
:0040134D 31FF                    xor edi, edi                                  EDI 变 0，用来做计数器。   
:0040134F EB15                    jmp 00401366                                  跳到算法开始。   
:00401351 0FBE443DD6              movsx eax, byte ptr \[ebp+edi\-2A\]              这是姓名第一个字符入 EAX。看看 EAX 里的数值。是 6C。l 的 ASCII 码就是 6C。所以我找到地方了。：）   
:00401356 0FBE543DEB              movsx edx, byte ptr \[ebp+edi\-15\]              这里是注册码第一个字符入 EDX。看看 EDX 的数值。是 39。9 的 ASCII 码就是 39。   
:0040135B 29FA                    sub edx, edi                                  用注册码第一个字符的 ASCII 码减去 EDI(现在为零)。EDX=69。   
:0040135D 39D0                    cmp eax, edx                                  比较 EAX 和 EDX 是否相等。当然不等了 EAX=6C，EDX=39。   
:0040135F 7404                    je 00401365                                  相等就继续。跳(这里用 r fl z 强行跳过。）   
:00401361 31C0                    xor eax, eax   
:00401363 EB1A                    jmp 0040137F                                  不等就挂。   
    
:00401365 47                      inc edi                                      计数器 EDI 加 1。   
  
:00401366 39F7                    cmp edi, esi                                  比较 EDI 是否等于 ESI（姓名长度）,也就是注册码是否计算完毕。   
:00401368 7CE7                    jl 00401351                                  相等就结束。不等继续。到这里第一次循环结束。第一位注册码验证完毕。在 0040135D 这里是关键的比较。以后的几次循环算法是一样的，只是数值不同。算法弄的差不多了。后面总结一下。   
:0040136A 6A00                    push 00000000   
:0040136C 6848204000              push 00402048   
:00401371 6850204000              push 00402050   
:00401376 53                      push ebx   
:00401377 E8E4010000              call 00401560                                注册成功！   
算法总结：程序的流程是这样子的   
1，比较输入姓名的长度是不是小于 5。   
2，比较注册码是不是没有输入。   
3，算法开始，第 1 位注册码要等于第 1 位姓名的 ASCII 码 6C-0（EDI 的值）。注册码第 1 位为 l(ASCII 码为 6C）   
            第 2 位注册码要等于第 2 位姓名的 ASCII 码 69-1。注册码第 2 位为 h   
            第 3 位注册码要等于第 3 位姓名的 ASCII 码 75-2。注册码第 3 位为 s。依次类推。   
            NAME：liuliu                              SERIAL：lhsiep   
到这里算法分析完毕。注册码也出来了。程序计算的流程也弄明白了。用你熟悉的编程语言来模仿一下程序作个注册机。   
VB 的注册机。   
先介绍一下里面用到的一般书上没有的函数。我用的是 VB6。0 企业版。   
LEN() 返回字符串的长度。   
ASC（）返回字符的 ASCII 码，VB 返回的是 10 进制的。   
MID（）返回参数指定的字符。用法：MID（目标字符串，开始的位数，取字符的个数）   
STRING（）把数字转换为字符。如：A=STRING（76）此时 A=“L”。   
启动 VB6，新建一个工程。在 FORM 上加两个 LABLE 控件。然后加两个 TEXTBOX 控件。一个 COMMANDBUTTON 控件。   
TEXT1。TEXT 用来获取 NAME，TEXT2。TEXT 用来输出 SERIAL。   
在 COMMAND1 控件的 CLICK 事件里如下代码：   
Private Sub Command1\_Click()   
Dim a, b, c, d, e                '变量声明。VB 不用特别声明变量的类型。多简单。：）   
text2.text="姓名不要小于 5 位"        
a = Text1.Text                  'a=输入的姓名   
b = Len(a)                      'b=姓名的长度   
c = 1                          '设一个计数器，类似于程序中的 EDI。   
If b < 5 Then GoTo a:          '判断姓名长度。小于 5 就什么也不做。   
Do                              ’循环语句，用来计算注册码。   
d = Mid(a, c, 1)                  
d = Asc(d)   
d = d + 1 - c   
d = String(1, d)   
e = e & d   
c = c + 1   
Loop While c <= b   
Text2.Text = e                  '输出注册码。            
a:   
End Sub    
然后用 P-CODE 编译。才 12KB。   
VC 的注册机   
先用 APPWIZARD 做一个默认的 DIALOG BASED。然后删掉上面的东西。加入两个 EDITBOX，一个 BUTTON。   
EDITBOX1 用来获得 NAME，EDITBOX2 用来输出 SERIAL。   
用 CLASSWIZARD 加入 EDITBOX1 的 MEMBER m\_name.EDITBOX2 的 MEMBER m\_key.   
用 CLASSWIZARD 在 BUTTON1 的 BN\_CLICKED 事件加个函数 OnButton1()   
加入如下代码：   
void CLLDlg::OnButton1()   
{   
    // TODO: Add your control notification handler code here    
    UpdateData(TRUE);   
    m\_key        = "";   
    char \* pName= m\_name.GetBuffer(0);   
  
    if(m\_name.GetLength() < 5)MessageBox("name 必须大于 5 个字母", "error", MB\_OK);   
    else{   
        for(int n=0;n<=m\_name.GetLength();n++)   
        {   
            m\_key += \*(pName+n)-n;   
        }   
        UpdateData(FALSE);   
    };   
    m\_name.ReleaseBuffer();   
       
}   
用 RELEASE 编译。   
下面是汇编的。我实在是没能力加上步骤。。。请参考 iczelion 的 win32asm 教程。   
; ml /c /coff /Cp LL.asm   
; link /SUBSYSTEM:WINDOWS /LIBPATH:d:\\masm32\\lib LL   
.386   
.model flat,stdcall   
option casemap:none   
WinMain proto :DWORD,:DWORD,:DWORD,:DWORD   
  
include d:\\masm32\\include\\windows.inc   
include d:\\masm32\\include\\user32.inc   
include d:\\masm32\\include\\kernel32.inc   
includelib d:\\masm32\\lib\\user32.lib   
includelib d:\\masm32\\lib\\kernel32.lib   
  
.data   
ClassName db "SimpleWinClass",0   
AppName  db "LL asm",0   
MenuName db "FirstMenu",0   
ButtonClassName db "button",0   
ButtonText db "get key",0   
EditClassName db "edit",0   
TestString db "Wow! I'm in an edit box now",0   
name5        db "name 必须大于 5 个字母",0    ;LL add   
msgError    db "error",0            ;LL add   
  
.data?   
hInstance HINSTANCE ?   
CommandLine LPSTR ?   
hwndButton HWND ?   
hwndEdit HWND ?   
keyBuffer db 100 dup(?)        ;LL add   
buffer db 512 dup(?)                    ; buffer to store the text retrieved from the edit box   
  
.const   
ButtonID equ 1                                ; The control ID of the button control   
EditID equ 2                                    ; The control ID of the edit control   
IDM\_HELLO equ 1   
IDM\_CLEAR equ 2   
IDM\_GETTEXT equ 3   
IDM\_EXIT equ 4   
  
.code   
start:   
    invoke GetModuleHandle, NULL   
    mov    hInstance,eax   
    invoke GetCommandLine   
    mov CommandLine,eax   
    invoke WinMain, hInstance,NULL,CommandLine, SW\_SHOWDEFAULT   
    invoke ExitProcess,eax   
  
WinMain proc hInst:HINSTANCE,hPrevInst:HINSTANCE,CmdLine:LPSTR,CmdShow:DWORD   
    LOCAL wc:WNDCLASSEX   
    LOCAL msg:MSG   
    LOCAL hwnd:HWND   
    mov  wc.cbSize,SIZEOF WNDCLASSEX   
    mov  wc.style, CS\_HREDRAW or CS\_VREDRAW   
    mov  wc.lpfnWndProc, OFFSET WndProc   
    mov  wc.cbClsExtra,NULL   
    mov  wc.cbWndExtra,NULL   
    push  hInst   
    pop  wc.hInstance   
    mov  wc.hbrBackground,COLOR\_BTNFACE+1   
    mov  wc.lpszMenuName,OFFSET MenuName   
    mov  wc.lpszClassName,OFFSET ClassName   
    invoke LoadIcon,NULL,IDI\_APPLICATION   
    mov  wc.hIcon,eax   
    mov  wc.hIconSm,eax   
    invoke LoadCursor,NULL,IDC\_ARROW   
    mov  wc.hCursor,eax   
    invoke RegisterClassEx, addr wc   
    invoke CreateWindowEx,WS\_EX\_CLIENTEDGE,ADDR ClassName, \\   
                        ADDR AppName, WS\_OVERLAPPEDWINDOW,\\   
                        CW\_USEDEFAULT, CW\_USEDEFAULT,\\   
                        300,200,NULL,NULL, hInst,NULL   
    mov  hwnd,eax   
    invoke ShowWindow, hwnd,SW\_SHOWNORMAL   
    invoke UpdateWindow, hwnd   
    .WHILE TRUE   
        invoke GetMessage, ADDR msg,NULL,0,0   
        .BREAK .IF (!eax)   
        invoke TranslateMessage, ADDR msg   
        invoke DispatchMessage, ADDR msg   
    .ENDW   
    mov    eax,msg.wParam   
    ret   
WinMain endp   
  
WndProc proc hWnd:HWND, uMsg:UINT, wParam:WPARAM, lParam:LPARAM   
    .IF uMsg==WM\_DESTROY   
        invoke PostQuitMessage,NULL   
    .ELSEIF uMsg==WM\_CREATE   
        invoke CreateWindowEx,WS\_EX\_CLIENTEDGE, ADDR EditClassName,NULL,\\   
                        WS\_CHILD or WS\_VISIBLE or WS\_BORDER or ES\_LEFT or\\   
                        ES\_AUTOHSCROLL,\\   
                        50,35,200,25,hWnd,8,hInstance,NULL   
        mov  hwndEdit,eax   
        invoke SetFocus, hwndEdit   
        invoke CreateWindowEx,NULL, ADDR ButtonClassName,ADDR ButtonText,\\   
                        WS\_CHILD or WS\_VISIBLE or BS\_DEFPUSHBUTTON,\\   
                        75,70,140,25,hWnd,ButtonID,hInstance,NULL   
        mov  hwndButton,eax   
    .ELSEIF uMsg==WM\_COMMAND   
        mov eax,wParam   
        .IF lParam==0   
            .IF ax\==IDM\_HELLO   
                invoke SetWindowText,hwndEdit,ADDR TestString   
            .ELSEIF ax\==IDM\_CLEAR   
                invoke SetWindowText,hwndEdit,NULL   
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*   
            .ELSEIF  ax\==IDM\_GETTEXT   
                invoke GetWindowText,hwndEdit,ADDR buffer,512    ;获得 name 串   
        mov    ebx,0   
        .WHILE byte ptr \[buffer+ebx\]!=0   
            mov    al,\[buffer+ebx\]   
            sub    al,bl   
            mov    \[keyBuffer+ebx\],al            ;变换后放入 key 串   
            inc    ebx;   
        .ENDW   
        .IF ebx<5   
            invoke MessageBox,NULL,offset name5, offset msgError,MB\_OK   
        .ELSE   
            invoke MessageBox,NULL,offset keyBuffer,ADDR AppName,MB\_OK      ;显示 key 串   
        .ENDIF   
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*   
        .ELSE   
                invoke DestroyWindow,hWnd   
            .ENDIF   
        .ELSE   
            .IF ax\==ButtonID   
                shr eax,16   
                .IF ax\==BN\_CLICKED   
                    invoke SendMessage,hWnd,WM\_COMMAND,IDM\_GETTEXT,0   
                .ENDIF   
            .ENDIF   
        .ENDIF   
    .ELSE   
        invoke DefWindowProc,hWnd,uMsg,wParam,lParam   
        ret   
    .ENDIF   
    xor    eax,eax   
    ret   
WndProc endp   
end start   
到这里这篇文章就差不多了。有些遗憾的是没有 DELPHI 的原代码。因为没见到懂 DELPHI 的人。小楼，AJJ 看到这个后能不能把 DELPHI 的注册机也加上呀。我知道的懂 DELPHI 的只有你们两个。：）   
                                                                              LL\[CCG\] & halfbit\[CCG\] 

#include "stdio.h"   
main()   
{   
unsigned char a\[80\];   
int d,i;   
unsigned long s;   
start:printf("\*\*\*\*\*\*\*\*\*-={ figugegl }=- Crackme #1\*\*\*\*\*\*\*\*\*\\nThis keygen is made by Pretender\\nPlease input your name  : ");   
gets(a);   
d=strlen(a);   
if(d<5) {printf("Your user name must longer than 4 letters!!please re-input\\n");goto start;}   
printf("Your Register code is  : ");   
for(i=0;i<d;i++)   
printf("%c",a\[i\]-i);   
  
printf("\\n \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*      ---    ---    ---\\n \*Welcome to WWW.CRACKNOW.COM\*      /      /      / --\\n \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*      ---    ---    --/\\n");   
} 

procedure TForm1.BitBtn1Click(Sender: TObject);   
var   
a,c,e:string;  //定义 A，C，E 为字符型，   
b,d:integer;  //定义 B，D 为数值型。   
  
begin   
a:=edit1.Text;    //让 A 的值等于 EDIT1 的值。   
  if length(a)<5 then  //LENGTH（）相当于 VB 中的 LEN（），如果注册名小于则报错。   
    showmessage('注册名不能小于五位!')   
  else   
  begin   
    for  b:=1 to length(a) do  ’循环语句，用来计算注册码。   
    begin   
      d:=ord(a\[b\]);    
      d:=d+1-b;        
      c:=chr(d);   
      e:=e+c;   
    end;   
    end;   
edit2.Text:=e;   
end;   
  
只是将 LL 大侠的 VB 版改为 DELPHI 的了，：）   
我试了还行。  4dd466d0e1e0cc123afbe504da2462ca