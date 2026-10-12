
```
    mov  wc.lpszClassName,OFFSET ClassName   

    invoke LoadIcon,NULL,IDI\_APPLICATION   

    mov  wc.hIcon,eax   

    mov  wc.hIconSm,eax   

    invoke LoadCursor,NULL,IDC\_ARROW   

    mov  wc.hCursor,eax   

    invoke RegisterClassEx, addr wc   

    invoke CreateWindowEx,WS\_EX\_CLIENTEDGE,ADDR ClassName, \   

                          ADDR AppName, WS\_OVERLAPPEDWINDOW,\   

                          CW\_USEDEFAULT, CW\_USEDEFAULT,\   

                          300,200,NULL,NULL, hInst,NULL   

    mov  hwnd,eax   

    invoke ShowWindow, hwnd,SW\_SHOWNORMAL   

    invoke UpdateWindow, hwnd   

    .WHILE TRUE   

        invoke GetMessage, ADDR msg,NULL,0,0   

        .BREAK .IF (!eax)   

        invoke TranslateMessage, ADDR msg   

        invoke DispatchMessage, ADDR msg   

    .ENDW   

    mov    eax,msg.wParam   

    ret   

WinMain endp   

WndProc proc hWnd:HWND, uMsg:UINT, wParam:WPARAM, lParam:LPARAM   

    .IF uMsg==WM\_DESTROY   

        invoke PostQuitMessage,NULL   

    .ELSEIF uMsg==WM\_CREATE   

        invoke CreateWindowEx,WS\_EX\_CLIENTEDGE, ADDR EditClassName,NULL,\   

                          WS\_CHILD or WS\_VISIBLE or WS\_BORDER or ES\_LEFT or\   

                          ES\_AUTOHSCROLL,\   

                          50,35,200,25,hWnd,8,hInstance,NULL   

        mov  hwndEdit,eax   

        invoke SetFocus, hwndEdit   

        invoke CreateWindowEx,NULL, ADDR ButtonClassName,ADDR ButtonText,\   

                          WS\_CHILD or WS\_VISIBLE or BS\_DEFPUSHBUTTON,\   

                          75,70,140,25,hWnd,ButtonID,hInstance,NULL   

        mov  hwndButton,eax   

    .ELSEIF uMsg==WM\_COMMAND   

        mov eax,wParam   

        .IF lParam==0   

            .IF ax\==IDM\_HELLO   

                invoke SetWindowText,hwndEdit,ADDR TestString   

            .ELSEIF ax\==IDM\_CLEAR   

                invoke SetWindowText,hwndEdit,NULL   

;********************************************************************   

            .ELSEIF  ax\==IDM\_GETTEXT   

                invoke GetWindowText,hwndEdit,ADDR buffer,512     ;获得 name 串   

               mov    ebx,0   

               .WHILE byte ptr \[buffer+ebx\]!=0   

                   mov    al,\[buffer+ebx\]   

                   sub    al,bl   

                   mov    \[keyBuffer+ebx\],al     ;变换后放入 key 串   

                   inc    ebx;   

               .ENDW   

               .IF ebx<5   

                   invoke MessageBox,NULL,offset name5, offset msgError,MB\_OK   

               .ELSE   

                   invoke MessageBox,NULL,offset keyBuffer,ADDR AppName,MB\_OK     ;显示 key 串   

               .ENDIF   

;********************************************************************   

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

    xor    eax,eax   

    ret   

WndProc endp   

end start
```

到这里这篇文章就差不多了。有些遗憾的是没有 `DELPHI` 的原代码。因为没见到懂 `DELPHI` 的人。小楼，`AJJ` 看到这个后能不能把 `DELPHI` 的注册机也加上呀。我知道的懂 `DELPHI` 的只有你们两个。：） 

$LL[CCG] \& halfbit[CCG]$

```c
#include "stdio.h"   

main()   

{   

unsigned char a\[80\];   

int d,i;   

unsigned long s;   

start:printf("\*\*\*\*\*\*\*\*\*-={ figugegl }=- Crackme #1\*\*\*\*\*\*\*\*\*\\nThis keygen is made by Pretender\\nPlease input your name  : ");   

gets(a);   

d=strlen(a);   

if(d<5) {printf("Your user name must longer than 4 letters!!please re-input\\n");goto start;}   

printf("Your Register code is  : ");   

for(i=0;i<d;i++)   

printf("%c",a\[i\]-i);   

printf("\\n \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*       ---     ---     ---\\n \*Welcome to WWW.CRACKNOW.COM\*       /       /       / --\\n \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*       ---     ---     --/\\n");   

}
```

```delphi
procedure TForm1.BitBtn1Click(Sender: TObject);   

var   

a,c,e:string;  //定义 A，C，E 为字符型，   

b,d:integer;  //定义 B，D 为数值型。   

begin   

a:=edit1.Text;    //让 A 的值等于 EDIT1 的值。   

  if length(a)<5 then  //LENGTH（）相当于 VB 中的 LEN（），如果注册名小于则报错。   

    showmessage('注册名不能小于五位!')   

  else   

  begin   

    for  b:=1 to length(a) do  ’循环语句，用来计算注册码。   

    begin   

      d:=ord(a\[b\]);   

      d:=d+1-b;   

      c:=chr(d);   

      e:=e+c;   

    end;   

    end;   

edit2.Text:=e;   

end;
```

只是将 `LL` 大侠的 `VB` 版改为 `DELPHI` 的了，：）

我试了还行。 `4dd466d0e1e0cc123afbe504da2462ca`
