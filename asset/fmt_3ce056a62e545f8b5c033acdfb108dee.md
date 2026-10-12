
```
00C1C839  50                  push eax
00C1C83A  E8 36B0FEFF         call 00C07875
00C1C83F  8985 A0D4FFFF       mov dword ptr ss:[ebp-2B60],eax
00C1C845  83BD A0D4FFFF 0>    cmp dword ptr ss:[ebp-2B60],0
00C1C84C  75 32               jnz short 00C1C880
00C1C84E  6A 01               push 1
00C1C850  8D85 80D2FFFF       lea eax,dword ptr ss:[ebp-2D80]
00C1C856  50                  push eax
00C1C857  E8 9F6CFFFF         call 00C134FB
00C1C85C  59                  pop ecx
00C1C85D  59                  pop ecx
00C1C85E  8D85 80D2FFFF       lea eax,dword ptr ss:[ebp-2D80]
00C1C864  50                  push eax
00C1C865  8D85 88D3FFFF       lea eax,dword ptr ss:[ebp-2C78]
00C1C86B  50                  push eax
00C1C86C  FFB5 84D3FFFF       push dword ptr ss:[ebp-2C7C]
00C1C872  E8 43A7FEFF         call 00C06FBA
00C1C877  83C4 0C             add esp,0C
00C1C87A  8985 A0D4FFFF       mov dword ptr ss:[ebp-2B60],eax
00C1C880  83BD A0D4FFFF 0>    cmp dword ptr ss:[ebp-2B60],0
00C1C887  75 58               jnz short 00C1C8E1
00C1C889  8B45 08             mov eax,dword ptr ss:[ebp+8]
00C1C88C  8B00                mov eax,dword ptr ds:[eax]
00C1C88E  C700 03000000       mov dword ptr ds:[eax],3
00C1C894  0FBE85 88D3FFFF    movsx eax,byte ptr ss:[ebp-2C78]
00C1C89B  85C0                test eax,eax
00C1C89D  74 0E               je short 00C1C8AD
00C1C89F  8D85 88D3FFFF       lea eax,dword ptr ss:[ebp-2C78]
00C1C8A5  8985 E0A8FFFF       mov dword ptr ss:[ebp+FFFFA8E0],eax
00C1C8AB  EB 0C               jmp short 00C1C8B9
00C1C8AD  8B85 84D3FFFF       mov eax,dword ptr ss:[ebp-2C7C]
00C1C8B3  8985 E0A8FFFF       mov dword ptr ss:[ebp+FFFFA8E0],eax
00C1C8B9  FF15 C072C200       call dword ptr ds:[C272C0]                ; ntdll.RtlGetLastWin32Error
00C1C8BF  50                  push eax
00C1C8C0  FFB5 E0A8FFFF       push dword ptr ss:[ebp+FFFFA8E0]
00C1C8C6  68 78DDC200         push 0C2DD78                               ; ASCII "File %s", error %d"
00C1C8CB  8B45 08             mov eax,dword ptr ss:[ebp+8]
00C1C8CE  FF70 04             push dword ptr ds:[eax+4]
00C1C8D1  FF15 1C73C200       call dword ptr ds:[C2731C]                ; msvcrt.sprintf
00C1C8D7  83C4 10             add esp,10
00C1C8DA  33C0                xor eax,eax
00C1C8DC  E9 7F1A0000         jmp 00C1E360
00C1C8E1  FFB5 A0D4FFFF       push dword ptr ss:[ebp-2B60]              ; Dll 基址进栈
00C1C8E7  E8 6294FEFF         call 00C05D4E
00C1C8EC  59                  pop ecx
00C1C8ED  83A5 9CD4FFFF 0>    and dword ptr ss:[ebp-2B64],0
00C1C8F4  A1 EC15C300         mov eax,dword ptr ds:[C315EC]
00C1C8F9  8985 F0A9FFFF       mov dword ptr ss:[ebp+FFFFA9F0],eax       ; 镜像基址送 [ebp+FFFFA9F0]
00C1C8FF  8B85 A0D4FFFF       mov eax,dword ptr ss:[ebp-2B60]           ; Dll 基址送 EAX
00C1C905  3B85 F0A9FFFF       cmp eax,dword ptr ss:[ebp+FFFFA9F0]
00C1C90B  75 0F               jnz short 00C1C91C
00C1C90D  C785 9CD4FFFF B>    mov dword ptr ss:[ebp-2B64],0C2BBB8
00C1C917  E9 93000000         jmp 00C1C9AF
00C1C91C  83A5 78D2FFFF 0>    and dword ptr ss:[ebp-2D88],0
00C1C923  C785 74D2FFFF D>    mov dword ptr ss:[ebp-2D8C],0C2C1D8
00C1C92D  EB 1C               jmp short 00C1C94B
00C1C92F  8B85 74D2FFFF       mov eax,dword ptr ss:[ebp-2D8C]
00C1C935  83C0 0C             add eax,0C
00C1C938  8985 74D2FFFF       mov dword ptr ss:[ebp-2D8C],eax
00C1C93E  8B85 78D2FFFF       mov eax,dword ptr ss:[ebp-2D88]
00C1C944  40                  inc eax
00C1C945  8985 78D2FFFF       mov dword ptr ss:[ebp-2D88],eax
00C1C94B  8B85 74D2FFFF       mov eax,dword ptr ss:[ebp-2D8C]
00C1C951  8338 00             cmp dword ptr ds:[eax],0
00C1C954  74 59               je short 00C1C9AF
00C1C956  8B85 74D2FFFF       mov eax,dword ptr ss:[ebp-2D8C]
00C1C95C  8B40 08             mov eax,dword ptr ds:[eax+8]
00C1C95F  83E0 01             and eax,1
00C1C962  85C0                test eax,eax
00C1C964  74 13               je short 00C1C979
00C1C966  B9 880FC300         mov ecx,0C30F88
00C1C96B  E8 77F1FDFF         call 00BFBAE7
00C1C970  0FB6C0              movzx eax,al
00C1C973  85C0                test eax,eax
00C1C975  74 02               je short 00C1C979
00C1C977 ^ EB B6             jmp short 00C1C92F
00C1C979  B9 880FC300         mov ecx,0C30F88
00C1C97E  E8 ADD1FEFF         call 00C09B30
00C1C983  8B8D 78D2FFFF       mov ecx,dword ptr ss:[ebp-2D88]
00C1C989  8B15 9C55C300       mov edx,dword ptr ds:[C3559C]
00C1C98F  8B0C8A              mov ecx,dword ptr ds:[edx+ecx*4]
00C1C992  33C8                xor ecx,eax
00C1C994  398D A0D4FFFF       cmp dword ptr ss:[ebp-2B60],ecx
00C1C99A  75 11               jnz short 00C1C9AD
00C1C99C  8B85 74D2FFFF       mov eax,dword ptr ss:[ebp-2D8C]
00C1C9A2  8B40 04             mov eax,dword ptr ds:[eax+4]
00C1C9A5  8985 9CD4FFFF       mov dword ptr ss:[ebp-2B64],eax
00C1C9AB  EB 02               jmp short 00C1C9AF
00C1C9AD ^ EB 80             jmp short 00C1C92F
00C1C9AF  80A5 94D4FFFF 0>    and byte ptr ss:[ebp-2B6C],0
00C1C9B6  83BD E4D7FFFF 0>    cmp dword ptr ss:[ebp-281C],0
00C1C9BD  75 39               jnz short 00C1C9F8
00C1C9BF  A0 AC15C300         mov al,byte ptr ds:[C315AC]
00C1C9C4  8885 ECA9FFFF       mov byte ptr ss:[ebp+FFFFA9EC],al
00C1C9CA  0FB685 ECA9FFFF    movzx eax,byte ptr ss:[ebp+FFFFA9EC]
00C1C9D1  85C0                test eax,eax
00C1C9D3  74 23               je short 00C1C9F8
00C1C9D5  8B85 90D4FFFF       mov eax,dword ptr ss:[ebp-2B70]
00C1C9DB  3B85 C0FEFFFF       cmp eax,dword ptr ss:[ebp-140]
00C1C9E1  72 15               jb short 00C1C9F8
00C1C9E3  8B85 90D4FFFF       mov eax,dword ptr ss:[ebp-2B70]
00C1C9E9  3B85 CCFEFFFF       cmp eax,dword ptr ss:[ebp-134]
00C1C9EF  73 07               jnb short 00C1C9F8
00C1C9F1  C685 94D4FFFF 0>    mov byte ptr ss:[ebp-2B6C],1
00C1C9F8  8B85 98D4FFFF       mov eax,dword ptr ss:[ebp-2B68]
00C1C9FE  40                  inc eax
00C1C9FF  8985 98D4FFFF       mov dword ptr ss:[ebp-2B68],eax
00C1CA05  83BD E4D7FFFF 0>    cmp dword ptr ss:[ebp-281C],0
00C1CA0C  74 4D               je short 00C1CA5B
00C1CA0E  8B85 90D4FFFF       mov eax,dword ptr ss:[ebp-2B70]
00C1CA14  2B85 E8D7FFFF       sub eax,dword ptr ss:[ebp-2818]
00C1CA1A  C1E8 02             shr eax,2
00C1CA1D  8985 70D2FFFF       mov dword ptr ss:[ebp-2D90],eax
00C1CA23  8B85 70D2FFFF       mov eax,dword ptr ss:[ebp-2D90]
00C1CA29  8B8D E4D7FFFF       mov ecx,dword ptr ss:[ebp-281C]
00C1CA2F  8D0481              lea eax,dword ptr ds:[ecx+eax*4]
00C1CA32  8985 10D9FFFF       mov dword ptr ss:[ebp-26F0],eax
00C1CA38  8B85 10D9FFFF       mov eax,dword ptr ss:[ebp-26F0]
00C1CA3E  8985 6CD9FFFF       mov dword ptr ss:[ebp-2694],eax
00C1CA44  8B85 44D9FFFF       mov eax,dword ptr ss:[ebp-26BC]
00C1CA4A  8B8D E4D7FFFF       mov ecx,dword ptr ss:[ebp-281C]
00C1CA50  8D0481              lea eax,dword ptr ds:[ecx+eax*4]
00C1CA53  8985 64D9FFFF       mov dword ptr ss:[ebp-269C],eax
00C1CA59  EB 59               jmp short 00C1CAB4
00C1CA5B  0FB685 94D4FFFF    movzx eax,byte ptr ss:[ebp-2B6C]
00C1CA62  85C0                test eax,eax
00C1CA64  74 30               je short 00C1CA96
00C1CA66  8B85 98D4FFFF       mov eax,dword ptr ss:[ebp-2B68]
00C1CA6C  C1E0 02             shl eax,2
00C1CA6F  50                  push eax
00C1CA70  E8 25970000         call 00C2619A                             ; jmp 到 msvcrt.??2@YAPAXI@Z
00C1CA75  59                  pop ecx
```
