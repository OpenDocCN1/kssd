
### 00407631 汇编分析
```
00407631  |.  E8 8ABB0400   call    004531C0      ;powmod(m,e,n,c)//计算 c=m^e mod n
00407636  |.  8D9424 200500>lea     edx, ss:[esp+520]
0040763D  |.  6A 00         push    0
0040763F  |.  52            push    edx
00407640  |.  8B4C24 18     mov     ecx, ss:[esp+18]
00407644  |.  51            push    ecx
00407645  |.  68 00010000   push    100
0040764A  |.  E8 A1C20400   call    004538F0      ;big_to_bytes(256,c,temp,FALSE);
0040764F  |.  8BD7          mov     edx, edi       这个 temp 的值就是后面运算需要的
00407651  |.  52            push    edx
00407652  |.  E8 B9A90400   call    00452010      ;mirkill(m);//释放内存
00407657  |.  8B5424 24     mov     edx, ss:[esp+24]
0040765B  |.  52            push    edx
0040765C  |.  E8 AFA90400   call    00452010      ;mirkill(c);//释放内存
00407661  |.  8B5424 34     mov     edx, ss:[esp+34]
00407665  |.  52            push    edx
00407666  |.  E8 A5A90400   call    00452010      ;mirkill(n);//释放内存
0040766B  |.  8BD6          mov     edx, esi
0040766D  |.  52            push    edx
0040766E  |.  E8 9DA90400   call    00452010      ;mirkill(e);//释放内存
00407673  |.  83C4 30       add     esp, 30
00407676  |.  E8 B5A90400   call    00452030      ;mirexit();//释放内存
0040767B  |.^ E9 5EFFFFFF   jmp     004075DE
00407680  |>  .
.
.
```

### 0045E6E5 汇编分析
```
0045E6E5  /$  FF7424 04     push    dword ptr ss:[esp+4]
0045E6E9  |.  E8 6CFFFFFF   call    0045E65A
0045E6EE  |.  59            pop     ecx
0045E6EF  \.  C3            retn
```

### 0045E65A 汇编分析
```
0045E65A  /$  53            push    ebx
0045E65B  |.  55            push    ebp
0045E65C  |.  56            push    esi
0045E65D  |.  57            push    edi
0045E65E  |.  8B7C24 14     mov     edi, ss:[esp+14]
0045E662  |>  833D B41B4900>/cmp     dword ptr ds:[491BB4], 1            ;  00491bb4 永远都是 1
0045E669  |.  7E 0F         |jle     short 0045E67A
0045E66B  |.  0FB607        |movzx   eax, byte ptr ds:[edi]
0045E66E  |.  6A 08         |push    8
0045E670  |.  50            |push    eax
0045E671  |.  E8 CC580000   |call    00463F42
0045E676  |.  59            |pop     ecx
0045E677  |.  59            |pop     ecx
0045E678  |.  EB 0F         |jmp     short 0045E689
0045E67A  |>  0FB607        |movzx   eax, byte ptr ds:[edi]              ;取出前面运算出的 temp 的值(rsa 的值)
0045E67D  |.  8B0D A8194900 |mov     ecx, ds:[4919A8]                  ;  dvdtomp3.004919B2
0045E683  |.  8A0441        |mov     al, ds:[ecx+eax*2]
0045E686  |.  83E0 08       |and     eax, 8        ;
0045E689  |>  85C0          |test    eax, eax
0045E68B  |.  74 03         |je      short 0045E690
0045E68D  |.  47            |inc     edi
0045E68E  |.^ EB D2         \jmp     short 0045E662
```
`[rsa[i]*2+4919B2]`是`*8`之类的就取下一个,继续循环,否则就走

