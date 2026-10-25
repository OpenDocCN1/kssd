
`mov eax, dword ptr [ebp-08]`

`:0057FEE5 E85A50E8FF`  
`call 00404F44`

`:0057FEEA 8BD0`  
`mov edx, eax`

`:0057FEEC 2BD6`  
`sub edx, esi`

`:0057FEEE B901000000`  
`mov ecx, 00000001`

`:0057FEF3 8B45F8`  
`mov eax, dword ptr [ebp-08]`

`:0057FEF6 E8A152E8FF`  
`call 0040519C`

`:0057FEFB 8B55E8`  
`mov edx, dword ptr [ebp-18]`

`:0057FEFE 8D45FC`  
`lea eax, dword ptr [ebp-04]`

`:0057FF01 E84650E8FF`  
`call 00404F4C`

`:0057FF06 46`  
`inc esi`

`:0057FF07 4B`  
`dec ebx`

`:0057FF08 75D4`  
`jne 0057FEDE`

`====>这个循环把858912倒序排列！`

`*`  
`Referenced by a (U)nconditional or (C)onditional Jump at Address:`  
`|:0057FED9(C)`  
`|`

`:0057FF0A 8B45FC`  
`mov eax, dword ptr [ebp-04]`

`====>EAX=219858`

`:0057FF0D E8229FE8FF`  
`call 00409E34`

`====>取219858的16进制值`

`:0057FF12 8945F0`  
`mov dword ptr [ebp-10], eax`

`====>[ebp-10]=00035AD2（H）=219858（D）`

`:0057FF15 8D45F8`  
`lea eax, dword ptr [ebp-08]`

`:0057FF18 E86F4DE8FF`  
`call 00404C8C`

`:0057FF1D 8B45EC`  
`mov eax, dword ptr [ebp-14]`

`:0057FF20 E81F50E8FF`  
`call 00404F44`

`:0057FF25 8BD8`  
`mov ebx, eax`

`:0057FF27 85DB`  
`test ebx, ebx`

`:0057FF29 0F8CDA000000`  
`jl 00580009`

`:0057FF2F 43`  
`inc ebx`

`:0057FF30 33F6`  
`xor esi, esi`

`*`  
`Referenced by a (U)nconditional or (C)onditional Jump at Address:`  
`|:00580003(C)`  
`|`

`:0057FF32 8D45FC`  
`lea eax, dword ptr [ebp-04]`

`:0057FF35 50`  
`push eax`

`:0057FF36 8B45EC`  
`mov eax, dword ptr [ebp-14]`

`:0057FF39 E80650E8FF`  
`call 00404F44`

`:0057FF3E 8BD0`  
`mov edx, eax`

`:0057FF40 2BD6`  
`sub edx, esi`

`:0057FF42 B901000000`  
`mov ecx, 00000001`

`:0057FF47 8B45EC`  
`mov eax, dword ptr [ebp-14]`

`:0057FF4A E84D52E8FF`  
`call 0040519C`

`:0057FF4F 8B45FC`  
`mov eax, dword ptr [ebp-04]`

`:0057FF52 BAC4005800`  
`mov edx, 005800C4`

`:0057FF57 E82C51E8FF`  
`call 00405088`

`:0057FF5C 0F8487000000`  
`je 0057FFE9`

`:0057FF62 8B45FC`  
`mov eax, dword ptr [ebp-04]`

`:0057FF65 BAD0005800`  
`mov edx, 005800D0`

`:0057FF6A E81951E8FF`  
`call 00405088`

`:0057FF6F 7478`  
`je 0057FFE9`

`:0057FF71 8B45FC`  
`mov eax, dword ptr [ebp-04]`

`:0057FF74 BADC005800`  
`mov edx, 005800DC`

`:0057FF79 E80A51E8FF`  
`call 00405088`

`:0057FF7E 7469`  
`je 0057FFE9`

`:0057FF80 8B45FC`  
`mov eax, dword ptr [ebp-04]`

`:0057FF83 BAE8005800`  
`mov edx, 005800E8`

`:0057FF88 E8FB50E8FF`  
`call 00405088`

`:0057FF8D 745A`  
`je 0057FFE9`

`:0057FF8F 8B45FC`  
`mov eax, dword ptr [ebp-04]`

`:0057FF92 BAE8005800`  
`mov edx, 005800E8`

`:0057FF97 E8EC50E8FF`  
`call 00405088`

`:0057FF9C 744B`  
`je 0057FFE9`

`:0057FF9E 8B45FC`  
`mov eax, dword ptr [ebp-04]`

`:0057FFA1 BAF4005800`  
`mov edx, 005800F4`

`:0057FFA6 E8DD50E8FF`  
`call 00405088`

`:0057FFAB 743C`  
`je 0057FFE9`

`:0057FFAD 8B45FC`  
`mov eax, dword ptr [ebp-04]`

`:0057FFB0 BA00015800`  
`mov edx, 00580100`

`:0057FFB5 E8CE50E8FF`  
`call 00405088`

`:0057FFBA 742D`  
`je 0057FFE9`

`:0057FFBC 8B45FC`  
`mov eax, dword ptr [ebp-04]`

`:0057FFBF BA0C015800`  
`mov edx, 0058010C`

`:0057FFC4 E8BF50E8FF`  
`call 00405088`

`:0057FFC9 741E`  
`je 0057FFE9`

`:0057FFCB 8B45FC`  
`mov eax, dword ptr [ebp-04]`

`:0057FFCE BA18015800`  
`mov edx, 00580118`

`:0057FFD3 E8B050E8FF`  
`call 00405088`

`:0057FFD8 740F`  
`je 0057FFE9`

`:0057FFDA 8B45FC`  
`mov eax, dword ptr [ebp-04]`

