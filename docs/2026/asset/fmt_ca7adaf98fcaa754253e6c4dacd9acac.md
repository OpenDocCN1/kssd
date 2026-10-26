
`004B47F3  |.  8B45 DC       |MOV EAX,DWORD PTR SS:[EBP-24]`

`004B47F6  |.  8BD0          |MOV EDX,EAX`

`004B47F8  |.  A1 30735200   |MOV EAX,DWORD PTR DS:[527330]`

`004B47FD  |.  8B00          |MOV EAX,DWORD PTR DS:[EAX]`

`004B47FF  |.  B9 01000000   |MOV ECX,1                               ; bytes = 1`

`004B4804  |.  8B38          |MOV EDI,DWORD PTR DS:[EAX]`

`004B4806  |.  FF57 04       |CALL DWORD PTR DS:[EDI+4]               ; get_buf(buf, bytes)?`

`004B4809  |.  46            |INC ESI                                 ; i++;`

`004B480A  |.  FF45 DC       |INC DWORD PTR SS:[EBP-24]               ; buf++;`

`004B480D  |.  83FE 0A       |CMP ESI,0A                              ; i<10?`

`004B4810  |.^ 75 C5         \JNZ SHORT SuperSok.004B47D7             ; loop`

`004B4812  |.  8D4D A0       LEA ECX,DWORD PTR SS:[EBP-60]            ; &result`

`004B4815  |.  8D45 E6       LEA EAX,DWORD PTR SS:[EBP-1A]            ; buf`

`004B4818  |.  66:BA 1700    MOV DX,17`

`004B481C  |.  E8 5F210400   CALL SuperSok.004F6980                   ; 解密 buf`

`004B4821  |.  8B45 A0       MOV EAX,DWORD PTR SS:[EBP-60]            ; result`

`004B4824  |.  8B15 FC6E5200 MOV EDX,DWORD PTR DS:[526EFC]            ;`

`004B482A  |.  8B12          MOV EDX,DWORD PTR DS:[EDX]`

`004B482C  |.  E8 5BF8F4FF   CALL SuperSok.0040408C                   ; CString_Compare(result, xx)?`

`004B4831  |.  74 26         JE SHORT SuperSok.004B4859               ; // 必须跳`

`004B4833  |.  8B0D 58715200 MOV ECX,DWORD PTR DS:[527158]            ; SuperSok.00ADEF9C`

`004B4839  |.  8B09          MOV ECX,DWORD PTR DS:[ECX]`

`004B483B  |.  8B15 A86F5200 MOV EDX,DWORD PTR DS:[526FA8]            ; SuperSok.00ADEFD0`

`004B4841  |.  8B12          MOV EDX,DWORD PTR DS:[EDX]`

`004B4843  |.  B8 03000000   MOV EAX,3`

`004B4848  |.  E8 5B220400   CALL SuperSok.004F6AA8`

`004B484D  |.  A1 18725200   MOV EAX,DWORD PTR DS:[527218]`

`004B4852  |.  8B00          MOV EAX,DWORD PTR DS:[EAX]`

`004B4854  |.  E8 33A2F9FF   CALL SuperSok.0044EA8C`

`004B4859  |>  8B83 2C030000 MOV EAX,DWORD PTR DS:[EBX+32C]`

`004B485F  |.  8B80 44020000 MOV EAX,DWORD PTR DS:[EAX+244]`

*/

跟踪结果:

a. 从文件offset 4600 取200 Bytes

b. 每 20 bytes 中取 第1 BYTE, 构成一长为10的BUF;

c. 10 Bytes, CALL `004F6980` 解密

d. 结果与一串相比, 必须一样,

e. 类似的CODE好多, 只有 OFFSET 与 index 有区别.

相比较的串是这样取出来的:

`004B4824  |.  8B15 FC6E5200 MOV EDX,DWORD PTR DS:[526EFC]`

`004B482A  |.  8B12          MOV EDX,DWORD PTR DS:[EDX]`

在 `OLLYDBG` 中 Search constants `526EFC`, 可找到下面的CODE

