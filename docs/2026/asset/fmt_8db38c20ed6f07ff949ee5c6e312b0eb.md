
```
    00930040  0E 1F BA 0E 00 B4 09 CD 21 B8 01 4C CD 21 54 68  ?.???L?Th
    00930050  69 73 20 70 72 6F 67 72 61 6D 20 63 61 6E 6E 6F  is program canno
    00930060  74 20 62 65 20 72 75 6E 20 69 6E 20 44 4F 53 20  t be run in DOS
    00930070  6D 6F 64 65 2E 0D 0D 0A 24 00 00 00 00 00 00 00  mode....$.......
    00930080  85 BB 29 40 C1 DA 47 13 C1 DA 47 13 C1 DA 47 13  吇)@邻G邻G邻G
    00930090  E6 1C 3A 13 C0 DA 47 13 A3 C5 54 13 C3 DA 47 13  ?:磊GＥT泌G
    009300A0  02 D5 1A 13 CE DA 47 13 C1 DA 46 13 FB DA 47 13  ?乌G邻FG

    // pe\_bound解密完成后，继续修改自身
    //修改自身内存属性,从基址开始4000000

    00921F0F    6A 40           push 40
    00921F11    8B8D C0FEFFFF   mov ecx,dword ptr ss:[ebp-140]
    00921F17    8B51 38         mov edx,dword ptr ds:[ecx+38]
    00921F1A    52              push edx
    00921F1B    8B85 48FFFFFF   mov eax,dword ptr ss:[ebp-B8]
    00921F21    50              push eax
    00921F22    8B8D CCFEFFFF   mov ecx,dword ptr ss:[ebp-134]
    00921F28    8B51 14         mov edx,dword ptr ds:[ecx+14]
    00921F2B    FFD2            call edx       //VirtualProtect

    // 拷贝930000这块内存的0 ~ 0x400 空间的数据到自身 0x400000 ~ 0x400400

    00921F53    8B8D D8FDFFFF   mov ecx,dword ptr ss:[ebp-228]
    00921F59    83C1 01         add ecx,1
    00921F5C    898D D8FDFFFF   mov dword ptr ss:[ebp-228],ecx
    00921F62    8B95 D8FDFFFF   mov edx,dword ptr ss:[ebp-228]
    00921F68    3B55 D4         cmp edx,dword ptr ss:[ebp-2C]
    00921F6B    73 1B           jnb short 00921F88
    00921F6D    8B45 DC         mov eax,dword ptr ss:[ebp-24]
    00921F70    0385 D8FDFFFF   add eax,dword ptr ss:[ebp-228]
    00921F76    8B8D F0FDFFFF   mov ecx,dword ptr ss:[ebp-210]
    00921F7C    038D D8FDFFFF   add ecx,dword ptr ss:[ebp-228]
    00921F82    8A11            mov dl,byte ptr ds:[ecx]
    00921F84    8810            mov byte ptr ds:[eax],dl
    00921F86  ^ EB CB           jmp short 00921F53

    // 0x400 后的数据按节的大小单独copy

    00921FBC    3985 D4FDFFFF   cmp dword ptr ss:[ebp-22C],eax  // 比较当前处理的节数量是否等于节总数
    00921FC2    0F8D E8000000   jge 009220B0
    00921FC8    8B8D DCFDFFFF   mov ecx,dword ptr ss:[ebp-224]
    00921FCE    0FB751 02       movzx edx,word ptr ds:[ecx+2]
    00921FD2    83EA 01         sub edx,1
    00921FD5    3995 D4FDFFFF   cmp dword ptr ss:[ebp-22C],edx
    00921FDB    75 1A           jnz short 00921FF7
    00921FDD    8B85 C0FEFFFF   mov eax,dword ptr ss:[ebp-140]
    00921FE3    8B8D ECFDFFFF   mov ecx,dword ptr ss:[ebp-214]
    00921FE9    8B50 38         mov edx,dword ptr ds:[eax+38]
    00921FEC    2B51 0C         sub edx,dword ptr ds:[ecx+C]
    00921FEF    8995 9CFDFFFF   mov dword ptr ss:[ebp-264],edx
    00921FF5    EB 18           jmp short 0092200F
    00921FF7    8B85 ECFDFFFF   mov eax,dword ptr ss:[ebp-214]
    00921FFD    8B8D ECFDFFFF   mov ecx,dword ptr ss:[ebp-214]
    00922003    8B50 34         mov edx,dword ptr ds:[eax+34]
    00922006    2B51 0C         sub edx,dword ptr ds:[ecx+C]
    00922009    8995 9CFDFFFF   mov dword ptr ss:[ebp-264],edx
    0092200F    8B85 9CFDFFFF   mov eax,dword ptr ss:[ebp-264]
    00922015    8985 CCFDFFFF   mov dword ptr ss:[ebp-234],eax
    0092201B    8B8D ECFDFFFF   mov ecx,dword ptr ss:[ebp-214]
    00922021    8B55 DC         mov edx,dword ptr ss:[ebp-24]
    00922024    0351 0C         add edx,dword ptr ds:[ecx+C]
    00922027    8995 D0FDFFFF   mov dword ptr ss:[ebp-230],edx
    0092202D    C785 B8FDFFFF 0>mov dword ptr ss:[ebp-248],0
    00922037    33C0            xor eax,eax
    00922039    8985 BCFDFFFF   mov dword ptr ss:[ebp-244],eax
    0092203F    8985 C0FDFFFF   mov dword ptr ss:[ebp-240],eax
    00922045    8985 C4FDFFFF   mov dword ptr ss:[ebp-23C],eax
    0092204B    8985 C8FDFFFF   mov dword ptr ss:[ebp-238],eax
    00922051    8B8D D0FDFFFF   mov ecx,dword ptr ss:[ebp-230]
    00922057    898D B4FDFFFF   mov dword ptr ss:[ebp-24C],ecx
    0092205D    8B95 ECFDFFFF   mov edx,dword ptr ss:[ebp-214]
    00922063    8B42 10         mov eax,dword ptr ds:[edx+10]
    00922066    3B85 CCFDFFFF   cmp eax,dword ptr ss:[ebp-234]   // 比较 SizeOfRawData 的大小，做copy前准备
    0092206C    76 0E           jbe short 0092207C
    0092206E    8B8D CCFDFFFF   mov ecx,dword ptr ss:[ebp-234]
    00922074    898D 6CFEFFFF   mov dword ptr ss:[ebp-194],ecx
    0092207A    EB 0F           jmp short 0092208B
    0092207C    8B95 ECFDFFFF   mov edx,dword ptr ss:[ebp-214]
    00922082    8B42 10         mov eax,dword ptr ds:[edx+10]
    00922085    8985 6CFEFFFF   mov dword ptr ss:[ebp-194],eax
    0092208B    8B8D 6CFEFFFF   mov ecx,dword ptr ss:[ebp-194]
    00922091    51              push ecx                         // 目的数据，就是自身的节偏移
    00922092    8B95 ECFDFFFF   mov edx,dword ptr ss:[ebp-214]
    00922098    8B45 D8         mov eax,dword ptr ss:[ebp-28]
    0092209B    0342 14         add eax,dword ptr ds:[edx+14]
    0092209E    50              push eax                         // 源数据，就是alloc 的那个buff 930000 + 节偏移
    0092209F    8B8D D0FDFFFF   mov ecx,dword ptr ss:[ebp-230]
    009220A5    51              push ecx                         // copy数据大小
    009220A6    E8 55DFFFFF     call 00920000                    // copy数据到自身
    009220AB  ^ E9 E4FEFFFF     jmp 00921F94

   // 查找自身的映像数据也就是0x400000，如果是跳到下方，做一系列修改操作

    0092225A    E8 61F2FFFF     call 009214C0                    // eax 中 获得PEB 基址
    0092225F    8945 94         mov dword ptr ss:[ebp-6C],eax
    00922262    8B45 94         mov eax,dword ptr ss:[ebp-6C]
    00922265    8B8D 48FFFFFF   mov ecx,dword ptr ss:[ebp-B8]
    0092226B    8948 08         mov dword ptr ds:[eax+8],ecx     // 修改PEB 中的ImageBaseAddress 数据为0x400000
    0092226E    E8 4DF2FFFF     call 009214C0                    // eax 中 获得PEB 基址
    00922273    8B50 0C         mov edx,dword ptr ds:[eax+C]     // PEB.Ldr
    00922276    83C2 0C         add edx,0C                       // PEB.Ldr.InLoadOrderModuleList
    00922279    8995 30FFFFFF   mov dword ptr ss:[ebp-D0],edx
    0092227F    8B85 30FFFFFF   mov eax,dword ptr ss:[ebp-D0]
    00922285    8B08            mov ecx,dword ptr ds:[eax]
    00922287    898D 28FEFFFF   mov dword ptr ss:[ebp-1D8],ecx
    0092228D    8B95 28FEFFFF   mov edx,dword ptr ss:[ebp-1D8]
    00922293    3B95 30FFFFFF   cmp edx,dword ptr ss:[ebp-D0]    // 遍历链表模块信息，查找自身模块
    00922299   /74 57           je short 009222F2
    0092229B   |8B85 28FEFFFF   mov eax,dword ptr ss:[ebp-1D8]
    009222A1   |8985 34FFFFFF   mov dword ptr ss:[ebp-CC],eax
    009222A7   |8B8D 34FFFFFF   mov ecx,dword ptr ss:[ebp-CC]
    009222AD   |8B51 18         mov edx,dword ptr ds:[ecx+18]
    009222B0   |3B95 48FFFFFF   cmp edx,dword ptr ss:[ebp-B8]    // 比较是不是自身模块0x400000
    009222B6   |75 2A           jnz short 009222E2
    009222B8   |8B85 C0FEFFFF   mov eax,dword ptr ss:[ebp-140]
    009222BE   |8B8D 48FFFFFF   mov ecx,dword ptr ss:[ebp-B8]
    009222C4   |0348 10         add ecx,dword ptr ds:[eax+10]    // 获得bound\_pe的入口点，但这个是经过修正的，已经被改为0x407f30
    009222C7   |8B95 34FFFFFF   mov edx,dword ptr ss:[ebp-CC]
    009222CD   |894A 1C         mov dword ptr ds:[edx+1C],ecx    // 改写原自身的入口点0x40109c为0x407f30
    009222D0   |8B85 34FFFFFF   mov eax,dword ptr ss:[ebp-CC]    // eax 保存当前 \_LDR\_DATA\_TABLE\_ENTRY 信息
    009222D6   |8B8D C0FEFFFF   mov ecx,dword ptr ss:[ebp-140]
    009222DC   |8B51 38         mov edx,dword ptr ds:[ecx+38]    // SizeOfImage
    009222DF   |8950 20         mov dword ptr ds:[eax+20],edx    // 修改SizeOfImage 为0xA000，原为0x19000
    009222E2   |8B85 28FEFFFF   mov eax,dword ptr ss:[ebp-1D8]
    009222E8   |8B08            mov ecx,dword ptr ds:[eax]
    009222EA   |898D 28FEFFFF   mov dword ptr ss:[ebp-1D8],ecx
    009222F0  ^|EB 9B           jmp short 0092228D               // 如果没有发现继续next

    // 找到，跳出了链表循环

    009222F2    8D95 70FEFFFF   lea edx,dword ptr ss:[ebp-190]
    009222F8    52              push edx
```
