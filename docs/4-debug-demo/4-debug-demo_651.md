

# 瑞星个人防火墙及瑞星 2002

> 原文：[`www.pediy.com/kssd/pediy04/kanxue080.htm`](https://www.pediy.com/kssd/pediy04/kanxue080.htm)

瑞星个人防火墙及 2002 杀毒通用破解   
  
2002 仍然没有改掉 2001 的老毛病，在 setup.dll 上捣鬼   
1.瑞星个人防火墙   
http://211.101.232.6/pcver/UpdatePad\_wnxio01sdghgyrising/rfw.exe   
rfw.exe 用 winrar 等解压缩软件解开,研究 setup.dll   
脱 setup.dll 的 upx 壳 upx -d setup.dll   
w32dasm 反汇编 setup.dll，选“函数“=〉“输出“，双击第一项 checkkey   
修改 eax=1 就不用密钥盘了，下面是改后的   
  
Exported fn(): CheckKey - Ord:0001h   
:100025F0 B801000000              mov eax, 00000001 \*\*\*   
:100025F5 C20800                  ret 0008 \*\*\*\*\*   
  
  
:100025F8 108B44240883            adc byte ptr \[ebx+83082444\], cl   
:100025FE F8                      clc   
:100025FF 06                      push es   
:10002600 0F87AD000000            ja 100026B3   
:10002606 FF2485B8260010          jmp dword ptr \[4\*eax+100026B8\]   
:1000260D E83EF2FFFF              call 10001850   
:10002612 C20800                  ret 0008   
  
2。2002 杀毒破解与上面相同，只是保护更弱，没加壳   
http://211.101.232.6/pcver/UpdatePad\_wnxio01sdghgyrising/rav.exe   
  
作者：风飘雪   
http://fpx.yeah.net   
2001。9。29  4dd466d0e1e0cc123afbe504da2462ca