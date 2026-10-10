

# Crunch V5.0 脱壳——Crunch.exe 主程序

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-537.htm`](https://www.pediy.com/kssd/pediy07/pediy7-537.htm)

Crunch V5.0 脱壳——Crunch.exe 主程序  
   
   
下载页面：  http://www.bitarts.com/index.htm  
软件大小：  5458 KB  
软件语言：  英文  
软件类别：  国外软件 / 共享版 / 压缩解压  
应用平台：  Win9x/NT/2000/XP  
推荐等级：  \*\*\*\*  
软件简介：  Crunch enables developers of Windows software to compress a program by up to 70% and still run without any performance difference. Crunch adds a layer of anti-debug routines into your program, preventing novice crackers from viewing or tampering with your software.  
  
【作者声明】：只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教！  
  
【调试环境】：WinXP、Ollydbg、PEiD、LordPE  
  
—————————————————————————————————   
【脱壳过程】：  
            
  
设置 Ollydbg 忽略所有异常，用 IsDebug 插件去掉 Ollydbg 的调试器标志  
PEiD Sign：  
\[Crunch V5.0 -> Bitarts\]  
signature = EB 15 03 00 00 00 06  
ep\_only = true  
————————————————————————  
一、脱壳的最佳时机  
  
  
004A5000    EB 15           jmp short Crunch.004A5017  
//进入 Ollydbg 后暂停在这  
004A5002    0300            add eax,dword ptr ds:\[eax\]  
  
F9 运行，弹出 Licence 窗口。下断：HE LoadLibraryA，然后点击 Continue  
中断后  去掉断点，Alt+F9 返回 004AB9B2 处  
  
004AB9AE    FFD7            call edi  
004AB9B0    FFD0            call eax  ; LoadLibraryA  
004AB9B2    5D              pop ebp ; Crunch.0048B000  
//返回这里  EBP=0048B000-基址 00400000＝0008B000  ★  Import Table RVA  
004AB9B3    EB 08           jmp short Crunch.004AB9BD  
  
此时代码已经解开，IAT 已经还原，正是脱壳的好时机！  
运行 LordPE 完全 Dump 出这个进程。  
  
  
—————————————————————————————————  
二、OEP  
  
  
找 OEP 很简单，可以使用 ESP 定律、第 2 区段内存断点、异常等方法  
也可以 Ctrl+S 在整个段块搜索命令序列：  
  pop dword ptr fs:\[0\]  
  add esp,4  
  pop ebp  
  popad  
  retn  
找到在 004A5BFD 处，设置硬件执行断点，Shift+F9 运行后中断  
  
004A5BF5    8B6424 08       mov esp,dword ptr ss:\[esp+8\]  
004A5BF9    8B6C24 08       mov ebp,dword ptr ss:\[esp+8\]  
004A5BFD    64:8F05 0000000>pop dword ptr fs:\[0\]  
//中断在这里  
004A5C04    83C4 04         add esp,4  
004A5C07    5D              pop ebp  
004A5C08    61              popad  
004A5C09    C3              retn  
//飞向光明之巅  ^O^  
  
  
004501B8    55              push ebp  
//OEP  
004501B9    8BEC            mov ebp,esp  
004501BB    6A FF           push -1  
004501BD    68 28654700     push Crunch.00476528  
004501C2    68 58044500     push Crunch.00450458    ; jmp to msvcrt.\_except\_handler3  
004501C7    64:A1 00000000  mov eax,dword ptr fs:\[0\]  
004501CD    50              push eax  
004501CE    64:8925 0000000>mov dword ptr fs:\[0\],esp  
004501D5    83EC 68         sub esp,68  
004501D8    53              push ebx  
004501D9    56              push esi  
004501DA    57              push edi  
004501DB    8965 E8         mov dword ptr ss:\[ebp-18\],esp  
004501DE    33DB            xor ebx,ebx  
004501E0    895D FC         mov dword ptr ss:\[ebp-4\],ebx  
004501E3    6A 02           push 2  
004501E5    FF15 40C44800   call dword ptr ds:\[48C440\]; msvcrt.\_\_set\_app\_type  
  
  
—————————————————————————————————  
三、PE 修正  
  
  
用 LordPE 修正 dumped.exe 的 OEP RVA=000501B8，Import Table RVA=0008B000  
可以看到 Export Table 原来在 RVA 00082B50 处，修正之  
重定位表没有破坏，直接修正重定位表 RVA=00099000，Size=5E24  
用 LordPE 把 IAT RVA 清 0，当然也可以把其数据挪移入.idata 段  
删除最后的.crt 和.edata 两个壳区段，用 WinHex 把其数据删掉  
OK，脱壳完成啦。  
  
BTW：主程序脱壳后的破解有意思，有兴趣的兄弟试试。  
  
  
—————————————————————————————————                                   
         ,     \_/   
        /| \_.-~/            \\\_     ,        青春都一晌  
       ( /~   /              \\~-.\_ |\\  
       \`\\\\  \_/                \\   ~\\ )          忍把浮名   
   \_-~~~-.)  )\_\_/;;,.          \\\_  //'  
  /'\_,\\   --~   \\ ~~~-  ,;;\\\_\_\_(  (.-~~~-.        换了破解轻狂  
 \`~ \_( ,\_..--\\ (     ,;'' /    ~--   /.\_\`\\   
  /~~//'   /' \`~\\         ) /--..\_, )\_  \`~  
  "  \`~"  "      \`"      /~'\`\\    \`\\\\~~\\     
                         "     "   "~'  ""  
      
           UnPacKed By :  fly  
            2005-04-14 18:30 

 4dd466d0e1e0cc123afbe504da2462ca