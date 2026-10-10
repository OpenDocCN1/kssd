
```asm
.text:587752B6 89 75 F4                                mov     dword ptr [ebp+pvarg.anonymous_0+4], esi
.text:587752B9 89 75 F8                                mov     dword ptr [ebp+pvarg.anonymous_0+8], esi
.text:587752BC 74 62                                   jz      short loc_58775320
.text:587752BE 66 3D 08 40                             cmp     ax, 4008h
.text:587752C2 74 5C                                   jz      short loc_58775320
.text:587752C4 66 83 F8 0B                             cmp     ax, VT_BOOL
.text:587752C8 74 24                                   jz      short loc_587752EE
.text:587752CA 66 3D 0B 40                             cmp     ax, 400Bh
.text:587752CE 74 24                                   jz      short loc_587752F4
.text:587752D0 6A 08                                   push    VT_BSTR         ; vt
.text:587752D2 6A 00                                   push    0               ; wFlags
.text:587752D4 8D 75 10                                lea     esi, [ebp+pvarSrc]
.text:587752D7 68 09 04 00 00                          push    409h           ; lcid
.text:587752DC 8B C6                                   mov     eax, esi
.text:587752DE 50                                      push    eax            ; pvarSrc
.text:587752DF 50                                      push    eax            ; pvargDest
.text:587752E0 FF 15 58 12 76 58                       call    ds:VariantChangeTypeEx(x,x,x,x,x)
```

6. 这里问题出现了，上面对 `VariantChangeTypeEx` 的调用可以看到，`pvarSrc` 和 `pvargDest` 均指向封装了 `TearoffThunk` 类的 Variant 变量。为了将转换的结果写入 `pvargDest` 指向的目标 Variant，`VariantChangeTypeEx` 必须对其原内容进行清理，因此它对 `pvargDest`（由于 `iepeers!CPersistUserData::setAttribute` 调用时传入的参数问题，这里其实也就是指向源 Variant）调用了 `VariantClear`，后者调用 `mshtml!PlainRelease` 减少了相应 `TearoffThunk` 类的访问计数。（感谢 27 楼对该处的指正）

引用:
```asm
.text:770F6B80 loc_770F6B80:                           ; CODE XREF: VariantChangeTypeEx(x,x,x,x,x)-1C5Dj
.text:770F6B80                                 ; VariantChangeTypeEx(x,x,x,x,x)-131Dj
.text:770F6B80                                 ; VariantChangeTypeEx(x,x,x,x,x)-12F2j
.text:770F6B80                                 ; VariantChangeTypeEx(x,x,x,x,x)-D60j
.text:770F6B80                                 ; VariantChangeTypeEx(x,x,x,x,x)-A5Bj
.text:770F6B80                                 ; VariantChangeTypeEx(x,x,x,x,x)-A06j ...
.text:770F6B80  test    ebx, ebx
.text:770F6B82  jl      loc_770F643E
.text:770F6B88  mov     ax, [ebp+vt]
.text:770F6B8C  mov     word ptr [ebp+pvarg], ax
.text:770F6B90  mov     eax, [ebp+pvargDest]
.text:770F6B93  cmp     word ptr [eax], VT_BSTR ; 目标变量的类型是否已转变为 VT_BSTR
.text:770F6B97  jnb     loc_770F4E04    ; 超过 8 则跳，这里目标 Variant 就是源 Variant，类型还是 VT_DISPATCH(9)，跳转实现
.text:770F4E04 loc_770F4E04:                           ; CODE XREF: VariantChangeTypeEx(x,x,x,x,x)+ADj
.text:770F4E04  push    eax            ; pvarg
.text:770F4E05  call    VariantClear(x)
```

7. 由于以上提到的原因，`iepeers!CPersistUserData::setAttribute` 每被调用一次（也就是那个脚本中的 for 循环每循环一次），相应 `TearoffThunk` 对象的访问计数就会被减少 1。多次的循环下来导致了其访问计数被减小为 0，从而 `mshtml!PlainRelease` 将其内存区域被程序写到 `mshtml.dll` 中的一个可用指针

引用:
```asm
.text:3E5B54C8                         loc_3E5B54C8:                           ; CODE XREF: PlainRelease(TEAROFF_THUNK *)+48Aj
.text:3E5B54C8 56                                      push    esi            ; Value
.text:3E5B54C9 8B 35 7C 13 50 3E                        mov     esi, ds:InterlockedExchange(x,x)
.text:3E5B54CF 68 64 9D 80 3E                           push    offset lpMem    ; Target
.text:3E5B54D4 FF D6                                    call    esi ; InterlockedExchange(x,x)
```

