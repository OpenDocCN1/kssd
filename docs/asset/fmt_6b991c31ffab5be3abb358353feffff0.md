
```
 000144A2: 2020                   and byte ptr [eax],ah  

 000144A4: 2020                   and byte ptr [eax],ah  

 000144A6: 45                     inc ebp  

 000144A7: 2D6D61696C             sub eax,6C69616D  

 000144AC: 2020                   and byte ptr [eax],ah  

 000144AE: 203A                   and byte ptr [edx],bh  

 000144B0: 2020                   and byte ptr [eax],ah  

 000144B2: 2020                   and byte ptr [eax],ah  

 000144B4: 2038                   and byte ptr [eax],bh  

 000144B6: 3530343400             xor eax,00343430  

 000144BB: 0000                   add byte ptr [eax],al  

 000144BD: 0000                   add byte ptr [eax],al  

 000144BF: 0000                   add byte ptr [eax],al  

 000144A0: 2020                   and byte ptr [eax],ah  

 000144A2: 2020                   and byte ptr [eax],ah  

 000144A4: 2020                   and byte ptr [eax],ah  

 000144A6: 45                     inc ebp  

 000144A7: 2D6D61696C             sub eax,6C69616D  

 000144AC: 2020                   and byte ptr [eax],ah  

 000144AE: 203A                   and byte ptr [edx],bh  

 000144B0: 2064696E               and byte ptr [ecx+ebp*2+6E],ah  

 000144B4: 672D626F7940           sub eax,40796F62  

 000144BA: 6E                     outsb dx,ds:[esi]  

 000144BB: 657465                 jz 00014523  

 000144BE: 61                     popad   

 000144BF: 7365                   jnb 00014526  

 000144C1: 2E636F6D               arpl word ptr cs:[edi+6D],bp  

 000144C5: 200D0A202020           and byte ptr [2020200A],cl  

 000144CB: 2020                   and byte ptr [eax],ah  

 000144CD: 2020                   and byte ptr [eax],ah  

 000144CF: 2020                   and byte ptr [eax],ah  

 000144D1: 2020                   and byte ptr [eax],ah  

 000144D3: 2020                   and byte ptr [eax],ah  

 000144D5: 2020                   and byte ptr [eax],ah  

 000144D7: 2020                   and byte ptr [eax],ah  

 000144D9: 2020                   and byte ptr [eax],ah  

 000144DB: 48                     dec eax  

 000144DC: 6F                    outsd dx,ds:[esi]  

 000144DD: 6D                    insd es:[edi],dx  

 000144DE: 657061                 jo 00014542  

 000144E1: 6765203A               and byte ptr gs:[bp+si],bh  

 000144E5: 206874                 and byte ptr [eax+74],ch  

 000144E8: 7470                  jz 0001455A  

 000144EA: 3A2F                  cmp ch,byte ptr [edi]  

 000144EC: 2F                    das  

 000144ED: 667364                 jnb 00014554  

 000144F0: 622E                  bound ebp,dword ptr [esi]  

 000144F2: 7965                  jns 00014559  

 000144F4: 61                    popad   

 000144F5: 682E6E6574             push 74656E2E  

 000144FA: 2020                   and byte ptr [eax],ah  

 000144FC: 2020                   and byte ptr [eax],ah  

 000144FE: 2020                   and byte ptr [eax],ah  

 00014500: 2020                   and byte ptr [eax],ah  

 00014502: 2020                   and byte ptr [eax],ah  

 00014504: 2020                   and byte ptr [eax],ah  

 00014506: 200D0A0D0A00           and byte ptr [000A0D0A],cl  

 0001450C: 0000                   add byte ptr [eax],al  

 0001450E: 0000                   add byte ptr [eax],al  

 00014510: 0000                   add byte ptr [eax],al  

 00014512: 0000                   add byte ptr [eax],al  

 00014514: 0000                   add byte ptr [eax],al  

 00014516: 0000                   add byte ptr [eax],al  

 00014518: 0000                   add byte ptr [eax],al  

 0001451A: 0000                   add byte ptr [eax],al  

 0001451C: 0000                   add byte ptr [eax],al  

 0001451E: 0000                   add byte ptr [eax],al  

 00014520: 0000                   add byte ptr [eax],al  

 00014522: 0000                   add byte ptr [eax],al  

 00014524: 0000                   add byte ptr [eax],al  

 00014526: 0000                   add byte ptr [eax],al  

 00014528: 0000                   add byte ptr [eax],al  

 0001452A: 0000                   add byte ptr [eax],al  

 0001452C: 0000                   add byte ptr [eax],al  

 0001452E: 0000                   add byte ptr [eax],al  

 00014530: 0000                   add byte ptr [eax],al  

 00014532: 0000                   add byte ptr [eax],al  

 00014534: 0000                   add byte ptr [eax],al  

 00014536: 0000                   add byte ptr [eax],al  

 00014538: 0000                   add byte ptr [eax],al  

 0001453A: 0000                   add byte ptr [eax],al  

 0001453C: 0000                   add byte ptr [eax],al  

 0001453E: 0000                   add byte ptr [eax],al  

 00014540: 0000                   add byte ptr [eax],al  

 00014542: 0000                   add byte ptr [eax],al  

 00014544: 49                     dec ecx  

 00014545: 6E                     outsb dx,ds:[esi]  

 00014546: 7661                   jbe 000145A9  

 00014548: 6C                     insb es:[edi],dx  

 00014549: 6964202050617373       imul esp,dword ptr [eax+20],73736150  

 00014551: 57                     push edi  

 00014552: 6F                     outsd dx,ds:[esi]  

 00014553: 7264                   jb 000145B9  

 00014555: 0000                   add byte ptr [eax],al  

 00014557: 0000                   add byte ptr [eax],al  

 00014559: 0000                   add byte ptr [eax],al  

 0001455B: 0000                   add byte ptr [eax],al  

 0001455D: 0000                   add byte ptr [eax],al  

 0001455F: 0000                   add byte ptr [eax],al  

 00014561: 0000                   add byte ptr [eax],al  

 00014563: 0000                   add byte ptr [eax],al  

 00014565: 0000                   add byte ptr [eax],al  

 00014567: 0000                   add byte ptr [eax],al  

 00014569: 0000                   add byte ptr [eax],al  

 0001456B: 0000                   add byte ptr [eax],al  

 0001456D: 0000                   add byte ptr [eax],al  

 0001456F: 0000                   add byte ptr [eax],al  

 00014571: 0000                   add byte ptr [eax],al  

 00014573: 0000                   add byte ptr [eax],al  

 00014575: 0000                   add byte ptr [eax],al  

 00014577: 0000                   add byte ptr [eax],al  

 00014579: 0000                   add byte ptr [eax],al  

 0001457B: 0000                   add byte ptr [eax],al  

 0001457D: 0000                   add byte ptr [eax],al  

 0001457F: 0000                   add byte ptr [eax],al  

 00014581: 0000                   add byte ptr [eax],al  

 00014583: 0000                   add byte ptr [eax],al  

 00014585: 0000                   add byte ptr [eax],al  

 00014587: 0000                   add byte ptr [eax],al  

 00014589: 0000                   add byte ptr [eax],al  

 0001458B: 0000                   add byte ptr [eax],al  

 0001458D: 0000                   add byte ptr [eax],al  

 0001458F: 0000                   add byte ptr [eax],al  

 00014591: 0000                   add byte ptr [eax],al  

 00014593: 0000                   add byte ptr [eax],al  

 00014595: 0000                   add byte ptr [eax],al  
```
