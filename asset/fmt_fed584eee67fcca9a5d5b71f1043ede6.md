
004455A8 6C 64 79 77 65 6C 45 62 4A 56 5A 37 58 34 56 4E ldyselEbJVZ7X4VN
004455B8 31 45 4C 4D 40 31 66 45 45 48 58 4E 21 78 71 21 1ELM@1fEEHXN!xq!
004455C8 68 61 46 73 48 53 3F 4B 66 40 23 45 62 76 63 45 haFsHS?Kf@#EbvcE
004455D8 7A 46 23 2D 2B 21 47 35 21 53 7C 3D 3D 24 2F 40 zF#-+!G5!S|==$/@
004455E8 21 2F 39 35 33 36 46 40 48 4A 7C 63 2D 78 67 34 !/9536F@HJ|c-xg4
004455F8 31 2D 34 3D 35 67 6A 2F 6A 2B 40 30 30 43 2A 2D 1-4=5gj/j+@00C*-
00445608 24 21 6E 33 33 3D 23 39 7C 32 3D 35 58 5A 2A 24 $!n33=#9|2=5XZ*$
00445618 7C 33 43 42 4D 41 53 44 |3CBMASD

☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆

依次所取的字符是：`r`、`1`、`j`、`d`、`D`、`l`、`t`、`r`、`c`、`R`

```asm
:0040C0A6 50                   push eax
:0040C0A7 E880E40100           call 0042A52C
:0040C0AC 8B442410             mov eax, dword ptr [esp+10]
:0040C0B0 46                   inc esi
:0040C0B1 3B70F8               cmp esi, dword ptr [eax-08]
:0040C0B4 7C89                 jl 0040C03F
                               ==== > 循环 10 次

; Referenced by a (U)nconditional or (C)onditional Jump at Address:
; :0040C03D(C)

:0040C0B6 8B48F8               mov ecx, dword ptr [eax-08]
:0040C0B9 33F6                 xor esi, esi
:0040C0BB 85C9                 test ecx, ecx
:0040C0BD 7E77                 jle 0040C136

; Referenced by a (U)nconditional or (C)onditional Jump at Address:
; :0040C134(C)

:0040C0BF 8A0C06               mov cl, byte ptr [esi+eax]
                               ==== > CL = 依次正序取 3208081252 字符的 HEX 值
:0040C0C2 6A01                 push 00000001
:0040C0C4 884C2444             mov byte ptr [esp+44], cl
:0040C0C8 8B542444             mov edx, dword ptr [esp+44]
:0040C0CC 52                   push edx
:0040C0CD 8D4C242C             lea ecx, dword ptr [esp+2C]
:0040C0D1 E8E7840100           call 004245BD
:0040C0D6 8B00                 mov eax, dword ptr [eax]
:0040C0D8 50                   push eax
:0040C0D9 FFD7                 call edi
                               ==== > 把所取字符的 HEX 值转化为数字，即：-30
:0040C0DB 8D4C2424             lea ecx, dword ptr [esp+24]
:0040C0DF 8BE8                 mov ebp, eax
                               ==== > EBP = EAX = 依次正序所取的数字
:0040C0E1 E843E00100           call 0042A129
:0040C0E6 8B4C2410             mov ecx, dword ptr [esp+10]
:0040C0EA 6A01                 push 00000001
:0040C0EC 8B41F8               mov eax, dword ptr [ecx-08]
                               ==== > 取 3208081252 长度 EAX = A
:0040C0EF 2BC6                 sub eax, esi
:0040C0F1 8A4408FF             mov al, byte ptr [eax+ecx-01]
                               ==== > AL = 依次倒序取 3208081252 字符的 HEX 值
:0040C0F5 88442424             mov byte ptr [esp+24], al
:0040C0F9 8B4C2424             mov ecx, dword ptr [esp+24]
:0040C0FD 51                   push ecx
:0040C0FE 8D4C2424             lea ecx, dword ptr [esp+24]
:0040C102 E8B6840100           call 004245BD
:0040C107 8B00                 mov eax, dword ptr [eax]
:0040C109 50                   push eax
:0040C10A FFD7                 call edi
                               ==== > 把所取字符的 HEX 值转化为数字，即：-30
:0040C10C 8D4C241C             lea ecx, dword ptr [esp+1C]
:0040C110 8BD8                 mov ebx, eax
                               ==== > EBX = EAX = 依次倒序所取的数字
:0040C112 E812E00100           call 0042A129
:0040C117 8D54AD00             lea edx, dword ptr [ebp+4*ebp]
                               ==== > EDX = 依次正序所取的数字 * 5
:0040C11B 8D4C2418             lea ecx, dword ptr [esp+18]
:0040C11F 8A8453BC554400       mov al, byte ptr [ebx+2*edx+004455BC]
                               ==== > AL = 以（依次倒序所取的数字+依次正序所取的数字 * 5 * 2）为指针从 [004455BC] 的表中取值！
```

☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆

`[004455BC]`内存中是一张表：

004455BC 40 31 66 45 45 48 58 4E 21 78 71 21 68 61 46 73 @1fEEHXN!xq!haFs
004455CC 48 53 3F 4B 66 40 23 45 62 76 63 45 7A 46 23 2D HS?Kf@#EbvcEzF#-
004455DC 2B 21 47 35 21 53 7C 3D 3D 24 2F 40 21 2F 39 35 +!G5!S|==$/@!/95
004455EC 33 36 46 40 48 4A 7C 63 2D 78 67 34 31 2D 34 3D 36F@HJ|c-xg41-4=
004455FC 35 67 6A 2F 6A 2B 40 30 30 43 2A 2D 24 21 6E 33 5gj/j+@00C*-$!n3
0044560C 33 3D 23 39 7C 32 3D 35 58 5A 2A 24 7C 33 43 42 3=#9|2=5XZ*$|3CB
0044561C 4D 41 53 44 MASD

☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆

依次所取的字符是：`+`、`v`、`f`、`=`、`!`、`3`、`？`、`f`、`H`、`E`

```asm
:0040C126 50                   push eax
:0040C127 E800E40100           call 0042A52C
:0040C12C 8B442410             mov eax, dword ptr [esp+10]
:0040C130 46                   inc esi
:0040C131 3B70F8               cmp esi, dword ptr [eax-08]
:0040C134 7C89                 jl 0040C0BF
                               ==== > 循环 10 次

; Referenced by a (U)nconditional or (C)onditional Jump at Address:
; :0040C0BD(C)

:0040C136 8D4C2414             lea ecx, dword ptr [esp+14]
:0040C13A 684C524400           push 0044524C
:0040C13F 8D542428             lea edx, dword ptr [esp+28]
:0040C143 51                   push ecx
:0040C144 52                   push edx
:0040C145 E874E20100           call 0042A3BE
                               ==== > 在第一组字符后插入 -
:0040C14A 8D4C2418             lea ecx, dword ptr [esp+18]
:0040C14E 8D542440             lea edx, dword ptr [esp+40]
:0040C152 51                   push ecx
:0040C153 B304                 mov bl, 04
:0040C155 50                   push eax
                               ==== > EAX = r1jdDltrcR-
:0040C156 52                   push edx
:0040C157 885C2440             mov byte ptr [esp+40], bl
:0040C15B E8F8E10100           call 0042A358
                               ==== > 将以上 2 次大循环所得的字符连接起来（中间已插入 -）
:0040C160 50                   push eax
                               ==== > EAX = r1jdDltrcR-+vf=!3?fHE 这就是我的注册码了！
:0040C161 8D4C2418             lea ecx, dword ptr [esp+18]
:0040C165 C644243805           mov [esp+38], 05
:0040C16A E8F3E00100           call 0042A262
:0040C16F 8D4C2440             lea ecx, dword ptr [esp+40]
:0040C173 885C2434             mov byte ptr [esp+34], bl
:0040C177 E8ADDF0100           call 0042A129
:0040C17C 8D4C2424             lea ecx, dword ptr [esp+24]
:0040C180 C644243403           mov [esp+34], 03
:0040C185 E89FDF0100           call 0042A129
:0040C18A 8B74243C             mov esi, dword ptr [esp+3C]
:0040C18E 8D442414             lea eax, dword ptr [esp+14]
:0040C192 50                   push eax
:0040C193 8BCE                 mov ecx, esi
:0040C195 E804DD0100           call 00429E9E
:0040C19A C744242801000000     mov [esp+28], 00000001
:0040C1A2 8D4C2410             lea ecx, dword ptr [esp+10]
:0040C1A6 C644243402           mov [esp+34], 02
:0040C1AB E879DF0100           call 0042A129
:0040C1B0 8D4C2418             lea ecx, dword ptr [esp+18]
:0040C1B4 C644243401           mov [esp+34], 01
:0040C1B9 E86BDF0100           call 0042A129
:0040C1BE 8D4C2414             lea ecx, dword ptr [esp+14]
:0040C1C2 C644243400           mov [esp+34], 00
:0040C1C7 E85DDF0100           call 0042A129
:0040C1CC 8B4C242C             mov ecx, dword ptr [esp+2C]
:0040C1D0 8BC6                 mov eax, esi
:0040C1D2 5F                   pop edi
:0040C1D3 5E                   pop esi
:0040C1D4 5D                   pop ebp
:0040C1D5 5B                   pop ebx
:0040C1D6 64890D00000000       mov dword ptr fs:[00000000], ecx
:0040C1DD 83C428               add esp, 00000028
```
