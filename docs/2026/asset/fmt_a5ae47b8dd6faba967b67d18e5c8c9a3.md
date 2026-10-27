
```
00426FB7      C1CA 08            ror edx,8

00426FBA      81E7 FF000000      and edi,0FF

00426FC0      81E2 FF000000      and edx,0FF

00426FC6      8B2CBD A4714600    mov ebp,dword ptr ds:[edi\*4+4671A4]     ; S-Box（0）

00426FCD      8B1495 A4754600    mov edx,dword ptr ds:[edx\*4+4675A4]     ; S-Box（1）

00426FD4      33F5               xor esi,ebp

00426FD6      8BA8 98000000      mov ebp,dword ptr ds:[eax+98]

00426FDC      03D6               add edx,esi

...

004278EE      33D5               xor edx,ebp

004278F0      890B               mov dword ptr ds:[ebx],ecx              ; 第1个值保存

004278F2      8B68 08            mov ebp,dword ptr ds:[eax+8]

004278F5      2BD5               sub edx,ebp

004278F7      8953 04            mov dword ptr ds:[ebx+4],edx            ; 第2个值保存

004278FA      8B50 0C            mov edx,dword ptr ds:[eax+C]

004278FD      2BF2               sub esi,edx

004278FF      8973 08            mov dword ptr ds:[ebx+8],esi            ; 第3个值保存

00427902      8B48 10            mov ecx,dword ptr ds:[eax+10]

00427905      2BF9               sub edi,ecx

00427907      897B 0C            mov dword ptr ds:[ebx+C],edi            ; 第4个值保存

0042790A      5F                 pop edi

0042790B      5E                 pop esi

0042790C      5D                 pop ebp

0042790D      5B                 pop ebx

0042790E      83C4 08            add esp,8

00427911      C2 0800            retn 8
```

注册码加密密钥：

`K[]`＝

```
38 DE E0 3D 29 D0 64 0C B2 92 40 C6 8D 79 DA BF 48 91 DF 59 17 27 67 8C 84 BD B1 B7 63 26 CF 53

19 A6 39 0F AB EF A9 21 16 BF 77 06 23 58 08 3F 22 A9 7A F7 67 92 FE 73 33 3D 16 2E 73 F3 7B 03

0E B4 F6 38 BF 2D 86 80 42 E7 AD AA 63 1D 8D 8C 50 12 D9 C4 0F D6 F6 78 B0 18 C3 9E 5B A8 F0 5E

7A 71 CD D0 33 F7 CD D6 F5 67 D4 4C CB 74 47 F0 59 3D 4C 03 FB 0B B7 DC B3 73 7D 0A 0F 48 9C 17

E7 ED 72 60 8F 79 6E CA 28 20 29 B4 CF 09 AF DA 73 AE C9 C4 EB 51 6A 7B E6 AA 17 C9 43 01 12 1B
```

## 6. 反推注册码：

其实用户名加密算法与注册码加密算法刚好是加解密的一对。

```
call MyVideoC.00426690    // 用户名加密
call MyVideoC.00426F90    // 注册码加密
```

如果`call MyVideoC.00426690`是加密的call，那么`call MyVideoC.00426F90`就是解密的call。

只要对换2者的密钥就能互相反推。

假设用户名：`cyto`，

`call MyVideoC.00426690`后为：`5F 9A 0D 9C E9 D9 66 C4 73 23 DB D0 78 98 97 89`

假设输入的注册码为`x`，

`call MyVideoC.00426F90`后要为：`5F 9A 0D 9C E9 D9 66 C4 73 23 DB D0 78 98 97 89`

那么就用`call MyVideoC.00426690`来反推注册码，只要替换`call MyVideoC.00426690`使用的密钥就可以了，密钥换为`call MyVideoC.00426F90`使用的密钥：

```
38 DE E0 3D 29 D0 64 0C B2 92 40 C6 8D 79 DA BF 48 91 DF 59 17 27 67 8C 84 BD B1 B7 63 26 CF 53

19 A6 39 0F AB EF A9 21 16 BF 77 06 23 58 08 3F 22 A9 7A F7 67 92 FE 73 33 3D 16 2E 73 F3 7B 03

0E B4 F6 38 BF 2D 86 80 42 E7 AD AA 63 1D 8D 8C 50 12 D9 C4 0F D6 F6 78 B0 18 C3 9E 5B A8 F0 5E

7A 71 CD D0 33 F7 CD D6 F5 67 D4 4C CB 74 47 F0 59 3D 4C 03 FB 0B B7 DC B3 73 7D 0A 0F 48 9C 17

E7 ED 72 60 8F 79 6E CA 28 20 29 B4 CF 09 AF DA 73 AE C9 C4 EB 51 6A 7B E6 AA 17 C9 43 01 12 1B
```

参数：`5F 9A 0D 9C E9 D9 66 C4 73 23 DB D0 78 98 97 89`

解密后：`25 8A 13 1D CE FC 8B C7 D0 AC ED D7 0A 2E 81 D9`

即

用户名：`cyto`

注册码：`258A131DCEFC8BC7D0ACEDD70A2E81D9`

注册成功！

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)
