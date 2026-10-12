
# Window user32 子动态库控件封装和消息分发浅析

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-128.htm`](https://www.pediy.com/kssd/pediy09/pediy09-128.htm)

> 众所周知：`WIN32` 子动态库 `user32.dll` 封装了系统定义的全局标准控件类和 `WNDPROCESS`，然后共享给各个 `Application`，系统启动会加载入 `user32`，然后注册它们的。仅仅是感兴趣没有其他目的

> 俺是一个小菜鸟，结合 `2k` 代码只分析应用层。 机器环境双核 `xp` `sp2`。

> `user32.dll` 中的函数命名一般有统一性，很有意思。呵呵。算是它的 `native` 风格定义吧。它们会进一步校验上层传的参数，并且适当设置 `GetLastError` 等细节，然后继续分发到 `ntdll` `Native` 层，或经过系统中断服务层进入 `win32.sys` 驱动层完成调用等。

> 比如：`DispatchMessageA` 调用 `DispatchMessageWorker`，又而会调用 `UserCallWinProcCheckWow`，然后最终调用到 `InternalCallWinProc`，把消息分发到我们的窗口的 `WinProc` 过程。 这个过程完全是在应用层进行的。

> 再如：`SendMessageA` --> `SendMessageWorker` --> `UserCallWinProcCheckWow` --> `InternalCallWinProc` --> `MyDiyWinProc`。也是在应用层。

> 这个：`DefWindowProc` --> `DefWindowProcWorker` --> `InternalCallWinProc` --> `RealDefWindowProcWorker` --> 消息表 --- 事件表调度 > `NtUserMessageCall` --> 进入内核和 `win32.sys` --> 处理完毕，返回应用层。有内核辅助处理。

> 先看看 `user32` 注册控件类的地方：

```
.text:77D2EC54  __stdcall RW_RegisterControls() proc near
.text:77D2EC54                 ; CODE XREF: ClientThreadSetup()+11D p
.text:77D2EC54                 ; CtxInitUser32():loc_77D4A7AD p
.text:77D2EC54

.text:77D2EC54 var_30           = dword ptr -30h
.text:77D2EC54 var_2C           = dword ptr -2Ch
.text:77D2EC54 var_28           = dword ptr -28h
.text:77D2EC54 var_20           = dword ptr -20h
.text:77D2EC54 var_1C           = dword ptr -1Ch
.text:77D2EC54 var_14           = dword ptr -14h
.text:77D2EC54 var_10           = dword ptr -10h
.text:77D2EC54 var_8            = dword ptr -8
.text:77D2EC54

.text:77D2EC54                 mov     edi, edi
.text:77D2EC56                 push    ebp
.text:77D2EC57                 mov     ebp, esp
.text:77D2EC59                 sub     esp, 30h
.text:77D2EC5C                 push    ebx
.text:77D2EC5D                 push    esi
.text:77D2EC5E                 push    edi
.text:77D2EC5F                 push    0Ch
.text:77D2EC61                 xor     eax, eax
.text:77D2EC63                 pop     ecx
.text:77D2EC64                 lea     edi, [ebp+var_30]
.text:77D2EC67                 rep stosd
.text:77D2EC69                 mov     eax, _hmodUser
.text:77D2EC6E                 xor     ebx, ebx
.text:77D2EC70                 inc     ebx
.text:77D2EC71                 xor     edi, edi
.text:77D2EC73                 mov     [ebp+var_30], 30h
.text:77D2EC7A                 mov     [ebp+var_1C], eax
.text:77D2EC7D                 xor     esi, esi
.text:77D2EC7D

.text:77D2EC7F

.text:77D2EC7F loc_77D2EC7F:

.text:77D2EC7F                 mov     eax, ds:dword_77D12C58[esi]---------->Win 标准控件类表,看下边。
.text:77D2EC85                 push    ds:dword_77D12C64[esi] ; lpCursorName
.text:77D2EC8B                 mov     [ebp+var_2C], eax
.text:77D2EC8E                 mov     eax, ds:off_77D12C5C[esi]
.text:77D2EC94                 mov     [ebp+var_28], eax
.text:77D2EC97                 mov     eax, ds:dword_77D12C60[esi]
.text:77D2EC9D                 push    edi               ; hInstance
.text:77D2EC9E                 mov     [ebp+var_20], eax
.text:77D2ECA1                 call    LoadCursorW(x,x)
.text:77D2ECA1

.text:77D2ECA6                 mov     [ebp+var_14], eax
.text:77D2ECA9                 mov     eax, ds:dword_77D12C68[esi]
.text:77D2ECAF                 mov     [ebp+var_10], eax
.text:77D2ECB2                 mov     eax, ds:off_77D12C6C[esi]
.text:77D2ECB8                 mov     [ebp+var_8], eax
.text:77D2ECBB                 xor     eax, eax
.text:77D2ECBD                 mov     ax, ds:word_77D12C70[esi]
.text:77D2ECC4                 push    edi
.text:77D2ECC5                 push    eax
.text:77D2ECC6                 push    edi
.text:77D2ECC7                 lea     eax, [ebp+var_30]
.text:77D2ECCA                 push    eax
.text:77D2ECCB                 call    RegisterClassExWOWW(x,x,x,x)---->RegisterClassEx 底层版,参数几乎都差不多,WNDCLASS 或 WNDCLASSEX，最终于调到 navtive api NtUserRegisterClassExWOW 完成应用层的注册.

.text:77D2ECCB

.text:77D2ECD0                 neg     ax
.text:77D2ECD3                 sbb     eax, eax
.text:77D2ECD5                 neg     eax
.text:77D2ECD7                 add     esi, 1Ch
.text:77D2ECDA                 and     ebx, eax
.text:77D2ECDC                 cmp     esi, 0FCh
.text:77D2ECE2                 jb      short loc_77D2EC7F------>循环完毕,注册定义的系统控件类。
.text:77D2ECE2

