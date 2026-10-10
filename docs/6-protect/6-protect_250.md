

# Solidedge

> 原文：[`www.pediy.com/kssd/pediy05/pediy50518.htm`](https://www.pediy.com/kssd/pediy05/pediy50518.htm)

UGFLEX——传奇在继续   
作者：macilaci  翻译：上官振宁   
需要的工具   
Debugger (SICE, OllyDbg－－非常棒的东东)，Wadsm 或者 IDA，Flexlm SDK 7.2 或者更高版本（非必须），一些最近的信息（从 Pilgrim, Nolan Blender 或其他的地方得到）。   
目标位置（对付得东东是个 CAD 软件，叫做 Solidedge。）   
http://www.ugsolutions.com   
http://www.solid-edge.com   
正文   
经过最初的嗅听之后，发现了读取 license 文件(selicense.dat)的 DLL。也许我们并没有创建 License 文件，所以 License Key 也没有，它是这样的：   
FEATURE SOLIDEDGECLASSIC sedemon 11.0 permanent uncounted \\   
    123456789ABC HOSTID=ANY ISSUER="I don't know"   
我们的 ugflex.dll 被 jutil.dll 替换。还有个 MAP 文件，格式如下：   
Address        Publics by Value              Rva+Base    Lib:Object   
0001:00000010  \_JSetVACallback@4              5a001010 f    auto.obj   
0001:00000040  ?IsAutomationEnabled@@YAHXZ    5a001040 f    auto.obj   
0001:00000050  ?IsLicenceValidForAutomation@@YAHXZ 5a001050 auto.obj   
  
为了与 idasym.exe 文件一起使用这个 MAP 文件，将 Rva+Base 还有 Lib:Object 列删除。然   
后将创建的 sym 文件与 symbol loader 一起使用（设断点易如反掌，接着往下看就明白了）   
  
由经验得知，实际上对于 License 的检查是在 LC\_CHECKOUT 进行的（摘自 flexlm 手册）   
  
lc\_checkout()   
SYNTAX  status = lc\_checkout( job ,feature ,version ,num\_lic ,flag ,code , dup\_group )   
说明：查出一个或多个具体的 feature(不好翻译，姑且用 e 文)。如果调用 lc\_checkout()的程序存在，那么 check 出的 license 将返回供另一用户使用。   
    如果将上面函数中的 flag 设置成 LM\_CO\_WAIT，那么这个程序将等待，一直到达到这个 feature 需要的 license 数目。License 文件的版本必须等于或高于 lc\_checkout 调用中的 version 参数。   
    If the license file is counted, that is, if the number of users specified on the FEATURE line is non-zero, lc\_checkout() will request the license from alicense server. If the number of users on the FEATURE line is uncounted, it will grant permission based on the contents of the license file only&#352;hostid,version, expiration date, etc.（感觉没什么用，没翻译）   
  
在 5a04cf70 处的\_lc\_checkout 是设断点的好地方！找到了 FEATURE 名后，就可以按照上面的格式建立自己的 license 文件了^\_^。为了保证正确性，给出所有的 FEATURE 名：SOLIDEDGECLASSIC, SOLIDEDGEADVANCEDPAR, SOLIDEDGEXPRESROUTE, SOLIDEDGEFEATURERECO, SOLIDEDGEXPAND3D,   
SEWEBPUBLISHER, SOLIDEDGEHANDBOOK, SOLIDEDGEMANAGER（note：有些 feature 如 expand3d 要在别的可执行程序中采用的到，以后再说^\_^）   
再来看 l\_good\_lic\_key 函数。\_lc\_checkout 命中三次而\_l\_good\_lic\_key 只有一次。挺败吧？下面是程序片断：   
5A04DA6C loc\_5A04DA6C:              ;CODE XREF \_lm\_start\_real+343j   
5A04DA6C                mov    edx, \[ebp+arg\_14\]   
5A04DA6F                push    edx            ; vendorcode 结构(这次有所改进-加密了)   
5A04DA70                mov    eax, \[ebp+var\_20\]   
5A04DA73                push    eax   
5A04DA74                mov    ecx, \[ebp+arg\_0\]    ;我们的 license information   
5A04DA77                push    ecx   
5A04DA78                call    \_l\_good\_lic\_key    ;检查 license   
5A04DA7D                add    esp, 0Ch   
5A04DA80                test    eax, eax   
5A04DA82                jnz    short loc\_5A04DA89    ;这个你应该很熟悉了吧:-)   
  
如果运行 part.exe 而没有反应，可能是应为你没有运行 seiges.exe。part.exe 中有些反 debug 的代码的部分。运行 seiges.exe 并将 eax 置成非零，将得到 status=OK.但是仅仅有 patch 并不能让人满意，所以我们接着往下看^\_^   
  
