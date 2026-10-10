

# PC-Guard 5.0 -> Blagoje Ceklic 之 Multi-Instrument 3.2 成功脱壳

> 原文：[`www.pediy.com/kssd/pediy11/122665.html`](https://www.pediy.com/kssd/pediy11/122665.html)

由于改了日期，不能贴出来了，只能讲一下思路了。  
  
用 PEiD 检查的结果：PC-Guard 5.0 -> Blagoje Ceklic  
  
OllyICE 载入目标文件  
异常设置不忽略 INT3 中断外的全部异常,OD 自动隐藏  
  
Alt+M 打开内存镜像，并在 00401000 下内存访问断点，Shift+F9 运行。  
  
跳过试用期窗口  
到达  00516542  
看代码，课确认是 OEP 处，注意 OEP 为 00116542  
  
【Dump 及修复 IAT】  
  
1、使用 OllyDump 插件去掉"Rebuild Import"选项后，直接在 OEP 处 dump 保存文件。  
  
注意：如果这时直接使用 ImportREC 修复文件，IAT 会存在一个无效函数，所以我们现在利用新工具 Universal Import Fixer (UIF)来获取及修正 IAT。  
  
2、在此时的 OD 中打开附件进程功能，找到目标进程，复制进程的 PID，然后打开 UIF 并粘贴到"Process ID"中，勾选"Fix Dircetly Imports"选项，OK，"Start"开始修复 IAT 表。  
UIF 修复记录中有  
IAT RVA :  
IAT SIZE:  
两个数据  
  
3、运行 ImportREC，设置选项：重建原始 FT、创建新的 IAT、修正 EP 到 OEP、使用来自磁盘的 PE 头文件头，最后找到进程并填写相关数据：  
并用 OEP，IAT RVA，IAT SIZE 完善。  
最后获取输入表，检查函数全部有效，修复抓取文件即可。  
  
接下来，比较关键  
利用 OD 载入修复后的抓取文件  
（具体的代码由于没有使用软件可供再试一次，只能直接用结果了）  
有段代码，是 60 EB 01 E3 EB 01 D4 E8 00 00 00 00 的花指令，直接改成 90 90 90 C3  
  
004B5560   $  55            push    ebp  
004B5561   .  8BEC          mov     ebp, esp  
004B5563   .  81EC 1C050000 sub     esp, 51C  
004B5569   .  8D85 E4FAFFFF lea     eax, dword ptr \[ebp-51C\]  
004B556F   .  C785 E4FAFFFF>mov     dword ptr \[ebp-51C\], 51C  
004B5579   .  50            push    eax  
004B557A   .  FF15 4CC95200 call    dword ptr \[52C94C\]               ;  MIs.00569D2F  
；此处调用的是花指令，直接返回即可！  
004B5580   .  85C0          test    eax, eax  
004B5582   .  EB 08         jmp     short 004B558C  
；直接改成原来是 74 08，直接改成 jmp  
004B5584   .  6A 00         push    0  
004B5586   .  FF15 00825700 call    dword ptr \[<&msvcrt.exit>\]       ;  msvcrt.exit  
004B558C   >  8B8D D0FCFFFF mov     ecx, dword ptr \[ebp-330\]  
004B5592   .  33C0          xor     eax, eax  
004B5594   .  3BC8          cmp     ecx, eax  
004B5596   .  EB 24         jmp     short 004B55BC  
；原来是 74 3D 还是 75 3D，直接改成 EB 24  
004B5598   .  3985 04FEFFFF cmp     dword ptr \[ebp-1FC\], eax  
004B559E   .  0F84 92010000 je      004B5736  
004B55A4   .  8B85 08FEFFFF mov     eax, dword ptr \[ebp-1F8\]  
004B55AA   .  83F8 FF       cmp     eax, -1  
004B55AD   .  0F84 83010000 je      004B5736  
004B55B3   .  83F8 1E       cmp     eax, 1E  
004B55B6   .  0F87 7A010000 ja      004B5736  
004B55BC   .  C705 D8E45400>mov     dword ptr \[54E4D8\], -2  
004B55C6   .  66:C705 E0645>mov     word ptr \[5564E0\], 1F  
004B55CF   .  33C0          xor     eax, eax  
004B55D1   .  8BE5          mov     esp, ebp  
004B55D3   .  5D            pop     ebp  
004B55D4   .  C3            retn  
改完后能够直接运行了！  
  
  
  
原破解 3.0 的帖子  
http://bbs.pediy.com/showthread.php?t=65882