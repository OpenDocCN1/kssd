

# Divx Avi Asf Wmv Wma Rm Rmvb V3.23 

> 原文：[`www.pediy.com/kssd/pediy06/pediy6343.htm`](https://www.pediy.com/kssd/pediy06/pediy6343.htm)

梦幻 Ollydbg 之 ACPr 修复篇——Divx Avi Asf Wmv Wma Rm Rmvb V3.23   
    
    
    
下载页面：  http://www.skycn.com/soft/11574.html    
软件大小：  1322 KB   
软件语言：  简体中文   
软件类别：  国产软件 / 共享版 / 文件修复   
应用平台：  Win9x/NT/2000/XP   
加入时间：  2003-08-28 11:11:24   
下载次数：  146813   
推荐等级：  \*\*\*   
开 发 商：  http://www.cnghost.com/   
  
【软件简介】：只需轻松的一次点击就可以修复不能拖动的或者不能播放的 divx avi asf wmv wma rm rmvb 文件。Divx Avi Asf Wmv Wma Rm Rmvb 修复器可以修复你通过 http,ftp,mms,rtsp 方式由于某些原因没有下载完全的 divx avi asf wmv wma rm rmvb 文件。修复后的文件可以流畅的播放，自由的拖动。Divx Avi Asf Wmv Wma Rm Rmvb 修复器也可以修复在播放过程中不能拖动的 divx avi asf wmv wma rm rmvb 文件。修复后的文件可以随意的拖动。Divx Avi Asf Wmv Wma Rm Rmvb 修复器还有另一个功能，他可以强行修复部分损坏的 divx avi asf wmv wma rm rmvb 文件。修复后的文件可以跳过坏的数据块，继续播放。如果一些播放器，例如 Mediaplayer,realplayer 提示 dvix avi asf wmv wma rm rmvb 文件不能播放或文件损坏，都可以尝试用 Divx Avi Asf Wmv Wma Rm Rmvb 修复器来修复。   
  
【作者声明】：初学 Crack，只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教！   
  
【调试环境】：WinXP、flyODBG、PEiD、LordPE、WinHex、ImportREC、HIEW   
  
—————————————————————————————————    
【脱壳过程】：   
             
             
这个 V3.23 版还是 ACProtect V1.09 加壳的，偶昨天才拿到手。前几天找了 VideoFixer V3.21 的入口，但是偶当时没能修复好“丢失”代码，这两天多方请教高人，加上偶不断调试，终于有了点结果！外面落起了今年入冬的第一场大雪，飞飞扬扬漫天琼玉，好冷的夜晚。来杯酒吧，庆祝偶的小有进展！感谢 jingulong、mikelong 、playar 等各位兄弟的指教！偶很菜，谢谢兄弟们的诲人不倦！来，干一杯！:-)   
  
       
一、寻找入口                     具体过程请参考偶前几天发的相关笔记   
  
  
006E7F0B     CD 01                int 1   
                                  ====>ACProtect 典型异常   
006E7F0D     40                   inc eax   
                                  ====>直接此处下断！Shift+F9 断在这里！CTRL+F9 执行到返回！   
006E7F0E     40                   inc eax   
006E7F0F     0BC0                 or eax,eax   
006E7F11     0F85 B6000000        jnz videofix.006E7FCD   
  
006E800E     61                   popad   
006E800F     61                   popad   
006E8010     C3                   retn   
                                  ====>返回到 006EBC8B   
  
006EBC8B     E9 05000000          jmp videofix.006EBC95   
  
下面都是注意用 F8 带过的地方！   
  
006EBD6A     E8 A7C2FFFF          call videofix.006E8016   
  
006EBE45     E8 FAC6FFFF          call videofix.006E8544   
  
006EBF86     E8 B9C5FFFF          call videofix.006E8544   
  
006EBFA4     E8 D0B8FFFF          call videofix.006E7879   
  
