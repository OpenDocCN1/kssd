

# KERNEL32.DLL

> 原文：[`www.pediy.com/kssd/pediy02/forum280.htm`](https://www.pediy.com/kssd/pediy02/forum280.htm)

 原标题：   
Cracking using KERNEL32.DLL??   
Amazing how things don't change!   
  
原文：http://208.50.16.104/madmasu.htm   
  
作者： madmax    翻译  DDXia\[CCG\]   
  
  
丁丁虾有话说：（那还用问，肯定有是废话啦^\_\*)===如果你是高手，请使用暴力，JMP 过去^\_^   
  
      Crack 也有段时间了，老是在想，一天到晚的 Crack，（这小子开始吹牛，踞我所知，已有   
3 个月没有发市啦:) 什么嘛！我可是天天都到看雪论坛报到的哦）有没有好的办法能提高自己的   
功力？很多人都关心。毕竟外国的 CrAcK 发展比较快，所以经常去外国网站，洋为中用嘛？   
今天无意中看到一篇文章，有比较大的启发。好的东东，大家分享嘛！！！顺便说一声，英语不   
是俺的母语（好象 tKC 也说过），也比较菜，翻译内容大概的意思应该都对吧！哈哈~~~~ :)   
  
英文的原文：http://208.50.16.104/madmasu.htm   
  
标题：使用 KERNEL32.DLL 破解？？？   
  
  
介绍：   
  
在 Internet 上有一个非常出色的游戏。。。Subspace，by VIE(Virgin)，(DDXia：好象没有听过)   
最近刚拿到它的零售版，。。。。 零售版程序中含有 CRC 检查和内存驻留（时时检查）（DDXia：好怕怕哦）   
我们应该如何 CrAck？？？   
  
  
工具：   
Winice 3.22   
Hiew 5.66   
W32Dasm 8.9 (Regged helps)   
API 知识 (有 Win32.Hlp 就行)   
  
  
游戏的 URL/FTP    
http://subspace.vie.com/   
  
  
教学   
SubSpace，是一个已经发布了一段时间的在线游戏。在玩游戏过程很难作弊。。。这就是运行它的乐趣，   
但不会玩这个游戏。现在游戏发行了零售版，在其中加入了 CD-Check。。。啊哦！！来，坐下来开始 CRACK，   
如果你有零售版，你会发现游戏提示"请插入 CD...."，并带有时间限制....哈哈~~~~~~....但有点我懂啦....   
于是进入 Winice。（DDXia：真是聪明！！）   
BPX GetDriveTypeA （DDXia：一般，寻找 CDROM 都是使用这个 API）   
  
  
捕获、进入 SubSpace 的代码中....这个程序的 CDCHEK 步骤如下：（DDXia：重点之一，具有普遍性）   
  
查 CDROM 的驱动器号-----------------------------GetDriveTypeA   
获取 CD 的空间--------------------------------------GetDiskSpaceFreeA   
如果等于零，那肯定是 CD。   
获取 CD 的标签 -------------------------------------GetVolumeInformationA   
如果标签=='SUBSPACE' 那么 CD OK   
打开 CD 根目录的'INTRO.FLC'。 ---------------CreateFileA   
如果存在， 那么 CD OK   
获取 INTRO.FLC 的大小---------------------------GetFileSize?   
比较它的大小   
如果相等，就设一个标志。   
  
既然大多数人读了这篇文章也不会去 CRACK 它，所以对原代码不会在这里贴出来。但最有趣的部分，也就是关键，   
是如何绕过检测部分。我们可以通过 HIEW 来修改跳过，但有 CRC 校验检测会中断程序，那么再补丁 CRC 检测，   
可是内存中的还有动态校验....真的很难 debug，在 Winice 中需要修改的断点就有 0CCh（DDXia:换算过来就是 204 处,   
哇！！我倒。。。。。。）.....该怎么办呢？？恩，考虑一下 API Hooking，看来没有什么希望。再想一想，   
.......TSR Crack！但是这可不是 DOS 环境啊！在 Window 中没有 AX=25XX, INT 21h 功能，那么哪里是突破口呢？   
.......YEAH ！Windows 的本身。   
  
