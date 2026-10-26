
```asm
0040B714    8B0B            mov ecx,dword ptr ds:[ebx]        //假注册码  ASCII "9876543210"
0040B716    50              push eax                          //真码入栈  //内存注册机
0040B717    51              push ecx                          //假码入栈
0040B718    FF15 54544100   call dword ptr ds:[<&MSVCRT.\_mbscmp>]      ; msvcrt.\_mbscmp     //调用MFC程序的真假码比对函数
0040B71E    83C4 14         add esp,14
0040B721    8D4C24 20       lea ecx,dword ptr ss:[esp+20]
0040B725    85C0            test eax,eax
0040B727    0F944424 17     sete byte ptr ss:[esp+17]
0040B72C    E8 CD5F0000     call <jmp.&MFC42.#800>
0040B731    8D4C24 18       lea ecx,dword ptr ss:[esp+18]
0040B735    C68424 B0000000>mov byte ptr ss:[esp+B0],6
0040B73D    E8 BC5F0000     call <jmp.&MFC42.#800>
0040B742    8D4C24 10       lea ecx,dword ptr ss:[esp+10]
0040B746    C68424 B0000000>mov byte ptr ss:[esp+B0],5
0040B74E    E8 AB5F0000     call <jmp.&MFC42.#800>
0040B753    8A4424 17       mov al,byte ptr ss:[esp+17]
0040B757    84C0            test al,al                        //al=0
0040B759    0F84 A1000000   je APM.0040B800                   //爆破点
0040B75F    51              push ecx
0040B760    8BCC            mov ecx,esp
0040B762    896424 14       mov dword ptr ss:[esp+14],esp
0040B766    57              push edi
0040B767    E8 52600000     call <jmp.&MFC42.#535>
0040B76C    8D8C24 8C000000 lea ecx,dword ptr ss:[esp+8C]
0040B773    E8 880E0000     call APM.0040C600
0040B778    51              push ecx
0040B779    8BCC            mov ecx,esp
0040B77B    896424 14       mov dword ptr ss:[esp+14],esp
0040B77F    55              push ebp
0040B780    E8 39600000     call <jmp.&MFC42.#535>
0040B785    8D4C24 4C       lea ecx,dword ptr ss:[esp+4C]
0040B789    E8 720E0000     call APM.0040C600
0040B78E    51              push ecx
0040B78F    8BCC            mov ecx,esp
0040B791    896424 14       mov dword ptr ss:[esp+14],esp
0040B795    53              push ebx
0040B796    E8 23600000     call <jmp.&MFC42.#535>
0040B79B    8D4C24 6C       lea ecx,dword ptr ss:[esp+6C]
0040B79F    E8 5C0E0000     call APM.0040C600
0040B7A4    68 02000080     push 80000002
0040B7A9    6A 00           push 0
0040B7AB    6A 00           push 0
0040B7AD    51              push ecx
0040B7AE    8BCC            mov ecx,esp
0040B7B0    896424 20       mov dword ptr ss:[esp+20],esp
0040B7B4    68 ACB54100     push APM.0041B5AC                          ; ASCII "Software\Microsoft\Windows\CurrentVersion\Uninstall\APM"
0040B7B9    E8 12600000     call <jmp.&MFC42.#537>
0040B7BE    8D4C24 38       lea ecx,dword ptr ss:[esp+38]
0040B7C2    E8 E9070000     call APM.0040BFB0
0040B7C7    6A 00           push 0
0040B7C9    8D4C24 2C       lea ecx,dword ptr ss:[esp+2C]
0040B7CD    C68424 B4000000>mov byte ptr ss:[esp+B4],8
0040B7D5    E8 560A0000     call APM.0040C230
0040B7DA    6A 40           push 40
0040B7DC    68 C4B14100     push APM.0041B1C4                          ; ASCII "Printer Monitor"
0040B7E1    68 50BA4100     push APM.0041BA50                          ; ASCII "Printer Monitor registration successful!"
0040B7E6    8BCE            mov ecx,esi
0040B7E8    E8 C55F0000     call <jmp.&MFC42.#4224>
0040B7ED    8D4C24 28       lea ecx,dword ptr ss:[esp+28]
0040B7F1    C68424 B0000000>mov byte ptr ss:[esp+B0],5
0040B7F9    E8 E2080000     call APM.0040C0E0
0040B7FE    EB 13           jmp short APM.0040B813
0040B800    6A 30           push 30
0040B802    68 C4B14100     push APM.0041B1C4                          ; ASCII "Printer Monitor"
0040B807    68 2CBA4100     push APM.0041BA2C                          ; ASCII "The registration key is not valid!"
0040B80C    8BCE            mov ecx,esi
0040B80E    E8 9F5F0000     call <jmp.&MFC42.#4224>
0040B813    8BCE            mov ecx,esi
0040B815    E8 285D0000     call <jmp.&MFC42.#4853>
0040B81A    8D4C24 68       lea ecx,dword ptr ss:[esp+68]
0040B81E    C68424 B0000000>mov byte ptr ss:[esp+B0],3
0040B826    E8 A50B0000     call APM.0040C3D0
0040B82B    8D4C24 48       lea ecx,dword ptr ss:[esp+48]
0040B82F    C68424 B0000000>mov byte ptr ss:[esp+B0],1
0040B837    E8 940B0000     call APM.0040C3D0
0040B83C    8D8C24 88000000 lea ecx,dword ptr ss:[esp+88]
0040B843    C78424 B0000000>mov dword ptr ss:[esp+B0],-1
0040B84E    E8 7D0B0000     call APM.0040C3D0
0040B853    8B8C24 A8000000 mov ecx,dword ptr ss:[esp+A8]
0040B85A    5F              pop edi
0040B85B    5E              pop esi
0040B85C    5D              pop ebp
0040B85D    64:890D 0000000>mov dword ptr fs:[0],ecx
0040B864    5B              pop ebx
0040B865    81C4 A4000000   add esp,0A4
0040B86B    C3              retn                                 //程序注册结束
```

