
# wkh文件加密器 4.9分析[分享]

- 【作者大名】YuanQiao
- 【使用工具】OD、ImportREC
- 【操作系统】Windows XP
- 【软件名称】wkh文件加密器 4.9
- 【下载地址】574K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3&6B7i4K6u0W2L8$3&6D9K9h3&6W2k6r3!0%4L8W2)9J5k6h3&6W2N6q4)9J5c8Y4y4G2k6Y4c8Q4x3V1j5J5x3o6f1&6z5q4)9J5k6h3S2@1L8b7`.`.
- 【加壳方式】UPX 0.89.6 - 1.02 / 1.05 - 1.24 -> Markus & Laszlo

## 软件简介

这是一个极好用的机密文件的小程序，是个绿色软件无需安装，他最大的特点是可以加密任何格式的文件，而且加密效果都很好。在新的版本中不但可以加密单个文件，还可以成批加密文件夹中的所有文件。使用方便，不学即会。是你轻松加密文件的首选。

## 破解声明

我是一只小菜鸟，偶得一点心得，愿与大家分享：）

## 内容

### 第一步脱壳

用OD载入程序停在这：

```
0050DAC0 > $  60            PUSHAD
0050DAC1   .  BE 00F04600   MOV ESI,wkh文件?0046F000
0050DAC6   .  8DBE 0020F9FF LEA EDI,DWORD PTR DS:[ESI+FFF92000]
0050DACC   .  57            PUSH EDI
0050DACD   .  83CD FF       OR EBP,FFFFFFFF
0050DAD0   .  EB 10         JMP SHORT wkh文件?0050DAE2
```

在命令窗口下`BP 0012FFA4`按F9运行。

```
0050DC1C   . /74 07         JE SHORT wkh文件?0050DC25
0050DC1E   . |8903          MOV DWORD PTR DS:[EBX],EAX
0050DC20   . |83C3 04       ADD EBX,4
0050DC23   .^|EB D8         JMP SHORT wkh文件?0050DBFD
0050DC25   > \FF96 6C1A1100 CALL DWORD PTR DS:[ESI+111A6C]
0050DC2B   >  61            POPAD
0050DC2C   .- E9 5A8EF0FF   JMP wkh文件?00416A8B------------》停在这,最后按一下F8。
```

---

```
00416A8B    6A 60           PUSH 60------------------------》停在这，在这里脱壳。
00416A8D    68 E8074400     PUSH wkh文件?004407E8
00416A92    E8 89150000     CALL wkh文件?00418020
00416A97    BF 94000000     MOV EDI,94
00416A9C    8BC7            MOV EAX,EDI
00416A9E    E8 6DF8FFFF     CALL wkh文件?00416310
00416AA3    8965 E8         MOV DWORD PTR SS:[EBP-18],ESP
00416AA6    8BF4            MOV ESI,ESP
00416AA8    893E            MOV DWORD PTR DS:[ESI],EDI
00416AAA    56              PUSH ESI
00416AAB    FF15 34934300   CALL DWORD PTR DS:[439334]               ; kernel32.GetVersionExA
```

脱壳之后不能运行，用ImportREC修复：

- OEP：`00016A8B`
- RAV：`00038FFC`
- 大小：`00000064C`

修复之后可运行，脱壳到此为止。

### 第二步注册

机器码为：`MFSTJOMGHW`

密码对照取数表：

```
0  1  2  3  4  5  6  7  8  9
48 41 4E 4F 45 4A 4C 42 46 50 57
```

```
M  F  S  T  J  O  M  G  H  W
4D 46 53 54 4A 4F 4D 47 48 57---->ASC码十六制数记为A
48 41 4E 4F 45 4A 4C 42 46 50---->用A对应减此数
5  5  5  5  5  5  1  5  2  7-----》这个就是我们要的注册码。（5555551527）
```

下面是用注册码算出机器码的算法：

```
00407CB0    33C9            XOR ECX,ECX
00407CB2    8A8C2E E0010000 MOV CL,BYTE PTR DS:[ESI+EBP+1E0]
00407CB9    51              PUSH ECX
00407CBA    56              PUSH ESI
00407CBB    8BCD            MOV ECX,EBP
00407CBD    E8 0EF9FFFF     CALL 111\_.004075D0
00407CC2    88842E E0010000 MOV BYTE PTR DS:[ESI+EBP+1E0],AL
00407CC9    46              INC ESI
00407CCA    83FE 0A         CMP ESI,0A
00407CCD  ^ 7C E1           JL SHORT 111\_.00407CB0
```

---

```
004075D0    8D4424 08       LEA EAX,DWORD PTR SS:[ESP+8]
004075D4    50              PUSH EAX
004075D5    E8 C2EF0000     CALL 111\_.0041659C
004075DA    8B4C24 08       MOV ECX,DWORD PTR SS:[ESP+8]
004075DE    83C4 04         ADD ESP,4
004075E1    83F9 09         CMP ECX,9
004075E4    77 43           JA SHORT 111\_.00407629
004075E6    FF248D 30764000 JMP DWORD PTR DS:[ECX\*4+407630]     ; 111\_.004075ED
004075ED    83C0 48         ADD EAX,48
004075F0    C2 0800         RETN 8
004075F3    83C0 41         ADD EAX,41
004075F6    C2 0800         RETN 8
004075F9    83C0 4E         ADD EAX,4E
004075FC    C2 0800         RETN 8
004075FF    83C0 4F         ADD EAX,4F
00407602    C2 0800         RETN 8
00407605    83C0 45         ADD EAX,45
00407608    C2 0800         RETN 8
0040760B    83C0 4A         ADD EAX,4A
0040760E    C2 0800         RETN 8
00407611    83C0 4C         ADD EAX,4C
00407614    C2 0800         RETN 8
00407617    83C0 42         ADD EAX,42
0040761A    C2 0800         RETN 8
0040761D    83C0 46         ADD EAX,46
00407620    C2 0800         RETN 8
00407623    83C0 50         ADD EAX,50
00407626    C2 0800         RETN 8
00407629    B0 57           MOV AL,57
0040762B    C2 0800         RETN 8
```

---

如果想继续玩注册的可以把`X:\WINDOWS\system32\wegmigdl.dll`这个文件删除。

下面是WIN-TC程序：

```
#include "Stdio.h"
#include "Conio.h"
#include "string.h"

int main(void)
{
char string[]= {"HANOEJLBFP"};
char str2[20];
char str3[20];
int i;
printf("请输入机器码(一般是大写字母）：");
scanf ("%s",str2);
printf("注册码为:");
for (i=0;i<strlen(string);i++)
    {
     str3[i]=str2[i]-string[i];
     printf("%d",str3[i]);
    }
  getch();
  return 0;
}
```

这是第一次编程，编的不好还请各位大鸟指点。

---

## 版权声明

本文纯属技术交流, 转载请注明作者并保持文章的完整, 谢谢!

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)
