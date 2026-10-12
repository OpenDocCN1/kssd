
# 某取名软件的注册算法分析

> 原文：[`www.pediy.com/kssd/pediy12/129607.html`](https://www.pediy.com/kssd/pediy12/129607.html)

【文章标题】: 宝宝取名软件 V10 的注册算法分析

【文章作者】: `cyane`

【下载地址】: 自己搜索下载

【加壳方式】: 无壳

【编写语言】: Microsoft Visual C++ 6.0

【使用工具】: `OD`

【破解平台】: `Win9x`/`NT`/`2000`/`XP`

【作者声明】: 只是感兴趣，没有其他目的。初次对算法进行学习，失误之处敬请诸位大侠赐教!

【详细过程】

`OD` 载入程序，`F9` 运行，注册码：`1234567890`，点击注册，提示注册码无效，下断 `MessageBoxA` 函数，可回溯到验证函数的入口。

```
00405F21  /.  55            push    ebp
00405F22  |.  8BEC          mov     ebp, esp
00405F24  |.  81EC 0C000000 sub     esp, 0C
00405F2A  |.  6A FF         push    -1
00405F2C  |.  6A 08         push    8
00405F2E  |.  68 740C0116   push    16010C74
00405F33  |.  68 6E0C0152   push    52010C6E
00405F38  |.  E8 16C70100   call    00422653                         ;  获取注册码

.....

00405F71  |>  837D F8 00    cmp     dword ptr [ebp-8], 0
00405F75  |.  0F84 8E000000 je      00406009                         ;  注册码不能为空
00405F7B  |.  C745 FC 0000>mov      dword ptr [ebp-4], 0
00405F82  |.  6A 00         push    0
00405F84  |.  8D45 FC       lea     eax, dword ptr [ebp-4]
00405F87  |.  50            push    eax
00405F88  |.  B8 35FC5200   mov     eax, 0052FC35                     ;  ASCII "Z2dRqSh2JZr2tRE3LZb1R3lRC1W36B"
.....                          ;  字符串解码[注册码不能为空]

0040602C  |.  6A 00         push    0
0040602E  |.  8D45 FC       lea     eax, dword ptr [ebp-4]
00406031  |.  50            push    eax
00406032  |.  B8 54FC5200   mov     eax, 0052FC54                     ;  ASCII "10cQxVf0RZdUrR53jBK1q3GZjTQ3uR31b3UZ81A3SZuTDbcRcT"
.....                          ;  字符串解码[注册失败，请联系客服]

004060CD  |.  83C4 18       add     esp, 18
004060D0  |.  E8 04000000   call    004060D9             ;  判断注册函数，F7 跟进
004060D5  |>  8BE5          mov     esp, ebp
004060D7  |.  5D            pop     ebp
004060D8  \.  C3            retn
//////////////////////////////////////////////////////////////////////

004060D9   $  55            push    ebp
004060DA   .  8BEC          mov     ebp, esp
004060DC   .  81EC 28000000 sub     esp, 28
004060E2   .  C745 FC 0000>mov      dword ptr [ebp-4], 0
004060E9   .  C745 F8 0000>mov      dword ptr [ebp-8], 0
004060F0   .  C745 F4 0000>mov      dword ptr [ebp-C], 0
004060F7   .  C745 F0 0000>mov      dword ptr [ebp-10], 0
004060FE   .  6A FF         push    -1
00406100   .  6A 08         push    8
00406102   .  68 730C0116   push    16010C73
00406107   .  68 6E0C0152   push    52010C6E
0040610C   .  E8 42C50100   call    00422653                         ;  获取机器码
00406111   .  83C4 10       add     esp, 10
00406114   .  8945 EC       mov     dword ptr [ebp-14], eax
00406117   .  6A FF         push    -1
00406119   .  6A 08         push    8
0040611B   .  68 740C0116   push    16010C74
00406120   .  68 6E0C0152   push    52010C6E
00406125   .  E8 29C50100   call    00422653                         ;  获取注册码
0040612A   .  83C4 10       add     esp, 10
0040612D   .  8945 E8       mov     dword ptr [ebp-18], eax
00406130   .  68 87FC5200   push    0052FC87                           ;  ASCII "&caozuo=1"
00406135   .  FF75 E8       push    dword ptr [ebp-18]
00406138   .  68 91FC5200   push    0052FC91                           ;  ASCII "&code="
0040613D   .  FF75 EC       push    dword ptr [ebp-14]
00406140   .  68 98FC5200   push    0052FC98                           ;  ASCII "?name="
00406145   .  FF35 D02F6C00 push    dword ptr [6C2FD0]
.....

0040619E   .  E8 AAC40100   call    0042264D                         ;  网络验证
//验证地址 http://xxx.com/more_txt/reg.php?name=[机器码]&code=1234567890&caozuo=1
//返回数据：text[对不起,您输入的注册码无效!]int[48]
.....
004062CA   .  68 17E94D00   push    004DE917
004062CF   .  FF75 F4       push    dword ptr [ebp-C]
004062D2   .  E8 DCAFFFFF   call    004012B3                         ;  判断数据
004062D7   .  83C4 08       add     esp, 8
004062DA   .  83F8 00       cmp     eax, 0
004062DD   .  0F84 20000000 je      00406303             ;  必须跳转
.....
0040637F   .  0F84 1A020000 je      0040659F             ;  不能跳转
//上面 2 个跳转是根据网络验证返回的数据进行判断，所以这里必须强制操作
.....
004063A6   .  50            push    eax
004063A7   .  B8 ADFC5200   mov     eax, 0052FCAD                      ;  ASCII "7LbJ8JfNyLkApLdIbM"
004063AC   .  8945 E4       mov     dword ptr [ebp-1C], eax         ;  字符串解码[\baby.reg]
.....
00406494   >  E8 A4040000   call    0040693D             ;  F7 跟进
//////////////////////////////////////////////////////////////////////

0040693D  /$  55            push    ebp
0040693E  |.  8BEC          mov     ebp, esp
00406940  |.  81EC 10000000 sub     esp, 10
00406946  |.  E8 05AAFFFF   call    00401350             ;  F7 跟进
0040694B  |.  8945 F8       mov     dword ptr [ebp-8], eax
//////////////////////////////////////////////////////////////////////

00401476   .  E8 08060000   call    00401A83                         ;  计算注册码函数，F7 跟进
//////////////////////////////////////////////////////////////////////

00401A83  /$  55            push    ebp
00401A84  |.  8BEC          mov     ebp, esp
00401A86  |.  81EC 38000000 sub     esp, 38
00401A8C  |.  C745 FC 0000>mov      dword ptr [ebp-4], 0
00401A93  |.  C745 F8 0000>mov      dword ptr [ebp-8], 0
00401A9A  |.  E8 12030000   call    00401DB1                         ;  获取硬盘型号函数
00401A9F  |.  8945 F4       mov     dword ptr [ebp-C], eax
00401AA2  |.  E8 DE0B0000   call    00402685
00401AA7  |.  8945 F0       mov     dword ptr [ebp-10], eax
00401AAA  |.  FF75 F0       push    dword ptr [ebp-10]
00401AAD  |.  FF75 F4       push    dword ptr [ebp-C]
00401AB0  |.  B9 02000000   mov     ecx, 2
00401AB5  |.  E8 9DF7FFFF   call    00401257                         ;  temp1=硬盘[修订版本后 2 位+硬盘型号+序列号后 4 位]
.....
00401B23  |.  E8 1F0B0200   call    00422647             ;  temp2=获取磁盘信息并进行部分运算，省略
.....                        ;  有兴趣的可以自己跟
00401B55  |.  B8 9DE94D00   mov     eax, 004DE99D                      ;  ASCII "RKuLoLsI6AoLsJ8LeNcJeL9N"
00401B5A  |.  8945 D8       mov     dword ptr [ebp-28], eax           ;  temp3=字符串解码[Must succeed]
00401B5D  |.  8D45 D8       lea     eax, dword ptr [ebp-28]
.....                        ;  temp=temp1+temp2+temp3
00401C2E  |.  E8 1A0A0200   call    0042264D                         ;  MD5(temp)函数，跟进
00401C33  |.  83C4 10       add     esp, 10             ;  我的 temp 为 K0Maxtor4D040H2LZPE1347721313Must succeed
.....
004A73D6   .  E8 95000000   call    004A7470                         ;  MD5(temp),跟进
//////////////////////////////////////////////////////////////////////

004A7470  /$  6A FF         push    -1
004A7472  |.  68 388E4D00   push    004D8E38                         ;  SE 处理程序安装
004A7477  |.  64:A1 000000>mov      eax, dword ptr fs:[0]
004A747D  |.  50            push    eax
004A747E  |.  64:8925 0000>mov      dword ptr fs:[0], esp
004A7485  |.  83EC 5C       sub     esp, 5C
004A7488  |.  8D4C24 00     lea     ecx, dword ptr [esp]
004A748C  |.  E8 4F000000   call    004A74E0                         ;  MD5Init
004A7491  |.  8B4424 70     mov     eax, dword ptr [esp+70]
004A7495  |.  8B4C24 6C     mov     ecx, dword ptr [esp+6C]
004A7499  |.  50            push    eax
004A749A  |.  51            push    ecx
004A749B  |.  8D4C24 08     lea     ecx, dword ptr [esp+8]
004A749F  |.  C74424 6C 000>mov     dword ptr [esp+6C], 0
004A74A7  |.  E8 64010000   call    004A7610                         ;  MD5Update
004A74AC  |.  8B5424 74     mov     edx, dword ptr [esp+74]
004A74B0  |.  8D4C24 00     lea     ecx, dword ptr [esp]
004A74B4  |.  52            push    edx
004A74B5  |.  E8 66000000   call    004A7520                         ;  MD5Final
```



```
004A74BA  |.  8B4C24 5C      mov     ecx, dword ptr [esp+5C]  
004A74BE  |.  64:890D 00000>mov     dword ptr fs:[0], ecx  
004A74C5  |.  83C4 68       add     esp, 68  
004A74C8  \\.  C3          retn  
```

//通过 `MD5Init` 函数可以看到 A/B/C/D 为标准的 MD5 数据。

//MD5 算法部分说明可以在论坛中搜索到，所以此处省略了。

//这里是对得到的硬件码部分进行 MD5，得到的 32 位数据就是软件的机器码了。

//我的机器得到的数据为 `f474d14e99a8ca601022afbe4d4ddafb`

//然后到了下面得函数

```
00401C68  |> /41         /inc     ecx  
00401C69  |. |51         |push    ecx  
00401C6A  |. |53         |push    ebx  
00401C6B  |. |890B       |mov     dword ptr [ebx], ecx  
00401C6D  |. |83F9 08    |cmp     ecx, 8  
00401C70  |. |0F8F 01010000 |jg    00401D77  
00401C76  |. |68 01030080 |push   80000301  
00401C7B  |. |6A 00      |push    0  
00401C7D  |. |FF75 F8    |push    dword ptr [ebp-8]  
00401C80  |. |68 01000000 |push   1  
00401C85  |. |BB 70374200 |mov    ebx, 00423770  
00401C8A  |. |E8 B8090200 |call   00422647  
00401C8F  |. |83C4 10    |add     esp, 10  
00401C92  |. |8945 F4    |mov     dword ptr [ebp-C], eax  
00401C95  |. |FF75 F4    |push    dword ptr [ebp-C]  
00401C98  |. |FF75 FC    |push    dword ptr [ebp-4]  
00401C9B  |. |B9 02000000 |mov    ecx, 2  
00401CA0  |. |E8 B2F5FFFF |call   00401257  
00401CA5  |. |83C4 08    |add     esp, 8  
00401CA8  |. |8945 F0    |mov     dword ptr [ebp-10], eax  
00401CAB  |. |8B5D F4    |mov     ebx, dword ptr [ebp-C]  
00401CAE  |. |85DB       |test    ebx, ebx  
00401CB0  |. |74 09      |je      short 00401CBB  
00401CB2  |. |53         |push    ebx  
00401CB3  |. |E8 83090200 |call   0042263B  
00401CB8  |. |83C4 04    |add     esp, 4  
00401CBB  |> |68 04000080 |push   80000004  
00401CC0  |. |6A 00      |push    0  
00401CC2  |. |8B45 F0    |mov     eax, dword ptr [ebp-10]  
00401CC5  |. |85C0       |test    eax, eax  
00401CC7  |. |75 05      |jnz     short 00401CCE  
00401CC9  |. |B8 17E94D00 |mov    eax, 004DE917  
00401CCE  |> |50         |push    eax  
00401CCF  |. |68 01000000 |push   1  
00401CD4  |. |BB C0354200 |mov    ebx, 004235C0  
00401CD9  |. |E8 69090200 |call   00422647  
00401CDE  |. |83C4 10    |add     esp, 10  
00401CE1  |. |8945 EC    |mov     dword ptr [ebp-14], eax  
00401CE4  |. |8B5D F0    |mov     ebx, dword ptr [ebp-10]  
00401CE7  |. |85DB       |test    ebx, ebx  
00401CE9  |. |74 09      |je      short 00401CF4  
00401CEB  |. |53         |push    ebx  
00401CEC  |. |E8 4A090200 |call   0042263B  
00401CF1  |. |83C4 04    |add     esp, 4  
00401CF4  |> |68 05000080 |push   80000005  
00401CF9  |. |6A 00      |push    0  
00401CFB  |. |8B45 EC    |mov     eax, dword ptr [ebp-14]  
00401CFE  |. |85C0       |test    eax, eax  
00401D00  |. |75 05      |jnz     short 00401D07  
00401D02  |. |B8 46E94D00 |mov    eax, 004DE946  
00401D07  |> |50         |push    eax  
00401D08  |. |68 01000000 |push   1  
00401D0D  |. |B8 09000000 |mov    eax, 9  
00401D12  |. |BB C0734A00 |mov    ebx, 004A73C0  
00401D17  |. |E8 31090200 |call   0042264D ; 分组计算注册码  
00401D1C  |. |83C4 10    |add     esp, 10  
00401D1F  |. |8945 E8    |mov     dword ptr [ebp-18], eax  
00401D22  |. |8B5D EC    |mov     ebx, dword ptr [ebp-14]  
00401D25  |. |85DB       |test    ebx, ebx  
00401D27  |. |74 09      |je      short 00401D32  
00401D29  |. |53         |push    ebx  
00401D2A  |. |E8 0C090200 |call   0042263B  
00401D2F  |. |83C4 04    |add     esp, 4  
00401D32  |> |FF75 E8    |push    dword ptr [ebp-18]  
00401D35  |. |FF75 FC    |push    dword ptr [ebp-4]  
00401D38  |. |B9 02000000 |mov    ecx, 2  
00401D3D  |. |E8 15F5FFFF |call   00401257  
00401D42  |. |83C4 08    |add     esp, 8  
00401D45  |. |8945 E4    |mov     dword ptr [ebp-1C], eax  
00401D48  |. |8B5D E8    |mov     ebx, dword ptr [ebp-18]  
00401D4B  |. |85DB       |test    ebx, ebx  
00401D4D  |. |74 09      |je      short 00401D58  
00401D4F  |. |53         |push    ebx  
00401D50  |. |E8 E6080200 |call   0042263B  
00401D55  |. |83C4 04    |add     esp, 4  
00401D58  |> |8B45 E4    |mov     eax, dword ptr [ebp-1C]  
00401D5B  |. |50         |push    eax  
00401D5C  |. |8B5D FC    |mov     ebx, dword ptr [ebp-4]  
00401D5F  |. |85DB       |test    ebx, ebx  
00401D61  |. |74 09      |je      short 00401D6C  
00401D63  |. |53         |push    ebx  
00401D64  |. |E8 D2080200 |call   0042263B  
00401D69  |. |83C4 04    |add     esp, 4  
00401D6C  |> |58         |pop     eax  
00401D6D  |. |8945 FC    |mov     dword ptr [ebp-4], eax  
00401D70  |. |5B         |pop     ebx  
00401D71  |. |59         |pop     ecx  
00401D72  |.^\E9 F1FEFFFF \\jmp 00401C68  
```

//上面函数段可理解伪 C

```
serial=temp=temp1+temp2+temp3       
for(i=1;i<=8;i++)  
{  
  temp=md5(serial)+i;  
  strcat(serial, temp);  
}
```

//这里可以知道注册码是 288 位。

.....

```
00401481   .  50          push    eax          ;  真注册码  
00401482   .  FF75 FC     push    dword ptr [ebp-4]        ;  假注册码  
00401485   .  E8 29FEFFFF call    004012B3     
```

//这里如果验证成功，会在软件目录下建立 `baby.reg` 的注册文件，并向 `server.ini` 文件写入注册码，以下分析就省略了。

//`baby.reg` 内的注册码就是我们得到的 288 位注册码了。通过输入注册码进行注册，首先要通过网络验证，这里有些麻烦。

//软件一开始运行的时候就会扫描目录下的 `baby.reg` 文件，并且明码比较，所以是很容易就找到注册码的。

//这里给出的伪 C 代码是可直接算出注册码的。