.text:77D2ECE4                 pop     edi
.text:77D2ECE5                 pop     esi
.text:77D2ECE6                 mov     eax, ebx
.text:77D2ECE8                 pop     ebx
.text:77D2ECE9                 leave
.text:77D2ECEA                 retn
.text:77D2ECEA

.text:77D2ECEA __stdcall RW_RegisterControls() endp
.text:77D2ECEA
```

> `use32.dll` 库中 `Win` 标准控件类表依次是：

> `Button` 类和其消息回调函数 `ButtonWndProcW`。 (UNICODE 版)
> `ComboLBox` 类和其消息回调函数 `ComboBoxWndProcW` 和 `ComboListBoxWndProcW` (UNICODE 版)
> `Edit` 类 对应 `EditWndProcW`
> `ListBox` 类对应 `ComboListBoxWndProcW`
> `MDIClient` 类 对应 `MDIClientWndProcW`
> `IME` 类对应 `ImeWndProcW`
> `Static` 类 `StaticWndProcW`

> 此表如下：

```
.text:77D12C58 dword_77D12C58   dd 408Bh                  ; DATA XREF: RW_RegisterControls():loc_77D2EC7F r
.text:77D12C5C off_77D12C5C     dd offset ButtonWndProcW(x,x,x,x)
.text:77D12C5C                  ; DATA XREF: RW_RegisterControls()+3A r
.text:77D12C60 dword_77D12C60   dd 4                      ; DATA XREF: RW_RegisterControls()+43 r
.text:77D12C64 dword_77D12C64   dd 7F00h                   ; DATA XREF: RW_RegisterControls()+31 r
.text:77D12C68 dword_77D12C68   dd 0                      ; DATA XREF: RW_RegisterControls()+55 r
.text:77D12C6C off_77D12C6C     dd offset s_Button         ; DATA XREF: RW_RegisterControls()+5E r
.text:77D12C6C                  ; "Button"
.text:77D12C70 word_77D12C70    dw 2A1h, 0, 408Bh, 0     ; DATA XREF: RW_RegisterControls()+69 r
.text:77D12C78                 dd offset ComboBoxWndProcW(x,x,x,x)
.text:77D12C7C                 dd 4, 7F00h, 0
.text:77D12C88                 dd offset s_Combobox     ; "ComboBox"
.text:77D12C8C                 dd 2A2h, 4808h
.text:77D12C94                 dd offset ComboListBoxWndProcW(x,x,x,x)
.text:77D12C98                 dd 4, 7F00h, 0
.text:77D12CA4                 dd offset s_Combolbox    ; "ComboLBox"
.text:77D12CA8                 dd 2A3h, 4808h
.text:77D12CB0                 dd offset DefDlgProcW(x,x,x,x)
.text:77D12CB4                 dd 1Eh, 7F00h, 0
.text:77D12CC0                 dd 8002h, 2A4h, 4088h
.text:77D12CCC                 dd offset EditWndProcW(x,x,x,x)
.text:77D12CD0                 dd 6, 7F01h, 0
.text:77D12CDC                 dd offset s_Edit         ; "Edit"
.text:77D12CE0                 dd 2A5h, 4088h
.text:77D12CE8                 dd offset ComboListBoxWndProcW(x,x,x,x)
.text:77D12CEC                 dd 4, 7F00h, 0
.text:77D12CF8                 dd offset s_Listbox      ; "ListBox"
.text:77D12CFC                 dd 2A6h, 4000h
.text:77D12D04                 dd offset MDIClientWndProcW(x,x,x,x)
.text:77D12D08                 dd 8, 7F00h, 0Dh
.text:77D12D14                 dd offset s_Mdiclient    ; "MDIClient"
.text:77D12D18                 dd 2A7h, 4000h
```



`.text:77D12D20` `dd` `offset` `ImeWndProcW(x,x,x,x)`  

`.text:77D12D24` `dd` `4,` `7F00h,` `0`  

`.text:77D12D30` `dd` `offset` `s_Ime` ; `"IME"`  

`.text:77D12D34` `db` `0A9h,` `2,` `2` `dup(0),` `88h,` `40h,` `2` `dup(0)`  

`.text:77D12D3C` `dd` `offset` `StaticWndProcW(x,x,x,x)`  

`.text:77D12D40` `dd` `4,` `7F00h,` `0`  

`.text:77D12D4C` `dd` `offset` `s_Static` ; `"Static"`  

`.text:77D12D50` `dd` `2A8h,` `0`  

呵呵，顺便把 `win2k` 代码复制出来一份给大家参考:  

```c
BOOL RW_RegisterControls(VOID)  
{  
   int        i;  
   WNDCLASSEX wndcls;  
   BOOL       fSuccess = TRUE;  
   
   static CONST struct {  
       UINT    style;  
       WNDPROC lpfnWndProcW;  
       int     cbWndExtra;  
       LPCTSTR lpszCursor;  
       HBRUSH  hbrBackground;  
       LPCTSTR lpszClassName;  
       WORD    fnid;----------------------------------->这个函数 ID 很有意思.我想这是微软定义区分系统控件的 WNDPROC 函数 ID,  
   } rc[] = {                                           //并且在 MessageTable 中也多次用这个 fnid 号来区分是哪个控件 WNDPROC  
                                                       // ，然后调用正确的 WNDPROC 地址。还有可能这个 MessageTable 表可以给窗体                              //回调函数进程 Hook 行为。还有这个定义的结构体与 WNDCLASS 和 WNDCLASSEX 有               // 区别 ！               
       {CS_GLOBALCLASS | CS_PARENTDC | CS_DBLCLKS | CS_HREDRAW | CS_VREDRAW,   
        ButtonWndProcW,  
        sizeof(BUTNWND) - sizeof(WND),  
        IDC_ARROW,  
        NULL,  
        L"Button",  
        FNID_BUTTON  
       },  
   
       {CS_GLOBALCLASS | CS_DBLCLKS | CS_PARENTDC | CS_VREDRAW | CS_HREDRAW,  
        ComboBoxWndProcW,  
        sizeof(COMBOWND) - sizeof(WND),  
        IDC_ARROW,  
        NULL,  
        L"ComboBox",  
        FNID_COMBOBOX  
       },  
   
       {CS_GLOBALCLASS | CS_DBLCLKS | CS_SAVEBITS,  
        ComboListBoxWndProcW,  
        sizeof(LBWND) - sizeof(WND),  
        IDC_ARROW,  
        NULL,  
        L"ComboLBox",  
        FNID_COMBOLISTBOX  
       },  
   
       {CS_GLOBALCLASS | CS_DBLCLKS | CS_SAVEBITS,  
        DefDlgProcW,  
        DLGWINDOWEXTRA,  
        IDC_ARROW,  
        NULL,  
        DIALOGCLASS,  
        FNID_DIALOG  
       },  
   
       {CS_GLOBALCLASS | CS_PARENTDC | CS_DBLCLKS,  
        EditWndProcW,  
        max((sizeof(EDITWND) - sizeof(WND)), CBEDITEXTRA),  
        IDC_IBEAM,  
        NULL,  
        L"Edit",  
        FNID_EDIT  
       },  
   
       {CS_GLOBALCLASS | CS_PARENTDC | CS_DBLCLKS,  
        ListBoxWndProcW,  
        sizeof(LBWND) - sizeof(WND),  
        IDC_ARROW,  
        NULL,  
        L"ListBox",  
        FNID_LISTBOX  
       },  
   
       {CS_GLOBALCLASS,  
        MDIClientWndProcW,  
        sizeof(MDIWND) - sizeof(WND),  
        IDC_ARROW,  
        (HBRUSH)(COLOR_APPWORKSPACE + 1),  
        L"MDIClient",  
        FNID_MDICLIENT  
       },  
   
       {CS_GLOBALCLASS,  
        ImeWndProcW,  
        sizeof(IMEWND) - sizeof(WND),  
        IDC_ARROW,  
        NULL,  
        L"IME",  
        FNID_IME  
       },  
   
       {CS_GLOBALCLASS | CS_PARENTDC | CS_DBLCLKS,  
        StaticWndProcW,  
        sizeof(STATWND) - sizeof(WND),  
        IDC_ARROW,  
        NULL,  
        L"Static",  
        FNID_STATIC  
       }  
   };  
   
   
   /*  
    * Classes are registered via the table.  
    */  
   RtlZeroMemory(&wndcls, sizeof(wndcls));  
   wndcls.cbSize       = sizeof(wndcls);  
   wndcls.hInstance    = hmodUser;  
   
   for (i = 0; i < (sizeof(rc)/sizeof(rc[0])); i++) {  
       wndcls.style        = rc[i].style;  
       wndcls.lpfnWndProc  = rc[i].lpfnWndProcW;  
       wndcls.cbWndExtra   = rc[i].cbWndExtra;  
       wndcls.hCursor      = LoadCursor(NULL, rc[i].lpszCursor);  
       wndcls.hbrBackground= rc[i].hbrBackground;  
       wndcls.lpszClassName= rc[i].lpszClassName;  
   
       fSuccess &= !!RegisterClassExWOWW(&wndcls, NULL, rc[i].fnid);  
   }  
   
   if (!fSuccess) {  
       RIPMSG0(RIP_WARNING, "RW_RegisterControls failed to register classes");  
   }  
   
   return fSuccess;  
}  
```  

`Win2k` 头文件 `user.h` 中找了 `fnid` 的说明,`fnid` 更类似(窗口类的)索引,有起始范围。有兴趣的可以看看.  

```c
/* 
 * Server side address constants. When we want to call a server side proc, 
 * we pass an index indentifying the function, rather than the server side 
 * address itself. More robust.  The functions between WNDPROCSTART/END 
 * have client side sutbs which map to this routines. 
 * 
 * Adding a new FNID (This is just what I figured out...so fix it if wrong or incomplete) 
 * -Decide what range it should be in: 
 *      FNID_WNDPROCSTART to FNID_WNDPROCEND: Server side proc with client 
 *        stub 
 *      FIND_CONTROLSTART to FNID_CONTROLEND: Client side controls with no 
 *        server side proc 
 *      After FNID_CONTROLEND: other, like server side only procs or client 
 *        side only.... 
 * -Make sure to adjust FNID_*START and FNID_*END appropriately. 
 * -If the ID is to be associated with a window class, and it is for all 
 *    windows of the class, make sure that the InternalRegisterClassEx call 
 *    receives the id as a parameter. 
 * -If in FNID_WNDPROCSTART-END range, make the proper STOCID call in InitFunctionTables. 
 * -Add proper FNID call in InitFunctionTables. 
 * -If the class has a client side worker function (pcls->lpfnWorker) or you expect 
 *   apps to send messages to it or call its window proc directly, define 
 *   a message table in kernel\server.c and initialize it in InitMessageTables. 
 * -If there is a client side for this proc, you probably need to add it to 
 *   PFNCLIENT. 
 * -Add the debug-only text description of this FNID to in gapszFNID in globals.c 
 * -See if you need to modify aiClassWow in client\client.c 
 * -Modify the gaFNIDtoICLS table in kernel\ntstubs.c 
 */ 
