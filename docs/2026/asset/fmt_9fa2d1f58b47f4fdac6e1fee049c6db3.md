
# Nullz CrackMe 1.1破解过程 (13千字)

软件名称：Nullz CrackMe 1.1

下载地址：http://go6.163.com/ddxia/crackme/Nullz\_crackMe1.1.zip

特点：共分五个小过程，难度逐级递增。

使用工具：TRW2000

我是刚学不久，平时也没太多时间练习，没写过过程，所以不足之处还请多多包涵啊!

好了，废话少说，我们开始吧！！（广告词？……）

## 1. 非常简单 (only Reg.Key)

`bpx hmemcpy`

`pmodule`

就到了下面的地方了

```
:004019D8 E801120000
    Call 00402BDE
:004019DD 8D542410
        lea edx, dword ptr [esp+10]
:004019E1 8D44242C
              lea eax, dword ptr [esp+2C]
:004019E5 52
  push edx    //注册码
:004019E6 50
              push eax    //你输入的假码
\* Reference To: KERNEL32.lstrcmpA, Ord:0295h
    |
:004019E7 FF1508404000
  Call dword ptr [00404008]    //比较函数
:004019ED 85C0
                  test eax, eax
:004019EF 755A
jne 00401A4B    //跳走就错
:004019F1 6870434000
        push 00404370
:004019F6 8D4C2410
            lea ecx, dword ptr [esp+10]
```

Reg.Key是个字符串常量：`qJT62aWfviq0P57JGs2FelQkX`

## 2. 简单 (User Name & Reg.Key)

还是`bpx hmemcpy`,`pmodule`.

```
\* Reference To: MFC42.Ordinal:0C1A, Ord:0C1Ah
                      |
:00401AF2 E8E7100000              Call 00402BDE
:00401AF7 8D542414
  lea edx, dword ptr [esp+14]
:00401AFB 52
              push edx    //User Name
\* Reference To: KERNEL32.lstrlenA, Ord:02A1h
      |
:00401AFC FF1500404000
    Call dword ptr [00404000]    //取字符串长度
:00401B02 8BF0
                  mov esi, eax
:00401B04 83FE05
cmp esi, 00000005    //User Name长度至少是5
:00401B07 7311
                jnb 00401B1A
:00401B09
6A40                    push 00000040
\* Possible StringData Ref from Data Obj ->"CrackMe"
        |
:00401B0B 6804514000
      push 00405104
\* Possible StringData Ref from Data Obj ->"User Name must have at least 5 "
      ->"characters."
                      |
:00401B10 68D8504000              push 004050D8
:00401B15 E9BA000000
jmp 00401BD4
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:
 |:00401B07(C)
 |
:00401B1A B801000000
        mov eax, 00000001
:00401B1F 33FF
                xor edi, edi
:00401B21
3BF0                    cmp esi, eax
:00401B23 7211
    jb 00401B36
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:
 |:00401B34(C)
 |
:00401B25 0FBE4C0414
          movsx ecx, byte ptr [esp+eax+14]
//注册码算法开始
:00401B2A 03CF
      add ecx, edi
:00401B2C 0FAFC8
          imul ecx, eax
:00401B2F 40
                  inc eax
:00401B30
8BF9                    mov edi, ecx
:00401B32 3BC6
    cmp eax, esi
:00401B34 76EF
          jbe 00401B25
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:
 |:00401B23(C)
 |
:00401B36 33C9
                  xor ecx, ecx
:00401B38 85F6
test esi, esi
:00401B3A 7620
      jbe 00401B5C
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:
 |:00401B5A(C)
 |
:00401B3C 0FBE6C0C14
            movsx ebp, byte ptr [esp+ecx+14]
:00401B41 8BC7
mov eax, edi
:00401B43 33D2
      xor edx, edx
:00401B45 F7F5
            div ebp
:00401B47 33D2
                xor edx, edx
:00401B49
BD0A000000              mov ebp, 0000000A
:00401B4E F7F5
div ebp
:00401B50 80C230
    add dl, 30
:00401B53 88540C48
      mov byte ptr [esp+ecx+48], dl
:00401B57 41
                  inc ecx
:00401B58
3BCE                    cmp ecx, esi
:00401B5A 72E0
    jb 00401B3C    //注册码算法结束
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:
 |:00401B3A(C)
 |
:00401B5C 8D542448
              lea edx, dword ptr [esp+48]
:00401B60 8D44247C                lea eax, dword ptr [esp+7C]
:00401B64 52
          push edx    //注册码
:00401B65
50
push eax    //你输入的Reg.Key
\* Reference To: KERNEL32.lstrcmpA, Ord:0295h
                |
:00401B66 FF1508404000
          Call dword ptr [00404008]    //比较函数
:00401B6C 85C0
  test eax, eax
:00401B6E 7550
        jne 00401BC0    //不同跳走
:00401B70 6870434000
            push 00404370
:00401B75 8D4C2414
              lea ecx, dword ptr [esp+14]
```

