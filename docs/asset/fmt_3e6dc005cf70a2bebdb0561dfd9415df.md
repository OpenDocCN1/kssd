
`004AAC66 FF5304 call dword ptr [ebx+$04]`

`004AAC69 8D5594 lea edx, [ebp-$6C]`

* `Reference to FormReg`

`004AAC6C 8B45FC mov eax, [ebp-$04]`

* `Reference to control TFormReg.EditCode : TEdit`

`004AAC6F 8B80E8020000 mov eax, [eax+$02E8]`

`004AAC75 E8727FF8FF call 00432BEC`

`004AAC7A 8B4594 mov eax, [ebp-$6C]`

`004AAC7D 50 push eax`

* `Possible String Reference to: 'RegCode'`

`004AAC7E B9D8AD4A00 mov ecx, $004AADD8`

* `Possible String Reference to: 'RegData'`

`004AAC83 BAB4AD4A00 mov edx, $004AADB4`

`004AAC88 8B45F8 mov eax, [ebp-$08]`

`004AAC8B 8B18 mov ebx, [eax]`

* `Possible reference to virtual method TIniFile.OFFS_04`

`004AAC8D FF5304 call dword ptr [ebx+$04]`

`004AAC90 33C0 xor eax, eax`

`004AAC92 5A pop edx`

`004AAC93 59 pop ecx`

`004AAC94 59 pop ecx`

`004AAC95 648910 mov fs:[eax], edx`

`****** FINALLY`

`004AAC98 68AFAC4A00 push $004AACAF`

`004AAC9D B201 mov dl, $01`

`004AAC9F 8B45F8 mov eax, [ebp-$08]`

`004AACA2 8B08 mov ecx, [eax]`

* `Possible reference to virtual method TIniFile.OFFS_04`

`004AACA4 FF51FC call dword ptr [ecx-$04]`

`004AACA7 C3 ret`

`004AACA8 E9B38AF5FF jmp 00403760`

`004AACAD EBEE jmp 004AAC9D`

`****** END`

* `Reference to FormReg`

`004AACAF 8B45FC mov eax, [ebp-$04]`

`004AACB2 E8F533FAFF call 0044E0AC`

`004AACB7 33C0 xor eax, eax`

`004AACB9 5A pop edx`

`004AACBA 59 pop ecx`

`004AACBB 59 pop ecx`

`004AACBC 648910 mov fs:[eax], edx`

`****** FINALLY`

* `Possible String Reference to: '_^[]门课募柚闷?`

`004AACBF 68D9AC4A00 p 4dd466d0e1e0cc123afbe504da2462ca`
