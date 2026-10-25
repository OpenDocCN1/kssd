
```
F7D1                    not ecx
:00421CE1 49                 dec ecx
:00421CE2 2BCD             sub ecx, ebp
:00421CE4 6685C9         test cx, cx
:00421CE7 7E32               jle 00421D1B
:00421CE9 33F6                xor esi, esi
:00421CEB 6685ED             test bp, bp
:00421CEE 7E22               jle 00421D12
\*
Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00421D10(C)
|
:00421CF0 8B15D4DB4200   mov edx, dword ptr [0042DBD4]
         ====>EDI=[0042DBD4]=wt
:00421CF6 0FBFC6               movsx eax, si
:00421CF9 8A1410         mov dl, byte ptr [eax+edx]
                 1、 ====>DL=77（H）
 即：w
         2、 ====>DL=74（H）  即：t
:00421CFC 80FA3F              cmp dl, 3F
:00421CFF 740B                je 00421D0C
:00421D01 0FBFF9              movsx edi, cx
:00421D04 03F8               add edi, eax
:00421D06 3A543C20           cmp dl, byte ptr [esp+edi+20]
                   1、 ====>DL=77
   [esp+eax+20]=39
               即：注册码的倒数第1位应是  w
                 2、 ====>DL=74
   [esp+eax+20]=30
               即：注册码的倒数第2位应是  t
:00421D0A 7506                jne 00421D12
             ====>不同则跳则OVER！  可下 R
FL Z 改变跳转
\* Referenced
by a (U)nconditional or (C)onditional Jump at Address:
|:00421CFF(C)
|
:00421D0C 46                    inc esi
:00421D0D 663BF5             cmp si, bp
:00421D10 7CDE               jl 00421CF0
                     ====>循环2次
\*
Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:00421CEE(C),
:00421D0A(C)
|
:00421D12 663BF5           cmp si, bp
:00421D15 7504               jne 00421D1B
:00421D17 FF442414           inc [esp+14]
\*
Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:00421CE7(C),
:00421D15(C)
|
:00421D1B 837C241402       cmp dword ptr [esp+14], 00000002
:00421D20 740A                je 00421D2C
:00421D22 B8FEFFFFFF          mov eax, FFFFFFFE
:00421D27 E941010000          jmp 00421E6D
\*
Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00421D20(C)
|
:00421D2C 8B3DF4DB4200           mov edi, dword ptr [0042DBF4]
:00421D32 83C9FF                 or ecx, FFFFFFFF
:00421D35 33C0                   xor eax, eax
:00421D37 F2                     repnz
:00421D38 AE                     scasb
:00421D39 8B3DD4DB4200           mov edi, dword ptr [0042DBD4]
:00421D3F F7D1                   not ecx
:00421D41 49                     dec ecx
:00421D42 8D740C20               lea esi, dword ptr [esp+ecx+20]
:00421D46 83C9FF                 or ecx, FFFFFFFF
:00421D49 F2                     repnz
:00421D4A AE                     scasb
:00421D4B F7D1                   not ecx
:00421D4D 49                     dec ecx
:00421D4E 8BD6                   mov edx, esi
:00421D50 2BD1                   sub edx, ecx
:00421D52 8BFE                   mov edi, esi
:00421D54 83C9FF                 or ecx, FFFFFFFF
:00421D57 F2                     repnz
:00421D58 AE                     scasb
:00421D59 F7D1                   not ecx
:00421D5B 49                     dec ecx
:00421D5C 88040A                 mov byte ptr [edx+ecx], al
:00421D5F 8BCE                   mov ecx, esi
   ====>ECX=DESI=72468  即去掉第1、2、9、10位后的试炼码
:00421D61 E84A5B0000             call 004278B0
       ====>检测上面的中间几位是否为数字？
:00421D66 85C0                   test eax, eax
:00421D68 750A                   jne 00421D74
                   ====>不是数字则不跳则OVER！
:00421D6A B8FDFFFFFF             mov eax, FFFFFFFD
:00421D6F E9F9000000             jmp 00421E6D
\*
Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00421D68(C)
|
:00421D74 BA64A14200             mov edx, 0042A164
         ====>EDX=0604      呵呵，程序自给的！
:00421D79 8BCE                   mov ecx, esi
             ====>ECX=ESI=572468
:00421D7B E8705B0000             call 004278F0
       ====>再次检测72468是否为数字？
不是数字则“invalid digital number！”。并且把72468转化为用16进制值表示！
:00421D80 8BF8                   mov edi, eax
             ====>EDI=EAI=00011B14（H）=72468（D）
:00421D82 66A154DC4200           mov ax, word ptr [0042DC54]
:00421D88 663D0100               cmp ax, 0001
:00421D8C 7546                   jne 00421DD4
:00421D8E 66A1FADB4200           mov ax, word ptr [0042DBFA]
:00421D94 8B1500DC4200           mov edx, dword ptr [0042DC00]
:00421D9A 33C9                   xor ecx, ecx
:00421D9C 8ACC                   mov cl, ah
:00421D9E 25FF000000             and eax, 000000FF
:00421DA3 8BF1                   mov esi, ecx
:00421DA5 8BC8                   mov ecx, eax
:00421DA7 E854FBFFFF             call 00421900
:00421DAC 8B15FCDB4200           mov edx, dword ptr [0042DBFC]
:00421DB2 03F8                   add edi, eax
:00421DB4 6685F6                 test si, si
:00421DB7 7504                   jne 00421DBD
:00421DB9 33C9                   xor ecx, ecx
:00421DBB EB03                   jmp 00421DC0
\*
Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00421DB7(C)
|
:00421DBD 8D4E01                 lea ecx, dword ptr [esi+01]
\*
Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00421DBB(U)
|
:00421DC0 E83BFBFFFF             call 00421900
:00421DC5 8BC8                   mov ecx, eax
:00421DC7 85C9                   test ecx, ecx
:00421DC9 7438                   je 00421E03
:00421DCB 8BC7                   mov eax, edi
:00421DCD 99                     cdq
:00421DCE F7F9                   idiv ecx
:00421DD0 8BC2                   mov eax, edx
:00421DD2 EB27                   jmp 00421DFB
\*
Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00421D8C(C)
|
:00421DD4 663D0200               cmp ax, 0002
:00421DD8 7529                   jne 00421E03
:00421DDA 8B15FCDB4200           mov edx, dword ptr [0042DBFC]
                   ====>EDX=7904
     取[00540288]内存处的值
:00421DE0 A100DC4200             mov eax, dword ptr [0042DC00]
               ====>EAX=qmx
☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆
[00540284]内存处的值：
程序自给？！
00540284  71
6D 78 00 37 39 30 34 00 02 00 1E 00 1E 00 D0  qmx.7904....
☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆
:00421DE5 8B0D38DD4200           mov ecx, dword ptr [0042DD38]
         ====>ECX=17359（H）=95605（D）呵呵，系统代码
:00421DEB 52                     push edx
:00421DEC 668B15FADB4200         mov dx, word ptr [0042DBFA]
:00421DF3 50                     push eax
:00421DF4 E897FBFFFF             call 00421990
                       ====>算法CALL！得出下面的EAX值。进入！
:00421DF9 2BC7                   sub eax, edi
             ====>EAX=3403B020 - 11B14=3402950C
\*
Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00421DD2(U)
|
:00421DFB 85C0                   test eax, eax
             ====>相减结果是否为0？即：上面2部分是否相等？
:00421DFD 0F848E000000           je 00421E91
     ====>不为0则不跳则OVER！
\*
Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:00421DC9(C),
:00421DD8(C)
|
:00421E03 B8FBFFFFFF             mov eax, FFFFFFFB
:00421E08 EB63                   jmp 00421E6D
:00422013 FF1500E24200           call dword ptr [0042E200]
         ====>BAD BOY！
―――――――――――――――――――――――――――――――――
进入算法CALL：
\*
Referenced by a CALL at Addresses:
|:00421DF4   , :004221DA
|
:00421990 53                     push ebx
:00421991 56                     push esi
:00421992 668BDA                 mov bx, dx
:00421995 8BF1                   mov esi, ecx
 ====>ESI=ECX=17359    呵呵，系统代码
:00421997 8B54240C               mov edx, dword ptr [esp+0C]
               ====>EDX=qmx
   从[00540284]处取值
:0042199B 8ACB                   mov cl, bl
:0042199D 57                     push edi
:0042199E 81E1FF000000           and ecx, 000000FF
:004219A4 E857FFFFFF             call 00421900
                     ====>对程序给的qmx进行运算得出下面的EAX值！进入关键CALL！
:004219A9 8B542414               mov edx, dword ptr [esp+14]
               ====>EDX=7904
   从[00540288]处取值
:004219AD 8BF8                   mov edi, eax
             ====>EDI=EAX=00003BEE（H）=15342（D）
:004219AF 33C0                   xor eax, eax
:004219B1 8AC7                   mov al, bh
:004219B3 6685C0                 test ax, ax
:004219B6 7512                   jne 004219CA
:004219B8 33C9                   xor ecx, ecx
:004219BA E841FFFFFF             call 00421900
```
