
# 易用会员管理软件 1.38 算法分析

> 原文：`www.pediy.com/kssd/pediy07/pediy7-113.htm`

【破解作者】 `gxjxdqk` [DFCG] [CZG] [D.4s] [OCN] `POJE`  
【作者邮箱】 `gxjxdqk@163.com`  
【使用工具】 `OD`、`PEID`  
【破解平台】 `Win9x/NT/2000/XP`  
【软件名称】 易用会员管理软件 1.38  
【下载地址】 `http://www.onlinedown.net/soft/26837.htm`  
【软件简介】 软件大小：`3751KB`

软件语言：简体中文

软件类别：国产软件/共享版/办公软件

运行环境：`Win9x/Me/NT/2000/XP`

加入时间：`2005-3-3 16:58:06`

【软件大小】 `3751KB`

【加壳方式】 无壳

【破解声明】 我是一只小菜鸟，偶得一点心得，愿与大家分享：）

---

## 【破解内容】

`PEID` 查壳没有，：） `DELPHI` 编写的试注册有出错提示框，好啊成功 80%

下断点 `MessageBoxExA`、`MessageBoxA`、`MessageBoxExW`、`MessageBoxW` 我几个都下因为我人懒不愿意一个个试：——）

断下后 `ALT+F9` 返回程序领空向上找关键点下断，重新运行程序被断在下面：

```assembly
00605897    .  68 D5596000       push Member.006059D5
0060589C    .  64:FF30          push dword ptr fs:[eax]
0060589F    .  64:8920         mov dword ptr fs:[eax],esp
006058A2    .  8B45 FC         mov eax,dword ptr ss:[ebp-4]
006058A5    .  E8 AE020000     call Member.00605B58   关键 CALL 出注册码了
006058AA    .  84C0            test al,al             AL 是否为 0 不为 0 就光明了
006058AC    .  0F84 DB000000  je Member.0060598D  爆点 AL 为 0 跳向错误
```

这是跟进 `006058A5` 的 `CALL`

```assembly
00605B58  /$  55               push ebp
00605B59  |.  8BEC             mov ebp,esp
00605B5B  |.  33C9             xor ecx,ecx
00605B5D  |.  51               push ecx
00605B5E  |.  51               push ecx
00605B5F  |.  51               push ecx
00605B60  |.  51               push ecx
00605B61  |.  51               push ecx
00605B62  |.  53               push ebx
00605B63  |.  56               push esi
00605B64  |.  8BF0             mov esi,eax
00605B66  |.  33C0             xor eax,eax
00605B68  |.  55               push ebp
00605B69  |.  68 135C6000      push Member.00605C13
00605B6E  |.  64:FF30          push dword ptr fs:[eax]
00605B71  |.  64:8920         mov dword ptr fs:[eax]
00605B74  |.  8D55 FC         lea edx,dword ptr ss:[ebp-4]
00605B77  |.  8B86 FC020000   mov eax,dword ptr ds:[esi+2FC]
00605B7D  |.  E8 8EA5E4FF     call Member.00450110             取注册码位数的 CALL
00605B82  |.  8B45 FC         mov eax,dword ptr ss:[ebp-4]         ;  假码给 EAX
00605B85  |.  50               push eax                           ;  假码入堆栈
00605B86  |.  8D55 F4         lea edx,dword ptr ss:[ebp-C]
00605B89  |.  8B86 F8020000   mov eax,dword ptr ds:[esi+2F8]
00605B8F  |.  E8 7CA5E4FF     call Member.00450110
00605B94  |.  8B55 F4         mov edx,dword ptr ss:[ebp-C]         ;  读出用户名给 EDX
00605B97  |.  8D4D F8         lea ecx,dword ptr ss:[ebp-8]
00605B9A  |.  8BC6             mov eax,esi
00605B9C  |.  E8 F3FAFFFF     call Member.00605694                 ;  算法 CALL
00605BA1  |.  8B55 F8         mov edx,dword ptr ss:[ebp-8]         ;  如果不跟算法到这就出真码给 EDX
00605BA4  |.  58               pop eax
00605BA5  |.  E8 6AF4DFFF     call Member.00405014
```

