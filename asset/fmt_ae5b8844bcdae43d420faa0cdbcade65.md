
::00410B0D::  68 14574200                PUSH 425714                         \->: Register  

::00410B12::  FFD6                       CALL ESI                          

::00410B14::  E8 518C0000               CALL 0041976A                      \:JMPDOWN    >>>: MFC42.DLL:?AfxGetThread@@YGPAVCWinThread@@XZ  

::00410B19::  85C0                       TEST EAX,EAX                      

::00410B1B::  74 0B                      JE SHORT 00410B28                  \:JMPDOWN  

::00410B1D::  8B10                       MOV EDX,[EAX]                    

::00410B1F::  8BC8                       MOV ECX,EAX                      

::00410B21::  FF52 7C                    CALL [EDX+7C]                   

::00410B24::  8BF0                       MOV ESI,EAX                      

::00410B26::  EB 02                      JMP SHORT 00410B2A               \:JMPDOWN  

::00410B28::  33F6                       XOR ESI,ESI                       \:BYJMP JmpBy:00410B1B,  

::00410B2A::  E8 918A0000               CALL 004195C0                     \:JMPDOWN\:BYJMP JmpBy:00410B26,    >>>: AVEDATA.DLL:?ge\_app@@YAPADXZ  

::00410B2F::  50                         PUSH EAX                          

::00410B30::  8BCE                       MOV ECX,ESI                      

::00410B32::  E8 518F0000               CALL 00419A88                     \:JMPDOWN    >>>: MFC42.DLL:?SetWindowTextA@CWnd@@QAEXPBD@Z  

::00410B37::  8D4C24 10                  LEA ECX,[ESP+10]                 

::00410B3B::  C78424 E0010000 FFFFFFFF  MOV DWORD PTR [ESP+1E0],-1       

::00410B46::  E8 B18A0000               CALL 004195FC                     \:JMPDOWN    >>>: MFC42.DLL:??1CString@@QAE@XZ  

::00410B4B::  8BCD                       MOV ECX,EBP                      

::00410B4D::  E8 6A8E0000               CALL 004199BC                     \:JMPDOWN    >>>: MFC42.DLL:?OnOK@CDialog@@MAEXXZ  

::00410B52::  8B8C24 D8010000           MOV ECX,[ESP+1D8]                 \:BYJMP JmpBy:004109ED,  

::00410B59::  5F                         POP EDI                          

::00410B5A::  5E                         POP ESI                          

::00410B5B::  5D                         POP EBP                          

::00410B5C::  5B                         POP EBX                          

::00410B5D::  64:890D 00000000          MOV FS:[0],ECX                   

::00410B64::  81C4 D4010000             ADD ESP,1D4                      

::00410B6A::  C3                         RETN                              

::00410B6A::  C3                         RETN                              

::00410B6B::  90                         NOP                              

::00410B6C::  90                         NOP                              

//这里还有 ge\_buy，也有趣的。贴出来、                 

::00410B70::  6A 05                      PUSH 5                           

::00410B72::  6A 00                      PUSH 0                           

::00410B74::  E8 538A0000               CALL 004195CC                     \:JMPDOWN    >>>: AVEDATA.DLL:?ge\_buy@@YAPADH@Z  

::00410B79::  83C4 04                    ADD ESP,4                      

::00410B7C::  50                         PUSH EAX                          

::00410B7D::  E8 8E9EFFFF               CALL 0040AA10                    \:JMPUP  

::00410B82::  83C4 08                    ADD ESP,8                      

::00410B85::  C3                         RETN                              

::00410B86::  90                         NOP                              

::00410B87::  90                         NOP                              

### code 2 end

### code 3 begin

////  OD 中, aveData.dll 的 ge\_check 函数，这段是得到正确的关键地  ////  

