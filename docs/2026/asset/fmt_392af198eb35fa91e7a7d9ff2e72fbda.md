
00402B6B  |.  FF15 7C8A4200    |call dword ptr ds:[<&MFC71.#4109\_ATL::CString>;  MFC71.7C188D88

00402B71  |.  50               |push eax

00402B72  |.  8D4C24 44        |lea ecx,dword ptr ss:[esp+44]

00402B76  |.  C64424 38 08     |mov byte ptr ss:[esp+38],8

00402B7B  |.  FF15 CC8A4200    |call dword ptr ds:[<&MFC71.#781\_ATL::CStringT>;  MFC71.7C150F15

00402B81  |.  8D4C24 44        |lea ecx,dword ptr ss:[esp+44]

00402B85  |.  FF15 FC8A4200    |call dword ptr ds:[<&MFC71.#578\_ATL::CStringT>;  MFC71.7C1771B1

00402B8B  |.  8D4C24 40        |lea ecx,dword ptr ss:[esp+40]

00402B8F  |.  FF15 F08A4200    |call dword ptr ds:[<&MFC71.#876\_ATL::CSimpleS>;  MFC71.7C158BCD

00402B95  |.  50               |push eax                                      ; /s

00402B96  |.  FFD5             |call ebp                                      ; \atol

00402B98  |.  83C4 04          |add esp,4

00402B9B  |.  8D4C24 40        |lea ecx,dword ptr ss:[esp+40]

00402B9F  |.  33F8             |xor edi,eax                                   ;  xor 上次计算结果

00402BA1  |.  885C24 34        |mov byte ptr ss:[esp+34],bl

00402BA5  |.  FF15 FC8A4200    |call dword ptr ds:[<&MFC71.#578\_ATL::CStringT>;  MFC71.7C1771B1

00402BAB  |.  8D4C24 14        |lea ecx,dword ptr ss:[esp+14]

00402BAF  |.  83C6 09          |add esi,9

00402BB2  |.  FF15 C48A4200    |call dword ptr ds:[<&MFC71.#2902\_ATL::CSimple>;  MFC71.7C146AB0

00402BB8  |.  3BF0             |cmp esi,eax

00402BBA  |.^ 7C 94            \jl short Lover.00402B50

00402BBC  |>  337C24 28        xor edi,dword ptr ss:[esp+28]                  ;  xor 用户名去前4位的值

00402BC0  |.  57               push edi

00402BC1  |.  8D4C24 14        lea ecx,dword ptr ss:[esp+14]                  ;  Email计算结果，记为SUM

00402BC5  |.  68 BC904200      push Lover.004290BC                            ;  ASCII "%d"

00402BCA  |.  51               push ecx

00402BCB  |.  FF15 888A4200    call dword ptr ds:[<&MFC71.#2322\_ATL::CStringT>;  MFC71.7C146A9D

00402BD1  |.  8B1D 688B4200    mov ebx,dword ptr ds:[<&MSVCR71.atoi>]         ;  MSVCR71.atoi

00402BD7  |.  83C4 0C          add esp,0C

00402BDA  |.  BD 01000000      mov ebp,1

00402BDF  |.  33FF             xor edi,edi

00402BE1  |>  8D4C24 10        /lea ecx,dword ptr ss:[esp+10]

00402BE5  |.  FF15 C48A4200    |call dword ptr ds:[<&MFC71.#2902\_ATL::CSimple>;  GetLength

00402BEB  |.  3BF8             |cmp edi,eax

00402BED  |.  7D 61            |jge short Lover.00402C50

00402BEF  |.  85ED             |test ebp,ebp                                  ;  区分奇偶位

00402BF1  |.  6A 01            |push 1

00402BF3  |.  8D4C24 14        |lea ecx,dword ptr ss:[esp+14]                 ;  计算结果

00402BF7  |.  57               |push edi

00402BF8  |.  74 2A            |je short Lover.00402C24

00402BFA  |.  8D5424 48        |lea edx,dword ptr ss:[esp+48]                 ;  保存注册码

00402BFE  |.  52               |push edx

00402BFF  |.  FF15 7C8A4200    |call dword ptr ds:[<&MFC71.#4109\_ATL::CString>;  MFC71.7C188D88

00402C05  |.  8BC8             |mov ecx,eax

00402C07  |.  FF15 F08A4200    |call dword ptr ds:[<&MFC71.#876\_ATL::CSimpleS>;  MFC71.7C158BCD

00402C0D  |.  50               |push eax                                      ; /s

00402C0E  |.  FFD3             |call ebx                                      ; \atoi

00402C10  |.  8BF0             |mov esi,eax                                   ;  分别取SUM[i]

00402C12  |.  83C4 04          |add esp,4

00402C15  |.  8D4C24 40        |lea ecx,dword ptr ss:[esp+40]

00402C19  |.  83C6 41          |add esi,41                                    ;  SUM[i]+41

00402C1C  |.  FF15 FC8A4200    |call dword ptr ds:[<&MFC71.#578\_ATL::CStringT>;  MFC71.7C1771B1

00402C22  |.  EB 38            |jmp short Lover.00402C5C

00402C24  |>  8D4424 4C        |lea eax,dword ptr ss:[esp+4C]

00402C28  |.  50               |push eax

00402C29  |.  FF15 7C8A4200    |call dword ptr ds:[<&MFC71.#4109\_ATL::CString>;  MFC71.7C188D88

00402C2F  |.  8BC8             |mov ecx,eax

00402C31  |.  FF15 F08A4200    |call dword ptr ds:[<&MFC71.#876\_ATL::CSimpleS>;  MFC71.7C158BCD

00402C37  |.  50               |push eax                                      ; /s

00402C38  |.  FFD3             |call ebx                                      ; \atoi

00402C3A  |.  BE 5A000000      |mov esi,5A                                    ;  $5A

00402C3F  |.  83C4 04          |add esp,4

00402C42  |.  8D4C24 44        |lea ecx,dword ptr ss:[esp+44]

00402C46  |.  2BF0             |sub esi,eax                                   ;  $5A-SUM[i]

00402C48  |.  FF15 FC8A4200    |call dword ptr ds:[<&MFC71.#578\_ATL::CStringT>;  MFC71.7C1771B1

00402C4E  |.  EB 0C            |jmp short Lover.00402C5C

00402C50  |>  8BF5             |mov esi,ebp

00402C52  |.  F7DE             |neg esi                                       ;  求补指令

00402C54  |.  1BF6             |sbb esi,esi

00402C56  |.  83E6 E7          |and esi,FFFFFFE7

00402C59  |.  83C6 5A          |add esi,5A

00402C5C  |>  56               |push esi

00402C5D  |.  8D4C24 24        |lea ecx,dword ptr ss:[esp+24]

00402C61  |.  FF15 808A4200    |call dword ptr ds:[<&MFC71.#908\_ATL::CStringT>;  MFC71.7C18B24E

00402C67  |.  83FF 03          |cmp edi,3

00402C6A  |.  75 0F            |jnz short Lover.00402C7B

00402C6C  |.  68 B8904200      |push Lover.004290B8

00402C71  |.  8D4C24 24        |lea ecx,dword ptr ss:[esp+24]

00402C75  |.  FF15 AC8A4200    |call dword ptr ds:[<&MFC71.#911\_ATL::CStringT>;  MFC71.7C14E587

00402C7B  |>  33C9             |xor ecx,ecx

00402C7D  |.  85ED             |test ebp,ebp

00402C7F  |.  0F94C1           |sete cl

00402C82  |.  47               |inc edi

00402C83  |.  83FF 09          |cmp edi,9                                     ;  \*注册码位数\*

00402C86  |.  8BE9             |mov ebp,ecx

00402C88  |.^ 0F8C 53FFFFFF    \jl Lover.00402BE1                             ;  循环产生注册码

00402C8E  |.  8B7424 3C        mov esi,dword ptr ss:[esp+3C]

00402C92  |.  8D5424 20        lea edx,dword ptr ss:[esp+20]

00402C96  |.  52               push edx

00402C97  |.  8BCE             mov ecx,esi

00402C99  |.  FF15 B08A4200    call dword ptr ds:[<&MFC71.#297\_ATL::CStringT<>;  MFC71.7C14E575

00402C9F  |.  8D4C24 10        lea ecx,dword ptr ss:[esp+10]

00402CA3  |.  FF15 FC8A4200    call dword ptr ds:[<&MFC71.#578\_ATL::CStringT<>;  MFC71.7C1771B1

00402CA9  |.  8D4C24 24        lea ecx,dword ptr ss:[esp+24]

00402CAD  |.  FF15 FC8A4200    call dword ptr ds:[<&MFC71.#578\_ATL::CStringT<>;  MFC71.7C1771B1

00402CB3  |.  8D4C24 14        lea ecx,dword ptr ss:[esp+14]

00402CB7  |.  FF15 FC8A4200    call dword ptr ds:[<&MFC71.#578\_ATL::CStringT<>;  MFC71.7C1771B1

00402CBD  |.  8D4C24 18        lea ecx,dword ptr ss:[esp+18]

00402CC1  |.  FF15 FC8A4200    call dword ptr ds:[<&MFC71.#578\_ATL::CStringT<>;  MFC71.7C1771B1

00402CC7  |.  8D4C24 1C        lea ecx,dword ptr ss:[esp+1C]

00402CCB  |.  FF15 FC8A4200    call dword ptr ds:[<&MFC71.#578\_ATL::CStringT<>;  MFC71.7C1771B1

00402CD1  |.  8D4C24 20        lea ecx,dword ptr ss:[esp+20]

00402CD5  |.  FF15 FC8A4200    call dword ptr ds:[<&MFC71.#578\_ATL::CStringT<>;  MFC71.7C1771B1

00402CDB  |.  8B4C24 2C        mov ecx,dword ptr ss:[esp+2C]

00402CDF  |.  5F               pop edi

00402CE0  |.  8BC6             mov eax,esi

00402CE2  |.  5E               pop esi

00402CE3  |.  5D               pop ebp

00402CE4  |.  5B               pop ebx

00402CE5  |.  64:890D 00000000 mov dword ptr fs:[0],ecx

00402CEC  |.  83C4 28          add esp,28

00402CEF  \.  C2 0C00          retn 0C

------------------------------------------------------------------------------------------------------

这里可是产生注册码的地方哦!

### 【算法总结】

1、取用户名后面部分(去除前4位)，转换为长整型数`Long`

2、循环将`Email`转换为10进制字符串，记为`Estr`

3、将`Estr`以长度9为单位，分别转换成`long`，并相互`XOR`，计算结果 `XOR` 用户名计算结果，记为`SUM`

4、`i＝1 to 9`
  
  假如`Sun`长度>=9 ,注册码奇数位=`Sum[i]+$41`,偶数位=`$5A-Sum[i]`

  假如`Sun`长度<9，注册码奇数位='A',偶数位='Z'

5、注册码第5位固定为'-'。

`Delphi 7.0` 注册机源代码表示如下:
