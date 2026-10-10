

# WaveSurround 3.2 Demo

> 原文：[`www.pediy.com/kssd/pediy03/forum1296.htm`](https://www.pediy.com/kssd/pediy03/forum1296.htm)

砍掉 WaveSurround 3.2 Demo 版的过期毛病   
Passion   
2001.9.x   
  
WaveSurround 3.2 是毛毛虫推荐的一个 WINAMP 环绕立体声插件，其 Demo 版本的时间限制形式比其老版本来说有了点变化，变成只允许使用 14 天，无 NAG 窗口。   
安装的时候用一下 Regmon，得知其安装的时间信息保存在   
HKLM/SoftWare/Wave Arts/WaveSurround for Winamp/Expiration 处，启动时检测系统时间，超过 14 天就过期，不过不留暗桩，时间调回来又能运行。   
反反 winamp 的 plugins 目录下的 dsp\_wa3d.dll，查 GetLocalTime 字串，就一个。追到其上层是：   
  
1000BDE7 Call 1000C930   
  
于是就看这儿附近。   
  
\* Possible Indirect StringData Ref from Data Obj ->"Software/Wave Arts/WaveSurround "   
                                        ->"for Winamp"   
                                  |   
:1000BDE2 68B8D00110              push 1001D0B8   
:1000BDE7 E8440B0000              call 1000C930   
:1000BDEC 83C404                  add esp, 00000004   
:1000BDEF 83F804                  cmp eax, 00000004   
:1000BDF2 0F84C1000000            je 1000BEB9        //跳到正确处。   
:1000BDF8 83F801                  cmp eax, 00000001   
:1000BDFB 754D                    jne 1000BE4A   
  
…………   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:1000BDF2(C)   
  
//1001E708 是个比较重要的全局变量，为 0 时大概报错，1 才正常。   
  
:1000BEB9 C70508E7011001000000    mov dword ptr \[1001E708\], 00000001   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:1000BC4D(C)   
|   
:1000BEC3 33C0                    xor eax, eax   
:1000BEC5 81C480020000            add esp, 00000280   
:1000BECB C3                      ret   
  
于是只有用暴力改字节了。   
1000BDF2 处：   
0F 84 C1 00 00 00  ->   
E9 C2 00 00 00 90   
也就是 je 1000BEB9 变成 jmp 1000BEB9 去掉过期限制。   
要完美的话查找“Demostration Version”，全改成了空格“20”，哈，不仅内容是正式版，连形式也是了。  4dd466d0e1e0cc123afbe504da2462ca