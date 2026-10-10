

# 游戏修改的常用方法之二 Dll 注入(asm 源码详注)

> 原文：[`www.pediy.com/kssd/pediy10/66625.html`](https://www.pediy.com/kssd/pediy10/66625.html)

本程序在 WinXP/SP2、ra2 之 1.006 英文版（有中国超牛机器人的那个）运行通过且稳定无误。  
;------------------------------------------------------------------------  
;文件一：Ra2MDF.Asm   启动游戏　和　DLL 注入  
;------------------------------------------------------------------------  
  
.586  
.model flat, stdcall  
option casemap :none  
  
include        windows.inc  
include        user32.inc  
includelib    user32.lib  
include        kernel32.inc  
includelib    kernel32.lib  
  
.code  
  
;------------------------------------------------------------------------  
;把本程序拷入游戏文件夹，运行本程序，游戏被启动，本程序在后台运行，无界面  
;本程序运行后启动“ra2.exe”，再由“ra2.exe”启动游戏程序，之后“ra2.exe”  
;无用了，停掉它以节约内存。  
;------------------------------------------------------------------------  
\_StartGame proc  
    LOCAL StartInfo:STARTUPINFO  
    LOCAL PI:PROCESS\_INFORMATION  
  
    invoke RtlZeroMemory,addr StartInfo,sizeof STARTUPINFO  
    mov StartInfo.cb,sizeof STARTUPINFO  
    mov ecx,GAME\_NAME  
    xor edx,edx  
    invoke CreateProcess,edx,ecx,edx,edx,edx,edx,edx,edx,addr StartInfo,addr PI ;启动游戏  
    invoke Sleep,15000 ;等 10 秒，游戏程序应该启动了吧?  
    invoke TerminateProcess,PI.hProcess,0h ;没用了，停掉它  
    invoke CloseHandle,PI.hProcess ;释放内存  
  
    ret  
GAME\_NAME:  
    db "ra2.exe",0  
  
\_StartGame endp  
  
start:  
;------------------------------------------------------------------------  
;  
;  
;------------------------------------------------------------------------  
Main proc uses edi esi  
    LOCAL @dwProcessID,@lpLoadLibrary,@lpDllName,@szDllFullPath\[MAX\_PATH\]  
  
    invoke \_StartGame    ;启动游戏  
  
    lea edi,@szDllFullPath  
    invoke    GetCurrentDirectory,MAX\_PATH,edi  
    call @F  
    db '\\Ra2MD.dll',0  
@@:    push edi  
    call lstrcat    ;获取 dll 的全路径文件名  
    call @F  
    db 'Kernel32.dll',0  
@@:    call GetModuleHandle  
    call @F  
    db 'LoadLibraryA',0  
@@:    push eax  
    call GetProcAddress    ;获取 LoadLibrary 函数地址  
    mov  @lpLoadLibrary,eax  
  
    invoke    GetForegroundWindow    ;取得游戏窗口的句柄  
    lea    esi,@dwProcessID  
    invoke    GetWindowThreadProcessId,eax,esi    ;获取进程 ID  
    invoke    OpenProcess,PROCESS\_CREATE\_THREAD or PROCESS\_VM\_OPERATION or \\  
        PROCESS\_VM\_WRITE,FALSE,@dwProcessID    ;打开进程  
    .if    eax  
        mov    esi,eax  
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
; 在进程中分配空间并将 DLL 文件名拷贝过去，然后创建一个 LoadLibrary 线程  
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
        invoke    VirtualAllocEx,eax,NULL,MAX\_PATH,MEM\_COMMIT,PAGE\_READWRITE;  
        .if    eax  
            mov    @lpDllName,eax  
            invoke    WriteProcessMemory,esi,eax,edi,MAX\_PATH,NULL  
            invoke    CreateRemoteThread,esi,NULL,0h,@lpLoadLibrary,@lpDllName,0h,NULL    ;完成 DLL 注入  
            invoke    CloseHandle,eax  
        .endif  
        invoke    CloseHandle,esi  
    .endif  
    invoke    ExitProcess,NULL  
Main endp  
  
end    start  
  
  
  
  
  
;-------------------------------------------------------------------------------------  
;文件二：Ra2DLL.Asm  用来被嵌入到其它进程执行的 dll,完成热键、搜索、修改等全部功能  
;作为游戏进程的一部分，可以直接访问游戏内存，不用 ReadProcessMemory 和 WriteProcessMemory  
;这样搜索速度相对要快一些。本例中所说的游戏版本与前面的相同。  
;-------------------------------------------------------------------------------------  
  
.586  
.model flat, stdcall  
option casemap :none  
  
include        windows.inc  
include    user32.inc  
includelib    user32.lib  
include        kernel32.inc  
includelib    kernel32.lib  
  
ADD\_DATA1    equ 008373cch    ;00883c84h    ;其他版本要改这个地址值  
;ADD\_DATA2    equ 008373d0h    ;00883c88h    ;这个不用  
ADD\_DATA3\_START equ 01000000h    ;搜索范围开始地址，可根据需要改动，但要跳过前两个地址  
ADD\_DATA3\_END    equ 0f600000h    ;搜索范围结束地址，可根据需要改动  
MEMSIZE    equ 10000h    ;每次读取数据块的大小，不宜太小  
  
.code  
  
\_GetDataAddr proc uses esi edi num,hmem  
    LOCAL N,ListMemSize,pListMem,ReadSize  
    LOCAL mbi:MEMORY\_BASIC\_INFORMATION  
  
    invoke GlobalLock,hmem ;锁定保存搜索结果的内存  
    mov  pListMem,eax  
    invoke GlobalSize,hmem;保存搜索结果的内存大小  
    mov    ListMemSize,eax  
  
    mov edi,ADD\_DATA1 ;保存金钱数，以便后面比较搜索，原理见前文  
    mov edi,\[edi\]  
    mov ecx,ADD\_DATA3\_START ;设置要搜索的内存地址范围开始处  
    .repeat ;循环搜索游戏内存  
@@:        invoke VirtualQuery,ecx,addr mbi,sizeof MEMORY\_BASIC\_INFORMATION ;返回页面虚拟信息  
        .if mbi.State == MEM\_COMMIT && mbi.Protect == PAGE\_READWRITE ;已提交且为可读写的区域，加速搜索  
            invoke IsBadWritePtr,mbi.BaseAddress,mbi.RegionSize  
            .if !eax  
                mov esi,mbi.BaseAddress  
                xor ecx,ecx  
                .repeat  
                    .if edi\==\[esi+ecx\] ;数值相等，找到了?  
                        mov eax,num ;地址 num 中记录了搜索结果的个数  
                        inc dword ptr\[eax\] ;搜索的结果个数加一  
                        mov eax,dword ptr\[eax\]  
                        shl eax,2h ;保存结果所需的内存大小  
                        .if eax\>ListMemSize ;如果搜索到的结果较多，内存用完，要重新分配内存  
                            push eax  
                            push ecx  
                            add  eax,1000h ;追加４Ｋ内存  
                            invoke GlobalReAlloc,hmem,eax,GMEM\_MOVEABLE ;重新分配内存,原来的数据被复制过来  
                            invoke GlobalLock,eax  
                            mov  pListMem,eax ;保存搜索结果的内存首地址  
                            invoke GlobalSize,hmem  
                            mov  ListMemSize,eax ;保存搜索结果的内存大小  
                            pop  ecx  
                            pop  eax  
                        .endif  
                        add  eax,pListMem ;相当于 pListMem\[num\]  
                        mov  edx,esi  
                        add  edx,ecx ;首地址＋偏移地址＝实际地址  
                        mov  \[eax\-4h\],edx ;搜索的结果保存起来,pListMem\[num-1\]＝实际地址  
                    .endif  
                    add ecx,4h ;金钱数为 DWORD 型数值，考虑到内存对齐，这里是不用担心漏掉的  
                .until    ecx\>=mbi.RegionSize  
            .endif  
        .endif  
        mov  ecx,mbi.BaseAddress  
        add  ecx,mbi.RegionSize ;下一区段首地址  
    .until ecx\>=ADD\_DATA3\_END ;下一区段在搜索范围之外了吗?是则完成第一遍搜索  
  
    ret  
  
\_GetDataAddr endp  
  
;------------------------------------------------------------------------  
;第二、三……遍的搜索，在第一次的结果中找，速度极快  
;第二次按下“\*”键便来到这里，一般只要两遍就可锁定。算法：有用地址向前移  
;结果个数由 num 返回，如果 num==1 就算找到正确的金钱地址了  
;------------------------------------------------------------------------  
\_FindAddrInList proc uses edi esi num,hmem  
    LOCAL N  
  
    invoke GlobalLock,hmem  
    mov    edi,eax ;前次搜索结果保存的内存首地址  
    xor    esi,esi ;指针，指向第一个结果  
    mov    N,esi ;本次搜索到的个数初始化为 0  
    .repeat ;逐个比较  
        mov edx,\[edi+esi\*4h\] ;相当于 edx=hmem\[esi\]  
        mov eax,ADD\_DATA1  
        mov eax,\[eax\]  
        .if eax\==\[edx\] ;等于金钱数吗?等则记录下来  
            push \[edi+esi\*4h\]  
            mov eax,N  
            pop \[edi+eax\*4h\] ;相当于 hmem\[N\]=hmem\[esi\],即把搜索到的结果向 hmem 内存前面移  
            inc N ;搜索到的个数加一  
        .endif  
        mov eax,num  
        inc esi ;指针指向下一个结果  
    .until esi\>=\[eax\] ;每个都比较过了吗?是则完成这次搜索  
    mov    edx,N ;这次搜索到的结果个数  
    mov    \[eax\],edx ;修改原来的个数  
    shl    edx,2h ;个数×4＝内存大小  
    invoke GlobalReAlloc,hmem,edx,GMEM\_MOVEABLE ;释放多余的内存  
  
    ret  
      
\_FindAddrInList endp  
  
;------------------------------------------------------------------------  
;找到了正确的地址，可以锁定金钱数值了:)，每五秒锁定一次，按“-”键停锁  
;游戏中钱数看起来有增有减，象未锁定一样，别人不容易发现你作弊  
;------------------------------------------------------------------------  
\_WriteProcessData proc uses edi esi hmem  
      
    mov    esi,5h ;要锁定的金钱数，别太多，多了是会招贼来偷的:)  
    invoke GlobalLock,hmem  
    mov    edi,\[eax\] ;游戏中真正的保存金钱数的地址  
    .repeat  
        .if esi\>=5h ;每五循环锁定一次  
            xor    esi,esi ;循环次数清 0  
            mov    dword ptr\[edi\],2000  
        .endif      
        invoke Sleep,1000 ;定时一秒  
        inc    esi ;循环次数加一  
        invoke GetAsyncKeyState,VK\_SUBTRACT ;按了“-”键吗?  
    .until eax;如果按了“-”键则结束循环  
      
    ret  
      
