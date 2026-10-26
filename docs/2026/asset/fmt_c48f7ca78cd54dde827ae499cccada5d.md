
```asm
.text:0042B20A                 mov     dl, [esp+eax+17h]        //转换大小尾

.text:0042B20E                 mov     cl, [esp+eax+16h]

.text:0042B212                 mov     [esp+eax+7Ch], dl

.text:0042B216                 mov     edx, [esp+eax+14h]

.text:0042B21A                 mov     [esp+eax+7Dh], cl

.text:0042B21E                 mov     cl, [esp+eax+14h]

.text:0042B222                 shr     edx, 8

.text:0042B225                 mov     [esp+eax+7Eh], dl

.text:0042B229                 mov     [esp+eax+7Fh], cl

.text:0042B22D                 add     eax, 4

.text:0042B230                 cmp     eax, 20h

.text:0042B233                 jl      short loc_42B20A

.text:0042B235                 lea     edx, [esp+7Ch]

.text:0042B239                 lea     ecx, [esp+10h]

.text:0042B23D                 push    edx

.text:0042B23E                 call    CString::CString(char const *)

.text:0042B243                 mov     eax, [esp+10h]

.text:0042B247                 mov     ecx, [esp+0Ch]

.text:0042B24B                 push    eax                    //RSA结果

.text:0042B24C                 push    ecx                    //用户名

.text:0042B24D                 call    esi ; _mbscmp          //比较两个字符串是否相等

.text:0042B24F                 add     esp, 8

.text:0042B252                 lea     ecx, [esp+10h]

.text:0042B256                 test    eax, eax

.text:0042B258                 mov     byte ptr [esp+0A4h], 6

.text:0042B260                 jz      loc_42B2EC             //相等则返回1

.text:0042B266                 call    CString::~CString(void)

.text:0042B26B                 lea     ecx, [esp+4Ch]

.text:0042B26F                 mov     byte ptr [esp+0A4h], 5

.text:0042B277                 call    sub_401BC7

.text:0042B27C                 lea     ecx, [esp+34h]

.text:0042B280                 mov     [esp+0A4h], bl

.text:0042B287                 call    sub_401BC7

.text:0042B28C                 lea     ecx, [esp+44h]

.text:0042B290                 mov     byte ptr [esp+0A4h], 8

.text:0042B298                 call    sub_401BC7

.text:0042B29D                 lea     ecx, [esp+3Ch]

.text:0042B2A1                 mov     byte ptr [esp+0A4h], 1

.text:0042B2A9                 call    sub_401BC7

.text:0042B2AE

.text:0042B2AE loc_42B2AE:

.text:0042B2AE

.text:0042B2AE                 lea     ecx, [esp+8]

.text:0042B2B2                 mov     byte ptr [esp+0A4h], 0

.text:0042B2BA                 call    CString::~CString(void)

.text:0042B2BF                 lea     ecx, [esp+0Ch]

.text:0042B2C3                 mov     dword ptr [esp+0A4h], 0FFFFFFFFh

.text:0042B2CE                 call    CString::~CString(void)

.text:0042B2D3                 pop     esi

.text:0042B2D4                 xor     eax, eax

.text:0042B2D6                 pop     ebx

.text:0042B2D7                 mov     ecx, [esp+94h]

.text:0042B2DE                 mov     large fs:0, ecx

.text:0042B2E5                 add     esp, 0A0h

.text:0042B2EB                 retn
```

---

总结：1。预处理注册码。

注册码分为8组，第8组 ＝ （5，4，3，2组之和）异或 第8组

第7组 ＝ （1，6组之和）异或 第7组

2。RSA256运算

`N` : 385AC5EEC9BF2B1AE3D3C333323B9C06DBF030EEE4475D85234E94C90FBF8A47

`E` : 10001

3。变换RSA256运算结果

把形如12 34 56 78 90 ab cd ef 的形式转换为78 56 34 12 ef cd ab 90的形式

4。变换后的结果和用户名比较，相等则注册成功，不等则失败。

注册算法是上面的逆运算，分解一个256位的`N`在我的机器上大概半小时的时间，注册算法就不给出了。感兴趣的可以自己做一下。

---

[全文完]

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)
