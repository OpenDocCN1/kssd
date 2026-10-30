
```
011F5528  |>  8B45 0C       /mov     eax, dword ptr [ebp+0xC]
011F552B  |. |0FBE08        |movsx   ecx, byte ptr [eax]             # 遇到sn[i] == 0x20空格的时候就停止
011F552E  |. |83F9 20       |cmp     ecx, 0x20
011F5531  |. |0F84 2C010000 |je      011F5663                      # 这里有个bug?
011F5537  |. |837D F8 08    |cmp     dword ptr [ebp-0x8], 0x8
011F553B  |. |75 0A         |jnz     short 011F5547
011F553D  |. |837D EC 03    |cmp     dword ptr [ebp-0x14], 0x3
011F5541  |. |0F84 0E010000 |je      011F5655
011F5547  |> |8B45 0C       |mov     eax, dword ptr [ebp+0xC]
011F554A  |. |0FBE08        |movsx   ecx, byte ptr [eax]
011F554D  |. |83F9 7A       |cmp     ecx, 0x7A
011F5550  |. |75 32         |jnz     short 011F5584                # sn[i] == 'z'
011F5552  |. |8B45 EC       |mov     eax, dword ptr [ebp-0x14]
011F5555  |. |0345 D4       |add     eax, dword ptr [ebp-0x2C]
011F5558  |. |83F8 0A       |cmp     eax, 0xA
011F555B  |. |7D 20         |jge     short 011F557D                # 不能超过行
011F555D  |. |8B45 EC       |mov     eax, dword ptr [ebp-0x14]
011F5560  |. |0345 D4       |add     eax, dword ptr [ebp-0x2C]
011F5563  |. |6BC0 28       |imul    eax, eax, 0x28                # 这里就可以看出0x28/4=0x0a，10个元素为一行的地图
011F5566  |. |0345 08       |add     eax, dword ptr [ebp+0x8]      # 定位到行
011F5569  |. |8B4D F8       |mov     ecx, dword ptr [ebp-0x8]
011F556C  |. |833C88 00     |cmp     dword ptr [eax+ecx*4], 0x0    # 定位到当前格子，为0才能走，为1就失败
011F5570  |. |75 09         |jnz     short 011F557B
011F5572  |. |8B45 EC       |mov     eax, dword ptr [ebp-0x14]
011F5575  |. |0345 D4       |add     eax, dword ptr [ebp-0x2C]     # 'z'是往下
011F5578  |. |8945 EC       |mov     dword ptr [ebp-0x14], eax
011F557B  |> |EB 07         |jmp     short 011F5584
011F557D  |> |32C0          |xor     al, al
011F557F  |. |E9 E1000000   |jmp     011F5665
011F5584  |> |8B45 0C       |mov     eax, dword ptr [ebp+0xC]
011F5587  |. |0FBE08        |movsx   ecx, byte ptr [eax]
011F558A  |. |83F9 6C       |cmp     ecx, 0x6C
011F558D  |. |75 45         |jnz     short 011F55D4                # 'l'是往右 
011F558F  |. |8B45 F8       |mov     eax, dword ptr [ebp-0x8]
011F5592  |. |0345 BC       |add     eax, dword ptr [ebp-0x44]
011F5595  |. |83F8 0A       |cmp     eax, 0xA
011F5598  |. |7D 3A         |jge     short 011F55D4
011F559A  |. |8B45 EC       |mov     eax, dword ptr [ebp-0x14]
011F559D  |. |6BC0 28       |imul    eax, eax, 0x28
011F55A0  |. |0345 08       |add     eax, dword ptr [ebp+0x8]
011F55A3  |. |8B4D F8       |mov     ecx, dword ptr [ebp-0x8]
011F55A6  |. |034D BC       |add     ecx, dword ptr [ebp-0x44]
011F55A9  |. |833C88 00     |cmp     dword ptr [eax+ecx*4], 0x0
011F55AD  |. |75 1E         |jnz     short 011F55CD
011F55AF  |. |8B45 EC       |mov     eax, dword ptr [ebp-0x14]
011F55B2  |. |6BC0 28       |imul    eax, eax, 0x28
011F55B5  |. |0345 08       |add     eax, dword ptr [ebp+0x8]
011F55B8  |. |8B4D F8       |mov     ecx, dword ptr [ebp-0x8]
011F55BB  |. |C70488 040000>|mov     dword ptr [eax+ecx*4], 0x4
011F55C2  |. |8B45 F8       |mov     eax, dword ptr [ebp-0x8]
011F55C5  |. |0345 BC       |add     eax, dword ptr [ebp-0x44]
011F55C8  |. |8945 F8       |mov     dword ptr [ebp-0x8], eax
011F55CB  |. |EB 07         |jmp     short 011F55D4
011F55CD  |> |32C0          |xor     al, al
011F55CF  |. |E9 91000000   |jmp     011F5665
011F55D4  |> |8B45 0C       |mov     eax, dword ptr [ebp+0xC]
011F55D7  |. |0FBE08        |movsx   ecx, byte ptr [eax]
011F55DA  |. |83F9 71       |cmp     ecx, 0x71                      # 'q'是往上
011F55DD  |. |75 3F         |jnz     short 011F561E
011F55DF  |. |8B45 EC       |mov     eax, dword ptr [ebp-0x14]
011F55E2  |. |0345 E0       |add     eax, dword ptr [ebp-0x20]
011F55E5  |. |78 37         |js      short 011F561E
011F55E7  |. |8B45 EC       |mov     eax, dword ptr [ebp-0x14]
011F55EA  |. |0345 E0       |add     eax, dword ptr [ebp-0x20]
011F55ED  |. |6BC0 28       |imul    eax, eax, 0x28
011F55F0  |. |0345 08       |add     eax, dword ptr [ebp+0x8]
011F55F3  |. |8B4D F8       |mov     ecx, dword ptr [ebp-0x8]
011F55F6  |. |833C88 00     |cmp     dword ptr [eax+ecx*4], 0x0
011F55FA  |. |75 1E         |jnz     short 011F561A
011F55FC  |. |8B45 EC       |mov     eax, dword ptr [ebp-0x14]
011F55FF  |. |6BC0 28       |imul    eax, eax, 0x28
011F5602  |. |0345 08       |add     eax, dword ptr [ebp+0x8]
011F5605  |. |8B4D F8       |mov     ecx, dword ptr [ebp-0x8]
011F5608  |. |C70488 040000>|mov     dword ptr [eax+ecx*4], 0x4
011F560F  |. |8B45 EC       |mov     eax, dword ptr [ebp-0x14]
011F5612  |. |0345 E0       |add     eax, dword ptr [ebp-0x20]
011F5615  |. |8945 EC       |mov     dword ptr [ebp-0x14], eax
011F5618  |. |EB 04         |jmp     short 011F561E
011F561A  |> |32C0          |xor     al, al
011F561C  |. |EB 47         |jmp     short 011F5665
011F561E  |> |8B45 0C       |mov     eax, dword ptr [ebp+0xC]
011F5621  |. |0FBE08        |movsx   ecx, byte ptr [eax]
011F5624  |. |83F9 70       |cmp     ecx, 0x70                     # 'p'是往左
011F5627  |. |75 2C         |jnz     short 011F5655
011F5629  |. |8B45 F8       |mov     eax, dword ptr [ebp-0x8]
011F562C  |. |0345 C8       |add     eax, dword ptr [ebp-0x38]
011F562F  |. |78 24         |js      short 011F5655
011F5631  |. |8B45 EC       |mov     eax, dword ptr [ebp-0x14]
011F5634  |. |6BC0 28       |imul    eax, eax, 0x28
011F5637  |. |0345 08       |add     eax, dword ptr [ebp+0x8]
011F563A  |. |8B4D F8       |mov     ecx, dword ptr [ebp-0x8]
011F563D  |. |034D C8       |add     ecx, dword ptr [ebp-0x38]
011F5640  |. |833C88 00     |cmp     dword ptr [eax+ecx*4], 0x0
011F5644  |. |75 0B         |jnz     short 011F5651
011F5646  |. |8B45 F8       |mov     eax, dword ptr [ebp-0x8]
011F5649  |. |0345 C8       |add     eax, dword ptr [ebp-0x38]
011F564C  |. |8945 F8       |mov     dword ptr [ebp-0x8], eax
011F564F  |. |EB 04         |jmp     short 011F5655
011F5651  |> |32C0          |xor     al, al
011F5653  |. |EB 10         |jmp     short 011F5665
011F5655  |> |8B45 0C       |mov     eax, dword ptr [ebp+0xC]
011F5658  |. |83C0 01       |add     eax, 0x1
011F565B  |. |8945 0C       |mov     dword ptr [ebp-0xC], eax
011F565E  |.^\E9 C5FEFFFF   \jmp     011F5528
011F5663  |>  B0 01         mov     al, 0x1                        # 返回1就成功了
011F5665  |>  5F            pop     edi
011F5666  |.  5E            pop     esi
011F5667  |.  5B            pop     ebx
011F5668  |.  81C4 14010000 add     esp, 0x114
011F566E  |.  3BEC          cmp     ebp, esp
011F5670  |.  E8 DC87FFFF   call    011EDE51
011F5675  |.  8BE5          mov     esp, ebp
011F5677  |.  5D            pop     ebp
011F5678  \.  C3            retn
```

## 四、最后

这不算走迷宫吧。

只要不踩 1 就成功了，也没有长度限制，没有判断最后一定要从左上角走到右上角，只是判断了输入的位置不能为 1 而已。

所以注册码只要一个`z`就成功了。

`base64`和摩斯电码都是标准的，两层`base64`：

`base64(base64(morse_code('z'))) = 'TFMwdUxpQT0='`

再加上后面 64 位的校验值（可以通过调试得到明文比较）：

`"b92a72497b685c31013347a7276f371f8cf91085ab8322009bfed2df41d94f94"`

得到一组注册码：

`TFMwdUxpQT0=b92a72497b685c31013347a7276f371f8cf91085ab8322009bfed2df41d94f94`

## 五、附

里面各种函数里都穿插着很多反调试的检测，`nop`掉后保存下来比较好调试。

完

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)
