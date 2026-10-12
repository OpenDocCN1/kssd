
# 反汇编代码分析

## 主函数返回逻辑

```
043EE454 |> 66:85C0 test ax, ax
043EE457 |. 66:8907 mov word ptr [edi], ax
043EE45A |. 7E 0F |jle short 043EE46B
043EE45C |. 8B16 |mov edx, dword ptr [esi]
043EE45E |. 8BCE |mov ecx, esi
043EE460 |. 0FBFC0 |movsx eax, ax
043EE463 |. 50 |push eax
043EE464 |. 8D47 02 |lea eax, dword ptr [edi+2]
043EE467 |. 50 |push eax
043EE468 |. FF52 20 |call dword ptr [edx+20]
043EE46B |> 0FBF85 D85FFF>|movsx eax, word ptr [ebp+FFFF5FD8]
043EE472 |. FF45 08 |inc dword ptr [ebp+8]
043EE475 |. 81C7 08280000 |add edi, 2808
043EE47B |. 3945 08 |cmp dword ptr [ebp+8], eax
043EE47E |.^ 7C B0 |jl short 043EE430
043EE480 |> 8D85 D85FFFFF lea eax, dword ptr [ebp+FFFF5FD8]
043EE486 |. 8B4D FC mov ecx, dword ptr [ebp-4]
043EE489 |. 8945 08 mov dword ptr [ebp+8], eax
043EE48C |. 8B45 0C mov eax, dword ptr [ebp+C]
043EE48F |. 66:8B40 0A mov ax, word ptr [eax+A]
043EE493 |. 66:8945 0C mov word ptr [ebp+C], ax
043EE497 |. FF75 0C push dword ptr [ebp+C]
043EE49A |. 8D45 08 lea eax, dword ptr [ebp+8]
043EE49D |. 50 push eax
043EE49E |. 68 9A1E3E04 push 043E1E9A
043EE4A3 |. E8 14000000 call 043EE4BC
043EE4A8 |. 5F pop edi
043EE4A9 |. 5E pop esi
043EE4AA |> C9 leave
043EE4AB \. C2 1400 retn 14
```

看见那个 `ASCII "Receive notify game event package"` 提示了吧。

跟进最后一个 `043EE4A3 |. E8 14000000 call 043EE4BC`：

## 字符串处理与调用

