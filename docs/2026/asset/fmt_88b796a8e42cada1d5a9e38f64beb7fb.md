
```
00417930    55              push ebp
00417931    8BEC            mov ebp,esp
00417933    83EC 24         sub esp,0x24
00417936    8B45 10         mov eax,dword ptr ss:[ebp+0x10]
00417939    8945 F4         mov dword ptr ss:[ebp-0xC],eax
0041793C    8B4D F4         mov ecx,dword ptr ss:[ebp-0xC]
0041793F    894D FC         mov dword ptr ss:[ebp-0x4],ecx
00417942    8B55 08         mov edx,dword ptr ss:[ebp+0x8]
00417945    8955 F0         mov dword ptr ss:[ebp-0x10],edx
00417948    C745 F8 0000000>mov dword ptr ss:[ebp-0x8],0x0
0041794F    C745 F8 0000000>mov dword ptr ss:[ebp-0x8],0x0
00417956    8B45 F8         mov eax,dword ptr ss:[ebp-0x8]
00417959    3B45 0C         cmp eax,dword ptr ss:[ebp+0xC]
0041795C    0F8D EF000000   jge dvdiphon.00417A51                    ; 判断是否结束
00417962    8B4D F0         mov ecx,dword ptr ss:[ebp-0x10]
00417965    034D F8         add ecx,dword ptr ss:[ebp-0x8]
00417968    0FB611          movzx edx,byte ptr ds:[ecx]
0041796B    8955 EC         mov dword ptr ss:[ebp-0x14],edx
0041796E    8B45 F8         mov eax,dword ptr ss:[ebp-0x8]
00417971    83C0 01         add eax,0x1
00417974    8945 F8         mov dword ptr ss:[ebp-0x8],eax
00417977    8B4D EC         mov ecx,dword ptr ss:[ebp-0x14]
0041797A    C1E1 08         shl ecx,0x8                              ; 第1字节左移8位
0041797D    894D EC         mov dword ptr ss:[ebp-0x14],ecx
00417980    8B55 F8         mov edx,dword ptr ss:[ebp-0x8]
00417983    3B55 0C         cmp edx,dword ptr ss:[ebp+0xC]
00417986    7D 0F           jge short dvdiphon.00417997
00417988    8B45 F0         mov eax,dword ptr ss:[ebp-0x10]
0041798B    0345 F8         add eax,dword ptr ss:[ebp-0x8]
0041798E    0FB608          movzx ecx,byte ptr ds:[eax]
00417991    034D EC         add ecx,dword ptr ss:[ebp-0x14]          ; 第2字节加上前面左移8位后的值
00417994    894D EC         mov dword ptr ss:[ebp-0x14],ecx          ; 存放
00417997    8B55 F8         mov edx,dword ptr ss:[ebp-0x8]
0041799A    83C2 01         add edx,0x1
0041799D    8955 F8         mov dword ptr ss:[ebp-0x8],edx
004179A0    8B45 EC         mov eax,dword ptr ss:[ebp-0x14]
004179A3    C1E0 08         shl eax,0x8                              ; 左移8位
004179A6    8945 EC         mov dword ptr ss:[ebp-0x14],eax
004179A9    8B4D F8         mov ecx,dword ptr ss:[ebp-0x8]
004179AC    3B4D 0C         cmp ecx,dword ptr ss:[ebp+0xC]
004179AF    7D 0F           jge short dvdiphon.004179C0
004179B1    8B55 F0         mov edx,dword ptr ss:[ebp-0x10]
004179B4    0355 F8         add edx,dword ptr ss:[ebp-0x8]
004179B7    0FB602          movzx eax,byte ptr ds:[edx]
004179BA    0345 EC         add eax,dword ptr ss:[ebp-0x14]          ; 第3字节加上前面左移8位后的值
004179BD    8945 EC         mov dword ptr ss:[ebp-0x14],eax
004179C0    8B4D F8         mov ecx,dword ptr ss:[ebp-0x8]
004179C3    83C1 01         add ecx,0x1
004179C6    894D F8         mov dword ptr ss:[ebp-0x8],ecx
004179C9    8B55 EC         mov edx,dword ptr ss:[ebp-0x14]
004179CC    81E2 0000FC00   and edx,0xFC0000                         ; 将上面计算得到的整数值进逻辑运算
004179D2    C1FA 12         sar edx,0x12                             ; 算术右移0x12 (逻辑运算后的值做为下标取字符)
004179D5    8B45 FC         mov eax,dword ptr ss:[ebp-0x4]
004179D8    8A8A 783B6100   mov cl,byte ptr ds:[edx+0x613B78]        ; ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/
004179DE    8808            mov byte ptr ds:[eax],cl                 ; 存放查找到的字符
004179E0    8B55 EC         mov edx,dword ptr ss:[ebp-0x14]
004179E3    81E2 00F00300   and edx,0x3F000
004179E9    C1FA 0C         sar edx,0xC
004179EC    8B45 FC         mov eax,dword ptr ss:[ebp-0x4]
004179EF    8A8A 783B6100   mov cl,byte ptr ds:[edx+0x613B78]        ; ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/
004179F5    8848 01         mov byte ptr ds:[eax+0x1],cl             ; 存放查找到的字符
004179F8    8B55 EC         mov edx,dword ptr ss:[ebp-0x14]
004179FB    81E2 C00F0000   and edx,0xFC0
00417A01    C1FA 06         sar edx,0x6
00417A04    8B45 FC         mov eax,dword ptr ss:[ebp-0x4]
00417A07    8A8A 783B6100   mov cl,byte ptr ds:[edx+0x613B78]        ; ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/
00417A0D    8848 02         mov byte ptr ds:[eax+0x2],cl             ; 存放查找到的字符
00417A10    8B55 EC         mov edx,dword ptr ss:[ebp-0x14]
00417A13    83E2 3F         and edx,0x3F
00417A16    8B45 FC         mov eax,dword ptr ss:[ebp-0x4]
00417A19    8A8A 783B6100   mov cl,byte ptr ds:[edx+0x613B78]        ; ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/
00417A1F    8848 03         mov byte ptr ds:[eax+0x3],cl             ; 存放查找到的字符
00417A22    8B55 F8         mov edx,dword ptr ss:[ebp-0x8]
00417A25    3B55 0C         cmp edx,dword ptr ss:[ebp+0xC]
00417A28    7E 07           jle short dvdiphon.00417A31
00417A2A    8B45 FC         mov eax,dword ptr ss:[ebp-0x4]
00417A2D    C640 03 3D      mov byte ptr ds:[eax+0x3],0x3D
00417A31    8B4D 0C         mov ecx,dword ptr ss:[ebp+0xC]
00417A34    83C1 01         add ecx,0x1
00417A37    394D F8         cmp dword ptr ss:[ebp-0x8],ecx
00417A3A    7E 07           jle short dvdiphon.00417A43
00417A3C    8B55 FC         mov edx,dword ptr ss:[ebp-0x4]
00417A3F    C642 02 3D      mov byte ptr ds:[edx+0x2],0x3D
00417A43    8B45 FC         mov eax,dword ptr ss:[ebp-0x4]
00417A46    83C0 04         add eax,0x4
00417A49    8945 FC         mov dword ptr ss:[ebp-0x4],eax
00417A4C  ^ E9 05FFFFFF     jmp dvdiphon.00417956
00417A51    8B4D FC         mov ecx,dword ptr ss:[ebp-0x4]           ; 结束
00417A54    C601 00         mov byte ptr ds:[ecx],0x0
00417A57    8B55 F4         mov edx,dword ptr ss:[ebp-0xC]
00417A5A    8955 E8         mov dword ptr ss:[ebp-0x18],edx
00417A5D    8B45 E8         mov eax,dword ptr ss:[ebp-0x18]
00417A60    83C0 01         add eax,0x1
00417A63    8945 E4         mov dword ptr ss:[ebp-0x1C],eax
00417A66    8B4D E8         mov ecx,dword ptr ss:[ebp-0x18]
00417A69    8A11            mov dl,byte ptr ds:[ecx]
00417A6B    8855 E3         mov byte ptr ss:[ebp-0x1D],dl
00417A6E    8345 E8 01      add dword ptr ss:[ebp-0x18],0x1
00417A72    807D E3 00      cmp byte ptr ss:[ebp-0x1D],0x0
00417A76  ^ 75 EE           jnz short dvdiphon.00417A66
00417A78    8B45 E8         mov eax,dword ptr ss:[ebp-0x18]
00417A7B    2B45 E4         sub eax,dword ptr ss:[ebp-0x1C]
00417A7E    8945 DC         mov dword ptr ss:[ebp-0x24],eax
00417A81    8B45 DC         mov eax,dword ptr ss:[ebp-0x24]
00417A84    8BE5            mov esp,ebp
00417A86    5D              pop ebp
```

11.将查找到的字符中小写字母转换成大写并判断是否有字符`'/'`、`'+'`，如果有就替换成`'O'`、`'E'`。
