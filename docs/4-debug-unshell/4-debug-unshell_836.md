

# 脱《唐诗三百首 1.2》壳（PEBundle 0.2 - 3.x -> Jeremy Collake）并提取 MID 数据

> 原文：[`www.pediy.com/kssd/pediy10/80863.html`](https://www.pediy.com/kssd/pediy10/80863.html)

【破文作者】 JIVI（乔伟）  
【使用工具】 Peid 0.94,OllyDbg（OllyIce）,exeScope,010Editor    
【破 J 平台】 WinXP   
【软件名称】 唐诗三百首 1.2           
【软件简介】 看名字就知道了   
  
 前两天在 PPStream 上看《侠女闯天关》。前几集里有一首不停出现的曲子，感觉蛮好听的，同时又有种非常熟悉的感觉。想了半天才想起来是很久以前用过的一个软件的背景音乐，这个软件就是我们今天的目标《唐诗三百首》（版本 1.2 ，新的版本背景音乐已经换了）。当时就想把这首曲子下下来听听，但不知是什么名字，于是百度之，有一人回答是 高山流水，下下来。听了一遍，对不上号。又搜了半天终是无果。后来想想，既然是软件的背景音乐，那个软件里自然是有的，幸好这个软件还是比较容易下的，下下来后，解压就能用，在解压的文件夹里看了一看，总共就六个文件，一个 EXE 和 sr.cys sr.edt ts.cys ts.edt 四个文件，外加一个帮助文件，看来这个背景音乐（MID 格式的）多半是被嵌到主程序里去了。既然这样只有手动提取了            
  
好的，下面开始提取的过程。  
首先还是查壳，因为无论是压缩壳还是加密壳你都是无法看到原 MID 数据的 ，无法直接提取，当然我们希望他是没壳的。但事实往往不如人意，用 PEID 查壳的结果是：  
PEBundle 0.2 - 3.x -> Jeremy Collake  
还好，这不是什么猛壳，用 ESP 定律（不懂 ESP 定律的，自行去搜索） 轻松搞定  
OD 载入（忽略所有异常）（注：有 HIDEOD 和 PHANTOM 两个隐藏 OD 的插件，具体他们有没有作用没试） 来到这里  
004A6000 > 9C pushfd  
004A6001 60 pushad  
004A6002 E8 02000000 call 004A6009  
004A6007 33C0 xor eax, eax  
004A6009 8BC4 mov eax, esp  
004A600B 83C0 04 add eax, 4  
004A600E 93 xchg eax, ebx  
004A600F 8BE3 mov esp, ebx  
004A6011 8B5B FC mov ebx, dword ptr \[ebx-4\]  
004A6014 81EB 07304000 sub ebx, 00403007  
004A601A 87DD xchg ebp, ebx  
004A601C 80BD DD3B4000 0>cmp byte ptr \[ebp+403BDD\], 0  
004A6023 74 21 je short 004A6046  
004A6025 8D85 D6384000 lea eax, dword ptr \[ebp+4038D6\]  
004A602B 50 push eax  
004A602C FF95 B2384000 call dword ptr \[ebp+4038B2\]  
004A6032 8D8D 2C3A4000 lea ecx, dword ptr \[ebp+403A2C\]  
004A6038 51 push ecx  
004A6039 50 push eax  
004A603A FF95 A2384000 call dword ptr \[ebp+4038A2\]  
004A6040 8985 3C3A4000 mov dword ptr \[ebp+403A3C\], eax  
004A6046 8DBD 703E4000 lea edi, dword ptr \[ebp+403E70\]  
004A604C 33C0 xor eax, eax  
  
开头就是两个 PUSHFD PUSHAD 典型的壳的处理模式，用于保护现场  
  
下硬件点   
在 Command 里输入 hw 0012ffc0 （为什么下这个，看 ESP 定律的原理就知道了）回车  
F9 断在 pushad 那句不用管，继续 F9 来到这里  
004A6376 C3 retn  
004A6377 C8 000000 enter 0, 0  
004A637B 57 push edi  
004A637C 56 push esi  
004A637D 8B75 08 mov esi, dword ptr \[ebp+8\]  
004A6380 8B7D 0C mov edi, dword ptr \[ebp+C\]  
004A6383 8A06 mov al, byte ptr \[esi\]  
004A6385 3C 61 cmp al, 61  
004A6387 72 06 jb short 004A638F  
004A6389 3C 7A cmp al, 7A  
004A638B 77 02 ja short 004A638F  
004A638D 04 E0 add al, 0E0  
004A638F 8A27 mov ah, byte ptr \[edi\]  
004A6391 80FC 61 cmp ah, 61  
004A6394 72 08 jb short 004A639E  
004A6396 80FC 7A cmp ah, 7A  
004A6399 77 03 ja short 004A639E  
004A639B 80C4 E0 add ah, 0E0  
004A639E 46 inc esi  
004A639F 47 inc edi  
  
继续 F9 来到这里   
00496001 60 pushad  
00496002 E8 02000000 call 00496009  
00496007 33C0 xor eax, eax  
00496009 8BC4 mov eax, esp  
0049600B 83C0 04 add eax, 4  
0049600E 93 xchg eax, ebx  
0049600F 8BE3 mov esp, ebx  
00496011 8B5B FC mov ebx, dword ptr \[ebx-4\]  
00496014 81EB 07304000 sub ebx, 00403007  
0049601A 87DD xchg ebp, ebx  
0049601C 80BD DD3B4000 0>cmp byte ptr \[ebp+403BDD\], 0  
00496023 74 21 je short 00496046  
00496025 8D85 D6384000 lea eax, dword ptr \[ebp+4038D6\]  
0049602B 50 push eax  
0049602C FF95 B2384000 call dword ptr \[ebp+4038B2\]  
00496032 8D8D 2C3A4000 lea ecx, dword ptr \[ebp+403A2C\]  
00496038 51 push ecx  
00496039 50 push eax  
第一句就是一个 pushad 想必又多保护了一层 ，看来还得继续  
F9 来到这里  
00496376 C3 retn  
00496377 C8 000000 enter 0, 0  
0049637B 57 push edi  
0049637C 56 push esi  
0049637D 8B75 08 mov esi, dword ptr \[ebp+8\]  
00496380 8B7D 0C mov edi, dword ptr \[ebp+C\]  
00496383 8A06 mov al, byte ptr \[esi\]  
00496385 3C 61 cmp al, 61  
00496387 72 06 jb short 0049638F  
00496389 3C 7A cmp al, 7A  
0049638B 77 02 ja short 0049638F  
0049638D 04 E0 add al, 0E0  
0049638F 8A27 mov ah, byte ptr \[edi\]  
00496391 80FC 61 cmp ah, 61  
00496394 72 08 jb short 0049639E  
00496396 80FC 7A cmp ah, 7A  
00496399 77 03 ja short 0049639E  
0049639B 80C4 E0 add ah, 0E0  
0049639E 46 inc esi  
0049639F 47 inc edi  
004963A0 837D 10 01 cmp dword ptr \[ebp+10\], 1  
看地址和代码都不象是 OEP，继续     
F9 来到这里  
0042C009 60 pushad  
0042C00A E8 02000000 call 0042C011  
0042C00F 33C0 xor eax, eax  
0042C011 8BC4 mov eax, esp  
0042C013 83C0 04 add eax, 4  
0042C016 93 xchg eax, ebx  
0042C017 8BE3 mov esp, ebx  
0042C019 8B5B FC mov ebx, dword ptr \[ebx-4\]  
0042C01C 81EB 3F904000 sub ebx, 0040903F  
0042C022 87DD xchg ebp, ebx  
0042C024 8B85 E6904000 mov eax, dword ptr \[ebp+4090E6\]  
0042C02A 0185 33904000 add dword ptr \[ebp+409033\], eax  
0042C030 66:C785 3090400>mov word ptr \[ebp+409030\], 9090  
0042C039 0185 DA904000 add dword ptr \[ebp+4090DA\], eax  
0042C03F 0185 DE904000 add dword ptr \[ebp+4090DE\], eax  
0042C045 0185 E2904000 add dword ptr \[ebp+4090E2\], eax  
0042C04B BB 7B110000 mov ebx, 117B  
0042C050 039D EA904000 add ebx, dword ptr \[ebp+4090EA\]  
0042C056 039D E6904000 add ebx, dword ptr \[ebp+4090E6\]  
0042C05C 53 push ebx  
0042C05D 8BC3 mov eax, ebx  
0042C05F 8BFB mov edi, ebx  
0042C061 2D AC904000 sub eax, 004090AC  
同样，开头就是 PUSHAD，不用讲  
继续 F9 来到这里   
0042D551 68 6B134000 push 0040136B  
0042D556 C2 0400 retn 4  
0042D559 8BB5 5B974000 mov esi, dword ptr \[ebp+40975B\]  
0042D55F 0BF6 or esi, esi  
0042D561 74 18 je short 0042D57B  
0042D563 8B95 E6904000 mov edx, dword ptr \[ebp+4090E6\]  
0042D569 03F2 add esi, edx  
0042D56B E8 0F000000 call 0042D57F  
0042D570 72 0B jb short 0042D57D  
0042D572 83C6 14 add esi, 14  
0042D575 837E 0C 00 cmp dword ptr \[esi+C\], 0  
0042D579 ^ 75 F0 jnz short 0042D56B  
0042D57B F8 clc  
0042D57C C3 retn  
0042D57D F9 stc  
0042D57E C3 retn  
0042D57F C785 31974000 0>mov dword ptr \[ebp+409731\], 0  
0042D589 8B0E mov ecx, dword ptr \[esi\]  
0042D58B 8B7E 10 mov edi, dword ptr \[esi+10\]  
0042D58E 0BC9 or ecx, ecx  
0042D590 75 02 jnz short 0042D594  
0042D592 8BCF mov ecx, edi  
还不是。  
继续 F9 来到这里   
0040136C 8B db 8B  
0040136D EC db EC  
0040136E 6A db 6A ; CHAR 'j'  
0040136F FF db FF  
00401370 68 db 68 ; CHAR 'h'  
00401371 B0 db B0  
00401372 40 db 40 ; CHAR '@'  
00401373 40 db 40 ; CHAR '@'  
00401374 00 db 00  
00401375 68 db 68 ; CHAR 'h'  
00401376 A0 db A0  
00401377 1E db 1E  
00401378 40 db 40 ; CHAR '@'  
00401379 00 db 00  
0040137A 64 db 64 ; CHAR 'd'  
0040137B A1 db A1  
0040137C 00 db 00  
0040137D 00 db 00  
0040137E 00 db 00  
0040137F 00 db 00  
00401380 50 db 50 ; CHAR 'P'  
00401381 64 db 64 ; CHAR 'd'  
00401382 89 db 89  
00401383 25 db 25 ; CHAR '%'  
  
看地址，再看看数据窗口，已经有明文出现了，看来到这里应该是到了 OEP 的下一个指令了，不过既然是 OEP 的下一个指令，当然应该是代码了，但看上面的那一堆东西，明明是数据，明显的软件作者在这里用了花指令。    
去花指令，在 OD 代码窗口 右键－》去除花指令－》obsidium   
提示去除了一个花指令，这时再看代码窗口。变成了这样  
0040136C 8BEC mov ebp, esp  
0040136E 6A FF push -1  
00401370 68 B0404000 push 004040B0  
00401375 68 A01E4000 push 00401EA0  
0040137A 64:A1 0000000>mov eax, dword ptr fs:\[0\]  
00401380 50 push eax  
00401381 64:8925 00000>mov dword ptr fs:\[0\], esp  
00401388 83EC 58 sub esp, 58  
0040138B 53 push ebx  
0040138C 56 push esi  
0040138D 57 push edi  
0040138E 8965 E8 mov dword ptr \[ebp-18\], esp  
00401391 FF15 30404000 call dword ptr \[404030\] ; kernel32.GetVersion  
00401397 33D2 xor edx, edx  
00401399 8AD4 mov dl, ah  
0040139B 8915 D4554000 mov dword ptr \[4055D4\], edx  
004013A1 8BC8 mov ecx, eax  
004013A3 81E1 FF000000 and ecx, 0FF  
004013A9 890D D0554000 mov dword ptr \[4055D0\], ecx  
004013AF C1E1 08 shl ecx, 8  
004013B2 03CA add ecx, edx  
004013B4 890D CC554000 mov dword ptr \[4055CC\], ecx    
  
看来已经到了 OEP 的第二句无疑了。  
那么 OEP 自然是 0000136B（断在了 0040136C,看上面代码的第一行的地址，那么 OEP 当然是他的上一个指令减去基址（00400000），也就是 0000136B）了。  
确定了 OEP，下一步当然是 DUMP 了。Od 有个插件就可以 DUMP。在这里就不使用 LORDPE＋IMPORTREC 了。  
选择 插件－》OllyDump－》Dump Debugged Process 在弹出的窗体的 Modify 里栏里填 136B(你打开时他默认是你当前断的位置，也就是 136C,你要修正) 点 dump 随便填个名字点保存即可。在这里填了 aaa.exe. 将这个文件存在了原来程序的那个目录里.这里用 peid 查看 aaa.exe.已经变了 VC6.0 了。看来壳已经脱好了，运行一下，也正常。脱壳到这一步算是完成了。  
  
            
脱壳完成，然后下一步，自然是提取数据了。提取数据，尤其是资源文件，当然少不了 ExeScope 了。用 EXESCOPE 打开 aaa.exe. 依次展开 资源->RC 数据->200 此时右边出现了十六进制和 ANSICC 数据。点文件－》导出，随便选个文件夹取个文字，在这里，选桌面，文件名字为 aaa.rc 。在这个文件的资源里除了 RCDATA 就是图标了，那 MID 在 RC 数据里，基本上可能性已经非常大了。不过还要确定。用十六进制编辑工具，010Editor 打开 aaa.rc.(当然如果更喜欢用 UltraEditor 这没关系的)他们两个差不多哈。（本来是准备看网上如果有什么 RCDATA 数据读取器的话，就直接下下来用的，但找了半天没有找到，所以只好手动从 RC 数据里提取 MID 文件了。  
在提取之前，当然要明白 MID 文件的格式 。   
  
每个 Mid 文件的开头都有如下内容，它们的十六进制代码为:“4d 54 68 64 00 00 00 06 ff ff nn nn dd dd”。  
前四个是 ASCII 字符“MThd”是用来鉴别是否 Mid 文件，而随后的四个字节是指明文件头描述部分的字节数，MID 的文件头占 6 个字节，所以这里一定是“00 00 00 06”，以下是剩余部分（文件头）的含义  
ff ff 指定 MID 的格式 00 00 表示 单音轨 00 01 多音轨同步 00 02 多音轨不同步  
nn nn 指定 MID 的轨道数   
dd dd 指定一个四分音符的 TICK 值，用于控制节奏。  
以上就是 MID 的文件头。紧跟着文件头的就是数据区了  
MID 的数据是由若干个格式相同的子数据构成的，这些子数据在多音轨的格式中记录了一个轨道的所有信息。多加一个音轨，就简单地把数据追加在前一音轨的后面就可以了，不过不要忘记更改文件头中的 nn nn(轨道数)。  
  
音轨，都以“4D 54 72 6B”开头，它其实是 ASCII 字符“MTrk”，其后跟着一个 4 个字节的整数，它标志了该轨道的字节数，这不包括前面的 4 个字节和本身的 4 个字节。   
好的，知道了以上的知识就差不多了，至于其它的格式对于提取 MID 来说没什么用处，在这里就不再介绍了。  
从上面的介绍可知。MID 文件是以 “MThd”开始的。那么我们就在刚才打开的 aaa.exe 搜索“MThd”注意大小写是区分的。找到了一处。在 13F3A 处，往后看四个字节 正好是 00 00 00 06 再往后二个字节 接照前面的介绍，指的是 单音轨，再往后二个字节 00 01 指只有一个轨道。再往后二个字节 00 61 这个指的是 四分音符的 TICK 值（节奏），到这里头文件完了，再往下进入了数据区，看看前四个字节果然是 MTrk.因为是单音轨，所以只会出现这一个 MTrk. 我们只要将 MID 的头部分和这一个 音轨数据提取出来就可以了。要想将音轨数据提取出来，还得接着刚才的继续往下看。在 MTrk 后面是 00 00 17 80 从前面的介绍可知， 这个值表示，这个音轨的数据量， 也就是说，从这四个字节后面开始的 1780(十六进制)个字节就是音轨的数据了。于是我们就可以算一下，这个音轨数据区的结尾（也就是 MID 的结尾了，因为这个 MID 只有一个音轨）的位置了。这四个字节后面的第一个位置是 13F50，那么 13f50+1780 ＝ 156D0   
那么这个 MID 文件的数据自然是从 13F3A 到 156D0 了 在编辑器里，选中这片区域 CTRL＋C。 点文件－》新建 粘贴， 然后 CTRL＋S 保存成 aaa.mid. 打开 aaa.mid 音乐已经出来了。 到这里，所有工作都做完了

上传的附件
| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || 唐诗三百首 1.2.rar \[附件请到论坛下载：[`bbs.pediy.com/showthread.php?t=80863`](http://bbs.pediy.com/showthread.php?t=80863)\]   
 |