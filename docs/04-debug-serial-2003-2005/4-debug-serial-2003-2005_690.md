
# 大家新年好！来个 Anti 的算法分析和注册机编写

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-96.htm`](https://www.pediy.com/kssd/pediy07/pediy7-96.htm)

大家新年好！该吃年夜饭的时候了，我来贴上以前写的一篇：

### Anti ****V2.6.1 算法分析和注册机编写

国产反木马软件,选择它是看它算法简单;

我这儿不能上网,资料不好找,很多东西都是按我自己的理解,有错误的地方请大家指出,谢谢!

用`Ollydbg1.10`,怎样追到算法部分就略了,算法代码如下:(许多地方都是用 C#描述的，大家将就看吧)

```
//在这之前注册码已被转换为大写格式

00403A34   /$  55                  push ebp
00403A35   |.  8BEC                mov ebp,esp
00403A37   |.  81C4 CCFDFFFF       add esp,-234
00403A3D   |.  B8 BC4E5500         mov eax,00554EBC
00403A42   |.  53                  push ebx
00403A43   |.  56                  push esi
00403A44   |.  57                  push edi
00403A45   |.  E8 9AAD1300         call 0053E7E4
00403A4A   |.  C745 F8 01000000    mov dword ptr ss:[ebp-8],1
00403A51   |.  8D55 08             lea edx,dword ptr ss:[ebp+8]
00403A54   |.  8D45 08             lea eax,dword ptr ss:[ebp+8]
00403A57   |.  E8 64AF1300         call 0053E9C0
00403A5C   |.  FF45 F8             inc dword ptr ss:[ebp-8]
00403A5F   |.  66:C745 EC 0800     mov word ptr ss:[ebp-14],8
00403A65   |.  68 C7000000         push 0C7
00403A6A   |.  837D 08 00          cmp dword ptr ss:[ebp+8],0
00403A6E   |.  74 05               je short 00403A75
00403A70   |.  8B55 08             mov edx,dword ptr ss:[ebp+8]
00403A73   |.  EB 05               jmp short 00403A7A
00403A75   |>  BA BC4B5500         mov edx,00554BBC
00403A7A   |>  52                  push edx                      ; |src
00403A7B   |.  8D85 0CFFFFFF       lea eax,dword ptr ss:[ebp-F4]       ; |
00403A81   |.  50                  push eax                      ; |dest
00403A82   |.  E8 27F91400         call <jmp.&CC3260MT._strncpy>      ; \_strncpy
00403A87   |.  83C4 0C             add esp,0C
00403A8A   |.  8D8D 0CFFFFFF       lea ecx,dword ptr ss:[ebp-F4]
00403A90   |.  68 80000000         push 80                       ; /maxlen = 80 (128.)
00403A95   |.  51                  push ecx                      ; |src
00403A96   |.  68 340D5900         push 00590D34                 ; |dest = 00590D34
00403A9B   |.  E8 0EF91400         call <jmp.&CC3260MT._strncpy>      ; \_strncpy
00403AA0   |.  83C4 0C             add esp,0C
00403AA3   |.  8D85 0CFFFFFF       lea eax,dword ptr ss:[ebp-F4]
00403AA9   |.  C645 D3 00          mov byte ptr ss:[ebp-2D],0
00403AAD   |.  50                  push eax
00403AAE   |.  E8 F9F71400         call <jmp.&CC3260MT.__lstrupr>
00403AB3   |.  59                  pop ecx
00403AB4   |.  8D9D 0CFFFFFF       lea ebx,dword ptr ss:[ebp-F4]
00403ABA   |.  68 BD4B5500         push 00554BBD
00403ABF   |.  53                  push ebx
00403AC0   |.  E8 78AC1300         call 0053E73D                 ;  找"-"
00403AC5   |.  83C4 08             add esp,8
00403AC8   |.  8BF0                mov esi,eax
00403ACA   |.  85F6                test esi,esi
00403ACC   |.  75 23               jnz short 00403AF1            ;  找不到就 OVER
00403ACE   |.  33C0                xor eax,eax
00403AD0   |.  BA 02000000         mov edx,2
00403AD5   |.  50                  push eax
00403AD6   |.  8D45 08             lea eax,dword ptr ss:[ebp+8]
00403AD9   |.  FF4D F8             dec dword ptr ss:[ebp-8]
00403ADC   |.  E8 07B11300         call 0053EBE8
00403AE1   |.  58                  pop eax
00403AE2   |.  8B55 DC             mov edx,dword ptr ss:[ebp-24]
00403AE5   |.  64:8915 00000000    mov dword ptr fs:[0],edx
00403AEC   |.  E9 1D020000         jmp 00403D0E
00403AF1   |>  C606 00             mov byte ptr ds:[esi],0
00403AF4   |.  46                  inc esi
00403AF5   |.  803E 00             cmp byte ptr ds:[esi],0       ;  是不是偷懒,只输了一段注册码?:>
00403AF8   |.  75 23               jnz short 00403B1D
00403AFA   |.  33C0                xor eax,eax
00403AFC   |.  BA 02000000         mov edx,2
00403B01   |.  50                  push eax
00403B02   |.  8D45 08             lea eax,dword ptr ss:[ebp+8]
00403B05   |.  FF4D F8             dec dword ptr ss:[ebp-8]
00403B08   |.  E8 DBB01300         call 0053EBE8
00403B0D   |.  58                  pop eax
00403B0E   |.  8B55 DC             mov edx,dword ptr ss:[ebp-24]
00403B11   |.  64:8915 00000000    mov dword ptr fs:[0],edx
00403B18   |.  E9 F1010000         jmp 00403D0E
00403B1D   |>  68 BF4B5500         push 00554BBF
00403B22   |.  56                  push esi
00403B23   |.  E8 15AC1300         call 0053E73D                 ;  找"-"
00403B28   |.  83C4 08             add esp,8
00403B2B   |.  8BF8                mov edi,eax
00403B2D   |.  85FF                test edi,edi
00403B2F   |.  75 23               jnz short 00403B54
00403B31   |.  33C0                xor eax,eax
00403B33   |.  BA 02000000         mov edx,2
00403B38   |.  50                  push eax
00403B39   |.  8D45 08             lea eax,dword ptr ss:[ebp+8]
00403B3C   |.  FF4D F8             dec dword ptr ss:[ebp-8]
00403B3F   |.  E8 A4B01300         call 0053EBE8
00403B44   |.  58                  pop eax
00403B45   |.  8B55 DC             mov edx,dword ptr ss:[ebp-24]
00403B48   |.  64:8915 00000000    mov dword ptr fs:[0],edx
00403B4F   |.  E9 BA010000         jmp 00403D0E
00403B54   |>  C607 00             mov byte ptr ds:[edi],0
00403B57   |.  47                  inc edi
00403B58   |.  803F 00             cmp byte ptr ds:[edi],0
00403B5B   |.  75 23               jnz short 00403B80
00403B5D   |.  33C0                xor eax,eax
00403B5F   |.  BA 02000000         mov edx,2
00403B64   |.  50                  push eax
00403B65   |.  8D45 08             lea eax,dword ptr ss:[ebp+8]
00403B68   |.  FF4D F8             dec dword ptr ss:[ebp-8]
00403B6B   |.  E8 78B01300         call 0053EBE8
00403B70   |.  58                  pop eax
00403B71   |.  8B55 DC             mov edx,dword ptr ss:[ebp-24]
00403B74   |.  64:8915 00000000    mov dword ptr fs:[0],edx
00403B7B   |.  E9 8E010000         jmp 00403D0E
00403B80   |>  53                  push ebx                  ; /长征结束了!注册码格式:AAAA-SSSS-DDDD
00403B81   |.  E8 92010000         call 00403D18             ; \第一段注册码转换为 int
```

进入:

```
00403D18   /$  55                  push ebp
00403D19   |.  8BEC                mov ebp,esp
00403D1B   |.  83C4 F8             add esp,-8
00403D1E   |.  53                  push ebx
00403D1F   |.  56                  push esi
00403D20   |.  8B5D 08             mov ebx,dword ptr ss:[ebp+8]
00403D23   |.  53                  push ebx                  ; /s
00403D24   |.  E8 79F61400         call <jmp.&CC3260MT._strlen>     ; \_strlen
00403D29   |.  59                  pop ecx
00403D2A   |.  8945 FC             mov dword ptr ss:[ebp-4],eax
00403D2D   |.  33F6                xor esi,esi
00403D2F   |.  33C0                xor eax,eax
00403D31   |.  8BCB                mov ecx,ebx
00403D33   |.  8945 F8             mov dword ptr ss:[ebp-8],eax
00403D36   |.  EB 44               jmp short 00403D7C
00403D38   |>  8A01                /mov al,byte ptr ds:[ecx]      ;取输入的字符串
00403D3A   |.  8A51 01             |mov dl,byte ptr ds:[ecx+1]    ;取两个
00403D3D   |.  0FBED8              |movsx ebx,al                ;编的话要用 sbyte 类型
00403D40   |.  83FB 46             |cmp ebx,46
00403D43   |.  7F 09               |jg short 00403D4E
00403D45   |.  83FB 41             |cmp ebx,41
00403D48   |.  7C 04               |jl short 00403D4E
00403D4A   |.  04 C9               |add al,0C9              ;如果有字符是"A"到"f"的,把 Hex 加上 0xc9,否则加上 0x40
00403D4C   |.  EB 02               |jmp short 00403D50      ;这样"0"到"9"就成了 0x0~0x9,"A"到"F"就成了 0xA~0xF,
```



```
00403D4E   |>  04 D0                 |add al,0D0                     ;很有意思,如果编注册机的话就简化了很多  
00403D50   |>  0FBEDA              |movsx ebx,dl  
00403D53   |.  83FB 46             |cmp ebx,46                     ;这是取得第二个  
00403D56   |.  7F 0A               |jg short 00403D62  
00403D58   |.  83FB 41             |cmp ebx,41  
00403D5B   |.  7C 05               |jl short 00403D62  
00403D5D   |.  80C2 C9             |add dl,0C9  
00403D60   |.  EB 03               |jmp short 00403D65  
00403D62   |    80C2 D0            |add dl,0D0  
00403D65   |>  0FBEC0              |movsx eax,al  
00403D68   |.  0FBED2              |movsx edx,dl  
00403D6B   |.  C1E0 04             |shl eax,4                      ;第一个 Int 左移 4  
00403D6E   |.  C1E6 08             |shl esi,8                      ;上次结果左移 8  
00403D71   |.  03C2                |add eax,edx                    ;把两次结果组合起来  
00403D73   |.  03F0                |add esi,eax                    ;把上次结果与这次组合  
00403D75   |.  8345 F8 02         |add dword ptr ss:[ebp-8],2  
00403D79   |.  83C1 02             |add ecx,2  
00403D7C   |>  8B45 F8             |  mov eax,dword ptr ss:[ebp-8]  
00403D7F   |.  3B45 FC             |cmp eax,dword ptr ss:[ebp-4]  
00403D82   \  ^ 72 B4             \jb short 00403D38  
00403D84   |.  8BC6                 mov eax,esi  
00403D86   |.  5E                   pop esi  
00403D87   |.  5B                   pop ebx  
00403D88   |.  59                   pop ecx  
00403D89   |.  59                   pop ecx  
00403D8A   |.  5D                   pop ebp  
00403D8B   \.  C3                   retn  
```

假设输入`"123ABC"`

程序先取`"1"`,`"2"`,`Hex` 分别为 `0x31`、`0x32`，因为 `0x30<0x41`，用 `sbyte` 类型的 `0x31+0xd0`，结果为 `0x1`

同样，`"2"` 的结果为 `0x2`，再把 `0x2<<4` 后加上 `0x2`，得 `0x12`，再加上上次结果（第一次当然为 `0x0`），最后结果 `0x12`；

整个字符串结果 `0x123ABC`；

这种算法使我们可以很轻松的编出注册机，我们要想把一个 `Int` 转换为可显示字符，就可以先把它转换为 `16` 进制形式的字符串，因为程序每次循环要处理两个字符，所以还要在长度不为偶数的字符串前加`"0"`，输出就可以了

退出 `Call`，继续：

```
00403B86   |.  59                   pop ecx  
00403B87   |.  8BD8                 mov ebx,eax  
00403B89   |.  56                   push esi                        ; /Arg1  
00403B8A   |.  E8 89010000          call 00403D18                   ; \\第二段注册码转换为 int  
00403B8F   |.  59                   pop ecx  
00403B90   |.  8945 D8             mov dword ptr ss:[ebp-28],eax  
00403B93   |.  57                   push edi                        ; /Arg1  
00403B94   |.  E8 7F010000          call 00403D18                   ; \\第三段注册码转换为 int  
00403B99   |.  59                   pop ecx  
00403B9A   |.  891D B40D5900        mov dword ptr ds:[590DB4],ebx  
00403BA0   |.  8B4D D8             mov ecx,dword ptr ss:[ebp-28]  
00403BA3   |.  33D2                 xor edx,edx  
00403BA5   |.  890D B80D5900        mov dword ptr ds:[590DB8],ecx               ;  第二段 int  
00403BAB   |.  A3 BC0D5900          mov dword ptr ds:[590DBC],eax               ;  第三段 int,好像没用到它??  
00403BB0   |.  8BC3                 mov eax,ebx                       ;  第一段 int,叫 sn1 吧  
00403BB2   |.  B9 C7020000          mov ecx,2C7  
00403BB7   |.  F7F1                 div ecx                           ;  sn1 / 0x2c7  
00403BB9   |.  8D0C9B              lea ecx,dword ptr ds:[ebx+ebx*4]            ;  sn1 +sn1 *4  
00403BBC   |.  8BD0                 mov edx,eax                       ;  sn1 / 0x2c7 的商  
00403BBE   |.  C1E1 03             shl ecx,3                       ;  (sn1 +sn1 *4)<<3  
00403BC1   |.  8D049B              lea eax,dword ptr ds:[ebx+ebx*4]  
00403BC4   |.  2BCB                 sub ecx,ebx                       ;  ((sn1 +sn1 *4)<<3) -sn1  
00403BC6   |.  8DBD 8CFEFFFF        lea edi,dword ptr ss:[ebp-174]  
00403BCC   |.  C1E1 03             shl ecx,3                       ;  (((sn1 +sn1 *4)<<3) -sn1)<<3  
00403BCF   |.  BE 784A5500          mov esi,00554A78  
00403BD4   |.  2BCB                 sub ecx,ebx                       ;  ((((sn1 +sn1 *4)<<3) -sn1)<<3)-sn1  
00403BD6   |.  D1E9                 shr ecx,1                       ;  (((((sn1 +sn1 *4)<<3) -sn1)<<3)-sn1)>>1  
00403BD8   |.  C1E8 03             shr eax,3                       ;  往上找 eax:(sn1 +sn1 *4  )>>3  
00403BDB   |.  03C1                 add eax,ecx                     ;  上面两个相加,太长了,我晕!叫它 sn1Temp 吧  
00403BDD   |.  B9 20000000          mov ecx,20  
00403BE2   |.  03D0                 add edx,eax                     ;  edx 是 sn1/0x2c7 的商 + sn1Temp  
00403BE4   |.  0FAFDA              imul ebx,edx                    ;  sn1 和上面的相乘,叫它 sn1Temp1  
00403BE7   |.  8D0452              lea eax,dword ptr ds:[edx+edx*2]            ;  edx=(sn1/0x2c7)+((sn1+sn1*4)>>3)+  
((((((sn1+sn1*4)<<3)-sn1)<<3)-sn1)>>1)  
00403BEA   |.  C1E0 04             shl eax,4                       ;  ((edx+edx*2)<<4)  
00403BED   |.  2BC2                 sub eax,edx                     ;  ((edx+edx*2)<<4)-edx  
00403BEF   |.  C1E0 02             shl eax,2                       ;  (((edx+edx*2)<<4)-edx)<<2  
00403BF2   |.  2BC2                 sub eax,edx                     ;  ((((edx+edx*2)<<4)-edx)<<2)-edx  
00403BF4   |.  03C3                 add eax,ebx                     ;  (((((edx+edx*2)<<4)-edx)<<2)-edx)+sn1Temp1  
00403BF6   |.  8BD0                 mov edx,eax  
00403BF8   |.  F7D2                 not edx                       ;  ~((((((edx+edx*2)<<4)-edx)<<2)-edx)+sn1Temp1)  
00403BFA   |.  F3:A5               rep movs dword ptr es:[edi],dword ptr ds:>;  俺们都是做苦力活的^\_^幸亏有计算机,要我算的话还不得算上十年八年!!  
00403BFC   |.  33DB                 xor ebx,ebx  
00403BFE   |.  33C0                 xor eax,eax  
00403C00   |.  8DB5 8CFEFFFF        lea esi,dword ptr ss:[ebp-174]  
00403C06   |>  8B0E                 /mov ecx,dword ptr ds:[esi]             ;  一个固定的表,开始读数据  
00403C08   |.  BF 01000000          |mov edi,1  
00403C0D   |.  D3E7                 |shl edi,cl                      ;  1<<*位,表中的数据,重要!  
00403C0F   |.  85D7                 |test edi,edx                    ;  拿它和上面的结果对比,做与运算  
00403C11   |.  0F97C1              |seta cl                       ;  设置标志  
00403C14   |.  83E1 01             |and ecx,1                       ;  还是保险点好^\_^  
00403C17   |.  83C6 04             |add esi,4                       ;  为下次循环做准备  
00403C1A   |.  8BF9                 |mov edi,ecx  
00403C1C   |.  8BC8                 |mov ecx,eax                     ;  第 n 次  
00403C1E   |.  D3E7                 |shl edi,cl                      ;  True 的话 1<<n,False 的话 0<<n  
00403C20   |.  0BDF                 |or ebx,edi                     ;  循环 n 次的到 ebx 的值  
00403C22   |.  40                   |inc eax                         ;  加 1  
00403C23   |.  83F8 20             |cmp eax,20                      ;  0x20 次  
00403C26   |.^ 7C DE               \\jl short 00403C06  
00403C28   |.  8BD3                 mov edx,ebx  
00403C2A   |.  81F2 918A4B3F         xor edx,3F4B8A91                      ;  结果⁰x3f4b8a91  
00403C30   |.  BE F84A5500          mov esi,00554AF8  
00403C35   |.  8DBD 2CFEFFFF        lea edi,dword ptr ss:[ebp-1D4]  
00403C3B   |.  B9 18000000          mov ecx,18  
00403C40   |.  F3:A5               rep movs dword ptr es:[edi],dword ptr ds:>  
00403C42   |.  66:C745 EC 0800     mov word ptr ss:[ebp-14],8  
00403C48   |.  33DB                 xor ebx,ebx  
00403C4A   |.  33C0                 xor eax,eax  
00403C4C   |.  8DB5 2CFEFFFF        lea esi,dword ptr ss:[ebp-1D4]  
00403C52   |>  8B0E                 /mov ecx,dword ptr ds:[esi]             ;  另一个固定的表.  
00403C54   |.  BF 01000000          |mov edi,1                         ;  同上  
00403C59   |.  D3E7                 |shl edi,cl                      ;  同上  
00403C5B   |.  85D7                 |test edi,edx                    ;  这个 edx 是经过上面运算的结果  
00403C5D   |.  0F97C1              |seta cl                       ;  同上  
```



```
00403C60   |.  83E1 01           |and ecx,1  

