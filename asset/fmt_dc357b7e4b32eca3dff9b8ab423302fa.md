
```asm
  xor ebx,dword ptr [aAcumulate+0Ch]  

  and ebx,dword ptr [aAcumulate-4]  

  xor ebx,esi  

  add ebx,edi  

  lea edx,dword ptr [edx+ebx+0F57C0FAFh]  

  mov edi,edx  

  shr edi,19h  

  shl edx,7  

  mov ebx,dword ptr [aAcumulate+0Ch]  

  or edi,edx  

  add edi,dword ptr [aAcumulate-4]  

  xor ebx,dword ptr [aAcumulate-4]  

  mov edx,dword ptr [eax+14h]  

  and ebx,edi  

  mov dword ptr [aAcumulate-2Ch],edx  

  xor ebx,dword ptr [aAcumulate+0Ch]  

  add ebx,edx  

  lea esi,dword ptr [esi+ebx+4787C62Ah]  

  mov ebx,dword ptr [aAcumulate+0Ch]  

  mov edx,esi  

  shr edx,14h  

  shl esi,0Ch  

  or edx,esi  

  mov esi,dword ptr [eax+18h]                   ;  向后取用户名 4 个字母  

  mov dword ptr [aAcumulate-20h],esi            ;  作为一个累计性质的值，保存  

  mov esi,dword ptr [aAcumulate-4]              

  add edx,edi  

  xor esi,edi  

  and esi,edx  

  xor esi,dword ptr [aAcumulate-4]  

  add esi,dword ptr [aAcumulate-20h]  

  lea ebx,dword ptr [ebx+esi+0A8304613h]  

  mov esi,ebx  

  shr esi,0Fh  

  shl ebx,11h  

  or esi,ebx  

  mov ebx,dword ptr [eax+1Ch]                  ;  向后取用户名 4 个字母  

  mov dword ptr [aAcumulate-0Ch],ebx           ;  作为一个累计性质的值，保存  

  mov ebx,edx  

  add esi,edx  

  xor ebx,edi  

  and ebx,esi  

  mov dword ptr [aAcumulate+0Ch],esi  

  mov esi,dword ptr [aAcumulate-4]  

  xor ebx,edi  

  add ebx,dword ptr [aAcumulate-0Ch]  

  lea esi,dword ptr [esi+ebx+0FD469501h]  

  mov ebx,esi  

  shl ebx,16h  

  shr esi,0Ah  

  or ebx,esi  

  mov esi,dword ptr [eax+20h]                   ;  向后取用户名 4 个字母  

  add ebx,dword ptr [aAcumulate+0Ch]            ;  作为一个累计性质的值，保存  

  mov dword ptr [aAcumulate-34h],esi  

  mov dword ptr [aAcumulate-4],ebx  

  mov ebx,edx  

  xor ebx,dword ptr [aAcumulate+0Ch]  

  and ebx,dword ptr [aAcumulate-4]  

  xor ebx,edx  

  add ebx,esi  

  lea edi,dword ptr [edi+ebx+698098D8h]  

  mov ebx,dword ptr [aAcumulate+0Ch]  

  xor ebx,dword ptr [aAcumulate-4]  

  mov esi,edi  

  shr esi,19h  

  shl edi,7  

  or esi,edi  

  mov edi,dword ptr [eax+24h]                  ;  向后取用户名 4 个字母  

  add esi,dword ptr [aAcumulate-4]  

  mov dword ptr [aAcumulate-44h],edi           ;  作为一个累计性质的值，保存  

  and ebx,esi  

  xor ebx,dword ptr [aAcumulate+0Ch]  

  add ebx,edi  

  lea edx,dword ptr [edx+ebx+8B44F7AFh]  

  mov ebx,dword ptr [aAcumulate+0Ch]  

  mov edi,edx  

  shr edi,14h  

  shl edx,0Ch  

  or edi,edx  

  mov edx,dword ptr [eax+28h]                   ;  向后取用户名 4 个字母  

  mov dword ptr [aAcumulate-18h],edx            ;  作为一个累计性质的值，保存  

  mov edx,dword ptr [aAcumulate-4]  

  add edi,esi  

  xor edx,esi  

  and edx,edi  

  xor edx,dword ptr [aAcumulate-4]  

  add edx,dword ptr [aAcumulate-18h]  

  lea ebx,dword ptr [ebx+edx+0FFFF5BB1h]  

  mov edx,ebx  

  shr edx,0Fh  

  shl ebx,11h  

  or edx,ebx  

  mov ebx,dword ptr [eax+2C0h]  

  mov dword ptr [aAcumulate-24h],ebx  

  mov ebx,edi  

  xor ebx,esi  

  add edx,edi  

  and ebx,edx  

  mov dword ptr [aAcumulate+0Ch],edx  

  mov edx,dword ptr [aAcumulate-4]  

  xor ebx,esi  

  add ebx,dword ptr [aAcumulate-24h]  

  lea edx,dword ptr [edx+ebx+895CD7BEh]  

  mov ebx,edx  

  shl ebx,16h  

  shr edx,0Ah  

  or ebx,edx  

  mov edx,dword ptr [eax+30h]                   ;  向后取用户名 4 个字母  

  add ebx,dword ptr [aAcumulate+0Ch]  

  mov dword ptr [aAcumulate-30h],edx            ;  作为一个累计性质的值，保存  

  mov dword ptr [aAcumulate-4],ebx  

  mov ebx,edi  

  xor ebx,dword ptr [aAcumulate+0Ch]  

  and ebx,dword ptr [aAcumulate-4]  

  xor ebx,edi  

  add ebx,edx  

  lea esi,dword ptr [esi+ebx+6B901122h]  

  mov ebx,dword ptr [aAcumulate+0Ch]  

  xor ebx,dword ptr [aAcumulate-4]  

  mov edx,esi  

  shr edx,19h  

  shl esi,7  

  or edx,esi  

  mov esi,dword ptr [eax+34h]                  ;  向后取用户名 4 个字母  

  add edx,dword ptr [aAcumulate-4]  

  mov dword ptr [aAcumulate-3Ch],esi            ;  作为一个累计性质的值，保存  

  and ebx,edx  

  xor ebx,dword ptr [aAcumulate+0Ch]  

  add ebx,esi  

  mov esi,dword ptr [eax+38h]                  ;  向后取用户名 4 个字母  

  mov dword ptr [aAcumulate-10h],esi           ;  作为一个累计性质的值，保存  

  mov esi,dword ptr [aAcumulate-4]  

  lea edi,dword ptr [edi+ebx+0FD987193h]  

  xor esi,edx  

  mov ebx,edi  

  mov eax,dword ptr [eax+3Ch]  

  shr ebx,14h  

  shl edi,0Ch  

  or ebx,edi  

  mov edi,dword ptr [aAcumulate+0Ch]  

  add ebx,edx  

  mov dword ptr [aAcumulate-38h],eax  

  and esi,ebx  

  mov dword ptr [aAcumulate+8],ebx  

  xor esi,dword ptr [aAcumulate-4]  

  add esi,dword ptr [aAcumulate-10h]  

  lea edi,dword ptr [edi+esi+0A679438Eh]  

  mov esi,edi  

  shr esi,0Fh  

  shl edi,11h  

  or esi,edi  

  mov edi,ebx  

  add esi,ebx  

  xor edi,edx  

  and edi,esi  

  xor edi,edx  

  add edi,eax  

  mov eax,dword ptr [aAcumulate-4]  

  lea eax,dword ptr [eax+edi+49B40821h]  

  mov edi,eax  

  shl edi,16h  

  shr eax,0Ah  

  or edi,eax  

  mov eax,esi  

  add edi,esi  

  xor eax,edi  

  and eax,ebx  

  xor eax,esi  

  add eax,dword ptr [aAcumulate-1Ch]  

  lea edx,dword ptr [edx+eax+0F61E2562h]  

  mov eax,edx  

  shr eax,1Bh  

  shl edx,5  

  or eax,edx  

  mov edx,edi  

  add eax,edi  

  xor edx,eax  

  and edx,esi  

  xor edx,edi  

  add edx,dword ptr [aAcumulate-20h]  

  lea ebx,dword ptr [ebx+edx+0C040B340h]  

  mov edx,ebx  

  shr edx,17h  

  shl ebx,9  

  or edx,ebx  

  add edx,eax  

  mov dword ptr [aAcumulate+8],edx  

  xor edx,eax  

  and edx,edi  

  mov ebx,dword ptr [aAcumulate+8]  

  xor edx,eax  

  add edx,dword ptr [aAcumulate-24h]  

  lea esi,dword ptr [esi+edx+265E5A51h]  

  mov edx,esi  

  shr edx,12h  

  shl esi,0Eh  

  or edx,esi  

  mov esi,dword ptr [aAcumulate+8]  

  add edx,dword ptr [aAcumulate+8]  

  xor esi,edx  

  and esi,eax  

  xor esi,dword ptr [aAcumulate+8]  

  add esi,dword ptr [aAcumulate-8]  

  lea edi,dword ptr [edi+esi+0E9B6C7AAh]  

  mov esi,edi  

  shl esi,14h  

  shr edi,0Ch  

  or esi,edi  

  mov edi,edx  

  add esi,edx  

  xor edi,esi  

  and edi,dword ptr [aAcumulate+8]  

  xor edi,edx  

  add edi,dword ptr [aAcumulate-2Ch]  

  lea eax,dword ptr [eax+edi+0D62F105Dh]  

  mov edi,eax  

  shr edi,1Bh  

  shl eax,5  

  or edi,eax  

  mov eax,esi  

  add edi,esi  

  xor eax,edi  

  and eax,edx  

  xor eax,esi  

  add eax,dword ptr [aAcumulate-18h]  

  lea ebx,dword ptr [ebx+eax+2441453h]  

  mov eax,ebx  

  shr eax,17h  

  shl ebx,9  

  or eax,ebx  

  add eax,edi  

  mov dword ptr [aAcumulate+8],eax  

  xor eax,edi  

  and eax,esi  

  xor eax,edi  

  add eax,dword ptr [aAcumulate-38h]  

  lea edx,dword ptr [edx+eax+0D8A1E681h]  

  mov eax,edx  

  shr eax,12h  

  shl edx,0Eh  

  or eax,edx  

  mov edx,dword ptr [aAcumulate+8]  

  add eax,dword ptr [aAcumulate+8]  

  xor edx,eax  

  and edx,edi  

  xor edx,dword ptr [aAcumulate+8]  

  add edx,dword ptr [aAcumulate-40h]  

  lea esi,dword ptr [esi+edx+0E7D3FBC8h]  

  mov edx,esi  

  shl edx,14h  

  shr esi,0Ch  

  or edx,esi  

  mov esi,eax  

  add edx,eax  

  mov ebx,dword ptr [aAcumulate+8]  

  xor esi,edx  

  and esi,dword ptr [aAcumulate+8]  

  xor esi,eax  

  add esi,dword ptr [aAcumulate-44h]  

  lea edi,dword ptr [edi+esi+21E1CDE6h]  

  mov esi,edi  

  shr esi,1Bh  

  shl edi,5  

  or esi,edi  

  mov edi,edx  

  add esi,edx  

  xor edi,esi  

  and edi,eax  

  xor edi,edx  

  add edi,dword ptr [aAcumulate-10h]  

  lea ebx,dword ptr [ebx+edi+0C33707D6h]  

  mov edi,ebx  

  shr edi,17h  

  shl ebx,9  

  or edi,ebx  

  add edi,esi  

  mov dword ptr [aAcumulate+8],edi  

  xor edi,esi  

  and edi,edx  

  mov ebx,dword ptr [aAcumulate+8]  

  xor edi,esi  

  add edi,dword ptr [aAcumulate-14h]  

  lea eax,dword ptr [eax+edi+0F4D50D87h]  

  mov edi,eax  

  shr edi,12h  
```
