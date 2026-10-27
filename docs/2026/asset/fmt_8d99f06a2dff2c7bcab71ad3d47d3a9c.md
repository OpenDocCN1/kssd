
# protectz v1.53

下载：http://newhua.infosail.com/down/protectz.exe

```
 \* Referenced by a CALL at Address:
 |:0048926F
 |
 :00488F8C 55
   push ebp
 :00488F8D 8BEC
      mov ebp, esp
 :00488F8F 83C4B0
          add esp, FFFFFFB0
 :00488F92 33D2
                  xor edx, edx
 :00488F94 8955B0
 mov dword ptr [ebp-50], edx
 :00488F97 8955B4
        mov dword ptr [ebp-4C], edx
 :00488F9A 8955B8
                mov dword ptr [ebp-48],
edx
 :00488F9D 8955BC
  mov dword ptr [ebp-44], edx
 :00488FA0 8955C8
          mov dword ptr [ebp-38], edx
 :00488FA3
8955C4                  mov dword
ptr [ebp-3C], edx
 :00488FA6 8945FC
      mov dword ptr [ebp-04], eax
 :00488FA9 33C0
                xor eax, eax
 :00488FAB
55
push ebp
 :00488FAC 6898914800
push 00489198
 :00488FB1 64FF30
    push dword ptr fs:[eax]
 :00488FB4 648920
            mov dword ptr fs:[eax], esp
 :00488FB7
8D55C8                  lea edx,
dword ptr [ebp-38]
 :00488FBA 8B45FC
      mov eax, dword ptr [ebp-04]
 :00488FBD 8B8004030000
          mov eax, dword ptr [eax+00000304]
 :00488FC3
E8687EFAFF              call 00430E30<---------------------------------读取注册码
 :00488FC8 8B45C8
 mov eax, dword ptr [ebp-38]
 :00488FCB 50
            push eax
 :00488FCC 8D55C4
                lea edx, dword ptr [ebp-3C]
 :00488FCF 8B45FC
 mov eax, dword ptr [ebp-04]
 :00488FD2 8B8000030000
    mov eax, dword ptr [eax+00000300]
 :00488FD8 E8537EFAFF
            call 00430E30<---------------------------------读取用户名
 :00488FDD 8B45C4
 mov eax, dword ptr [ebp-3C]
 :00488FE0 5A
            pop edx
 :00488FE1 E8AEF9FFFF
            call 00488994<-----------------(1)-------------比较核心，跟入
 :00488FE6 85C0
  test eax, eax
 :00488FE8 0F8482010000
  je 00489170
 ================================================================================================
 \* Referenced by a CALL at Address:<------------------------call (1)
|:00488FE1
 |
 :00488994 55
          push ebp
 :00488995 8BEC
              mov ebp, esp
 :00488997 B905000000
            mov ecx, 00000005
 \* Referenced
by a (U)nconditional or (C)onditional Jump at Address:
 |:004889A1(C)
 |
 :0048899C 6A00
  push 00000000
 :0048899E 6A00
        push 00000000
 :004889A0 49
                dec ecx
 :004889A1
75F9                    jne
0048899C
 :004889A3 51
      push ecx
 :004889A4 8955F8
          mov dword ptr [ebp-08], edx<-------------注册码
 :004889A7 8945FC
 mov dword ptr [ebp-04], eax<-------------用户名
 :004889AA 8B45FC
              mov eax, dword ptr [ebp-04]
:004889AD E86AB6F7FF              call 0040401C
 :004889B2 8B45F8
 mov eax, dword ptr [ebp-08]
 :004889B5 E862B6F7FF
      call 0040401C
 :004889BA 33C0
            xor eax, eax
 :004889BC 55
                    push ebp
 :004889BD 68AE8B4800              push
00488BAE
 :004889C2 64FF30
    push dword ptr fs:[eax]
 :004889C5 648920
            mov dword ptr fs:[eax], esp
 :004889C8
33C0                    xor
eax, eax
 :004889CA 8945F4
    mov dword ptr [ebp-0C], eax
 :004889CD 8D45F0
            lea eax, dword ptr [ebp-10]
 :004889D0
8B55F8                  mov edx,
dword ptr [ebp-08]
 :004889D3 E8A8B2F7FF
    call 00403C80
 :004889D8 8B55F0
        mov edx, dword ptr [ebp-10]
 :004889DB B8C48B4800
            mov eax, 00488BC4
 :004889E0 E86FB7F7FF
            call 00404154<--------------------------注册码中'-'的位置
 :004889E5 8845EB
 mov byte ptr [ebp-15], al<--------------在第9位
 :004889E8 807DEB00
              cmp byte ptr [ebp-15], 00
:004889EC 0F8687010000            jbe 00488B79<---------------------------没有'-'就完了
 :004889F2 33D2
  xor edx, edx
 :004889F4 8A55EB
      mov dl, byte ptr [ebp-15]
 :004889F7 8D45F0
              lea eax, dword ptr [ebp-10]
:004889FA B901000000              mov ecx,
00000001
 :004889FF E8ACB6F7FF
 call 004040B0<--------------------------去掉注册码中的'-'
 :00488A04 8B45F0
                mov eax, dword ptr [ebp-10]
 :00488A07 E85CB4F7FF              call
00403E68<--------------------------计算注册码的长度
 :00488A0C 83F810
              cmp eax, 00000010<----------------------不包括'-',要16位
 :00488A0F 0F8564010000            jne 00488B79<---------------------------不然就完蛋
 :00488A15 8D45EC
 lea eax, dword ptr [ebp-14]
 :00488A18 E8CBB1F7FF
      call 00403BE8
 :00488A1D C645EB01
          mov [ebp-15], 01
 \* Referenced by
a (U)nconditional or (C)onditional Jump at Address:
 |:00488A60(C)
 |
 :00488A21 8D45E0
 lea eax, dword ptr [ebp-20]
 :00488A24 50
            push eax
 :00488A25 33C0
                xor eax, eax
 :00488A27
8A45EB                  mov al, byte
ptr [ebp-15]
 :00488A2A 8BD0
      mov edx, eax
 :00488A2C 03D2
            add edx, edx
 :00488A2E 4A
                    dec edx
 :00488A2F B902000000              mov
ecx, 00000002
 :00488A34 8B45F0
    mov eax, dword ptr [ebp-10]<-----------去掉'-'后,的注册码
 :00488A37
E834B6F7FF              call 00404070<-------------------------取前两位
 :00488A3C 8B45E0
 mov eax, dword ptr [ebp-20]
 :00488A3F E870FDFFFF
      call 004887B4<-------------------------转换成16进制并放在al中
 :00488A44 8BD0
 mov edx, eax<--------------------------"12"==>0x12
 :00488A46 8D45E4
                lea eax, dword ptr [ebp-1C]
 :00488A49 E842B3F7FF              call
00403D90
 :00488A4E 8B55E4
    mov edx, dword ptr [ebp-1C]
 :00488A51 8D45EC
            lea eax, dword ptr [ebp-14]
 :00488A54
E817B4F7FF              call 00403E70
:00488A59 FE45EB
 inc [ebp-15]
 :00488A5C 807DEB09
    cmp byte ptr [ebp-15], 09
 :00488A60 75BF
              jne 00488A21<--------------------------循环8次
 :00488A62 8D45F0
 lea eax, dword ptr [ebp-10]
 :00488A65 E87EB1F7FF
      call 00403BE8
 :00488A6A 8B45EC
          mov eax, dword ptr [ebp-14]
 :00488A6D
E8F6B3F7FF              call 00403E68
:00488A72 84C0
 test al, al
 :00488A74 7636
      jbe 00488AAC
 :00488A76 8845EA
          mov byte ptr [ebp-16], al
 :00488A79 C645EB01
              mov [ebp-15], 01
 \*
Referenced by a (U)nconditional or (C)onditional Jump at Address:
 |:00488AAA(C)
 |
 :00488A7D 8D45DC
  lea eax, dword ptr [ebp-24]
 :00488A80 33D2
            xor edx, edx
 :00488A82 8A55EB
                mov dl, byte ptr [ebp-15]
 :00488A85 8B4DEC
 mov ecx, dword ptr [ebp-14]
 :00488A88 8A5411FF
        mov dl, byte ptr [ecx+edx-01]
 :00488A8C 8A4DEB
                mov cl, byte ptr [ebp-15]
 :00488A8F 80C134
 add cl, 34
      12 xor (1+34)==0x27
 :00488A92 32D1
              xor dl, cl
                    34 xor (2+34)==0x02
 :00488A94 E8F7B2F7FF              call
00403D90
    56 xor (3+34)==0x61
 :00488A99 8B55DC
          mov edx, dword ptr [ebp-24]
    78 xor (4+34)==0x40
 :00488A9C 8D45F0
          lea eax, dword ptr [ebp-10]
    87 xor (5+34)==0xbe
 :00488A9F E8CCB3F7FF
        call 00403E70
            65 xor (6+34)==0x5f
 :00488AA4 FE45EB
                inc [ebp-15]
                    43 xor (7+34)==0x78
 :00488AA7 FE4DEA
 dec [ebp-16]<----------------------- 21 xor (8+34)==0x1d
 :00488AAA 75D1
                  jne 00488A7D
 \* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00488A74(C)
 |
 :00488AAC 8D45EC
      lea eax, dword ptr [ebp-14]
 :00488AAF 50
                  push eax
 :00488AB0
B9BB000000              mov ecx, 000000BB
:00488AB5 BA07000000              mov edx,
00000007
 :00488ABA 8B45F0
    mov eax, dword ptr [ebp-10]<-----------27 02 61 40 be 5f 78 1d
 :00488ABD E8AEFDFFFF              call
00488870<--------------(1.1)------计算出8位的用户名,跟入
 :00488AC2 8D45F0
                lea eax, dword ptr [ebp-10]
 :00488AC5 E81EB1F7FF              call
00403BE8
 :00488ACA 8B45FC
    mov eax, dword ptr [ebp-04]<-----------输入的用户名
 :00488ACD
E896B3F7FF              call 00403E68<-------------------------计算长度
 :00488AD2 84C0
  test al, al
 :00488AD4 764C
        jbe 00488B22
 :00488AD6 8845EA
            mov byte ptr [ebp-16], al
 :00488AD9
```
