

# Private exe Protector 1.8 IAT 加密分析\[添加 IATFixer\]

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-593.htm`](https://www.pediy.com/kssd/pediy08/pediy8-593.htm)

Private exe Protector 1.8 IAT 加密分析  
水平有限，只有 IAT 加密分析，其他的保护一律没开，也没分析。实例程序用的加密解密光盘里面的 MD5calc.加壳方式是可选的加密方式都不开.( IAT 加密是默认的)  
  
加壳后的程序的 IAT 被完全清空了，运行后，壳直接向 IAT 中填充壳 IAT 解密函数的地址。然后，通过计算得到正确的 API 地址，直接进入 API 然后返回。  
OD 载入，文件被作过手脚。只能停在 system breakpoint.   //晕，今天又停在了正常的 OEP.不知道是怎么搞得。  
He GetVersion.  
F9 运行。第二次断下后。运行到返回。向上看看就是 OEP.  
随便找一个 call \[IAT\]，在数据窗口中跟随，看到 IAT 了。  
这是这个例子的 IAT  
00405000  004121A6  MD5calcu.004121A6  
00405004  00412221  MD5calcu.00412221  
00405008  00412287  MD5calcu.00412287  
0040500C  004122E8  MD5calcu.004122E8  
00405010  0041231C  MD5calcu.0041231C  
00405014  00412391  MD5calcu.00412391  
00405018  00412415  MD5calcu.00412415  
0040501C  0041248C  MD5calcu.0041248C  
00405020  00412522  MD5calcu.00412522  
00405024  00412589  MD5calcu.00412589  
00405028  00412630  MD5calcu.00412630  
0040502C  0041269E  MD5calcu.0041269E  
00405030  00412709  MD5calcu.00412709  
00405034  00412784  MD5calcu.00412784  
00405038  0041280D  MD5calcu.0041280D  
0040503C  00412887  MD5calcu.00412887  
00405040  004128F8  MD5calcu.004128F8  
00405044  00412928  MD5calcu.00412928  
00405048  004129BD  MD5calcu.004129BD  
0040504C  00412A42  MD5calcu.00412A42  
00405050  00412AC3  MD5calcu.00412AC3  
00405054  00412B69  MD5calcu.00412B69  
00405058  00412BD3  MD5calcu.00412BD3  
0040505C  00412C01  MD5calcu.00412C01  
00405060  00412CA2  MD5calcu.00412CA2  
00405064  00412CF4  MD5calcu.00412CF4  
00405068  00412D8B  MD5calcu.00412D8B  
0040506C  00412DEB  MD5calcu.00412DEB  
00405070  00412E32  MD5calcu.00412E32  
00405074  00412ECE  MD5calcu.00412ECE  
00405078  00412F4D  MD5calcu.00412F4D  
0040507C  00412FD5  MD5calcu.00412FD5  
00405080  00413023  MD5calcu.00413023  
00405084  00413047  MD5calcu.00413047  
00405088  00413097  MD5calcu.00413097  
0040508C  0041312B  MD5calcu.0041312B  
00405090  00000000  
00405094  00411D92  MD5calcu.00411D92  
00405098  00411DE8  MD5calcu.00411DE8  
0040509C  00411E6F  MD5calcu.00411E6F  
004050A0  00411F03  MD5calcu.00411F03  
004050A4  00411F99  MD5calcu.00411F99  
004050A8  0041201F  MD5calcu.0041201F  
004050AC  0041204F  MD5calcu.0041204F  
004050B0  004120C2  MD5calcu.004120C2  
004050B4  0041212E  MD5calcu.0041212E  
  
可见，全部被壳的函数填充了，每个函数都不一样。  
多跟踪几个解密过程。可以发现如下的规律。  
  
解密的过程：  
一共有三层  
第一层：  
每个 API 函数，对应一段相应的解密函数。  
刚进入的这段函数，都是执行下面的过程  
1.push 一个 unsigned int 立即数。  
2.计算进一步解密函数的地址（第二层解密函数地址）.  
3．进入第二层函数。  
其中步骤 1 有 3 种实现方式:  
(1)  
push xxxxxxxx  
mov \[esp\] xxxxxxxx  
(2)  
push r32  
mov \[esp\] xxxxxxxx  
(3)  
push xxxxxxxx  
步骤 2 也有若干种实现方式  
但是变化仅仅是使用不同的寄存器。而且总是得到同一个地址（对同一个程序，同一台机器来说）  
步骤 3 是通过在堆栈上写入步骤 2 计算出来的地址,通过最后的 retn 实现的.  
其中一段的实际代码如下:  
push    edi  
mov     \[esp+4+var\_4\], 0B182DFA9h  
push    36F05CB9h  
push    eax  
mov     eax, ds:dword\_4081FE  
add     eax, 0C224h  
mov     \[esp+0Ch+var\_8\], eax  
pop     eax  
retn    //进入第二层  
我这里的第二层函数的地址是 003AC224  
(注意,刚刚进入第 2 层的时候,\[esp\]=上一层步骤 1 push 的立即数)  
代码如下:  
003AC224    55                 push    ebp  
003AC225    89E5               mov     ebp, esp    
003AC227    50                 push    eax  
003AC228    53                 push    ebx  
003AC229    51                 push    ecx  
003AC22A    52                 push    edx  
003AC22B    56                 push    esi  
003AC22C    57                 push    edi         //上面的操作相当 pushad  
003AC22D    8B45 04            mov     eax, \[ebp+4\]  
003AC230    50                 push    eax         //第一层步骤 1push 的立即数  
003AC231    E8 B2FFFFFF        call    003AC1E8   //进入第三层  
003AC236    8945 04            mov     \[ebp+4\], eax  
003AC239    5F                 pop     edi        
003AC23A    5E                 pop     esi  
003AC23B    5A                 pop     edx  
003AC23C    59                 pop     ecx  
003AC23D    5B                 pop     ebx  
003AC23E    58                 pop     eax        //相当 popad  
003AC23F    5D                 pop     ebp  
003AC240    C3                 retn  
进入第三层函数:  
003AC1E8    55                 push    ebp  
003AC1E9    8BEC               mov     ebp, esp  
003AC1EB    56                 push    esi  
003AC1EC    8B4D 08            mov     ecx, \[ebp+8\]  
003AC1EF    33F6               xor     esi, esi  
003AC1F1    A1 58003B00        mov     eax, \[3B0058\]  
003AC1F6    83C0 04            add     eax, 4  
003AC1F9    8B10               mov     edx, \[eax\]  
003AC1FB    83C0 04            add     eax, 4  
003AC1FE    85D2               test    edx, edx  
003AC200    76 18              jbe     short 003AC21A  
003AC202    3B08               cmp     ecx, \[eax\]  
003AC204    75 0E              jnz     short 003AC214  
003AC206    83C0 04            add     eax, 4  
003AC209    8B30               mov     esi, \[eax\]  
003AC20B    A1 58003B00        mov     eax, \[3B0058\]  
003AC210    3330               xor     esi, \[eax\]  
003AC212    EB 06              jmp     short 003AC21A  
003AC214    83C0 08            add     eax, 8  
003AC217    4A                 dec     edx  
003AC218  ^ 75 E8              jnz     short 003AC202  
003AC21A    8BC6               mov     eax, esi  
003AC21C    5E                 pop     esi  
003AC21D    5D                 pop     ebp  
003AC21E    C2 0400            retn    4  
  
通过分析上面的代码知:  
API 地址的解密方式如下:  
在\[3B0058\]== 00BB0000(不同的程序,不同的机器地址不一样)有一个表,结构如下   
  
表头是下列结构  
struct tablehead  
{  
DWORD MagicNum;  
DWORD count;     //API 结构的个数  
}  
表身是一个由下列结构组成的数组  
struct API  
{  
DWORD ID;  
DWORD key;  
}  
  
在 00BB0000 存放的方式是表头+表身  
  
IAT 中的正确的 API 地址为:  
在表中查找第一层步骤 1 中的立即数。也就是说用找 struct API 结构中的 ID 来和第一层步骤 1 中的那个立即数比较。  
如果相等  
那么  
API Address = Key  xor MagicNum;  
例如:  
下面是我这个程序的表:  
00BB0000  6A55116E  
00BB0004  0000002D  
00BB0008  4844D879  
00BB000C  1D83BD70  
00BB0010  865A2255  
00BB0014  1D86D824  
00BB0018  D4C9B887  
00BB001C  1D84B9C3  
00BB0020  369744FB  
00BB0024  1D87037F  
00BB0028  AC9E8550  
00BB002C  1D87024A  
00BB0030  509362AB  
00BB0034  1D87E2F4  
00BB0038  BB32A0F6  
00BB003C  1D84DAEB  
00BB0040  3B9BF46E  
00BB0044  1D87733E  
00BB0048  6CAACE09  
00BB004C  1D86A072  
00BB0050  BAFA03D7  
00BB0054  16D5D7A1  
00BB0058  C1E6CFF5  
00BB005C  16D43D16  
00BB0060  62F88496  
00BB0064  16D5B5EE  
00BB0068  962C31C7  
00BB006C  16D69DD7  
00BB0070  5752A351  
00BB0074  16D5DFAA  
00BB0078  A3861600  
00BB007C  16D63F45  
00BB0080  72F11E39  
00BB0084  16D58DC3  
00BB0088  3FC1BD8D  
00BB008C  16DD3EAA  
00BB0090  C97C1FFF  
00BB0094  16D5BD46  
00BB0098  B1866570  
00BB009C  6A6FD19E  
00BB00A0  ADE2CB95  
00BB00A4  16D50F80  
00BB00A8  2D66B1C5  
00BB00AC  16D43DE3  
00BB00B0  4CCF1A0F  
00BB00B4  16D405C5  
00BB00B8  251097CC  
00BB00BC  6A6FD032  
00BB00C0  AB40BF8D  
假如第一层中的立即数是 4844D879  
 那么 API 地址就为 1D83BD70 xor 6A55116E=77D6AC1E  
  
修复的方法见附件中的源代码  
这个程序只能修复这个实例程序的 IAT,因为那张解密表我是直接写到了程序里面。要修复其他程序，把那张表替换了就可以了。  
发现有些函数 壳是自己实现的，解密出来的函数还是指向壳里，这部分我是手动修复的。正在研究自动修复的方法。  
  
IAT 修复程序使用方法：  
先用 OD,跑到 OEP 或者 OEP 附近。然后打开程序，填写要求的数据，点修复按钮就可以了。  
源代码有点乱，下个版本改进。  
  
已知的壳自己实现的 API:  
GetModuleHandleA  
ExitProcess  
GetModuleFileNameA  
  
BTW:ETG 和 CMIX 是什么东西啊？ 

 4dd466d0e1e0cc123afbe504da2462ca