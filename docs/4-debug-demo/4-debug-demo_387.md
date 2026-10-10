

# Eltima 系列软件和谐方法

> 原文：[`www.pediy.com/kssd/pediy10/94942.html`](https://www.pediy.com/kssd/pediy10/94942.html)

Step1:  
脱掉 Armadillo  
Step2:  
搜索程序内存空间，如果用 Armadillo Nanomites Fixer 处理过 Nanomites 则特征码为：  
EB 03 90 90 90 90 90 90 有可能在程序里会有多处,其中会有一处的代码形如这样  

代码:

```
0046FB70    55              push    ebp
0046FB71    8BEC            mov     ebp, esp
0046FB73    51              push    ecx
0046FB74    C745 FC 0000000>mov     dword ptr \[ebp-4\], 0
0046FB7B    EB 03           jmp     short 0046FB80

```

没处理过或用其它软件处理则搜索  
EB 03 ?? ?? ??  90 90 90 90  
鼠标放到 46FB70 这一行  
可以看到  
代码:

```
本地调用来自 0040DAEE, 0041819B, 00419273, 0042122B, 004283C2, 
0044476E, 00444803, 00445FE9, 004478B6, 0044B2D5, 0044C629, 
0044D0F6, 004536E1, 004551B0, 0045A729, 0046151A, 00461747, 
00462A92, 00463467, 00465057, 00465279, 00468071, 00471CF4　

```

随便选择一个。返回  
代码:

```
0041819B  |.  E8 D0790500   call    0046FB70///返回到这里
004181A0  |.  3D 10010000   cmp     eax, 110                         ;  Switch (cases 27..20B)
004181A5  |.  7F 1B         jg      short 004181C2
004181A7  |.  74 0C         je      short 004181B5
004181A9  |.  83F8 27       cmp     eax, 27
004181AC  |.  74 2D         je      short 004181DB
004181AE  |.  3D C3000000   cmp     eax, 0C3
004181B3  |.  EB 17         jmp     short 004181CC
004181B5  |>  C705 185D8F00>mov     dword ptr \[8F5D18\], 1            ;  Case 110 (WM\_INITDIALOG) of switch 004181A0
004181BF  |.  32C0          xor     al, al
004181C1  |.  C3            retn

```

这个函数返回的就是授权的类型，有商业旗舰版，个人版……等等……自选。只需要修改刚才那个函数的 eax 返回值即可  
  
随便找一个跳转跟随连续按两次回车，让后你就进了  
004181DB  |> \\E8 90890500   call    00470B70                         ;  Cases 27 (WM\_ICONERASEBKGND),20B (WM\_XBUTTONDOWN) of switch 004181A0  
  
这个函数  

引用: 00470C16  |.  83CA FF       or      edx, FFFFFFFF  
00470C19  |.  F0:0FC111     lock xadd dword ptr \[ecx\], edx  
00470C1D  |.  4A            dec     edx  
00470C1E  |.  85D2          test    edx, edx  
00470C20  |.  7F 08         jg      short 00470C2A  
00470C22  |.  8B08          mov     ecx, dword ptr \[eax\]  
00470C24  |.  8B11          mov     edx, dword ptr \[ecx\]  
00470C26  |.  50            push    eax  
00470C27  |.  FF52 04       call    dword ptr \[edx+4\]  
00470C2A  |>  32C0          xor     al, al  
00470C2C  |.  E9 D3000000   jmp     00470D04///主要是修改跳转地址，为了让这个函数返回 1，所以把跳转地址提前一条指令 

找到这样一个地方，修改  
  
这样就大功告成……  
  
  
其实还有最重要的一点我们没做……  
Step3:  
找一片空地，写补丁代码  
  
代码:

```
$ "Azure\[LCG\]"
;$
pushad
mov ebx,dword ptr \[5AF484\]//SetEnvironmentVariableA
push 05AE4E0//你的名字
push 00401000//需要设置的几个变量
call    ebx
push 05AE4E0
push 00401000
call    ebx
push 05AE4E0
push 00401000
call    ebx
popad
jmp  00465D23///跳向 OEP

```

  
需要设置的变量需要自己找……可以从程序里找所有调用 GetEnvironmentVariableA 的地方……它查的啥你就设置啥。  
step4:  
右键保存  
step5:  
设置 OEP 为你刚才的这段代码，然后运行……看到啥了？哈哈……好戏上演咯……  
  
此方法适用于 Eltima 公司的大部分软件，有空的自己去搞……  
  
step6:  
为了 Anti YingCracker 党，再加个强壳：ZP,NP,VMP 亦可  
  
  
总结一下……一个公司不能所有的软件都用一种保护方法……保护方式必须多样，否则就好比这个公司的软件一样……一种方法……通杀= =![](img/e71d9210a215d5b66755d038631e0c83.jpg "恐怖!")  
  
  
作者:Azure\[LCG\]  
时间:2009.8.3  
转载时请保持文章完整，谢谢！