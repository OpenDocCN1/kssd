
0046567B    68 84334000     push `Windows\_.00403384`                   ; ASCII `".key"`  

00465680    FF75 F8         push `dword` ptr `ss:\[ebp-8\]`  

00465683    68 E7324000    push `Windows\_.004032E7`  

00465688    FF75 FC        push `dword` ptr `ss:\[ebp-4\]`  

0046568B    B9 04000000    mov `ecx`, 4  

00465690    E8 6BFBFFFF    call `Windows\_.00465200`  

00465695    83C4 10        add `esp`, 10  

00465698    8945 F4        mov `dword` ptr `ss:\[ebp-C\]`, `eax`  

0046569B    8B5D FC        mov `ebx`, `dword` ptr `ss:\[ebp-4\]`  

0046569E    85DB           test `ebx`, `ebx`  

004656A0    74 09          je short `Windows\_.004656AB`  

004656A2    53             push `ebx`  

004656A3    E8 66220300    call `Windows\_.0049790E`  

004656A8    83C4 04        add `esp`, 4  

004656AB    8B5D F8        mov `ebx`, `dword` ptr `ss:\[ebp-8\]`  

004656AE    85DB           test `ebx`, `ebx`  

004656B0    74 09          je short `Windows\_.004656BB`  

004656B2    53             push `ebx`  

004656B3    E8 56220300    call `Windows\_.0049790E`  

004656B8    83C4 04        add `esp`, 4  

004656BB    68 04000080    push `80000004`  

004656C0    6A 00          push 0  

004656C2    8B45 F4        mov `eax`, `dword` ptr `ss:\[ebp-C\]`  

004656C5    85C0           test `eax`, `eax`  

004656C7    75 05          jnz short `Windows\_.004656CE`  

004656C9    B8 E9324000    mov `eax`, `Windows\_.004032E9`  

004656CE    50             push `eax`  

004656CF    68 01000000    push 1  

004656D4    BB 4C020000    mov `ebx`, `24C`  

004656D9    E8 48220300    call `Windows\_.00497926`  

004656DE    83C4 10        add `esp`, 10  

004656E1    8945 F0        mov `dword` ptr `ss:\[ebp-10\]`, `eax`  

004656E4    8B5D F4        mov `ebx`, `dword` ptr `ss:\[ebp-C\]`  

004656E7    85DB           test `ebx`, `ebx`  

004656E9    74 09          je short `Windows\_.004656F4`  

004656EB    53             push `ebx`  

004656EC    E8 1D220300    call `Windows\_.0049790E`  

004656F1    83C4 04        add `esp`, 4  

004656F4    837D F0 00     cmp `dword` ptr `ss:\[ebp-10\]`, 0  

004656F8    0F85 0A000000  jnz `Windows\_.00465708`  

004656FE    B8 00000000    mov `eax`, 0   //这里就是爆破点了。  

修改为  

004656FE    B8 01000000    mov `eax`, 1  

00465703    E9 93010000     jmp `Windows\_.0046589B`  //这里就跳走了，慢着，把 `Eax` 里面的值为 1 你才能走!  

00465708    6A FF          push -1  

0046570A    6A 08          push 8  

0046570C    68 5B0E0116    push `16010E5B`     //下面一大堆加密算法。  

00465711    68 01000152    push `52010001`  

00465716    E8 05220300    call `Windows\_.00497920`  

0046571B    83C4 10        add `esp`, 10  

0046571E    8945 FC        mov `dword` ptr `ss:\[ebp-4\]`, `eax`  

00465721    68 04000080    push `80000004`  

00465726    6A 00          push 0  

00465728    8B45 FC        mov `eax`, `dword` ptr `ss:\[ebp-4\]`  

0046572B    85C0           test `eax`, `eax`  

0046572D    75 05          jnz short `Windows\_.00465734`  

0046572F    B8 E9324000    mov `eax`, `Windows\_.004032E9`  

00465734    50             push `eax`  

00465735    68 01000000    push 1  

0046573A    BB 98010000    mov `ebx`, `198`  

0046573F    E8 E2210300    call `Windows\_.00497926`  

00465744    83C4 10        add `esp`, 10  

00465747    8945 F8        mov `dword` ptr `ss:\[ebp-8\]`, `eax`  

0046574A    8B5D FC        mov `ebx`, `dword` ptr `ss:\[ebp-4\]`  

0046574D    85DB           test `ebx`, `ebx`  

0046574F    74 09          je short `Windows\_.0046575A`  

00465751    53             push `ebx`  

00465752    E8 B7210300    call `Windows\_.0049790E`  

00465757    83C4 04        add `esp`, 4  

0046575A    6A FF          push -1  

0046575C    6A 08          push 8  

0046575E    68 5B0E0116    push `16010E5B`  

00465763    68 01000152    push `52010001`  

00465768    E8 B3210300    call `Windows\_.00497920`  

0046576D    83C4 10        add `esp`, 10  

00465770    8945 F4        mov `dword` ptr `ss:\[ebp-C\]`, `eax`  

00465773    68 84334000    push `Windows\_.00403384`                   ; ASCII `".key"`  

00465778    FF75 F4        push `dword` ptr `ss:\[ebp-C\]`  

