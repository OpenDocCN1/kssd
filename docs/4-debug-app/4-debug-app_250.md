

# 游戏《chaos;head》资源解密算法逆向不完整版

> 原文：[`www.pediy.com/kssd/pediy10/89099.html`](https://www.pediy.com/kssd/pediy10/89099.html)

首先，我要解释一下为什么这篇逆向分析的题目中有“不完全”这几个字。因为偶是一只小菜鸟，水平有限，而这个游戏的加密方式异常复杂，因此偶未能逆出最后一步的解密过程。只能将前面的解密过程分析写出来。所以请各位看官见谅……  
游戏的资源文件以 npa 以及 ngs 后缀名结尾，其中 ngs 猜测应该是游戏的对白内容的文件。而 npa 文件中有 cg.npa（cg 资源文件），sound.npa 以及 voice.npa（声音文件），system.npa（系统设置文件）以及 nss.npa（脚本资源文件）。而一般游戏脚本在解密完后都能明显地看出来的，于是我就找 nss.npa 来开刀了。  
游戏使用了 AlphaDisc 加密，这个保护是以外壳的形式将游戏保护起来的。这个外壳有检测调试器（打开\\\\.\\SICE 以及\\\\.\\TRW 等驱动来检查，查找窗口名称等）的功能，甚至运行 FileMon 跟 RegMon 这些监视软件游戏都不能启动。并且外壳有完整性校验（不太确定，因为免 cd 补丁中只用了 3 个字节来修改了一个跳转，而游戏却可以运行）。顺带还破坏了程序的输入表了。  
用 OD 载入游戏的时候要在调试选项忽略掉所有异常，然后对 CreateFileA 下断，一直按 F9 运行到打开的文件是 nss.npa 文件为止：  

引用: 004010CE  |.  6A 00         |push    0  
004010D0  |.  6A 01         |push    1  
004010D2  |.  6A 03         |push    3  
004010D4  |.  6A 00         |push    0  
004010D6  |.  6A 00         |push    0  
004010D8  |.  68 00000080   |push    80000000  
004010DD  |.  8D8424 D40300>|lea     eax, dword ptr \[esp+3D4\]  
004010E4  |.  50            |push    eax                             ;  E:\\Program Files\\Nitroplus\\CHAOS;HEAD\\nss.npa  
004010E5  |.  FF15 F0004E00 |call    dword ptr \[4E00F0\]              ;  CreateFileA  
004010EB  |.  8BF8          |mov     edi, eax  
004010ED  |.  83FF FF       |cmp     edi, -1  
004010F0  |.  897C24 14     |mov     dword ptr \[esp+14\], edi  
004010F4  |.  0F84 19020000 |je      00401313  
004010FA  |.  6A 29         |push    29  
004010FC  |.  8D4C24 20     |lea     ecx, dword ptr \[esp+20\]  
00401100  |.  6A 00         |push    0  
00401102  |.  51            |push    ecx  
00401103  |.  E8 A8E00B00   |call    004BF1B0  
00401108  |.  83C4 0C       |add     esp, 0C  
0040110B  |.  6A 00         |push    0                               ;  lpOverlapped  
0040110D  |.  8D5424 1C     |lea     edx, dword ptr \[esp+1C\]  
00401111  |.  52            |push    edx                             ;  pBytesRead  
00401112  |.  6A 29         |push    29                              ;  BytesToRead  
00401114  |.  8D4424 28     |lea     eax, dword ptr \[esp+28\]  
00401118  |.  50            |push    eax                             ;  Buffer  
00401119  |.  57            |push    edi                             ;  hFile  
0040111A  |.  FF15 88014E00 |call    dword ptr \[4E0188\]              ;  ReadFile  
00401120  |.  85C0          |test    eax, eax  
00401122  |.  0F84 E4010000 |je      0040130C                        ;  读取不成功则返回  
00401128  |.  837C24 18 29  |cmp     dword ptr \[esp+18\], 29  
0040112D  |.  0F85 D9010000 |jnz     0040130C                        ;  读出字节不为 29 则返回 

上面的代码是打开 nss.npa 文件然后读出 29 个字节.而读出的数据如下:  

引用: 0012EC94  4E 50 41 01 00 00 00 14 3A 01 00 12 6B 01 00 01  NPA ... : . k .   
0012ECA4  01 FD 00 00 00 00 00 00 00 FD 00 00 00 00 00 00   ?......?.....  
0012ECB4  00 00 00 00 00 35 30 00 00                       .....50.. 

用 WinHex 打开其他几个资源文件,发现开头 11 个字节是一样的.而开头 4 个字节是明显的文件格式的标志.  
接下来的代码是分别保存读取出来的数据以及判断文件头,一直到 0x4012cf 的一个 call,跟进去后发现是一大串读取文件以及解密的操作.首先是一个设定文件指针的操作:  

引用: 0040190C  |> \\8B8424 180100>mov     eax, dword ptr \[esp+118\]         ;  取文件句柄  
00401913  |.  53            push    ebx  
00401914  |.  55            push    ebp  
00401915  |.  57            push    edi  
00401916  |.  6A 00         push    0                                ;  dwMoveMethod  
00401918  |.  6A 00         push    0                                ;  lpDistanceToMoveHigh  
0040191A  |.  6A 29         push    29                               ;  DistanceToMove  
0040191C  |.  50            push    eax                              ;  hFile  
0040191D  |.  FF15 78014E00 call    dword ptr \[4E0178\]               ;  SetFilePointer  
00401923  |.  837E 14 00    cmp     dword ptr \[esi+14\], 0            ;  判断是否成功  
00401927  |.  C74424 14 000>mov     dword ptr \[esp+14\], 0  
0040192F  |.  0F86 13030000 jbe     00401C48                         ;  不成功则返回 

这段代码是将文件指针设定在开头 29 个字节的位置。接下来又是一串读取以及解密的操作.  

引用: 00401940  |> /83BE 34020000>/cmp     dword ptr \[esi+234\], 0  
00401947  |. |68 80000000   |push    80  
0040194C  |. |6A 00         |push    0  
0040194E  |. |75 38         |jnz     short 00401988  
00401950  |. |FF15 64004E00 |call    dword ptr \[4E0064\]              ;  GetProcessHeap  
00401956  |. |50            |push    eax  
00401957  |. |FF15 68004E00 |call    dword ptr \[4E0068\]              ;  HeapAlloc  
0040195D  |. |8BE8          |mov     ebp, eax  
0040195F  |. |85ED          |test    ebp, ebp  
00401961  |. |74 0E         |je      short 00401971                  ;  分配无效则返回  
00401963  |. |68 80000000   |push    80  
00401968  |. |55            |push    ebp  
00401969  |. |E8 422E0500   |call    004547B0                        ;  内存清 0，相当于 RtlZeroMemory  
0040196E  |. |83C4 08       |add     esp, 8  
00401971  |> |85ED          |test    ebp, ebp  
00401973  |. |89AE 34020000 |mov     dword ptr \[esi+234\], ebp  
00401979  |. |75 3B         |jnz     short 004019B6                  ;  内存地址有效则跳转  
0040197B  |> |5F            |pop     edi  
0040197C  |. |5D            |pop     ebp  
0040197D  |. |5B            |pop     ebx  
0040197E  |. |33C0          |xor     eax, eax  
00401980  |. |5E            |pop     esi  
00401981  |. |81C4 10010000 |add     esp, 110  
00401987  |. |C3            |retn  
00401988  |> |FF15 64004E00 |call    dword ptr \[4E0064\]              ;  GetProcessHeap  
0040198E  |. |50            |push    eax  
0040198F  |. |FF15 68004E00 |call    dword ptr \[4E0068\]              ;  HeapAlloc  
00401995  |. |8BF8          |mov     edi, eax  
00401997  |. |85FF          |test    edi, edi  
00401999  |. |74 0E         |je      short 004019A9  
0040199B  |. |68 80000000   |push    80  
004019A0  |. |57            |push    edi  
004019A1  |. |E8 0A2E0500   |call    004547B0                        ;  内存清 0，相当于 ZeroMemory  
004019A6  |. |83C4 08       |add     esp, 8  
004019A9  |> |85FF          |test    edi, edi  
004019AB  |. |897D 34       |mov     dword ptr \[ebp+34\], edi  
004019AE  |.^|74 CB         |je      short 0040197B  
004019B0  |. |896F 30       |mov     dword ptr \[edi+30\], ebp  
004019B3  |. |8B6D 34       |mov     ebp, dword ptr \[ebp+34\]  
004019B6  |> |8B8424 240100>|mov     eax, dword ptr \[esp+124\]        ;  跳到这里，取句柄  
004019BD  |. |6A 00         |push    0                               ;  lpOverlapped  
004019BF  |. |8D4C24 14     |lea     ecx, dword ptr \[esp+14\]  
004019C3  |. |51            |push    ecx                             ;  lpNumberOfBytesRead  
004019C4  |. |6A 04         |push    4                               ;  nNumberOfBytesToRead  
004019C6  |. |8D5424 24     |lea     edx, dword ptr \[esp+24\]  
004019CA  |. |52            |push    edx                             ;  lpBuffer  
004019CB  |. |50            |push    eax                             ;  hFile  
004019CC  |. |FF15 88014E00 |call    dword ptr \[4E0188\]              ;  ReadFile  
004019D2  |. |85C0          |test    eax, eax                        ;  判断是否成功  
004019D4  |.^|74 A5         |je      short 0040197B                  ;  失败则函数直接返回  
004019D6  |. |837C24 10 04  |cmp     dword ptr \[esp+10\], 4           ;  判断读出的字节数  
004019DB  |.^|75 9E         |jnz     short 0040197B                  ;  读出字节数不为 4 则直接返回  
004019DD  |. |68 04010000   |push    104  
004019E2  |. |8D4C24 20     |lea     ecx, dword ptr \[esp+20\]  
004019E6  |. |6A 00         |push    0  
004019E8  |. |51            |push    ecx  
004019E9  |. |E8 C2D70B00   |call    004BF1B0                        ;  内存清 0  
004019EE  |. |33DB          |xor     ebx, ebx  
004019F0  |. |83C4 0C       |add     esp, 0C                         ;  平衡堆栈  
004019F3  |. |395C24 18     |cmp     dword ptr \[esp+18\], ebx         ;  读出四个字节是脚本名称的长度  
004019F7  |. |0F86 96000000 |jbe     00401A93                        ;  少于等于 0 则跳 

这段代码是先分配 80 个字节的空间,这个空间在下面会用到.然后在 nss.npa 文件中读取 4 个字节,这 4 个字节从下面的部分可以看出是资源名称的长度.  
接下来是一个循环,每次读取一个字节,然后进行解密:  

引用: 00401A00  |> /8B8424 240100>|/mov     eax, dword ptr \[esp+124\]       ;  取句柄  
00401A07  |. |6A 00         ||push    0  
00401A09  |. |8D5424 14     ||lea     edx, dword ptr \[esp+14\]  
00401A0D  |. |52            ||push    edx  
00401A0E  |. |6A 01         ||push    1  
00401A10  |. |8D7C1C 28     ||lea     edi, dword ptr \[esp+ebx+28\]  
00401A14  |. |57            ||push    edi  
00401A15  |. |50            ||push    eax  
00401A16  |. |FF15 88014E00 ||call    dword ptr \[4E0188\]             ;  ReadFile  
00401A1C  |. |85C0          ||test    eax, eax  
00401A1E  |.^|0F84 57FFFFFF ||je      0040197B                       ;  读取失败直接返回  
00401A24  |. |837C24 10 01  ||cmp     dword ptr \[esp+10\], 1  
00401A29  |.^|0F85 4CFFFFFF ||jnz     0040197B                       ;  读出字节数不是 1 也直接返回  
00401A2F  |. |8B76 08       ||mov     esi, dword ptr \[esi+8\]         ;  开始解密读出的字节,\[esi+8\]=0x00013a14  
00401A32  |. |8B8C24 280100>||mov     ecx, dword ptr \[esp+128\]  
00401A39  |. |0FAF71 0C     ||imul    esi, dword ptr \[ecx+C\]         ;  \[ecx+c\]=0x00016b12  
00401A3D  |. |8AC3          ||mov     al, bl  
00401A3F  |. |B1 FC         ||mov     cl, 0FC  
00401A41  |. |F6E9          ||imul    cl  
00401A43  |. |8AC8          ||mov     cl, al  
00401A45  |. |8BD6          ||mov     edx, esi  
00401A47  |. |C1EA 18       ||shr     edx, 18  
00401A4A  |. |2ACA          ||sub     cl, dl  
00401A4C  |. |8BD6          ||mov     edx, esi  
00401A4E  |. |C1EA 10       ||shr     edx, 10  
00401A51  |. |2ACA          ||sub     cl, dl  
00401A53  |. |8BC6          ||mov     eax, esi  
00401A55  |. |8BB424 280100>||mov     esi, dword ptr \[esp+128\]  
00401A5C  |. |C1E8 08       ||shr     eax, 8  
00401A5F  |. |2AC8          ||sub     cl, al  
00401A61  |. |8A46 08       ||mov     al, byte ptr \[esi+8\]  
00401A64  |. |F66E 0C       ||imul    byte ptr \[esi+C\]  
00401A67  |. |2AC8          ||sub     cl, al  
00401A69  |. |8B4424 14     ||mov     eax, dword ptr \[esp+14\]        ；\[esp+14\]的值总为 0  
00401A6D  |. |8BD0          ||mov     edx, eax  
00401A6F  |. |C1EA 18       ||shr     edx, 18  
00401A72  |. |2ACA          ||sub     cl, dl  
00401A74  |. |8BD0          ||mov     edx, eax  
00401A76  |. |C1EA 10       ||shr     edx, 10  
00401A79  |. |2ACA          ||sub     cl, dl  
00401A7B  |. |8BD0          ||mov     edx, eax  
00401A7D  |. |C1EA 08       ||shr     edx, 8  
00401A80  |. |2ACA          ||sub     cl, dl  
00401A82  |. |2AC8          ||sub     cl, al  
00401A84  |. |000F          ||add     byte ptr \[edi\], cl  
00401A86  |. |83C3 01       ||add     ebx, 1  
00401A89  |. |3B5C24 18     ||cmp     ebx, dword ptr \[esp+18\]  
00401A8D  |.^\\0F82 6DFFFFFF |\\jb      00401A00        ;跳回到前面继续读取一个字节来进行解密 

上面的解密函数写成 C 代码大概就是如下:  
代码:

```
void deresourcename(char \*name,//name 指针指向要解密的字符串
          BYTE len,//字符串长度
          DWORD a,//传入的是在文件读出的数据，此处是 0x00013a14
          DWORD b,//传入的是在文件读出的 a 值后一个双字，此处为 0x00016b12
          DWORD num//资源序号
          )
{
  int temp1,temp3;
  BYTE t2,t4;
  BYTE i,j;
  for(i=0;i<len;i++)
  {

    temp1=a;    //temp1=0x00013a14
    temp1\*=b;    //temp1\*=0x0016b12
    t2=i;
    t2\*=0xfc;
    for(j=3;j>=0;j--)
    {
      temp3=temp1;
      temp3>>=j\*8;
      t2-=(char)temp3;
    }
    t4=(char)a;    //t4=0x14;
    t4\*=(char)b;  //t4\*=0x12;
    t2-=t4;
    temp1=num;    
    for(j=3;j>=0;j--)
    {
      temp3=temp1;
      temp3>>=j\*8;
      t2-=(char)temp3;
    }

    name\[i\]+=t2;
  }
}
 

```

貌似弄成 C 更难看了- -|||  
不过既然叫逆向当然要将算法写成 C 才行，解密完资源名后，接下来的是一段复制资源名到新分配的内存空间然后连续读取 1 个字节加 4 个双字。接下来比较重要的一段代码是：  

引用: 00401BB9  |.  8BB424 280100>|mov     esi, dword ptr \[esp+128\]        ;  取文件头地址  
00401BC0  |.  0FB646 11     |movzx   eax, byte ptr \[esi+11\]  
00401BC4  |.  8B7D 00       |mov     edi, dword ptr \[ebp\]            ;  取脚本名地址  
00401BC7  |.  8945 08       |mov     dword ptr \[ebp+8\], eax  
00401BCA  |.  0FB64E 10     |movzx   ecx, byte ptr \[esi+10\]  
00401BCE  |.  894D 0C       |mov     dword ptr \[ebp+C\], ecx  
00401BD1  |.  8B56 28       |mov     edx, dword ptr \[esi+28\]         ;  将第一次从文件中读出的最后一个双字保存到 edx 中  
00401BD4  |.  83C2 29       |add     edx, 29                         ;  29 是第一次读出的大小，跟 edx 相加表示什么？看来这个就是文件头的大小了  
00401BD7  |.  8BC7          |mov     eax, edi  
00401BD9  |.  8955 28       |mov     dword ptr \[ebp+28\], edx  
00401BDC  |.  E8 EF030000   |call    00401FD0                        ;  计算资源名的 hash 值  
00401BE1  |.  8BD8          |mov     ebx, eax  
00401BE3  |.  8BCF          |mov     ecx, edi  
00401BE5  |.  895D 2C       |mov     dword ptr \[ebp+2C\], ebx         ;  保存 hash 值  
00401BE8  |.  8D41 01       |lea     eax, dword ptr \[ecx+1\]  
  

其中计算 hash 的那个 call 代码如下:  

引用: 00401FD0  /$  56            push    esi  
00401FD1  |.  8BF0          mov     esi, eax                         ;  eax=文件名地址  
00401FD3  |.  8A06          mov     al, byte ptr \[esi\]               ;  取第一个字符  
00401FD5  |.  84C0          test    al, al                           ;  为空则直接返回  
00401FD7  |.  BA 21436587   mov     edx, 87654321  
00401FDC  |.  74 10         je      short 00401FEE  
00401FDE  |.  8BCE          mov     ecx, esi  
00401FE0  |>  0FB6C0        /movzx   eax, al  
00401FE3  |.  83C1 01       |add     ecx, 1  
00401FE6  |.  2BD0          |sub     edx, eax  
00401FE8  |.  8A01          |mov     al, byte ptr \[ecx\]  
00401FEA  |.  84C0          |test    al, al  
00401FEC  |.^ 75 F2         \\jnz     short 00401FE0  
00401FEE  |>  8BC6          mov     eax, esi  
00401FF0  |.  8D70 01       lea     esi, dword ptr \[eax+1\]  
00401FF3  |>  8A08          /mov     cl, byte ptr \[eax\]  
00401FF5  |.  83C0 01       |add     eax, 1  
00401FF8  |.  84C9          |test    cl, cl  
00401FFA  |.^ 75 F7         \\jnz     short 00401FF3  
00401FFC  |.  2BC6          sub     eax, esi  
00401FFE  |.  0FAFC2        imul    eax, edx  
00402001  |.  5E            pop     esi  
00402002  \\.  C3            retn 

写成 C 代码如下:  
代码:

```
DWORD namehash(char \*name)
{
  DOWRD temp=0x87654321,i=0;
  while(name\[i\])
  {
    \*(char\*)&temp-=name\[i\];
    i++;
  }
  return i\*temp;
}

```

接下来就是循环的最后一部分代码:  

引用: 00401C24  |.  8B46 0C       |mov     eax, dword ptr \[esi+C\]          ;  \[ESI+C\]=0x00016B12  
00401C27  |.  0FAF46 08     |imul    eax, dword ptr \[esi+8\]          ;  \[esi+8\]=0x00013a14  
00401C2B  |.  03C3          |add     eax, ebx                        ;  ebx 为文件名的 hash 值  
00401C2D  |.  0FAF45 24     |imul    eax, dword ptr \[ebp+24\]         ;  \[ebp+24\]为上面最后一次在文件读取的双字  
00401C31  |.  8945 14       |mov     dword ptr \[ebp+14\], eax         ;  保存到文件头结构偏移为 0x14 的地方中  
00401C34  |.  8B4424 14     |mov     eax, dword ptr \[esp+14\]  
00401C38  |.  83C0 01       |add     eax, 1  
00401C3B  |.  3B46 14       |cmp     eax, dword ptr \[esi+14\]         ;  \[esi+14\]=要解密文件头个数  
00401C3E  |.  894424 14     |mov     dword ptr \[esp+14\], eax  
00401C42  |.^ 0F82 F8FCFFFF \\jb      00401940                        ;  往回跳 

现在来看看内存窗口中的数据：  
代码:

```
01364518  00 00 00 00 00 00 00 00 11 00 11 00 81 01 08 01  ........ . .?  
01364528  B0 45 36 01 02 00 00 00 01 00 00 00 01 00 00 00  6  ... ... ...
01364538  16 04 00 00 9F CA 3F 49 00 00 00 00 FF 23 00 00    ..?I....#..
01364548  16 04 00 00 2F 1A 00 00 5E 30 00 00 29 79 EE EE    ../ ..⁰..)y 铑
01364558  18 44 36 01 38 46 36 01 00 00 00 00 00 00 00 00   D6 8F6 ........
01364568  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
01364578  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
01364588  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
01364598  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
013645A8  11 00 11 00 90 01 08 01 62 6F 6F 74 5F 91 E6 88   . .?  boot\_ 
013645B8  EA 8F CD 2E 6E 73 73 00 00 00 00 00 00 00 00 00  ?nss.........
013645C8  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
013645D8  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
013645E8  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
013645F8  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
01364608  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
01364618  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
01364628  00 00 00 00 00 00 00 00 11 00 11 00 E3 01 08 01  ........ . .?  
01364638  C0 46 36 01 02 00 00 00 01 00 00 00 01 00 00 00  6  ... ... ...
01364648  F1 03 00 00 82 D5 1A 91 00 00 00 00 15 28 00 00  ?.. ?... (..
01364658  F1 03 00 00 59 1A 00 00 5E 30 00 00 EA 7B EE EE  ?..Y ..⁰..铑
01364668  28 45 36 01 48 47 36 01 00 00 00 00 00 00 00 00  (E6 HG6 ........
01364678  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
01364688  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
01364698  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
013646A8  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
013646B8  11 00 11 00 F2 01 08 01 62 6F 6F 74 5F 91 E6 8E   . .?  boot\_ 
013646C8  B5 8F CD 2E 6E 73 73 00 00 00 00 00 00 00 00 00  ?nss.........

```

好了，从这些数据中我们可以看出不少有效的信息，根据以上的数据我可以得出以下文件头结构：  
代码:

```
struct ResourceHeader{
  DWORD nameoffset;      //文件名在内存中的地址
  DWORD resourcetype;      //不确定,在解密过程中读出的第一个字节数据
  DWORD unknown1;        //第一次读出文件的 0x29 个字节中的第 0x10 个
  DWORD unknown2;        //第一次读出文件的 0x29 个字节中的第 0xf 个
  DWORD unknownsize;      //与从文件中读出的数据有关，具体作用不详
  DWORD key1;        //最后一个变换得出的数值，用于后面解密
  DWORD unknown4;        //解密过程中读出的第一个双字的数据
  DWORD dataRVA;        //数据相对文件头的偏移，解密过程中读出的第二个双字的数据
  DWORD sizeoffile;      //资源文件的实际大小，解密过程中读出的第三个双字的数据
  DWORD unknown5;        //解密过程中读出的最后一个双字
  DWORD sizeofheader;      //所有文件头的大小
  DWORD hash;          //资源名的 hash 值
  ResourceHeader \*Flink;    //指向前继节点
  ResourceHeader \*Blink;    //指向后继节点
};

```

其实上面有些数据到目前都不能确定的，是我在后面的调试中得出来的。其实完整的分析到此为止了，后面的分析很乱，而且也没分析个所以然来。一是由于这个游戏的加密算法实在是太变态，二自然是由于我水平不够（在此要再次向写出这个游戏提取软件的痴汉公贼大大膜拜啊）。  
后面的断点怎么下？我一开始打算是继续对 CreateFileA 下的，但这样太麻烦，因为程序要不停打开 SoftICE 跟 FileMon 等软件的驱动来进行反调试，而且老是喜欢打开资源文件什么都不做又将其关闭，于是我想到对内存中的文件头结构下内存访问断点。  
于是断到这个地方：  

引用: 00401840  |> /8B48 2C       /mov     ecx, dword ptr \[eax+2C\]  
00401843  |. |3B4C24 14     |cmp     ecx, dword ptr \[esp+14\]         ;  比较 hash 值  
00401847  |. |0F85 8E000000 |jnz     004018DB  
0040184D  |. |83FB 04       |cmp     ebx, 4  
00401850  |. |8B5424 18     |mov     edx, dword ptr \[esp+18\] 

假如不是要分析 nss.npa 文件的话在第一次断下后可以就在 cmp ebx,4 那里按 F4 然后继续分析了,但我想要分析脚本文件,于是就取消内存断点,跳转语句下一句语句按 f4,知道 boot.nss 的字符串出现到堆栈窗口.当找到目标后,对 CreateFileA,SetFilePointer 以及 ReadFile 下断,在对 boot.nss 对应的文件头结构下内存访问断点.在创建完文件后停到以下地方:  

引用: 00401CB8   .  8B40 20       mov     eax, dword ptr \[eax+20\]          ;  资源大小的源数据？  
00401CBB   .  57            push    edi  
00401CBC   .  83C0 01       add     eax, 1  
00401CBF   .  E8 8C2A0500   call    00454750  
00401CC4   .  8BF8          mov     edi, eax  
00401CC6   .  85FF          test    edi, edi  
00401CC8   .  75 03         jnz     short 00401CCD                   ;  分配内存失败则返回  
00401CCA   .  5F            pop     edi  
00401CCB   .  5E            pop     esi  
00401CCC   .  C3            retn  
00401CCD   >  8B86 48010000 mov     eax, dword ptr \[esi+148\]         ;  取头文件结构地址  
00401CD3   .  8B48 28       mov     ecx, dword ptr \[eax+28\]          ;  全部头文件的大小  
00401CD6   .  0348 1C       add     ecx, dword ptr \[eax+1C\]          ;  资源的内容在文件中的偏移  
00401CD9   .  8B96 10010000 mov     edx, dword ptr \[esi+110\]  
00401CDF   .  53            push    ebx  
00401CE0   .  6A 00         push    0  
00401CE2   .  51            push    ecx  
00401CE3   .  52            push    edx  
00401CE4   .  E8 D25E0B00   call    004B7BBB                         ;  SetFilePointer 以及相关处理工作  
00401CE9   .  8B9E 48010000 mov     ebx, dword ptr \[esi+148\]         ;  取头文件结构地址  
00401CEF   .  8B86 10010000 mov     eax, dword ptr \[esi+110\]  
00401CF5   .  50            push    eax  
00401CF6   .  8BCB          mov     ecx, ebx  
00401CF8   .  8B51 20       mov     edx, dword ptr \[ecx+20\]  
00401CFB   .  52            push    edx  
00401CFC   .  6A 01         push    1  
00401CFE   .  57            push    edi  
00401CFF   .  E8 23640B00   call    004B8127                         ;  读取文件以及相关操作  
00401D04   .  83C4 1C       add     esp, 1C 

好了,这下总算搞明白那些结构中大部分数据的作用了.接下来 f9 继续运行就是读取数据,读取数据的大小比较诡异,总是读取两次,而且第二次读出的数据进行了一系列作用不明的变换.而第一次读出的数据根据结构中 unknownsize 的不同会是一次读取 2000 个字节或一次读出 1000 个字节.  
对读出的数据下内存访问断点,找到一段变换过程:  

引用: 00401D41   > /0FB61438      movzx   edx, byte ptr \[eax+edi\]        
00401D45   . |8A92 F8EC5100 mov     dl, byte ptr \[edx+51ECF8\]        ;  0x51ECF8 中的数据对于这个游戏来说是固定不变的  
00401D4B   . |2AD1          sub     dl, cl  
00401D4D   . |2AD0          sub     dl, al  
00401D4F   . |881438        mov     byte ptr \[eax+edi\], dl  
00401D52   . |83C0 01       add     eax, 1  
00401D55   . |3BC5          cmp     eax, ebp            ;ebp 保存的是文件头结构中 unknownsize 的值  
00401D57   .^\\72 E8         jb      short 00401D41                   ;  跳回去 

这段写成 C 代码如下:  
代码:

```
{
  char \*buff;
  char \*key=0x51ecf8;
  char tmp;
  int i;
  for(i=0;i<unknownsize;i++)
  {
    tmp=key\[buff\[i\]\];
    tmp-=(char)key1;  //key1 为文件头中的数据
    tmp-=i;
    buff\[i\]=tmp;
  }
}

```

接下来又是一段分配内存的操作,接着就是真正的解密函数了,限于水平,最后的解密函数没逆出来,于是这篇文章就只好到此为止了。  
虽然这次逆向没能将游戏的资源解密算法完整逆向出来，但经过此次分析实在是认识到自己还需继续努力学习啊。