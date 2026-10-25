
## 关键 CALL 分析

```text
* Reference To: MSVBVM60.__vbaFreeVarList, Ord:0000h
           |
:0041C6DF FF1530104000            Call dword
ptr [00401030]
:0041C6E5 83C414                  add
esp, 00000014
* Referenced by a (U)nconditional or (C)onditional Jump at
Address:
|:0041C65F(U)
|
* Reference To: MSVBVM60.__vbaExitProc, Ord:0000h
           |
:0041C6E8 FF1588104000            Call dword
ptr [00401088]
:0041C6EE 6840C74100              push
0041C740
:0041C6F3 EB4A
 jmp 0041C73F
```

---

### 进入关键 CALL

进入关键 CALL：`0041C5CC call dword ptr [edx+00000710]`

…… ……省略…… ……

```text
* Reference To: MSVBVM60.__vbaStrMove, Ord:0000h
           |
:0041C841 8B3D40124000            mov edi, dword
ptr [00401240]
:0041C847 FFD7
 call edi
:0041C849 6A01
 push 00000001
:0041C84B 6AFF
 push FFFFFFFF
:0041C84D 6A01
 push 00000001
:0041C84F 68286E4000              push
00406E28
:0041C854 6824844000              push
00408424
:0041C859 8B4E54                  mov
ecx, dword ptr [esi+54]
:0041C85C 51
   push ecx
           ====>ECX=fly-13572468
     试炼码
* Reference To: MSVBVM60.rtcReplace, Ord:02C8h
           |
:0041C85D FF158C114000            Call dword
ptr [0040118C]
           ====>去除试炼码中的-
:0041C863 8BD0
     mov edx, eax
           ====>EDX=fly13572468
:0041C865 8D4DCC
     lea ecx, dword ptr [ebp-34]
:0041C868 FFD7
 call edi
:0041C86A 8B55CC                  mov
edx, dword ptr [ebp-34]
:0041C86D 52
   push edx
:0041C86E 8B4654                  mov
eax, dword ptr [esi+54]
:0041C871 50
   push eax
* Reference To: MSVBVM60.__vbaStrCmp, Ord:0000h
           |
:0041C872 FF1524114000            Call dword
ptr [00401124]
           ====>其实这段就是检测试炼码中有无-
:0041C878 8BD8
     mov ebx, eax
           ====>返回值。 有 - 则EAX=1
:0041C87A F7DB
     neg ebx
:0041C87C 1BDB
 sbb ebx, ebx
:0041C87E 43
   inc ebx
           ====>EBX=0
:0041C87F 8B4DCC
     mov ecx, dword ptr [ebp-34]
           ====>ECX=fly13572468
:0041C882 51
       push ecx
* Reference To: MSVBVM60.__vbaLenBstr, Ord:0000h
           |
:0041C883 8B3D28104000            mov edi, dword
ptr [00401028]
:0041C889 FFD7
 call edi
           ====>取fly13572468的长度 EAX=B
:0041C88B 33D2
     xor edx, edx
:0041C88D 83F808                  cmp
eax, 00000008
           ====>和8位比较
:0041C890 0F9EC2
     setle dl
           ====>设置DL值
:0041C893 0BDA
     or ebx, edx
:0041C895 0F8540010000            jne 0041C9DB
           ====>跳则OVER！爆破点①！
:0041C89B 8B45CC
     mov eax, dword ptr [ebp-34]
           ====>EAX=fly13572468
:0041C89E 50
       push eax
:0041C89F FFD7
 call edi
           ====>取fly13572468的长度
:0041C8A1 8985FCFEFFFF            mov
dword ptr [ebp+FFFFFEFC], eax
           ====>[ebp+FFFFFEFC]=B
:0041C8A7 C7465801000000          mov
[esi+58], 00000001
* Reference To: MSVBVM60.__vbaVarTstEq, Ord:0000h
           |
:0041C8AE 8B1D2C114000            mov ebx, dword
ptr [0040112C]
* Referenced by a (U)nconditional or (C)onditional Jump at
Address:
|:0041C996(U)
|
:0041C8B4 8B4658                  mov
eax, dword ptr [esi+58]
:0041C8B7 3B85FCFEFFFF            cmp eax, dword
ptr [ebp+FFFFFEFC]
:0041C8BD 0F8F18010000            jg 0041C9DB
:0041C8C3 C745BC08000000          mov [ebp-44], 00000008
:0041C8CA C745B402000000          mov [ebp-4C], 00000002
:0041C8D1 8D4DCC                  lea
ecx, dword ptr [ebp-34]
:0041C8D4 898D4CFFFFFF            mov dword
ptr [ebp+FFFFFF4C], ecx
:0041C8DA C78544FFFFFF08400000    mov dword ptr [ebp+FFFFFF44],
00004008
:0041C8E4 8D55B4                  lea
edx, dword ptr [ebp-4C]
:0041C8E7 52
   push edx
:0041C8E8 50
   push eax
:0041C8E9 8D8544FFFFFF            lea eax, dword
ptr [ebp+FFFFFF44]
:0041C8EF 50
   push eax
:0041C8F0 8D4DA4                  lea
ecx, dword ptr [ebp-5C]
:0041C8F3 51
   push ecx
* Reference To: MSVBVM60.rtcMidCharVar, Ord:0278h
           |
:0041C8F4 FF15FC104000            Call dword
ptr [004010FC]
           ====>依次8位取fly13572468
           ====>fly13572
           …… ……省略…… ……
           ====>13572468
:0041C8FA 8D55A4                  lea
edx, dword ptr [ebp-5C]
:0041C8FD 52
   push edx
:0041C8FE 8D45C8                  lea
eax, dword ptr [ebp-38]
:0041C901 50
   push eax
* Reference To: MSVBVM60.__vbaStrVarVal, Ord:0000h
           |
:0041C902 FF15B0114000            Call dword
ptr [004011B0]
:0041C908 50
   push eax
* Reference To: MSVBVM60.rtcR8ValFromBstr, Ord:0245h
           |
:0041C909 FF1578124000            Call dword
ptr [00401278]
:0041C90F DC25581D4000            fsub qword
ptr [00401D58]
           …… ……省略…… ……
           ====>13572468 - 8072353=5500115.0000000000000
:0041C915 DD5D9C                  fstp
qword ptr [ebp-64]
:0041C918 DFE0
 fstsw ax
:0041C91A A80D
 test al, 0D
:0041C91C 0F8514040000            jne 0041CD36
:0041C922 C7459405000000          mov [ebp-6C], 00000005
:0041C929 8D4D94                  lea
ecx, dword ptr [ebp-6C]
:0041C92C 51
   push ecx
:0041C92D 8D5584                  lea
edx, dword ptr [ebp-7C]
:0041C930 52
   push edx
* Reference To: MSVBVM60.rtcHexVarFromVar, Ord:023Dh
           |
:0041C931 FF15E4114000            Call dword
ptr [004011E4]
           …… ……省略…… ……
           ====>取5500115的16进制值=53ECD3
:0041C937 8B45E0                  mov
eax, dword ptr [ebp-20]
           ====>EAX=131A4F8
      程序给的注册源码
:0041C93A 89851CFFFFFF            mov
dword ptr [ebp+FFFFFF1C], eax
:0041C940 C78514FFFFFF08800000    mov dword ptr [ebp+FFFFFF14],
00008008
:0041C94A 8D4D84                  lea
ecx, dword ptr [ebp-7C]
:0041C94D 51
   push ecx
:0041C94E 8D9514FFFFFF            lea edx, dword
ptr [ebp+FFFFFF14]
:0041C954 52
   push edx
:0041C955 FFD3
 call ebx
           ====>比较CALL！53ECD3和131A4F8比较！相等则OK！
:0041C957 668BF8                  mov
di, ax
:0041C95A 8D4DC8                  lea
ecx, dword ptr [ebp-38]
* Reference To: MSVBVM60.__vbaFreeStr, Ord:0000h
           |
:0041C95D FF1574124000            Call dword
ptr [00401274]
:0041C963 8D4584                  lea
eax, dword ptr [ebp-7C]
:0041C966 50
   push eax
:0041C967 8D4D94                  lea
ecx, dword ptr [ebp-6C]
:0041C96A 51
   push ecx
:0041C96B 8D55A4                  lea
edx, dword ptr [ebp-5C]
:0041C96E 52
   push edx
:0041C96F 8D45B4                  lea
eax, dword ptr [ebp-4C]
:0041C972 50
   push eax
:0041C973 6A04
 push 00000004
* Reference To: MSVBVM60.__vbaFreeVarList, Ord:0000h
           |
:0041C975 FF1530104000            Call dword
ptr [00401030]
:0041C97B 83C414                  add
esp, 00000014
:0041C97E 6685FF                  test
di, di
:0041C981 7518
 jne 0041C99B
           ====>爆破点②！
:0041C983 8B4E58
     mov ecx, dword ptr [esi+58]
:0041C986 B801000000              mov eax,
00000001
:0041C98B 03C8
 add ecx, eax
:0041C98D 0F80A8030000            jo 0041CD3B
:0041C993 894E58                  mov
dword ptr [esi+58], ecx
:0041C996 E919FFFFFF              jmp 0041C8B4
           ====>循环，直至取完试炼码！
```

---

## 算法总结

1. 注册码格式为：`N-K`
2. `N` 是注册名，可以为任意字符
3. `K` 的最后 8 位数字 = 注册源码 `131A4F8` + `7B2CA1` 结果的 10 进制值

---

## 完美爆破

```asm
1. 0041C895 0F8540010000            jne 0041C9DB
   改为：909090909090            NOP掉！

2. 0041C981 7518
   jne 0041C99B
   改为：EB18
   JMP 0041C99B
```

---

## 注册信息保存

```text
[HKEY_CURRENT_USER\Software\VB and VBA Program Settings\notepad\regist]
"regnumber"="fly-28103065"
```

---

## 整理

源代码：`131A4F8`
注册码：`fly-28103065`

---

```text
        ,     _/

       /| _.-~/            \\_

    ,        青春都一饷

      ( /~   /

 \~-._ |\

      `\\  _/

   \   ~\ )          忍把浮名

  _-~~~-.)  )__/;;,.          \\_

 //'

 /'_,\   --~   \ ~~~-  ,;;\\___(  (.-~~~-.

       换了破解轻狂

`~ _( ,_..--\ (     ,;'' /    ~--   /._`\
```
