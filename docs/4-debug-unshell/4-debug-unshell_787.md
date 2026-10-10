

# Obsidium V1.1.1.4

> 原文：[`www.pediy.com/kssd/pediy06/pediy6644.htm`](https://www.pediy.com/kssd/pediy06/pediy6644.htm)

**Obsidium V1.1.1.4 脱壳——jingulong 兄的 UnPackIt.EXE**  
           
           
           
Obsidium：  [`www.obsidium.de/show.php?download`](http://www.obsidium.de/show.php?download)   
软件大小：  1 MB  
  
【软件简介】：The Obsidium software protection and licensing system allows you to protect your program from unauthorized modifications (i.e. "cracking") and provides you with a reliable yet easy to implement licensing system. 看雪工具栏介绍：“一款优秀的壳，它可以分块加密等。  
  
【作者声明】：只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教！  
  
【调试环境】：WinXP、Ollydbg、PEiD、LordPE、ImportREC 

【实例下载】：点击此处本地下载（或鼠标右键另存为）  
  
—————————————————————————————————   
【脱壳过程】：  
                    
                  
              
因为 Obsidium 对程序代码重定位，使得脱壳有点烦人。壳把原程序的资源放在 00002000，为了省点事不移动资源段，我学习了 ZILOT 大侠脱 Obsidium V1.0.0.61 主程序的方法，最后补上了重定位后的代码段。当然，这样脱壳不算完美，只是能运行罢了。  
  
jingulong 兄可能是为了降低脱壳的难度，所以拿 Win98 的记事本做了这个 UnPackIt，使得脱壳稍微轻松点。Obsidium V1.1.1.4 主程序的脱壳比较麻烦，需要修复加密代码或者补上解密代码段。    
  
—————————————————————————————————  
一、寻找 OEP、Dump 进程  
           
             
老规矩：用 IsDebug 1.4 插件去掉 Ollydbg 的调试器标志。忽略除了“整数被 0 除”之外的所有其他异常选项。  
          

> ```
> 代码:
> 00407000     EB 02               jmp short unpackit.00407004//进入 OD 后停在这
> 003A161B     F7F0                div eax//第 1 次异常
> 003A4FA9     F7F0                div eax//第 2 次异常
> 003A5363     F7F0                div eax//第 3 次异常
> 003A54D9     F7F0                div eax//第 4 次异常   输入表处理
> 003A381D     F7F0                div eax//第 5 次异常
> 003A3BD1     F7F0                div eax//第 6 次异常
> 003A381D     F7F0                div eax//第 7 次异常
> 003A3BD1     F7F0                div eax//第 8 次异常   
> 00408818     F7F0                div eax//第 9 次异常   跳 OEP
> 
> ```
> 
>   

  
  
OK，当第 5 次 003A381D 异常时，Ctrl+G:003A54D9  (第 4 次异常地址)   
到达 003A3BD1 时 Ctrl+F 搜索命令：test word ptr ds:\[esi\],20   

> ```
> 代码:
> 003A64F0     66:F706 2000        test word ptr ds:\[esi\],20//找到这里
> 003A64F5     74 59               je short 003A6550
> 
> ```
> 
>   

   
呵呵，记下这个：003A64F0  ◆      这一步为下面的修复输入表提供点方便啦   
  
☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆  
看看 00408818 处第 9 次异常的堆栈：  
  
0012FF4C    0012FFE0  指针到下一个 SEH 记录  
0012FF50    0040884C  SE 句柄//此处下断  
☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆  
  

> ```
> 代码:
> 0040884C     C8 000000           enter 0,0//此处下断，Shift+F9 断在这
> 00408850     EB 02               jmp short unpackit.00408854
> 
> ```
> 
>   

  
  
Ctrl+F 在当前位置下搜索命令：mov dword ptr ds:\[eax+0B8\],edx  在 004088D9 处  
  

> ```
> 代码:
> 004088D9     8990 B8000000       mov dword ptr ds:\[eax+B8\],edx//此处下断，F9 断在这 EDX=00408B78
> 004088DF     EB 03               jmp short unpackit.004088E4
> 
> ```
> 
>   

  
  
Ctrl+G:00408B78   下断，F9 运行，断下  
  

> ```
> 代码:
> 00408B78     E8 B7000000         call unpackit.00408C34//F7 进入
> 
> ```
> 
>   

  
  

> ```
> 代码:
> 00408C34     F8                  clc
> 00408C35     73 06               jnb short unpackit.00408C3D
> 00408C3D     60                  pushad
> 00408C3E     836C24 20 05        sub dword ptr ss:\[esp+20\],5
> 00408C43     8B4C24 20           mov ecx,dword ptr ss:\[esp+20\]
> 00408C47     F9                  stc
> 00408C48     72 02               jb short unpackit.00408C4C
> 00408C4C     C601 FA             mov byte ptr ds:\[ecx\],0FA
> 00408C4F     EB 04               jmp short unpackit.00408C55
> 00408C55     C741 01 80BAF580    mov dword ptr ds:\[ecx+1\],80F5BA80
> 00408C5C     F8                  clc
> 00408C5D     73 05               jnb short unpackit.00408C64
> 00408C64     BB BC000000         mov ebx,0BC
> 00408C69     F8                  clc
> 00408C6A     73 01               jnb short unpackit.00408C6D
> 00408C6D     B8 E8ED7C07         mov eax,77CEDE8
> 00408C72     C009 0F             ror byte ptr ds:\[ecx\],0F
> 00408C75     C109 5F             ror dword ptr ds:\[ecx\],5F
> 00408C78     F9                  stc
> 00408C79     72 06               jb short unpackit.00408C81
> 00408C81     F8                  clc
> 00408C82     73 03               jnb short unpackit.00408C87
> 00408C87     83C1 04             add ecx,4
> 00408C8A     F8                  clc
> 00408C8B     73 01               jnb short unpackit.00408C8E
> 00408C8E     81E8 8CB4A71A       sub eax,1AA7B48C
> 00408C94     83EB 04             sub ebx,4
> 00408C97     0F85 D5FFFFFF       jnz unpackit.00408C72
> 00408C9E     73 05               jnb short unpackit.00408CA5
> 00408CA5     61                  popad
> 00408CA6     F9                  stc
> 00408CA7     72 04               jb short unpackit.00408CAD
> 00408CAD     C3                  retn//返回 00408B78 这段代码其实就是再次解码 00408B78
> 
> 00408B78     EB 01               jmp short unpackit.00408B7B
> 
> ```
> 
>   

  
  
再次来到 00408B78 处时，Ctrl+F 在 当前位置下 搜索命令： jmp edi  在 00408C12 处，当然是 F2 再下断了，F9 会断在那里的  
  

> ```
> 代码:
> 00408C12     FFE7                jmp edi//跳至 003AAC06
> 
> 003AAC06     E8 00000000         call 003AAC0B
> 003AAC0B     EB 03               jmp short 003AAC10
> 
> ```
> 
>   

  
  
F7 向下走几步就行了  
  

> ```
> 代码:
> 003AAC48     0356 10             add edx,dword ptr ds:\[esi+10\]
> //EDX=000010D3+00A10000=00A110D3 由壳中返回程序的地址    伪 OEP 值  ★
> 003AAC4B     EB 03               jmp short 003AAC50
> 
> 003AAC6B     61                  popad
> 003AAC6C     EB 01               jmp short 003AAC6F
> 
> 003AAC6F     9D                  popfd
> 003AAC70     EB 02               jmp short 003AAC74
> 
> 003AAC74     EB 02               jmp short 003AAC78
> 
> 003AAC78     E9 56646600         jmp 00A110D3//飞向光明之巅！
> 
> ```
> 
>   

  
  
根据原记事本程序补上 Stolen Code，呵呵，偷巧了。  
脱其他 Obsidium 壳程序可以根据程序运行代码、伪 OEP 处的堆栈和寄存器等情况，还原 OEP 处代码。  
  

> ```
> 代码:
> 00A110CC     55                  push ebp//OEP  ★
> 00A110CD     8BEC                mov ebp,esp
> 00A110CF     83EC 44             sub esp,44
> 00A110D2     56                  push esi//上面是补上的 Stolen Code
> 00A110D3     FF15 E463A100       call dword ptr ds:\[A163E4\]//返回这里
> 00A110D9     8BF0                mov esi,eax
> 00A110DB     8A00                mov al,byte ptr ds:\[eax\]
> 00A110DD     3C 22               cmp al,22
> 00A110DF     75 1B               jnz short 00A110FC
> 00A110E1     56                  push esi
> 00A110E2     FF15 F464A100       call dword ptr ds:\[A164F4\]
> 
> ```
> 
>   

  
  
  
运行 LordPE，先完全 Dump 这个进程，然后“区域脱壳”，地址=00A10000，大小=00008000，得到 Region00A10000-00A18000.dmp 文件。用 LordPE 打开 dumped.exe，从磁盘载入 Region00A10000-00A18000.dmp 区段，改其 VOffset=00610000。只保留 LordPE 的“验证 PE”选项，对 dumped.exe 重建 PE。OK，Dump 完毕！  
           
              
—————————————————————————————————  
二、避开 IAT 加密，得到正确的输入表   
          
           
重新载入 unpackit.EXE，还记得刚开始在第 5 次 003A381D 异常时偶搜索的地址吗？003A64F0  
          

> ```
> 代码:
> 003A54D9     F7F0                div eax//第 4 次异常   输入表处理
> 
> ```
> 
>   

  
  
在第 4 次异常时偶们在 003A64F0 处下  内存访问  断点   
然后 Shift+F9 运行，中断数十次就会来到 003A64F0 处，取消内存断点  
  

> ```
> 代码:
> 003A64F0     66:F706 2000        test word ptr ds:\[esi\],20
> //①、改为 test word ptr ds:\[esi\],8  ◆
> 003A64F5     74 59               je short 003A6550
> //②、改为 jnz 003A6550 ◆
> 
> 003A6517     FF75 18             push dword ptr ss:\[ebp+18\]
> 003A651A     FF50 4C             call dword ptr ds:\[eax+4C\]
> 003A651D     85C0                test eax,eax
> 003A651F     74 44               je short 003A6565
> //③、改为 je 003A6550 ◆
> 
> 003A653D     FF76 04             push dword ptr ds:\[esi+4\]
> 003A6540     FF75 18             push dword ptr ss:\[ebp+18\]
> 003A6543     FF50 4C             call dword ptr ds:\[eax+4C\]
> 003A6546     85C0                test eax,eax
> 003A6548     74 1B               je short 003A6565
> //④、改为 je 003A6550 ◆ 
> 
> 003A654E     8907                mov dword ptr ds:\[edi\],eax
> //正确的函数进入正确的地址
> 003A6550     83C6 08             add esi,8
> 003A6553     83C7 04             add edi,4
> 003A6556     FF4D 08             dec dword ptr ss:\[ebp+8\]
> 003A6559     75 95               jnz short 003A64F0//循环
> 
> ```
> 
>   

   
          
因为这个东东的特殊函数不多，所以就不再改了。F9 运行，程序达到第 5 次异常，IAT 处理完毕。  
          
运行 ImportREC，填入 RVA=006162E4，Size=00000240，点“Get Import”，得到输入表。  
有 1 个特殊函数：006163E4  ? 0000  003A6D54//为：GetCommandLineA  
          
其他是填充的垃圾数据，CUT 掉就行了。改 OEP＝006110CC，FixDump！  
删除 00007000 区段，去掉 LordPE 的“脱壳修复”选项，重建 PE 优化一下 dumped\_.exe 即可。  
          
          
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
  
                2004-04-18  20:16 4dd466d0e1e0cc123afbe504da2462ca