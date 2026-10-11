
```assembly
11238A4C  |.  D3E3          ||shl      ebx, cl
11238A4E  |.  8B4D 08       ||mov      ecx, dword ptr ss:[ebp+8]
11238A51  |.  C1FA 05       ||sar      edx, 5
11238A54  |.  855C91 08     ||test     dword ptr ds:[ecx+edx*4+8], eb>
11238A58  |.  74 09         ||je       short 11238A63
11238A5A  |.  8A0C38        ||mov      cl, byte ptr ds:[eax+edi]
11238A5D  |.  80C1 01       ||add      cl, 1
11238A60  |.  880C38        ||mov      byte ptr ds:[eax+edi], cl
11238A63  |>  D02438        ||shl      byte ptr ds:[eax+edi], 1
11238A66  |.  8D56 FE       ||lea      edx, dword ptr ds:[esi-2]
11238A69  |.  8BCA          ||mov      ecx, edx
11238A6B  |.  83E1 1F       ||and      ecx, 1F
11238A6E  |.  BB 01000000   ||mov      ebx, 1
11238A73  |.  D3E3          ||shl      ebx, cl
11238A75  |.  8B4D 08       ||mov      ecx, dword ptr ss:[ebp+8]
11238A78  |.  C1FA 05       ||sar      edx, 5
11238A7B  |.  855C91 08     ||test     dword ptr ds:[ecx+edx*4+8], eb>
11238A7F  |.  74 09         ||je       short 11238A8A
11238A81  |.  8A0C38        ||mov      cl, byte ptr ds:[eax+edi]
11238A84  |.  80C1 01       ||add      cl, 1
11238A87  |.  880C38        ||mov      byte ptr ds:[eax+edi], cl
11238A8A  |>  83EE 04       ||sub      esi, 4
11238A8D  |.  836D F4 01    ||sub      dword ptr ss:[ebp-C], 1
11238A91  |.^ 0F85 59FFFFFF |\jnz      112389F0
11238A97  |.  8B75 0C       |mov      esi, dword ptr ss:[ebp+C]
11238A9A  |.  8345 F8 08    |add      dword ptr ss:[ebp-8], 8
11238A9E  |.  83C0 01       |add      eax, 1
11238AA1  |.  3BC6          |cmp      eax, esi
11238AA3  |.^ 0F82 37FFFFFF \jb       112389E0
11238AA9  |>  8D55 80       lea      edx, dword ptr ss:[ebp-80]
11238AAC  |.  52            push     edx                        ;  edx 为 sha 的数据结构
11238AAD  |.  E8 6E93EDFF   call     #207                       ;  sha 初始化
11238AB2  |.  56            push     esi                        ; |esi 为 edi 的长度
11238AB3  |.  8D45 80       lea      eax, dword ptr ss:[ebp-80]     ; |
11238AB6  |.  57            push     edi                        ; |edi 为分离出的 11 位数组
11238AB7  |.  50            push     eax                        ; |edx 为 sha 的数据结构
11238AB8  |.  E8 A393EDFF   call     #206                       ; \sha update 11 位值
11238ABD  |.  8B75 10       mov      esi, dword ptr ss:[ebp+10]
11238AC0  |.  8BC6          mov      eax, esi
11238AC2  |.  83C4 10       add      esp, 10
11238AC5  |.  8D50 01       lea      edx, dword ptr ds:[eax+1]
11238AC8  |>  8A08          /mov     cl, byte ptr ds:[eax]
11238ACA  |.  83C0 01       |add     eax, 1
11238ACD  |.  84C9          |test    cl, cl
11238ACF  |.^ 75 F7         \jnz     short 11238AC8
11238AD1  |.  2BC2          sub      eax, edx
11238AD3  |.  50            push     eax                        ;  eax 为 esi 的长度
11238AD4  |.  8D4D 80       lea      ecx, dword ptr ss:[ebp-80]
11238AD7  |.  56            push     esi                        ;  esi 为 11234201 代码处返回的数组，11234201 位于注册算法开始的地方
11238AD8  |.  51            push     ecx                        ;  ecx 为 sha 的数据结构
11238AD9  |.  E8 8293EDFF   call     #206                       ;  sha update 循环提供的常量数组
11238ADE  |.  8B75 18       mov      esi, dword ptr ss:[ebp+18]
11238AE1  |.  8B5D 14       mov      ebx, dword ptr ss:[ebp+14]
11238AE4  |.  56            push     esi                        ;  esi 为 ebx 的长度
11238AE5  |.  8D55 80       lea      edx, dword ptr ss:[ebp-80]
11238AE8  |.  53            push     ebx                        ;  ebx 为上层 11238B62 代码处提供的数组
11238AE9  |.  52            push     edx                        ;  ecx 为 sha 的数据结构
11238AEA  |.  E8 7193EDFF   call     #206                       ;  sha update 固定数组
11238AEF  |.  8D45 80       lea      eax, dword ptr ss:[ebp-80]
11238AF2  |.  50            push     eax
11238AF3  |.  8D4D DC       lea      ecx, dword ptr ss:[ebp-24]
11238AF6  |.  51            push     ecx                        ;  ecx 为 hash 后返回数组地址
11238AF7  |.  E8 243CF1FF   call     #205                       ;  sha final 处理
11238AFC  |.  56            push     esi                        ; |/n = 20 (32.)
11238AFD  |.  6A 00         push     0                           ; ||c = 00
11238AFF  |.  53            push     ebx                        ; ||s
11238B00  |.  E8 7BB90F00   call     <jmp.&MSVCR80.memset>      ; |\memset
11238B05  |.  8B75 1C       mov      esi, dword ptr ss:[ebp+1C]     ; |
11238B08  |.  6A 28         push     28                         ; |Arg1 = 00000028
11238B0A  |.  8D4D DC       lea      ecx, dword ptr ss:[ebp-24]     ; |
11238B0D  |.  E8 EEFDFFFF   call     11238900                     ; \这个函数比较简单，对 hash 后的 20 位值
11238B12  |.  83C4 30       add      esp, 30                      ;  通过处理，异或转变为 5 位数值存入到 esi 的地址
11238B15  |.  5B            pop      ebx
11238B16  |>  57            push     edi                        ; /block
11238B17  |.  FF15 BC253911 call     dword ptr ds:[<&MSVCR80.free>]  ; \free
11238B1D  |.  83C4 04       add      esp, 4
11238B20  |.  5F            pop      edi
11238B21  |.  5E            pop      esi
11238B22  |.  8BE5          mov      esp, ebp
11238B24  |.  5D            pop      ebp
11238B25  \.  C3            retn
```

