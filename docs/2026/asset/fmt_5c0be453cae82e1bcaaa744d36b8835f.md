
`seg000:004D524A` `sub_4D524A` proc near ; CODE XREF: 
`sub_4D51AA+31p` 
`seg000:004D524A` 
`seg000:004D524A` `arg_0` = dword ptr 8 
;`arg_0`: 缓冲区基址，也可能指向具体变量 
`seg000:004D524A` `arg_4` = dword ptr 0Ch 
;`arg_4`: 数据段基址 
`seg000:004D524A` `arg_8` = dword ptr 10h 
;`arg_8`: 字符串表/某特定字符串 基址

```
seg000:004D524A 
push ebp 
seg000:004D524B mov ebp, esp 
seg000:004D524D push ebx 
seg000:004D524E push esi 
seg000:004D524F push edi 
seg000:004D5250 mov eax, [ebp+arg_4] 
;eax = arg_4 
seg000:004D5253 mov ebx, [ebp+arg_0] 
;ebx = arg_0 
seg000:004D5256 mov edi, [ebp+arg_8] 
;edi = arg_8 
seg000:004D5259 mov edx, [eax+20h] 
seg000:004D525C mov ecx, [edx] 
seg000:004D525E mov [ebx], ecx 
;[ebx] =[ [eax+20h]] 
seg000:004D5260 mov eax, [ebp+arg_4] 
;eax = arg_4 
seg000:004D5263 mov edx, [eax+20h] 
seg000:004D5266 add edx, 4 
seg000:004D5269 mov ecx, [edx] 
seg000:004D526B mov [ebx+4], ecx 
;[ebx+4] =[ [eax+20h]+4]
;由于出现了ebx + 4的字样，可见，arg_0是被当作缓冲区基址来使用的。现在可以确定arg_0 
;为缓冲区基址，而不是某特定变量地址。
 ;类似地，[eax+20h]极可能是一个缓冲区地址，而eax为arg_4,数据段基址。
seg000:004D526E push edi 
;edi为arg_8,字符串表/字符串 基址 
;参数0压栈 
seg000:004D526F call dword ptr [ebx] 
;这个call不知道调用的是什么，须用调试器了。结果证明是调用得LoadLibraryA,参数 
;为”Kernel32.dll”。同时，也可以确定 [arg_4 + 20h] + 0 的地方保存的是LoadLibraryA 的地址, 
;arg_0 + 0 的地方保存LoadLibraryA地址。 
seg000:004D5271 mov esi, eax 
;esi 为kernel32.dll 的基址 
seg000:004D5273 test esi, esi 
seg000:004D5275 jz loc_4D5307 
;如果kernel32.dll加载失败，函数直接返回0 
seg000:004D527B mov eax, [ebp+arg_4] 
seg000:004D527E push dword ptr [eax+24h] 
seg000:004D5281 push esi 
;esi为kernel32.dll模块基址 
seg000:004D5282 call dword ptr [ebx+4] 
; 这里又得用调试器了，发现调用的是GetProcAddress，参数为“GetTempPathW” 
;可见，[arg_4 + 20h] +4 位置保存的是GetProcAddress 的地址 
;arg_0 + 4 的地方保存GetProcAddress的地址
;arg_4 + 24h 处保存指向“GetTempPathW”字符串的指针 
seg000:004D5285 mov edx, [ebp+arg_4] 
seg000:004D5288 mov ecx, [edx+20h] 
seg000:004D528B mov [ecx], eax 
;[[arg_4+20h]+0] = GetProcAddress(“GetTempPathW”) 
;[arg_4 + 20h] +0 的位置原来保存的是LoadLibraryA的地址，现在被替换为GetTempPathW的 
;地址 
seg000:004D528D mov eax, [ebp+arg_4] 
seg000:004D5290 push dword ptr [eax+28h] seg000:004D5293 push esi 
;esi为kernel32.dll的基址 seg000:004D5294 call dword ptr [ebx+4] 
;调用GetProcAddress，参数为kernel32的基址和字符串“GetLastError”，其中“GetLastErr ;or”来自于调试结果。 
seg000:004D5297 mov edx, [ebp+arg_4] 
seg000:004D529A mov ecx, [edx+20h] 
seg000:004D529D add ecx, 4 
seg000:004D52A0 mov [ecx], eax 
;[[arg_4+20h]+4] = GetProcAddress(“GetLastError”) 
;[arg_4 + 20h] +4 的位置原来保存的是GetProcAddress的地址，现在被替换为GetLastErro ;r的地址 
seg000:004D52A2 lea eax, [edi+0Dh]
;edi是arg_8，字符串/字符串表 基址，值为004D50AC。而0Dh正好为字符串“kernel32.dll” 
;（包括结尾的0）的长度。所以这里edi+0dh是在寻址下一个字符串“GetTempPathA”
;可见，edi被当作字符串表基址去处理的。也就是说，arg_8是字符串表基址，而不是用于 
;某个特定字符串的基址。 
seg000:004D52A5 push eax 
;eax指向字符串“GetTempPathA” 
seg000:004D52A6 push esi 
;esi为kernel32.dll的基址 
seg000:004D52A7 call dword ptr [ebx+4] 
;调用GetProcAddress 
seg000:004D52AA mov [ebx+8], eax 
;[arg_0 + 8 ] = GetProcAddress(“GetTempPathA”) 
;arg_0 + 8 的地方保存GetTempPathA的地址 
seg000:004D52AD lea edx, [edi+1Ah] 
seg000:004D52B0 push edx 
seg000:004D52B1 push esi 
seg000:004D52B2 call dword ptr [ebx+4] 
seg000:004D52B5 mov [ebx+0Ch], eax 
;arg_0 + 0ch 的地方保存GetTempFileNameA的地址 
seg000:004D52B8 lea ecx, [edi+2Bh] 
seg000:004D52BB push ecx 
seg000:004D52BC push esi 
seg000:004D52BD call dword ptr [ebx+4] 
seg000:004D52C0 mov [ebx+10h], eax 
;arg_0 + 10h 的地方保存CreateFileA的地址 
seg000:004D52C3 lea eax, [edi+37h] 
seg000:004D52C6 push eax 
seg000:004D52C7 push esi 
seg000:004D52C8 call dword ptr [ebx+4] 
seg000:004D52CB mov [ebx+14h], eax 
;arg_0 + 14h 的地方保存ReadFile的地址
seg000:004D52CE lea edx, [edi+40h] 
seg000:004D52D1 push edx 
seg000:004D52D2 push esi 
seg000:004D52D3 call dword ptr [ebx+4] 
seg000:004D52D6 mov [ebx+18h], eax 
;arg_0 + 18h 的地方保存WriteFile的地址 
seg000:004D52D9 lea ecx, [edi+4Ah] 
seg000:004D52DC push ecx 
seg000:004D52DD push esi 
seg000:004D52DE call dword ptr [ebx+4] 
seg000:004D52E1 mov [ebx+1Ch], eax 
;arg_0 + 1ch 的地方保存SetFilePointer的地址 
seg000:004D52E4 lea eax, [edi+59h] 
seg000:004D52E7 push eax 
seg000:004D52E8 push esi
seg000:004D52E9 call dword ptr [ebx+4] 
seg000:004D52EC mov [ebx+20h], eax 
;arg_0 + 20h 的地方保存CloseHandle的地址 
seg000:004D52EF lea edx, [edi+65h] 
seg000:004D52F2 push edx 
seg000:004D52F3 push esi 
seg000:004D52F4 call dword ptr [ebx+4] 
seg000:004D52F7 mov [ebx+24h], eax 
;arg_0 + 24h 的地方保存GetTickCount的地址 
seg000:004D52FA lea ecx, [edi+72h] 
seg000:004D52FD push ecx 
seg000:004D52FE push esi 
seg000:004D52FF call dword ptr [ebx+4] 
seg000:004D5302 mov [ebx+28h], eax 
;arg_0 + 28h 的地方保存GetModuleFileNameA的地址 
seg000:004D5305 jmp short loc_4D530B 
;去4D530B继续 
seg000:004D5307 ; --------------------------------------------------------------------------- seg000:004D5307 
seg000:004D5307 loc_4D5307: ; CODE XREF: 
sub_4D524A+2Bj 
seg000:004D5307 xor eax, eax 
seg000:004D5309 jmp short loc_4D534C s
eg000:004D530B ; ---------------------------------------------------------------------------
seg000:004D530B 
seg000:004D530B loc_4D530B: ; CODE XREF: 
sub_4D524A+BBj 
seg000:004D530B lea edx, [edi+85h] seg000:004D5311 push edx seg000:004D5312 call dword ptr [ebx] 
seg000:004D5314 mov esi, eax 
;esi = LoadLibraryA(“ADVAPI32.dll”); 
;esi仍然为模块基址 
seg000:004D5316 test esi, esi 
seg000:004D5318 jz short loc_4D5346 
;模块加载失败，函数直接返回0 
seg000:004D531A lea eax, [edi+92h] 
seg000:004D5320 push eax 
seg000:004D5321 push esi 
seg000:004D5322 call dword ptr [ebx+4] 
seg000:004D5325 mov [ebx+2Ch], eax 
;arg_0 + 2ch 的地方保存RegOpenKeyExA的地址 
seg000:004D5328 lea edx, [edi+0A0h] 
seg000:004D532E push edx
seg000:004D532F push esi 
seg000:004D5330 call dword ptr [ebx+4] 
seg000:004D5333 mov [ebx+30h], eax 
;arg_0 + 30h 的地方保存RegQueryValueExA的地址 
seg000:004D5336 add edi, 0B1h 
;直接调整edi指向字符串“RegCloseKey” 
seg000:004D533C push edi 
seg000:004D533D push esi seg000:004D533E call dword ptr [ebx+4] seg000:004D5341 mov [ebx+34h], eax 
;arg_0 + 34h 的地方保存RegCloseKey的地址 
seg000:004D5344 jmp short loc_4D534A 
;处理成功完成，函数返回1。 
seg000:004D5346 ; --------------------------------------------------------------------------- seg000:004D5346 
seg000:004D5346 loc_4D5346: ; CODE XREF: 
sub_4D524A+CEj 
seg000:004D5346 xor eax, eax 
seg000:004D5348 jmp short loc_4D534C 
seg000:004D534A ; --------------------------------------------------------------------------- seg000:004D534A 
seg000:004D534A loc_4D534A: ; CODE XREF: 
sub_4D524A+FAj 
seg000:004D534A mov al, 1 
seg000:004D534C 
seg000:004D534C loc_4D534C: ; CODE XREF: 
sub_4D524A+BFj 
seg000:004D534C ; sub_4D524A+FEj
seg000:004D534C pop edi 
seg000:004D534D pop esi 
seg000:004D534E pop ebx 
seg000:004D534F pop ebp 
seg000:004D5350 retn 0Ch 
seg000:004D5350 sub_4D524A endp
```

