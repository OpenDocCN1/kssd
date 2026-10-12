
### 00485F5E - 00485F7B

```
00485F5E   .  8955 DC       MOV DWORD PTR SS:[EBP-24],EDX  

00485F61   .  8B45 D8       MOV EAX,DWORD PTR SS:[EBP-28]            ;  eax=前面算出来的第二个解密算子  

00485F64   .  8B4D D4       MOV ECX,DWORD PTR SS:[EBP-2C]            ;  ecx=常数，用于解密  

00485F67   .  8B5D E0       MOV EBX,DWORD PTR SS:[EBP-20]            ;  ebx=缓冲区大小  

00485F6A   .  8B75 DC       MOV ESI,DWORD PTR SS:[EBP-24]            ;  esi=缓冲区地址  

00485F6D   .  C1EB 02       SHR EBX,2                                 ;  这里到下面的跳转都是对缓冲区的解密换算  

00485F70   >  C1C1 04       ROL ECX,4  

00485F73   .  3106          XOR DWORD PTR DS:[ESI],EAX  

00485F75   .  D3C8          ROR EAX,CL  

00485F77   .  83C6 04       ADD ESI,4  

00485F7A   .  4B            DEC EBX  

00485F7B   .^ 75 F3        JNZ SHORT ORE\_TUBA.00485F70        ;跳回去继续循环  
```

### 00485F7D - 00485F80

```
00485F7D   .  8A45 EB       MOV AL,BYTE PTR SS:[EBP-15]  

00485F80   .  8B12          MOV EDX,DWORD PTR DS:[EDX]               ;  [EDX]为解密后的第一个双字的内容，为 0x1a2，会是啥呢，我猜测是脚本文件的个数  
```

可以看到`[EBP-34]`中存储的就是从文件中读出的第二个双字，将它与上面的算法算出来的 `b` 相异或就得到 `0x3f03`，然后可以看出分配了 `0x3f03` 个字节的内存空间，再从文件中读出 `0x3f03` 个字节的内容。然后对读出的内容进行解密，写成 C 代码如下：

代码:

```
int data[]//缓冲区地址
int n=0x31746285;
int key=0xa742f274;
for(i=0;i<0x3f03/4;i++)
{
  n=rol(n,4);
  data[i]^=key;
  key=ror(key,*(char*)&n);
}
```

### 0048621F - 0048624A

后面还有一段读取缓冲区的操作：

```
0048621F   .  8B3E          MOV EDI,DWORD PTR DS:[ESI]               ;  读取缓冲区的数据，这个是什么，下面就知道  

00486221   .  83C6 04       ADD ESI,4  

00486224   .  84C0          TEST AL,AL  

00486226   .  8973 20       MOV DWORD PTR DS:[EBX+20],ESI  

00486229   .  74 1A         JE SHORT ORE\_TUBA.00486245  

0048622B   .  8D4D EC       LEA ECX,DWORD PTR SS:[EBP-14]  

0048622E   .  8D95 D0FEFFFF LEA EDX,DWORD PTR SS:[EBP-130]  

00486234   .  51            PUSH ECX  

00486235   .  52            PUSH EDX  

00486236   .  56            PUSH ESI  

00486237   .  8BCB          MOV ECX,EBX  

00486239   .  C745 EC FFFFF>MOV DWORD PTR SS:[EBP-14],-1  

00486240   .  E8 BB020000   CALL ORE\_TUBA.00486500  

00486245   >  03F7          ADD ESI,EDI                               ;  将前面读出的双子与缓冲区偏移地址相加，再保存起来  

00486247   >  8A45 10       MOV AL,BYTE PTR SS:[EBP+10]  

0048624A   .  8973 2C       MOV DWORD PTR DS:[EBX+2C],ESI  
```

再看看数据窗口中缓冲区的数据：

```
00AE4008  A2 01 00 00 00 00 BF 29 00 00                    ?....?..  
```

`a`