8. 之后在 `CWindow::PrivateQueryInterface` 对 `CreateTearoffThunk` 的调用中，上面那个“可用指针”被其重用。

引用:
```asm
.text:3E5B55FD                         ; Attributes: bp-based frame
.text:3E5B55FD
.text:3E5B55FD                         public: virtual long __stdcall CWindow::PrivateQueryInterface(struct _GUID const &, void *) proc near
.text:3E5B55FD                                                                               ; DATA XREF: .text:const CWindow::`vftable'{for `CBase'}o
.text:3E5B55FD
.text:3E5B55FD                         arg_0           = dword ptr  8
.text:3E5B55FD                         arg_4           = dword ptr  0Ch
.text:3E5B55FD                         arg_8           = dword ptr  10h
.text:3E5B55FD
.text:3E5B55FD                         ; FUNCTION CHUNK AT .text:3E51AD5E SIZE 00000030 BYTES
.text:3E5B55FD                         ; FUNCTION CHUNK AT .text:3E5B6B34 SIZE 00000042 BYTES
.text:3E5B55FD                         ; FUNCTION CHUNK AT .text:3E5B6BA6 SIZE 00000025 BYTES
.text:3E5B55FD                         ; FUNCTION CHUNK AT .text:3E5B6BF0 SIZE 00000048 BYTES
.text:3E5B55FD                         ; FUNCTION CHUNK AT .text:3E5B8CBE SIZE 0000000D BYTES
.text:3E5B55FD                         ; FUNCTION CHUNK AT .text:3E5DAD16 SIZE 0000001F BYTES
.text:3E5B55FD                         ; FUNCTION CHUNK AT .text:3E5EC29C SIZE 0000001F BYTES
.text:3E5B55FD                         ; FUNCTION CHUNK AT .text:3E601117 SIZE 0000000A BYTES
.text:3E5B55FD                         ; FUNCTION CHUNK AT .text:3E62C921 SIZE 0000000A BYTES
.text:3E5B55FD                         ; FUNCTION CHUNK AT .text:3E648C4D SIZE 0000000A BYTES
.text:3E5B55FD                         ; FUNCTION CHUNK AT .text:3E667A04 SIZE 0000001F BYTES
.text:3E5B55FD                         ; FUNCTION CHUNK AT .text:3E6B6D3B SIZE 0000000D BYTES
.text:3E5B55FD
.text:3E5B55FD 8B FF                                   mov     edi, edi
.text:3E5B55FF 55                                      push    ebp
.text:3E5B5600 8B EC                                   mov     ebp, esp
.text:3E5B5602 53                                      push    ebx
.text:3E5B5603 8B 5D 10                                mov     ebx, [ebp+arg_8]
.text:3E5B5606 56                                      push    esi
.text:3E5B5607 8B 75 0C                                mov     esi, [ebp+arg_4]
.text:3E5B560A 33 D2                                   xor     edx, edx
.text:3E5B560C 89 13                                   mov     [ebx], edx
.text:3E5B560E 8B 06                                   mov     eax, [esi]
.text:3E5B5610 B9 11 04 51 30                          mov     ecx, 30510411h
.text:3E5B5615 3B C1                                   cmp     eax, ecx
.text:3E5B5617 57                                      push    edi
.text:3E5B5618 0F 87 46 01 00 00                       ja      loc_3E5B5764
.text:3E5B561E 0F 84 FD 72 07 00                       jz      loc_3E62C921
.text:3E5B5624 B9 94 F5 50 30                          mov     ecx, 3050F594h
.text:3E5B5629 3B C1                                   cmp     eax, ecx
.text:3E5B562B 0F 86 75 15 00 00                       jbe     loc_3E5B6BA6
.text:3E5B5631 2D B1 F6 50 30                          sub     eax, 3050F6B1h
.text:3E5B5636 0F 84 56 E3 0E 00                       jz      loc_3E6A3992
.text:3E5B563C 83 E8 1E                                sub     eax, 1Eh
.text:3E5B563F 0F 84 2E E3 0E 00                       jz      loc_3E6A3973
.text:3E5B5645 83 E8 0D                                sub     eax, 0Dh
.text:3E5B5648 0F 84 E2 B5 0B 00                       jz      near ptr loc_3E670C2C+4
.text:3E5B564E 2D 32 0D 00 00                          sub     eax, 0D32h
```
