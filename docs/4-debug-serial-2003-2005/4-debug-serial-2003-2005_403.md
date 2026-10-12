
# 家庭电脑相册系统标准版 6

> 原文：`www.pediy.com/kssd/pediy06/pediy6951.htm`

**【破解作者】** `mejy`、`BCG`、`DFCG`、`FCG`

**【作者邮箱】** `yjychao@sohu.com`

**【使用工具】** `OD`、`FI`、`W32ASM`

**【破解平台】** `Win9x/NT/2000/XP`

**【软件名称】** 家庭相册制作系统标准版 `B60`

**【下载地址】** `http://www.skycn.com/soft/2631.html`

**【软件简介】** 本软件是为家庭电脑用户搜集图片和珍藏摄影照片而专门开发的应用工具软件。该软件易学易用，其制作的多媒体电脑相册具有图片检索、自动播放和音乐欣赏等多种功能，并能脱离原制作系统独立运行。将生成的相册刻录成光盘，更可永久收藏。当然，你也可以配合其它软件，制作出游戏画面集锦、风景名胜集锦、百科知识宝典、明星集锦和毕业纪念册等多媒体光盘。企事业单位也可以利用它收藏管理图片档案资料。十分钟成为电脑相册制作大师已不是梦想。

**【软件大小】** `956K`

**【加壳方式】** `aspack2.12`

**【破解声明】** 我是一只小菜鸟，偶得一点心得，愿与大家分享：）如有雷同，实属巧合！献丑了！

---

## 【破解内容】

### 家庭相册制作系统算法分析

利用 `FI` 检测主程序 `Albummake.exe`，利用 `Aspack` 加壳，利用自动脱壳或手动脱壳轻松脱去，检测发现是 `DELPHI` 编写的，我们进入程序试着输入试练码 `123456789－123` 发现程序提示"注册失败"，我们利用 `DEDE` 反汇编，这里利用 `W32ASM` 来反汇编，利用"字符串数据参考"找到"注册失败"的提示，我们双击来到 `W32ASM`，我们向上找到下面的位置，好了。下面利用 `OD` 来动态调试：

```assembly
004A51D8  /. 55             PUSH EBP
004A51D9  |. 8BEC           MOV EBP,ESP
………………………………………………………….省略
004A51ED  |. 55             PUSH EBP
。。。。。。。。。。。。。。。。。。。。。。。。。。。。。。。。。。。。。。。。。省略部分
004A524A  |. 8D4D F8        LEA ECX,DWORD PTR SS:[EBP-8]
004A524D  |. 5A             POP EDX
004A524E  |. E8 4122FFFF    CALL unpak.00497494 注册码计算关键函数跟进※※※
004A5253  |. 8D95 6CFEFFFF  LEA EDX,DWORD PTR SS:[EBP-194]
004A5259  |. 8B83 1C070000  MOV EAX,DWORD PTR DS:[EBX+71C]
。。。。。。。。。。。。。。。。。。。。。。。。。。省略部分
004A5285  |. E8 1238F6FF    CALL unpak.00408A9C
004A528A  |. 8B95 68FEFFFF  MOV EDX,DWORD PTR SS:[EBP-198]
004A5290  |. 58             POP EAX  将输入的注册码第二步分入栈
004A5291  |. E8 22F8F5FF    CALL unpak.00404AB8 比较正确的注册码与第二步分
004A5296     0F85 A8010000  JNZ unpak.004A5444
004A529C  |. 8D95 60FEFFFF  LEA EDX,DWORD PTR SS:[EBP-1A0]
004A52A2  |. A1 9C9B4A00    MOV EAX,DWORD PTR DS:[4A9B9C]
004A52A7  |. 8B00           MOV EAX,DWORD PTR DS:[EAX]
004A52A9  |. E8 2AA4FBFF    CALL unpak.0045F6D8
004A52AE  |. 8B85 60FEFFFF  MOV EAX,DWORD PTR SS:[EBP-1A0]
004A52B4  |. 8D95 64FEFFFF  LEA EDX,DWORD PTR SS:[EBP-19C]
004A52BA  |. E8 1D3FF6FF    CALL unpak.004091DC
004A52BF  |. 8B85 64FEFFFF  MOV EAX,DWORD PTR SS:[EBP-19C]
004A52C5  |. 8D55 FC        LEA EDX,DWORD PTR SS:[EBP-4]
004A52C8  |. E8 CF37F6FF    CALL unpak.00408A9C
004A52CD  |. 8D85 5CFEFFFF  LEA EAX,DWORD PTR SS:[EBP-1A4]
004A52D3  |. B9 4C554A00    MOV ECX,unpak.004A554C           ;  ASCII "data3"
```

