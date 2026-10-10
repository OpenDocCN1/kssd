

# Macromedia Dreamweaver MX V6.0 

> 原文：[`www.pediy.com/kssd/pediy06/pediy6678.htm`](https://www.pediy.com/kssd/pediy06/pediy6678.htm)

【作者声明】：只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教！  
               
【调试环境】：WinXP、Ollydbg V1.10、PEiD、LordPE、ImportREC  
               
—————————————————————————————————   
【脱壳过程】：  
            
           
朋友拿来这个东东让我看看，30 天的试用，Vbox 的标记，看其调用的 vboxa.dll 版本是 4.5.0.0，所以知道了是 Vbox V4.5。恰巧很久没写东西了，索性记录一下过程吧。目标程序自己搜索。               
—————————————————————————————————  
设置 Ollydbg 忽略所有异常选项。老规矩：用 IsDebug 1.4 插件去掉 Ollydbg 的调试器标志。  
     
    
00D71248     75 00               jnz short Dreamwea.00D7124A//进入 OD 后停在这  
00D7124A     90                  nop  
00D7124B     32E0                xor ah,al  
00D7124D     8BC5                mov eax,ebp  
00D7124F     40                  inc eax  
00D71250     86E0                xchg al,ah  
00D71252     85C0                test eax,eax  
00D71254     EB 00               jmp short Dreamwea.00D71256  
             
Shift+F9 运行，确定几个入口提示，弹出经典的 Vbox 保护画面。  
现在下断：BP GetProcAddress，然后点“试用”，中断后取消断点。  
Ctrl+F9 两次，就来到处理 IAT 的地方了。  
        
0700B17B     E8 26FBFFFF         call vboxta.0700ACA6  
//GetProcAddress  
0700B180     83C4 14             add esp,14  
//返回这里  
0700B183     85C0                test eax,eax  
0700B185     0F84 80000000       je vboxta.0700B20B  
0700B18B     FF45 F0             inc dword ptr ss:\[ebp\-10\]  
0700B18E     8906                mov dword ptr ds:\[esi\],eax  
//函数写入  
0700B190     8B45 F0             mov eax,dword ptr ss:\[ebp\-10\]  
0700B193     83C7 04             add edi,4  
0700B196     3B45 D8             cmp eax,dword ptr ss:\[ebp\-28\]  
0700B199     72 88               jb short vboxta.0700B123  
//循环  
             
因为 Vbox V4.5 没加密输入表（除了一个函数），所以这里看看而已，就不必多管了。  
————————————————————————  
下断：BP FreeLibrary    
中断后取消断点，Ctrl+F9 返回  
             
070081E2     FF15 D0C10407       call dword ptr ds:\[704C1D0\]  ; kernel32.FreeLibrary  
070081E8     8B76 14             mov esi,dword ptr ds:\[esi+14\]; Dreamwea.009801C0  
//返回这里。\[esi+14\]＝009801C0    这就是 OEP 值！★  
      
想看看 Vbox V4.5 是如何跳 OEP 的？OK，我们在\[esi+14\]＝\[015B0014\]转存处下内存访问断点，F9 运行，中断。  
             
016009B9     8B48 14             mov ecx,dword ptr ds:\[eax+14\]; Dreamwea.009801C0  
016009BC     EB 02               jmp short 016009C0  
016009C0     894D F0             mov dword ptr ss:\[ebp\-10\],ecx; Dreamwea.009801C0  
016009C3     EB 01               jmp short 016009C6  
016009C6     8B5D F0             mov ebx,dword ptr ss:\[ebp\-10\]; Dreamwea.009801C0  
016009C9     EB 02               jmp short 016009CD  
016009CD     FFE3                jmp ebx                      ; Dreamwea.009801C0  
//飞向光明之巅！ ^O^  
             
————————————————————————  
             
009801C0     6A 02               push 2  
//在这儿用 LordPE 纠正 ImageSize 后完全 DUMP 这个进程  
009801C2     FF15 08C2B900       call dword ptr ds:\[B9C208\]  ; kernel32.SetErrorMode  
009801C8     68 40019800         push Dreamwea.00980140  
009801CD     FF15 0CC2B900       call dword ptr ds:\[B9C20C\]  ; kernel32.SetUnhandledExceptionFilter  
009801D3     E9 20221700         jmp Dreamwea.00AF23F8  
             
————————————————————————  
运行 ImportREC，选择这个进程。把 OEP 改为 005801C0，点 IT AutoSearch，点“Get Import”，有一个无效函数：  
0079DB54  ?  0000  0700EBBF  
我们 Ctrl+G:0700EBBF 看看：  
0700EBBF     55                  push ebp  
0700EBC0     8BEC                mov ebp,esp  
0700EBC2     E8 A7FFFFFF         call vboxta.0700EB6E  
0700EBC7     FF75 18             push dword ptr ss:\[ebp+18\]  
0700EBCA     FF75 14             push dword ptr ss:\[ebp+14\]  
0700EBCD     FF75 10             push dword ptr ss:\[ebp+10\]  
0700EBD0     FF75 0C             push dword ptr ss:\[ebp+C\]  
0700EBD3     FF75 08             push dword ptr ss:\[ebp+8\]  
0700EBD6     FF15 D4C30407       call dword ptr ds:\[704C3D4\]; USER32.PeekMessageA  
0700EBDC     5D                  pop ebp  
0700EBDD     C2 1400             retn 14  
             
把这个函数修改为 USER32.PeekMessageA，FixDump！  
OK，启动时不见试用提示，Vbox 荡然无存了。  
                 
               
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
      
  
     Cracked By 巢水工作坊——fly \[FCG\]\[NUKE\]\[DCM\]  
                2004-08-15  17:00

 4dd466d0e1e0cc123afbe504da2462ca