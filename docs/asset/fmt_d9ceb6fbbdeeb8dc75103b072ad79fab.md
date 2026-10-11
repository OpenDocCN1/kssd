
# 敏思硬盘卫士 2.2

> 原文：[`www.pediy.com/kssd/pediy03/forum1175.htm`](https://www.pediy.com/kssd/pediy03/forum1175.htm)

该软件无壳，无暗桩，无 `crc`，属典型的“三无产品”，最适合我等新手练手了，呵呵~~~~~!!!

1. 运行 `trw 2000` 和程序，并填入假注册码：`7878748`
2. `ctrl+N`
3. `bpx hmemcpy`
4. `g or F5`
5. 回到程序点确定
6. 成功拦截
7. `bc *`
8. `pmodule`
9. 按 `F12` 七下后再按 `F10` 来到如下代码：

```
...
:004A4144 8B45FC       mov eax, [ebp-04] <---Fuck sn:7878748
:004A4147 50           push eax
:004A4148 8D55F4       lea edx, [ebp-0C]
:004A414B 8B8304030000 mov eax, [ebx+00000304]
:004A4151 E8FAD6F8FF   call 00431850
:004A4156 8B45F4       mov eax, [ebp-0C] <---存放的机器码
:004A4159 8D4DF8       lea ecx, [ebp-08]
:004A415C 66BA0D00     mov dx, 000D
:004A4160 E8A3F8FFFF   call 004A3A08
:004A4165 8B55F8       mov edx, [ebp-08] <---Real sn
:004A4168 58           pop eax
:004A4169 E83EFEF5FF   call 00403FAC <---比较真假注册码的 call
:004A416E 7423         je 004A4193
```

在上面代码三个注释处下 `D` 命令可分别看到

`Fuck sn:` `7878748`

`machine sn:` `873934337758`

`Real sn:` `2981003264`

我在用 `crackcode 2000` 做注册机时碰到一个问题，就是不知道什么时候该用 `0`，`1`，`2` 等不同模式。特向各位大侠求教。各位大侠一定要帮帮我！！！

```c
#include<stdio.h>
#include<string.h>

main()
{
    char sn[99];
    long code,b,i,j,l,p,a;

    printf("*************************\n *KeyGen by TAE![CCG]*\n *************************\n\n");
    printf("Please input your sn :");
    gets(sn);

    l=strlen(sn);
    for (i=0;i<l;i++)
    {
        if (sn[i]>=0x61&& sn[i]<=0x7a) sn[i]-=20;
        if (i%2!=0) sn[i]+=0x0D;else sn[i]+=0x1A;
    }

    b=sn[0]+1;
    code=b*0x1e61;
    for (j=1;j<l;j++)
    {
        p=j+1;
        a=sn[j]+p;
        code*=a;
    }

    printf("Your registration code:%ld",code);
}
```

`4dd466d0e1e0cc123afbe504da2462ca`
