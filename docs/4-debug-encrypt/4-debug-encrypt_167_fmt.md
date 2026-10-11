
# 一个比较另类的 ECC stuff

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-28.htm`](https://www.pediy.com/kssd/pediy08/pediy8-28.htm)

这个 `crackme` 是 tkm 2004 年的 strainer, 用 ECC 保护的。虽然 ECC 大家玩得比较熟了，不过这个东东的保护方法还是有点意思。

用 `PEiD` 查壳并脱壳，`unpack` 不是目的，算法才是关键。`unpack` 后载入 `ida`, 用 `miracl` 的 `sig`，或

`http://www.secretashell.com/cryptocrk/tutorials/miracl.calls.zip` 这里的方法识别 `miracl` 库函数，检查序列号的流程一目了然。

## 1. 获取 `name`, `sn1`, `sn2`

```
代码:

* * *

Un_FSG_:0040194C loc_40194C:
Un_FSG_:0040194C push 50h
Un_FSG_:0040194E push offset name
Un_FSG_:00401953 push 2
Un_FSG_:00401955 push edi
Un_FSG_:00401956 call GetDlgItemText ; name>=4
Un_FSG_:0040195B mov esi, eax
Un_FSG_:0040195D push 50h
Un_FSG_:0040195F push offset sn1
Un_FSG_:00401964 push 3
Un_FSG_:00401966 push edi
Un_FSG_:00401967 call GetDlgItemText ; sn1>=4
Un_FSG_:0040196C mov ebx, eax
Un_FSG_:0040196E push 50h
Un_FSG_:00401970 push offset sn2
Un_FSG_:00401975 push 4
Un_FSG_:00401977 push edi
Un_FSG_:00401978 call GetDlgItemText ; sn2>=1

* * *
```

## 2. 检查序列号

### 1) 椭圆曲线 `E(a,b,p)`、点(`pt1`,`pt2`,`pt3`,`pt4`,`pt5`)初始化

```
代码:

* * *

Un_FSG_:0040149C call epoint_init
Un_FSG_:004014A1 mov edi, eax
Un_FSG_:004014A3 mov [esp+1F8h+var_1F8], 0
Un_FSG_:004014AA call _mirvar
Un_FSG_:004014AF mov [ebp+pt2_x], eax
Un_FSG_:004014B5 call epoint_init
Un_FSG_:004014BA mov [esp+1F8h+var_1F8], 0
Un_FSG_:004014C1 mov [ebp+pt2], eax
Un_FSG_:004014C7 call _mirvar
Un_FSG_:004014CC mov [ebp+big_t], eax
Un_FSG_:004014D2 call epoint_init
Un_FSG_:004014D7 mov [ebp+pt3], eax
Un_FSG_:004014DD call epoint_init
Un_FSG_:004014E2 mov [ebp+pt4], eax
Un_FSG_:004014E8 call epoint_init
Un_FSG_:004014ED mov dword ptr [esi+220h], 10h
Un_FSG_:004014F7 mov [ebp+pt5], eax
...
Un_FSG_:00401512 mov ebx, [ebp+a]
Un_FSG_:00401518 push offset a416d656e657369 ; "416D656E65736961"
Un_FSG_:0040151D push ebx
Un_FSG_:0040151E call _cinstr
Un_FSG_:00401523 pop eax
Un_FSG_:00401524 pop edx
Un_FSG_:00401525 mov eax, [ebp+b]
Un_FSG_:0040152B push offset a1b35b7093fee5a ; "1B35B7093FEE5AE601A"
Un_FSG_:00401530 push eax
Un_FSG_:00401531 call _cinstr
Un_FSG_:00401536 pop ebx
Un_FSG_:00401537 pop esi
Un_FSG_:00401538 mov esi, [ebp+p]
Un_FSG_:0040153E push offset aFffffffdffffff ; "FFFFFFFDFFFFFFFFFFFFFFFFFFFFFFFF"
Un_FSG_:00401543 push esi
Un_FSG_:00401544 call _cinstr
Un_FSG_:00401549 pop eax
Un_FSG_:0040154A pop edx
Un_FSG_:0040154B mov edx, [ebp+pt1_x]
Un_FSG_:00401551 push offset a71263a72c2fdfb ; "71263A72C2FDFB8FE851182B408210A4"
Un_FSG_:00401556 push edx
Un_FSG_:00401557 call _cinstr
Un_FSG_:0040155C mov eax, [ebp+pt2_x]
Un_FSG_:00401562 pop ecx
Un_FSG_:00401563 pop ebx
Un_FSG_:00401564 push offset a2c052124bdd94e ; "2C052124BDD94E5645E99B01DCECA28D"
Un_FSG_:00401569 push eax
Un_FSG_:0040156A call _cinstr
Un_FSG_:0040156F mov eax, [ebp+p]
Un_FSG_:00401575 push 1
Un_FSG_:00401577 push eax
Un_FSG_:00401578 mov eax, [ebp+b]
Un_FSG_:0040157E push eax
Un_FSG_:0040157F mov eax, [ebp+a]
Un_FSG_:00401585 push eax
Un_FSG_:00401586 call _ecurve_init
Un_FSG_:0040158B mov eax, [ebp+pt1_x]
Un_FSG_:00401591 add esp, 20h
Un_FSG_:00401594 push edi ; pt1
Un_FSG_:00401595 push 0
Un_FSG_:00401597 push eax
Un_FSG_:00401598 mov eax, [ebp+pt1_x]
Un_FSG_:0040159E push eax
Un_FSG_:0040159F call _epoint_set
Un_FSG_:004015A4 mov ecx, [ebp+pt2_x]
Un_FSG_:004015AA mov eax, [ebp+pt2]
Un_FSG_:004015B0 mov ebx, [ebp+pt2_x]
Un_FSG_:004015B6 push eax
Un_FSG_:004015B7 push 1
Un_FSG_:004015B9 push ebx
Un_FSG_:004015BA push ecx
Un_FSG_:004015BB call _epoint_set

* * *
```

其中 `E` 的参数为:

`a:416D656E65736961`

`b:1B35B7093FEE5AE601A`

`p:FFFFFFFDFFFFFFFFFFFFFFFFFFFFFFFF`

`pt1` 的 `x` 坐标为: `71263A72C2FDFB8FE851182B408210A4`

`pt2` 的 `x` 坐标为: `2C052124BDD94E5645E99B01DCECA28D`

### 2) `sn1`,`sn2` 转变为大数, 并变形

```
代码:

* * *

Un_FSG_:004015C0 mov esi, [ebp+big_sn1]
Un_FSG_:004015C6 add esp, 18h
Un_FSG_:004015C9 movsx ebx, ds:Serial_sign
Un_FSG_:004015D0 push offset sn1
Un_FSG_:004015D5 sub ebx, 30h
Un_FSG_:004015D8 push esi
Un_FSG_:004015D9 lea esi, [ebp+var_1A8]
Un_FSG_:004015DF call _cinstr
Un_FSG_:004015E4 pop eax
Un_FSG_:004015E5 pop edx
Un_FSG_:004015E6 mov eax, [ebp+big_sn2]
Un_FSG_:004015EC push offset sn2
Un_FSG_:004015F1 push eax
Un_FSG_:004015F2 call _cinstr

* * *
```

其中 `sn1` 的第一个 `byte` 是正负标志位，剩下的部分转化为大数



### 1. 代码段

