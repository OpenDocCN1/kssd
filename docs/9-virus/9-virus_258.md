

# 某木马初步分析

> 原文：[`www.pediy.com/kssd/pediy06/pediy6513.htm`](https://www.pediy.com/kssd/pediy06/pediy6513.htm)

某木马初步分析  
kongfoo/2004.5.25  
　　今天早上把 QQ 挂在线上，过了一会有个人加了好友，过了一会就发了个文件  
过来，“照片 62.EXE”，JPG 的图标，用 PEiD 看一下是 ASPACK 的壳，反正正  
想要找些东西来玩玩（上个星期搞了一个星期某壳的 unpacker 还未成功，郁  
闷中）。  
　　ASPACK 的壳好脱啦，直接去入口＋3ae 的地方就是出口。  

> ```
> 代码:
> 0040A3AF    61              POPAD
> 0040A3B0    75 08           JNZ SHORT 照片 62.0040A3BA
> 0040A3B2    B8 01000000     MOV EAX,1
> 0040A3B7    C2 0C00         RETN 0C
> 0040A3BA    68 E2154000     PUSH 照片 62.004015E2  ==去 OEP
> 0040A3BF    C3              RETN
> 
> ```
> 
>   

  
　　有壳文件 111K，脱出来 56K，好像和 ASPACK 的压缩效率不相符。。。先不管。  
　　脱出来后先 bpx j 看看都用了什么 API，心里面先有个底![](img/26ed9fa78f7d02ee5a28c57dc1e1e2a8.jpg)。没发现什么  
有大杀伤力的调用，可以放心跟![](img/26ed9fa78f7d02ee5a28c57dc1e1e2a8.jpg)  
　　看代码是 VC 的程序。dump 窗口数据段有 80c834bc.jpg 的字串，呵呵，  
一定是要解一个图片出来明修栈道，暗渡陈仓啦。  

> ```
> 代码:
> 00403D8D    FF15 A0D04000   CALL DWORD PTR DS:\[<&kernel32.CreateFile>; kernel32.CreateFileA  ==打开自己。
> 
> 00401C23    FF15 3CD04000   CALL DWORD PTR DS:\[<&kernel32.SetFilePoi>; kernel32.SetFilePointer  ==去文件尾。
> 
> 00403926    FF15 6CD04000   CALL DWORD PTR DS:\[<&kernel32.ReadFile>\] ; kernel32.ReadFile  ==读 1000 字节。
> 
> 00401C23    FF15 3CD04000   CALL DWORD PTR DS:\[<&kernel32.SetFilePoi>; kernel32.SetFilePointer  ==去文件尾（就是上面的代码啦）。
> 
> 0040102E    E8 EE030000     CALL dumped\_.00401421  ==可以看到 401421/4013ca/4013bf 都被调用了 3 次
> 00401033    53              PUSH EBX                      ==401421 就是设置文件指针，4013ca 读文件
> 00401034    8D5424 30       LEA EDX,DWORD PTR SS:\[ESP+30\]
> 00401038    6A 06           PUSH 6
> 0040103A    52              PUSH EDX
> 0040103B    E8 8A030000     CALL dumped\_.004013CA
> 00401040    8D4424 38       LEA EAX,DWORD PTR SS:\[ESP+38\]
> 00401044    50              PUSH EAX
> 00401045    E8 75030000     CALL dumped\_.004013BF
> 0040104A    6A 02           PUSH 2
> 0040104C    6A F4           PUSH -0C
> 0040104E    53              PUSH EBX
> 0040104F    8BE8            MOV EBP,EAX
> 00401051    E8 CB030000     CALL dumped\_.00401421
> 00401056    53              PUSH EBX
> 00401057    8D4C24 44       LEA ECX,DWORD PTR SS:\[ESP+44\]
> 0040105B    6A 06           PUSH 6
> 0040105D    51              PUSH ECX
> 0040105E    E8 67030000     CALL dumped\_.004013CA
> 00401063    8D5424 4C       LEA EDX,DWORD PTR SS:\[ESP+4C\]
> 00401067    52              PUSH EDX
> 00401068    E8 52030000     CALL dumped\_.004013BF
> 0040106D    6A 02           PUSH 2
> 0040106F    6A FA           PUSH -6
> 00401071    53              PUSH EBX
> 00401072    894424 54       MOV DWORD PTR SS:\[ESP+54\],EAX
> 00401076    E8 A6030000     CALL dumped\_.00401421
> 0040107B    83C4 44         ADD ESP,44
> 0040107E    8D4424 28       LEA EAX,DWORD PTR SS:\[ESP+28\]
> 00401082    53              PUSH EBX
> 00401083    6A 07           PUSH 7
> 00401085    50              PUSH EAX
> 00401086    E8 3F030000     CALL dumped\_.004013CA
> 0040108B    8D4C24 34       LEA ECX,DWORD PTR SS:\[ESP+34\]
> 0040108F    51              PUSH ECX
> 00401090    E8 2A030000     CALL dumped\_.004013BF
> 00401095    83C4 10         ADD ESP,10
> 00401098    8D5424 30       LEA EDX,DWORD PTR SS:\[ESP+30\]
> 0040109C    894424 14       MOV DWORD PTR SS:\[ESP+14\],EAX
> 004010A0    68 80000000     PUSH 80
> 004010A5    52              PUSH EDX
> 004010A6    FF15 08D04000   CALL DWORD PTR DS:\[<&kernel32.GetSystemD>; kernel32.GetSystemDirectoryA  ==很明显了
> 004010AC    BF 5C704000     MOV EDI,dumped\_.0040705C                 ; ASCII "\\help3721.dll"  ==看到 3721 就有点&^$#$@#感觉$%^%#$
> 
> 00403D8D    FF15 A0D04000   CALL DWORD PTR DS:\[<&kernel32.CreateFile>; kernel32.CreateFileA  ==生成%systemdir%\\help3721.dll
> 
> 00401C23    FF15 3CD04000   CALL DWORD PTR DS:\[<&kernel32.SetFilePoi>; kernel32.SetFilePointer  ==去文件头
> 
> 004011BE    FF15 04D04000   CALL DWORD PTR DS:\[<&kernel32.GetTempPat>; kernel32.GetTempPathA  ==图片要解在临时目录
> 004011C4    BF 30704000     MOV EDI,dumped\_.00407030                 ; ASCII "80C834BC.jpg"
> 
> 00403D8D    FF15 A0D04000   CALL DWORD PTR DS:\[<&kernel32.CreateFile>; kernel32.CreateFileA  ==生成图片
> 
> 00401266    FF15 C0D04000   CALL DWORD PTR DS:\[<&shell32.ShellExecut>; shell32.ShellExecuteA  ==打开图片
> 
> 00401276    FF15 60D04000   CALL DWORD PTR DS:\[<&kernel32.WinExec>\]  ; kernel32.WinExec  ==用 rundll32 打开 help3721.dll
>                                                                                          ==rundll32 %system%\\help3721.dll,Rundll32
> 
> ```
> 
>   

  
　　之后程序结束。由于脱了壳的关系，原来附加在文件里面的图片、DLL 都被脱  
掉了，所以 2 个文件生成后，都是 0byte 大小的。知道了代码流程，跟踪一下有壳的  
程序，生成 DLL 后复制出来（有壳的程序生成的图片是一个 PLMM 哦![](img/26ed9fa78f7d02ee5a28c57dc1e1e2a8.jpg)）。  
  
　　DLL 也被加壳了。用 OD 打开，入口：  

> ```
> 代码:
> 10007001 >  60              PUSHAD
> 10007002    E8 03000000     CALL help3721.1000700A
> 10007007  - E9 EB045D45     JMP 555D74F7
> 　　又是 ASPACK，去 10007001+3ae 的地方找出口：
> 100073AF    61              POPAD
> 100073B0    75 08           JNZ SHORT help3721.100073BA
> 100073B2    B8 01000000     MOV EAX,1
> 100073B7    C2 0C00         RETN 0C
> 100073BA    68 7B230010     PUSH help3721.1000237B  ==去 OEP
> 100073BF    C3              RETN
> 
> ```
> 
>   

  
　　LordPE，点一下 OD 的 loaddll.exe，在下面的列表选 help3721.dll，再 dump full。  
ImpREC 选 loaddll 进程，Pick DLL，选 help3721.dll，OEP 填 237b。IAT autoseach，  
Get Import，Fix dump。呵呵，DLL 的脱壳就这么简单。  
　　再看看 DLL 的代码：  

> ```
> 代码:
> 1000128B    68 A4400010     PUSH dumped\_.100040A4                    ; ASCII "help3721.dll"
> 10001290    A3 28460010     MOV DWORD PTR DS:\[10004628\],EAX
> 10001295    FF15 44A00010   CALL DWORD PTR DS:\<&kernel32.GetModuleH>; kernel32.GetModuleHandleA
> 1000129B    50              PUSH EAX
> 1000129C    FF15 1CA00010   CALL DWORD PTR DS:\[<&kernel32.DisableThr>; kernel32.DisableThreadLibraryCalls  ==这个 API 我可是第一次见
>                                                                        ==看看 API 文档，原来是禁 DLL\_THREAD\_ATTACH/DETACH 通知用的，呵呵，虚惊一场
> 
> ```
> 
>   

  
　　初始化代码之后，bpx j 看看用了什么 API，咦，这次可是大大的有杀伤力，  
用了消息钩子，用了 ReadProcessMemory，用了 msvcrt 的文件操作函数，用了注  
册表操作函数，用了 ws2\_32 的网络操作函数。。。  
　　木马该用的好像都用了![大概看看 API 调用情况后就收工啦。用 rundll32 载  入 DLL 跟用 OD 的 loaddll 载入可不一样，会真正把木马运行起来的。    　　程序就不放上来了，免得变成放毒者![](img/26ed9fa78f7d02ee5a28c57dc1e1e2a8.jpg)放个图灌一下水![](img/26ed9fa78f7d02ee5a28c57dc1e1e2a8.jpg)  
  
  
\===================================  
平台：xp　　工具：OllyDBG1.10 修改版  
\=================================== 4dd466d0e1e0cc123afbe504da2462ca