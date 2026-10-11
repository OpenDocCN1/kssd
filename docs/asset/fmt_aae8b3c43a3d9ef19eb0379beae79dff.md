
004D439B   |.  40                        |inc eax

004D439C   |.  83FA 38                   |cmp edx,38

004D439F   |.^ 75 AD                    \jnz short <unpack.loc_4D434E>

004D43A1   |.  8B75 F0                  |mov esi,dword ptr ss:[ebp-10]

004D43A4   |.  8B5D F4                  |mov ebx,dword ptr ss:[ebp-C]

004D43A7   |.  8BE5                     |mov esp,ebp

004D43A9   |.  5D                       |pop ebp

004D43AA   \\.  C2 0400                 |retn 4

```
{
004D452C   |.  BF 10000000             mov edi,10
004D4531   |.  BB 94E45300             mov ebx,unpack.0053E494
004D4536   |.  8B75 FC                 mov esi,dword ptr ss:[ebp-4]                ;  offset SubKey
004D4539 <>|>  8D45 F1                 /lea eax,dword ptr ss:[ebp-F]               ;  loc_4D4539
004D453C   |.  8A0B                    |mov cl,byte ptr ds:[ebx]
004D453E   |.  BA 03000000             |mov edx,3
004D4543   |.  E8 00FFFFFF             |call <unpack.sub_4D4448>
004D4548   |.  8D45 ED                 |lea eax,dword ptr ss:[ebp-13]
004D454B   |.  8A0B                    |mov cl,byte ptr ds:[ebx]
004D454D   |.  BA 03000000             |mov edx,3
004D4552   |.  E8 F1FEFFFF             |call <unpack.sub_4D4448>
004D4557   |.  8A55 F1                 |mov dl,byte ptr ss:[ebp-F]
004D455A   |.  C1E2 04                 |shl edx,4
004D455D   |.  8A45 F2                 |mov al,byte ptr ss:[ebp-E]
004D4560   |.  33C9                    |xor ecx,ecx
004D4562   |.  8AC8                    |mov cl,al
004D4564   |.  C1E9 04                 |shr ecx,4
004D4567   |.  0AD1                    |or dl,cl
004D4569   |.  8855 E6                 |mov byte ptr ss:[ebp-1A],dl
004D456C   |.  C1E0 04                 |shl eax,4
004D456F   |.  33D2                    |xor edx,edx
004D4571   |.  8A55 F3                 |mov dl,byte ptr ss:[ebp-D]
004D4574   |.  C1EA 04                 |shr edx,4
004D4577   |.  0AC2                    |or al,dl
004D4579   |.  8845 E7                 |mov byte ptr ss:[ebp-19],al
004D457C   |.  8A55 F3                 |mov dl,byte ptr ss:[ebp-D]
004D457F   |.  C1E2 04                 |shl edx,4
004D4582   |.  8A45 F4                 |mov al,byte ptr ss:[ebp-C]
004D4585   |.  33C9                    |xor ecx,ecx
004D4587   |.  8AC8                    |mov cl,al
004D4589   |.  C1E9 04                 |shr ecx,4
004D458C   |.  0AD1                    |or dl,cl
004D458E   |.  8855 E8                 |mov byte ptr ss:[ebp-18],dl
004D4591   |.  C1E0 04                 |shl eax,4
004D4594   |.  0A45 ED                 |or al,byte ptr ss:[ebp-13]
004D4597   |.  8845 E9                 |mov byte ptr ss:[ebp-17],al
004D459A   |.  8A45 EE                 |mov al,byte ptr ss:[ebp-12]
004D459D   |.  8845 EA                 |mov byte ptr ss:[ebp-16],al
004D45A0   |.  8A45 EF                 |mov al,byte ptr ss:[ebp-11]
004D45A3   |.  8845 EB                 |mov byte ptr ss:[ebp-15],al
004D45A6   |.  8A45 F0                 |mov al,byte ptr ss:[ebp-10]
004D45A9   |.  8845 EC                 |mov byte ptr ss:[ebp-14],al
004D45AC   |.  6A 05                   |push 5                                    ; /Arg1 = 00000005
004D45AE   |.  8BCE                    |mov ecx,esi                               ; |
004D45B0   |.  8D45 E6                 |lea eax,dword ptr ss:[ebp-1A]             ; |
004D45B3   |.  BA 06000000             |mov edx,6                                 ; |
004D45B8   |.  E8 F3FDFFFF             |call <unpack.sub_4D43B0>                  ; \unpack.004D43B0
004D45BD   |.  83C6 06                 |add esi,6
004D45C0   |.  43                       |inc ebx
004D45C1   |.  4F                       |dec edi
004D45C2   |.^ 0F85 71FFFFFF           \jnz <unpack.loc_4D4539>
004D45C8   |.  8B7D D8                 mov edi,dword ptr ss:[ebp-28]             ;这里最后生成 16 个子密钥
}
```

