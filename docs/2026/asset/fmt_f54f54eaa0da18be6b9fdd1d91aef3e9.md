
```asm
  .386
  .model flat, stdcall
  option casemap :none
include  windows.inc
include  user32.inc
includelib user32.lib
include  kernel32.inc
includelib kernel32.lib
include  Comctl32.inc
includelib Comctl32.lib
ModifyData  STRUCT
 dwSize  DWORD  ?
 Pos   DWORD  ?
 Old   BYTE 25 dup (?)
 New   BYTE 25 dup (?)
ModifyData ENDS
  .data?
hInstance  dd  ?
dwProcessId  dd  ?
hProcess  dd  ?
stPass  ModifyData <?>  ;过关
stNoBall ModifyData <?>  ;不生成球
stFan1  ModifyData <?>  ;风扇可用
stFan2  ModifyData <?>  ;风扇可用
stEnergy ModifyData <?>
stSelect ModifyData <?>  ;选关
  .data
szClass  db 'MainWindow',0
szCaption db 'Atomica Deluxe 2.52',0
szStRun  db '正在运行',0
szStNoRun db '没有运行',0
szStErr  db '错误',0
Flag  db 0
szLarge  db '不能大于40，否则不能玩',0
szDataNoMatch db '数据不匹配，可能是版本不对',13,10
    db '本人用的版本是：',13,10
    db 'ATOMICA Deluxe',13,10
    db 'version 2.52',13,10
    db 'March 17, 2003',13,10,13,10
    db 'by PopCap Games',0

szPass  db 0Fh,85h,17h,01,00,00,0
szNoBall db 8Ah,5Dh,0FFh,0
szFan1  db 0Fh,85h,0A8h,00,00,00,0
szFan2  db 0Fh,84h,8Ah,00,00,00,0
szEnergy db 75h,0
szModifyEnergy db 0ebh,0
szModify db 90h,90h,90h,90h,90h,90h,0
szOldSelect db 8Bh, 91h,0C8h,77h,77h,77h, 8Bh,81h, 0CCh,77h, 77h, 77h, 42h,40h,89h, 91h,0C8h,77h,77h,77h,0
szNewSelect db 0B8h,99h,99h,99h,99h,90h,90h, 90h,90h, 90h,90h,90h,90h,90h,89h, 81h,0C8h,99h,99h,99h,0
szBtnSelect db '选关(&S)',0
szBtnHuifu db '恢复代码',0
bBtn  db 1
.code
Modify    proc uses ebx hWnd,lpst,action
 local szBuf[50]:byte
 mov ebx,lpst
 assume ebx:ptr ModifyData
 .if action  ;action is true then modify
  invoke RtlZeroMemory,addr szBuf,sizeof szBuf
  invoke ReadProcessMemory,hProcess,[ebx].Pos,addr szBuf,[ebx].dwSize,NULL
  invoke lstrcmp,addr szBuf,addr [ebx].Old
  .if eax
   invoke MessageBox,hWnd,addr szDataNoMatch,addr szStErr,MB_ICONERROR
   xor eax,eax
   dec eax
  .else
   invoke WriteProcessMemory,hProcess,[ebx].Pos,addr[ebx].New,[ebx].dwSize,NULL
  .endif
 .else
  invoke WriteProcessMemory,hProcess,[ebx].Pos,addr[ebx].Old,[ebx].dwSize,NULL
 .endif
 ret
Modify endp
ModifyEnable proc hwnd,flag
 invoke GetDlgItem,hwnd,1000
 invoke EnableWindow,eax,flag
 invoke GetDlgItem,hwnd,1001
 invoke EnableWindow,eax,flag
 invoke GetDlgItem,hwnd,1002
 invoke EnableWindow,eax,flag
 invoke GetDlgItem,hwnd,1004
 invoke EnableWindow,eax,flag
 invoke GetDlgItem,hwnd,1005
 invoke EnableWindow,eax,flag
 invoke GetDlgItem,hwnd,1006
 invoke EnableWindow,eax,flag
 invoke GetDlgItem,hwnd,21
 invoke EnableWindow,eax,flag
 invoke GetDlgItem,hwnd,24
 invoke EnableWindow,eax,flag
 invoke GetDlgItem,hwnd,26
 invoke EnableWindow,eax,flag
 ret
ModifyEnable endp
FindProgram  proc hWnd
 invoke FindWindow,addr szClass,NULL
 .if eax
  push eax
  invoke SetDlgItemText,hWnd,11,addr szStRun
  invoke ModifyEnable,hWnd,TRUE
  pop eax
  .if Flag==0
   invoke GetWindowThreadProcessId,eax,addr dwProcessId
   invoke OpenProcess,PROCESS_ALL_ACCESS,FALSE,dwProcessId
   .if eax
    mov hProcess,eax
    mov Flag,1
   .else
    invoke SetDlgItemText,hWnd,11,addr szStErr
    invoke ModifyEnable,hWnd,FALSE
   .endif
  .endif
 .else
  invoke SetDlgItemText,hWnd,11,addr szStNoRun
  invoke ModifyEnable,hWnd,FALSE
 .endif
 ret
 FindProgram endp
_ProcDlgMain proc uses ebx edi esi hWnd,wMsg,wParam,lParam
LOCAL szBuffer[20]:byte
  mov eax,wMsg
  .if eax == WM_CLOSE
   invoke EndDialog,hWnd,NULL
  .elseif eax == WM_INITDIALOG
   invoke LoadIcon,hInstance,1
   invoke SendMessage,hWnd,WM_SETICON,ICON_BIG,eax
   invoke SendMessage,hWnd,WM_COMMAND,1003,NULL

   lea ebx,stPass
   assume ebx:ptr ModifyData
   mov [ebx].dwSize,6
   mov [ebx].Pos,004122FDh
   invoke lstrcpy,addr [ebx].Old,addr szPass
   invoke lstrcpyn,addr [ebx].New,addr szModify,13

   lea ebx,stNoBall
   assume ebx:ptr ModifyData
   mov [ebx].dwSize,3
   mov [ebx].Pos,004126B1h
   invoke lstrcpy,addr [ebx].Old,addr szNoBall
   invoke lstrcpyn,addr [ebx].New,addr szModify,7

   lea ebx,stFan1
   assume ebx:ptr ModifyData
   mov [ebx].dwSize,6
   mov [ebx].Pos,00433277h
   invoke lstrcpy,addr [ebx].Old,addr szFan1
   invoke lstrcpyn,addr [ebx].New,addr szModify,13

   lea ebx,stFan2
   assume ebx:ptr ModifyData
   mov [ebx].dwSize,6
   mov [ebx].Pos,00433295h
   invoke lstrcpy,addr [ebx].Old,addr szFan2
   invoke lstrcpyn,addr [ebx].New,addr szModify,13
   lea ebx,stEnergy
   assume ebx:ptr ModifyData
   mov [ebx].dwSize,1
   mov [ebx].Pos,004126B6h
   invoke lstrcpy,addr [ebx].Old,addr szEnergy
   invoke lstrcpy,addr [ebx].New,addr szModifyEnergy

   lea ebx,stSelect
   assume ebx:ptr ModifyData
   mov [ebx].dwSize,20
   mov [ebx].Pos,00432B80h
   invoke lstrcpy,addr [ebx].Old,addr szOldSelect
   lea ebx,[ebx].Old
   mov word ptr[ebx+3],0
   mov byte ptr [ebx+5],0
   mov word ptr[ebx+9],0
   mov byte ptr [ebx+11],0
   mov word ptr[ebx+17],0
   mov byte ptr [ebx+19],0
  .elseif eax == WM_COMMAND
   mov eax,wParam
   .if ax == IDOK
    invoke SendMessage,hWnd,WM_CLOSE,NULL,NULL
   .elseif ax==1003
    invoke FindProgram,hWnd
   .elseif ax==1001 ;选关
    invoke GetDlgItemInt,hWnd,1000,NULL,FALSE
    .if eax>40
     invoke MessageBox,hWnd,addr szLarge,addr szStErr,0
     ret
    .endif
    push eax
    lea ebx,stSelect
    assume ebx:ptr ModifyData
    invoke lstrcpy,addr [ebx].New,addr szNewSelect
    pop eax
    lea ebx,[ebx].New
    mov byte ptr [ebx+1],al
    mov byte ptr [ebx+2],ah
    rol eax,16
    mov byte ptr [ebx+3],al
    mov byte ptr [ebx+4],ah
    mov word ptr[ebx+17],0
    mov byte ptr [ebx+19],0

    .if bBtn
     invoke Modify,hWnd,addr stSelect,1
     invoke SetDlgItemText,hWnd,1001,addr szBtnHuifu
     mov bBtn,0
    .else
     invoke Modify,hWnd,addr stSelect,0
     invoke SetDlgItemText,hWnd,1001,addr szBtnSelect
     mov bBtn,1
    .endif
   .elseif ax==1007
    invoke Modify,hWnd,addr stSelect,0
   .elseif ax==1002 ;风扇总可用
    invoke IsDlgButtonChecked,hWnd,1002
    .if eax == BST_CHECKED
     ;启用风扇
     invoke Modify,hWnd,addr stFan2,1
     invoke Modify,hWnd,addr stFan1,1
     inc eax
     .if eax==0
      invoke CheckDlgButton,hWnd,1002,BST_UNCHECKED
     .endif
    .else
     ;取消风扇
     invoke Modify,hWnd,addr stFan1,0
    .endif
   .elseif ax==1004 ;每消一组就增加能量
    invoke IsDlgButtonChecked,hWnd,1004
    .if eax == BST_CHECKED
     ;每消一组就增加能量
     invoke Modify,hWnd,addr stEnergy,1
     inc eax
     .if eax==0
      invoke CheckDlgButton,hWnd,1004,BST_UNCHECKED
     .endif
    .else
     ;每消一组就增加能量
     invoke Modify,hWnd,addr stEnergy,0
    .endif
   .elseif ax==1005 ;过关
    invoke IsDlgButtonChecked,hWnd,1005
    .if eax == BST_CHECKED
     ;启用过关
     invoke Modify,hWnd,addr stPass,1
     inc eax
     .if eax==0
      invoke CheckDlgButton,hWnd,1005,BST_UNCHECKED
     .endif
    .else
     ;取消过关
     invoke Modify,hWnd,addr stPass,0
    .endif
   .elseif ax==1006 ;不生成新的球
    invoke IsDlgButtonChecked,hWnd,1006
    .if eax == BST_CHECKED
     ;不生成新的球
     invoke Modify,hWnd,addr stNoBall,1
     inc eax
     .if eax==0
      invoke CheckDlgButton,hWnd,1006,BST_UNCHECKED
     .endif
    .else
     ;生成新的球
     invoke Modify,hWnd,addr stNoBall,0
    .endif
   .endif
  .else
   mov eax,FALSE
   ret
  .endif
  mov eax,TRUE
  ret
 _ProcDlgMain endp
start:
  invoke InitCommonControls
  invoke GetModuleHandle,NULL
  mov hInstance,eax
  invoke DialogBoxParam,hInstance,101,NULL,offset _ProcDlgMain,NULL
  invoke ExitProcess,NULL
 end start
```

---

## 经验总结

写游戏修改器基本上都是搜索出地址，进行分析。仅限于单机版，我决定不玩网络游戏，所以也不想研究了，也不知道网游是不是这样(应该有极少数是这样的吧)。

---

## 版权声明

本文原创于看雪技术论坛, 转载请注明作者并保持文章的完整, 谢谢!

---

文章写于2007-08-29

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

上传的附件：

- [ModifyAtomica.rar](javascript:void(0);)
  （6.05kb，55次下载）
