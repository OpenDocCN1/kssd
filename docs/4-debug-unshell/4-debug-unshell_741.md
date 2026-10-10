

# UnFoxAll 2.10

> 原文：[`www.pediy.com/kssd/pediy06/pediy6340.htm`](https://www.pediy.com/kssd/pediy06/pediy6340.htm)

LOCK98 V1.00.28 脱壳 —— 反编译专家 UnFoxAll 2.10 版   
    
    
    
下载页面：  http://secrecy.ayinfo.ha.cn/soft/110.htm   
软件大小：  1M   
  
【软件简介】：可以完整的从 FOX 所有版本（FoxBase,FoxPro2.x for DOS/Windows,Visual FoxPro 3.0，Visual FoxPro 5.0,Visual 6.0）开发的 EXE,.APP,.FOX,.SPX,.MPX,.QPX,.PJT 及其他文件中完整反编译出所有 源程序及其他文件。是反编译工作的又一利器。   
  
【作者声明】：初学 Crack，只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教！   
  
【调试环境】：WinXP、Ollydbg1.09、PEiD、LordPE   
  
—————————————————————————————————    
【脱壳过程】：   
             
                   
这个版本的 UnFoxAll 是以前从白菜乐园下载的，看雪也有，不清楚是否也是加的这种壳。   
  
LOCK98 V1.00.28 现在很少见了，看说明是软盘加密的程序，但是这个 unfoxall.exe 用 FI 看显示是“LOCK98 v1.00.28  keenvim \_LOCK98\_”，偶不清楚啦，姑且当其是这个壳吧，手脱试试。用 PEID 看却显示“UPX 0.80-0.84”，晕，可能是加了 2 层壳，刚开始的代码不是典型的 UPX，而最后的入口方式倒是 UPX 的典型 POPAD、JMP。   
  
  
用 Ollydbg 手动脱壳,老规矩：载入后弹出“是压缩代码——要继续进行分析吗？”，点“否”。   
  
005FE0A0     55                   push ebp   
                                  ====>进入 OD 后断在这！   
  
如果一步步走的话会有数十个 LOOP 循环用 SMC 来还原程序的代码,很长。这段可能就是 LOCK98 代码。   
偶们没必要这样费时跟踪。:-)   下断：BP LoadLibraryA       F9 运行，断下！然后取消断点   
  
77E605D8     837C24 04 00         cmp dword ptr ss:\[esp+4\],0   
                                  ====>断在这！CTRL+F9 执行到返回   
77E605DD     53                   push ebx   
77E605DE     56                   push esi   
77E605DF     74 19                je short kernel32.77E605FA   
77E605E1     68 9C5BE777          push kernel32.77E75B9C                ; ASCII "twain\_32.dll"   
77E605E6     FF7424 10            push dword ptr ss:\[esp+10\]   
77E605EA     FF15 9013E477        call dword ptr ds:\[<&ntdll.\_strcmpi>\] ; ntdll.\_stricmp   
77E605F0     85C0                 test eax,eax   
77E605F2     59                   pop ecx   
77E605F3     59                   pop ecx   
77E605F4     0F84 76AF0100        je kernel32.77E7B570   
77E605FA     6A 00                push 0   
77E605FC     6A 00                push 0   
77E605FE     FF7424 14            push dword ptr ss:\[esp+14\]   
77E60602     E8 B1FFFFFF          call kernel32.LoadLibraryExA   
77E60607     5E                   pop esi   
77E60608     5B                   pop ebx   
77E60609     C2 0400              retn 4   
                                  ====>返回到 005F7128   
  
  
————————————————————————   
  
005F7110     74 3A                je short unfoxall.005F714C   
005F7112     8B5F 04              mov ebx,dword ptr ds:\[edi+4\]   
005F7115     8D8430 8C881F00      lea eax,dword ptr ds:\[eax+esi+1F888C\]   
005F711C     01F3                 add ebx,esi   
005F711E     50                   push eax   
005F711F     83C7 08              add edi,8   
005F7122     FF96 68891F00        call dword ptr ds:\[esi+1F8968\]   
                                  ====>这个 CALL KERNEL LoadLibraryA   
