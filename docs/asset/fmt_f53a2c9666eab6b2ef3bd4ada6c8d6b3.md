
`0053CAF8 | .^ 75>\\jnz short Micro.0053CACE ; 这一段是将上面机器码 1-9 的数字组合反过来`

`0053CAFA | > 8B>mov eax,dword ptr ss:[ebp-4] ; 取出反过来的数字`

`0053CAFD | . E8>call Micro.004B88B0 ; 将反过来的数字转化为 16 进制数据`

`0053CB02 | . 89>mov dword ptr ss:[ebp-10],eax ; 储存在这里`

`0053CB05 | . 8D>lea eax,dword ptr ss:[ebp-8]`

`0053CB08 | . E8>call Micro.004B4050`

`0053CB0D | . 8B>mov eax,dword ptr ss:[ebp-14] ; 又次得到机器码`

`0053CB10 | . E8>call Micro.004B4310`

`0053CB15 | . 8B>mov ebx,eax`

`0053CB17 | . 85>test ebx,ebx`

`0053CB19 | . 0F>jl Micro.0053CBF9`

`0053CB1F | . 43 inc ebx`

`0053CB20 | . 33>xor esi,esi`

`0053CB22 | > 8D>/lea eax,dword ptr ss:[ebp-4]`

`0053CB25 | . 50 |push eax`

`0053CB26 | . 8B>|mov eax,dword ptr ss:[ebp-14]`

`0053CB29 | . E8>|call Micro.004B4310`

`0053CB2E | . 8B>|mov edx,eax`

`0053CB30 | . 2B>|sub edx,esi`

`0053CB32 | . B9>|mov ecx,1`

`0053CB37 | . 8B>|mov eax,dword ptr ss:[ebp-14]`

`0053CB3A | . E8>|call Micro.004B4570`

`0053CB3F | . 8B>|mov eax,dword ptr ss:[ebp-4]`

`0053CB42 | . BA>|mov edx,Micro.0053CCB4`

`0053CB47 | . E8>|call Micro.004B445C`

`0053CB4C | . 0F>|je Micro.0053CBD9`

`0053CB52 | . 8B>|mov eax,dword ptr ss:[ebp-4]`

`0053CB55 | . BA>|mov edx,Micro.0053CCC0`

`0053CB5A | . E8>|call Micro.004B445C`

`0053CB5F | . 74>|je short Micro.0053CBD9`

`0053CB61 | . 8B>|mov eax,dword ptr ss:[ebp-4]`

`0053CB64 | . BA>|mov edx,Micro.0053CCCC`

`0053CB69 | . E8>|call Micro.004B445C`

`0053CB6E | . 74>|je short Micro.0053CBD9`

`0053CB70 | . 8B>|mov eax,dword ptr ss:[ebp-4]`

`0053CB73 | . BA>|mov edx,Micro.0053CCD8`

`0053CB78 | . E8>|call Micro.004B445C`

`0053CB7D | . 74>|je short Micro.0053CBD9`

`0053CB7F | . 8B>|mov eax,dword ptr ss:[ebp-4]`

`0053CB82 | . BA>|mov edx,Micro.0053CCD8`

`0053CB87 | . E8>|call Micro.004B445C`

`0053CB8C | . 74>|je short Micro.0053CBD9`

`0053CB8E | . 8B>|mov eax,dword ptr ss:[ebp-4]`

`0053CB91 | . BA>|mov edx,Micro.0053CCE4`

`0053CB96 | . E8>|call Micro.004B445C`

`0053CB9B | . 74>|je short Micro.0053CBD9`

`0053CB9D | . 8B>|mov eax,dword ptr ss:[ebp-4]`

`0053CBA0 | . BA>|mov edx,Micro.0053CCF0`

`0053CBA5 | . E8>|call Micro.004B445C`

`0053CBAA | . 74>|je short Micro.0053CBD9`

`0053CBAC | . 8B>|mov eax,dword ptr ss:[ebp-4]`

`0053CBAF | . BA>|mov edx,Micro.0053CCFC`

`0053CBB4 | . E8>|call Micro.004B445C`

`0053CBB9 | . 74>|je short Micro.0053CBD9`

`0053CBBB | . 8B>|mov eax,dword ptr ss:[ebp-4]`

`0053CBBE | . BA>|mov edx,Micro.0053CD08`

