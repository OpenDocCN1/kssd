
# 脱2层未知变形壳 ―― `philips.exe`

脱2层未知变形壳 ―― `philips.exe`

相关页面：  http://tongtian.net/pediybbs/viewtopic.php?p=9289#

软件大小：  270 KB

【软件简介】：Philips Az@lis/Ozeo/Xenium/    Fisio Service Tool - V2.3

【作者声明】：初学Crack，只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教！

【调试环境】：Win98、Ollydbg1.09、PEiD、C32ASM、ImportREC

―――――――――――――――――――――――――――――――――

【脱壳过程】：

因为原程序只能在Win98下运行，所以只好在98下脱壳了。不清楚所加的壳名字。:-(

用Ollydbg手动脱壳,老规矩：载入后弹出“是压缩代码――要继续进行分析吗？”，点“否”。

```asm
00925000     60                   pushad

                                  ====>进入OD后断在这！

00925001     F4                   hlt

00925002     E9 3D040000          jmp PHILIPS.00925444

00925444     BB 136BB6DF          mov ebx,DFB66B13

00925449     66:8BEF              mov bp,di

0092544C     66:8BF0              mov si,ax

0092544F     E8 14000000          call PHILIPS.00925468

00925454     A7                   cmps dword ptr ds:[esi],dword ptr es:[edi]

00925455     15 FF5841B0          adc eax,B04158FF

0092545A     808D 73BADA4A A9     or byte ptr ss:[ebp+4ADABA73],0A9

00925461     FC                   cld

00925462     5B                   pop ebx

00925463     B8 0F3E0CF6          mov eax,F60C3E0F

00925468     80CA A1              or dl,0A1

0092546B     E9 14000000          jmp PHILIPS.00925484

00925484     E9 1C000000          jmp PHILIPS.009254A5

009254A5     5F                   pop edi

009254A6     E9 1C000000          jmp PHILIPS.009254C7

009254C7     0FBFCE               movsx ecx,si

009254CA     0FBFD1               movsx edx,cx

009254CD     66:8BE8              mov bp,ax

009254D0     81C7 2AB51AAC        add edi,AC1AB52A

009254D6     E9 14000000          jmp PHILIPS.009254EF

009254EF     66:8BF0              mov si,ax

009254F2     B5 A9                mov ch,0A9

009254F4     81E5 E9CD3E23        and ebp,233ECDE9

009254FA     B9 C326B174          mov ecx,74B126C3

009254FF     80B7 C14AE553 31     xor byte ptr ds:[edi+53E54AC1],31

00925506     81E5 A340ECBC        and ebp,BCEC40A3

0092550C     0FBFCD               movsx ecx,bp

0092550F     0FBFEE               movsx ebp,si

00925512     43                   inc ebx

00925513     BA 3F579FDC          mov edx,DC9F573F

00925518     E9 1C000000          jmp PHILIPS.00925539

00925539     81EF 01000000        sub edi,1

0092553F     E9 1C000000          jmp PHILIPS.00925560

00925560     81D9 B13BC93A        sbb ecx,3AC93BB1

00925566     8BF3                 mov esi,ebx

00925568     66:8BEA              mov bp,dx

0092556B     E9 1C000000          jmp PHILIPS.0092558C

0092558C     66:8BEB              mov bp,bx

0092558F     81FE 4C6FB6DF        cmp esi,DFB66F4C

00925595     0F85 5FFFFFFF        jnz PHILIPS.009254FA

                                  ====>F4下去

0092559B     80E9 33              sub cl,33

0092559E     E9 98FEFFFF          jmp PHILIPS.0092543B

0092543B     E9 C7FBFFFF          jmp PHILIPS.00925007

00925007     E8 24040000          call PHILIPS.00925430

0092500C     EB 00                jmp short PHILIPS.0092500E

0092500E     BB 30394400          mov ebx,PHILIPS.00443930

00925013     03DD                 add ebx,ebp

00925015     2B9D D03F4400        sub ebx,dword ptr ss:[ebp+443FD0]

0092501B     83BD FC494400 00     cmp dword ptr ss:[ebp+4449FC],0

00925022     899D FC494400        mov dword ptr ss:[ebp+4449FC],ebx

00925028     0F85 66030000        jnz PHILIPS.00925394

0092502E     C785 33394400 000000>mov dword ptr ss:[ebp+443933],0

00925038     8D85 044A4400        lea eax,dword ptr ss:[ebp+444A04]

0092503E     50                   push eax

0092503F     FF95 004B4400        call dword ptr ss:[ebp+444B00]

00925045     8985 004A4400        mov dword ptr ss:[ebp+444A00],eax

0092504B     8BF8                 mov edi,eax

0092504D     8D9D 114A4400        lea ebx,dword ptr ss:[ebp+444A11]

00925053     53                   push ebx

00925054     50                   push eax

00925055     FF95 FC4A4400        call dword ptr ss:[ebp+444AFC]

0092505B     8985 FC3F4400        mov dword ptr ss:[ebp+443FFC],eax

00925061     8D9D 1E4A4400        lea ebx,dword ptr ss:[ebp+444A1E]

00925067     53                   push ebx

00925068     57                   push edi

00925069     FF95 FC4A4400        call dword ptr ss:[ebp+444AFC]

0092506F     8985 00404400        mov dword ptr ss:[ebp+444000],eax

00925075     8D85 B5394400        lea eax,dword ptr ss:[ebp+4439B5]

0092507B     FFE0                 jmp eax                         ; PHILIPS.00925085

00925085     8B9D D83F4400        mov ebx,dword ptr ss:[ebp+443FD8]

0092508B     0BDB                 or ebx,ebx

0092508D     74 0A                je short PHILIPS.00925099

00925099     8DB5 19404400        lea esi,dword ptr ss:[ebp+444019]

0092509F     833E 00              cmp dword ptr ds:[esi],0

009250A2     0F84 1F010000        je PHILIPS.009251C7

009250A8     8DB5 19404400        lea esi,dword ptr ss:[ebp+444019]

009250AE     6A 04                push 4

009250B0     68 00100000          push 1000

009250B5     68 00180000          push 1800

009250BA     6A 00                push 0

009250BC     FF95 FC3F4400        call dword ptr ss:[ebp+443FFC]

009250C2     8985 F83F4400        mov dword ptr ss:[ebp+443FF8],eax

009250C8     8B46 04              mov eax,dword ptr ds:[esi+4]

009250CB     05 0E010000          add eax,10E

009250D0     6A 04                push 4

009250D2     68 00100000          push 1000

009250D7     50                   push eax

009250D8     6A 00                push 0

009250DA     FF95 FC3F4400        call dword ptr ss:[ebp+443FFC]

009250E0     8985 F43F4400        mov dword ptr ss:[ebp+443FF4],eax

009250E6     56                   push esi

009250E7     8B1E                 mov ebx,dword ptr ds:[esi]

009250E9     039D FC494400        add ebx,dword ptr ss:[ebp+4449FC]

009250EF     FFB5 F83F4400        push dword ptr ss:[ebp+443FF8]

009250F5     FF76 04              push dword ptr ds:[esi+4]

009250F8     50                   push eax

009250F9     53                   push ebx

009250FA     E8 DA060000          call PHILIPS.009257D9

009250FF     80BD 10404400 00     cmp byte ptr ss:[ebp+444010],0

00925106     75 5E                jnz short PHILIPS.00925166

00925108     FE85 10404400        inc byte ptr ss:[ebp+444010]

0092510E     8B3E                 mov edi,dword ptr ds:[esi]

00925110     03BD FC494400        add edi,dword ptr ss:[ebp+4449FC]

00925116     FF37                 push dword ptr ds:[edi]

00925118     C607 C3              mov byte ptr ds:[edi],0C3

0092511B     FFD7                 call edi

0092511D     8F07                 pop dword ptr ds:[edi]

0092511F     50                   push eax

00925120     51                   push ecx

00925121     56                   push esi

00925122     53                   push ebx

00925123     8BC8                 mov ecx,eax

00925125     83E9 06              sub ecx,6

00925128     8BB5 F43F4400        mov esi,dword ptr ss:[ebp+443FF4]

0092512E     33DB                 xor ebx,ebx

00925130     0BC9                 or ecx,ecx

00925132     74 2E                je short PHILIPS.00925162

00925136     AC                   lods byte ptr ds:[esi]

00925137     3C E8                cmp al,0E8

00925139     74 0A                je short PHILIPS.00925145

0092513B     EB 00                jmp short PHILIPS.0092513D

0092513D     3C E9                cmp al,0E9

0092513F     74 04                je short PHILIPS.00925145

00925141     43                   inc ebx

00925142     49                   dec ecx

00925143     EB EB                jmp short PHILIPS.00925130

00925145     8B06                 mov eax,dword ptr ds:[esi]

00925147     EB 0A                jmp short PHILIPS.00925153

00925149     803E 0D              cmp byte ptr ds:[esi],0D

0092514C     75 F3                jnz short PHILIPS.00925141

0092514E     24 00                and al,0

00925150     C1C0 18              rol eax,18

00925153     2BC3                 sub eax,ebx
```