```
0045E690  |>  0FB637        movzx   esi, byte ptr ds:[edi]
0045E693  |.  47            inc     edi
0045E694  |.  83FE 2D       cmp     esi, 2D                         ;  是 2D 在返回前就会把结果 neg eax
0045E697  |.  8BEE          mov     ebp, esi
0045E699  |.  74 05         je      short 0045E6A0
0045E69B  |.  83FE 2B       cmp     esi, 2B                         ;  如果是 2D，2B 就取下一位
0045E69E  |.  75 04         jnz     short 0045E6A4
0045E6A0  |>  0FB637        movzx   esi, byte ptr ds:[edi]
0045E6A3  |.  47            inc     edi
0045E6A4  |>  33DB          xor     ebx, ebx
0045E6A6  |>  833D B41B4900>/cmp     dword ptr ds:[491BB4], 1
0045E6AD  |.  7E 0C         |jle     short 0045E6BB
0045E6AF  |.  6A 04         |push    4
0045E6B1  |.  56            |push    esi
0045E6B2  |.  E8 8B580000   |call    00463F42
0045E6B7  |.  59            |pop     ecx
0045E6B8  |.  59            |pop     ecx
0045E6B9  |.  EB 0B         |jmp     short 0045E6C6
0045E6BB  |>  A1 A8194900   |mov     eax, ds:[4919A8]
0045E6C0  |.  8A0470        |mov     al, ds:[eax+esi*2]             ;  4919A8+Rsa[i]*2
0045E6C3  |.  83E0 04       |and     eax, 4                         ; 35(5),23(3),ab(b)
0045E6C6  |>  85C0          |test    eax, eax
0045E6C8  |.  74 0D         |je      short 0045E6D7
0045E6CA  |.  8D049B        |lea     eax, ds:[ebx+ebx*4]             ;  eax=ebx*5,ebx 初始为 0
0045E6CD  |.  8D5C46 D0     |lea     ebx, ds:[esi+eax*2-30]          ;  ebx=ebx*5*2-30+RSA[i]
0045E6D1  |.  0FB637        |movzx   esi, byte ptr ds:[edi]
0045E6D4  |.  47            |inc     edi
0045E6D5  |.^ EB CF         \jmp     short 0045E6A6
0045E6D7  |>  83FD 2D       cmp     ebp, 2D
0045E6DA  |.  8BC3          mov     eax, ebx
0045E6DC  |.  75 02         jnz     short 0045E6E0
0045E6DE  |.  F7D8          neg     eax
0045E6E0  |>  5F            pop     edi
0045E6E1  |.  5E            pop     esi
0045E6E2  |.  5D            pop     ebp
0045E6E3  |.  5B            pop     ebx
0045E6E4  \.  C3            retn
```
`[rsa[i]*2+4919B2]`是`*4`之类的就运算， 取下一个继续循环,否则退出循环

我们假设 `rsa[i]=FF`，`4919B2<=[rsa 的值*2+4919B2]<=491BB0`，下面我们就去看看这块区域的值

`rsa[i]={0x9,0xA,0xB,0xC,0xD,0x20}`   `[rsa[i]*2+4919B2]`的结果是`*8`

`rsa[i]={0x30,0x31,0x32,0x33,0x34,0x35,0x36,0x37,0x38,0x39}``[rsa[i]*2+4919B2]`的结果是`*4`

### 序列号三
我们利用她的流程来个最简单的:)

第一次=1,   `rsa[i]={0x31,0x3A) ,(1:)`    `185B973541F05CF1`   `3A` 随便选个,只要不上面的表就可以了

第二次=3,   `rsa[i]={0x33,0x3A) ,(3:)`    `1177CA7E07FB44E1`

第三次=2,   `rsa[i]={0x32,0x3A) ;(2:)`用 rsa 运算得出 `53945D865D558F7A`

`serail=185B973541F05CF11177CA7E07FB44E153945D865D558F7A`

典型的利用 Miracl 大数运算库进行 RSA 算法的实例,那些函数看 Miracl 的参考

用 Rsatools2 工具输入模数 `n`，得到下面的几组数据

```
//P(HEX)10138CEE3          94E13703

//Q(HEX)A0FEDEFB    F8AD4EBB

//N(HEX)a1c397b69b55b291,    909ef746744d1931

//D(HEX)3BD60FDDDF705D29,  55F020BFB0DE4E81

//E(HEX)10001

//Keysize(Bits)=64
```

刚好弄完看世界杯![](img/dd27b7eb3e47f4f9a8b69ac3c32485f5.jpg)

`4dd466d0e1e0cc123afbe504da2462ca`
