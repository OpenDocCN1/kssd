
# 星际霸主 1.9j 分析

> 原文：[`www.pediy.com/kssd/pediy12/128871.html`](https://www.pediy.com/kssd/pediy12/128871.html)

【文章标题】: 星际霸主 1.9j 分析  
【文章作者】: `thomasyzh`  
【作者邮箱】: `machinesy@gmail.com`  
【软件名称】: 星际霸主 1.9j  
【软件大小】: 5.16 MB  
【下载地址】: 自己搜索下载  
【加壳方式】: `Themida/WinLicense` V1.8.X-V1.9.X DLL  
【软件介绍】: 一个星际争霸 1 的游戏外挂  
【作者声明】: 只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教!

--------------------------------------------------------------------------------

【详细过程】

这是一个破解任务，一位朋友请我帮他破解，价格是 1K。因为不是为了赚这个票子，所以就直接发表出来了。在发表之前

我是在考虑这个帖子的标题写软件破解，还是软件分析。因为实际上现在有两个方法完成这个破解，1 是继续按照我现在的

思路把这个东西做完了。2 是我去把 `Virtualizer` 加的虚拟器代码还原掉了。实际上我选择了 2，所以我就没有继续把 1 的路子

走完了。最后我把改 `vm` 代码还原后，一定的能够破译掉。

那么，我把 1 的分析过程写出来，所以我把这文章的标题就写做分析了。

一些文件啊，咋的咋的那些信息我就不说了。我直接说结论。

1. 这是个大量使用 `hook` 挂接星际争霸的游戏外挂。
2. 这个外挂是使用 `http` 验证的一个外挂。
3. 这个外挂的验证在 `scHelper.king` 的代码里边。
4. 这个外挂的数据收发在 `scking.exe` 里边，然后通过内存共享传送到 `scHelper.king` 里边。大概应该在那个 `sckhook.dll` 里边。

因为加了 `tmd` 壳。我并没有选择一开始就 `CreateProcess` 这种方式，而是让它在内存里解密完，并且 `anti` 代码跑完的石斛 `attack`

上去。最佳的效果是直接 `attack` 到加密验证代码那里。

于是

1. `Hook` 一个函数，这个函数必须要是 `scHelper.king` 里边调用的一个函数。
2. 在 `scHelper.king` 一开始跑的时候，用一断 `hook` 代码把整个线程拥塞起来，然后我们 `attack`, 然后恢复拥塞。太好了 `messagebox` 满足我们的需求。

于是就有了这样一段代码

```
;在此文档的文档工具栏项目上单击右键->参数属性
.386
.model flat, stdcall
option casemap :none
include windows.inc
include user32.inc
include kernel32.inc
includelib user32.lib
includelib kernel32.lib
inlinehookfun PROTO :DWORD,:DWORD,:DWORD
MyZwSetInformationThread PROTO
Myconnect PROTO
MyCreateThread PROTO
patchcode PROTO
ThreadProc PROTO
.data
  lpszByDll           db "Welcome",0
  lpszErrorQuery      db "query error",0
  lpszErrorProtected  db "protected error",0
  lpszErrorProtected2 db "protected error2",0
  lpszErrorGetModule  db "get module handle error",0
  lpszconnect         db "no cmp error",0
  lpZwSetInformationThread db "ZwSetInformationThread",0
  lpNtDll           db "NtDll.dll",0
  lpWS2_32Dll       db "ws2_32.dll",0
  lpConnect         db "connect",0
  lpKernel32Dll     db "kernel32.dll",0
  lpCreateThread    db "CreateThread",0
  lpthreadmsg       db "thread msg",0
  lptestbreakpointer db "testbreakpointer",0
  lpHideDebug       db "Hide Debug",0
  lphookcode        db  0e9h,90h,90h,90h,90h,90h,90h,90h,90h,90h
.data?
  hInstance dd ?
  hinsNtDll dd ?
  addrZwSetInformationThread dd ?
  myaddrZwSetInformationThread dd ?
  hinsWS2_32            dd ?
  addrconnect           dd ?
  myaddrconnect         dd ?
  hinsKernel32          dd ?
  addrCreateThread      dd ?
  mydddrCreateThread    dd ?
.CODE
;入口.如果 DLL 需要加载资源,需要保存 hIinstDLL 这个句柄到全局变量.它才是模块句柄
;使用 GetModuleHandle 获得的永远是主程序的句柄
LibMain proc hInstDLL:DWORD, reason:DWORD, unused:DWORD
  .if reason == DLL_PROCESS_ATTACH          ;动态库被加载时调用,返回 0 加载失败!
    mov eax,hInstDLL
    mov hInstance,eax
    invoke GetModuleHandle,addr lpNtDll
    .if eax == NULL
      invoke MessageBox,NULL,addr lpszErrorGetModule,addr lpszErrorGetModule,MB_OK
      ret
    .endif
    mov hinsNtDll,eax
    invoke GetProcAddress,hinsNtDll,addr lpZwSetInformationThread
    mov addrZwSetInformationThread,eax
    mov eax,MyZwSetInformationThread
    mov myaddrZwSetInformationThread,eax
    invoke inlinehookfun,myaddrZwSetInformationThread,addrZwSetInformationThread,0
    invoke LoadLibrary,addr lpWS2_32Dll
    .if eax == NULL
      invoke MessageBox,NULL,addr lpszErrorGetModule,addr lpszErrorGetModule,MB_OK
      ret
    .endif
    mov hinsWS2_32,eax
    invoke GetProcAddress,hinsWS2_32,addr lpConnect
    mov addrconnect,eax
    mov eax,Myconnect
    mov myaddrconnect,eax
    invoke inlinehookfun,myaddrconnect,addrconnect,0
    ;invoke patchcode
    invoke CreateThread,NULL,NULL,ThreadProc,NULL,NULL,NULL
    invoke GetModuleHandle,addr lpKernel32Dll
    mov   hinsKernel32,eax
    invoke GetProcAddress,hinsKernel32,addr lpCreateThread
    mov   addrCreateThread,eax
    mov   eax,MyCreateThread
    mov   mydddrCreateThread,eax
    invoke inlinehookfun,mydddrCreateThread,addrCreateThread,0
    mov eax,TRUE
    ret
  .elseif reason == DLL_PROCESS_DETACH
  .elseif reason == DLL_THREAD_ATTACH
  .elseif reason == DLL_THREAD_DETACH
    ;添加处理代码
  .endif
ret
LibMain Endp
;供主程序调用的函数
MsgBox proc hWnd,lpszText,fStyle
  invoke MessageBox,hWnd,lpszText,offset lpszByDll,fStyle
ret
MsgBox endp
inlinehookfun  proc dDest,dSrc,len
  LOCAL @mbi:MEMORY_BASIC_INFORMATION
  LOCAL @oldpro:DWORD
  invoke VirtualQuery,dSrc,addr @mbi,sizeof MEMORY_BASIC_INFORMATION
  .if eax== FALSE
    invoke MessageBox,NULL,addr lpszErrorQuery,addr lpszErrorQuery,MB_OK
    ret
  .else
    invoke VirtualProtect,@mbi.BaseAddress,@mbi.RegionSize,PAGE_EXECUTE_READWRITE,addr @oldpro
    .if eax == FALSE
      invoke MessageBox,NULL,addr lpszErrorProtected,addr lpszErrorProtected,MB_OK
      ret
    .else
      mov eax,dDest
      mov ebx,dSrc
      sub eax,ebx
      sub eax,5
      lea ebx,lphookcode
      mov DWORD ptr [ebx+1],eax
      mov eax,dSrc
      mov ch,0e9h
      mov BYTE ptr [eax],ch
      mov ecx,DWORD ptr [ebx+1]
      mov DWORD ptr [eax+1],ecx
      ;modify code
      invoke VirtualProtect,@mbi.BaseAddress,@mbi.RegionSize,@oldpro,addr @oldpro
      .if eax == FALSE
        invoke MessageBox,NULL,addr lpszErrorProtected2,addr lpszErrorProtected2,MB_OK
        ret
      .endif
    .endif
  .endif
  ret
inlinehookfun  endp
patchcode proc
  Local   @mbi : MEMORY_BASIC_INFORMATION
  Local   @oldpro:DWORD
  invoke  VirtualQuery,2044747h,addr @mbi,sizeof MEMORY_BASIC_INFORMATION
  invoke  VirtualProtect,@mbi.BaseAddress,@mbi.RegionSize,PAGE_EXECUTE_READWRITE,addr @oldpro
  mov     eax,204474Eh
  mov     dh,0ebh
  mov     BYTE ptr [eax],dh
  invoke  VirtualProtect,@mbi.BaseAddress,@mbi.RegionSize,@oldpro,addr @oldpro
patchcode endp
MyZwSetInformationThread  proc
  pushad
  pushfd
  ;invoke  MessageBox,NULL,addr lptestbreakpointer,addr lptestbreakpointer,MB_OK
                    ;ret ;2
  mov eax,DWORD ptr [esp+20h+04h+08h]
  cmp eax,11h
  jnz CMP_NO
  invoke MessageBox,NULL,addr lpHideDebug,addr lpHideDebug,MB_OK
CMP_NO:
  popfd
  popad
  mov eax,0e5h
  mov edx,addrZwSetInformationThread
  add edx,5
  jmp edx
ret
MyZwSetInformationThread   endp
Myconnect proc
pushad
pushfd
                  ;ret ;2
  mov eax,DWORD ptr [esp+20h+04h+08h]
  mov edx,DWORD ptr [eax+04h]
  cmp edx,0b75ab8abh
  jnz  NO_CMP
  invoke MessageBox,NULL,addr lpszconnect,addr lpszconnect,MB_OK
NO_CMP:
popfd
popad
mov  edi,edi
push ebp
mov  ebp,esp
mov  eax,addrconnect
add  eax,5
jmp  eax
Myconnect endp
ThreadProc   proc
  invoke Sleep,5000
  ;invoke patchcode
  ret
ThreadProc  endp
MyCreateThread proc
  pushad
  pushfd
    mov eax,DWORD ptr [esp+20h+04h]
    sub eax,  02020000h
    ja   leb_2
leb_1:
    jmp NO
leb_2:
    mov eax,DWORD ptr [esp+20h+04h]
    sub eax,026fd000h
    jna leb_4
leb_3:
    jmp NO
leb_4:
```
