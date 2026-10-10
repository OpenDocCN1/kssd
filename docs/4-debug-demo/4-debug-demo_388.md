

# EXCEL 服务器 2006 企业版 V6.8 的加密思路研究

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-145.htm`](https://www.pediy.com/kssd/pediy09/pediy09-145.htm)

>     有个朋友找我，说看看勤哲 EXCEL 服务器，我便从网上下了个 6。8 企业版，初步一看，认为还是以为简单，以前我记得有人解过 5.6 的版本，但具说是解的不全,过不了试用期! 而且原程序没有加壳，且有一个月的试用期！  
>     
>   还是老规矩，用 OD 载入 EXCEL，运行后，直接启动登录界面，  
>   300019C4 E> $  6A 74      push 74  
>   300019C6    .  68 101B003>push EXCEL.30001B10  
>   300019CB    .  E8 4C01000>call EXCEL.30001B1C  
>   300019D0    .  33DB       xor ebx,ebx  
>   300019D2    .  895D E0    mov dword ptr ss:\[ebp-20\],ebx  
>   300019D5    .  53         push ebx                              ; /pModule => NULL  
>   300019D6    .  8B3D D0140>mov edi,dword ptr ds:\[<&KERNEL32.GetM>; |kernel32.GetModuleHandleA  
>   300019DC    .  FFD7       call edi                              ; \\GetModuleHandleA  
>   300019DE    .  66:8138 4D>cmp word ptr ds:\[eax\],5A4D  
>     
>   此时，设置 OD 事件中断为：当 DLL 载入时，我们点登录按扭  
>     
>   当中断在 ESSA。DLL 时，我们进入 DLL 中，  
>   DLL 中应该有以下几个加密点：  
>   1）试用版的 NAG 提示：即 30 天  
>   2）5 个客户端的限制  
>   3）文件校验；即当你修改了一个或几个文件时，他会让你再也写不了服务器了！  
>     
>   对上述几点的跟踪：首先可以下断：MessageBoxA ,跟到 NAG 的提示与客户端的限制！  
>   NAG 提示的关键代码：本人未做详细记录，有兴趣自己调试！  
>   02C5E5DA    .  51         push ecx  
>   02C5E5DB    .  8942 08    mov dword ptr ds:\[edx+8\],eax  
>   02C5E5DE    .  8B45 8C    mov eax,dword ptr ss:\[ebp-74\]  
>   02C5E5E1    .  8942 0C    mov dword ptr ds:\[edx+C\],eax  
>   02C5E5E4    .  8D55 A0    lea edx,dword ptr ss:\[ebp-60\]  
>   02C5E5E7    .  52         push edx  
>   02C5E5E8    .  FFD7       call edi                              ;  <&MSVBVM60.\_\_vbaVarIndexLoad>  
>   02C5E5EA    .  8B1D 2413B>mov ebx,dword ptr ds:\[<&MSVBVM60.\_\_vb>;  MSVBVM60.\_\_vbaI2ErrVar  
>   02C5E5F0    .  83C4 1C    add esp,1C  
>   02C5E5F3    .  50         push eax  
>   02C5E5F4    .  FFD3       call ebx                              ;  <&MSVBVM60.\_\_vbaI2ErrVar>  
>   以下是试用期的检查：  
>     
>   02C2B719    .  8945 B0    mov dword ptr ss:\[ebp-50\],eax  
>   02C2B71C    .  74 56      je short ESSa.02C2B774  
>   02C2B71E    .  8B85 E4FEF>mov eax,dword ptr ss:\[ebp-11C\]  
>   02C2B724    .  8B95 D8FEF>mov edx,dword ptr ss:\[ebp-128\]  
>   02C2B72A    .  8975 BC    mov dword ptr ss:\[ebp-44\],esi  
>   02C2B72D    .  8DB8 9C11C>lea edi,dword ptr ds:\[eax+2C6119C\]  
>   02C2B733    .  C702 67040>mov dword ptr ds:\[edx\],80040467  
>   02C2B739    .  3937       cmp dword ptr ds:\[edi\],esi  
>   02C2B73B    .  75 0C      jnz short ESSa.02C2B749  
>   02C2B73D    .  57         push edi  
>   02C2B73E    .  68 2CA6BE0>push ESSa.02BEA62C  
>   02C2B743    .  FF15 8012B>call dword ptr ds:\[<&MSVBVM60.\_\_vbaNe>;  MSVBVM60.\_\_vbaNew2  
>   02C2B749    >  8B3F       mov edi,dword ptr ds:\[edi\]  
>     
>   文件校验用的是 CRC，可以手工改过跳转，也可以修正 CRC，任选一种！  
>     
>   以做完后，存为一个全新的 essa.dll!  
>     
>   此时我启动都感觉正常了，以为大功告成了！ 可当用旧版数据，或数据量达到 30 天的后的时候，你发现，EXCEL 服务器停止工作了！  
>   此时才知道，远没有这么简单，EXCEL 服务器，作为一种服务程序的提供，另外一重加密就是在服务提供对 SQL 数据进行检查定期限！  
>     
>   我们首先打开  
>   ESSvcMgr.exe ，点启动服务后，很快就停止了！ 用 OD 载入  
>     
>   004017C4 E> $  68 9829400>push ESSvcMgr.00402998  
>   004017C9    .  E8 EEFFFFF>call <jmp.&MSVBVM60.#100>  
>   004017CE    .  0000       add byte ptr ds:\[eax\],al  
>   004017D0    .  0000       add byte ptr ds:\[eax\],al  
>   004017D2    .  0000       add byte ptr ds:\[eax\],al  
>   004017D4    .  3000       xor byte ptr ds:\[eax\],al  
>   004017D6    .  0000       add byte ptr ds:\[eax\],al  
>   004017D8    .  40         inc eax  
>   以下为启动标志  
>   004053B4    .  66:85DB    test bx,bx  
>   004053B7       74 07      je short ESSvcMgr.004053C0  
>   004053B9    .  C745 E4 FF>mov dword ptr ss:\[ebp-1C\],-1  
>   004053C0    >  8B1D 48104>mov ebx,dword ptr ds:\[<&MSVBVM60.\_\_vb>;  MSVBVM60.\_\_vbaHresultCheckObj  
>   004053C6    >  8B45 E4    mov eax,dword ptr ss:\[ebp-1C\]  
>     
>   00406C40    .  33FF       xor edi,edi  
>   00406C42    .  68 9439400>push ESSvcMgr.00403994                ;  UNICODE "ESService"  
>   00406C47    .  897D D8    mov dword ptr ss:\[ebp-28\],edi  
>   00406C4A    .  E8 E102000>call ESSvcMgr.00406F30  
>   00406C4F    .  66:3B46 34 cmp ax,word ptr ds:\[esi+34\]  
>   00406C53       74 31      je short ESSvcMgr.00406C86  
>   00406C55    .  8B16       mov edx,dword ptr ds:\[esi\]  
>   00406C57    .  8D4D D8    lea ecx,dword ptr ss:\[ebp-28\]  
>   00406C5A    .  51         push ecx  
>   00406C5B    .  50         push eax  
>   00406C5C    .  56         push esi  
>   00406C5D    .  66:8946 34 mov word ptr ds:\[esi+34\],ax  
>     
>   本以为在这改后，可以使用的，但实际上搞错了，这个程序只是服务的 HOOK 的报告，就是侦察服务的运行状态，而非真正的服务控制！  
>   而我们看出，真正的是 ESService 服务！对 ESService 用 IDA 反汇编后，你可以看到以下函数！  
>   名称位于 ESServic, 条目 47  
>    地址=0040600C  
>    区段=.rdata  
>    类型=输入  
>    名称=ADVAPI32.SetServiceStatus  
>   名称位于 ESServic, 条目 51  
>    地址=00406008  
>    区段=.rdata  
>    类型=输入    (已知)  
>    名称=ADVAPI32.StartServiceCtrlDispatcherA  
>   名称位于 ESServic, 条目 41  
>    地址=0040601C  
>    区段=.rdata  
>    类型=输入  
>    名称=ADVAPI32.OpenServiceA  
>   这些才是服务真正的启动点！  
>   服务控制的原型如下：  
>   #include "ntservice.h"  
>     
>   class CMyService : public CNTService  
>   {  
>   public:  
>       CMyService();  
>       virtual BOOL OnInit();  
>       virtual void Run();  
>       virtual BOOL OnUserControl(DWORD dwOpcode);  
>     
>       void SaveStatus();  
>     
>       // Control parameters  
>       int m\_iStartParam;  
>       int m\_iIncParam;  
>     
>       // Current state  
>       int m\_iState;  
>   };  
>     
>   而对于 一个服务的调试，我们常常和附加的方式进行！这点在 MSDN 中也有描述，  
>   我们对他 IDA 反汇编后，可以找到相关函数，加密的方式主要是：时间的控制！修改后，再启动。便正常工作了！  
>     
>   本文只对加密方法做个介绍，不汲及软件的破解，欢迎你来和我交流！  
>     
> \--------------------------------------------------------------------------------  
> 【经验总结】  
>   服务的加密应该还是有点特色的，多种方法混合使用，如文件校验、数据时间校验！都会给解密带来难度！由于本人文字水  
>   平有限，写的不好，请不要向我扔石头！  
>     
> \--------------------------------------------------------------------------------  
> 【版权声明】: 谢谢观看, 转载请注明作者并保持文章的完整, 谢谢!http://newsoft88.ys168.com 

 4dd466d0e1e0cc123afbe504da2462ca