006EC05C     E8 ACC7FFFF          call videofix.006E880D   
  
006EC099     E8 26EEFFFF          call videofix.006EAEC4   
  
006EE409     E8 018DFFFF          call videofix.006E710F   
  
006EE535     E8 DC9AFFFF          call videofix.006E8016   
  
006EE57A     E8 FCBAFFFF          call videofix.006EA07B   
  
006EE5AB     E8 949FFFFF          call videofix.006E8544   
  
  …… …… 省 略 …… ……       
  
006FC802     E8 BD91FEFF          call videofix.006E59C4   
                                  ====>F8 带过   
006FC807     8B85 11BC4100        mov eax,dword ptr ss:\[ebp+41BC11\]   
                                  ====>EAX\=000DBA0A   
006FC80D     0385 383F4000        add eax,dword ptr ss:\[ebp+403F38\]   
                                  ====>EAX\=000DBA0A + 00400000=004DBA0A   这就是返回的入口值   
006FC813     8985 11BC4100        mov dword ptr ss:\[ebp+41BC11\],eax    
                                  ====>\[ebp+41BA4C\]＝\[006FCC11\]＝EAX=004DBA0A   
在内存\[006FCC11\]处下 内存访问 断点。   F9 运行，数秒钟后程序自动停下！ :-)   
  
006FCC0B     FF25 11CC6F00        jmp dword ptr ds:\[6FCC11\] ; videofix.004DBA0A   
                                  ====>飞向光明之巅！   
  
————————————————————————   
  
004DBA0A     53                   push ebx   
                                  ====>可以在这儿用 LordPE 完全 DUMP 这个进程   
004DBA0B     B8 4CB54D00          mov eax,videofix.004DB54C   
004DBA10     E8 23B6F2FF          call videofix.00407038   
004DBA15     8B1D A0DE4D00        mov ebx,dword ptr ds:\[4DDEA0\]    
004DBA1B     8B03                 mov eax,dword ptr ds:\[ebx\]   
004DBA1D     E8 C2B0F8FF          call videofix.00466AE4   
  
  
—————————————————————————————————   
  
但是偶运行修复输入表后的程序出错。呵呵，这是壳的效果！   
  
00404735     E8 D9E92D00          call 006E3113   
                                  ====>这里出错！进去看看   
  
006E3113     FF25 BC501400        jmp dword ptr ds:\[1450BC\]//出错    
006E3119     FF25 C0501400        jmp dword ptr ds:\[1450C0\]//出错    
006E311F     FF25 C4501400        jmp dword ptr ds:\[1450C4\]//出错    
  
原来脱壳后\[1450BC\]等里面的代码全部不是正确的数据！几百条代码没了，程序当然不“干活”了。:-(   
  
  
—————————————————————————————————   
  
二、寻找“迷失”的代码   
  
  
就是这部分，偶看了其他教程后不得其解，原来…… 暂时别问偶是如何找到这个地方的，如果你能为了这个问题如偶般调试上百次就会知道了，偶很菜，所以只能用笨方法：不断调试！   
  
因为偶在原程序代码里还原丢失代码脱壳后发现图标竟然也没了，所以就费点事，给程序“增肥”一下。复制 videofixer.exe 备用。让偶来对 videofixer.exe 原程序动个小“手术”：用 LordPE 给其加个新的区段：大小 00070000 ，然后用 WinHex 在程序末尾粘贴 70000 长度的 00 字节。设置 LordPE 的“重建”选项，只保留“验证 PE”一个选项，最后用 LordPE 重建“增肥”后的 复件 videofixer.exe 。   
  
用 flyODBG 载入“增肥”后的 复件 videofixer.exe  开始还原丢失代码！取消以前的所有断点。   
  
77F7F571     C3                   retn   
                                  ====>进入 OD 后断在这！下命令：G 006E5BAB   
  
弹出“压缩”提示后停在 006E2000  继续：G 006E5BAB  约 10 秒后偶们来到目的地！   
  
006E5BAB     FF95 F1434000        call dword ptr ss:\[ebp+4043F1\]   
                                  ====>停在这里！   
006E5BB1     8BF8                 mov edi,eax   
                                  ====>EAX\=00145008  把返回值改为：00705008  狸猫换太子:-)    