可以看出第一个双字是疑似脚本文件个数，中间两个字节作用未知，然后一个双字是一个偏移，用结构可以表达如下：

代码:

```
struct header{
  DWORD numberoffile;
  BYTE unknow[2];
  DWORD offset;
……}；
```

### 00472ED0 - 00472F61

这个函数剩下的地方都没什么猛料，回到原来的函数后继续跟发现都是一些字符串的处理，一直跟到下面地址为 `0x004750eb` 的一个 `call`，看到压栈的参数有一个为字符串`"SCRIPT/LL\_APP.SYS"`，跟进去看看：

```
00472ED0   .  55            PUSH EBP  

00472ED1   .  8BEC          MOV EBP,ESP  

00472ED3   .  6A FF         PUSH -1  

00472ED5   .  68 C0AB4E00   PUSH ORE\_TUBA.004EABC0                    ;  SE 处理程序安装  

00472EDA   .  64:A1 0000000>MOV EAX,DWORD PTR FS:[0]  

00472EE0   .  50            PUSH EAX  

00472EE1   .  64:8925 00000>MOV DWORD PTR FS:[0],ESP  

00472EE8   .  81EC 44020000 SUB ESP,244  

00472EEE   .  8B45 08       MOV EAX,DWORD PTR SS:[EBP+8]  

00472EF1   .  53            PUSH EBX  

00472EF2   .  56            PUSH ESI  

00472EF3   .  57            PUSH EDI  

00472EF4   .  8BD9          MOV EBX,ECX  

00472EF6   .  8965 F0       MOV DWORD PTR SS:[EBP-10],ESP  

00472EF9   .  6A 00         PUSH 0  

00472EFB   .  8D8D B4FEFFFF LEA ECX,DWORD PTR SS:[EBP-14C]  

00472F01   .  50            PUSH EAX  

00472F02   .  51            PUSH ECX  

00472F03   .  895D E4       MOV DWORD PTR SS:[EBP-1C],EBX  

00472F06   .  C745 E8 FFFFF>MOV DWORD PTR SS:[EBP-18],-1  

00472F0D   .  C745 FC 00000>MOV DWORD PTR SS:[EBP-4],0  

00472F14   .  E8 A71A0000   CALL ORE\_TUBA.004749C0                    ;  处理字符串  

00472F19   .  83C4 0C       ADD ESP,0C  

00472F1C   .  8945 EC       MOV DWORD PTR SS:[EBP-14],EAX  

00472F1F   .  85C0          TEST EAX,EAX  

00472F21   .  75 15         JNZ SHORT ORE\_TUBA.00472F38  

00472F23   .  8D55 CC       LEA EDX,DWORD PTR SS:[EBP-34]  

00472F26   .  68 68604F00   PUSH ORE\_TUBA.004F6068                    ;  /Arg2 = 004F6068  

00472F2B   .  52            PUSH EDX                                    ;  |Arg1  

00472F2C   .  C745 CC 05000>MOV DWORD PTR SS:[EBP-34],5                ;  |  

00472F33   .  E8 5A8D0600   CALL ORE\_TUBA.004DBC92                    ;  \\ORE\_TUBA.004DBC92  

00472F38   >  8B35 B0D14E00 MOV ESI,DWORD PTR DS:[<&KERNEL32.CreateF>;  kernel32.CreateFileA  

00472F3E   .  6A 00         PUSH 0                                      ;  /hTemplateFile = NULL  

00472F40   .  68 80000000   PUSH 80                                     ;  |Attributes = NORMAL  

00472F45   .  6A 03         PUSH 3                                      ;  |Mode = OPEN\_EXISTING  

00472F47   .  6A 00         PUSH 0                                      ;  |pSecurity = NULL  

00472F49   .  6A 01         PUSH 1                                      ;  |ShareMode = FILE\_SHARE\_READ  

00472F4B   .  8D85 B4FEFFFF LEA EAX,DWORD PTR SS:[EBP-14C]             ;  |  

00472F51   .  68 00000080   PUSH 80000000                               ;  |Access = GENERIC\_READ  

00472F56   .  50            PUSH EAX                                    ;  |FileName="I:\\Program Files\\Navel\\\\SCRIPT\\LL\_APP.SYS"  

00472F57   .  FFD6          CALL ESI                                    ;  \\CreateFileA  

00472F59   .  8BF8          MOV EDI,EAX  

00472F5B   .  83FF FF       CMP EDI,-1  

00472F5E   .  897D E8       MOV DWORD PTR SS:[EBP-18],EDI  

00472F61   .  74 7E         JE SHORT ORE\_TUBA.00472FE1        ;跳转  
```

