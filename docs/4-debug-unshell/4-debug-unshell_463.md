

# RegRun2 V3.30

> 原文：[`www.pediy.com/kssd/pediy05/pediy50322.htm`](https://www.pediy.com/kssd/pediy05/pediy50322.htm)

http://wycrack.51.net/learn/un\_RegRun2.zib   
下载完后改后缀名为 ZIP   
  
                            RegRun2 V3.30 脫殼說明   
  
大家如對這篇文章有不懂之處,請先看\[DFCG\]脫殼教學之 aspr 引入表修複篇.http://dfcg.9842.org/   
  
  
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*   
  
                      \*\*\*\*\*\[Regrun2.exe 的脫殼過程\]\*\*\*\*\*   
  
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*   
  
1,用 loader 找到殼的入口點:OEP=00401000   
  
2,在入口點 dump 下程式保存爲 un\_regrun2.exe   
  
3,用 W32dasm 反編譯程式查找字串:jmp dword ptr \[00            
  找到引入表的範圍在:0058a174,大小爲:00000ec0h   
  
4,在 un\_regrun2.exe 中找一片全 00 地區。我找的是:005e3000,大小爲:000017000h   
  
5,運行\[DFCG\]的 un\_aspr1.0,在窗口中輸入 0058a174 和 005e3000.   
  
6,按確定.在出現第二個提示窗口的時候，記下正確的引入表入口點: 001e3108 .   
  用 lordpe ,dump 出 0058a174 後的 ec0h 字節數據和 005e3000 後的 000017000h 的數據.   
  保存爲 0058a174~ec0.dmp 和 005e3000~17000.dmp 文件.   
  
7,用 dump 出的數據替換 un\_regrun2.exe 中對應地址的數據.   
  
8,改正引入表入口點爲 001e3108.   
  
9,修複其他的幾個 aspr 特殊 API.如下表:   
  
KERNEL32.DLL 中的未修複函數:   
  
0018a434    空函數        先讓其指向 Getcommandlinea 函數,以確保函數的連續性，但修改 0053afe2 處的指令爲:RETN 04,如下:   
  
0053AFDC          $-FF25 30A45>JMP DWORD PTR DS:\[<&KERNEL32.F>;  kernel32.FreeLibrary   
0053AFE2          $ C2 0400    RETN 4   
0053AFE5            90        NOP   
0053AFE6            90        NOP   
0053AFE7            90        NOP   
0053AFE8          $-FF25 38A45>JMP DWORD PTR DS:\[<&KERNEL32.G>;  kernel32.GetACP   
  
  
0018a440    GetCommandLineA   
0018a444    GetCurrentProcess   
0018a448    GetModuleHandleA   
0018a488    GetProcAddress   
  
0018a518    空函數        先讓其指向 Getcommandlinea 函數,以確保函數的連續性，但修改 0053b138 處的指令如下:   
  
0053B132          $-FF25 14A55800    JMP DWORD PTR DS:\[<&KERNEL32.L>;  kernel32.LocalFree   
0053B138          $ C2 0400          RETN 4   
0053B13B            90              NOP   
0053B13C            90              NOP   
0053B13D            90              NOP   
  
  
COMCTR32.DLL 中的未修複函數:   
  
0018a6bc    InitCommonControls   
  
  
結束語:   
    附上脫殼後再用 upx1.20 壓縮的 regrun2.exe.其他的幾個程式也可用這種方法脫殼。引入表的需修複函數相似.   
  
  
                                              -={完成}=-  4dd466d0e1e0cc123afbe504da2462ca