5A04E995                add    edx, 54h   
.   
.   
5A04E99C                push    eax   
5A04E99D                call    \_l\_extract\_date   
.   
.   
5A04E9B1                push    ecx            ;我们的 vendorcode structure   
5A04E9B2                mov    edx, \[ebp+var\_CC\]   
5A04E9B8                push    edx          ;feature（特征名）   
5A04E9B9                mov    eax, \[ebp+arg\_4\]   
5A04E9BC                push    eax   
5A04E9BD                mov    ecx, \[ebp+arg\_0\]   
5A04E9C0                push    ecx   
5A04E9C1                call    \_l\_ckout\_crypt   
  
跟踪上面的代码我们发现\_l\_ckout\_crypt 做了一些重要的工作。这里只改变了 seed 的值而没有改变 key 的值。由于 key 的值没有改变，我们姑且认为他们已经被解密了，但 seed 的值仍然隐藏着！继续追踪，\_real\_crypt 出现了（败了，这么个败名字）   
5A050B44                mov    eax, \[ebp+arg\_0\]   
5A050B47                push    eax   
5A050B48                call    \_real\_crypt      
5A050B4D                add    esp, 10h   
  
下面是\_real\_crypt 的代码片断：   
  
.   
.   
5A050ECB                push    ecx   
5A050ECC                call    \_l\_getattr              ;取得 license attributes   
.   
.   
5A05106A                push    eax   
5A05106B                call    \_l\_good\_bin\_date        ; date   
.   
.   
5A0510DA                mov    edx, \[ebp+arg\_0\]   
5A0510DD                push    edx   
5A0510DE                call    \_move\_in\_hostid    ; 从 license 文件里获取 hostid   
.   
.   
5A051456                push    offset aDup\_group ; "DUP\_GROUP" ;DUP\_GROUP=UHD 意味着可能的分组为(DUP\_USER|DUP\_HOST|DUP\_DISPLAY),   
所以对于用户在一台主机上，另外的对于 feature 的使用不会销毁另外的 licenses. （看不太懂哦:(  )   
  
5A05145B                mov    eax, \[ebp+arg\_4\]   
5A05145E                mov    ecx, \[eax+94h\]   
5A051464                push    ecx   
5A051465                call    \_addi            ;把这个加到 license 中   
.   
.   
5A051855                push    ecx   
5A051856                call    \_l\_ckout\_string\_key            ;这个怎样？   
5A05185B                add    esp, 18h   
  
\_l\_ckout\_string\_key 值得注意！   
5A052902 \_l\_ckout\_string\_key proc near          ; CODE XREF: \_real\_crypt+AD1p   
  
  
5A052902                push    ebp   
5A052903                mov    ebp, esp   
5A052905                sub    esp, 1E4h   
.   
.   
.   
5A053755              call      \_our\_encrypt2            5A05375A                add    esp, 4   
5A05375D                jmp    short loc\_5A05376C   
5A05375F ; ΖΖΖΖΖΖΖΖΖΖΖΖΖΖΖΖΖΖΖΖΖΖΖΖΖΖΖΖΖΖΖΖΖΖΖΖΖ   
5A05375F   
5A05375F loc\_5A05375F:          ; CODE XREF: \_l\_ckout\_string\_key+E43j   
5A05375F                                        ; \_l\_ckout\_string\_key+E4Cj   
5A05375F                push    offset byte\_5A0C5820   
5A053764                call    \_our\_encrypt        ;the second encrypt   
.   
.   
.   
5A05384B                mov    ecx, \[ebp+var\_188\]   
5A053851                add    ecx, 1   
5A053854                mov    \[ebp+var\_188\], ecx   
5A05385A   
5A05385A loc\_5A05385A:          ; CODE XREF: \_l\_ckout\_string\_key+F47j   
5A05385A                mov    edx, \[ebp+var\_188\]   
5A053860                cmp    edx, \[ebp+var\_18C\]   
5A053866                jge    loc\_5A0539E0   
5A05386C                mov    eax, \[ebp+var\_188\]   
5A053872                mov    cl, \[ebp+eax\*2+var\_168\]   
5A053879                mov    \[ebp+var\_1CC\], cl   
5A05387F                call    ds:\_\_p\_\_\_mb\_cur\_max   
5A053885                cmp    dword ptr \[eax\], 1   
.   
.   
.   
5A0539CB                xor    eax, eax   
5A0539CD                mov    al, byte\_5A0C5820\[edx\]              ; 最终 - 比较 license   
5A0539D3                cmp    ecx, eax   
5A0539D5                jz      short loc\_5A0539DB              ;比较下一个 byte   
5A0539D7                xor    eax, eax                  ;错误的 key   
5A0539D9                jmp    short loc\_5A053A01                
5A0539DB ; ΖΖΖΖΖΖΖΖΖΖΖΖΖΖΖΖΖΖΖΖΖΖΖΖΖΖΖΖΖΖΖΖΖΖΖΖΖ   
5A0539DB   
5A0539DB loc\_5A0539DB:        ; CODE XREF: \_l\_ckout\_string\_key+10D3j   
5A0539DB                jmp    loc\_5A05384B        ;loop back   
  