00403C63   |.  83C6 04           |add esi,4  

00403C66   |.  8BF9              |mov edi,ecx  

00403C68   |.  8BC8              |mov ecx,eax  

00403C6A   |.  D3E7              |shl edi,cl  

00403C6C   |.  0BDF              |or ebx,edi  

00403C6E   |.  40                |inc eax  

00403C6F   |.  83F8 18           |cmp eax,18                      

00403C72   |.^ 7C DE            \jl short 00403C52  

00403C74   |.  8BD3              mov edx,ebx                       ;  转移  

00403C76   |.  8B5D D8           mov ebx,dword ptr ss:\[ebp-28\]         ;  我们的 sn2 到现在才登场  

00403C79   |.  F7D3              not ebx                           ;  ~sn2  

00403C7B       C1EB 08           shr ebx,8                           ;  (~sn2)>>8  

00403C7E   |.  33C0              xor eax,eax  

00403C80   |.  8945 D4           mov dword ptr ss:\[ebp-2C\],eax  

00403C83   |.  8DBD CCFDFFFF      lea edi,dword ptr ss:\[ebp-234\]  

00403C89   |.  66:C745 EC 0800    mov word ptr ss:\[ebp-14\],8  

00403C8F   |.  BE 584B5500        mov esi,00554B58  

