
### 反汇编代码与注释

```
004882C8   |.  75 16                   |jnz short LeapFTP.004882E0           与的结果不为零，接受检查，看其 ASCII 码是否在 40 和 5B 之间
004882CA   |.  8B45 FC                 |mov eax,dword ptr ss:[ebp-4]
004882CD   |.  8A4418 FF               |mov al,byte ptr ds:[eax+ebx-1]
004882D1   |.  E8 4EFFFFFF             |call LeapFTP.00488224              为零时，如第 4 位，到这里接受特别检查
004882D6   |.  84C0                    |test al,al
004882D8   |.  0F84 C5000000           |je LeapFTP.004883A3
004882DE   |.  EB 22                   |jmp short LeapFTP.00488302
004882E0   |>  8BC3                    |mov eax,ebx
004882E2   |.  B9 05000000             |mov ecx,5                          到这里
004882E7   |.  99                       |cdq                              双字变四字，扩展到 EDX，EDX 用来保存 Idiv 的余数
004882E8   |.  F7F9                     |idiv ecx                         EAX idiv EDX
004882EA   |.  85D2                    |test edx,edx                     余数作与运算，修改 Flag？？，为零就跳！
004882EC   |.  74 14                   |je short LeapFTP.00488302        1 idiv 5  余数是 1
004882EE   |.  8B45 FC                 |mov eax,dword ptr ss:[ebp-4]      假码[ebp-4]＝1234-4321-5678-8765
004882F1   |.  8A4418 FF               |mov al,byte ptr ds:[eax+ebx-1]     第一次：[eax+ebx-1]指向第一位假码 1 的 ASCII 码 31
004882F5   |.  E8 3EFFFFFF             |call LeapFTP.00488238             在这个 Call 里，它与 40，5B 比较，jb 40 会使 EAX xor EAX ,返回零   ，40 就是@，41 是大写字母 A  ,5A 就是大写的 Z ，5B 就是[
004882FA   |.  84C0                    |test al,al
004882FC   |.  0F84 A1000000           |je LeapFTP.004883A3              LeapFTP.004883A3 这可是死地，由此可见，注册码不应该是数字！而是位于 40 和 5B 之间的字母！
00488302   |>  8B45 FC                 |mov eax,dword ptr ss:[ebp-4]
00488305   |.  8A4418 FF               |mov al,byte ptr ds:[eax+ebx-1]
00488309   |.  3C 2D                   |cmp al,2D                       是不是连接符
0048830B   |.  74 2D                   |je short LeapFTP.0048833A        到循环尾部，连接符的 ASCII 当然不能累加，跳到循环尾部
0048830D   |.  83FB 05                 |cmp ebx,5                       看看是不是第五位，是第五位了，那就意味着要处理第二部分注册码了
00488310   |.  7D 0C                   |jge short LeapFTP.0048831E       是第五位，则到 0048831E 处理：第二部分注册码
00488312   |.  8B55 FC                 |mov edx,dword ptr ss:[ebp-4]
00488315   |.  25 FF000000            |and eax,0FF                     屏蔽掉高四位，只取得低四位，这里是 31
0048831A   |.  03F0                    |add esi,eax                   开始累加
0048831C   |.  EB 1C                   |jmp short LeapFTP.0048833A      处理结束，到循环尾部
0048831E   |>  83FB 0A                 |cmp ebx,0A                     是第十位吗？
00488321   |.  7D 0C                   |jge short LeapFTP.0048832F      是第十位，则到 0048832F，处理第三部分注册码
00488323   |.  8B55 FC                 |mov edx,dword ptr ss:[ebp-4]
00488326   |.  25 FF000000            |and eax,0FF
0048832B   |.  03F8                    |add edi,eax
0048832D   |.  EB 0B                   |jmp short LeapFTP.0048833A
0048832F   |>  8B55 FC                 |mov edx,dword ptr ss:[ebp-4]
00488332   |.  25 FF000000            |and eax,0FF
00488337   |.  0145 F4                 |add dword ptr ss:[ebp-C],eax
0048833A   |>  43                      |inc ebx                         EBX 自加 1，指向下一位注册码 2
0048833B   |.  83FB 0F                 |cmp ebx,0F                      EBX 的值与 0F 相比
0048833E   |.^ 0F85 74FFFFFF           \\jnz LeapFTP.004882B8           1 cmp F ,还没到 15！继续循环。
00488344   |.  8D0C37                  |lea ecx,dword ptr ds:[edi+esi]    是第十五位了吗？是的，那么就 EDI+ESI 存于 ECX
00488347   |.  034D F4                 |add ecx,dword ptr ss:[ebp-C]     把第三部分注册码的累加值也加起来，存于 ECX
0048834A   |.  8BC6                    |mov eax,esi                     第一部分累加值送到 EAX
0048834C   |.  BB 1A000000            |mov ebx,1A                      EBX=1A
00488351   |.  99                       |cdq
00488352   |.  F7FB                     |idiv ebx                      取余
00488354   |.  83C2 41                 |add edx,41                    余数再加上 41，就变成字母
00488357   |.  8B45 FC                 |mov eax,dword ptr ss:[ebp-4]
0048835A   |.  3A50 0F                 |cmp dl,byte ptr ds:[eax+F]      与第四部分假码的第一个比较
0048835D   |.  75 44                   |jnz short LeapFTP.004883A3      相等就 OK！继续下一步。否则就玩完！  爆破点 1,Nop 掉
0048835F   |.  8BC7                    |mov eax,edi                     第二部分的累加值送到 EAX
00488361   |.  BB 1A000000            |mov ebx,1A
00488366   |.  99                       |cdq
00488367   |.  F7FB                     |idiv ebx
00488369   |.  83C2 41                 |add edx,41                    同样的操作
0048836C   |.  8B45 FC                 |mov eax,dword ptr ss:[ebp-4]
0048836F   |.  3A50 10                 |cmp dl,byte ptr ds:[eax+10]     与第四部分的第二个相比
00488372   |.  75 2F                   |jnz short LeapFTP.004883A3                                  爆破点 2，Nop 掉
00488374   |.  8B45 F4                 |mov eax,dword ptr ss:[ebp-C]
00488377   |.  BB 1A000000            |mov ebx,1A
0048837C   |.  99                       |cdq
0048837D   |.  F7FB                     |idiv ebx
0048837F   |.  83C2 41                 |add edx,41
00488382   |.  8B45 FC                 |mov eax,dword ptr ss:[ebp-4]
00488385   |.  3A50 11                 |cmp dl,byte ptr ds:[eax+11]
00488388   |.  75 19                   |jnz short LeapFTP.004883A3                               爆破点 3，Nop 掉
0048838A   |.  8BC1                    |mov eax,ecx
0048838C   |.  B9 1A000000            |mov ecx,1A
00488391   |.  99                       |cdq
00488392   |.  F7F9                     |idiv ecx
00488394   |.  83C2 41                 |add edx,41
00488397   |.  8B45 FC                 |mov eax,dword ptr ss:[ebp-4]
0048839A   |.  3A50 12                 |cmp dl,byte ptr ds:[eax+12]
0048839D   |.  75 04                   |jnz short LeapFTP.004883A3                             爆破点 4，Nop 掉，然后，输入合法的假码即可，不过为什么要爆破呢？
0048839F   |.  C645 FB 01              |mov byte ptr ss:[ebp-5],1         一路过关，之后，给标志赋值！！
004883A3   |>  33C0                    |xor eax,eax                       循环的尾部
004883A5   |.  5A                       |pop edx
004883A6   |.  59                       |pop ecx
004883A7   |.  59                       |pop ecx
004883A8   |.  64:8910                  |mov dword ptr fs:[eax],edx
004883AB   |.  68 C0834800             |push LeapFTP.004883C0
004883B0   |>  8D45 FC                 |lea eax,dword ptr ss:[ebp-4]
004883B3   |.  E8 3CB9F7FF             |call LeapFTP.00403CF4          这个 Call 将标志 byte ptr ss:[ebp-5]中的值转送到 AL
004883B8   |\.  C3                      |retn                          返回去测试标志值！！
```

\*\*\*\*\*\*\*\*这个 Call 将标志 byte ptr ss:[ebp-5]中的值转送到 AL\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

```
004883C0    .  8A45 FB                 |mov al,byte ptr ss:[ebp-5]
004883C3    .  5F                       |pop edi
004883C4    .  5E                       |pop esi
004883C5    .  5B                       |pop ebx
```