```
00B71E36     2BC2                     sub eax,edx
00B71E38     8D9424 C8010000          lea edx,dword ptr ss:[esp+1C8]
00B71E3F     03C6                     add eax,esi
00B71E41     03C7                     add eax,edi
00B71E43     50                       push eax
00B71E44     68 8CE4B700              push aveData.00B7E48C                  ; ASCII "%08lX"
00B71E49     52                       push edx
00B71E4A     E8 59240000              call aveData.00B742A8
00B71E4F     8B8C24 E8020000          mov ecx,dword ptr ss:[esp+2E8]
00B71E56     8D8424 D4010000          lea eax,dword ptr ss:[esp+1D4]
00B71E5D     6A 08                    push 8
00B71E5F     50                       push eax
00B71E60     51                       push ecx
```

//看 od 寄存器窗口，很可能 EAX 是真的 key 了。果然是的。

```
EAX 0012A12C ASCII "C9575F31"   

ECX 00379990 ASCII "77498864"   

EDX 0012A133

EBX 40C06277
```

```
00B71E61     E8 0A240000              call aveData.00B74270
00B71E66     83C4 18                   add esp,18
00B71E69     85C0                      test eax,eax
00B71E6B     5F                        pop edi
00B71E6C     5E                        pop esi
00B71E6D     5D                        pop ebp
00B71E6E     5B                        pop ebx
00B71E6F     0F85 83000000            jnz aveData.00B71EF8
```

//往后面看看知道，需要在这里跳转，否则会清掉正确的 key。

//在这里做一下内存 patch,跳往我们想要的地方，这样 return 后就得到 key 了。

```
//0F85 83000000 -> 90e9 7c000000 

//(NOP, JMP aveData.00B71EF1)
```

...省略若干行...

```
00B71EC5     E8 A6100000               call    aveData.00B72F70
00B71ECA     8D4C24 00                 lea     ecx,dword ptr ss:[esp]
00B71ECE     C78424 C0020000 FFFFFFFF  mov     dword ptr ss:[esp+2C0],-1
00B71ED9     E8 92100000               call    aveData.00B72F70
00B71EDE     B8 01000000               mov     eax,1
00B71EE3     8B8C24 B8020000           mov     ecx,dword ptr ss:[esp+2B8]
00B71EEA     64:890D 00000000          mov     dword ptr fs:[0],ecx
```

//修改，让他跳到这里 00B71EF1

```
00B71EF1     81C4 C4020000             add     esp,2C4
00B71EF7     C3                        retn
```

//  真的 key 位于 ESP-0x2E4;

//  用 MOV  ECX, dword ptr SS:\[ESP-0x2E4\] 来保存即可。

//本来是跳到 00B71EF8

```
00B71EF8     8D4C24 30                 lea     ecx,dword ptr ss:[esp+30]
00B71EFC     C68424 C0020000 0C        mov     byte ptr ss:[esp+2C0],0C
```

...省略若干行...

```
00B71F61     8B8C24 B8020000           mov     ecx,dword ptr ss:[esp+2B8]
00B71F68     33C0                      xor     eax,eax
00B71F6A     64:890D 00000000          mov     dword ptr fs:[0],ecx
00B71F71     81C4 C4020000             add     esp,2C4
00B71F77     C3                        retn
```

### code 3 end

### EndOfFile  readyu 20061003 

`keygen` 已经好些人用过了，不是纸上谈兵的。:)  

这文章是很久之后抽空写的，把以前的编译时的代码找出来的。  

写的时候，`keygen` 重新编译，测试通过。算了不少的 key。  

这段地址是可写的。它恰好是壳解压后的代码段，没有设置只读保护。  

另外 `memcpy` 是 `Kernel32.dll` 中的 `RtlMoveMemory` 这个 API 的封装，  

与 `WriteProcessMemory` 不同。  

`RtlMoveMemory` 不会检查读写该内存所应有的权限，  

如果不可写，会立刻 XX 内存不能为 written 崩掉的。也能立刻发现。  

引用:  *最初由** cutezhang **发布*  

有疑问:`keygen` 的代码测试通过了吗?DLL 内存代码段怎么随便就修改了,  

似乎还要先设置一下 dll 模块可写
