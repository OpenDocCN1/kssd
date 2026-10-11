
```asm
01262284    55               push    ebp
01262285    8BEC             mov     ebp, esp
01262287    83C4 D8          add     esp, -28
0126228A    53               push    ebx
0126228B    56               push    esi
0126228C    57               push    edi
0126228D    894D F8          mov     dword ptr [ebp-8], ecx
01262290    8955 F0          mov     dword ptr [ebp-10], edx
01262293    8945 F4          mov     dword ptr [ebp-C], eax
01262296    8B75 18          mov     esi, dword ptr [ebp+18]            ; 当前 API 所用的数据组指针
01262299    8B45 F4          mov     eax, dword ptr [ebp-C]            ; 基本数据
0126229C    8D58 40          lea     ebx, dword ptr [eax+40]           ; 基本数据+40,随机函数 index
0126229F    EB 01            jmp     short 012622A2
012622A1    90               nop
012622A2    A1 54672601      mov     eax, dword ptr [1266754]
012622A7    C600 DD          mov     byte ptr [eax], 0DD
012622AA    33C0             xor     eax, eax
012622AC    8A43 07          mov     al, byte ptr [ebx+7]              ; index7
012622AF    8D0440           lea     eax, dword ptr [eax+eax*2]
012622B2    8B55 F4          mov     edx, dword ptr [ebp-C]
012622B5    8B7C82 68        mov     edi, dword ptr [edx+eax*4+68]
012622B9    8B06             mov     eax, dword ptr [esi]
012622BB    FFD7             call    edi                                ; mov eax,[eax+9]
012622BD    8945 EC          mov     dword ptr [ebp-14], eax           ; 保存
012622C0    33C0             xor     eax, eax
012622C2    8A43 08          mov     al, byte ptr [ebx+8]              ; index8
012622C5    8D0440           lea     eax, dword ptr [eax+eax*2]
012622C8    8B55 F4          mov     edx, dword ptr [ebp-C]
012622CB    8B7C82 68        mov     edi, dword ptr [edx+eax*4+68]
012622CF    8B06             mov     eax, dword ptr [esi]
012622D1    FFD7             call    edi                                ; mov eax,[eax+11]
012622D3    8945 E8          mov     dword ptr [ebp-18], eax           ; 保存
012622D6    EB 01            jmp     short 012622D9
012622D8    90               nop
012622D9    33C0             xor     eax, eax
012622DB    8A43 02          mov     al, byte ptr [ebx+2]              ; index2
012622DE    8D0440           lea     eax, dword ptr [eax+eax*2]
012622E1    8B55 F4          mov     edx, dword ptr [ebp-C]
012622E4    8B7C82 68        mov     edi, dword ptr [edx+eax*4+68]
012622E8    8B06             mov     eax, dword ptr [esi]
012622EA    FFD7             call    edi                                ; mov eax,[eax+2C]
012622EC    8BF8             mov     edi, eax
012622EE    33C0             xor     eax, eax
012622F0    8A43 06          mov     al, byte ptr [ebx+6]              ; index6
012622F3    8D0440           lea     eax, dword ptr [eax+eax*2]
012622F6    8B55 F4          mov     edx, dword ptr [ebp-C]
012622F9    8B5482 68        mov     edx, dword ptr [edx+eax*4+68]
012622FD    8B06             mov     eax, dword ptr [esi]
012622FF    FFD2             call    edx                                ; mov eax,byte ptr [eax]
01262301    8845 DF          mov     byte ptr [ebp-21], al             ; 保存
01262304    8B45 F4          mov     eax, dword ptr [ebp-C]            ; 基本数据
01262307    03B8 E0000000    add     edi, dword ptr [eax+E0]           ; 计算代码 hash 的偏移：[+2C]+offset
0126230D    EB 01            jmp     short 01262310
0126230F    90               nop
01262310    57               push    edi
01262311    33C0             xor     eax, eax
01262313    8A45 DF          mov     al, byte ptr [ebp-21]             ; [+0]
01262316    05 FF000000      add     eax, 0FF                          ; [+0]+0xFF
0126231B    50               push    eax                                ; 计算代码 hash 的长度：[+0]+0xFF
0126231C    8B45 F4          mov     eax, dword ptr [ebp-C]
0126231F    E8 4CFCFFFF      call    01261F70                           ; 计算代码 hash 的基址：fixaddr=126099C
01262324    8BC8             mov     ecx, eax
01262326    8B45 F4          mov     eax, dword ptr [ebp-C]
01262329    8B50 10          mov     edx, dword ptr [eax+10]           ; 计算代码段 HASH 时使用的代码起止位置数据表指针
0126232C    8B45 F4          mov     eax, dword ptr [ebp-C]            ; 基本数据
0126232F    E8 70EAFFFF      call    01260DA4                           ; hash(fixaddr+[+2C]+offset,[+0]+0xFF)
01262334    8945 D8          mov     dword ptr [ebp-28], eax           ; 保存 hash 结果
01262337    EB 01            jmp     short 0126233A
01262339    90               nop
0126233A    33C0             xor     eax, eax
0126233C    8A43 03          mov     al, byte ptr [ebx+3]              ; index3
0126233F    8D0440           lea     eax, dword ptr [eax+eax*2]
01262342    8B55 F4          mov     edx, dword ptr [ebp-C]
01262345    8B7C82 68        mov     edi, dword ptr [edx+eax*4+68]
01262349    8B06             mov     eax, dword ptr [esi]
0126234B    FFD7             call    edi                                ; mov eax,[eax+20]
0126234D    8BF8             mov     edi, eax
0126234F    8B45 F4          mov     eax, dword ptr [ebp-C]
01262352    03B8 E0000000    add     edi, dword ptr [eax+E0]            ; [+20]+offset
01262358    8B45 EC          mov     eax, dword ptr [ebp-14]           ; [+9]
0126235B    03C7             add     eax, edi                          ; [+20]+offset+[+9]
0126235D    0345 D8          add     eax, dword ptr [ebp-28]           ; [+20]+offset+[+9]+hash
01262360    8945 EC          mov     dword ptr [ebp-14], eax           ; API 所用的 DLL 名称 HASH：[+20]+offset+[+9]+hash
01262363    8B45 E8          mov     eax, dword ptr [ebp-18]           ; [+11]
01262366    2BC7             sub     eax, edi                          ; [+11]-( [+20]+offset)
01262368    2B45 D8          sub     eax, dword ptr [ebp-28]           ; [+11]-( [+20]+offset)-hash
0126236B    8945 E8          mov     dword ptr [ebp-18], eax           ; API 名称字符串 HASH：[+11]-( [+20]+offset)-hash
0126236E    33C0             xor     eax, eax
01262370    8A43 01          mov     al, byte ptr [ebx+1]              ; index1
01262373    8D0440           lea     eax, dword ptr [eax+eax*2]
01262376    8B55 F4          mov     edx, dword ptr [ebp-C]
01262379    8B5482 68        mov     edx, dword ptr [edx+eax*4+68]
0126237D    8B06             mov     eax, dword ptr [esi]
0126237F    FFD2             call    edx                                ; mov eax,byte ptr [eax+26]
01262381    8BD8             mov     ebx, eax
01262383    EB 01            jmp     short 01262386
01262385    90               nop
01262386    8D45 E4          lea     eax, dword ptr [ebp-1C]
01262389    50               push    eax                                ; 用于保存计算结果的地址指针
0126238A    66:8B4D EC       mov     cx, word ptr [ebp-14]             ; API 所用的 DLL 名称 HASH
0126238E    66:8B55 E8       mov     dx, word ptr [ebp-18]             ; API 名称字符串 HASH
01262392    8B45 F4          mov     eax, dword ptr [ebp-C]            ; 基本数据
01262395    E8 02F7FFFF      call    01261A9C                           ; 查找 API 相关 HASH 并获取 API 地址
0126239A    84C0             test    al, al
0126239C    8B45 F4          mov     eax, dword ptr [ebp-C]
0126239F    8B80 E0000000    mov     eax, dword ptr [eax+E0]           ; offset
012623A5    0345 E4          add     eax, dword ptr [ebp-1C]           ; 计算结果+offset:API 实际地址
012623A8    8945 FC          mov     dword ptr [ebp-4], eax           ; !!!!!!!!!!!!!API ADDRESS!!
```

到这里终于出现了所调用 API 的实际地址，再后面的代码就是把 API 地址偏移保存到一个表中，更改 `CALL 1410000` 为 `CALL 1460004`，避免以后重复获取 API 地址；恢复堆栈和现场，跳到 API 入口继续执行。

重点分析一下这一段代码：

1. 根据数据组中 `dword:[+2C]` 和 `byte:[+0]` 处的值计算壳代码的 HASH 值：`hash=hash(fixaddr+[+2C]+offset,[+0]+0xFF)`

2. 计算出 API 所用的 DLL 名称 HASH：`[+20]+offset+[+9]+hash`

3. 计算出 API 名称字符串 HASH：`[+11]-( [+20]+offset)-hash`

4. 根据 DLL 名称 HASH 和 API 名称 HASH 查找导入表字符串数据组，并获取 API 地址

其中加方括号的为使用随机函数的功能，共使用到了 6 个随机函数 index：

`7:mov eax,[eax+9]`

`8:mov eax,[eax+11]`

`2:mov eax,[eax+2C]`

`6:mov eax,byte ptr [eax]`

`3:mov eax,[eax+20]`