#define FNID_START                 0x0000029A 
#define FNID_WNDPROCSTART          0x0000029A 
 
#define FNID_SCROLLBAR             0x0000029A      // xxxSBWndProc; 
#define FNID_ICONTITLE             0x0000029B      // xxxDefWindowProc; 
#define FNID_MENU                  0x0000029C      // xxxMenuWindowProc; 
#define FNID_DESKTOP               0x0000029D      // xxxDesktopWndProc; 
#define FNID_DEFWINDOWPROC         0x0000029E      // xxxDefWindowProc; 
 
#define FNID_WNDPROCEND            0x0000029E      // see PatchThreadWindows 
#define FNID_CONTROLSTART          0x0000029F 
 
#define FNID_BUTTON                0x0000029F      // No server side proc 
#define FNID_COMBOBOX              0x000002A0      // No server side proc 
#define FNID_COMBOLISTBOX          0x000002A1      // No server side proc 
#define FNID_DIALOG                0x000002A2      // No server side proc 
#define FNID_EDIT                  0x000002A3      // No server side proc 
#define FNID_LISTBOX               0x000002A4      // No server side proc 
#define FNID_MDICLIENT             0x000002A5      // No server side proc 
#define FNID_STATIC                0x000002A6      // No server side proc 
```



```
#define FNID_IME                    0x000002A7      // No server side proc

