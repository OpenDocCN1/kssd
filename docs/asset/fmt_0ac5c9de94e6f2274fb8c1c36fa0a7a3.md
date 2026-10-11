
## 汇编分析：注册码验证逻辑

```
006472D7   |>  03C1            |add eax,ecx  

006472D9   |.  8A9C04 08020000 |mov bl,byte ptr ss:[esp+eax+208]                 ;  取注册码 a 位值,  

006472E0   |.  C0E3 04         |shl bl,4  

006472E3   |.  029C04 09020000 |add bl,byte ptr ss:[esp+eax+209]                 ;  取注册码 a+1 位值  

006472EA   |.  80EB 51         |sub bl,51  

006472ED   |.  3AD3            |cmp dl,bl  

006472EF   |.  0F85 06040000   |jnz PDFEdit.006476FB  

006472F5   |.  83C1 02         |add ecx,2  

006472F8   |.  83C6 04         |add esi,4  

006472FB   |.  83F9 08         |cmp ecx,8  

006472FE   |.^ 7C C6           \jl short PDFEdit.006472C6  

00647300   |.  8B9424 20140000 |mov edx,dword ptr ss:[esp+1420]  
```

看到调用 `GetPrivateProfileStringA` ![](img/d23858dd221cce728a3a61b46badb044.jpg "恐怖!")

原来 `pedkey.txt` 文件其实就是一个 ini 文件，保存注册信息的。

### 构造合法 `pedkey` 文件

经单步跟踪构造合法 `pedkey` 文件：

```ini
[PDF Editor]  

SN="PED"  

Modules="MyModules"  

Users="qqjack"  

Licensee="Unregistered Retail Customer"  

ExpireDate="2012/12/12"  

Sign="MySelf"  

LicenseDate="2011/04/16"  

CDKey="111111111111111111111111111"
```

跟踪发现 `SN` 必须为 `PED`，`Licensee="Unregistered Retail Customer"`。其它随意。

这样可以顺利到达注册码计算处。

#### 跟入计算核心

```
0064725A   |.  E8 213D0100     call PDFEdit.0065AF80                               ;  对数据区 buf 中的数据进行计算  

0065AF80   /$  6A FF           push -1  

0065AF82   |.  68 687B6A00     push PDFEdit.006A7B68                               ;  SE 句柄安装  

0065AF87   |.  64:A1 00000000  mov eax,dword ptr fs:[0]  

0065AF8D   |.  50              push eax  

0065AF8E   |.  64:8925 0000000>mov dword ptr fs:[0],esp  

0065AF95   |.  83EC 2C         sub esp,2C  

0065AF98   |.  56              push esi  

0065AF99   |.  6A 01           push 1                                              ; /Arg1 = 00000001  

0065AF9B   |.  8D4C24 08       lea ecx,dword ptr ss:[esp+8]                        ; |向地址 ECX 写入  

0065AF9F   |.  E8 0CA20000     call PDFEdit.006651B0                               ; \\算法预处理处，对变量 buf2 进行初始化  

0065AFA4   |.  8B4424 08       mov eax,dword ptr ss:[esp+8]  

0065AFA8   |.  C74424 38 00000>mov dword ptr ss:[esp+38],0  

0065AFB0   |.  8B48 04         mov ecx,dword ptr ds:[eax+4]  

0065AFB3   |.  8B4424 48       mov eax,dword ptr ss:[esp+48]  

0065AFB7   |.  50              push eax  

0065AFB8   |.  8B540C 0C       mov edx,dword ptr ss:[esp+ecx+C]  

0065AFBC   |.  8D740C 0C       lea esi,dword ptr ss:[esp+ecx+C]  

0065AFC0   |.  8B4C24 48       mov ecx,dword ptr ss:[esp+48]  

0065AFC4   |.  51              push ecx                                            ;  数据区 buf 地址  

0065AFC5   |.  8BCE            mov ecx,esi  

0065AFC7   |.  FF52 04         call dword ptr ds:[edx+4]                           ;  对修改的部分注册码进行计算,结果存入 buf2  

0065AFCA   |.  8B4424 40       mov eax,dword ptr ss:[esp+40]  

0065AFCE   |.  8B16            mov edx,dword ptr ds:[esi]  

0065AFD0   |.  50              push eax  

0065AFD1   |.  8BCE            mov ecx,esi                                         ;  计算结果  

0065AFD3   |.  FF52 08         call dword ptr ds:[edx+8]                           ;  进行 buf2 二次计算，得出最后运算结果  

0065AFD6   |.  8D4C24 2C       lea ecx,dword ptr ss:[esp+2C]  

0065AFDA   |.  C74424 38 FFFFF>mov dword ptr ss:[esp+38],-1  

0065AFE2   |.  E8 09880000     call PDFEdit.006637F0  

0065AFE7   |.  8B4C24 30       mov ecx,dword ptr ss:[esp+30]  

0065AFEB   |.  5E              pop esi  

0065AFEC   |.  64:890D 0000000>mov dword ptr fs:[0],ecx  

0065AFF3   |.  83C4 38         add esp,38  

0065AFF6   \\.  C3              retn
```

