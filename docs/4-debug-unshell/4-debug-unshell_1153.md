

# VFP&exe 2.0

> 原文：[`www.pediy.com/kssd/pediy05/pediy50208.htm`](https://www.pediy.com/kssd/pediy05/pediy50208.htm)

关于 VFP&exe2.0 的研究过程   
  
脱壳后:   
:0047248E 3B45D8                  cmp eax, dword ptr \[ebp\-28\]   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:0047248C(C)   
|   
:00472491 7411                    je 004724A4====>74 改 eb 跳过死循环   
:00472493 3B75F0                  cmp esi, dword ptr \[ebp\-10\]   
:00472496 7C0C                    jl 004724A4   
\*\*\*===========   
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:004724A2(C)   
|   
:00472498 8BC6                    mov eax, esi   
:0047249A 2BC3                    sub eax, ebx   
:0047249C 8945F0                  mov dword ptr \[ebp\-10\], eax   
:0047249F 3B75F0                  cmp esi, dword ptr \[ebp\-10\]   
:004724A2 7DF4                    jge 00472498   
\*\*\*===========此处为一死循环，是一个典型暗桩，就是它造成程序虽然正常脱壳但却不能运行。   
\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:   
|:00472491(C), :00472496(C)   
|   
:004724A4 8B45E0                  mov eax, dword ptr \[ebp\-20\]   
:004724A7 8945F8                  mov dword ptr \[ebp\-08\], eax   
:004724AA 8B45E4                  mov eax, dword ptr \[ebp\-1C\]   
:004724AD 8945FC                  mov dword ptr \[ebp\-04\], eax   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:   
|:00472332(C), :0047237B(U)   
|   
:004724B0 33C0                    xor eax, eax   
  
  
然后进一步破解:   
  
:004733AE E8E909F9FF              call 00403D9C   
:004733B3 7516                    jne 004733CB====>跳到注册版处   
:004733B5 B8E8A94700              mov eax, 0047A9E8   
:004733BA B962000000              mov ecx, 00000062   
  
\* Possible StringData Ref from Code Obj ->"ABCDEF20"   
                                  |   
:004733BF BAF8354700              mov edx, 004735F8   
:004733C4 E8DBF6FFFF              call 00472AA4   
:004733C9 EB64                    jmp 0047342F====>这一跳就成未注册版了   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:004733B3(C)   
|   
:004733CB 8D55EC                  lea edx, dword ptr \[ebp\-14\]   
    .   
    .   
    .   
:0047340D E8EE15FBFF              call 00424A00   
  
\* Possible StringData Ref from Code Obj ->"注册给"   
                                  |   
:00473412 BA0C364700              mov edx, 0047360C   
继续向下看:   
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:004733C9(U)   
|   
:0047342F 8B8330030000            mov eax, dword ptr \[ebx+00000330\]   
:00473435 83783400                cmp dword ptr \[eax+34\], 00000000   
:00473439 7512                    jne 0047344D===>相等则为专业版,不相等则为企业版   
  
\* Possible StringData Ref from Code Obj ->"专业版"   
                                  |   
:0047343B BA1C364700              mov edx, 0047361C   
:00473440 8B8320030000            mov eax, dword ptr \[ebx+00000320\]   
:00473446 E8F516FBFF              call 00424B40   
:0047344B EB10                    jmp 0047345D   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:00473439(C)   
|   
  
\* Possible StringData Ref from Code Obj ->"企业版"   
                                  |   
:0047344D BA2C364700              mov edx, 0047362C   
:00473452 8B8320030000            mov eax, dword ptr \[ebx+00000320\]   
:00473458 E8E316FBFF              call 00424B40   
  
  
upx+cryptor 壳的去除   
  
程序：老王的 vfp&exe2.0 演示版 2002.4.29 版   
下载地址：http://cfyn.yeah.net   
工具：importREC v1.4.2+, fi249, trw2000,bw2000,winhex   
fi249 报告该程序用 upx+cryptor 加密   
找入口:BW2000 报告为 476824-400000=076824   
运行 trw2000   
bpx 476824   
运行 vfpexe2.exe 断下后   
suspend   
运行 importRECua 并激活 vfpexe2   
入口改为 76824 点 IAT AotuSearch,点 Get Imports   
点右键->选 select code section(s)条,将 name 下的框全选   
点 Full dump 存盘保存   
用 winhex 调入脱壳后的文件,将入口点 09e000 修改为 076824   
运行脱壳后的文件......没反应???   
经用 trw2000 跟踪发现在 47248e 处有一陷阱   
47248e CMP EAX,\[EBP\-28\]   
记下\[EBP\-28\]处的值为 01f2fee26,eax\=03a8e3d6   
用 winhex 在程序中找 4A91F201 换为 B214A803 即大功告成   
至于演示版 700K 的限制有两处很好改，自己看着办吧。   
改演示版 700K 限制   
找：833D78A9470072   
第一处将 7E5D 改为 EB5d   
在 4738e3 处:   
第二处将 0F8E93000000 改为 90E993000000   
第二处陷阱：47433a B820A10700 MOV EAX,0007A120   
改为: 47433a B8203F0d00 MOV eax,000d3f20   
这样改后还须再次修改 3f0 处的数据   
trw2000 再次调入程序   
G 47248E   
EAX\=03a8e4c3   
将 3F0 处改为 C3E4A803   
  
刚下载的可能不是一个版本，入口点：476824   
  
脱壳前后密钥存放位置一样，没加密，在 03f0 处，此版本为 26EEF201   
比较的地方在 47248e CMP EAX,\[EBP\-28\]   
将 3F0 处的数据改为 EAX 中的数据(注意倒着放)如 EAX=03A8E3D6 改文件时应   
为 D6E3A803.   
  
小球的破解过程:   
  
我也不知道是否成功了，因为老王说演示版加密不完全。因为我没有注册版，所以就交给你作了，你可以用注册版和破解版对同一个软件加壳，然后用工具比较一下。哈哈哈。我想这个破解版应该不是完美的。   
下面是你要的简要的过程，也没有什么复杂的，只是要慢慢的试。   
经过一步一步的跟踪，我下断 473342，然后 d edx，可以看到我的序列号。那就再用 bpm \[序列号\]，很快发现有不少的 call 和 jz，jnz。只有慢慢试了，我没有什么好办法阿！而后发现 4735f8 里面放着 ABCEF20，这个数可以在 vfpexe2.exe 中找到，老王用这个数作了处理，有 98 次循环，我懒得看。处理后的数据放在 112c170 开始的一片空间里，而后我没有发现老王又对这个地址进行访问。不知道原因。而后发现了 4733ae 这个地址，是两个－1 在比较，在 vfpexe2.exe 中也可以找到，我试了用 r lf z，竟然成了注册版了。哈哈，其实也就这么简单。（但是不知道是否破解成功）而后发现了一个企业版的功能，在 473435 这个地址有个 cmp，我为了防止暗桩用 mov 给它付了个 1，但愿可以躲过以后的暗桩。就这样有可以变成企业版了。我想大概和专业版一样吧？哈哈。至于注册名吗？在 vfpexe2.exe 中也可以找到便宜量在 8dd47 处。   
好了，大概的过程就是这样的，现在看看我竟然没有修改 vfpexe.dll 文件。发现没有 vfpexe.dll 文件时依然是注册版，由此可见我的破解是 90%不成功的。因为老王是通过这个 dll 文件来注册的，我竟然没有用到？哈哈！  4dd466d0e1e0cc123afbe504da2462ca