从（`00605B9C`）算法 `CALL` 跟进的

```assembly
00605694  /$  55               push ebp
00605695  |.  8BEC             mov ebp,esp
00605697  |.  51               push ecx
00605698  |.  B9 04000000      mov ecx,4
0060569D  |>  6A 00           /push 0
0060569F  |.  6A 00           |push 0
006056A1  |.  49               |dec ecx
006056A2  |.^ 75 F9           \jnz short Member.0060569D
006056A4  |.  51               push ecx
006056A5  |.  874D FC         xchg dword ptr ss:[ebp-4],ecx
006056A8  |.  53               push ebx
006056A9  |.  56               push esi
006056AA  |.  57               push edi
006056AB  |.  8BF9             mov edi,ecx
006056AD  |.  8955 FC         mov dword ptr ss:[ebp-4],edx         ;  用户名给 EBP-4
006056B0  |.  8B45 FC         mov eax,dword ptr ss:[ebp-4]         ;  用户名给 EAX
006056B3  |.  E8 00FADFFF     call Member.004050B8
006056B8  |.  33C0             xor eax,eax
006056BA  |.  55               push ebp
006056BB  |.  68 55586000      push Member.00605855
006056C0  |.  64:FF30          push dword ptr fs:[eax]
006056C3  |.  64:8920         mov dword ptr fs:[eax],esp
006056C6  |.  8BC7             mov eax,edi
006056C8  |.  E8 4BF5DFFF     call Member.00404C18
006056CD  |.  8B45 FC         mov eax,dword ptr ss:[ebp-4]         ;  用户名给 EAX
006056D0  |.  E8 FBF7DFFF     call Member.00404ED0                 ;  取用户名位数的 CALL
006056D5  |.  8BF0             mov esi,eax                          ;  位数给 ESI
006056D7  |.  85F6             test esi,esi
006056D9  |.  7E 26           jle short Member.00605701            ;  如用户名为 0 就跳
006056DB  |.  BB 01000000      mov ebx,1                            ;  ebx 付初值 1
006056E0  |>  8D4D EC         /lea ecx,dword ptr ss:[ebp-14]
006056E3  |.  8B45 FC         |mov eax,dword ptr ss:[ebp-4]        ;  用户名给 EAX
006056E6  |.  0FB64418 FF    |movzx eax,byte ptr ds:[eax+ebx-1]   ;  取用户名一位
006056EB  |.  33D2           |xor edx,edx                         ;  消 0
006056ED  |.  E8 9E49E0FF    |call Member.0040A090
006056F2  |.  8B55 EC         |mov edx,dword ptr ss:[ebp-14]       ;  一位用户名给 EDX
006056F5  |.  8D45 F8         |lea eax,dword ptr ss:[ebp-8]
006056F8  |.  E8 DBF7DFFF     |call Member.00404ED8
006056FD  |.  43               |inc ebx                             ;  EBX+1（累加器）
006056FE  |.  4E               dec esi
006056FF  |.^ 75 DF           \jnz short Member.006056E0           ;  没有算完用户名位数跳，在循环
00605701  |>  8B45 F8         mov eax,dword ptr ss:[ebp-8]         ;  用户名 16 进制给 EAX（616263）
00605704  |.  E8 C7F7DFFF     call Member.00404ED0
00605709  |.  8BF0             mov esi,eax                          ;  用户名 16 进制位数不给 ESI（6）
0060570B  |.  85F6             test esi,esi
0060570D  |.  7E 2C           jle short Member.0060573B            ;  位数是不否小于等于 0 等于跳
0060570F  |.  BB 01000000      mov ebx,1
00605714  |>  8B45 F8         /mov eax,dword ptr ss:[ebp-8]   这个循环是把用户名的十六进制反排的循环
00605717  |.  E8 B4F7DFFF     |call Member.00404ED0
0060571C  |.  2BC3             |sub eax,ebx                         ;  16 进制位数-1 给 EAX
0060571E  |.  8B55 F8         |mov edx,dword ptr ss:[ebp-8]
00605721  |.  8A1402          |mov dl,byte ptr ds:[edx+eax]        ;  倒取一位 16 进给 DL
00605724  |.  8D45 E8         |lea eax,dword ptr ss:[ebp-18]
00605727  |.  E8 CCF6DFFF     |call Member.00404DF8
0060572C  |.  8B55 E8         |mov edx,dword ptr ss:[ebp-18]       ;  3 给 EDX
0060572F  |.  8D45 F4         |lea eax,dword ptr ss:[ebp-C]
00605732  |.  E8 A1F7DFFF     |call Member.00404ED8
00605737  |.  43               |inc ebx
00605738  |.  4E               dec esi
00605739  |.^ 75 D9           \jnz short Member.00605714
0060573B  |>  8D45 F8         lea eax,dword ptr ss:[ebp-8]
0060573E  |.  50               push eax
0060573F  |.  B9 04000000      mov ecx,4
00605744  |.  BA 01000000      mov edx,1
00605749  |.  8B45 F4         mov eax,dword ptr ss:[ebp-C]         ;  反排好的十六进制名给 EAX（362616）
0060574C  |.  E8 D7F9DFFF     call Member.00405128
00605751  |.  8D45 F4         lea eax,dword ptr ss:[ebp-C]
00605754  |.  50               push eax
00605755  |.  B9 04000000      mov ecx,4
0060575A  |.  BA 05000000      mov edx,5
0060575F  |.  8B45 F4         mov eax,dword ptr ss:[ebp-C]         ;  反排好的十六进制名给 EAX（362616）
00605762  |.  E8 C1F9DFFF     call Member.00405128                 ;  取前四位
```



