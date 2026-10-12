
## 汇编代码分析

====> `[0048BDAB]`=76 比较第 3 位！

```
:00486412 7412                  je 00486426
```
====> 不跳则 OVER！

```
:00486414 803D60BD480000        cmp byte ptr [0048BD60], 00
:0048641B 33C0                  xor eax, eax
:0048641D EB02                  jmp 00486421
:0048641F B001                  mov al, 01
```

Referenced by a (U)nconditional or (C)onditional Jump at Address: `:0048641D(U)`

```
:00486421 A260BD4800            mov byte ptr [0048BD60], al
```

Referenced by a (U)nconditional or (C)onditional Jump at Address: `:00486412(C)`

```
:00486426 A08ABD4800            mov al, byte ptr [0048BD8A]
```
====> `[0048BD8A]`=62

```
:0048642B 3A05CBBD4800          cmp al, byte ptr [0048BDCB]
```
====> `[0048BDCB]`=18 比较第 5 位！

```
:00486431 7412                  je 00486445
```
====> 不跳则 OVER！

```
:00486433 803D60BD480000        cmp byte ptr [0048BD60], 00
:0048643A 33C0                  xor eax, eax
:0048643C EB02                  jmp 00486440
:0048643E B001                  mov al, 01
```

Referenced by a (U)nconditional or (C)onditional Jump at Address: `:0048643C(U)`

```
:00486440 A260BD4800            mov byte ptr [0048BD60], al
```

Referenced by a (U)nconditional or (C)onditional Jump at Address: `:00486431(C)`

```
:00486445 A081BD4800            mov al, byte ptr [0048BD81]
```
====> `[0048BD81]`=13

```
:0048644A 3A059BBD4800          cmp al, byte ptr [0048BD9B]
```
====> `[0048BD9B]`=73 比较第 2 位！

```
:00486450 7412                  je 00486464
```
====> 不跳则 OVER！

```
:00486452 803D60BD480000        cmp byte ptr [0048BD60], 00
:00486459 33C0                  xor eax, eax
:0048645B EB02                  jmp 0048645F
:0048645D B001                  mov al, 01
```

Referenced by a (U)nconditional or (C)onditional Jump at Address: `:0048645B(U)`

```
:0048645F A260BD4800            mov byte ptr [0048BD60], al
```

Referenced by a (U)nconditional or (C)onditional Jump at Address: `:00486450(C)`

```
:00486464 A087BD4800            mov al, byte ptr [0048BD87]
```
====> `[0048BD87]`=77

```
:00486469 3A05BBBD4800          cmp al, byte ptr [0048BDBB]
```
====> `[0048BDBB]`=09 比较第 4 位！

```
:0048646F 7412                  je 00486483
```
====> 不跳则 OVER！

```
:00486471 803D60BD480000        cmp byte ptr [0048BD60], 00
:00486478 33C0                  xor eax, eax
:0048647A EB02                  jmp 0048647E
:0048647C B001                  mov al, 01
```

Referenced by a (U)nconditional or (C)onditional Jump at Address: `:0048647A(U)`

```
:0048647E A260BD4800            mov byte ptr [0048BD60], al
```

Referenced by a (U)nconditional or (C)onditional Jump at Address: `:0048646F(C)`

```
:00486483 A07EBD4800            mov al, byte ptr [0048BD7E]
```
====> `[0048BD7E]`=21

```
:00486488 3A058BBD4800          cmp al, byte ptr [0048BD8B]
```
====> `[0048BD8B]`=48 比较第 1 位！

```
:0048648E 7412                  je 004864A2
```
====> 不跳则 OVER！

```
:00486490 803D60BD480000        cmp byte ptr [0048BD60], 00
:00486497 33C0                  xor eax, eax
:00486499 EB02                  jmp 0048649D
:0048649B B001                  mov al, 01
```

Referenced by a (U)nconditional or (C)onditional Jump at Address: `:00486499(U)`

```
:0048649D A260BD4800            mov byte ptr [0048BD60], al
```

Referenced by a (U)nconditional or (C)onditional Jump at Address: `:0048648E(C)`

```
:004864A2 A08DBD4800            mov al, byte ptr [0048BD8D]
```
====> `[0048BD8D]`=54

