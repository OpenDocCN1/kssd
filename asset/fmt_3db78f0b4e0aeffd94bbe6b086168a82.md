
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