```
Un_FSG__:004015F7              mov     eax, [ebp+big_t]
Un_FSG__:004015FD              push    eax
Un_FSG__:004015FE              mov     eax, [ebp+p]
Un_FSG__:00401604              push    eax
Un_FSG__:00401605              mov     eax, [ebp+big_sn1]
Un_FSG__:0040160B              push    1
Un_FSG__:0040160D              push    eax
Un_FSG__:0040160E              call    _powmod
Un_FSG__:00401613              mov     eax, [ebp+big_t]
Un_FSG__:00401619              add     esp, 20h
Un_FSG__:0040161C              mov     edx, [ebp+pt3]
Un_FSG__:00401622              push    edx
Un_FSG__:00401623              push    ebx
Un_FSG__:00401624              xor     ebx, ebx
Un_FSG__:00401626              push    eax
Un_FSG__:00401627              mov     eax, [ebp+big_t]
Un_FSG__:0040162D              push    eax
Un_FSG__:0040162E              call    _epoint_set     
```

`pt3` 的 `x` 坐标为 `big_sn1%p`

### 3) 计算 `name` 的 `sha` 值并变形

```
Un_FSG__:00401636              call    _shs_init
Un_FSG__:0040163B              mov     al, ds:name
Un_FSG__:00401640              add     esp, 10h
Un_FSG__:00401643              test    al, al
Un_FSG__:00401645              jz      short loc_401661
Un_FSG__:00401647 
Un_FSG__:00401647 loc_401647:                       ; CODE XREF: _check+27Fj
Un_FSG__:00401647              movsx   ecx, al
Un_FSG__:0040164A              push    edx
Un_FSG__:0040164B              inc     ebx
Un_FSG__:0040164C              push    edx
Un_FSG__:0040164D              push    ecx
Un_FSG__:0040164E              push    esi
Un_FSG__:0040164F              call    _shs_process
Un_FSG__:00401654              mov     al, ds:name[ebx]
Un_FSG__:0040165A              add     esp, 10h
Un_FSG__:0040165D              test    al, al
Un_FSG__:0040165F              jnz     short loc_401647
Un_FSG__:00401661 
Un_FSG__:00401661 loc_401661:                       ; CODE XREF: _check+265j
Un_FSG__:00401661              push    ebx
Un_FSG__:00401662              push    ebx
Un_FSG__:00401663              lea     ebx, [ebp+var_38]
Un_FSG__:00401666              push    ebx
Un_FSG__:00401667              push    esi
Un_FSG__:00401668              call    _shs_hash
Un_FSG__:0040166D              mov     edx, [ebp+big_hName]
Un_FSG__:00401673              add     esp, 0Ch
Un_FSG__:00401676              push    edx
Un_FSG__:00401677              push    ebx
Un_FSG__:00401678              push    14h
Un_FSG__:0040167A              call    _bytes_to_big
Un_FSG__:0040167F              mov     eax, [ebp+big_hName]
Un_FSG__:00401685              push    eax
Un_FSG__:00401686              mov     eax, [ebp+p2]    ;FFFFFFFE000000006FAFED9C3353F08D
Un_FSG__:0040168C              push    eax
Un_FSG__:0040168D              mov     eax, [ebp+big_hName]
Un_FSG__:00401693              push    3
Un_FSG__:00401695              push    eax
Un_FSG__:00401696              call    _powmod
```

`big_hName=sha(name)³%p2`

`p2` 被初始化为 `FFFFFFFE000000006FAFED9C3353F08D`

### 4) 计算中间值

