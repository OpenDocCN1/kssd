
# HappyIcon v2.55----------极其简单，高手勿入 (7千字)

## 下载

下载：`http://newhua.ruyi.com/down/happyicone.zip`

原以为是 keyfile 保护，下下来才发现不是；顺手把它破了，极其简单，高手勿进。

## 破解过程

下断 `bpx getdlgitemtexta`，F12 一下，来到这里：

```
:0041700F  FFD6                 call esi              <--------- name: lancelot
:00417011  85C0                 test eax, eax
:00417013  7538                 jne 0041704D

========================================================================================
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
| :00417013(C)
* Possible Reference to String Resource ID=00020: "&Cancel"
:0041704D  B914000000           mov ecx, 00000014
:00417052  33C0                 xor eax, eax
:00417054  8D7C2468             lea edi, dword ptr [esp+68]
* Possible Reference to Menu: MenuID_0080
* Possible Reference to Dialog: DialogID_00BD, CONTROL_ID:0080, "??"
:00417058  6880000000           push 00000080
:0041705D  F3                   repz
:0041705E  AB                   stosd
:0041705F  8D44246C             lea eax, dword ptr [esp+6C]
:00417063  50                   push eax
* Possible Reference to Dialog: DialogID_00D8, CONTROL_ID:0461, ""
:00417064  6861040000           push 00000461
:00417069  55                   push ebp
:0041706A  FFD6                 call esi              <-------------------- first name: CCG
:0041706C  85C0                 test eax, eax
:0041706E  7538                 jne 004170A8

=======================================================================================
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
| :0041706E(C)
* Possible Reference to String Resource ID=00020: "&Cancel"
:004170A8  B914000000           mov ecx, 00000014
:004170AD  33C0                 xor eax, eax
:004170AF  8DBC24BC000000       lea edi, dword ptr [esp+000000BC]
* Possible Reference to Menu: MenuID_0080
* Possible Reference to Dialog: DialogID_00BD, CONTROL_ID:0080, "??"
:004170B6  6880000000           push 00000080
:004170BB  F3                   repz
:004170BC  AB                   stosd
:004170BD  8D8C24C0000000       lea ecx, dword ptr [esp+000000C0]
:004170C4  51                   push ecx
* Possible Reference to Dialog: DialogID_00D8, CONTROL_ID:0471, ""
:004170C5  6871040000           push 00000471
:004170CA  55                   push ebp
:004170CB  FFD6                 call esi              <---------------------- key: 43434343
:004170CD  85C0                 test eax, eax
:004170CF  7538                 jne 00417109

======================================================================================
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
| :004170CF(C)
* Reference To: USER32.wsprintfA, Ord:02B3h
:00417109  8B1D48434200         mov ebx, dword ptr [00424348]
:0041710F  8D542414             lea edx, dword ptr [esp+14]      <-------- lancelot
:00417113  52                   push edx
:00417114  8D44246C             lea eax, dword ptr [esp+6C]      <-------- CCG
* Possible StringData Ref from Data Obj -> "HappyIcon"
:00417118  6878714200           push 00427178                    <---------------------- HappyIcon
:0041711D  50                   push eax
:0041711E  8D8C24A0050000       lea ecx, dword ptr [esp+000005A0]
* Possible StringData Ref from Data Obj -> "%s%s%s"
:00417125  681C9C4200           push 00429C1C
:0041712A  51                   push ecx
:0041712B  FFD3                 call ebx
:0041712D  8DBC24A8050000       lea edi, dword ptr [esp+000005A8] <--- CCGHappyIconlancelot
:00417134  83C9FF               or ecx, FFFFFFFF
:00417137  33C0                 xor eax, eax
:00417139  83C414               add esp, 00000014
:0041713C  F2                   repnz
:0041713D  AE                   scasb
:0041713E  F7D1                 not ecx                          <---------------------------- 0x15
:00417140  2BF9                 sub edi, ecx
:00417142  8D942490010000       lea edx, dword ptr [esp+00000190]
:00417149  8BC1                 mov eax, ecx
:0041714B  8BF7                 mov esi, edi
:0041714D  8BFA                 mov edi, edx
:0041714F  C1E902               shr ecx, 02
:00417152  F3                   repz
:00417153  A5                   movsd
:00417154  8BC8                 mov ecx, eax
:00417156  8D842490010000       lea eax, dword ptr [esp+00000190]
:0041715D  83E103               and ecx, 00000003
:00417160  F3                   repz
:00417161  A4                   movsb                         <----------------------------- 这里一定要用 F8，带过，不知为什么
:00417162  8A8C2490010000       mov cl, byte ptr [esp+00000190]
:00417169  84C9                 test cl, cl                    <----------------------- 'C'==0x43
:0041716B  741F                 je 0041718C
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
| :0041718A(C)
:0041716D  80385F               cmp byte ptr [eax], 5F        <----------- '_' == 0x5f
:00417170  7503                 jne 00417175
:00417172  C60020               mov byte ptr [eax], 20        <----------- 如果是 '_' 就用 ' '（空格）代替
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
| :00417170(C)
:00417175  0FBE08               movsx ecx, byte ptr [eax]     <--------- 'C'==0x43
:00417178  334C2410             xor ecx, dword ptr [esp+10]   <-------- 0x43 XOR 0xffffffff == 0xffffffbc
:0041717C  81F1CE9A5713         xor ecx, 13579ACE             <------------ 0xffffffbc XOR 0x13579ace == 0xeca86572
:00417182  40                   inc eax
:00417183  894C2410             mov dword ptr [esp+10], ecx   <-------- 把结果放在 [esp+10]，参加下次循环
:00417187  803800               cmp byte ptr [eax], 00        <------------- 字符串未结束，则继续循环
:0041718A  75E1                 jne 0041716D                  <----------------------- 循环结束后 ecx == 0xffffffd1
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
| :0041716B(C)
:0041718C  8D9424BC000000       lea edx, dword ptr [esp+000000BC] <--- 假注册码
:00417193  52                   push edx
:00417194  E8FF290000           call 00419B98                  <----------------------- 转换为16进制
:00417199  8B4C2414             mov ecx, dword ptr [esp+14]
:0041719D  83C404               add esp, 00000004
:004171A0  81F1F0BD6824         xor ecx, 2468BDF0               <------------ 0xffffffd1 XOR 0x2468bdf0 == 0xdb974221
:004171A6  3BC1                 cmp eax, ecx                    <------------------- 比较真假注册码的16进制值
:004171A8  742E                 je 004171D8
:004171AA  6A10                 push 00000010
```

