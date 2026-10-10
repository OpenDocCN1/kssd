

# 一款油价屏串口通信软件的协议破解

> 原文：[`www.pediy.com/kssd/pediy10/90778.html`](https://www.pediy.com/kssd/pediy10/90778.html)

分享一下，技术很菜~~~，高手可以跳过。  
公司的国外客户想抢别人的生意，要复制一款油价屏的软件，通信协议未知。  
拿到软件，先用 peid 查了一下，VC++写的。  
先了解一下软件，点击 Change Com 1/=...按钮，com 口在增加。点击下面的按钮，编辑框提示 ko，我猜可以是失败的意思。  
开 OD,下断 bpx ReadFile，在开一个串口查看器，实时查看当前串口数据。  
点击 AFFICHAGE，断在  

代码:

```
00401923  |. /73 28         jnb     short 0040194D
00401925  |> |391D 40C24100 /cmp     dword ptr \[41C240\], ebx
0040192B  |. |75 20         |jnz     short 0040194D
0040192D  |. |8B0D 3CC24100 |mov     ecx, dword ptr \[41C23C\]
00401933  |. |53            |push    ebx                             ; /pOverlapped
00401934  |. |68 40C24100   |push    0041C240                        ; |pBytesRead = CPU6FE.0041C240
00401939  |. |8D4424 20     |lea     eax, dword ptr \[esp+20\]         ; |
0040193D  |. |6A 01         |push    1                               ; |BytesToRead = 1
0040193F  |. |50            |push    eax                             ; |Buffer
00401940  |. |51            |push    ecx                             ; |hFile => 00000154 (window)
00401941  |. |FF15 38414100 |call    dword ptr \[<&KERNEL32.ReadFile>>; \\ReadFile
00401947  |. |FFD6          |call    esi
00401949  |. |3BC7          |cmp     eax, edi
0040194B  |.^|72 D8         \\jb      short 00401925
0040194D  |> \\807C24 18 06  cmp     byte ptr \[esp+18\], 6
00401952  |.  0F85 19010000 jnz     00401A71                         ;  此处修改跳转方向
00401958  |.  8D7C24 4C     lea     edi, dword ptr \[esp+4C\]
0040195C  |.  83C9 FF       or      ecx, FFFFFFFF
0040195F  |.  33C0          xor     eax, eax

```

00401941  |.  FF15 38414100 |call    dword ptr \[<&KERNEL32.ReadFile>>; \\ReadFile  我们停在这里了。串口调试器上显示 05  
  
  
F8 单步，但这里 0040194D  |> \\807C24 18 06  cmp     byte ptr \[esp+18\], 6 停止，  
然后修改 byte ptr \[esp+18\]内容 ，让其等于 6，然后 F9，断下  
代码:

```
004019B7  |.  8B15 3CC24100 |mov     edx, dword ptr \[41C23C\]
004019BD  |.  53            |push    ebx                             ; /pOverlapped
004019BE  |.  68 40C24100   |push    0041C240                        ; |pBytesRead = CPU6FE.0041C240
004019C3  |.  8D4C24 20     |lea     ecx, dword ptr \[esp+20\]         ; |
004019C7  |.  6A 01         |push    1                               ; |BytesToRead = 1
004019C9  |.  51            |push    ecx                             ; |Buffer
004019CA  |.  52            |push    edx                             ; |hFile => 00000154 (window)
004019CB  |.  FF15 38414100 |call    dword ptr \[<&KERNEL32.ReadFile>>; \\ReadFile
004019D1  |.  FFD6          |call    esi
004019D3  |.  3BC7          |cmp     eax, edi
004019D5  |.^ 72 D8         \\jb      short 004019AF
004019D7  |>  807C24 18 15  cmp     byte ptr \[esp+18\], 15
004019DC  |.  0F85 80000000 jnz     00401A62

```

停在 004019CB  |.  FF15 38414100 |call    dword ptr \[<&KERNEL32.ReadFile>>; \\ReadFile  
串口调试器上显示 05 02 33 30 38 31 2C 31 31 31 37 32 2C 32 32 32 31 33 2C 33 33 33 33 34 2C 34 34 34 36 35 2C 35 35 35 39 36 2C 36 36 36 03 02   
此时 按照原来的思路，修改  
004019D7  |>  807C24 18 15  cmp     byte ptr \[esp+18\], 15 处 cmp     byte ptr \[esp+18\]为 15，发现数据又发了一遍，所以，这里是不能修改的，而 05 02 33 30 38 31 2C 31 31 31 37 32 2C 32 32 32 31 33 2C 33 33 33 33 34 2C 34 34 34 36 35 2C 35 35 35 39 36 2C 36 36 36 03 02 采用了 LRC 校验，这个有兴趣的可以慢慢跟，  
继续 F9，  
代码:

```
00401A48  |.  53            |push    ebx                             ; /pOverlapped
00401A49  |.  68 40C24100   |push    0041C240                        ; |pBytesRead = CPU6FE.0041C240
00401A4E  |.  8D5424 20     |lea     edx, dword ptr \[esp+20\]         ; |
00401A52  |.  6A 01         |push    1                               ; |BytesToRead = 1
00401A54  |.  52            |push    edx                             ; |Buffer
00401A55  |.  50            |push    eax                             ; |hFile => 00000154 (window)
00401A56  |.  FF15 38414100 |call    dword ptr \[<&KERNEL32.ReadFile>>; \\ReadFile
00401A5C  |.  FFD6          |call    esi
00401A5E  |.  3BC7          |cmp     eax, edi
00401A60  |.^ 72 D9         \\jb      short 00401A3B
00401A62  |>  8A4424 18     mov     al, byte ptr \[esp+18\]            ;  串口读
00401A66  |.  C605 28C24100>mov     byte ptr \[41C228\], 1
00401A6D  |.  3C 06         cmp     al, 6
00401A6F  |.  74 06         je      short 00401A77

```

停在  
00401A56  |.  FF15 38414100 |call    dword ptr \<&KERNEL32.ReadFile>>; \\ReadFile  
  
然后我们看  
00401A6D  |.  3C 06         cmp     al, 6  
我们尝试 修改 al 为 6 ，然后 F9  
软件运行了，看，AFFICHAGE 按钮下的编辑框显示 OK，我们协议就破解清楚了，显示上位机发送 05，然后下位机 06 应答，然后是上位机发送数据块，然后下位机应答 06.  
到此结束，谢谢大家，请批评指正拍砖灌水。

上传的附件
| ![ || CPU6FE.rar  
 |