进入 `11238900`，通过 `ida` 分析的代码如下:

```c
v4 = (unsigned __int8)(*(__int8 *)(a1 + 4) ^ *(__int8 *)(a1 + 9) ^ *(__int8 *)(a1 + 14) ^ *(__int8 *)(a1 + 19));
HIDWORD(v5) = (unsigned __int64)(v4 << 8) >> 32;
LODWORD(v5) = (unsigned __int8)(*(__int8 *)(a1 + 3) ^ *(__int8 *)(a1 + 8) ^ *(__int8 *)(a1 + 13) ^ *(__int8 *)(a1 + 18)) | ((__DWORD)v4 << 8);
HIDWORD(v5) = (unsigned __int64)(v5 << 8) >> 32;
LODWORD(v5) = (unsigned __int8)(*(__int8 *)(a1 + 2) ^ *(__int8 *)(a1 + 7) ^ *(__int8 *)(a1 + 12) ^ *(__int8 *)(a1 + 17)) | ((__DWORD)v5 << 8);
HIDWORD(v5) = (unsigned __int64)(v5 << 8) >> 32;
LODWORD(v5) = (unsigned __int8)(*(__int8 *)(a1 + 1) ^ *(__int8 *)(a1 + 6) ^ *(__int8 *)(a1 + 11) ^ *(__int8 *)(a1 + 16)) | ((__DWORD)v5 << 8);
result = (unsigned __int8)(*(__int8 *)a1 ^ *(__int8 *)(a1 + 5) ^ *(__int8 *)(a1 + 10) ^ *(__int8 *)(a1 + 15));
*(__DWORD *)a2 = result | ((__DWORD)v5 << 8);
*(__DWORD *)(a2 + 4) = (unsigned __int64)(v5 << 8) >> 32;
```

简单来说，就是 `sha` 后的 20 位值分别进行异或，产生 5 位的值，对压入 `11238b30` 第三个地址赋值，至此返回到 `1123A2E7`，判断对比 `ebx`，`edi` 的值是否相同。

然后程序再返回到 `11234286`，再到 `11233D80` 来判断时间数组，进入 `11233D80`。

```assembly
11233E4A  |.  8B93 10020000 mov      edx, dword ptr ds:[ebx+210]
11233E50  |.  52            push     edx                        ; /Arg2
11233E51  |.  56            push     esi                        ; |Arg1
11233E52  |.  E8 19EEEDFF   call     11112C70                   ; \vmwareba.11112C70
11233E57  |.  83C4 08       add      esp, 8
11233E5A  |>  80BB EC010000>cmp     byte ptr ds:[ebx+1EC], 0          ;  前 2 4 字节随便为 0
11233E61  |.  0F84 56030000 je       112341BD
11233E67  |.  83BB F0010000>cmp     dword ptr ds:[ebx+1F0], 0        ;  判断该位是否为 0,不为 0 有时间限制，0 没时间限制
11233E6E  |.  0F86 49030000 jbe      112341BD
```

`11233D80` 其他的代码不多说，这里只说 `11233E67`，`[ebx+1F0]` 主要是 `11239EF0` 作处理的，可参考之前 `11239EF0` 的说明。

### 【软件加密思路】

1. 在一个循环中，分别获取注册码，在索引数组中获取索引值，根据索引值获取注册码的值，计算注册码在固定数组中的位置为 `Index`。
   转换数组 `A` 循环从 `0x7C` 位开始到 `0` 位结束，每一循环处理 5 位，每一次判断 `Index` 低位为 1 时，对其数组 `A` 置 1，否则不作处理，然后 `Index` 右移一位。
2. 根据步骤 1 数组 `A` 进行处理，循环控制从 `0` 到 `105` 位之间的处理，分别获取当前位数的值 `C`，内循环控制 `0` 到 `20` 位,分别对 3 求余，然后根据求余的值，去到新数组 `B`，从 `0`，`40`，`80` 位开始的索引范围内，然后根据值 `C` 判断是否对数组 `B` 当前位置 1，然后索引加一，最后从 `107` 位开始到 `128` 位，直接原值到 `B` 数组。
3. 经过上述两步后，还原出数组 `B`，然后经过 `0x11239CD0` 的处理，获取第 14 位的值，判断是否在合理值范围内。
4. 根据数组 `B`，然后经过 `0x11239CD0` 的处理，判断第 9 位的值，是否有时间限制，产生时间数组 `C`。
