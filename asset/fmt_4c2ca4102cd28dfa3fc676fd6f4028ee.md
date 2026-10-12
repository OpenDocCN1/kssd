
```
:00404F93 DB442414                  fild dword ptr [esp+14]  
                                     ==>EAX 增 1
:00404F97 3BC6                      cmp eax, esi  
                                     ==>EAX 已增 1，所以只取前 5 位的值累加
:00404F99 DEC1                      faddp st(1), st(0)  
                                     ==>其实是浮点数累加  
                                     ==>st(0)=75+83+80+78+82=398.00000000000000000
:00404F9B 7CE9                      jl 00404F86  
                                     ==>循环
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:00404F84(C)  
|  

```
:00404F9D D9C0                      fld st(0), st(0)  
:00404F9F D9FE                      fsin  
                                     ==>取 SIN 值 398.00000000000000000=0.8317580087191733248
:00404FA1 D9FF                      fcos  
                                     ==>取 COS 值 0.8317580087191733248=0.6735774280714966016
:00404FA3 D9FE                      fsin  
                                     ==>取 SIN 值 0.6735774280714966016=0.6237860730035777536
:00404FA5 D9FF                      fcos  
                                     ==>取 COS 值 0.6237860730035777536=0.8116727871807730688
:00404FA7 D9FE                      fsin  
                                     ==>取 SIN 值 0.8116727871807730688=0.7254395446159557632
:00404FA9 DD542414                  fst qword ptr [esp+14]  
:00404FAD DC1DD8DD4000              fcomp qword ptr [0040DDD8]  
:00404FB3 DFE0                      fstsw ax  
:00404FB5 F6C401                   test ah, 01  
:00404FB8 7423                      je 00404FDD  
                                     ==>跳了下去 这个测试不清楚。
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:00404FDB(C)  
|  

```
:00404FBA DC0DD0DD4000              fmul qword ptr [0040DDD0]  
:00404FC0 D9C0                      fld st(0), st(0)  
:00404FC2 D9FE                      fsin  
:00404FC4 D9FF                      fcos  
:00404FC6 D9FE                      fsin  
:00404FC8 D9FF                      fcos  
:00404FCA D9FE                      fsin  
:00404FCC DD542414                  fst qword ptr [esp+14]  
:00404FD0 DC1DD8DD4000              fcomp qword ptr [0040DDD8]  
:00404FD6 DFE0                      fstsw ax  
:00404FD8 F6C401                   test ah, 01  
:00404FDB 75DD                      jne 00404FBA
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:00404FB8(C)  
|  

```
:00404FDD 8B542418                  mov edx, dword ptr [esp+18]  
:00404FE1 8B442414                  mov eax, dword ptr [esp+14]  
:00404FE5 52                        push edx  
:00404FE6 50                        push eax  
:00404FE7 8D8C24E0000000            lea ecx, dword ptr [esp+000000E0]

* Possible StringData Ref from Data Obj ->"%.14f"  
                                     |
:00404FEE 6874134100                push 00411374  
:00404FF3 51                        push ecx  
:00404FF4 DDD8                      fstp st(0)

* Reference To: MSVCRT.sprintf, Ord:02B2h  
                                     |
:00404FF6 FF1528D44000              Call dword ptr [0040D428]  
                                     ==>四舍五入取 0.7254395446159557632 的前 16 位
:00404FFC 8DBC24E8000000            lea edi, dword ptr [esp+000000E8]  
                                     ==>EDI=0.72543954461596
:00405003 8BCD                      mov ecx, ebp  
:00405005 33C0                      xor eax, eax  
:00405007 83C410                    add esp, 00000010  
:0040500A 33D2                      xor edx, edx  
:0040500C F2                       repnz  
:0040500D AE                        scasb  
:0040500E F7D1                     not ecx  
:00405010 49                       dec ecx  
:00405011 83E902                    sub ecx, 00000002  
:00405014 7427                      je 0040503D
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:0040503B(C)  
|  

