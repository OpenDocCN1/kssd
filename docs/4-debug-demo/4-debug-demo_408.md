

# R-Studio Demo 

> 原文：[`www.pediy.com/kssd/pediy05/pediy50223.htm`](https://www.pediy.com/kssd/pediy05/pediy50223.htm)

这是个多系统文件恢复程序，使用很简单，演示版有 64K 文件大小恢复限制   
调试环境：win98,我的机器没有安装 2000，或者 XP，请自己参考测试   
使用工具，W32DASM，TRW2000，hiew.   
限制分析：   
    这个工具在被恢复文件小于 64K 时，正常运行，一旦文件大于 64K 就会跳出一个演示版本的限制窗口，而且程序不进行任何操作。那我们就开始对它分析吧，先检测看看，哦，它没有加壳，那就不客气了，把它的主程序反汇编好了，看看，奇怪，任何对我有用的参考都没有，那就进行动态跟踪好了，用 TRW 进行跟踪，跳出窗口，下 PMOUDLE，哦，跳出来的地方不是程序的所在，那就按 F12 吧，好了，程序终于被拦截到了，看看程序所在的域，是一个 rs-an 的地方，奇怪了，我记得主程序不是这个名字，好象也没一个这样的 DLL 文件呀，退出来找找看，哦，有一个 bin 文件，呵呵，把它改个名字，呵呵，W32DASM 终于发挥威力了，好爽，下面就从提示入手吧   
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:0041DBF8(U)   
|   
:0041DC0B 8B542420                mov edx, dword ptr \[esp+20\]   
:0041DC0F 8B02                    mov eax, dword ptr \[edx\]   
:0041DC11 50                      push eax   
:0041DC12 8B8424E4010000          mov eax, dword ptr \[esp+000001E4\]   
:0041DC19 6A40                    push 00000040   
:0041DC1B 50                      push eax   
:0041DC1C 55                      push ebp   
:0041DC1D E89E520800              call 004A2EC0   
:0041DC22 83C404                  add esp, 00000004   
:0041DC25 50                      push eax   
  
\* Possible Reference to String Resource ID=41500: "%s limitation!   
Size of file %s  exceed %u KB limit for %s"--->看到提示了吗？提示 A   
                                  |   
:0041DC26 681CA20000              push 0000A21C   
:0041DC2B E820620B00              call 004D3E50   
再来看看另一个提示在哪里: )   
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:004C9395(C)   
|   
:004C93A2 6A00                    push 00000000   
:004C93A4 E8179BFDFF              call 004A2EC0   
:004C93A9 83C404                  add esp, 00000004   
:004C93AC 50                      push eax   
:004C93AD 6A40                    push 00000040   
  
\* Possible Reference to String Resource ID=47363: "Data size is greater than %d KB limit for %s."--->又一个提示 提示 B   
                                  |   
:004C93AF 6803B90000              push 0000B903   
:004C93B4 E897AA0000              call 004D3E50   
闲着也是闲着,往上看看谁在调用它,呵呵,又看到一个提示了   
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:004C9305(C)   
|   
  
\* Possible Reference to String Resource ID=47362: " failed. "   
                                  |   
:004C931A 6802B90000              push 0000B902   
:004C931F E82CAB0000              call 004D3E50   
呵呵,上面还有   
\* Referenced by a CALL at Addresses:   
|:004C9CC9  , :004C9FFD  , :004CA0DD  , :004CA3B0  , :004CA490    
|:004CA5FE  , :004CA65F --->这里表明这里好多过程喜欢它    
|   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:   
|:00529756(U), :00529776(U), :00529796(U), :005297E1(U)   
|   
:004C9280 55                      push ebp   
:004C9281 8BE9                    mov ebp, ecx   
:004C9283 56                      push esi   
:004C9284 57                      push edi   
:004C9285 8A4509                  mov al, byte ptr \[ebp+09\]   
:004C9288 84C0                    test al, al   
:004C928A 0F84B5020000            je 004C9545   
:004C9290 8B7500                  mov esi, dword ptr \[ebp+00\]--->曾在这里做文章,结果发现不管下面显示什么,小文件在这里就已经恢复好了,看来是先恢复后显示消息,那就往前看吧   
:004C9293 85F6                    test esi, esi   
:004C9295 7564                    jne 004C92FB   
:004C9297 8A4508                  mov al, byte ptr \[ebp+08\]   
:004C929A 84C0                    test al, al   
:004C929C 0F84A3020000            je 004C9545   
  
\* Possible Reference to String Resource ID=47361: " OK."   
                                  |   
