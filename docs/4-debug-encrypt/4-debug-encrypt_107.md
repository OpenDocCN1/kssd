

# rsa 计算的小工具

> 原文：[`www.pediy.com/kssd/pediy06/pediy6869.htm`](https://www.pediy.com/kssd/pediy06/pediy6869.htm)

好久没有作品可以放上来与大家分享了，今特奉上一款本人独立创作完成的完全用汇编语言编写而成的小工具"rsa 计算器"。输入输出全部都用十六进制格式，可对数据进行运算，而不是对文本。

点击下载：附件！  
  
n 能到 1024 位。也有疑问，位数再高不知为何不行。（改变 DD\_NUM 等值的数即可改变位数） 

;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>  
    .386  
    .model flat, stdcall  
    option casemap :none  
;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>  
include    windows.inc  
include    user32.inc  
includelib  user32.lib  
include    kernel32.inc  
includelib  kernel32.lib  
;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>  
ICO\_MAIN  equ    1000  ;图标  
DLG\_MAIN  equ    100  
  
DD\_NUM     EQU   8;32    ;位数除以 32，即 DWORD 数  
;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>  
    .data?  
  
hInstance  dd    ?  
  
actural\_b  dd  DD\_NUM \* 2 dup (0)  
actural\_n  dd  DD\_NUM \* 2 dup (0)  
actural\_m  dd  DD\_NUM \* 2 dup (0)  
result    dd  DD\_NUM \* 2 dup (0)  
presult    dd  ?  
  
    .data  
  
rsa\_a  dd  DD\_NUM dup (0)  
rsa\_b  dd  DD\_NUM dup (0)  
rsa\_c  dd  DD\_NUM dup (0)  
rsa\_n  dd  DD\_NUM dup (0)  
rsa\_p  dd  DD\_NUM dup (0)  
rsa\_m  dd  DD\_NUM dup (0)  
rsa\_back dd  DD\_NUM dup (0)  
  
    .const  
default\_b  db  "10001",0  ;设置默认的私钥  
default\_n  db  "C7B71C573A60F571",0  ;设置默认的模  
default\_m  db  "BCF0DB712E6595F1",0    ;设置默认的数据  
num\_error  db  "有非法字符！",0  
blank\_error  db  "输入所有数据！",0  
format    db   "%X",0  
format8    db   "%8X",0  
can\_not\_factor  db  "这个 n 是素数，不能分解！",0  
;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>  
    .code  
;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>  
rsa  proc  
  push ebp  
  mov  ebp,esp  
  
;------------------------------;  置初值  
  mov rsa\_c,1  
;-------------------------------循环计算  
rsa0:  mov eax,rsa\_b    ;bl;   
  and eax,1        ;b&1  
  .if eax == 0        ;b 偶数?  
;------------------------------b = b >> 1  ;b 是偶数时  
    rcl eax,1      
    mov esi,DD\_NUM ;2  
    .repeat  
      rcr eax,1  
      rcr \[rsa\_b-4+esi\*4\],1    
      rcl eax,1  
      dec esi  
    .until esi == 0  
;------------------------------a = (a \* a) mod n  
    lea  eax,rsa\_a  ;c--\[EBP\-08h\]\[EBP\-04h\]   
    call bigmul    ;<--调用两数相乘再除以 n 的子程序  
    lea  eax,rsa\_a  ;c--\[EBP\-08h\]\[EBP\-04h\]   
    call bigdiv  
  .else  
    sub rsa\_b,1  ;b = b - 1 只需将 b 的最低位置 0 即可  
;------------------------------c = (c \* a) mod n  
    lea  eax,rsa\_c    ;a  
    call bigmul    ;<--调用两数相乘再除以 n 的子程序  
    lea  eax,rsa\_c    ;a  
    call bigdiv  
  .endif  
;------------------------------判断 b 是否 0  while(b)  
  mov esi,DD\_NUM ;  
  .repeat  
    cmp \[rsa\_b-4+esi\*4\],0  ;判断 b 是否 0  
    ja rsa0  
    dec esi  
  .until esi == 0  
;------------------------------计算结束  
  pop      ebp  
  retn  
rsa  endp  
;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>  
;-----------------------------两数相乘的子程序，这里利用了数据连续存储的特点，乘积的长度与两个数据一样，  
bigmul  proc      ;超出的部分会自动存到余数所在的空间  
  mov ebx,eax    ;唯一的一个输入参数，是一个乘数的地址，另一个乘数固定为 rsa\_a  
  lea ecx,rsa\_a    ;乘积存放在 rsa\_p 和 rsa\_m 合起来的连续空间  
  xor eax,eax  
  xor esi,esi  
  .repeat  
    mov \[rsa\_p+esi\*4\],eax    ;乘积置初值 0  
    inc esi  
  .until  esi == DD\_NUM + DD\_NUM ;4  
  
  xor edi,edi  
  push ebp  
  .repeat  
    lea ecx,\[ecx+edi\*4\]    ;ecx\=ecx+edi\*4  
    xor esi,esi      ;esi\=0  
    lea ebp,\[rsa\_p+edi\*4\]  
    .repeat  
      mov eax,\[ebx+esi\*4\]  
      mul DWORD ptr \[ecx\]  
      lea ebp,\[ebp+esi\*4\]  
      add DWORD ptr \[ebp\],eax  
      adc DWORD ptr \[ebp+4\],edx  
      adc DWORD ptr \[ebp+8\],0  
      inc esi  
    .until esi == DD\_NUM ;  
    inc edi  
  .until edi == DD\_NUM ;  
  pop ebp  
  retn  
bigmul  endp  
;--------------------------------两个数相除 取余数  
bigdiv  proc      ;被除数为 rsa\_p，存在于 rsa\_p 至 rsa\_m 的连续空间，除数为 rsa\_n  
  mov ebx,eax  
  mov edx,DD\_NUM \* 20H ;<--设循环次数  
  
mydiv:  xor esi,esi  
  inc esi  
  mov ecx,DD\_NUM + DD\_NUM - 1  
  shl DWORD ptr \[rsa\_p\],1;被除数左移 1 位，rsa\_p 至 rsa\_m 连续左移  
shiftp:  rcl DWORD ptr \[rsa\_p+esi\*4\],1  
  inc esi  
  loop shiftp  
  
  mov esi,DD\_NUM;2    ;比较  
  .repeat  
    mov eax,\[rsa\_n-4+esi\*4\]  
    cmp \[rsa\_m-4+esi\*4\],eax  
    jc  loop0  
    ja  asubb  
    dec esi  
  .until esi == 0  
  
asubb:  xor esi,esi    ;相减  
  clc  
  mov ecx,DD\_NUM  
msubn:  mov eax,\[rsa\_n+esi\*4\]  
  sbb \[rsa\_m+esi\*4\],eax  
  inc esi  
  loop msubn  
  
  inc DWORD ptr \[rsa\_p\]  ;为了取商，加上了这一段  
  xor esi,esi  
  inc esi  
  mov ecx,DD\_NUM - 1  
padd1:  adc DWORD ptr \[rsa\_p+esi\*4\],0  
  inc esi  
  loop padd1  
  
loop0:  dec edx  
  jnz mydiv  
;相除结束，\[rsa\_p\]是商，\[rsa\_m\]是余数  
  xor esi,esi  
  .repeat  
    mov eax,\[rsa\_m+esi\*4\]    ;保存余数  
    mov \[ebx+esi\*4\],eax  
    inc esi  
  .until esi == DD\_NUM ;2  
  
  retn  
bigdiv  endp  
;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>  
scan\_data  proc  hWnd  
;检查输入数据的长度  
  mov edi,eax  
  invoke lstrlen,eax  
  .if eax ==0   
    jmp nonum  
  .endif  
  
;检查输入数据中是否有非法字符  
scan\_b:  
  mov al,BYTE ptr \[edi\]  
  .if al == 0  
    xor eax,eax  
    inc eax  
    jmp scan\_end  
  .elseif al >= '0' && al <= '9' || al >= 'A' && al <= 'F' || al >= 'a' && al <= 'f'  
    inc edi  
    jmp scan\_b  
  .else  
    jmp notnum  
  .endif  
notnum:    
  invoke SetDlgItemText,hWnd,104,addr num\_error  
  xor eax,eax  
  jmp scan\_end  
nonum:    
  invoke SetDlgItemText,hWnd,104,addr blank\_error  
  xor eax,eax  
scan\_end:  
  pop ebp  
  retn 4  
scan\_data  endp  
;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>  
stringtoint  proc  
  mov esi,eax  
  mov edi,edx  
  invoke lstrlen,eax  
  xchg eax,esi  
  mov edx,edi  
  xor ebx,ebx  
conv:  mov cl,BYTE ptr \[eax+esi\-1\]  
  .if cl >= 'A' && cl <= 'F'  
    sub cl,'A'-0ah  
  .elseif cl >= 'a' && cl <= 'f'  
    sub cl,'a'-0ah  
  .else  
    sub cl,'0'  
  .endif  
  mov bl,cl  
  
  dec esi  
  jz oddbit  
  mov cl,BYTE ptr \[eax+esi\-1\]  
  .if cl >= 'A' && cl <= 'F'  
    sub cl,'A'-0ah  
  .elseif cl >= 'a' && cl <= 'f'  
    sub cl,'a'-0ah  
  .else  
    sub cl,'0'  
  .endif  
  shl cl,4  
  add bl,cl  
  mov BYTE ptr \[edx\],bl  
  inc edx  
  dec esi  
  jnz conv  
  jmp convert\_end  
  
oddbit:  mov BYTE ptr \[edx\],bl  
    
convert\_end:  
  retn  
stringtoint  endp  
;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>  
inttostring  proc  
  mov esi,eax  
  mov edi,edx  
  invoke lstrlen,eax  
  cdq  
  mov ecx,4  
  div ecx  
  mov ebx,eax  
  xor eax,eax  
  lea eax,result  
  mov presult,eax  
  .if edx != 0  
    mov eax,DWORD ptr \[esi+ebx\*4\]  
    invoke wsprintf,presult,addr format,eax  
    add presult,eax  
  .endif  
  .while ebx > 0  
    mov eax,DWORD ptr \[esi+ebx\*4-4\]  
    invoke wsprintf,presult,addr format,eax  
    add presult,eax  
    dec ebx  
  .endw  
  retn  
inttostring  endp  
;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>  
\_ProcDlgMain  proc  uses ebx edi esi hWnd,wMsg,wParam,lParam  
  
    mov  eax,wMsg  
    .if  eax == WM\_CLOSE  
      invoke  EndDialog,hWnd,NULL  
    .elseif  eax == WM\_INITDIALOG  
      invoke  LoadIcon,hInstance,ICO\_MAIN  
      invoke  SendMessage,hWnd,WM\_SETICON,ICON\_BIG,eax  
      invoke SetDlgItemText,hWnd,101,addr default\_b  
      invoke SetDlgItemText,hWnd,102,addr default\_n  
      invoke SetDlgItemText,hWnd,103,addr default\_m  
    .elseif  eax == WM\_COMMAND  
      mov  eax,wParam  
      .if  ax == IDCANCEL   
        invoke  EndDialog,hWnd,NULL  
      .elseif  ax == IDOK  
        invoke SetDlgItemText,hWnd,104,NULL   
        lea eax,actural\_b  
        xor edx,edx  
        mov ecx,DD\_NUM \* 8 + 1   
clrmem:        mov \[eax + ecx \* 4 - 4\],edx  
        loop clrmem  
        lea eax,rsa\_a  
        mov ecx,DD\_NUM \* 6 + 1  
clrmem1:      mov \[eax + ecx \* 4 - 4\],edx  
        loop clrmem1  
  
        invoke GetDlgItemText,hWnd,101,addr actural\_b,sizeof actural\_b  
        invoke GetDlgItemText,hWnd,102,addr actural\_n,sizeof actural\_n  
        invoke GetDlgItemText,hWnd,103,addr actural\_m,sizeof actural\_m  
        lea eax,actural\_b  
        invoke scan\_data,hWnd  
        .if eax   
          lea eax,actural\_n  
          invoke scan\_data,hWnd  
          .if eax  
            lea eax,actural\_m  
            invoke scan\_data,hWnd  
            .if eax   
              lea eax,actural\_b  
              lea edx,rsa\_b  
              call stringtoint  
              lea eax,actural\_n  
              lea edx,rsa\_n  
              call stringtoint  
              lea eax,actural\_m  
              lea edx,rsa\_a  
              call stringtoint  
              call rsa  
              lea eax,rsa\_c  
              lea edx,result  
              call inttostring  
              invoke SetDlgItemText,hWnd,104,addr result  
            .endif  
          .endif  
        .endif  
      .endif  
    .else  
      mov  eax,FALSE  
      ret  
    .endif  
    mov  eax,TRUE  
    ret  
  
\_ProcDlgMain  endp  
;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>  
start:  
    invoke  GetModuleHandle,NULL  
    mov  hInstance,eax  
    invoke  DialogBoxParam,hInstance,DLG\_MAIN,NULL,offset \_ProcDlgMain,NULL  
    invoke  ExitProcess,NULL  
;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>  
    end  start 

 4dd466d0e1e0cc123afbe504da2462ca