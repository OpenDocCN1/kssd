

# 蠕虫 srv32.exe 逆向分析笔记 5

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-173.htm`](https://www.pediy.com/kssd/pediy07/pediy7-173.htm)

文件名称：srv32.exe  
蠕虫名称：Net-Worm.Win32.Opasoft.s  
工具：   IDA 4.5.1, SoftICE3.1  
  
  
  
第 3，4 篇中我们一直在创建的那个线程（401DB0）里打转，跟着程序的思路走我们又回到  
第 2 篇创建线程的位置：  
  
  
CODE:004012C3 loc\_4012C3:  
CODE:004012C3                 call    sub\_40148D  
CODE:004012C8   
CODE:004012C8 loc\_4012C8:   
CODE:004012C8                 call    sub\_401575      ; 返回值 1 或 0  
CODE:004012CD                 test    eax, eax        ; 如果打开文件 srv32tsk 出错,eax == 0  
CODE:004012CF                 jnz     short loc\_4012E5  
CODE:004012D1                 call    sub\_40148D  ;---->线程（401DB0）在这里创建  
CODE:004012D6                 push    0FFFFFFFFh      ; dwMilliseconds = INFINITE  
CODE:004012D8                 push    ds:hThreadHandle ; hHandle  
CODE:004012DE                 call    WaitForSingleObject ; 等待线程结束  
CODE:004012E3                 jmp     short loc\_4012C8  
CODE:004012E5 ; ///////////////////////////////////////////////////////////////////////////  
CODE:004012E5   
CODE:004012E5 loc\_4012E5:  
CODE:004012E5                 push    offset dword\_406597  
CODE:004012EA                 call    sub\_403332  
CODE:004012EF                 add     esp, 4  
CODE:004012F2                 mov     ds:byte\_406285, 1  
CODE:004012F9                 push    ds:dword\_4065A7  
CODE:004012FF                 push    offset unk\_4065AB  
CODE:00401304                 push    offset unk\_40659F  
CODE:00401309                 call    sub\_403431  
CODE:0040130E                 test    ds:byte\_406285, 1  
CODE:00401315                 jz      short loc\_4012A3  
CODE:00401317                 call    sub\_40178E  
CODE:0040131C                 push    eax  
CODE:0040131D                 call    sub\_401675  
CODE:00401322                 call    sub\_401634  
CODE:00401327                 jmp     short loc\_4012C3  
CODE:00401329 ; ///////////////////////////////////////////////////////////////////////////  
CODE:00401329   
CODE:00401329 locret\_401329:  
CODE:00401329                 leave  
CODE:0040132A                 retn  
CODE:0040132A sub\_40123E      endp  
  
  
在主线程 sub\_40123E 中调用 call sub\_40148D 创建了第 3，4 篇中介绍的线程，随后主线程  
又调用 WaitForSingleObject(hThreadhandle, INFINITE)等待创建的线程（401DB0）  
结束，WaitForSingleObject 返回后主线程回跳倒 loc\_4012C8：在 sub\_401575 中对文件  
srv32tsk 进行了读写操作如下：  
  
CODE:00401575 sub\_401575      proc near  
CODE:00401575   
CODE:00401575 var\_ReturnValue = dword ptr -4  
CODE:00401575   
CODE:00401575                 enter   4, 0  
CODE:00401579                 push    esi  
CODE:0040157A                 mov     \[ebp+var\_ReturnValue\], 0  
CODE:00401581                 push    offset dword\_406587  
CODE:00401586                 call    icy\_ReadFileSrv32tsk ; 读文件 Srv32tsk 的内容,存放到\[406587\]  
CODE:0040158B                 cmp     eax, 0FFFFFFFFh  
CODE:0040158E                 jz      short loc\_4015E1 ; 打开文件 srv32tsk 失败则退出并返回 0  
CODE:00401590                 mov     esi, eax  
CODE:00401592                 mov     \[ebp+var\_ReturnValue\], 1  
CODE:00401599                 cmp     ds:dword\_406593, 0 ; 从文件 srv32tsk 开始的第 4 个 DWORD 值  
CODE:004015A0                 jz      short loc\_4015AA  
CODE:004015A2                 push    esi             ; hObject  
CODE:004015A3                 call    CloseHandle     ; 关闭文件  
CODE:004015A8                 jmp     short loc\_4015E1 ; 退出  
CODE:004015AA ; ///////////////////////////////////////////////////////////////////////////  
CODE:004015AA   
CODE:004015AA loc\_4015AA:  
CODE:004015AA                 cmp     ds:dword\_40658B, 0 ; 从文件开始的第 2 个 DWORD 值  
CODE:004015B1                 jz      short loc\_4015D9  
CODE:004015B3   
CODE:004015B3 loc\_4015B3:  
CODE:004015B3                 call    sub\_40171C  
CODE:004015B8                 mov     ds:dword\_406593, 1 ;  从文件 srv32tsk 开始的第 4 个 DWORD 值  
CODE:004015C2                 mov     ds:dword\_40658B, 0 ; 从文件 srv32tsk 开始的第 2 个 DWORD 值  
CODE:004015CC                 push    offset dword\_406587  
CODE:004015D1                 push    esi  
CODE:004015D2                 call    icy\_WriteFileSrv32tsk ; 写文件 srv32tsk  
CODE:004015D7                 jmp     short loc\_4015E1  
CODE:004015D9 ; ///////////////////////////////////////////////////////////////////////////  
CODE:004015D9   
CODE:004015D9 loc\_4015D9:   
CODE:004015D9                 inc     ds:dword\_40658F ; 从文件 srv32tsk 开始的第 3 个 DWORD 值  
CODE:004015DF                 jmp     short loc\_4015B3  
CODE:004015E1 ; ///////////////////////////////////////////////////////////////////////////  
CODE:004015E1   
CODE:004015E1 loc\_4015E1:  
CODE:004015E1                 mov     eax, \[ebp+var\_ReturnValue\]  
CODE:004015E4                 pop     esi  
CODE:004015E5                 leave  
CODE:004015E6                 retn  
CODE:004015E6 sub\_401575      endp  
  
先调用 icy\_ReadFileSrv32tsk 读文件 srv32tsk，上一篇从 Internet 得到的 srv32tsk 的  
结构如下：  
struct tagSrv32tskFile  
{  
  DWORD dwTValue;    //ebp-834h  
  DWORD unknown1 = 1;  //ebp-830h  
  DWORD dwKValue;    //ebp-82Ch  
  DWORD unknown2 = 0;  //ebp-828h  
  DWORD unknown3;    //ebp-824h  
  DWORD unknown4;    //ebp-820h  
  INT64 PValue;    //ebp-81Ch  
  DWORD dwCount;    //ebp-814h  
  BYTE CValue\[744\]  //ebp-810h  
};  
  
判断第 4 个 DWORD 值是否为 0，0 表示第一次操作 srv32tsk 文件，需要调用下面的一些函数对文件 srv32tsk  
的某些字段进行转换，1 表示已经操作过 srv32tsk，在 sub\_40171C 中对 dwKValue 和存放 srv32tsk 内容的  
缓冲区地址的值 进行 RCL 移位等转换操作后把结果存到 srv32tsk 的第 5 个 DW 值，把第 6 个 DWORD 值清 0。  
  
 call    sub\_40171C：  
  
 CODE:0040171C sub\_40171C      proc near               ; CODE XREF: sub\_401575+3Ep  
CODE:0040171C                 mov     eax, ds:dword\_40658F ; dword\_40658F:dwKValue  
CODE:00401721                 rcl     eax, 0Dh  
CODE:00401724                 rcl     edx, 1          ; edx 为内存中 srv32tsk 的缓冲区地址  
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
  
接下来 icy\_WriteFileSrv32tsk 用更改后的缓冲区内容重写 srv32tsk 文件，随后 sub\_401575 退出返回值 1。  
  
CODE:004012C8                 call    sub\_401575      ; 返回值 1 或 0  
CODE:004012CD                 test    eax, eax        ; 如果打开文件 srv32tsk 出错,eax == 0  
CODE:004012CF                 jnz     short loc\_4012E5  
  
跳转实现，jmp 到：  
  
  
  
CODE:004012E5 ; ///////////////////////////////////////////////////////////////////////////  
CODE:004012E5   
CODE:004012E5 loc\_4012E5:  
CODE:004012E5                 push    offset dword\_406597 ; srv32tsk 第 5 个 DWORD 值  
CODE:004012EA                 call    sub\_403332  
CODE:004012EF                 add     esp, 4  
CODE:004012F2                 mov     ds:byte\_406285, 1  
CODE:004012F9                 push    ds:dword\_4065A7  
CODE:004012FF                 push    offset unk\_4065AB  
CODE:00401304                 push    offset unk\_40659F  
CODE:00401309                 call    sub\_403431  
CODE:0040130E                 test    ds:byte\_406285, 1  
CODE:00401315                 jz      short loc\_4012A3  
CODE:00401317                 call    sub\_40178E  
CODE:0040131C                 push    eax  
CODE:0040131D                 call    sub\_401675  
CODE:00401322                 call    sub\_401634  
CODE:00401327                 jmp     short loc\_4012C3  
CODE:00401329 ; ///////////////////////////////////////////////////////////////////////////  
CODE:00401329   
CODE:00401329 locret\_401329:  
CODE:00401329                 leave  
CODE:0040132A                 retn  
CODE:0040132A sub\_40123E      endp  
  
  
这里本应该调用几个函数 sub\_403332,sub\_403431,sub\_40178E,sub\_401675,sub\_401634  
Jmp 到 loc\_4012C3 继续 call    sub\_40148D，可在 sub\_403431 中一直循环好像出不来了，  
我想作者不会是为了耗系统资源而写的这个函数吧，不过偶实属菜鸟，希望能看得懂函数 sub\_403431  
的大虾们指点一二，到这里整个程序的执行已经基本结束，你也许认为 srv32 没有传播自己，也没对  
网络进行任何得攻击；作为蠕虫怎么会不传播自己呢，还记得在第 4 篇得时候有两个线程没有被执行吗.  
  
在如下位置：  
  
CODE:00402224                 call    sub\_401C49      ; 写注册表 srv32  
CODE:00402229                 cmp     ds:Data, 0  
CODE:00402230                 jz      short loc\_402239  
CODE:00402232                 call    sub\_4030EB  <<<<<这里创建了两个线程，做个标记  
CODE:00402237                 jmp     short loc\_40223E  
  
由于每次 ds:Data 的值都为 0，所以这两个线程被跳过；Data 的值为从 http://63.246.135.48/r.php?t=0  
读到的 D=?的值，srv32 的作者可能是通过 D=0 中止了蠕虫的继续传播。  
  
  
附件:srv32 IDA 反汇编文件  

 4dd466d0e1e0cc123afbe504da2462ca