
# 美萍安全卫士V8.45注册机制作分析过程，及注册机！ (11千字)

```
:00402BF0 691540C04A0005840808    imul edx, dword ptr [004AC040],
08088405
:00402BFA 42
  inc edx
:00402BFB 891540C04A00            mov dword ptr
[004AC040], edx
:00402C01 F7E2
  mul edx
:00402C03 89D0
  mov eax, edx
:00402C05 C3
  ret
([4ac040]*8088405+1)*edx取高16位
------------------------------------------------------------------------
* Referenced by a CALL at Addresses:
|:00475A02  , :00475A86
|
:00474EC4 55
  push ebp
:00474EC5 8BEC
  mov ebp, esp
:00474EC7 83C4F0
add esp, FFFFFFF0
:00474ECA 53
  push ebx
:00474ECB 56
  push esi
:00474ECC 57
  push edi
:00474ECD 33D2
  xor edx, edx
:00474ECF 8955F4
mov dword ptr [ebp-0C], edx
:00474ED2 8BD8
  mov ebx, eax
:00474ED4 33C0
  xor eax, eax
:00474ED6 55
  push ebp
* Possible StringData Ref from Code Obj ->"殂鹈^]"
            |
:00474ED7 68D8504700              push 004750D8
:00474EDC 64FF30
push dword ptr fs:[eax]
:00474EDF 648920
mov dword ptr fs:[eax], esp
:00474EE2 BE3D010000              mov esi,
0000013D
* Possible StringData Ref from Data Obj ->"鸟I"
            |
:00474EE7 8B0DB0AF4A00            mov ecx, dword
ptr [004AAFB0]
:00474EED 8B09
  mov ecx, dword ptr [ecx]
:00474EEF 8B15E8AD4A00            mov edx, dword
ptr [004AADE8]
:00474EF5 8B12
  mov edx, dword ptr [edx]
:00474EF7 8D45F4
lea eax, dword ptr [ebp-0C]
:00474EFA E819EEF8FF              call 00403D18
:00474EFF 8B45F4
mov eax, dword ptr [ebp-0C]
:00474F02 E8152CF9FF              call 00407B1C---------->这里不知是搞什么的，大侠指点
:00474F07 84C0
  test al, al
:00474F09 7405
  je 00474F10
:00474F0B BE3D020000              mov esi,
0000023D
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00474F09(C)
|
:00474F10 8BC3
  mov eax, ebx
:00474F12 99
  cdq
:00474F13 F7FE
  idiv esi----------->机器号/13D
:00474F15 A1D0AF4A00              mov eax,
dword ptr [004AAFD0]
:00474F1A 8910
  mov dword ptr [eax], edx---->取余数
:00474F1C B84A100000              mov eax,
0000104A
:00474F21 E8CADCF8FF              call 00402BF0--->上余数与8088405相乘在加1再与104a相乘
:00474F26 8BF0
  mov esi, eax----->取上结果的高16位
:00474F28 B8C6160000              mov eax,
000016C6------->this
:00474F2D E8BEDCF8FF              call 00402BF0
:00474F32 03F0
  add esi, eax------>上两个余数相加
:00474F34 8975F0
mov dword ptr [ebp-10], esi
:00474F37 DB45F0
fild dword ptr [ebp-10]---->取结果到浮点寄存器st0
:00474F3A D80D449E4A00            fmul dword ptr
[004A9E44]--->st0*1.172367
:00474F40 83C4FC
add esp, FFFFFFFC
:00474F43 D91C24
fstp dword ptr [esp]
:00474F46 9B
  wait
:00474F47 E848D0FFFF              call 00471F94----->上结果的取整数部分
:00474F4C 8945F0
mov dword ptr [ebp-10], eax
:00474F4F DB45F0
fild dword ptr [ebp-10]
:00474F52 D80D449E4A00            fmul dword ptr
[004A9E44]--->乘1。172367
:00474F58 83C4FC
add esp, FFFFFFFC
:00474F5B D91C24
fstp dword ptr [esp]
:00474F5E 9B
  wait
:00474F5F E830D0FFFF              call 00471F94--------->取整数部分
:00474F64 8BF0
  mov esi, eax--------->这是注册号的一部分，共四部分求和。后面不详细分析了，自己看看吧~！
:00474F66 8BC3
  mov eax, ebx
:00474F68 B911000000              mov ecx,
00000011
:00474F6D 99
  cdq
:00474F6E F7F9
  idiv ecx
:00474F70 83F213
xor edx, 00000013
:00474F73 8955F0
mov dword ptr [ebp-10], edx
:00474F76 DB45F0
fild dword ptr [ebp-10]
:00474F79 D80D489E4A00            fmul dword ptr
[004A9E48]-->乘3。712
:00474F7F 83C4FC
add esp, FFFFFFFC
:00474F82 D91C24
fstp dword ptr [esp]
:00474F85 9B
  wait
:00474F86 E809D0FFFF              call 00471F94
:00474F8B 8B15D0AF4A00            mov edx, dword
ptr [004AAFD0]
:00474F91 8902
  mov dword ptr [edx], eax
:00474F93 B8770F0000              mov eax,
00000F77
:00474F98 E853DCF8FF              call 00402BF0
:00474F9D 8BF8
  mov edi, eax
:00474F9F B8E1210000              mov eax,
000021E1
:00474FA4 E847DCF8FF              call 00402BF0
:00474FA9 03F8
  add edi, eax
:00474FAB B817000000              mov eax,
00000017
:00474FB0 E83BDCF8FF              call 00402BF0
:00474FB5 33F8
  xor edi, eax--->edi中为第二部分
:00474FB7 D905409E4A00            fld dword ptr
[004A9E40]-->13.7757
:00474FBD D805449E4A00            fadd dword ptr
[004A9E44]--->1。172367
:00474FC3 D805489E4A00            fadd dword ptr
[004A9E48]---->3.712三个求和:00474FC9 8BC3
        mov eax, ebx
:00474FCB B90F000000              mov ecx,
0000000F
:00474FD0 99
  cdq
:00474FD1 F7F9
  idiv ecx
:00474FD3 83F20D
xor edx, 0000000D
:00474FD6 8955F0
mov dword ptr [ebp-10], edx
:00474FD9 DB45F0
fild dword ptr [ebp-10]
:00474FDC DEC1
  faddp st(1), st(0)
:00474FDE 83C4FC
add esp, FFFFFFFC
:00474FE1 D91C24
fstp dword ptr [esp]
:00474FE4 9B
  wait
:00474FE5 E8AACFFFFF              call 00471F94
:00474FEA 8B15D0AF4A00            mov edx, dword
ptr [004AAFD0]
:00474FF0 8902
  mov dword ptr [edx], eax
:00474FF2 B88B0F0000              mov eax,
00000F8B
:00474FF7 E8F4DBF8FF              call 00402BF0
:00474FFC 50
  push eax
:00474FFD B8691C0000              mov eax,
00001C69
:00475002 E8E9DBF8FF              call 00402BF0
:00475007 5A
  pop edx
:00475008 03D0
  add edx, eax
:0047500A 52
  push edx
:0047500B B8590E0000              mov eax,
00000E59
:00475010 E8DBDBF8FF              call 00402BF0
:00475015 5A
  pop edx
:00475016 03D0
  add edx, eax
:00475018 52
  push edx
:00475019 B8EF0D0000              mov eax,
00000DEF
:0047501E E8CDDBF8FF              call 00402BF0
:00475023 5A
  pop edx
:00475024 03D0
  add edx, eax
:00475026 52
  push edx
:00475027 B872210000              mov eax,
00002172
:0047502C E8BFDBF8FF              call 00402BF0
:00475031 5A
  pop edx
:00475032 03D0
  add edx, eax
:00475034 8955FC
mov dword ptr [ebp-04], edx--->edx中为第三部分
:00475037 D905E8504700            fld dword ptr
[004750E8]
:0047503D D80D449E4A00            fmul dword ptr
[004A9E44]
:00475043 83C4FC
add esp, FFFFFFFC
:00475046 D91C24
fstp dword ptr [esp]
:00475049 9B
  wait
:0047504A E845CFFFFF              call 00471F94
:0047504F 50
  push eax
:00475050 8BC3
  mov eax, ebx
:00475052 5A
  pop edx
:00475053 8BCA
  mov ecx, edx
:00475055 99
  cdq
:00475056 F7F9
  idiv ecx
:00475058 8BDA
  mov ebx, edx
:0047505A B8B4000000              mov eax,
000000B4
:0047505F E88CDBF8FF              call 00402BF0
:00475064 8945F0
mov dword ptr [ebp-10], eax
:00475067 DB45F0
fild dword ptr [ebp-10]
:0047506A E885D9F8FF              call 004029F4---->求正弦值
:0047506F D80DEC504700            fmul dword ptr
[004750EC]--->结果乘100
:00475075 E8A6D9F8FF              call 00402A20
:0047507A 8945F8
mov dword ptr [ebp-08], eax
:0047507D 895DF0
mov dword ptr [ebp-10], ebx
:00475080 DB45F0
fild dword ptr [ebp-10]
:00475083 D80D409E4A00            fmul dword ptr
[004A9E40]
:00475089 83C4FC
add esp, FFFFFFFC
:0047508C D91C24
fstp dword ptr [esp]
:0047508F 9B
  wait
:00475090 E8FFCEFFFF              call 00471F94
:00475095 0345F8
add eax, dword ptr [ebp-08]
:00475098 8B15D0AF4A00            mov edx, dword
ptr [004AAFD0]
:0047509E 8902
  mov dword ptr [edx], eax
:004750A0 B84F0F0000              mov eax,
00000F4F
:004750A5 E846DBF8FF              call 00402BF0
:004750AA 8BD8
  mov ebx, eax
:004750AC B8D11D0000              mov eax,
00001DD1
:004750B1 E83ADBF8FF              call 00402BF0
:004750B6 03D8
  add ebx, eax--->ebx中为第四部分。
:004750B8 8BC3
  mov eax, ebx
:004750BA 8D1C37
lea ebx, dword ptr [edi+esi]
:004750BD 035DFC
add ebx, dword ptr [ebp-04]
:004750C0 03D8
  add ebx, eax--->这三条指令是求和
:004750C2 33C0
  xor eax, eax
:004750C4 5A
  pop edx
:004750C5 59
  pop ecx
:004750C6 59
  pop ecx
:004750C7 648910
mov dword ptr fs:[eax], edx
* Possible StringData Ref from Code Obj ->"\_[迕"
            |
:004750CA 68DF504700              push 004750DF
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004750DD(U)
|
:004750CF 8D45F4
lea eax, dword ptr [ebp-0C]
```
