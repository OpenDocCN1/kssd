
# Flashfxp 1.4

> 原文：[`www.pediy.com/kssd/pediy04/kanxue061.htm`](https://www.pediy.com/kssd/pediy04/kanxue061.htm)

## 脱 Flashfxp 1.4 的壳

`Flashfxp 1.4` 用 `fi` 查不出是加的什么壳，后来看到精品讨论汉化这个东西的时候才知道用的是 `telock`。`telock` 的壳保护很强，好像要比 `aspr` 的麻烦。我学脱这个还为时尚早，不过早晚都要碰的，花了 n 个小时来搞定它。特写此文，让你觉得脱 `telock` 的壳好像会很 easy;-)

`unpack(2001.9.26)`

`Flashfxp 1.4` 是用 `delphi` 写的，hoho 运气真好:)，更新一下用快速寻找 `delphi` 程序入口的方法(试过才知道我以前的方法=脱裤子放屁)：执行 `Flashfxp 1.4`，接着用 `winhex` 编辑 `Flashfxp` 的主内存，选择搜索文本，填 `runtime`,执行搜索，搜到后,向前找到离 `runtime` 最近的机器码为 `55 8B EC` 的地方就是程序的 `oep`，对应的地址是 `0052B8B0`，ok 完活。

ok，下面要在入口处脱壳，老规矩打开 `SuperBPM`，点 `erase`，用 `trw1.03` 娃娃修改过的版(因为其他任意一版 `trw` 都会死机的)载入 `Flashfxp`，下 `g 0052B8B0`，下 `suspend`，用 `prodump` 选 `dump(full)`，非法操作，我倒~~~~~，怎么这么麻烦呀:(，后来参考一下 `hying` 和 `fpc` 大哥的关于 `telock` 的脱壳资料，hoho，`telock` 会把原来的 `section` 数改写为 `ffff`,所以只要把原来的 `section` 数写回去就行:)。用 `winhex` 内存编辑功能把 `400106-400107` 填回原来的值 `0900`，再用 `prodump` 就成功啦。

接着修复 `import table`

天哪，这都是什么乱七八糟的呀。`it` 完全被 `telock` 改的面目全非了，用 `ImportREC` 根本无法修复。没办法跟踪程序吧，从 `telock` 重建 `It` 部分开始吧，用加了 `superbpm` 和 `icedump` 的 `s-ice` 重新载入 `Flashfxp`，下 `bpm 870000`，`f5`。在这里停下：

