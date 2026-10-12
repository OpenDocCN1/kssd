
```
:004011F7 8D4C241C                lea ecx, dword ptr [esp+1C]
:004011FB 8D542420                lea edx, dword ptr [esp+20]
:004011FF 51                      push ecx
:00401200 52                      push edx
:00401201 53                      push ebx
:00401202 E869FEFFFF             call 00401070================>BF_Enc(xl,xr,key)
:00401207 8B44242C                mov eax, dword ptr [esp+2C]
:0040120B 8B4C2428                mov ecx, dword ptr [esp+28]
:0040120F 8946FC                  mov dword ptr [esi-04], eax
:00401212 890E                    mov dword ptr [esi], ecx
:00401214 83C40C                  add esp, 0000000C
:00401217 83C608                  add esi, 00000008
:0040121A 4F                      dec edi
:0040121B 75DA                    jne 004011F7
:0040121D 4D                      dec ebp
:0040121E 75D2                    jne 004011F2
:00401220 5F                      pop edi
:00401221 5E                      pop esi
:00401222 5D                      pop ebp
:00401223 5B                      pop ebx
:00401224 59                      pop ecx
:00401225 C3                      ret
```

\======================`Init_Key` 过程分析完毕============================

\======================================================================

\=============================分析详细总结=============================

\======================================================================

\=====>`BF_Enc(ComputerID,key="ChinaCrackingGroup")`
* Possible StringData Ref from Data Obj ->"ChinaCrackingGroup"

```
:00401434 6830804000                push 00408030
:00401439 6880894000                push 00408980
:0040143E E8EDFCFFFF                call 00401130===>Init_Key
...........
:00401667 68EC994000                push 004099EC
:0040166C 68F0994000                push 004099F0
:00401671 6880894000                push 00408980
:00401676 E8F5F9FFFF                call 00401070===>BF_Enc
```

\=====>`BF_Enc(ComputerID,key="ChinaCrackingGroup")`

\======================================================================

\=====>`BF_Dec(Code,key="CrackingForFun")`
* Possible StringData Ref from Data Obj ->"CrackingForFun"

```
:004016C1 6844804000                push 00408044
:004016C6 6880894000                push 00408980
:004016CB E860FAFFFF                call 00401130===>Init_Key
...........
:004015D9 51                      push ecx
:004015DA 52                      push edx
:004015DB 6880894000                push 00408980
:004015E0 E8EBFAFFFF                call 004010D0===>BF_Dec
```

\=====>`BF_Dec(Code,key="CrackingForFun")`

\======================================================================

\=====>`BF_Enc("blowfish",key=ProductID)`

```
:0040131F 6880894000                push 00408980
:00401324 E807FEFFFF                call 00401130===>Init_Key
:00401329 68EC994000                push 004099EC
:0040132E 68F0994000                push 004099F0
:00401333 6880894000                push 00408980
:00401338 C705F0994000626C6F77      mov dword ptr [004099F0], 776F6C62
:00401342 C705EC99400066697368      mov dword ptr [004099EC], 68736966
:0040134C E81FFDFFFF                call 00401070===>BF_Enc
```

\=====>`BF_Enc("blowfish",key=ProductID)`

\======================================================================

\=====>最后分析结果

```
ComputerID=BF_Enc("blowfish",key=ProductID)

x=BF_Dec(Code,key="CrackingForFun")

y=BF_Enc(ComputerID,key="ChinaCrackingGroup")

x=y 则注册成功；

我们要得到正确的注册码，那么
Code=BF_Enc(x,key="CrackingForFun");
  =BF_Enc(y,key="CrackingForFun");
  =BF_Enc(BF_Enc(ComputerID,key="ChinaCrackingGroup"),key="CrackingForFun");

  如果更进一步，那么
  =BF_Enc(BF_Enc(BF_Enc("blowfish",
                      key=ProductID),
                key="ChinaCrackingGroup"),
            key="CrackingForFun");

这样我们便可以编写它的 keygen 了
```

\=====>

\======================================================================

`4dd466d0e1e0cc123afbe504da2462ca`
