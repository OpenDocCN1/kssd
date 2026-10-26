
```asm
0046B989    6A 00                    push 0
0046B98B    6A 01                    push 1
0046B98D    A1 0CF09900              mov eax,dword ptr ds:[99F00C]
0046B992    50                       push eax
0046B993    E8 A8ACF9FF              call NOTEPAD.00406640                   ; jmp 到 kernel32.ReleaseSemaphore
0046B998    8B45 FC                  mov eax,dword ptr ss:[ebp-4]
0046B99B    E8 88B4FFFF              call NOTEPAD.00466E28
0046B9A0    A1 A02C4700              mov eax,dword ptr ds:[472CA0]
0046B9A5    8B00                     mov eax,dword ptr ds:[eax]
0046B9A7    BA 4CD94600              mov edx,NOTEPAD.0046D94C                ; ASCII "REGISTERED"
0046B9AC    E8 FB85F9FF              call NOTEPAD.00403FAC
0046B9B1    75 0C                    jnz short NOTEPAD.0046B9BF
0046B9B3    C685 3FFFFFFF 01         mov byte ptr ss:[ebp-C1],1
0046B9BA    E9 50120000              jmp NOTEPAD.0046CC0F
0046B9BF    A1 A02C4700              mov eax,dword ptr ds:[472CA0]
0046B9C4    8B00                     mov eax,dword ptr ds:[eax]
0046B9C6    BA 4CD94600              mov edx,NOTEPAD.0046D94C                ; ASCII "REGISTERED"
0046B9CB    E8 DC85F9FF              call NOTEPAD.00403FAC
0046B9D0    74 19                    je short NOTEPAD.0046B9EB
0046B9D2    A1 442D4700              mov eax,dword ptr ds:[472D44]
0046B9D7    8B00                     mov eax,dword ptr ds:[eax]
0046B9D9    E8 1695FDFF              call NOTEPAD.00444EF4
0046B9DE    8B45 FC                  mov eax,dword ptr ss:[ebp-4]
0046B9E1    E8 6662FDFF              call NOTEPAD.00441C4C
0046B9E6    E9 011B0000              jmp NOTEPAD.0046D4EC
0046B9EB    837D D4 00               cmp dword ptr ss:[ebp-2C],0
0046B9EF    0F84 420D0000            je NOTEPAD.0046C737
0046B9F5    A1 1C2B4700              mov eax,dword ptr ds:[472B1C]
0046B9FA    8038 01                  cmp byte ptr ds:[eax],1
0046B9FD    75 2C                    jnz short NOTEPAD.0046BA2B              ; 跳走咯
0046B9FF    8B0D 642C4700            mov ecx,dword ptr ds:[472C64]           ; NOTEPAD.00473888
0046BA05    8B09                     mov ecx,dword ptr ds:[ecx]
0046BA07    8B15 482C4700            mov edx,dword ptr ds:[472C48]           ; NOTEPAD.00473884
0046BA0D    8B12                     mov edx,dword ptr ds:[edx]
0046BA0F    8B45 FC                  mov eax,dword ptr ss:[ebp-4]
0046BA12    E8 3D8FFFFF              call NOTEPAD.00464954
0046BA17    A1 442D4700              mov eax,dword ptr ds:[472D44]
0046BA1C    8B00                     mov eax,dword ptr ds:[eax]
0046BA1E    E8 D194FDFF              call NOTEPAD.00444EF4
0046BA23    8B45 FC                  mov eax,dword ptr ss:[ebp-4]
0046BA26    E8 2162FDFF              call NOTEPAD.00441C4C
0046BA2B    A1 2FD49900              mov eax,dword ptr ds:[99D42F]
0046BA30    8945 BC                  mov dword ptr ss:[ebp-44],eax
0046BA33    A1 1FD09900              mov eax,dword ptr ds:[99D01F]
0046BA38    83E0 40                  and eax,40
0046BA3B    83F8 40                  cmp eax,40
0046BA3E    75 0B                    jnz short NOTEPAD.0046BA4B
0046BA40    8D55 94                  lea edx,dword ptr ss:[ebp-6C]
0046BA43    8B45 FC                  mov eax,dword ptr ss:[ebp-4]
0046BA46    E8 898AFFFF              call NOTEPAD.004644D4
0046BA4B    A1 1FD09900              mov eax,dword ptr ds:[99D01F]
0046BA50    83E0 10                  and eax,10
0046BA53    83F8 10                  cmp eax,10
0046BA56    75 2E                    jnz short NOTEPAD.0046BA86
0046BA58    8D45 94                  lea eax,dword ptr ss:[ebp-6C]
0046BA5B    E8 508DFFFF              call NOTEPAD.004647B0
0046BA60    837D 94 00               cmp dword ptr ss:[ebp-6C],0
0046BA64    75 20                    jnz short NOTEPAD.0046BA86
0046BA66    E8 0D85FEFF              call NOTEPAD.00453F78
0046BA6B    8985 68DDF9FF            mov dword ptr ss:[ebp+FFF9DD68],eax
0046BA71    DB85 68DDF9FF            fild dword ptr ss:[ebp+FFF9DD68]
0046BA77    83C4 F4                  add esp,-0C
0046BA7A    DB3C24                   fstp tbyte ptr ss:[esp]
0046BA7D    9B                       wait
0046BA7E    8D45 94                  lea eax,dword ptr ss:[ebp-6C]
0046BA81    E8 C2D7F9FF              call NOTEPAD.00409248
0046BA86    A1 1FD09900              mov eax,dword ptr ds:[99D01F]
0046BA8B    83E0 01                  and eax,1
0046BA8E    48                       dec eax
0046BA8F    75 19                    jnz short NOTEPAD.0046BAAA
0046BA91    8D85 84DEF9FF            lea eax,dword ptr ss:[ebp+FFF9DE84]
0046BA97    E8 1884FEFF              call NOTEPAD.00453EB4
0046BA9C    8B95 84DEF9FF            mov edx,dword ptr ss:[ebp+FFF9DE84]
0046BAA2    8D45 94                  lea eax,dword ptr ss:[ebp-6C]
0046BAA5    E8 FA83F9FF              call NOTEPAD.00403EA4
0046BAAA    A1 1FD09900              mov eax,dword ptr ds:[99D01F]
0046BAAF    83E0 02                  and eax,2
0046BAB2    83F8 02                  cmp eax,2
0046BAB5    75 31                    jnz short NOTEPAD.0046BAE8
0046BAB7    E8 BC84FEFF              call NOTEPAD.00453F78
0046BABC    8985 68DDF9FF            mov dword ptr ss:[ebp+FFF9DD68],eax
0046BAC2    DB85 68DDF9FF            fild dword ptr ss:[ebp+FFF9DD68]
0046BAC8    83C4 F4                  add esp,-0C
0046BACB    DB3C24                   fstp tbyte ptr ss:[esp]
0046BACE    9B                       wait
0046BACF    8D85 84DEF9FF            lea eax,dword ptr ss:[ebp+FFF9DE84]
0046BAD5    E8 6ED7F9FF              call NOTEPAD.00409248
0046BADA    8B95 84DEF9FF            mov edx,dword ptr ss:[ebp+FFF9DE84]
0046BAE0    8D45 94                  lea eax,dword ptr ss:[ebp-6C]
0046BAE3    E8 BC83F9FF              call NOTEPAD.00403EA4
0046BAE8    A1 1FD09900              mov eax,dword ptr ds:[99D01F]
0046BAED    83E0 04                  and eax,4
0046BAF0    83F8 04                  cmp eax,4
0046BAF3    75 19                    jnz short NOTEPAD.0046BB0E
0046BAF5    8D85 84DEF9FF            lea eax,dword ptr ss:[ebp+FFF9DE84]
0046BAFB    E8 7883FEFF              call NOTEPAD.00453E78
0046BB00    8B95 84DEF9FF            mov edx,dword ptr ss:[ebp+FFF9DE84]
0046BB06    8D45 94                  lea eax,dword ptr ss:[ebp-6C]
0046BB09    E8 9683F9FF              call NOTEPAD.00403EA4
0046BB0E    A1 1FD09900              mov eax,dword ptr ds:[99D01F]
0046BB13    83E0 08                  and eax,8
0046BB16    83F8 08                  cmp eax,8
0046BB19    75 7D                    jnz short NOTEPAD.0046BB98
0046BB1B    E8 748BFFFF              call NOTEPAD.00464694
0046BB20    83F8 01                  cmp eax,1
0046BB23    75 73                    jnz short NOTEPAD.0046BB98
0046BB25    8B45 FC                  mov eax,dword ptr ss:[ebp-4]
0046BB28    8B80 D4020000            mov eax,dword ptr ds:[eax+2D4]
0046BB2E    E8 A1DDFEFF              call NOTEPAD.004598D4
0046BB33    E8 20B8FEFF              call NOTEPAD.00457358
0046BB38    83E0 7F                  and eax,7F
0046BB3B    8D95 84DEF9FF            lea edx,dword ptr ss:[ebp+FFF9DE84]
0046BB41    E8 22C6F9FF              call NOTEPAD.00408168
0046BB46    FFB5 84DEF9FF            push dword ptr ss:[ebp+FFF9DE84]
0046BB4C    8B45 FC                  mov eax,dword ptr ss:[ebp-4]
0046BB4F    8B80 D4020000            mov eax,dword ptr ds:[eax+2D4]
0046BB55    FF70 30                  push dword ptr ds:[eax+30]
0046BB58    8B45 FC                  mov eax,dword ptr ss:[ebp-4]
0046BB5B    8B80 D4020000            mov eax,dword ptr ds:[eax+2D4]
0046BB61    E8 6EDDFEFF              call NOTEPAD.004598D4
0046BB66    FF70 0C                  push dword ptr ds:[eax+C]
0046BB69    8B45 FC                  mov eax,dword ptr ss:[ebp-4]
0046BB6C    8B80 D4020000            mov eax,dword ptr ds:[eax+2D4]
0046BB72    E8 5DDDFEFF              call NOTEPAD.004598D4
0046BB77    FF70 08                  push dword ptr ds:[eax+8]
```
