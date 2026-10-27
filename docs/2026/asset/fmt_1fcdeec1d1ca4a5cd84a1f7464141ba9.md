
00406016     C685 15274000 01     mov byte ptr ss:[ebp+402715],1

0040601D     E9 E4000000          jmp VGCRYPT.00406106

                                  ====>跳  呵呵，省了一段路程

00406106     E8 1B000000          call VGCRYPT.00406126

0040610B     8B6424 08            mov esp,dword ptr ss:[esp+8]

0040610F     E8 DFFFFFFF          call VGCRYPT.004060F3

00406114     C685 CC264000 C3     mov byte ptr ss:[ebp+4026CC],0C3

0040611B     E8 4BFFFFFF          call VGCRYPT.0040606B

00406120   ^ E9 16FFFFFF          jmp VGCRYPT.0040603B

00406125     B7 64                mov bh,64

00406127     67:FF36 0000         push dword ptr ds:[0]

0040612C     64:67:8926 0000      mov dword ptr fs:[0],esp

00406132     8DB5 47264000        lea esi,dword ptr ss:[ebp+402647]

00406138     8BFE                 mov edi,esi

0040613A     B9 CA000000          mov ecx,0CA

0040613F     8AA5 11274000        mov ah,byte ptr ss:[ebp+402711]

00406145     AC                   lods byte ptr ds:[esi]

00406146     32C4                 xor al,ah

00406148     FEC4                 inc ah

0040614A     C0C4 02              rol ah,2

0040614D     80C4 90              add ah,90

00406150     AA                   stos byte ptr es:[edi]

00406151   ^ E2 F2                loopd short VGCRYPT.00406145

00406153   ^ E9 CBFEFFFF          jmp VGCRYPT.00406023

                                  ====>直接F4到这，跳出LOOP！

00406023     E8 00000000          call VGCRYPT.00406028

00406028     8B9D 05274000        mov ebx,dword ptr ss:[ebp+402705]

0040602E     83C3 28              add ebx,28

00406031     58                   pop eax

00406032     2BC3                 sub eax,ebx

00406034     8985 0D274000        mov dword ptr ss:[ebp+40270D],eax

0040603A     CC                   int3

0040603B     8DBD 24264000        lea edi,dword ptr ss:[ebp+402624]

00406041     B9 3B000000          mov ecx,3B

00406046     F3:AA                rep stos byte ptr es:[edi]

00406048     64:67:8F06 0000      pop dword ptr fs:[0]

0040604E     5A                   pop edx

0040604F     8385 0D274000 01     add dword ptr ss:[ebp+40270D],1

00406056     8509                 test dword ptr ds:[ecx],ecx

00406058     27                   daa

00406059     40                   inc eax

0040605A     0061 9D              add byte ptr ds:[ecx-63],ah

0040605D     8B9A 09274000        mov ebx,dword ptr ds:[edx+402709]

00406063     898A 09274000        mov dword ptr ds:[edx+402709],ecx

                                  ====>F4直接到这！

00406069     FFE3                 jmp ebx

                                  ====>跳至00405000

――――――――――――――――――――――――

### 四、第4层壳

00405000     9C                   pushfd

00405001     55                   push ebp

00405002     E8 EC000000          call VGCRYPT.004050F3

00405007     87D5                 xchg ebp,edx

00405009     5D                   pop ebp

0040500A     60                   pushad

0040500B     87D5                 xchg ebp,edx

0040500D     80BD 15274000 01     cmp byte ptr ss:[ebp+402715],1

00405014     74 39                je short VGCRYPT.0040504F

00405016     C685 15274000 01     mov byte ptr ss:[ebp+402715],1

0040501D     E9 E4000000          jmp VGCRYPT.00405106

                                  ====>跳

00405106     E8 1B000000          call VGCRYPT.00405126

0040510B     8B6424 08            mov esp,dword ptr ss:[esp+8]

0040510F     E8 DFFFFFFF          call VGCRYPT.004050F3

00405114     C685 CC264000 C3     mov byte ptr ss:[ebp+4026CC],0C3

0040511B     E8 4BFFFFFF          call VGCRYPT.0040506B

00405120   ^ E9 16FFFFFF          jmp VGCRYPT.0040503B

00405125     B7 64                mov bh,64

00405127     67:FF36 0000         push dword ptr ds:[0]

0040512C     64:67:8926 0000      mov dword ptr fs:[0],esp

00405132     8DB5 47264000        lea esi,dword ptr ss:[ebp+402647]

00405138     8BFE                 mov edi,esi

0040513A     B9 CA000000          mov ecx,0CA

0040513F     8AA5 11274000        mov ah,byte ptr ss:[ebp+402711]

00405145     AC                   lods byte ptr ds:[esi]

00405146     32C4                 xor al,ah

00405148     FEC4                 inc ah

0040514A     C0C4 02              rol ah,2

0040514D     80C4 90              add ah,90

00405150     AA                   stos byte ptr es:[edi]

00405151   ^ E2 F2                loopd short VGCRYPT.00405145

00405153   ^ E9 CBFEFFFF          jmp VGCRYPT.00405023

                                  ====>直接F4到这，跳出LOOP！

00405023     E8 00000000          call VGCRYPT.00405028

00405028     8B9D 05274000        mov ebx,dword ptr ss:[ebp+402705]

0040502E     83C3 28              add ebx,28

00405031     58                   pop eax

00405032     2BC3                 sub eax,ebx

00405034     8985 0D274000        mov dword ptr ss:[ebp+40270D],eax

0040503A     CC                   int3

0040503B     8DBD 24264000        lea edi,dword ptr ss:[ebp+402624]

00405041     B9 3B000000          mov ecx,3B

00405046     F3:AA                rep stos byte ptr es:[edi]

00405048     64:67:8F06 0000      pop dword ptr fs:[0]

0040504E     5A                   pop edx

0040504F     8B85 0D274000        mov eax,dword ptr ss:[ebp+40270D]

00405055     0185 09274000        add dword ptr ss:[ebp+402709],eax

0040505B     61                   popad

0040505C     9D                   popfd

0040505D     8B9A 09274000        mov ebx,dword ptr ds:[edx+402709]

00405063     898A 09274000        mov dword ptr ds:[edx+402709],ecx

                                  ====>F4直接到这！

00405069     FFE3                 jmp ebx

                                  ====>跳至00401000  这就是OEP值  :-)

――――――――――――――――――――――――

00401000       E8                 db E8

                                  ====>在这儿用LordPE完全DUMP这个进程

00401001       51                 db 51

00401002       06                 db 06

―――――――――――――――――――――――

停在OEP处，运行ImportREC，选择这个进程。把OEP改为00001000，点IT AutoSearch，点“Get Import”，

函数都是有效的。FixDump，正常运行！  8.5K ->40K

―――――――――――――――――――――――――――――――――

         ,     \_/

        /| \_.-~/            \_     ,        青春都一饷

       ( /~   /              ~-.\_ |

       `\  \_/                   ~ )          忍把浮名

   \_-~~~-.)  )\_\_/;;,.          \_  //'

  /'\_,   --~    ~~~-  ,;;\_\_\_(  (.-~~~-.        换了破解轻狂

 `~ \_( ,\_..-- (     ,;'' /    ~--   /.\_`

  /~~//'   /' `~         ) /--..\_, )\_  `~

  "  `~"  "      `"      /~'`    `\~~

                         "     "   "~'  ""

               Cracked By 巢水工作坊――fly [OCN][FCG]

                       2003-10-02  02:20