将注册成功后的数据写入安装目录下的 `data3` 中。

```assembly
004A52D8  |. 8B55 FC        MOV EDX,DWORD PTR SS:[EBP-4]
```

以下略。

### ※※※※※※※※※※※※※※※※ 注册码计算函数 ※※※※※※※※※※※※※※※※

```assembly
00497494  /$ 55            PUSH EBP
00497495  |. 8BEC          MOV EBP,ESP
00497497  |. 51            PUSH ECX
00497498  |. B9 07000000   MOV ECX,7
0049749D  |> 6A 00         /PUSH 0
0049749F  |. 6A 00         |PUSH 0
004974A1  |. 49            |DEC ECX
004974A2  |. 75 F9         JNZ SHORT unpak.0049749D
004974A4  |. 874D FC       XCHG DWORD PTR SS:[EBP-4],ECX
004974A7  |. 53            PUSH EBX
004974A8  |. 56            PUSH ESI
004974A9  |. 8BF1          MOV ESI,ECX
004974AB  |. 8955 F8       MOV DWORD PTR SS:[EBP-8],EDX

; 输入注册码的前一部分

004974AE  |. 8945 FC       MOV DWORD PTR SS:[EBP-4],EAX EAX 为机器号
004974B1  |. 8B45 FC       MOV EAX,DWORD PTR SS:[EBP-4]
004974B4  |. E8 A3D6F6FF   CALL unpak.00404B5C
004974B9  |. 8B45 F8       MOV EAX,DWORD PTR SS:[EBP-8]
004974BC  |. E8 9BD6F6FF   CALL unpak.00404B5C
004974C1  |. 33C0          XOR EAX,EAX
004974C3  |. 55            PUSH EBP
004974C4  |. 68 BD764900  PUSH unpak.004976BD
004974C9  |. 64:FF30       PUSH DWORD PTR FS:[EAX]
004974CC  |. 64:8920       MOV DWORD PTR FS:[EAX],ESP
004974CF  |. 8D45 F4       LEA EAX,DWORD PTR SS:[EBP-C]
004974D2  |. 50            PUSH EAX
004974D3  |. 8D55 E8       LEA EDX,DWORD PTR SS:[EBP-18]
004974D6  |. 8B45 FC       MOV EAX,DWORD PTR SS:[EBP-4]
004974D9  |. E8 BE15F7FF   CALL unpak.00408A9C
004974DE  |. 8B45 E8       MOV EAX,DWORD PTR SS:[EBP-18]
004974E1  |. B9 0B000000   MOV ECX,0B  参数
004974E6  |. BA 03000000   MOV EDX,3    参数
004974EB  |. E8 DCD6F6FF   CALL unpak.00404BCC

; 从机器号的第三位开始取长度为 0xB 的字符串。我得 ASCII "49602621006"
; 下面多次调用了这个函数进行求子字符串，记结果为字符串①

004974F0  |. 8D45 F0       LEA EAX,DWORD PTR SS:[EBP-10]
004974F3  |. 50            PUSH EAX
004974F4  |. B9 06000000   MOV ECX,6
004974F9  |. BA 06000000   MOV EDX,6
004974FE  |. 8B45 F4       MOV EAX,DWORD PTR SS:[EBP-C]
00497501  |. E8 C6D6F6FF   CALL unpak.00404BCC ; 从"49602621006"字符串中的第六位取长度为 6 的字符串，因为 DELPHI 的数组下标从 1 开始的所以结果为"621006"字符串②
00497506  |. 8B45 F0       MOV EAX,DWORD PTR SS:[EBP-10]

; EAX 中是上面的字符串，利用下面的函数将字符串转化成数字并且转化成十进制结果保存在 EAX 中

00497509  |. E8 BE18F7FF   CALL unpak.00408DCC ; 将机器号得后六位转化成十六进制
0049750E  |. 8BD8          MOV EBX,EAX
00497510  |. 8BC3          MOV EAX,EBX
00497512  |. B9 03000000   MOV ECX,3
00497517  |. 99            CDQ
00497518  |. F7F9          IDIV ECX ; 将上面得十六进制数除以 3 商保存在 EAX
0049751A  |. 8BD8          MOV EBX,EAX ; 将商转移给 EBX 中
0049751C  |. EB 06         JMP SHORT unpak.00497524
0049751E  |> 8BC3          /MOV EAX,EBX
00497520  |. 03C0          |ADD EAX,EAX
00497522  |. 8BD8          |MOV EBX,EAX
00497524  |> 8D55 E4       LEA EDX,DWORD PTR SS:[EBP-1C]
00497527  |. 8BC3          |MOV EAX,EBX
00497529  |. E8 BE17F7FF   |CALL unpak.00408CEC ; 将商 EBX 中数转化成 10 进制并且转化为字符串 ASCII "207002" 记为字符串③
0049752E  |. 8B45 E4       |MOV EAX,DWORD PTR SS:[EBP-1C]
00497531  |. E8 3ED4F6FF   |CALL unpak.00404974 ; 取得字符串的长度
00497536  |. 83F8 06       |CMP EAX,6    ; 和 6 比较
00497539  |. 7C E3         JL SHORT unpak.0049751E

; 如果小于 6 跳回，将上面计算的余数乘 2，接着转化直到其长度大于等于 6

0049753B  |. 8D55 F0       LEA EDX,DWORD PTR SS:[EBP-10]
0049753E  |. 8BC3          MOV EAX,EBX
00497540  |. E8 A717F7FF   CALL unpak.00408CEC
00497545  |. 8D45 F0       LEA EAX,DWORD PTR SS:[EBP-10]
00497548  |. 50            PUSH EAX
00497549  |. B9 06000000   MOV ECX,6
0049754E  |. BA 01000000   MOV EDX,1
00497553  |. 8B45 F0       MOV EAX,DWORD PTR SS:[EBP-10]
00497556  |. E8 71D6F6FF   CALL unpak.00404BCC ; 如果上面的字符串长度超过 6 则从第一个位置开始取长度为 6 的字符串，这里长度刚好等于 6 所以结果为"207002"
0049755B  |. 8D45 EC       LEA EAX,DWORD PTR SS:[EBP-14]
0049755E  |. 50            PUSH EAX
0049755F  |. B9 05000000   MOV ECX,5
00497564  |. BA 01000000   MOV EDX,1
00497569  |. 8B45 F4       MOV EAX,DWORD PTR SS:[EBP-C]
0049756C  |. E8 5BD6F6FF   CALL unpak.00404BCC

; 从机器号去掉 B6 后的第 1 位开始取长度为 5 的字符串。也就是剩余的字符串 0ASCII "49602"
; 字符串④

00497571  |. 8D55 E0       LEA EDX,DWORD PTR SS:[EBP-20]
00497574  |. 8B45 F8       MOV EAX,DWORD PTR SS:[EBP-8]

; 看见输入注册吗的第一部分"123456789"

00497577  |. E8 2015F7FF   CALL unpak.00408A9C
0049757C  |. 8B45 E0       MOV EAX,DWORD PTR SS:[EBP-20]
0049757F  |. E8 4818F7FF   CALL unpak.00408DCC ; 将上面的数字转化为十六进制数
00497584  |. 8BD8          MOV EBX,EAX ; 转化的结果保存在 EBX 中
00497586  |. 8B45 EC       MOV EAX,DWORD PTR SS:[EBP-14]
00497589  |. E8 3E18F7FF   CALL unpak.00408DCC ; 将上面的字符串 4 转化为十六进制
0049758E  |. 03D8          ADD EBX,EAX ; 将上面转化后的数相加结果保存在 EBX 中
00497590  |. D1FB          SAR EBX,1 ; 算术右移 1 位，也就是将 EBX/2
00497592  |. 79 0A         JNS SHORT unpak.0049759E
00497594  |. 83D3 00       ADC EBX,0
00497597  |. EB 05         JMP SHORT unpak.0049759E
00497599  |> 8D045B        /LEA EAX,DWORD PTR DS:[EBX+EBX*2]
0049759C  |. 8BD8          |MOV EBX,EAX
0049759E  |> 8D55 DC       LEA EDX,DWORD PTR SS:[EBP-24]
004975A1  |. 8BC3          |MOV EAX,EBX
004975A3  |. E8 4417F7FF   |CALL unpak.00408CEC
```