```
043EE4BC /$ B8 48054004 mov eax, 04400548
043EE4C1 |. E8 9A100100 call 043FF560
043EE4C6 |. 83EC 14 sub esp, 14
043EE4C9 |. 8A45 13 mov al, byte ptr [ebp+13]
043EE4CC |. 56 push esi
043EE4CD |. 57 push edi
043EE4CE |. 33F6 xor esi, esi
043EE4D0 |. 8BF9 mov edi, ecx
043EE4D2 |. 56 push esi
043EE4D3 |. 56 push esi
043EE4D4 |. 8D4D E0 lea ecx, dword ptr [ebp-20]
043EE4D7 |. 8845 E0 mov byte ptr [ebp-20], al
043EE4DA |. E8 5732FFFF call 043E1736
043EE4DF |. 8945 E4 mov dword ptr [ebp-1C], eax
043EE4E2 |. 8975 E8 mov dword ptr [ebp-18], esi
043EE4E5 |. 8D45 E0 lea eax, dword ptr [ebp-20]
043EE4E8 |. 8BCF mov ecx, edi
043EE4EA |. 50 push eax
043EE4EB |. 8975 FC mov dword ptr [ebp-4], esi
043EE4EE |. E8 F23D0000 call 043F22E5
043EE4F3 |. 8975 EC mov dword ptr [ebp-14], esi
043EE4F6 |. 8975 F0 mov dword ptr [ebp-10], esi
043EE4F9 |> 3975 E8 /cmp dword ptr [ebp-18], esi
043EE4FC |. 74 2E |je short 043EE52C
043EE4FE |. 8D45 F0 |lea eax, dword ptr [ebp-10]
043EE501 |. 8BCF |mov ecx, edi
043EE503 |. 50 |push eax
043EE504 |. 8D45 EC |lea eax, dword ptr [ebp-14]
043EE507 |. 50 |push eax
043EE508 |. 8D45 E0 |lea eax, dword ptr [ebp-20]
043EE50B |. 50 |push eax
043EE50C |. E8 F567FFFF |call 043E4D06
043EE511 |. 84C0 |test al, al
043EE513 |.^ 74 E4 |je short 043EE4F9
043EE515 |. 66:8B45 F0 |mov ax, word ptr [ebp-10]
043EE519 |. 66:3B45 10 |cmp ax, word ptr [ebp+10]
043EE51D |.^ 75 DA |jnz short 043EE4F9
043EE51F |. 8B45 0C |mov eax, dword ptr [ebp+C]
043EE522 |. 8B4D EC |mov ecx, dword ptr [ebp-14]
043EE525 |. FF30 |push dword ptr [eax]
043EE527 |. FF55 08 |call dword ptr [ebp+8]
043EE52A |.^ EB CD \jmp short 043EE4F9
043EE52C |> 834D FC FF or dword ptr [ebp-4], FFFFFFFF
043EE530 |. 8D4D E0 lea ecx, dword ptr [ebp-20]
043EE533 |. E8 802FFFFF call 043E14B8
043EE538 |. 8B4D F4 mov ecx, dword ptr [ebp-C]
043EE53B |. 5F pop edi
043EE53C |. 5E pop esi
043EE53D |. 64:890D 00000>mov dword ptr fs:[0], ecx
043EE544 |. C9 leave
043EE545 \. C2 0C00 retn 0C
```

很眼熟啊，应该跟进 `call dword ptr [ebp+8]`

## 消息分发逻辑

```
043E1E9A . 8B01 mov eax, dword ptr [ecx] ; MGRoom.03391B44
043E1E9C . FF60 04 jmp dword ptr [eax+4]
```

看见没有，对于不同的消息处理将从这里分道扬镳，跳到:

```
0333F2DF 8B49 0C mov ecx, dword ptr [ecx+C]
0333F2E2 85C9 test ecx, ecx
0333F2E4 74 09 je short 0333F2EF
0333F2E6 8B01 mov eax, dword ptr [ecx]
0333F2E8 FF7424 04 push dword ptr [esp+4]
0333F2EC FF50 04 call dword ptr [eax+4]
0333F2EF C2 0400 retn 4
```

只有一个 call，跟进来到 MGRoom 领空:

## MGRoom 模块分析