上面的循环与版本 7 中的有些类似（现在是 7.2）。5A0C5820 是产生真正的 key 的地方！这里有必要说明一下 key 的产生：如果没有 license key，我们的 license（格式正确）是混乱的，两者好像是通过 addi function 函数联系到一起的。所以现在程序的任务就是计算 vendorseed,并在 4 个 vendorkey 的帮助下同混乱的 license 混合（可能是做某种运算^\_^）。上面说得工作将在 5A0C5820 处完成。在 5A0C5820 处 bpm，我们得到：   
  
.   
.   
5A053334                mov    byte\_5A0C5820\[ecx\], al    ;here we go   
5A05333A                jmp    short loc\_5A0532F9   
5A05333C loc\_5A05333C:          ; CODE XREF: \_l\_ckout\_string\_key+A0Dj   
5A05333C                cmp    \[ebp+var\_188\], 0   
  
从 5A05333C 处开始追踪，我们发现了计算 seed 的代码：   
  
5A0533EA                xor    eax, ebx        ;计算正确的 seed   
5A0533EC                push    eax            ;入栈   
5A0533ED                call    sub\_5A053AD3   
5A0533F2                add    esp, 4   
5A0533F5                mov    \[ebp+seed\_one\], eax   
5A0533FB                mov    edx, \[ebp+seed\_one\]   
5A053401                and    edx, 0FFh   
.   
.   
5A0534C9                add    eax, 1   
5A0534CC                mov    \[ebp+var\_10\], eax   
5A0534CF                mov    \[ebp+seed\_one\], 3D4DA1D6h  ;隐藏 seed   
.   
.   
5A053548                push    eax    ;第二个 seed 入栈   
5A053549                call    sub\_5A053AD3   
5A05354E                add    esp, 4   
5A053551                mov    \[ebp+seed\_two\], eax   
5A053557                mov    edx, \[ebp+seed\_two\]   
.   
.   
5A053628                mov    \[ebp+var\_10\], eax   
5A05362B                mov    \[ebp+seed\_two\], 3D4DA1D6h  ;隐藏 5A053635                jmp    loc\_5A053735   
  
将 seed 隐藏能很好的防止内存 dump 工具将 seed 解出来。解 seed 的过程只能在特定位置并且是运行时才可以！   
  
总结：   
LC\_CHECKOUT ---   
        |   
        |   
        \_l\_good\_lic\_key— /\*加密的 keys\*/   
                  |   
                  |   
                  \_l\_ckout\_crypt---/\*解密的 keys\*/   
                            |   
                            |   
                            \_real\_crypt---   
                                  |   
                                |   
                        \_l\_ckout\_string\_key---   
                        |   
                        |   
                5A0533EA  /\*计算第一个 seed\*/   
                        |   
                        |   
                5A053547  /\*计算第二个 seed\*/   
                        |   
                        |   
                5A0539DB <-   
                        |   
                        |   
                5A0539CD---  /\*license 比较\*/   
现在我们可以修改 lm\_code.h 并且编译 lmcrypt。使用 3D4DA1D6h 这样的字节模式，我们在 AddInlm.dll 中找类似的位置。其实我们只需要 LC\_CHECKOUT 作为 feature 名。使用其他的模式能得到相应的位置（对于没有 flexlm SDK 的朋友来说，比较有用）：   
jutil.dll          AddInlm.dll        Selicwiz.exe   
5A0533EC        046CAEDC         00428C6C   
.   
5A0539CD        046CB4BD         0042924D   
  
The origin of the above code is the object lm\_ckout.obj winthin the lmgr.lib library.（大体意思是上面的代码从那里来的）   
  
                                             |   
                                             |   
                                             5A0533EA  /\*计算第一个 seed\*/   
                                             |   
                                             5A053547  /\*计算第二个 seed\*/   
                                             |   
                                             5A0539DB <-                                                      
                                                 |      |   
                                                  5A0539CD---  /\*license 比较\*/   
                                             |   
                                            --   
翻译得不好，大虾莫笑:P  4dd466d0e1e0cc123afbe504da2462ca