```
00605767   |.  8B45 F8             mov eax,dword ptr ss:[ebp-8]             ;  前四位给 EAX  

0060576A   |.  E8 61F7DFFF         call Member.00404ED0  

0060576F   |.  83F8 04            cmp eax,4                              ;  跟 4 比等于就跳  

00605772   |.  7D 2F              jge short Member.006057A3  

00605774   |.  8B45 F8            mov eax,dword ptr ss:[ebp-8]  

00605777   |.  E8 54F7DFFF         call Member.00404ED0  

0060577C   |.  8BD8               mov ebx,eax  

0060577E   |.  83FB 03            cmp ebx,3  

00605781   |.  7F 20              jg short Member.006057A3  

00605783   |>  8D4D E4            /lea ecx,dword ptr ss:[ebp-1C]  

00605786   |.  8BC3              |mov eax,ebx  

00605788   |.  C1E0 02           |shl eax,2  

0060578B   |.  33D2              |xor edx,edx  

0060578D   |.  E8 FE48E0FF       |call Member.0040A090  

00605792   |.  8B55 E4           |mov edx,dword ptr ss:[ebp-1C]  

00605795   |.  8D45 F8           |lea eax,dword ptr ss:[ebp-8]  

00605798   |.  E8 3BF7DFFF       |call Member.00404ED8  

0060579D   |.  43                |inc ebx  

0060579E   |.  83FB 04           |cmp ebx,4  

006057A1   |.^ 75 E0             \jnz short Member.00605783  

006057A3   |>  8B45 F4            mov eax,dword ptr ss:[ebp-C]             ;  后四位给 EAX  

006057A6   |.  E8 25F7DFFF         call Member.00404ED0  

006057AB   |.  83F8 04            cmp eax,4  

006057AE   |.  7D 2F              jge short Member.006057DF               ;  EAX 是否为 4 如果取到后面了位数不大于或等于 4 位就不跳  

006057B0   |.  8B45 F4            mov eax,dword ptr ss:[ebp-C]  

006057B3   |.  E8 18F7DFFF         call Member.00404ED0  

006057B8   |.  8BD8               mov ebx,eax  

006057BA   |.  83FB 03            cmp ebx,3  

006057BD   |.  7F 20              jg short Member.006057DF  

006057BF   |>  8D4D E0            /lea ecx,dword ptr ss:[ebp-20]  

006057C2   |.  8BC3              |mov eax,ebx  

006057C4   |.  C1E0 02           |shl eax,2  

006057C7   |.  33D2              |xor edx,edx  

006057C9   |.  E8 C248E0FF       |call Member.0040A090  

006057CE   |.  8B55 E0           |mov edx,dword ptr ss:[ebp-20]  

006057D1   |.  8D45 F4           |lea eax,dword ptr ss:[ebp-C]  

006057D4   |.  E8 FFF6DFFF       |call Member.00404ED8  

006057D9   |.  43                |inc ebx  

006057DA   |.  83FB 04           |cmp ebx,4  

006057DD   |.^ 75 E0             \jnz short Member.006057BF  

006057DF   |>  8D45 F0            lea eax,dword ptr ss:[ebp-10]  

006057E2   |.  BA 6C586000        mov edx,Member.0060586C                ;  meme45erpe 给 EDX  

006057E7   |.  E8 C4F4DFFF         call Member.00404CB0  

006057EC   |.  8D45 DC            lea eax,dword ptr ss:[ebp-24]  

006057EF   |.  50                push eax  

006057F0   |.  B9 04000000        mov ecx,4  

006057F5   |.  BA 01000000        mov edx,1  

006057FA   |.  8B45 F0            mov eax,dword ptr ss:[ebp-10]           ;  meme45erpe 给 EAX  

006057FD   |.  E8 26F9DFFF         call Member.00405128                   ;  取 meme45erpe 常数的四位  

00605802   |.  FF75 DC            push dword ptr ss:[ebp-24]              ;  取 meme45erpe 常数的四位入堆栈  

00605805   |.  68 80586000        push Member.00605880  

0060580A   |.  FF75 F8            push dword ptr ss:[ebp-8]  

0060580D   |.  8D45 D8            lea eax,dword ptr ss:[ebp-28]  

00605810   |.  50                push eax  

00605811   |.  B9 05000000        mov ecx,5  

00605816   |.  BA 05000000        mov edx,5  

0060581B   |.  8B45 F0            mov eax,dword ptr ss:[ebp-10]           ;  取 mem45erpe 给 EAX  

0060581E   |.  E8 05F9DFFF         call Member.00405128                   ;  取 mem45erpe 下五位  

00605823   |.  FF75 D8            push dword ptr ss:[ebp-28]  

00605826   |.  68 80586000        push Member.00605880  

0060582B   |.  FF75 F4            push dword ptr ss:[ebp-C]               ;  反排好的十六进制名第五位到第八位入堆栈  

0060582E   |.  8BC7               mov eax,edi  

00605830   |.  BA 06000000        mov edx,6  

00605835   |.  E8 56F7DFFF         call Member.00404F90  

0060583A   |.  33C0               xor eax,eax  

0060583C   |.  5A                pop edx  

0060583D   |.  59                pop ecx  

0060583E   |.  59                pop ecx  

0060583F   |.  64:8910           mov dword ptr fs:[eax],edx  

00605842   |.  68 5C586000        push Member.0060585C  

00605847   |>  8D45 D8            lea eax,dword ptr ss:[ebp-28]  

0060584A   |.  BA 0A000000        mov edx,0A  

0060584F   |.  E8 E8F3DFFF         call Member.00404C3C                  

00605854   \.  C3                retn  
```

## 破解总结

### 总结

一、取用户名十六进制来反排序（`A`）

二、用到的常数为 `mem45erpe` （`B`）

三、注册码的组合方式为

`B` 的前四位 - `A` 前四位 + 常数的后五位 - `A` 的第五位到第八位

#### 附一组注册信息

`gxjxdqk[DFCG][D.4s][CZG][OCN]POJE`

`mem4-54A45erpe-F405`

### 内存注册机做法

`00605BA5`

中断 `1`

`E8`

`5`

`EDX`

--------------------------------------------------------------------------------

## 版权声明

本文纯属技术交流，转载请注明作者并保持文章的完整，谢谢!

`4dd466d0e1e0cc123afbe504da2462ca`