#define FNID_CONTROLEND             0x000002A7

#define FNID_HKINLPCWPEXSTRUCT      0x000002A8

#define FNID_HKINLPCWPRETEXSTRUCT    0x000002A9

#define FNID_DEFFRAMEPROC           0x000002AA      // No server side proc

#define FNID_DEFMDICHILDPROC        0x000002AB      // No server side proc

#define FNID_MB_DLGPROC             0x000002AC      // No server side proc

#define FNID_MDIACTIVATEDLGPROC     0x000002AD      // No server side proc

#define FNID_SENDMESSAGE            0x000002AE

#define FNID_SENDMESSAGEFF          0x000002AF

#define FNID_SENDMESSAGEEX          0x000002B0

#define FNID_CALLWINDOWPROC         0x000002B1

#define FNID_SENDMESSAGEBSM         0x000002B2

#define FNID_SWITCH                 0x000002B3      // Just used by GetTopMostInserAfter

#define FNID_TOOLTIP                0x000002B4

#define FNID_END                    0x000002B4
```

-----------------------------------------------------------------------------------

注册完了系统控件类，然后随便拉个控件 `Button` 类的回调看看,只是感兴趣：

```
text:77D2614C  ; int __stdcall ButtonWndProcW(int,UINT Msg,int wParam,int lParam)
.text:77D2614C __stdcall ButtonWndProcW(x, x, x, x) proc near
.text:77D2614C                                             
.text:77D2614C arg_0               = dword ptr  8
.text:77D2614C Msg                 = dword ptr  0Ch
.text:77D2614C wParam              = dword ptr  10h
.text:77D2614C lParam              = dword ptr  14h
.text:77D2614C

.text:77D2614C                 mov     edi, edi
.text:77D2614E                 push    ebp
.text:77D2614F                 mov     ebp, esp
.text:77D26151                 mov     ecx, [ebp+arg_0]
.text:77D26154                 push    esi
.text:77D26155                 call    ValidateHwnd(x)
.text:77D26155

.text:77D2615A                 mov     esi, eax
.text:77D2615C                 test    esi, esi
.text:77D2615E                 jz      short loc_77D26198
.text:77D2615E

.text:77D26160                 mov     edx, [ebp+Msg]
.text:77D26163                 cmp     edx, dword_77D700E8
.text:77D26169                 ja      short loc_77D26189
.text:77D26169

.text:77D2616B                 xor     eax, eax
.text:77D2616D                 mov     ecx, edx
.text:77D2616F                 and     ecx, 7
.text:77D26172                 inc     eax
.text:77D26173                 shl     eax, cl
.text:77D26175                 push    edi
.text:77D26176                 mov     edi, dword_77D700EC
.text:77D2617C                 mov     ecx, edx
.text:77D2617E                 shr     ecx, 3
.text:77D26181                 mov     cl, [ecx+edi]
.text:77D26184                 test    al, cl
.text:77D26186                 pop     edi
.text:77D26187                 jnz     short loc_77D2619D
.text:77D26187

.text:77D26189
.text:77D26189 loc_77D26189:                        
.text:77D26189                 push    0               ; int
.text:77D2618B                 push    [ebp+lParam]    ; lParam
.text:77D2618E                 push    [ebp+wParam]    ; wParam
.text:77D26191                 push    edx             ; Msg
.text:77D26192                 push    esi             ; int
.text:77D26193                 call    DefWindowProcWorker(x,x,x,x,x)
.text:77D26193

.text:77D26198
.text:77D26198 loc_77D26198:                       

.text:77D26198                 pop     esi
.text:77D26199                 pop     ebp
.text:77D2619A                 retn    10h
.text:77D2619A

.text:77D2619D ; -------------------------------------------------------------
.text:77D2619D

.text:77D2619D loc_77D2619D:                        
.text:77D2619D                 push    0               ; int
.text:77D2619F                 push    [ebp+lParam]    ; int
.text:77D261A2                 push    [ebp+wParam]    ; int
.text:77D261A5                 push    edx             ; Msg
.text:77D261A6                 push    esi             ; int
.text:77D261A7                 call    ButtonWndProcWorker(x,x,x,x,x)------>
.text:77D261A7

.text:77D261AC                 jmp     short loc_77D26198
.text:77D261AC

