
# drip

> 原文：[`www.pediy.com/kssd/pediy02/forum206.htm`](https://www.pediy.com/kssd/pediy02/forum206.htm)

整理磁盘时误将 `drip` 的破解文件给删除了，麻烦你手中有的话给我再发一次。

汉化已好：`http://go.163.com/~sons/soft/DRIP.EXE`

另外还有一个程序：

可以在美国特许厅 `USUTO` 上下载文件，

`http://go.163.com/~sons/soft/djusp170.lzh` 315k

未注册只能下载 5 个文件。麻烦哪位帮我破解一下。

在此先谢谢。

`drip`

注册码错误得到"蔬杰澳迋獖詧醾羵膫軅?"，用 `W32DASM` 修改版反汇编

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

```
|:004037B3(C)  <--***
|
:0040385E 8B761C                mov esi, dword ptr [esi+1C]
:00403861 85F6                  test esi, esi
:00403863 7403                  je 00403868
:00403865 8B761C                mov esi, dword ptr [esi+1C]

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:00403863(C)
|
:00403868 6A30                  push 00000030

\* Possible StringData Ref from Data Obj ->"搊榐幐攕"
                                |
:0040386A 6850A44600            push 0046A450

\* Possible StringData Ref from Data Obj ->"蔬杰澳迋獖詧醾羵膫軅?"
                                |
:0040386F 6834A44600            push 0046A434
```

再看

```
:004037A9 E8F8140200            call 00424CA6    <--重点
:004037AE 83C408                add esp, 00000008
:004037B1 85C0                  test eax, eax
:004037B3 0F85A5000000          jne 0040385E
```

所以用 `F8` 进入`:004037A9 E8F8140200       call 00424CA6`

走到

```
:00424CE6 8945F8                mov dword ptr [ebp-08], eax
```

下指令 `d eax`, 就看到注册码

我的是

```
xiaolou
aaa
01E5
```

`DJUSP` 注册

用 `WDASM` 反汇编

```
:004698E5 E8FAFEFFFF            call 004697E4
:004698EA 8BD8                  mov ebx, eax
:004698EC 8D45F8                lea eax, dword ptr [ebp-08]
:004698EF E834A3F9FF            call 00403C28

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:004698E1(U)
|
:004698F4 84DB                  test bl, bl   <--bl=1 注册成功
:004698F6 752C                  jne 00469924
:004698F8 6A20                  push 00000020
:004698FA E869D9F9FF            Call 00407268
:004698FF 6A10                  push 00000010

\* Possible StringData Ref from Code Obj ->"Registration"
                                |
:00469901 68A0994600            push 004699A0

\* Possible StringData Ref from Code Obj ->"Password incorrect. "
                                |
:00469906 68B0994600            push 004699B0
:0046990B 8BC7                  mov eax, edi
:0046990D E8D2C2FCFF            call 00435BE4
```

注意到`:004698F4 84DB       test bl, bl`

处 `BL` 由`:004698EA 8BD8       mov ebx, eax` 赋值,所以用 `F8` 进入

```
:004698E5 E8FAFEFFFF       call 004697E4
```

......

```
:00469847 8BC3                  mov eax, ebx      <--eax 为输入数值的 16 进制
:00469849 3513487D00            xor eax, 007D4813 <--异或
:0046984E 3DF0198E00            cmp eax, 008E19F0 <--比较
:00469853 7504                  jne 00469859
```

所以我得到 `PASSWORD: 1767996592`

此外,我用 `WDASM` 修改版不能正确反汇编, 看来还是要用原版. `4dd466d0e1e0cc123afbe504da2462ca`
