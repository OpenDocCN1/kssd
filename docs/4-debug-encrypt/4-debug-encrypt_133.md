
# TMG keygenmersa

> 原文：[`www.pediy.com/kssd/pediy04/kanxue084.htm`](https://www.pediy.com/kssd/pediy04/kanxue084.htm)

输入：
```
username: lancelot
company:  [CCG]
code:     1234567890123456789012345678901234567890
```

=========================================================================================

Referenced by a CALL at Address: `:0040107A`

```
:00401187 C8080000                enter 0008, 00
:0040118B BFF0214000              mov edi, 004021F0
:00401190 33C0                    xor eax, eax
:00401192 6A08                    push 00000008
:00401194 59                      pop ecx
:00401195 F3                      repz
:00401196 AB                      stosd
:00401197 68F0204000              push 004020F0
:0040119C 6A3D                    push 0000003D
:0040119E 6A0D                    push 0000000D
:004011A0 FF35DC204000            push dword ptr [004020DC]

Reference To: USER32.SendMessageA, Ord:01C0h

:004011A6 E8890B0000              Call 00401D34<---------------------读取用户名
:004011AB 83F805                  cmp eax, 00000005<-----------------是否小于 5 个字
:004011AE 0F8CE7010000            jl 0040139B<-----------------------小于则认为是无效用户名
:004011B4 8945FC                  mov dword ptr [ebp-04], eax
:004011B7 6870214000              push 00402170
:004011BC 6A3D                    push 0000003D
:004011BE 6A0D                    push 0000000D
:004011C0 FF35E0204000            push dword ptr [004020E0]

Reference To: USER32.SendMessageA, Ord:01C0h

:004011C6 E8690B0000              Call 00401D34<---------------------读取公司名
:004011CB 83F805                  cmp eax, 00000005<-----------------是否小于 5 个字
:004011CE 0F8CE0010000            jl 004013B4<-----------------------小于则认为是无效公司名
:004011D4 8945F8                  mov dword ptr [ebp-08], eax
:004011D7 68F0214000              push 004021F0
:004011DC 6A32                    push 00000032
:004011DE 6A0D                    push 0000000D
:004011E0 FF35E4204000            push dword ptr [004020E4]

Reference To: USER32.SendMessageA, Ord:01C0h

:004011E6 E8490B0000              Call 00401D34<---------------------读取注册码
:004011EB 66C705702340000A00      mov word ptr [00402370], 000A
:004011F4 6A03                    push 00000003
:004011F6 33D2                    xor edx, edx
:004011F8 69C0B5070000            imul eax, 000007B5
:004011FE 5B                      pop ebx
:004011FF F7F3                    div ebx<----------------------------此处要让 eax==0x66c2
:00401201 8D803E99FFFF            lea eax, dword ptr [eax+FFFF993E]<--此处要让 eax==0
:00401207 83F801                  cmp eax, 00000001<------------------比较完后要让 CF==1
:0040120A 1BC0                    sbb eax, eax
:0040120C 0F8470010000            je 00401382<-----------------------不能跳,所以注册码长度为 40 位
:00401212 BF72234000              mov edi, 00402372
:00401217 BEF0214000              mov esi, 004021F0
:0040121C 56                      push esi

Reference To: USER32.CharUpperA, Ord:002Ah

:0040121D E8240B0000              Call 00401D46
:00401222 6A14                    push 00000014
:00401224 59                      pop ecx

Referenced by a (U)nconditional or (C)onditional Jump at Address:
:00401254(C)

:00401225 66AD                    lodsw
:00401227 66257F7F                and ax, 7F7F
:0040122B 662D3030                sub ax, 3030
:0040122F 3C19                    cmp al, 19
:00401231 0F874B010000            ja 00401382<------------------------每 1 位注册码的 ASCII 值要小于 0x39
:00401237 80FC19                  cmp ah, 19
:0040123A 0F8742010000            ja 00401382<------------------------每 1 位注册码的 ASCII 值要小于 0x39
:00401240 0FB6DC                  movzx ebx, ah
:00401243 0FB6C0                  movzx eax, al
:00401246 8A8058124000            mov al, byte ptr [eax+00401258]
:0040124C 0A8373124000            or al, byte ptr [ebx+00401273]
:00401252 AA                      stosb
:00401253 49                      dec ecx
:00401254 7FCF                    jg 00401225<------------------------以上将注册码每两位合并
:00401256 EB36                    jmp 0040128E<-----------------------即 12345678=>0x78563412

Referenced by a (U)nconditional or (C)onditional Jump at Address:
:00401256(U)

:0040128E BF30224000              mov edi, 00402230
:00401293 33C0                    xor eax, eax<-----------------------eax==0
:00401295 6A05                    push 00000005
:00401297 59                      pop ecx
:00401298 F3                      repz
:00401299 AB                      stosd
:0040129A 48                      dec eax<----------------------------eax==ffffffff
:0040129B BEF0204000              mov esi, 004020F0<-----------------"lancelot"<---------用户名
:004012A0 BF30224000              mov edi, 00402230
:004012A5 8B4DFC                  mov ecx, dword ptr [ebp-04]<-------8 个字
:004012A8 33D2                    xor edx, edx

Referenced by a (U)nconditional or (C)onditional Jump at Address:
:004012E0(C)

:004012AA AC                      lodsb
:004012AB 32E0                    xor ah, al
:004012AD 69C013731789            imul eax, 89177313
:004012B3 25AA55AA55              and eax, 55AA55AA
:004012B8 69C081932912            imul eax, 12299381
:004012BE 3511AA55AA              xor eax, AA55AA11
:004012C3 6BC061                  imul eax, 00000061
:004012C6 32E0                    xor ah, al
:004012C8 0D18010310              or eax, 10030118
:004012D3 D3D0                    rcl eax, cl
:004012D5 310417                  xor dword ptr [edi+edx], eax
:004012D8 83C203                  add edx, 00000003
:004012DB 83E20F                  and edx, 0000000F
:004012DE 42                      inc edx
:004012DF 49                      dec ecx<---------------------------计数器减 1
:004012E0 7FC8                    jg 004012AA<---------------循环<--结果:d331d86e5fa0445cb96d6ba55e9f91823ec06285
:004012E2 BE70214000              mov esi, 00402170<-----------------"[CCG]"<------公司名
:004012E7 BF30224000              mov edi, 00402230
:004012EC 8B4DF8                  mov ecx, dword ptr [ebp-08]<-------5 个字
:004012EF 33D2                    xor edx, edx

Referenced by a (U)nconditional or (C)onditional Jump at Address:
:00401327(C)

:004012F1 AC                      lodsb
:004012F2 2AE0                    sub ah, al
:004012F4 69C013731589            imul eax, 89157313
:004012FA 25AA55AA55              and eax, 55AA55AA
:004012FF 69C087932912            imul eax, 12299387
:00401305 0D11AA55AA              or eax, AA55AA11
:0040130A 6BC061                  imul eax, 00000061
:0040130D 3514010110              xor eax, 10010114
:00401312 69C079829107            imul eax, 07918279
:00401318 32E0                    xor ah, al
:0040131A D3D8                    rcr eax, cl
:0040131C 310417                  xor dword ptr [edi+edx], eax
:0040131F 83C203                  add edx, 00000003
:00401322 83E20F                  and edx, 0000000F
:00401325 42                      inc edx
:00401326 49                      dec ecx<---------------------------计数器减 1
:00401327 7FC8                    jg 004012F1<----------------循环<--结果:422055b97f4b15f5f9412ef971abd42827f382cd
:00401329 2B4708                  sub eax, dword ptr [edi+08]<-------eax==0x48e03319-0xf92e41f9==0x4fb1f120
:0040132C 69C015488134            imul eax, 34814815<----------------eax==0x9689c7a0
```



:00401332 `014710` `add dword ptr [edi+10], eax` `0xcd82f327+0x9689c7a0==0x640cbac7`

:00401335 `C1E80B` `shr eax, 0B` `eax==0x0012d138`

:00401338 `83E003` `and eax, 00000003` `eax==0x00`

:0040133B `8807` `mov byte ptr [edi], al` 结果: `002055b97f4b15f5f9412ef971abd428c7ba0c64`

:0040133D `6870224000` `push 00402270`

:00401342 `6870234000` `push 00402370` `0a001234567890123456789012345678901234567890` 输入的注册码(C')

\* Possible StringData Ref from Data Obj ->""

|

:00401347 `6800204000` `push 00402000` `0a00d2e9bf9b3d258e479d8cc23c7a33e1f8ebb3adb1` 这就是(n)

:0040134C `E87C000000` `call 004013CD` (1)开始 RSA 计算!!!

:00401351 `6A05` `push 00000005`

:00401353 `59` `pop ecx`

:00401354 `BE72224000` `mov esi, 00402272` (1)的结果-----`96e10aebbf748d31c4cdc7e1846a2361e3575546`

:00401359 `BF30224000` `mov edi, 00402230` `002055b97f4b15f5f9412ef971abd428c7ba0c64`=(用户名+公司名)也就是(M)

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00401367(C)
|

:0040135E `AD` `lodsd`

:0040135F `3107` `xor dword ptr [edi], eax`

:00401361 `751F` `jne 00401382` 要让 xor 后的值为 0

:00401363 `83C704` `add edi, 00000004`

:00401366 `49` `dec ecx`

:00401367 `7FF5` `jg 0040135E`

:00401369 `6A00` `push 00000000`

\* Possible StringData Ref from Data Obj ->"Well done" 成功

|

:0040136B `68C9204000` `push 004020C9`

\======================================================

... ... 到这里后，停了好几天，因跟进 call (1) 后，不知所云，无法推出 e 和 d

多谢伪装者提供的老外的过程，马上照着做，嘻嘻... ...

\=====================================================================================================

已知：`n = p * q`

`(d * e) mod ((p-1) * (q-1)) = 1`

`M = (C^e) mod n`

`C = (M^d) mod n`

按照老外的教程:*** 要用大写字母,试了好久才知道，呵呵，错怪老外了 ***

`n = D2E9BF9B3D258E479D8CC23C7A33E1F8EBB3ADB1` 160 Bits 没错!

`e = 65537` 由于程序中 RSA 计算前没有入栈 e, 所以 e 为预设值(default)--65537

用 `RSA tool 2 v1.4`, 输入 n (要用大写字母,试了好久才知道，呵呵，错怪老外了), 分解因子

`p = D49CAA009DADB2128673`

`q = FDF4549DF6ACE8640E4B`

`d = C6546E0C11ACCE2543DD1150C4CE7A05A4C8FA3D`

剩下的就是写注册机了

\=======================================================

          ,;~;,
            /\_
           ( /
           (()   //)    
           | \  ,,;;'\
           __ _( )m=(lancelot(================--------
         /' ' '()/~' '.(, |
       ,;()|| | ~
     ,;' \ /-(.;, )           兰斯洛特[CCG][FCG]
         ) / ) /
       //       ||               2001.10.05
       )\_       )\_

\======================================================== 4dd466d0e1e0cc123afbe504da2462ca
