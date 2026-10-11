
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