.text:77D261AC __stdcall ButtonWndProcW(x, x, x, x) endp
```

参考它的 win32k 代码如下:

`\\win2k\\private\\ntos\\w32\\ntuser\\client\\btnctl.c`

```
LRESULT WINAPI ButtonWndProcW(  
    HWND hwnd,  
    UINT message,  
    WPARAM wParam,  
    LPARAM lParam)  
{  
    PWND pwnd;  

    if ((pwnd = ValidateHwnd(hwnd)) == NULL) {  
        return (0L);  
    }  

    /*
     * If the control is not interested in this message,
     * pass it to DefWindowProc.
     */  
    if (!FWINDOWMSG(message, FNID_BUTTON))------------------>这里识别 FNID_BUTTON ID 了.  
        return DefWindowProcWorker(pwnd, message, wParam, lParam, FALSE);  

    return ButtonWndProcWorker(pwnd, message, wParam, lParam, FALSE);  
}
```

`ButtonWndProcWorker` 这个方法处理控件 `Button` 的所关心 Window 消息,本身的控件消息,硬件消息等,  
精简了一下,目的是看看它处理的消息。有兴趣的朋友查看`\\win2k\\private\\ntos\\w32\\ntuser\\client\\btnctl.c`。

```
RESULT APIENTRY ButtonWndProcWorker(  
    PWND pwnd,  
    UINT message,  
    WPARAM wParam,  
    LPARAM lParam,  
    DWORD fAnsi)  
{  
    switch (message) {  
    case WM_NCHITTEST:  
        ...
          break;  

    case WM_ERASEBKGND:  
       ...
        return (LONG)TRUE;  

    case WM_PRINTCLIENT:  
        xxxBNPaint(pbutn, (HDC)wParam);  
        break;  

    case WM_PAINT:  
          ....
        break;  

    case WM_SETFOCUS:  
       ....
        break;  

    case WM_GETDLGCODE:    
    case WM_CAPTURECHANGED:  
        ....
        break;  

    case WM_KILLFOCUS:  

       ......
          break;  
    case WM_LBUTTONDBLCLK:  
          ....
    case WM_LBUTTONUP:  
        if (BUTTONSTATE(pbutn) & BST_MOUSE) {  
            xxxBNReleaseCapture(pbutn, TRUE);  
        }  
        break;  

    case WM_MOUSEMOVE:  
       ......
          break;  
    case WM_LBUTTONDOWN:  
         ......
          break;  

    case WM_CHAR:  
     ......
          break;  

    case BM_CLICK:  
       ......
          break;  
          
    case WM_KEYDOWN:  
       ......
          break;  

    case WM_KEYUP:  
    case WM_SYSKEYUP:  
          
        ......
          break;  

    case BM_GETSTATE:  
       
       ......
          break;  
    case BM_SETSTATE:  
         ......
          break;  
          
  
    case BM_GETCHECK:  
    case BM_SETCHECK:  
       
       ......
          break;  

    case BM_SETSTYLE:  
       ......
          break;  
    case WM_SETTEXT:  
    case WM_ENABLE:  
       ......
          break;  

    case WM_SETFONT:  
       ......
          break;  
    case WM_GETFONT:  
        ......
          break;  
    case BM_GETIMAGE:  
    case BM_SETIMAGE:  
       ......
          break;  
    case WM_NCDESTROY:  
    case WM_FINALDESTROY:  
       ....
       break;  

    case WM_NCCREATE:  
        ....
      break;
```



```c
    case WM_INPUTLANGCHANGEREQUEST:
      ...
    break;

    case WM_UPDATEUISTATE:
     ...
       break;

    default:
    CallDWP:
       return DefWindowProcWorker(pwnd, message, wParam, lParam, fAnsi);
    }

    return 0L;
}
```

===================================================================================

另外 `user32` 有两个辅助表用于继续分发消息，比较有意思！

一个是消息表: `MessageTable`。记的笨笨雄也以前提出来过这个东东！

一个是消息所对应跳转的事件表: `gapfnScSendMessage` 。

两个表在下面给出：

首先根据消息的 `ID` 在 `MessageTable` 表取出消息 `ID` 所对应的 `sig`,然后与 `3F` 相加,

得出的结果是它所对应事件表 `gapfnScSendMessage` 的索引然后跳到到它的事件地址中。

随便找个消息分发函数比如：`DefWindowProc`--->`RealDefWindowProc`--> `RealDefWindowProcWorker`，

真正用到 `MessageTable` 索引是到了 `RealDefWindowProcWorker` 这个函数中.`win2k` 代码好象没有找到个函数！

这个过程用 `od` 跟不错,`od` 跟踪到这里：

```asm
77D1B3FD    81FF 00040000   cmp     edi, 400------------>如果消息 ID 超出 WM_USER(0x0400)，不用 MessageTalbe
77D1B403    FF75 18         push    dword ptr [ebp+18]----WParam
77D1B406    68 9E020000     push    29E
77D1B40B    6A 00           push    0
77D1B40D    FF75 14         push    dword ptr [ebp+14]--->LPRAM
77D1B410    FF75 10         push    dword ptr [ebp+10]
77D1B413    57              push    edi------->msg
77D1B414    50              push    eax
77D1B415    0F83 4AF70200   jnb     77D4AB65----->上面的判断跳转
77D1B41B    33C9            xor     ecx, ecx
77D1B41D    8A8F E814D177   mov     cl, byte ptr [edi+77D114E8]---->7D114E8 就是 MessageTable 表，edi 是消息 ID 号！
77D1B423    83E1 3F         and     ecx, 3F------>运算一下
77D1B426    FF148D E818D177 call    dword ptr [ecx*4+77D118E8]  ------->77D118E8 就是 gapfnScSendMessage 的地址,ecx*4 它的索引！
77D1B42D  ^ E9 FAFEFFFF     jmp     77D1B32C
```

上面汇编对应的 `win2k` 代码()：

```c
  pmsg->message >= WM_USER ? (ULONG_PTR)SfnDWORD :
               (ULONG_PTR)gapfnScSendMessage[MessageTable[pmsg->message].iFunction];
