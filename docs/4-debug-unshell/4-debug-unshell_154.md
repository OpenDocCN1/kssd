

# Acprotect 学习笔记

> 原文：[`www.pediy.com/kssd/pediy11/114168.html`](https://www.pediy.com/kssd/pediy11/114168.html)

**垃圾前言**  
  
这个壳早就已经被各位大牛肢解了，而 且也不更新了，现在的软件基本上不会用它来加壳了吧。我写这篇文章只是来过下手瘾  
  
其实 Acprotect 以前就分析过 了(主要是分析 IAT 处理),现在写篇文章来作个记录吧。  
  
同样来给 MM.exe 加个壳吧，这里我用的是 1.41 版的来加壳。  
  
![`bbs.pediy.com/picture.php?albumid=141&pictureid=464`](http://bbs.pediy.com/picture.php?albumid=141&pictureid=464)  
  
好吧，运行一下，MM1.exe 能运 行。OD 载入。跟一下,发现全是花指令  
  
既然它这么多花指令,那我们就不具体 F7 了。我们直接我们的目标,IAT 处理分析  
  
忽略所有异常访问,SOD 全选  
  
  
1.1 找 OEP+修复  
  
先到 OEP 去吧.怎么到 OEP?  
  
方法很多哈.理解了原理,用哪种方法 看大家的爱好,  
  
这里我用两次内存断点法。在.rsrc 段下内存访问断点.f9 运行,断下  
  
![`bbs.pediy.com/picture.php?albumid=141&pictureid=465`](http://bbs.pediy.com/picture.php?albumid=141&pictureid=465)  
  
现在代码段应该解码了吧  
  
到 00401000 去看下  
  
![`bbs.pediy.com/picture.php?albumid=141&pictureid=466`](http://bbs.pediy.com/picture.php?albumid=141&pictureid=466)  
  
Boolean 都出来了，证明已经解 码了  
  
那么接下来程序要到 OEP 就应该会到 代码段去执行代码了吧  
  
在代码段 00401000 下内存访问 断点，F9。断下。  
  
![`bbs.pediy.com/picture.php?albumid=141&pictureid=467`](http://bbs.pediy.com/picture.php?albumid=141&pictureid=467)  
  
咦这里是 Delphi 程序的笫一个 Call 吧。为什么不是断在 OEP,看看堆栈  
  
![`bbs.pediy.com/picture.php?albumid=141&pictureid=468`](http://bbs.pediy.com/picture.php?albumid=141&pictureid=468)  
  
返回到 004A3071　， 这 个 004A3071 是在哪里呢看看区段信息  
  
![`bbs.pediy.com/picture.php?albumid=141&pictureid=470`](http://bbs.pediy.com/picture.php?albumid=141&pictureid=470)  
  
原来是在壳段中,那么为什么不会断下 OEP 应该知道了吧。  
  
它把程序的 OEP 移动到壳段而且进行 了变形。我们不管它的变形,我们主要分析它的 IAT 哈  
  
既然找到了笫一个 call 那就在 004A306B 这里下个硬件执行断点吧。然后重载程序。F9。断下。  
  
![`bbs.pediy.com/picture.php?albumid=141&pictureid=471`](http://bbs.pediy.com/picture.php?albumid=141&pictureid=471)  
  
我们来修复下 OEP，  
  
找个未回味加壳的 delphi 程序  
  
![`bbs.pediy.com/picture.php?albumid=141&pictureid=472`](http://bbs.pediy.com/picture.php?albumid=141&pictureid=472)  
  
下面就直接 patch 代码来修复 OEP  
eax 的值看下断下后的 eax 的值就行了  
  
![`bbs.pediy.com/picture.php?albumid=141&pictureid=473`](http://bbs.pediy.com/picture.php?albumid=141&pictureid=473)  
  
004A3060 新建 EIP。取消两 个 nop 的修改。  
  
OEP 修复完毕  
  
1.2，修复 IAT  
  
现在跟进去笫一个 CALL  
  
004A3060 55 push ebp  
004A3061 8BEC mov ebp, esp  
004A3063 83C4 F0 add esp, -10  
004A3066 B8 4C2B4600 mov eax, 462B4C  
004A306B FF15 B5D84800 call dword ptr \[48D8B5\] ; F7 跟进  
004A3071 A1 E8484600 mov eax, dword ptr \[4648E8\]  
\-----------------------------------------------------------------------------  
00405C34 53 push ebx ;到这，  
00405C35 8BD8 mov ebx, eax  
00405C37 33C0 xor eax, eax  
00405C39 A3 9C304600 mov dword ptr \[46309C\], eax  
00405C3E 6A 00 push 0  
00405C40 E8 2BFFFFFF call 00405B70 ; 继续 F7 跟进  
00405C45 A3 64564600 mov dword ptr \[465664\], eax  
00405C4A A1 64564600 mov eax, dword ptr \[465664\]  
\------------------------------------------------------------------------------  
  
到这里 就是 iat 重定向了吧，都定向到壳段去了  
  
 ![`bbs.pediy.com/picture.php?albumid=141&pictureid=474`](http://bbs.pediy.com/picture.php?albumid=141&pictureid=474)  
  
先数据窗口中跟随内存地址  
  
  
00466110  00000000  
  
00466114  00000000  
  
00466118  0048D010   MM1.0048D010  
  
0046611C  0048D01D   MM1.0048D01D  
  
00466120  0048D02A   MM1.0048D02A  
  
00466124  0048D037   MM1.0048D037  
  
00466128  0048D044   MM1.0048D044  
  
0046612C  0048D051   MM1.0048D051  
  
00466130  0048D05E   MM1.0048D05E  
  
00466134  0048D06B   MM1.0048D06B  
  
00466138  0048D078   MM1.0048D078  
  
0046613C  0048D085   MM1.0048D085  
  
00466140  0048D092   MM1.0048D092  
  
00466144  0048D09F   MM1.0048D09F  
  
00466148  0048D0AC   MM1.0048D0AC  
  
0046614C  0048D0B9   MM1.0048D0B9  
  
00466150  0048D0C6   MM1.0048D0C6  
  
00466154  0048D0D3   MM1.0048D0D3  
  
00466158  0048D0E0   MM1.0048D0E0  
  
0046615C  0048D0ED   MM1.0048D0ED  
  
00466160  0048D0FA   MM1.0048D0FA  
  
00466164  0048D107   MM1.0048D107  
  
00466168  0048D114   MM1.0048D114  
  
0046616C  0048D121   MM1.0048D121  
  
00466170  0048D12E   MM1.0048D12E  
  
00466174  0048D13B   MM1.0048D13B  
  
00466178  0048D148   MM1.0048D148  
  
0046617C  0048D155   MM1.0048D155  
  
00466180  0048D162   MM1.0048D162  
  
00466184  0048D16F   MM1.0048D16F  
  
00466188  0048D17C   MM1.0048D17C  
  
0046618C  0048D189   MM1.0048D189  
  
00466190  0048D196   MM1.0048D196  
  
00466194  0048D1A3   MM1.0048D1A3  
  
00466198  0048D1B0   MM1.0048D1B0  
  
0046619C  0048D1BD   MM1.0048D1BD  
  
004661A0  00000000  
  
004661A4  0048D1CA   MM1.0048D1CA  
  
004661A8  0048D1D7   MM1.0048D1D7  
  
004661AC  0048D1E4   MM1.0048D1E4  
  
004661B0  0048D1F1   MM1.0048D1F1  
  
004661B4  00000000  
  
004661B8  77DA7ABB   advapi32.RegQueryValueExA  
  
004661BC  77DA7852   advapi32.RegOpenKeyExA  
  
004661C0  77DA6C27   advapi32.RegCloseKey  
  
004661C4  00000000  
  
004661C8  770F4880   oleaut32.SysFreeString  
  
004661CC  770FA3EC   oleaut32.SysReAllocStringLen  
  
004661D0  770F4B39   oleaut32.SysAllocStringLen  
  
004661D4  00000000  
  
004661D8  0048D1FE   MM1.0048D1FE  
  
004661DC  0048D20B   MM1.0048D20B  
  
   
  
这就是被处理后的 IAT 的，看到有一 些函数被处理了，有一些函数没被处理。  
  
我们跟一个看看吧。这里选择笫一个 0048D010。到 0048D010 去新建个 EIP  
看到这是是一段重复的差 不多的代码  
  
\[IMG\]![`bbs.pediy.com/picture.php?albumid=141&pictureid=475`](http://bbs.pediy.com/picture.php?albumid=141&pictureid=475)\[/IMG\]  
  
我们 F7 单步走  
  
当我们走到 0048D01C 这里的时候我们发现栈顶中出现了真实的 API 地址  
  
 ![`bbs.pediy.com/picture.php?albumid=141&pictureid=476`](http://bbs.pediy.com/picture.php?albumid=141&pictureid=476)  
  
它这里是先 push 了一个值支栈顶, 然后把这个值和另外一个值进行异或不得到了真实函数的地址.  
  
我们再跟一个函数看下是不是也是这 样，我们就跟笫二个 0048D01D，吧，在 0048D01D 新建 EIP，然后跟  
  
发现当走到 retn 的时候又出现了真 实 API 的函数地址。  
  
  
  
那么我们只要每个函数都这个跟一下到 retn 的时候再把栈顶的真实地址写回 IAT 里，这样就能得找正确的 IAT 了。  
这样手动一个个跟估计我 会 xx 掉，那 么我们就用脚本来帮我人来完成这个工作  
  

引用: var Old\_esp  
var cur\_addr  
var old\_eip  
  
mov old\_esp,esp //保存当前的 eip,和 esp，脚本运行完后我们要还原它们  
mov old\_eip,eip  
mov cur\_addr,00466114 //00450114 是 IAT 的起始地址-4  
  
loop:  
mov esp,old\_esp //还原一下 esp//在解密真实函数地址的时候会 push 一个值，我们要还原它  
add cur\_addr,4 //IAT 笫一个地址  
cmp cur\_addr,00466708 ;004516d4 是 IAT 末尾+4  
jae exit //到达 IAT 末尾就结束  
cmp \[cur\_addr\],10000000 //因为有一些函数没有被壳处理掉,所以我们不需要还原  
jae loop  
cmp \[cur\_addr\],0 // 有一些是作为一个 dll 的结束标志,其值为 0，我们不需要处理  
je loop  
mov eip,\[cur\_addr\] ;在目的地址新建 EIP  
sti //F7 两步到 retn，栈顶会出现真实函数地址  
sti  
mov \[cur\_addr\],\[esp\] //把真实函数地址写回去  
jmp loop  
exit:  
mov esp,old\_esp //还原 esp,eip  
mov eip,old\_eip  
ret 

好运行脚本，脚本 跑完后我们看下 IAT  
  
![`bbs.pediy.com/picture.php?albumid=141&pictureid=477`](http://bbs.pediy.com/picture.php?albumid=141&pictureid=477)  
  
发现函数基本都修复完了，不过还有两个还没修复  
  
  
004661AC  0049BE4B   MM1.0049BE4B  
  
004664DC  0049BE4B   MM1.0049BE4B  
  
这两个函数都是指 向　0049BE４B，就是执行同一个函数了，那个我们去 0049BE4B 新建个 EIP，跟一下  
  
0049BE4B 60 pushad  
0049BE4C 85DE testesi, ebx  
0049BE4E 45 incebp  
0049BE4F F9 stc  
0049BE50 03DE addebx, esi  
0049BE52 50 pusheax  
0049BE53 E8 01000000 call 0049BE59 ; 0049BE59  
0049BE58 - E9 83C40458 jmp 584E82E0  
  
发现这里和刚才的加密方式不同，F7 来跟，我这里就不贴代码给大家看了，因为这个加密的没刚  
  
才的那么简单，跟几步就出来了。  
  
这里要跟很久，解密完后，在 EAX 和 ECX 中会出现函数地址  
  
  
EAX 77D507EA USER32.MessageBoxA  
  
ECX 0012FFC8  
  
EDX 8054C6ED  
  
EBX 77D507EA USER32.MessageBoxA  
  
ESP 0012FFAC  
  
  
那么我们知道了这个函数就是 MessageBoxA.我们就修改还没修复的两个函数为  
  
MessageBoxA 就行了。  
  
IAT 修复完毕。  
  
  
由于在跟 MessageBoxA 的时 候程序的一些环境已经变了，所以我们重来,这一次就不用跟 MessagBoxA 了，直接修复。  
  
Lordpe 来 dump.Import Rec 来修复,直接自动搜索 IAT 会找不到，需要我们自己填写 IAT 地址和大小  
  
最后程序运行成功。  
  
最后上个图,此图只为增加人气用,没 其它目的,要不我的文章没人看就不好了  
  
![`bbs.pediy.com/picture.php?albumid=141&pictureid=478`](http://bbs.pediy.com/picture.php?albumid=141&pictureid=478)  
  
  
**2.IAT 处理过程的分析**  
  
根据 Kissy 牛的指点,分析了下 IAT 处理的具体过程,多谢 Kissy 牛![](img/6821a6175013046370981b6035d43ab4.jpg "使眼色")  
  
004A147F    03F2            addesi, edx; esi =00466000 - >IID  
  
00466000  00 00 00 00 00 00 00 00 00 00 00 00 0C 67 06 00  .............g.  
00466010  18 61 06 00 00 00 00 00 00 00 00 00 00 00 00 00  a.............  
00466020  84 69 06 00 A4 61 06 00 00 00 00 00 00 00 00 00  ..........  
00466030  00 00 00 00 CA 69 06 00 B8 61 06 00 00 00 00 00  ..........  
00466040  00 00 00 00 00 00 00 00 0A 6A 06 00 C8 61 06 00  .........j..  
00466050  00 00 00 00 00 00 00 00 00 00 00 00 52 6A 06 00  ............Rj.  
00466060  D8 61 06 00 00 00 00 00 00 00 00 00 00 00 00 00  .............  
00466070  9E 6A 06 00 EC 61 06 00 00 00 00 00 00 00 00 00  ..........  
00466080  00 00 00 00 DE 6A 06 00 FC 61 06 00 00 00 00 00  ..........  
00466090  00 00 00 00 00 00 00 00 04 6F 06 00 F8 62 06 00  ........o..  
004660A0  00 00 00 00 00 00 00 00 00 00 00 00 52 6F 06 00  ............Ro.  
004660B0  08 63 06 00 00 00 00 00 00 00 00 00 00 00 00 00  c.............  
004660C0  C4 73 06 00 10 64 06 00 00 00 00 00 00 00 00 00  .d.........  
004660D0  00 00 00 00 B6 7D 06 00 84 66 06 00 00 00 00 00  ..........  
004660E0  00 00 00 00 00 00 00 00 CC 7D 06 00 8C 66 06 00  ..........  
  
;这里是被加密了的输入表，不过它的输入表结构是完整的  
004A1481    8B46 0C         moveax, dwordptr \[esi+C\]           ; ;eax -> IID.Name  
004A1484    0BC0            oreax, eax  
004A1486    0F84 25020000   je      004A16B1                         ; 004A16B1  
004A148C    8366 0C 00      anddwordptr \[esi+C\], 0             ; ;断在这,用 0 填充充掉 IID.Name  
004A1490    03C2            addeax, edx; ;+00400000 ->IID.Name VA  
004A1492    8BD8            movebx, eax  
004A1494    56              pushesi  
004A1495    57              pushedi  
004A1496    50              pusheax  
004A1497    8BF3            movesi, ebx  
004A1499    8BFB            movedi, ebx  
004A149B    AC              lodsbyteptr \[esi\]                   ; 下面这几句是解密 DLL 的名字,并且写回原来的地方  
004A149C    C0C0 03         rolal, 3  
004A149F    AA              stosbyteptres:\[edi\]  
004A14A0    803F 00         cmpbyteptr \[edi\], 0  
004A14A3  ^ 75 F6           jnzshort 004A149B                   ; 循环解密 DLL 名字  
004A14A5    58              popeax  
004A14A6    5F              popedi  
004A14A7    5E              popesi  
004A14A8    50              pusheax  
004A14A9    FF95 20854100   calldwordptr \[ebp+418520\]           ; GetModuleHandleA 获取 DLL 模块句柄  
004A14AF    0BC0            oreax, eax  
004A14B1    75 43           jnzshort 004A14F6                   ; 获取成功跳  
  
  
004A14F6    60              pushad; ;下面这一段会把刚才解密出来的 DLL 名字用 0 填充掉  
004A14F7    2BC0            subeax, eax  
004A14F9    8803            movbyteptr \[ebx\], al  
004A14FB    43              incebx  
004A14FC    3803            cmpbyteptr \[ebx\], al  
004A14FE  ^ 75 F9           jnzshort 004A14F9                      ; ;循环填充  
004A1500    61              popad  
004A1501    8985 3EF84000   movdwordptr \[ebp+40F83E\], eax; ;保存 DLL 模块句柄  
004A1507    C785 42F84000 0>movdwordptr \[ebp+40F842\], 0  
004A1511    8B95 46F84000   movedx, dwordptr \[ebp+40F846\]         ; ;00400000  
004A1517    8B06            moveax, dwordptr \[esi\]                ; ;eax ->OriginalFirstThunk  
004A1519    0BC0            oreax, eax; ;OriginalFirstThunk  是否为空?  
004A151B    75 07           jnzshort 004A1524                      ; ;不为空就直接使用它,为空就使用 FirstThunk  
004A151D    90              nop  
004A151E    90              nop  
004A151F    90              nop  
004A1520    90              nop  
004A1521    8B46 10         moveax, dwordptr \[esi+10\]             ; ;FirstThunk->IMAGE\_THUNK\_DATA  
004A1524    03C2            addeax, edx; ;00400000  
004A1526    0385 42F84000   addeax, dwordptr \[ebp+40F842\]  
004A152C    8B18            movebx, dwordptr \[eax\]                ; ;IMAGE\_IMPORT\_BY\_NAME  
004A152E    8B7E 10         movedi, dwordptr \[esi+10\]  
004A1531    03FA            addedi, edx  
004A1533    03BD 42F84000   addedi, dwordptr \[ebp+40F842\]  
004A1539    85DB            testebx, ebx; ;是否已经处理完一个 DLL?  
004A153B    0F84 62010000   je      004A16A3                            ; 004A16A3  
004A1541    F7C3 00000080   testebx, 80000000                       ; ;Hint 的高位是否为 1?  
004A1547    75 1D           jnzshort 004A1566                      ; 004A1566  
004A1549    90              nop  
004A154A    90              nop  
004A154B    90              nop  
004A154C    90              nop  
004A154D    03DA            addebx, edx; ;00400000  
004A154F    83C3 02         addebx, 2                              ; ;跳过 Hint  
004A1552    56              pushesi  
004A1553    57              pushedi  
004A1554    50              pusheax  
004A1555    8BF3            movesi, ebx  
004A1557    8BFB            movedi, ebx  
004A1559    AC              lodsbyteptr \[esi\]                      ; ;下面是解密 API 的名字，写回原处  
004A155A    C0C0 03         rolal, 3  
004A155D    AA              stosbyteptres:\[edi\]  
004A155E    803F 00         cmpbyteptr \[edi\], 0  
004A1561  ^ 75 F6           jnzshort 004A1559                      ; ;循环解密  
004A1563    58              popeax  
004A1564    5F              popedi  
004A1565    5E              popesi  
004A1566    3B9D 46F84000   cmpebx, dwordptr \[ebp+40F846\]         ; ;00400000  
004A156C    7C 11           jlshort 004A157F                      ; 004A157F  
004A156E    90              nop  
004A156F    90              nop  
004A1570    90              nop  
004A1571    90              nop  
004A1572    83BD 1A204000 0>cmpdwordptr \[ebp+40201A\], 0  
004A1579    75 0A           jnzshort 004A1585                      ; 004A1585  
004A157B    90              nop  
004A157C    90              nop  
004A157D    90              nop  
004A157E    90              nop  
004A157F    81E3 FFFFFF0F   andebx, 0FFFFFFF  
004A1585    53              pushebx  
004A1586    FFB5 3EF84000   pushdwordptr \[ebp+40F83E\]  
004A158C    FF95 1C854100   calldwordptr \[ebp+41851C\]              ; ;GetProcAddress 获取函数地址  
004A1592    3B9D 46F84000   cmpebx, dwordptr \[ebp+40F846\]         ; ;00400000  
004A1598    7C 0F           jlshort 004A15A9                      ; 004A15A9  
004A159A    90              nop  
004A159B    90              nop  
004A159C    90              nop  
004A159D    90              nop  
004A159E    60              pushad; ;下面又会循环用 0 填充掉刚才解密出来的 API 名字  
004A159F    2BC0            subeax, eax  
004A15A1    8803            movbyteptr \[ebx\], al  
004A15A3    43              incebx  
004A15A4    3803            cmpbyteptr \[ebx\], al  
004A15A6  ^ 75 F9           jnzshort 004A15A1                      ; 004A15A1  
004A15A8    61              popad  
004A15A9    0BC0            oreax, eax; ;获取地址是否成功?  
004A15AB  ^ 0F84 15FFFFFF   je      004A14C6                            ; 004A14C6  
004A15B1    3B85 2C854100   cmpeax, dwordptr \[ebp+41852C\]         ; ;判断这个 API 是否是 MessageBoxA?  
004A15B7    74 20           jeshort 004A15D9                      ; 004A15D9  
004A15B9    90              nop  
004A15BA    90              nop  
004A15BB    90              nop  
004A15BC    90              nop  
004A15BD    3B85 C4FD4000   cmpeax, dwordptr \[ebp+40FDC4\]         ; ;判断是不是 RegisterHotKey?  
004A15C3    74 09           jeshort 004A15CE                      ; 004A15CE  
004A15C5    90              nop  
004A15C6    90              nop  
004A15C7    90              nop  
004A15C8    90              nop  
004A15C9    EB 14           jmpshort 004A15DF                      ; ;跳  
004A15CB    90              nop  
004A15CC    90              nop  
004A15CD    90              nop  
004A15CE    8D85 31FE4000   leaeax, dwordptr \[ebp+40FE31\]  
004A15D4    EB 09           jmpshort 004A15DF                      ; 004A15DF  
004A15D6    90              nop  
004A15D7    90              nop  
004A15D8    90              nop  
004A15D9    8D85 4BFE4000   leaeax, dwordptr \[ebp+40FE4B\]  
004A15DF    56              pushesi  
004A15E0    FFB5 3EF84000   pushdwordptr \[ebp+40F83E\]  
004A15E6    5E              popesi  
004A15E7    39B5 12204000   cmpdwordptr \[ebp+402012\], esi; ;是不是 Kernel32.dll?  
004A15ED    74 15           jeshort 004A1604                      ; 004A1604  
004A15EF    90              nop  
004A15F0    90              nop  
004A15F1    90              nop  
004A15F2    90              nop  
004A15F3    39B5 16204000   cmpdwordptr \[ebp+402016\], esi  
004A15F9    74 09           jeshort 004A1604                      ; 004A1604  
004A15FB    90              nop  
004A15FC    90              nop  
004A15FD    90              nop  
004A15FE    90              nop  
004A15FF    EB 63           jmpshort 004A1664                      ; 004A1664  
004A1601    90              nop  
004A1602    90              nop  
004A1603    90              nop  
004A1604    80BD 16564100 0>cmpbyteptr \[ebp+415616\], 0  
004A160B    74 57           jeshort 004A1664                      ; 004A1664  
004A160D    90              nop  
004A160E    90              nop  
004A160F    90              nop  
004A1610    90              nop  
004A1611    EB 07           jmpshort 004A161A                      ; 004A161A  
004A1613    90              nop  
004A1614    90              nop  
004A1615    90              nop  
004A1616    0100            adddwordptr \[eax\], eax; ;下面的一段自成一个函数,API 会重定向到这个函数  
004A1618    0000            addbyteptr \[eax\], al  
004A161A    8BB5 0BF94000   movesi, dwordptr \[ebp+40F90B\]  
004A1620    83C6 0D         addesi, 0D  
004A1623    81EE 02184000   subesi, 401802  
004A1629    2BF5            subesi, ebp  
004A162B    83FE 00         cmpesi, 0  
004A162E    7F 34           jgshort 004A1664                      ; 004A1664  
004A1630    90              nop  
004A1631    90              nop  
004A1632    90              nop  
004A1633    90              nop  
004A1634    8BB5 0BF94000   movesi, dwordptr \[ebp+40F90B\]  
004A163A    53              pushebx  
004A163B    50              pusheax  
004A163C    E8 8DB2FFFF     call    0049C8CE                            ; 0049C8CE  
004A1641    8BD8            movebx, eax  
004A1643    58              popeax  
004A1644    33C3            xoreax, ebx  
004A1646    C606 68         movbyteptr \[esi\], 68  
004A1649    8946 01         movdwordptr \[esi+1\], eax  
004A164C    C746 05 8134240>movdwordptr \[esi+5\], 243481  
004A1653    895E 08         movdwordptr \[esi+8\], ebx  
004A1656    C646 0C C3      movbyteptr \[esi+C\], 0C3  
004A165A    5B              popebx; ;到这里后我们到 esi 的地址去看看,发面这就是解密 API 地址的函数。  
  
0048D010    68 F02096ED     push    ED9620F0  
0048D015    813424 8A330591 xordwordptr \[esp\], 9105338A  
0048D01C    C3              retn  
  
  
004A165B    8BC6            moveax, esi; eax = 自成的函数地址  
004A165D    8385 0BF94000 0>adddwordptr \[ebp+40F90B\], 0D  
004A1664    5E              popesi  
004A1665    60              pushad  
004A1666    8BD0            movedx, eax  
004A1668    2BBD 46F84000   subedi, dwordptr \[ebp+40F846\]         ; ;edi->IAT,存放函数地址的 RVA  
004A166E    8BC7            moveax, edi  
004A1670    B9 01010000     movecx, 101  
004A1675    8DBD EBEC4000   leaedi, dwordptr \[ebp+40ECEB\]  
004A167B    F2:AF           repnescasdwordptres:\[edi\]  
004A167D    0BC9            orecx, ecx  
004A167F    74 13           jeshort 004A1694                      ; ;跳  
004A1681    90              nop  
004A1682    90              nop  
004A1683    90              nop  
004A1684    90              nop  
004A1685    81E9 01010000   subecx, 101  
004A168B    F7D1            notecx  
004A168D    89948D EBE84000 movdwordptr \[ebp+ecx\*4+40E8EB\], edx  
004A1694    61              popad  
004A1695    8907            movdwordptr \[edi\], eax; 用生成的函数地址来填充掉 IAT 中的函数地址  
004A1697    8385 42F84000 0>adddwordptr \[ebp+40F842\], 4           ; ;加 4，处理下一个 API  
004A169E  ^ E9 6EFEFFFF     jmp     004A1511                            ; ;循环处理 API  
004A16A3    83C6 14         addesi, 14                             ; ;加 20，esi 指向下一个 IID  
004A16A6    8B95 46F84000   movedx, dwordptr \[ebp+40F846\]  
004A16AC  ^ E9 D0FDFFFF     jmp     004A1481                            ; ;处理下一个 DLL  
004A16B1    8DBD EBEC4000   leaedi, dwordptr \[ebp+40ECEB\]  
004A16B7    33C0            xoreax, eax  
  
;我们直接在 004A16B1，F4，那么 IAT 的处理就完毕了，我们到 0048D010 去看看，  
  
0048D010    68 F02096ED     push    ED9620F0  
0048D015    813424 8A330591 xordwordptr \[esp\], 9105338A  
0048D01C    C3              retn  
0048D01D    68 0453A0B0     push    B0A05304  
0048D022    813424 E44332CC xordwordptr \[esp\], CC3243E4  
0048D029    C3              retn  
0048D02A    68 4A8B1C69     push    691C8B4A  
0048D02F    813424 4A9B8E15 xordwordptr \[esp\], 158E9B4A  
0048D036    C3              retn  
0048D037    68 1BCC1C7C     push    7C1CCC1B  
0048D03C    813424 8A539C00 xordwordptr \[esp\], 9C538A  
0048D043    C3              retn  
0048D044    68 34E1457C     push    7C45E134  
0048D049    813424 B07AC500 xordwordptr \[esp\], 0C57AB0  
0048D050    C3              retn  
0048D051    68 414BDC7E     push    7EDC4B41  
0048D056    813424 B0D15C02 xordwordptr \[esp\], 25CD1B0  
0048D05D    C3              retn  
  
发现就是生成的一系统解密函数，而且 IAT 中地函数地址都指向了这里。  
  
我们来总结一下它的流程吧  
  
用 0 填充 IID.Name -> 解密 DLL 名字 -> 获取 DLL 模块句柄 -> 销毁 DLL 名字 -> 解密 API 名字 ->获取 API 地址 ->  
销毁 API 名字 ->处理特殊函数 -> 生成一个函数 －>修改 IAT，把 API 重定向到这个函数  
  
好吧，分析完了,我们知道刚开始的时候输入表的结构是完整的，不过 DLL 和 API 名字都是被加密了的。  
它在处理 IAT 的时候又会解密出 DLL 和 API 名字并且写回原来的输入表中，不过它后面又会销毁掉，。  
那么我们想要它还原正确的输入表就不能让它销毁掉解密出来的 DLL 和 API 名字。  
004A148C 是销毁 IID.Name NOP 掉  
  
004A14F6 － 004A1500 是销毁解密出来的 DLL 名字  NOP 掉  
  
004A159E － 004A15A8 是销毁解密出来的 API 名字 NOP 掉  
  
004A1695 是 API 重定向 NOP 掉  
  
然后用：LordPe dump 出来，修正 OEP。IAT 为 66000，大小为 1000 就行了，这样的程序就能直接运行了，不用 Import Rec 修复  
我就不演示这个过程了[附件 43983](http://bbs.pediy.com/attachment.php?attachmentid=43983)