跟入 `call PDFEdit.006651B0`；`\\算法预处理处，对变量 buf2 进行初始化`。其调用函数如下：

```
00665250   /$  33C0            xor eax,eax  

00665252   |.  8941 10         mov dword ptr ds:[ecx+10],eax  

00665255   |.  8941 0C         mov dword ptr ds:[ecx+C],eax  

00665258   |.  8B41 20         mov eax,dword ptr ds:[ecx+20]  

0066525B   |.  C700 01234567   mov dword ptr ds:[eax],67452301                      ;  保存常量到 buf2[0]  

00665261   |.  8B51 20         mov edx,dword ptr ds:[ecx+20]  

00665264   |.  C742 04 89ABCDE>mov dword ptr ds:[edx+4],EFCDAB89                   ;  保存常量到 buf2[1]  

0066526B   |.  8B41 20         mov eax,dword ptr ds:[ecx+20]  

0066526E   |.  C740 08 FEDCBA9>mov dword ptr ds:[eax+8],98BADCFE                   ;  保存常量到 buf2[2]  

00665275   |.  8B49 20         mov ecx,dword ptr ds:[ecx+20]  

00665278   |.  C741 0C 7654321>mov dword ptr ds:[ecx+C],10325476                   ;  保存常量到 buf2[3]  

0066527F   \\.  C3              retn
```

跟入 `call dword ptr ds:[edx+4]`；`对修改的部分注册码进行计算,结果存入 buf2`。调用如下：

实际这里调用了两次，第一次第二个参数为 `buf`；第二次第二个参数为 `buf+sizeof(buf)/2`。

```
00665287    .  50               push eax                                            ;  数据区 buf/buf+sizeof(buf)/2  

00665288    .  51               push ecx                                            ;  buf2  

00665289    .  E8 B2000000     call PDFEdit.00665340                               ;  计算核心函数
```

跟入 `call dword ptr ds:[edx+8]`；`进行 buf2 二次计算，得出最后运算结果`。

```
006652A0    > /53               push ebx  

006652A1    . |8BD9             mov ebx,ecx  

006652A3    . |55               push ebp  

006652A4    . |56               push esi  

006652A5    . |8D6B D8          lea ebp,dword ptr ds:[ebx-28]  

006652A8    . |68 80000000      push 80  

006652AD    . |6A 38            push 38  

006652AF    . |8BCD             mov ecx,ebp  

006652B1    . |E8 DAE6FFFF      call PDFEdit.00663990                               ;  计算新的数据区 buf3  

006652B6    . |8B73 F0          mov esi,dword ptr ds:[ebx-10]  

006652B9    . |3BF6             cmp esi,esi  

006652BB    . |74 0B            je short PDFEdit.006652C8  

006652BD    . |57               push edi  

006652BE    . |B9 0E000000      mov ecx,0E  

006652C3    . |8BFE             mov edi,esi  

006652C5    . |F3:A5            rep movs dword ptr es:[edi],dword ptr ds:[esi]  

006652C7    . |5F               pop edi  

006652C8    > |8B43 F0          mov eax,dword ptr ds:[ebx-10]                       ;  地址  

006652CB    . |8B4B E4          mov ecx,dword ptr ds:[ebx-1C]  

006652CE    . |8948 38          mov dword ptr ds:[eax+38],ecx                       ;  将 ecx 值置入 buf3 的倒数第八字节后，经调试发现 ecx 值固定为 0x400  

006652D1    . |8B53 F0          mov edx,dword ptr ds:[ebx-10]  

006652D4    . |8B43 E8          mov eax,dword ptr ds:[ebx-18]  

006652D7    . |8942 3C          mov dword ptr ds:[edx+3C],eax                       ;  将 eax 值置入 buf3 的倒数第四字节后，经调试发现 ecx 值固定为 0  

006652DA    . |8B4B F0          mov ecx,dword ptr ds:[ebx-10]  

006652DD    . |8B53 F8          mov edx,dword ptr ds:[ebx-8]  

006652E0    . |51               push ecx                                            ;  新数据区 buf3  

006652E1    . |52               push edx                                            ;  buf2  

006652E2    . |E8 59000000     call PDFEdit.00665340             ; 核心函数 计算最终结果
```

跟入 `call PDFEdit.00663990`；`计算新的数据区 buf3`。

```
00663990   /$  53               push ebx  

00663991   |.  56               push esi  

00663992   |.  8BF1             mov esi,ecx  

00663994   |.  8B5C24 0C        mov ebx,dword ptr ss:[esp+C]                          ;  第二参数固定为 0x38  

00663998   |.  57               push edi  

00663999   |.  8B56 0C          mov edx,dword ptr ds:[esi+C]                          ;  此处值固定为 0x400  

0066399C   |.  8B46 08          mov eax,dword ptr ds:[esi+8]                          ;  此处值固定为 0x40  

0066399F   |.  8B4E 18          mov ecx,dword ptr ds:[esi+18]                        ;  新数据区 buf3 地址  

006639A2   |.  C1EA 03          shr edx,3  

006639A5   |.  48               dec eax  

006639A6   |.  23D0             and edx,eax  

006639A8   |.  8A4424 14        mov al,byte ptr ss:[esp+14]                           ;  第一参数固定为 0x80
```
