
# 光盘卫士 v1.5

> 原文：[`www.pediy.com/kssd/pediy05/pediy50597.htm`](https://www.pediy.com/kssd/pediy05/pediy50597.htm)

这个软件早就升级到 `1.9` 了，且到 `1.9` 是免费的，贴这个破解过程似乎没有太大的意义了，但我想应该可以用来学习一下，也想在精华 5 在充个数。

**破解者**：`HMILY[CCG]`

**软件名称**：光盘卫士 `v1.5`

**破解说明**：这个软件是用 `aspack` `v2.00` 压缩的，脱壳后汇编找以下内容：

```
:0047212A E8C173FFFF                call 004694F0 ->重点，跟进去
:0047212F 84C0                      test al, al   ->测试 al=0 吗？
:00472131 0F84BE000000              je 004721F5    ->相等则跳，跳就 game over

* Possible StringData Ref from Code Obj ->"user"

:00472137 6884224700                push 00472284
:0047213C 8D55F0                    lea edx, dword ptr [ebp-10]
:0047213F 8B83E8020000              mov eax, dword ptr [ebx+000002E8]
:00472145 E87276FBFF                call 004297BC
:0047214A 8B55F0                    mov edx, dword ptr [ebp-10]
:0047214D 8D4DF4                    lea ecx, dword ptr [ebp-0C]
:00472150 A14C984700                mov eax, dword ptr [0047984C]
:00472155 8B00                      mov eax, dword ptr [eax]
:00472157 E820330000                call 0047547C
:0047215C 8B45F4                    mov eax, dword ptr [ebp-0C]
:0047215F 50                        push eax
:00472160 A14C984700                mov eax, dword ptr [0047984C]
:00472165 8B00                      mov eax, dword ptr [eax]

* Possible StringData Ref from Code Obj ->"software\microsoft\windows\currentversion\syst"
                                ->"em\Qu\cdsafe" ->向注册表中加入注册信息，用户名和
                                注册码都是加过密的，把 sn 删除又是未注册版

:00472167 B994224700                mov ecx, 00472294
:0047216C BA02000080                mov edx, 80000002
:00472171 E816300000                call 0047518C
:00472176 68D8224700                push 004722D8
:0047217B 8D55E8                    lea edx, dword ptr [ebp-18]
:0047217E 8B83E4020000              mov eax, dword ptr [ebx+000002E4]
:00472184 E83376FBFF                call 004297BC
:00472189 8B55E8                    mov edx, dword ptr [ebp-18]
:0047218C 8D4DEC                    lea ecx, dword ptr [ebp-14]
:0047218F A14C984700                mov eax, dword ptr [0047984C]
:00472194 8B00                      mov eax, dword ptr [eax]
:00472196 E8E1320000                call 0047547C
:0047219B 8B45EC                    mov eax, dword ptr [ebp-14]
:0047219E 50                        push eax
:0047219F A14C984700                mov eax, dword ptr [0047984C]
:004721A4 8B00                      mov eax, dword ptr [eax]

* Possible StringData Ref from Code Obj ->"software\microsoft\windows\currentversion\syst"
                                ->"em\Qu\cdsafe"

:004721A6 B994224700                mov ecx, 00472294
:004721AB BA02000080                mov edx, 80000002
:004721B0 E8D72F0000                call 0047518C
:004721B5 A14C984700                mov eax, dword ptr [0047984C]
:004721BA 8B00                      mov eax, dword ptr [eax]
:004721BC C7808C030000FFFFFFFF      mov dword ptr [ebx+0000038C], FFFFFFFF
:004721C6 A174974700                mov eax, dword ptr [00479774]
:004721CB 8B00                      mov eax, dword ptr [eax]
:004721CD 8B80D4020000              mov eax, dword ptr [eax+000002D4]

* Possible StringData Ref from Code Obj ->"感谢您的使用本软件"

:004721D3 BAE4224700                mov edx, 004722E4
:004721D8 E80F76FBFF                call 004297EC
:004721DD A174974700                mov eax, dword ptr [00479774]
:004721E2 8B00                      mov eax, dword ptr [eax]
:004721E4 8B10                      mov edx, dword ptr [eax]
:004721E6 FF92D8000000              call dword ptr [edx+000000D8]
:004721EC 8BC3                      mov eax, ebx
:004721EE E82D1FFDFF                call 00444120
:004721F3 EB3D                      jmp 00472232

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00472131(C)

:004721F5 A174974700                mov eax, dword ptr [00479774]
:004721FA 8B00                      mov eax, dword ptr [eax]
:004721FC 8B80E4020000              mov eax, dword ptr [eax+000002E4]

* Possible StringData Ref from Code Obj ->"错误"

:00472202 BA00234700                mov edx, 00472300
:00472207 E8E075FBFF                call 004297EC
:0047220C A174974700                mov eax, dword ptr [00479774]
:00472211 8B00                      mov eax, dword ptr [eax]
:00472213 8B80D4020000              mov eax, dword ptr [eax+000002D4]

* Possible StringData Ref from Code Obj ->"注册码不正确"

:00472219 BA10234700                mov edx, 00472310
:0047221E E8C975FBFF                call 004297EC
:00472223 A174974700                mov eax, dword ptr [00479774]
:00472228 8B00                      mov eax, dword ptr [eax]
:0047222A 8B10                      mov edx, dword ptr [eax]
:0047222C FF92D8000000              call dword ptr [edx+000000D8]

==============================================================================================
* Referenced by a CALL at Addresses:
|:0047212A   , :00475A09

:004694F0 55                        push ebp      ->跟进上面那个 call 来到这里
:004694F1 8BEC                      mov ebp, esp  ->暴力破解改这里 558BEC->改为 B001C3
:004694F3 83C4F8                    add esp, FFFFFFF8
:004694F6 53                        push ebx
:004694F7 56                        push esi
:004694F8 33DB                      xor ebx, ebx
:004694FA 895DF8                    mov dword ptr [ebp-08], ebx
:004694FD 894DFC                    mov dword ptr [ebp-04], ecx
:00469500 8BF2                      mov esi, edx
:00469502 8BD8                      mov ebx, eax
:00469504 8B45FC                    mov eax, dword ptr [ebp-04]
:00469507 E8D8A9F9FF                call 00403EE4
:0046950C 8B4508                    mov eax, dword ptr [ebp+08]
:0046950F E8D0A9F9FF                call 00403EE4
:00469514 33C0                      xor eax, eax
:00469516 55                        push ebp
:00469517 6877954600                push 00469577
:0046951C 64FF30                    push dword ptr fs:[eax]
:0046951F 648920                    mov dword ptr fs:[eax], esp
:00469522 837DFC00                  cmp dword ptr [ebp-04], 00000000
:00469526 7504                      jne 0046952C
:00469528 33DB                      xor ebx, ebx
:0046952A EB28                      jmp 00469554

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00469526(C)

:0046952C 85F6                      test esi, esi
:0046952E 7504                      jne 00469534
:00469530 33DB                      xor ebx, ebx
:00469532 EB20                      jmp 00469554

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0046952E(C)

:00469534 8D45F8                    lea eax, dword ptr [ebp-08]
:00469537 50                        push eax
:00469538 8B4DFC                    mov ecx, dword ptr [ebp-04]
:0046953B 8BD6                      mov edx, esi
:0046953D 8BC3                      mov eax, ebx
:0046953F E880FeFFFF                call 004693C4                ->注册码的计算，跟进去看看
:00469544 8B55F8                    mov edx, dword ptr [ebp-08]
:00469547 8B4508                    mov eax, dword ptr [ebp+08]
:0046954A E8F1A8F9FF                call 00403E40
:0046954F 0F94C0                    sete al
:00469552 8BD8                      mov ebx, eax

==============================================================================================
* Referenced by a CALL at Addresses:
|:0046932E   , :0046938F   , :0046953F

:004693C4 55                        push ebp ->跟进注册码计算 call 来到这里
:004693C5 8BEC                      mov ebp, esp
```



## 反汇编分析

以下是软件注册码验证逻辑的相关汇编代码片段：

```assembly
:004693C7 6A00                 push 00000000  
:004693C9 6A00                 push 00000000  
:004693CB 6A00                 push 00000000  
:004693CD 6A00                 push 00000000  
:004693CF 6A00                 push 00000000  
:004693D1 53                   push ebx  
:004693D2 56                   push esi  
:004693D3 894DFC               mov dword ptr [ebp-04], ecx  
:004693D6 8BF2                 mov esi, edx  
:004693D8 8B45FC               mov eax, dword ptr [ebp-04]  
:004693DB E804ABF9FF           call 00403EE4  
:004693E0 33C0                 xor eax, eax  
:004693E2 55                   push ebp  
:004693E3 68C2944600           push 004694C2  
:004693E8 64FF30               push dword ptr fs:[eax]  
:004693EB 648920               mov dword ptr fs:[eax], esp  
:004693EE 8D45F8               lea eax, dword ptr [ebp-08]

* Possible StringData Ref from Code Obj ->"Error"  
                    |  

:004693F1 BADC944600           mov edx, 004694DC  
:004693F6 E84DA7F9FF           call 00403B48  
:004693FB 85F6                 test esi, esi  
:004693FD 0F8499000000         je 0046949C  
:00469403 837DFC00             cmp dword ptr [ebp-04], 00000000  
:00469407 0F848F000000         je 0046949C  
:0046940D 8B45FC               mov eax, dword ptr [ebp-04]  
:00469410 E81BA9F9FF           call 00403D30 ->取注册名位数  
:00469415 8BD8                 mov ebx, eax ->将位数传给 ebx  
:00469417 0FAFDE               imul ebx, esi ->esi=25F5 是一个基数，ebx=位数*0x25F5  
:0046941A 8B45FC               mov eax, dword ptr [ebp-04]  
:0046941D 0FB600               movzx eax, byte ptr [eax] ->注册名第一位  
:00469420 69C09A020000         imul eax, 0000029A       ->eax=eax*0x29A  
:00469426 03D8                 add ebx, eax             ->ebx=eax+ebx 得到注册码第一部分  
:00469428 8D55F4               lea edx, dword ptr [ebp-0C]  
:0046942B 8BC3                 mov eax, ebx  
:0046942D E81AECF9FF           call 0040804C  
:00469432 8B55F4               mov edx, dword ptr [ebp-0C]  
:00469435 8D45F8               lea eax, dword ptr [ebp-08]  
:00469438 B9EC944600           mov ecx, 004694EC  
:0046943D E83AA9F9FF           call 00403D7C  
:00469442 8B45FC               mov eax, dword ptr [ebp-04] ->注册名传给 eax  
:00469445 0FB600               movzx eax, byte ptr [eax] ->取注册名第一位传给 eax  
:00469448 F7EE                 imul esi                 ->eax=第一位*基数(0x25F5)  
:0046944A 6BD87B               imul ebx, eax, 0000007B  ->ebx=eax*0x7B  
:0046944D FF75F8               push [ebp-08]  
:00469450 8D55F0               lea edx, dword ptr [ebp-10]  
:00469453 8BC3                 mov eax, ebx  
:00469455 E8F2EBF9FF           call 0040804C  
:0046945A FF75F0               push [ebp-10]  
:0046945D 68EC944600           push 004694EC  
:00469462 8D45F8               lea eax, dword ptr [ebp-08]  
:00469465 BA03000000           mov edx, 00000003  
:0046946A E881A9F9FF           call 00403DF0  
:0046946F 8B45FC               mov eax, dword ptr [ebp-04]  
:00469472 E8B9A8F9FF           call 00403D30  
:00469477 8B55FC               mov edx, dword ptr [ebp-04] ->注册名传给 eax  
:0046947A 0FB612               movzx edx, byte ptr [edx]   ->取注册名第一位传给 edx  
:0046947D F7EA                 imul edx                   ->eax=注册名位数*edx  
:0046947F 69D8D5190000        imul ebx, eax, 000019D5    ->ebx=eax*0x19D5  
:00469485 03DE                 add ebx, esi               ->ebx=ebx+0x25F5  
:00469487 8D55EC               lea edx, dword ptr [ebp-14]  
:0046948A 8BC3                 mov eax, ebx  
:0046948C E8BBEBF9FF           call 0040804C  
:00469491 8B55EC               mov edx, dword ptr [ebp-14]  
:00469494 8D45F8               lea eax, dword ptr [ebp-08]  
:00469497 E89CA8F9FF           call 00403D38

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:  
:004693FD(C), :00469407(C)  

:0046949C 8B4508               mov eax, dword ptr [ebp+08]  
:0046949F 8B55F8               mov edx, dword ptr [ebp-08]  
:004694A2 E85DA6F9FF           call 00403B04  
:004694A7 33C0                 xor eax, eax  
:004694A9 5A                   pop edx  
:004694AA 59                   pop ecx  
:004694AB 59                   pop ecx  
:004694AC 648910               mov dword ptr fs:[eax], edx  
:004694AF 68C9944600           push 004694C9

* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
:004694C7(U)  

:004694B4 8D45EC               lea eax, dword ptr [ebp-14]  
:004694B7 BA05000000           mov edx, 00000005  
:004694BC E813A6F9FF           call 00403AD4  
:004694C1 C3                   ret  
============================================================================================================================
```

## 总结

分析出了这个软件注册码的计算，怎么能少得了注册机呢！

`TC v2.0` 下调试通过。

```c
#include <stdio.h>  
#include <string.h>  

int main(void)  
{  
unsigned long a,b,c,d,x;  
unsigned char name[60],*p=name;  

strat:printf("CDsafe v1.5 keygen by HMILY[CCG][BCG]\n");  
printf("My QQ: 5289322  E-mail: gyyxll@ynmail.com\n");  
printf("******************************************\n");  
printf("Please enter your name = ");  
gets(name);  
a=strlen(name);  
if(*p=='\0'){ printf("Your user name is empty !!! please re-input\n");goto strat;}

x=name[0];  
b=a*0x25F5+x*0x29A;  
c=x*0x25F5*0x7B;  
d=a*x*0x19D5+0x25F5;

printf("Your register code is  = %ld-%ld-%ld\n",b,c,d);  
}
```

`4dd466d0e1e0cc123afbe504da2462ca`