```
0187:00576E02 FF0424             INC     DWord Ptr [ESP]
0187:00576E05 891F               MOV     [EDI],EBX
0187:00576E07 83C304             ADD     EBX,00000004
0187:00576E0A 83C704             ADD     EDI,00000004
0187:00576E0D 83242403           AND     DWord Ptr [ESP],00000003
0187:00576E11 833C2400           CMP     DWord Ptr [ESP],00000000
0187:00576E15 742E               JZ      00576E45
0187:00576E17 833C2401           CMP     DWord Ptr [ESP],00000001
0187:00576E1B 7430               JZ      00576E4D
0187:00576E1D 833C2402           CMP     DWord Ptr [ESP],00000002
0187:00576E21 7447               JZ      00576E6A
0187:00576E23 C144240C10         ROL     DWord Ptr [ESP+0C],10
0187:00576E28 668B44240C         MOV     AX,[ESP+0C]
0187:00576E2D 66AB               STOSW
0187:00576E2F 8BC3               MOV     EAX,EBX
0187:00576E31 C1E803             SHR     EAX,03
0187:00576E34 83E003             AND     EAX,00000003
0187:00576E37 8A440404           MOV     AL,[ESP+1*EAX+04]
0187:00576E3B AA                 STOSB
0187:00576E3C B0C3               MOV     AL,C3
0187:00576E3E AA                 STOSB
0187:00576E3F 66B8FF35           MOV     AX,35FF
0187:00576E43 EB2D               JMP     00576E72
0187:00576E45 8B442408           MOV     EAX,[ESP+08]
0187:00576E49 AB                 STOSD
0187:00576E4A 4F                 DEC     EDI
0187:00576E4B EBEF               JMP     00576E3C
0187:00576E4D 668B44240C         MOV     AX,[ESP+0C]
0187:00576E52 66AB               STOSW
0187:00576E54 B0C3               MOV     AL,C3
0187:00576E56 AA                 STOSB
0187:00576E57 8BC3               MOV     EAX,EBX
0187:00576E59 C1E803             SHR     EAX,03
0187:00576E5C 83E003             AND     EAX,00000003
0187:00576E5F 8A440404           MOV     AL,[ESP+1*EAX+04]
0187:00576E63 AA                 STOSB
0187:00576E64 66B8FF35           MOV     AX,35FF
0187:00576E68 EB08               JMP     00576E72
0187:00576E6A 8B442410           MOV     EAX,[ESP+10]
0187:00576E6E AB                 STOSD
0187:00576E6F 4F                 DEC     EDI
0187:00576E70 EBCA               JMP     00576E3C
0187:00576E72 E28C               LOOP    00576E00
0187:00576E74 66AB               STOSW
0187:00576E76 83C414             ADD     ESP,00000014
0187:00576E79 61                 POPAD
0187:00576E7A 89BDAAB04000       MOV     [EBP+0040B0AA],EDI     //MOV [EBP+0040B0AA],EBX
                   899DAAB04000
0187:00576E80 8BBDAAB04000       MOV     EDI,[EBP+0040B0AA]     //MOV EDI,[EBP+0040B0B2]
                   8BBDB2B04000
0187:00576E86 8B85B2B04000       MOV     EAX,[EBP+0040B0B2]
0187:00576E8C 0385AAAF4000       ADD     EAX,[EBP+0040AFAA]
0187:00576E92 8B8DAEB04000       MOV     ECX,[EBP+0040B0AE]
0187:00576E98 8908               MOV     [EAX],ECX               //将函数名写到正确的位置
0187:00576E9A 8385AEB040000A     ADD     DWord Ptr [EBP+0040B0AE],0000000A
0187:00576EA1 EB08               JMP     00576EAB
0187:00576EA3 838DAAB04000FF     OR      DWord Ptr [EBP+0040B0AA],FFFFFFFF
0187:00576EAA 61                 POPAD
0187:00576EAB 03BDAAAF4000       ADD     EDI,[EBP+0040AFAA]
0187:00576EB1 85DB               TEST    EBX,EBX
0187:00576EB3 0F84B4000000       JZ      00576F6D
0187:00576EB9 F7C300000080       TEST    EBX,80000000
0187:00576EBF 6A00               PUSH    00000000
0187:00576EC1 7506               JNZ     00576EC9
0187:00576EC3 8D5C1302           LEA     EBX,[EBX+1*EDX+02]
0187:00576EC7 EB3C               JMP     00576F05
0187:00576EC9 FF0424             INC     DWord Ptr [ESP]
0187:00576ECC 8B85A6AF4000       MOV     EAX,[EBP+0040AFA6]
0187:00576ED2 3B859AB04000       CMP     EAX,[EBP+0040B09A]
0187:00576ED8 752B               JNZ     00576F05
0187:00576EDA 81E3FFFFFF7F      AND     EBX,7FFFFFFF
0187:00576EE0 8BD3               MOV     EDX,EBX
0187:00576EE2 8D1495FCFFFFFF     LEA     EDX,[FFFFFFFC+4*EDX]
0187:00576EE9 8B9DA6AF4000       MOV     EBX,[EBP+0040AFA6]
0187:00576EEF 8B433C             MOV     EAX,[EBX+3C]
0187:00576EF2 8B441878           MOV     EAX,[EAX+1*EBX+78]
0187:00576EF6 035C181C           ADD     EBX,[EAX+1*EBX+1C]
0187:00576EFA 8B041A             MOV     EAX,[EDX+1*EBX]
0187:00576EFD 0385A6AF4000       ADD     EAX,[EBP+0040AFA6]
0187:00576F03 EB13               JMP     00576F18
0187:00576F05 81E3FFFFFF7F      AND     EBX,7FFFFFFF
0187:00576F0B 53                 PUSH    EBX
0187:00576F0C FFB5A6AF4000       PUSH    DWord Ptr [EBP+0040AFA6]
0187:00576F12 FF9514AF4000       CALL    Near [`KERNEL32!GetProcAddress`]  //读取函数名
0187:00576F18 40                 INC     EAX
0187:00576F19 48                 DEC     EAX
0187:00576F1A 7532               JNZ     00576F4E
0187:00576F1C 58                 POP     EAX
0187:00576F1D F9                 STC
0187:00576F1E 0F829BFDFFFF       JB      00576CBF
0187:00576F24 47                 INC     EDI
0187:00576F25 44                 INC     ESP
0187:00576F26 49                 DEC     ECX
0187:00576F27 3332               XOR     ESI,[EDX]
0187:00576F29 2E44               INC     ESP
0187:00576F2B 4C                 DEC     ESP
0187:00576F2C 4C                 DEC     ESP
0187:00576F2D 55                 PUSH    EBP
0187:00576F2E 53                 PUSH    EBX
0187:00576F2F 45                 INC     EBP
0187:00576F30 52                 PUSH    EDX
0187:00576F31 3332               XOR     ESI,[EDX]
0187:00576F33 2E44               INC     ESP
0187:00576F35 4C                 DEC     ESP
0187:00576F36 4C                 DEC     ESP
0187:00576F37 53                 PUSH    EBX
0187:00576F38 48                 DEC     EAX
0187:00576F39 45                 INC     EBP
0187:00576F3A 4C                 DEC     ESP
0187:00576F3B 4C                 DEC     ESP
0187:00576F3C 3332               XOR     ESI,[EDX]
0187:00576F3E 2E44               INC     ESP
0187:00576F40 4C                 DEC     ESP
0187:00576F41 4C                 DEC     ESP
0187:00576F42 4B                 DEC     EBX
0187:00576F43 45                 INC     EBP
0187:00576F44 52                 PUSH    EDX
0187:00576F45 4E                 DEC     ESI
0187:00576F46 45                 INC     EBP
0187:00576F47 4C                 DEC     ESP
0187:00576F48 3332               XOR     ESI,[EDX]
0187:00576F4A 2E44               INC     ESP
0187:00576F4C 4C                 DEC     ESP
0187:00576F4D 4C                 DEC     ESP
0187:00576F4E 8907               MOV     [EDI],EAX               //把函数名写回去
0187:00576F50 58                 POP     EAX
0187:00576F51 48                 DEC     EAX
0187:00576F52 740D               JZ      00576F61
```