005F7128     92                   xchg eax,edx   
005F7129     8A07                 mov al,byte ptr ds:\[edi\]   
005F712B     47                   inc edi   
005F712C     08C0                 or al,al   
005F712E     74 DC                je short unfoxall.005F710C   
005F7130     52                   push edx   
005F7131     89F9                 mov ecx,edi   
005F7133     57                   push edi   
005F7134     48                   dec eax   
005F7135     F2:AE                repne scas byte ptr es:\[edi\]   
005F7137     52                   push edx   
005F7138     FF96 6C891F00        call dword ptr ds:\[esi+1F896C\]   
                                  ====>这个 CALL KERNEL GetProcAddress    
005F713E     5A                   pop edx   
005F713F     09C0                 or eax,eax   
005F7141     74 07                je short unfoxall.005F714A   
005F7143     8903                 mov dword ptr ds:\[ebx\],eax   
005F7145     83C3 04              add ebx,4   
005F7148     EB DF                jmp short unfoxall.005F7129   
                                  ====>注意这个循环！向上找发现 005F7110 可以跳下去！   
005F714A     61                   popad   
005F714B     C3                   retn   
  
005F714C     8D5E FC              lea ebx,dword ptr ds:\[esi\-4\]   
                                  ====>这里设断     F9 断在此处   
005F714F     83C7 04              add edi,4   
005F7152     31C0                 xor eax,eax   
005F7154     8A07                 mov al,byte ptr ds:\[edi\]   
005F7156     47                   inc edi   
005F7157     09C0                 or eax,eax   
005F7159     74 22                je short unfoxall.005F717D   
005F715B     3C EF                cmp al,0EF   
005F715D     77 11                ja short unfoxall.005F7170   
005F715F     01C3                 add ebx,eax   
005F7161     8B03                 mov eax,dword ptr ds:\[ebx\]   
005F7163     86C4                 xchg ah,al   
005F7165     C1C0 10              rol eax,10   
005F7168     86C4                 xchg ah,al   
005F716A     01F0                 add eax,esi   
005F716C     8903                 mov dword ptr ds:\[ebx\],eax   
005F716E     EB E2                jmp short unfoxall.005F7152   
                                  ====>注意这个循环！向上找发现 005F7159 可以跳下去！   
005F7170     24 0F                and al,0F   
005F7172     C1E0 10              shl eax,10   
005F7175     66:8B07              mov ax,word ptr ds:\[edi\]   
005F7178     83C7 02              add edi,2   
005F717B     EB E2                jmp short unfoxall.005F715F   
005F717D     61                   popad   
                                  ====>这里设断     F9 断在此处   
005F717E     E9 49DFEAFF          jmp unfoxall.004A50CC   
                                  ====>飞向光明之巅！   
  
  
————————————————————————   
  
004A50CC     55                   push ebp   
                                  ====>在这儿用 LordPE 完全 DUMP 这个进程   
004A50CD     8BEC                 mov ebp,esp   
004A50CF     83C4 F4              add esp,-0C   
004A50D2     53                   push ebx   
004A50D3     B8 144E4A00          mov eax,unfoxall.004A4E14   
004A50D8     E8 E314F6FF          call unfoxall.004065C0   
004A50DD     8B1D B4894A00        mov ebx,dword ptr ds:\[4A89B4\]   
004A50E3     8B03                 mov eax,dword ptr ds:\[ebx\]   
004A50E5     E8 DA4BFAFF          call unfoxall.00449CC4   
  
  
———————————————————————   
  
运行 ImportREC，选择这个进程。把 OEP 改为 000A50CC，点 IT AutoSearch，点“Get Import”，所有的函数都是有效的。FixDump，正常运行！  629K ->2.01M   用 FI 看脱壳后的程序是 Delphi 编写，可以跨平台运行，能够正常反编译其他 VF 的例子。如果不是 LOCK98，还请大家谅解。  :-)   
  
     
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
  
                     2003-10-28  15:35  
 4dd466d0e1e0cc123afbe504da2462ca