:004C92A2 6801B90000              push 0000B901   
:004C92A7 E8A4AB0000              call 004D3E50   
通过下断实践,只有 4C9CC9 会在执行恢复时被中断,那就去看看它好了,哦,这段程序我怎么才能知道我得从哪里开始跟踪呢,往上翻翻,呵呵 LOOK!   
:004C9948 50                      push eax   
  
\* Possible Reference to String Resource ID=47367: "Restoring file %s"--->好象正好是我想要的消息 : )   
                                  |   
:004C9949 6807B90000              push 0000B907   
:004C994E E8FDA40000              call 004D3E50--->就在这里下断开始跟踪吧 : ),反复的跟踪证明,程序在不声不响的情况下做了它愿意做的事情,滑头 : ) 呵呵.不过接下来的东西,开始有点意思了   
:004C999B 817D2800000100          cmp dword ptr \[ebp+28\], 00010000--->让我猜猜,你喜欢这句吗?   
:004C99A2 7218                    jb 004C99BC   
一路无话,来到这里   
\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:   
|:004C99DF(C), :004C99E7(C)   
|   
:004C9AA0 8B4D0C                  mov ecx, dword ptr \[ebp+0C\]   
:004C9AA3 8BC5                    mov eax, ebp   
:004C9AA5 F7D8                    neg eax   
:004C9AA7 1BC0                    sbb eax, eax   
:004C9AA9 80E102                  and cl, 02   
:004C9AAC 23C6                    and eax, esi   
:004C9AAE 8B5520                  mov edx, dword ptr \[ebp+20\]   
:004C9AB1 F6D9                    neg cl   
:004C9AB3 1BC9                    sbb ecx, ecx   
:004C9AB5 F7D9                    neg ecx   
:004C9AB7 41                      inc ecx   
:004C9AB8 51                      push ecx   
:004C9AB9 52                      push edx   
:004C9ABA 50                      push eax   
:004C9ABB E8301B0000              call 004CB5F0--->大量的事实证明,这个地方是个很关键的地方,当文件小于 64K 时,带过这里,就会产生一个恢复好的文件,那我们怎么能不进去看看呢 : ),走,走,走,来到这里   
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:004CB659(C)   
|   
:004CB668 6800100000              push 00001000   
:004CB66D 6800000100              push 00010000--->猜猜看什么意思 : )   
:004CB672 8D4C241C                lea ecx, dword ptr \[esp+1C\]   
:004CB676 E8B58CFEFF              call 004B4330--->这个 CALL 是检查你的心有多黑的地方，太黑就不好了 : )   
:004CB67B 8B442414                mov eax, dword ptr \[esp+14\]   
:004CB67F C644242801              mov \[esp+28\], 01   
:004CB684 3BC3                    cmp eax, ebx   
:004CB686 0F840E020000            je 004CB89A   
:004CB68C 817C241800000100        cmp dword ptr \[esp+18\], 00010000   
:004CB694 0F8500020000            jne 004CB89A   
:004CB69A 8B17                    mov edx, dword ptr \[edi\]   
:004CB69C 8BCF                    mov ecx, edi   
:004CB69E FF5210                  call \[edx+10\]--->EAX 中返回待恢复文件的长度   
:004CB6A1 F6450401                test \[ebp+04\], 01   
:004CB6A5 7546                    jne 004CB6ED   
:004CB6A7 3BD3                    cmp edx, ebx   
:004CB6A9 0F8CD5000000            jl 004CB784   
:004CB6AF 7F07                    jg 004CB6B8   
:004CB6B1 3D00000100              cmp eax, 00010000--->比较长度 : )   
:004CB6B6 7635                    jbe 004CB6ED   
还有这里   
:004CB6ED 3BD3                    cmp edx, ebx   
:004CB6EF 0F8C8F000000            jl 004CB784   
:004CB6F5 7F08                    jg 004CB6FF   
:004CB6F7 3BC3                    cmp eax, ebx   
:004CB6F9 0F8685000000            jbe 004CB784   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:004CB6F5(C)   
|   
:004CB6FF 385D10                  cmp byte ptr \[ebp+10\], bl   
:004CB702 0F85BE000000            jne 004CB7C6   
:004CB708 8B4C2418                mov ecx, dword ptr \[esp+18\]--->是什么东西搬到了 ECX 里呢?还记得那个 PUSH 吗?都是老朋友 : )   
:004CB70C 3BD3                    cmp edx, ebx   
:004CB70E 7F0A                    jg 004CB71A   
:004CB710 7C04                    jl 004CB716   
:004CB712 3BC1                    cmp eax, ecx\--->再亲热一下: )   
:004CB714 7304                    jnb 004CB71A   
想想看,你想把那个数值改多大,看看你的心有多黑,我嘛,只 100 兆,还好啦: ),下面就 OK 了: )   
  
  
hello!    2002,07,10  4dd466d0e1e0cc123afbe504da2462ca