00403C94   |.  B9 18000000        mov ecx,18  

00403C99   |.  F3:A5             rep movs dword ptr es:\[edi\],dword ptr ds:>  

00403C9B   |.  8DB5 CCFDFFFF      lea esi,dword ptr ss:\[ebp-234\]  

00403CA1   |.  33C0              xor eax,eax  

00403CA3   |>  8B0E               /mov ecx,dword ptr ds:\[esi\]         ;这次除了表之外全和上次一样了  

00403CA5   |.  BF 01000000        |mov edi,1  

00403CAA   |.  D3E7               |shl edi,cl  

00403CAC   |.  85DF               |test edi,ebx  

00403CAE   |.  0F97C1             |seta cl  

00403CB1   |.  83E1 01            |and ecx,1  

00403CB4   |.  83C6 04            |add esi,4  

00403CB7   |.  8BF9               |mov edi,ecx  

00403CB9   |.  8BC8               |mov ecx,eax  

00403CBB   |.  D3E7               |shl edi,cl  

00403CBD   |.  097D D4            |or dword ptr ss:\[ebp-2C\],edi  

00403CC0   |.  40                 |inc eax  

00403CC1   |.  83F8 18            |cmp eax,18  

00403CC4   |.^ 7C DD             \jl short 00403CA3  

