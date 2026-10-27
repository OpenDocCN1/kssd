
```
004AC320   |.  897424 14   mov dword ptr ss:[esp+14],esi    ;  n20放入内存地址0012ebfc"3F485550"

004AC324   |.  C1E3 10     shl ebx,10                       ;  n22左移10次记为n24

004AC327   |.  0BCB        or ecx,ebx                       ;  n23与n24作或运算记为n25

004AC329   |.  40          inc eax

004AC32A   |.  33DB        xor ebx,ebx                      ;  清空ebx

004AC32C   |.  8A38        mov bh,byte ptr ds:[eax]         ;  A1的第十一位0入bh记为n26

004AC32E   |.  0BCB        or ecx,ebx                       ;  n25与n26作或运算记为n27

004AC330   |.  40          inc eax

004AC331   |.  33DB        xor ebx,ebx                      ;  清空ebx

004AC333   |.  8A18        mov bl,byte ptr ds:[eax]         ;  A1的第十二位0入bl记为n28

004AC335   |.  0BCB        or ecx,ebx                       ;  n27与n28作或运算记为n29

004AC337   |.  8B5D 40     mov ebx,dword ptr ss:[ebp+40]    ;  字符“92iO"入ebx记为b3"4F693239"

004AC33A   |.  40          inc eax

004AC33B   |.  33CB        xor ecx,ebx                      ;  N29与b3作异或运算记为n30"4F693239"

004AC33D   |.  33DB        xor ebx,ebx                      ;  清空ebx

004AC33F   |.  894C24 1C   mov dword ptr ss:[esp+1C],ecx    ;  n30放入内存地址0012ec04"4F693239"

004AC343   |.  8A18        mov bl,byte ptr ds:[eax]         ;  A1的第十三位0入bl记为n31

004AC345   |.  8BF3        mov esi,ebx                      ;  n31入esi

004AC347   |.  33DB        xor ebx,ebx                      ;  清空ebx

004AC349   |.  8A58 01     mov bl,byte ptr ds:[eax+1]       ;  A1的第十四位0入bl记为n32

004AC34C   |.  C1E6 18     shl esi,18                       ;  n31左移18次记为n33

004AC34F   |.  40          inc eax

004AC350   |.  C1E3 10     shl ebx,10                       ;  n32左移10次记为n34

004AC353   |.  0BF3        or esi,ebx                       ;  n33与n34作或运算记为n35

004AC355   |.  40          inc eax

004AC356   |.  33DB        xor ebx,ebx                      ;  清空ebx

004AC358   |.  8A38        mov bh,byte ptr ds:[eax]         ;  A1的第十五位0入bh记为n36

004AC35A   |.  8A58 01     mov bl,byte ptr ds:[eax+1]       ;  A1的第十六位0入bl记为n37

004AC35D   |.  8B45 44     mov eax,dword ptr ss:[ebp+44]    ;  字符”4873“放入eax记为b4"33373834"

004AC360   |.  0BDE        or ebx,esi                       ;  ebx与n35作或运算记为n38

004AC362   |.  8BB5 D80300>mov esi,dword ptr ss:[ebp+3D8]   ;  字符"E"入esi记为n39

004AC368   |.  33C3        xor eax,ebx                      ;  b4与n38作异或运算记为n40"33373834"

004AC36A   |.  897424 20   mov dword ptr ss:[esp+20],esi    ;  n39放入内存地址0012ec08"E"

004AC36E   |.  894424 10   mov dword ptr ss:[esp+10],eax    ;  n40放入内存地址0012ebf8"33373834"

004AC372   |.  83FE 01     cmp esi,1

004AC375   |.  0F8E 350100>jle 1.004AC4B0

004AC37B   |.  83C5 60     add ebp,60

004AC37E   |.  4E          dec esi                          ;  n39减一记为n41

004AC37F   |.  897424 64   mov dword ptr ss:[esp+64],esi    ;  n41放入内存地址0012ec4c"D"

004AC383   |>  33DB        /xor ebx,ebx                     ;  清空ebx

004AC385   |.  83C5 20     |add ebp,20

004AC388   |.  8ADC        |mov bl,ah                       ;  n40的第二位放入bl记为n42"38"

004AC38A   |.  33C0        |xor eax,eax                     ;  清空eax

004AC38C   |.  8A4424 1E   |mov al,byte ptr ss:[esp+1E]     ;  n30的第二位”i“入al记为n43"69"

004AC390   |.  8B349D 4867>|mov esi,dword ptr ds:[ebx*4+526>;  n42*4+526748查表得"07090E07"记为n44入esi

004AC397   |.  8B3C85 4863>|mov edi,dword ptr ds:[eax*4+526>;  n30*4+526348查表得"10E9F9F9"记为n45入edi

004AC39E   |.  33C0        |xor eax,eax                     ;  清空eax

004AC3A0   |.  8A4424 17   |mov al,byte ptr ss:[esp+17]     ;  n20的第一位”？“入al记为n46"3f"

004AC3A4   |.  33F7        |xor esi,edi                     ;  n44与n45作异或运算记为n47”17E0F7FE“

004AC3A6   |.  8B7D DC     |mov edi,dword ptr ss:[ebp-24]   ;  b2入edi"4D485550"

004AC3A9   |.  8B1C85 485F>|mov ebx,dword ptr ds:[eax*4+525>;  n20*4+525f48查表得"EA75759F"记为n48入ebx

004AC3B0   |.  8BC2        |mov eax,edx                     ;  n9入eax

004AC3B2   |.  25 FF000000 |and eax,0FF                     ;  n9与0FF作与运算记为n49"00000028"

004AC3B7   |.  33F3        |xor esi,ebx                     ;  n47与n48作异或运算记为n50”FD958261“

004AC3B9   |.  8B1C85 486B>|mov ebx,dword ptr ds:[eax*4+526>;  n49*4+526b48查表得”34345C68“记为n51

004AC3C0   |.  33C0        |xor eax,eax                     ;  清空eax

004AC3C2   |.  8A4424 12   |mov al,byte ptr ss:[esp+12]     ;  n40的第二位”7“入al记为n52

004AC3C6   |.  33F3        |xor esi,ebx                     ;  n50与n51作异或运算记为n53”C9A1DE09“

004AC3C8   |.  33DB        |xor ebx,ebx                     ;  清空ebx

004AC3CA   |.  33F7        |xor esi,edi                     ;  n53与b2作异或运算记为n54”84E98B59“

004AC3CC   |.  8A5C24 1F   |mov bl,byte ptr ss:[esp+1F]     ;  B3的第四位”O“入bl记为n55

004AC3D0   |.  8B3C85 4863>|mov edi,dword ptr ds:[eax*4+526>;  n52*4+526348查表得”"B52F9A9A"记为n56入edi

004AC3D7   |.  8B049D 485F>|mov eax,dword ptr ds:[ebx*4+525>;  n55*4+525f48查表得“13848487”记为n57入eax

004AC3DE   |.  33F8        |xor edi,eax                     ;  n57与n56作异或运算记为n58"A6AB1E0D"

004AC3E0   |.  33C0        |xor eax,eax                     ;  清空eax

004AC3E2   |.  8AC6        |mov al,dh                       ;  n10的第二位“）”入al记为n58

004AC3E4   |.  8B1485 4867>|mov edx,dword ptr ds:[eax*4+526>;  n58*4+526748查表得“A5F451A5“记为n59入edx

004AC3EB <>|.  8B4424 14   |mov eax,dword ptr ss:[esp+14]   ;  n20入eax

004AC3EF   |.  33FA        |xor edi,edx                     ;  n58与n59作异或运算记为n60”035F4FA8“

004AC3F1   |.  8BD0        |mov edx,eax                     ;  N20入edx

004AC3F3   |.  81E2 FF0000>|and edx,0FF                     ;  n20与0FF作与运算记为n61"00000050"

004AC3F9   |.  8B1C95 486B>|mov ebx,dword ptr ds:[edx*4+526>;  n61*4+526b48查表得”5353F5A6“记为N62入ebx

004AC400   |.  33D2        |xor edx,edx                     ;  清空edx

004AC402   |.  8A5424 13   |mov dl,byte ptr ss:[esp+13]     ;  n40的第四位”3“入dl记为n63

004AC406   |.  33FB        |xor edi,ebx                     ;  n60与n62作异或运算记为n64"500CBA0E"

004AC408   |.  8B5D E0     |mov ebx,dword ptr ss:[ebp-20]   ;  n30入ebx

004AC40B   |.  33FB        |xor edi,ebx                     ;  n64与n30作异或运算记为n65"1F658837"

004AC40D   |.  33DB        |xor ebx,ebx                     ;  清空ebx

004AC40F   |.  8ADC        |mov bl,ah                       ;  n20的第二位”U"入bl记为n66

004AC411   |.  8B0495 485F>|mov eax,dword ptr ds:[edx*4+525>;  n63*4+525f48查表得“9DC3C35E”记为n67入eax

004AC418   |.  8B149D 4867>|mov edx,dword ptr ds:[ebx*4+526>;  n66*4+526748查表得“FC1FE3FC”记为n68入edx

004AC41F   |.  33C2        |xor eax,edx                     ;  n67与n68作异或运算记为n69“61DC20A2”

004AC421   |.  33D2        |xor edx,edx                     ;  清空edx

004AC423   |.  8A5424 1A   |mov dl,byte ptr ss:[esp+1A]     ;  n10的第二位“07”入dl记为n70

004AC427   |.  8B1C95 4863>|mov ebx,dword ptr ds:[edx*4+526>;  n70*4+526348查表得“5491C5C5”记为n71入ebx

004AC42E   |.  8BD1        |mov edx,ecx                     ;  n30入edx

004AC430   |.  81E2 FF0000>|and edx,0FF                     ;  n30与OFF作与运算记为n72"39"

004AC436   |.  33C3        |xor eax,ebx                     ;  n69与n71作异或运算记为n73"354DE567"

004AC438   |.  8B1C95 486B>|mov ebx,dword ptr ds:[edx*4+526>;  n72*4+526b48查表得“12123624”记为n74入ebx

004AC43F   |.  8B55 E4     |mov edx,dword ptr ss:[ebp-1C]   ;  n40入edx

004AC442   |.  33C3        |xor eax,ebx                     ;  n73与n74作异或运算记为n75“275FD343”

004AC444   |.  33C2        |xor eax,edx                     ;  n75与N40作异或运算记为n76"1468EB77“

004AC446   |.  33D2        |xor edx,edx                     ;  清空edx

004AC448   |.  8AD5        |mov dl,ch                       ;  n30的第二位”2“入dl记为n77

004AC44A   |.  33C9        |xor ecx,ecx                     ;  清空ecx

004AC44C   |.  8A4C24 16   |mov cl,byte ptr ss:[esp+16]     ;  n20的第二位”H“入cl记为n78

004AC450   |.  897424 14   |mov dword ptr ss:[esp+14],esi   ;  n54入内存地址0012ebfc"84e98b59"

004AC454   |.  8B1495 4867>|mov edx,dword ptr ds:[edx*4+526>;  n77*4+526748查表得”23654623“记为n79入edx

004AC45B   |.  8B7424 64   |mov esi,dword ptr ss:[esp+64]   ;  n41入esi

004AC45F   |.  8B1C8D 4863>|mov ebx,dword ptr ds:[ecx*4+526>;  n78*4+526348查表得”F6A45252“记为n80入ebx
```