```

`win2k` 定义 `MessageTable` 表部分如下(`message.h`)：

```c
CONST MSG_TABLE_ENTRY MessageTable[] = {
    {IMSG_DWORD, FALSE, FALSE},               // WM_NULL                  0x0000
    {IMSG_INLPCREATESTRUCT,  TRUE,  TRUE},     // WM_CREATE                0x0001
    {IMSG_DWORD, FALSE, FALSE},               // WM_DESTROY               0x0002
    {IMSG_DWORD, FALSE, FALSE},               // WM_MOVE                  0x0003
    {IMSG_DWORD, FALSE, FALSE},               // WM_SIZEWAIT              0x0004
    {IMSG_DWORD, FALSE, FALSE},               // WM_SIZE                  0x0005
    {IMSG_DWORD, FALSE, FALSE},               // WM_ACTIVATE              0x0006
    {IMSG_DWORD, FALSE, FALSE},               // WM_SETFOCUS              0x0007
    {IMSG_DWORD, FALSE, FALSE},               // WM_KILLFOCUS             0x0008
    {IMSG_DWORD, FALSE, FALSE},               // WM_SETVISIBLE            0x0009
    {IMSG_DWORD, FALSE, FALSE},               // WM_ENABLE                0x000A
    {IMSG_DWORD, FALSE, FALSE},               // WM_SETREDRAW             0x000B
    {IMSG_INSTRINGNULL,  TRUE,  TRUE},        // WM_SETTEXT               0x000C
    {IMSG_OUTSTRING,  TRUE,  TRUE},           // WM_GETTEXT               0x000D
    {IMSG_GETDBCSTEXTLENGTHS,  TRUE,  TRUE},  // WM_GETTEXTLENGTH         0x000E
    {IMSG_DWORD, FALSE, FALSE},               // WM_PAINT                 0x000F

    {IMSG_DWORD, FALSE, FALSE},               // WM_CLOSE                 0x0010
    {IMSG_DWORD, FALSE, FALSE},               // WM_QUERYENDSESSION       0x0011
    {IMSG_DWORD, FALSE, FALSE},               // WM_QUIT                  0x0012
    {IMSG_DWORD, FALSE, FALSE},               // WM_QUERYOPEN             0x0013
    {IMSG_DWORD, FALSE,  TRUE},               // WM_ERASEBKGND            0x0014
    {IMSG_DWORD, FALSE, FALSE},               // WM_SYSCOLORCHANGE        0x0015
    {IMSG_DWORD, FALSE, FALSE},               // WM_ENDSESSION            0x0016
    {IMSG_DWORD, FALSE, FALSE},               // WM_SYSTEMERROR           0x0017
    {IMSG_DWORD, FALSE, FALSE},               // WM_SHOWWINDOW            0x0018
    {IMSG_RESERVED, FALSE, FALSE},            // WM_CTLCOLOR              0x0019
    {IMSG_INSTRINGNULL,  TRUE,  TRUE},        // WM_WININICHANGE          0x001A
    {IMSG_INSTRING,  TRUE,  TRUE},            // WM_DEVMODECHANGE         0x001B
    {IMSG_DWORD, FALSE, FALSE},               // WM_ACTIVATEAPP           0x001C
    {IMSG_DWORD, FALSE, FALSE},               // WM_FONTCHANGE            0x001D
    {IMSG_DWORD, FALSE, FALSE},               // WM_TIMECHANGE            0x001E
    {IMSG_DWORD, FALSE, FALSE},               // WM_CANCELMODE            0x001F

    {IMSG_DWORD, FALSE, FALSE},               // WM_SETCURSOR             0x0020
    {IMSG_DWORD, FALSE, FALSE},               // WM_MOUSEACTIVATE         0x0021
    {IMSG_DWORD, FALSE, FALSE},               // WM_CHILDACTIVATE         0x0022
    {IMSG_DWORD, FALSE, FALSE},               // WM_QUEUESYNC             0x0023
    {IMSG_INOUTLPPOINT5, FALSE,  TRUE},       // WM_GETMINMAXINFO         0x0024
    {IMSG_EMPTY, FALSE, FALSE},               // empty                    0x0025
    {IMSG_DWORD, FALSE, FALSE},               // WM_PAINTICON             0x0026
    {IMSG_DWORD, FALSE,  TRUE},               // WM_ICONERASEBKGND        0x0027
    {IMSG_DWORD, FALSE, FALSE},               // WM_NEXTDLGCTL            0x0028
    {IMSG_DWORD, FALSE, FALSE},               // WM_ALTTABACTIVE          0x0029
    {IMSG_DWORD, FALSE, FALSE},               // WM_SPOOLERSTATUS         0x002A
    {IMSG_INLPDRAWITEMSTRUCT, FALSE,  TRUE},   // WM_DRAWITEM              0x002B
    {IMSG_INOUTLPMEASUREITEMSTRUCT, FALSE,  TRUE},// WM_MEASUREITEM      0x002C
    {IMSG_INLPDELETEITEMSTRUCT, FALSE,  TRUE}, // WM_DELETEITEM           0x002D
    {IMSG_DWORD, FALSE, FALSE},               // WM_VKEYTOITEM            0x002E
    {IMSG_INWPARAMCHAR,  TRUE, FALSE},        // WM_CHARTOITEM            0x002F

    {IMSG_DWORD, FALSE, FALSE},               // WM_SETFONT               0x0030
    {IMSG_DWORD, FALSE,  TRUE},               // WM_GETFONT               0x0031
    {IMSG_DWORD, FALSE, FALSE},               // WM_SETHOTKEY             0x0032
    {IMSG_DWORD, FALSE, FALSE},               // WM_GETHOTKEY             0x0033
    {IMSG_DWORD, FALSE, FALSE},               // WM_FILESYSCHANGE          0x0034
    {IMSG_DWORD, FALSE, FALSE},               // WM_ISACTIVEICON          0x0035
    {IMSG_DWORD, FALSE, FALSE},               // WM_QUERYPARKICON         0x0036
    {IMSG_DWORD, FALSE, FALSE},               // WM_QUERYDRAGICON         0x0037
    {IMSG_INLPHLPSTRUCT, FALSE,  TRUE},       // WM_WINHELP               0x0038
    {IMSG_INLPCOMPAREITEMSTRUCT, FALSE,  TRUE}, // WM_COMPAREITEM         0x0039
    {IMSG_DWORD, FALSE, FALSE},               // WM_FULLSCREEN            0x003A
    {IMSG_DWORD, FALSE, FALSE},               // WM_CLIENTSHUTDOWN        0x003B
    {IMSG_KERNELONLY, FALSE, TRUE},           // WM_DDEMLEVENT            0x003C
    {IMSG_EMPTY, FALSE, FALSE},               // empty                    0x003D
}
```



```c
   {IMSG_EMPTY, FALSE, FALSE},                 // empty                     0x003E  
   {IMSG_DWORD, FALSE, FALSE},                 // MM_CALCSCROLL            0x003F  
   
   {IMSG_RESERVED, FALSE, FALSE},              // WM_TESTING               0x0040  
   {IMSG_DWORD, FALSE, FALSE},                 // WM_COMPACTING            0x0041  
   
   {IMSG_RESERVED, FALSE, FALSE},              // WM_OTHERWINDOWCREATED    0x0042  
   {IMSG_RESERVED, FALSE, FALSE},              // WM_OTHERWINDOWDESTROYED  0x0043  
   {IMSG_RESERVED, FALSE, FALSE},              // WM_COMMNOTIFY            0x0044  
   {IMSG_RESERVED, FALSE, FALSE},              // WM_MEDIASTATUSCHANGE     0x0045  
   {IMSG_INOUTLPWINDOWPOS, FALSE,  TRUE},      // WM_WINDOWPOSCHANGING     0x0046  
   {IMSG_INLPWINDOWPOS, FALSE,  TRUE},         // WM_WINDOWPOSCHANGED      0x0047  
   
   {IMSG_RESERVED, FALSE, FALSE},              // WM_POWER                 0x0048  
   {IMSG_COPYGLOBALDATA,  TRUE,  TRUE},        // WM_COPYGLOBALDATA        0x0049  
   {IMSG_COPYDATA, FALSE,  TRUE},              // WM_COPYDATA              0x004A  
   {IMSG_RESERVED, FALSE, FALSE},              // WM_CANCELJOURNAL         0x004B  
   {IMSG_LOGONNOTIFY, FALSE, FALSE},           // WM_LOGONNOTIFY           0x004C  
   {IMSG_DWORD, FALSE, FALSE},                 // WM_KEYF1                 0x004D  
   {IMSG_DWORD, FALSE, FALSE},                 // WM_NOTIFY                0x004E  
   {IMSG_RESERVED, FALSE, FALSE},              // WM_ACCESS_WINDOW         0x004f  
   
  ... 
  ... 
  ... 
}
```

对应跳转的事件表:`gapfnScSendMessage`

```c
/*
 * Message thunks.
 */
