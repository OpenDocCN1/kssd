

# JDPack\[铁甲\] V1.01

> 原文：[`www.pediy.com/kssd/pediy06/pediy6646.htm`](https://www.pediy.com/kssd/pediy06/pediy6646.htm)

                                 
【作者声明】：只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教！  
           
【调试环境】：WinXP、Ollydbg1.10、WinHex、LordPE、ImportREC 

【实例下载】：点击此处下载（或鼠标右键另存为）  
               
—————————————————————————————————   
【脱壳过程】：  
            
           
           
JDPack\[铁甲\] 算是弱壳了，已经很久没有升级了。用其加壳的 DLL 的重定位表没有加密，但是壳运行后会把重定位表的数据清零，我们要做的就是找到重定位表的地址，然后阻止壳把重定位数据清零！  
—————————————————————————————————  
一、阻止壳对重定位表清零+使用 ESP 定律到达 OEP  
               
               

> ```
> 代码:
> 003B7000     60                  pushad//进入 OD 后停在这
> 003B7001     E8 00000000         call EdrLib.003B7006
> 003B7006     5D                  pop ebp
> 003B7007     8BD5                mov edx,ebp//ESP=0006F8CC
> 003B7009     81ED C62B4000       sub ebp,402BC6
> 003B700F     2B95 61344000       sub edx,dword ptr ss:\[ebp+403461\]
> 003B7015     81EA 06000000       sub edx,6
> 003B701B     8995 65344000       mov dword ptr ss:\[ebp+403465\],edx
> 003B7021     83BD 69344000 00    cmp dword ptr ss:\[ebp+403469\],0
> 003B7028     0F85 BC030000       jnz EdrLib.003B73EA
> 
> ```
> 
>   

  
             
F7 走几步，003B7007 处时 ESP=0006F8CC，可以使用 ESP 定律，快速到达 OEP  

> ```
> 代码:
> 0006F8CC  01 00 00 00 FC F8 06 00 08 F9 06 00 EC F8 06 00   ................
> 
> ```
> 
>   

  
在 0006F8CC 转存处的 4 个字节上下 硬件访问->Word 断点，F9 就可以到 OEP 了。  
但我们现在要寻找重定位表，所以继续向下走几步吧。  
             

> ```
> 代码:
> 003B702E     C785 69344000 01000>mov dword ptr ss:\[ebp+403469\],1
> 003B7038     B9 88070000         mov ecx,788
> 003B703D     8DB5 182C4000       lea esi,dword ptr ss:\[ebp+402C18\]
> 003B7043     8A85 60344000       mov al,byte ptr ss:\[ebp+403460\]
> 003B7049     8A1E                mov bl,byte ptr ds:\[esi\]
> 003B704B     32C3                xor al,bl
> 003B704D     8806                mov byte ptr ds:\[esi\],al
> 003B704F     889D 60344000       mov byte ptr ss:\[ebp+403460\],bl
> 003B7055     46                  inc esi
> 003B7056     E2 EB               loopd short EdrLib.003B7043//解码
> 
> ```
> 
>   

  
             

> ```
> 代码:
> 003B7058     9C                  pushfd//F4 到这，程序解码完毕
> 003B7059     58                  pop eax
> 003B705A     F6C4 01             test ah,1
> 003B705D     74 07               je short EdrLib.003B7066
> 
> ```
> 
>   

  
             
提供一个简便的方法来寻找 JDPack 处理重定位数据的代码。Ctrl+S 在整个段块搜索命令序列：  

> ```
> 代码:
> sub ecx,8
> shr ecx,1
> 
> ```
> 
>   

  
找到在 003B7134 处，在其上的 003B7120 处下断，F9 运行，中断在 003B7120 处  
             

> ```
> 代码:
> 003B70FC     83BD E9314000 00    cmp dword ptr ss:\[ebp+4031E9\],0
> //\[ebp+4031E9\]=00006000    ★    重定位表的 RVA
> 003B7103     0F84 8A000000       je EdrLib.003B7193
> 003B7109     8B95 65344000       mov edx,dword ptr ss:\[ebp+403465\]
> //\[ebp+403465\]=003B0000          当前基址
> 003B710F     8B85 E5314000       mov eax,dword ptr ss:\[ebp+4031E5\]
> //\[ebp+4031E5\]=00400000          原来基址
> 003B7115     2BD0                sub edx,eax
> 003B7117     74 7A               je short EdrLib.003B7193
> //如与映像基址不符则重定位处理！
> 003B7119     8BC2                mov eax,edx
> 003B711B     C1E8 10             shr eax,10
> 003B711E     33DB                xor ebx,ebx
> 003B7120     8BB5 E9314000       mov esi,dword ptr ss:\[ebp+4031E9\]
> 003B7126     03B5 65344000       add esi,dword ptr ss:\[ebp+403465\]
> 003B712C     833E 00             cmp dword ptr ds:\[esi\],0
> 003B712F     74 62               je short EdrLib.003B7193
> //重定位表处理结束，跳转
> 003B7131     8B4E 04             mov ecx,dword ptr ds:\[esi+4\]
> 003B7134     83E9 08             sub ecx,8
> 003B7137     D1E9                shr ecx,1
> 003B7139     8B3E                mov edi,dword ptr ds:\[esi\]
> 003B713B     03BD 65344000       add edi,dword ptr ss:\[ebp+403465\]
> 003B7141     83C6 08             add esi,8
> 003B7144     66:8B1E             mov bx,word ptr ds:\[esi\]
> 003B7147     C1EB 0C             shr ebx,0C
> 003B714A     83FB 01             cmp ebx,1
> 003B714D     74 0C               je short EdrLib.003B715B
> 003B714F     83FB 02             cmp ebx,2
> 003B7152     74 16               je short EdrLib.003B716A
> 003B7154     83FB 03             cmp ebx,3
> 003B7157     74 20               je short EdrLib.003B7179
> 003B7159     EB 2C               jmp short EdrLib.003B7187
> 003B715B     66:8B1E             mov bx,word ptr ds:\[esi\]
> 003B715E     81E3 FF0F0000       and ebx,0FFF
> 003B7164     66:01043B           add word ptr ds:\[ebx+edi\],ax
> 003B7168     EB 1D               jmp short EdrLib.003B7187
> 003B716A     66:8B1E             mov bx,word ptr ds:\[esi\]
> 003B716D     81E3 FF0F0000       and ebx,0FFF
> 003B7173     66:01143B           add word ptr ds:\[ebx+edi\],dx
> 003B7177     EB 0E               jmp short EdrLib.003B7187
> 003B7179     66:8B1E             mov bx,word ptr ds:\[esi\]
> 003B717C     81E3 FF0F0000       and ebx,0FFF
> 003B7182     01143B              add dword ptr ds:\[ebx+edi\],edx
> 003B7185     EB 00               jmp short EdrLib.003B7187
> 003B7187     66:C706 0000        mov word ptr ds:\[esi\],0
> //就是这里把原来的重定位表数据清 0！★  NOP 掉！
> 003B718C     83C6 02             add esi,2
> 003B718F     E2 B3               loopd short EdrLib.003B7144//循环处理
> 003B7191     EB 99               jmp short EdrLib.003B712C
> 
> ```
> 
>   

  
             
把 003B7187 处 NOP 掉，在 003B7193 处下断，F9 运行，中断，重定位处理完毕。  
此时 ESI=003B63B0，就是重定位表的结束地址！  
             

> ```
> 代码:
> 003B7193     8D9D 89334000       lea ebx,dword ptr ss:\[ebp+403389\]//ESI=003B63B0 ★
> 003B7199     53                  push ebx
> 003B719A     FF95 E8334000       call dword ptr ss:\[ebp+4033E8\]
> 
> ```
> 
>   

  
             
重定位表 RVA=00006000，大小＝003B63B0－003B6000＝000003B0  
             
现在我们可以 F9 运行了，继续使用 ESP 定律，别忘了在 003B7007 处对 0006F8CC 下的硬件访问断点。  
             

> ```
> 代码:
> 003B7415     50                  push eax;EdrLib.003B11C9//F9 运行，中断在这里
> 003B7416     C3                  retn//飞向光明之点巅！
> 
> ```
> 
>   

  
             
         

> ```
> 代码:    
> 003B11C9     55                  push ebp//OEP ★
> 003B11CA     8BEC                mov ebp,esp
> 003B11CC     53                  push ebx
> 003B11CD     8B5D 08             mov ebx,dword ptr ss:\[ebp+8\]
> 003B11D0     56                  push esi
> 003B11D1     8B75 0C             mov esi,dword ptr ss:\[ebp+C\]
> 003B11D4     57                  push edi
> 003B11D5     8B7D 10             mov edi,dword ptr ss:\[ebp+10\]
> 003B11D8     85F6                test esi,esi
> 003B11DA     75 09               jnz short EdrLib.003B11E5
> 
> ```
> 
>   

  
               
用 LordPE 选中 Ollydbg 的 loaddll.exe 的进程，在下面的列表里选择 EdrLib.dll，然后完整脱壳，得到 dumped.dll。  
             
             
—————————————————————————————————  
二、输入表  
               
                          
随便从程序找一个 API 调用，如：  
003B10FD     FF15 20403B00       call dword ptr ds:\[3B4020\]; kernel32.GetVersion  
在转存中跟随 3B4020，上下看到许多函数地址，很明显的可以找到 IAT 开始和结束的地址：  
              

> ```
> 代码:            
> 003B3FF0  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
> 003B4000  1D 51 C4 77 1C 3A C4 77 3E E7 C4 77 CC D2 C4 77  .Q.w.:.w>..w...w
>              
> 003B40B0  CE 7C E5 77 05 74 E5 77 F9 81 E5 77 EB 41 E4 77  .|.w.t.w...w.A.w
> 003B40C0  66 C8 E5 77 3E 18 F6 77 00 00 00 00 00 00 00 00  f..w>..w........
> 
> ```
> 
>   

  
               
开始地址＝003B4000  
结束地址＝003B40C9  
               
运行 ImportREC，注意：去掉“使用来自磁盘的 PE 部首”的选项！  
               
选中 Ollydbg 的 loaddll.exe 的进程，然后点“选取 DLL”，选择 EdrLib.dll，填入 RVA＝00004000、大小＝000000C8 ，点“Get Import”，得到输入表。改 OEP=000011C9，FixDump！  
             
             
—————————————————————————————————  
三、PE 修正  
               
               
用 LordPE 修正 dumped\_.dll 的重定位表 RVA=00006000、大小＝000003B0，保存之。  
再用 LordPE 修正 dumped\_.dll 的基址为 003B0000，OK，脱壳完成啦。  
如果想简单优化一下，可以删除.jdpack 区段，再用 LordPE 去掉“清除重定位表”选项重建 PE 即可。  
                 
             
—————————————————————————————————      

> ```
> 代码:                           
>          ,     \_/ 
>         /| \_.-~/            \\\_     ,        青春都一晌
>        ( /~   /              \\~-.\_ |\\
>        \`\\\\  \_/                \\   ~\\           忍把浮名 
>    \_-~~~-.)  \_\_/;;,.          \\\_  //'
>   /'\_,\\   --~   \\ ~~~-  ,;;\\\_\_\_(  (.-~~~-.        换了破解轻狂
>  \`~ \_( ,\_..--\\ (     ,;'' /    ~--   /.\_\`\\ 
>   /~~//'   /' \`~\\         ![](img/e45fac309594d4f9580d902b0ed40a73.jpg) /--..\_, ![](img/e45fac309594d4f9580d902b0ed40a73.jpg)\_  \`~
>   "  \`~"  "      \`"      /~'\`\\    \`\\\\~~\\   
>                          "     "   "~'  ""
> 
> ```
> 
>   

  
      
  
     Cracked By 巢水工作坊——fly \[OCN\]\[FCG\]\[NUKE\]\[DCM\]  
  
               2004-05-31  15:00  4dd466d0e1e0cc123afbe504da2462ca