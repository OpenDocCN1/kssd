
# 紫禁城反黄卫士个人版

> 原文：[`www.pediy.com/kssd/pediy04/kanxue194.htm`](https://www.pediy.com/kssd/pediy04/kanxue194.htm)

紫禁城反黄卫士个人版注册算法：

破解工具：`TRW2000`

`Hview`

`w32dasm 8.93 黄金中文版`

运行程序，打开注册的对话框，在注册框中随便输入注册码，运行 `TRW2000`，下断点 `BPX hmemcpy`，按注册按钮，被 `TRW` 中断，用 `PMODULE` 返回主程序，如下：

```
:004A12ED 8B8538FEFFFF             mov eax, dword ptr [ebp+FFFFFE38]  <====返回到这里

:004A12F3 8D55F8                   lea edx, dword ptr [ebp-08]

:004A12F6 E8417AF6FF               call 00408D3C

:004A12FB 8B45F8                   mov eax, dword ptr [ebp-08]     <====取得第一个注册框的字符

* Possible StringData Ref from Code Obj ->"KYNT"

                              |

:004A12FE BAB4164A00               mov edx, 004A16B4

:004A1303 E8983AF6FF               call 00404DA0     <=====比较第一个注册码是否是 KYNT

:004A1308 741A                     je 004A1324       <=====如果是则继续比较注册码,所以第一个注册框的注册码一定是 KYNT

:004A130A 6A00                     push 00000000

:004A130C 668B0DBC164A00            mov cx, word ptr [004A16BC]

:004A1313 33D2                     xor edx, edx

* Possible StringData Ref from Code Obj ->"您输入的序列号有误，请检查输入。"  <====不是 KYNT,则注册失败

                              |

:004A1315 B8C8164A00               mov eax, 004A16C8

:004A131A E8753BF9FF               call 00434E94

:004A131F E947030000               jmp 004A166B

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:004A1308(C)

|

:004A1324 8D55F4                   lea edx, dword ptr [ebp-0C]

:004A1327 8B8314030000             mov eax, dword ptr [ebx+00000314]

:004A132D E836A1F9FF               call 0043B468

:004A1332 8D55F0                   lea edx, dword ptr [ebp-10]

:004A1335 8B8318030000             mov eax, dword ptr [ebx+00000318]

:004A133B E828A1F9FF               call 0043B468

:004A1340 8D55EC                   lea edx, dword ptr [ebp-14]

:004A1343 8B831C030000             mov eax, dword ptr [ebx+0000031C]

:004A1349 E81AA1F9FF               call 0043B468

:004A134E 8D55E8                   lea edx, dword ptr [ebp-18]

:004A1351 8B8320030000             mov eax, dword ptr [ebx+00000320]

:004A1357 E80CA1F9FF               call 0043B468

:004A135C 8D55FC                   lea edx, dword ptr [ebp-04]

:004A135F 8B8308030000             mov eax, dword ptr [ebx+00000308]

:004A1365 E8FEA0F9FF               call 0043B468

:004A136A 8B45F8                   mov eax, dword ptr [ebp-08]

:004A136D E8EA38F6FF               call 00404C5C                     <====这段代码是比较每一个注册框中的注册码是不是 4 位

:004A1372 83F804                   cmp eax, 00000004

:004A1375 7534                     jne 004A13AB                      <====不是则注册失败

:004A1377 8B45F4                   mov eax, dword ptr [ebp-0C]

:004A137A E8DD38F6FF               call 00404C5C

:004A137F 83F804                   cmp eax, 00000004

:004A1382 7527                     jne 004A13AB

:004A1384 8B45F0                   mov eax, dword ptr [ebp-10]

:004A1387 E8D038F6FF               call 00404C5C

:004A138C 83F804                   cmp eax, 00000004

:004A138F 751A                     jne 004A13AB

:004A1391 8B45EC                   mov eax, dword ptr [ebp-14]

:004A1394 E8C338F6FF               call 00404C5C

:004A1399 83F804                   cmp eax, 00000004

:004A139C 750D                     jne 004A13AB

:004A139E 8B45E8                   mov eax, dword ptr [ebp-18]

:004A13A1 E8B638F6FF               call 00404C5C

:004A13A6 83F804                   cmp eax, 00000004

:004A13A9 741A                     je 004A13C5               <=====全部都是 4 位,则跳到运算注册码的代码处

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

|:004A1375(C), :004A1382(C), :004A138F(C), :004A139C(C)

|

:004A13AB 6A00                     push 00000000

:004A13AD 668B0DBC164A00            mov cx, word ptr [004A16BC]

:004A13B4 33D2                     xor edx, edx

* Possible StringData Ref from Code Obj ->"您输入的序列号有误，请检查输入。"

                              |

:004A13B6 B8C8164A00               mov eax, 004A16C8

:004A13BB E8D43AF9FF               call 00434E94

:004A13C0 E9A6020000               jmp 004A166B

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:004A13A9(C)

|

:004A13C5 33C0                     xor eax, eax

:004A13C7 55                       push ebp

:004A13C8 6806144A00               push 004A1406

:004A13CD 64FF30                   push dword ptr fs:[eax]

:004A13D0 648920                   mov dword ptr fs:[eax], esp

:004A13D3 8B45F4                   mov eax, dword ptr [ebp-0C]  <==== 取第二个注册框的注册码

:004A13D6 E8317FF6FF               call 0040930C                     <====注册码运算,结果是注册码的十六进制数

:004A13DB 8BD8                     mov ebx, eax                   <====将结果存入 EBX

:004A13DD 8B45F0                   mov eax, dword ptr [ebp-10]  <==== 取第三个注册框的注册码

:004A13E0 E8277FF6FF               call 0040930C                     <====注册码运算,结果是注册码的十六进制数

:004A13E5 8BF0                     mov esi, eax                   <====将结果存入 ESi

:004A13E7 8B45EC                   mov eax, dword ptr [ebp-14]  <==== 取第四个注册框的注册码

:004A13EA E81D7FF6FF               call 0040930C                     <====注册码运算,结果是注册码的十六进制数

:004A13EF 8BF8                     mov edi, eax                   <====将结果存入 EDI

:004A13F1 8B45E8                   mov eax, dword ptr [ebp-18]  <==== 取第五个注册框的注册码

:004A13F4 E8137FF6FF               call 0040930C                     <====注册码运算,结果是注册码的十六进制数

:004A13F9 8945E4                   mov dword ptr [ebp-1C], eax  <====将结果存入[EBP-1C]

:004A13FC 33C0                     xor eax, eax

:004A13FE 5A                       pop edx

:004A13FF 59                       pop ecx

:004A1400 59                       pop ecx

:004A1401 648910                   mov dword ptr fs:[eax], edx

:004A1404 EB29                     jmp 004A142F

:004A1406 E92D2CF6FF               jmp 00404038

:004A140B 6A00                     push 00000000

:004A140D 668B0DBC164A00            mov cx, word ptr [004A16BC]

:004A1414 33D2                     xor edx, edx

* Possible StringData Ref from Code Obj ->"您输入的序列号有误，请检查输入。"

                              |

:004A1416 B8C8164A00               mov eax, 004A16C8

:004A141B E8743AF9FF               call 00434E94

:004A1420 E83F30F6FF               call 00404464

:004A1425 E941020000               jmp 004A166B

:004A142A E83530F6FF               call 00404464

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:004A1404(U)

|

:004A142F 81FBE8030000             cmp ebx, 000003E8   <====比较计算结果是否小于 3E8(即 1000,所以注册码必须大于 1000，在注册算法里面有相应的指令)

:004A1435 7C19                     jl 004A1450

:004A1437 81FEE8030000             cmp esi, 000003E8

:004A143D 7C11                     jl 004A1450

:004A143F 81FFE8030000             cmp edi, 000003E8

:004A1445 7C09                     jl 004A1450

:004A1447 817DE4E8030000            cmp dword ptr [ebp-1C], 000003E8

:004A144E 7D1A                     jge 004A146A  <====跳到注册码比较的代码

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

|:004A1435(C), :004A143D(C), :004A1445(C)

|

:004A1450 6A00                     push 00000000

:004A1452 668B0DBC164A00            mov cx, word ptr [004A16BC]

:004A1459 33D2                     xor edx, edx

* Possible StringData Ref from Code Obj ->"您输入的序列号有误，请检查输入。"

                              |

:004A145B B8C8164A00               mov eax, 004A16C8

:004A1460 E82F3AF9FF               call 00434E94

:004A1465 E901020000               jmp 004A166B

* Referenced by a (U)nconditional or (C)onditional Jump at Address:  <=====这里的代码是判断第二至第五个注册框的注册码是不是全部是相同的,如果全部相同,则注册失败

|:004A144E(C)

|
```



### 汇编代码片段

```
:004A146A 3BF3                cmp esi, ebx    <====比较第三组和第二组
:004A146C 7523                jne 004A1491    <====不相等则继续计算
:004A146E 3BFB                cmp edi, ebx    <====比较第四组和第二组
:004A1470 751F                jne 004A1491    <====不相等则继续计算
:004A1472 3B5DE4              cmp ebx, dword ptr [ebp-1C]  <=====比较第五组和第二组
:004A1475 751A                jne 004A1491    <====不相等则继续计算
:004A1477 6A00                push 00000000   <====注册失败
:004A1479 668B0DBC164A00      mov cx, word ptr [004A16BC]
:004A1480 33D2                xor edx, edx

* Possible StringData Ref from Code Obj ->"您输入的序列号有误，请检查输入。"

                                |
:004A1482 B8C8164A00           mov eax, 004A16C8
:004A1487 E8083AF9FF           call 00434E94
:004A148C E9DA010000          jmp 004A166B

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:004A146C(C), :004A1470(C), :004A1475(C)
|

:004A1491 85DB                test ebx, ebx
:004A1493 740E                je 004A14A3
:004A1495 85F6                test esi, esi
:004A1497 740A                je 004A14A3
:004A1499 85FF                test edi, edi
:004A149B 7406                je 004A14A3
:004A149D 837DE400            cmp dword ptr [ebp-1C], 00000000
:004A14A1 751A                jne 004A14BD

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:004A1493(C), :004A1497(C), :004A149B(C)
|

:004A14A3 6A00                push 00000000
:004A14A5 668B0DBC164A00      mov cx, word ptr [004A16BC]
:004A14AC 33D2                xor edx, edx

* Possible StringData Ref from Code Obj ->"您输入的序列号有误，请检查输入。"

                                |
:004A14AE B8C8164A00          mov eax, 004A16C8
:004A14B3 E8DC39F9FF          call 00434E94
:004A14B8 E9AE010000          jmp 004A166B

* Referenced by a (U)nconditional or (C)onditional Jump at Address:  <=====计算并比较注册码
|:004A14A1(C)
|

:004A14BD 8B45E4              mov eax, dword ptr [ebp-1C]  <====从这里可以整理出计算公式:(第二组+第五组)-第四组=第二组
:004A14C0 03C3                add eax, ebx                 <====有这个公式你就可以编出注册机了
:004A14C2 2BC7                sub eax, edi
:004A14C4 3BF0                cmp esi, eax
:004A14C6 741A                je 004A14E2
:004A14C8 6A00                push 00000000
:004A14CA 668B0DBC164A00      mov cx, word ptr [004A16BC]
:004A14D1 33D2                xor edx, edx

* Possible StringData Ref from Code Obj ->"您输入的序列号有误，请检查输入。"

                                |
:004A14D3 B8C8164A00          mov eax, 004A16C8
:004A14D8 E8B739F9FF          call 00434E94
:004A14DD E989010000          jmp 004A166B
```

### Pascal 代码片段

```pascal
procedure TForm1.Button3Click(Sender: TObject);

label

  loop1;

const

  key1='KYNT';

var

  key2_4_5: array [1..3] of integer;

  key3:integer;

  i:integer;

  ran:integer;

begin

loop1:  for i:=1 to 3 do
        begin
          repeat
          ran:=random(9999);
        until ran>1000;
          key2_4_5[i]:=ran;
          end;
          key3:=(key2_4_5[1]+key2_4_5[3])-key2_4_5[2];
          if key3<1000 then goto  loop1;
          if key3>9999 then goto  loop1;
          edit1.Text :=key1;
          edit2.Text :=inttostr(key2_4_5[1]);
          edit3.Text:=inttostr(key3);
          edit4.Text :=inttostr(key2_4_5[2]);
          edit5.Text :=inttostr(key2_4_5[3]);

end;
```

`4dd466d0e1e0cc123afbe504da2462ca`
