

# 完美卸载 XP V9.15

> 原文：[`www.pediy.com/kssd/pediy06/pediy6426.htm`](https://www.pediy.com/kssd/pediy06/pediy6426.htm)

【脱文作者】 weiyi75\[Dfcg\]   
  
【作者邮箱】 weiyi75@sohu.com    
  
【作者主页】 Dfcg 官方大本营   
  
【使用工具】 Peid0.91,Ollydbg1.10b(反 Antidbg 版),ImportREC1.42,,Freeres,ResScope,PEedit   
  
【破解平台】 Win2000/XP   
  
【软件名称】 完美卸载 XP V9.15   
  
【下载地址】 天空软件站   
  
【软件简介】：软件共有 3 个小程序，分别是：磁盘垃圾清理软件、软件安装监视器、软件卸载工具，是一套功能比较强大的安装/卸载/清理工具，新版更加全面的系统监视，让软件安装的一举一动历历在目，二次清理技术，软件卸载后将不留任何垃圾，磁盘垃圾清理的数量更多，软件运行更加稳定。    
  
【软件大小】 4.64M   
  
【加壳方式】 UltraProtect 1.x -> RISCO Software Inc.   
  
【破解声明】 我是一只小菜鸟，偶得一点心得，愿与大家分享。   
  
【破解目的】 寻找 Stolen Code 具体内容,欺骗 Acprotect 完全解码。   
  
 先用 peid 查壳，UltraProtect 1.x -> RISCO Software Inc.这个版本估计是用 Acprotect1.10 Build123 以上的版本加密的。此版本的 Stolen Code 隐藏更深，硬件断点设置不当根本拦截不住 Stolen Code,模拟跟踪我的 Win2000 平台 5-15 分钟可以截获 Stolen Code,Xp 平台遇到无效指令或单步跟踪陷阱。根据 9.14 版的调试经验   
  
OD 异常设置不忽略 INT3 异常，其余全部忽略,载入程序,用插件隐藏 OD。   
  
04AE000 >  60              pushad    //加壳程序入口点，F9 运行。    
004AE001    FC              cld   
004AE002    48              dec eax   
004AE003    66:81C2 2FAC    add dx, 0AC2F   
004AE008    50              push eax   
004AE009    E8 01000000     call NetClean.004AE00F   
004AE00E    EA 5858668B F55>jmp far 50F5:8B665858   
004AE015    E8 01000000     call NetClean.004AE01B   
004AE01A    EB 58           jmp short NetClean.004AE074   
004AE01C    58              pop eax   
004AE01D    87F2            xchg edx, esi   
004AE01F    E8 01000000     call NetClean.004AE025   
004AE024    9A 83042406 C30>call far 0FC3:06240483   
004AE02B    8801            mov byte ptr ds:\[ecx\], al   
004AE02D    0000            add byte ptr ds:\[eax\], al   
004AE02F    0048 E8         add byte ptr ds:\[eax\-18\], cl   
004AE032    0100            add dword ptr ds:\[eax\], eax   
004AE034    0000            add byte ptr ds:\[eax\], al   
004AE036  - 76 83           jbe short NetClean.004ADFBB   
...........................................................   
  
INT3 中断   
  
004BCB23    90              nop  //典型 Acprotect 小于 1.20 版最后一次异常。    
004BCB24    64:67:8F06 0000 pop dword ptr fs:\[0\]   
004BCB2A    83C4 04         add esp, 4   
004BCB2D    60              pushad   
004BCB2E    E8 00000000     call NetClean.004BCB33   
004BCB33    5E              pop esi   
004BCB34    83EE 06         sub esi, 6   
004BCB37    B9 5B000000     mov ecx, 5B   
004BCB3C    29CE            sub esi, ecx   
004BCB3E    BA F631BE1F     mov edx, 1FBE31F6   
004BCB43    C1E9 02         shr ecx, 2   
004BCB46    83E9 02         sub ecx, 2   
004BCB49    83F9 00         cmp ecx, 0   
004BCB4C    7C 1A           jl short NetClean.004BCB68   
004BCB4E    8B048E          mov eax, dword ptr ds:\[esi+ecx\*4\]   
........................................................................   
  
堆栈内容   
  
0012FF58   0012FFE0  指针到下一个 SEH 记录   
0012FF5C   004BCAF0  SE 句柄      //这里同原来不同，为了跟踪到 Stolen Code，选择合适的地点跟踪很关键，在跟踪 N 个 Acprotect 程序后，找到了关键点，这些没什么技术可言，只是不断的实际，无数次的失败总结。右键对 004BCAF0 转存中跟随，下内存访问断点。    
0012FF60   00000035   
........................................................................   
  
004BCAF0    8B4424 04       mov eax, dword ptr ss:\[esp+4\]  //下断点 Shift+F9 中断 1    
004BCAF4    8B4C24 0C       mov ecx, dword ptr ss:\[esp+C\]   
004BCAF8    FF81 B8000000   inc dword ptr ds:\[ecx+B8\]   
004BCAFE    8B00            mov eax, dword ptr ds:\[eax\]   
004BCB00    2D 03000080     sub eax, 80000003   
004BCB05    75 12           jnz short NetClean.004BCB19   
004BCB07    90              nop   
004BCB08    90              nop   
004BCB09    90              nop   
004BCB0A    90              nop   
004BCB0B    33C0            xor eax, eax   
004BCB0D    8941 04         mov dword ptr ds:\[ecx+4\], eax   
004BCB10    8941 08         mov dword ptr ds:\[ecx+8\], eax   
004BCB13    8941 0C         mov dword ptr ds:\[ecx+C\], eax   
004BCB16    8941 10         mov dword ptr ds:\[ecx+10\], eax   
004BCB19    C3              retn   
........................................................................   
  
004BCB4E    8B048E          mov eax, dword ptr ds:\[esi+ecx\*4\] //下断点 Shift+F9 中断 2,清除内存断点    
004BCB51    8B5C8E 04       mov ebx, dword ptr ds:\[esi+ecx\*4+4\]   
004BCB55    03C3            add eax, ebx   
004BCB57    C1C0 13         rol eax, 13   
004BCB5A    2BC2            sub eax, edx   
004BCB5C    81EA DA4777C0   sub edx, C07747DA   
004BCB62    89048E          mov dword ptr ds:\[esi+ecx\*4\], eax   
004BCB65    49              dec ecx   
004BCB66  ^ EB E1           jmp short NetClean.004BCB49   
004BCB68    61              popad   
004BCB69    61              popad   
004BCB6A    C3              retn //F4 直接下来，这时是模拟跟踪的时候了。    
.......................................................................   
  
命令行下  tc ebp\==12fff0   
  
为什么要设置这个命令，先前已说明这是 Acprotect 的死穴之一，对准它点吧。   
  
004CBB31    55              push ebp   
004CBB32    8BEC            mov ebp, esp   
004CBB34    6A FF           push -1   
004CBB36    90              nop   
004CBB37    60              pushad   
004CBB38    60              pushad   
004CBB39    E8 00000000     call NetClean.004CBB3E   
  
Ctrl+F11 跟踪进入每个 Call,1 分钟左右。   
  
004CBB31    55              push ebp           //Stolen Code    
004CBB32    8BEC            mov ebp, esp      //Stolen Code //一切尽在掌握    
004CBB34    6A FF           push -1         //Stolen Code        
004CBB36    90              nop            //注意它填充了一个 nop,所以 ImportREC 修复时有点.....    
004CBB37    60              pushad   
004CBB38    60              pushad   
004CBB39    E8 00000000     call NetClean.004CBB3E   
  
Btw:只有用 INT3 中断，才能顺利截获这个版本的 Stolen Code,如果你用内存异常 Tc ebp\==12fff0,你会踏进它早已布下的单步异常陷阱。当然硬件断点 12fff0 也无法断下，因为你试图中断时 12fff0 已经写入数据，中断失败   
    
我们来看看硬件断点。   
  
004BCB4E    8B048E          mov eax, dword ptr ds:\[esi+ecx\*4\]     
004BCB51    8B5C8E 04       mov ebx, dword ptr ds:\[esi+ecx\*4+4\]   
004BCB55    03C3            add eax, ebx   
004BCB57    C1C0 13         rol eax, 13   
004BCB5A    2BC2            sub eax, edx   
004BCB5C    81EA DA4777C0   sub edx, C07747DA   
004BCB62    89048E          mov dword ptr ds:\[esi+ecx\*4\], eax   
004BCB65    49              dec ecx   
004BCB66  ^ EB E1           jmp short NetClean.004BCB49   
004BCB68    61              popad   
004BCB69    61              popad    
004BCB6A    C3              retn  //同样 F4 直接下来    
  
命令行下断点   
    
d 12ffc0   
  
0012FFC0  00 00 00 00 C7 14 E6 77  ....?鎤   
0012FFC8  E6 17 F5 77 78 17 F5 77  ?鮳 x 鮳   
0012FFD0  00 F0 FD 7F F0 7C 11 F7  .瘕饇    
0012FFF0  00 00 00 00 00 00 00 00  ........  注意这时 12ffc0 还没有写入数据。   
0012FFF8  00 E0 4A 00 00 00 00 00  .郕.....   
    
我们对准 0012ffc0 下一个硬件访问 dword 断点，F9 运行，硬件中断 1   
  
如果你对 0012ffc0 下一个硬件写入 dword 断点，F9 运行,你就会硬件中断 2   
  
004CBB31    55              push ebp           //Stolen Code  //硬件中断 1    
004CBB32    8BEC            mov ebp, esp      //Stolen Code //一切尽在掌握 硬件中断 2    
004CBB34    6A FF           push -1         //Stolen Code        
004CBB36    90              nop   
004CBB37    60              pushad   
004CBB38    60              pushad   
004CBB39    E8 00000000     call NetClean.004CBB3E   
........................................................................   
    
这个程序抽了 5 个字节。   
  
ALT+M 打开内存镜像断点,对准它的第二个死穴点。      
  
内存镜像，项目 26   
  
 地址=00401000     //Code 段下内存访问断点   
 大小=00047000 (290816.)   
 Owner=NetClean 00400000   
 区段=.text   
 包含=code   
 类型=Imag 01001002   
 访问=R   
 初始访问=RWE   
    
F9 运行到达   
  
004432D9    68 D8B24400     push NetClean.0044B2D8   
004432DE    68 94344400     push NetClean.00443494     //临时 Oep,滚动条向上看,如何判断抽掉多少字节，用滚动条是代码会混乱，右键分析代码,如我现在调整的样式,标签 1    
004432E3    64:A1 00000000  mov eax, dword ptr fs:\[0\]   
004432E9    50              push eax   
004432EA    64:8925 0000000>mov dword ptr fs:\[0\], esp   
004432F1    83EC 68         sub esp, 68   
004432F4    53              push ebx   
004432F5    56              push esi   
004432F6    57              push edi   
004432F7    8965 E8         mov dword ptr ss:\[ebp\-18\], esp   
004432FA    33DB            xor ebx, ebx   
004432FC    895D FC         mov dword ptr ss:\[ebp\-4\], ebx   
004432FF    6A 02           push 2   
00443301    FF15 70874400   call dword ptr ds:\[448770\]               ; MSVCRT.\_\_set\_app\_type   
  
    
  
标签 1   
  
    
  
004432D1  .  C2 1000       retn 10    
  
004432D4      60            db 60     //确定真 Oep 为 004432d4                         ;  CHAR '\`'    
  
004432D5      CA            db CA   
  
004432D6      B5            db B5   
  
004432D7   .  D4 11         aam 11   
  
004432D9   .  68 D8B24400   push NetClean.0044B2D8   
  
004432DE   .  68 94344400   push NetClean.00443494                   ;  jmp to MSVCRT.\_except\_handler3; SE handler installation   
  
004432E3   .  64:A1 0000000>mov eax, dword ptr fs:\[0\]   
  
004432E9   .  50            push eax   
  
004432EA   .  64:8925 00000>mov dword ptr fs:\[0\], esp   
  
004432F1   .  83EC 68       sub esp, 68   
  
004432F4   .  53            push ebx   
  
004432F5   .  56            push esi   
  
004432F6   .  57            push edi   
  
004432F7   .  8965 E8       mov dword ptr ss:\[ebp\-18\], esp   
  
004432FA   .  33DB          xor ebx, ebx   
  
004432FC   .  895D FC       mov dword ptr ss:\[ebp\-4\], ebx   
  
004432FF   .  6A 02         push 2   
  
00443301   .  FF15 70874400 call dword ptr ds:\[448770\]               ;  MSVCRT.\_\_set\_app\_type   
  
........................................................................   
  
    
  
OD 直接修复找到的 5 个字节。   
  
    
  
004432D4 >/$  55            push ebp   //用 Od 插件修正入口为 432D4 直接脱壳吧，重建输入表的勾去掉    
  
004432D5  |.  8BEC          mov ebp, esp   
  
004432D7  |.  6A FF         push -1   
  
004432D9  |.  68 D8B24400   push Unpack\_.0044B2D8   
  
004432DE  |.  68 94344400   push <jmp.&msvcrt.\_except\_handler3>      ;  SE handler installation   
  
004432E3  |.  64:A1 0000000>mov eax, dword ptr fs:\[0\]   
  
004432E9  |.  50            push eax   
  
004432EA  |.  64:8925 00000>mov dword ptr fs:\[0\], esp   
  
004432F1  |.  83EC 68       sub esp, 68   
  
004432F4  |.  53            push ebx   
  
004432F5  |.  56            push esi   
  
004432F6  |.  57            push edi   
  
004432F7  |.  8965 E8       mov dword ptr ss:\[ebp\-18\], esp   
  
004432FA  |.  33DB          xor ebx, ebx   
  
004432FC  |.  895D FC       mov dword ptr ss:\[ebp\-4\], ebx   
  
004432FF  |.  6A 02         push 2   
  
00443301  |.  FF15 70874400 call dword ptr ds:\[<&msvcrt.\_\_set\_app\_ty>;  MSVCRT.\_\_set\_app\_type   
  
........................................................................   
  
    
  
修复 IAT 必须关闭 OD 单独开加壳程序修复，不然 ImportREC 停止响应，最后用 ImportREC Oep 填 432D3,跟踪等级三轻松修复,有 10 个指针提示无法修复，你可以验证一下，经过验证全部是垃圾指针，通常 ImportREC 可全部修复指针，如有没有一般都是垃圾，拿剪刀全部 Cut 掉，Oep 改回为 432D4 修复后无法运行,寻找出错原因。   
  
    
  
寻找出错原因。   
  
    
  
004431C4  |.  68 00404500   PUSH 2\_.00454000   
  
004431C9  |.  E8 DC000000   CALL <JMP.&msvcrt.\_initterm>  //这个 Call 可能是新版 Acprotect 的解码技术,进这个 Call 里面，循环解码，没解完程序就崩溃。    
  
004431CE  |.  83C4 24       ADD ESP,24  //这里类似代码，我没有再次跟踪。    
  
    
  
其实本版本 Acprotect 嵌有 Oep 处代码检验程序，发现 Oep 处代码被替换，立即拒绝解码，你失去了关键的代码当然程序无法运行，还要承认 N 次错误确定按钮。   
  
    
  
004CBB31    55              push ebp           //Stolen Code    
  
004CBB32    8BEC            mov ebp, esp      //Stolen Code //一切尽在掌握    
  
004CBB34    6A FF           push -1         //Stolen Code      
  
    
  
我们已经掌握。   
  
    
  
004432D9   .  68 D8B24400   push NetClean.0044B2D8   //临时 Oep 也知道。    
  
    
  
现在做什么,让程序认为你没有脱壳,将 perplex 段里面有用的内容全部解码。   
  
    
  
方法   
  
    
  
push ebp       //Stolen Code    
  
mov ebp, esp  //Stolen Code                //一切尽在掌握    
  
push -1      //Stolen Code   
  
jmp  004432D9   
  
    
  
Od 载入脱壳修复后的程序,Ctrl+G 004AE000   
  
    
  
004AE000    60              pushad   //4AE000 是原来壳的入口点，狸猫换太子，将 Oep 入口代码复制到这里。    
  
004AE001    FC              cld   
  
004AE002    48              dec eax   
  
004AE003    66:81C2 2FAC    add dx, 0AC2F   
  
004AE008    50              push eax   
  
004AE009    E8 01000000     call Unpack\_.004AE00F   
  
004AE00E    EA 5858668B F55>jmp far 50F5:8B665858   
  
004AE015    E8 01000000     call Unpack\_.004AE01B   
  
004AE01A    EB 58           jmp short Unpack\_.004AE074   
  
004AE01C    58              pop eax   
  
004AE01D    87F2            xchg edx, esi   
  
004AE01F    E8 01000000     call Unpack\_.004AE025   
  
004AE024    9A 83042406 C30>call far 0FC3:06240483   
  
004AE02B    8801            mov byte ptr ds:\[ecx\], al   
  
004AE02D    0000            add byte ptr ds:\[eax\], al   
  
004AE02F    0048 E8         add byte ptr ds:\[eax\-18\], cl   
  
004AE032    0100            add dword ptr ds:\[eax\], eax   
  
004AE034    0000            add byte ptr ds:\[eax\], al   
  
004AE036  - 76 83           jbe short Unpack\_.004ADFBB   
  
.............................................................   
  
    
  
004AE000    55              push ebp   
  
004AE001    8BEC            mov ebp, esp   
  
004AE003    6A FF           push -1   
  
004AE005  - E9 CF52F9FF     jmp Unpack\_.004432D9   
  
004AE00A    90              nop       
  
004AE00B    90              nop    //注意保持代码完整，填入 4 个 Nop    
  
004AE00C    90              nop   
  
004AE00D    90              nop   
  
004AE00E    EA 5858668B F55>jmp far 50F5:8B665858   
  
004AE015    E8 01000000     call Unpack\_.004AE01B   
  
004AE01A    EB 58           jmp short Unpack\_.004AE074   
  
004AE01C    58              pop eax   
  
004AE01D    87F2            xchg edx, esi   
  
004AE01F    E8 01000000     call Unpack\_.004AE025   
  
004AE024    9A 83042406 C30>call far 0FC3:06240483   
  
004AE02B    8801            mov byte ptr ds:\[ecx\], al   
  
004AE02D    0000            add byte ptr ds:\[eax\], al   
  
004AE02F    0048 E8         add byte ptr ds:\[eax\-18\], cl   
  
004AE032    0100            add dword ptr ds:\[eax\], eax   
  
004AE034    0000            add byte ptr ds:\[eax\], al   
  
.........................................................   
  
    
  
将替换的代码复制到程序中，另存为一个文件，用 PEedit 修正入口为 AC000 正常运行。   
  
    
  
完美卸载 XP 很久已经就是 DEMO 版,破解基本没希望，针对 C 语言资源容易修改的特点。用 ResScope 修改主界面 102 对话框资源,将本软件授权给: NOP 掉。合作伙伴: 修改为 本软件授权给 David,保存时错误，看来资源没完全释放啊，再用 Freeres 释放资源，可以反汇编，必尽这只是模拟脱壳。继续完成修改过程，看胜利截图，聊博一笑。   
  
  
    
  
【破解总结】发现 C 语言的 Acprotect 的壳采用了 Stolen Code,动态解码，用 Acprotect 保护的它人没加上 RSAKEY 就有强劲的保护,新版 Acprotect 异常设置应选 INT3 中断，模拟跟踪或硬件断点截获 Stolen Code。   
  
\--------------------------------------------------------------------------------    
   
  
    
  
【版权声明】 本文纯属技术交流, 转载请注明作者并保持文章的完整, 谢谢! 

* * *

**David**: 

Oep 改回为 432D4 修复后无法运行,寻找出错原因。    
  
寻找出错原因。    
  
004431C4  |.  68 00404500   PUSH 2\_.00454000    
  
004431C9  |.  E8 DC000000   CALL <JMP.&msvcrt.\_initterm>  //这个 Call 可能是新版 Acprotect 的解码技术,进这个 Call 里面，循环解码，没解完程序就崩溃。     
  
004431CE  |.  83C4 24       ADD ESP,24  //这里类似代码，我没有再次跟踪。     
     
其实本版本 Acprotect 嵌有 Oep 处代码检验程序，发现 Oep 处代码被替换，立即拒绝解码，你失去了关键的代码当然程序无法运行，还要承认 N 次错误确定按钮。   
  
我测试了一下跨平台，一点问题都没有，因为部分代码临时解压，估计跨平台能力不错，这个办法可以反汇编程序，可以修改资源，也是个不错的方法，Acprotect 真正可怕的是用 RSAKEY 嵌入校验解码，无 KEY 就无法完全解码，最多是走 DEMO 路线。   
  
Dfcg 的 YOCK 有更好的修复方法，你可以问问他。  
  
 4dd466d0e1e0cc123afbe504da2462ca