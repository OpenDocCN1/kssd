
# 斗地主 V1.1

> 原文：[`www.pediy.com/kssd/pediy02/forum193.htm`](https://www.pediy.com/kssd/pediy02/forum193.htm)

## 1. 下 `bpx hmemcpy`

每次跳出的“注册码无效，无法注册！”对话框好像随机的，无法拦截，别的中断也不行，请指教。

## 2. 在 Delphi 程序中为何找不到错误对话框

在这个 Delphi 程序中，为何找不到跳出错误对话框上的字“注册码无效，无法注册！”（使用 `FI` 检测程序没有压缩）（使用 `UltraEdit 7.10a`、`Hex Workshop v3.02`、`WinHex 9.4`）

## 斗地主 V1.1 破解实录

应网友的要求，我把以前破解的记录整理贴出来，供大家相互交流学习。不妥之处，还请各位高手指点。

**使用工具：**`TRW 1.13`

**找到输入注册信息对话框：填入以下内容**

- 注册姓名：咪咪
- 邮箱地址：`bandi@yeah.net`
- 注册码：`7878787878787878`

**起动 `TRW 1.13` 后按 `CTRL+N`**

**下命令：**`bpx hmemcpy`

```
    :g
```

**按马上注册，嘣跳到 `TRW`**

**下命令：**`bc *`

```
    :pmodule
```

来到程序的领空

**紧接着按 `F12` `N` 次 (失败的前一次)**

**再按 `F10`：**

```
0167:0045F3DD 8BC6             MOV      EAX,ESI<-----这时光棒停在这一句,好开始追. Let's Go!

0167:0045F3DF E84439FAFF       CALL     00402D28

0167:0045F3E4 803DB028460000   CMP      BYTE [004628B0],00

0167:0045F3EB 746A             JZ       0045F457

0167:0045F3ED 8D55F8           LEA      EDX,[EBP-08]

0167:0045F3F0 A1A8284600       MOV      EAX,[004628A8]

0167:0045F3F5 E8A2F5FEFF       CALL     0044E99C

0167:0045F3FA A1AC284600       MOV      EAX,[004628AC]

0167:0045F3FF 50               PUSH     EAX

0167:0045F400 8D45FC           LEA      EAX,[EBP-04]

0167:0045F403 50               PUSH     EAX

0167:0045F404 8B0DA4284600     MOV      ECX,[004628A4]

0167:0045F40A 8B55F8           MOV      EDX,[EBP-08]

0167:0045F40D A144294600       MOV      EAX,[00462944]

0167:0045F412 E805EEFEFF       CALL     0044E21C

0167:0045F417 8D4DF4           LEA      ECX,[EBP-0C]

0167:0045F41A 8B55F8           MOV      EDX,[EBP-08]

0167:0045F41D 8B45FC           MOV      EAX,[EBP-04]

0167:0045F420 E883F6FEFF       CALL     0044EAA8

0167:0045F425 8B55F4           MOV      EDX,[EBP-0C]

0167:0045F428 8D45FC           LEA      EAX,[EBP-04]

0167:0045F42B E81845FAFF       CALL     00403948

0167:0045F430 A1A8284600       MOV      EAX,[004628A8]

0167:0045F435 8B55FC           MOV      EDX,[EBP-04]

0167:0045F438 E8FF47FAFF       CALL     00403C3C---->注册比较的 CALL,追进去...!

0167:0045F43D 740C             JZ       0045F44B-->这里不跳,你就死定了.

0167:0045F43F A154294600       MOV      EAX,[00462954]

0167:0045F444 E84F3DFDFF       CALL     00433198--->走过这句就出现注册失败的对话框了.

0167:0045F449 EB0C             JMP      SHORT 0045F457

0167:0045F44B A13C294600       MOV      EAX,[0046293C]

0167:0045F450 E8433DFDFF       CALL     00433198---注册成功的 CALL

0167:0045F455 B301             MOV      BL,01

0167:0045F457 33C0             XOR      EAX,EAX

0167:0045F459 5A               POP      EDX

0167:0045F45A 59               POP      ECX

0167:0045F45B 59               POP      ECX

0167:0045F45C 648910           MOV      [FS:EAX],EDX

0167:0045F45F 6879F44500       PUSH     DWORD 0045F479

0167:0045F464 8D45F4           LEA      EAX,[EBP-0C]

0167:0045F467 BA03000000       MOV      EDX,03

0167:0045F46C E86344FAFF       CALL     004038D4

0167:0045F471 C3               RET
```

在 `0167:0045F438` 这一句。按 `F8` 进入后，我们来到这里：

```
0167:00403C3C 53               PUSH     EBX

0167:00403C3D 56               PUSH     ESI

0167:00403C3E 57               PUSH     EDI

0167:00403C3F 89C6             MOV      ESI,EAX

0167:00403C41 89D7             MOV      EDI,EDX

0167:00403C43 39D0             CMP      EAX,EDX\--这里是注册比较的地方

d eax 你乱输入的注册码:7878787878787878

d edx 正确的注册码:    72TGRK68S8447MQX(我看到的是这样的,你的呢?)

0167:00403C45 0F848F000000     JZ       NEAR 00403CDA

0167:00403C4B 85F6             TEST     ESI,ESI

0167:00403C4D 7468             JZ       00403CB7

0167:00403C4F 85FF             TEST     EDI,EDI

0167:00403C51 746B             JZ       00403CBE

0167:00403C53 8B46FC           MOV      EAX,[ESI-04]

0167:00403C56 8B57FC           MOV      EDX,[EDI-04]

0167:00403C59 29D0             SUB      EAX,EDX

0167:00403C5B 7702             JA       00403C5F

0167:00403C5D 01C2             ADD      EDX,EAX

0167:00403C5F 52               PUSH     EDX

0167:00403C60 C1EA02           SHR      EDX,02

0167:00403C63 7426             JZ       00403C8B

0167:00403C65 8B0E             MOV      ECX,[ESI]

0167:00403C67 8B1F             MOV      EBX,[EDI]

0167:00403C69 39D9             CMP      ECX,EBX

0167:00403C6B 7558             JNZ      00403CC5

0167:00403C6D 4A               DEC      EDX

0167:00403C6E 7415             JZ       00403C85

0167:00403C70 8B4E04           MOV      ECX,[ESI+04]

0167:00403C73 8B5F04           MOV      EBX,[EDI+04]

0167:00403C76 39D9             CMP      ECX,EBX

0167:00403C78 754B             JNZ      00403CC5

0167:00403C7A 83C608           ADD      ESI,BYTE +08

0167:00403C7D 83C708           ADD      EDI,BYTE +08

0167:00403C80 4A               DEC      EDX

0167:00403C81 75E2             JNZ      00403C65

0167:00403C83 EB06             JMP      SHORT 00403C8B

0167:00403C85 83C604           ADD      ESI,BYTE +04

0167:00403C88 83C704           ADD      EDI,BYTE +04

0167:00403C8B 5A               POP      EDX

0167:00403C8C 83E203           AND      EDX,BYTE +03

0167:00403C8F 7422             JZ       00403CB3

0167:00403C91 8B0E             MOV      ECX,[ESI]

0167:00403C93 8B1F             MOV      EBX,[EDI]

0167:00403C95 38D9             CMP      CL,BL

0167:00403C97 7541             JNZ      00403CDA

0167:00403C99 4A               DEC      EDX

0167:00403C9A 7417             JZ       00403CB3

0167:00403C9C 38FD             CMP      CH,BH

0167:00403C9E 753A             JNZ      00403CDA

0167:00403CA0 4A               DEC      EDX

0167:00403CA1 7410             JZ       00403CB3

0167:00403CA3 81E30000FF00     AND      EBX,00FF0000

0167:00403CA9 81E10000FF00     AND      ECX,00FF0000

0167:00403CAF 39D9             CMP      ECX,EBX

0167:00403CB1 7527             JNZ      00403CDA

0167:00403CB3 01C0             ADD      EAX,EAX

0167:00403CB5 EB23             JMP      SHORT 00403CDA

0167:00403CB7 8B57FC           MOV      EDX,[EDI-04]

0167:00403CBA 29D0             SUB      EAX,EDX

0167:00403CBC EB1C             JMP      SHORT 00403CDA

0167:00403CBE 8B46FC           MOV      EAX,[ESI-04]

0167:00403CC1 29D0             SUB      EAX,EDX

0167:00403CC3 EB15             JMP      SHORT 00403CDA

0167:00403CC5 5A               POP      EDX

0167:00403CC6 38D9             CMP      CL,BL

0167:00403CC8 7510             JNZ      00403CDA

0167:00403CCA 38FD             CMP      CH,BH

0167:00403CCC 750C             JNZ      00403CDA

0167:00403CCE C1E910           SHR      ECX,10

0167:00403CD1 C1EB10           SHR      EBX,10

0167:00403CD4 38D9             CMP      CL,BL

0167:00403CD6 7502             JNZ      00403CDA

0167:00403CD8 38FD             CMP      CH,BH

0167:00403CDA 5F               POP      EDI

0167:00403CDB 5E               POP      ESI

0167:00403CDC 5B               POP      EBX

0167:00403CDD C3               RET\-----------这里返回
```

## 整理如下

- 注册姓名：咪咪
- 邮箱地址：`bandi@yeah.net`
- 注册码：`72TGRK68S8447MQX`

按注册按钮后注册成功。注册成功后软件把注册信息写入到注册表中：

`[HKEY_CURRENT_USER\Software\Littlegrass]`

`[HKEY_CURRENT_USER\Software\Littlegrass\LandOwner]`

- `"Version"="1.1 共享版"`
- `"RegInfo1"="jg 妟妬 mp 卥 ijj 厙?"`
- `"RegInfo3"="bandi@yeah.net"`
- `"RegInfo2"="咪咪"`

删除这些键值后，该软件又变为未注册版了。

好了浪费了大家不少时间，希望你也能按照我的方法破解成功。 ^_^ `bandi`

从这里下载的：`http://www.win2000.com.cn/duodz.htm`

居然还非得用中文名。

- `Name`: 郭大志
- `Mail`: `123@456.789`
- `Code`: `MBTR-CFTH-JTPA-4RI6`

它先把你输入的 4 部分注册码合并在一起，然后比较：

```
0167:0045E792  MOV      EDX,[EBP-0C]

0167:0045E795  LEA      EAX,[EBP-04]

0167:0045E798  CALL     0040392C

0167:0045E79D  MOV      EAX,[0045FD5C]

0167:0045E7A2  MOV      EAX,[EAX]         //错误的注册码

0167:0045E7A4  MOV      EDX,[EBP-04]     //正确的注册码

0167:0045E7A7  CALL     00403C20         //比较

0167:0045E7AC  JZ       0045E7BA
```
