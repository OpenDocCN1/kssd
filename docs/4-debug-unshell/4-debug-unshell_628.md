

# 过 Enigma Protector 2.xx 注册保护

> 原文：[`www.pediy.com/kssd/pediy11/126331.html`](https://www.pediy.com/kssd/pediy11/126331.html)

【文章标题】: 绕过 Enigma Protector 2.xx 注册保护  
【文章作者】: CodeGame  
【作者邮箱】: CodeGame@Yeah.Net  
【作者主页】: http://blog.csdn.net/codegame  
【作者 QQ 号】: 441673604  
【软件名称】: windows 计算器  
【软件大小】: 669kb  
【下载地址】: windows xp 系统自带  
【加壳方式】: The Enigma Protector 2.20 正版  
【保护方式】: The Enigma Protector  2.20 正版  
【编写语言】: VC  
【使用工具】: OllyDBG  
【操作平台】: Windows XP sp3  
【软件介绍】: 1+1=2  
【作者声明】: 文笔菜的很请谅解，只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教!  
\--------------------------------------------------------------------------------  
【详细过程】  
  参考了 http://unpack.cn/thread-59541-1-2.html 定位方式，迅速定位到了 Enigma 注册授权位置:  
  010F2CBF    E8 CC76F3FF     call calc\_em.0102A390  
  010F2CC4    8B45 E8         mov eax,dword ptr ss:\[ebp-0x18\]  
  010F2CC7    E8 FC78F3FF     call calc\_em.0102A5C8  
  010F2CCC    50              push eax  
  010F2CCD    E8 DE90FFFF     call calc\_em.010EBDB0     ; Check Registration Key Info  
  010F2CD2    85C0            test eax,eax              ; Eax =1 Success!  
  010F2CD4    0F95C0          setne al  
  010F2CD7    8B15 98FE1001   mov edx,dword ptr ds:\[0x110FE98\]      
  010F2CDD    8B12            mov edx,dword ptr ds:\[edx\]  
  010F2CDF    8842 50         mov byte ptr ds:\[edx+0x50\],al     ;<<<跟踪此处\[edx+0x50\]内存变量  
  010F2CE2    A1 98FE1001     mov eax,dword ptr ds:\[0x110FE98\]  
  
    
  这里 010EBDB0 这个 CALL 负责检测注册码是否正确，正确返回 1 否则返回 0,并把检测结果直接写入:\[edx+0x50\]内，于是再继续跟踪:\[edx+0x50\]的地址发现是在这个地方进行检测：  
01162B2D    8D049B          lea eax,dword ptr ds:\[ebx+ebx\*4\]  
01162B30    8B5486 10       mov edx,dword ptr ds:\[esi+eax\*4+0x10\]  
01162B34    8B45 FC         mov eax,dword ptr ss:\[ebp-0x4\]           ; HookIt eax=5 and edx=1 and ebx=1 and \[ebp-0x4\]=0  
01162B37    E8 30F8FFFF     call calc\_em.0116236C  
01162B3C    8945 F4         mov dword ptr ss:\[ebp-0xC\],eax  
01162B3F    E9 09010000     jmp calc\_em.01162C4D  
01162B44    8D049B          lea eax,dword ptr ds:\[ebx+ebx\*4\]  
  
0116236C 的 CALL 负责读取检测上面写入的标志，但这个位置由于是公用函数 N 多代码进行调用直接补丁肯定行不通再加上 Enigma 有多线程内联补丁保护因此不能直接硬写此处代码，经过跟踪分析发现执行到 01162B34  处并且 eax=5 and edx=1 and ebx=1 and \[ebp-0x4\]=0 此时做补丁是可行的，所以我们采用了硬件断点 HOOK 来判断实现。  
    
  1:定位 PatchAddress:  
    
  先看此块内存信息：  
   地址=01026000  
   大小=002F4000 (3096576.)  
   属主=calc\_em  01000000  
   区段=  
   类型=Imag 01001002  
   访问=R  
   初始访问=RWE  
    
  这块内存实际是 Enigma 的内置 DLL 授权模块，此块 DLL 是被加密压缩过，因此也无法直 patch，通过对比加不同的程序发现这块区域解压后的代码都不变：  
 $+13CB2D >  8D049B          lea eax,dword ptr ds:\[ebx+ebx\*4\]  
$+13CB30 >  8B5486 10       mov edx,dword ptr ds:\[esi+eax\*4+0x10\]  
$+13CB34 >  8B45 FC         mov eax,dword ptr ss:\[ebp-0x4\]           ; HookIt eax=5 and edx=1 and ebx=1 and \[ebp-0x4\]=0  
$+13CB37 >  E8 30F8FFFF     call calc\_em.0116236C  
$+13CB3C >  8945 F4         mov dword ptr ss:\[ebp-0xC\],eax  
$+13CB3F >  E9 09010000     jmp calc\_em.01162C4D  
$+13CB44 >  8D049B          lea eax,dword ptr ds:\[ebx+ebx\*4\]  
  
    
  因此 PatchAddress = BaseAddress+PatchOffsetAddress，通过上面分析得到 Enigma Protector 2.20 的 PatchOffsetAddress =  0x13CB34，不同版本的 PatchOffsetAddress 有可能不一样，BaseAddress 的获取就更简单了，直接搜索区段判断 VirtualSize 为 0x002F4000 的 VirtualAddress+GetModuleHandle(0)即为 BaseAddress，整理公式 PatchAddress = GetModuleHandle(0)+VirtualAddress+0x13CB34 定位完毕。  
    
  2.硬件 Hook:  
  这里我们采用 AddVectoredExceptionHandler 向量化异常 API 来实现，具体细节请自己 google，重点讲下 Hook 触发后的过程  
由于我们 Patch 点为  
$+13CB34 >  8B45 FC         mov eax,dword ptr ss:\[ebp-0x4\]           ; HookIt eax=5 and edx=1 and ebx=1 and \[ebp-0x4\]=0  
$+13CB37 >  E8 30F8FFFF     call calc\_em.0116236C  
因此需要先判断 eax,edx,ebx 和 \[ebp-0x4\]地址内的地址变量:  
  
  if (pException^.ContextRecord^.Eax = 5) and (pException^.ContextRecord^.Ebx = 1)  and (pException^.ContextRecord^.Edx = 1) and (pdword(pdword(pException^.ContextRecord^.Ebp - 4)^)^ = 0) then  
  begin  
    pdword(pdword(pException^.ContextRecord^.Ebp - 4)^)^ := 1;  
  end;  
  pException^.ContextRecord^.Eax := pdword(pException^.ContextRecord^.Ebp - 4)^;  
  PException^.ContextRecord^.Eip := PException^.ContextRecord^.Eip + 3; //Next  
  
  
3.整体封装：  
这里我们采用是把硬件 HOOK 和处理的过程都封装成 DLL 然后导出一个 GoPatch 的函数供目标程序调用，那么如何使目标程序加载并执行我们的 GoPatch 函数呢， 办法很多比如注入，远线程等等。。这里我采用了劫持 EIP 的方式使目标程序加载我们的 DLL 并执行 GoPatch 函数。  
  
到这里已经全部完成，运行程序直接拖拽需要 Patch 的 exe 到窗口即可直接绕过 Enigma 的注册保护直接执行。  
附件为跳保护程序和计算器 DEMO，理论上使用 2.xx 版本加壳的程序都适用。  
  
![](img/0800d8fe0e6d7aeef78306bc622f8153.jpg)  
  
  
计算器 DEMO： calc\_em.rar  
  
EnigmaPatch: PatchEnigma.rar