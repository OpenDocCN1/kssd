

# 一个未知壳的脱壳笔记

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-561.htm`](https://www.pediy.com/kssd/pediy07/pediy7-561.htm)

【破解作者】 clide2000\[DFCG\]\[OCN\]  
【作者邮箱】 54arma@sina.com  
【作者主页】 www.chinadfcg.com  
【使用工具】 Ollydbg;LoadPe;ImportREC;WinHex;topo12  
【破解平台】 Win9x/NT/2000/XP  
【软件名称】 家庭 VCD 相册制作系统 标准版 2.0   
【下载地址】 http://www.jcok.com/soft/VCDAlbumDIYB.exe  
【软件简介】 《家庭 VCD 相册制作系统》是制作 VCD 相册的最佳选择。可轻松将您喜欢的图片，数码相机拍出的照片和扫描的相片进行数码变换处理，配上音乐、图象特效、文字等，在数分钟之内制造出一流的专业效果的家庭 VCD 数码相册。  
【软件大小】 2.39M  
【加壳方式】 未知壳  
【破解声明】 我是一只小菜鸟，偶得一点心得，愿与大家分享：）  
\--------------------------------------------------------------------------------  
【破解内容】  
  
  
一寻找 OEP 和 dump:  
经调试分析，发现此壳会先申请一段内存空间，然后把部分代码解压到新申请的空间，而且以后会经常调用。所以我们要在原文件尾部新建一个大小一样的区段，用于存放这部分代码：  
1 寻找申请内存空间的大小：  
首先设置 OD 忽略所在异常。OD 载入后，停在：  
006DE000 >  E8 AA000000       call VCDAlbum.006DE0AF  
006DE005    2D E02D0000       sub eax, 2DE0  
006DE00A    0000              add byte ptr ds:\[eax\], al  
006DE00C    0000              add byte ptr ds:\[eax\], al  
006DE00E    0000              add byte ptr ds:\[eax\], al  
006DE010    003D E02D002D     add byte ptr ds:\[2D002DE0\], bh  
006DE016    E0 2D             loopdne short VCDAlbum.006DE045  
006DE018    0000              add byte ptr ds:\[eax\], al  
  
下 bp VirtualAlloc 断点，f9 运行，会中断在:  
  
77E5AC72 >  55                push ebp                                   ; VCDAlbum.<ModuleEntryPoint>  
77E5AC73    8BEC              mov ebp, esp  
77E5AC75    FF75 14           push dword ptr ss:\[ebp+14\]  
77E5AC78    FF75 10           push dword ptr ss:\[ebp+10\]  
77E5AC7B    FF75 0C           push dword ptr ss:\[ebp+C\]  
77E5AC7E    FF75 08           push dword ptr ss:\[ebp+8\]  
77E5AC81    6A FF             push -1  
77E5AC83    E8 9CFFFFFF       call kernel32.VirtualAllocEx  
77E5AC88    5D                pop ebp  
77E5AC89    C2 1000           retn 10  
  
看看现在的堆栈内容，记住加\*\*\*\*一行中的 Size 的大小,然后 Ctrl+f9 后，eax 中，放的就是申请到的内存的起始地址了，即 370000：  
  
0012FFB0    006DE0E3  /CALL to VirtualAlloc from VCDAlbum.006DE0DD  
0012FFB4    00000000  |Address = NULL  
0012FFB8    000068B0  |Size = 68B0 (26800.)                  \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
0012FFBC    00001000  |AllocationType = MEM\_COMMIT  
0012FFC0    00000004  \\Protect = PAGE\_READWRITE  
  
此时我们记住 68b0H 这个十六进制数什么，就可以先关闭 OD 加载的进程  
2,给原程序增加新的区段（就是加个新区段，用来存放原来放入 370000 段的代码）  
  1)用 WinHew 之类的工具先在程序的尾部增加 7000H 的空间(原文件大小是 B41FF,增加 7000H(十进制应该是 28672)空间后大小为 BB1FF.  
  
  2)用 topo12.exe 工具增加一个新的区段，大小为 28672H。成功后信息如下：  
    28672 bytes added at:  
    -memory address: 006E1000h  
    -file offset:    000b4200h  
  不放心的话，可以用 loadPe 载入刚修改完的程序看看  
  正常的话会看到最下面多出一个名为.topo0 的区段，其具体信息是(只列出了新加段的信息)  
  
  Name        VOffset        VSize       ROffset         RSize          Flags  
  
  .topo0      002E1000       00007000    000B4200        00007000       E0000020  
  
  
3 好，下面可以寻找 OEP 和 dump 了  
  
 1)OD 重新加载这个程序  
  
  下 bp VirtualAlloc 断点，f9 运行，会中断在:  
77E5AC72 >  55                push ebp                                   ; VCDAlbum.<ModuleEntryPoint>  
77E5AC73    8BEC              mov ebp, esp  
77E5AC75    FF75 14           push dword ptr ss:\[ebp+14\]  
77E5AC78    FF75 10           push dword ptr ss:\[ebp+10\]  
77E5AC7B    FF75 0C           push dword ptr ss:\[ebp+C\]  
77E5AC7E    FF75 08           push dword ptr ss:\[ebp+8\]  
77E5AC81    6A FF             push -1  
77E5AC83    E8 9CFFFFFF       call kernel32.VirtualAllocEx  
77E5AC88    5D                pop ebp  
77E5AC89    C2 1000           retn 10                 ；直接在这一行上按 F4  
  
  取消断点，然后直接在  77E5AC89 一行上按 F4，之后，把 eax 中的 00370000 修改成 006E1000  
  下 bp VirtualFree 中断两次后，Ctrl+f9 返回到这：  
  
00372FBD    5B                pop ebx  
00372FBE    83C3 0C           add ebx, 0C  
00372FC1  ^ EB B3             jmp short 00372F76  
00372FC3    8B85 B8020000     mov eax, dword ptr ss:\[ebp+2B8\]  
00372FC9    0BC0              or eax, eax  
00372FCB    0F85 81000000     jnz 00373052  
00372FD1    8BBD C0020000     mov edi, dword ptr ss:\[ebp+2C0\]  
  
清除断点：现在 Ctrl+s 搜索:  
mov edx, dword ptr ds:\[edx+4\]  
mov dword ptr ds:\[edx+50\], 1000  
mov dword ptr ss:\[ebp+258\], ebp  
mov eax, dword ptr ss:\[ebp+2C8\]  
add eax, dword ptr ss:\[ebp+2B4\]  
jmp eax  
  
找到：  
  
00373126    8B52 04           mov edx, dword ptr ds:\[edx+4\]  
00373129    C742 50 00100000  mov dword ptr ds:\[edx+50\], 1000  
00373130    89AD 58020000     mov dword ptr ss:\[ebp+258\], ebp  
00373136    8B85 C8020000     mov eax, dword ptr ss:\[ebp+2C8\]  
0037313C    0385 B4020000     add eax, dword ptr ss:\[ebp+2B4\]  
00373142  - FFE0              jmp eax                 ;在这里 F2 下断点，中断后，eax 中就是 OEP  
  
在上面 00373142  - FFE0              jmp eax 中断后，按一下 F7 后来到 OEP  
  
0050BE3C    55                push ebp                   ;这里就是 OEP 了  
0050BE3D    8BEC              mov ebp, esp  
0050BE3F    83C4 F0           add esp, -10  
0050BE42    B8 ECB95000       mov eax, VCDAlbum.0050B9EC  
0050BE47    E8 A4AEEFFF       call VCDAlbum.00406CF0  
0050BE4C    A1 3CF55000       mov eax, dword ptr ds:\[50F53C\]  
0050BE51    8B00              mov eax, dword ptr ds:\[eax\]  
0050BE53    E8 3097F7FF       call VCDAlbum.00485588  
0050BE58    A1 3CF55000       mov eax, dword ptr ds:\[50F53C\]  
0050BE5D    8B00              mov eax, dword ptr ds:\[eax\]  
0050BE5F    BA 9CBE5000       mov edx, VCDAlbum.0050BE9C  
0050BE64    E8 1793F7FF       call VCDAlbum.00485180  
0050BE69    8B0D 00F45000     mov ecx, dword ptr ds:\[50F400\]             ; VCDAlbum.00510FD8  
  
  
  
现在可以可以用 LoadPe 来 dump 了，右击该进程后，首先执行一下"correct ImageSize"，然后 dump full 出文件。  
  
  
  
二寻找 IAT 的 Rav 和 Size(不知道如果不用 ImportREC 来修复，还请大家指点)  
  
再次从新载入程序，下 bp GetProcAddress 断点，F9 运行，中断第三次后，Ctrl+f9 返回到:  
003730B5    8907           mov dword ptr ds:\[edi\], eax           ;注意这里的\[edi\]地址中的内空，就是 IAT 的 Rva 了,此时\[edi\]地址是 511208  
003730B7    5A             pop edx  
003730B8    0FB642 FF      movzx eax, byte ptr ds:\[edx-1\]  
003730BC    03D0           add edx, eax  
003730BE    42             inc edx  
003730BF    83C7 04        add edi, 4  
003730C2    59             pop ecx  
003730C3  ^ E2 CA          loopd short 0037308F  
003730C5  ^ EB 93          jmp short 0037305A  
  
  
由 003730C5  ^ EB 93          jmp short 0037305A 来到上面看一下  
  
0037305A    8B3A           mov edi, dword ptr ds:\[edx\]  
0037305C    0BFF           or edi, edi  
0037305E    75 02          jnz short 00373062              ;注意这里，当 jnz 不跳时，表示 IAT 表处理完毕  
00373060    EB 65          jmp short 003730C7              ;可以直接在这里下断。  
00373062    03BD B4020000  add edi, dword ptr ss:\[ebp+2B4\]  
00373068    83C2 05        add edx, 5  
0037306B    8BF2           mov esi, edx  
  
当程序中断在 00373060    EB 65          jmp short 003730C7 里，可以看到 IAT 是的大小是从 511208~511B08  
  
  
  
现在可以请出 ImportREC,OEP 为:10BE3C  RVA 为：111208，Size：用 1000 即可  
  
修复后可以正常运行。但还需要解决跨平台问题。  
  
方法很简单（在此对 fly 的指点表示衷心的感谢，方法是会了，但原理不清楚，可以是不同平台下同一功能的函数名字不一样？还请大家指点）  
具体方法是在修复前，先手动在 ImportREC 里，把 kernel32.dll 中的 RestoreLastError 修改为 SetLastError 这个函数  
这样就可以解决跨平台问题了(在 xp\_sp1 和 win2000\_sp4 下测试通过)  
  
  
  
  
\--------------------------------------------------------------------------------  
【破解总结】  
  
  
第一次脱这样的壳，上面只是我的笔记，有点乱，有哪些不对的地方，请大家指点。  
在此对帮助和指们我们这样小菜瓜大侠们表示由衷的感谢  
另：此软件脱壳后还有两处暗桩，有兴趣的可以玩玩，不在不这里公布了  
\--------------------------------------------------------------------------------  
【版权声明】 本文纯属技术交流, 转载请注明作者并保持文章的完整, 谢谢! 

鼓励一下  
这个应该是 hying 旧版壳  
  
另外：没必要 bp VirtualAlloc 后修改 EAX 值 

去年这个时候老看到这种壳，  
也有好多人发脱文了  
搜一下 Hying  
[`bbs.pediy.com/showthread.php...mp;pagenumber=1`](http://bbs.pediy.com/showthread.php?s=&threadid=6606&perpage=15&highlight=Hying&pagenumber=1)  
[`bbs.pediy.com/showthread.php...highlight=Hying`](http://bbs.pediy.com/showthread.php?s=&threadid=11218&highlight=Hying)  
[`bbs.pediy.com/showthread.php...highlight=Hying`](http://bbs.pediy.com/showthread.php?s=&threadid=11271&highlight=Hying)  
  
新版的 IAT 会乱序，修复麻烦 

 4dd466d0e1e0cc123afbe504da2462ca