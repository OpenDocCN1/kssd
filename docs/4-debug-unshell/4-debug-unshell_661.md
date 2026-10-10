

# Execryptor 2.2.5.1 脱壳笔记

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-654.htm`](https://www.pediy.com/kssd/pediy07/pediy7-654.htm)

前几天看到 h 前辈破解了 ExeCryptor 2.2.6  
贴子在这里 http://bbs.pediy.com/showthread.php?s=&threadid=17286  
  
南蛮妈妈不自量力 从看雪主页下载 ExeCryptor 2.2.5.1 来学习一下  
下载在这里 http://www.pediy.com/tools/PACK/Protectors/EXECryptor/EXECryptor%20V2.2.5.1.zip  
  
  
  
XPsp2 + DS3.2 + IceExt0.67 + flyOD1.1 + WinHex  
  
过程比较乱, 更像是笔记, 希望得到您的指点  
  
  
一 oep  
  
运行主程序, 第一个区段是从 401000 到 4C8000  
所以我用 WinHex 的 RAM 编辑器, 打开 EXECryptor 的主要内存, alt-g 到 4C8000, 向上找  
因为是 Delphi 的程序, 所以向上翻翻,向下翻翻, 可以确定 oep 是 4C74D0  
  
  
二 dump  
  
这个东东我在 OD 下跑不起来, 在 SI 下也跑不起来, 但是在 SI 下似乎跑得更远  
  
006DEE6A >  E8 F7FEFFFF      call EXECrypt.006DED66                    //这个是 ep   
006DEE6F    05 272D0000      add eax,2D27  
006DEE74    FFE0             jmp eax                                       
006DEE76    E8 EBFEFFFF      call EXECrypt.006DED66                    //这个是 TLS 的 CALLBACK, 比 ep 先执行  
006DEE7B    05 DE220000      add eax,22DE  
006DEE80  ^ FFE0             jmp eax                                      
006DEE82    E8 04000000      call EXECrypt.006DEE8B  
  
下 2 个断点  
bpmb 6DEE6A x          //在 ep 处下断  
bpmb 6DEE76 x          //在 TLS 的 CALL 下断  
  
主程序跑起来  
先断在 6DEE76 这个时候 d 一下 401000, u 一下 4C74D0(oep)显然是空的  
bd 掉 6DEE76 这个断点, 继续跑  
  
断在 ep 处,也就是 6DEE6A  
这个时候发现各个区段都已经解码了, 4C74D0(oep)处是  
  
004C74D0   E9 A0C30D00       jmp  005A3875  
  
bpmb 4C74D0 x  
bpmb 5A3875 x  
再跑  
  
虽然没有断在 4C74D0 但断在了 5A3875   
这时候, ESP 是 13FFC4, EBP 是 13FFF0  
13FFC4 的值是 7C816D4F 正好是接在 kernel32 的 CALL oep 的后面  
  
南蛮妈妈觉得到这儿可以 dump 了, 清掉所有断点  
壳在 TLS 的 CALL 中创建了 10 几个线程  
这个时候如果把当前 eip 改成 jmp eip 出来用 LordPE dump 是不行的  
如果把线程都杀了, 写个 int 3 ,让 od 接管来 dump 也不太稳定  
用 IceExt 的!dump  
  
总大小是 2DF000  
理想的应该是!dump \\??\\f:\\1.bin 400000 2DF000  
但 Windows 管理内存是段页式的, 并不是所有的页都在内存中  
  
找一块空地, r eip , 6DEF00   
a 6DEF00  
写一段话,保证要 dump 的都在内存中  
  
006DEF00    B8 00104000      mov eax,EXECrypt.00401000  
006DEF05    8B18             mov ebx,dword ptr ds:\[eax\]  
006DEF07    05 00100000      add eax,1000  
006DEF0C    3D 00804C00      cmp eax,EXECrypt.004C8000  
006DEF11  ^ 7C F2            jl short EXECrypt.006DEF05  
006DEF13    90               nop  
006DEF14    90               nop  
  
执行到 6DEF13 后, 就可以  
!dump \\??\\f:\\1.bin 400000 c8000  //PE 头和 CODE 段  
再从 4C8000 到 519000 过一遍  
!dump \\??\\f:\\2.bin 4c8000 51000  //DATA 段到 rsrc 段  
再是 519000 到 5AF000  
!dump \\??\\f:\\3.bin 519000 96000  //m.2mwp5l 段  
最后是 5AF000 到 6DF000  
!dump \\??\\f:\\4.bin 5af000 130000 //nt5pe0gm 段  
  
  
三 修复 PE 格式  
  
把 1.bin  2.bin 3.bin 4.bin 用 WinHex 拼起来  
然后用 LordPE 打开 拼起来的文件, 修一些东西  
  
EntryPoint:    002DEE6A -> C74D0  
SectionTable:  把每个 ROffset 改成和 VOffset 一样的值; RSize 改成和 VSize 一样的值  
TlsTable:      (001AF110, 00000018) -> (F2000, 18)  
  
得到 1.exe  
  
  
四  输入表  
随便找个 delphi 的程序把输入表扣出来, 贴到 1.exe  
贴的时候, 要选上新建 IAT, 去掉修复 ep to oep ,选上新增区段  
南蛮妈妈借的是 flashfxp  
得到 1\_.exe, 这时可以用 od 载入 1\_.exe  
看到 IAT 是在 4ED168 开始  
  
修复加密的输入表， 南蛮妈妈的处理方法是  
随便找一个属于 kernel32 中的  
eip 改成到加密的地址的入口  
对 kernel32 的.text 段下内存访问断点  
走着走着， 会看到，它把 kernel32 的导出函数地址一个一个拿出来加密， 和真实的 api 地址的密文比较  
  
0056E485    3B45 F8         cmp eax,dword ptr ss:\[ebp-8\]  
0056E488  ^ 0F85 8434FFFF   jnz 00561912                                       ; 比较 api 加密的密文, 不等则跳上去计算下一个  
0056E48E    8B45 EC         mov eax,dword ptr ss:\[ebp-14\]                      ; 相等, 找到了, 这里可以下断  
0056E491    E9 59530300     jmp 005A37EF                                       ; 1\_.005A37EF  
  
再走着走着  
  
004FFCCC    8B52 1C         mov edx,dword ptr ds:\[edx+1C\]  
004FFCCF    0355 FC         add edx,dword ptr ss:\[ebp-4\]                       ;   
004FFCD2    03C2            add eax,edx                                        ;   
004FFCD4    8B00            mov eax,dword ptr ds:\[eax\]  
004FFCD6    0345 FC         add eax,dword ptr ss:\[ebp-4\]                       ; 这一句做完后， eax 即是真实的 api 地址  
004FFCD9  - E9 E2C50700     jmp 0057C2C0                                       ; 1\_.0057C2C0  
004FFCDE    C600 8B         mov byte ptr ds:\[eax\],8B  
  
  
在 oep 处写一段代码修复  
  
004C74D0 >  BE 68D14E00     mov esi,4ED168  
004C74D5    81FE 10D94E00   cmp esi,4ED910  
004C74DB    77 20           ja short 004C74FD                                  ; 完成后跳走  
004C74DD    AD              lods dword ptr ds:\[esi\]                            ; IAT 中取一个出来  
004C74DE    85C0            test eax,eax  
004C74E0  ^ 74 F3           je short 004C74D5                                  ; 如果是 0，取下一个  
004C74E2    3D 00000001     cmp eax,1000000  
004C74E7  ^ 77 EC           ja short 004C74D5                                  ; 如果>1000000 表示没有加密，放过  
004C74E9    56              push esi                                           ;   
004C74EA    8BD4            mov edx,esp  
004C74EC    83EA 04         sub edx,4  
004C74EF    8915 10754C00   mov dword ptr ds:\[4C7510\],edx                      ; 保存 esp  
004C74F5    FFD0            call eax                                           ; 加密的 iat 跑进去  
004C74F7    5E              pop esi                                            ; 得到真实的 api 地址后写回去  
004C74F8    8946 FC         mov dword ptr ds:\[esi-4\],eax  
004C74FB  ^ EB D8           jmp short 004C74D5                                 ; 继续下一个  
004C74FD    90              nop                             
  
004FFCD6    0345 FC         add eax,dword ptr ss:\[ebp-4\]                       ; 这一句做完后， eax 即是真实的 api 地址  
004FFCD9    8B25 10754C00   mov esp,dword ptr ds:\[4C7510\]                      ; 恢复 oep 修复代码处的堆栈  
004FFCDF    C3              retn                                               ; 返回 oep 处的修复代码  
  
跑完后得到了，全部有效的 IAT  
可以用 ImportREC 扣出来 再次修复原来的 1.exe  
得到 2.exe  
  
  
五 跨平台  
修复跨平台累啊, 壳在 oep 前 对 m.2mwp5l 段的 N 个地方进行了初始化, 存放一些东西  
有壳使用的加密的 api 地址, 有壳 GlobalAlloc 过的空间入口  
南蛮妈妈必须把这些都找出来, 全部清 0, 使脱壳后的程序跑到那些地方时好重新初始化  
方法就是去掉内存访问异常, 在 OD 下跑, 一个一个追出来, 体力活  
52E9C4 52E9A8 4FFE84 575D20 539A24 575D7C 575D70 52E9DC 52E9D0 575D50  
4FFEC4 55E914 5A0704 5A06F8 5776FC 520558 52AED4 5776F0 54FDD8   
大概还有一些南蛮妈妈可能没有找全, 找到的地址统统的清 0  
脱壳后程序启动时有三个线程地址空间, 有 2 个是 anti 和校验的, 1 个是检验注册码的  
所以关掉 2 个, 一共 3 种情况, 都试一次,就知道 nop 掉哪 2 个了  
  
  
  
  
南蛮妈妈 Special Thanks to heXer 

 4dd466d0e1e0cc123afbe504da2462ca