转化成十进制字符串 ASCII "61753195"

```
004975A8  .| 8B45 DC        |MOV EAX,DWORD PTR SS:[EBP-24]
004975AB  .| E8 C4D3F6FF    |CALL unpak.00404974 得到字符串的长度
004975B0  .| 83F8 06        |CMP EAX,6
004975B3  .| 7C E4          |JL SHORT unpak.00497599 小于 6 跳走
```

如果长度小于 6 则将 `EBX` 的值乘上 3 继续转化

```
004975B5  .| 8D55 EC        |LEA EDX,DWORD PTR SS:[EBP-14]
004975B8  .| 8BC3           |MOV EAX,EBX
004975BA  .| E8 2D17F7FF    |CALL unpak.00408CEC
004975BF  .| 8D45 EC        |LEA EAX,DWORD PTR SS:[EBP-14]
004975C2  .| 50             |PUSH EAX
004975C3  .| B9 06000000    |MOV ECX,6
004975C8  .| BA 01000000    |MOV EDX,1
004975CD  .| 8B45 EC        |MOV EAX,DWORD PTR SS:[EBP-14]
004975D0  .| E8 F7D5F6FF    |CALL unpak.00404BCC 因为上面字符串长度为 8>6 所以取上面字符串从第一位开始长度为 6 的子字符串结果为 ASCII "617531"
004975D5  .| 8D45 F4        |LEA EAX,DWORD PTR SS:[EBP-C]
004975D8  .| 8B4D EC        |MOV ECX,DWORD PTR SS:[EBP-14]
004975DB  .| 8B55 F0        |MOV EDX,DWORD PTR SS:[EBP-10]取前 6 位
004975DE  .| E8 DDD3F6FF    |CALL unpak.004049C0 将上面求得的字符串连接起来
0012F9AC  0119B724  ASCII "207002617531"
004975E3  .| 8D45 F0        |LEA EAX,DWORD PTR SS:[EBP-10]
004975E6  .| E8 D1D0F6FF    |CALL unpak.004046BC
004975EB  .| 8D45 EC        |LEA EAX,DWORD PTR SS:[EBP-14]
004975EE  .| E8 C9D0F6FF    |CALL unpak.004046BC
004975F3  .| BB 01000000    |MOV EBX,1
```