有了这个想法，于是到 IRC 中发表（DDXia：外国的 Cracker 非常喜欢到 IRC 中交流），但是这个想法就好象   
自己准备去修理一颗核弹，和内核玩简直就是发疯了（DDXia：呵呵~~~~~我还是非常欣赏他的冒险精神和创   
造力）我们知道在 SubSpace 保护程序中，第一次调用的 API 是 GetDriveTypeA，这个 API 返回一个数值，   
2 代表软驱，3 代表硬盘，5 代表 CDROM，（至于其他的数值，我们不关心）.....实际上，我们要象 FAKECD   
程序那样攻击它的保护....当用 TSR 模拟一个 CD 的时候，我们在硬盘中模拟那些必需的文件。   
  
  
在 Winice 中，输入：   
U GetDriveTypeA    （DDXia：俺孤漏寡闻，可是第一听说这样用 U 命令啊！！！牢记在心！哈哈~~~~，   
        又学到一招）   
将看到 API 被反汇编过的代码......下面的代码是其中的一部分，也可以静态反汇编 kernel32.dll。但你看到的程序，   
会有些莫名其妙的比较后，使用 JMP 跳转到其他的 API：   
（DDXia：也许代码部分会有不一样，那是因为 kernel32.dll 版本不同）   
  
:BFF777C4 57                      push edi   
:BFF777C5 6A21                    push 00000021   
:BFF777C7 2BD2                    sub edx, edx   
:BFF777C9 68EFE2F9BF              push BFF9E2EF   
:BFF777CE 64FF32                  push dword ptr fs:\[edx\]   
:BFF777D1 648922                  mov dword ptr fs:\[edx\], esp   
:BFF777D4 8B7C2414                mov edi, dword ptr \[esp+14\]   
:BFF777D8 0BFF                    or edi, edi   
:BFF777DA 7407                    je BFF777E3   
:BFF777DC 2BC0                    sub eax, eax   
:BFF777DE 8D48FF                  lea ecx, dword ptr \[eax\-01\]   
:BFF777E1 F2                      repnz   
:BFF777E2 AE                      scasb   
  
\* Referenced by a Jump at Address:BFF777DA(C)   
|   
:BFF777E3 648F02                  pop dword ptr fs:\[edx\]   
:BFF777E6 83C408                  add esp, 00000008   
:BFF777E9 5F                      pop edi   
:BFF777EA E9E5D4FFFF              jmp BFF74CD4   
  
  
在这个程序中，我们如何放入自己的代码呢？！？在 JE 处设 BPX，结果是它会不断被中断。所以   
BPX BFF777DA，然后运行 Explorer 或者某些程序就会被 BPX 中断。你将看到什么都不做，也会中断！   
所以我们敢肯定（这主要因为是微软写的代码）这段程序有点垃圾......让我们修改和优化它：   
（游戏中第一次调用是检查 C：）   
  
:BFF777C4 57                      xor  edx,edx      ;  标志位   
:BFF777C5 6A21                    mov  eax,\[esp\]    ;  得到正在调用的程序的地址   
:BFF777C7 2BD2                    cmp  eax,413e6e  ; 是否是游戏的 CD-CHECK   
:BFF777C9 68EFE2F9BF              jnz  BFF777EA    ; 如果不是，继续   
:BFF777CE 64FF32                  mov  \[esp+4\],eax  ;  保存返回地址   
:BFF777D1 648922                  pop  eax          ;        ESP+4   
:BFF777D4 8B7C2414                mov  eax,5        ; 返回 5，表示我们是 CDROM   
:BFF777D8 0BFF                    ret   
:BFF777DA 7407                    je BFF777E3      ; 微软代码   
:BFF777DC 2BC0                    sub eax, eax      ; 这里！   
:BFF777DE 8D48FF                  lea ecx, dword ptr \[eax\-01\]   
:BFF777E1 F2                      repnz   
:BFF777E2 AE                      scasb   
  
\* Referenced by a Jump at Address:BFF777DA(C)   
|   
:BFF777E3 648F02                  pop dword ptr fs:\[edx\]   
:BFF777E6 83C408                  add esp, 00000008   
:BFF777E9 5F                      pop edi   
:BFF777EA E9E5D4FFFF              jmp BFF74CD4   
  