00403CC6   |.  8B5D D4           mov ebx,dword ptr ss:\[ebp-2C\]  

00403CC9   |.  3BDA              cmp ebx,edx                        ;比较 sn1 结果和 sn2 结果,经典!  

00403CCB   |.  75 23             jnz short 00403CF0  

00403CCD   |.  B8 01000000        mov eax,1  

00403CD2   |.  BA 02000000        mov edx,2  

00403CD7   |.  50                push eax  

00403CD8   |.  8D45 08            lea eax,dword ptr ss:\[ebp+8\]  

00403CDB   |.  FF4D F8            dec dword ptr ss:\[ebp-8\]  

00403CDE   |.  E8 05AF1300        call 0053EBE8  

00403CE3   |.  58                pop eax  

00403CE4   |.  8B55 DC            mov edx,dword ptr ss:\[ebp-24\]  

00403CE7   |.  64:8915 00000000   mov dword ptr fs:\[0\],edx  

00403CEE   |.  EB 1E              jmp short 00403D0E  

00403CF0   |>  33C0              xor eax,eax  

00403CF2   |.  BA 02000000        mov edx,2  

00403CF7   |.  50                push eax  

00403CF8   |.  8D45 08            lea eax,dword ptr ss:\[ebp+8\]  

00403CFB   |.  FF4D F8            dec dword ptr ss:\[ebp-8\]  