# FlashFXP 1.4 脱壳与反汇编分析

## 汇编片段

```
0187:00576F54 40               INC    EAX   
0187:00576F55 F8               CLC     
0187:00576F56 668943FE         MOV    [EBX-02],AX   
0187:00576F5A 8803             MOV    [EBX],AL   
0187:00576F5C 43               INC    EBX   
0187:00576F5D 3803             CMP    [EBX],AL   
0187:00576F5F 75F9             JNZ    00576F5A   
0187:00576F61 8385AAAF400004   ADD    DWord Ptr [EBP+0040AFAA],00000004  //继续下一个   
0187:00576F68 E9D4FDFFFF       JMP    00576D41   
0187:00576F6D 83C614           ADD    ESI,00000014   
0187:00576F70 8B95BEAF4000     MOV    EDX,[EBP+0040AFBE]   
0187:00576F76 E9B1FCFFFF       JMP    00576C2C   
0187:00576F7B 61               POPAD   
```

## 操作说明

抱歉，我的表达能力太差了，我只能读懂那段的意思，实在是说不出来，语文没学好:(，按我改的做吧，就可以把正确的函数写到正确的位置去，具体有兴趣自己跟一下吧，看看就知道了。

这样做完所有的函数都可以认出，但还有一个问题，这个同样也很难表达，就是该分段的地址还是被 `telock` 破坏掉了，就是原来是 `00000000` 的地方被 `telock` 写进了垃圾代码，`ImportREC` 这时就抓瞎了，还认为这有东西呢，所以我们必须把它还原，在原来的地址写回 `00000000`。如何找呢，这个我还没找到 easy 的方法，哪位大脱哥有好方法别保留呀，教教偶吧:)。

