
# 继续一个扭曲了 `CALL` 的小程序,高手来试试用脚本还原

> 原文：[`www.pediy.com/kssd/pediy11/125367.html`](https://www.pediy.com/kssd/pediy11/125367.html)

引用: 这是上次那个 `http://bbs.pediy.com/showthread.php?t=124927`

这次比上次那个会难一点吧![](img/962324edb48d57effae0c0571850a66b.jpg "酷")

上传的附件

| ![](img/1baac1a3eff5ac15dae6f33dccc134d9.jpg) || `复件WenW64.zip`  

 |

楼主好厉害啊，搞了半天都搞不明白，不过要还原还是可以的。

```
00404170   jmp         00404179  
00404172   xor         dword ptr [esp],0x4142        ; 'AB'  
00404179   xor         dword ptr [esp],0x3839        ; '89'  
00404180   add         esp,4  
00404183   jmp         dword ptr [esp-0x4]  
```

手工还原一下：

```
00401000   push        0  
00401002   call        00404000                //GetModuleHandleA  
00401007   mov         [0040303C],eax  
0040100C   push        0xA  
0040100E   push        0  
00401010   push        0  
00401012   push        dword ptr [0040303C]  
00401018   call        00404000                //00401024  
0040101D   push        0  
0040101F   call        00404000                
00401024   push        ebp  
00401025   mov         ebp,esp  
00401027   add         esp,0xB0  
0040102A   mov         dword ptr [ebp-0x30],0x30        ; '0'  
00401031   mov         dword ptr [ebp-0x2C],0x2003  
00401038   mov         dword ptr [ebp-0x28],0x401443  
0040103F   mov         dword ptr [ebp-0x24],0x0  
00401046   mov         dword ptr [ebp-0x20],0x0  
0040104D   push        dword ptr [ebp+0x8]  
00401050   pop         [ebp-0x1C]  
00401053   mov         dword ptr [ebp-0x10],0x10  
0040105A   mov         dword ptr [ebp-0xC],0x0  
00401061   mov         dword ptr [ebp-0x8],0x403000        ; ".Alone"  
00401068   push        0x64        ; 'd'  
0040106A   push        dword ptr [ebp+0x8]  
0040106D   call        00404000                //LoadIconA  
00401072   mov         [ebp-0x18],eax  
00401075   push        0x7F00  
0040107A   push        0  
0040107C   call        00404000                //LoadCursorA  
00401081   mov         [ebp-0x14],eax  
00401084   mov         dword ptr [ebp-0x4],0x0  
0040108B   lea         eax,[ebp-0x30]  
0040108E   push        eax  
0040108F   call        00404000                //RegisterClassExA  
00401094   push        0  
00401096   push        dword ptr [ebp+0x8]  
00401099   push        0  
0040109B   push        0  
0040109D   push        0x320  
004010A2   push        0x320  
004010A7   push        0xC8  
004010AC   push        0xC8  
004010B1   push        0xCF0000  
004010B6   push        0x403027        ; "[易经]六十四卦圆图"  
004010BB   push        0x403000        ; ".Alone"  
004010C0   push        0  
004010C2   call        00404000                //CreateWindowExA  
004010C7   mov         [ebp-0x50],eax  
004010CA   push        1  
004010CC   push        dword ptr [ebp-0x50]  
004010CF   call        00404000                //ShowWindow  
004010D4   push        dword ptr [ebp-0x50]  
004010D7   call        00404000                //UpdateWindow  
004010DC   push        0  
004010DE   push        0  
004010E0   push        0  
004010E2   lea         eax,[ebp-0x4C]  
004010E5   push        eax  
004010E6   call        00404000                //GetMessageA  
004010EB   cmp         eax,0  
004010EE   jz          00401104  
004010F0   lea         eax,[ebp-0x4C]  
004010F3   push        eax  
004010F4   call        00404000                //TranslateMessage  
004010F9   lea         eax,[ebp-0x4C]  
004010FC   push        eax  
004010FD   call        00404000                //DispatchMessageA
```
