
# Search32 最新版 V5.08 注册算法笔记 (二)

**（续上篇）**

## 初始 ID 字符串处理

```asm
:100013BB 3BF3                 cmp esi, ebx
:100013BD 0F84F2030000         je 100017B5//判断字串存在与否的跳转; ****
:100013C3 46                   inc esi
:100013C4 56                   push esi
:100013C5 89742428             mov dword ptr [esp+28], esi
:100013C9 E808020100           call 100115D6// 参考 :10001350 的 call;
:100013CE 8DBC24AC000000       lea edi, dword ptr [esp+000000AC]//指向输入的 ID 字符串;
:100013D5 83C9FF               or ecx, FFFFFFFF
:100013D8 33C0                 xor eax, eax
:100013DA 83C404               add esp, 00000004
:100013DD F2                   repnz
:100013DE AE                   scasb
:100013DF F7D1                 not ecx
:100013E1 49                   dec ecx//算它的长度;
:100013E2 83F920               cmp ecx, 00000020//长度是否 20h 位？
:100013E5 732E                 jnb 10001415
:100013E7 8BFE                 mov edi, esi
```

... (省略部分汇编代码) ...

```asm
:10001404 8BCA                 mov ecx, edx
:10001406 4F                   dec edi
:10001407 C1E902               shr ecx, 02
:1000140A F3                   repz
:1000140B A5                   movsd
:1000140C 8BCA                 mov ecx, edx
:1000140E 83E103               and ecx, 00000003
:10001411 F3                   repz
:10001412 A4                   movsb//将“SRCH32_D.DLL”移动到 ID 字符串的后面，得新 ID 字串；
                                //即 “MOONLITE&GROUP [FCG]SRCH32_D.DLL”
:10001413 EB0B                 jmp 10001420
```

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:
```
:100013E5(C)
```

```asm
:10001415 8D840C9C000000       lea eax, dword ptr [esp+ecx+0000009C]
:1000141C 89442424             mov dword ptr [esp+24], eax
```

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:
```
:10001413(U)
```

```asm
:10001420 8BB424F4020000       mov esi, dword ptr [esp+000002F4]//指向输入的注册码;
:10001427 83C9FF               or ecx, FFFFFFFF
:1000142A 8BFE                 mov edi, esi
:1000142C 33C0                 xor eax, eax
:1000142E F2                   repnz
:1000142F AE                   scasb
:10001430 F7D1                 not ecx
:10001432 49                   dec ecx//计算所输入的注册码的长度;
:10001433 8BD1                 mov edx, ecx//送 edx;
:10001435 83FA18               cmp edx, 00000018//18h=24 位;
:10001438 89542418             mov dword ptr [esp+18], edx//将注册码的长度保存;
:1000143C 0F8273030000         jb 100017B5//注册码的长度小于 24 位，就完了; ****
:10001442 B907000000           mov ecx, 00000007//置循环次数;
:10001447 B84D4D4D4D           mov eax, 4D4D4D4D
:1000144C 8D7C2448             lea edi, dword ptr [esp+48]
:10001450 F3                   repz
:10001451 AB                   stosd
:10001452 66AB                 stosw
:10001454 AA                   stosb//到此，将 4*7+2+1=31 字节用“4D”填满:
:10001455 B81F000000           mov eax, 0000001F
:1000145A B14D                 mov cl, 4D
```

... (省略部分汇编代码) ...

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:
```
:100014FB(C)
```

```asm
:10001475 0FBE0C33             movsx ecx, byte ptr [ebx+esi]//将输入的注册码的第一个字符送 ecx;
:10001479 51                   push ecx
:1000147A E8D7970000           call 1000AC56//将注册码字符查表，0～9 数字对应“84”,返回 eax=4;
:1000147F 83C404               add esp, 00000004
:10001482 85C0                 test eax, eax
:10001484 7512                 jne 10001498
```

------------------------------------------------------------------------------------------------------------------------------

将输入的注册码按奇数和偶数位分两组，并按不同次序排列;

如果注册码的形式是：

