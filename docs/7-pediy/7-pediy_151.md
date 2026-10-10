

# Code Injection 破解 Armadillo V4.20 加壳程序

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-595.htm`](https://www.pediy.com/kssd/pediy07/pediy7-595.htm)

Code Injection 破解 Armadillo V4.20 单进程加壳程序——FTPRush Unicode V1.0.RC6.build.568              
             
               
下载页面：  http://www.ftprush.com  
软件大小：  3342 KB  
软件语言：  简体中文  
软件类别：  国外软件 / 共享版 / FTP 工具  
应用平台：  Win9x/NT/2000/XP  
加入时间：  2005-07-12 11:56:33  
下载次数：  55097  
推荐等级：  \*\*\*\*  
软件介绍：  一个高性能的 FTP 客户端, 通过它你可以轻松地上传下载和 FXP(Ftp 服务器之间的文件传输), 完全多线程的构造, 让你自由自在地存取 FTP 服务器的资源。主要的特性：连接到多个 ftp 服务器、支持 Ftp 代理、脚本支持、多语言支持、容易使用、自动 CRC 校验、支持 mIRC 宣告。  
               
【作者声明】：只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教  
               
【调试环境】：WinXP、OllyDBD、PEiD、LordPE、IDA  
               
—————————————————————————————————   
【破解过程】：  
            
     
Code Injection 用来 Patch 的思路如下：  
  
1、在壳中寻找适当的切入点  
2、Hook 特定 Api，取得对壳代码的某些控制权  
3、搞定壳的检验  
4、干掉壳所设置的限制，一般为时间限制  
5、程序代码解开后 Patch 程序中的限制代码  
  
下面主要是 stephenteh 的成果，我只是整理和演示一下。  
—————————————————————————————————  
一、在壳中寻找适当的切入点  
               
                 
用 PE 工具观察 Armadillo 加壳后的 FTPRush.exe 区段情况  
  
No.   名称        VSize        VOffset      RSize        ROffset      Charact     
01    CODE        0045AEAC     00001000     00000000     00000000     60000020       
02    DATA        00016B80     0045C000     00000000     00000000     C0000040       
03    BSS         00037C45     00473000     00000000     00000000     C0000000       
04    .idata      000041D4     004AB000     00000000     00000000     C0000040       
05    .tls        00000024     004B0000     00000000     00000000     C0000000       
06    .rdata      00000018     004B1000     00001000     00001000     50000040       
07    .reloc      00045160     004B2000     00000000     00000000     50000040       
08    .text       00050000     004F8000     00041000     00002000     E0000020       
09    .adata      00010000     00548000     0000D000     00043000     E0000020       
0A    .data       00020000     00558000     0000A000     00050000     C0000040       
0B    .pdata      00250000     00578000     00250000     0005A000     C0000040       
0C    .rsrc       00043000     007C8000     00043000     002AA000     50000040       
  
可以看到壳区段.data 和.pdata 的区段属性都是可读可写的，并且 Armadillo 没有对这 2 个段进行校验。  
因此可以把 Patch 代码放在这 2 个段中的空白处。  
我选择.data 段 RVA＝00561990（ROffset＝00059990）作为放置 Patch 代码的开始处。  
  
再察看其输入表，选择 KERNEL32.dll 的 GetProcAddress 作为第一个桥梁  
0091F0D3    FF15 70809500   call dword ptr ds:\[958070\]; kernel32.GetProcAddress  
00920E87    FF15 B0809500   call dword ptr ds:\[9580B0\]; kernel32.ReadFile  
0091EE1C    FF15 68809500   call dword ptr ds:\[958068\]; kernel32.VirtualProtect  
  
GetProcAddress、ReadFile、VirtualProtect 是我们将要使用到的三个函数。  
  
  
—————————————————————————————————  
二、第一个接口+第二个接口 Hook GetProcAddress  
  
  
使用修改版 OllyDBD，否则你要手动处理 OutputDebugStringA  
设置 OllyDBD 忽略所有异常选项。用 IsDebug 插件去掉 OllyDBD 的调试器标志。  
复制 FTPRush.exe，另存为 1.eXe，可以开 2 个 OllyDBD 分别载入这 2 个程序。  
  
009336D3    55              push ebp  
//进入 OllyDBD 后暂停在这  
009336D4    8BEC            mov ebp,esp  
009336D6    6A FF           push -1  
009336D8    68 F8DE9500     push 95DEF8  
009336DD    68 10349300     push 933410  
009336E2    64:A1 00000000  mov eax,dword ptr fs:\[0\]  
009336E8    50              push eax  
009336E9    64:8925 0000000>mov dword ptr fs:\[0\],esp  
009336F0    83EC 58         sub esp,58  
009336F3    53              push ebx  
009336F4    56              push esi  
009336F5    57              push edi  
009336F6    8965 E8         mov dword ptr ss:\[ebp-18\],esp  
009336F9    FF15 88819500   call dword ptr ds:\[958188\] ; kernel32.GetVersion  
009336FF    33D2            xor edx,edx  
00933701    8AD4            mov dl,ah  
00933703    8915 84F59500   mov dword ptr ds:\[95F584\],edx  
00933709    8BC8            mov ecx,eax  
0093370B    81E1 FF000000   and ecx,0FF  
00933711    890D 80F59500   mov dword ptr ds:\[95F580\],ecx  
00933717    C1E1 08         shl ecx,8  
0093371A    03CA            add ecx,edx  
0093371C    890D 7CF59500   mov dword ptr ds:\[95F57C\],ecx  
00933722    C1E8 10         shr eax,10  
00933725    A3 78F59500     mov dword ptr ds:\[95F578\],eax  
0093372A    33F6            xor esi,esi  
0093372C    56              push esi  
0093372D    E8 78160000     call 00934DAA  
00933732    59              pop ecx  
00933733    85C0            test eax,eax  
00933735    75 08           jnz short 0093373F  
00933737    6A 1C           push 1C  
00933739    E8 B0000000     call 009337EE  
0093373E    59              pop ecx  
0093373F    8975 FC         mov dword ptr ss:\[ebp-4\],esi  
00933742    E8 43130000     call 00934A8A  
00933747    FF15 8C809500   call dword ptr ds:\[95808C\] ; kernel32.GetCommandLineA  
0093374D    A3 840B9600     mov dword ptr ds:\[960B84\],eax  
00933752    E8 01120000     call 00934958  
00933757    A3 D8F59500     mov dword ptr ds:\[95F5D8\],eax  
0093375C    E8 AA0F0000     call 0093470B  
00933761    E8 EC0E0000     call 00934652  
00933766    E8 2DFAFFFF     call 00933198  
0093376B    8975 D0         mov dword ptr ss:\[ebp-30\],esi  
0093376E    8D45 A4         lea eax,dword ptr ss:\[ebp-5C\]  
00933771    50              push eax  
00933772    FF15 90809500   call dword ptr ds:\[958090\] ; kernel32.GetStartupInfoA  
00933778    E8 7D0E0000     call 009345FA  
0093377D    8945 9C         mov dword ptr ss:\[ebp-64\],eax  
00933780    F645 D0 01      test byte ptr ss:\[ebp-30\],1  
00933784    74 06           je short 0093378C  
00933786    0FB745 D4       movzx eax,word ptr ss:\[ebp-2C\]  
0093378A    EB 03           jmp short 0093378F  
0093378C    6A 0A           push 0A  
0093378E    58              pop eax  
0093378F    50              push eax  
00933790    FF75 9C         push dword ptr ss:\[ebp-64\]  
00933793    56              push esi  
00933794    56              push esi  
00933795    FF15 4C809500   call dword ptr ds:\[95804C\] ; kernel32.GetModuleHandleA  
0093379B    50              push eax  
0093379C    E8 CFC7FEFF     call 0091FF70  
//过了此 CALL 壳就全部操作完成，程序运行，因此选择在这里面做 Patch ★  
009337A1    8945 A0         mov dword ptr ss:\[ebp-60\],eax  
009337A4    50              push eax  
009337A5    E8 1BFAFFFF     call 009331C5  
009337AA    8B45 EC         mov eax,dword ptr ss:\[ebp-14\]  
009337AD    8B08            mov ecx,dword ptr ds:\[eax\]  
009337AF    8B09            mov ecx,dword ptr ds:\[ecx\]  
009337B1    894D 98         mov dword ptr ss:\[ebp-68\],ecx  
009337B4    50              push eax  
009337B5    51              push ecx  
009337B6    E8 BB0C0000     call 00934476  
009337BB    59              pop ecx  
009337BC    59              pop ecx  
009337BD    C3              retn  
  
在 OllyDBD 里面直接修改 1.eXe，然后保存。  
  
0091FF70    55              push ebp  
//修改为：jmp 00961990  ★   第 1 个接口  
0091FF71    8BEC            mov ebp,esp  
0091FF73    B8 0C200000     mov eax,200C  
0091FF78    E8 932D0100     call 00932D10   
  
在 00961990 处键入以下代码：  
  
00961990    60              pushad  
00961991    A1 70809500     mov eax,dword ptr ds:\[958070\]  
//\[958070\]中保存的是 GetProcAddress 的系统地址  
00961996    A3 801A9600     mov dword ptr ds:\[961A80\],eax  
//保存 GetProcAddress 系统地址到\[961A80\]处  
0096199B    C705 70809500 C>mov dword ptr ds:\[958070\],9619C6  
//修改原来的\[958070\]处为 9619C6  
//这样壳所有的 GetProcAddress 调用都会到 9619C6 处理，我们取得了控制 ★  
009619A5    A1 B0809500     mov eax,dword ptr ds:\[9580B0\]  
//\[9580B0\]中保存的是 ReadFile 的系统地址  
009619AA    A3 841A9600     mov dword ptr ds:\[961A84\],eax  
//保存 ReadFile 系统地址到\[961A84\]处  
009619AF    C705 70FF9100 5>mov dword ptr ds:\[91FF70\],B8EC8B55  
//还原 0091FF70 处代码  
009619B9    C605 74FF9100 0>mov byte ptr ds:\[91FF74\],0C  
//还原 0091FF74 处代码  
009619C0    61              popad  
009619C1    E9 AAE5FBFF     jmp 0091FF70  
//跳回 0091FF70 处继续原来的流程  
  
上面是我们在壳身上做了第 1 个接口，运行我们的代码，再做第 2 个接口 Hook 壳的 GetProcAddress 调用。然后还原壳身上第 1 个接口处修改的代码，了无痕迹，再继续壳的运行流程。  
  
  
—————————————————————————————————  
三、跟踪获取原版 FTPRush.exe 的壳检验值  
  
  
FTPRush.exe 是采用 Armadillo 目前的最新版本 V4.20 保护的，检验比以前的版本多了。  
  
现在我们在另个 OllyDBD 里调试原版 FTPRush.exe  
下断：BP ReadFile，Shift+F9，中断 3 次后 Alt+F9 返回  
  
012117EB    FF15 84B12101   call dword ptr ds:\[121B184\] ; kernel32.SetFilePointer  
012117F1    53              push ebx  
012117F2    8D85 ACFEFFFF   lea eax,dword ptr ss:\[ebp-154\]  
012117F8    50              push eax  
012117F9    56              push esi  
012117FA    FFB5 08F7FFFF   push dword ptr ss:\[ebp-8F8\]  
01211800    FFB5 90FDFFFF   push dword ptr ss:\[ebp-270\]  
01211806    FF15 88B22101   call dword ptr ds:\[121B288\] ; kernel32.ReadFile  
0121180C    8B85 08F7FFFF   mov eax,dword ptr ss:\[ebp-8F8\]  
//返回这里，记录一些参数  ★  
01211812    8985 48F9FFFF   mov dword ptr ss:\[ebp-6B8\],eax  
01211818    03C6            add eax,esi  
0121181A    8985 44F9FFFF   mov dword ptr ss:\[ebp-6BC\],eax  
01211820    3985 48F9FFFF   cmp dword ptr ss:\[ebp-6B8\],eax  
01211826    73 33           jnb short 0121185B  
01211828    8B8D 48F9FFFF   mov ecx,dword ptr ss:\[ebp-6B8\]  
0121182E    8A09            mov cl,byte ptr ds:\[ecx\]  
01211830    81E1 FF000000   and ecx,0FF  
01211836    8B55 D8         mov edx,dword ptr ss:\[ebp-28\]  
01211839    81E2 FF000000   and edx,0FF  
0121183F    33CA            xor ecx,edx  
01211841    8B55 D8         mov edx,dword ptr ss:\[ebp-28\]  
01211844    C1EA 08         shr edx,8  
01211847    8B8C8D 90F9FFFF mov ecx,dword ptr ss:\[ebp+ecx\*4-670\]  
0121184E    33CA            xor ecx,edx  
01211850    894D D8         mov dword ptr ss:\[ebp-28\],ecx  
01211853    FF85 48F9FFFF   inc dword ptr ss:\[ebp-6B8\]  
01211859    EB C5           jmp short 01211820  
0121185B    56              push esi  
0121185C    FFB5 70F9FFFF   push dword ptr ss:\[ebp-690\]  
01211862    8D45 B4         lea eax,dword ptr ss:\[ebp-4C\]  
01211865    50              push eax  
01211866    E8 5767FFFF     call 01207FC2  
0121186B    8B85 70F9FFFF   mov eax,dword ptr ss:\[ebp-690\]  
01211871    8985 04F7FFFF   mov dword ptr ss:\[ebp-8FC\],eax  
01211877    50              push eax  
01211878    E8 37910000     call 0121A9B4 ; jmp to msvcrt.operator delete  
0121187D    83C4 10         add esp,10  
01211880    FF85 7CF9FFFF   inc dword ptr ss:\[ebp-684\]  
01211886    8385 A4FDFFFF 2>add dword ptr ss:\[ebp-25C\],28  
0121188D    E9 CEFEFFFF     jmp 01211760  
01211892    C705 70102201 1>mov dword ptr ds:\[1221070\],1221A1C  
0121189C    FFB5 90FDFFFF   push dword ptr ss:\[ebp-270\]  
012118A2    FF15 54B22101   call dword ptr ds:\[121B254\] ; kernel32.CloseHandle  
012118A8    BE C0482201     mov esi,12248C0  
012118AD    8BCE            mov ecx,esi  
012118AF    E8 5C4D0000     call 01216610  
012118B4    84C0            test al,al  
012118B6    74 21           je short 012118D9  
012118B8    68 00010000     push 100  
012118BD    8D85 B4FEFFFF   lea eax,dword ptr ss:\[ebp-14C\]  
012118C3    50              push eax  
012118C4    FF35 A4152201   push dword ptr ds:\[12215A4\]  
012118CA    E8 62900000     call 0121A931  
012118CF    83C4 0C         add esp,0C  
012118D2    50              push eax  
012118D3    FF15 1CB22101   call dword ptr ds:\[121B21C\] ; kernel32.OutputDebugStringA  
//下面进行检验和解码  
012118D9    C705 70102201 1>mov dword ptr ds:\[1221070\],1221A10  
012118E3    8B0D 084F2201   mov ecx,dword ptr ds:\[1224F08\] ; 00958370  
012118E9    8B41 50         mov eax,dword ptr ds:\[ecx+50\]  
012118EC    3341 30         xor eax,dword ptr ds:\[ecx+30\]  
012118EF    3341 28         xor eax,dword ptr ds:\[ecx+28\]  
012118F2    3345 D8         xor eax,dword ptr ss:\[ebp-28\]  
012118F5    8985 CCF6FFFF   mov dword ptr ss:\[ebp-934\],eax  
012118FB    8945 CC         mov dword ptr ss:\[ebp-34\],eax  
  
当我们自 ReadFile 处返回 0121180C 时，在数据窗口查看、记录以下参数：  
\[ebp-28\]＝0012D724  03 35 5C 84   
\[ebp-40\]＝0012D70C  1A DA 15 EA  
\[ebp-44\]＝0012D708  30 72 97 D3  
\[ebp-48\]＝0012D704  42 CB A4 9D  
\[ebp-4C\]＝0012D700  65 0B 4E 08  
  
\[ebp-28\]处是检验值。壳还会用\[ebp-40\]、\[ebp-44\]、\[ebp-48\]、\[ebp-4C\]处数据来解压某些数据，这是 Armadillo V4.20 新加的手段，如果这些值被修改则会解压出错。  
其实对于 Armadillo V4.X 加壳程序，对 OutputDebugStringA 做 Hook 应该更简便。  
  
  
—————————————————————————————————  
四、搞定 Armadillo 的检验  
  
  
继续修改 1.eXe，壳所有对 GetProcAddress 的调用都会到 9619C6 处理  
  
009619C6    8B4424 08       mov eax,dword ptr ss:\[esp+8\]  
009619CA    3D 00000100     cmp eax,10000  
009619CF    72 19           jb short 009619EA ; jmp to kernel32.GetProcAddress  
009619D1    8138 52656164   cmp dword ptr ds:\[eax\],64616552  
//比较处理的是否是 ReadFile  
009619D7    75 11           jnz short 009619EA ; jmp to kernel32.GetProcAddress  
009619D9    8178 04 46696C6>cmp dword ptr ds:\[eax+4\],656C6946  
009619E0    75 08           jnz short 009619EA ; jmp to kernel32.GetProcAddress  
009619E2    B8 F0199600     mov eax,9619F0  
//是则 Hook  壳所有对 ReadFile 的调用都会到 9619F0 处理 ★  
009619E7    C2 0800         retn 8  
  
009619EA    FF25 801A9600   jmp dword ptr ds:\[961A80\] ; kernel32.GetProcAddress  
009619F0    8B0424          mov eax,dword ptr ss:\[esp\]  
009619F3    A3 891A9600     mov dword ptr ds:\[961A89\],eax  
//保存返回地址  
009619F8    C70424 051A9600 mov dword ptr ss:\[esp\],961A05  
//让 ReadFile 后返回到 961A05  
009619FF    FF25 841A9600   jmp dword ptr ds:\[961A84\] ; kernel32.ReadFile  
//ReadFile  
00961A05    FE0D 881A9600   dec byte ptr ds:\[961A88\]  
//\[961A88\]处提前写入 3 ★  
00961A0B    75 45           jnz short 00961A52  
//ReadFile 进行 3 次  
00961A0D    C605 0B1A9600 E>mov byte ptr ds:\[961A0B\],0EB  
//3 次后修改 00961A0B jmp 00961A52  
00961A14    C745 D8 03355C8>mov dword ptr ss:\[ebp-28\],845C3503  
//修正\[ebp-28\]处为跟踪原版得到的校验值 ★  
00961A1B    C745 C0 1ADA15E>mov dword ptr ss:\[ebp-40\],EA15DA1A  
00961A22    C745 BC 307297D>mov dword ptr ss:\[ebp-44\],D3977230  
00961A29    C745 B8 42CBA49>mov dword ptr ss:\[ebp-48\],9DA4CB42  
00961A30    C745 B4 650B4E0>mov dword ptr ss:\[ebp-4C\],84E0B65  
//修改\[ebp-40\]、\[ebp-44\]、\[ebp-48\]、\[ebp-4C\]处为跟踪原版得到的值 ★  
  
上面已经还原了 Armadillo 的校验值和解码参数。  
  
  
—————————————————————————————————  
五、干掉壳所设置的时间限制  
  
  
下面来解决 Armadillo 对软件设置的时间限制  
调试原版 FTPRush.exe 的 OllyDBD 还暂停在 ReadFile 的返回处  
Ctrl+S 在整个段块搜索命令序列：  
  push edi  
  or eax,0FFFFFFFF  
  push ebx  
  mov dword ptr ds:\[esi+60C\],eax  
  mov dword ptr ds:\[esi+608\],eax  
  mov dword ptr ds:\[esi+604\],eax  
找到在 011FBAC7 处  
  
011FBAAE    55              push ebp  
011FBAAF    8BEC            mov ebp,esp  
011FBAB1    51              push ecx  
011FBAB2    53              push ebx  
011FBAB3    33DB            xor ebx,ebx  
011FBAB5    381D DE4E2201   cmp byte ptr ds:\[1224EDE\],bl  
011FBABB    56              push esi  
011FBABC    8BF1            mov esi,ecx  
011FBABE    74 07           je short 011FBAC7  
//调试发现，只要这里不跳，则程序就不会过期  ★  
011FBAC0    32C0            xor al,al  
011FBAC2    E9 84020000     jmp 011FBD4B  
011FBAC7    57              push edi  
011FBAC8    83C8 FF         or eax,FFFFFFFF  
011FBACB    53              push ebx  
011FBACC    8986 0C060000   mov dword ptr ds:\[esi+60C\],eax  
011FBAD2    8986 08060000   mov dword ptr ds:\[esi+608\],eax  
011FBAD8    8986 04060000   mov dword ptr ds:\[esi+604\],eax  
011FBADE    8986 00060000   mov dword ptr ds:\[esi+600\],eax  
011FBAE4    899E 14060000   mov dword ptr ds:\[esi+614\],ebx  
011FBAEA    899E 10060000   mov dword ptr ds:\[esi+610\],ebx  
011FBAF0    E8 45D90000     call 0120943A  
011FBAF5    0FB7D0          movzx edx,ax  
011FBAF8    8A86 E2050000   mov al,byte ptr ds:\[esi+5E2\]  
011FBAFE    59              pop ecx  
011FBAFF    3C 01           cmp al,1  
011FBB01    75 43           jnz short 011FBB46  
011FBB03    8B86 BC050000   mov eax,dword ptr ds:\[esi+5BC\]  
011FBB09    3BC3            cmp eax,ebx  
011FBB0B    75 0E           jnz short 011FBB1B  
011FBB0D    8B86 4C050000   mov eax,dword ptr ds:\[esi+54C\]  
011FBB13    0FB78446 240500>movzx eax,word ptr ds:\[esi+eax\*2+524\]  
011FBB1B    8B8E C0050000   mov ecx,dword ptr ds:\[esi+5C0\]  
011FBB21    8986 0C060000   mov dword ptr ds:\[esi+60C\],eax  
011FBB27    2BC2            sub eax,edx  
011FBB29    898E 10060000   mov dword ptr ds:\[esi+610\],ecx  
011FBB2F    8986 00060000   mov dword ptr ds:\[esi+600\],eax  
011FBB35    0F89 1C010000   jns 011FBC57  
011FBB3B    899E 00060000   mov dword ptr ds:\[esi+600\],ebx  
011FBB41    E9 11010000     jmp 011FBC57  
011FBB46    3C 02           cmp al,2  
011FBB48    75 3F           jnz short 011FBB89  
011FBB4A    8B86 BC050000   mov eax,dword ptr ds:\[esi+5BC\]  
011FBB50    3BC3            cmp eax,ebx  
011FBB52    75 0E           jnz short 011FBB62  
011FBB54    8B86 4C050000   mov eax,dword ptr ds:\[esi+54C\]  
011FBB5A    0FB78446 240500>movzx eax,word ptr ds:\[esi+eax\*2+524\]  
011FBB62    8B8E C0050000   mov ecx,dword ptr ds:\[esi+5C0\]  
011FBB68    898E 10060000   mov dword ptr ds:\[esi+610\],ecx  
011FBB6E    0FB78E 22050000 movzx ecx,word ptr ds:\[esi+522\]  
011FBB75    2BCA            sub ecx,edx  
011FBB77    03C1            add eax,ecx  
011FBB79    3BC3            cmp eax,ebx  
011FBB7B    8986 00060000   mov dword ptr ds:\[esi+600\],eax  
011FBB81    0F8D D0000000   jge 011FBC57  
011FBB87    EB B2           jmp short 011FBB3B  
011FBB89    3C 03           cmp al,3  
011FBB8B    0F84 93000000   je 011FBC24  
011FBB91    3C 05           cmp al,5  
011FBB93    0F84 8B000000   je 011FBC24  
011FBB99    3C 06           cmp al,6  
011FBB9B    75 63           jnz short 011FBC00  
  
继续在 1.eXe 里面 Patch  
  
00961A37    C705 BEF99100 E>mov dword ptr ds:\[91F9BE\],420F8E9  
//选择 91F9BE 处作为 Patch 程序代码的接口 ★ 为何选择这里，下面再解释  
00961A41    50              push eax  
00961A42    A1 891A9600     mov eax,dword ptr ds:\[961A89\]  
//获取返回地址  
00961A47    2D 4E5D0100     sub eax,15D4E ★  
//计算需要修改时间限制的地址处 EAX=0121180C-15D4E=011FBABE  
00961A4C    66:C700 9090    mov word ptr ds:\[eax\],9090  
//修改 011FBABE 处为 NOP ★  
00961A51    58              pop eax  
00961A52    FF35 891A9600   push dword ptr ds:\[961A89\]  
00961A58    C3              retn  
//返回 0121180C 处继续壳的流程  
  
其实我们在 00961A4C 修改 011FBABE 处为 NOP 时会引发异常，Armadillo 新版又增加了手段，把这个壳区段设置成只读执行属性。可以再开个 OllyDBG 跟踪 FTPRush.exe 原版，看看是哪里设置这个区段属性的。  
  
BP VirtualProtect  中断后看堆栈：  
0012D6C4   0091EE22  /CALL 到 VirtualProtect 来自 原版 FTPR.0091EE1C  
0012D6C8   011F1000  |Address = 011F1000     正是需要修改的壳区段基址  
0012D6CC   00029DEA  |Size = 29DEA (171498.)  
0012D6D0   00000020  |NewProtect = PAGE\_EXECUTE\_READ  ★  
0012D6D4   0012D6F0  \\pOldProtect = 0012D6F0  
  
Ctrl+G：0091EE22  
0091EE0C    8D55 A4         lea edx,dword ptr ss:\[ebp-5C\]  
0091EE0F    52              push edx  
0091EE10    8B45 BC         mov eax,dword ptr ss:\[ebp-44\]  
//修改为： push 40  ★  
0091EE13    50              push eax  
//Nop  
0091EE14    8B4D B8         mov ecx,dword ptr ss:\[ebp-48\]  
0091EE17    51              push ecx  
0091EE18    8B55 A0         mov edx,dword ptr ss:\[ebp-60\]  
0091EE1B    52              push edx  
0091EE1C    FF15 68809500   call dword ptr ds:\[958068\] ; kernel32.VirtualProtect  
  
直接在 1.eXe 里面修改 0091EE10 处为 push 40 就行了。  
0012D6C4   0091EE22  /CALL 到 VirtualProtect 来自 原版 FTPR.0091EE1C  
0012D6C8   011F1000  |Address = 011F1000  
0012D6CC   00029DEA  |Size = 29DEA (171498.)  
0012D6D0   00000040  |NewProtect = PAGE\_EXECUTE\_READWRITE  
0012D6D4   0012D6F0  \\pOldProtect = 0012D6F0  
  
  
—————————————————————————————————  
六、Patch 程序中的限制代码  
  
  
最后的时刻到了，也是我们最终的目的：修改壳保护程序中的内部代码  
具体该在哪里设置此接口？其实只要壳对程序解码完毕就可以修改了，具体在哪里看你自己的选择了。  
继续调试原版 FTPRush.exe，Alt+M 在第二区段 00401000 设置内存访问断点，Shift+F9 中断在 OEP  
  
0085BE48    55              push ebp  
//OEP  
0085BE49    8BEC            mov ebp,esp  
0085BE4B    83C4 F4         add esp,-0C  
0085BE4E    B8 48B18500     mov eax,85B148  
0085BE53    E8 6CCEBAFF     call 00408CC4  
0085BE58    A1 A4258700     mov eax,dword ptr ds:\[8725A4\]  
0085BE5D    8B00            mov eax,dword ptr ds:\[eax\]  
0085BE5F    E8 FC5DBFFF     call 00451C60  
  
看此时堆栈：  
0012D794   0121742F  返回到 0121742F  
0012D798   00400000  ASCII "MZP"  
0012D79C   00000000  
0012D7A0   00151F2A  
0012D7A4   0000000A  
0012D7A8   0012FF2C  
0012D7AC   00000000  
0012D7B0   7FFD6000  
0012D7B4   0091F9C9  返回到 原版 FTPR.0091F9C9 来自 01217303  
  
Ctrl+G：0091F9C9     看到 0091F9C3 里面就走向 OEP 了  
  
0091F9BE    68 08EE9500     push 95EE08  
0091F9C3    FF15 30EE9500   call dword ptr ds:\[95EE30\]  
0091F9C9    83C4 04         add esp,4  
  
所以我在上面 00961A37 处把 0091F9BE 设置为接口。当然，你也可以选择其他地方。  
00961A37    mov dword ptr ds:\[91F9BE\],420F8E9  
//就是修改  0091F9BE jmp 00961ABB  
  
破解分析过程就不详述了，分析发现 00787A5D 处不跳即可显示已注册。  
00787A56    E8 3115C8FF     call 00408F8C ; <jmp.&kernel32.GetEnvironmentVariableA>  
00787A5B    85C0            test eax,eax  
00787A5D    74 25           je short 00787A84  
//应该 Nop  
00787A5F    8D85 FCFEFFFF   lea eax,dword ptr ss:\[ebp-104\]  
00787A65    8D95 00FFFFFF   lea edx,dword ptr ss:\[ebp-100\]  
00787A6B    B9 00010000     mov ecx,100  
00787A70    E8 83CAC7FF     call 004044F8  
  
继续在 1.eXe 里面 Patch  
  
00961ABB    60              pushad  
00961ABC    54              push esp  
00961ABD    6A 40           push 40  
00961ABF    68 00B04500     push 45B000  
00961AC4    68 00104000     push 401000  
00961AC9    FF15 68809500   call dword ptr ds:\[958068\] ; kernel32.VirtualProtect  
//设置 401000 段为 PAGE\_EXECUTE\_READWRITE，否则写入会异常  
00961ACF    66:C705 5D7A780>mov word ptr ds:\[787A5D\],9090  
//2 个字节的修改竟然如此麻烦   ★  
00961AD8    C705 BEF99100 6>mov dword ptr ds:\[91F9BE\],95EE0868  
//还原 91F9BE 接口处代码，还是还原吧，养成良好习惯  
00961AE2    61              popad  
00961AE3    E9 D6DEFBFF     jmp 0091F9BE  
//继续壳的流程  
  
至于 Single User License、VIP User License、Site License 等授权形式就自己找吧。  
另外，软件的脚本功能应该是需要 key 来解开部分功能代码，没有 key 就算了。  
随意找个工具制作 Patch 了，目前我还是觉得用 t.p.e 比较顺手。  
  
运用之妙，存乎一心。仅仅看是比较郁闷的，还是多多试炼吧。  
  
Game Over  
  
  
—————————————————————————————————                                  
         ,     \_/   
        /  \_.-~/            \\\_     ,        青春都一晌  
       ( /~   /              \\~-.\_  \\  
       \`\\\\  \_/                \\   ~\\ )          忍把浮名   
   \_-~~~-.)  )\_\_/;;,.          \\\_  //'  
  /'\_,\\   --~   \\ ~~~-  ,;;\\\_\_\_(  (.-~~~-.        换了脱壳轻狂  
 \`~ \_( ,\_..--\\ (     ,;'' /    ~--   /.\_\`\\   
  /~~//'   /' \`~\\         ) /--..\_, )\_  \`~  
  "  \`~"  "      \`"      /~'\`\\    \`\\\\~~\\     
                         "     "   "~'  ""  
      
         CracKed By :  stephenteh 、fly  
                2005-07-23 22:00 

 4dd466d0e1e0cc123afbe504da2462ca