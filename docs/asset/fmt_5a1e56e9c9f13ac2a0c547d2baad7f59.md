
### 汇编代码片段（加密逻辑）

```asm
00F71548    C1F8 02         sar     eax, 2                 ; 全位右移 2 位  
00F7154B    0AC8            or      cl, al                 ; 跟 cl 做或运算  
00F7154D    8B55 FC         mov     edx, dword ptr [ebp-4]  
00F71550    8B45 08         mov     eax, dword ptr [ebp+8]  
00F71553    0FB61410        movzx   edx, byte ptr [eax+edx]  
00F71557    83E2 40         and     edx, 40                ; 保留倒数第七位  
00F7155A    C1FA 04         sar     edx, 4                 ; 全位右移 4 位  
00F7155D    0ACA            or      cl, dl                 ; 跟 cl 做或运算  
00F7155F    8B45 FC         mov     eax, dword ptr [ebp-4]  
00F71562    8B55 08         mov     edx, dword ptr [ebp+8]  
00F71565    0FB60402        movzx   eax, byte ptr [edx+eax]  
00F71569    25 80000000     and     eax, 80                ; 保留倒数第八位  
00F7156E    C1F8 06         sar     eax, 6                 ; 全位右移 6 位  
00F71571    0AC8            or      cl, al                 ; 跟 cl 做或运算  
00F71573    884D FB         mov     byte ptr [ebp-5], cl  
00F71576    8A55 FB         mov     dl, byte ptr [ebp-5]  
00F71579    8B4D FC         mov     ecx, dword ptr [ebp-4]  
00F7157C    8B45 08         mov     eax, dword ptr [ebp+8]  
00F7157F    881408          mov     byte ptr [eax+ecx], dl  
00F71582    FF45 FC         inc     dword ptr [ebp-4]     ; 加一  
00F71585    8B55 FC         mov     edx, dword ptr [ebp-4]  
00F71588    3B55 0C         cmp     edx, dword ptr [ebp+C]  
00F7158B    ^ 0F8C 58FFFFFF jl      00F714E9               ; 判断是否加密完成（也就是报文中的第 2 字节的数据判断）  
00F71591    B8 01000000     mov     eax, 1  
00F71596    59              pop     ecx  
00F71597    59              pop     ecx  
00F71598    5D              pop     ebp  
00F71599    C3              retn  
```

### 加密后的报文数据

加密后的报文，已经看不懂啦！！

```
0129FBB8  80 62 80 89 95 EE F7 F6 4C E6 24 1E B3 4C F3 31  b 铟??  
0129FBC8  29 EF 80 20 CD EC 49 48 E8 49 01 20 68 68 68 68  ) 挽 IH hhhh  
0129FBD8  68 68 91 20 00 30 A8 66 89 04 A0 31 C8 D9 49 39  hh?.0??荣 I9  
0129FBE8  C8 59 39 D9 39 C8 C8 C8 21 21 AC 3D 5C 9C 4D 3D  9?热?!?\=  
0129FBF8  9C 5C F1 B1 C9 39 D8 39 48 58 39 48 C8 C8 68 00  癖??HX9H 热 h.  
0129FC08  00 98 95 00                             ...  
```

开始发包，我们来截取一个包看看数据（我用的是科来 6.7 交流），下图高亮为数据域：

![](img/3235d28b1f4a05d11abb21eb327c7621.jpg)

### 服务器返回数据

![](img/a9d2f6119f79aedc2a5700d2efed955a.jpg)

#### 堆栈中搜索返回数据

```
0129FBB8  01 9C 1D 12 B9 38 CB 63 B4 4F 9A 50 A3 06 0A E2  ???.  
0129FBC8  22 57 81 81 80 20 40 49 D8 E8 E8 48 D9 58 C8 E8  "W @I 罔辱  
0129FBD8  C8 59 68 68 D8 49 E8 90 81 78 11 81 10 98 50 80  hh?  
0129FBE8  00 00 01 00 00 81 00 00 10 00 00 90 00 00 00 00  ....?...?...  
0129FBF8  00 58 11 00 00 00 00 D8 11 00 00 00 00 59 11 00  .X....?....Y.  
0129FC08  00 00 00 D9 11 00 00 00 00 68 11 00 00 00 00 89  ...?....h....  
0129FC18  81 80 08 81 00                       ?..  
```

### 解密操作

解密他！！用 `OD` 在此数据区上下内存访问中断，`F9` 放飞程序。段在这个函数中：

```asm
00405A0B   .  FF15 389E4200 call    dword ptr [429E38]           ;  接收数据包数据解密  
```

#### 解密函数逻辑