我的方法：在程序破坏 it 之前把 it 的结构导出，记下分段地址，然后在程序的领空 `bpm` 这个地址 `w`，在被 `telock` 破坏后，手工填回去。例如：`load` 程序，下 `bpm 870000`，下 `g`，先把我前面说的要改的地方改了，然后下 `a eip,jmp eip`，`f5`。接着用 `ImportREC` 提取未破坏的 it 结构，找到第一个分段处(我不知道这个改怎么叫)，是 `00533268`，然后按 `ctrl+d` 返回程序，把刚才改的 `jmp eip` 还原回去，下，`bpm 533268 w`，`g`，被拦到，下 `dd 533268` 把垃圾代码写回 `00000000`，以此类推，还好只需要改 22 次，不累吧，比手工填函数可快多了呀。最后按 `f5` 退出，用 `ImportREC` 重建 it 成功，别忘了把 `dump_.exe` 的 `oep` 改成 `0012B8B0`。

完活，哪位最后有其他好方法可用的话，千万记得告诉我呀，谢啦。

最后，论坛精华 3 中的脱壳文章不是太旧就是太少了，偶想把它充实些，不过力不从心，让我在短时期内搞这么多东西实在是太 xxxx 了。偶再过几天就要去学校报道了，这次要住校，tmd 又要去浪费 time 了，我不可能把我的 55gb 的资料全带过去，所以，唉......在此之前我能赶多少算多少吧，自己研究真的好麻烦:(。这种教育制度纯属是浪费 time，三个词送给它 shit,fuck,ft。

sorry 过于激动了，到此为止，写这些废话是因为刚收到的录取通知。

感谢你能够坚持下来 hoho:)

下载：`http://zombieys.cn.hongnet.com/FlashFXP14.rar`

最后贴个 key，`51/唐朝/精品`，都有提供，谢谢他们了:)

让我算这个 key?下辈子吧，不过偶倒是看到 `1.3` 的 keygen 了，mo god~~~~~~

## 注册数据

```
-------- FlashFXP Registration Data START --------

Susan Kennedy

N/A

sulusux@subspacemail.com

00000001

09-22-2001

4vUA6=EdIUAN9R+htrOcAcwOLmRKBi55oEj7aZ2W2Ua

mTLay0Gy=4lVgvE8T6SLhzUuUOzYiyphrY1z4soKJT+

jxByazfcFCRXXM7gIPjRCL3tf8gw+oKAHwaEtP+6vq

-------- FlashFXP Registration Data END --------
```

2001.9.26

`zombieys[CCG]`

## ASCII Art

```
                                      .-"       "-.       unpacked by zombieys[CCG] >
                                      /              \       qq:1789655               >
                                     |      ★       |   http://zombieys.yeah.net     >
                                     |,  .-.  .-.  ,|   http://zombieys.126.com     >
                                       |)(\__/ \__)(|     zombieys.cn.hongnet.com    >
                                       |/   /\    \|                           >
                 (@_@)   (_   ^^   _)     Thanks for your supports          >
      _   )\____\_____|IIIIII|__/\_______                                  >
_)@8@8{<______|-\IIIIII/|___China Crack Group_zombieys__
```

## FlashFXP v1.4 分析

`http://www.flashfxp.com`

scouting... `C:\Program Files\FlashFXP\FlashFXP.exe` - 756224 bytes big

Entry Point: `00176000` Image Base: `00400000`

Base of Data: `0012C000` Size of Image: `00178000`

Tools used:

- `Procdump`
- `Softice`
- `icedump + iceload`

看了 zombieys 的《脱 Flashfxp 1.4 的壳》一文，也下个 `FlashFXP v1.4` 练练。

还好比较简单，大家也可找来练练手脚。

仅供新手参考。

### 一，用 `tElock` 保护的 `FlashFXP` 中的反跟踪代码。

用 `bpint 3` 切入