006E5BB3     81C7 A00F0000        add edi,0FA0   
006E5BB9     50                   push eax   
006E5BBA     B9 70170000          mov ecx,1770   
006E5BBF     8DB5 05204000        lea esi,dword ptr ss:\[ebp+402005\]   
006E5BC5     F3:A4                rep movs byte ptr es:\[edi\],byte ptr ds:\[esi\]   
006E5BC7     5A                   pop edx   
006E5BC8     8BF2                 mov esi,edx   
006E5BCA     81C6 A00F0000        add esi,0FA0   
006E5BD0     8BFE                 mov edi,esi   
006E5BD2     B9 70170000          mov ecx,1770   
006E5BD7     AC                   lods byte ptr ds:\[esi\]   
006E5BD8     32C3                 xor al,bl   
006E5BDA     AA                   stos byte ptr es:\[edi\]   
006E5BDB     E2 FA                loopd short videofix.006E5BD7   
006E5BDD     8BFA                 mov edi,edx   
006E5BDF     B9 E8030000          mov ecx,3E8   
006E5BE4     8BC2                 mov eax,edx   
006E5BE6     05 A00F0000          add eax,0FA0   
006E5BEB     AB                   stos dword ptr es:\[edi\]   
006E5BEC     83C0 06              add eax,6   
006E5BEF     E2 FA                loopd short videofix.006E5BEB   
006E5BF1     B9 E8030000          mov ecx,3E8   
006E5BF6     8DBD 05204000        lea edi,dword ptr ss:\[ebp+402005\]   
006E5BFC     8BC2                 mov eax,edx   
006E5BFE     66:C707 FF25         mov word ptr ds:\[edi\],25FF   
006E5C03     8947 02              mov dword ptr ds:\[edi+2\],eax   
006E5C06     83C7 06              add edi,6   
006E5C09     83C0 04              add eax,4   
006E5C0C     E2 F0                loopd short videofix.006E5BFE   
006E5C0E     60                   pushad   
006E5C0F     E8 00000000          call videofix.006E5C14   
006E5C14     5E                   pop esi   
006E5C15     83EE 06              sub esi,6   
006E5C18     B9 92000000          mov ecx,92   
006E5C1D     29CE                 sub esi,ecx   
006E5C1F     BA 171F86BE          mov edx,BE861F17   
006E5C24     C1E9 02              shr ecx,2   
006E5C27     83E9 02              sub ecx,2   
006E5C2A     83F9 00              cmp ecx,0   
006E5C2D     7C 1A                jl short videofix.006E5C49   
006E5C2F     8B048E               mov eax,dword ptr ds:\[esi+ecx\*4\]   
006E5C32     8B5C8E 04            mov ebx,dword ptr ds:\[esi+ecx\*4+4\]   
006E5C36     33C3                 xor eax,ebx   
006E5C38     C1C0 05              rol eax,5   
006E5C3B     2BC2                 sub eax,edx   
006E5C3D     81C2 CF79F835        add edx,35F879CF   
006E5C43     89048E               mov dword ptr ds:\[esi+ecx\*4\],eax   
006E5C46     49                   dec ecx   
006E5C47     EB E1                jmp short videofix.006E5C2A   
006E5C49     61                   popad   
006E5C4A     61                   popad   
006E5C4B     C3                   retn   
  
OK!所有丢失代码全部还原在偶新加的区段里面！呵呵，看你往哪跑！:-)  F9 运行程序   
  
006E7F0B     CD 01                int 1   
                                  ====>中断在典型异常处    