`0053CBC3 | . E8>|call Micro.004B445C`

`0053CBC8 | . 74>|je short Micro.0053CBD9`

`0053CBCA | . 8B>|mov eax,dword ptr ss:[ebp-4]`

`0053CBCD | . BA>|mov edx,Micro.0053CD14`

`0053CBD2 | . E8>|call Micro.004B445C`

`0053CBD7 | . 75>|jnz short Micro.0053CBF1`

`0053CBD9 | > 8D>|lea eax,dword ptr ss:[ebp-8]`

`0053CBDC | . 8B>|mov edx,dword ptr ss:[ebp-4]`

`0053CBDF | . E8>|call Micro.004B4318`

`0053CBE4 | . 8B>|mov eax,dword ptr ss:[ebp-8]`

`0053CBE7 | . E8>|call Micro.004B4310`

`0053CBEC | . 83>|cmp eax,6`

`0053CBEF | . 74>|je short Micro.0053CBF9`

`0053CBF1 | > 46 |inc esi`

`0053CBF2 | . 4B |dec ebx`

`0053CBF3 | .^ 0F>\\jnz Micro.0053CB22`

`0053CBF9 | > 83>cmp dword ptr ss:[ebp-8],0 ; 得到反过来的机器码中的 1-9 的数字`

`0053CBFD | . 74>je short Micro.0053CC0C`

`0053CBFF | . 8B>mov eax,dword ptr ss:[ebp-8]`

`0053CC02 | . E8>call Micro.004B88B0 ; 将反过来的数字转化为 16 进制数据 00035474`

`0053CC07 | . 89>mov dword ptr ss:[ebp-C],eax ; 储存`

`0053CC0A | . EB>jmp short Micro.0053CC13`

`0053CC0C | > C7>mov dword ptr ss:[ebp-C],0A9BD8`

`0053CC13 | > 8D>lea eax,dword ptr ss:[ebp-4]`

`0053CC16 | . E8>call Micro.004B4050`

`0053CC1B | . 8B>mov eax,dword ptr ss:[ebp-8]`

`0053CC1E | . E8>call Micro.004B4310`

`0053CC23 | . 8B>mov ebx,eax`

`0053CC25 | . 4B dec ebx`

`0053CC26 | . 85>test ebx,ebx`

`0053CC28 | . 7C>jl short Micro.0053CC59`

`0053CC2A | . 43 inc ebx`

`0053CC2B | . 33>xor esi,esi`

`0053CC2D | > 8D>/lea eax,dword ptr ss:[ebp-1C]`

`0053CC30 | . 50 |push eax`

`0053CC31 | . 8B>|mov eax,dword ptr ss:[ebp-8]`

`0053CC34 | . E8>|call Micro.004B4310`

`0053CC39 | . 8B>|mov edx,eax`

`0053CC3B | . 2B>|sub edx,esi`

`0053CC3D | . B9>|mov ecx,1`

`0053CC42 | . 8B>|mov eax,dword ptr ss:[ebp-8]`

`0053CC45 | . E8>|call Micro.004B4570`

`0053CC4A | . 8B>|mov edx,dword ptr ss:[ebp-1C]`

`0053CC4D | . 8D>|lea eax,dword ptr ss:[ebp-4]`

`0053CC50 | . E8>|call Micro.004B4318`

`0053CC55 | . 46 |inc esi`

`0053CC56 | . 4B |dec ebx`

`0053CC57 | .^ 75>\\jnz short Micro.0053CC2D ; 这个循环又是将反过来的机器码中的 1-9 数字再次反过来`

`0053CC59 | > 8B>mov eax,dword ptr ss:[ebp-4] ; 得到正过来的机器码中的 1-9 的数字`

`0053CC5C | . E8>call Micro.004B88B0 ; 将正过来的机器码中的 1-9 的数字转化为 16 进制 000C8E1C`

`0053CC61 | . 6B>imul esi,edi,79 ; 将正过来的机器码中的 1-9 的数字转化为 16 进制乘以 79`

`0053CC64 | . 6B>imul edx,dword ptr ss:[ebp-C],0B ; 将反过来机器码中 1-9 的数字转化为 16 进制的数据乘以 0B`

`0053CC68 | . 03>add esi,edx ; 把两次相乘的结果相加`

