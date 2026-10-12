
# 瑞星杀毒 2001

> 原文：[`www.pediy.com/kssd/pediy03/forum513.htm`](https://www.pediy.com/kssd/pediy03/forum513.htm)

在无钥匙盘下，解瑞星杀毒软件【2001 版】

一、用 `TYP` 测试瑞星杀毒软件【2001 版】是被哪种壳给加密了

哇是 `UPX1.03`，没有现成工具，我才不脱呢。（我懒的要死）

二、运行 `TRW2000`

三、运行安装文件 `setup.exe`

提示插入 A 号盘。

四、按 `TRW2000` 的热键，（`ctrl+n`）

停在系统里面了，不管它 3,7,28。

`TRW2000` 有个好用的命令 `pmodule` 就用它

五、返回到提示按取消，`TRW2000` 给激活了

怎么会给激活了（是命令 `pmodule`）

看到我们兴趣的地方了

```
017F:00406660  MOV      EAX,[ECX+02D8]
017F:00406666  CMP      EAX,BYTE +07
017F:00406669  JZ       00406670
017F:0040666B  CMP      EAX,BYTE +64
017F:0040666E  JNZ      00406680
017F:00406670  XOR      EAX,EAX
017F:00406672  PUSH     EAX
017F:00406673  MOV      EAX,[ESP+08]
017F:00406677  PUSH     EAX
017F:00406678  CALL     `SETUP!CheckKey`  <-------检查钥匙盘
017F:0040667D  RET      04
017F:00406680  CMP      EAX,BYTE +14
017F:00406683  JNZ      00406698
017F:00406685  MOV      EAX,02
017F:0040668A  PUSH     EAX
017F:0040668B  MOV      EAX,[ESP+08]
017F:0040668F  PUSH     EAX
017F:00406690  CALL     `SETUP!CheckKey`  <-------检查钥匙盘
017F:00406695  RET      04                <---------停在这里

----------------------
017F:004064B0  CALL     00406660          <-------主 call
017F:004064B5  TEST     EAX,EAX           <-------test eax
017F:004064B7  JZ       NEAR 004065EC     <-------出错跳转...
017F:004064BD  PUSH     EBX
017F:004064BE  LEA      ECX,[ESP+08]
017F:004064C2  PUSH     EDI
017F:004064C3  LEA      EAX,[ESI+034C]
017F:004064C9  PUSH     ECX
```

制造补丁我使用 `R!SC's Process Patcher v1.5.1`

（内存动态补丁制作软件） `4dd466d0e1e0cc123afbe504da2462ca`