0046577B    B9 02000000    mov `ecx`, 2  

00465780    E8 7BFAFFFF    call `Windows\_.00465200`  

00465785    83C4 08        add `esp`, 8  

00465788    8945 F0        mov `dword` ptr `ss:\[ebp-10\]`, `eax`  

0046578B    8B5D F4        mov `ebx`, `dword` ptr `ss:\[ebp-C\]`  

0046578E    85DB           test `ebx`, `ebx`  

00465790    74 09          je short `Windows\_.0046579B`  

00465792    53             push `ebx`  

00465793    E8 76210300    call `Windows\_.0049790E`  

00465798    83C4 04        add `esp`, 4  

0046579B    BB 06000000    mov `ebx`, 6  

004657A0    E8 97FEFFFF    call `Windows\_.0046563C`  

004657A5    68 04000080    push `80000004`  

004657AA    6A 00          push 0  

004657AC    8B45 F0        mov `eax`, `dword` ptr `ss:\[ebp-10\]`  

004657AF    85C0           test `eax`, `eax`  

004657B1    75 05          jnz short `Windows\_.004657B8`  

004657B3    B8 E9324000    mov `eax`, `Windows\_.004032E9`  

004657B8    50             push `eax`  

004657B9    68 03000000    push 3  

004657BE    BB 70020000    mov `ebx`, `270`  

004657C3    E8 5E210300    call `Windows\_.00497926`  

004657C8    83C4 28        add `esp`, 28  

004657CB    8945 EC        mov `dword` ptr `ss:\[ebp-14\]`, `eax`  

004657CE    8B5D F0        mov `ebx`, `dword` ptr `ss:\[ebp-10\]`  

004657D1    85DB           test `ebx`, `ebx`  

004657D3    74 09          je short `Windows\_.004657DE`  

004657D5    53             push `ebx`  

004657D6    E8 33210300    call `Windows\_.0049790E`  

004657DB    83C4 04        add `esp`, 4  

004657DE    6A 00          push 0  

004657E0    6A 00          push 0  

004657E2    6A 00          push 0  

004657E4    68 01030080    push `80000301`  

004657E9    6A 00          push 0  

004657EB    FF75 EC        push `dword` ptr `ss:\[ebp-14\]`  

004657EE    68 02000000    push 2  

004657F3    BB 9C020000    mov `ebx`, `29C`  

004657F8    E8 29210300    call `Windows\_.00497926`  

004657FD    83C4 1C        add `esp`, `1C`  

00465800    8945 E8        mov `dword` ptr `ss:\[ebp-18\]`, `eax`  

00465803    68 04000080    push `80000004`  

00465808    6A 00          push 0  

0046580A    68 89334000    push `Windows\_.00403389`                   ; ASCII  

`"89C40FBE91E58CB2E918DF3B5E29A94E418B105D5CE86B1955F1485F7F3CBE931CA3B4C55F55A792004637745833DBF9`  

`AE6B54070F32D7589AC1FB4B0ACEC27F4E675B5BF1330D177BC3E3CD64C7F38ECF5FE6C1819656C37F5F4F6F482D2D29B`  

`3087A8A17C7517C7A10F5E9AFFD1AD123290794592EB"...`  

0046580F    68 04000080    push `80000004`  

00465814    6A 00          push 0  

00465816    68 8A344000    push `Windows\_.0040348A`                   ; ASCII `"16AC3"`  

0046581B    68 04000080    push `80000004`  

00465820    6A 00          push 0  

00465822    8B45 E8        mov `eax`, `dword` ptr `ss:\[ebp-18\]`  

00465825    85C0           test `eax`, `eax`  

00465827    75 05          jnz short `Windows\_.0046582E`  

00465829    B8 E9324000    mov `eax`, `Windows\_.004032E9`  

0046582E    50             push `eax`  

0046582F    68 05000080    push `80000005`  

00465834    6A 00          push 0  

00465836    8B45 F8        mov `eax`, `dword` ptr `ss:\[ebp-8\]`  

00465839    85C0           test `eax`, `eax`  

0046583B    75 05          jnz short `Windows\_.00465842`  

0046583D    B8 90344000    mov `eax`, `Windows\_.00403490`  

00465842    50             push `eax`  

00465843    68 04000000    push 4  

00465848    BB 18000000    mov `ebx`, `18`  

0046584D    B8 05000000    mov `eax`, 5  

00465852    E8 C3200300    call `Windows\_.0049791A`  

00465857    83C4 34        add `esp`, `34`  

0046585A    8945 E4        mov `dword` ptr `ss:\[ebp-1C\]`, `eax`  

0046585D    8B5D F8        mov `ebx`, `dword` ptr `ss:\[ebp-8\]`  

00465860    85DB           test `ebx`, `ebx`  

00465862    74 09          je short `Windows\_.0046586D`  

00465864    53             push `ebx`  

00465865    E8 A4200300    call `Windows\_.0049790E`  

0046586A    83C4 04        add `esp`, 4  

0046586D    8B5D E8        mov `ebx`, `dword` ptr `ss:\[ebp-18\]`  

00465870    85DB           test `ebx`, `ebx`  