下面计算注册吗的第一部分，前面取偶数位连接起来

```
004975F8  |> 8BC3           |/MOV EAX,EBX
004975FA  .| D1F8           |SAR EAX,1 算术右移 1 位 EAX＝EAX/2
004975FC  .| 79 03          |JNS SHORT unpak.00497601
004975FE  .| 83D0 00        |ADC EAX,0
00497601  |> 03C0           |ADD EAX,EAX 也就是 EAX*2 因为大家知道在 C++或者 DELPHI 中，如果对一个整数进行先除以 2 再乘 2，若是偶数其结果和原来数据的相同，若是奇数则结果不想等，这里就是根据这个来得
00497603  .| 3BD8           |CMP EBX,EAX
00497605  .| 75 37          |JNZ SHORT unpak.0049763E
00497607  .| 8D45 D0        |LEA EAX,DWORD PTR SS:[EBP-30]
0049760A  .| 50             |PUSH EAX
0049760B  .| B9 01000000    |MOV ECX,1
00497610  .| 8BD3           |MOV EDX,EBX
00497612  .| 8B45 F4        |MOV EAX,DWORD PTR SS:[EBP-C]
00497615  .| E8 B2D5F6FF    |CALL unpak.00404BCC
0049761A  .| 8B4D D0        |MOV ECX,DWORD PTR SS:[EBP-30]
0049761D  .| 8D45 D4        |LEA EAX,DWORD PTR SS:[EBP-2C]
00497620  .| 8B55 F0        |MOV EDX,DWORD PTR SS:[EBP-10]
00497623  .| E8 98D3F6FF    |CALL unpak.004049C0 连接函数
00497628  .| 8B45 D4        |MOV EAX,DWORD PTR SS:[EBP-2C]
0049762B  .| 8D55 D8        |LEA EDX,DWORD PTR SS:[EBP-28]
0049762E  .| E8 6914F7FF    |CALL unpak.00408A9C
00497633  .| 8B55 D8        |MOV EDX,DWORD PTR SS:[EBP-28]
00497636  .| 8D45 F0        |LEA EAX,DWORD PTR SS:[EBP-10]
00497639  .| E8 16D1F6FF    |CALL unpak.00404754 连接求得的每位字符
0049763E  |> 43             |INC EBX
0049763F  .| 83FB 0D        |CMP EBX,0D 看看是否已经完成
00497642  .| 75 B4          |JNZ SHORT unpak.004975F8
00497644  .| BB 01000000    |MOV EBX,1
```

取字符串的偶数位（注意第一位下标为 1），并逐位连接结果为： ASCII "002151"