```asm
00F7159C  >>push    ebp  
00F7159D   >mov     ebp, esp  
00F7159F   >add     esp, -8  
00F715A2   >cmp     dword ptr [ebp+C], 0  
00F715A6   >jg      short 00F715AF  
00F715A8   >xor     eax, eax  
00F715AA   >jmp     00F71679  
00F715AF   >cmp     dword ptr [ebp+8], 0  
00F715B3   >jnz     short 00F715BC  
00F715B5   >xor     eax, eax  
00F715B7   >jmp     00F71679  
00F715BC   >xor     edx, edx  
00F715BE   >mov     dword ptr [ebp-4], edx  
00F715C1   >mov     ecx, dword ptr [ebp-4]  
00F715C4   >cmp     ecx, dword ptr [ebp+C]  
00F715C7   >jge     00F71674  
00F715CD   >mov     eax, dword ptr [ebp-4]  
00F715D0   >mov     edx, dword ptr [ebp+8]  
00F715D3   >mov     cl, byte ptr [edx+eax]        ; 取加密字符串  
00F715D6   >and     cl, 1                       ; 保留最后一位  
00F715D9   >add     ecx, ecx                    ; 自加  
00F715DB   >mov     eax, dword ptr [ebp-4]  
00F715DE   >mov     edx, dword ptr [ebp+8]  
00F715E1   >mov     al, byte ptr [edx+eax]        ; 取加密字符串  
00F715E4   >and     al, 2                       ; 保留倒数第二位  
00F715E6   >shl     eax, 6                     ; 左移 6 位  
00F715E9   >or      cl, al                     ; 再或运算,以下同理  
00F715EB   >mov     edx, dword ptr [ebp-4]  
00F715EE   >mov     eax, dword ptr [ebp+8]  
00F715F1   >mov     dl, byte ptr [eax+edx]  
00F715F4   >and     dl, 4  
00F715F7   >shl     edx, 4  
00F715FA   >or      cl, dl  
00F715FC   >mov     eax, dword ptr [ebp-4]  
00F715FF   >mov     edx, dword ptr [ebp+8]  
00F71602   >mov     al, byte ptr [edx+eax]  
00F71605   >and     al, 8  
00F71607   >shl     eax, 2  
00F7160A   >or      cl, al  
00F7160C   >mov     edx, dword ptr [ebp-4]  
00F7160F   >mov     eax, dword ptr [ebp+8]  
00F71612   >movzx   edx, byte ptr [eax+edx]  
00F71616   >and     edx, 10  
00F71619   >sar     edx, 2  
00F7161C   >or      cl, dl  
00F7161E   >mov     eax, dword ptr [ebp-4]  
00F71621   >mov     edx, dword ptr [ebp+8]  
00F71624   >movzx   eax, byte ptr [edx+eax]  
00F71628   >and     eax, 20  
00F7162B   >sar     eax, 2  
00F7162E   >or      cl, al  
00F71630   >mov     edx, dword ptr [ebp-4]  
00F71633   >mov     eax, dword ptr [ebp+8]  
00F71636   >movzx   edx, byte ptr [eax+edx]  
00F7163A   >and     edx, 40  
00F7163D   >sar     edx, 2  
00F71640   >or      cl, dl  
00F71642   >mov     eax, dword ptr [ebp-4]  
00F71645   >mov     edx, dword ptr [ebp+8]  
00F71648   >movzx   eax, byte ptr [edx+eax]  
00F7164C   >and     eax, 80  
00F71651   >sar     eax, 7  
00F71654   >or      cl, al  
00F71656   >mov     byte ptr [ebp-5], cl  
00F71659   >mov     dl, byte ptr [ebp-5]  
00F7165C   >mov     ecx, dword ptr [ebp-4]  
00F7165F   >mov     eax, dword ptr [ebp+8]  
00F71662   >mov     byte ptr [eax+ecx], dl      ; 写入解密数据  
00F71665   >inc     dword ptr [ebp-4]  
00F71668   >mov     edx, dword ptr [ebp-4]  
00F7166B   >cmp     edx, dword ptr [ebp+C]  
00F7166E  ^>jl      00F715CD            ; 判断是否解密完成  
00F71674   >mov     eax, 1  
00F71679   >pop     ecx  
00F7167A   >pop     ecx  
00F7167B   >pop     ebp  
00F7167C   >retn  
```

### 解密后的数据

解密完成的数据如下（注意红色标记，那个为密钥，每次都不一样）：

```
0129FDB8  02 65 66 84 2F 2C B3 9A 4D F2 A5 14 8B C0 A0 99  ef?,M 颔  
0129FDC8  88 D6 03 03 01 08 10 32 35 39 39 30 37 34 31 39  259907419  
0129FDD8  31 36 38 38 35 32 39 05 03 3C 06 03 04 25 14 01  1688529<%  
0129FDE8  00 00 02 00 00 03 00 00 04 00 00 05 00 00 00 00  ............  
0129FDF8  00 34 06 00 00 00 00 35 06 00 00 00 00 36 06 00  .4....5....6.  
0129FE08  00 00 00 37 06 00 00 00 00 38 06 00 00 00 00 23  ...7....8....#  
0129FE18  03 01 20 03 00                              .  
```

#### 密钥分析

红色标记前 1 位 `0x10` 位密钥长度 16 字节，红色标记的为密钥：

```
0129FDC8                 32 35 39 39 30 37 34 31 39       59907419  
0129FDD8  31 36 38 38 35 32                              168852  
```

### 第二个包的构造分析

继续分析第二个包是怎么构造的！慢慢跟，来到下面这个函数：

```asm
00404E3C   >call    00405DD0      ；第二个包的构造函数 F7  
```

#### 构造函数 `00405DD0` 细节

```asm
00405DD0  />sub     esp, 34  
00405DD3  |>push    ebx  
00405DD4  |>push    ebp  
00405DD5  |>mov     ebp, dword ptr [esp+40]  
00405DD9  |>push    esi  
00405DDA  |>push    edi  
00405DDB  |>mov     edi, 0042CC10               ;  9488171348284164  
00405DE0  |>mov     byte ptr [ebp], 3           ;  写 03  
00405DE4  |>inc     ebp  
00405DE5  |>mov     dword ptr [esp+1C], ebp  
00405DE9  |>add     ebp, 11  
00405DEC  |>mov     esi, 0042CC10               ;  9488171348284164  
00405DF1  |>mov     byte ptr [ebp], 7           ;  跳过 17 字节写 07  
00405DF5  |>inc     ebp  
00405DF6  |>mov     byte ptr [ebp], 8           ;  写 08  
00405DFA  |>mov     eax, dword ptr [453FA8]  
00405DFF  |>inc     ebp  
00405E00  |>add     eax, 10  
00405E03  |>mov     ecx, ebp  
```