到此，函数 `sub_4D524A` 的分析就完成了。很明显，该函数的主要作用是导入病毒需要的一些 API 函数入口地址。在 idb 文件中，笔者将该函数重命名为 `ImportMyFunc`。现在依据分析过程中红色的文字，我们还可以比较准确地确定调用此函数时传入的三个参数的意义。

`esi`：字符串表基址

`[edi+20h]`:一个指向函数地址表的指针，该表的前二个元素分别为 `LoadLibraryA` 和
`GetProcAddress` 的地址。过程中分别被替换为 `GetTempPathW` 和 `GetLastError`
的地址

`[edi+24h]`:指向字符串“`GetTempPathW`”

`[edi+28h]`:指向字符串“`GetLastError`”

`ecx`：缓冲区基址（指向 `v_38`），具体被用作函数地址表基址（每个元素大小为双字）

### 分析函数 `sub_004D51AA`

```
seg000:004D51E0 test al, al 
seg000:004D51E2 jz short loc_4D5242 
;如果sub_4D524A 的返回值为 0（失败），则退出病毒体，控制传递给OEP
```

### 分析函数 `sub_004D5356`(调用部分)

```
seg000:004D51E4 push esi 
;esi为字符串表指针 ;参数2压栈 
seg000:004D51E5 lea eax, [ebp+var_38] 
seg000:004D51E8 push eax 
;eax 为函数地址表指针 
;参数1压栈 
seg000:004D51E9 lea edx, [ebp+var_140] 
seg000:004D51EF push edx 
;edx 指向v_140,由于v_140还未被初始化，猜测v_140应该是一个接收数据的变量或者 
;或者缓冲区，这要等到sub_4D5356的函数体分析完成后才能确定 
;参数0压栈 
seg000:004D51F0 call sub_4D5356
```

### 分析函数 `sub_004D5356`(实现部分)
