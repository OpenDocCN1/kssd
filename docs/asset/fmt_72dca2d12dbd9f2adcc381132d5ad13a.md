
```asm
0040D522  |  8B4E 08              mov ecx,dword ptr ds:[esi+8]          ; 从后面可以看出，这是变形输入表的 OriginalFirstThunk  

0040D525  |  0BC9                or ecx,ecx  

0040D527  |  75 03               jnz short Yoda's_C.0040D52C          ; 不为 0 则进行处理，为 0 则找 FirstThunk。本例非 0  

0040D529  |  8B4E 04             mov ecx,dword ptr ds:[esi+4]          ; 从后面可以看出，这是变形输入表的 FirstThunk  

0040D52C  |  038D 64254000       add ecx,dword ptr ss:[ebp+402564]     ; RVA+ImageBaseAddress 定位 OriginalFirstThunk  

0040D532  |  8B56 04             mov edx,dword ptr ds:[esi+4]          ; FirstThunk  

0040D535     0395 64254000       add edx,dword ptr ss:[ebp+402564]     ; RVA+ImageBaseAddress 定位  

0040D53B  | /E9 C3000000         jmp Yoda's_C.0040D603                ; 这里是一个大循环  

0040D540  | |F701 00000080       test dword ptr ds:[ecx],80000000      ; 从这个特征值几乎可以确定[ecx]就是一个 IMAGE_THUNK_DATA 了，判断是 by_NAME 还是 by_INDEX  

0040D546  | |75 4B               jnz short Yoda's_C.0040D593          ; 若 IMAGE_ORDINAL_FLAG32 则转向 by_Index 方式处理  

0040D548  | |8B01               mov eax,dword ptr ds:[ecx]            ; by_Name 的处理  

0040D54A  | |83C0 02             add eax,2                             ; 开始看到这里还以为是搞什么名堂，等循环了一遍才知道，eax+2 是跳过 IMAGE_IMPORT_BY_NAME 结构的成员 Hint，定位到 Name 成员，以解密出函数名。而这个 eax 来自[ecx] ，ecx 来自[esi+8]，因此，[esi+8]就是 OriginalFirstThunk 了！  

0040D54D  | |0385 64254000       add eax,dword ptr ss:[ebp+402564]     ; RVA+ImageBaseAddress 定位待解密函数名称字符串的地址  

0040D553  | |50                  push eax                              ; 保存函数名地址  

0040D554  | |E8 8BFFFFFF         call Yoda's_C.0040D4E4                ; 就是前面用过的那个字符串解密函数，在这里是解密函数名  

  {  
  0040D4E4     56                push esi  

  0040D4E5     57                push edi  

  0040D4E6     8BF0              mov esi,eax  

  0040D4E8     8BF8              mov edi,eax  

  0040D4EA    /AC                lods byte ptr ds:[esi]  

  0040D4EB    |C0C8 04           ror al,4                      ; 简单地将每字节循环右移 4 位  

  0040D4EE    |AA                stos byte ptr es:[edi]        ; 这里将解密后的字符回写  

  0040D4EF    |803F 00           cmp byte ptr ds:[edi],0  

  0040D4F2   ^\75 F6              jnz short Yoda's_C.0040D4EA   ; 没到字符串末尾就继续解密  

  0040D4F4     5F                pop edi  

  0040D4F5     5E                pop esi  

  0040D4F6     C3                retn  

  }  

0040D559  | |58                  pop eax                           ; 此时弹出的是解密后的函数名  

0040D55A  | |8BF8               mov edi,eax                         ; 地址保存  

0040D55C  | |52                  push edx                          ; IAT 待填充的地址  

0040D55D  | |51                  push ecx                          ; FirstThunk  

0040D55E  | |50                  push eax                          ; 参数 2：lpProcName  

0040D55F  | |53                  push ebx                          ; 参数 1：hModule  

0040D560  | |FF95 F4264000       call dword ptr ss:[ebp+4026F4]      ; kernel32.GetProcAddress  

0040D566  | |0BC0                or eax,eax  

0040D568  | |75 07               jnz short Yoda's_C.0040D571          ; 成功则跳，否则 game over  

0040D56A  | |59                  pop ecx  

0040D56B  | |5A                  pop edx  

0040D56C  | |E9 AB000000         jmp Yoda's_C.0040D61C  

0040D571  | |59                  pop ecx                           ; FirstThunk  

0040D572  | |5A                  pop edx                           ; IAT 待填充的地址  

0040D573  | |60                  pushad  

0040D574  | |F785 6C254000 04000000  test dword ptr ss:[ebp+40256C],4    ; 3C  

0040D57E  | |74 0E               je short Yoda's_C.0040D58E  

0040D580  | |8D85 1B234000       lea eax,dword ptr ss:[ebp+40231B]     ; 取下面调用子函数后返回地址：0040D58E  

0040D586  | |50                  push eax                          ; 用 push+jmp 的形式调用子函数，这里压入子程序的返回地址  

0040D587  | |8BC7                mov eax,edi                       ; edi=函数名地址  

0040D589  | |E9 DB010000         jmp Yoda's_C.0040D769               ; 跳入子函数。功能是将 eax 处的字符串（这里是函数名）擦除，与上面擦除 dll 名字是同一函数  

  {  
  0040D769    /EB 04              jmp short Yoda's_C.0040D76F  

  0040D76B    |C600 00             mov byte ptr ds:[eax],0  

  0040D76E    |40                  inc eax  

  0040D76F    \8038 00              cmp byte ptr ds:[eax],0       ; 把刚刚解密出来的函数名称擦除了，即把 IMAGE_IMPORT_BY_NAME 的 Name 成员清 0，只保留了 Hint 成员  

  0040D772   ^ 75 F7               jnz short Yoda's_C.0040D76B  

  0040D774     C3                  retn  

  }  

0040D58E  | |61                  popad                              ; 调用子函数后返回到这里  

0040D58F  | |8902               mov dword ptr ds:[edx],eax          ; 填充 IAT(by_Name 的处理)  

0040D591  | |EB 19               jmp short Yoda's_C.0040D5AC          ; 跳  

0040D593  | |52                  push edx                          ; by_Index 的处理（本例中未执行到）  

0040D594  | |51                  push ecx  

0040D595  | |8B01               mov eax,dword ptr ds:[ecx]  

0040D597  | |2D 00000080          sub eax,80000000  

0040D59C  | |50                  push eax  

0040D59D  | |53                  push ebx  

0040D59E  | |FF95 F4264000       call dword ptr ss:[ebp+4026F4]      ; kernel32.GetProcAddress  

0040D5A4  | |85C0                test eax,eax  

0040D5A6  | |74 74               je short Yoda's_C.0040D61C  

0040D5A8  | |59                  pop ecx  

0040D5A9  | |5A                  pop edx  

0040D5AA  | |8902               mov dword ptr ds:[edx],eax          ; 填充 IAT  

跳到了这里：  

0040D5AC  | |F785 6C254000 20000000  test dword ptr ss:[ebp+40256C],20   ; 3C 两种方式分别填充 IAT 后都转到这里  

0040D5B6  | |74 45               je short Yoda's_C.0040D5FD            ; =20 则跳到下一个  

0040D5B8  | |83BD 78254000 00     cmp dword ptr ss:[ebp+402578],0       ; 1  

0040D5BF  | |74 14               je short Yoda's_C.0040D5D5  

0040D5C1  | |81FB 00000070         cmp ebx,70000000                     ; dll 句柄(ebx=7D590000 (shell32.7D590000))  

0040D5C7  | |72 08               jb short Yoda's_C.0040D5D1  

0040D5C9  | |81FB FFFFFF77         cmp ebx,77FFFFFF                     ; user32.wsprintfA 的句柄=77D10000  

0040D5CF /| |76 0E               jbe short Yoda's_C.0040D5DF          ; user32.dll 的函数在此跳  

0040D5D1 ||/|EB 2A              jmp short Yoda's_C.0040D5FD          ; 跳  

0040D5D3 ||||EB 0A              jmp short Yoda's_C.0040D5DF  

0040D5D5 ||||81FB 00000080       cmp ebx,80000000  

0040D5DB ||||73 02               jnb short Yoda's_C.0040D5DF  

0040D5DD ||||EB 1E              jmp short Yoda's_C.0040D5FD  

0040D5DF \|||57                  push edi                         ; 如果我们在处理完第 1 个 dll 后在 0040D619 处 F4 自动处理完剩余 dll 的函数，则看不到这里的关键处理，所以要有耐心，把隐藏在中间的步骤看清楚。  

0040D5E0  |||56                  push esi  

0040D5E1  |||8DBD D2274000       lea edi,dword ptr ss:[ebp+4027D2]    ; 地址=0040DA45，buffer，此时保存的是堆地址  

0040D5E7  |||8B77 04             mov esi,dword ptr ds:[edi+4]         ; buffer+4 处是前面保存的第 2 份堆地址 001431E0  

0040D5EA  |||8932               mov dword ptr ds:[edx],esi           ; 将刚刚填充的函数地址又改为申请的堆地址  

0040D5EC  |||2BC6               sub eax,esi                         ; 从下面几行得知此 2 句 sub 是计算跳转的相对距离  

0040D5EE  |||83E8 05             sub eax,5                         ; -5 是因为 jmp 指令 E9XXXXXXXX 本身占 5 字节  

0040D5F1  |||C606 E9             mov byte ptr ds:[esi],0E9           ; 前面申请的堆在这里派上用场。0E9 是 jmp 指令
```
