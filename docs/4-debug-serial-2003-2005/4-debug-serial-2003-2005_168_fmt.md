
# IE 修复大师 V1.4

> 原文：[`www.pediy.com/kssd/pediy06/pediy6091.htm`](https://www.pediy.com/kssd/pediy06/pediy6091.htm)

## 龙旋风- IE 修复大师 V1.4 算法分析 + 注册机源码

作者：`PaulYoung`[`CCG`]`[`OCN`]`

工具：`SoftIce`, `TURBO C 2.0`

时间：`2003-6-2`

主页：`http://www.zigsoft.net/`

----------------------------------------

这个软件是重启验证的，我怎么找到这些地方？你自己想想吧……

```
:004C4C17 E8ECFCFFFF            call 004C4908                    /* 关键计算 */

:004C4C1C 8B45EC               mov eax, dword ptr [ebp-14]

:004C4C1F E8F0FBF3FF           call 00404814

:004C4C24 83E802               sub eax, 00000002

:004C4C27 50                   push eax

:004C4C28 8D55E8               lea edx, dword ptr [ebp-18]

:004C4C2B 8B45FC               mov eax, dword ptr [ebp-04]

:004C4C2E E8D5FCFFFF          call 004C4908

:004C4C33 8B45E8               mov eax, dword ptr [ebp-18]

:004C4C36 33D2                 xor edx, edx

:004C4C38 59                   pop ecx

:004C4C39 E82EFEF3FF          call 00404A6C

:004C4C3E 837DFC00             cmp dword ptr [ebp-04], 00000000

:004C4C42 0F84C6000000         je 004C4D0E

:004C4C48 837DF800             cmp dword ptr [ebp-08], 00000000

:004C4C4C 0F84BC000000         je 004C4D0E

:004C4C52 8B45F0               mov eax, dword ptr [ebp-10]

:004C4C55 8B55F8               mov edx, dword ptr [ebp-08]

:004C4C58 E8FBFCF3FF          call 00404958                      /* 把你输入的注册码去掉末两位后与计算结果比较 */

:004C4C5D 0F85AB000000         jne 004C4D0E

:004C4C63 B201                 mov dl, 01
```

----------------------------------------

* Referenced by a `CALL` at Addresses:

`:004C4C17`, `:004C4C2E`, `:004C59C4`

```
:004C4908 55                   push ebp

:004C4909 8BEC                 mov ebp, esp

:004C490B B906000000           mov ecx, 00000006
```

* Referenced by a `(U)nconditional` or `(C)onditional` `Jump` at Address:

`:004C4915` `(C)`

```
:004C4910 6A00                 push 00000000

:004C4912 6A00                 push 00000000

:004C4914 49                   dec ecx

:004C4915 75F9                 jne 004C4910

:004C4917 53                   push ebx

:004C4918 56                   push esi
```

* Referenced by a `(U)nconditional` or `(C)onditional` `Jump` at Address:

`:004C48A2` `(C)`

```
:004C4919 57                   push edi

:004C491A 8955F8               mov dword ptr [ebp-08], edx

:004C491D 8945FC               mov dword ptr [ebp-04], eax

:004C4920 8B45FC               mov eax, dword ptr [ebp-04]

:004C4923 E8D400F4FF          call 004049FC

:004C4928 33C0                 xor eax, eax

:004C492A 55                   push ebp

:004C492B 68EC4A4C00           push 004C4AEC

:004C4930 64FF30               push dword ptr fs:[eax]

:004C4933 648920               mov dword ptr fs:[eax], esp

:004C4936 B201                 mov dl, 01
```

* Possible `StringData` `Ref` from `Code` `Obj` -> `"x 凙"`

```
:004C4938 A17C5B4100           mov eax, dword ptr [00415B7C]

:004C493D E87EEEF3FF          call 004037C0

:004C4942 8945EC               mov dword ptr [ebp-14], eax
```

* Referenced by a `(U)nconditional` or `(C)onditional` `Jump` at Address:

`:004C48DB` `(C)`

```
:004C4945 33C0                 xor eax, eax

:004C4947 55                   push ebp

:004C4948 686B4A4C00           push 004C4A6B

:004C494D 64FF30               push dword ptr fs:[eax]

:004C4950 648920               mov dword ptr fs:[eax], esp

:004C4953 8D45F4               lea eax, dword ptr [ebp-0C]

:004C4956 8B55FC               mov edx, dword ptr [ebp-04]

:004C4959 E896FCF3FF          call 004045F4

:004C495E 8B45F4               mov eax, dword ptr [ebp-0C]

:004C4961 E8AEFEF3FF          call 00404814

:004C4966 8BF0                 mov esi, eax

:004C4968 8B45F4               mov eax, dword ptr [ebp-0C]

:004C496B E8A4FEF3FF          call 00404814

:004C4970 8BD8                 mov ebx, eax

:004C4972 85DB                 test ebx, ebx

:004C4974 0F8EAA000000         jle 004C4A24
```

* Referenced by a `(U)nconditional` or `(C)onditional` `Jump` at Address:

`:004C4A1E` `(C)`

```
:004C497A 8BC3                 mov eax, ebx                  /* ebx 是取用户名的下标 */

:004C497C B903000000           mov ecx, 00000003

:004C4981 99                   cdq

:004C4982 F7F9                 idiv ecx

:004C4984 85D2                 test edx, edx

:004C4986 752B                 jne 004C49B3                  /* 求余 3，等于 0 则不跳，不等则跳 */

:004C4988 8B45F4               mov eax, dword ptr [ebp-0C]

:004C498B 0FB64418FF           movzx eax, byte ptr [eax+ebx-01]

:004C4990 8BD6                 mov edx, esi

:004C4992 2BD3                 sub edx, ebx

:004C4994 8B4DF4               mov ecx, dword ptr [ebp-0C]

:004C4997 0FB65411FF           movzx edx, byte ptr [ecx+edx-01]

:004C499C 03C2                 add eax, edx

:004C499E 8D55E8               lea edx, dword ptr [ebp-18]

:004C49A1 E8DE42F4FF          call 00408C84                   /* 上面几行是计算出一个数值并转换成字符串形式 */

:004C49A6 8B55E8               mov edx, dword ptr [ebp-18]

:004C49A9 8B45EC               mov eax, dword ptr [ebp-14]

:004C49AC 8B08                 mov ecx, dword ptr [eax]

:004C49AE FF5138               call [ecx+38]

:004C49B1 EB68                 jmp 004C4A1B
```

* Referenced by a `(U)nconditional` or `(C)onditional` `Jump` at Address:

`:004C4986` `(C)`

```
:004C49B3 8BC3                 mov eax, ebx                  /* 004C4986 跳则来此，实际是对下标进行奇偶性判断 */

:004C49B5 2501000080           and eax, 80000001

:004C49BA 7905                 jns 004C49C1

:004C49BC 48                   dec eax

:004C49BD 83C8FE               or eax, FFFFFFFE

:004C49C0 40                   inc eax
```

* Referenced by a `(U)nconditional` or `(C)onditional` `Jump` at Address:

`:004C49BA` `(C)`

```
:004C49C1 85C0                 test eax, eax

:004C49C3 7538                 jne 004C49FD                  /* eax==0 则不跳，eax==1 则跳 */

:004C49C5 8B45F4               mov eax, dword ptr [ebp-0C]

:004C49C8 0FB64418FF           movzx eax, byte ptr [eax+ebx-01]

:004C49CD 8BD6                 mov edx, esi

:004C49CF 2BD3                 sub edx, ebx

:004C49D1 8B4DF4               mov ecx, dword ptr [ebp-0C]

:004C49D4 0FB65411FF           movzx edx, byte ptr [ecx+edx-01]

:004C49D9 F7EA                 imul edx

:004C49DB 03C0                 add eax, eax

:004C49DD B90C000000           mov ecx, 0000000C

:004C49E2 33D2                 xor edx, edx

:004C49E4 F7F1                 div ecx

:004C49E6 8BC2                 mov eax, edx

:004C49E8 8D55E4               lea edx, dword ptr [ebp-1C]

:004C49EB E89442F4FF          call 00408C84                 /* eax==0 时，上面几行计算出一个数值并转换成字符形式 */

:004C49F0 8B55E4               mov edx, dword ptr [ebp-1C]

:004C49F3 8B45EC               mov eax, dword ptr [ebp-14]

:004C49F6 8B08                 mov ecx, dword ptr [eax]

:004C49F8 FF5138               call [ecx+38]

:004C49FB EB1E                 jmp 004C4A1B
```

* Referenced by a `(U)nconditional` or `(C)onditional` `Jump` at Address:

`:004C49C3` `(C)`



:004C49FD 8B45F4    mov eax, dword ptr [ebp-0C]

:004C4A00 0FB64418FF movzx eax, byte ptr [eax+ebx-01]

:004C4A05 83C002    add eax, 00000002

:004C4A08 8D55E0    lea edx, dword ptr [ebp-20]

:004C4A0B E87442F4FF call 00408C84    /* eax==1 时，上面几行计算出一个数值并转换成字符形式 */

:004C4A10 8B55E0    mov edx, dword ptr [ebp-20]

:004C4A13 8B45EC    mov eax, dword ptr [ebp-14]

:004C4A16 8B08      mov ecx, dword ptr [eax]

:004C4A18 FF5138    call [ecx+38]

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

|:004C49B1(U), :004C49FB(U)

:004C4A1B 4B        dec ebx

:004C4A1C 85DB      test ebx, ebx

:004C4A1E 0F8F56FFFFFF jg 004C497A    /* 形成循环 */

下面是把上面计算出的各个字符串颠倒过来连接成一个新的字符串，作为注册码第一部份……

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:004C4974(C)

:004C4A24 8B45EC    mov eax, dword ptr [ebp-14]

:004C4A27 8B10      mov edx, dword ptr [eax]

:004C4A29 FF5214    call [edx+14]

:004C4A2C 8BF0      mov esi, eax

:004C4A2E 4E        dec esi

:004C4A2F 85F6      test esi, esi

:004C4A31 7C22      jl 004C4A55

:004C4A33 46        inc esi

:004C4A34 33DB      xor ebx, ebx

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:004C4A53(C)

:004C4A36 8D4DDC    lea ecx, dword ptr [ebp-24]

:004C4A39 8BD3      mov edx, ebx

:004C4A3B 8B45EC    mov eax, dword ptr [ebp-14]

:004C4A3E 8B38      mov edi, dword ptr [eax]

:004C4A40 FF570C    call [edi+0C]

:004C4A43 8B55DC    mov edx, dword ptr [ebp-24]

:004C4A46 8D45F0    lea eax, dword ptr [ebp-10]

:004C4A49 8B4DF0    mov ecx, dword ptr [ebp-10]

:004C4A4C E80FFEF3FF call 00404860

:004C4A51 43        inc ebx

:004C4A52 4E        dec esi

:004C4A53 75E1      jne 004C4A36

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:004C4A31(C)

:004C4A55 33C0      xor eax, eax

:004C4A57 5A        pop edx

:004C4A58 59        pop ecx

:004C4A59 59        pop ecx

:004C4A5A 648910    mov dword ptr fs:[eax], edx

:004C4A5D 68724A4C00 push 004C4A72

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:004C4A70(U)

:004C4A62 8B45EC    mov eax, dword ptr [ebp-14]

:004C4A65 E886EDF3FF call 004037F0

:004C4A6A C3        ret

上面取用户名各字符时，是从末位开始取的，全部算完后又倒序排序各个字符串，其实等同于直接从用户名首位开始取字符计算，并将所有字符串顺次排列。我写的注册机就是这样的…… 我可不会多此一举！

如果你用上面计算出的结果，再在后面添加两位，注册后第一次运行会显示成功，但第二次运行仍是未注册，并将你注册表中的注册信息全部清除……也就是说，末两位并非是任意的。

下面就是末两位的计算……

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:004C4EE9(C)

:004C4EC5 8B55FC    mov edx, dword ptr [ebp-04]

:004C4EC8 8A5C32FF mov bl, byte ptr [edx+esi-01]

:004C4ECC 8BD3      mov edx, ebx

:004C4ECE 80C2BF    add dl, BF

:004C4ED1 80EA1A    sub dl, 1A

:004C4ED4 7311      jnb 004C4EE7

:004C4ED6 8B45FC    mov eax, dword ptr [ebp-04]

:004C4ED9 33C0      xor eax, eax

:004C4EDB 8AC3      mov al, bl

:004C4EDD 8D55F0    lea edx, dword ptr [ebp-10]

:004C4EE0 E89F3DF4FF call 00408C84

:004C4EE5 EB04      jmp 004C4EEB

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:004C4ED4(C)

:004C4EE7 46        inc esi

:004C4EE8 48        dec eax

:004C4EE9 75DA      jne 004C4EC5

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

|:004C4EC0(C), :004C4EE5(U)

:004C4EEB 8B45F0    mov eax, dword ptr [ebp-10]

:004C4EEE 8B55F8    mov edx, dword ptr [ebp-08]

:004C4EF1 E862FAF3FF call 00404958    /* 关键比较 */

:004C4EF6 7510      jne 004C4F08

上面实际是对除用户名末位外的各个字符进行检查，当检查到有大写英文字母时，跳出循环并将此英文字母 10 进制值转换成字符串形式，再与你输入的末两位注册码进行比较……没有大写英文字母？不相等？Bye my cracker!

附上 `TURBO C 2.0` 写的注册机！谁有更好更简洁的写法，欢迎指教！！

```c
main()
{
int i,j,len;
unsigned char name[80];

clrscr();

printf("龙旋风- IE 修复大师 V1.4 2003 新年钻石版 Build 0528 注册机  \n\n       * PaulYoung[CCG] 作品      [ 2003-06-03 ] *\n\n 注意事项：\n1.用户名长度必须在 8 个字符或 4 个汉字以上！除用户名末位外至少要包含 1 个以上的\n  大写英文字母，否则注册码将会无效！\n2.本注册机支持中文用户名！\n\n 请输入用户名 : ");

gets(name);

len=strlen(name);

printf("你的注册码是 : ");

for(i=0,j=1;name[i]!='\0';i++,j++)
 if(j%3>0&&j%2==0)       printf("%d",(unsigned long)name[i]*name[len-j-1]*2%0xC);
  else if(j%3>0&&j%2==1) printf("%d",name[i]+2);
   else if(j%3==0)       printf("%d",name[i]+name[len-j-1]);

for(i=0;i<len-1;i++)
   if (name[i]>0x40&&name[i]<0x5B) break;
   printf("%d\n\n",name[i]);

getch();
}
```

谢谢观赏！ 散场……

`4dd466d0e1e0cc123afbe504da2462ca`