```
:004864A7 3A05DBBD4800          cmp al, byte ptr [0048BDDB]
```
====> `[0048BDDB]`=26 比较第 6 位！

```
:004864AD 7412                  je 004864C1
```
====> 不跳则 OVER！

```
:004864AF 803D60BD480000        cmp byte ptr [0048BD60], 00
:004864B6 33C0                  xor eax, eax
:004864B8 EB02                  jmp 004864BC
:004864BA B001                  mov al, 01
```

Referenced by a (U)nconditional or (C)onditional Jump at Address: `:004864B8(U)`

```
:004864BC A260BD4800            mov byte ptr [0048BD60], al
```

Referenced by a (U)nconditional or (C)onditional Jump at Address: `:004864AD(C)`

```
:004864C1 A093BD4800            mov al, byte ptr [0048BD93]
```
====> `[0048BD93]`=B8

```
:004864C6 3A05FBBD4800          cmp al, byte ptr [0048BDFB]
```
====> `[0048BDFB]`=C3 比较第 8 位！

```
:004864CC 7412                  je 004864E0
```
====> 不跳则 OVER！

```
:004864CE 803D60BD480000        cmp byte ptr [0048BD60], 00
:004864D5 33C0                  xor eax, eax
:004864D7 EB02                  jmp 004864DB
:004864D9 B001                  mov al, 01
```

Referenced by a (U)nconditional or (C)onditional Jump at Address: `:004864D7(U)`

```
:004864DB A260BD4800            mov byte ptr [0048BD60], al
```

Referenced by a (U)nconditional or (C)onditional Jump at Address: `:004864CC(C)`

```
:004864E0 A090BD4800            mov al, byte ptr [0048BD90]
```
====> `[0048BD90]`=46

```
:004864E5 3A05EBBD4800          cmp al, byte ptr [0048BDEB]
```
====> `[0048BDEB]`=3D 比较第 7 位！

```
:004864EB 7412                  je 004864FF
```
====> 不跳则 OVER！

```
:004864ED 803D60BD480000        cmp byte ptr [0048BD60], 00
:004864F4 33C0                  xor eax, eax
:004864F6 EB02                  jmp 004864FA
:004864F8 B001                  mov al, 01
```

Referenced by a (U)nconditional or (C)onditional Jump at Address: `:004864F6(U)`

```
:004864FA A260BD4800            mov byte ptr [0048BD60], al
```

Referenced by a (U)nconditional or (C)onditional Jump at Address: `:004864EB(C)`

```
:004864FF 8A1D60BD4800          mov bl, byte ptr [0048BD60]
```
====> `BL`=`[0048BD60]`=01 置 1 就 OK 了！

```
:00486505 EB06                  jmp 0048650D
```

Referenced by a (U)nconditional or (C)onditional Jump at Addresses: `:0048635F(C), :00486376(C)`

```
:00486507 33DB                  xor ebx, ebx
:00486509 EB02                  jmp 0048650D
```

Referenced by a (U)nconditional or (C)onditional Jump at Address: `:00486348(C)`

```
:0048650B 33DB                  xor ebx, ebx
```

Referenced by a (U)nconditional or (C)onditional Jump at Addresses: `:004863BE(U), :004863D4(U), :00486505(U), :00486509(U)`

```
:0048650D 33C0                  xor eax, eax
:0048650F 5A                    pop edx
:00486510 59                    pop ecx
:00486511 59                    pop ecx
:00486512 648910                mov dword ptr fs:[eax], edx
:00486515 682F654800            push 0048652F
```

Referenced by a (U)nconditional or (C)onditional Jump at Address: `:0048652D(U)`

```
:0048651A 8D45F8                lea eax, dword ptr [ebp-08]
:0048651D BA02000000            mov edx, 00000002
:00486522 E871DFF7FF            call 00404498
:00486527 C3                    ret
:00486528 E96FD9F7FF            jmp 00403E9C
:0048652D EBEB                  jmp 0048651A
:0048652F 8BC3                  mov eax, ebx
:00486531 5B                    pop ebx
:00486532 59                    pop ecx
:00486533 59                    pop ecx
:00486534 5D                    pop ebp
```