```
03329344 55 push ebp
03329345 8BEC mov ebp, esp
03329347 81EC C0040000 sub esp, 4C0
0332934D 53 push ebx
0332934E 56 push esi
0332934F 57 push edi
03329350 8BF9 mov edi, ecx
03329352 68 94703A03 push 033A7094 ; ASCII "Receive notify game event package"
03329357 8DB7 64FFFFFF lea esi, dword ptr [edi-9C]
0332935D 56 push esi
0332935E E8 C3700100 call 03340426
03329363 8365 FC 00 and dword ptr [ebp-4], 0
03329367 59 pop ecx
03329368 59 pop ecx
03329369 8D45 FC lea eax, dword ptr [ebp-4]
0332936C 50 push eax
0332936D 8D8F 50FFFFFF lea ecx, dword ptr [edi-B0]
03329373 E8 8EAFFEFF call 03314306
03329378 85C0 test eax, eax
0332937A 0F84 C8000000 je 03329448
03329380 8B5D 08 mov ebx, dword ptr [ebp+8]
03329383 8B4B 0A mov ecx, dword ptr [ebx+A]
03329386 0FBF43 08 movsx eax, word ptr [ebx+8]
0332938A 85C9 test ecx, ecx
0332938C 75 22 jnz short 033293B0
0332938E 8B4D FC mov ecx, dword ptr [ebp-4]
03329391 83C3 0E add ebx, 0E
03329394 83C0 FC add eax, -4
03329397 53 push ebx
03329398 8B11 mov edx, dword ptr [ecx]
0332939A 50 push eax
0332939B FF52 4C call dword ptr [edx+4C]
0332939E 68 78703A03 push 033A7078 ; ASCII "Game event is sent to game"
033293A3 56 push esi
033293A4 E8 7D700100 call 03340426
033293A9 59 pop ecx
033293AA 59 pop ecx
033293AB E9 98000000 jmp 03329448
033293B0 83F9 01 cmp ecx, 1
033293B3 0F84 8F000000 je 03329448
033293B9 83F9 02 cmp ecx, 2
033293BC 0F85 86000000 jnz 03329448
033293C2 80A5 40FBFFFF 0>and byte ptr [ebp-4C0], 0
033293C9 8D50 FC lea edx, dword ptr [eax-4]
033293CC B9 2E010000 mov ecx, 12E
033293D1 33C0 xor eax, eax
033293D3 8DBD 44FBFFFF lea edi, dword ptr [ebp-4BC]
033293D9 81FA B0040000 cmp edx, 4B0
033293DF 8955 08 mov dword ptr [ebp+8], edx
033293E2 F3:AB rep stos dword ptr es:[edi]
033293E4 76 0D jbe short 033293F3
033293E6 68 94653A03 push 033A6594
; ASCII " PupleMessage, but the message is too long"
033293EB 56 push esi
033293EC E8 CA7D0200 call 033511BB
033293F1 ^ EB B6 jmp short 033293A9
033293F3 68 5C703A03 push 033A705C ; ASCII "send PupleMessage to game"
033293F8 56 push esi
033293F9 E8 28700100 call 03340426
033293FE BE BC040000 mov esi, 4BC
03329403 8D85 40FBFFFF lea eax, dword ptr [ebp-4C0]
03329409 56 push esi
0332940A 6A 00 push 0
0332940C 50 push eax
0332940D E8 FA840500 call <jmp.&MSVCRT.memset>
03329412 8B45 08 mov eax, dword ptr [ebp+8]
03329415 83C3 0E add ebx, 0E
03329418 8985 48FBFFFF mov dword ptr [ebp-4B8], eax
0332941E 50 push eax
0332941F 8D85 4CFBFFFF lea eax, dword ptr [ebp-4B4]
03329425 53 push ebx
03329426 50 push eax
03329427 C685 40FBFFFF 3>mov byte ptr [ebp-4C0], 31
0332942E E8 DF840500 call <jmp.&MSVCRT.memcpy>
03329433 8B4D FC mov ecx, dword ptr [ebp-4]
03329436 83C4 20 add esp, 20
03329439 8D95 40FBFFFF lea edx, dword ptr [ebp-4C0]
0332943F 8B01 mov eax, dword ptr [ecx]
03329441 52 push edx
03329442 56 push esi
03329443 6A 31 push 31
03329445 FF50 54 call dword ptr [eax+54]
03329448 5F pop edi
03329449 5E pop esi
0332944A 5B pop ebx
0332944B C9 leave
0332944C C2 0400 retn 4
```

看见那个 `ASCII "Game event is sent to game"` 串了吗?说明上面那个 call 是把消息发送给 game 的，跟进此 call：

## 后续调用链

```
0331272C 56 push esi
0331272D 8BF1 mov esi, ecx
0331272F 57 push edi
03312730 8B06 mov eax, dword ptr [esi]
03312732 FF50 44 call dword ptr [eax+44]
03312735 8BF8 mov edi, eax
03312737 85FF test edi, edi
03312739 74 3A je short 03312775
0331273B 8B7C24 10 mov edi, dword ptr [esp+10]
0331273F 0FB607 movzx eax, byte ptr [edi]
03312742 50 push eax
03312743 8D46 14 lea eax, dword ptr [esi+14]
03312746 FF7424 10 push dword ptr [esp+10]
```
