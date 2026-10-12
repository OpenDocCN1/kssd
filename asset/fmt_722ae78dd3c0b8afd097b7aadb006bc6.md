
# 0041EE08 |. 8D75 CC lea esi,dword ptr ss:\[ebp-34\]

# 0041EE0B |. 8D7D EC lea edi,dword ptr ss:\[ebp-14\]

# 0041EE0E |. A5 movs dword ptr es:\[edi\],dword ptr ds:\[esi\]

# 0041EE0F |. A5 movs dword ptr es:\[edi\],dword ptr ds:\[esi\]

# 0041EE10 |. A5 movs dword ptr es:\[edi\],dword ptr ds:\[esi\]

# 0041EE11 |. A5 movs dword ptr es:\[edi\],dword ptr ds:\[esi\]

# 0041EE12 |. 8B45 F4 mov eax,dword ptr ss:\[ebp-C\]

# 0041EE15 |. 81C3 A41B0000 add ebx,1BA4

# 0041EE1B |. 85DB test ebx,ebx

# 0041EE1D |. 8D48 FF lea ecx,dword ptr ds:\[eax-1\]

# 0041EE20 |. 74 03 je short Thunder.0041EE25

# 0041EE22 |. 8B5B 20 mov ebx,dword ptr ds:\[ebx+20\]

# 0041EE25 |> 6A 04 push 4 ; /Flags = SWP\_NOZORDER

# 0041EE27 |. 2BC1 sub eax,ecx ; |

# 0041EE29 |. 68 C0000000 push 0C0 ; |Height = C0 (192.)

# 0041EE2E |. 50 push eax ; |Width

# 0041EE2F |. 68 3F010000 push 13F ; |Y = 13F (319.)

# 0041EE34 |. 51 push ecx ; |X

# 0041EE35 |. 6A 00 push 0 ; |hAfterWnd = HWND\_TOP

# 0041EE37 |. 53 push ebx ; |hWnd

# 0041EE38 |. FF75 FC push dword ptr ss:\[ebp-4\] ; |hDefer

# 0041EE3B |. FF15 4C8F5500 call dword ptr ds:\[<&USER32.DeferWindowPos>\] ; \DeferWindowPos

# 0041EE41 |. FF75 FC push dword ptr ss:\[ebp-4\] ; /hDefer

# 0041EE44 |. FF15 548F5500 call dword ptr ds:\[<&USER32.EndDeferWindowPos>\] ; \EndDeferWindowPos

# 0041EE4A |. 5F pop edi

# 0041EE4B |. 5E pop esi

# 0041EE4C |. 5B pop ebx

# 0041EE4D |. C9 leave

# 0041EE4E \. C3 retn

\=============================

\=============================

PEdiy

用`UltraEdit`修改相应代码：

`1ED76`处`3C->00`

`1EDB5`处`03->3F`

`1EDBB`处`3C->00`

\=============================

\=============================

Greets

感谢`PEiD`/`C32ASM`/`OllyDbg`及其插件的开发、修改者，没有他们的杰出工作也就没有本文。

\=============================

\=============================

MrBeer\[CCG\]

`19/Jul/2005`

\=============================

`4dd466d0e1e0cc123afbe504da2462ca`
