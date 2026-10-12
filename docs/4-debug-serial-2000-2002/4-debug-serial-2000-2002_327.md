
# flashget0.95

> 原文：[`www.pediy.com/kssd/pediy03/forum637.htm`](https://www.pediy.com/kssd/pediy03/forum637.htm)

广告广告，无处可逃（`flashget0.95` 破解记录)

- 软件名称：网际快车 `FlashGet`(`JetCar`)
- 日期：2001.2.16
- 最新版本：`0.95`
- 文件大小：`1008KB`
- 软件授权：免费软件
- 使用平台：`Win9x`/`Me`/`NT`/`2000`
- 下载 `http://software.wx88.net/down/fg095.zip`

广告太烦人，国货当自强。最优秀的国产软件之一，别说你没听过。

软件在输入完注册码后并不立即进行比对，先把你输入的用户名和注册码存放在注册表中，在下一次软件启动的时候才作比对。

### 注册码存放注册表位置

`\\HKEY\_CURRENT\_USER\\SoftWare\\JetCar\\JetCar\\General` 下

- `RegName` = 用户名
- `RegPass` = 注册码

(`trw` 下可用 `bpx RegQueryValueExA` 设断，追注册码)

### 软件在启动时的比对为关键点

`w32dasm` 反汇编,分析（略，可参考阿凡提文章）

```asm
0167:00415084 8B44240C       MOV       EAX,[ESP+0C] 注册表读出码
0167:00415088 8B4C2414       MOV       ECX,[ESP+14] 根据注册表用户名算出码
0167:0041508C 50             PUSH      EAX
0167:0041508D 51             PUSH      ECX
0167:0041508E E8CBAB0500    CALL      0046FC5E 关键比较
0167:00415093 83C408        ADD       ESP,BYTE +08
0167:00415096 885C2428      MOV       [ESP+28],BL
0167:0041509A 85C0         TEST      EAX,EAX
0167:0041509C 5F           POP       EDI
0167:0041509D 8D4C2410     LEA       ECX,[ESP+10]
0167:004150A1 7537         JNZ       004150DA  出错跳(爆破 75==》74）
0167:004150A3 E88BE00600   CALL      00483133
0167:004150A8 8D4C2408     LEA       ECX,[ESP+08]
0167:004150AC C644242400   MOV       BYTE [ESP+24],00
```

### 快速破解：

1. 先运行 `flashget0.95`，然后在菜单上选“帮助->移掉广告横幅”，
   用户名称：`guodong`，注册码：`12345678`，按“确认”键，然后退出。
2. 用 `trw2000` 载入 `jetcar.exe`
3. `bpx 0041508C`
4. `go`
5. `d ecx     30617KB2M6`
6. `d eax     12345678`
7. 整理
   用户名 `guodong`
   注册码 `30617KB2M6`

哪位大哥作个注册机吧

作者 风飘雪

主页 `http://duba.126.com`

```c
#include "stdio.h"

main()

{

static char string[81]={'x','x','x','x','x','x','x','x'};

char sp[8]={'h','y','t','z','l','h','y','t'};

unsigned long i,a=0,b=0;

char c;

loop:printf("**********FLASHGET V0.00~~0.95**\nThis keygen is maked by Pretender\nPlease input your name\n");

gets(string);

printf("Your name is             ");

puts(string);

b=strlen(string);

if(b<6)

{printf("This keygen is BETA 0.001,Your user name must more than 5 chars!!\n");goto loop;}

printf("Your Register code is:  ");

for(i=0;i<8;i++)

{if(string[i]<0) string[i]+=128;

if(i<b||b<6)

string[i]=sp[i]^string[i];

else

string[i]=sp[i]^string[i-b+1];}

a=string[0]*123;

a=a+string[1]*4121212;

a=a+string[2]*25252335;

a=a+string[3]*541321;

a=a+string[4]*988908;

a=a+string[5]*1225252335;

a=a+string[6]*12412412;

a=a+string[7]*879878912;

for(i=0;i<10;i++)

{string[i]=a%10;a=a/10;}

printf("%c",string[9]+48);

printf("%c",string[8]+48);

printf("%c",string[7]+48);

printf("%c",string[6]+48);

printf("%c",string[5]+48);

printf("%c",string[7]+21+48);

printf("%c",string[6]+17+48);

printf("%c",string[2]+48);

printf("%c",string[2]+27+48);

printf("%c",string[0]+48);

printf("\nWelcome to WWW.CRACKNOW.COM\n");

}

有点缺陷~~~用户名必须大于 5 个字符~~~~补考之后想办法解决~~呵呵~~
```

```vb
Dim x, y As Double

Dim a(7)

Private Sub Text1_Change()

 yhm = Trim(Text1.Text)

 l = Len(yhm)

 l1 = l

 If l > 0 Then

   j = 1

   For i = 0 To 7

     If j <= l1 Then

       b = Asc(Mid(yhm, j, 1))

       j = j + 1

       If b < 0 Then

         b = b + 65536

         a(i) = b \\ 256

         i = i + 1

         If i <= 7 Then a(i) = b Mod 256: l = l + 1

       Else

         a(i) = b

       End If

     Else

       a(i) = &H78

     End If

   Next

   s = "hytzlhyt"

   For i = 0 To 7

     If i < l Or i < 5 Then

       a(i) = (a(i) Xor Asc(Mid(s, i + 1, 1))) And 127

     Else

       a(i) = (a(i - 5) Xor Asc(Mid(s, i + 1, 1))) And 127

     End If

   Next

   x = a(0) * 123

   x = x + a(1) * &H3EE27C

   x = x + a(2) * &H18151EF

   x = x + a(3) * &H84289

   x = x + a(4) * &HF16EC

   x = x + a(5) * &H4907DDEF

   x = x + a(6) * &HBD65FC

   x = x + a(7) * &H3471E300

   Do While x >= 4294967296#

     x = x - 4294967296#

   Loop

   zcm = Trim(Str(x))

   zcm = "00000000" + zcm

   l = Len(zcm)

   Mid(zcm, l - 1, 1) = Chr(Asc(Mid(zcm, l - 2, 1)) + &H1B)

   Mid(zcm, l - 3, 1) = Chr(Asc(Mid(zcm, l - 6, 1)) + &H11)

   Mid(zcm, l - 4, 1) = Chr(Asc(Mid(zcm, l - 7, 1)) + &H15)

   Text2.Text = Right(zcm, 10)

 Else

   Text2.Text = ""

 End If

End Sub
```

写一个这样的 `CRACKCODE.INI`,采用 `Crackcode2000` 的增强模式(模式 2),以下是我根据上面写的内容:

```ini
[Options]

CommandLine=jetcar.exe               程序的执行文件名

Mode=1                                取用模式 1 读取注册码

First_Break_Address=41508C            程式的中断地址

First_Break_Address_Code=50           中断程式处的 1 Byte 代码

First_Break_Address_Code_Lenth=1     中断行代码的长度（单位：byte）

Save_Code_Address=ECX                 存放注册码的内存地址的寄存器名
```

`4dd466d0e1e0cc123afbe504da2462ca`