`:0057FFDD BA24015800`  
`mov edx, 00580124`

`:0057FFE2 E8A150E8FF`  
`call 00405088`

`:0057FFE7 7518`  
`jne 00580001`

`* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:`  
`|:0057FF5C(C), :0057FF6F(C),`  
`:0057FF7E(C), :0057FF8D(C), :0057FF9C(C)`  
`|:0057FFAB(C), :0057FFBA(C), :0057FFC9(C),`  
`:0057FFD8(C)`  
`|`  

`:0057FFE9 8D45F8`  
`lea eax, dword ptr [ebp-08]`

`:0057FFEC 8B55FC`  
`mov edx, dword ptr [ebp-04]`

`:0057FFEF E8584FE8FF`  
`call 00404F4C`

`:0057FFF4 8B45F8`  
`mov eax, dword ptr [ebp-08]`

`:0057FFF7 E8484FE8FF`  
`call 00404F44`

`:0057FFFC 83F806`  
`cmp eax, 00000006`

`:0057FFFF 7408`  
`je 00580009`

`*`  
`Referenced by a (U)nconditional or (C)onditional Jump at Address:`  
`|:0057FFE7(C)`  
`|`  

`:00580001 46`  
`inc esi`

`:00580002 4B`  
`dec ebx`

`:00580003 0F8529FFFFFF`  
`jne 0057FF32`

`====>呵呵，这循环其实是倒序取用户号中大于0的数字！`

`*`  
`Referenced by a (U)nconditional or (C)onditional Jump at Addresses:`  
`|:0057FF29(C),`  
`:0057FFFF(C)`  
`|`  

`:00580009 837DF800`  
`cmp dword ptr [ebp-08], 00000000`

`:0058000D 740D`  
`je 0058001C`

`:0058000F 8B45F8`  
`mov eax, dword ptr [ebp-08]`

`====>EAX=[ebp-08]=219858`

`:00580012 E81D9EE8FF`  
`call 00409E34`

`====>取219858的16进制值`

`:00580017 8945F4`  
`mov dword ptr [ebp-0C], eax`

`====>[ebp-0C]=00035AD2（H）=219858（D）`

`:0058001A EB07`  
`jmp 00580023`

`* Referenced by a (U)nconditional or (C)onditional Jump at Address:`  
`|:0058000D(C)`  
`|`  

`:0058001C C745F4D89B0A00`  
`mov [ebp-0C], 000A9BD8`

`*`  
`Referenced by a (U)nconditional or (C)onditional Jump at Address:`  
`|:0058001A(U)`  
`|`  

`:00580023 8D45FC`  
`lea eax, dword ptr [ebp-04]`

`:00580026 E8614CE8FF`  
`call 00404C8C`

`:0058002B 8B45F8`  
`mov eax, dword ptr [ebp-08]`

`:0058002E E8114FE8FF`  
`call 00404F44`

`:00580033 8BD8`  
`mov ebx, eax`

`:00580035 4B`  
`dec ebx`

`:00580036 85DB`  
`test ebx, ebx`

`:00580038 7C2F`  
`jl 00580069`

`:0058003A 43`  
`inc ebx`

`:0058003B 33F6`  
`xor esi, esi`

`* Referenced by a (U)nconditional or (C)onditional Jump at Address:`  
`|:00580067(C)`  
`|`  

`:0058003D 8D45E4`  
`lea eax, dword ptr [ebp-1C]`

`:00580040 50`  
`push eax`

`:00580041 8B45F8`  
`mov eax, dword ptr [ebp-08]`

`:00580044 E8FB4EE8FF`  
`call 00404F44`

`:00580049 8BD0`  
`mov edx, eax`

`:0058004B 2BD6`  
`sub edx, esi`

`:0058004D B901000000`  
`mov ecx, 00000001`

`:00580052 8B45F8`  
`mov eax, dword ptr [ebp-08]`

`:00580055 E84251E8FF`  
`call 0040519C`

`:0058005A 8B55E4`  
`mov edx, dword ptr [ebp-1C]`

`:0058005D 8D45FC`  
`lea eax, dword ptr [ebp-04]`

`:00580060 E8E74EE8FF`  
`call 00404F4C`

`:00580065 46`  
`inc esi`

`:00580066 4B`  
`dec ebx`

`:00580067 75D4`  
`jne 0058003D`

`====>这个循环把219858倒序排列！晕，为什么重复。`

`*`  
`Referenced by a (U)nconditional or (C)onditional Jump at Address:`  
`|:00580038(C)`  
`|`  

`:00580069 8B45FC`  
`mov eax, dword ptr [ebp-04]`

`====>EAX=858921`

`:0058006C E8C39DE8FF`  
`call 00409E34`

`====>取858912的16进制值=000D1B20`

`:00580071 6BF779`  
`imul esi, edi, 00000079`

`====>ESI=000D1B20 *`  
`79=0631D220`

`:00580074 6B55F40B`  
`imul edx, dword ptr [ebp-0C], 0000000B`

`====>EDX=00035AD2`  
`* 0B=0024E706`

`:00580078 03F2`  
`add esi, edx`

`====>ESI=0631D220 + 0024E706=0656B926`

`:0058007A 0375F0`  
`add esi, dword ptr [ebp-10]`

`====>ESI=0656B926 + 00035AD2=065A13F8`

`:0058007D 03F0`  
`add esi, eax`

`====>ESI=065A13F8 + 000D1B20=06672F18`  
`====>06672F18的10进制值107425560就是我的注册码了！`

`:0058007F 8BDE`  
`mov ebx, esi`

`:00580081 33C0`  
`xor eax, eax`

`:00580083 5A`  
`pop edx`

`:00580084 59`  
`pop ecx`

`:00580085 59`  
`pop`