```
017F:0057655E  8B642408          MOV      ESP,[ESP+08]    <-(1)-SEH proc address
017F:00576562  8B6C2408          MOV      EBP,[ESP+08]
017F:00576566  8D85FE040000      LEA      EAX,[EBP+000004FE]
017F:0057656C  50                PUSH     EAX
017F:0057656D  C3                RET     <-(1)-回到 017F:0057657B
017F:0057656E  2BC0              SUB      EAX,EAX
017F:00576570  7402              JZ       00576574
017F:00576572  CD20              INT      20 VXDJmp A7E9,10CC
017F:00576574  CC                INT      3    <-(1)-bpint 3 切入
017F:00576575  90                NOP
017F:00576576  E9A7000000        JMP      00576622    <--退出处理
017F:0057657B  8D8510050000      LEA      EAX,[EBP+00000510]
017F:00576581  89442404          MOV      [ESP+04],EAX
017F:00576585  646789260000      MOV      FS:[0000],ESP
017F:0057658B  EB10              JMP      0057659D    <-(1)-可按 F7 直接运行到此处
017F:0057658D  8B642408          MOV      ESP,[ESP+08]    <-(2)-SEH proc address
017F:00576591  8B6C2408          MOV      EBP,[ESP+08]
017F:00576595  8D852F050000      LEA      EAX,[EBP+0000052F]
017F:0057659B  50                PUSH     EAX
017F:0057659C  C3                RET     <-(2)-回到 017F:005765AC
017F:0057659D  2BDB              SUB      EBX,EBX
017F:0057659F  8BD3              MOV      EDX,EBX
017F:005765A1  B872472388        MOV      EAX,88234772
017F:005765A6  F7F3              DIV      EBX    <-(2)-除 0
017F:005765A8  85D2              TEST     EDX,EDX
017F:005765AA  7476              JZ       00576622    <--退出处理
017F:005765AC  8D8541050000      LEA      EAX,[EBP+00000541]
017F:005765B2  89442404          MOV      [ESP+04],EAX
017F:005765B6  646789260000      MOV      FS:[0000],ESP
017F:005765BC  EB10              JMP      005765CE    <-(2)-可按 F7 直接运行到此处
017F:005765BE  8B642408          MOV      ESP,[ESP+08]
017F:005765C2  8B6C2408          MOV      EBP,[ESP+08]
017F:005765C6  8D855F050000      LEA      EAX,[EBP+0000055F]
017F:005765CC  50                PUSH     EAX
017F:005765CD  C3                RET
017F:005765CE  66B80043          MOV      AX,4300    <-(3)-Method of detection of the WinICE handler in the int68h (V86)
017F:005765D2  CD68              INT      68
017F:005765D4  66057B0C          ADD      AX,0C7B
017F:005765D8  6648              DEC      AX    <-(3)-相当于 cmp   ax,0F386h
017F:005765DA  7446              JZ       00576622    <-(3)-是 00 那么 SoftICE_Detected,r fl z
017F:005765DC  8D8571050000       LEA      EAX,[EBP+00000571]
017F:005765E2  89442404          MOV      [ESP+04],EAX
017F:005765E6  646789260000      MOV      FS:[0000],ESP
017F:005765EC  EB10              JMP      005765FE
017F:005765EE  8B642408          MOV      ESP,[ESP+08]
017F:005765F2  8B6C2408          MOV      EBP,[ESP+08]
017F:005765F6  8D8599050000      LEA      EAX,[EBP+00000599]
017F:005765FC  50                PUSH     EAX
017F:005765FD  C3                RET
017F:005765FE  EB01              JMP      00576601    <--花指令
017F:00576601  33DB              XOR      EBX,EBX   
017F:00576603  8BC3              MOV      EAX,EBX    <-(4)-比较善良，没想超度你
017F:00576605  66BE4746          MOV      SI,4647    <-(4)-magic values
017F:00576609  66BF4D4A          MOV      DI,4A4D    <-(4)-magic values
017F:0057660D  CC                INT      3    <-(4)-常用的检测方式
017F:0057660E  F8                CLC
017F:0057660F  6681FE4746        CMP      SI,4647    <-(4)--SI=4647h 则 SoftICE_Detected
017F:00576614  750C              JNZ      00576622    <-(4)-存在 SoftICE 则退出,r fl z
017F:00576616  64678F060000      POP      DWORD PTR FS:[0000]
017F:0057661C  83C404            ADD      ESP,04
017F:0057661F  5D                POP      EBP
017F:00576620  EB16              JMP      00576638
017F:00576622  64678F060000      POP      DWORD PTR FS:[0000]
017F:00576628  83C404            ADD      ESP,04
017F:0057662B  5D                POP      EBP
017F:0057662C  81EDE59B4000      SUB      EBP,00409BE5
017F:00576632  E946090000        JMP      00576F7D
017F:00576637  C3                RET
```