/*
`0050D141   .  A1 48705200   MOV EAX,DWORD PTR DS:[527048]`

`0050D146   .  C600 00       MOV BYTE PTR DS:[EAX],0`

`0050D149   .  6A 00         PUSH 0`

`0050D14B   .  8D45 F4       LEA EAX,DWORD PTR SS:[EBP-C]             ;  buf for RegName`

`0050D14E   .  50            PUSH EAX`

`0050D14F   .  A1 4C735200   MOV EAX,DWORD PTR DS:[52734C]`

`0050D154   .  8B00          MOV EAX,DWORD PTR DS:[EAX]`

`0050D156   .  B9 4CE35000   MOV ECX,SuperSok.0050E34C                ;  ASCII "Name"`

`0050D15B   .  BA E4E45000   MOV EDX,SuperSok.0050E4E4                ;  ASCII "RegName"`

`0050D160   .  E8 D31EF7FF   CALL SuperSok.0047F038                   ;  ReadFromRegister(...) // read RegName`

`0050D165   .  6A 00         PUSH 0`

`0050D167   .  8D85 30FFFFFF LEA EAX,DWORD PTR SS:[EBP-D0]            ;  buf for RegEmail`

`0050D16D   .  50            PUSH EAX`

`0050D16E   .  A1 4C735200   MOV EAX,DWORD PTR DS:[52734C]`

`0050D173   .  8B00          MOV EAX,DWORD PTR DS:[EAX]`

`0050D175   .  B9 F4E45000   MOV ECX,SuperSok.0050E4F4                ;  ASCII "Email"`

`0050D17A   .  BA 04E55000   MOV EDX,SuperSok.0050E504                ;  ASCII "RegEmail"`

`0050D17F   .  E8 B41EF7FF   CALL SuperSok.0047F038                   ;  ReadFreomRegister(...) // read RegEmail`

`0050D184   .  8B95 30FFFFFF MOV EDX,DWORD PTR SS:[EBP-D0]            ;  str_Email`

`0050D18A   .  8D45 F4       LEA EAX,DWORD PTR SS:[EBP-C]             ;  str_Name`

`0050D18D   .  E8 F26DEFFF   CALL SuperSok.00403F84                   ;  strNameEmail = strcat(Name, Email)`

`0050D192   .  837D F4 00    CMP DWORD PTR SS:[EBP-C],0`

`0050D196   .  0F84 CB010000 JE SuperSok.0050D367`

`0050D19C   .  8B45 F4       MOV EAX,DWORD PTR SS:[EBP-C]`

`0050D19F   .  E8 6097FEFF   CALL SuperSok.004F6904                   ;  magic_num = Func1(strNameEmail);`

`0050D1A4   .  8D95 2CFFFFFF LEA EDX,DWORD PTR SS:[EBP-D4]`

`0050D1AA   .  E8 BD96FEFF   CALL SuperSok.004F686C                   ;  sprintf(magic_buf, "%10u", magic_num);`

`0050D1AF   .  8B95 2CFFFFFF MOV EDX,DWORD PTR SS:[EBP-D4]`

`0050D1B5   .  A1 FC6E5200   MOV EAX,DWORD PTR DS:[526EFC]            ;  String @ [526EFC]  *****************`

`0050D1BA   .  E8 916BEFFF   CALL SuperSok.00403D50                   ;  String.buf=magic_buf;`

`0050D1BF   .  A1 206F5200   MOV EAX,DWORD PTR DS:[526F20]`

`0050D1C4   .  FF30          PUSH DWORD PTR DS:[EAX]`

`0050D1C6   .  68 18E55000   PUSH SuperSok.0050E518                   ;  ASCII "fsk"`

`0050D1CB   .  6A 00         PUSH 0`

`0050D1CD   .  6A 7B         PUSH 7B`

`0050D1CF   .  6A 00         PUSH 0`

`0050D1D1   .  68 A0860100   PUSH 186A0                               ;  100000`

`0050D1D6   .  A1 FC6E5200   MOV EAX,DWORD PTR DS:[526EFC]            ;  String @[526EFC]`

`0050D1DB   .  8B00          MOV EAX,DWORD PTR DS:[EAX]               ;  String.buf`

`0050D1DD   .  E8 0ABFEFFF   CALL SuperSok.004090EC                   ;  magic_num = sscanf("%10u", buf);`

`0050D1E2   .  E8 EA99EFFF   CALL SuperSok.00406BD1                   ;  magic_num%10000`

`0050D1E7   .  8985 20FFFFFF MOV DWORD PTR SS:[EBP-E0],EAX`

`0050D1ED   .  8995 24FFFFFF MOV DWORD PTR SS:[EBP-DC],EDX`

`0050D1F3   .  DFAD 20FFFFFF FILD QWORD PTR SS:[EBP-E0]               ;  (double)int`

`0050D1F9   .  DB2D 1CE55000 FLD TBYTE PTR DS:[50E51C]                ;  3.14`

`0050D1FF   .  DEC9          FMULP ST(1),ST`

`0050D201   .  D9FA          FSQRT`

`0050D203   .  E8 D058EFFF   CALL SuperSok.00402AD8`

`0050D208   .  E8 8B98EFFF   CALL SuperSok.00406A98`

`0050D20D   .  52            PUSH EDX                                 ; /Arg2`

`0050D20E   .  50            PUSH EAX                                 ; |Arg1`

`0050D20F   .  8D85 28FFFFFF LEA EAX,DWORD PTR SS:[EBP-D8]            ; |`

`0050D215   .  E8 26BEEFFF   CALL SuperSok.00409040                   ; \SuperSok.00409040`

`0050D21A   .  FFB5 28FFFFFF PUSH DWORD PTR SS:[EBP-D8]`

`0050D220   .  68 30E55000   PUSH SuperSok.0050E530                   ; ASCII ".fsk"`

`0050D225   .  8D45 F0       LEA EAX,DWORD PTR SS:[EBP-10]`

`0050D228   .  BA 04000000   MOV EDX,4`

`0050D22D   .  E8 0A6EEFFF   CALL SuperSok.0040403C`

`0050D232   .  8B45 F0       MOV EAX,DWORD PTR SS:[EBP-10]`

`0050D235   .  E8 C6C0EFFF   CALL SuperSok.00409300`
*/

