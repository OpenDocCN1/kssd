

# 蠕虫 srv32.exe 逆向分析笔记 2 -- 初始化篇

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-155.htm`](https://www.pediy.com/kssd/pediy07/pediy7-155.htm)

文件名称：srv32.exe  
蠕虫名称：Net-Worm.Win32.Opasoft.s  
工具：   IDA 4.5.1, OllyDbg V1.10  
  
CODE:0040123E sub\_40123E      proc near  
CODE:0040123E Buffer          = byte ptr -100h  
CODE:0040123E                 enter   100h, 0  
CODE:00401242                 push    offset aSrv3231415 ; lpName  
CODE:00401247                 push    0FFFFFFFFh         ; bInitialOwner  
CODE:00401249                 push    0                  ; lpMutexAttributes  
CODE:0040124B                 call    CreateMutexA       ; 创建一个互斥对象"Srv3231415"  
CODE:00401250                 call    GetLastError  
CODE:00401255                 test    eax, eax  
CODE:00401257                 jnz     locret\_401329      ; 创建互斥失败则跳转;退出程序  
CODE:0040125D                 call    GetCurrentProcess  
CODE:00401262                 push    40h                ; dwPriorityClass  
CODE:00401264                 push    eax                ; hProcess  
CODE:00401265                 call    SetPriorityClass   ; 设置优先级为 IDLE\_PRIORITY\_CLASS ＝ 0x40  
CODE:0040126A                 lea     esi, \[ebp+Buffer\]  
CODE:00401270                 push    100h               ; uSize  
CODE:00401275                 push    esi                ; lpBuffer  
CODE:00401276                 call    GetWindowsDirectoryA ; 获得 Windows 目录  
CODE:0040127B                 push    esi                ; lpPathName  
CODE:0040127C                 call    SetCurrentDirectoryA ; 设置当前目录为 Windows 目录  
CODE:00401281                 call    sub\_401C98         ; 查询是否写了注册表项  
CODE:00401281                                            ; HKLM\\Software\\Microsoft\\Windows\\CurrentVersion\\srv32  
CODE:00401286                 cmp     ds:Data, 0  
CODE:0040128D                 jz      short loc\_401294   ; 没有写注册表项或  
CODE:0040128D                                            ; HKLM\\...\\srv32\\D = 0 则跳转  
CODE:0040128F                 call    sub\_4030EB  
  
进入 sub\_40123E 后，首先创建了一个互斥 Srv3231415，失败的话此程序就退出了；  
然后又去查询注册表 HKLM\\Software\\Microsoft\\Windows\\CurrenteVersion\\srv32 下  
"D"的值，如果 srv32 存在 D 的键值不为 0 则调用 sub\_4030EB，代码如下：  
  
CODE:004030EB sub\_4030EB      proc near  
CODE:004030EB ThreadId        = dword ptr -4  
CODE:004030EB                 enter   4, 0  
CODE:004030EF                 cmp     ds:dword\_406004, 0  
CODE:004030F6                 jnz     locret\_403184  
CODE:004030FC                 mov     ds:dword\_406004, 1  
CODE:00403106                 push    18Ch            ; uBytes  
CODE:0040310B                 push    0               ; uFlags  
CODE:0040310D                 call    LocalAlloc  
CODE:00403112                 mov     esi, eax  
CODE:00403114                 push    esi             ; lpWSAData  
CODE:00403115                 push    101h            ; wVersionRequested  
CODE:0040311A                 call    WSAStartup  
CODE:0040311F                 mov     edi, eax  
CODE:00403121                 push    esi             ; hMem  
CODE:00403122                 call    LocalFree  
CODE:00403127                 test    edi, edi  
CODE:00403129                 jnz     short loc\_403186  
CODE:0040312B                 push    1               ; protocol:ICMP  
CODE:0040312D                 push    3               ; type: SOCK\_RAW  
CODE:0040312F                 push    2               ; AF\_INET  
CODE:00403131                 call    socket  
CODE:00403136                 cmp     eax, 0FFFFFFFFh  
CODE:00403139                 jz      short loc\_403192  
CODE:0040313B                 mov     ds:s, eax  
CODE:00403140                 lea     eax, \[ebp+ThreadId\]  
CODE:00403143                 push    eax             ; lpThreadId  
CODE:00403144                 push    0               ; dwCreationFlags  
CODE:00403146                 push    ds:s            ; lpParameter  
CODE:0040314C                 push    offset sub\_403045 ; lpStartAddress  
CODE:00403151                 push    1000h           ; dwStackSize  
CODE:00403156                 push    0               ; lpThreadAttributes  
CODE:00403158                 call    CreateThread  
CODE:0040315D                 lea     eax, \[ebp+ThreadId\]  
CODE:00403160                 push    eax             ; lpThreadId  
CODE:00403161                 push    0               ; dwCreationFlags  
CODE:00403163                 push    ds:s            ; lpParameter  
CODE:00403169                 push    offset sub\_402EEE ; lpStartAddress  
CODE:0040316E                 push    1000h           ; dwStackSize  
CODE:00403173                 push    0               ; lpThreadAttributes  
CODE:00403175                 call    CreateThread  
CODE:0040317A                 mov     ds:dword\_406004, 2  
CODE:00403184   
CODE:00403184 locret\_403184:  
CODE:00403184                 leave  
CODE:00403185                 retn  
CODE:00403186 ; ///////////////////////////////////////////////////////////////////////////  
CODE:00403186   
CODE:00403186 loc\_403186:                             ; CODE XREF: sub\_4030EB+3Ej  
CODE:00403186                 mov     ds:dword\_406004, 0  
CODE:00403190                 leave  
CODE:00403191                 retn  
CODE:00403192 ; ///////////////////////////////////////////////////////////////////////////  
CODE:00403192   
CODE:00403192 loc\_403192:                             ; CODE XREF: sub\_4030EB+4Ej  
CODE:00403192                 call    WSACleanup  
CODE:00403197                 mov     ds:dword\_406004, 0  
CODE:004031A1                 leave  
CODE:004031A2                 retn  
CODE:004031A2 sub\_4030EB      endp  
  
  
在 sub\_4030EB 里打开了 socket，并创建了两个线程（动作越来越明显了 ^…^）。  
由于我第一次执行没有此键值所以跳转了，我们就先看跳转的情况吧：  
  
CODE:00401294 loc\_401294:  
CODE:00401294                 mov     ds:dword\_406587, 0  
CODE:0040129E                 call    sub\_4033E7  
  
调用了一个 call sub\_4033E7 跟进去看看...  
  
CODE:004033E7 sub\_4033E7      proc near  
CODE:004033E7                 push    ebp  
CODE:004033E8                 mov     esi, offset unk\_406DB4  
CODE:004033ED                 mov     edi, offset dword\_407750  
CODE:004033F2                 mov     ebp, 0  
  
CODE:004033F7 loc\_4033F7:  
CODE:004033F7                 cmp     ebp, 20h  
CODE:004033FA                 jnb     short loc\_40342F ; 不低于 0x20 就跳转,退出循环 1  
CODE:004033FC                 mov     ah, 1  
  
CODE:004033FE loc\_4033FE:   
CODE:004033FE                 cmp     ah, 40h  
CODE:00403401                 ja      short loc\_40342A ; 大于 0x40 就跳转  
CODE:00403403                 test    ah, 1  
CODE:00403406                 jz      short loc\_403409 ; ah 的第 0 位为 0 则跳转  
CODE:00403408                 lodsb  
  
CODE:00403409 loc\_403409:  
CODE:00403409                 shl     al, 1  
CODE:0040340B                 jnb     short loc\_403417  
CODE:0040340D                 movzx   ecx, ss:byte\_406EB4\[ebp\]  
CODE:00403414                 bts     \[edi\], ecx  
  
CODE:00403417 loc\_403417:  
CODE:00403417                 inc     ebp  
CODE:00403418                 test    ebp, 3  
CODE:0040341E                 jnz     short loc\_403409  
CODE:00403420                 sub     ebp, 4  
CODE:00403423                 inc     ah  
CODE:00403425                 add     edi, 4  
CODE:00403428                 jmp     short loc\_4033FE  
CODE:0040342A ; //////////////////////////////////////////////////////////////  
CODE:0040342A loc\_40342A:  
CODE:0040342A                 add     ebp, 4  
CODE:0040342D                 jmp     short loc\_4033F7  
CODE:0040342F ; ///////////////////////////////////////////////////////////////  
CODE:0040342F   
CODE:0040342F loc\_40342F:  
CODE:0040342F                 pop     ebp  
CODE:00403430                 retn  
CODE:00403430 sub\_4033E7      endp  
  
这段代码用 C 表示：  
  
BYTE array\_406DB4\[\] =  
{  
  0xE0, 0x4F, 0xD7, 0x14, 0x2E, 0xF2, 0xBD, 0x81,  
  0x3A, 0xA6, 0x6C, 0xCB, 0x59, 0x95, 0x03, 0x78,  
  0x4F, 0x1C, 0xE8, 0x82, 0xD4, 0x69, 0x21, 0xB7,  
  0xF5, 0xCB, 0x93, 0x7E, 0x3A, 0xA0, 0x56, 0x0D,  
  0xF3, 0x1D, 0x84, 0xE7, 0x6F, 0xB2, 0x38, 0x4E,  
  0x9C, 0x70, 0x21, 0xDA, 0xC6, 0x09, 0x5B, 0xA5,  
  0x0D, 0xE8, 0x7A, 0xB1, 0xA3, 0x4F, 0xD4, 0x12,  
  0x5B, 0x86, 0xC7, 0x6C, 0x90, 0x35, 0x2E, 0xF9,  
  0xAD, 0x07, 0x90, 0xE9, 0x63, 0x34, 0xF6, 0x5A,  
  0x12, 0xD8, 0xC5, 0x7E, 0xBC, 0x4B, 0x2F, 0x81,  
  0xD1, 0x6A, 0x4D, 0x90, 0x86, 0xF9, 0x38, 0x07,   
  0xB4, 0x1F, 0x2E, 0xC3, 0x5B, 0xA5, 0xE2, 0x7C,  
  0x7D, 0xD8, 0xEB, 0x35, 0x06, 0x6F, 0x90, 0xA3,   
  0x14, 0x27, 0x82, 0x5C, 0xB1, 0xCA, 0x4E, 0xF9,  
  0xA3, 0x6F, 0x90, 0x06, 0xCA, 0xB1, 0x7D, 0xD8,   
  0xF9, 0x14, 0x35, 0xEB, 0x5C, 0x27, 0x82, 0x4E,  
  0x2E, 0xCB, 0x42, 0x1C, 0x74, 0xA7, 0xBD, 0x61,   
  0x85, 0x50, 0x3F, 0xFA, 0xD3, 0x09, 0xE8, 0x96,  
  0x4B, 0x28, 0x1C, 0xB7, 0xA1, 0xDE, 0x72, 0x8D,   
  0xF6, 0x9F, 0xC0, 0x59, 0x6A, 0x34, 0x05, 0xE3,  
  0xCA, 0x1F, 0xA4, 0xF2, 0x97, 0x2C, 0x69, 0x85,   
  0x06, 0xD1, 0x3D, 0x4E, 0xE0, 0x7B, 0x53, 0xB8,  
  0x94, 0xE3, 0xF2, 0x5C, 0x29, 0x85, 0xCF, 0x3A,   
  0x7B, 0x0E, 0x41, 0xA7, 0x16, 0xD0, 0xB8, 0x6D,  
  0x4D, 0xB0, 0x2B, 0xE7, 0xF4, 0x09, 0x81, 0xDA,   
  0x3E, 0xC3, 0x95, 0x7C, 0x52, 0xAF, 0x68, 0x16,  
  0x16, 0x4B, 0xBD, 0xD8, 0xC1, 0x34, 0x7A, 0xE7,   
  0xA9, 0xF5, 0x60, 0x8F, 0x0E, 0x52, 0x93, 0x2C,  
  0xD1, 0x2F, 0x8D, 0x48, 0x6A, 0xF3, 0xB7, 0x14,   
  0xAC, 0x95, 0x36, 0xEB, 0x50, 0x0E, 0xC9, 0x72,  
  0x72, 0xB1, 0x4E, 0x17, 0x94, 0xCA, 0xE8, 0x2D,   
  0x0F, 0x6C, 0xA9, 0xD0, 0xF3, 0x35, 0x56, 0x8B  
}  
BYTE array\_406EB4\[\] =  
{  
  0x1A, 0x12, 0x0C, 0x04, 0x16, 0x07, 0x01, 0x11,  
  0x0B, 0x13, 0x05, 0x1D, 0x09, 0x0F, 0x19, 0x02,  
  0x1B, 0x15, 0x0A, 0x00, 0x1F, 0x06, 0x18, 0x10,  
  0x03, 0x17, 0x0D, 0x1C, 0x1E, 0x08, 0x14, 0x0E  
};  
/\*\*\*\*\*十进制表示  
BYTE array\_406EB4\[\] =  
{  
  26,18,12,04,22,07,01,17,  
  11,19,05,29,09,15,25,02,  
  27,21,10,00,31,06,24,16,  
  03,23,13,28,30,08,20,14  
}  
\*\*\*\*\*/  
  
  
DWORD pdw\_407750\[8\*0x40\];  
  
//i=ebp, j=ah, k=al  
int i,j,m=0,n;  
BYTE k,temp;  
BOOL b;  
for(i=0; i< 0x20; i+=4)  
{  
    j = 1;  
    while(j <= 0x40)  
    {  
  //奇偶判断  
        if( (j & 1) != 0)  
        {  
      k = array\_406DB4\[m\];  
      m++;  
  }  
  
  do{  
      b = k & 0x80;  
      k << 1;  
      if(b)  
      {  
          temp = array\_406EB4\[i\];  
    \*pwd\_407750 |= (temp+1);    
      }  
  
      i++;  
        
  }while( (i&3) !=0 );  
    
  i -= 4;    
  j++  
  pdw\_407750++; //edi += 4;    
    }  
}  
  
此处生成了 256 字节如下数据：  
  
00407750  00 10 04 04 00 00 00 00 00 00 04 00 10 10 04 04  ................  
00407760  10 00 04 04 10 10 04 00 10 00 00 00 00 00 04 00  ................  
00407770  00 10 00 00 00 10 04 04 10 10 04 04 00 10 00 00  ................  
00407780  10 10 00 04 10 00 04 04 00 00 00 04 10 00 00 00  ................  
00407790  10 10 00 00 00 10 00 04 00 10 00 04 00 10 04 00  ................  
004077A0  00 10 04 00 00 00 04 04 00 00 04 04 10 10 00 04  ................  
004077B0  10 00 04 00 10 00 00 04 10 00 00 04 10 00 04 00  ................  
004077C0  00 00 00 00 10 10 00 00 10 10 04 00 00 00 00 04  ................  
004077D0  00 00 04 00 10 10 04 04 10 00 00 00 00 00 04 04  ................  
004077E0  00 10 04 04 00 00 00 04 00 00 00 04 00 10 00 00  ................  
004077F0  10 00 04 04 00 00 04 00 00 10 04 00 10 00 00 04  ................  
00407800  00 10 00 00 10 00 00 00 10 10 00 04 10 10 04 00  ................  
00407810  10 10 04 04 10 00 04 00 00 00 04 04 10 10 00 04  ................  
00407820  10 00 00 04 10 10 00 00 10 10 04 00 00 10 04 04  ................  
00407830  10 10 00 00 00 10 00 04 00 10 00 04 00 00 00 00  ................  
00407840  10 00 04 00 00 10 04 00 00 00 00 00 10 00 04 04  ................  
  
现在还不知道这些数据做什么用的，先继续往下看...  
  
CODE:004012A3 loc\_4012A3:  
CODE:004012A3                 push    offset aSrv32res ; "Srv32Res"  
CODE:004012A8                 call    sub\_4019A5      ; 打开已存在的文件 srv32res，并得到其大小  
CODE:004012AD                 inc     eax             ; eax 为文件 srv32res 的大小  
CODE:004012AE                 cmp     eax, 9  
CODE:004012B1                 jnb     short loc\_4012C3 ; 如果文件 srv32res 大于 8 个字节则跳转  
CODE:004012B3                 push    offset aSccss   ; "sccss"  
CODE:004012B8                 call    sub\_4019A5      ; 打开文件 sccss  
CODE:004012BD                 inc     eax  
CODE:004012BE                 cmp     eax, 9  
CODE:004012C1                 jb      short loc\_4012C8 ; 如果文件 sccss 小于 8 个字节则跳转  
  
这里调用 sub\_4019A5 来判断 Windows 目录下文件 Srv32Res 和 sccss 的大小，第一次执行到这里没有这两个  
文件，所以跳转了。  
  
CODE:004012C3 loc\_4012C3:  
CODE:004012C3                 call    sub\_40148D  
CODE:004012C8   
CODE:004012C8 loc\_4012C8:  
CODE:004012C8                 call    sub\_401575  
CODE:004012CD                 test    eax, eax        ; 如果打开文件 srv32tsk 出错,eax == 0  
CODE:004012CF                 jnz     short loc\_4012E5  
CODE:004012D1                 call    sub\_40148D  
CODE:004012D6                 push    0FFFFFFFFh      ; dwMilliseconds = INFINITE  
CODE:004012D8                 push    ds:hHandle      ; hHandle  
CODE:004012DE                 call    WaitForSingleObject  
CODE:004012E3                 jmp     short loc\_4012C8  
  
这一段代码会不断执行 sub\_401575,sub\_40148D 直到 sub\_401575 的返回值（eax）不等于 0；  
看了这两个函数应该事关键了，我们先看看函数 sub\_401575:  
  
CODE:00401575 sub\_401575      proc near   
CODE:00401575 var\_4           = dword ptr -4  
CODE:00401575                 enter   4, 0  
CODE:00401579                 push    esi  
CODE:0040157A                 mov     \[ebp+var\_4\], 0  
CODE:00401581                 push    offset dword\_406587  
CODE:00401586                 call    sub\_4014CA      ; 读文件 Srv32tsk 的内容,存放到\[406587\]  
CODE:0040158B                 cmp     eax, 0FFFFFFFFh  
CODE:0040158E                 jz      short loc\_4015E1 ; 打开文件 srv32tsk 失败则退出并返回 0  
CODE:00401590                 mov     esi, eax  
CODE:00401592                 mov     \[ebp+var\_4\], 1  
CODE:00401599                 cmp     ds:dword\_406593, 0  
CODE:004015A0                 jz      short loc\_4015AA  
CODE:004015A2                 push    esi             ; hObject  
CODE:004015A3                 call    CloseHandle  
CODE:004015A8                 jmp     short loc\_4015E1  
  
跟进 sub\_4014CA 看看它读文件的操作  
  
CODE:004014CA sub\_4014CA      proc near  
CODE:004014CA NumberOfBytesRead= dword ptr -8  
CODE:004014CA lpBuffer        = dword ptr  8  
CODE:004014CA   
CODE:004014CA                 enter   8, 0  
CODE:004014CE                 push    0               ; hTemplateFile  
CODE:004014D0                 push    0               ; dwFlagsAndAttributes  
CODE:004014D2                 push    3               ; OPEN\_EXISTING  
CODE:004014D4                 push    0               ; lpSecurityAttributes  
CODE:004014D6                 push    0               ; dwShareMode  
CODE:004014D8                 push    0C0000000h      ; dwDesiredAccess  
CODE:004014DD                 push    offset aSrv32tsk ; lpFileName = "Srv32tsk"  
CODE:004014E2                 call    CreateFileA  
CODE:004014E7                 cmp     eax, 0FFFFFFFFh  
CODE:004014EA                 jz      short locret\_40152D ; 打开文件失败则退出  
  
打开 Windows 目录下的文件 srv32tsk，到 CreateFileA 会失败，我们可以先建一个文件  
srv32tsk，然后看它对文件做了些什么操作  
  
  
CODE:004014EC                 mov     esi, eax  
CODE:004014EE                 lea     eax, \[ebp-8\]  
CODE:004014F1                 push    0               ; lpOverlapped  
CODE:004014F3                 push    eax             ; lpNumberOfBytesRead  
CODE:004014F4                 push    824h            ; nNumberOfBytesToRead = 2084B  
CODE:004014F9                 push    dword ptr \[ebp+8\] ; lpBuffer  
CODE:004014FC                 push    esi             ; hFile  
CODE:004014FD                 call    ReadFile  
CODE:00401502                 cmp     dword ptr \[ebp-8\], 2Ch ; 读了文件 srv32tsk 的字节数和 44 比较  
CODE:00401506                 jb      short loc\_401522 ; 小于 44 个字节退出  
CODE:00401508                 mov     edx, \[ebp+8\]    ; 读取的内容  
CODE:0040150B                 mov     eax, \[edx+20h\]  ; 文件 srv32tsk 的第 32 个字节开始的 DWORD 值  
CODE:0040150E                 shl     eax, 3  
CODE:00401511                 add     eax, 24h  
CODE:00401514                 cmp     eax, \[ebp-8\]    ; 把上面的 DWORD 值乘以 8 加上 24 后和文件大小比较  
CODE:00401517                 jnz     short loc\_401522  
CODE:00401519                 cmp     byte ptr \[edx\], 0 ; 判断文件的第一个字节是否为 0  
CODE:0040151C                 jz      short loc\_401522  
CODE:0040151E                 mov     eax, esi  
CODE:00401520                 jmp     short locret\_40152D  
CODE:00401522 ; ///////////////////////////////////////////////////////////////////////////  
CODE:00401522   
CODE:00401522 loc\_401522:   
CODE:00401522                 push    esi             ; hObject  
CODE:00401523                 call    CloseHandle  
CODE:00401528                 mov     eax, 0FFFFFFFFh  
CODE:0040152D   
CODE:0040152D locret\_40152D:   
CODE:0040152D                 leave  
CODE:0040152E                 retn    4  
CODE:0040152E sub\_4014CA      endp  
  
返回到函数 sub\_401575：  
  
CODE:0040158B                 cmp     eax, 0FFFFFFFFh  
CODE:0040158E                 jz      short loc\_4015E1 ; 打开文件 srv32tsk 失败则退出并返回 0  
CODE:00401590                 mov     esi, eax  
CODE:00401592                 mov     \[ebp+var\_4\], 1  
CODE:00401599                 cmp     ds:dword\_406593, 0 ; 从文件 srv32tsk 开始的第 4 个 DWORD 值  
CODE:004015A0                 jz      short loc\_4015AA  
CODE:004015A2                 push    esi             ; hObject  
CODE:004015A3                 call    CloseHandle     ; 关闭文件  
CODE:004015A8                 jmp     short loc\_4015E1 ; 退出  
CODE:004015AA ; ///////////////////////////////////////////////////////////////////////////  
CODE:004015AA   
CODE:004015AA loc\_4015AA:                             ; CODE XREF: sub\_401575+2Bj  
CODE:004015AA                 cmp     ds:dword\_40658B, 0 ; 从文件开始的第 2 个 DWORD 值  
CODE:004015B1                 jz      short loc\_4015D9  
CODE:004015B3   
CODE:004015B3 loc\_4015B3:   
CODE:004015B3                 call    sub\_40171C  
CODE:004015B8                 mov     ds:dword\_406593, 1 ; 从文件 srv32tsk 开始的第 4 个 DWORD 值  
CODE:004015C2                 mov     ds:dword\_40658B, 0 ; 从文件 srv32tsk 开始的第 2 个 DWORD 值  
CODE:004015CC                 push    offset dword\_406587  
CODE:004015D1                 push    esi  
CODE:004015D2                 call    sub\_401531      ; 写文件 srv32tsk  
CODE:004015D7                 jmp     short loc\_4015E1  
CODE:004015D9 ; ///////////////////////////////////////////////////////////////////////////  
CODE:004015D9   
CODE:004015D9 loc\_4015D9:  
CODE:004015D9                 inc     ds:dword\_40658F ; 从文件 srv32tsk 开始的第 3 个 DWORD 值  
CODE:004015DF                 jmp     short loc\_4015B3  
CODE:004015E1 ; ///////////////////////////////////////////////////////////////////////////  
CODE:004015E1   
CODE:004015E1 loc\_4015E1:  
CODE:004015E1                 mov     eax, \[ebp+var\_4\]  
CODE:004015E4                 pop     esi  
CODE:004015E5                 leave  
CODE:004015E6                 retn  
CODE:004015E6 sub\_401575      endp  
  
文件 srv32tsk 打开失败的话直接退出，如果读文件成功，比较从文件开始读的第 4 个 DWORD  
值是否为 0，不等于 0 关闭文件退出此函数，等于 0 的话比较第 2 个 DWORD 值是否为 0，如果第  
2 个 DWORD 值不为 0 的话调用 sub\_40171C，然后又调用 sub\_401531 写文件 srv32tsk。  
  
文件 srv32tsk 的头部的结构大概是这样的  
struct tagSRV32TSK\_HEAD  
{  
  DWORD dword\_406587;  
  DWORD dword\_40658B;  
  DWORD dword\_40658F;  
  DWORD dword\_406593;  
  DWORD dword\_406597;  
  DWORD dword\_40659B;  
  BYTE\[8\] unk\_40659F;  
  DWORD dword\_4065A7;   
};  
  
这一段代码里调用了一个函数 sub\_40171C:  
  
CODE:0040171C sub\_40171C      proc near  
CODE:0040171C                 mov     eax, ds:dword\_40658F ; 从文件 srv32tsk 开始的第 3 个 DWORD 值  
CODE:00401721                 rcl     eax, 0Dh  
CODE:00401724                 rcl     edx, 1  
CODE:00401726                 rcl     eax, 1  
CODE:00401728                 rcl     edx, 1  
CODE:0040172A                 rcl     eax, 1  
CODE:0040172C                 rcl     edx, 1  
CODE:0040172E                 rcl     eax, 1  
CODE:00401730                 rcl     edx, 1  
CODE:00401732                 rcl     eax, 1  
CODE:00401734                 rcl     edx, 4  
CODE:00401737                 rcl     eax, 1  
CODE:00401739                 rcl     edx, 1  
CODE:0040173B                 rcl     eax, 1  
CODE:0040173D                 rcl     edx, 3  
CODE:00401740                 rcl     eax, 1  
CODE:00401742                 rcl     edx, 1  
CODE:00401744                 rcl     eax, 1  
CODE:00401746                 rcl     edx, 1  
CODE:00401748                 rcl     eax, 1  
CODE:0040174A                 rcl     edx, 2  
CODE:0040174D                 rcl     eax, 1  
CODE:0040174F                 rcl     edx, 1  
CODE:00401751                 rcl     eax, 1  
CODE:00401753                 rcl     edx, 1  
CODE:00401755                 rcl     eax, 1  
CODE:00401757                 rcl     edx, 1  
CODE:00401759                 rcl     eax, 1  
CODE:0040175B                 rcl     edx, 1  
CODE:0040175D                 rcl     eax, 1  
CODE:0040175F                 rcl     edx, 1  
CODE:00401761                 rcl     eax, 1  
CODE:00401763                 rcl     edx, 1  
CODE:00401765                 rcl     eax, 1  
CODE:00401767                 rcl     edx, 2  
CODE:0040176A                 rcl     eax, 1  
CODE:0040176C                 rcl     edx, 1  
CODE:0040176E                 rcl     eax, 1  
CODE:00401770                 rcl     edx, 1  
CODE:00401772                 rcl     eax, 1  
CODE:00401774                 rcl     edx, 6  
CODE:00401777                 and     edx, 0F8CEFEE0h  
CODE:0040177D                 bswap   edx  
CODE:0040177F                 mov     ds:dword\_406597, edx ; 文件 srv32tsk 的第 5 个 DW 值  
CODE:00401785                 xor     edx, edx  
CODE:00401787                 mov     ds:dword\_40659B, edx ; 文件 srv32tsk 的第 6 个 DW 值  
CODE:0040178D                 retn  
CODE:0040178D sub\_40171C      endp  
  
对文件 srv32tsk 操作完后返回到 sub\_40123E  
  
CODE:004012CD                 test    eax, eax        ; 如果打开文件 srv32tsk 出错,eax == 0  
CODE:004012CF                 jnz     short loc\_4012E5  
CODE:004012D1                 call    sub\_40148D  
CODE:004012D6                 push    0FFFFFFFFh      ; dwMilliseconds = INFINITE  
CODE:004012D8                 push    ds:hHandle      ; hHandle  
CODE:004012DE                 call    WaitForSingleObject  
CODE:004012E3                 jmp     short loc\_4012C8  
  
现在还有一个会频繁执行的函数 sub\_40148D  
  
CODE:0040148D sub\_40148D      proc near  
CODE:0040148D                 cmp     ds:ThreadId, 0  
CODE:00401494   
CODE:00401494 loc\_401494:  
CODE:00401494                 jnz     short locret\_4014C9  
CODE:00401496                 cmp     ds:hHandle, 0FFFFFFFFh  
CODE:0040149D                 jz      short loc\_4014AA  
CODE:0040149F                 push    ds:hHandle      ; hObject  
CODE:004014A5                 call    CloseHandle  
CODE:004014AA   
CODE:004014AA loc\_4014AA:   
CODE:004014AA                 push    offset ThreadId ; lpThreadId  
CODE:004014AF                 push    0               ; dwCreationFlags  
CODE:004014B1                 push    0               ; lpParameter  
CODE:004014B3                 push    401DB0h         ; lpStartAddress  
CODE:004014B8                 push    2000h           ; dwStackSize  
CODE:004014BD                 push    0               ; lpThreadAttributes  
CODE:004014BF                 call    CreateThread  
CODE:004014C4                 mov     ds:hHandle, eax  
CODE:004014C9   
CODE:004014C9 locret\_4014C9:  
CODE:004014C9                 retn  
CODE:004014C9 sub\_40148D      endp  
  
创建了一个线程（^\_^，到这里才刚刚开始噢！），肯定不会干什么好事，下次  
我们再详细分析这个线程。  
  
到这里有个 2 问题：  
1. 由函数 sub\_4033E7 算出的内存数据做何用？  
2. 文件 srv32Res,sccss 何 srv32tsk 做何用？ 

 4dd466d0e1e0cc123afbe504da2462ca