```
Un_FSG__:0040169B              mov     eax, [ebp+pt4]
Un_FSG__:004016A1              add     esp, 1Ch
Un_FSG__:004016A4              push    eax
Un_FSG__:004016A5              mov     eax, [ebp+pt3]
Un_FSG__:004016AB              push    eax
Un_FSG__:004016AC              mov     eax, [ebp+big_sn2]
Un_FSG__:004016B2              push    eax
Un_FSG__:004016B3              call    _ecurve_mult     ; pt4=pt3*big_sn2
Un_FSG__:004016B8              mov     eax, [ebp+big_hName]
Un_FSG__:004016BE              add     esp, 0Ch
Un_FSG__:004016C1              push    edi
Un_FSG__:004016C2              push    edi
Un_FSG__:004016C3              push    eax
Un_FSG__:004016C4              call    _ecurve_mult     ; pt1=pt1*big_hName
Un_FSG__:004016C9              mov     ebx, [ebp+big_sn1]
Un_FSG__:004016CF              add     esp, 0Ch
Un_FSG__:004016D2              mov     ecx, [ebp+pt5]
Un_FSG__:004016D8              mov     esi, [ebp+pt2]
Un_FSG__:004016DE              push    ecx
Un_FSG__:004016DF              push    esi
Un_FSG__:004016E0              push    ebx
Un_FSG__:004016E1              call    _ecurve_mult     ; pt5=pt2*big_sn1
Un_FSG__:004016E6              pop     eax
Un_FSG__:004016E7              pop     edx
Un_FSG__:004016E8              mov     edx, [ebp+pt5]
Un_FSG__:004016EE              push    edx
Un_FSG__:004016EF              push    edi
Un_FSG__:004016F0              call    _ecurve_add      ; pt5=pt5+pt1
Un_FSG__:004016F0                                  ; pt5=pt1*big_hName+
Un_FSG__:004016F0                                  ;        pt2*big_sn1
```

`pt4=pt3*big_sn2`

`pt5=pt5+pt1`

`=pt1*big_hName+pt2*big_sn1`

### 5) 比较

通过两处进行比较

```
Un_FSG__:004016F5              mov     eax, [ebp+pt4]
Un_FSG__:004016FB              pop     ecx
Un_FSG__:004016FC              pop     ebx
Un_FSG__:004016FD              push    eax
Un_FSG__:004016FE              mov     eax, [ebp+pt5]
Un_FSG__:00401704              push    eax
Un_FSG__:00401705              call    _epoint_comp     ; pt4=t5
...
Un_FSG__:004017F5              mov     eax, [ebp+pt5]
Un_FSG__:004017FB              push    edi
Un_FSG__:004017FC              push    eax
Un_FSG__:004017FD              call    _epoint_comp     ;pt5!=pt1
```

如果 `pt4=pt5` 且 `pt5!=pt1` 则通过

## 3. Keygen

最终只要 `pt3*big_sn2=pt1*big_hName+pt2*big_sn1` 即可通过检查

其中 `pt3` 的 `x` 坐标为 `big_sn1%p`

`big_hName` 是 `name` 的 `hash` 值变形得来，是不可逆的，同样 `pt1`，`pt2` 是给定的也做不了文章，能下手的就是 `pt3`，`sn1`，`sn2` 这几部分

不妨试试，令 `pt3=pt1+pt2`，`big_sn2=big_hName`

那么方程被简化为 `pt2*big_hName=pt2*big_sn1`

椭圆曲线作为一个阶数为 `n` 的有限群，它上面任意一点满足 `pt*n=indefinite`（因为元素的阶是群的阶的因子）。如果椭圆曲线不好理解，可以拿模质数的乘法群来类比。`indefinite` 相当于 `1`。在 `{1,2,3,4,5,6}` 这几个数构成的 `mod 7` 群里，群的阶（元素个数）为 `6`，任意一个数的 `6` 次方除以 `7` 均余 `1` 也就是那个单位元。

所以只要 `big_sn1%n=big_hName` 即可

那么最终我们得到了一组同余方程

`big_sn1=pt3_x   mod p`

`big_sn1=big_hName  mod n`

得到 `sn1`

利用 `big_sn2=big_hName`

得到 `sn2`

当时在 `keygen` 的时候我对 `miracl` 还不熟，我自己手工解这个同余方程，大家可以用 `miracl` 实现一个更好的 `keygen`，下面是 `keygen` 的链接

**`keygen`**

`4dd466d0e1e0cc123afbe504da2462ca`