typedef LRESULT (APIENTRY *SFNSCSENDMESSAGE)(PWND, UINT, WPARAM, LPARAM,   //定义在 userk.h
        ULONG_PTR, PROC, DWORD, PSMS);

extern CONST SFNSCSENDMESSAGE gapfnScSendMessage[];//定义在 globals.h
```

`gapfnScSendMessage` 表如下：

```asm
.text:77D118E8 gapfnScSendMessage dd offset NtUserMessageCall(x,x,x,x,x,x,x)
.text:77D118EC                dd offset NtUserMessageCall(x,x,x,x,x,x,x)
.text:77D118F0                dd offset NtUserMessageCall(x,x,x,x,x,x,x)
.text:77D118F4                dd offset NtUserMessageCall(x,x,x,x,x,x,x)
.text:77D118F8                dd offset NtUserMessageCall(x,x,x,x,x,x,x)
.text:77D118FC                dd offset NtUserMessageCall(x,x,x,x,x,x,x)
.text:77D11900                dd offset NtUserMessageCall(x,x,x,x,x,x,x)
.text:77D11904                dd offset NtUserMessageCall(x,x,x,x,x,x,x)
.text:77D11908                dd offset NtUserMessageCall(x,x,x,x,x,x,x)
.text:77D1190C                dd offset NtUserMessageCall(x,x,x,x,x,x,x)
.text:77D11910                dd offset NtUserMessageCall(x,x,x,x,x,x,x)
.text:77D11914                dd offset NtUserMessageCall(x,x,x,x,x,x,x)
.text:77D11918                dd offset NtUserMessageCall(x,x,x,x,x,x,x)
.text:77D1191C off_77D1191C    dd offset NtUserMessageCall(x,x,x,x,x,x,x)
.text:77D1191C                                ; DATA XREF: RealDefWindowProcWorker(x,x,x,x,x)+1DF0 r
.text:77D11920                dd offset NtUserMessageCall(x,x,x,x,x,x,x)
.text:77D11924                dd offset fnCOPYGLOBALDATA(x,x,x,x,x,x,x)
.text:77D11928                dd offset NtUserMessageCall(x,x,x,x,x,x,x)
.text:77D1192C                dd offset NtUserMessageCall(x,x,x,x,x,x,x)
.text:77D11930                dd offset NtUserMessageCall(x,x,x,x,x,x,x)
.text:77D11934                dd offset NtUserMessageCall(x,x,x,x,x,x,x)
.text:77D11938                dd offset NtUserMessageCall(x,x,x,x,x,x,x)
.text:77D1193C                dd offset NtUserMessageCall(x,x,x,x,x,x,x)
.text:77D11940                dd offset NtUserMessageCall(x,x,x,x,x,x,x)
.text:77D11944                dd offset NtUserMessageCall(x,x,x,x,x,x,x)
.text:77D11948                dd offset NtUserMessageCall(x,x,x,x,x,x,x)
.text:77D1194C                dd offset NtUserMessageCall(x,x,x,x,x,x,x)
.text:77D11950                dd offset NtUserMessageCall(x,x,x,x,x,x,x)
.text:77D11954                dd offset NtUserMessageCall(x,x,x,x,x,x,x)
.text:77D11958                dd offset NtUserMessageCall(x,x,x,x,x,x,x)
.text:77D1195C                dd offset NtUserMessageCall(x,x,x,x,x,x,x)
.text:77D11960                dd offset NtUserMessageCall(x,x,x,x,x,x,x)
.text:77D11964                dd offset NtUserMessageCall(x,x,x,x,x,x,x)
.text:77D11968                dd offset NtUserMessageCall(x,x,x,x,x,x,x)
.text:77D1196C                dd offset NtUserMessageCall(x,x,x,x,x,x,x)
.text:77D11970                dd offset NtUserMessageCall(x,x,x,x,x,x,x)
.text:77D11974                dd offset fnINDEVICECHANGE(x,x,x,x,x,x,x)
.text:77D11978                dd offset NtUserMessageCall(x,x,x,x,x,x,x)
.text:77D1197C                dd offset NtUserMessageCall(x,x,x,x,x,x,x)
.text:77D11980                dd offset fnINSIZECLIPBRD(x,x,x,x,x,x,x)
.text:77D11984                dd offset fnINSIZECLIPBRD(x,x,x,x,x,x,x)
.text:77D11988                dd offset NtUserMessageCall(x,x,x,x,x,x,x)
.text:77D1198C                dd offset NtUserMessageCall(x,x,x,x,x,x,x)
.text:77D11990                dd offset NtUserMessageCall(x,x,x,x,x,x,x)
.text:77D11994                dd offset NtUserMessageCall(x,x,x,x,x,x,x)
.text:77D11998                dd offset NtUserMessageCall(x,x,x,x,x,x,x)
.text:77D1199C                dd offset fnCBGETEDITSEL(x,x,x,x,x,x,x)
.text:77D119A0                dd offset fnEMSETSEL(x,x,x,x,x,x,x)
.text:77D119A4                dd offset fnINWPARAMDBCSCHAR(x,x,x,x,x,x,x)
.text:77D119A8                dd offset fnCBGETEDITSEL(x,x,x,x,x,x,x)
.text:77D119AC                dd offset fnIMECONTROL(x,x,x,x,x,x,x)
.text:77D119B0                dd offset NtUserMessageCall(x,x,x,x,x,x,x)
.text:77D119B4                dd offset NtUserMessageCall(x,x,x,x,x,x,x)
.text:77D119B8                dd offset NtUserMessageCall(x,x,x,x,x,x,x)
.text:77D119BC                dd offset NtUserMessageCall(x,x,x,x,x,x,x)
.text:77D119C0                dd offset NtUserMessageCall(x,x,x,x,x,x,x)
.text:77D119C4                dd offset NtUserMessageCall(x,x,x,x,x,x,x)
```

`NtUserMessageCall` 是个 native api，它又跑到系统核心层去了。其他的 `fnCBGETEDITSEL`，`fnIMECONTROL` 等感兴趣的自己跟或者查看 2k 代码。

最后给出 `DefWindowProcWorker` 关系处理的 win 消息,感兴趣的可以看看精简了一下：

```c
LRESULT DefWindowProcWorker(          //定义在 clmsg.c 中
    PWND pwnd,
    UINT message,
    WPARAM wParam,
    LPARAM lParam,
    DWORD fAnsi)
{
    
    switch (message) {

    case WM_HELP:
        
        return(0L);
      
    case WM_MOUSEWHEEL:
        
        break;
        ......
    case WM_CONTEXTMENU:
      
        break;
      ......
    case WM_RBUTTONUP:
    case WM_APPCOMMAND:
      ......
        break;
    case WM_NCXBUTTONUP:
    case WM_XBUTTONUP:
        
                break;
  
    case WM_WINDOWPOSCHANGED: {
        PWINDOWPOS ppos = (PWINDOWPOS)lParam;

        return 0;
        }

    case WM_MOUSEACTIVATE: {
      ......

        return ((LOWORD(lParam) == HTCAPTION) && (HIWORD(lParam) == WM_LBUTTONDOWN )) ?
               (LONG)MA_NOACTIVATE : (LONG)MA_ACTIVATE;
        }

    case WM_CTLCOLORESCROLLBAR:
      
          return((LRESULT)gpsi->hbrGray);
```



```c
    }

    case WM_CTLCOLOREDIT:
        goto SetColor;

    case WM_CTLCOLOREDIT:
    case WM_CTLCOLORDLG:
    case WM_CTLCOLORMSGBOX:
         ......
    case WM_CTLCOLOREDIT:             // here for WOW only
    case WM_CTLCOLOREDIT:
    case WM_CTLCOLOREDIT:
        ......
    case WM_NCHITTEST:
        return FindNCHit(pwnd, (LONG)lParam);

    case WM_GETTEXT:
             ......
                 return cchSrc;
            
    case WM_GETTEXTLENGTH:
        ......
            return cch;
        }
        return 0L;

    case WM_QUERYDRAGICON:
       ......
        return (LRESULT)LoadIconW(pwnd->hModule, MAKEINTRESOURCE(1));

    case WM_QUERYOPEN:
    case WM_QUERYENDSESSION:
    case WM_DEVICECHANGE:
    case WM_POWERBROADCAST:
        return TRUE;

    case WM_KEYDOWN:
        if (wParam == VK_F10) {
            return CsSendMessage(hwnd, message, wParam, lParam, 0L,
                    FNID_DEFWINDOWPROC, fAnsi);
        }
        break;

    case WM_SYSKEYDOWN:
        if ((HIWORD(lParam) & SYS_ALTERNATE) || (wParam == VK_F10) ||
                (wParam == VK_ESCAPE))
            return CsSendMessage(hwnd, message, wParam, lParam, 0L,
                    FNID_DEFWINDOWPROC, fAnsi);
        break;

    case WM_CHARTOITEM:
    case WM_VKEYTOITEM:
        /*
         * Do default processing for keystrokes into owner draw listboxes.
         */
        return -1;

    case WM_ACTIVATE:
        if (LOWORD(wParam))
            return CsSendMessage(hwnd, message, wParam, lParam, 0L,
                    FNID_DEFWINDOWPROC, fAnsi);
        break;

    case WM_SHOWWINDOW:
        if (lParam != 0)
            return CsSendMessage(hwnd, message, wParam, lParam, 0L,
                    FNID_DEFWINDOWPROC, fAnsi);
        break;

    case WM_DROPOBJECT:
       return DO_DROPFILE;

    case WM_WINDOWPOSCHANGING:
        /*
         * If the window's size is changing, adjust the passed-in size
         */
        #define ppos ((WINDOWPOS *)lParam)
        if (!(ppos->flags & SWP_NOSIZE))
            return CsSendMessage(hwnd, message, wParam, lParam, 0L,
                    FNID_DEFWINDOWPROC, fAnsi);
        #undef ppos
        break;

    case WM_KLUDGEMINRECT:
         ......
        break;

       .........................
}
```

其他的公共控件见 `comdlg32` 等库。感兴趣可以自己研究一下，虽然 `2k` 代码有点老，不过参考价值还是很大的！

基本完毕了，没有明确的目标，纯属兴趣！

我是一只小菜鸟，多谢大家指教！！！

`4dd466d0e1e0cc123afbe504da2462ca`
