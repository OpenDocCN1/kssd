
# BadCopy99V2.0

> 原文：[`www.pediy.com/kssd/pediy05/pediy50202.htm`](https://www.pediy.com/kssd/pediy05/pediy50202.htm)

BadCopy99V2.0 注册机 (MASM 版) 下载: 华军

注册机下载 `fpxxp.com/bc2-key.exe`

作者: 风飘雪 `http://fpx.yeah.net`

```asm
; #########################################################################
; KeyGenerator for xxx, coded by fpx 2002（注册机模板）
; #########################################################################

    .386
    .model flat, stdcall
    option casemap :none ; case sensitive 区分大小写

; #########################################################################

    include c:\masm32\include\windows.inc
    include c:\masm32\include\user32.inc
    include c:\masm32\include\kernel32.inc
    include c:\masm32\include\gdi32.inc
    include c:\masm32\include\masm32.inc
    includelib c:\masm32\lib\user32.lib
    includelib c:\masm32\lib\kernel32.lib
    includelib c:\masm32\lib\gdi32.lib
    includelib c:\masm32\lib\masm32.lib

; #########################################################################

    szText MACRO Name, Text:VARARG
        LOCAL lbl
        jmp lbl
            Name db Text,0
        lbl:
    ENDM
    WndProc PROTO :DWORD,:DWORD,:DWORD,:DWORD

.data
    TheName     dd 0
    Button      dd 0
    hInstance   dd 0
    hIconImage  dd 0
    hIcon       dd 0
    dlgname     db "KEYGEN",0
    errorLabel  db "Error"
    szName      db 30 dup(?)
    szKey       db 10 dup(?)
    szWS        db '%d',0

; #########################################################################

.code

start:
    invoke GetModuleHandle, NULL
    mov hInstance, eax
    ; call dialog box stored in resource file
    invoke DialogBoxParam,hInstance,ADDR dlgname,0,ADDR WndProc,0
    invoke ExitProcess,eax

; #########################################################################

WndProc proc hWin:DWORD, uMsg:DWORD, wParam:DWORD, lParam :DWORD
    LOCAL Ps:PAINTSTRUCT
    ; initialization
    .if uMsg == WM_INITDIALOG
        szText dlgTitle,"fpx's KeyGenerator for BadCopy99V2.0"
        invoke SendMessage,hWin,WM_SETTEXT,0,ADDR dlgTitle
        invoke LoadIcon,hInstance,200
        mov hIcon, eax
        invoke SendMessage,hWin,WM_SETICON,1,hIcon
        invoke GetDlgItem,hWin,100
        mov TheName, eax
        invoke GetDlgItem,hWin,1000
        mov Button, eax
        xor eax, eax
        ret
    .elseif uMsg == WM_COMMAND
        .if wParam == 1000
            invoke GetDlgItemText,hWin,100,offset szName,30 ; 获得姓名
            cmp eax,1
            jl NoName ; 检查姓名输入情况
            push eax ; 姓名长度
            push offset szName ; 姓名=dlgitemtext 入栈
            call generateKey ; 调用算码程序
            invoke wsprintf,offset szKey,offset szWS,eax ; 写序列号
            invoke SetDlgItemText,hWin,22,offset szKey ; 将序列号显示出来
            jmp NoError
            NoName: ; 如果没输姓名
            szText errorMsg,"I Need Characters :)"
            invoke MessageBox,hWin,ADDR errorMsg,ADDR errorLabel,MB_OK
            NoError:
        .endif
    .elseif uMsg == WM_CLOSE
        invoke EndDialog,hWin,0
    .elseif uMsg == WM_PAINT
        invoke BeginPaint,hWin,ADDR Ps
        invoke FrameWindow,hWin,0,1,1
        invoke FrameWindow,hWin,1,1,0
        invoke EndPaint,hWin,ADDR Ps
        xor eax, eax
        ret
    .endif
    ret
WndProc endp

; ########################################################################
; 算码过程,基本上参考反汇编代码，可见学好 32 位汇编真的很重要 :)
; ########################################################################

generateKey:
    push ebp
    mov ebp,esp
    push ebx
    push ecx
    push edx
    push edi
    push esi
    push ebp
    xor esi,esi ; esi 清零
    mov ecx,eax ; 姓名长度传入 ecx
    mov eax,1
addr_1:
    mov edx,[offset szName]
    mov dl,[eax+szName-1] ; 依次取姓名
    and edx,255 ; FF 即 255
    mov ebx,edx ; 注册码最后储存在 esi
    imul ebx,eax
    add esi,ebx
    mov ebx,edx
    inc eax
    dec ecx
    jnz addr_1
    mov eax, 6695bh
    sub eax,esi ; 结果存入 eax
    pop ebp
    pop esi
    pop edi
    pop edx
    pop ecx
    pop ebx
    leave
    retn 00008h

; ########################################################################
end start
```

`badcopy pro 3.0` 不能运行在中文 `WIN` 系统上，去装个 `E` 文的 `98` 吧。

没有注册的时候可以读取光盘、软盘中的文件，但是最后保存的时候却提示没有注册，不能保存，这是不要急着点确定，到它的安装目录下找找，可以找到一个临时文件，和你要读的那个文件一样大，把它拷贝出来。但发现不能用，如果你用 `16` 进制编辑器打开，会发现里面的内容是每个字节与一个特殊字符异或后得到的，因为对同一个字符异或 `2` 次可以得到本身，所以只要用 `badcopy pro 3.0` 将拷贝出来的临时文件再读一次，再拷贝出第二次的临时文件就可以用了。

```asm
.const

.data
szHomePage db "http://www.365hz.net",0
szEmail    db "mailto:ljyljx@163.com",0
szErrMess  db "输入的序列号不正确！",0
szFmt      db "%u",0
szBuffer   db 20 dup(0)

.code
mov esi,eax
invoke lstrlen,esi
mov ecx,eax
mov eax,1
xor edi,edi

n1:
movzx edx,byte ptr [esi+eax-1]
and edx,255
mov ebx,edx
imul ebx,eax
add edi,ebx
inc eax
dec ecx
jnz n1
mov eax,6695bh
sub eax,edi
invoke wsprintf,addr szBuffer,addr szFmt,eax
lea eax,szBuffer
```
