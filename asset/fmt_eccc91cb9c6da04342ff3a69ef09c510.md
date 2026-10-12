
```assembly
004923A4  |.⁰F85 70FFFFFF  \JNZ ssb.0049231A                  ; 以上循环 3 次  

004923AA  |. 8D45 EC       LEA EAX,DWORD PTR SS:[EBP-14]       ; 倒转的 18 位密码  

004923AD  |. 8B55 E8       MOV EDX,DWORD PTR SS:[EBP-18]       ; 生成的 21 位数字，入 edx  

004923B0  |. E8 1319F7FF   CALL ssb.00403CC8  

004923B5  |. BB 01000000   MOV EBX,1  

004923BA  |. 8D75 C8       LEA ESI,DWORD PTR SS:[EBP-38]  

004923BD  |> 8D45 9C       /LEA EAX,DWORD PTR SS:[EBP-64]  

004923C0  |. 50            |PUSH EAX  

004923C1  |. 8D145B        |LEA EDX,DWORD PTR DS:[EBX+EBX*2]  

004923C4  |. 83EA 02       |SUB EDX,2  

004923C7  |. B9 03000000   |MOV ECX,3  

004923CC  |. 8B45 EC       |MOV EAX,DWORD PTR SS:[EBP-14]       ; 生成的 21 位数字，入 eax  

004923CF  |. E8 E41CF7FF    |CALL ssb.004040B8                   ; 从第[(ebx+ebx*2)-2]位开始，取 3 位  

004923D4  |. 8B45 9C       |MOV EAX,DWORD PTR SS:[EBP-64]  

004923D7  |. E8 7C67F7FF   |CALL ssb.00408B58                   ; 3 位数字，转换为十六进制  

004923DC  |. 8906          |MOV DWORD PTR DS:[ESI],EAX  

004923DE  |. 43            |INC EBX  

004923DF  |. 83C6 04       |ADD ESI,4  

004923E2  |. 83FB 08       |CMP EBX,8                           ; 循环 7 次  

004923E5  |.⁷⁵ D6          \JNZ SHORT ssb.004923BD              ; 以上为把 21 位十进制数字，分成 7 段  

004923E7  |. BB FAFFFFFF    MOV EBX,-6  

004923EC  |. 8D45 E0       LEA EAX,DWORD PTR SS:[EBP-20]  

004923EF  |> 8B10          /MOV EDX,DWORD PTR DS:[EAX]          ; 第(8-ebx)段入 edx  

004923F1  |. 3B50 FC       |CMP EDX,DWORD PTR DS:[EAX-4]        ; 第(8-ebx)段与第(8-ebx-1)段是否相等  

004923F4  |. 7D 06         |JGE SHORT ssb.004923FC              ; 如第(8-ebx)段比第(8-ebx-1)段小，则第(8-ebx)段+3e8  

004923F6  |. 8100 E8030000 |ADD DWORD PTR DS:[EAX],3E8          ; 如小，则第(8-ebx)段+3e8  

004923FC  |> 8B50 FC       |MOV EDX,DWORD PTR DS:[EAX-4]       ; 第(8-ebx)-1 段入 edx  

004923FF  |. 2910          |SUB DWORD PTR DS:[EAX],EDX          ; 第(8-ebx)段-第 6 段  

00492401  |. 83E8 04       |SUB EAX,4  

00492404  |. 43            |INC EBX  

00492405  |.⁷⁵ E8          \JNZ SHORT ssb.004923EF             ; 循环 7 次,把以上分成 7 段，作相应变换 7 段  

00492407  |. 8D45 EC       LEA EAX,DWORD PTR SS:[EBP-14]       ; 生成的 21 位数字，入 eax  

0049240A  |. E8 2118F7FF   CALL ssb.00403C30  

0049240F  |. BB 07000000   MOV EBX,7  

00492414  |. 8D75 C8       LEA ESI,DWORD PTR SS:[EBP-38]  

00492417  |> 8D55 E8       /LEA EDX,DWORD PTR SS:[EBP-18]      ; 生成的 21 位数字，入 edx  

0049241A  |. 8B06          |MOV EAX,DWORD PTR DS:[ESI]         ; 变换的第 ebx 段入 eax  

0049241C  |. E8 9766F7FF   |CALL ssb.00408AB8                 ; 把十六进制数转换为十进制  

00492421  |. 8B45 E8       |MOV EAX,DWORD PTR SS:[EBP-18]      ; 十进制数入 eax  

00492424  |. E8 871AF7FF   |CALL ssb.00403EB0                 ; 取位数  

00492429  |. BF 03000000   |MOV EDI,3  

0049242E  |. 2BF8          |SUB EDI,EAX  

00492430  |. 85FF          |TEST EDI,EDI  

00492432  |. 7E 13         |JLE SHORT ssb.00492447  

00492434  |> 8D45 E8       |/LEA EAX,DWORD PTR SS:[EBP-18]  

00492437  |. 8B4D E8       ||MOV ECX,DWORD PTR SS:[EBP-18]  

0049243A  |. BA 7C264900   ||MOV EDX,ssb.0049267C             ; 0 入 edx  

0049243F  |. E8 B81AF7FF   ||CALL ssb.00403EFC  

00492444  |. 4F            ||DEC EDI  

00492445  |.⁷⁵ ED          |\JNZ SHORT ssb.00492434           ; 如果十进制数小于 3 位，则在前面插入 0  

00492447  |> 8D45 EC       |LEA EAX,DWORD PTR SS:[EBP-14]  

0049244A  |. 8B55 E8       |MOV EDX,DWORD PTR SS:[EBP-18]  

0049244D  |. E8 661AF7FF   |CALL ssb.00403EB8                 ; 7 段依次形成 21 位，3*7=21  

00492452  |. 83C6 04       |ADD ESI,4  

00492455  |. 4B            |DEC EBX  

00492456  |.⁷⁵ BF         \JNZ SHORT ssb.00492417  

00492458  |. BB 01000000   MOV EBX,1  

0049245D  |> 8D45 98       /LEA EAX,DWORD PTR SS:[EBP-68]  

00492460  |. 50            |PUSH EAX  

00492461  |. B9 01000000   |MOV ECX,1  

00492466  |. 8BD3          |MOV EDX,EBX  

00492468  |. 8B45 EC       |MOV EAX,DWORD PTR SS:[EBP-14]     ; 形成 21 位十进制数入 eax SS:  

0049246B  |. E8 481CF7FF   |CALL ssb.004040B8                 ; 从第 ebx 位开始，取 1 位数字 0B8  

00492470  |. 8B45 98       |MOV EAX,DWORD PTR SS:[EBP-68]  

00492473  |. BA 88264900   |MOV EDX,ssb.00492688             ; 9 入 edx  

00492478  |. E8 431BF7FF   |CALL ssb.00403FC0  

0049247D  |. 0F87 60010000 |JA ssb.004925E3  

00492483  |. 8D45 94       |LEA EAX,DWORD PTR SS:[EBP-6C]  

00492486  |. 50            |PUSH EAX  

00492487  |. B9 01000000   |MOV ECX,1  

0049248C  |. 8BD3          |MOV EDX,EBX  

0049248E  |. 8B45 EC       |MOV EAX,DWORD PTR SS:[EBP-14]     ; 形成 21 位十进制数入 eax   

00492491  |. E8 221CF7FF   |CALL ssb.004040B8  

00492496  |. 8B45 94       |MOV EAX,DWORD PTR SS:[EBP-6C]     ; 0 入 eax  

00492499  |. BA 7C264900   |MOV EDX,ssb.0049267C  

0049249E  |. E8 1D1BF7FF   |CALL ssb.00403FC0  

004924A3  |. 0F82 3A010000 |JB ssb.004925E3  

004924A9  |. 43            |INC EBX  

004924AA  |. 83FB 16       |CMP EBX,16                        ; 循环 21 次 BX,  

004924AD  |.⁷⁵ AE          \JNZ SHORT ssb.0049245D            ; 以上为判断从形成 21 位十进制数是否为数字  

004924AF  |. 8D45 E8       LEA EAX,DWORD PTR SS:[EBP-18]  

004924B2  |. 50            PUSH EAX  

004924B3  |. 8B45 EC       MOV EAX,DWORD PTR SS:[EBP-14]      ; 形成 21 位十进制数入 eax  

004924B6  |. E8 F519F7FF   CALL ssb.00403EB0                  ; 取位数  

004924BB  |. 8BC8          MOV ECX,EAX  

004924BD  |. BA 03000000   MOV EDX,3  

004924C2  |. 8B45 EC       MOV EAX,DWORD PTR SS:[EBP-14]      ; 形成 21 位十进制数入 eax  

004924C5  |. E8 EE1BF7FF   CALL ssb.004040B8                  ; 从第 3 位开始，取 21 位，即取第 3 位开始的后面所有位，形成 19 位  

004924CA  |. BE 01000000   MOV ESI,1                         ;esi=1  

004924CF  |. BB 01000000   MOV EBX,1  

004924D4  |> 8D45 90       /LEA EAX,DWORD PTR SS:[EBP-70]  

004924D7  |. 50            |PUSH EAX  

004924D8  |. B9 01000000   |MOV ECX,1  

004924DD  |. 8BD3          |MOV EDX,EBX  

004924DF  |. 8B45 E8       |MOV EAX,DWORD PTR SS:[EBP-18]     ; 形成 19 位数入 eax  

004924E2  |. E8 D11BF7FF   |CALL ssb.004040B8                 ; 从第 ebx 开始取 1 位  

004924E7  |. 8B45 90       |MOV EAX,DWORD PTR SS:[EBP-70]  

004924EA  |. E8 6966F7FF   |CALL ssb.00408B58                 ; 转换为十六进制,入 eax  

004924EF  |. 8BF8          |MOV EDI,EAX                       ;  edi=eax  

004924F1  |. 85FF          |TEST EDI,EDI  

004924F3  |. 74 03         |JE SHORT ssb.004924F8  

004924F5  |. 0FAFF7        |IMUL ESI,EDI                      ;  esi=esi*edi，如数为 0 不参与运算  

004924F8  |> 83FE 64       |CMP ESI,64                        ;  esi 是否小于 64  

004924FB  |. 7E 0C         |JLE SHORT ssb.00492509  

004924FD  |. 8BC6          |MOV EAX,ESI  

004924FF  |. B9 64000000   |MOV ECX,64  

00492504  |. 99            |CDQ  

00492505  |. F7F9          |IDIV ECX  

00492507  |. 8BF0          |MOV ESI,EAX                       ;  商入 esi  

00492509  |> 43            |INC EBX  

0049250A  |. 83FB 14       |CMP EBX,14                        ;  循环 19 次  

0049250D  |.⁷⁵ C5          \JNZ SHORT ssb.004924D4  

0049250F  |. 8BC6          MOV EAX,ESI  

00492511  |. B9 64000000   MOV ECX,64  

00492516  |. 99            CDQ  

00492517  |. F7F9          IDIV ECX  

00492519  |. 8BF2          MOV ESI,EDX                       ;  取 esi 的 64 模数，并入 esi  

0049251B  |. 83FE 0A       CMP ESI,0A  

0049251E  |. 7D 03         JGE SHORT ssb.00492523            ;  如果 esi 小于 0a,则 esi=esi+0a  

00492520  |. 83C6 0A       ADD ESI,0A  

00492523  |> 8D45 8C       LEA EAX,DWORD PTR SS:[EBP-74]  

00492526  |. 50            PUSH EAX  
```
