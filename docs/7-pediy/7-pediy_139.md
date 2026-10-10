

# Dll 注入 NetTransport 2.25.337

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-404.htm`](https://www.pediy.com/kssd/pediy08/pediy8-404.htm)

【文章标题】: Dll 注入 NetTransport 2.25.337  
【文章作者】: wynney  
【软件名称】: NetTransport 2.25.337 Unicode  
【软件大小】: 3.46M  
【下载地址】: 打包下载  
【编写语言】: Microsoft Visual C++ 6.0  
【作者声明】: 冲冲看雪的气氛  
\--------------------------------------------------------------------------------  
【详细过程】  
  前言：前天看到 NetTransport 出了新版，而且是 Unicode 版本的，遂下来看了看，于是就有了下文  
    
  1、未注册，启动就有注册提示框  
  2、未注册，使用过程中不定时弹出注册框  
    
  一、定位注册框  
    
  F7 配合 F8 很容易就找到掉用注册框的地方  
    
  00443EB9     E8 4272FFFF        call NetTrans.0043B100            ; 如果是的话那这里就是关键 Call 了  
  00443EBE     8B4C24 18          mov ecx,dword ptr ss:\[esp+18\]  
  00443EC2     8B41 F8            mov eax,dword ptr ds:\[ecx-8\]      ; \[ecx-8\]=\[728776CC\]，记住他！  
  00443EC5     85C0               test eax,eax  
  00443EC7     75 78              jnz short NetTrans.00443F41       ; 看看/跳过的话那不就不用注册了？  
  00443EC9     6A 00              push 0  
  00443ECB     8D8C24 50010000    lea ecx,dword ptr ss:\[esp+150\]  
  00443ED2     E8 79860200        call NetTrans.0046C550  
  00443ED7     8D8C24 4C010000    lea ecx,dword ptr ss:\[esp+14C\]  
  00443EDE     C68424 04100000 0B mov byte ptr ss:\[esp+1004\],0B  
  00443EE6     E8 F72C0800        call <jmp.&MFC42u.#2506>          ; 调用注册框  
  00443EEB     3D 55050000        cmp eax,555  
    
  我们发现\[ecx-8\]=\[728776CC\]=1 才可以跳过去  
    
  小插曲：在 00443EB9 下 F2，再 F9 发现程序直接退出，图省事，不想追其原因，改在 00443EB9 下硬件执行断点吧  
    
  重载，F9，中断在 00443EB9，F7，来看看几段代码吧  
    
  0043B124     57                 push edi  
  0043B125     8BCE               mov ecx,esi  
  0043B127     E8 B0BD0800        call <jmp.&MFC42u.#2606>      ; 看看下面的注册表。。  
  0043B12C     8BBC24 90000000    mov edi,dword ptr ss:\[esp+90\] ; 呵呵，这个 Call 还真是关键哦  
  0043B133     85FF               test edi,edi  
  0043B135     74 06              je short NetTrans.0043B13D  
  0043B137     C707 00000000      mov dword ptr ds:\[edi\],0  
  0043B13D     6A 00              push 0  
  0043B13F     68 50534F00        push NetTrans.004F5350        ; UNICODE "Key"  
  0043B144     68 28534F00        push NetTrans.004F5328        ; UNICODE "SOFTWARE\\Xi\\NetXfer"  
  0043B149     8D4424 20          lea eax,dword ptr ss:\[esp+20\]  
  0043B14D     68 02000080        push 80000002  
  0043B152     50                 push eax  
  0043B153     B9 38B04F00        mov ecx,NetTrans.004FB038  
  0043B158     E8 D39B0000        call NetTrans.00444D30        ; 看到了吧，我们找对了  
  0043B15D     6A 00              push 0  
  0043B15F     68 14534F00        push NetTrans.004F5314        ; UNICODE "Signature"  
  0043B164     68 28534F00        push NetTrans.004F5328        ; UNICODE "SOFTWARE\\Xi\\NetXfer"  
  0043B169     8D4C24 1C          lea ecx,dword ptr ss:\[esp+1C\]  
  0043B16D     68 02000080        push 80000002  
  0043B172     51                 push ecx  
  0043B173     B9 38B04F00        mov ecx,NetTrans.004FB038  
    
    
  0043B2A8     8B41 F8            mov eax,dword ptr ds:\[ecx-8\]  
  0043B2AB     85C0               test eax,eax  
  0043B2AD     0F85 AA000000      jnz NetTrans.0043B35D  
  0043B2B3     8D5424 3C          lea edx,dword ptr ss:\[esp+3C\]  
  0043B2B7     8D8424 8C000000    lea eax,dword ptr ss:\[esp+8C\]  
  0043B2BE     52                 push edx  
  0043B2BF     8D8C24 94000000    lea ecx,dword ptr ss:\[esp+94\]  
  0043B2C6     50                 push eax  
  0043B2C7     51                 push ecx  
  0043B2C8     E8 CBBA0800        call <jmp.&MFC42u.#925>           ; 这里又是什么？  
  0043B2CD     8D9424 90000000    lea edx,dword ptr ss:\[esp+90\]  
  0043B2D4     68 00534F00        push NetTrans.004F5300            ; UNICODE ".key.kqm"  
  0043B2D9     8D4424 24          lea eax,dword ptr ss:\[esp+24\]  
  0043B2DD     52                 push edx  
  0043B2DE     50                 push eax  
  0043B2DF     C68424 90000000 05 mov byte ptr ss:\[esp+90\],5  
  0043B2E7     E8 ACBA0800        call <jmp.&MFC42u.#925>  
  0043B2EC     8B18               mov ebx,dword ptr ds:\[eax\]  
  0043B2EE     8D8C24 90000000    lea ecx,dword ptr ss:\[esp+90\]  
  0043B2F5     68 F4524F00        push NetTrans.004F52F4            ; UNICODE ".key"  
  0043B2FA     8D5424 20          lea edx,dword ptr ss:\[esp+20\]     ; 看样子像是注册成功后的 key 备份  
    
    
  0043B35F     8B42 F8            mov eax,dword ptr ds:\[edx-8\]  
  0043B362     85C0               test eax,eax  
  0043B364     75 22              jnz short NetTrans.0043B388  
  0043B366     57                 push edi  
  0043B367     56                 push esi                          ; key 的名称  
  0043B368     68 E4524F00        push NetTrans.004F52E4            ; UNICODE "NT2REG"  
  0043B36D     8D4C24 30          lea ecx,dword ptr ss:\[esp+30\]  
  0043B371     E8 BA210800        call NetTrans.004BD530  
  0043B376     50                 push eax  
  0043B377     8D4C24 40          lea ecx,dword ptr ss:\[esp+40\]  
    
    
  0043B3B5     8B41 F8            mov eax,dword ptr ds:\[ecx-8\]  
  0043B3B8     85C0               test eax,eax  
  0043B3BA     0F85 AA000000      jnz NetTrans.0043B46A  
  0043B3C0     8D9424 8C000000    lea edx,dword ptr ss:\[esp+8C\]  
  0043B3C7     68 E4524F00        push NetTrans.004F52E4            ; UNICODE "NT2REG"  
  0043B3CC     8D8424 94000000    lea eax,dword ptr ss:\[esp+94\]  
  0043B3D3     52                 push edx  
  0043B3D4     50                 push eax  
  0043B3D5     E8 BEB90800        call <jmp.&MFC42u.#925>  
  0043B3DA     8D8C24 90000000    lea ecx,dword ptr ss:\[esp+90\]  
  0043B3E1     68 00534F00        push NetTrans.004F5300            ; UNICODE ".key.kqm"  
  0043B3E6     8D5424 20          lea edx,dword ptr ss:\[esp+20\]     ; 注册表备份  
  0043B3EA     51                 push ecx  
  0043B3EB     52                 push edx  
  0043B3EC     C68424 90000000 08 mov byte ptr ss:\[esp+90\],8  
  0043B3F4     E8 9FB90800        call <jmp.&MFC42u.#925>  
  0043B3F9     8B18               mov ebx,dword ptr ds:\[eax\]  
  0043B3FB     8D8424 90000000    lea eax,dword ptr ss:\[esp+90\]     ; 前提是要注册成功了才有  
  0043B402     68 F4524F00        push NetTrans.004F52F4            ; UNICODE ".key"  
  0043B407     8D4C24 24          lea ecx,dword ptr ss:\[esp+24\]  
  0043B40B     50                 push eax  
    
    
  看过上面 3 小段大家就应该知道，这里是关键  
    
  提示：看上面看似关键的跳转不是受\[ecx-8\]就是受\[edx-8\]的控制，似乎是有关联的\[是与不是分析便知\]  
    
  二、借助 MFC42u.DLL 注入  
    
  接下来就是慢慢分析了  
    
  0043B127     E8 B0BD0800        call <jmp.&MFC42u.#2606>          ; F7  
    
  004C6EDC   - FF25 88524D00      jmp dword ptr ds:\[<&MFC42u.#2606>>; MFC42u.#2606，F8  
    
    
  727A49E4 M>  8B01               mov eax,dword ptr ds:\[ecx\]        ; 来到这里，发现正处于 MFC42u.DLL 的领空  
  727A49E6     8378 F8 00         cmp dword ptr ds:\[eax-8\],0        ; ds:\[728776CC\]=00000000  
  727A49EA     74 15              je short MFC42u.727A4A01          ; 好咯，看到上面就应该灵机一动了  
  727A49EC     8378 F4 00         cmp dword ptr ds:\[eax-C\],0        ; 到 system32 目录下找到 MFC42u.DLL  
  727A49F0     7C 05              jl short MFC42u.727A49F7          ; 把它拷贝到程序目录  
  727A49F2   ^ E9 B5FFFFFF        jmp MFC42u.#5509                  ; 关掉 OD，重新载入，以便于程序调用的是本目录下的 MFC42u.DLL  
  727A49F7     68 F4D68772        push offset MFC42u.#6408  
  727A49FC     E8 91FDFFFF        call MFC42u.#861  
  727A4A01     C3                 retn  
    
  拷贝 MFC42u.DLL 到程序目录后，OD 载入程序，F9 跑一下，Alt+E，看看程序调用的是否是本目录下的 MFC42u.DLL  
    
  OK，一切准备妥当，Patch 这个 MFC42u.DLL 吧，Patch 它的目的是为了使\[ecx-8\]=\[728776CC\]=1，\[728776C8\]=00000001，在 MFC42u.DLL 找块空白代码  
  我这里找的是 72843C71（找不会被重定位的空白）  
    
  727A49E4 M>  8B01               mov eax,dword ptr ds:\[ecx\]  
  727A49E6     E9 86F20900        jmp MFC42u.72843C71               ; 跳向 Patch 位置  
  727A49EB     90                 nop  
  727A49EC     8378 F4 00         cmp dword ptr ds:\[eax-C\],0        ; \[eax-0C\]也必须=1  
  727A49F0     7C 05              jl short MFC42u.727A49F7  
  727A49F2   ^ E9 B5FFFFFF        jmp MFC42u.#5509  
  727A49F7     68 F4D68772        push offset MFC42u.#6408  
  727A49FC     E8 91FDFFFF        call MFC42u.#861  
  727A4A01     C3                 retn  
    
    
  72843C71     C740 F8 01000000   mov dword ptr ds:\[eax-8\],1        ; 先赋值 1  
  72843C78     8378 F8 00         cmp dword ptr ds:\[eax-8\],0        ; 原代码挪下  
  72843C7C   ^ 0F84 7F0DF6FF      je MFC42u.727A4A01                ; 原代码挪下  
  72843C82     C740 F4 01000000   mov dword ptr ds:\[eax-C\],1        ; 赋值 1  
  72843C89   ^ E9 5D0DF6FF        jmp MFC42u.727A49EB               ; 跳回原代码执行流程  
    
    
  C7 40 F8 01 00 00 00 83 78 F8 00 0F 84 7F 0D F6 FF C7 40 F4 01 00 00 00 E9 5D 0D F6 FF  
    
  复制修改--》覆盖本目录下的 MFC42u.DLL  
    
  再运行 NetTransport 看看，启动时的注册框没了，关于里面显示：注册给，因为注册表里没任何信息嘛，关于里的注册钮也变成灰色了  
  经过测试\[挂了几个小时\]，也没弹出那讨厌的注册框了：0  工作完成，谢谢观赏。  
    
    
    
\--------------------------------------------------------------------------------------  
【经验总结】  
  1、喜欢外挂的朋友可以把这个思路转移到破解外挂中  
  2、文中分析不够，这是体力活也是基本功，本问旨在介绍思路  
  3、个人愚见，Dll 注入大致可分为：现成的 Dll 注入\[系统 Dll 和程序自带的 Dll\]和自己写的 Dll  
  4、呵呵，不知道这算不算是注入了，瞎写啦：）  
\--------------------------------------------------------------------------------------  
【版权声明】: 本文原创于看雪技术论坛, 转载请注明作者并保持文章的完整, 谢谢!  
  
                                                       2006 年 10 月 31 日  
  
The download link is: http://www.sendspace.com/file/lmherq  
  
  
Patch 了的 MFC42u.dll 也在包内:0,有这个 DLL 在就不需要注册了，如果想看看未注册的效果，就把 MFC42u.dll 改成其他名字  
  
   
  
如有兴趣你可继续分析把名字 patch 进去~ 

【详细过程】  
  写在前面：看了 wynney 大大的《Dll 注入 NetTransport 2.25.337 》一文，觉得深受启发！不过 wynney 在定位注册框的时候写得比较简略，只是说“F7 配合 F8 很容易就找到掉用注册框的地方”。我问他能不能详细说说这个过程，wynney 说这个是基本功！其实，OD 有一个自动跟踪的功能，用这个功能在定位注册框、NAG 等对话框时非常有用和简便。只是不知为什么，我见到现在很少有人提到这个功能，于是便有了本文，说得不对的地方还请各位指正！  
  OD 加载 NetTransport 2.25.337 后，CTRL+F8 ;让 OD 自动跟踪！  
  7C935C93    5E              pop     esi  
  7C935C94    F7FE            idiv    esi  
  7C935C96    85D2            test    edx, edx     ;稍等一会后 OD 停在这里！  
  7C935C98    75 32           jnz     short 7C935CCC  
  7C935C9A    66:8B71 02      mov     si, \[ecx+2\]  
  7C935C9E    53              push    ebx  
    
  这时由于主程序（NetTransport）还没有运行起来，故继续 CTRL+F8 第二次：  
  00443EA7   .  6A 00         push    0                                         ; /Arg2 = 00000000  
  00443EA9   .  50            push    eax                                       ; |Arg1  
  00443EAA   .  8D8C24 A40000>lea     ecx, \[esp+A4\]                             ; |  
  00443EB1   .  C68424 0C1000>mov     byte ptr \[esp+100C\], 0A                   ; |  
  00443EB9   .  E8 4272FFFF   call    0043B100                           \\NetTrans.0043B100  
  00443EBE   .  8B4C24 18     mov     ecx, \[esp+18\]  
  00443EC2   .  8B41 F8       mov     eax, \[ecx-8\]     ;\[ecx-8\]=\[728776CC\]，记住他  
  00443EC5   .  85C0          test    eax, eax  
  00443EC7   .  75 78         jnz     short 00443F41    ;看看/跳过的话那不就不用注册了？  
  00443EC9   .  6A 00         push    0  
  00443ECB   .  8D8C24 500100>lea     ecx, \[esp+150\]  
  00443ED2   .  E8 79860200   call    0046C550  
  00443ED7   .  8D8C24 4C0100>lea     ecx, \[esp+14C\]  
  00443EDE   .  C68424 041000>mov     byte ptr \[esp+1004\], 0B  
  00443EE6   .  E8 F72C0800   call    <jmp.&MFC42u.#2506>     ; 稍等一会后 OD 停在这里！  
  00443EEB   .  3D 55050000   cmp     eax, 555  
  00443EF0   .  75 13         jnz     short 00443F05  
  00443EF2   .  8D5424 18     lea     edx, \[esp+18\]  
    
  这时留意到主程序已经运行了（注册对话框已经弹出了）!  
  故 00443EE6   .  E8 F72C0800   call    <jmp.&MFC42u.#2506> 这个 call 就是调用注册对话框的 call!  
  我们发现\[ecx-8\]=\[728776CC\]=1 才可以跳过去这个 CALL!在此之后，wynney 是通过追踪关键 Call 的方法来找到对程序打补丁的地方的。我认为不用这样大费周折的，只要直接找到程序哪里对\[728776CC\]进行赋值的不就可以了！为了找到程序对\[728776CC\]进行赋值的地方，于是 CTRL＋F2 重新加载程序，然后对内存地址\[728776CC\]下硬件字节写入断点！F9 运行  
  727A47E0    8B06            mov     eax, \[esi\]  
  727A47E2    83C4 0C         add     esp, 0C  
  727A47E5    8978 F8         mov     \[eax-8\], edi        ;这里对内存地址\[728776CC\]进行赋值！  
  727A47E8    8B06            mov     eax, \[esi\]             ;OD 即时中断在这里！                     
  727A47EA    66:832403 00    and     word ptr \[ebx+eax\], 0  
    
  于是我们把 727A47E5    8978 F8         mov     \[eax-8\], edi   改为  jmp 72843C71  ;跳向 Patch 位置  
    
  72843C71    C640 F8 01      mov     byte ptr \[728776CC\], 1    ;对内存地址\[728776CC\]进行赋值 1！  
  72843C75    8B06            mov     eax, \[esi\]              ;原代码挪下  
  72843C77  ^ E9 6E0BF6FF     jmp     727A47EA               ;跳回原代码执行流程  
    
  复制修改--》覆盖本目录下的 MFC42u.DLL  
      
    再运行 NetTransport 看看，启动时的注册框没了，关于里面显示：注册给，因为注册表里没任何信息嘛，关于里的注册钮也变成灰色了  
    经过测试\[挂了几个小时\]，也没弹出那讨厌的注册框了：0  工作完成，谢谢观赏。(这句话是抄 wynney 的！）  
  写在后面：本文旨在提供另一种破解思路，欢迎指正！

 4dd466d0e1e0cc123afbe504da2462ca