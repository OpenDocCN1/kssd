

# 破解  三国群英 2 

> 原文：[`www.pediy.com/kssd/pediy03/forum1258.htm`](https://www.pediy.com/kssd/pediy03/forum1258.htm)

破解  三国群英 2   
这个游戏我解了很久的了，只不过没有写出来罢了。（这个太容易解了）  
  首先运行 TRW2000  
  按 CTRL+N  
  输入：bpx getdrivetypea  
  
:00453A97 8945F8                  mov dword ptr \[ebp-08\], eax <---这些不用管它了，直追到:4302f2  
:00453A9A 837DF805                cmp dword ptr \[ebp-08\], 00000005  
:00453A9E 750B                    jne 00453AAB  
:00453AA0 8B55FC                  mov edx, dword ptr \[ebp-04\]  
:00453AA3 891510044900            mov dword ptr \[00490410\], edx  
:00453AA9 EB02                    jmp 00453AAD  
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:00453A9E(C)  
|  
:00453AAB EBC8                    jmp 00453A75  
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:  
|:00453A82(C), :00453AA9(U)  
|  
:00453AAD 8BE5                    mov esp, ebp  
:00453AAF 5D                      pop ebp  
:00453AB0 C3                      ret  
  \*  
  \*  
  \*  
 追到：  
:004302F2 E856FFFFFF              call 0043024D  <---将这个 CALL 斩了就行了。(即将 e856ffffff 改为：9090909090 就可以了，自己再用 HIEW 改了它就能不用光盘玩三国群英 2 了）  
:004302F7 E8541F0200              call 00452250  
  
\* Possible StringData Ref from Data Obj ->"SANGO2.PAK"  
                                  |  
:004302FC 685CB54800              push 0048B55C  
:00430301 E8DD220200              call 004525E3  
  
  
2001 年 7 月 20 日写  
富强工作室  
E-MAIL: lfq168@163.com  
主页:lfq668.yeah.net