`0053CC6A | . 03>add esi,dword ptr ss:[ebp-10] ; 结果再加上反过来机器码中 1-9 的数字转化为 16 进制的数据`

`0053CC6D | . 03>add esi,eax ; 结果再加上正过来的机器码中的 1-9 的数字转化为 16 进制 这个就是注册码之一`

`0053CC6F | . 8B>mov ebx,esi`

`0053CC71 | . 33>xor eax,eax`

`0053CC73 | . 5A pop edx`

---

进入 `0053D5C0 . E8>call Micro.0053CD18 ; 这里就是计算第 2 个注册码的地方`

---

`0053CD32 | . 64>mov dword ptr fs:[eax],esp`

`0053CD35 | . 8D>lea eax,dword ptr ss:[ebp-C]`

`0053CD38 | . 8B>mov edx,dword ptr ds:[ebx] ; 取出机器码`

`0053CD3A | . E8>call Micro.004B40E8`

`0053CD3F | . 8D>lea eax,dword ptr ss:[ebp-8]`

`0053CD42 | . E8>call Micro.004B4050`

`0053CD47 | . 8B>mov eax,dword ptr ss:[ebp-C]`

`0053CD4A | . E8>call Micro.004B4310 ; 得到机器码的位数`

`0053CD4F | . 8B>mov ebx,eax`

`0053CD51 | . 85>test ebx,ebx`

`0053CD53 | . 0F>jle Micro.0053CE2B`

`0053CD59 | . BE>mov esi,1`

`0053CD5E | > 8D>/lea eax,dword ptr ss:[ebp-4]`

`0053CD61 | . 50 |push eax`

`0053CD62 | . B9>|mov ecx,1`

`0053CD67 | . 8B>|mov edx,esi`

`0053CD69 | . 8B>|mov eax,dword ptr ss:[ebp-C] ; 取出机器码`

`0053CD6C | . E8>|call Micro.004B4570 ; 依次得到每一位的机器码`

`0053CD71 | . 8B>|mov eax,dword ptr ss:[ebp-4] ; 取出此次机器码的 ASCII 的地址`

`0053CD74 | . BA>|mov edx,Micro.0053D03C ; 固定值 31`

`0053CD79 | . E8>|call Micro.004B445C ; 检查这一位的机器码的 ASCII 是否是 31`

`0053CD7E | . 0F>|je Micro.0053CE0B`

`0053CD84 | . 8B>|mov eax,dword ptr ss:[ebp-4] ; 取出此次机器码的 ASCII 的地址`

`0053CD87 | . BA>|mov edx,Micro.0053D048 ; 固定值 32`

`0053CD8C | . E8>|call Micro.004B445C ; 检查这一位的机器码的 ASCII 是否是 32`

`0053CD91 | . 74>|je short Micro.0053CE0B`

`0053CD93 | . 8B>|mov eax,dword ptr ss:[ebp-4] ; 取出此次机器码的 ASCII 的地址`

`0053CD96 | . BA>|mov edx,Micro.0053D054 ; 固定值 33`

`0053CD9B | . E8>|call Micro.004B445C ; 检查这一位的机器码的 ASCII 是否是 33`

`0053CDA0 | . 74>|je short Micro.0053CE0B`

`0053CDA2 | . 8B>|mov eax,dword ptr ss:[ebp-4] ; 取出此次机器码的 ASCII 的地址`

`0053CDA5 | . BA>|mov edx,Micro.0053D060 ; 固定值 34`

`0053CDAA | . E8>|call Micro.004B445C ; 检查这一位的机器码的 ASCII 是否是 34`

`0053CDAF | . 74>|je short Micro.0053CE0B`

`0053CDB1 | . 8B>|mov eax,dword ptr ss:[ebp-4] ; 取出此次机器码的 ASCII 的地址`

`0053CDB4 | . BA>|mov edx,Micro.0053D060 ; 固定值 34`

`0053CDB9 | . E8>|call Micro.004B445C ; 检查这一位的机器码的 ASCII 是否是 34`

`0053CDBE | . 74>|je short Micro.0053CE0B`

`0053CDC0 | . 8B>|mov eax,dword ptr ss:[ebp-4] ; 取出此次机器码的 ASCII 的地址`

`0053CDC3 | . BA>|mov edx,Micro.0053D06C ; 固定值 35`
