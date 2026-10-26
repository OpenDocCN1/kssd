
```
00384523    83F8 0D             cmp eax,0D              //第十三个45349e

  call <jmp.&user32.InflateRect>

00384526    75 0F               jnz short 00384537

00384528    E9 CD0B0000         jmp 003850FA

0038452D    EB 03               jmp short 00384532

0038452F    CD20 E9E9E302       vxdjump 2E3E9E9

00384535    0000                add byte ptr ds:[eax],al

00384537    83F8 0E             cmp eax,0E     //第十四个 47173E=<jmp.&user32.IntersectRect>

0038453A    75 21               jnz short 0038455D

0038453C    E9 DD0B0000         jmp 0038511E

00384541    E8 04000000         call 0038454A

00384546    C783 EB0E58EB 02CD2>mov dword ptr ds:[ebx+EB580EEB],8320C>

00384550    C002 EB             rol byte ptr ds:[edx],0EB

00384553    01E9                add ecx,ebp

00384555    50                  push eax

00384556    C3                  retn

00384557    C7                  ???                                   ; 未知命令

00384558    E9 BD020000         jmp 0038481A

0038455D    83F8 0F             cmp eax,0F    //第十五个 452DAE=<jmp.&user32.IsRectEmpty>

00384560    75 11               jnz short 00384573

00384562    E9 7D0C0000         jmp 003851E4

00384567    7C 03               jl short 0038456C

00384569    EB 03               jmp short 0038456E

0038456B    0F74FB              pcmpeqb mm7,mm3

0038456E    E9 A7020000         jmp 0038481A

00384573    83F8 10             cmp eax,10     //第十六个456FBE=<jmp.&kernel32.LoadLibraryA>

00384576    0F85 E4000000       jnz 00384660

0038457C    50                  push eax

0038457D    60                  pushad

0038457E    E8 00000000         call 00384583

00384583    5D                  pop ebp

00384584    81ED D6334100       sub ebp,4133D6

0038458A    8B4424 28           mov eax,dword ptr ss:[esp+28]

0038458E    50                  push eax

0038458F    8D85 43454100       lea eax,dword ptr ss:[ebp+414543]

00384595    50                  push eax

00384596    FF95 8F434100       call dword ptr ss:[ebp+41438F]

0038459C    0BC0                or eax,eax

0038459E    75 0D               jnz short 003845AD

003845A0    C74424 20 78787878  mov dword ptr ss:[esp+20],78787878

003845A8    61                  popad

003845A9    58                  pop eax

003845AA    C2 0400             retn 4

003845AD    8B4424 28           mov eax,dword ptr ss:[esp+28]

003845B1    66:8B08             mov cx,word ptr ds:[eax]

003845B4    66:83F9 01          cmp cx,1

003845B8    75 1E               jnz short 003845D8

003845BA    8B85 D4414100       mov eax,dword ptr ss:[ebp+4141D4]

003845C0    DD00                fld qword ptr ds:[eax]

003845C2    8385 D4414100 08    add dword ptr ss:[ebp+4141D4],8

003845C9    83C0 08             add eax,8

003845CC    DC00                fadd qword ptr ds:[eax]

003845CE    DD18                fstp qword ptr ds:[eax]

003845D0    9B                  wait

003845D1    61                  popad

003845D2    58                  pop eax

003845D3    C2 0400             retn 4

003845D6    EB 66               jmp short 0038463E

003845D8    66:83F9 02          cmp cx,2

003845DC    75 1C               jnz short 003845FA

003845DE    8385 D4414100 08    add dword ptr ss:[ebp+4141D4],8

003845E5    8B85 D4414100       mov eax,dword ptr ss:[ebp+4141D4]

003845EB    DD00                fld qword ptr ds:[eax]

003845ED    DC60 F8             fsub qword ptr ds:[eax-8]

003845F0    DD18                fstp qword ptr ds:[eax]

003845F2    9B                  wait

003845F3    61                  popad

003845F4    58                  pop eax

003845F5    C2 0400             retn 4

003845F8    EB 44               jmp short 0038463E

003845FA    66:83F9 03          cmp cx,3

003845FE    75 1E               jnz short 0038461E

00384600    8B85 D4414100       mov eax,dword ptr ss:[ebp+4141D4]

00384606    DD00                fld qword ptr ds:[eax]

00384608    8385 D4414100 08    add dword ptr ss:[ebp+4141D4],8

0038460F    83C0 08             add eax,8

00384612    DC08                fmul qword ptr ds:[eax]

00384614    DD18                fstp qword ptr ds:[eax]

00384616    9B                  wait

00384617    61                  popad

00384618    58                  pop eax

00384619    C2 0400             retn 4

0038461C    EB 20               jmp short 0038463E

0038461E    66:83F9 04          cmp cx,4

00384622    75 1A               jnz short 0038463E

00384624    8385 D4414100 08    add dword ptr ss:[ebp+4141D4],8

0038462B    8B85 D4414100       mov eax,dword ptr ss:[ebp+4141D4]

00384631    DD00                fld qword ptr ds:[eax]

00384633    DC70 F8             fdiv qword ptr ds:[eax-8]

00384636    DD18                fstp qword ptr ds:[eax]

00384638    9B                  wait

00384639    61                  popad

0038463A    58                  pop eax

0038463B    C2 0400             retn 4

0038463E    8B85 34414100       mov eax,dword ptr ss:[ebp+414134]

00384644    894424 20           mov dword ptr ss:[esp+20],eax

00384648    61                  popad

00384649    83C4 04             add esp,4

0038464C    FF6424 FC           jmp dword ptr ss:[esp-4]

00384650    E8 03000000         call 00384658

00384655    CD20 D683C404       vxdjump 4C483D6

0038465B    E9 BA010000         jmp 0038481A

00384660    83F8 11             cmp eax,11       //第十七个 427741=<jmp.&kernel32.MulDiv>

00384663    75 27               jnz short 0038468C

00384665    8BCC                mov ecx,esp

00384667    FF71 0C             push dword ptr ds:[ecx+C]

0038466A    FF71 08             push dword ptr ds:[ecx+8]

0038466D    FF71 04             push dword ptr ds:[ecx+4]

00384670    E8 200B0000         call 00385195

00384675    83C4 10             add esp,10

00384678    FF6424 F0           jmp dword ptr ss:[esp-10]

0038467C    E8 03000000         call 00384684

00384681    CD20 D683C404       vxdjump 4C483D6

00384687    E9 8E010000         jmp 0038481A

0038468C    83F8 12             cmp eax,12  //第十八个 44FFCA=<jmp.&user32.PtInRect>

0038468F    75 0F               jnz short 003846A0

00384691    E9 C20B0000         jmp 00385258

00384696    EB 03               jmp short 0038469B

00384698    90                  nop

00384699    90                  nop

0038469A    90                  nop

0038469B    E9 7A010000         jmp 0038481A

003846A0    83F8 13             cmp eax,13       //第十九个 474B62=<jmp.&user32.OffsetRect>

003846A3    75 1E               jnz short 003846C3

003846A5    E9 E00B0000         jmp 0038528A

003846AA    E8 04000000         call 003846B3

003846AF    0FEB0CC9            por mm1,qword ptr ds:[ecx+ecx*8]

003846B3    58                  pop eax

003846B4    EB 01               jmp short 003846B7

003846B6    0F40EB              cmovo ebp,ebx

003846B9    01CD                add ebp,ecx

003846BB    FFE0                jmp eax

003846BD    C7                  ???                                   ; 未知命令

003846BE    E9 57010000         jmp 0038481A

003846C3    83F8 14             cmp eax,14               //第二十个 (user32.SendMessageA)

003846C6    75 33               jnz short 003846FB

003846C8    50                  push eax

003846C9    60                  pushad

003846CA    E8 00000000         call 003846CF

003846CF    5D                  pop ebp

003846D0    81ED 22354100       sub ebp,413522

003846D6    8BFC                mov edi,esp

003846D8    FF77 34             push dword ptr ds:[edi+34]

003846DB    FF77 30             push dword ptr ds:[edi+30]

003846DE    FF77 2C             push dword ptr ds:[edi+2C]

003846E1    FF77 28             push dword ptr ds:[edi+28]

003846E4    FF95 66424100       call dword ptr ss:[ebp+414266]  //ss:[00385413]=77D1702F

(user32.SendMessageA)

003846EA    8947 20             mov dword ptr ds:[edi+20],eax

003846ED    61                  popad

003846EE    58                  pop eax

003846EF    83C4 14             add esp,14

003846F2    FF6424 EC           jmp dword ptr ss:[esp-14]
```