## 5. 分析总结:

Crack的难点:

a. 注册没有错误提示, 文件名动态计算

b. `CALL FindFirstFileA` 判断文件是否存在, 而不是用 `CreateFileA`

c. keyfile SIZE 大， >10K

d. Read Keyfile 是 有多少就读多少, 在分析清楚前, 不好确定 SIZE

e. 不是一次判断, 很多地方, 很难暴破

作者加密/算法的不足

a. 加密壳等于没有

b. 没有反跟踪, 没有防 DEDE

c. 加密算法强度不高,

d. 判断点很多,分析一处就可解决相同的其它多处: OFFSET 与 index 成正比

## 6. 注册机

```c
#include <stdlib.h>

#include <string.h>

#include <stdio.h>

#include <math.h>

#include <time.h>

// @00526B28
unsigned long value1[51] = {
    0x0000CE6D,    0x000084C3,    0x00007176,    0x00004B82,
    0x0000E387,    0x00018605,    0x0000F66F,    0x00005C04,
    0x00005C45,    0x0000B0BC,    0x0000AB60,    0x0000AB83,
    0x0000CFFA,    0x0000A8CD,    0x000108F8,    0x00015DCB,
    0x0000D2B1,    0x00011EC5,    0x0000CC79,    0x0000B2B1,
    0x0000BEE9,    0x0000337D,    0x0000AA70,    0x00012253,
    0x0000E058,    0x00014A72,    0x00005AF2,    0x0000DFEA,
    0x0000B2A4,    0x00015D4E,    0x00017FBA,    0x0000809E,
    0x0000D4FA,    0x0000B5D0,    0x0000B6C9,    0x00015D71,
    0x00016B9B,    0x0001802A,    0x000121F0,    0x00013648,
    0x000181C2,    0x000102C8,    0x0000B236,    0x0000DC16,
    0x000152D7,    0x0000328F,    0x00011E21,    0x000123E2,
        0x0000536B,    0x0000B257,    0x0000B045
};

// @00526BF4
unsigned long value2[51] = {
```