`a1 a2 a3 a4 a5 a6 a7 a8 a9 a10 a11 a12 a13 a14 a15 a16 a17 a18 a19 a20 a21 a22 a23 a24`

则分组后排列为：

*   奇数位组: `a1 a3 a5 a7 a9 a11 a13 a15 a17 a19 a21 a23`
*   偶数位组: `a24 a20 a16 a12 a8 a4 a2 a6 a10 a14 a18 a20` （以 `a2` 为对称中心）

------------------------------------------------------------------------------------------------------------------------------

```asm
:100014F8 43                   inc ebx
:100014F9 3BD8                 cmp ebx, eax
:100014FB 0F8274FFFFFF         jb 10001475
```

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:
```
:1000146F(C)
```

```asm
:10001501 8D7C2448             lea edi, dword ptr [esp+48]//指向偶数位组地址: 0070f01c;
:10001505 83C9FF               or ecx, FFFFFFFF
:10001508 33C0                 xor eax, eax
:1000150A C6442C4800           mov [esp+ebp+48], 00
:1000150F F2                   repnz
:10001510 AE                   scasb//计算偶数位组中（从 70f01c 开始）后半部分的长度;
:10001511 8B542420             mov edx, dword ptr [esp+20]
:10001515 F7D1                 not ecx
:10001517 49                   dec ecx
:10001518 C644142800           mov [esp+edx+28], 00
:1000151D 83E1FE               and ecx, FFFFFFFE
:10001520 83F90A               cmp ecx, 0000000A//该长度与“A”比较;
:10001523 0F828C020000         jb 100017B5//小于则失败; ****
:10001529 8D442448             lea eax, dword ptr [esp+48]//指向偶数位组地址: 0070f01c;
:1000152D 50                   push eax
:1000152E E818970000           call 1000AC4B//偶数位组后半部分的计算, 结果送 eax;
                                //比方这部分的 ASI 字符的形式是: 00000083450, 那么
                                //eax 返回 83450 的 HEX 值; 假如该 ASCII 字符大于
                                //FFFFFFFFh=4294967295, 则返回的 eax 就不是此规律了;
:10001533 8B7C2428             mov edi, dword ptr [esp+28]
:10001537 8BD8                 mov ebx, eax//eax 送 ebx 保存;
:10001539 A1086C0110           mov eax, dword ptr [10016C08]//置数;
:1000153E 83C9FF               or ecx, FFFFFFFF
:10001541 33D8                 xor ebx, eax
:10001543 33C0                 xor eax, eax
```

... (省略部分汇编代码) ...

```asm
:1000155D 8BCD                 mov ecx, ebp
:1000155F 4F                   dec edi
:10001560 C1E902               shr ecx, 02
:10001563 F3                   repz
:10001564 A5                   movsd
:10001565 8BCD                 mov ecx, ebp
:10001567 83E103               and ecx, 00000003
:1000156A F3                   repz
:1000156B A4                   movsb//将“SRCH32_D.DLL”移动并接到奇数位组的后面,得新奇数位组字串;
:1000156C 8DBC24A8000000       lea edi, dword ptr [esp+000000A8]//指向新 ID 字串;
:10001573 83C9FF               or ecx, FFFFFFFF
:10001576 F2                   repnz
:10001577 AE                   scasb
:10001578 F7D1                 not ecx
:1000157A 49                   dec ecx//得新 ID 字串的长度，我的是 20h;
:1000157B 8D7C2428             lea edi, dword ptr [esp+28]//指向奇数位组字串;
:1000157F 8BD1                 mov edx, ecx
:10001581 83C9FF               or ecx, FFFFFFFF
:10001584 F2                   repnz
:10001585 AE                   scasb
:10001586 F7D1                 not ecx
:10001588 49                   dec ecx//计算它的长度;
:10001589 83FA20               cmp edx, 00000020//新 ID 字串的长度与 20h 比较;
:1000158C 894C2418             mov dword ptr [esp+18], ecx
:10001590 0F821F020000         jb 100017B5//小于的话，命运就惨了！！****
:10001596 85D2                 test edx, edx
:10001598 7617                 jbe 100015B1
```

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:
```
:100015AF(C)
```
