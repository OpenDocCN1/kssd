

# 1tox

> 原文：[`www.pediy.com/kssd/pediy02/forum205.htm`](https://www.pediy.com/kssd/pediy02/forum205.htm)

 一个不错的文件分割与合并工具，可在华军软件园下载。网上已有破解，但那是暴力破解法。   
程序破解难度不太，程序注册后产生一个名为 1TOX.LIC 的文本文件，注册信息保存在内。   
1.暴力破解法：   
  改 83C4043BC1742E6A10   
    83C4043BC1752E6A10   
2.注册码找法：   
  在 0167：00409950 CMP EAX，ECX  （此时直接下 R EAX ECX 也可）   
  ？ EAX 输入的注册码   
  ？ ECX 真正的注册码   
相应的程序代码如下   
:004098AF 8D442414                lea eax, dword ptr \[esp+14\]   
:004098B3 8D4C2468                lea ecx, dword ptr \[esp+68\]   
:004098B7 50                      push eax  此处的 EAX 为输入的 NAME   
  
\* Possible StringData Ref from Data Obj ->"1toX"   
                                  |   
:004098B8 6868844200              push 00428468   
:004098BD 51                      push ecx              此处 ECX 为输入的 FIRST NAME   
:004098BE 8D9424B8060000          lea edx, dword ptr \[esp+000006B8\]   
。   
。   
:0040992C 8B442410                mov eax, dword ptr \[esp+10\]   
:00409930 8D9424BC000000          lea edx, dword ptr \[esp+000000BC\]   
:00409937 35F0BD6824              xor eax, 2468BDF0   
:0040993C 52                      push edx   
:0040993D A308C94200              mov dword ptr \[0042C908\], eax   
:00409942 E8010E0100              call 0041A748  计算注册码的 CALL   
:00409947 8B0D08C94200            mov ecx, dword ptr \[0042C908\]   
:0040994D 83C404                  add esp, 00000004   
:00409950 3BC1                    cmp eax, ecx  此处的 ECX 为你想要的。   
:00409952 742E                    je 00409982    不等就注册不成功   
:00409954 6A10                    push 00000010   
  
另此程序的注册有个怪现象，你可在 00409950 处下 R ECX EAX 程序注册成功，在程序的 ABOUT 项中显示的注册码为你   
输入的假码，重新启动后程序依然为注册版（假码也可注册？）。只要一改动或删除 1TOX.lic 文件，就可成为非注册版。   
  
另 WINPATCH1。2 的注册码可在 0167：0041CA37 PUSH EBP 时 D EAX 得到。（一个好用的 PATCH 制作工具）   
WINHEX9。53   
改 7515B003E817C1F8FF   
  EB3A--------------   
  803D9D75450000   
  -------------1   
即可。   4dd466d0e1e0cc123afbe504da2462ca