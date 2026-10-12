
## 汇编代码分析与修改

```
  00433960     3803             cmp byte ptr ds:[ebx],al

  00433962   ^ 75 F9            jnz short NgaMy.0043395D

  00433964     61               popad

  00433965     0BC0             or eax,eax

  00433967   ^ 0F84 15FFFFFF    je NgaMy.00433882

  0043396D     3B85 9CE24100    cmp eax,dword ptr ss:[ebp+41E29C]     ; 比较是否是 MessageBoxA

  00433973     74 20            je short NgaMy.00433995              ; nop 掉

  00433975     90               nop

  00433976     90               nop

  00433977     90               nop

  00433978     90               nop

  00433979     3B85 9D014100    cmp eax,dword ptr ss:[ebp+41019D]     ; 比较是否是 RegisterHotKey

  0043397F     74 09            je short NgaMy.0043398A              ; nop 掉

  00433981     90               nop

  00433982     90               nop

  00433983     90               nop

  00433984     90               nop

  00433985     EB 14            jmp short NgaMy.0043399B

  00433987     90               nop

  00433988     90               nop

  00433989     90               nop

  0043398A     8D85 0A024100    lea eax,dword ptr ss:[ebp+41020A]

  00433990     EB 09            jmp short NgaMy.0043399B

  00433992     90               nop

  00433993     90               nop

  00433994     90               nop

  00433995     8D85 24024100    lea eax,dword ptr ss:[ebp+410224]

  0043399B     56               push esi

  0043399C     FFB5 17FC4000    push dword ptr ss:[ebp+40FC17]

  004339A2     5E               pop esi

  004339A3     39B5 FA234000    cmp dword ptr ss:[ebp+4023FA],esi

  004339A9     74 15            je short NgaMy.004339C0

  004339AB     90               nop

  004339AC     90               nop

  004339AD     90               nop

  004339AE     90               nop

  004339AF     39B5 FE234000    cmp dword ptr ss:[ebp+4023FE],esi

  004339B5     74 09            je short NgaMy.004339C0

  004339B7     90               nop

  004339B8     90               nop

  004339B9     90               nop

  004339BA     90               nop

  004339BB     EB 63            jmp short NgaMy.00433A20

  004339BD     90               nop

  004339BE     90               nop

  004339BF     90               nop

  004339C0     80BD D2594100 00 cmp byte ptr ss:[ebp+4159D2],0

  004339C7     74 57            je short NgaMy.00433A20             ; Magic Jump，改 Jmp

  004339C9     90               nop

  004339CA     90               nop

  004339CB     90               nop

  004339CC     90               nop

  004339CD     EB 07            jmp short NgaMy.004339D6

  004339CF     90               nop

  004339D0     90               nop

  004339D1     90               nop

  004339D2     0100             add dword ptr ds:[eax],eax

  004339D4     0000             add byte ptr ds:[eax],al

  004339D6     8BB5 E4FC4000    mov esi,dword ptr ss:[ebp+40FCE4]

  004339DC     83C6 0D          add esi,0D

  004339DF     81EE EA1B4000    sub esi,NgaMy.00401BEA

  004339E5     2BF5             sub esi,ebp

  004339E7     83FE 00          cmp esi,0

  004339EA     7F 34            jg short NgaMy.00433A20

  004339EC     90               nop

  004339ED     90               nop
```

作如上修改后，就可以用`ImportREC`来修复`IAT`了，到`00401000`段，下内存访问断点

### 第 1 次 Shift+F9