\_WriteProcessData endp  
  
;------------------------------  
;新线程，检查按键，响应用户请求  
;------------------------------  
ThreadProc Proc    pParam:LPVOID  
    Local    @dwNum,@hMem  
  
    invoke GlobalAlloc,GMEM\_MOVEABLE,1000h ;预留内存空间，搜索时用来保存结果  
    mov @hMem,eax ;搜索结果个数的不确定性需要我们用 GlobalReAlloc 来重新分配内存大小  
    mov @dwNum,0h ;搜索结果个数初始化为 0  
  
    .while TRUE  
        invoke GetAsyncKeyState,VK\_SUBTRACT ;按了“-”键吗?  
        .if eax  
            mov @dwNum,0h ;搜索结果个数置 0，表示从未搜索过  
            invoke SetCursorPos,9h,9h ;把鼠标置于屏幕左上角，提醒游戏者，第一遍搜索时特别有用  
        .endif  
  
        invoke GetAsyncKeyState,VK\_MULTIPLY ;按了“\*”键吗?  
        .if eax ;按了热键“\*”  
            .if @dwNum==0h ;搜索结果个数为 0 则从未搜索过，进行第一遍搜索  
                invoke \_GetDataAddr,addr @dwNum,@hMem  
            .elseif @dwNum>1h ;搜索结果个数非 0 则至少已搜过一遍且未找到正确地址  
                invoke \_FindAddrInList,addr @dwNum,@hMem  ;再搜它一遍或几遍  
            .endif  
          
            .if @dwNum==1h ;搜索结果为１，说明找到正确地址了  
                invoke \_WriteProcessData,@hMem ;去锁定它!  
            .endif                  
            invoke MessageBeep,MB\_OK ;发声提示游戏者，按键收到，该做的本程序都做过了  
            invoke SetCursorPos,9h,9h ;把鼠标置于屏幕左上角，提醒游戏者，第一遍搜索时特别有用  
        .endif  
        invoke Sleep,1000  
    .endw  
    invoke GlobalUnlock,@hMem  
    invoke GlobalFree,@hMem  
    ret  
  
ThreadProc endP  
  
DllEntry Proc    \_hInstance,\_dwReason,\_dwReserved  
    LOCAL @dwThreadID  
  
    .if    \_dwReason == DLL\_PROCESS\_ATTACH  
        invoke    CreateThread,NULL,0h,offset ThreadProc,NULL,NULL,addr @dwThreadID ;创建一个新线程  
        invoke    CloseHandle,eax  
    .endif  
    mov    eax,TRUE  
    ret  
  
DllEntry Endp  
  
End DllEntry