# 017F:00576638 8BDD MOV EBX,EBP

## 二、用 tElock 保护的 FlashFXP 脱壳

### 跳过 tElock 保护的反跟踪代码

tELock 找 `OEP` 很简单：`bpx VirtualProtectEx` 中断后，稍微跟几步看见 `jmp [esp-30]`，就跳到 `OEP` 了。

```
/dump 400000 178000 c:\FFXP.exe
```

修改 `Entry Point:0052B8B0-00400000`，以及 `Number of Sections` 为原来的数值。

以下是破坏 `Import Table` 的特征代码:

```
017F:00576C16  8B95BEAF4000       MOV     EDX,[EBP+0040AFBE]

017F:00576C1C  8BB5AEAF4000       MOV     ESI,[EBP+0040AFAE]

017F:00576C22  85F6              TEST    ESI,ESI          <--首次调用时的 ESI 就是 Import Tabler 的 RVA

017F:00576C24  0F849D030000      JZ      00576FC7

017F:00576C2A  03F2              ADD     ESI,EDX          <-- EDX=00400000

017F:00576C2C  83A5AAB0400000    AND     DWORD PTR [EBP+0040B0AA],00

017F:00576C33  8B460C            MOV     EAX,[ESI+0C]

017F:00576C36  83660C00          AND     DWORD PTR [ESI+0C],00     <--开始把 IMAGE_DATA_DIRECTORY 中的 Name 清 0

017F:00576C3A  85C0              TEST    EAX,EAX

017F:00576C3C  0F8485030000      JZ      00576FC7
```

在清零之前可 `DUMP` 出完整的 `Import Table`，然后用 `Procdump` 导入 `Import Table`，再修改 `Import Table` 的 `RVA` 值为相应的值(`133000`)。

完工。

### 简便方法

1. 先对付反跟踪代码，再 `bpx VirtualProtectEx` 中断，`F12` 返回。
2. 找特征代码，记下地址。（`s 400000 l 00178000 8B 46 0C 83 66 0C 00 85 C0`）下次 `g` 到此处 `DUMP` 出完整的 `Import Table`。
3. 找到 `OEP`，并 `DUMP`，最后作修正。

以上方法在 `tElock8.0`、`9.0` 上试验通过。

## 目的：在内存中将 FlashFXP 未被破坏的 IT 完整地提取出来

（至于找 `TELock` 的 `OEP`、修复的方法等，`zombieys`、`hying`、`liotta` 已经讲得很详细了）

### 第一步：找到 IT 的位置

找 `IT` 自然要依赖于 `IT` 的结构特征。`IT` 是个数组，数组中的每个元素是如下的一个结构(`#include <winnt.h>`)，对应着引入的一个 `DLL`：

```c
typedef struct _IMAGE_IMPORT_DESCRIPTOR {
    union {
        DWORD  Characteristics;            // 0 for terminating null import descriptor
        DWORD  OriginalFirstThunk;         // RVA to original unbound IAT (PIMAGE_THUNK_DATA)
    };
    DWORD  TimeDateStamp;                  // 0 if not bound,
                                           // -1 if bound, and real date\timestamp
                                           //    in IMAGE_DIRECTORY_ENTRY_BOUND_IMPORT (new BIND)
                                           // O.W. date/time stamp of DLL  bound to (Old BIND)
    DWORD  ForwarderChain;                 // -1 if no forwarders
    DWORD  Name;                           //DLL name
    DWORD  FirstThunk;                     // RVA to IAT (if  bound this IAT has actual addresses)
} IMAGE_IMPORT_DESCRIPTOR;
```

`PE Loader/unpacker` 在加载 `DLL` 时，过程大致如下：