你能看到，我们非常暴力的截取了这个 API........每一次调用，我们都去 检查是否是被游戏调用.......非常简单！   
现在我们欺骗它 C:就是 CDROM，我们再到磁盘剩余空间的检查：   
（原程序和上面的很相相似，但也是很多垃圾....谁能解决？因为这段程序是大了点.....下面是新的改良过的   
程序！）   
（DDXia：不过在 NT 中就没有怎么多的程序废话了，比较紧凑，WIN98 没有看过是如何的？）   
  
:BFF778C5 2BD2                    xor  edx,edx      ; 好象所有的微软都喜欢这代码呢？   
:BFF778C7 68B8E2F9BF              mov  eax,\[esp\]    ;  得到正在调用的程序地址   
:BFF778CC 64FF32                  cmp  eax,491E22  ;是否是 SubSpace?   
:BFF778CF 648922                  jnz  BFF77900    ;如果不是，继续   
:BFF778D2 8B4C240C                add  esp,14h      ;  修改堆栈，保证返回正常   
:BFF778D6 E302                    mov  \[esp\],eax    ; 返回指针   
:BFF778D8 8A01                    mov  \[esp+18\],eax ; 这里是空间大小返回。（DDXia：我觉应该是 edx，   
                                                                                    而不是 eax，也许是笔误）   
:BFF778DA 8B4C2410                ret              ; 返回   
:BFF778DE E302                    jcxz BFF778EA    ; 剩下的代码不变   
:BFF778E0 8B01                    mov eax, dword ptr \[ecx\]   
:BFF778E2 8B4C2414                mov ecx, dword ptr \[esp+14\]   
:BFF778E6 E302                    jcxz BFF778EA   
:BFF778E8 8B01                    mov eax, dword ptr \[ecx\]   
:BFF778EA 8B4C2418                mov ecx, dword ptr \[esp+18\]   
:BFF778EE E302                    jcxz BFF778F2   
:BFF778F0 8B01                    mov eax, dword ptr \[ecx\]   
:BFF778F2 8B4C241C                mov ecx, dword ptr \[esp+1C\]   
:BFF778F6 E302                    jcxz BFF778FA   
:BFF778F8 8B01                    mov eax, dword ptr \[ecx\]   
:BFF778FA 648F02                  pop dword ptr fs:\[edx\]   
:BFF778FD 83C404                  add esp, 00000004   
:BFF77900 E954D5FFFF              jmp BFF74E59   
  
  
这样看来，我们截获这段程序，同时强迫它返回磁盘剩余空间为零，C:象个 CDROM.......现在剩下的就是标签检   
查和文件检查.......这样简单，修改 C：的标签为 SUBSPACE，和还使用一小段 ASM 编的程序，我们编个大小为   
37,433,486 的文件..然后修改 KERNEL32.DLL，重新启动。不停的画十字架吧，企求不要系统崩溃吧！！   
  
后语：   
这是非常有趣的 CRACK 的一种方法.....不过我从来没有看见任何一个人使用这种方法，也许是因为它太   
冒险，同时还有一个兼容性的问题。我主要是想交流我发现的想法，（DDXia：YEAH！Crack 的精神   
所在，不断的探索和传播）也许在未来会考虑这种技术.....这主要的问题是修改了 KERNEL32.DLL，因为   
它有许多的版本。如果 Patch 所有的版本那简直就是发疯了。   
  
非常惊奇的是无论 DOS 走多远， its roots still follow us in cracking!（DDXia：这句太难翻译了）   
  
丁丁虾再说几句：（暴力法-----Crack 第一式，牢记 EB----JMP，跳过、跳过、再跳过）   
    的确,有些疯狂，但他思考的方式和他的自创力都是值得学习！翻译的目的是希望大家能比较深层   
的思考问题。   
                  （丁丁虾长长地、长长地吐了一口气）终于完成翻译了！请多多指教！拜托啦！好久好久都没有   
写过什么东东了！就当给大家的一份小小小的礼吧！！！现在凌晨 1：00 还差 10 分钟。该收工了！   
  
                              DDXia\[CCG\]   
                            2000.8.29  01:50  

 标题: 任何 patch 都必须基于对程序的正确理解，显然这个作者并未对 kernel32.dll 中被改动的部分有透彻的了解，只是赌了一把而已   
  
内容:  

 4dd466d0e1e0cc123afbe504da2462ca