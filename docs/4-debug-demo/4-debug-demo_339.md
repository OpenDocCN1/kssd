

# GRDuw 4.1.0

> 原文：[`www.pediy.com/kssd/pediy04/kanxue501.htm`](https://www.pediy.com/kssd/pediy04/kanxue501.htm)

作者：时空幻影  
时间：2001 年 12 月 10 日  
使用工具：TRW2000 v1.22 已注册版、W32DSM 汉化白金版  
软件名称：GRDuw  
软件来源：《数码时代》杂志（2001 年 4 月刊）——配套光盘  
软件版本：4.1.0 汉化版（由水手创作室汉化）  
软件简介：相当不错的磁盘工具，可帮你将 1.44MB 的磁片格式化成高压缩的 1.72MB 磁片。另外还有许多其它好用的功能，譬如只需读取一次磁盘  
即可重复拷贝多份相同的磁盘 (包括抽取式磁碟机)、能夠比较两張磁片的內容、可存取磁碟影像文件 (disk image file) 待稍后用来复制磁片，  
此外也可以为每一个影像文件加上适当的注解以增加文件的可读性、可最佳化磁片的格式来提升资料读取的速度、可修复软/硬碟并自动记录损坏的  
磁道并且将该区的资料移到安全的区域存放。  
  
    这个软件我在很久以前就想做它的注册文件生成器，无奈功力所限，跟踪的时候总是摸不着头脑，找不到注册算法的部分，直到前几天读了 TAE!\[CCG\]  
的暴破文章后，有所启发，觉得该软件是用一个全局变量来作注册标志，经过我的跟踪分析，发现只要有不对的地方，都会有一条指令送注册失败值到全局  
变量（假设这个全局变量为 reg），这个全局变量就是下面的\[ESI+000005B8\]或者是\[EBX+000005B8\]，所以只要全部把这些指令机器代码中的  
0000001000000000 改成 0000000000000000 就可完美暴破。  
如:0041B304 C786B805000001000000    mov dword ptr \[esi+000005B8\], 00000001  
                         ^  
                       改为 0  
但是本人有个癖好，除非万不得已的情况下（如注册算法不可逆等），是不会暴破的，所以我决定把注册文件搞出来，经过十几个小时的破解后我成功了。  
    现在我们开始我们的破解之旅吧！先用 TRW2000 载入 GRDUW，然后点击"LOAD"，程序会被拦下，设置断点 BPX CREATEFILEA，然后按 F5，程序会再次被拦  
下，接着输入指令 PMODULE，就会停在如下所示的地方：  
  
\* Reference To: KERNEL32.CreateFileA, Ord:0034h  
                                  |  
:0042A501 FF1538724400            Call dword ptr \[00447238\]  
:0042A507 8BF0                    mov esi, eax  <--停在这里  
:0042A509 3BF7                    cmp esi, edi  
:0042A50B 7514                    jne 0042A521  <--如果存在 GRDuw.key 的话会跳转  
  
    继续按 F10，经过几个 ret 后会来到如下地方：  
\* Possible StringData Ref from Data Obj ->"rb"  <--表明注册文件打开模式为"rb"  
                                  |  
:0041B2C4 68A0314500              push 004531A0  
:0041B2C9 8D463C                  lea eax, dword ptr \[esi+3C\]  
:0041B2CC C7864005000000000000    mov dword ptr \[esi+00000540\], 00000000  
:0041B2D6 50                      push eax  
:0041B2D7 E8C1500000              call 0042039D  <--检查是否存在注册文件 GRDuw.key，存在的话 eax 为存放文件内容的首地址，否则为 0  
:0041B2DC 83C408                  add esp, 00000008  <--来到这里  
:0041B2DF 8D8E48050000            lea ecx, dword ptr \[esi+00000548\]  
:0041B2E5 8BF8                    mov edi, eax  
:0041B2E7 E8B4300000              call 0041E3A0  <--MD5 算法的赋初始值，注意该软件的 MD5 初始值不是标准的 MD5 初始值  
:0041B2EC C786B405000010AD4100    mov dword ptr \[esi+000005B4\], 0041AD10  <--送下一个检查子程序的地址  
:0041B2F6 85FF                    test edi, edi  
:0041B2F8 7517                    jne 0041B311  <--应该跳转  
:0041B2FA C786B405000020A94100    mov dword ptr \[esi+000005B4\], 0041A920  <--送注册失败处理子程序的地址  
  
\* Possible Reference to String Resource ID=00001: "萣  
\*"  
                                  |  
:0041B304 C786B805000001000000    mov dword ptr \[esi+000005B8\], 00000001  <--把注册失败值送全局变量 reg  
:0041B30E 5F                      pop edi  
:0041B30F 5E                      pop esi  
:0041B310 C3                      ret  
  
  
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:0041B2F8(C)  
|  
:0041B311 57                      push edi  
  
\* Possible Reference to Dialog: DialogID\_0084, CONTROL\_ID:0400, "疹 c(&F)(檈<)"  
                                  |  
  
\* Possible Reference to String Resource ID=01024: "\*<:(圅)"  
                                  |  
:0041B312 6800040000              push 00000400  
:0041B317 8D8E40010000            lea ecx, dword ptr \[esi+00000140\]  
  
\* Possible Reference to String Resource ID=00001: "萣  
\*"  
                                  |  
:0041B31D 6A01                    push 00000001  
:0041B31F 51                      push ecx  
:0041B320 E8304F0000              call 00420255  <--求注册文件 GRDuw.key 的长度，并把长度值送 eax 中  
:0041B325 83C410                  add esp, 00000010  
:0041B328 898640050000            mov dword ptr \[esi+00000540\], eax  
:0041B32E 85C0                    test eax, eax  
:0041B330 7514                    jne 0041B346  <--应该跳转  
:0041B332 C786B405000020A94100    mov dword ptr \[esi+000005B4\], 0041A920  <--送注册失败处理子程序的地址  
  
\* Possible Reference to String Resource ID=00001: "萣  
\*"  
                                  |  
:0041B33C C786B805000001000000    mov dword ptr \[esi+000005B8\], 00000001  <--把注册失败值送全局变量 reg  
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:0041B330(C)  
|  
:0041B346 57                      push edi  
:0041B347 E88C4E0000              call 004201D8  
:0041B34C 83C404                  add esp, 00000004  
:0041B34F 5F                      pop edi  
:0041B350 5E                      pop esi  
:0041B351 C3                      ret  
  
    经过上面这个 ret 后会来到如下地方：  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:0041ACF5(C)  
|  
:0041ACE0 8BCE                    mov ecx, esi  
:0041ACE2 8B86B4050000            mov eax, dword ptr \[esi+000005B4\]  
:0041ACE8 FFD0                    call eax  
:0041ACEA 8B86B4050000            mov eax, dword ptr \[esi+000005B4\]  <--来到这里，注意留意上面那个 call eax，以后会调用几次，目前为第一次  
:0041ACF0 3D40AD4100              cmp eax, 0041AD40  
:0041ACF5 75E9                    jne 0041ACE0  
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:0041ACDE(C)  
|  
:0041ACF7 8BCE                    mov ecx, esi  
:0041ACF9 E8CC8B0100              call 004338CA  
:0041ACFE 5E                      pop esi  
:0041ACFF C20C00                  ret 000C  
  
    我在网上找了很久，不论是在国外还是国内，都发现只有补丁程序，没有看到注册文件和注册文件的生成器，我怀疑是不是由于 CRACKERS 发现其注册算法  
含有 MD5 算法，所以就没有继续跟踪了。其实作该软件的注册机根本不需对 MD5 算法进行求逆。呵呵！！！  
  
//\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
  
    第二次调用 call eax 会来到如下地方：  
:0041AD10 C781B405000070B44100    mov dword ptr \[ebx+000005B4\], 0041B470  <--送下一个检查子程序的地址  
:0041AD1A 8B8140050000            mov eax, dword ptr \[ecx+00000540\]  <--eax 为注册文件 GRDuw.key 的长度  
:0041AD20 85C0                    test eax, eax  <--测试长度是否为 0  
:0041AD22 741B                    je 0041AD3F  <--不应该跳转  
:0041AD24 3DDC000000              cmp eax, 000000DC  <--比较长度是否不小于 0xDC 即 220 个字节  
:0041AD29 7314                    jnb 0041AD3F  <--是的话就跳转，应该要跳转  
:0041AD2B C781B405000020A94100    mov dword ptr \[ebx+000005B4\], 0041A920  <--送注册失败处理子程序的地址  
  
\* Possible Reference to String Resource ID=00001: "萣  
\*"  
                                  |  
:0041AD35 C781B805000001000000    mov dword ptr \[ebx+000005B8\], 00000001  <--把注册失败值送全局变量 reg  
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:  
|:0041AD22(C), :0041AD29(C)  
|  
:0041AD3F C3                      ret  
  
//\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
  
    第三次调用 call eax 会来到如下地方：  
:0041B470 56                      push esi  
:0041B471 57                      push edi  
:0041B472 8BF1                    mov esi, ecx  
:0041B474 E863610200              call 004415DC  
:0041B479 8B5010                  mov edx, dword ptr \[eax+10\]  <--edx 为存放字符串"GRDuw"的首地址  
:0041B47C 83C9FF                  or ecx, FFFFFFFF  
:0041B47F 8BFA                    mov edi, edx  
:0041B481 33C0                    xor eax, eax  
:0041B483 F2                      repnz  
:0041B484 AE                      scasb  
:0041B485 F7D1                    not ecx  
:0041B487 49                      dec ecx  
:0041B488 8BF9                    mov edi, ecx  <--edi 为上面字符串的长度  
:0041B48A C786B4050000C0BC4100    mov dword ptr \[esi+000005B4\], 0041BCC0  
:0041B494 57                      push edi  
:0041B495 8D8640010000            lea eax, dword ptr \[esi+00000140\]  
:0041B49B 52                      push edx  
:0041B49C 50                      push eax  
:0041B49D E85E4B0100              call 00430000  <--比较注册文件 GRDuw.key 中的前 5 个字符是否为"GRDuw"，是的话 eax 为 0  
:0041B4A2 83C40C                  add esp, 0000000C  
:0041B4A5 85C0                    test eax, eax  
:0041B4A7 750A                    jne 0041B4B3  <--不应该跳转  
:0041B4A9 80BC374001000020        cmp byte ptr \[edi+esi+00000140\], 20  <--比较注册文件 GRDuw.key 中的前 5 个字符是否为空格符  
:0041B4B1 7414                    je 0041B4C7  <--应该跳转  
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:0041B4A7(C)  
|  
:0041B4B3 C786B405000020A94100    mov dword ptr \[esi+000005B4\], 0041A920  <--送注册失败处理子程序的地址  
  
\* Possible Reference to String Resource ID=00001: "萣  
\*"  
                                  |  
:0041B4BD C786B805000001000000    mov dword ptr \[esi+000005B8\], 00000001  <--把注册失败值送全局变量 reg  
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:0041B4B1(C)  
|  
:0041B4C7 5F                      pop edi  
:0041B4C8 5E                      pop esi  
:0041B4C9 C3                      ret  
  
//\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
  
    第四次调用 call eax 会来到如下地方：  
:0041BCC0 56                      push esi  
:0041BCC1 8BF1                    mov esi, ecx  
:0041BCC3 8B8E40050000            mov ecx, dword ptr \[esi+00000540\]  
:0041BCC9 8D8640010000            lea eax, dword ptr \[esi+00000140\]  
:0041BCCF 898644050000            mov dword ptr \[esi+00000544\], eax  
:0041BCD5 C684314001000000        mov byte ptr \[ecx+esi+00000140\], 00  
:0041BCDD C786B4050000D0B44100    mov dword ptr \[esi+000005B4\], 0041B4D0  <--送下一个检查子程序的地址  
:0041BCE7 6A3D                    push 0000003D  <--3D 为"="的 ascii 码  
:0041BCE9 50                      push eax  <--eax 为存放注册文件 GRDuw.key 内容的首地址  
:0041BCEA E8014D0000              call 004209F0  <--检查注册文件 GRDuw.key 中是否含有字符"="，有的话则 eax 为存放该字符的地址，否则 eax 为存放注册文件内容的首地址  
:0041BCEF 83C408                  add esp, 00000008  
:0041BCF2 85C0                    test eax, eax  
:0041BCF4 7419                    je 0041BD0F  
:0041BCF6 8A50FF                  mov dl, byte ptr \[eax\-01\]  
:0041BCF9 B120                    mov cl, 20  
:0041BCFB 3AD1                    cmp dl, cl  <--检查字符"="的前一个字符是否为空格符  
:0041BCFD 7510                    jne 0041BD0F  <--不应该跳转  
:0041BCFF 384801                  cmp byte ptr \[eax+01\], cl  <--检查字符"="的后一个字符是否为空格符  
:0041BD02 750B                    jne 0041BD0F  <--不应该跳转  
:0041BD04 83C002                  add eax, 00000002  
:0041BD07 898644050000            mov dword ptr \[esi+00000544\], eax  
:0041BD0D 5E                      pop esi  
:0041BD0E C3                      ret  
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:  
|:0041BCF4(C), :0041BCFD(C), :0041BD02(C)  
|  
:0041BD0F C786B405000020A94100    mov dword ptr \[esi+000005B4\], 0041A920  <--送注册失败处理子程序的地址  
  
\* Possible Reference to String Resource ID=00001: "萣  
\*"  
                                  |  
:0041BD19 C786B805000001000000    mov dword ptr \[esi+000005B8\], 00000001  <--把注册失败值送全局变量 reg  
:0041BD23 5E                      pop esi  
:0041BD24 C3                      ret  
  
//\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
  
    第五次调用 call eax 会来到如下地方：  
:0041B4D0 56                      push esi  
:0041B4D1 8BF1                    mov esi, ecx  
:0041B4D3 57                      push edi  
:0041B4D4 8B8644050000            mov eax, dword ptr \[esi+00000544\]  <--eax 为存放注册文件中等号后面的注册码的首地址  
:0041B4DA 8D8E40010000            lea ecx, dword ptr \[esi+00000140\]  <--ecx 为存放注册文件内容的首地址  
:0041B4E0 2BC6                    sub eax, esi  
:0041B4E2 8DBE48050000            lea edi, dword ptr \[esi+00000548\]  
:0041B4E8 2D40010000              sub eax, 00000140  
:0041B4ED 50                      push eax  
:0041B4EE 51                      push ecx  
:0041B4EF 8BCF                    mov ecx, edi  
:0041B4F1 E8DA2E0000              call 0041E3D0  <--把注册文件中注册码前面的内容(包括等号及其后面的空格符)存到 MD5 算法的初始值后面  
:0041B4F6 C786B405000030BD4100    mov dword ptr \[esi+000005B4\], 0041BD30  <--送下一个检查子程序的地址  
:0041B500 8BCF                    mov ecx, edi  
:0041B502 E8792F0000              call 0041E480  <--核心 call，按 F8 进入  
:0041B507 8B8644050000            mov eax, dword ptr \[esi+00000544\]  <--eax 为存放注册文件中注册码部分的首地址  
  
\* Possible Reference to String Resource ID=00001: "萣  
\*"  
                                  |  
:0041B50D 6A01                    push 00000001  
:0041B50F 8A08                    mov cl, byte ptr \[eax\]  
:0041B511 40                      inc eax  
:0041B512 51                      push ecx  
:0041B513 8BCF                    mov ecx, edi  
:0041B515 898644050000            mov dword ptr \[esi+00000544\], eax2  
:0041B51B E870300000              call 0041E590  <--注册码为 64 位（16 个字符），这个 call 比对第 1 个字符  
:0041B520 85C0                    test eax, eax  
:0041B522 751D                    jne 0041B541  <--不应该跳转  
:0041B524 8B8644050000            mov eax, dword ptr \[esi+00000544\]  
:0041B52A 6A00                    push 00000000  
:0041B52C 8A08                    mov cl, byte ptr \[eax\]  
:0041B52E 40                      inc eax  
:0041B52F 51                      push ecx  
:0041B530 8BCF                    mov ecx, edi  
:0041B532 898644050000            mov dword ptr \[esi+00000544\], eax  
:0041B538 E853300000              call 0041E590  <--这个 call 比对第 2 个字符  
:0041B53D 85C0                    test eax, eax  
:0041B53F 7414                    je 0041B555  <--应该跳转  
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:0041B522(C)  
|  
:0041B541 C786B405000020A94100    mov dword ptr \[esi+000005B4\], 0041A920  <--送注册失败处理子程序的地址  
  
\* Possible Reference to String Resource ID=00001: "萣  
\*"  
                                  |  
:0041B54B C786B805000001000000    mov dword ptr \[esi+000005B8\], 00000001  <--把注册失败值送全局变量 reg  
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:0041B53F(C)  
|  
:0041B555 5F                      pop edi  
:0041B556 5E                      pop esi  
:0041B557 C3                      ret  
  
    在上面的核心 call 按 F8 进入后会来到如下地方：  
\* Referenced by a CALL at Addresses:  
|:0041AF26   , :0041B261   , :0041B502     
|  
:0041E480 83EC40                  sub esp, 00000040  
:0041E483 56                      push esi  
:0041E484 8BF1                    mov esi, ecx  
:0041E486 8B06                    mov eax, dword ptr \[esi\]  
:0041E488 8B4E04                  mov ecx, dword ptr \[esi+04\]  
:0041E48B 8944243C                mov dword ptr \[esp+3C\], eax  
:0041E48F 894C2440                mov dword ptr \[esp+40\], ecx  
:0041E493 C1E803                  shr eax, 03  
:0041E496 83E03F                  and eax, 0000003F  
:0041E499 B938000000              mov ecx, 00000038  
:0041E49E 83F838                  cmp eax, 00000038  
:0041E4A1 7C05                    jl 0041E4A8  
  
\* Possible Reference to String Resource ID=00120: "辶\* 蠀 X:z"  
                                  |  
:0041E4A3 B978000000              mov ecx, 00000078  
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:0041E4A1(C)  
|  
:0041E4A8 2BC8                    sub ecx, eax  
:0041E4AA 55                      push ebp  
:0041E4AB 57                      push edi  
:0041E4AC 51                      push ecx  
  
\* Possible StringData Ref from Data Obj ->"€"  
                                  |  
:0041E4AD 6840484500              push 00454840  
:0041E4B2 8BCE                    mov ecx, esi  
:0041E4B4 E817FFFFFF              call 0041E3D0  <--如果注册文件中注册码前面的内容长度不足 0x40 的话，则先在其后面添一个 0x80，然后在用 0x00 填充，直到长度为 0x40 为止  
:0041E4B9 8D461A                  lea eax, dword ptr \[esi+1A\]  
:0041E4BC 8D7C240C                lea edi, dword ptr \[esp+0C\]  
  
\* Possible Reference to String Resource ID=00014: "馼 = %lu"  
                                  |  
:0041E4C0 BD0E000000              mov ebp, 0000000E  
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:0041E4EA(C)  
|  
:0041E4C5 33D2                    xor edx, edx  
:0041E4C7 33C9                    xor ecx, ecx  
:0041E4C9 8A7001                  mov dh, byte ptr \[eax+01\]  
:0041E4CC 8A48FF                  mov cl, byte ptr \[eax\-01\]  
:0041E4CF 8A10                    mov dl, byte ptr \[eax\]  
:0041E4D1 83C704                  add edi, 00000004  
:0041E4D4 C1E208                  shl edx, 08  
:0041E4D7 0BD1                    or edx, ecx  
:0041E4D9 33C9                    xor ecx, ecx  
:0041E4DB 8A48FE                  mov cl, byte ptr \[eax\-02\]  
:0041E4DE 83C004                  add eax, 00000004  
:0041E4E1 C1E208                  shl edx, 08  
:0041E4E4 0BD1                    or edx, ecx  
:0041E4E6 4D                      dec ebp  
:0041E4E7 8957FC                  mov dword ptr \[edi\-04\], edx  
:0041E4EA 75D9                    jne 0041E4C5  
:0041E4EC 8D54240C                lea edx, dword ptr \[esp+0C\]  <--edx 为存放 MD5 算法的初始值的首地址  
:0041E4F0 8D7E08                  lea edi, dword ptr \[esi+08\]  <--edi 为存放注册文件中非注册码的部分及填充部分的首地址  
:0041E4F3 52                      push edx  
:0041E4F4 57                      push edi  
:0041E4F5 8BCE                    mov ecx, esi  
:0041E4F7 E854010000              call 0041E650  <--进入 MD5 算法，按 F8 进入(不想看 MD5 算法的话可按 F10 带过此 call)  
:0041E4FC 8BC7                    mov eax, edi  <--edi 为计算后的结果的首地址  
:0041E4FE 8D4E59                  lea ecx, dword ptr \[esi+59\]  
:0041E501 5F                      pop edi  
  
\* Possible Reference to String Resource ID=00004: ".\*(&H)"  
                                  |  
:0041E502 BE04000000              mov esi, 00000004  
:0041E507 5D                      pop ebp  
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:0041E52E(C)  
|  
:0041E508 8A10                    mov dl, byte ptr \[eax\]  
:0041E50A 83C004                  add eax, 00000004  
:0041E50D 8851FF                  mov byte ptr \[ecx\-01\], dl  
:0041E510 8B50FC                  mov edx, dword ptr \[eax\-04\]  
:0041E513 C1EA08                  shr edx, 08  
:0041E516 8811                    mov byte ptr \[ecx\], dl  
:0041E518 8B50FC                  mov edx, dword ptr \[eax\-04\]  
:0041E51B C1EA10                  shr edx, 10  
:0041E51E 885101                  mov byte ptr \[ecx+01\], dl  
:0041E521 8B50FC                  mov edx, dword ptr \[eax\-04\]  
:0041E524 C1EA18                  shr edx, 18  
:0041E527 885102                  mov byte ptr \[ecx+02\], dl  
:0041E52A 83C104                  add ecx, 00000004  
:0041E52D 4E                      dec esi  
:0041E52E 75D8                    jne 0041E508  
:0041E530 5E                      pop esi  
:0041E531 83C440                  add esp, 00000040  
:0041E534 C3                      ret  
  
    在上面的核心 call 按 F8 进入后会来到如下地方：  
\* Referenced by a CALL at Addresses:  
|:0041E45B   , :0041E4F7     
|  
:0041E650 83EC44                  sub esp, 00000044  
:0041E653 8B44244C                mov eax, dword ptr \[esp+4C\]  <--eax 为存放注册文件中非注册码的部分及填充部分的首地址  
:0041E657 53                      push ebx  
:0041E658 8B5C244C                mov ebx, dword ptr \[esp+4C\]  <--ebx 为存放 MD5 算法的初始值的首地址  
:0041E65C 55                      push ebp  
:0041E65D 56                      push esi  
:0041E65E 57                      push edi  
:0041E65F 8B38                    mov edi, dword ptr \[eax\]  
:0041E661 8B7304                  mov esi, dword ptr \[ebx+04\]  
:0041E664 8B4B0C                  mov ecx, dword ptr \[ebx+0C\]  
:0041E667 8B6B08                  mov ebp, dword ptr \[ebx+08\]  
:0041E66A 897C245C                mov dword ptr \[esp+5C\], edi  
:0041E66E 8BFE                    mov edi, esi  
:0041E670 F7D7                    not edi  
:0041E672 23F9                    and edi, ecx  
:0041E674 8B13                    mov edx, dword ptr \[ebx\]  
:0041E676 8BCD                    mov ecx, ebp  
:0041E678 23CE                    and ecx, esi  
:0041E67A 0BF9                    or edi, ecx  
:0041E67C 8B4C245C                mov ecx, dword ptr \[esp+5C\]  
:0041E680 03F9                    add edi, ecx  
:0041E682 8D941778A46AD7          lea edx, dword ptr \[edi+edx\-28955B88\]  
:0041E689 8BFE                    mov edi, esi  
:0041E68B 8BCA                    mov ecx, edx  
:0041E68D C1E919                  shr ecx, 19  
:0041E690 C1E207                  shl edx, 07  
:0041E693 0BCA                    or ecx, edx  
:0041E695 8B5004                  mov edx, dword ptr \[eax+04\]  
:0041E698 03CE                    add ecx, esi  
:0041E69A 89542428                mov dword ptr \[esp+28\], edx  
:0041E69E 8BD1                    mov edx, ecx  
:0041E6A0 23F9                    and edi, ecx  
:0041E6A2 F7D2                    not edx  
:0041E6A4 23D5                    and edx, ebp  
:0041E6A6 0BD7                    or edx, edi  
:0041E6A8 8B7C2428                mov edi, dword ptr \[esp+28\]  
:0041E6AC 03D7                    add edx, edi  
:0041E6AE 8B7B0C                  mov edi, dword ptr \[ebx+0C\]  
:0041E6B1 8DBC1756B7C7E8          lea edi, dword ptr \[edi+edx\-173848AA\]  
:0041E6B8 8BD7                    mov edx, edi  
:0041E6BA C1EA14                  shr edx, 14  
:0041E6BD C1E70C                  shl edi, 0C  
:0041E6C0 0BD7                    or edx, edi  
:0041E6C2 8B7808                  mov edi, dword ptr \[eax+08\]  
:0041E6C5 03D1                    add edx, ecx  
:0041E6C7 897C2440                mov dword ptr \[esp+40\], edi  
:0041E6CB 8BFA                    mov edi, edx  
:0041E6CD F7D7                    not edi  
:0041E6CF 23FE                    and edi, esi  
:0041E6D1 8BF2                    mov esi, edx  
:0041E6D3 23F1                    and esi, ecx  
:0041E6D5 0BFE                    or edi, esi  
:0041E6D7 8B742440                mov esi, dword ptr \[esp+40\]  
:0041E6DB 03FE                    add edi, esi  
:0041E6DD 8DBC2FDB702024          lea edi, dword ptr \[edi+ebp+242070DB\]  
:0041E6E4 8BEA                    mov ebp, edx  
:0041E6E6 8BF7                    mov esi, edi  
:0041E6E8 C1EE0F                  shr esi, 0F  
:0041E6EB C1E711                  shl edi, 11  
:0041E6EE 0BF7                    or esi, edi  
:0041E6F0 8B780C                  mov edi, dword ptr \[eax+0C\]  
:0041E6F3 03F2                    add esi, edx  
:0041E6F5 897C2420                mov dword ptr \[esp+20\], edi  
:0041E6F9 8BFE                    mov edi, esi  
:0041E6FB 23EE                    and ebp, esi  
:0041E6FD F7D7                    not edi  
:0041E6FF 23F9                    and edi, ecx  
:0041E701 0BFD                    or edi, ebp  
:0041E703 8B6C2420                mov ebp, dword ptr \[esp+20\]  
:0041E707 03FD                    add edi, ebp  
:0041E709 8B6B04                  mov ebp, dword ptr \[ebx+04\]  
:0041E70C 8DAC2FEECEBDC1          lea ebp, dword ptr \[edi+ebp\-3E423112\]  
:0041E713 8BFD                    mov edi, ebp  
:0041E715 C1E716                  shl edi, 16  
:0041E718 C1ED0A                  shr ebp, 0A  
:0041E71B 0BFD                    or edi, ebp  
:0041E71D 8B6810                  mov ebp, dword ptr \[eax+10\]  
:0041E720 03FE                    add edi, esi  
:0041E722 896C2438                mov dword ptr \[esp+38\], ebp  
:0041E726 897C2458                mov dword ptr \[esp+58\], edi  
:0041E72A F7D7                    not edi  
:0041E72C 23FA                    and edi, edx  
:0041E72E 8BEE                    mov ebp, esi  
:0041E730 236C2458                and ebp, dword ptr \[esp+58\]  
:0041E734 0BFD                    or edi, ebp  
:0041E736 8B6C2438                mov ebp, dword ptr \[esp+38\]  
:0041E73A 03FD                    add edi, ebp  
:0041E73C 8B6814                  mov ebp, dword ptr \[eax+14\]  
:0041E73F 896C2418                mov dword ptr \[esp+18\], ebp  
:0041E743 8D8C39AF0F7CF5          lea ecx, dword ptr \[ecx+edi\-0A83F051\]  
:0041E74A 8BF9                    mov edi, ecx  
:0041E74C C1EF19                  shr edi, 19  
:0041E74F C1E107                  shl ecx, 07  
:0041E752 0BF9                    or edi, ecx  
:0041E754 8B4C2458                mov ecx, dword ptr \[esp+58\]  
:0041E758 03F9                    add edi, ecx  
:0041E75A 8BEF                    mov ebp, edi  
:0041E75C 23CF                    and ecx, edi  
:0041E75E F7D5                    not ebp  
:0041E760 23EE                    and ebp, esi  
:0041E762 0BE9                    or ebp, ecx  
:0041E764 8B4C2418                mov ecx, dword ptr \[esp+18\]  
:0041E768 03E9                    add ebp, ecx  
:0041E76A 8D942A2AC68747          lea edx, dword ptr \[edx+ebp+4787C62A\]  
:0041E771 8B6C2458                mov ebp, dword ptr \[esp+58\]  
:0041E775 8BCA                    mov ecx, edx  
:0041E777 C1E914                  shr ecx, 14  
:0041E77A C1E20C                  shl edx, 0C  
:0041E77D 0BCA                    or ecx, edx  
:0041E77F 8B5018                  mov edx, dword ptr \[eax+18\]  
:0041E782 03CF                    add ecx, edi  
:0041E784 89542430                mov dword ptr \[esp+30\], edx  
:0041E788 8BD1                    mov edx, ecx  
:0041E78A F7D2                    not edx  
:0041E78C 23D5                    and edx, ebp  
:0041E78E 8BE9                    mov ebp, ecx  
:0041E790 23EF                    and ebp, edi  
:0041E792 0BD5                    or edx, ebp  
:0041E794 8B6C2430                mov ebp, dword ptr \[esp+30\]  
:0041E798 03D5                    add edx, ebp  
:0041E79A 8BE9                    mov ebp, ecx  
:0041E79C 8DB416134630A8          lea esi, dword ptr \[esi+edx\-57CFB9ED\]  
:0041E7A3 8BD6                    mov edx, esi  
:0041E7A5 C1EA0F                  shr edx, 0F  
:0041E7A8 C1E611                  shl esi, 11  
:0041E7AB 0BD6                    or edx, esi  
:0041E7AD 8B701C                  mov esi, dword ptr \[eax+1C\]  
:0041E7B0 03D1                    add edx, ecx  
:0041E7B2 89742410                mov dword ptr \[esp+10\], esi  
:0041E7B6 8BF2                    mov esi, edx  
:0041E7B8 23EA                    and ebp, edx  
:0041E7BA F7D6                    not esi  
:0041E7BC 23F7                    and esi, edi  
:0041E7BE 0BF5                    or esi, ebp  
:0041E7C0 8B6C2410                mov ebp, dword ptr \[esp+10\]  
:0041E7C4 03F5                    add esi, ebp  
:0041E7C6 8B6C2458                mov ebp, dword ptr \[esp+58\]  
:0041E7CA 8DAC2E019546FD          lea ebp, dword ptr \[esi+ebp\-02B96AFF\]  
:0041E7D1 8BF5                    mov esi, ebp  
:0041E7D3 C1E616                  shl esi, 16  
:0041E7D6 C1ED0A                  shr ebp, 0A  
:0041E7D9 0BF5                    or esi, ebp  
:0041E7DB 8B6820                  mov ebp, dword ptr \[eax+20\]  
:0041E7DE 03F2                    add esi, edx  
:0041E7E0 896C242C                mov dword ptr \[esp+2C\], ebp  
:0041E7E4 89742458                mov dword ptr \[esp+58\], esi  
:0041E7E8 8BEA                    mov ebp, edx  
:0041E7EA 236C2458                and ebp, dword ptr \[esp+58\]  
:0041E7EE F7D6                    not esi  
:0041E7F0 23F1                    and esi, ecx  
:0041E7F2 0BF5                    or esi, ebp  
:0041E7F4 8B6C242C                mov ebp, dword ptr \[esp+2C\]  
:0041E7F8 03F5                    add esi, ebp  
:0041E7FA 8B6824                  mov ebp, dword ptr \[eax+24\]  
:0041E7FD 896C2444                mov dword ptr \[esp+44\], ebp  
:0041E801 8DBC37D8988069          lea edi, dword ptr \[edi+esi+698098D8\]  
:0041E808 8BF7                    mov esi, edi  
:0041E80A C1EE19                  shr esi, 19  
:0041E80D C1E707                  shl edi, 07  
:0041E810 0BF7                    or esi, edi  
:0041E812 8B7C2458                mov edi, dword ptr \[esp+58\]  
:0041E816 03F7                    add esi, edi  
:0041E818 8BEE                    mov ebp, esi  
:0041E81A F7D5                    not ebp  
:0041E81C 23EA                    and ebp, edx  
:0041E81E 23FE                    and edi, esi  
:0041E820 0BEF                    or ebp, edi  
:0041E822 8B7C2444                mov edi, dword ptr \[esp+44\]  
:0041E826 03EF                    add ebp, edi  
:0041E828 8D8C29AFF7448B          lea ecx, dword ptr \[ecx+ebp\-74BB0851\]  
:0041E82F 8B6C2458                mov ebp, dword ptr \[esp+58\]  
:0041E833 8BF9                    mov edi, ecx  
:0041E835 C1EF14                  shr edi, 14  
:0041E838 C1E10C                  shl ecx, 0C  
:0041E83B 0BF9                    or edi, ecx  
:0041E83D 8B4828                  mov ecx, dword ptr \[eax+28\]  
:0041E840 03FE                    add edi, esi  
:0041E842 894C2424                mov dword ptr \[esp+24\], ecx  
:0041E846 8BCF                    mov ecx, edi  
:0041E848 F7D1                    not ecx  
:0041E84A 23CD                    and ecx, ebp  
:0041E84C 8BEF                    mov ebp, edi  
:0041E84E 23EE                    and ebp, esi  
:0041E850 0BCD                    or ecx, ebp  
:0041E852 8B6C2424                mov ebp, dword ptr \[esp+24\]  
:0041E856 03CD                    add ecx, ebp  
:0041E858 8BEF                    mov ebp, edi  
:0041E85A 8D940AB15BFFFF          lea edx, dword ptr \[edx+ecx\-0000A44F\]  
:0041E861 8BCA                    mov ecx, edx  
:0041E863 C1E90F                  shr ecx, 0F  
:0041E866 C1E211                  shl edx, 11  
:0041E869 0BCA                    or ecx, edx  
:0041E86B 8B502C                  mov edx, dword ptr \[eax+2C\]  
:0041E86E 03CF                    add ecx, edi  
:0041E870 8954243C                mov dword ptr \[esp+3C\], edx  
:0041E874 8BD1                    mov edx, ecx  
:0041E876 23E9                    and ebp, ecx  
:0041E878 F7D2                    not edx  
:0041E87A 23D6                    and edx, esi  
:0041E87C 0BD5                    or edx, ebp  
:0041E87E 8B6C243C                mov ebp, dword ptr \[esp+3C\]  
:0041E882 03D5                    add edx, ebp  
:0041E884 8B6C2458                mov ebp, dword ptr \[esp+58\]  
:0041E888 8DAC2ABED75C89          lea ebp, dword ptr \[edx+ebp\-76A32842\]  
:0041E88F 8BD5                    mov edx, ebp  
:0041E891 C1E216                  shl edx, 16  
:0041E894 C1ED0A                  shr ebp, 0A  
:0041E897 0BD5                    or edx, ebp  
:0041E899 8B6830                  mov ebp, dword ptr \[eax+30\]  
:0041E89C 03D1                    add edx, ecx  
:0041E89E 896C241C                mov dword ptr \[esp+1C\], ebp  
:0041E8A2 89542458                mov dword ptr \[esp+58\], edx  
:0041E8A6 8BE9                    mov ebp, ecx  
:0041E8A8 236C2458                and ebp, dword ptr \[esp+58\]  
:0041E8AC F7D2                    not edx  
:0041E8AE 23D7                    and edx, edi  
:0041E8B0 0BD5                    or edx, ebp  
:0041E8B2 8B6C241C                mov ebp, dword ptr \[esp+1C\]  
:0041E8B6 03D5                    add edx, ebp  
:0041E8B8 8B6834                  mov ebp, dword ptr \[eax+34\]  
:0041E8BB 896C2434                mov dword ptr \[esp+34\], ebp  
:0041E8BF 8DB4162211906B          lea esi, dword ptr \[esi+edx+6B901122\]  
:0041E8C6 8BD6                    mov edx, esi  
:0041E8C8 C1EA19                  shr edx, 19  
:0041E8CB C1E607                  shl esi, 07  
:0041E8CE 0BD6                    or edx, esi  
:0041E8D0 8B742458                mov esi, dword ptr \[esp+58\]  
:0041E8D4 03D6                    add edx, esi  
:0041E8D6 8BEA                    mov ebp, edx  
:0041E8D8 23F2                    and esi, edx  
:0041E8DA F7D5                    not ebp  
:0041E8DC 23E9                    and ebp, ecx  
:0041E8DE 0BEE                    or ebp, esi  
:0041E8E0 8B742434                mov esi, dword ptr \[esp+34\]  
:0041E8E4 03EE                    add ebp, esi  
:0041E8E6 8DBC2F937198FD          lea edi, dword ptr \[edi+ebp\-02678E6D\]  
:0041E8ED 8BF7                    mov esi, edi  
:0041E8EF C1EE14                  shr esi, 14  
:0041E8F2 C1E70C                  shl edi, 0C  
:0041E8F5 0BF7                    or esi, edi  
:0041E8F7 03F2                    add esi, edx  
:0041E8F9 8BFE                    mov edi, esi  
:0041E8FB F7D7                    not edi  
:0041E8FD 897C244C                mov dword ptr \[esp+4C\], edi  
:0041E901 8B6838                  mov ebp, dword ptr \[eax+38\]  
:0041E904 8B403C                  mov eax, dword ptr \[eax+3C\]  
:0041E907 896C2414                mov dword ptr \[esp+14\], ebp  
:0041E90B 8B6C2458                mov ebp, dword ptr \[esp+58\]  
:0041E90F 23FD                    and edi, ebp  
:0041E911 8BEE                    mov ebp, esi  
:0041E913 23EA                    and ebp, edx  
:0041E915 89442448                mov dword ptr \[esp+48\], eax  
:0041E919 0BFD                    or edi, ebp  
:0041E91B 8B6C2414                mov ebp, dword ptr \[esp+14\]  
:0041E91F 03FD                    add edi, ebp  
:0041E921 8BEE                    mov ebp, esi  
:0041E923 8D8C398E4379A6          lea ecx, dword ptr \[ecx+edi\-5986BC72\]  
:0041E92A 8BF9                    mov edi, ecx  
:0041E92C C1EF0F                  shr edi, 0F  
:0041E92F C1E111                  shl ecx, 11  
:0041E932 0BF9                    or edi, ecx  
:0041E934 03FE                    add edi, esi  
:0041E936 8BCF                    mov ecx, edi  
:0041E938 23EF                    and ebp, edi  
:0041E93A F7D1                    not ecx  
:0041E93C 894C2450                mov dword ptr \[esp+50\], ecx  
:0041E940 23CA                    and ecx, edx  
:0041E942 0BCD                    or ecx, ebp  
:0041E944 8BEE                    mov ebp, esi  
:0041E946 03C8                    add ecx, eax  
:0041E948 8B442458                mov eax, dword ptr \[esp+58\]  
:0041E94C 8D84082108B449          lea eax, dword ptr \[eax+ecx+49B40821\]  
:0041E953 8BC8                    mov ecx, eax  
:0041E955 C1E116                  shl ecx, 16  
:0041E958 C1E80A                  shr eax, 0A  
:0041E95B 0BC8                    or ecx, eax  
:0041E95D 8B44244C                mov eax, dword ptr \[esp+4C\]  
:0041E961 03CF                    add ecx, edi  
:0041E963 23C7                    and eax, edi  
:0041E965 23E9                    and ebp, ecx  
:0041E967 0BC5                    or eax, ebp  
:0041E969 8B6C2428                mov ebp, dword ptr \[esp+28\]  
:0041E96D 03C5                    add eax, ebp  
:0041E96F 8BEF                    mov ebp, edi  
:0041E971 8D840262251EF6          lea eax, dword ptr \[edx+eax\-09E1DA9E\]  
:0041E978 8BD0                    mov edx, eax  
:0041E97A C1EA1B                  shr edx, 1B  
:0041E97D C1E005                  shl eax, 05  
:0041E980 0BD0                    or edx, eax  
:0041E982 8B442450                mov eax, dword ptr \[esp+50\]  
:0041E986 03D1                    add edx, ecx  
:0041E988 23C1                    and eax, ecx  
:0041E98A 23EA                    and ebp, edx  
:0041E98C 0BC5                    or eax, ebp  
:0041E98E 8B6C2430                mov ebp, dword ptr \[esp+30\]  
:0041E992 03C5                    add eax, ebp  
:0041E994 8D840640B340C0          lea eax, dword ptr \[esi+eax\-3FBF4CC0\]  
:0041E99B 8BF0                    mov esi, eax  
:0041E99D C1EE17                  shr esi, 17  
:0041E9A0 C1E009                  shl eax, 09  
:0041E9A3 0BF0                    or esi, eax  
:0041E9A5 8BC1                    mov eax, ecx  
:0041E9A7 03F2                    add esi, edx  
:0041E9A9 F7D0                    not eax  
:0041E9AB 8BEE                    mov ebp, esi  
:0041E9AD 23C2                    and eax, edx  
:0041E9AF 23E9                    and ebp, ecx  
:0041E9B1 0BC5                    or eax, ebp  
:0041E9B3 8B6C243C                mov ebp, dword ptr \[esp+3C\]  
:0041E9B7 03C5                    add eax, ebp  
:0041E9B9 8D8407515A5E26          lea eax, dword ptr \[edi+eax+265E5A51\]  
:0041E9C0 8BF8                    mov edi, eax  
:0041E9C2 C1EF12                  shr edi, 12  
:0041E9C5 C1E00E                  shl eax, 0E  
:0041E9C8 0BF8                    or edi, eax  
:0041E9CA 8BC2                    mov eax, edx  
:0041E9CC 03FE                    add edi, esi  
:0041E9CE F7D0                    not eax  
:0041E9D0 8BEF                    mov ebp, edi  
:0041E9D2 23C6                    and eax, esi  
:0041E9D4 23EA                    and ebp, edx  
:0041E9D6 0BC5                    or eax, ebp  
:0041E9D8 8B6C245C                mov ebp, dword ptr \[esp+5C\]  
:0041E9DC 03C5                    add eax, ebp  
:0041E9DE 8D8401AAC7B6E9          lea eax, dword ptr \[ecx+eax\-16493856\]  
:0041E9E5 8BC8                    mov ecx, eax  
:0041E9E7 8BEE                    mov ebp, esi  
:0041E9E9 C1E80C                  shr eax, 0C  
:0041E9EC C1E114                  shl ecx, 14  
:0041E9EF 0BC8                    or ecx, eax  
:0041E9F1 8BC6                    mov eax, esi  
:0041E9F3 F7D0                    not eax  
:0041E9F5 03CF                    add ecx, edi  
:0041E9F7 23C7                    and eax, edi  
:0041E9F9 23E9                    and ebp, ecx  
:0041E9FB 0BC5                    or eax, ebp  
:0041E9FD 8B6C2418                mov ebp, dword ptr \[esp+18\]  
:0041EA01 03C5                    add eax, ebp  
:0041EA03 8BEF                    mov ebp, edi  
:0041EA05 8D84025D102FD6          lea eax, dword ptr \[edx+eax\-29D0EFA3\]  
:0041EA0C 8BD0                    mov edx, eax  
:0041EA0E C1E005                  shl eax, 05  
:0041EA11 C1EA1B                  shr edx, 1B  
:0041EA14 0BD0                    or edx, eax  
:0041EA16 8BC7                    mov eax, edi  
:0041EA18 03D1                    add edx, ecx  
:0041EA1A F7D0                    not eax  
:0041EA1C 23C1                    and eax, ecx  
:0041EA1E 23EA                    and ebp, edx  
:0041EA20 0BC5                    or eax, ebp  
:0041EA22 8B6C2424                mov ebp, dword ptr \[esp+24\]  
:0041EA26 03C5                    add eax, ebp  
:0041EA28 8D840653144402          lea eax, dword ptr \[esi+eax+02441453\]  
:0041EA2F 8BF0                    mov esi, eax  
:0041EA31 C1EE17                  shr esi, 17  
:0041EA34 C1E009                  shl eax, 09  
:0041EA37 0BF0                    or esi, eax  
:0041EA39 8BC1                    mov eax, ecx  
:0041EA3B 03F2                    add esi, edx  
:0041EA3D F7D0                    not eax  
:0041EA3F 8BEE                    mov ebp, esi  
:0041EA41 23C2                    and eax, edx  
:0041EA43 23E9                    and ebp, ecx  
:0041EA45 0BC5                    or eax, ebp  
:0041EA47 8B6C2448                mov ebp, dword ptr \[esp+48\]  
:0041EA4B 03C5                    add eax, ebp  
:0041EA4D 8D840781E6A1D8          lea eax, dword ptr \[edi+eax\-275E197F\]  
:0041EA54 8BF8                    mov edi, eax  
:0041EA56 C1EF12                  shr edi, 12  
:0041EA59 C1E00E                  shl eax, 0E  
:0041EA5C 0BF8                    or edi, eax  
:0041EA5E 8BC2                    mov eax, edx  
:0041EA60 03FE                    add edi, esi  
:0041EA62 F7D0                    not eax  
:0041EA64 8BEF                    mov ebp, edi  
:0041EA66 23C6                    and eax, esi  
:0041EA68 23EA                    and ebp, edx  
:0041EA6A 0BC5                    or eax, ebp  
:0041EA6C 8B6C2438                mov ebp, dword ptr \[esp+38\]  
:0041EA70 03C5                    add eax, ebp  
:0041EA72 8BEE                    mov ebp, esi  
:0041EA74 8D8401C8FBD3E7          lea eax, dword ptr \[ecx+eax\-182C0438\]  
:0041EA7B 8BC8                    mov ecx, eax  
:0041EA7D C1E114                  shl ecx, 14  
:0041EA80 C1E80C                  shr eax, 0C  
:0041EA83 0BC8                    or ecx, eax  
:0041EA85 8BC6                    mov eax, esi  
:0041EA87 03CF                    add ecx, edi  
:0041EA89 F7D0                    not eax  
:0041EA8B 23C7                    and eax, edi  
:0041EA8D 23E9                    and ebp, ecx  
:0041EA8F 0BC5                    or eax, ebp  
:0041EA91 8B6C2444                mov ebp, dword ptr \[esp+44\]  
:0041EA95 03C5                    add eax, ebp  
:0041EA97 8BEF                    mov ebp, edi  
:0041EA99 8D8402E6CDE121          lea eax, dword ptr \[edx+eax+21E1CDE6\]  
:0041EAA0 8BD0                    mov edx, eax  
:0041EAA2 C1EA1B                  shr edx, 1B  
:0041EAA5 C1E005                  shl eax, 05  
:0041EAA8 0BD0                    or edx, eax  
:0041EAAA 8BC7                    mov eax, edi  
:0041EAAC 03D1                    add edx, ecx  
:0041EAAE F7D0                    not eax  
:0041EAB0 23C1                    and eax, ecx  
:0041EAB2 23EA                    and ebp, edx  
:0041EAB4 0BC5                    or eax, ebp  
:0041EAB6 03442414                add eax, dword ptr \[esp+14\]  
:0041EABA 8D8406D60737C3          lea eax, dword ptr \[esi+eax\-3CC8F82A\]  
:0041EAC1 8BF0                    mov esi, eax  
:0041EAC3 C1E009                  shl eax, 09  
:0041EAC6 C1EE17                  shr esi, 17  
:0041EAC9 0BF0                    or esi, eax  
:0041EACB 8BC1                    mov eax, ecx  
:0041EACD 03F2                    add esi, edx  
:0041EACF F7D0                    not eax  
:0041EAD1 8BEE                    mov ebp, esi  
:0041EAD3 23C2                    and eax, edx  
:0041EAD5 23E9                    and ebp, ecx  
:0041EAD7 0BC5                    or eax, ebp  
:0041EAD9 8B6C2420                mov ebp, dword ptr \[esp+20\]  
:0041EADD 03C5                    add eax, ebp  
:0041EADF 8D8407870DD5F4          lea eax, dword ptr \[edi+eax\-0B2AF279\]  
:0041EAE6 8BF8                    mov edi, eax  
:0041EAE8 C1E00E                  shl eax, 0E  
:0041EAEB C1EF12                  shr edi, 12  
:0041EAEE 0BF8                    or edi, eax  
:0041EAF0 8BC2                    mov eax, edx  
:0041EAF2 03FE                    add edi, esi  
:0041EAF4 F7D0                    not eax  
:0041EAF6 8BEF                    mov ebp, edi  
:0041EAF8 23C6                    and eax, esi  
:0041EAFA 23EA                    and ebp, edx  
:0041EAFC 0BC5                    or eax, ebp  
:0041EAFE 8B6C242C                mov ebp, dword ptr \[esp+2C\]  
:0041EB02 03C5                    add eax, ebp  
:0041EB04 8BEE                    mov ebp, esi  
:0041EB06 8D8401ED145A45          lea eax, dword ptr \[ecx+eax+455A14ED\]  
:0041EB0D 8BC8                    mov ecx, eax  
:0041EB0F C1E114                  shl ecx, 14  
:0041EB12 C1E80C                  shr eax, 0C  
:0041EB15 0BC8                    or ecx, eax  
:0041EB17 8BC6                    mov eax, esi  
:0041EB19 03CF                    add ecx, edi  
:0041EB1B F7D0                    not eax  
:0041EB1D 23C7                    and eax, edi  
:0041EB1F 23E9                    and ebp, ecx  
:0041EB21 0BC5                    or eax, ebp  
:0041EB23 8B6C2434                mov ebp, dword ptr \[esp+34\]  
:0041EB27 03C5                    add eax, ebp  
:0041EB29 8BEF                    mov ebp, edi  
:0041EB2B 8D840205E9E3A9          lea eax, dword ptr \[edx+eax\-561C16FB\]  
:0041EB32 8BD0                    mov edx, eax  
:0041EB34 C1EA1B                  shr edx, 1B  
:0041EB37 C1E005                  shl eax, 05  
:0041EB3A 0BD0                    or edx, eax  
:0041EB3C 8BC7                    mov eax, edi  
:0041EB3E 03D1                    add edx, ecx  
:0041EB40 F7D0                    not eax  
:0041EB42 23C1                    and eax, ecx  
:0041EB44 23EA                    and ebp, edx  
:0041EB46 0BC5                    or eax, ebp  
:0041EB48 8B6C2440                mov ebp, dword ptr \[esp+40\]  
:0041EB4C 03C5                    add eax, ebp  
:0041EB4E 8D8406F8A3EFFC          lea eax, dword ptr \[esi+eax\-03105C08\]  
:0041EB55 8BF0                    mov esi, eax  
:0041EB57 C1EE17                  shr esi, 17  
:0041EB5A C1E009                  shl eax, 09  
:0041EB5D 0BF0                    or esi, eax  
:0041EB5F 8BC1                    mov eax, ecx  
:0041EB61 03F2                    add esi, edx  
:0041EB63 F7D0                    not eax  
:0041EB65 8BEE                    mov ebp, esi  
:0041EB67 23C2                    and eax, edx  
:0041EB69 23E9                    and ebp, ecx  
:0041EB6B 0BC5                    or eax, ebp  
:0041EB6D 8B6C2410                mov ebp, dword ptr \[esp+10\]  
:0041EB71 03C5                    add eax, ebp  
:0041EB73 8D8407D9026F67          lea eax, dword ptr \[edi+eax+676F02D9\]  
:0041EB7A 8BF8                    mov edi, eax  
:0041EB7C C1EF12                  shr edi, 12  
:0041EB7F C1E00E                  shl eax, 0E  
:0041EB82 0BF8                    or edi, eax  
:0041EB84 8BC2                    mov eax, edx  
:0041EB86 F7D0                    not eax  
:0041EB88 03FE                    add edi, esi  
:0041EB8A 23C6                    and eax, esi  
:0041EB8C 8BEF                    mov ebp, edi  
:0041EB8E 23EA                    and ebp, edx  
:0041EB90 0BC5                    or eax, ebp  
:0041EB92 8B6C241C                mov ebp, dword ptr \[esp+1C\]  
:0041EB96 03C5                    add eax, ebp  
:0041EB98 8B6C2418                mov ebp, dword ptr \[esp+18\]  
:0041EB9C 8D84018A4C2A8D          lea eax, dword ptr \[ecx+eax\-72D5B376\]  
:0041EBA3 8BC8                    mov ecx, eax  
:0041EBA5 C1E80C                  shr eax, 0C  
:0041EBA8 C1E114                  shl ecx, 14  
:0041EBAB 0BC8                    or ecx, eax  
:0041EBAD 8BC6                    mov eax, esi  
:0041EBAF 33C7                    xor eax, edi  
:0041EBB1 03CF                    add ecx, edi  
:0041EBB3 33C1                    xor eax, ecx  
:0041EBB5 03C5                    add eax, ebp  
:0041EBB7 8B6C242C                mov ebp, dword ptr \[esp+2C\]  
:0041EBBB 8D84024239FAFF          lea eax, dword ptr \[edx+eax\-0005C6BE\]  
:0041EBC2 8BD0                    mov edx, eax  
:0041EBC4 C1E004                  shl eax, 04  
:0041EBC7 C1EA1C                  shr edx, 1C  
:0041EBCA 0BD0                    or edx, eax  
:0041EBCC 8BC7                    mov eax, edi  
:0041EBCE 33C1                    xor eax, ecx  
:0041EBD0 03D1                    add edx, ecx  
:0041EBD2 33C2                    xor eax, edx  
:0041EBD4 03C5                    add eax, ebp  
:0041EBD6 8B6C243C                mov ebp, dword ptr \[esp+3C\]  
:0041EBDA 8D840681F67187          lea eax, dword ptr \[esi+eax\-788E097F\]  
:0041EBE1 8BF0                    mov esi, eax  
:0041EBE3 C1EE15                  shr esi, 15  
:0041EBE6 C1E00B                  shl eax, 0B  
:0041EBE9 0BF0                    or esi, eax  
:0041EBEB 03F2                    add esi, edx  
:0041EBED 8BC6                    mov eax, esi  
:0041EBEF 33C1                    xor eax, ecx  
:0041EBF1 33C2                    xor eax, edx  
:0041EBF3 03C5                    add eax, ebp  
:0041EBF5 8B6C2414                mov ebp, dword ptr \[esp+14\]  
:0041EBF9 8D840722619D6D          lea eax, dword ptr \[edi+eax+6D9D6122\]  
:0041EC00 8BF8                    mov edi, eax  
:0041EC02 C1EF10                  shr edi, 10  
:0041EC05 C1E010                  shl eax, 10  
:0041EC08 0BF8                    or edi, eax  
:0041EC0A 8BC6                    mov eax, esi  
:0041EC0C 03FE                    add edi, esi  
:0041EC0E 33C7                    xor eax, edi  
:0041EC10 89442458                mov dword ptr \[esp+58\], eax  
:0041EC14 33C2                    xor eax, edx  
:0041EC16 03C5                    add eax, ebp  
:0041EC18 8B6C2428                mov ebp, dword ptr \[esp+28\]  
:0041EC1C 8D84010C38E5FD          lea eax, dword ptr \[ecx+eax\-021AC7F4\]  
:0041EC23 8BC8                    mov ecx, eax  
:0041EC25 C1E117                  shl ecx, 17  
:0041EC28 C1E809                  shr eax, 09  
:0041EC2B 0BC8                    or ecx, eax  
:0041EC2D 8B442458                mov eax, dword ptr \[esp+58\]  
:0041EC31 03CF                    add ecx, edi  
:0041EC33 33C1                    xor eax, ecx  
:0041EC35 03C5                    add eax, ebp  
:0041EC37 8B6C2438                mov ebp, dword ptr \[esp+38\]  
:0041EC3B 8D840244EABEA4          lea eax, dword ptr \[edx+eax\-5B4115BC\]  
:0041EC42 8BD0                    mov edx, eax  
:0041EC44 C1EA1C                  shr edx, 1C  
:0041EC47 C1E004                  shl eax, 04  
:0041EC4A 0BD0                    or edx, eax  
:0041EC4C 8BC7                    mov eax, edi  
:0041EC4E 03D1                    add edx, ecx  
:0041EC50 33C1                    xor eax, ecx  
:0041EC52 33C2                    xor eax, edx  
:0041EC54 03C5                    add eax, ebp  
:0041EC56 8B6C2410                mov ebp, dword ptr \[esp+10\]  
:0041EC5A 8D8406A9CFDE4B          lea eax, dword ptr \[esi+eax+4BDECFA9\]  
:0041EC61 8BF0                    mov esi, eax  
:0041EC63 C1EE15                  shr esi, 15  
:0041EC66 C1E00B                  shl eax, 0B  
:0041EC69 0BF0                    or esi, eax  
:0041EC6B 03F2                    add esi, edx  
:0041EC6D 8BC6                    mov eax, esi  
:0041EC6F 33C1                    xor eax, ecx  
:0041EC71 33C2                    xor eax, edx  
:0041EC73 03C5                    add eax, ebp  
:0041EC75 8B6C2424                mov ebp, dword ptr \[esp+24\]  
:0041EC79 8D8407604BBBF6          lea eax, dword ptr \[edi+eax\-0944B4A0\]  
:0041EC80 8BF8                    mov edi, eax  
:0041EC82 C1E010                  shl eax, 10  
:0041EC85 C1EF10                  shr edi, 10  
:0041EC88 0BF8                    or edi, eax  
:0041EC8A 8BC6                    mov eax, esi  
:0041EC8C 03FE                    add edi, esi  
:0041EC8E 33C7                    xor eax, edi  
:0041EC90 89442458                mov dword ptr \[esp+58\], eax  
:0041EC94 33C2                    xor eax, edx  
:0041EC96 03C5                    add eax, ebp  
:0041EC98 8B6C2434                mov ebp, dword ptr \[esp+34\]  
:0041EC9C 8D840170BCBFBE          lea eax, dword ptr \[ecx+eax\-41404390\]  
:0041ECA3 8BC8                    mov ecx, eax  
:0041ECA5 C1E809                  shr eax, 09  
:0041ECA8 C1E117                  shl ecx, 17  
:0041ECAB 0BC8                    or ecx, eax  
:0041ECAD 8B442458                mov eax, dword ptr \[esp+58\]  
:0041ECB1 03CF                    add ecx, edi  
:0041ECB3 33C1                    xor eax, ecx  
:0041ECB5 03C5                    add eax, ebp  
:0041ECB7 8B6C245C                mov ebp, dword ptr \[esp+5C\]  
:0041ECBB 8D8402C67E9B28          lea eax, dword ptr \[edx+eax+289B7EC6\]  
:0041ECC2 8BD0                    mov edx, eax  
:0041ECC4 C1EA1C                  shr edx, 1C  
:0041ECC7 C1E004                  shl eax, 04  
:0041ECCA 0BD0                    or edx, eax  
:0041ECCC 8BC7                    mov eax, edi  
:0041ECCE 03D1                    add edx, ecx  
:0041ECD0 33C1                    xor eax, ecx  
:0041ECD2 33C2                    xor eax, edx  
:0041ECD4 03C5                    add eax, ebp  
:0041ECD6 8B6C2420                mov ebp, dword ptr \[esp+20\]  
:0041ECDA 8D8406FA27A1EA          lea eax, dword ptr \[esi+eax\-155ED806\]  
:0041ECE1 8BF0                    mov esi, eax  
:0041ECE3 C1EE15                  shr esi, 15  
:0041ECE6 C1E00B                  shl eax, 0B  
:0041ECE9 0BF0                    or esi, eax  
:0041ECEB 03F2                    add esi, edx  
:0041ECED 8BC6                    mov eax, esi  
:0041ECEF 33C1                    xor eax, ecx  
:0041ECF1 33C2                    xor eax, edx  
:0041ECF3 03C5                    add eax, ebp  
:0041ECF5 8B6C2430                mov ebp, dword ptr \[esp+30\]  
:0041ECF9 8D84078530EFD4          lea eax, dword ptr \[edi+eax\-2B10CF7B\]  
:0041ED00 8BF8                    mov edi, eax  
:0041ED02 C1EF10                  shr edi, 10  
:0041ED05 C1E010                  shl eax, 10  
:0041ED08 0BF8                    or edi, eax  
:0041ED0A 8BC6                    mov eax, esi  
:0041ED0C 03FE                    add edi, esi  
:0041ED0E 33C7                    xor eax, edi  
:0041ED10 89442458                mov dword ptr \[esp+58\], eax  
:0041ED14 33C2                    xor eax, edx  
:0041ED16 03C5                    add eax, ebp  
:0041ED18 8B6C2444                mov ebp, dword ptr \[esp+44\]  
:0041ED1C 8D8401051D8804          lea eax, dword ptr \[ecx+eax+04881D05\]  
:0041ED23 8BC8                    mov ecx, eax  
:0041ED25 C1E117                  shl ecx, 17  
:0041ED28 C1E809                  shr eax, 09  
:0041ED2B 0BC8                    or ecx, eax  
:0041ED2D 8B442458                mov eax, dword ptr \[esp+58\]  
:0041ED31 03CF                    add ecx, edi  
:0041ED33 33C1                    xor eax, ecx  
:0041ED35 03C5                    add eax, ebp  
:0041ED37 8B6C241C                mov ebp, dword ptr \[esp+1C\]  
:0041ED3B 8D840239D0D4D9          lea eax, dword ptr \[edx+eax\-262B2FC7\]  
:0041ED42 8BD0                    mov edx, eax  
:0041ED44 C1EA1C                  shr edx, 1C  
:0041ED47 C1E004                  shl eax, 04  
:0041ED4A 0BD0                    or edx, eax  
:0041ED4C 8BC7                    mov eax, edi  
:0041ED4E 03D1                    add edx, ecx  
:0041ED50 33C1                    xor eax, ecx  
:0041ED52 33C2                    xor eax, edx  
:0041ED54 03C5                    add eax, ebp  
:0041ED56 8D8406E599DBE6          lea eax, dword ptr \[esi+eax\-1924661B\]  
:0041ED5D 8BF0                    mov esi, eax  
:0041ED5F C1EE15                  shr esi, 15  
:0041ED62 C1E00B                  shl eax, 0B  
:0041ED65 0BF0                    or esi, eax  
:0041ED67 8B6C2448                mov ebp, dword ptr \[esp+48\]  
:0041ED6B 03F2                    add esi, edx  
:0041ED6D 8BC6                    mov eax, esi  
:0041ED6F 33C1                    xor eax, ecx  
:0041ED71 33C2                    xor eax, edx  
:0041ED73 03C5                    add eax, ebp  
:0041ED75 8B6C2440                mov ebp, dword ptr \[esp+40\]  
:0041ED79 8D8407F87CA21F          lea eax, dword ptr \[edi+eax+1FA27CF8\]  
:0041ED80 8BF8                    mov edi, eax  
:0041ED82 C1E010                  shl eax, 10  
:0041ED85 C1EF10                  shr edi, 10  
:0041ED88 0BF8                    or edi, eax  
:0041ED8A 8BC6                    mov eax, esi  
:0041ED8C 03FE                    add edi, esi  
:0041ED8E 33C7                    xor eax, edi  
:0041ED90 33C2                    xor eax, edx  
:0041ED92 03C5                    add eax, ebp  
:0041ED94 8B6C245C                mov ebp, dword ptr \[esp+5C\]  
:0041ED98 8D84016556ACC4          lea eax, dword ptr \[ecx+eax\-3B53A99B\]  
:0041ED9F 8BC8                    mov ecx, eax  
:0041EDA1 C1E809                  shr eax, 09  
:0041EDA4 C1E117                  shl ecx, 17  
:0041EDA7 0BC8                    or ecx, eax  
:0041EDA9 8BC6                    mov eax, esi  
:0041EDAB F7D0                    not eax  
:0041EDAD 03CF                    add ecx, edi  
:0041EDAF 0BC1                    or eax, ecx  
:0041EDB1 33C7                    xor eax, edi  
:0041EDB3 03C5                    add eax, ebp  
:0041EDB5 8B6C2410                mov ebp, dword ptr \[esp+10\]  
:0041EDB9 8D8402442229F4          lea eax, dword ptr \[edx+eax\-0BD6DDBC\]  
:0041EDC0 8BD0                    mov edx, eax  
:0041EDC2 C1EA1A                  shr edx, 1A  
:0041EDC5 C1E006                  shl eax, 06  
:0041EDC8 0BD0                    or edx, eax  
:0041EDCA 8BC7                    mov eax, edi  
:0041EDCC 03D1                    add edx, ecx  
:0041EDCE F7D0                    not eax  
:0041EDD0 0BC2                    or eax, edx  
:0041EDD2 33C1                    xor eax, ecx  
:0041EDD4 03C5                    add eax, ebp  
:0041EDD6 8B6C2414                mov ebp, dword ptr \[esp+14\]  
:0041EDDA 8D840697FF2A43          lea eax, dword ptr \[esi+eax+432AFF97\]  
:0041EDE1 8BF0                    mov esi, eax  
:0041EDE3 C1EE16                  shr esi, 16  
:0041EDE6 C1E00A                  shl eax, 0A  
:0041EDE9 0BF0                    or esi, eax  
:0041EDEB 8BC1                    mov eax, ecx  
:0041EDED 03F2                    add esi, edx  
:0041EDEF F7D0                    not eax  
:0041EDF1 0BC6                    or eax, esi  
:0041EDF3 33C2                    xor eax, edx  
:0041EDF5 03C5                    add eax, ebp  
:0041EDF7 8B6C2418                mov ebp, dword ptr \[esp+18\]  
:0041EDFB 8D8407A72394AB          lea eax, dword ptr \[edi+eax\-546BDC59\]  
:0041EE02 8BF8                    mov edi, eax  
:0041EE04 C1EF11                  shr edi, 11  
:0041EE07 C1E00F                  shl eax, 0F  
:0041EE0A 0BF8                    or edi, eax  
:0041EE0C 8BC2                    mov eax, edx  
:0041EE0E 03FE                    add edi, esi  
:0041EE10 F7D0                    not eax  
:0041EE12 0BC7                    or eax, edi  
:0041EE14 33C6                    xor eax, esi  
:0041EE16 03C5                    add eax, ebp  
:0041EE18 8B6C241C                mov ebp, dword ptr \[esp+1C\]  
:0041EE1C 8D840139A093FC          lea eax, dword ptr \[ecx+eax\-036C5FC7\]  
:0041EE23 8BC8                    mov ecx, eax  
:0041EE25 C1E115                  shl ecx, 15  
:0041EE28 C1E80B                  shr eax, 0B  
:0041EE2B 0BC8                    or ecx, eax  
:0041EE2D 8BC6                    mov eax, esi  
:0041EE2F 03CF                    add ecx, edi  
:0041EE31 F7D0                    not eax  
:0041EE33 0BC1                    or eax, ecx  
:0041EE35 33C7                    xor eax, edi  
:0041EE37 03C5                    add eax, ebp  
:0041EE39 8D8402C3595B65          lea eax, dword ptr \[edx+eax+655B59C3\]  
:0041EE40 8BD0                    mov edx, eax  
:0041EE42 C1E006                  shl eax, 06  
:0041EE45 C1EA1A                  shr edx, 1A  
:0041EE48 8B6C2420                mov ebp, dword ptr \[esp+20\]  
:0041EE4C 0BD0                    or edx, eax  
:0041EE4E 8BC7                    mov eax, edi  
:0041EE50 03D1                    add edx, ecx  
:0041EE52 F7D0                    not eax  
:0041EE54 0BC2                    or eax, edx  
:0041EE56 33C1                    xor eax, ecx  
:0041EE58 03C5                    add eax, ebp  
:0041EE5A 8B6C2424                mov ebp, dword ptr \[esp+24\]  
:0041EE5E 8D840692CC0C8F          lea eax, dword ptr \[esi+eax\-70F3336E\]  
:0041EE65 8BF0                    mov esi, eax  
:0041EE67 C1E00A                  shl eax, 0A  
:0041EE6A C1EE16                  shr esi, 16  
:0041EE6D 0BF0                    or esi, eax  
:0041EE6F 8BC1                    mov eax, ecx  
:0041EE71 03F2                    add esi, edx  
:0041EE73 F7D0                    not eax  
:0041EE75 0BC6                    or eax, esi  
:0041EE77 33C2                    xor eax, edx  
:0041EE79 03C5                    add eax, ebp  
:0041EE7B 8B6C2428                mov ebp, dword ptr \[esp+28\]  
:0041EE7F 8D84077DF4EFFF          lea eax, dword ptr \[edi+eax\-00100B83\]  
:0041EE86 8BF8                    mov edi, eax  
:0041EE88 C1EF11                  shr edi, 11  
:0041EE8B C1E00F                  shl eax, 0F  
:0041EE8E 0BF8                    or edi, eax  
:0041EE90 8BC2                    mov eax, edx  
:0041EE92 03FE                    add edi, esi  
:0041EE94 F7D0                    not eax  
:0041EE96 0BC7                    or eax, edi  
:0041EE98 33C6                    xor eax, esi  
:0041EE9A 03C5                    add eax, ebp  
:0041EE9C 8B6C242C                mov ebp, dword ptr \[esp+2C\]  
:0041EEA0 8D8401D15D8485          lea eax, dword ptr \[ecx+eax\-7A7BA22F\]  
:0041EEA7 8BC8                    mov ecx, eax  
:0041EEA9 C1E115                  shl ecx, 15  
:0041EEAC C1E80B                  shr eax, 0B  
:0041EEAF 0BC8                    or ecx, eax  
:0041EEB1 8BC6                    mov eax, esi  
:0041EEB3 03CF                    add ecx, edi  
:0041EEB5 F7D0                    not eax  
:0041EEB7 0BC1                    or eax, ecx  
:0041EEB9 33C7                    xor eax, edi  
:0041EEBB 03C5                    add eax, ebp  
:0041EEBD 8B6C2448                mov ebp, dword ptr \[esp+48\]  
:0041EEC1 8D84024F7EA86F          lea eax, dword ptr \[edx+eax+6FA87E4F\]  
:0041EEC8 8BD0                    mov edx, eax  
:0041EECA C1EA1A                  shr edx, 1A  
:0041EECD C1E006                  shl eax, 06  
:0041EED0 0BD0                    or edx, eax  
:0041EED2 8BC7                    mov eax, edi  
:0041EED4 03D1                    add edx, ecx  
:0041EED6 F7D0                    not eax  
:0041EED8 0BC2                    or eax, edx  
:0041EEDA 33C1                    xor eax, ecx  
:0041EEDC 03C5                    add eax, ebp  
:0041EEDE 8B6C2430                mov ebp, dword ptr \[esp+30\]  
:0041EEE2 8DB406E0E62CFE          lea esi, dword ptr \[esi+eax\-01D31920\]  
:0041EEE9 8BC6                    mov eax, esi  
:0041EEEB C1E816                  shr eax, 16  
:0041EEEE C1E60A                  shl esi, 0A  
:0041EEF1 0BC6                    or eax, esi  
:0041EEF3 8BF1                    mov esi, ecx  
:0041EEF5 03C2                    add eax, edx  
:0041EEF7 F7D6                    not esi  
:0041EEF9 0BF0                    or esi, eax  
:0041EEFB 33F2                    xor esi, edx  
:0041EEFD 03F5                    add esi, ebp  
:0041EEFF 8B6C2434                mov ebp, dword ptr \[esp+34\]  
:0041EF03 8DBC37144301A3          lea edi, dword ptr \[edi+esi\-5CFEBCEC\]  
:0041EF0A 8BF7                    mov esi, edi  
:0041EF0C C1EE11                  shr esi, 11  
:0041EF0F C1E70F                  shl edi, 0F  
:0041EF12 0BF7                    or esi, edi  
:0041EF14 8BFA                    mov edi, edx  
:0041EF16 03F0                    add esi, eax  
:0041EF18 F7D7                    not edi  
:0041EF1A 0BFE                    or edi, esi  
:0041EF1C 33F8                    xor edi, eax  
:0041EF1E 03FD                    add edi, ebp  
:0041EF20 8B6C2438                mov ebp, dword ptr \[esp+38\]  
:0041EF24 8DBC39A111084E          lea edi, dword ptr \[ecx+edi+4E0811A1\]  
:0041EF2B 8BCF                    mov ecx, edi  
:0041EF2D C1E115                  shl ecx, 15  
:0041EF30 C1EF0B                  shr edi, 0B  
:0041EF33 0BCF                    or ecx, edi  
:0041EF35 8BF8                    mov edi, eax  
:0041EF37 03CE                    add ecx, esi  
:0041EF39 F7D7                    not edi  
:0041EF3B 0BF9                    or edi, ecx  
:0041EF3D 33FE                    xor edi, esi  
:0041EF3F 03FD                    add edi, ebp  
:0041EF41 8B6C243C                mov ebp, dword ptr \[esp+3C\]  
:0041EF45 8DBC3A827E53F7          lea edi, dword ptr \[edx+edi\-08AC817E\]  
:0041EF4C 8BD7                    mov edx, edi  
:0041EF4E C1EA1A                  shr edx, 1A  
:0041EF51 C1E706                  shl edi, 06  
:0041EF54 0BD7                    or edx, edi  
:0041EF56 8BFE                    mov edi, esi  
:0041EF58 03D1                    add edx, ecx  
:0041EF5A F7D7                    not edi  
:0041EF5C 0BFA                    or edi, edx  
:0041EF5E 33F9                    xor edi, ecx  
:0041EF60 03FD                    add edi, ebp  
:0041EF62 8B6C2440                mov ebp, dword ptr \[esp+40\]  
:0041EF66 8DBC3835F23ABD          lea edi, dword ptr \[eax+edi\-42C50DCB\]  
:0041EF6D 8BC7                    mov eax, edi  
:0041EF6F C1E816                  shr eax, 16  
:0041EF72 C1E70A                  shl edi, 0A  
:0041EF75 0BC7                    or eax, edi  
:0041EF77 8BF9                    mov edi, ecx  
:0041EF79 03C2                    add eax, edx  
:0041EF7B F7D7                    not edi  
:0041EF7D 0BF8                    or edi, eax  
:0041EF7F 33FA                    xor edi, edx  
:0041EF81 03FD                    add edi, ebp  
:0041EF83 8B6C2444                mov ebp, dword ptr \[esp+44\]  
:0041EF87 8DB43EBBD2D72A          lea esi, dword ptr \[esi+edi+2AD7D2BB\]  
:0041EF8E 8BFE                    mov edi, esi  
:0041EF90 C1EF11                  shr edi, 11  
:0041EF93 C1E60F                  shl esi, 0F  
:0041EF96 0BFE                    or edi, esi  
:0041EF98 8BF2                    mov esi, edx  
:0041EF9A 03F8                    add edi, eax  
:0041EF9C F7D6                    not esi  
:0041EF9E 0BF7                    or esi, edi  
:0041EFA0 33F0                    xor esi, eax  
:0041EFA2 03F5                    add esi, ebp  
:0041EFA4 8D8C3191D386EB          lea ecx, dword ptr \[ecx+esi\-14792C6F\]  
:0041EFAB 8B33                    mov esi, dword ptr \[ebx\]  
:0041EFAD 03F2                    add esi, edx  
:0041EFAF 8BD1                    mov edx, ecx  
:0041EFB1 C1E215                  shl edx, 15  
:0041EFB4 C1E90B                  shr ecx, 0B  
:0041EFB7 0BD1                    or edx, ecx  
:0041EFB9 8B4B08                  mov ecx, dword ptr \[ebx+08\]  
:0041EFBC 8933                    mov dword ptr \[ebx\], esi  
:0041EFBE 8B7304                  mov esi, dword ptr \[ebx+04\]  
:0041EFC1 03CF                    add ecx, edi  
:0041EFC3 03D6                    add edx, esi  
:0041EFC5 894B08                  mov dword ptr \[ebx+08\], ecx  
:0041EFC8 8B4B0C                  mov ecx, dword ptr \[ebx+0C\]  
:0041EFCB 03D7                    add edx, edi  
:0041EFCD 03C8                    add ecx, eax  
:0041EFCF 5F                      pop edi  
:0041EFD0 5E                      pop esi  
:0041EFD1 895304                  mov dword ptr \[ebx+04\], edx  
:0041EFD4 894B0C                  mov dword ptr \[ebx+0C\], ecx  
:0041EFD7 5D                      pop ebp  
:0041EFD8 5B                      pop ebx  
:0041EFD9 83C444                  add esp, 00000044  
:0041EFDC C20800                  ret 0008  
  
//\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
  
    第六次调用 call eax 会来到如下地方：  
:0041BD30 81EC04040000            sub esp, 00000404  
:0041BD36 53                      push ebx  
:0041BD37 8BD9                    mov ebx, ecx  
:0041BD39 56                      push esi  
:0041BD3A 57                      push edi  
:0041BD3B C783B405000060B54100    mov dword ptr \[ebx+000005B4\], 0041B560  <--送下一个检查子程序的地址  
:0041BD45 8DB348050000            lea esi, dword ptr \[ebx+00000548\]  
  
\* Possible Reference to String Resource ID=00003: "  
\*"  
                                  |  
:0041BD4B BF03000000              mov edi, 00000003  <--设置循环次数为 3  
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:0041BD9F(C)  
|  
:0041BD50 8B8344050000            mov eax, dword ptr \[ebx+00000544\]  
:0041BD56 6A00                    push 00000000  
:0041BD58 8A08                    mov cl, byte ptr \[eax\]  
:0041BD5A 40                      inc eax  
:0041BD5B 51                      push ecx  
:0041BD5C 8BCE                    mov ecx, esi  
:0041BD5E 898344050000            mov dword ptr \[ebx+00000544\], eax  
:0041BD64 E827280000              call 0041E590  <--这个 call 比对第 3、5、7 个字符  
:0041BD69 85C0                    test eax, eax  
:0041BD6B 751D                    jne 0041BD8A  <--不应该跳转  
:0041BD6D 8B8344050000            mov eax, dword ptr \[ebx+00000544\]  
:0041BD73 6A00                    push 00000000  
:0041BD75 8A08                    mov cl, byte ptr \[eax\]  
:0041BD77 40                      inc eax  
:0041BD78 51                      push ecx  
:0041BD79 8BCE                    mov ecx, esi  
:0041BD7B 898344050000            mov dword ptr \[ebx+00000544\], eax  
:0041BD81 E80A280000              call 0041E590  <--这个 call 比对第 4、6、8 个字符  
:0041BD86 85C0                    test eax, eax  
:0041BD88 7414                    je 0041BD9E  <--应该跳转  
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:0041BD6B(C)  
|  
:0041BD8A C783B405000020A94100    mov dword ptr \[ebx+000005B4\], 0041A920  <--送注册失败处理子程序的地址  
  
\* Possible Reference to String Resource ID=00001: "萣  
\*"  
                                  |  
:0041BD94 C783B805000001000000    mov dword ptr \[ebx+000005B8\], 00000001  <--把注册失败值送全局变量 reg  
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:0041BD88(C)  
|  
:0041BD9E 4F                      dec edi  
:0041BD9F 75AF                    jne 0041BD50  
:0041BDA1 55                      push ebp  
:0041BDA2 8DAB40010000            lea ebp, dword ptr \[ebx+00000140\]  
:0041BDA8 8BFD                    mov edi, ebp  
:0041BDAA 83C9FF                  or ecx, FFFFFFFF  
:0041BDAD 33C0                    xor eax, eax  
:0041BDAF 8D542414                lea edx, dword ptr \[esp+14\]  
:0041BDB3 F2                      repnz  
:0041BDB4 AE                      scasb  
:0041BDB5 F7D1                    not ecx  
:0041BDB7 2BF9                    sub edi, ecx  
:0041BDB9 8BC1                    mov eax, ecx  
:0041BDBB 8BF7                    mov esi, edi  
:0041BDBD 8BFA                    mov edi, edx  
:0041BDBF C1E902                  shr ecx, 02  
:0041BDC2 F3                      repz  
:0041BDC3 A5                      movsd  
:0041BDC4 8BC8                    mov ecx, eax  
:0041BDC6 83E103                  and ecx, 00000003  
:0041BDC9 F3                      repz  
:0041BDCA A4                      movsb  
:0041BDCB 8D4C2414                lea ecx, dword ptr \[esp+14\]  
:0041BDCF 51                      push ecx  
:0041BDD0 E81C430100              call 004300F1  
  
\* Possible StringData Ref from Data Obj ->"下\*"  <--该字符串解密后为"PC'"  
                                  |  
:0041BDD5 BF10474500              mov edi, 00454710  
:0041BDDA 83C9FF                  or ecx, FFFFFFFF  
:0041BDDD 33C0                    xor eax, eax  
:0041BDDF 83C404                  add esp, 00000004  
:0041BDE2 F2                      repnz  
:0041BDE3 AE                      scasb  
:0041BDE4 F7D1                    not ecx  
:0041BDE6 2BF9                    sub edi, ecx  
:0041BDE8 8D93D4060000            lea edx, dword ptr \[ebx+000006D4\]  
:0041BDEE 8BC1                    mov eax, ecx  
:0041BDF0 8BF7                    mov esi, edi  
:0041BDF2 8BFA                    mov edi, edx  
:0041BDF4 8BD0                    mov edx, eax  
:0041BDF6 8BC7                    mov eax, edi  
:0041BDF8 C1E902                  shr ecx, 02  
:0041BDFB F3                      repz  
:0041BDFC A5                      movsd  
:0041BDFD 8BCA                    mov ecx, edx  
:0041BDFF 83E103                  and ecx, 00000003  
:0041BE02 F3                      repz  
:0041BE03 A4                      movsb  
:0041BE04 8A08                    mov cl, byte ptr \[eax\]  
:0041BE06 84C9                    test cl, cl  
:0041BE08 740D                    je 0041BE17  
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:0041BE15(C)  
|  
:0041BE0A 80E97F                  sub cl, 7F  <--加密字符串的解密算法  
:0041BE0D 8808                    mov byte ptr \[eax\], cl  
:0041BE0F 8A4801                  mov cl, byte ptr \[eax+01\]  
:0041BE12 40                      inc eax  
:0041BE13 84C9                    test cl, cl  
:0041BE15 75F3                    jne 0041BE0A  
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:0041BE08(C)  
|  
:0041BE17 8D83D4060000            lea eax, dword ptr \[ebx+000006D4\]  
:0041BE1D 50                      push eax  
:0041BE1E 8D442418                lea eax, dword ptr \[esp+18\]  
:0041BE22 50                      push eax  
:0041BE23 E8A8590000              call 004217D0  <--检查注册文件中是否有黑名单中的"PC'"  
:0041BE28 83C408                  add esp, 00000008  
:0041BE2B 85C0                    test eax, eax  
:0041BE2D 7414                    je 0041BE43  <--应该跳转  
:0041BE2F C783B405000020A94100    mov dword ptr \[ebx+000005B4\], 0041A920  <--送注册失败处理子程序的地址  
  
\* Possible Reference to String Resource ID=00001: "萣  
\*"  
                                  |  
:0041BE39 C783B805000001000000    mov dword ptr \[ebx+000005B8\], 00000001  <--把注册失败值送全局变量 reg  
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:0041BE2D(C)  
|  
:0041BE43 8BFD                    mov edi, ebp  
:0041BE45 83C9FF                  or ecx, FFFFFFFF  
:0041BE48 33C0                    xor eax, eax  
:0041BE4A 8D542414                lea edx, dword ptr \[esp+14\]  
:0041BE4E F2                      repnz  
:0041BE4F AE                      scasb  
:0041BE50 F7D1                    not ecx  
:0041BE52 2BF9                    sub edi, ecx  
:0041BE54 8BC1                    mov eax, ecx  
:0041BE56 8BF7                    mov esi, edi  
:0041BE58 8BFA                    mov edi, edx  
:0041BE5A C1E902                  shr ecx, 02  
:0041BE5D F3                      repz  
:0041BE5E A5                      movsd  
:0041BE5F 8BC8                    mov ecx, eax  
:0041BE61 83E103                  and ecx, 00000003  
:0041BE64 F3                      repz  
:0041BE65 A4                      movsb  
:0041BE66 8D4C2414                lea ecx, dword ptr \[esp+14\]  
:0041BE6A 51                      push ecx  
:0041BE6B E881420100              call 004300F1  
  
\* Possible StringData Ref from Data Obj ->"子难\*"  <--该字符串解密后为"XTERM"  
                                  |  
:0041BE70 BF08474500              mov edi, 00454708  
:0041BE75 83C9FF                  or ecx, FFFFFFFF  
:0041BE78 33C0                    xor eax, eax  
:0041BE7A 83C404                  add esp, 00000004  
:0041BE7D F2                      repnz  
:0041BE7E AE                      scasb  
:0041BE7F F7D1                    not ecx  
:0041BE81 2BF9                    sub edi, ecx  
:0041BE83 5D                      pop ebp  
:0041BE84 8BF7                    mov esi, edi  
:0041BE86 8BD1                    mov edx, ecx  
:0041BE88 8DBBD4060000            lea edi, dword ptr \[ebx+000006D4\]  
:0041BE8E 8BC7                    mov eax, edi  
:0041BE90 C1E902                  shr ecx, 02  
:0041BE93 F3                      repz  
:0041BE94 A5                      movsd  
:0041BE95 8BCA                    mov ecx, edx  
:0041BE97 83E103                  and ecx, 00000003  
:0041BE9A F3                      repz  
:0041BE9B A4                      movsb  
:0041BE9C 8A08                    mov cl, byte ptr \[eax\]  
:0041BE9E 84C9                    test cl, cl  
:0041BEA0 740D                    je 0041BEAF  
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:0041BEAD(C)  
|  
:0041BEA2 80E97F                  sub cl, 7F  <--加密字符串的解密算法  
:0041BEA5 8808                    mov byte ptr \[eax\], cl  
:0041BEA7 8A4801                  mov cl, byte ptr \[eax+01\]  
:0041BEAA 40                      inc eax  
:0041BEAB 84C9                    test cl, cl  
:0041BEAD 75F3                    jne 0041BEA2  
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:0041BEA0(C)  
|  
:0041BEAF 8D83D4060000            lea eax, dword ptr \[ebx+000006D4\]  
:0041BEB5 50                      push eax  
:0041BEB6 8D442414                lea eax, dword ptr \[esp+14\]  
:0041BEBA 50                      push eax  
:0041BEBB E810590000              call 004217D0  <--检查注册文件中是否有黑名单中的"XTERM"  
:0041BEC0 83C408                  add esp, 00000008  
:0041BEC3 85C0                    test eax, eax  
:0041BEC5 7414                    je 0041BEDB  <--应该跳转  
:0041BEC7 C783B405000020A94100    mov dword ptr \[ebx+000005B4\], 0041A920  <--送注册失败处理子程序的地址  
  
\* Possible Reference to String Resource ID=00001: "萣  
\*"  
                                  |  
:0041BED1 C783B805000001000000    mov dword ptr \[ebx+000005B8\], 00000001  <--把注册失败值送全局变量 reg  
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:0041BEC5(C)  
|  
:0041BEDB 5F                      pop edi  
:0041BEDC 5E                      pop esi  
:0041BEDD 5B                      pop ebx  
:0041BEDE 81C404040000            add esp, 00000404  
:0041BEE4 C3                      ret  
  
//\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
  
    第七次调用 call eax 会来到如下地方：  
:0041B560 81EC04040000            sub esp, 00000404  
:0041B566 53                      push ebx  
:0041B567 8BD9                    mov ebx, ecx  
:0041B569 56                      push esi  
:0041B56A C783B405000040B74100    mov dword ptr \[ebx+000005B4\], 0041B740  <--送下一个检查子程序的地址  
:0041B574 8B8344050000            mov eax, dword ptr \[ebx+00000544\]  
:0041B57A 8DB348050000            lea esi, dword ptr \[ebx+00000548\]  
:0041B580 6A00                    push 00000000  
:0041B582 8A08                    mov cl, byte ptr \[eax\]  
:0041B584 40                      inc eax  
:0041B585 51                      push ecx  
:0041B586 8BCE                    mov ecx, esi  
:0041B588 898344050000            mov dword ptr \[ebx+00000544\], eax  
:0041B58E E8FD2F0000              call 0041E590  <--这个 call 比对第 9 个字符  
:0041B593 85C0                    test eax, eax  
:0041B595 751D                    jne 0041B5B4  <--不应该跳转  
:0041B597 8B8344050000            mov eax, dword ptr \[ebx+00000544\]  
:0041B59D 6A00                    push 00000000  
:0041B59F 8A08                    mov cl, byte ptr \[eax\]  
:0041B5A1 40                      inc eax  
:0041B5A2 51                      push ecx  
:0041B5A3 8BCE                    mov ecx, esi  
:0041B5A5 898344050000            mov dword ptr \[ebx+00000544\], eax  
:0041B5AB E8E02F0000              call 0041E590  <--这个 call 比对第 10 个字符  
:0041B5B0 85C0                    test eax, eax  
:0041B5B2 7414                    je 0041B5C8  <--应该跳转  
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:0041B595(C)  
|  
:0041B5B4 C783B405000020A94100    mov dword ptr \[ebx+000005B4\], 0041A920  <--送注册失败处理子程序的地址  
  
\* Possible Reference to String Resource ID=00001: "萣  
\*"  
                                  |  
:0041B5BE C783B805000001000000    mov dword ptr \[ebx+000005B8\], 00000001  <--把注册失败值送全局变量 reg  
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:0041B5B2(C)  
|  
:0041B5C8 55                      push ebp  
:0041B5C9 8DAB40010000            lea ebp, dword ptr \[ebx+00000140\]  
:0041B5CF 57                      push edi  
:0041B5D0 8BFD                    mov edi, ebp  
:0041B5D2 83C9FF                  or ecx, FFFFFFFF  
:0041B5D5 33C0                    xor eax, eax  
:0041B5D7 F2                      repnz  
:0041B5D8 AE                      scasb  
:0041B5D9 F7D1                    not ecx  
:0041B5DB 2BF9                    sub edi, ecx  
:0041B5DD 8D542414                lea edx, dword ptr \[esp+14\]  
:0041B5E1 8BC1                    mov eax, ecx  
:0041B5E3 8BF7                    mov esi, edi  
:0041B5E5 8BFA                    mov edi, edx  
:0041B5E7 C1E902                  shr ecx, 02  
:0041B5EA F3                      repz  
:0041B5EB A5                      movsd  
:0041B5EC 8BC8                    mov ecx, eax  
:0041B5EE 83E103                  and ecx, 00000003  
:0041B5F1 F3                      repz  
:0041B5F2 A4                      movsb  
:0041B5F3 8D4C2414                lea ecx, dword ptr \[esp+14\]  
:0041B5F7 51                      push ecx  
:0041B5F8 E8F44A0100              call 004300F1  
  
\* Possible StringData Ref from Data Obj ->"晌虑耐熐囊夷"  <--该字符串解密后为"JOCHEN HESSE"  
                                  |  
:0041B5FD BF7C464500              mov edi, 0045467C  
:0041B602 83C9FF                  or ecx, FFFFFFFF  
:0041B605 33C0                    xor eax, eax  
:0041B607 83C404                  add esp, 00000004  
:0041B60A F2                      repnz  
:0041B60B AE                      scasb  
:0041B60C F7D1                    not ecx  
:0041B60E 2BF9                    sub edi, ecx  
:0041B610 8D93D4060000            lea edx, dword ptr \[ebx+000006D4\]  
:0041B616 8BC1                    mov eax, ecx  
:0041B618 8BF7                    mov esi, edi  
:0041B61A 8BFA                    mov edi, edx  
:0041B61C 8BD0                    mov edx, eax  
:0041B61E 8BC7                    mov eax, edi  
:0041B620 C1E902                  shr ecx, 02  
:0041B623 F3                      repz  
:0041B624 A5                      movsd  
:0041B625 8BCA                    mov ecx, edx  
:0041B627 83E103                  and ecx, 00000003  
:0041B62A F3                      repz  
:0041B62B A4                      movsb  
:0041B62C 8A08                    mov cl, byte ptr \[eax\]  
:0041B62E 84C9                    test cl, cl  
:0041B630 740D                    je 0041B63F  
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:0041B63D(C)  
|  
:0041B632 80E97F                  sub cl, 7F  <--加密字符串的解密算法  
:0041B635 8808                    mov byte ptr \[eax\], cl  
:0041B637 8A4801                  mov cl, byte ptr \[eax+01\]  
:0041B63A 40                      inc eax  
:0041B63B 84C9                    test cl, cl  
:0041B63D 75F3                    jne 0041B632  
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:0041B630(C)  
|  
:0041B63F 8D83D4060000            lea eax, dword ptr \[ebx+000006D4\]  
:0041B645 50                      push eax  
:0041B646 8D442418                lea eax, dword ptr \[esp+18\]  
:0041B64A 50                      push eax  
:0041B64B E880610000              call 004217D0  <--检查注册文件中是否有黑名单中的"JOCHEN HESSE"  
:0041B650 83C408                  add esp, 00000008  
:0041B653 85C0                    test eax, eax  
:0041B655 7414                    je 0041B66B  <--应该跳转  
:0041B657 C783B405000020A94100    mov dword ptr \[ebx+000005B4\], 0041A920  <--送注册失败处理子程序的地址  
  
\* Possible Reference to String Resource ID=00001: "萣  
\*"  
                                  |  
:0041B661 C783B805000001000000    mov dword ptr \[ebx+000005B8\], 00000001  <--把注册失败值送全局变量 reg  
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:0041B655(C)  
|  
:0041B66B 8BFD                    mov edi, ebp  
:0041B66D 83C9FF                  or ecx, FFFFFFFF  
:0041B670 33C0                    xor eax, eax  
:0041B672 8D542414                lea edx, dword ptr \[esp+14\]  
:0041B676 F2                      repnz  
:0041B677 AE                      scasb  
:0041B678 F7D1                    not ecx  
:0041B67A 2BF9                    sub edi, ecx  
:0041B67C 8BC1                    mov eax, ecx  
:0041B67E 8BF7                    mov esi, edi  
:0041B680 8BFA                    mov edi, edx  
:0041B682 C1E902                  shr ecx, 02  
:0041B685 F3                      repz  
:0041B686 A5                      movsd  
:0041B687 8BC8                    mov ecx, eax  
:0041B689 83E103                  and ecx, 00000003  
:0041B68C F3                      repz  
:0041B68D A4                      movsb  
:0041B68E 8D4C2414                lea ecx, dword ptr \[esp+14\]  
:0041B692 51                      push ecx  
:0041B693 E8594A0100              call 004300F1  
  
\* Possible StringData Ref from Data Obj ->"卵缆\*"  <--该字符串解密后为"CRACK"  
                                  |  
:0041B698 BF74464500              mov edi, 00454674  
:0041B69D 83C9FF                  or ecx, FFFFFFFF  
:0041B6A0 33C0                    xor eax, eax  
:0041B6A2 83C404                  add esp, 00000004  
:0041B6A5 F2                      repnz  
:0041B6A6 AE                      scasb  
:0041B6A7 F7D1                    not ecx  
:0041B6A9 2BF9                    sub edi, ecx  
:0041B6AB 8BF7                    mov esi, edi  
:0041B6AD 8BD1                    mov edx, ecx  
:0041B6AF 8DBBD4060000            lea edi, dword ptr \[ebx+000006D4\]  
:0041B6B5 8BC7                    mov eax, edi  
:0041B6B7 C1E902                  shr ecx, 02  
:0041B6BA F3                      repz  
:0041B6BB A5                      movsd  
:0041B6BC 8BCA                    mov ecx, edx  
:0041B6BE 83E103                  and ecx, 00000003  
:0041B6C1 F3                      repz  
:0041B6C2 A4                      movsb  
:0041B6C3 8A08                    mov cl, byte ptr \[eax\]  
:0041B6C5 5F                      pop edi  
:0041B6C6 84C9                    test cl, cl  
:0041B6C8 5D                      pop ebp  
:0041B6C9 740D                    je 0041B6D8  
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:0041B6D6(C)  
|  
:0041B6CB 80E97F                  sub cl, 7F  <--加密字符串的解密算法  
:0041B6CE 8808                    mov byte ptr \[eax\], cl  
:0041B6D0 8A4801                  mov cl, byte ptr \[eax+01\]  
:0041B6D3 40                      inc eax  
:0041B6D4 84C9                    test cl, cl  
:0041B6D6 75F3                    jne 0041B6CB  
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:0041B6C9(C)  
|  
:0041B6D8 8D83D4060000            lea eax, dword ptr \[ebx+000006D4\]  
:0041B6DE 50                      push eax  
:0041B6DF 8D442410                lea eax, dword ptr \[esp+10\]  
:0041B6E3 50                      push eax  
:0041B6E4 E8E7600000              call 004217D0  <--检查注册文件中是否有黑名单中的"CRACK"  
:0041B6E9 83C408                  add esp, 00000008  
:0041B6EC 85C0                    test eax, eax  
:0041B6EE 7414                    je 0041B704  <--应该跳转  
:0041B6F0 C783B405000020A94100    mov dword ptr \[ebx+000005B4\], 0041A920  <--送注册失败处理子程序的地址  
  
\* Possible Reference to String Resource ID=00001: "萣  
\*"  
                                  |  
:0041B6FA C783B805000001000000    mov dword ptr \[ebx+000005B8\], 00000001  <--把注册失败值送全局变量 reg  
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:0041B6EE(C)  
|  
:0041B704 5E                      pop esi  
:0041B705 5B                      pop ebx  
:0041B706 81C404040000            add esp, 00000404  
:0041B70C C3                      ret  
  
//\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
  
    第八次调用 call eax 会来到如下地方：  
:0041B740 81EC04040000            sub esp, 00000404  
:0041B746 53                      push ebx  
:0041B747 8BD9                    mov ebx, ecx  
:0041B749 56                      push esi  
:0041B74A 57                      push edi  
:0041B74B C783B405000060B34100    mov dword ptr \[ebx+000005B4\], 0041B360  <--送下一个检查子程序的地址  
:0041B755 8DB348050000            lea esi, dword ptr \[ebx+00000548\]  
  
\* Possible Reference to String Resource ID=00002: "分  
忈 o"  
                                  |  
:0041B75B BF02000000              mov edi, 00000002  <--设置循环次数为 2  
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:0041B7AF(C)  
|  
:0041B760 8B8344050000            mov eax, dword ptr \[ebx+00000544\]  
:0041B766 6A00                    push 00000000  
:0041B768 8A08                    mov cl, byte ptr \[eax\]  
:0041B76A 40                      inc eax  
:0041B76B 51                      push ecx  
:0041B76C 8BCE                    mov ecx, esi  
:0041B76E 898344050000            mov dword ptr \[ebx+00000544\], eax  
:0041B774 E8172E0000              call 0041E590  <--这个 call 比对第 11、13 个字符  
:0041B779 85C0                    test eax, eax  
:0041B77B 751D                    jne 0041B79A  <--不应该跳转  
:0041B77D 8B8344050000            mov eax, dword ptr \[ebx+00000544\]  
:0041B783 6A00                    push 00000000  
:0041B785 8A08                    mov cl, byte ptr \[eax\]  
:0041B787 40                      inc eax  
:0041B788 51                      push ecx  
:0041B789 8BCE                    mov ecx, esi  
:0041B78B 898344050000            mov dword ptr \[ebx+00000544\], eax  
:0041B791 E8FA2D0000              call 0041E590  <--这个 call 比对第 12、14 个字符  
:0041B796 85C0                    test eax, eax  
:0041B798 7414                    je 0041B7AE  <--应该跳转  
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:0041B77B(C)  
|  
:0041B79A C783B405000020A94100    mov dword ptr \[ebx+000005B4\], 0041A920  <--送注册失败处理子程序的地址  
  
\* Possible Reference to String Resource ID=00001: "萣  
\*"  
                                  |  
:0041B7A4 C783B805000001000000    mov dword ptr \[ebx+000005B8\], 00000001  <--把注册失败值送全局变量 reg  
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:0041B798(C)  
|  
:0041B7AE 4F                      dec edi  
:0041B7AF 75AF                    jne 0041B760  
:0041B7B1 55                      push ebp  
:0041B7B2 8DAB40010000            lea ebp, dword ptr \[ebx+00000140\]  
:0041B7B8 8BFD                    mov edi, ebp  
:0041B7BA 83C9FF                  or ecx, FFFFFFFF  
:0041B7BD 33C0                    xor eax, eax  
:0041B7BF 8D542414                lea edx, dword ptr \[esp+14\]  
:0041B7C3 F2                      repnz  
:0041B7C4 AE                      scasb  
:0041B7C5 F7D1                    not ecx  
:0041B7C7 2BF9                    sub edi, ecx  
:0041B7C9 8BC1                    mov eax, ecx  
:0041B7CB 8BF7                    mov esi, edi  
:0041B7CD 8BFA                    mov edi, edx  
:0041B7CF C1E902                  shr ecx, 02  
:0041B7D2 F3                      repz  
:0041B7D3 A5                      movsd  
:0041B7D4 8BC8                    mov ecx, eax  
:0041B7D6 83E103                  and ecx, 00000003  
:0041B7D9 F3                      repz  
:0041B7DA A4                      movsb  
:0041B7DB 8D4C2414                lea ecx, dword ptr \[esp+14\]  
:0041B7DF 51                      push ecx  
:0041B7E0 E80C490100              call 004300F1  
  
\* Possible StringData Ref from Data Obj ->"嗜怂难"  <--该字符串解密后为"KILLER"  
                                  |  
:0041B7E5 BFA0464500              mov edi, 004546A0  
:0041B7EA 83C9FF                  or ecx, FFFFFFFF  
:0041B7ED 33C0                    xor eax, eax  
:0041B7EF 83C404                  add esp, 00000004  
:0041B7F2 F2                      repnz  
:0041B7F3 AE                      scasb  
:0041B7F4 F7D1                    not ecx  
:0041B7F6 2BF9                    sub edi, ecx  
:0041B7F8 8D93D4060000            lea edx, dword ptr \[ebx+000006D4\]  
:0041B7FE 8BC1                    mov eax, ecx  
:0041B800 8BF7                    mov esi, edi  
:0041B802 8BFA                    mov edi, edx  
:0041B804 8BD0                    mov edx, eax  
:0041B806 8BC7                    mov eax, edi  
:0041B808 C1E902                  shr ecx, 02  
:0041B80B F3                      repz  
:0041B80C A5                      movsd  
:0041B80D 8BCA                    mov ecx, edx  
:0041B80F 83E103                  and ecx, 00000003  
:0041B812 F3                      repz  
:0041B813 A4                      movsb  
:0041B814 8A08                    mov cl, byte ptr \[eax\]  
:0041B816 84C9                    test cl, cl  
:0041B818 740D                    je 0041B827  
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:0041B825(C)  
|  
:0041B81A 80E97F                  sub cl, 7F  <--加密字符串的解密算法  
:0041B81D 8808                    mov byte ptr \[eax\], cl  
:0041B81F 8A4801                  mov cl, byte ptr \[eax+01\]  
:0041B822 40                      inc eax  
:0041B823 84C9                    test cl, cl  
:0041B825 75F3                    jne 0041B81A  
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:0041B818(C)  
|  
:0041B827 8D83D4060000            lea eax, dword ptr \[ebx+000006D4\]  
:0041B82D 50                      push eax  
:0041B82E 8D442418                lea eax, dword ptr \[esp+18\]  
:0041B832 50                      push eax  
:0041B833 E8985F0000              call 004217D0  <--检查注册文件中是否有黑名单中的"KILLER"  
:0041B838 83C408                  add esp, 00000008  
:0041B83B 85C0                    test eax, eax  
:0041B83D 7414                    je 0041B853  <--应该跳转  
:0041B83F C783B405000020A94100    mov dword ptr \[ebx+000005B4\], 0041A920  <--送注册失败处理子程序的地址  
  
\* Possible Reference to String Resource ID=00001: "萣  
\*"  
                                  |  
:0041B849 C783B805000001000000    mov dword ptr \[ebx+000005B8\], 00000001  <--把注册失败值送全局变量 reg  
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:0041B83D(C)  
|  
:0041B853 6A00                    push 00000000  
:0041B855 6880000000              push 00000080  
  
\* Possible Reference to String Resource ID=00003: "  
\*"  
                                  |  
:0041B85A 6A03                    push 00000003  
:0041B85C 6A00                    push 00000000  
  
\* Possible Reference to String Resource ID=00003: "  
\*"  
                                  |  
:0041B85E 6A03                    push 00000003  
:0041B860 68000000C0              push C0000000  
  
\* Possible StringData Ref from Data Obj ->"\\\\.\\SICE"  
                                  |  
:0041B865 6894464500              push 00454694  
  
\* Reference To: KERNEL32.CreateFileA, Ord:0034h  
                                  |  
:0041B86A FF1538724400            Call dword ptr \[00447238\]  <--通过调用 API 函数 CreateFileA 检查是否正在运行 SICE，还好我们还有国人的骄傲——TRW2000  
:0041B870 83F8FF                  cmp eax, FFFFFFFF  
:0041B873 741B                    je 0041B890  <--应该跳转  
:0041B875 50                      push eax  
  
\* Reference To: KERNEL32.CloseHandle, Ord:001Bh  
                                  |  
:0041B876 FF154C724400            Call dword ptr \[0044724C\]  
:0041B87C C783B405000020A94100    mov dword ptr \[ebx+000005B4\], 0041A920  <--送注册失败处理子程序的地址  
  
\* Possible Reference to String Resource ID=00001: "萣  
\*"  
                                  |  
:0041B886 C783B805000001000000    mov dword ptr \[ebx+000005B8\], 00000001  <--把注册失败值送全局变量 reg  
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:0041B873(C)  
|  
:0041B890 8BFD                    mov edi, ebp  
:0041B892 83C9FF                  or ecx, FFFFFFFF  
:0041B895 33C0                    xor eax, eax  
:0041B897 8D542414                lea edx, dword ptr \[esp+14\]  
:0041B89B F2                      repnz  
:0041B89C AE                      scasb  
:0041B89D F7D1                    not ecx  
:0041B89F 2BF9                    sub edi, ecx  
:0041B8A1 8BC1                    mov eax, ecx  
:0041B8A3 8BF7                    mov esi, edi  
:0041B8A5 8BFA                    mov edi, edx  
:0041B8A7 C1E902                  shr ecx, 02  
:0041B8AA F3                      repz  
:0041B8AB A5                      movsd  
:0041B8AC 8BC8                    mov ecx, eax  
:0041B8AE 83E103                  and ecx, 00000003  
:0041B8B1 F3                      repz  
:0041B8B2 A4                      movsb  
:0041B8B3 8D4C2414                lea ecx, dword ptr \[esp+14\]  
:0041B8B7 51                      push ecx  
:0041B8B8 E834480100              call 004300F1  
  
\* Possible StringData Ref from Data Obj ->"虑琅\*"  <--该字符串解密后为"CHAFE"  
                                  |  
:0041B8BD BF8C464500              mov edi, 0045468C  
:0041B8C2 83C9FF                  or ecx, FFFFFFFF  
:0041B8C5 33C0                    xor eax, eax  
:0041B8C7 83C404                  add esp, 00000004  
:0041B8CA F2                      repnz  
:0041B8CB AE                      scasb  
:0041B8CC F7D1                    not ecx  
:0041B8CE 2BF9                    sub edi, ecx  
:0041B8D0 5D                      pop ebp  
:0041B8D1 8BF7                    mov esi, edi  
:0041B8D3 8BD1                    mov edx, ecx  
:0041B8D5 8DBBD4060000            lea edi, dword ptr \[ebx+000006D4\]  
:0041B8DB 8BC7                    mov eax, edi  
:0041B8DD C1E902                  shr ecx, 02  
:0041B8E0 F3                      repz  
:0041B8E1 A5                      movsd  
:0041B8E2 8BCA                    mov ecx, edx  
:0041B8E4 83E103                  and ecx, 00000003  
:0041B8E7 F3                      repz  
:0041B8E8 A4                      movsb  
:0041B8E9 8A08                    mov cl, byte ptr \[eax\]  
:0041B8EB 84C9                    test cl, cl  
:0041B8ED 740D                    je 0041B8FC  
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:0041B8FA(C)  
|  
:0041B8EF 80E97F                  sub cl, 7F  <--加密字符串的解密算法  
:0041B8F2 8808                    mov byte ptr \[eax\], cl  
:0041B8F4 8A4801                  mov cl, byte ptr \[eax+01\]  
:0041B8F7 40                      inc eax  
:0041B8F8 84C9                    test cl, cl  
:0041B8FA 75F3                    jne 0041B8EF  
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:0041B8ED(C)  
|  
:0041B8FC 8D83D4060000            lea eax, dword ptr \[ebx+000006D4\]  
:0041B902 50                      push eax  
:0041B903 8D442414                lea eax, dword ptr \[esp+14\]  
:0041B907 50                      push eax  
:0041B908 E8C35E0000              call 004217D0  <--检查注册文件中是否有黑名单中的"CHAFE"  
:0041B90D 83C408                  add esp, 00000008  
:0041B910 85C0                    test eax, eax  
:0041B912 7414                    je 0041B928  <--应该跳转  
:0041B914 C783B405000020A94100    mov dword ptr \[ebx+000005B4\], 0041A920  <--送注册失败处理子程序的地址  
  
\* Possible Reference to String Resource ID=00001: "萣  
\*"  
                                  |  
:0041B91E C783B805000001000000    mov dword ptr \[ebx+000005B8\], 00000001  <--把注册失败值送全局变量 reg  
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:0041B912(C)  
|  
:0041B928 5F                      pop edi  
:0041B929 5E                      pop esi  
:0041B92A 5B                      pop ebx  
:0041B92B 81C404040000            add esp, 00000404  
:0041B931 C3                      ret  
  
//\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
  
    第九次调用 call eax 会来到如下地方：  
:0041B360 81EC00040000            sub esp, 00000400  
:0041B366 53                      push ebx  
:0041B367 8BD9                    mov ebx, ecx  
:0041B369 56                      push esi  
:0041B36A C783B405000040B94100    mov dword ptr \[ebx+000005B4\], 0041B940  <--送下一个检查子程序的地址  
:0041B374 8B8344050000            mov eax, dword ptr \[ebx+00000544\]  
:0041B37A 8DB348050000            lea esi, dword ptr \[ebx+00000548\]  
:0041B380 6A00                    push 00000000  
:0041B382 8A08                    mov cl, byte ptr \[eax\]  
:0041B384 40                      inc eax  
:0041B385 51                      push ecx  
:0041B386 8BCE                    mov ecx, esi  
:0041B388 898344050000            mov dword ptr \[ebx+00000544\], eax  
:0041B38E E8FD310000              call 0041E590  <--这个 call 比对第 15 个字符  
:0041B393 85C0                    test eax, eax  
:0041B395 751D                    jne 0041B3B4  <--不应该跳转  
:0041B397 8B8344050000            mov eax, dword ptr \[ebx+00000544\]  
:0041B39D 6A00                    push 00000000  
:0041B39F 8A08                    mov cl, byte ptr \[eax\]  
:0041B3A1 40                      inc eax  
:0041B3A2 51                      push ecx  
:0041B3A3 8BCE                    mov ecx, esi  
:0041B3A5 898344050000            mov dword ptr \[ebx+00000544\], eax  
:0041B3AB E8E0310000              call 0041E590  <--这个 call 比对第 16 个字符  
:0041B3B0 85C0                    test eax, eax  
:0041B3B2 7414                    je 0041B3C8  <--应该跳转  
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:0041B395(C)  
|  
:0041B3B4 C783B405000020A94100    mov dword ptr \[ebx+000005B4\], 0041A920  <--送注册失败处理子程序的地址  
  
\* Possible Reference to String Resource ID=00001: "萣  
\*"  
                                  |  
:0041B3BE C783B805000001000000    mov dword ptr \[ebx+000005B8\], 00000001  <--把注册失败值送全局变量 reg  
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:0041B3B2(C)  
|  
:0041B3C8 55                      push ebp  
:0041B3C9 57                      push edi  
:0041B3CA 8DBB40010000            lea edi, dword ptr \[ebx+00000140\]  
:0041B3D0 83C9FF                  or ecx, FFFFFFFF  
:0041B3D3 33C0                    xor eax, eax  
:0041B3D5 8D542410                lea edx, dword ptr \[esp+10\]  
:0041B3D9 F2                      repnz  
:0041B3DA AE                      scasb  
:0041B3DB F7D1                    not ecx  
:0041B3DD 2BF9                    sub edi, ecx  
:0041B3DF 8BC1                    mov eax, ecx  
:0041B3E1 8BF7                    mov esi, edi  
:0041B3E3 8BFA                    mov edi, edx  
:0041B3E5 C1E902                  shr ecx, 02  
:0041B3E8 F3                      repz  
:0041B3E9 A5                      movsd  
:0041B3EA 8BC8                    mov ecx, eax  
:0041B3EC 83E103                  and ecx, 00000003  
:0041B3EF F3                      repz  
:0041B3F0 A4                      movsb  
:0041B3F1 8D4C2410                lea ecx, dword ptr \[esp+10\]  
:0041B3F5 51                      push ecx  
:0041B3F6 E8F64C0100              call 004300F1  
  
\* Possible StringData Ref from Data Obj ->"恃缆\*"  <--该字符串解密后为"KRACK"  
                                  |  
:0041B3FB BF6C464500              mov edi, 0045466C  
:0041B400 83C9FF                  or ecx, FFFFFFFF  
:0041B403 33C0                    xor eax, eax  
:0041B405 83C404                  add esp, 00000004  
:0041B408 F2                      repnz  
:0041B409 AE                      scasb  
:0041B40A F7D1                    not ecx  
:0041B40C 2BF9                    sub edi, ecx  
:0041B40E 8DABD4060000            lea ebp, dword ptr \[ebx+000006D4\]  
:0041B414 8BF7                    mov esi, edi  
:0041B416 8BD1                    mov edx, ecx  
:0041B418 8BFD                    mov edi, ebp  
:0041B41A 8BC7                    mov eax, edi  
:0041B41C C1E902                  shr ecx, 02  
:0041B41F F3                      repz  
:0041B420 A5                      movsd  
:0041B421 8BCA                    mov ecx, edx  
:0041B423 83E103                  and ecx, 00000003  
:0041B426 F3                      repz  
:0041B427 A4                      movsb  
:0041B428 8A08                    mov cl, byte ptr \[eax\]  
:0041B42A 84C9                    test cl, cl  
:0041B42C 740D                    je 0041B43B  
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:0041B439(C)  
|  
:0041B42E 80E97F                  sub cl, 7F  <--加密字符串的解密算法  
:0041B431 8808                    mov byte ptr \[eax\], cl  
:0041B433 8A4801                  mov cl, byte ptr \[eax+01\]  
:0041B436 40                      inc eax  
:0041B437 84C9                    test cl, cl  
:0041B439 75F3                    jne 0041B42E  
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:0041B42C(C)  
|  
:0041B43B 8D442410                lea eax, dword ptr \[esp+10\]  
:0041B43F 55                      push ebp  
:0041B440 50                      push eax  
:0041B441 E88A630000              call 004217D0  <--检查注册文件中是否有黑名单中的"KRACK"  
:0041B446 83C408                  add esp, 00000008  
:0041B449 85C0                    test eax, eax  
:0041B44B 5F                      pop edi  
:0041B44C 5D                      pop ebp  
:0041B44D 7414                    je 0041B463  <--应该跳转  
:0041B44F C783B405000020A94100    mov dword ptr \[ebx+000005B4\], 0041A920  <--送注册失败处理子程序的地址  
  
\* Possible Reference to String Resource ID=00001: "萣  
\*"  
                                  |  
:0041B459 C783B805000001000000    mov dword ptr \[ebx+000005B8\], 00000001  <--把注册失败值送全局变量 reg  
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:0041B44D(C)  
|  
:0041B463 5E                      pop esi  
:0041B464 5B                      pop ebx  
:0041B465 81C400040000            add esp, 00000400  
:0041B46B C3                      ret  
  
//\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
  
    第十次调用 call eax 会来到如下地方：  
:0041B940 83EC10                  sub esp, 00000010  
:0041B943 53                      push ebx  
:0041B944 55                      push ebp  
:0041B945 8BE9                    mov ebp, ecx  
:0041B947 56                      push esi  
:0041B948 57                      push edi  
:0041B949 6A3D                    push 0000003D  
:0041B94B 8D8540010000            lea eax, dword ptr \[ebp+00000140\]  
:0041B951 33FF                    xor edi, edi  
:0041B953 50                      push eax  
:0041B954 C685D006000000          mov byte ptr \[ebp+000006D0\], 00  
:0041B95B E8005F0000              call 00421860  <--查找第一个"="字符  
:0041B960 83C408                  add esp, 00000008  
:0041B963 8BF0                    mov esi, eax  
:0041B965 89BDC8060000            mov dword ptr \[ebp+000006C8\], edi  
:0041B96B C785B4050000F0BE4100    mov dword ptr \[ebp+000005B4\], 0041BEF0  <--送下一个检查子程序的地址  
:0041B975 3BF7                    cmp esi, edi  
:0041B977 0F84C2010000            je 0041BB3F  
:0041B97D 83C602                  add esi, 00000002  
  
\* Possible Reference to String Resource ID=00010: ";×谿: = %lu"  
                                  |  
:0041B980 6A0A                    push 0000000A  
:0041B982 56                      push esi  
:0041B983 E8D85E0000              call 00421860  
:0041B988 8BD8                    mov ebx, eax  
:0041B98A 83C408                  add esp, 00000008  
:0041B98D 2BDE                    sub ebx, esi  
:0041B98F 8DBDBC050000            lea edi, dword ptr \[ebp+000005BC\]  
:0041B995 4B                      dec ebx  
:0041B996 8D95D4060000            lea edx, dword ptr \[ebp+000006D4\]  
:0041B99C 8BCB                    mov ecx, ebx  
:0041B99E 8BC1                    mov eax, ecx  
:0041B9A0 C1E902                  shr ecx, 02  
:0041B9A3 F3                      repz  
:0041B9A4 A5                      movsd  
:0041B9A5 8BC8                    mov ecx, eax  
:0041B9A7 33C0                    xor eax, eax  
:0041B9A9 83E103                  and ecx, 00000003  
:0041B9AC F3                      repz  
:0041B9AD A4                      movsb  
  
\* Possible StringData Ref from Data Obj ->"镊镨皲蚬\*"  <--该字符串解密后为"Expires:"  
                                  |  
:0041B9AE BFFC464500              mov edi, 004546FC  
:0041B9B3 83C9FF                  or ecx, FFFFFFFF  
:0041B9B6 F2                      repnz  
:0041B9B7 AE                      scasb  
:0041B9B8 F7D1                    not ecx  
:0041B9BA 2BF9                    sub edi, ecx  
:0041B9BC 8BC1                    mov eax, ecx  
:0041B9BE 8BF7                    mov esi, edi  
:0041B9C0 8BFA                    mov edi, edx  
:0041B9C2 89442410                mov dword ptr \[esp+10\], eax  
:0041B9C6 8BC7                    mov eax, edi  
:0041B9C8 C1E902                  shr ecx, 02  
:0041B9CB F3                      repz  
:0041B9CC A5                      movsd  
:0041B9CD 8B4C2410                mov ecx, dword ptr \[esp+10\]  
:0041B9D1 83E103                  and ecx, 00000003  
:0041B9D4 F3                      repz  
:0041B9D5 A4                      movsb  
:0041B9D6 8A08                    mov cl, byte ptr \[eax\]  
:0041B9D8 84C9                    test cl, cl  
:0041B9DA 740D                    je 0041B9E9  
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:0041B9E7(C)  
|  
:0041B9DC 80E97F                  sub cl, 7F  <--加密字符串的解密算法  
:0041B9DF 8808                    mov byte ptr \[eax\], cl  
:0041B9E1 8A4801                  mov cl, byte ptr \[eax+01\]  
:0041B9E4 40                      inc eax  
:0041B9E5 84C9                    test cl, cl  
:0041B9E7 75F3                    jne 0041B9DC  
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:0041B9DA(C)  
|  
:0041B9E9 8D95D4060000            lea edx, dword ptr \[ebp+000006D4\]  
:0041B9EF 83C9FF                  or ecx, FFFFFFFF  
:0041B9F2 8BFA                    mov edi, edx  
:0041B9F4 33C0                    xor eax, eax  
:0041B9F6 F2                      repnz  
:0041B9F7 AE                      scasb  
:0041B9F8 F7D1                    not ecx  
:0041B9FA 49                      dec ecx  
:0041B9FB 8BFA                    mov edi, edx  
:0041B9FD 8DB5BC050000            lea esi, dword ptr \[ebp+000005BC\]  
:0041BA03 33C0                    xor eax, eax  
:0041BA05 894C2410                mov dword ptr \[esp+10\], ecx  
:0041BA09 F3                      repz  <--这条指令和下一条检查注册文件中是否含有字符串"Expires: "  
:0041BA0A A6                      cmpsb  
:0041BA0B 0F85B4000000            jne 0041BAC5  <--应该跳转  
:0041BA11 8D4C241C                lea ecx, dword ptr \[esp+1C\]  
:0041BA15 8D542418                lea edx, dword ptr \[esp+18\]  
:0041BA19 51                      push ecx  
:0041BA1A 8B4C2414                mov ecx, dword ptr \[esp+14\]  
:0041BA1E 8D442418                lea eax, dword ptr \[esp+18\]  
:0041BA22 52                      push edx  
:0041BA23 50                      push eax  
:0041BA24 8D9429BC050000          lea edx, dword ptr \[ecx+ebp+000005BC\]  
  
\* Possible StringData Ref from Data Obj ->"%2d/%2d/%4d"  <--试用期所到日期(月,日,年)  
                                  |  
:0041BA2B 68F0464500              push 004546F0  
:0041BA30 52                      push edx  
:0041BA31 E8774F0000              call 004209AD  
:0041BA36 8B442428                mov eax, dword ptr \[esp+28\]  
:0041BA3A 8B4C242C                mov ecx, dword ptr \[esp+2C\]  
:0041BA3E 8B542430                mov edx, dword ptr \[esp+30\]  
:0041BA42 83C414                  add esp, 00000014  
:0041BA45 25FFFF0000              and eax, 0000FFFF  
:0041BA4A 81E1FFFF0000            and ecx, 0000FFFF  
:0041BA50 6A00                    push 00000000  
:0041BA52 6A00                    push 00000000  
:0041BA54 6A00                    push 00000000  
:0041BA56 50                      push eax  
:0041BA57 81E2FFFF0000            and edx, 0000FFFF  
:0041BA5D 51                      push ecx  
:0041BA5E 52                      push edx  
:0041BA5F 8D8DBC060000            lea ecx, dword ptr \[ebp+000006BC\]  
:0041BA65 E89E440000              call 0041FF08  
:0041BA6A 8B442410                mov eax, dword ptr \[esp+10\]  
:0041BA6E 6A2D                    push 0000002D  
  
\* Possible Reference to String Resource ID=00001: "萣  
\*"  
                                  |  
:0041BA70 C785C806000001000000    mov dword ptr \[ebp+000006C8\], 00000001  
:0041BA7A 8D8C28C6050000          lea ecx, dword ptr \[eax+ebp+000005C6\]  
:0041BA81 51                      push ecx  
:0041BA82 E8D95D0000              call 00421860  
:0041BA87 83C408                  add esp, 00000008  
:0041BA8A 85C0                    test eax, eax  
:0041BA8C 741D                    je 0041BAAB  
:0041BA8E 83C002                  add eax, 00000002  
:0041BA91 668B10                  mov dx, word ptr \[eax\]  
:0041BA94 668995D0060000          mov word ptr \[ebp+000006D0\], dx  
:0041BA9B 8A4002                  mov al, byte ptr \[eax+02\]  
:0041BA9E C685D306000000          mov byte ptr \[ebp+000006D3\], 00  
:0041BAA5 8885D2060000            mov byte ptr \[ebp+000006D2\], al  
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:0041BA8C(C)  
|  
:0041BAAB C785B405000090AD4100    mov dword ptr \[ebp+000005B4\], 0041AD90  
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:0041BB1A(C)  
|  
:0041BAB5 5F                      pop edi  
:0041BAB6 C6842BBC05000000        mov byte ptr \[ebx+ebp+000005BC\], 00  
:0041BABE 5E                      pop esi  
:0041BABF 5D                      pop ebp  
:0041BAC0 5B                      pop ebx  
:0041BAC1 83C410                  add esp, 00000010  
:0041BAC4 C3                      ret  
  
  
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:0041BA0B(C)  
|  
  
\* Possible StringData Ref from Data Obj ->"ⅱⅱ熦铘熰皲熗斡燂漶扈篌溷燇顭镟蝌燇玷驘巳履鸵"  
                                        ->"臒彖脘燇顭囗鵁铙玟癍ⅱⅱ"  <--该字符串解密后为"#### You are NOT permitted to pass this LICENSE file to any other.####"  
                                  |  
:0041BAC5 BFA8464500              mov edi, 004546A8  
:0041BACA 83C9FF                  or ecx, FFFFFFFF  
:0041BACD 33C0                    xor eax, eax  
:0041BACF F2                      repnz  
:0041BAD0 AE                      scasb  
:0041BAD1 F7D1                    not ecx  
:0041BAD3 2BF9                    sub edi, ecx  
:0041BAD5 8BC1                    mov eax, ecx  
:0041BAD7 8BF7                    mov esi, edi  
:0041BAD9 8BFA                    mov edi, edx  
:0041BADB 8944241C                mov dword ptr \[esp+1C\], eax  
:0041BADF 8BC7                    mov eax, edi  
:0041BAE1 C1E902                  shr ecx, 02  
:0041BAE4 F3                      repz  
:0041BAE5 A5                      movsd  
:0041BAE6 8B4C241C                mov ecx, dword ptr \[esp+1C\]  
:0041BAEA 83E103                  and ecx, 00000003  
:0041BAED F3                      repz  
:0041BAEE A4                      movsb  
:0041BAEF 8A08                    mov cl, byte ptr \[eax\]  
:0041BAF1 84C9                    test cl, cl  
:0041BAF3 740D                    je 0041BB02  
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:0041BB00(C)  
|  
:0041BAF5 80E97F                  sub cl, 7F  <--加密字符串的解密算法  
:0041BAF8 8808                    mov byte ptr \[eax\], cl  
:0041BAFA 8A4801                  mov cl, byte ptr \[eax+01\]  
:0041BAFD 40                      inc eax  
:0041BAFE 84C9                    test cl, cl  
:0041BB00 75F3                    jne 0041BAF5  
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:0041BAF3(C)  
|  
:0041BB02 8D85D4060000            lea eax, dword ptr \[ebp+000006D4\]  
:0041BB08 50                      push eax  
:0041BB09 8D8540010000            lea eax, dword ptr \[ebp+00000140\]  
:0041BB0F 50                      push eax  
:0041BB10 E8BB5C0000              call 004217D0  <--检查注册文件中是否含有字符串"#### You are NOT permitted to pass this LICENSE file to any other.####"  
:0041BB15 83C408                  add esp, 00000008  
:0041BB18 85C0                    test eax, eax  
:0041BB1A 7599                    jne 0041BAB5  <--应该跳转  
:0041BB1C C785B405000020A94100    mov dword ptr \[ebp+000005B4\], 0041A920  <--送注册失败处理子程序的地址  
  
\* Possible Reference to String Resource ID=00001: "萣  
\*"  
                                  |  
:0041BB26 C785B805000001000000    mov dword ptr \[ebp+000005B8\], 00000001  <--把注册失败值送全局变量 reg  
:0041BB30 5F                      pop edi  
:0041BB31 88842BBC050000          mov byte ptr \[ebx+ebp+000005BC\], al  
:0041BB38 5E                      pop esi  
:0041BB39 5D                      pop ebp  
:0041BB3A 5B                      pop ebx  
:0041BB3B 83C410                  add esp, 00000010  
:0041BB3E C3                      ret  
  
//\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
    最后，我总结一下注册算法：  
    程序先检查软件所在目录下是否含有注册文件 GRDuw.key，然后检查文件长度是否不为 0 并且是否大于 220 字节，接着检查注册文件中的前五个字符是否  
为"GRDuw"，并且检查第六个字符是否为空格符，然后再把注册文件中的注册码前面的部分（包括换行符和回车符）进行 MD5 计算，把得出的 128 位结果的前  
64 位与后 64 位进行异或运算，检查注册码的前 16 个字符是否与这个 64 位十六进制结果转换成的字符串是否相等，在上一步检查中顺带检查注册文件中是否  
含有下列黑名单："PC'"、"XTERM"、"JOCHEN HESSE"、"CRACK"、"KILLER"、"CHAFE"、"KRACK"，另外如果文件中有"Expires:"字符串的话就会把后面的  
32 位整数作为许可文件的过期日期，最后检查文件中是否含有字符串（该字符串和上面的黑名单可以在注册文件中除最开始的任何地方）：  
"#### You are NOT permitted to pass this LICENSE file to any other.####"。  
    至此，我想大家应该可以做出一个自己的 KEYFILE 了，希望大家也能写出一个 KEYFILE MAKER（注册文件生成器）。  
  
    啊，终于写完了，写心得真是太……~!@#$%^&\*()…… 

* * *

 4dd466d0e1e0cc123afbe504da2462ca