## 总结

Name: `lancelot`   First name: `CCG`   Key: `43434343`

```
CCGHappyIconlancelot:0x43,0x43,0x47,0x48,0x61,0x70,0x70,0x79,0x49,0x63,0x6f,0x6e,0x6c,0x61,0x6e,
                     0x6e,0x65,0x6c,0x6f,0x74
```

```
(1) 0x43 xor 0xffffffff == 0xffffffbc
(2) 0xffffffbc xor 0x13579ace == 0xeca86572
(1) 0x43 xor 0xeca86572 == 0xeca86531
(2) 0xeca86531 xor 0x13579ace == 0xffffffff

... ... 略 ... ...

结果：0xffffffd1
```

再把 `0xffffffd1 xor 0x2468bdf0 == 0xdb974221 == 3684123169` <----------- 这就是注册码了。

所以 Name: `lancelot`      First name: `CCG`

Key: `3684123169`

```
,;~;,

                 /\_

               (  /

          (()      //)

          | \\  ,,;;'\

    \_\_ \_(  )m=(lancelot(================--------

    /'  ' '()/~' '.(, |

       ,;(

  )||    |  ~

     ,;' \    /-(.;,

)            兰斯洛特[CCG][FCG]

    ) /      ) /

         //

      ||

      2001.09.17

         )\_\

    )\_\
```
