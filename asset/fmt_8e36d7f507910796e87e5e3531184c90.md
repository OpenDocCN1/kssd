
# 计算 PE 头部校验和

> 原文：[`www.pediy.com/kssd/pediy05/pediy50316.htm`](https://www.pediy.com/kssd/pediy05/pediy50316.htm)

```asm
;                 如何计算 PE 头部校验和

;

;

; 你应该已经见过 LoadPE 的校正 PE 头 checksum 的功能,想给你自己的工具加一个这种功能?

; yes, it's easy! let's GO! GO! GO!

;

; PE checksum 计算算法:

; PE 文件修改的时候一般可不用计算 checksum, 但是由于 OS 对系统文件校验其 checksum

; 因此修改 PE 的时候有时会需要更新其 checksum 值, 一般可以用 imagehelp.dll 中的函数

; 进行计算, 不过实际上这玩意很简单, 我们完全可以自己来做. 假设 PE 文件大小 x, 若为奇数

; 字节则用 0 补足文件使 x 为偶数, 然后取文件中所有的字进行带进位加法操作(adc), 最后和

; 文件大小进行带进位加法操作(adc) 得到结果即为 checksum 结果. 当然在计算前应置 pe.CheckSum 域为 0.

; 具体见下面的 cal_checksum 例程.

; enjoy it.

;                                     YJ.Hume.冷雨

include '%fasinc%/win32a.inc'

include '%fasinc%/pestruc.h'

struc OPENFILENAME
{
  .lStructSize        dd      ?
  .hwndOwner         dd      ?
  .hInstance         dd      ?
  .lpstrFilter       dd      ?
  .lpstrCustomFilter dd      ?
  .nMaxCustFilter    dd      ?
  .nFilterIndex      dd      ?
  .lpstrFile         dd      ?
  .nMaxFile          dd      ?
  .lpstrFileTitle    dd      ?
  .nMaxFileTitle     dd      ?
  .lpstrInitialDir   dd      ?
  .lpstrTitle        dd      ?
  .Flags             dd      ?
  .nFileOffset       dw      ?
  .nFileExtension    dw      ?
  .lpstrDefExt       dd      ?
  .lCustData         dd      ?
  .lpfnHook          dd      ?
  .lpTemplateName    dd      ?
  .size=$-.lStructSize
}

virtual       at 0
ofn    OPENFILENAME
end virtual

;-----------------------------------------

.data

opf OPENFILENAME

lpStrFilter   db "PE Files",0,"\*.exe;\*.dll",0
              db "WDM Files",0,"\*.sys",0,0
zTit           db "Hume's PE checkSum 计算器,2K2",0
initial_dir    db ".",0
ms000          db "File OPERATION ERR!",0
fmt            db "The PE-CheckSum of Current file IS:%X",0
buf:           rb 256

.code

StArT:
        mov   [opf.lStructSize],ofn.size
        mov   [opf.lpstrFile],buf
        mov   [opf.nMaxFile],256
        mov   [opf.lpstrFilter],lpStrFilter
        mov   [opf.lpstrTitle],zTit
        mov   [opf.lpstrInitialDir],initial_dir
        mov   [opf.Flags],81000h
        invoke  GetOpenFileName,opf
        or      eax,eax
        je      exit
        mov   eax,ofn.lpstrFile
        invoke  CreateFile,[opf.lpstrFile],0x80000000,0,0,3,0,0
        inc   eax
        je      err
        dec   eax
        push  eax
        invoke  GetFileSize,eax,0
        push  eax
        push  eax
        invoke  VirtualAlloc,0,eax,1000h,4
        or      eax,eax
        je      err
        pop   ecx
        push  eax
        mov   esi,eax
        mov   edx,[esp+8]
        invoke  ReadFile,edx,eax,ecx,esp,0
        mov   edi,[esi+3ch]
        add   edi,esi
        mov   dword [edi+peh.CheckSum],0
        mov   ecx,[esp+4]
        inc   ecx
        shr   ecx,1
        xor   ebp,ebp
        clc
    cal_checksum:
        adc   bp,word [esi]
        inc   esi
        inc   esi
        loop  cal_checksum
        mov   ebx,[esp+4]
        add   ebp,ebx
        pop   eax
        invoke  VirtualFree,eax,ebx,4000h
        pop   eax
        pop   eax
        invoke  CloseHandle,eax
        invoke  wsprintf,buf,fmt,ebp
        invoke  MessageBox,0,buf,zTit,40h
exit:
        invoke  ExitProcess,0
    err:
        invoke  MessageBox,0,ms000,zTit,20h
        jmp   byte exit

.end StArT  4dd466d0e1e0cc123afbe504da2462ca
```