```
:00405016 8A8414DA000000            mov al, byte ptr [esp+edx+000000DA]  
                                     ==>依次取 0.725439544615956 小数点后字符的 HEX 值
:0040501D 8DBC24D8000000            lea edi, dword ptr [esp+000000D8]  
:00405024 0441                      add al, 41  
                                     ==>依次+41 只记 5 位了，因为后面只取 5 位。即：用户名位数-1  
                                     1、    ==>37 + 41=78    即：x  
                                     2、    ==>32 + 41=73    即：s  
                                     3、    ==>35 + 41=76    即：v  
                                     4、    ==>34 + 41=75    即：u  
                                     5、    ==>33 + 41=74    即：t
:00405026 8BCD                      mov ecx, ebp  
:00405028 88841488000000            mov byte ptr [esp+edx+00000088], al  
                                     ==>保存在 [esp+edx+00000088]
:0040502F 33C0                      xor eax, eax  
:00405031 42                       inc edx  
:00405032 F2                       repnz  
:00405033 AE                       scasb  
:00405034 F7D1                     not ecx  
:00405036 83C1FD                    add ecx, FFFFFFFD  
:00405039 3BD1                      cmp edx, ecx  
:0040503B 72D9                      jb 00405016  
                                     ==>循环
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:00405014(C)  
|  

```
:0040503D 8D8C2488000000            lea ecx, dword ptr [esp+00000088]  
:00405044 C684348800000000           mov byte ptr [esp+esi+00000088], 00  
                                     ==>多于 5 位的换成 00 了。只取用户名位数-1
:0040504C 51                        push ecx  
                                     ==>ECX=xsvut
:0040504D FFD3                      call ebx  
                                     ==>把 xsvut 转换成大写字母 XSVUT
:0040504F 8D7C2464                  lea edi, dword ptr [esp+64]  
                                     ==>EDI=13572468           试炼码
:00405053 8BCD                      mov ecx, ebp  
:00405055 33C0                      xor eax, eax  
:00405057 8DB42488000000            lea esi, dword ptr [esp+00000088]  
                                     ==>ESI=XSVUT               注册码
:0040505E F2                       repnz  
:0040505F AE                       scasb  
:00405060 F7D1                     not ecx  
:00405062 49                       dec ecx  
:00405063 8D7C2464                  lea edi, dword ptr [esp+64]  
:00405067 33D2                      xor edx, edx  
:00405069 89AC2430010000            mov dword ptr [esp+00000130], ebp  
:00405070 F3                       repz  
:00405071 A6                       cmpsb  
                                     ==>逐位比较。有一处不同就 OVER 了。
:00405072 8D4C2410                  lea ecx, dword ptr [esp+10]  
:00405076 7528                      jne 004050A0  
                                     ==>跳则 OVER！
```

* Reference To: MFC42.Ordinal:0320, Ord:0320h  
|  
```
:00405078 E8D3640000                Call 0040B550  
:0040507D B001                      mov al, 01  
                                     ==>置 1 则 OK！
:0040507F 8B8C2428010000            mov ecx, dword ptr [esp+00000128]  
:00405086 64890D00000000            mov dword ptr fs:[00000000], ecx  
:0040508D 5F                        pop edi  
:0040508E 5E                        pop esi  
:0040508F 5D                        pop ebp  
:00405090 5B                        pop ebx  
:00405091 8BE5                      mov esp, ebp  
:00405093 5D                        pop ebp  
:00405094 C3                        ret
```

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:  
|:00404ED3(C), :00404EE7(C)  
|  
```
:00405095 89AC2430010000            mov dword ptr [esp+00000130], ebp
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:00404E0A(U)  
|  
```
:0040509C 8D4C2410                  lea ecx, dword ptr [esp+10]
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:00405076(C)  
|

* Reference To: MFC42.Ordinal:0320, Ord:0320h  
|  
```
:004050A0 E8AB640000                Call 0040B550
```
