

# Rav 2005 中 HOOK 的初步分析

> 原文：[`www.pediy.com/kssd/pediy06/pediy6553.htm`](https://www.pediy.com/kssd/pediy06/pediy6553.htm)

分析了半天，准备放弃了。不过因为和驱动有关，而且分析过程也和 RE 挂钩，所以放上来给兄弟们看看。有知道的讨论一下，没有懂 NT 内核的作个参考，算废物利用吧。  
  
废话几句，记得 Xtreme 是用驱动的……寒一个……  
  
\===================================================================  
对 Rav 2005 中 HOOK 的初步分析  
  
昨天升级了一下 RAV2004。本以为很快就完，但升级程序却拖了近一个小时。启动后发现，RAV 已经被升级到了 2005。这事本应该就这么完了，但恰巧我对各种软件的内存占用非常敏感，弄了个 empty.exe 放在 bat 里，定期清理一下内存。执行了一遍 bat 却发现 empty 报错，说无法清理 ravmond.exe！于是本能地觉得有名堂。赶紧分析了一下 empty。  
  
从出错信息入手，可以找到下面的代码（注释是我加的）：  
  
.text:01001603 loc\_1001603:  
.text:01001603                 call    sub\_1001D1B  
.text:01001608                 mov     eax, dword\_12D3180 ; EAX 里应该是 PID  
.text:0100160D                 test    eax, eax  
.text:0100160F                 jz      short loc\_100163D  
.text:01001611                 push    eax  
.text:01001612                 call    sub\_1001F2C     ; 清理内存  
.text:01001617                 test    eax, eax  
.text:01001619                 jnz     short loc\_1001636  
.text:0100161B                 push    dword\_12D3180  
.text:01001621                 push    offset aCouldNotEmptyW ; "could not empty working set for process"...  
.text:01001626                 call    ds:printf  
.text:0100162C                 pop     ecx  
  
　　OK，到 sub\_1001F2C 去看看：  
  
.text:01001F2C                 push    ebp  
.text:01001F2D                 mov     ebp, esp  
.text:01001F2F                 push    ecx  
.text:01001F30                 push    esi  
.text:01001F31                 push    edi  
.text:01001F32                 push    \[ebp+MaximumWorkingSetSize\] ; dwProcessId  
.text:01001F35                 xor     edi, edi  
.text:01001F37                 push    edi             ; bInheritHandle  
.text:01001F38                 push    1F0FFFh         ; dwDesiredAccess  
.text:01001F3D                 call    ds:OpenProcess  
.text:01001F43                 mov     esi, eax  
.text:01001F45                 cmp     esi, edi  
.text:01001F47                 jnz     short loc\_1001F4D  
.text:01001F49                 xor     eax, eax  
.text:01001F4B                 jmp     short loc\_1001F77  
  
首先是以 PROCESS\_ALL\_ACCESS 权限调用 OpenProcess。但就是在这里，esi=edi，跳到出口 loc\_1001F77。  
  
好，于是写段调用 OpenProcess 的程序。结果发现，对 ravmond 的操作结果是返回 ERROR\_INVALID\_PARAMETER？？！！但是我的参数没有写错啊！然后开始怀疑 bInheritHandle 参数，但改了还是一样。那么，换 PID。换了 SMSS 的 PID，说没有权限，很正常。换 delphi32 的 PID，可以正常打开。那么，就是说，参数绝对是正确的，只是 RAV 玩了个花样。那样，按照系统给的 PID 去找就找不到了。  
  
既然这样，那就拿 RAV 开刀。先翻了一下 RAV 的目录，有几个驱动文件引起了我的注意：  
  
Hooksys.sys  
hookbase.sys  
HOOKREG.sys  
HookCont.sys  
HookApi.sys  
  
难道是 OpenProcess 被 HOOK 了？于是先在 HookApi 里查 import，没有可疑的 API。hookbase？Hooksys？HookCont？都没有。  
  
还是写程序看看……GetProcAddress 却报告说没有异常。  
  
没办法，查找 ravmond 吧……HOOKREG 里居然有 ravmond 的字样！改之，重新启动 RAV 的服务，bingo！在任务管理器里 k 进程时显示没有权限！那么很明显是被 HOOK 了。  
  
以前玩过 rootkit（只是玩而已，写不来），本来是应该想到的……  
  
那么反汇编 HOOKREG，查找 ravmond 的 xref……在 00011BEF 有一个 push，向上找到函数入口，得到整个函数：  
  
.text:00011B74 sub\_11B74       proc near  
.text:00011B74  
.text:00011B74 var\_2C          = dword ptr -2Ch  
.text:00011B74 var\_28          = byte ptr -28h  
.text:00011B74 var\_9           = byte ptr -9  
.text:00011B74 var\_8           = dword ptr -8  
.text:00011B74 var\_4           = dword ptr -4  
.text:00011B74 arg\_0           = dword ptr  8  
.text:00011B74  
.text:00011B74                 push    ebp  
.text:00011B75                 mov     ebp, esp  
.text:00011B77                 sub     esp, 2Ch  
.text:00011B7A                 mov     \[ebp+var\_2C\], 0  
.text:00011B81                 mov     \[ebp+var\_28\], 0  
.text:00011B85                 call    ds:IoGetCurrentProcess  
.text:00011B8B                 mov     \[ebp+var\_8\], eax  
//指向当前进程开头的指针  
.text:00011B8E                 mov     eax, \[ebp+var\_8\]  
.text:00011B91                 add     eax, dword\_12F48  
//dword\_12F48 的真实偏移  
.text:00011B97                 mov     \[ebp+var\_4\], eax  
.text:00011B9A                 mov     ecx, \[ebp+var\_4\]  
.text:00011B9D                 push    ecx  
.text:00011B9E                 lea     edx, \[ebp+var\_28\]  
//缓冲的长度是$1F  
.text:00011BA1                 push    edx  
.text:00011BA2                 call    strcpy  
.text:00011BA7                 add     esp, 8  
//复制  
.text:00011BAA                 mov     \[ebp+var\_9\], 0  
.text:00011BAE                 lea     eax, \[ebp+var\_28\]  
.text:00011BB1                 push    eax             ; char \*  
.text:00011BB2                 call    ds:\_strupr  
.text:00011BB8                 add     esp, 4  
//大写  
.text:00011BBB                 cmp     \[ebp+arg\_0\], 0  
.text:00011BBF                 jz      short loc\_11BD1  
//若参数是 0 就走  
.text:00011BC1                 lea     ecx, \[ebp+var\_28\]  
//dword\_12F48 的复件  
.text:00011BC4                 push    ecx  
.text:00011BC5                 mov     edx, \[ebp+arg\_0\]  
.text:00011BC8                 push    edx  
.text:00011BC9                 call    strcpy  
.text:00011BCE                 add     esp, 8  
//这样看，参数应该是 PChar  
//参数非 0 就给个 dword\_12F48 的复件  
.text:00011BD1 loc\_11BD1:  
.text:00011BD1                 push    offset dword\_11B50  
.text:00011BD6                 lea     eax, \[ebp+var\_28\]  
.text:00011BD9                 push    eax  
.text:00011BDA                 call    strcmp  
.text:00011BDF                 add     esp, 8  
.text:00011BE2                 test    eax, eax  
.text:00011BE4                 jnz     short loc\_11BEF  
//和 dword\_11B50 比较（RAV.EXE），若不等就继续，否则返回 2  
.text:00011BE6                 mov     \[ebp+var\_2C\], 2  
.text:00011BED                 jmp     short loc\_11C41  
.text:00011BEF loc\_11BEF:  
.text:00011BEF                 push    offset dword\_11B58  
.text:00011BF4                 lea     ecx, \[ebp+var\_28\]  
.text:00011BF7                 push    ecx  
.text:00011BF8                 call    strcmp  
.text:00011BFD                 add     esp, 8  
.text:00011C00                 test    eax, eax  
.text:00011C02                 jz      short loc\_11C19  
//和 dword\_11B58 比较（RAVMOND.EXE），若相等就返回 3，否则继续  
.text:00011C04                 push    offset dword\_11B64  
.text:00011C09                 lea     edx, \[ebp+var\_28\]  
.text:00011C0C                 push    edx  
.text:00011C0D                 call    strcmp  
.text:00011C12                 add     esp, 8  
.text:00011C15                 test    eax, eax  
.text:00011C17                 jnz     short loc\_11C22  
//和 dword\_11B64 比较（RAVMON.EXE），若相等就返回 3，否则继续  
.text:00011C19 loc\_11C19:  
.text:00011C19                 mov     \[ebp+var\_2C\], 3  
.text:00011C20                 jmp     short loc\_11C41  
.text:00011C22 loc\_11C22:  
.text:00011C22                 push    3               ; size\_t  
.text:00011C24                 push    offset aRav     ; char \*  
.text:00011C29                 lea     eax, \[ebp+var\_28\]  
.text:00011C2C                 push    eax             ; char \*  
.text:00011C2D                 call    ds:strncmp  
.text:00011C33                 add     esp, 0Ch  
.text:00011C36                 test    eax, eax  
.text:00011C38                 jnz     short loc\_11C41  
//检查 var\_28 的头 3 个字符是否 RAV，是就返回 4，不是就直接返回  
.text:00011C3A                 mov     \[ebp+var\_2C\], 4  
.text:00011C41 loc\_11C41:  
.text:00011C41                 mov     eax, \[ebp+var\_2C\]  
.text:00011C44                 mov     esp, ebp  
.text:00011C46                 pop     ebp  
.text:00011C47                 retn    4  
.text:00011C47 sub\_11B74       endp  
  
我们可以看到，这个东西有点象 call/pop 的手法。好处就是可以不要数据段（反正特征字符串是只读的）。  
  
那么，这只是一个判断。判断 dword\_12F48 是否含有特定的字符串，若是 RAV 的核心就返回 3，主程序返回 2，外围程序返回 4，其他返回 0。  
  
那么来看看 sub\_11B74 的 xref，有三处：11D27、119E6、115E2，对应的 sub 分别是 sub\_11D00、sub\_119D0、sub\_115BC。  
  
sub\_115BC 有 4 个 xref，全部是 code 引用，sub\_119D0 只有 1 个 data 引用的 xref，而 11D00，有 3 个 xref，data 引用。  
  
先看 11D00，xref 全在 start 里：  
  
.text:00010453 8B 45 08                          mov     eax, \[ebp+DriverObject\]  
.text:00010456 C7 40 38 00 1D 01+                mov     dword ptr \[eax+38h\], offset sub\_11D00  
.text:0001045D 8B 4D 08                          mov     ecx, \[ebp+DriverObject\]  
.text:00010460 C7 41 40 00 1D 01+                mov     dword ptr \[ecx+40h\], offset sub\_11D00  
.text:00010467 8B 55 08                          mov     edx, \[ebp+DriverObject\]  
.text:0001046A C7 82 80 00 00 00+                mov     dword ptr \[edx+80h\], offset sub\_11D00  
  
下面就 dump 一下和 DriverObject 有关的代码（eax,ecx,edx 都是临时使用的）：  
  
.text:00010308                   ; int \_\_stdcall start(PDRIVER\_OBJECT DriverObject)  
.text:00010308                                   public start  
.text:00010308                   start           proc near  
.text:00010308  
.text:00010308                   var\_64          = dword ptr -64h  
.text:00010308                   var\_60          = dword ptr -60h  
.text:00010308                   SymbolicLinkName= UNICODE\_STRING ptr -5Ch  
.text:00010308                   DeviceObject    = dword ptr -54h  
.text:00010308                   DeviceName      = UNICODE\_STRING ptr -50h  
.text:00010308                   SourceString    = word ptr -48h  
.text:00010308                   var\_28          = dword ptr -28h  
.text:00010308                   DriverObject    = dword ptr  8  
.text:00010308  
.text:00010308 55                                push    ebp  
.text:00010309 8B EC                             mov     ebp, esp  
.text:0001030B 83 EC 64                          sub     esp, 64h  
.text:0001030E 56                                push    esi  
.text:0001030F 57                                push    edi  
.text:00010310 C7 45 AC 00 00 00+                mov     \[ebp+DeviceObject\], 0  
.....  
.text:0001036A 8D 55 AC                          lea     edx, \[ebp+DeviceObject\]  
.text:0001036D 52                                push    edx             ; DeviceObject  
.text:0001036E 6A 00                             push    0               ; Exclusive  
.text:00010370 6A 00                             push    0               ; DeviceCharacteristics  
.text:00010372 68 00 84 00 00                    push    8400h           ; DeviceType  
.text:00010377 8D 45 B0                          lea     eax, \[ebp+DeviceName\]  
.text:0001037A 50                                push    eax             ; DeviceName  
.text:0001037B 6A 00                             push    0               ; DeviceExtensionSize  
.text:0001037D 8B 4D 08                          mov     ecx, \[ebp+DriverObject\]  
.text:00010380 51                                push    ecx             ; DriverObject  
.text:00010381 FF 15 DC 26 01 00                 call    ds:IoCreateDevice  
.....  
.text:000103BD 8B 45 08                          mov     eax, \[ebp+DriverObject\]  
.text:000103C0 8B 48 04                          mov     ecx, \[eax+4\]  
.text:000103C3 51                                push    ecx             ; DeviceObject  
.text:000103C4 FF 15 D4 26 01 00                 call    ds:IoDeleteDevice  
.text:000103CA 8B 45 9C                          mov     eax, \[ebp+var\_64\]  
.....  
.text:00010453 8B 45 08                          mov     eax, \[ebp+DriverObject\]  
.text:00010456 C7 40 38 00 1D 01+                mov     dword ptr \[eax+38h\], offset sub\_11D00  
.text:0001045D 8B 4D 08                          mov     ecx, \[ebp+DriverObject\]  
.text:00010460 C7 41 40 00 1D 01+                mov     dword ptr \[ecx+40h\], offset sub\_11D00  
.text:00010467 8B 55 08                          mov     edx, \[ebp+DriverObject\]  
.text:0001046A C7 82 80 00 00 00+                mov     dword ptr \[edx+80h\], offset sub\_11D00  
.text:00010474 8B 45 08                          mov     eax, \[ebp+DriverObject\]  
.text:00010477 C7 40 70 BF 1D 01+                mov     dword ptr \[eax+70h\], offset sub\_11DBF  
.text:0001047E 8B 4D 08                          mov     ecx, \[ebp+DriverObject\]  
.text:00010481 C7 41 34 68 05 01+                mov     dword ptr \[ecx+34h\], offset sub\_10568  
  
就是填充 PDRIVER\_OBJECT，其他没有什么。这里的大概意思是创建一个内核 Device，然后定义几个 Function。问题是，这个驱动的 Type 是自定义的，而我又不懂 Driver……  
  
不过还好，119D0 的 xref 有点意思：  
  
.text:00010CBA                 mov     eax, ds:ZwSetValueKey  
.text:00010CBF                 mov     ecx, \[eax+1\]  
.text:00010CC2                 mov     edx, ds:KeServiceDescriptorTable  
.text:00010CC8                 mov     eax, \[edx\]  
.text:00010CCA                 mov     dword ptr \[eax+ecx\*4\], offset sub\_119D0  
  
那么，这里是修改 ServiceDescriptorTable 里 ZwSetValueKey 的入口。联想一下，应该是不允许修改 RAVMOND 服务的 Key。  
  
最后看看 115BC，xref 对应的 sub 有 111A0，112F0，11360 和 119D0。而它们都是只有一个 data 的 xref，应该是 function 了：  
  
.text:00010CD1                 mov     ecx, ds:ZwCreateKey  
.text:00010CD7                 mov     edx, \[ecx+1\]  
.text:00010CDA                 mov     eax, ds:KeServiceDescriptorTable  
.text:00010CDF                 mov     ecx, \[eax\]  
.text:00010CE1                 mov     dword ptr \[ecx+edx\*4\], offset sub\_111A0  
.text:00010CE8                 mov     edx, ds:ZwDeleteValueKey  
.text:00010CEE                 mov     eax, \[edx+1\]  
.text:00010CF1                 mov     ecx, ds:KeServiceDescriptorTable  
.text:00010CF7                 mov     edx, \[ecx\]  
.text:00010CF9                 mov     dword ptr \[edx+eax\*4\], offset sub\_11360  
.text:00010D00                 mov     eax, ds:ZwDeleteKey  
.text:00010D05                 mov     ecx, \[eax+1\]  
.text:00010D08                 mov     edx, ds:KeServiceDescriptorTable  
.text:00010D0E                 mov     eax, \[edx\]  
.text:00010D10                 mov     dword ptr \[eax+ecx\*4\], offset sub\_112F0  
  
没有什么，都是常规的 Reg 读写。失望 ing……  
  
回头再找，居然在 Hooksys 里看见了 ravmond！看来下次要用 Hex editer 了……用 BIEW 虽然方便，但很容易看漏的。而且，还犯了轻信 IDA 的错误……  
  
OK，再看 Hooksys！有了刚才的经验，这次应该很简单了。  
  
很快发现字符串'RAV.EXE'在 11D70 处。判断子程序是 sub\_11ED4。但是居然没有使用 KeServiceDescriptorTable！而且驱动的 Type 是 8300h，自定义。  
  
下面就没有什么进展了。在网上搜了搜，发现有人问过类似的问题。不过他要知道的是如何完美地对文件系统 Hook。  
  
不过可以肯定的是，RAV 没有使用常规的方法来 Hook。有可能是通过 FileSystem 或自己直接计算内存地址来实现 Hook 的。  
  
PS:在小四的站看了篇 Hume 的文章，讲通过 SEH 获取 Ker32 基址的。在 IDA 里搜了搜，发现有  
  
.text:00012CD9                 mov     eax, large fs:4  
.text:00012CDF                 mov     \[ebp+VirtualAddress\], eax  
  
很可能就是这个了……暴力搜索，然后改 cr0，直接修改物理地址，完美 Hook……猜测而已……  
  
TMD 简直跟 rootkit 没什么区别了。隐蔽得一塌糊涂，拿来做 rootkit 连 import 也不要，想 k 的话就乖乖地从头做逆向工程吧……不过做一遍也有好处，现在起码对驱动有点了解了。以前？觉得象天书……

 4dd466d0e1e0cc123afbe504da2462ca