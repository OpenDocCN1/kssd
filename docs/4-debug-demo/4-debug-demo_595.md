

# 倚天 II 账号密码的破解

> 原文：[`www.pediy.com/kssd/pediy06/pediy6838.htm`](https://www.pediy.com/kssd/pediy06/pediy6838.htm)

【破解作者】 shoooo  
【作者邮箱】 shoooo314@163.com  
【使用工具】 softice 套装 2.7 iceext 插件，Iris  
【破解平台】 win2000 sp4   
【破解目标】 账号，密码  
【软件名称】 倚天 II (公测中)  
【破解时间】 2004-12-20  
【加壳方式】 无壳  
【破解声明】 我是一只小菜鸟，偶得一点心得，愿与大家分享：）  
             本文旨在讨论网络游戏的破解思路，错误或不妥之处，各位批评指正  
             文章中的代码粘贴自 IDA，我 softice 不会 dump 汇编代码  
             由于更新频繁或其它原因，内存地址可能存在出入  
  
一，寻找登录包  
  
    输入错误的用户名或密码，让 TCP 流尽早结束，这样交流的数据包较少，容易找  
  
    截包工具有很多，看个人喜好了~~我 Iris 用惯了  
    启动倚天 II，出现登录界面，随便找个服务器，以用户名 shoooo 密码 1234 登录，报错。看看截的包包，在登录过程中，含有数据的共有 9  
  
个包，包中的数据都是无规则的 16 进制，可以肯定是经过加密的。很容易发现前面 7 个包长度较短，内容近似。  
    第 8 个包是客户端发给服务器，长度有 56 字节如下  
                C8 9E 23 AC BA 93 FD 12   05 72 11 85 BF D0 E7 BB  
                D1 C0 A9 36 35 AA 5B 0B   05 72 11 85 BF D0 E7 BB  
                F3 9C 41 BD C1 62 6B 7A   22 E5 37 46 B7 13 1E 00  
                B2 BC FA 80 11 B6 2D 82   
    第 9 个包从服务器发来给客户端，长度为 16 字节如下  
                0E 6B 32 F1 40 39 80 69   05 72 11 85 BF D0 E7 BB   
  
    猜测第 8 个包是登录包，第 9 个登录包是登录响应包。  
  
    用相同的账号再登录一次  
    重点观察第 8 个包如下，并发现前 32 字节与前次登录中的数据相同  
                C8 9E 23 AC BA 93 FD 12   05 72 11 85 BF D0 E7 BB  
                D1 C0 A9 36 35 AA 5B 0B   05 72 11 85 BF D0 E7 BB  
                5C 7C 2A 03 B6 CF D3 39   FE 04 3D 80 C8 B0 E3 25  
                5A F8 EA 9E 62 2A 13 87   
      
    重点观察第 9 个包如下，与前次完全相同  
                0E 6B 32 F1 40 39 80 69   05 72 11 85 BF D0 E7 BB   
  
    省略 n 次的相同或不同账号密码组合的登录试验......  
  
    这时我们得到如下结论  
    a. 登录包长度固定为 56 字节，若以相同的账号，密码登录，登录包的前 32 字节相同。登录的错误响应包数据相同，固定 16 字节  
    b. 56，32，16 都是 8 的倍数，加密算法极有可能是 8 字节的分组算法  
  
  
二，寻找发送 buffer  
  
    可以从输入的账号，密码开始跟，也可以从发送的数据包反向跟，我个人倾向于后者，这里采用后者的方法  
  
    请出 softice，symbol loader 加载 ws2\_32.dll，倚天 2 有防 softice，用上 iceext 插件顺利躲过  
    启动客户端至登录界面  
    ctrl-d 呼出  
    确认右下角显示为倚天 II 的领空 metein2(以下不再提示)，如果不是可多试几次或用 addr 命令  
  
    下断点 bpx send  
    下断点 bpx wsasend  
    随便选个服务器，输用户名 shoooo，密码 1234。点登录  
    softice 在 send 处断下，按 F11 回到调用 send 的地方，看 EAX 的值，EAX 是调用 send 后的返回值也就是实际发送的字节长度，我们想要看到的  
  
是 56 字节，即十六进制的 38。现在不是，ctrl-d 让它继续跑，马上又在 send 处断下，F11，看 EAX 的值，不是就 ctrl-d 再继续  
    约 5 到 6 次后，EAX 的值显示为 38，立即对准调用 send 的 EIP:499722 下断点，同时清除 send,wsasend 断点，按若干次 ctrl-d，让登录过程结束  
  
                 .text:00499716                 push    0               ; flags  
                 .text:00499718                 push    eax             ; len  
                 .text:00499719                 push    dword ptr \[esi+24h\] ; buf  
                 .text:0049971C                 push    dword ptr \[esi+244h\] ; s  
                 .text:00499722                 call    send            此处下断点  
                 .text:00499727                 mov     edi, eax  
                 .text:00499729                 test    edi, edi  
  
    再次点击登录，这次在 EIP:499722 处断下  
    下命令 d esi+24  
                 B8 07 EE 01 08 08 00 00  .......          
    下命令 d 1ee07b8  
                 C8 9E 23 AC BA 93 FD 12  05 72 11 85 BF D0 E7 BB   
                 D1 C0 A9 36 35 AA 5B 0B  05 72 11 85 BF D0 E7 BB  
                 0D 15 41 28 73 52 5A 05  CF 87 39 DF 74 35 24 1C  
                 87 7B 41 D6 18 21 BA B3  
    这便是实际发送的登录包的数据  
  
三，寻找加密过程  
  
    即寻找发送 buffer 的数据从何而来，不断的下内存断点，直到找到我们需要的账号，密码的明文      
  
    只保留 499722 的 eip 断点  
    下断点 bpmd 1ee07b8  
    以 shoooo,1234 再次登录  
    第一次断在 1ee07b8 的内存写入，先别急着看，ctrl-d 继续  
    第二次断在了 EIP:499722 处。  
    好了，到这里清楚了，第一次断下的地方便是对发送 buffer 的头 8 个字节的写入  
    再次登录，断在第一处  
  
                   .text:0048F30D                 mov     \[edx\], eax      这里 edx 值是 1ee07b8  
                   .text:0048F30F                 mov     \[edx+4\], ecx    断在此处  
                   .text:0048F312                 pop     ebx  
                   .text:0048F313                 pop     ebp  
                   .text:0048F314                 retn                    跳转到 48F3D0  
  
    取消 1ee07b8 的内存断点，F10 单步跟踪，48F314 的 retn 语句后，跳转到了 48F3D0  
                     
                   .text:0048F383                 push    ebp  
                   .....省略若干行  
                   .text:0048F3C0              /  push    \[ebp+arg\_0\]          
                   .text:0048F3C3              |  push    \[ebp+arg\_8\]  
                   .text:0048F3C6              |  push    dword ptr \[edi\]  
                   .text:0048F3C8              |  push    dword ptr \[edi+4\] 指向明文，待输出密文的指针入栈  
                   .text:0048F3CB              |  call    sub\_48F2AA        核心加密，8 字节明文加密，8 字节密文输出  
                   .text:0048F3D0              |  add     \[ebp+arg\_0\], 8    准备下次加密的 8 字节明文  
                   .text:0048F3D4              |  add     esp, 10h  
                   .text:0048F3D7              |  add     edi, 8            准备下次输出的 8 字节密文  
                   .text:0048F3DA              |  dec     \[ebp+arg\_C\]  
                   .text:0048F3DD              \\  jnz     short loc\_48F3C0  跳上去继续加密下一组  
                   .text:0048F3DF                 pop     edi  
                   .text:0048F3E0                 mov     eax, esi  
                   .text:0048F3E2                 pop     esi  
                   .text:0048F3E3                 pop     ebp  
                   .text:0048F3E4                 retn                     跳转到 499706  
  
    48F3E4 返回后，到了 499706  
  
                   .text:004996EF                 push    edi             需要加密的字节数  
                   .text:004996F0                 lea     ecx, \[esi+42h\]  
                   .text:004996F3                 push    ecx             指向密钥  
                   .text:004996F4                 mov     ecx, \[esi+30h\]  
                   .text:004996F7                 add     ecx, eax  
                   .text:004996F9                 mov     eax, \[esi+24h\]  
                   .text:004996FC                 add     eax, \[esi+2Ch\]  
                   .text:004996FF                 push    ecx             指向登录包的明文(包含账号，密码)  
                   .text:00499700                 push    eax             指向登录包的待输出的密文,这里是 1ee07b8  
                   .text:00499701                 call    sub\_48F383      最外层的加密函数  
                   .text:00499706                 add     \[esi+2Ch\], eax  
  
    此时可以清除先前的所有断点，在 EIP: 499071 处下断，跟踪整个加密过程，到这里不难发现登录包的明文必须是 8 字节的倍数，每 8 个字节  
  
调用 call 48F2AA 进行加密，输出 8 字节的密文  
  
四，加密算法求逆  
  
   既然服务器能对我们发送的加密登录包作出登录失败的反应，显然它是可逆的  
  
   重点观察 48F2AA 处的核心加密函数，分析出相应的解密算法  
      
                   .text:0048F2AA                 push    ebp  
                   .text:0048F2AB                 mov     ebp, esp  
                   .text:0048F2AD                 mov     eax, \[ebp+arg\_4\]  
                   .text:0048F2B0                 mov     ecx, \[ebp+arg\_0\]  
                   .text:0048F2B3                 mov     edx, \[ebp+arg\_8\]  
                   .text:0048F2B6                 push    ebx  
                   .text:0048F2B7                 push    esi  
                   .text:0048F2B8                 push    edi  
                   .text:0048F2B9                 xor     esi, esi  
                   .text:0048F2BB                 mov     \[ebp+arg\_4\], 20h  作 32 轮  
                   .text:0048F2C2                 mov     edi, ecx  
                   .text:0048F2C4                 shr     edi, 5  
                   .text:0048F2C7                 mov     ebx, ecx  
                   .text:0048F2C9                 shl     ebx, 4  
                   .text:0048F2CC                 xor     edi, ebx  
                   .text:0048F2CE                 add     edi, ecx  
                   .text:0048F2D0                 mov     ebx, esi  
                   .text:0048F2D2                 and     ebx, 3  
                   .text:0048F2D5                 mov     ebx, \[edx+ebx\*4\]  
                   .text:0048F2D8                 add     ebx, esi  
                   .text:0048F2DA                 xor     edi, ebx  
                   .text:0048F2DC                 add     eax, edi  
                   .text:0048F2DE                 mov     edi, eax  
                   .text:0048F2E0                 shr     edi, 5  
                   .text:0048F2E3                 mov     ebx, eax  
                   .text:0048F2E5                 shl     ebx, 4  
                   .text:0048F2E8                 xor     edi, ebx  
                   .text:0048F2EA                 sub     esi, 61C88647h    加密函数中唯一出现的奇怪常数  
                   .text:0048F2F0                 mov     ebx, esi  
                   .text:0048F2F2                 shr     ebx, 0Bh  
                   .text:0048F2F5                 and     ebx, 3  
                   .text:0048F2F8                 mov     ebx, \[edx+ebx\*4\]  
                   .text:0048F2FB                 add     edi, eax  
                   .text:0048F2FD                 add     ebx, esi  
                   .text:0048F2FF                 xor     edi, ebx  
                   .text:0048F301                 add     ecx, edi  
                   .text:0048F303                 dec     \[ebp+arg\_4\]  
                   .text:0048F306                 jnz     short loc\_48F2C2  
                   .text:0048F308                 mov     edx, \[ebp+arg\_C\]  
                   .text:0048F30B                 pop     edi  
                   .text:0048F30C                 pop     esi  
                   .text:0048F30D                 mov     \[edx\], eax  
                   .text:0048F30F                 mov     \[edx+4\], ecx  
                   .text:0048F312                 pop     ebx  
                   .text:0048F313                 pop     ebp  
                   .text:0048F314                 retn  
  
    一般到这里就要开始体力劳动了，不过或许可以碰碰运气   
    google 关键字 0x61C88647  很快便能找到这个公开的加密解密算法 ^\_^  
    本文以讨论思路为主，破解到此为止，在网吧玩游戏的朋友小心哦~

 4dd466d0e1e0cc123afbe504da2462ca