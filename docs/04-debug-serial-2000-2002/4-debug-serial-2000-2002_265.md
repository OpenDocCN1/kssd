
# CrackMe1.0

> 原文：[`www.pediy.com/kssd/pediy03/forum1237.htm`](https://www.pediy.com/kssd/pediy03/forum1237.htm)

CrackMe1.0 的破解

Wrote By NYDoll

自从研究 CCG 的 CrackMe 之后 便开始对 CrackMe 这个东西很感兴趣，所以自己也想作出一个来，所以上网下载了一些，回来研究想借鉴一下经验，我们来看看这个 CrackMe，比较典型 BC 编写的，有它特定的结构，试试你就知道了，比较适合初学者破解，即使你写不出注册机，找出注册码是应该一点问题都没有的，然后慢慢就可以看懂算法，到时候写注册机的容易了。^\_^ 罗嗦了好多，我们还是回来看看程序：

我用的断点是 `BPX HMEMCPY`，算得上是一个万能断点了，一般的程序用这个断点都可以拦下来。

来到这里，我们开始步步观察算法：

```
* Reference To: USER32.GetDlgItem, Ord:0000h
:004010B6 E8E79B0000                Call 0040ACA2
:004010BB 8BF0                      mov esi, eax
:004010BD 8D8548FFFFFF              lea eax, dword ptr [ebp+FFFFFF48]
:004010C3 50                        push eax //用户名入栈
:004010C4 E867050000                call 00401630
:004010C9 59                        pop ecx
:004010CA 8945D8                    mov dword ptr [ebp-28], eax //将用户名压入地址[EBP-28]
:004010CD 8D95E4FEFFFF              lea edx, dword ptr [ebp+FFFFFEE4]
:004010D3 52                        push edx //注册码入栈
:004010D4 E857050000                call 00401630
:004010D9 59                        pop ecx
:004010DA 68EAB04000                push 0040B0EA
:004010DF E84C050000                call 00401630
:004010E4 59                        pop ecx
:004010E5 680EB14000                push 0040B10E
:004010EA E841050000                call 00401630
:004010EF 59                        pop ecx
:004010F0 837DD803                  cmp dword ptr [ebp-28], 00000003 //对比用户名位数是否小于 3
:004010F4 7E7B                      jle 00401171 //小于或者等于则跳转到：00401171
:004010F6 90                        nop
:004010F7 90                        nop
:004010F8 90                        nop
:004010F9 90                        nop
:004010FA 33C9                      xor ecx, ecx //ECX 寄存器清零
:004010FC 33D2                      xor edx, edx //EDX 寄存器清零
:004010FE 33DB                      xor ebx, ebx //EBX 寄存器清零
:00401100 33C0                      xor eax, eax //EAX 寄存器清零
:00401102 837DD832                  cmp dword ptr [ebp-28], 00000032 //对比用户名位数是否小于 50
:00401106 7D69                      jge 00401171 //大于或者等于则跳转到：00401171
:00401108 90                        nop
:00401109 90                        nop
:0040110A 90                        nop
:0040110B 90                        nop
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0040111C(C)
:0040110C 0FBE840D48FFFFFF          movsx eax, byte ptr [ebp+ecx-000000B8] //依次取用户名的每一位放入 EAX
:00401114 41                        inc ecx //ECX 寄存器加一
:00401115 33C1                      xor eax, ecx //将 EAX 和 ECX 寄存器的值进行异或运算后放入 EAX 寄存器
:00401117 03D8                      add ebx, eax //EBX 存放每次循环后 EAX 值的累加结果
:00401119 3B4DD8                    cmp ecx, dword ptr [ebp-28] //检测用户名的字符是否取完
:0040111C 75EE                      jne 0040110C //取完则跳出循环
:0040111E 6BC006                    imul eax, 00000006 //将用户名最后一位字符与 ECX 寄存器的值异或后的结果乘以 6，然后将结果保存到 EAX 中。
:00401121 C1E307                    shl ebx, 07 //EBX 寄存器的值逻辑左移 7，结果存放到 EDX 寄存器中
:00401124 03C3                      add eax, ebx //将 EAX 寄存器和 EBX 寄存器中的值相加。
:00401126 8945C8                    mov dword ptr [ebp-38], eax //将结果压入地址[EBP-38]中
:00401129 FF75C8                    push [ebp-38] //压入结果
* Possible StringData Ref from Data Obj ->"%lX"
:0040112C 6838B44000                push 0040B438
:00401131 8D8D80FEFFFF              lea ecx, dword ptr [ebp+FFFFFE80]
:00401137 51                        push ecx
:00401138 E8873D0000                call 00404EC4
:0040113D 83C40C                    add esp, 0000000C
:00401140 8D8580FEFFFF              lea eax, dword ptr [ebp+FFFFFE80]
:00401146 50                        push eax
:00401147 8D95E4FEFFFF              lea edx, dword ptr [ebp+FFFFFEE4]
:0040114D 52                        push edx
```
将上面的计算结果转换成 16 进制，它的十六进制值就是真正的注册码。

```
* Reference To: KERNEL32.lstrcmpA, Ord:0000h
:0040114E E8339C0000                Call 0040AD86 //对比真假注册码
:00401153 85C0                      test eax, eax
:00401155 750D                      jne 00401164 //若对比失败，则跳转。
* Possible StringData Ref from Data Obj ->"恭喜您! 如果您能编写出这个 CrackMe 的 KeyGen, "
->"请写一个教程给我 ;).
->"
* Reference To: USER32.SetWindowTextA, Ord:0000h
:00401157 683CB44000                push 0040B43C
:0040115C 56                        push esi
:0040115D E8289B0000                Call 0040AC8A
:00401162 EB18                      jmp 0040117C
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00401155(C)
* Possible StringData Ref from Data Obj ->"这个注册码是无效的!! 请再试一遍... "
->": <尚未注册>
* Reference To: USER32.SetWindowTextA, Ord:0000h
:00401164 6890B44000                push 0040B490
:00401169 56                        push esi
:0040116A E81B9B0000                Call 0040AC8A
:0040116F EB0B                      jmp 0040117C
* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:004010F4(C), :00401106(C)
* Possible StringData Ref from Data Obj ->"用户名必须为 4 个字符以上 50 个字符一下 "
->"!!
* Reference To: USER32.SetWindowTextA, Ord:0000h
:00401171 68C9B44000                push 0040B4C9
:00401176 56                        push esi
:00401177 E80E9B0000                Call 0040AC8A
* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:00401162(U), :0040116F(U)
:0040117C 5F                        pop edi
:0040117D 5E                        pop esi
:0040117E 5B                        pop ebx
:0040117F 8BE5                      mov esp, ebp
:00401181 5D                        pop ebp
:00401182 C3                        ret
```

接下来我们来总结一下算法，举例如用户名为 `NYDoll`，`CrackMe` 依次取出用户名的每一位字符，转换成十进制，然后与当前字符的位数进行异或运算，然后将每次异或运算后的值累加到 `EBX` 寄存器中，`NYDoll` 的累加结果为 `559`，然后将累加结果逻辑左移 `7` 位，将其结果保存到 `EBX` 中。我的结果为 `71552`。将注册名的最后一位与注册名的位数进行异或运算，其值乘以 `6`，然后将结果保存到 `EAX` 寄存器中，即 `l` 与 `6` 进行异或运算然后再乘以 `6`，结果为 `636`。将 `71552` 与 `636` 的和 `72188` 的 `16` 进制值 `119FC` 入栈，即 `119FC` 为 `NYDoll` 相对应的注册码。

```c
main()
{
    long a,i,c,d,h=0,f,sn;
    char name[50];
    printf(" *************************\n *KeyGen by TAE![CCG]*\n *************************\n\n");
    printf("Please input your name:");
    gets(name);
    printf("Your registration code:");
    d=strlen(name);
    for (i=0;i<d;i++)
    {
        c=name[i]^(i+1);
        h+=c;
    }
    a=name[i-1];
    a^=i;
    f=a*6;
    h=h<<7;
    sn=f+h;
    printf("%lX",sn);
}
```
`4dd466d0e1e0cc123afbe504da2462ca`
