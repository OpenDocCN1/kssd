
```
0039162F     0185 71674000        add dword ptr ss:[ebp+406771],eax                      ; kernel32.GetCommandLineA 获取此函数地址后面会用到

003927BE  7C812F3D  kernel32.GetCommandLineA

0039169C     50                   push eax                                               ; kernel32.GetCommandLineA

003916C4     8B85 71674000        mov eax,dword ptr ss:[ebp+406771]                      ; kernel32.GetCommandLineA

003916EE     8038 CC              cmp byte ptr ds:[eax],0CC

0039173B     58                   pop eax                                                ; kernel32.GetCommandLineA

0039179F     FF95 71674000        call dword ptr ss:[ebp+406771]                         ; kernel32.GetCommandLineA

003917C5     50                   push eax

003917EB     8BF8                 mov edi,eax

0039180F     32C0                 xor al,al

00391832     B9 FFFFFFFF          mov ecx,-1

0039185A     F2:AE                repne scas byte ptr es:[edi]

0039187B     F7D9                 neg ecx　　　　　　　　　　　　　　　　　　　          ；ｅｃｘ路径名长度

003918A1     49                   dec ecx

003918C5     5E                   pop esi                                                ; 00142358

00391911     BF D7664000          mov edi,4066D7

0039193D     03FD                 add edi,ebp

00391987     33DB                 xor ebx,ebx

003919FC     8A1E                 mov bl,byte ptr ds:[esi]

00391A24     80FB 5C              cmp bl,5C

00391ABB     881F                 mov byte ptr ds:[edi],bl

00391AE0     46                   inc esi　　　　　　　　　　　　　　　　　　　　        ；ｅｓｉ指向的是程序路径名

00391B09     47                   inc edi

003919FC     8A1E                 mov bl,byte ptr ds:[esi]

00391A24     80FB 5C              cmp bl,5C

00391A4B    /75 47                jnz short 00391A94

00391A4D    /EB 3A                jmp short 00391A89

00391A6D     B3 2D                mov bl,2D

00391ABB     881F                 mov byte ptr ds:[edi],bl　　　　　　　　　　           ；主要是将路径名中的＼转化为－

00391B7C     C607 00              mov byte ptr ds:[edi],0

00391BF4     B8 27674000          mov eax,406727

00391C1F     03C5                 add eax,ebp

00391C47     50                   push eax

00391C6D     B8 23674000          mov eax,406723

00391C98     03C5                 add eax,ebp

00391CC0     50                   push eax

00391CE0     6A 00                push 0

00391D08     68 03001F00          push 1F0003

00391D30     6A 00                push 0

00391D52     6A 00                push 0

00391D77     6A 00                push 0

00391D9B     B8 C6664000          mov eax,4066C6

00391DBF     03C5                 add eax,ebp

00391DE2     50                   push eax

00391E02     8B85 AF664000        mov eax,dword ptr ss:[ebp+4066AF]

00391E27     50                   push eax

00391E6E     50                   push eax

00391E97     8B85 64674000        mov eax,dword ptr ss:[ebp+406764]                      ; Advapi32.RegCreateKeyExA

00391EC0     8038 CC              cmp byte ptr ds:[eax],0CC

00391F0B     58                   pop eax                                                ; Advapi32.RegCreateKeyExA

00391F4E     FF95 64674000        call dword ptr ss:[ebp+406764]                         ; Advapi32.RegCreateKeyExA

0012FF74   80000002                键

0012FF78   00392713　　　　　　　　　子键

0012FF7C   00000000

00391F93     8B85 C2664000        mov eax,dword ptr ss:[ebp+4066C2]

00391FBB     50                   push eax

00391FE1     B8 B9664000          mov eax,4066B9

0039200B     03C5                 add eax,ebp

00392030     8B18                 mov ebx,dword ptr ds:[eax]

00392052     899D 73714000        mov dword ptr ss:[ebp+407173],ebx

0039207F     50                   push eax

003920A6     6A 03                push 3

003920C9     6A 00                push 0

003920EC     B8 B3664000          mov eax,4066B3

00392111     03C5                 add eax,ebp

0039213A     50                   push eax

0039215B     8B85 23674000        mov eax,dword ptr ss:[ebp+406723]

00392185     50                   push eax

003921CC     50                   push eax

003921F2     8B85 68674000        mov eax,dword ptr ss:[ebp+406768]                      ; Advapi32.RegSetValueExA

00392219     8038 CC              cmp byte ptr ds:[eax],0CC

00392268     58                   pop eax                                                ; Advapi32.RegSetValueExA

003922B5     FF95 68674000        call dword ptr ss:[ebp+406768]                         ; Advapi32.RegSetValueExA

00392319     0FB6B5 30784000      movzx esi,byte ptr ss:[ebp+407830]　　　　　　　　　　　ｅｓｉ　＝　３

0039233F     8BFD                 mov edi,ebp

0039238A     8B9D 2C784000        mov ebx,dword ptr ss:[ebp+40782C]                      ; KRYPTON.00400000

003923B4     8B87 35784000        mov eax,dword ptr ds:[edi+407835]

003923DB     03D8                 add ebx,eax

00392400     8B8F 39784000        mov ecx,dword ptr ds:[edi+407839]　　　　　　　　　　　；再一次进行解码

　　直接对ｃｔｒ　＋　ｇ　找到　ｇｅｔｐｒｏｃａｄｄｒｅｓｓ　在ｒｅｔｎ　８　处　下断　

003928AC     83BD 6F714000 00     cmp dword ptr ss:[ebp+40716F],0

003928B3     74 13                je short 003928C8

003928B5     8B8D 6F714000        mov ecx,dword ptr ss:[ebp+40716F]                      ; KRYPTON.0041D176

003928BB     90                   nop

003928BC     90                   nop

003928BD     90                   nop

003928BE     D221                 shl byte ptr ds:[ecx],cl

003928C0     3001                 xor byte ptr ds:[ecx],al

003928C2     41                   inc ecx

003928C3     8039 00              cmp byte ptr ds:[ecx],0　　

这段代码处理前后如下：

ecx : 0041d168

0041D168  45 78 69 74 50 72 6F 63  ExitProc

0041D170  65 73 73 00 00 00 1D 95  ess...

处理后

0041D168  82 82 82 82 82 82 82 82  倐倐倐倐

0041D170  82 82 82 00 00 00 1D 95  倐?..

0039298A     8B9D 34BE4100        mov ebx,dword ptr ss:[ebp+41BE34]                       ；00370000赋给ebx

003929AF     8985 96714000        mov dword ptr ss:[ebp+407196],eax                      ; kernel32.ExitProcess 赋值给3931e3

003929DC     8DBD 7B714000        lea edi,dword ptr ss:[ebp+40717B]                      ;3931c8赋值给edi

00392A02     8385 A2714000 1F     add dword ptr ss:[ebp+4071A2],1F                       ；初始值为零

00392A2B     039D A2714000        add ebx,dword ptr ss:[ebp+4071A2]

00392A57     899D A6714000        mov dword ptr ss:[ebp+4071A6],ebx                      ；赋给003931f3

00392A80     8BC3                 mov eax,ebx

00392AA1     8A0F                 mov cl,byte ptr ds:[edi]

00392AC3     880B                 mov byte ptr ds:[ebx],cl

00392AE8     43                   inc ebx

00392B09     47                   inc edi

00392B2B     833F 00              cmp dword ptr ds:[edi],0

00392B55   ^\0F85 46FFFFFF        jnz 00392AA1

00392B5B    /EB 3B                jmp short 00392B98                                   ；复制完毕就跳       3931c8~  e6 ------》 37001f

00392B9F     8BC8                 mov ecx,eax                                          ；0037001f

00392BC7     83C1 1B              add ecx,1B

00392BEC     8948 02              mov dword ptr ds:[eax+2],ecx                         ；0037003A

0037001F  81 35 3A 00 37 00 11 11  ?:.7.

00370027  11 11 A1 96 71 40 00 81  q@.

00392C11     8948 11              mov dword ptr ds:[eax+11],ecx

0037001F  81 35 3A 00 37 00 11 11  ?:.7.

00370027  11 11 A1 96 71 40 00 81  q@.

00392C39     8948 0B              mov dword ptr ds:[eax+B],ecx

00392C86     8BD8                 mov ebx,eax                ；0037001f

00392CA8     8B85 73714000        mov eax,dword ptr ss:[ebp+407173]                      ；  CBFF22BB

00392CD5     B9 6D4EC641          mov ecx,41C64E6D

00392CFE     F7E1                 mul ecx

00392D21     05 39300000          add eax,3039

00392D4D     25 FFFFFF7F          and eax,7FFFFFFF

00392D79     8985 73714000        mov dword ptr ss:[ebp+407173],eax

00392DCC     3D 0000003F          cmp eax,3F000000
```
