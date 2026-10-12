
```
:004831A6 53                   push ebx   
* Possible StringData Ref from Data Obj ->"Luo98437"   
                              |   

:004831A7 68AC785400          push 005478AC   

:004831AC 8D4C2410            lea ecx, dword ptr [esp+10]   

:004831B0 E8D10F0600          call 004E4186   

:004831B5 85C0                test eax, eax   

:004831B7 0F8DD4000000        jnl 00483291   

:004831BD 53                  push ebx   

* Possible StringData Ref from Data Obj ->"Luo98"   

                              |   

:004831BE 68A4785400          push 005478A4   

:004831C3 8D4C2410            lea ecx, dword ptr [esp+10]   

:004831C7 E8BA0F0600          call 004E4186   

:004831CC 85C0                test eax, eax   

:004831CE 7D17                jge 004831E7   

:004831D0 53                  push ebx   

* Possible StringData Ref from Data Obj ->"Luo01"   

                              |   

:004831D1 689C785400          push 0054789C   

:004831D6 8D4C2410            lea ecx, dword ptr [esp+10]   

:004831DA E8A70F0600          call 004E4186   

:004831DF 85C0                test eax, eax   

:004831E1 0F8CAA000000        jl 00483291   

* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:004831CE(C)   

|   

:004831E7 53                  push ebx   

* Possible StringData Ref from Data Obj ->"Luo984"   

                              |   

:004831E8 6894785400          push 00547894   

:004831ED 8D4C2410            lea ecx, dword ptr [esp+10]   

:004831F1 E8900F0600          call 004E4186   

:004831F6 85C0                test eax, eax   

:004831F8 7D26                jge 00483220   

:004831FA 53                  push ebx   

* Possible StringData Ref from Data Obj ->"Luo985"   

                              |   

:004831FB 688C785400          push 0054788C   

:00483200 8D4C2410            lea ecx, dword ptr [esp+10]   

:00483204 E87D0F0600          call 004E4186   

:00483209 85C0                test eax, eax   

:0048320B 7D13                jge 00483220   

:0048320D 53                  push ebx   

* Possible StringData Ref from Data Obj ->"Luo014"   

                              |   

:0048320E 6884785400          push 00547884   

:00483213 8D4C2410            lea ecx, dword ptr [esp+10]   

:00483217 E86A0F0600          call 004E4186   

:0048321C 85C0                test eax, eax   

:0048321E 7C14                jl 00483234   

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:   
|:004831F8(C), :0048320B(C)   

|   

:00483220 C705E0040A0202000000   mov dword ptr [020A04E0], 00000002       ====>这一处   

:0048322A C786406BC90001000000   mov dword ptr [esi+00C96B40], 00000001   

* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:0048321E(C)   

|   

:00483234 53                  push ebx   

* Possible StringData Ref from Data Obj ->"Luo982"   

                              |   

:00483235 687C785400          push 0054787C   

:0048323A 8D4C2410            lea ecx, dword ptr [esp+10]   

:0048323E E8430F0600          call 004E4186   

:00483243 85C0                test eax, eax   

:00483245 7D13                jge 0048325A   

:00483247 53                  push ebx   

* Possible StringData Ref from Data Obj ->"Luo012"   

                              |   

:00483248 6874785400          push 00547874   

:0048324D 8D4C2410            lea ecx, dword ptr [esp+10]   

:00483251 E8300F0600          call 004E4186   

:00483256 85C0                test eax, eax   

:00483258 7C10                jl 0048326A   

* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:00483245(C)   

|   

:0048325A C705E0040A0201000000   mov dword ptr [020A04E0], 00000001   

:00483264 899E406BC900         mov dword ptr [esi+00C96B40], ebx   

* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:00483258(C)   

|   

:0048326A 8D4C2408            lea ecx, dword ptr [esp+08]   

:0048326E C7442420FFFFFFFF     mov [esp+20], FFFFFFFF   

:00483276 E83D750600          call 004EA7B8   

:0048327B 5E                  pop esi   

:0048327C B801000000          mov eax, 0000001   

:00483281 5B                  pop ebx   

:00483282 8B4C2410            mov ecx, dword ptr [esp+10]   

:00483286 64890D00000000      mov dword ptr fs:[00000000], ecx   

:0048328D 83C41C              add esp, 0000001C   

:00483290 C3                  ret   

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:   
|:004831A0(C), :004831B7(C), :004831E1(C)   

|   

:00483291 8D4C2408            lea ecx, dword ptr [esp+08]   

:00483295 C7442420FFFFFFFF     mov [esp+20], FFFFFFFF   

:0048329D E816750600          call 004EA7B8   

* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:00483164(C)   

|   

:004832A2 8B4C2418            mov ecx, dword ptr [esp+18]   

:004832A6 5E                  pop esi   

:004832A7 33C0                xor eax, eax   

:004832A9 5B                  pop ebx   

:004832AA 64890D00000000      mov dword ptr fs:[00000000], ecx   

:004832B1 83C41C              add esp, 0000001C   

:004832B4 C3                  ret   

.........   

........   

:004832DE 90                  nop   

:004832DF 90                  nop   

:004832E0 56                  push esi   

:004832E1 8BF1                mov esi, ecx   

:004832E3 E81B820600          call 004EB503   

:004832E8 A1E0040A02          mov eax, dword ptr [020A04E0]   ====>注意   

:004832ED 85C0                test eax, eax   

:004832EF 750D                jne 004832FE   

* Possible StringData Ref from Data Obj ->"????2000(学习版)"   
      ; ====>！就是这，往下看   

|   

:004832F1 68EC785400          push 005478EC   

:004832F6 8D4E5C              lea ecx, dword ptr [esi+5C]   

:004832F9 E843760600          call 004EA941   

* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:004832EF(C)   

|   

:004832FE 833DE0040A0201      cmp dword ptr [020A04E0], 00000001 ==>注意   

:00483305 750D                jne 00483314   

* Possible StringData Ref from Data Obj ->"???/2000(设计版)"  ===>！！   

                              |   

:00483307 68D8785400          push 005478D8   

:0048330C 8D4E5C              lea ecx, dword ptr [esi+5C]   

:0048330F E82D760600          call 004EA941   

* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:00483305(C)   

|   

:00483314 833DE0040A0202      cmp dword ptr [020A04E0], 00000002 ==>注意   

:0048331B 750D                jne 0048332A   

* Possible StringData Ref from Data Obj ->"????2000(企业版)"    ===>！！！   

                              |   

:0048331D 68C4785400          push 005478C4   

:00483322 8D4E5C              lea ecx, dword ptr [esi+5C]   

:00483325 E817760600          call 004EA941   

* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:0048331B(C)   

|   

:0048332A 6A00                push 00000000   

:0048332C 8BCE                mov ecx, esi   

:0048332E E88E5E0600          call 004E91C1   

:00483333 B801000000          mov eax, 00000001   

:00483338 5E                  pop esi   

:00483339 C3                  ret   
```

;;;你发现什么了？`dword ptr [020A04E0]`的值代表什么，不用我说了。开始，我并没有去改`dword ptr [020A04E0]`的值，我只是改跳转，跳到`"??????(企业版)"`，`about`显示是企业版，但功能还是学习版。

;;;;好，我们来改`dword ptr [020A04E0]`的值，使他为`"00000002"`。查找`dword ptr [020A04E0]`，我们找到不少，我们还是看看这种`"mov dword ptr [020A04E0], 00000002"`地方
