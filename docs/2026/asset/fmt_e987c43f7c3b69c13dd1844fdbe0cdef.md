
``` 
007770C6    33DB                    xor ebx,ebx
007770C8  ^ EB E7                   jmp short ACProtec.007770B1
007770CA    83FB 02                 cmp ebx,2
007770CD  ^ 74 BA                   je short ACProtec.00777089
007770CF    33ED                    xor ebp,ebp
007770D1    8A142F                  mov dl,byte ptr ds:[edi+ebp]
007770D4    88142E                  mov byte ptr ds:[esi+ebp],dl
007770D7    83FD 04                 cmp ebp,4
007770DA    0F84 D3000000           je ACProtec.007771B3
007770E0    45                      inc ebp
007770E1    3BC5                    cmp eax,ebp
007770E3  ^ 75 EC                   jnz short ACProtec.007770D1
007770E5    47                      inc edi
007770E6    33C0                    xor eax,eax
007770E8    83FB 01                 cmp ebx,1
007770EB  ^ 75 E4                   jnz short ACProtec.007770D1
007770ED    47                      inc edi
007770EE  ^ EB E1                   jmp short ACProtec.007770D1
007770F0    66:8B1438               mov dx,word ptr ds:[eax+edi]
007770F4    66:83C2 28              add dx,28
007770F8    8BD8                    mov ebx,eax
007770FA    43                      inc ebx
007770FB    43                      inc ebx
007770FC    66:39143B               cmp word ptr ds:[ebx+edi],dx
00777100    74 07                   je short ACProtec.00777109
00777102    83FB 08                 cmp ebx,8
00777105  ^ 72 F4                   jb short ACProtec.007770FB
00777107  ^ EB 80                   jmp short ACProtec.00777089
00777109    83F8 00                 cmp eax,0
0077710C    75 23                   jnz short ACProtec.00777131
0077710E    8D43 FE                 lea eax,dword ptr ds:[ebx-2]
00777111    83C7 02                 add edi,2
00777114    33ED                    xor ebp,ebp
00777116    8A142F                  mov dl,byte ptr ds:[edi+ebp]
00777119    88142E                  mov byte ptr ds:[esi+ebp],dl
0077711C    83FD 04                 cmp ebp,4
0077711F    0F84 8E000000           je ACProtec.007771B3
00777125    45                      inc ebp
00777126    3BC5                    cmp eax,ebp
00777128  ^ 75 EC                   jnz short ACProtec.00777116
0077712A    83C7 02                 add edi,2
0077712D    33C0                    xor eax,eax
0077712F  ^ EB E5                   jmp short ACProtec.00777116
00777131    2BD8                    sub ebx,eax
00777133    33ED                    xor ebp,ebp
00777135    8A142F                  mov dl,byte ptr ds:[edi+ebp]
00777138    88142E                  mov byte ptr ds:[esi+ebp],dl
0077713B    83FD 04                 cmp ebp,4
0077713E    74 73                   je short ACProtec.007771B3
00777140    45                      inc ebp
00777141    3BC5                    cmp eax,ebp
00777143  ^ 75 F0                   jnz short ACProtec.00777135
00777145    83C7 02                 add edi,2
00777148    33C0                    xor eax,eax
0077714A    83FB 02                 cmp ebx,2
0077714D  ^ 75 E6                   jnz short ACProtec.00777135
0077714F    83C7 02                 add edi,2
00777152  ^ EB E1                   jmp short ACProtec.00777135
00777154    66:8B1438               mov dx,word ptr ds:[eax+edi]
00777158    8BD8                    mov ebx,eax
0077715A    43                      inc ebx
0077715B    43                      inc ebx
0077715C    66:39143B               cmp word ptr ds:[ebx+edi],dx
00777160    74 0A                   je short ACProtec.0077716C
00777162    83FB 08                 cmp ebx,8
00777165  ^ 72 F4                   jb short ACProtec.0077715B
00777167  ^ E9 1DFFFFFF             jmp ACProtec.00777089
0077716C    83F8 00                 cmp eax,0
0077716F    75 1F                   jnz short ACProtec.00777190
00777171    8D43 FE                 lea eax,dword ptr ds:[ebx-2]
00777174    83C7 02                 add edi,2
00777177    33ED                    xor ebp,ebp
00777179    8A142F                  mov dl,byte ptr ds:[edi+ebp]
0077717C    88142E                  mov byte ptr ds:[esi+ebp],dl
0077717F    83FD 04                 cmp ebp,4
00777182    74 2F                   je short ACProtec.007771B3
00777184    45                      inc ebp
00777185    3BC5                    cmp eax,ebp
00777187  ^ 75 F0                   jnz short ACProtec.00777179
00777189    83C7 02                 add edi,2
0077718C    33C0                    xor eax,eax
0077718E  ^ EB E9                   jmp short ACProtec.00777179
00777190    2BD8                    sub ebx,eax
00777192    33ED                    xor ebp,ebp
00777194    8A142F                  mov dl,byte ptr ds:[edi+ebp]
00777197    88142E                  mov byte ptr ds:[esi+ebp],dl
0077719A    83FD 04                 cmp ebp,4
0077719D    74 14                   je short ACProtec.007771B3
0077719F    45                      inc ebp
007771A0    3BC5                    cmp eax,ebp
007771A2  ^ 75 F0                   jnz short ACProtec.00777194
007771A4    83C7 02                 add edi,2
007771A7    33C0                    xor eax,eax
007771A9    83FB 02                 cmp ebx,2
007771AC  ^ 75 E6                   jnz short ACProtec.00777194
007771AE    83C7 02                 add edi,2
007771B1  ^ EB E1                   jmp short ACProtec.00777194
007771B3    41                      inc ecx
007771B4  ^ E9 8CFEFFFF             jmp ACProtec.00777045
```