```
00497649  |> 8BC3           |/MOV EAX,EBX
0049764B  .| D1F8           |SAR EAX,1 右移 1 位
0049764D  .| 79 03          |JNS SHORT unpak.00497652
0049764F  .| 83D0 00        |ADC EAX,0
00497652  |> 03C0           |ADD EAX,EAX
00497654  .| 3BD8           |CMP EBX,EAX 和上面一样这里取奇数位然后连接
00497656  .| 74 37          |JE SHORT unpak.0049768F
00497658  .| 8D45 C4        |LEA EAX,DWORD PTR SS:[EBP-3C]
0049765B  .| 50             |PUSH EAX
0049765C  .| B9 01000000    |MOV ECX,1
00497661  .| 8BD3           |MOV EDX,EBX
00497663  .| 8B45 F4        |MOV EAX,DWORD PTR SS:[EBP-C]
00497666  .| E8 61D5F6FF    |CALL unpak.00404BCC
0049766B  .| 8B4D C4        |MOV ECX,DWORD PTR SS:[EBP-3C]
0049766E  .| 8D45 C8        |LEA EAX,DWORD PTR SS:[EBP-38]
00497671  .| 8B55 EC        |MOV EDX,DWORD PTR SS:[EBP-14]
00497674  .| E8 47D3F6FF    |CALL unpak.004049C0
00497679  .| 8B45 C8        |MOV EAX,DWORD PTR SS:[EBP-38]
0049767C  .| 8D55 CC        |LEA EDX,DWORD PTR SS:[EBP-34]
0049767F  .| E8 1814F7FF    |CALL unpak.00408A9C
00497684  .| 8B55 CC        |MOV EDX,DWORD PTR SS:[EBP-34]
00497687  .| 8D45 EC        |LEA EAX,DWORD PTR SS:[EBP-14]
0049768A  .| E8 C5D0F6FF    |CALL unpak.00404754
0049768F  |> 43             |INC EBX
00497690  .| 83FB 0D        |CMP EBX,0D 比较
00497693  .| 75 B4          |JNZ SHORT unpak.00497649 循环 2
```

结果位 ASCII "270673"

```
00497695  .| 8BC6           |MOV EAX,ESI
00497697  .| 8B4D EC        |MOV ECX,DWORD PTR SS:[EBP-14]
0049769A  .| 8B55 F0        |MOV EDX,DWORD PTR SS:[EBP-10]
0049769D  .| E8 1ED3F6FF    |CALL unpak.004049C0 将上面两部分连接起来
004976A2  .| 33C0           |XOR EAX,EAX
004976A4  .| 5A             |POP EDX
004976A5  .| 59             |POP ECX
004976A6  .| 59             |POP ECX
004976A7  .| 64:8910        |MOV DWORD PTR FS:[EAX],EDX
004976AA  .| 68 C4764900    |PUSH unpak.004976C4
004976AF  |> 8D45 C4        |LEA EAX,DWORD PTR SS:[EBP-3C]
004976B2  .| BA 0F000000    |MOV EDX,0F
004976B7  .| E8 24D0F6FF    |CALL unpak.004046E0
004976BC  .  C3             |RETN
```

--------------------------------------------------------------------------------

## 破解总结

算法总结：先得到软件序列号，然后去掉编号的前两位得到一个新的字符串 `1`，从字符串 `1` 的第 `1` 个开始 `5` 个字符作为第一部分记为字符串 `A`，从第 `6` 位开始的 `6` 个字符做为第二部分记为字符串 `B`，如果还有多余的字符串则抛弃掉。然后将 `B` 转化为十六进制，除以 `3` 的商在转化为十进制的字符串 `C`，并通过转化将 `C` 串的长度最终大于等于 `6`，若 `>6` 再取前六个字符记为 `G`。接着取输入注册码的第一部分数字转化成十六进制，接着将字符串 `A` 转化成十六进制，并将这两个数加起来除以 `2`，再转化成十进制字符串，取前六位记为 `H`，然后将 `G` 和 `H` 连接起来记为 `M`。然后将 `M` 按顺序先取所有偶数位，再取所有奇数位，重新连接成新的字符串，这就是你的注册码的第二部分。

注册信息保存在安装目录下的 `data3` 文件中。删去该文件可重新注册。

--------------------------------------------------------------------------------

## 版权声明

本文纯属技术交流，转载请注明作者并保持文章的完整，谢谢！

`4dd466d0e1e0cc123afbe504da2462ca`