00403CFE   |.  E8 E5AE1300        call 0053EBE8  

00403D03   |.  58                pop eax  

00403D04   |.  8B55 DC            mov edx,dword ptr ss:\[ebp-24\]  

00403D07   |.  64:8915 00000000   mov dword ptr fs:\[0\],edx  

00403D0E   |>  5F                pop edi  

00403D0F   |.  5E                pop esi  

00403D10   |.  5B                pop ebx  

00403D11   |.  8BE5               mov esp,ebp  

00403D13   |.  5D                pop ebp  

00403D14   \\.  C3                ret  
```

我把下面的循环次数叫 `n`，加密后的 `sn1` 还叫 `sn1`，则 `sn2` 经过下面运算后应等于 `sn1`。

`sn2` 计算过程：

输入假的 `sn2`。

把 `sn2` 转换为 `int`。

`~sn2`。

`(~sn2)>>8`。

`T`＝一张固定的表。

`1<<T`。

`Test (1<<T),sn2` //结果和 `sn2` 做与运算(不返回结果,设置标志)，根据结果：`True`,`False`.进行下一步运算。

`True` 的话用 `1<<n`。

`False` 的话用 `0<<n` 相当于空操作；

用一个初始值为零的数(设为 `sn2A`)与上面结果做或运算;

循环 `0x18` 次。

得到 `sn2A`，要想注册成功，要使 `sn2A`＝`sn1`；

为了逆算出 `sn2`，我当初把这个循环拆为两部分：

1. 根据 `sn2A` 的值逆算出每次循环的 `Bool` 值
2. 根据 `Bool` 值逆算出 `sn2`

这两个问题似乎并不好解决，我试了各种方法，都没用，于是乎就现放下，等有灵感时再解决；

这样一放就是几个月..........

一天，闲着无聊时又试了一下，有了新发现----我真是傻的可以-\_\_-。

其实第一个问题很好解决，因为 `sn1` 的最后一次循环和 `sn2` 的循环是一样的，要想的到 `Bool` 值，直接记录 `sn2` 最后一次循环中的 `Bool` 值，化为 `Bool` 数组后为我所用就行了(我当初怎么没想到???)。

第二个问题:

```
00403C52   |>  8B0E               /mov ecx,dword ptr ds:\[esi\]        ;这张固定的表是从 0~0x17 乱序排列的  