```
引用: 00403D38     68 8C3D4000       push NgaMy.00403D8C               ; 停在这
  00403D3D     64:A1 00000000     mov eax,dword ptr fs:[0]
  00403D43     50                 push eax
  00403D44     8B4424 10          mov eax,dword ptr ss:[esp+10]
  00403D48     896C24 10          mov dword ptr ss:[esp+10],ebp
  00403D4C     8D6C24 10          lea ebp,dword ptr ss:[esp+10]
  00403D50     2BE0               sub esp,eax
  00403D52     53                 push ebx
  00403D53     56                 push esi
  00403D54     57                 push edi
  00403D55     8B45 F8            mov eax,dword ptr ss:[ebp-8]
  00403D58     8965 E8            mov dword ptr ss:[ebp-18],esp
  00403D5B     50                 push eax
  00403D5C     8B45 FC            mov eax,dword ptr ss:[ebp-4]
  00403D5F     C745 FC FFFFFFFF   mov dword ptr ss:[ebp-4],-1
  00403D66     8945 F8            mov dword ptr ss:[ebp-8],eax
  00403D69     8D45 F0            lea eax,dword ptr ss:[ebp-10]
  00403D6C     64:A3 00000000     mov dword ptr fs:[0],eax
  00403D72     C3                 retn                                ; F4，F8
```

### 第 2 次 Shift+F9

```
引用: 00405560     3D 00100000      cmp eax,1000                       ; 停在这
  00405565     73 0E              jnb short NgaMy.00405575
  00405567     F7D8               neg eax
  00405569     03C4               add eax,esp
  0040556B     83C0 04            add eax,4
  0040556E     8500               test dword ptr ds:[eax],eax
  00405570     94                 xchg eax,esp
  00405571     8B00               mov eax,dword ptr ds:[eax]
  00405573     50                 push eax
  00405574     C3                 retn                                ; F4，F8
```

### 第 3 次 Shift+F9，到达 FOEP

```
引用:   0040305C     83F9 02          cmp ecx,2                          ; FOEP，上面被抽了不少字节
  0040305F     74 0C              je short NgaMy.0040306D
  00403061     81CE 00800000      or esi,8000
  00403067     8935 B0DE4000      mov dword ptr ds:[40DEB0],esi
  0040306D     C1E0 08            shl eax,8
```

到此，脱壳吧，`ImportREC`修复，`OEP`填`305C`，全部有效，`Fix Dump`。

现在脱壳修复后的文件当然是运行不了的，因为`OEP`没修复嘛：）

### 三、Find Stolen Code Binary

`Ctrl+F12`，重新再入

```
引用:   0041F000 N>  60               pushad                             ; EP，F8
  0041F001     E8 01000000        call NgaMy.0041F007                  ; ESP=0012FFA4,hr 0012FFA4,Shift+F9
```

几次`Shift+F9`后来到这里

```
引用:   004365F4     8915 F5FD4100    mov dword ptr ds:[41FDF5],edx       ; 几次 Shift+F9 后来到这里，删除硬件断点
  004365FA     FF35 F5FD4100    push dword ptr ds:[41FDF5]              ; 这里就是 Stolen Code 开始的地方了
  00436600     8F05 2DFE4100    pop dword ptr ds:[41FE2D]              ; ntdll.KiFastSystemCallRet
  00436606     FF35 2DFE4100    push dword ptr ds:[41FE2D]             ; ntdll.KiFastSystemCallRet
  0043660C     C70424 60000000  mov dword ptr ss:[esp],60
  00436613     56                 push esi
  00436614     890C24             mov dword ptr ss:[esp],ecx
  00436617     68 8DFD4100        push NgaMy.0041FD8D
  0043661C     59                 pop ecx
  0043661D     8919               mov dword ptr ds:[ecx],ebx
  0043661F     8B0C24             mov ecx,dword ptr ss:[esp]
  00436622     8F05 ADFE4100     pop dword ptr ds:[41FEAD]
  00436628     FF35 8DFD4100     push dword ptr ds:[41FD8D]
  0043662E     C70424 48A24000   mov dword ptr ss:[esp],NgaMy.0040A248
  00436635     8905 B9FD4100     mov dword ptr ds:[41FDB9],eax
  0043663B     FF35 B9FD4100     push dword ptr ds:[41FDB9]
  00436641     90                 nop
  00436642     90                 nop
  00436643     60                 pushad
  00436644     E8 01000000        call NgaMy.0043664A                  ; F4，ESP=0012FF98，hr esp，F9
```

二进制复制`pushad`上面的代码\[nop可以不复制\]