1. 先根据 `Name` 用 `LoadLibraryA` 加载一个 `DLL`；
2. 根据 `FirstThunk` 找到 `IMAGE_IMPORT_BY_NAME` 数组，用 `GetProcAddress` 取得该 `DLL` 中的相应函数的地址填入 `IAT`。`FirstThunk` 就是指向 `IAT`。

```c
typedef struct _IMAGE_IMPORT_BY_NAME {
    WORD   Hint;
    BYTE   Name[1];
} IMAGE_IMPORT_BY_NAME, *PIMAGE_IMPORT_BY_NAME;
```

找 `IT` 的位置的过程和上面的过程有些相反。利用调试器设 `bpx LoadLibraryA`、`bpx GetProcAddress` 断点，我们能够直接得到的条件是 `DLL` 文件名及该串的内存地址、每个函数的 `Hint` 和函数名的内存地址、`IAT` 的位置（即 `GetProcAddress` 的返回值所要写入的位置），这样就至少有 3 条途径可以找到 `IT` 的位置：

1. 根据 `DLL Name`（在看雪教学中已有详细描述）。
2. 根据 `Hint` 和函数名的地址。
3. 根据 `IAT` 的位置。

这 3 条途径用的是同一种方法（这是找 `IT` 的一种通用方法，并不限于 `TELock` 加的壳）。

#### 1、用 BPMB LoadLibraryA 设断点

用 `BPMB LoadLibraryA X DO "d *(esp+4)"` 设断点，看见加载 `oleaut32.dll`：

```
0023:00533DBC 61656C6F  32337475  6C6C642E  00000000      oleaut32.dll....
0023:00533DCC 69726156  43746E61  676E6168  70795465      VariantChangeTyp
0023:00533DDC 00784565  61560000  6E616972  706F4374      eEx...VariantCop
0023:00533DEC 646E4979  00000000  69726156  43746E61      yInd....VariantC
0023:00533DFC 7261656C  00000000  53737953  6E697274      lear....SysStrin
0023:00533E0C 6E654C67  00000000  46737953  53656572      gLen....SysFreeS
0023:00533E1C 6E697274  00000067  52737953  6C6C4165      tring...SysReAll
0023:00533E2C 7453636F  676E6972  006E654C  79530000      ocStringLen...Sy
```

#### 2、先尝试第一条途径

字符串 `"oleaut32.dll"` 的 `RVA` 为 `00133DBC`，于是在内存中搜索这个 `RVA`，希望能够找到存放有该 `RVA` 值的某个 `IMAGE_IMPORT_DESCRIPTOR` 结构，也就等于是找到了 `IT`。

```
:s cs:400000 l 178000 bc,3d,13,00

Pattern not found
```

没有找到。因为 `TELock` 已经把相应位置上的值清 `0` 了。（实际上也可以用下一个 `DLL` 的文件名来进行搜索，因为 `TELock` 此时尚未处理下一个 `DLL`，它不可能提前把下一个 `DLL` 的 `IMAGE_IMPORT_DESCRIPTOR` 结构清掉。）

#### 3、再尝试第 2 条途径

注意到第一个函数 `VariantChangeTypeEx` 的 `Hint` 的地址是 `00533DCA`，于是搜索 `RVA` 值 `00133DCA`，希望能够找到 `FirstThunk` 所指向的那个数组（其实该数组也就是 `IAT`），进而再找到 `FirstThunk` 的位置。

```
:s cs:400000 l 178000 ca,3d,13,00

Pattern found at 001B:0053328C (0013328C)
```

找到之后看一下 `IAT`。由于此时还未把 `GetProcAddress` 的返回值写入 `IAT`，所以此时 `IAT` 中存放的都是指向 `IMAGE_IMPORT_BY_NAME` 的指针（`RVA`）：

