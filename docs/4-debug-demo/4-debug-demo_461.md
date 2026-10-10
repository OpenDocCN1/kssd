

# 星际霸主 1.9j 分析

> 原文：[`www.pediy.com/kssd/pediy12/128871.html`](https://www.pediy.com/kssd/pediy12/128871.html)

【文章标题】: 星际霸主 1.9j 分析  
【文章作者】: thomasyzh  
【作者邮箱】: machinesy@gmail.com  
【软件名称】: 星际霸主 1.9j  
【软件大小】: 5.16 MB   
【下载地址】: 自己搜索下载  
【加壳方式】: Themida/WinLicense V1.8.X-V1.9.X DLL   
【软件介绍】: 一个星际争霸 1 的游戏外挂  
【作者声明】: 只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教!  
\--------------------------------------------------------------------------------  
【详细过程】  
  这是一个破解任务，一位朋友请我帮他破解，价格是 1K。因为不是为了赚这个票子，所以就直接发表出来了。在发表之前  
  我是在考虑这个帖子的标题写软件破解，还是软件分析。因为实际上现在有两个方法完成这个破解，1 是继续按照我现在的  
  思路把这个东西做完了。2 是我去把 Virtualizer 加的虚拟器代码还原掉了。实际上我选择了 2，所以我就没有继续把 1 的路子  
  走完了。最后我把改 vm 代码还原后，一定的能够破译掉。  
  那么，我把 1 的分析过程写出来，所以我把这文章的标题就写做分析了。  
    
  一些文件啊，咋的咋的那些信息我就不说了。我直接说结论。  
    
  1 这是个大量使用 hook 挂接星际争霸的游戏外挂。  
  2 这个外挂是使用 http 验证的一个外挂。  
  3 这个外挂的验证在 scHelper.king 的代码里边。  
  4 这个外挂的数据收发在 scking.exe 里边，然后通过内存共享传送到 scHelper.king 里边。大概应该在那个 sckhook.dll 里边。  
    
  因为加了 tmd 壳。我并没有选择一开始就 CreateProcess 这种方式，而是让它在内存里解密完，并且 anti 代码跑完的石斛 attack  
  上去。最佳的效果是直接 attack 到加密验证代码那里。  
    
  于是  
  1.Hook 一个函数，这个函数必须要是 scHelper.king 里边调用的一个函数。  
  2.在 scHelper.king 一开始跑的时候，用一断 hook 代码把整个线程拥塞起来，然后我们 attack,然后恢复拥塞。太好了 messagebox 满足我们的需求。  
    
  于是就有了这样一段代码  
    
  ;在此文档的文档工具栏项目上单击右键->参数属性  
    
  .386  
  .model flat, stdcall  
  option casemap :none  
    
  include windows.inc  
  include user32.inc  
  include kernel32.inc  
    
  includelib user32.lib  
  includelib kernel32.lib  
    
  inlinehookfun  PROTO :DWORD,:DWORD,:DWORD    
  MyZwSetInformationThread PROTO   
  Myconnect           PROTO  
  MyCreateThread         PROTO  
    
  patchcode           PROTO  
  ThreadProc           PROTO  
    
  .data  
    lpszByDll             db "Welcome",0  
    lpszErrorQuery         db "query error",0  
    lpszErrorProtected      db "protected error",0  
    lpszErrorProtected2      db "protected error2",0  
    lpszErrorGetModule       db "get module handle error",0  
    lpszconnect           db "no cmp error",0  
      
    lpZwSetInformationThread db "ZwSetInformationThread",0  
    lpNtDll             db "NtDll.dll",0  
      
    lpWS2\_32Dll           db "ws2\_32.dll",0  
    lpConnect           db "connect",0  
      
    lpKernel32Dll         db "kernel32.dll",0  
    lpCreateThread         db "CreateThread",0  
      
      
    lpthreadmsg          db "thread msg",0  
      
    lptestbreakpointer     db "testbreakpointer",0  
    lpHideDebug           db "Hide Debug",0  
    lphookcode            db  0e9h,90h,90h,90h,90h,90h,90h,90h,90h,90h    
      
      
      
      
  .data?  
    hInstance dd ?  
    hinsNtDll dd ?  
    addrZwSetInformationThread dd ?  
    myaddrZwSetInformationThread dd ?  
      
    hinsWS2\_32            dd ?  
    addrconnect            dd ?  
    myaddrconnect          dd ?  
      
    hinsKernel32          dd ?  
    addrCreateThread        dd ?  
    mydddrCreateThread      dd ?  
      
  .CODE  
    
  ;入口.如果 DLL 需要加载资源,需要保存 hIinstDLL 这个句柄到全局变量.它才是模块句柄  
  ;使用 GetModuleHandle 获得的永远是主程序的句柄  
  LibMain proc hInstDLL:DWORD, reason:DWORD, unused:DWORD  
    .if reason == DLL\_PROCESS\_ATTACH          ;动态库被加载时调用,返回 0 加载失败!  
      mov eax,hInstDLL  
      mov hInstance,eax  
        
      invoke GetModuleHandle,addr lpNtDll  
      .if eax == NULL  
        invoke MessageBox,NULL,addr lpszErrorGetModule,addr lpszErrorGetModule,MB\_OK  
        ret  
      .endif  
      mov hinsNtDll,eax  
      invoke GetProcAddress,hinsNtDll,addr lpZwSetInformationThread  
      mov addrZwSetInformationThread,eax  
      mov eax,MyZwSetInformationThread  
      mov myaddrZwSetInformationThread,eax  
        
      invoke inlinehookfun,myaddrZwSetInformationThread,addrZwSetInformationThread,0  
        
        
      invoke LoadLibrary,addr lpWS2\_32Dll  
      .if eax == NULL  
        invoke MessageBox,NULL,addr lpszErrorGetModule,addr lpszErrorGetModule,MB\_OK  
        ret  
      .endif  
      mov hinsWS2\_32,eax  
      invoke GetProcAddress,hinsWS2\_32,addr lpConnect  
      mov addrconnect,eax  
      mov eax,Myconnect  
      mov myaddrconnect,eax  
      invoke inlinehookfun,myaddrconnect,addrconnect,0  
        
      ;invoke patchcode  
      invoke CreateThread,NULL,NULL,ThreadProc,NULL,NULL,NULL  
        
        
      invoke GetModuleHandle,addr lpKernel32Dll  
      mov    hinsKernel32,eax  
      invoke GetProcAddress,hinsKernel32,addr lpCreateThread  
      mov    addrCreateThread,eax  
      mov    eax,MyCreateThread  
      mov    mydddrCreateThread,eax  
      invoke inlinehookfun,mydddrCreateThread,addrCreateThread,0  
        
        
      mov eax,TRUE  
      ret  
    .elseif reason == DLL\_PROCESS\_DETACH  
        
    .elseif reason == DLL\_THREAD\_ATTACH  
        
    .elseif reason == DLL\_THREAD\_DETACH  
      ;添加处理代码  
    .endif  
  ret  
  LibMain Endp  
    
  ;供主程序调用的函数  
  MsgBox proc hWnd,lpszText,fStyle  
    invoke MessageBox,hWnd,lpszText,offset lpszByDll,fStyle  
  ret  
  MsgBox endp  
    
    
  inlinehookfun  proc dDest,dSrc,len  
    LOCAL @mbi:MEMORY\_BASIC\_INFORMATION   
    LOCAL @oldpro:DWORD  
    invoke VirtualQuery,dSrc,addr @mbi,sizeof MEMORY\_BASIC\_INFORMATION  
    .if eax== FALSE  
      invoke MessageBox,NULL,addr lpszErrorQuery,addr lpszErrorQuery,MB\_OK    
      ret  
    .else  
      invoke VirtualProtect,@mbi.BaseAddress,@mbi.RegionSize,PAGE\_EXECUTE\_READWRITE,addr @oldpro  
      .if eax == FALSE  
        invoke MessageBox,NULL,addr lpszErrorProtected,addr lpszErrorProtected,MB\_OK  
        ret  
      .else  
          
        mov eax,dDest  
        mov ebx,dSrc  
        sub eax,ebx  
        sub eax,5  
        lea ebx,lphookcode  
        mov DWORD ptr \[ebx+1\],eax  
          
        mov eax,dSrc  
        mov ch,0e9h  
        mov BYTE ptr \[eax\],ch  
        mov ecx,DWORD ptr \[ebx+1\]  
        mov DWORD ptr \[eax+1\],ecx  
        ;modify code  
          
        invoke VirtualProtect,@mbi.BaseAddress,@mbi.RegionSize,@oldpro,addr @oldpro  
        .if eax == FALSE  
          invoke MessageBox,NULL,addr lpszErrorProtected2,addr lpszErrorProtected2,MB\_OK  
          ret  
        .endif    
        
      .endif  
    .endif  
      
    ret  
  inlinehookfun  endp  
    
  patchcode proc  
    Local   @mbi : MEMORY\_BASIC\_INFORMATION  
    Local   @oldpro:DWORD  
    invoke   VirtualQuery,2044747h,addr @mbi,sizeof MEMORY\_BASIC\_INFORMATION  
    invoke   VirtualProtect,@mbi.BaseAddress,@mbi.RegionSize,PAGE\_EXECUTE\_READWRITE,addr @oldpro  
    mov     eax,204474Eh  
    mov      dh,0ebh  
    mov      BYTE ptr \[eax\],dh  
    invoke   VirtualProtect,@mbi.BaseAddress,@mbi.RegionSize,@oldpro,addr @oldpro  
    
  patchcode endp  
    
  MyZwSetInformationThread  proc  
    
    pushad  
    pushfd  
    ;invoke  MessageBox,NULL,addr lptestbreakpointer,addr lptestbreakpointer,MB\_OK  
                       ;ret ;2  
    mov eax,DWORD ptr \[esp+20h+04h+08h\]    
    cmp eax,11h  
    jnz CMP\_NO  
      
    invoke MessageBox,NULL,addr lpHideDebug,addr lpHideDebug,MB\_OK  
      
  CMP\_NO:    
    popfd  
    popad  
    mov eax,0e5h  
    mov edx,addrZwSetInformationThread  
    add edx,5  
    jmp edx  
  ret  
  MyZwSetInformationThread   endp  
    
    
  Myconnect proc  
  pushad  
  pushfd  
                      ;ret ;2          
    mov eax,DWORD ptr \[esp+20h+04h+08h\]    
    mov edx,DWORD ptr \[eax+04h\]  
    cmp edx,0b75ab8abh  
    jnz  NO\_CMP  
    invoke MessageBox,NULL,addr lpszconnect,addr lpszconnect,MB\_OK  
    
  NO\_CMP:  
    
  popfd  
  popad  
  mov  edi,edi  
  push ebp  
  mov  ebp,esp  
  mov  eax,addrconnect  
  add  eax,5  
  jmp  eax  
    
    
    
  Myconnect endp  
    
  ThreadProc   proc  
    invoke Sleep,5000  
    ;invoke patchcode  
    ret  
  ThreadProc  endp  
    
  MyCreateThread proc  
    pushad  
    pushfd  
      mov eax,DWORD ptr \[esp+20h+04h\]  
        
      sub eax,  02020000h   
      ja   leb\_2  
  leb\_1:  
      jmp NO    
  leb\_2:  
      mov eax,DWORD ptr \[esp+20h+04h\]  
      sub eax,026fd000h     
      jna leb\_4  
  leb\_3:  
      jmp NO  
  leb\_4:      
      invoke MessageBox,NULL,addr lpthreadmsg,addr lpthreadmsg,MB\_YESNO  
        
  NO:    
    popfd  
    popad  
      
    mov  edi,edi  
    push ebp  
    mov  ebp,esp  
      
    mov  eax,addrCreateThread  
    add  eax,5  
    jmp  eax  
  MyCreateThread endp  
    
  End LibMain  
    
    
  实际上，在这个 dll 汇编代码里，我挂了几个函数，都是为了测试用的。  
    
  但是这正有用的只有一个 CreateThread 看看挂接后做了什么  
    
  MyCreateThread proc  
    pushad  
    pushfd  
      mov eax,DWORD ptr \[esp+20h+04h\]  
        
      sub eax,  02020000h   
      ja   leb\_2  
  leb\_1:  
      jmp NO    
  leb\_2:  
      mov eax,DWORD ptr \[esp+20h+04h\]  
      sub eax,026fd000h     
      jna leb\_4  
  leb\_3:  
      jmp NO  
  leb\_4:      
      invoke MessageBox,NULL,addr lpthreadmsg,addr lpthreadmsg,MB\_YESNO  
        
  NO:    
    popfd  
    popad  
      
    mov  edi,edi  
    push ebp  
    mov  ebp,esp  
      
    mov  eax,addrCreateThread  
    add  eax,5  
    jmp  eax  
  MyCreateThread endp  
    
    
  写了一段代码判断是不是被 scHelper.king 这个模块调用的 CreateProcess 如果是的话就 MessageBox 一下。范围代码判断如下  
      mov eax,DWORD ptr \[esp+20h+04h\]  
        
      sub eax,  02020000h   
      ja   leb\_2  
  leb\_1:  
      jmp NO    
  leb\_2:  
      mov eax,DWORD ptr \[esp+20h+04h\]  
      sub eax,026fd000h     
      jna leb\_4  
  leb\_3:  
      jmp NO  
  leb\_4:      
      invoke MessageBox,NULL,addr lpthreadmsg,addr lpthreadmsg,MB\_YESNO  
        
  先取一个 esp 的 ret 模块地址如果满足大于 02020000h 小于 026fd000h 那么就 messagebox.  
  为啥要 createthread 喃！~？？？？一个 dll 如果不写个线程去完成很多初使化的话，那这样做是不合适的。因为会拥塞主线程的线程。  
    
  还有细节要处理下。Themida 壳挂接了 DbgUiRemoteBreakin 这个函数，需要恢复之，然后我们就能带壳调试了。然后在 messageboxa 的尾巴上打上断点。  
  我们就可以发现线程函数就是这个了  
  02028964   .  55            push    ebp  
  02028965   .  8BEC          mov     ebp, esp  
  02028967   .  81C4 58F3FFFF add     esp, -0CA8  
  0202896D   .  B8 A45E2302   mov     eax, 02235EA4  
  02028972   .  53            push    ebx  
  02028973   .  56            push    esi  
  02028974   .  57            push    edi  
  02028975   .  E8 D2DE0800   call    020B684C  
    
    
    
  然后就进入到这里。然后它就开始他疯狂的 vm 了。进过一段时间的分析，我们就找到了 vm\_call\_fun 的位置。具体如何找到 vm\_call\_fun 其实一点都不难  
  我是直接用 od 的自动跟入，跟入每一个分支，然后看到它跑到非 vm 的代码里边后，再用减号后退，就直接找到了 vm\_call\_fun.  
    
    
    
    
  01B815C0    8B0424          mov     eax, dword ptr \[esp\]             ; vm\_call\_XXX  
  01B815C3    8DB0 44010000   lea     esi, dword ptr \[eax+144\]  
  01B815C9    B9 1C000000     mov     ecx, 1C  
  01B815CE    8B78 3C         mov     edi, dword ptr \[eax+3C\]  
  01B815D1    F3:A5           rep     movs dword ptr es:\[edi\], dword p>  
  01B815D3    5F              pop     edi  
  01B815D4    C747 38 000000F>mov     dword ptr \[edi+38\], F0000000  
  01B815DB    FF77 70         push    dword ptr \[edi+70\]  
  01B815DE    FF77 74         push    dword ptr \[edi+74\]  
  01B815E1    FFB7 84000000   push    dword ptr \[edi+84\]  
  01B815E7    FFB7 8C000000   push    dword ptr \[edi+8C\]  
  01B815ED    FF77 7C         push    dword ptr \[edi+7C\]  
  01B815F0    FFB7 AC000000   push    dword ptr \[edi+AC\]  
  01B815F6    FFB7 A4000000   push    dword ptr \[edi+A4\]  
  01B815FC    FFB7 94000000   push    dword ptr \[edi+94\]  
  01B81602    FFB7 9C000000   push    dword ptr \[edi+9C\]  
  01B81608    C747 28 0000000>mov     dword ptr \[edi+28\], 0  
  01B8160F    61              popad  
  01B81610    9D              popfd  
  01B81611    C3              retn  
    
    
  这里就是 vm\_Call\_Fun.  
    
  虚拟机代码的还原部分，我会另外开篇帖子再写写，这里我就不再深入细解了。  
  然后我们我们在 vm\_call\_fun 看这个验证 dll 是调用了哪些没有被 vm 的代码。有可能像 memcpy 这样功能的函数，他没有去 vm 掉或者怎样怎样。  
  这样，我们就又找了这样一个函数。这个函数就是 memcpy  
    
  020B6418  /$  55            push    ebp  
  020B6419  |.  8BEC          mov     ebp, esp  
  020B641B  |.  56            push    esi  
  020B641C  |.  57            push    edi  
  020B641D  |.  8B7D 08       mov     edi, dword ptr \[ebp+8\]  
  020B6420  |.  8BC7          mov     eax, edi  
  020B6422  |.  8B75 0C       mov     esi, dword ptr \[ebp+C\]  
  020B6425  |.  8B4D 10       mov     ecx, dword ptr \[ebp+10\]  
  020B6428  |.  8BD1          mov     edx, ecx  
  020B642A  |.  D1E9          shr     ecx, 1  
  020B642C  |.  D1E9          shr     ecx, 1  
  020B642E  |.  FC            cld  
  020B642F  |.  F3:A5         rep     movs dword ptr es:\[edi\], dword p>  
  020B6431  |.  8BCA          mov     ecx, edx  
  020B6433  |.  83E1 03       and     ecx, 3  
  020B6436  |.  F3:A4         rep     movs byte ptr es:\[edi\], byte ptr>  
  020B6438  |.  5F            pop     edi  
  020B6439  |.  5E            pop     esi  
  020B643A  |.  5D            pop     ebp  
  020B643B  \\.  C3            retn  
    
    
    
  然后我们就在这下断点看。  
  2  
  087FF0DC   020C2FBC  返回到 scHelper.020C2FBC 来自 scHelper.020B6418  
  087FF0E0   04512630  
  087FF0E4   04512614  ASCII "1234567890abcdef"""  
  087FF0E8   00000010  
  087FF0EC   0223A7F3  scHelper.0223A7F3  
  087FF0F0   04512566  
  087FF0F4   087FF130  指向下一个 SEH 记录的指针  
  087FF0F8   020B6893  SE 处理程序  
  087FF0FC   02242AD0  scHelper.02242AD0  
  087FF100   087FF0EC  
    
  3  
  087FF0D8   020C2FBC  返回到 scHelper.020C2FBC 来自 scHelper.020B6418  
  087FF0DC   04512650  
  087FF0E0   04512639  ASCII "0abcdef"  
  087FF0E4   00000001  
  087FF0E8   00000001  
  087FF0EC   0000000A  
    
  最终，我们就找到这个函数，这个函数就是报文的组织函数  
    
  020C3448  /$  55            push    ebp                              ;  EDX\_IS\_INDEX  
  020C3449  |.  8BEC          mov     ebp, esp  
  020C344B  |.  83C4 D0       add     esp, -30  
  020C344E  |.  53            push    ebx  
  020C344F  |.  56            push    esi  
  020C3450  |.  57            push    edi  
  020C3451  |.  894D D0       mov     dword ptr \[ebp-30\], ecx  
  020C3454  |.  8BDA          mov     ebx, edx  
  020C3456  |.  8BF8          mov     edi, eax  
  020C3458  |.  B8 342D2402   mov     eax, 02242D34  
  020C345D  |.  E8 EA33FFFF   call    020B684C  
    
    
    
  这里的 edx,就是一个 index，从随机变量到字符的转换。  
  我们为什么要找随机变量？因为我们要定死随机变量。定死随机变量后，我们就能够以同样的发送报文得到同样的应答报文，这样，我们也就破译了这个外挂了。  
  到这里，同学们如果呀破译这个外挂的话，只有再去分析完，这个 edx,到底来自于内存的哪里！~~~然后去把这个 edx,定死。然后再去通信一次，这个外挂就破解了。  
    
    
    
    
    
    
    
    
    
    
\--------------------------------------------------------------------------------  
【经验总结】  
  咋说喃，vm 还原这个东西不一句两句就说的清楚的。我自己也在学习。过两天，我去把这个外挂的 vm 还原放出来。有兴趣的  
  朋友也可以不还原 vm 破之。顺着我这个思路就成了。  
    
\--------------------------------------------------------------------------------  
【版权声明】: 本文原创于看雪技术论坛, 转载请注明作者并保持文章的完整, 谢谢!  
  
                                                       2011 年 02 月 01 日 18:19:50

上传的附件
| ![](img/67af4f1ef9c98996f43e89e749c7dcf5.jpg) || 霸主 1.9j.rar  
 |