---------------------------------------------------------------------------------------

16 个 48 位的子密钥

00541DAC  F0 B6 6E 75 28 30 F0 BE F6 82 C2 85 F4 F6 72 D2  鸲 nu(0 鹁鰝聟赧 r

00541DBC  07 81 E6 D7 76 9A 03 09 EE D3 57 52 73 00 2F D3  佹譾?.钣 WRs./

00541DCC  7B 70 21 28 AF 51 FB E0 38 0A BF 4B D9 64 32 3A  {p!(疩8.縆賒 2:

00541DDC  3F 59 DB 8A C5 01 3F 69 CD 4A 47 04 1B 6D 9D D8  ?Y 蹔??i 蚃 Gm 澵

00541DEC  41 88 5D 2D BD C0 52 09 D7 AC AD D2 32 28 DB AE  A 圿-嚼 R.赚2(郛

00541DFC  A6 B0 1B 28 F8 BE AE 10 3A 32 E1 BE A6 88 41 C5  Π(?:2 峋A

---------------------------------------------------------------------------------------

下面就是把解密出的用户名再进行加密，虽然密钥初始化为`"SecurityOne2005c"`，但实际上还是只用了`"Security"`

```
004D4C93   |.  E8 40FCFFFF             call <unpack.sub_4D48D8>        ;对用户名 DES 加密
00539AB3   |.  8B45 D8                 mov eax,dword ptr ss:[ebp-28]    ;加密后的结果
00539AB6   |.  8D4D DC                 lea ecx,dword ptr ss:[ebp-24]
00539AB9   |.  BA 0A000000             mov edx,0A                      ;取最左边的 10 个字符
00539ABE <>|.  E8 71DCEFFF             call unpack.00437734            ;  ->StrUtils.LeftStr(AnsiString;Integer):AnsiString;overload;
00539AC3   |.  8D45 DC                 lea eax,dword ptr ss:[ebp-24]
00539AC6   |.  50                      push eax
00539AC7   |.  8D55 D0                 lea edx,dword ptr ss:[ebp-30]
00539ACA   |.  8B45 FC                 mov eax,dword ptr ss:[ebp-4]
00539ACD   |.  E8 6ABFF9FF             call unpack.004D5A3C
```

--------------------------------------------------------------------------------------------

```
004D5941   |.  E8 AEFEFFFF             call unpack.004D57F4
```

---------------------------------------------------------------------------------------------

下面就是 MD5 变形的地方，标准 MD5 初始化的四个常数为

--------------------------------------------------

```
state[0] = 0x67452301;
state[1] = 0xefcdab89;
state[2] = 0x98badcfe;
state[3] = 0x10325476;
```

----------------------------------------------------

而这个变形的 MD5 变就变在这个常数上

```
{
004D57F4   /$  C700 4365789A           mov dword ptr ds:[eax],9A786543      ;这里四个 constants 发生变化
004D57FA   |.  C740 04 01234567        mov dword ptr ds:[eax+4],67452301
004D5801   |.  C740 08 76543210        mov dword ptr ds:[eax+8],10325476
004D5808   |.  C740 0C A9876543        mov dword ptr ds:[eax+C],436587A9
004D580F   |.  33D2                    xor edx,edx
004D5811   |.  8950 10                 mov dword ptr ds:[eax+10],edx
004D5814   |.  33D2                    xor edx,edx
004D5816   |.  8950 14                 mov dword ptr ds:[eax+14],edx
004D5819   |.  83C0 18                 add eax,18
004D581C   |.  BA 40000000             mov edx,40
004D5821   |.  E8 FE18F3FF             call unpack.00407124
004D5826   \\.  C3                     retn
}
```

----------------------------------------------------------------------------------

```
004D5946   |.  8B45 FC                 mov eax,dword ptr ss:[ebp-4]
004D5949   |.  E8 B6EAF2FF             call unpack.00404404
004D594E   |.  50                      push eax
004D594F   |.  8B45 FC                 mov eax,dword ptr ss:[ebp-4]
004D5952   |.  E8 ADECF2FF             call unpack.00404604
004D5957   |.  8BD0                    mov edx,eax
004D5959   |.  8D45 A4                 lea eax,dword ptr ss:[ebp-5C]
004D595C   |.  59                      pop ecx
004D595D   |.  E8 C6FEFFFF             call unpack.004D5828
004D5962   |.  8BD3                    mov edx,ebx
004D5964   |.  8D45 A4                 lea eax,dword ptr ss:[ebp-5C]
004D5967   |.  E8 3CFFFFFF             call unpack.004D58A8
```

--------------------------------------------------------------------