00403C54   |.  BF 01000000        |mov edi,1                          

00403C59   |.  D3E7               |shl edi,cl                           

00403C5B   |.  85D7               |test edi,edx                         ;edx 就是要求的值
```

假设 `edx=0x1234`,把它化为二进制就是:`1001000110100b`。

把 `1` 右移 `k`(`k` 是每次循环取的表的值)再与 `edx` 做与运算,是为了检测 `edx` 二进制第 `k` 位是否为 `1`,是的话就是 `True`,否则 `False`;

例如:

`cl=3`

`1<<3=0x8`

二进制为:`1000b`

与 `0x1234` 做余运算:

`0x1234=1001000110100b`

`1<<3=               1000b`

这样就是 `False` 了。

既然我们已经得到了所需的 `Bool` 数组(由第一个问题解决的),就可以根据这个数组,遇到 `True` 就使加上(`1<<k`),遇到 `False` 什么也不做,循环 `0x18` 次。

就得到 `edx` 了;

怎么说不清楚~~~~还是看代码吧(c#):

```csharp
//省略部分系统初始化代码

private void button1_Click(object sender, System.EventArgs e)               
{  
  bool[] or=new bool [0x20];                     //这个就是我要用的 Bool 数组,下面是三张表:  
  int[] num1=  
{0x9,0x16,0x4,0x17,0x11,0xa,0xc,0x8,0x1b,0xf,0x7,0x5,0x1,0x1a,0x1f,0x1d,0x3,0x2,0xb,0x1c,0x10,0x12,0xd,0x0,0x1e,0x6,0x13,0x14  
,0x18,0x19,0x15,0xe};  

  int[] num2=  
{0x14,0x0,0x15,0x18,0x7,0x9,0x1c,0x8,0x2,0xb,0x1d,0x1b,0xc,0x4,0x5,0xe,0x6,0x11,0x17,0xa,0x13,0x16,0xd,0x1f};  

  int[] num3=  
{0xe,0x8,0x17,0x13,0x6,0x15,0x1,0x0,0x4,0x10,0xf,0x3,0x7,0x16,0x5,0xb,0x12,0xa,0xd,0x9,0x14,0x11,0xc,0x2}; //  

  int sn1=toint(textBox1.Text );                   //这个函数把 sn1 转换为 Int  
  int sn1num1=letgo(sn1);                          //LetGo!!就是 sn1 的初次运算  
  int sn1num2=xunhuan(sn1num1,num1,ref or);        //经过初次运算后的结果进入第一次循环  
  int sn1num3=xunhuan(sn1num2⁰x3F4B8A91,num2,ref or);  //稍加运算后进入第一次循环  
  int sn2=~((workout(num3,or))<<8);                 //根据结果算出 sn2.  
  string strsn2=Convert.ToString (sn2,16);           //仔细看一下由输入的 String 转换为 Int 的过程(地  
  址:00403D18~00403D8B)就明白,要想还原 sn2 为 String,直接转换 16 进制,长度不是偶数的,前面补零就可以了  
  if((strsn2.Length %2)!=0)  
  {  
    strsn2="0"+strsn2;  
  }                                            

  textBox2.Text =strsn2.ToUpper ();        //根据 sn1 计算出了 sn2,sn3 可随便写  
}      


