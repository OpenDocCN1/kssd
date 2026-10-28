
```
004034CA    3200                xor al,byte ptr ds:[eax]

004034CC    0000                add byte ptr ds:[eax],al

004034CE    D7                  xlat byte ptr ds:[ebx+al]

004034CF    A2 C8EBB2E2         mov byte ptr ds:[E2B2EBC8],al

004034D4    CA D421             retf 21D4

String //Inject1-2 and 注入测试

004034D7    0000                add byte ptr ds:[eax],al

004034D9    0000                add byte ptr ds:[eax],al

004034DB    0000                add byte ptr ds:[eax],al

004034DD    0000                add byte ptr ds:[eax],al

004034DF    0000                add byte ptr ds:[eax],al

004034E1    0000                add byte ptr ds:[eax],al

004034E3    0000                add byte ptr ds:[eax],al

004034E5    0000                add byte ptr ds:[eax],al

004034E7    0000                add byte ptr ds:[eax],al

004034E9    0000                add byte ptr ds:[eax],al

004034EB    0000                add byte ptr ds:[eax],al

004034ED    0000                add byte ptr ds:[eax],al

004034EF    0000                add byte ptr ds:[eax],al

004034F1    0000                add byte ptr ds:[eax],al

004034F3    0000                add byte ptr ds:[eax],al

004034F5    0000                add byte ptr ds:[eax],al

004034F7    0000                add byte ptr ds:[eax],al

004034F9    0000                add byte ptr ds:[eax],al

004034FB    0000                add byte ptr ds:[eax],al

004034FD    0000                add byte ptr ds:[eax],al

004034FF    00C7                add bh,al

00403501    05 82344000         add eax,Inject1\_.00403482

00403506    8000 00             add byte ptr ds:[eax],0

00403509    00C7                add bh,al

0040350B    05 86344000         add eax,Inject1\_.00403486

00403510    0037                add byte ptr ds:[edi],dh

00403512    40                  inc eax

00403513    00C7                add bh,al

00403515    05 9E344000         add eax,Inject1\_.0040349E

0040351A    1000                adc byte ptr ds:[eax],al

0040351C    0000                add byte ptr ds:[eax],al

0040351E    B8 C2344000         mov eax,Inject1\_.004034C2                       ; ASCII "Inject1-2"

00403523    A3 A6344000         mov dword ptr ds:[4034A6],eax

00403528    68 82344000         push Inject1\_.00403482

0040352D    E8 9AFEFFFF         call Inject1\_.RegisterClassA                    ; jmp to user32.RegisterClassA

00403532    6A 00               push 0

00403534    68 00004000         push Inject1\_.00400000                          ; ASCII "MZP"

00403539    6A 00               push 0

0040353B    6A 00               push 0

0040353D    68 AA000000         push 0AA

00403542    68 0E010000         push 10E

00403547    6A 64               push 64

00403549    6A 64               push 64

0040354B    68 00000810         push 10080000

00403550    68 CE344000         push Inject1\_.004034CE

00403555    68 C2344000         push Inject1\_.004034C2                          ; ASCII "Inject1-2"

0040355A    6A 00               push 0

0040355C    E8 5BFEFFFF         call Inject1\_.CreateWindowExA                   ; jmp to user32.CreateWindowExA

00403561    A3 D9344000         mov dword ptr ds:[4034D9],eax

00403566    EB 0C               jmp short Inject1\_.00403574

00403568    53                  push ebx

00403569    E8 56FEFFFF         call Inject1\_.TranslateMessage                  ; jmp to user32.TranslateMessage

0040356E    53                  push ebx

0040356F    E8 68FEFFFF         call Inject1\_.DispatchMessageA                  ; jmp to user32.DispatchMessageA

00403574    6A 00               push 0

00403576    6A 00               push 0

00403578    A1 D9344000         mov eax,dword ptr ds:[4034D9]

0040357D    50                  push eax

0040357E    53                  push ebx

0040357F    E8 50FEFFFF         call Inject1\_.GetMessageA                       ; jmp to user32.GetMessageA

00403584    85C0                test eax,eax

00403586  ^ 75 E0               jnz short Inject1\_.00403568

00403588    5B                  pop ebx

00403589    C3                  retn

0040358A    0000                add byte ptr ds:[eax],al

0040358C    0000                add byte ptr ds:[eax],al

0040358E    0000                add byte ptr ds:[eax],al

00403590    0000                add byte ptr ds:[eax],al

00403592    0000                add byte ptr ds:[eax],al

00403594    0000                add byte ptr ds:[eax],al

00403596    0000                add byte ptr ds:[eax],al

00403598    0000                add byte ptr ds:[eax],al

0040359A    0000                add byte ptr ds:[eax],al

0040359C    0000                add byte ptr ds:[eax],al

0040359E    0000                add byte ptr ds:[eax],al

004035A0    0000                add byte ptr ds:[eax],al

004035A2    0000                add byte ptr ds:[eax],al

004035A4    0000                add byte ptr ds:[eax],al

004035A6    0000                add byte ptr ds:[eax],al

004035A8    0000                add byte ptr ds:[eax],al

004035AA    0000                add byte ptr ds:[eax],al

004035AC    0000                add byte ptr ds:[eax],al

004035AE    0000                add byte ptr ds:[eax],al

004035B0    0000                add byte ptr ds:[eax],al

004035B2    0000                add byte ptr ds:[eax],al

004035B4    0000                add byte ptr ds:[eax],al

004035B6    0000                add byte ptr ds:[eax],al

004035B8    0000                add byte ptr ds:[eax],al

004035BA    0000                add byte ptr ds:[eax],al

004035BC    0000                add byte ptr ds:[eax],al

004035BE    0000                add byte ptr ds:[eax],al

004035C0    0000                add byte ptr ds:[eax],al

004035C2    0000                add byte ptr ds:[eax],al

004035C4    0000                add byte ptr ds:[eax],al

004035C6    0000                add byte ptr ds:[eax],al

004035C8    0000                add byte ptr ds:[eax],al

004035CA    0000                add byte ptr ds:[eax],al

004035CC    0000                add byte ptr ds:[eax],al

004035CE    0000                add byte ptr ds:[eax],al

004035D0    0000                add byte ptr ds:[eax],al

004035D2    0000                add byte ptr ds:[eax],al

004035D4    0000                add byte ptr ds:[eax],al

004035D6    0000                add byte ptr ds:[eax],al

004035D8    0000                add byte ptr ds:[eax],al

004035DA    0000                add byte ptr ds:[eax],al

004035DC    0000                add byte ptr ds:[eax],al

004035DE    0000                add byte ptr ds:[eax],al

004035E0    0000                add byte ptr ds:[eax],al

004035E2    0000                add byte ptr ds:[eax],al

004035E4    0000                add byte ptr ds:[eax],al

004035E6    0000                add byte ptr ds:[eax],al

004035E8    0000                add byte ptr ds:[eax],al

004035EA    0000                add byte ptr ds:[eax],al

004035EC    0000                add byte ptr ds:[eax],al

004035EE    0000                add byte ptr ds:[eax],al

004035F0    0000                add byte ptr ds:[eax],al

004035F2    0000                add byte ptr ds:[eax],al

004035F4    0000                add byte ptr ds:[eax],al

004035F6    0000                add byte ptr ds:[eax],al

004035F8    0000                add byte ptr ds:[eax],al

004035FA    0000                add byte ptr ds:[eax],al

004035FC    0000                add byte ptr ds:[eax],al

004035FE    0000                add byte ptr ds:[eax],al

00403600    0000                add byte ptr ds:[eax],al

00403602    0000                add byte ptr ds:[eax],al

00403604    0000                add byte ptr ds:[eax],al

00403606    0000                add byte ptr ds:[eax],al

00403608    0000                add byte ptr ds:[eax],al

0040360A    0000                add byte ptr ds:[eax],al

0040360C    0000                add byte ptr ds:[eax],al

0040360E    0000                add byte ptr ds:[eax],al
```