这个打开一个不存在的文件，但看文件名应该猜到，这个就是解密后的脚本文件了。但由于现在文件还不存在，所以会跳到后面的代码继续执行，一开始还是一些字符串处理的流程，然后到了 `0x0047310d` 的一个 `call`，跟进去看看，一开始是一个嵌套的循环，处理字符串的。循环后出来到了下面一个 `call`。

### 00486840 - 0048687D

```
00486840   .  81EC 04010000 SUB ESP,104                               ;  抬高堆栈  

00486846   .  56            PUSH ESI  

00486847   .  8BF1          MOV ESI,ECX  

00486849   .  8A46 0D       MOV AL,BYTE PTR DS:[ESI+D]  

0048684C   .  84C0          TEST AL,AL  

0048684E   .  74 6C         JE SHORT ORE\_TUBA.004868BC  

00486850   .  8B8424 0C0100>MOV EAX,DWORD PTR SS:[ESP+10C]  

00486857   .  8D4C24 04     LEA ECX,DWORD PTR SS:[ESP+4]  

0048685B   .  50            PUSH EAX                                    ;  /String2  

0048685C   .  51            PUSH ECX                                    ;  |String1  

0048685D   .  FF15 00D14E00 CALL DWORD PTR DS:[<&KERNEL32.lstrcpyA>] ;  \\lstrcpyA  

00486863   .  8D5424 04     LEA EDX,DWORD PTR SS:[ESP+4]  

00486867   .  52            PUSH EDX                                    ;  /StringOrChar  

00486868   .  FF15 54D3E00 CALL DWORD PTR DS:[<&USER32.CharLowerA>] ;  \\CharLowerA  

0048686E   .  8A4E 24       MOV CL,BYTE PTR DS:[ESI+24]  

00486871   .  8B56 20       MOV EDX,DWORD PTR DS:[ESI+20]             ;  [ESI+20]中的数据是 0x00AE4012，而前面从文件中读取 0x3f03 字节的内容的缓冲区是 0x00AE4008  

00486874   .  8D4424 04     LEA EAX,DWORD PTR SS:[ESP+4]              ;  取“ll\_app.sys”地址  

00486878   .  6A 00         PUSH 0  

0048687A   .  50            PUSH EAX  

0048687B   .  51            PUSH ECX  

0048687C   .  52            PUSH EDX  

0048687D   .  E8 DEC30400   CALL ORE\_TUBA.004D2C60                   ;  又是一个处理字符串的 call  
```

### 00486898 - 004868AB

然后一直单步到下面这个 `call`:

```
00486898   .  51            PUSH ECX                                    ;  /Arg5  

00486899   .  8B8C24 180100>MOV ECX,DWORD PTR SS:[ESP+118]             ;  |  

004868A0   .  52            PUSH EDX                                    ;  |Arg4  

004868A1   .  8B9424 180100>MOV EDX,DWORD PTR SS:[ESP+118]             ;  |  

004868A8   .  51            PUSH ECX                                    ;  |Arg3  

004868A9   .  52            PUSH EDX                                    ;  |Arg2  

004868AA   .  50            PUSH EAX                                    ;  |Arg1  

004868AB   .  8BCE          MOV ECX,ESI                                 ;  |  
```
