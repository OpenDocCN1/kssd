
:0040624D 80EA06  sub dl, 06

:00406250 7217  jb 00406269

:00406252 80C2E6  add dl, E6

:00406255 80EA06  sub dl, 06

:00406258 721C  jb 00406276

:0040625A EB7A  jmp 004062D6 <==循环了 5 次后，就这里跳出

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:00406248(C)

|

:0040625C 8BF8  mov edi, eax

:0040625E 81E7FF000000  and edi, 000000FF

:00406264 83EF30  sub edi, 00000030

:00406267 EB18  jmp 00406281

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:00406250(C)

|

:00406269 8BF8  mov edi, eax

:0040626B 81E7FF000000  and edi, 000000FF

:00406271 83EF37  sub edi, 00000037

:00406274 EB0B  jmp 00406281

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:00406258(C)

|

:00406276 8BF8  mov edi, eax

:00406278 81E7FF000000  and edi, 000000FF

:0040627E 83EF57  sub edi, 00000057

\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

|:00406267(U), :00406274(U)

|

:00406281 837C240C00  cmp dword ptr [esp+0C], 00000000

:00406286 7509  jne 00406291

:00406288 837C240800  cmp dword ptr [esp+08], 00000000

:0040628D 7247  jb 004062D6

:0040628F EB02  jmp 00406293

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:00406286(C)

|

:00406291 7C43  jl 004062D6

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:0040628F(U)

|

:00406293 817C240CFFFFFF07  cmp dword ptr [esp+0C], 07FFFFFF

:0040629B 7509  jne 004062A6

:0040629D 837C2408FF  cmp dword ptr [esp+08], FFFFFFFF

:004062A2 7604  jbe 004062A8

:004062A4 EB30  jmp 004062D6

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:0040629B(C)

|

:004062A6 7F2E  jg 004062D6

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:004062A2(C)

|

:004062A8 8BC7  mov eax, edi

:004062AA 99  cdq

:004062AB 52  push edx

:004062AC 50  push eax

:004062AD 8B442410  mov eax, dword ptr [esp+10]

:004062B1 8B542414  mov edx, dword ptr [esp+14]

:004062B5 0FA4C204  shld edx, eax, 04

:004062B9 C1E004  shl eax, 04

:004062BC 030424  add eax, dword ptr [esp]

:004062BF 13542404  adc edx, dword ptr [esp+04]

:004062C3 83C408  add esp, 00000008

:004062C6 89442408  mov dword ptr [esp+08], eax

:004062CA 8954240C  mov dword ptr [esp+0C], edx

:004062CE 45  inc ebp

:004062CF 33DB  xor ebx, ebx

:004062D1 E966FFFFFF  jmp 0040623C  <===从这里往上跳构成循环，5 次

\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

|:0040625A(U), :0040628D(C), :00406291(C), :004062A4(U), :004062A6(C)

|

:004062D6 807C241000  cmp byte ptr [esp+10], 00  <==从 0040625A 行跳到这里

:004062DB 0F84D3000000  je 004063B4  <==是 0，所以再次跳走

:004062E1 8B442408  mov eax, dword ptr [esp+08]

:004062E5 8B54240C  mov edx, dword ptr [esp+0C]

:004062E9 F7D8  neg eax

:004062EB 83D200  adc edx, 00000000

:004062EE F7DA  neg edx

:004062F0 89442408  mov dword ptr [esp+08], eax

:004062F4 8954240C  mov dword ptr [esp+0C], edx

:004062F8 E9B7000000  jmp 004063B4

\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

|:0040621D(C), :0040622D(C), :0040636B(U)

|

:004062FD 8A442EFF  mov al, byte ptr [esi+ebp\-01]

\*\*\*\*\*\* 如果，前两位不是 0x，则从 0040621D 跳到这一行，依次取注册码的值，放入 AL

:00406301 8BD0  mov edx, eax

:00406303 80C2D0  add dl, D0

:00406306 80EA0A  sub dl, 0A

:00406309 7362  jnb 0040636D

:0040630B 8BF8  mov edi, eax

:0040630D 81E7FF000000  and edi, 000000FF

:00406313 83EF30  sub edi, 00000030

:00406316 837C240C00  cmp dword ptr [esp+0C], 00000000

:0040631B 7509  jne 00406326

:0040631D 837C240800  cmp dword ptr [esp+08], 00000000

:00406322 7249  jb 0040636D

:00406324 EB02  jmp 00406328

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:0040631B(C)

|

:00406326 7C45  jl 0040636D

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:00406324(U)

|

:00406328 817C240CCCCCCC0C  cmp dword ptr [esp+0C], 0CCCCCCC

:00406330 750C  jne 0040633E

:00406332 817C2408CCCCCCCC  cmp dword ptr [esp+08], CCCCCCCC

:0040633A 7604  jbe 00406340

:0040633C EB2F  jmp 0040636D

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:00406330(C)

|

:0040633E 7F2D  jg 0040636D

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:0040633A(C)

|

:00406340 6A00  push 00000000

:00406342 6A0A  push 0000000A

:00406344 8B442410  mov eax, dword ptr [esp+10]

:00406348 8B542414  mov edx, dword ptr [esp+14]

:0040634C E873FCFFFF  call 00405FC4

:00406351 52  push edx

:00406352 50  push eax

:00406353 8BC7  mov eax, edi

:00406355 99  cdq

:00406356 030424  add eax, dword ptr [esp]

:00406359 13542404  adc edx, dword ptr [esp+04]

:0040635D 83C408  add esp, 00000008

:00406360 89442408  mov dword ptr [esp+08], eax

:00406364 8954240C  mov dword ptr [esp+0C], edx

:00406368 45  inc ebp

:00406369 33DB  xor ebx, ebx

:0040636B EB90  jmp 004062FD

\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

|:00406309(C), :00406322(C), :00406326(C), :0040633C(U), :0040633E(C)

|

:0040636D 807C241000  cmp byte ptr [esp+10], 00

:00406372 7417  je 0040638B

:00406374 8B442408  mov eax, dword ptr [esp+08]

:00406378 8B54240C  mov edx, dword ptr [esp+0C]

:0040637C F7D8  neg eax

:0040637E 83D200  adc edx, 00000000

:00406381 F7DA  neg edx

:00406383 89442408  mov dword ptr [esp+08], eax

:00406387 8954240C  mov dword ptr [esp+0C], edx

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:00406372(C)

|

:0040638B 837C240C00  cmp dword ptr [esp+0C], 00000000

:00406390 7505  jne 00406397

:00406392 837C240800  cmp dword ptr [esp+08], 00000000

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:00406390(C)

|

:00406397 741B  je 004063B4

:00406399 837C240C00  cmp dword ptr [esp+0C], 00000000

:0040639E 750A  jne 004063AA

:004063A0 837C240800  cmp dword ptr [esp+08], 00000000

:004063A5 0F92C0  setb al