006E7F0D     40                   inc eax   
                                  ====>直接此处下断！Shift+F9 断在这里！   
  
在内存 004DBA0A 处下内存访问断点，F9 运行程序，直达入口！DUMP 出程序！   
  
再看看原先丢失的代码：   
  
00404735     E8 D9E92D00          call 复件\_vid.006E3113   
  
006E3113     FF25 BC507000        jmp dword ptr ds:\[7050BC\]   ; 复件\_vid.007060B6   
006E3119     FF25 C0507000        jmp dword ptr ds:\[7050C0\]   ; 复件\_vid.007060BC   
006E311F     FF25 C4507000        jmp dword ptr ds:\[7050C4\]   ; 复件\_vid.007060C2   
006E3125     FF25 C8507000        jmp dword ptr ds:\[7050C8\]   ; 复件\_vid.007060C8   
006E312B     FF25 CC507000        jmp dword ptr ds:\[7050CC\]   ; 复件\_vid.007060CE   
   …… …… 省 略 …… ……      迷失方向的游子终于找到家啦！:-) :-)   
  
————————————————————————   
  
重新运行 videofixer.exe，运行 ImportREC，选择这个进程。把 OEP 改为 000DBA0A，点 IT AutoSearch，点“Get Import”，函数无效，用“追踪层次 3”全部修复。FixDump！   
  
  
—————————————————————————————————   
  
三、仿真 OEP 处的 Stolen Code    
  
  
偶看过几个 ACProtect 加壳的程序，发现 OEP 处的 6 个字节代码均被抽去了，只有某些 OEP 为 00401000 的程序没有被 ACProtect 动手脚。偶学习了 jingulong 兄的《用 OD 对新版 Aspr 加壳程序的手动脱壳及修复 》试试仿真 OEP 处的 Stolen Code 。   
  
EBP\=0012FFC0 //参考值   
ESP\=0012FFB0 //参考值   
  
用 HIEW 把 004DBA0A 的前 6 个字节改为：55 8B EC 83 C4 F0  再用 LordPE 把 OEP 改为 000DBA04，重建 PE。   
  
仿真 Stolen Code 后的 OEP:          错误之处请大家指教！   
    
004DBA04       55                 push ebp   
004DBA05       8BEC               mov ebp,esp   
004DBA07       83C4 F0            add esp,-10   
004DBA0A       53                 push ebx   
004DBA0B       B8 4CB54D00        mov eax,dumped\_.004DB54C   
004DBA10       E8 23B6F2FF        call dumped\_.00407038   
  
  
脱壳后的程序用 PEiD 看是 Delphi 编写。虽然脱壳后即提示过期，但是偶不用这个东东，只为脱壳。 :-)   
  
最后的问题：脱壳后的程序虽然可以修改、反编译并且不反 FileMon、TRW 等工具了，但是对于非 Explorer.exe、CMD.exe 的上级进程依然是 KILL，好厉害的 ACProtect！外面的雪更大了，无语 …… ……   
  
    
—————————————————————————————————   
       
                                   
         ,     \_/    
        /| \_.-~/            \_     ,        青春都一饷   
       ( /~   /              ~-.\_ |   
       \`\\  \_/                   ~ )          忍把浮名    
   \_-~~~-.)  )\_\_/;;,.          \_  //'    
  /'\_,   --~    ~~~-  ,;;\_\_\_(  (.-~~~-.        换了破解轻狂   
 \`~ \_( ,\_..-- (     ,;'' /    ~--   /.\_\`    
  /~~//'   /' \`~         ) /--..\_, )\_  \`~    
  "  \`~"  "      \`"      /~'\`    \`\\~~      
                         "     "   "~'  ""   
  
       
  
            Cracked By 巢水工作坊——fly \[OCN\]\[FCG\]   
  
                    2003-11-07  00:00   
 4dd466d0e1e0cc123afbe504da2462ca