### 跟进: `0040B70D` `E8 EEB6FFFF` `call APM.00406E00`

```asm
00406E00    6A FF           push -1
00406E02    68 A8294100     push APM.004129A8
00406E07    64:A1 00000000  mov eax,dword ptr fs:[0]
00406E0D    50              push eax
00406E0E    64:8925 0000000>mov dword ptr fs:[0],esp
00406E15    83EC 60         sub esp,60
00406E18    56              push esi
00406E19    8B7424 7C       mov esi,dword ptr ss:[esp+7C]
00406E1D    57              push edi
00406E1E    8B7C24 7C       mov edi,dword ptr ss:[esp+7C]
00406E22    6A 00           push 0
00406E24    56              push esi
00406E25    57              push edi
00406E26    C74424 14 00000>mov dword ptr ss:[esp+14],0
00406E2E    E8 7BAA0000     call <jmp.&MFC42.#1187>
00406E33    8D4C24 0C       lea ecx,dword ptr ss:[esp+C]
00406E37    E8 840A0000     call APM.004078C0
00406E3C    56              push esi
00406E3D    57              push edi
00406E3E    8D4C24 14       lea ecx,dword ptr ss:[esp+14]
00406E42    C74424 78 00000>mov dword ptr ss:[esp+78],0
00406E4A    E8 610C0000     call APM.00407AB0
00406E4F    8B7424 78       mov esi,dword ptr ss:[esp+78]
00406E53    8D4C24 0C       lea ecx,dword ptr ss:[esp+C]
00406E57    56              push esi
00406E58    E8 A30A0000     call APM.00407900             //算法CALL，跟进！！
00406E5D    8B4C24 68       mov ecx,dword ptr ss:[esp+68]
00406E61    8BC6            mov eax,esi
00406E63    5F              pop edi                       //字符串验证是否合法，ASCII "AmplusnetAPM24gb\_1227@163.comKuNgBiM"
00406E64    5E              pop esi
00406E65    64:890D 0000000>mov dword ptr fs:[0],ecx
00406E6C    83C4 6C         add esp,6C
00406E6F    C3              retn                          //计算完毕，返回正确注册信息
```

### 跟进 `00406E58` `E8 A30A0000` `call APM.00407900`

```asm
00407900    6A FF           push -1
00407902    68 EF294100     push APM.004129EF
00407907    64:A1 00000000  mov eax,dword ptr fs:[0]
0040790D    50              push eax
0040790E    64:8925 0000000>mov dword ptr fs:[0],esp
00407915    83EC 28         sub esp,28
00407918    53              push ebx
00407919    55              push ebp
0040791A    56              push esi
0040791B    57              push edi
0040791C    8BF9            mov edi,ecx
0040791E    C74424 1C 00000>mov dword ptr ss:[esp+1C],0
00407926    33F6            xor esi,esi
00407928    8D6F 44         lea ebp,dword ptr ds:[edi+44]
0040792B    8BCD            mov ecx,ebp
0040792D    8A01            mov al,byte ptr ds:[ecx]
0040792F    83C1 04         add ecx,4
00407932    884434 20       mov byte ptr ss:[esp+esi+20],al
00407936    8B41 FC         mov eax,dword ptr ds:[ecx-4]
00407939    8BD0            mov edx,eax
0040793B    83C6 04         add esi,4
0040793E    C1EA 08         shr edx,8
00407941    885434 1D       mov byte ptr ss:[esp+esi+1D],dl
00407945    8BD0            mov edx,eax
00407947    C1EA 10         shr edx,10
0040794A    C1E8 18         shr eax,18
0040794D    885434 1E       mov byte ptr ss:[esp+esi+1E],dl
00407951    884434 1F       mov byte ptr ss:[esp+esi+1F],al
00407955    83FE 08         cmp esi,8
```
