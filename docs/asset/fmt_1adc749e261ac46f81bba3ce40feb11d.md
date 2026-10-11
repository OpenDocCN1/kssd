
# 序列号方式

> 原文：[`www.pediy.com/kssd/tutorial/chap6-1-11.htm`](https://www.pediy.com/kssd/tutorial/chap6-1-11.htm)

## 目录

目录＞＞第六章

### 习题列表

- **1、习题一** `chap6-1-1-01` 2K Serial 中
- **2、习题二** `chap6-1-1-02` 2K code 稍难
- **3、习题三** `chap6-1-1-03` 16K Serial (需要 `cw3220.dll`) 中
- **4、习题四** `chap6-1-1-04` 5K Name/Serial 中
- **5、习题五** `chap6-1-1-05` 103K Name/Serial 中
- **6、习题六** `chap6-1-1-06` 91K code 中
- **7、习题七** `chap6-1-1-07` 26K Name/Group/Serial
- **8、习题八** `chap6-1-1-08` 5K Name/Serial 中
- **9、习题九** `chap6-1-1-09` 119K Name/Serial 中
- **10、习题十** `chap6-1-1-10` 34K Name/Group/Serial 最难

### 1、习题一答案

```text
破解 chap6-1-1-01  

* Referenced by a CALL at Address:  

|:004010B0  
  

|  

:004010C9 56                       push esi  
:004010CA 57                       push edi  
:004010CB 51                       push ecx  
:004010CC 33F6                     xor esi, esi  
:004010CE 33FF                     xor edi, edi  
:004010D0 B908000000               mov ecx, 00000008          ---定义循环次数  
:004010D5 BE44304000               mov esi, 00403044          ---调出输入的密码  
:004010DA 803632                   xor byte ptr [esi], 32      ---把密码个字节与 32 取异或  
:004010DD 46                       inc esi  
:004010DE E2FA                     loop 004010DA               ---向上循环  
:004010E0 BE44304000               mov esi, 00403044          
:004010E5 B904000000               mov ecx, 00000004          ---8 变 4  
:004010EA 8A06                     mov al, byte ptr [esi]  
:004010EC 8A5E01                   mov bl, byte ptr [esi+01]  
:004010EF 32C3                     xor al, bl                 ---把变化后的密码前后取异或  
:004010F1 88874C304000             mov byte ptr [edi+0040304C], al  
:004010F7 83C602                   add esi, 00000002          
:004010FA 47                       inc edi  
:004010FB E2ED                     loop 004010EA               ---向上循环  
:004010FD BE4C304000               mov esi, 0040304C            ---将生成的 4 个数变成 2 个  
:00401102 8A06                     mov al, byte ptr [esi]  
:00401104 8A5E01                   mov bl, byte ptr [esi+01]  
:00401107 32C3                     xor al, bl  
:00401109 8A5E02                   mov bl, byte ptr [esi+02]  
:0040110C 8A4E03                   mov cl, byte ptr [esi+03]  
:0040110F 32D9                     xor bl, cl  
:00401111 32C3                     xor al, bl                   ---再由 2 个变 1 个放入 AL  
:00401113 B908000000               mov ecx, 00000008  
:00401118 BE44304000               mov esi, 00403044  
:0040111D 3006                     xor byte ptr [esi], al      ---将生成的 1 个与原来的取异或  
:0040111F 46                       inc esi  
:00401120 E2FB                     loop 0040111D  
:00401122 B908000000               mov ecx, 00000008            ---从这往下开始比较  
:00401127 BE44304000               mov esi, 00403044            ---放入算出的结果  

* Possible StringData Ref from Data Obj ->"q"  
                           |  
:0040112C BF08304000               mov edi, 00403008            ---放入正确的结果  
:00401131 8A06                     mov al, byte ptr [esi]  
:00401133 3A07                     cmp al, byte ptr [edi]  
:00401135 751D                     jne 00401154                 ---跳向出错  
:00401137 46                       inc esi  
:00401138 47                       inc edi  
:00401139 E2F6                     loop 00401131                 ---向上循环  
:0040113B 6A40                     push 00000040  
```

模拟运算:

如果输入 `12345678`

| 步骤 | 内容 |
| :--- | :--- |
| 机器码 | `31 32 33 34 35 36 37 38` |
| 与 `32` 异或 | `03 00 01 06 07 04 05 0A` ----(1) |
| 8 变 4 为 | `03 07 03 0F` |
| 4 变 2 为 | `04 0C` |
| 2 变 1 为 | `08` ----(2) |
| (1)与 `08` 取异或 | `0B 08 09 0E 0F 0C 0D 02` |
| `00403008` 内正确的为 | `71 18 59 1B 79 42 45 4C` |

根据正确反推注册码: (关键是如何计算(2))

由算法可知(2)是由机器码反复取异或得到，其实由它的正确的密码重复这一算法也可求的(2)，实验得出。缺少证明。

| 步骤 | 内容 |
| :--- | :--- |
| 机器码 | `71 18 59 1B 79 42 45 4C` |
| 8 变 4 为 | `69 42 3B 09` |
| 4 变 2 为 | `2B 32` |
| 2 变 1 为 | `19` ----正确的密码的(2)值应为 `19` |

接着反推正确的注册码:

| 步骤 | 内容 |
| :--- | :--- |
| 机器码 | `71 18 59 1B 79 42 45 4C` |
| 与 `19` 取异或 | `68 01 40 02 60 5B 5C 55` |
| 与 `32` 取异或 | `5A 33 72 30 52 69 6E 67` |

查表得正确的注册码为: `Z 3 r 0 R i n g` (`Z3r0Ring`)

`ZXEM 2000.3.20`

### 2、习题二答案

```text
破解 chap6-1-1-02  

* Referenced by a CALL at Address:  

|:004011A9  
  

|  

:0040120B C8000000                 enter 0000, 00  
:0040120F 53                       push ebx  
:00401210 52                       push edx  
:00401211 33C0                     xor eax, eax  
:00401213 B8A6204000               mov eax, 004020A6         ----把输入的密码放入 EAX  
:00401218 803800                   cmp byte ptr [eax], 00  
:0040121B 7460                     je 0040127D  
:0040121D 33DB                     xor ebx, ebx  
:0040121F 33D2                     xor edx, edx  
:00401221 8A18                     mov bl, byte ptr [eax]    ----开始处理密码 过程见下（1）  
:00401223 C1C308                   rol ebx, 08  
:00401226 03D3                     add edx, ebx              ----处理后的都放入 EDX  
:00401228 40                       inc eax  
:00401229 803800                   cmp byte ptr [eax], 00  
:0040122C 75F3                     jne 00401221  
:0040122E 52                       push edx  
:0040122F 6854204000               push 00402054  
:00401234 68BF204000               push 004020BF  
:00401239 E88F000000               Call 004012CD  
:0040123E BBBF204000               mov ebx, 004020BF         ----与正确的开始比较  
:00401243 803B38                   cmp byte ptr [ebx], 38  
:00401246 7535                     jne 0040127D  
:00401248 807B0144                 cmp byte ptr [ebx+01], 44  
:0040124C 752F                     jne 0040127D  
:0040124E 807B0243                 cmp byte ptr [ebx+02], 43  
:00401252 7529                     jne 0040127D  
:00401254 807B0341                 cmp byte ptr [ebx+03], 41  
:00401258 7523                     jne 0040127D  
:0040125A 807B0446                 cmp byte ptr [ebx+04], 46  
:0040125E 751D                     jne 0040127D  
:00401260 807B0533                 cmp byte ptr [ebx+05], 33  
:00401264 7517                     jne 0040127D  
:00401266 807B0636                 cmp byte ptr [ebx+06], 36  
:0040126A 7511                     jne 0040127D  
:0040126C 807B0738                 cmp byte ptr [ebx+07], 38  
:00401270 750B                     jne 0040127D  
:00401272 B801000000               mov eax, 00000001  
:00401277 5A                       pop edx  
:00401278 5B                       pop ebx  
:00401279 C9                       leave  
:0040127A C20400                   ret 0004  
```

（1）过程分析：

如果输入 `123456`

过程：

```text
00 00 31 00  
00 31 32 00  
31 32 33 00  
32 33 34 31  
33 34 35 32  
+） 34 35 36 33  
结果： CB 00 35 96  
```

而正确的应为：`8D CA F3 68`

我们通过上面的模拟分析可以大致的了解运算过程，由于相加结果是如上的有很多，我们就可以大胆的假设，来推出一组数。

我就推出一组 6 位的：`""$%=)`

`ZXEM 2000.3.23`

### 3、习题三答案

这题我们用函数 `getdlgitemtexta` 设断，它作用是得指定输入框输入字符串。也是一个常用的函数。

在注册框中输入：`12345678`

然后设断：`bpx getdlgitemtexta`

点击 `OK`, 你将中断在 `SOFTICE`，具体如下：

```text
* Reference To: USER32.GetDlgItemTextA, Ord:0000h  
                           |  
:0040115D E8E4030000               Call 00401546  
:00401162 8D4DF4                   lea ecx, dword ptr [ebp-0C]  
:00401165 51                       push ecx  
:00401166 E811FFFFFF               call 0040107C<----此 CALL 计算密码  
:0040116B 59                       pop ecx  
```