```
:dd 53328c

001B:0053328C 00133DCA  00133DE0  00133DF2  00133E02      .=...=...=...>..
001B:0053329C 00133E12  00133E22  00133E38  00000000      .>..">..8>......
001B:005332AC 00133E5A  00133E68  00133E76  00133E84      Z>..h>..v>...>..
001B:005332BC 00133E98  00000000  00133EBC  00133ECE      .>.......>...>..
001B:005332CC 00133EE2  00133EF6  00133F06  00133F14      .>...>...?...?..
001B:005332DC 00133F24  00133F36  00133F44  00133F58      $?..6?..D?..X?..
001B:005332EC 00133F70  00000000  00133F96  00133FA2      p?.......?...?..
001B:005332FC 00133FAE  00133FBA  00133FC6  00133FDC      .?...?...?...?..
```

#### 4、再看究竟是哪个 FirstThunk 指向了上面的 IAT

由于该 `IAT` 的 `RVA` 为 `0013328C`，故：

```
:s cs:400000 l 178000 8c,32,13,00

Pattern found at 001B:0053304C (0013304C)
```

#### 5、定位 IT 起始位置

这样就找到了 `oleaut32.dll` 所对应的 `FirstThunk` 的位置是 `0053304C`。在此位置附近看一下，就可以找到 `IT` 的起始位置为 `00533000`，长度为 `0x1CC`（问号表示未被映射的内存空间；最初的 4 个 `IMAGE_IMPORT_DESCRIPTOR` 结构已经被清除了一部分）：

```
001B:00532FE0 ????????  ????????  ????????  ????????      ................
001B:00532FF0 ????????  ????????  ????????  ????????      ................
001B:00533000 00000000  00000000  00000000  00000000      ................
001B:00533010 001331CC  00000000  00000000  00000000      .1..............
001B:00533020 00000000  0013326C  00000000  00000000      ....l2..........
001B:00533030 00000000  00000000  0013327C  00000000      ........|2......
001B:00533040 00000000  00000000  00000000  0013328C      .............2..
001B:00533050 00000000  00000000  00000000  00133E4C      ............L>..
001B:00533060 001332AC  00000000  00000000  00000000      .2..............
001B:00533070 00133EAE  001332C4  00000000  00000000      .>...2..........
001B:00533080 00000000  00133F88  001332F4  00000000      .....?...2......
001B:00533090 00000000  00000000  001346D4  0013349C      .........F...4..
```

### 第二步：找到破坏 IT 的代码

由于 `TELock` 清除了 `IT` 中指向 `DLL` 文件名的 `RVA`，故只要设个 `bpm` 断点，看它什么时候清这个文件名的，就可以找到破坏 `IT` 的代码。

第一个 `DLL` 文件名的 `RVA` 存放在 `0053300C` 处，故 `BPM 00533000+0C W do "dd 533000"`：

```
001B:00576C16  8B95BEAF4000       MOV     EDX,[EBP+0040AFBE]

001B:00576C1C  8BB5AEAF4000       MOV     ESI,[EBP+0040AFAE]

001B:00576C22  85F6              TEST    ESI,ESI

001B:00576C24  0F849D030000      JZ      00576FC7

001B:00576C2A  03F2              ADD     ESI,EDX

001B:00576C2C  83A5AAB0400000    AND     DWORD PTR [EBP+0040B0AA],00

001B:00576C33  8B460C            MOV     EAX,[ESI+0C]

001B:00576C36  83660C00          AND     DWORD PTR [ESI+0C],00      //清除指向每个 DLL 的 Name 的 RVA 值

001B:00576C3A  85C0              TEST    EAX,EAX

001B:00576C3C  0F8485030000      JZ      00576FC7

001B:00576C42  03C2              ADD     EAX,EDX

001B:00576C44  8BD8              MOV     EBX,EAX

001B:00576C46  50                PUSH    EAX

001B:00576C47  FF9518AF4000      CALL    [EBP+0040AF18]

001B:00576C4D  85C0              TEST    EAX,EAX

001B:00576C4F  0F858C000000      JNZ     00576CE1

001B:00576C55  53                PUSH    EBX
```

在第一次执行上面 `CS:00576C36` 处的 `AND` 之前将 `533000`～`535FFF` 之间的内容（共 `0x3000` 字节，不仅包含 `IT`，还包含 `IAT` 等）`dump` 出来就行了。

所以觉得 `TELock` 目前还没有 `ASProtect` 强

`4dd466d0e1e0cc123afbe504da2462ca`
