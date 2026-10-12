
```asm
:0047B5D4 8BD7                 mov edx, edi

:0047B5D6 8B45FC               mov eax, dword ptr [ebp-04]

:0047B5D9 E8EE77F8FF           call 00402DCC    //将整个密码变为 16 进制

:0047B5DE 8BD0                 mov edx, eax

:0047B5E0 8955DC               mov dword ptr [ebp-24], edx

:0047B5E3 DB45DC               fild dword ptr [ebp-24]

:0047B5E6 D81D44B64700         fcomp dword ptr [0047B644]

:0047B5EC DFE0                 fstsw ax         //COPY 状态寄存器到 AX   |=>密码是否大于 1000000

:0047B5EE 9E                   sahf             //COPY 状态位到标志寄存器中

:0047B5EF 7211                 jb 0047B602      //如小于就 JUMP

:0047B5F1 8955D4               mov dword ptr [ebp-2C], edx

:0047B5F4 DB45D4               fild dword ptr [ebp-2C]

:0047B5F7 D81D48B64700         fcomp dword ptr [0047B648]

:0047B5FD DFE0                 fstsw ax

:0047B5FF 9E                   sahf

:0047B600 7603                 jbe 0047B605     //如小于等于则 JUMP

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:0047B5EF(C)

|

:0047B602 C60601               mov byte ptr [esi], 01

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

|:0047B4B4(U), :0047B600(C)

|

:0047B605 33C0                 xor eax, eax

:0047B607 5A                   pop edx

:0047B608 59                   pop ecx

:0047B609 59                   pop ecx

:0047B60A 648910               mov dword ptr fs:[eax], edx

:0047B60D 6837B64700           push 0047B637

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:0047B635(U)

|

:0047B612 8D45D8               lea eax, dword ptr [ebp-28]

:0047B615 E89687F8FF           call 00403DB0

:0047B61A 8D45E0               lea eax, dword ptr [ebp-20]

:0047B61D BA06000000           mov edx, 00000006

:0047B622 E8AD87F8FF           call 00403DD4

:0047B627 8D45FC               lea eax, dword ptr [ebp-04]

:0047B62A E88187F8FF           call 00403DB0

:0047B62F C3                    ret
```

注册后写入注册表：`HKEY_USERS\.DEFAULT\Software\Teddyware\BananaSplitter`

`"RegName"="(可以随便填)"`

`"RegNum"="1288543"` <===这个数是注册器算出来的

### 算法总结

假设输入密码为 `S`（必须为 7 位，且全部为数字 0-9）注册时输入的 `NAME` 根本没用。

将密码 `S` 的每一位分别换为 16 进制数，分别为 `S1` `S2` `S3` `S4` `S5` `S6` `S7`

`S1*2+S2+S3*2+S4+S5*2+S6=A`

`[A-(取十进制形式 A 十位上的数)*10]+1=B`

`B` 是否等于 `S7`

并且 `1000000<S<3000000`

### 注册器

注册码还真不少!

```c
main()

{

int a,b,s1,s2,s3,s4,s5,s6,s7;

long i=1000000;

for(a=0,b=0;i<=3000000;i++)

  {

  s1=i/1000000;

  s2=i/100000%10;

  s3=i/10000%100%10;

  s4=i/1000%1000%100%10;

  s5=i/100%10000%1000%100%10;

  s6=i/10%100000%10000%1000%100%10;

  s7=i%10;

  a=s1*2+s2+s3*2+s4+s5*2+s6;

  b=a-a/10*10+1;

  if(b==s7)

    printf("\t%ld",i);

  }

}
```

如有错误请指出，EMAIL:`CL517@YEAH.NET`

最后谢谢大家能够听我哆嗦完!

garfield cat `4dd466d0e1e0cc123afbe504da2462ca`
