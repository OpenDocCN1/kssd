
# 完美卸载6.0注册算法分析 (2千字)

`M1`,`M2`,`M3`,`M4`=======>魔术数组

`R1`-`R2`-`R3`-`R4`=======>预注册码

`C1`-`C2`-`C3`-`C4`-`C5`====>注册码

=============================================================

`C5`随机产生,但必须在[1-$8C]之内

```
:10003CB3 8B8424C4020000          mov eax, dword ptr
[esp+000002C4]========>C5
:10003CBA 8B8C24B4020000          mov ecx, dword ptr
[esp+000002B4]========>C1
:10003CC1 8B6C242C                mov
ebp, dword ptr [esp+2C]==============>R2
:10003CC5 83C410
add esp, 00000010
:10003CC8 8B148530AB0110          mov edx, dword ptr
[4*eax+1001AB30]======>M1
:10003CCF 2BCD
  sub ecx, ebp
:10003CD1 3BD1
  cmp edx, ecx===================>C1=M1[C5]+R2
:10003CD3 0F851B020000            jne 10003EF4==========>错误
:10003CD9 8B9424A8020000          mov edx, dword ptr
[esp+000002A8]========>C2
:10003CE0 8B742418                mov
esi, dword ptr [esp+18]==============>R1
:10003CE4 8B0C85FCA70110          mov ecx, dword ptr
[4*eax+1001A7FC]======>M2
:10003CEB 2BD6
  sub edx, esi
:10003CED 3BCA
  cmp ecx, edx===================>C2=M2[C5]+R1
:10003CEF 0F85FF010000            jne 10003EF4
:10003CF5 8B8C24AC020000          mov ecx, dword ptr
[esp+000002AC]========>C3
:10003CFC 8B742410                mov
esi, dword ptr [esp+10]==============>R4
:10003D00 8B1485C8A40110          mov edx, dword ptr
[4*eax+1001A4C8]======>M3
:10003D07 2BCE
  sub ecx, esi
:10003D09 3BD1
  cmp edx, ecx===================>C3=M3[C5]+R4
:10003D0B 0F85E3010000            jne 10003EF4
:10003D11 8B9424B0020000          mov edx, dword ptr
[esp+000002B0]========>C4
:10003D18 8B742414                mov
esi, dword ptr [esp+14]==============>R3
:10003D1C 8B0C8594A10110          mov ecx, dword ptr
[4*eax+1001A194]======>M4
:10003D23 2BD6
  sub edx, esi
:10003D25 3BCA
  cmp ecx, edx===================>C4=M4[C5]+R3
:10003D27 0F85C7010000            jne 10003EF4
```

.....下面的，我没去看了..............

.....应该还有蹊跷,我没时间看啦！....注册器已经写好.....