---

## 四、还原嵌套加密清除垃圾

这可是件极其艰巨的工作，这里只简要说说：

看下面代码，在`0078586D`处下断，留意`[esp+10]`，等于5时将要解码段保存，等于4时将5保存的段还原，等于2时用`key`来解下面的代码，等于3时用`key`来加密上面的代码。

那我们可以想办法另它只解而不还原，最后再来把垃圾代码去除，呵呵，有几十处之多呀！大家可以打开看看有多少个90就知道有多垃圾啦。

---

``` 
0047C7DC    60                      pushad
0047C7DD    6A 05                   push 5
0047C7DF    6A 00                   push 0
0047C7E1    6A 00                   push 0
0047C7E3    6A FF                   push -1
0047C7E5    E8 83903000             call ACProtec.0078586D
0047C7EA    61                      popad
```

---

## 五、还原多态连接

原代码:

---

``` 
0078230B    60                      pushad
0078230C    8BEF                    mov ebp,edi
----------------
----------------
----------------
007824B6    E8 453C0000             call ACProtec.00786100
007824BB    8B4424 20               mov eax,dword ptr ss:[esp+20]
007824BF    2B85 46F84000           sub eax,dword ptr ss:[ebp+40F846]
007824C5    B9 E9030000             mov ecx,3E9
007824CA    8DBD 63D54000           lea edi,dword ptr ss:[ebp+40D563]
007824D0    F2:AF                   repne scas dword ptr es:[edi]
007824D2    0BC9                    or ecx,ecx
007824D4    75 04                   jnz short ACProtec.007824DA
007824D6    90                      nop
007824D7    90                      nop
007824D8    90                      nop
007824D9    90                      nop
007824DA    81E9 E9030000           sub ecx,3E9
007824E0    F7D1                    not ecx
007824E2    0FB69C0D 03E54000       movzx ebx,byte ptr ss:[ebp+ecx+40E503]
007824EA    8D849D EBE84000         lea eax,dword ptr ss:[ebp+ebx\*4+40E8EB]
007824F1    8DBD 07184000           lea edi,dword ptr ss:[ebp+401807]
007824F7    66:C707 FF25            mov word ptr ds:[edi],25FF
007824FC    8947 02                 mov dword ptr ds:[edi+2],eax
007824FF    C647 06 C3              mov byte ptr ds:[edi+6],0C3
00782503    FF7424 20               push dword ptr ss:[esp+20]
00782507    8DBD 07184000           lea edi,dword ptr ss:[ebp+401807]
0078250D    33C9                    xor ecx,ecx
0078250F    83F9 08                 cmp ecx,8
00782512    74 0E                   je short ACProtec.00782522
00782514    90                      nop
```
