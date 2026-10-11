
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

0038468C    83F8 12             cmp eax,12   //第十八个 44FFCA=<jmp.&user32.PtInRect>  

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

003846BD    C7                  ???                             ; 未知命令  

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

003846F6    E9 1F010000         jmp 0038481A  

003846FB    83F8 15             cmp eax,15               //第二十一个 没有用到  

003846FE    75 1E               jnz short 0038471E  

00384700    E9 050B0000         jmp 0038520A  

00384705    E8 04000000         call 0038470E  

0038470A    0FEB0CC9            por mm1,qword ptr ds:[ecx+ecx*8]  

0038470E    58                  pop eax  

0038470F    EB 01               jmp short 00384712  

00384711    0F40EB              cmovo ebp,ebx  

00384714    01CD                add ebp,ecx  

00384716    FFE0                jmp eax  

00384718    C7                  ???                             ; 未知命令  

00384719    E9 FC000000         jmp 0038481A  

0038471E    83F8 16             cmp eax,16       //第二十二个 4BC1EF=<jmp.&wsock32.send>  

00384721    75 67               jnz short 0038478A  

00384723    55                  push ebp  

00384724    8BEC                mov ebp,esp  

00384726    E8 00000000         call 0038472B  

0038472B    59                  pop ecx  

0038472C    81E9 7E354100       sub ecx,41357E  

00384732    51                  push ecx  

00384733    51                  push ecx  

00384734    8B45 10             mov eax,dword ptr ss:[ebp+10]  

00384737    6A 00               push 0  

00384739    8945 F8             mov dword ptr ss:[ebp-8],eax  

0038473C    8B45 0C             mov eax,dword ptr ss:[ebp+C]  

0038473F    6A 00               push 0  

00384741    8945 FC             mov dword ptr ss:[ebp-4],eax  

00384744    FF75 14             push dword ptr ss:[ebp+14]  

00384747    8D45 10             lea eax,dword ptr ss:[ebp+10]  

0038474A    50                  push eax  

0038474B    8D45 F8             lea eax,dword ptr ss:[ebp-8]  

0038474E    6A 01               push 1  

00384750    50                  push eax  

00384751    FF75 08             push dword ptr ss:[ebp+8]  

00384754    8D89 65494100       lea ecx,dword ptr ds:[ecx+414965]  

0038475A    FFD1                call ecx  

0038475C    83F8 FF             cmp eax,-1  

0038475F    75 03               jnz short 00384764  

00384761    0945 10             or dword ptr ss:[ebp+10],eax  

00384764    8B45 10             mov eax,dword ptr ss:[ebp+10]  

00384767    8BE5                mov esp,ebp  

00384769    5D                  pop ebp  

0038476A    83C4 14             add esp,14  

0038476D    FF6424 EC           jmp dword ptr ss:[esp-14]  

00384771    E8 04000000         call 0038477A  

00384776    0FEB0CC9            por mm1,qword ptr ds:[ecx+ecx*8]  

0038477A    58                  pop eax  

0038477B    EB 01               jmp short 0038477E  

0038477D    0F40EB              cmovo ebp,ebx  

00384780    01CD                add ebp,ecx  

00384782    FFE0                jmp eax  

00384784    C7                  ???                             ; 未知命令  

00384785    E9 90000000         jmp 0038481A  

0038478A    83F8 17             cmp eax,17   //第二十三个 4BC39F=<jmp.&wsock32.recv>  

0038478D    0F85 87000000       jnz 0038481A  

00384793    55                  push ebp  

00384794    8BEC                mov ebp,esp  

00384796    E8 00000000         call 0038479B  

0038479B    59                  pop ecx  

0038479C    81E9 EE354100       sub ecx,4135EE  

003847A2    51                  push ecx  

003847A3    51                  push ecx  

003847A4    8B45 10             mov eax,dword ptr ss:[ebp+10]  

003847A7    6A 00               push 0  

003847A9    8945 F8             mov dword ptr ss:[ebp-8],eax  

003847AC    8B45 0C             mov eax,dword ptr ss:[ebp+C]  

003847AF    8945 FC             mov dword ptr ss:[ebp-4],eax  

003847B2    8B45 14             mov eax,dword ptr ss:[ebp+14]  

003847B5    8945 0C             mov dword ptr ss:[ebp+C],eax  

003847B8    8D45 0C             lea eax,dword ptr ss:[ebp+C]  

003847BB    6A 00               push 0  

003847BD    50                  push eax  

003847BE    8D45 10             lea eax,dword ptr ss:[ebp+10]  

003847C1    50                  push eax  

003847C2    8D45 F8             lea eax,dword ptr ss:[ebp-8]  

003847C5    6A 01               push 1  

003847C7    50                  push eax  

003847C8    FF75 08             push dword ptr ss:[ebp+8]  