我的注册码： User Name: `eighth`

Reg.Key: `252303`

## 3. 一般简单 （User Name & Reg.Key）

老一套`bpx hmemcpy`,`pmodule`.

```
\* Reference To: MFC42.Ordinal:0C19, Ord:0C19h
            |
:00401C7E E8670F0000
          Call 00402BEA
:00401C83 8B6C2418
              mov ebp, dword ptr [esp+18]
  //User Name
:00401C87 8B55F8
      mov edx, dword ptr [ebp-08]    //User Name长度
:00401C8A 83FA05
cmp edx, 00000005
:00401C8D 7D11
        jge 00401CA0
      //长度不小于5
:00401C8F 6A40
          push 00000040
\* Possible StringData Ref from Data Obj ->"CrackMe"
                    |
:00401C91
6804514000              push 00405104
\* Possible StringData Ref from Data Obj ->"User Name must have at least 5 "
                  ->"characters."
            |
:00401C96 68D8504000
          push 004050D8
:00401C9B E9E2000000
            jmp 00401D82
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:
 |:00401C8D(C)
|
:00401CA0 33C0
  xor eax, eax
:00401CA2 3BD3
        cmp edx, ebx
:00401CA4 7E3C
              jle 00401CE2
:00401CA6 B901000000
            mov ecx, 00000001
:00401CAB 33FF
                  xor edi, edi
:00401CAD 2BCD
sub ecx, ebp
:00401CAF 894C241C
    mov dword ptr [esp+1C], ecx
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:
 |:00401CDA(C)
 |
:00401CB3 0FBE5C0500
            movsx ebx, byte ptr [ebp+eax]
  //注册码算法开始
:00401CB8 8D4C0500
    lea ecx, dword ptr [ebp+eax]
:00401CBC 03F3
              add esi, ebx
:00401CBE 8BD8
                  mov ebx, eax
:00401CC0 C1E308
shl ebx, 08
:00401CC3 33F3
      xor esi, ebx
:00401CC5 8B5C241C
          mov ebx, dword ptr [esp+1C]
:00401CC9
03D9                    add ebx, ecx
:00401CCB 8BCF
    mov ecx, edi
:00401CCD 0FAFF3
        imul esi, ebx
:00401CD0 F7D1
              not ecx
:00401CD2 0FAFF1
                imul esi, ecx
:00401CD5
40
inc eax
:00401CD6 03FA
    add edi, edx
:00401CD8 3BC2
          cmp eax, edx
:00401CDA 7CD7
                jl 00401CB3
//注册码算法结束
:00401CDC 8B7C2420
  mov edi, dword ptr [esp+20]
:00401CE0 33DB
            xor ebx, ebx
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:
 |:00401CA4(C)
|
:00401CE2 56
    push esi
:00401CE3 8D542414
      lea edx, dword ptr [esp+14]
\* Possible StringData Ref from Data Obj ->"%lu"
                    |
:00401CE7
682C514000              push 0040512C
:00401CEC 52
  push edx
\* Reference To: MFC42.Ordinal:0B02, Ord:0B02h
        |
:00401CED E8F20E0000
      Call 00402BE4
:00401CF2 8B74241C
          mov esi, dword ptr [esp+1C]    //真码
:00401CF6 8B442420
mov eax, dword ptr [esp+20]    //假码
:00401CFA 83C40C
              add esp, 0000000C
\*
Referenced by a (U)nconditional or (C)onditional Jump at Address:
 |:00401D1B(C)
 |
:00401CFD 8A10
    mov dl, byte ptr [eax]    //逐位比较，真搞不懂，多此一举嘛！？
:00401CFF
8ACA                    mov cl, dl
:00401D01 3A16
    cmp dl, byte ptr [esi]
:00401D03 751C
            jne 00401D21
:00401D05 3ACB
                  cmp cl, bl
:00401D07 7414
je 00401D1D
:00401D09 8A5001
    mov dl, byte ptr [eax+01]
:00401D0C 8ACA
              mov cl, dl
:00401D0E 3A5601
                cmp dl, byte ptr [esi+01]
:00401D11 750E
  jne 00401D21
:00401D13 83C002
      add eax, 00000002
:00401D16 83C602
            add esi, 00000002
:00401D19 3ACB
                  cmp cl, bl
:00401D1B 75E0
jne 00401CFD
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:
 |:00401D07(C)
 |
:00401D1D 33C0
              xor eax, eax
:00401D1F EB05
                  jmp 00401D26
```

我的注册码： User Name: `eighth`

Reg.Key: `2990909536`

## 4. 一般吧 （User Name,Company & Reg.Key）

不过Company好像没用到啊！？

管它呢，`bpx hmemcpy`，`pmodule`再说。

```
\* Reference To: MFC42.Ordinal:0C19, Ord:0C19h
                |
:00401E43 E8A20D0000
            Call 00402BEA
```
