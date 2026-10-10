

# 加壳标志

> 原文：[`www.pediy.com/kssd/pediy02/forum214.htm`](https://www.pediy.com/kssd/pediy02/forum214.htm)

 有关判断各种软件加壳标志的方法------在 PECompact 中找到的。   
  
作者：        该职位空缺，有意者可与本人联系。限额一名，MM 优先。   
写作日期：    2000-08-06   
  
【前言】   
在掌握了对 PECompact  v1.40b1 的脱壳后，一直没太注意这个软件。今天为了寻找一下该压缩加壳软件的加壳标志，就又重新找出来了。跟踪一遍才发现，程序编的简洁高效。（看来作者很了不起哦，有可能是用 Win32ASM 编写的，当然跟踪起来很方便了，不象跟踪有些软件，就象到垃圾场里去捡垃圾。）不过有意思的是该软件对其他压缩软件已经加壳过的程序不再加壳，哈哈，这样剩了我不少时间，不用一一去分析其他加壳软件的加壳标志了。从这里找就是了。（注：当然这种方法找出的标志不一定有用。因为加壳软件其自身的判断标准可能与此不同。这个软件说不定也是找一种偷懒的方法来判断的。和我一样 --- 哈哈，两个懒虫，一个不如一个。:-) ）   
  
以下是我找到的程序分析加壳标志的代码部分。我加了点注释。（也可能与实际有出入，懒得去验证了）   
  
●第一部分：主程序代码   
:0040385D 66813E5045              cmp word ptr \[esi\], 4550        <--这里当然是判断是否是可执行文件的 PE 标志了   
:00403862 0F85FF000000            jne 00403967                <--不同，那还有戏唱...   
:00403868 833DC9A3400001          cmp dword ptr \[0040A3C9\], 00000001   
:0040386F 740D                    je 0040387E   
:00403871 817E0C5045434F          cmp dword ptr \[esi+0C\], 4F434550    <--判断是否是 'PECO'，这是 PECompact 加壳的标志   
:00403878 0F841C010000            je 0040399A                <--相等，那还用说，自然去显示你的程序被加壳了。   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:0040386F(C)   
|   
:0040387E 56                      push esi   
:0040387F E8E42F0000              call 00406868                <--该调用测试是否被 NeoLite 加壳了   
:00403884 0F820E020000            jb 00403A98                <--小于 0，说明加壳了   
:0040388A 56                      push esi   
:0040388B E8B72F0000              call 00406847                <--该调用测试是否被 BitArts 加壳了   
:00403890 0F8228020000            jb 00403ABE                <--小于 0，说明加壳了   
:00403896 56                      push esi   
:00403897 E8762F0000              call 00406812                <--该调用测试.....自已看 E 文吧....The selected file appears to have already a symbiote of some type installed, it may or may not function after compression.   
:0040389C 0F8242020000            jb 00403AE4                <--小于 0，.....看懂了上面的话，自己来说吧！干嘛！   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:   
|:00403AE8(C), :00403B03(C)   
|   
:004038A2 0FB74614                movzx eax, word ptr \[esi+14\]       
:004038A6 83C018                  add eax, 00000018   
:004038A9 03C6                    add eax, esi                <--EAX 定位到 Section 的 Name 指针处   
:004038AB 81382E736872            cmp dword ptr \[eax\], 7268732E        <--判断是否是 '.shr'，测试是否被 Shrinker 加壳了   
:004038B1 0F8495010000            je 00403A4C                <--相等，以后我懒得说了   
:004038B7 813855505830            cmp dword ptr \[eax\], 30585055        <--判断是否是 'UPX0'，测试是否被 UPX 加壳了   
:004038BD 0F84AF010000            je 00403A72   
:004038C3 56                      push esi   
:004038C4 E8E02E0000              call 004067A9   
:004038C9 81382E575750            cmp dword ptr \[eax\], 5057572E        <--判断是否是 '.WWP'，测试是否被 WWPack32 加壳了   
:004038CF 0F8417010000            je 004039EC   
:004038D5 83E828                  sub eax, 00000028   
:004038D8 81382E617370            cmp dword ptr \[eax\], 7073612E        <--判断是否是 '.asp'，测试是否被 ASPack 加壳了   
:004038DE 0F8446010000            je 00403A2A   
:004038E4 81382E706B6C            cmp dword ptr \[eax\], 6C6B702E        <--判断是否是 '.pkl'，测试是否被 PKLite32 加壳了   
:004038EA 0F841B010000            je 00403A0B   
  
  
●第二部分：子程序代码部分   
  
①第一个子程序段   
\* Referenced by a CALL at Address:   
|:0040387F    
|   
:00406868 C8000000                enter 0000, 00   
:0040686C 60                      pushad   
:0040686D FF7508                  push \[ebp+08\]       
:00406870 E834FFFFFF              call 004067A9                <--该调用返回 Sections 的 Name 的指针到 EAX       
:00406875 81382E6E656F            cmp dword ptr \[eax\], 6F656E2E        <--判断是否是 '.neo'，这是加壳标志   
:0040687B 7406                    je 00406883                <--相等，加壳了   
:0040687D 61                      popad   
:0040687E F8                      clc   
:0040687F C9                      leave   
:00406880 C20400                  ret 0004   
  