public int toint(string input)                                           //把 sn1 转换为 Int (地址:00403D18~00403D8B) 
{  
  char[] charcode=input.ToUpper().ToCharArray ();  
  sbyte[] sbytecode=new sbyte [input.Length +1];        //要 movsx,所以我用 sbyte.  
  int esi=0;  
  for (int i=0;i<input.Length;i++)  
  {  
    sbytecode[i]=(sbyte)charcode[i];  
  }  
  for(int j=0;j<input.Length )  
  {  
       sbyte codetmp1=sbytecode[j];         

 sbyte codetmp2=sbytecode[j+1];  

if(codetmp1>0x46 | codetmp1<0x41)  

{  
  codetmp1=(sbyte)(codetmp1+0xd0);  

}  

else  

{  
  codetmp1=(sbyte)(codetmp1+0xc9);           

}  

if(codetmp2>0x46 | codetmp2<0x41)  

{  
  codetmp2=(sbyte)(codetmp2+0xd0);  

}  

else  

{  
  codetmp2=(sbyte)(codetmp2+0xc9);           

}   

  esi=(esi<<8)+((int)codetmp1<<4)+(int)codetmp2;                  

    j+=2;             

}  

      return esi;         

}  

public int letgo(int sn1)                                         

{       

  uint sn1tmp=(uint)sn1;  

  uint sn1tmp1,sn1tmp2;       

  sn1tmp1=((((((((sn1tmp+sn1tmp*4)<<3)-sn1tmp)<<3)-sn1tmp)>>1)+((sn1tmp+sn1tmp*4)>>3))+(sn1tmp/0x2c7)); 
```



```csharp
    sn1tmp2=sn1tmp1*sn1tmp;

    sn1=(int)((((((sn1tmp1+sn1tmp1*2)<<4)-sn1tmp1)<<2)-sn1tmp1+sn1tmp2));

    sn1=~sn1;

    return sn1;

}

public int xunhuan(int sn1,int[] numArr,ref bool[]or)         //循环

{

  int ebx=0;

  for(int i=0;i<numArr.Length ;i++)

  {

    if(Convert.ToBoolean((1<<numArr[i])&sn1)==true)

    {

      ebx|=(1<<i);

      or[i]=true;

    }

    else

    {

      or[i]=false;

    }

  }

  return ebx;

}

public int workout(int[] numArr,bool[] or)                         //计算 sn2 的 Int 值

{

  int sn2=0;

  for(int i=0;i<numArr.Length ;i++)

  {

    if(or[i]==true)

    {

      sn2=sn2+(1<<numArr[i]);

    }

  }

      return sn2;

}
```

`4dd466d0e1e0cc123afbe504da2462ca`