②第二个子程序段   
\* Referenced by a CALL at Address:   
|:0040388B    
|   
:00406847 C8000000                enter 0000, 00   
:0040684B 60                      pushad   
:0040684C FF7508                  push \[ebp+08\]   
:0040684F E855FFFFFF              call 004067A9                <--该调用返回 Sections 的 Name 的指针到 EAX   
:00406854 813842697441            cmp dword ptr \[eax\], 41746942        <--判断是否是 'BitA'，这是 BitArts 加壳标志   
:0040685A 7406                    je 00406862   
:0040685C 61                      popad   
:0040685D F8                      clc   
:0040685E C9                      leave   
:0040685F C20400                  ret 0004   
  
③跳转后的显示部分，这还用去解释吗？   
:00403A98 837D1001                cmp dword ptr \[ebp+10\], 00000001   
:00403A9C 0F8515FFFFFF            jne 004039B7   
:0040399A 837D1001                cmp dword ptr \[ebp+10\], 00000001   
:0040399E 7517                    jne 004039B7   
:004039A0 6A10                    push 00000010   
  
\* Possible StringData Ref from Code Obj ->"PECompact v1.40b1, "   
                                  |   
:004039A2 6808B24000              push 0040B208   
  
\* Possible StringData Ref from Code Obj ->"File appears to already be compressed."   
                                  |   
:004039A7 68BBB24000              push 0040B2BB   
:004039AC FF351CB94000            push dword ptr \[0040B91C\]   
:004039B2 E8CD090000              call 00404384   
  
【后记】   
我们已经知道了 PECompact 判断的方法，修改跳转就很简单了。我用 Notepad.exe 加壳后测试了一下，加一道壳，有三个 Sections，加两道壳，又加了二个 Section，以后再加壳，就没加 Section 了。yi,是不是 PECompact 开始只做表面功夫了，我一下加了八道壳，开始跟踪，结果在内存中跳来跳去的，我头也跟晕了....。又加了几十道壳，Notepad.exe 开始变得越来越肥大，但还可以运行。以后有时间再分析分析其他加壳软件的加壳标志，然后来个交叉加壳。每个各加三遍..........然后再............。老天，以后你不会用这种方法加壳你的软件来对付我吧........@#$@#$@ 昏倒...@#$@#$@#$。   
  
补充：从跟踪 ASPack v2.1 和 ASProtect v1.0 的情况来看，好象 PECompact 检测加壳标志的方法与其自身判断方法不同哦。  

 修补 PECompact v1.41b1 使之可以反复加壳的方法----这是 SE??什么方法的吧？别写错了。不然文化局的同志就来了。   
  
作者：        该职位空缺，有意者可与本人联系。限额一名，MM 优先。   
写作日期：    2000-08-07   
  
【前言】   
昨天，找到了 PECompact v1.41b1 判断其加壳标志的地方。今天继续追踪。   
  
  
从前文中知道只要改变跳转方向可以使 PECompact v1.41b1 反复加壳。   
  
:00403871 817E0C5045434F          cmp dword ptr \[esi+0C\], 4F434550    <--判断是否是 'PECO'，这是 PECompact 加壳的标志   
:00403878 0F841C010000            je 0040399A                <--就是要改这里   
  
不巧的是，在分析其他软件的过程中改动了日期，这次分析时 PECompact 跳出框框来说俺使用过期了。哈！真是没事找事，正好一起作手术！我偷懒没有去分析注册的方法。（其实不偷懒也未必找得到，现在俺的破解功夫早已成了花架子了）只是简单地让他不过期就行了。找到以下一处代码：（至于其他判断过期的地方，我就没有去管了，刚看了 Predator 做内科手术的方法，我也想试试这种 Patch 的方法。）   
  
:00405262 6883A24000              push 0040A283   
  
\* Reference To: KERNEL32.GetLocalTime, Ord:00F5h   
                                |   
:00405267 E80F270000              Call 0040797B   
:0040526C 66813D83A24000D007      cmp word ptr \[0040A283\], 07D0   
:00405275 750A                    jne 00405281   
:00405277 66833D85A2400007        cmp word ptr \[0040A285\], 0007   
:0040527F 761C                    jbe 0040529D                <--我要改这里的跳转方向，不跳就 Game Over 了。   
  
先找到脱壳后可以让我加入 Patch 代码的地方。找到 340 处，这里宽敞明亮，适合居住。   
然后，来找找芝麻开门的地方。   
  
  
00008090 6A00FFD0 FFA59A21 4000619D 68007041 j......!@.a.h.pA   
000080A0 00C30000 40000080 41000000 00100000 ....@...A.......   
  
这里的 809D - 80A0 的字节为 00704100，对应长整数为 00417000，正好是脱壳第四层的入口点。三层壳过后代码已经还原，好！这里可以设计大门，让他到我家坐坐再走。   
把 00704100 改为 40034000。   
  
现在再来设计我们的居室 （Patch 代码）   
  
C7057838400090909090    mov dword ptr \[00403878\],90909090   
66C7057C3840009090    mov word ptr \[0040387C\],9090   
C6057F524000EB        mov byte ptr \[0040527F\],EB   
6800704100        push 00417000   
C3            ret   
  
把此处十六进制代码加入 340 处。（注：此方法是对未脱壳的 PECompact 而言，什么？你拿脱壳后的 PECompact.EXE 来做手术，完了！病人死定了。）   
  
测试，OK!怎么 PECompact.EXE 没判断 CRC 的地方。   
  
【后记】   
这种动内科手术的方法确实高效。几行代码就搞定了，不用又去启动 C++Builder 了，让我的老老爷机也轻松了一把。   4dd466d0e1e0cc123afbe504da2462ca