
# login

> 原文：[`www.pediy.com/kssd/pediy03/forum856.htm`](https://www.pediy.com/kssd/pediy03/forum856.htm)

输入 14 个数字后,下 bpx hmemcpy,f12 12 次,来到这里

:004494C0 8B45E4  mov eax, dword ptr \[ebp\-1C\]  <--bpm eax r

:004494C3 8D55E8  lea edx, dword ptr \[ebp\-18\]

:004494C6 E81DE2FBFF    call 004076E8

按 f10 继续,回在这里停下

:00407705 807C1FFF20    cmp byte ptr \[edi+ebx\-01\], 20 <--是否空格

:0040770A 76F4  jbe 00407700

继续走在这个地方移动数据到内存的另一个地方

:004027BF F3  repz

:004027C0 A5  movsd

:004027C1 89C1  mov ecx, eax

:004027C3 83E103  and ecx, 00000003

:004027C6 F3  repz

:004027C7 A4  movsb

:004027C8 5F  pop edi

:004027C9 5E  pop esi

:004027CA C3  ret

:00449528 FF75D8  push \[ebp\-28\]

:0044952B 8B45FC  mov eax, dword ptr \[ebp\-04\]

:0044952E 0554030000    add eax, 00000354

:00449533 BA04000000    mov edx, 00000004

:00449538 E8DFA5FBFF    call 00403B1C  <---f8 跟进

:00445CA9 885C38FF      mov byte ptr \[eax+edi\-01\], bl

:00445CAD 8B45F0      mov eax, dword ptr \[ebp\-10\]

:00445CB0 8A4438FF      mov al, byte ptr \[eax+edi\-01\]

:00445CB4 3C30      cmp al, 30      //是否在 0~9 之间

:00445CB6 7207      jb 00445CBF

:00445CB8 8B55F0      mov edx, dword ptr \[ebp\-10\]

:00445CBB 3C39      cmp al, 39

:00445CBD 760D      jbe 00445CCC

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:00445CB6(C)

|

:00445CBF 8B45F8      mov eax, dword ptr \[ebp\-08\]

:00445CC2 E819DBFBFF      call 004037E0

:00445CC7 E9ED030000      jmp 004460B9

\----------------------

:00445D33 80FB30      cmp bl, 30

:00445D36 0F85B8010000    jne 00445EF4

:00445D3C 8D45F0      lea eax, dword ptr \[ebp\-10\]

:00445D3F 8B55F4      mov edx, dword ptr \[ebp\-0C\]

:00445D42 E831DBFBFF      call 00403878

:00445D47 8D45F0      lea eax, dword ptr \[ebp\-10\]

:00445D4A E8DDDEFBFF      call 00403C2C

:00445D4F 8B55F4      mov edx, dword ptr \[ebp\-0C\]

:00445D52 8A12      mov dl, byte ptr \[edx\]    //\[edx\]存放 Regcode\[1..12\]

:00445D54 885007      mov byte ptr \[eax+07\], dl    //eax\[7\]=regcode\[1\]

:00445D57 8D45F0      lea eax, dword ptr \[ebp\-10\]

:00445D5A E8CDDEFBFF      call 00403C2C

:00445D5F 8B55F4      mov edx, dword ptr \[ebp\-0C\]

:00445D62 8A5201      mov dl, byte ptr \[edx+01\]

:00445D65 88500B      mov byte ptr \[eax+0B\], dl    //eax\[11\]=regcode\[2\]

:00445D68 8D45F0      lea eax, dword ptr \[ebp\-10\]

:00445D6B E8BCDEFBFF      call 00403C2C

:00445D70 8B55F4      mov edx, dword ptr \[ebp\-0C\]

:00445D73 8A5202      mov dl, byte ptr \[edx+02\]

:00445D76 8810      mov byte ptr \[eax\], dl    //eax\[1\]=regcode\[3\]

:00445D78 8D45F0      lea eax, dword ptr \[ebp\-10\]

:00445D7B E8ACDEFBFF      call 00403C2C

:00445D80 8B55F4      mov edx, dword ptr \[ebp\-0C\]

:00445D83 8A5203      mov dl, byte ptr \[edx+03\]

:00445D86 88500A      mov byte ptr \[eax+0A\], dl    //eax\[10\]=regcode\[4\]

:00445D89 8D45F0      lea eax, dword ptr \[ebp\-10\]

:00445D8C E89BDEFBFF      call 00403C2C

:00445D91 8B55F4      mov edx, dword ptr \[ebp\-0C\]

:00445D94 8A5204      mov dl, byte ptr \[edx+04\]

:00445D97 885009      mov byte ptr \[eax+09\], dl    //eax\[9\]=regcode\[5\]

:00445D9A 8D45F0      lea eax, dword ptr \[ebp\-10\]

:00445D9D E88ADEFBFF      call 00403C2C

:00445DA2 8B55F4      mov edx, dword ptr \[ebp\-0C\]

:00445DA5 8A5205      mov dl, byte ptr \[edx+05\]

:00445DA8 885003      mov byte ptr \[eax+03\], dl    //eax\[4\]=regcode\[6\]

:00445DAB 8D45F0      lea eax, dword ptr \[ebp\-10\]

:00445DAE E879DEFBFF      call 00403C2C

:00445DB3 8B55F4      mov edx, dword ptr \[ebp\-0C\]

:00445DB6 8A5206      mov dl, byte ptr \[edx+06\]

:00445DB9 885005      mov byte ptr \[eax+05\], dl    //eax\[5\]=regcode\[7\]

:00445DBC 8D45F0      lea eax, dword ptr \[ebp\-10\]

:00445DBF E868DEFBFF      call 00403C2C

:00445DC4 8B55F4      mov edx, dword ptr \[ebp\-0C\]

:00445DC7 8A5207      mov dl, byte ptr \[edx+07\]

:00445DCA 885002      mov byte ptr \[eax+02\], dl    //eax\[2\]=regcode\[8\]

:00445DCD 8D45F0      lea eax, dword ptr \[ebp\-10\]

:00445DD0 E857DEFBFF      call 00403C2C

:00445DD5 8B55F4      mov edx, dword ptr \[ebp\-0C\]

:00445DD8 8A5208      mov dl, byte ptr \[edx+08\]

:00445DDB 885004      mov byte ptr \[eax+04\], dl    //eax\[4\]=regcode\[9\]

:00445DDE 8D45F0      lea eax, dword ptr \[ebp\-10\]

:00445DE1 E846DEFBFF      call 00403C2C

:00445DE6 8B55F4      mov edx, dword ptr \[ebp\-0C\]

:00445DE9 8A5209      mov dl, byte ptr \[edx+09\]

:00445DEC 885001      mov byte ptr \[eax+01\], dl    //eax\[1\]=regcode\[10\]

:00445DEF 8D45F0      lea eax, dword ptr \[ebp\-10\]

:00445DF2 E835DEFBFF      call 00403C2C

:00445DF7 8B55F4      mov edx, dword ptr \[ebp\-0C\]

:00445DFA 8A520A      mov dl, byte ptr \[edx+0A\]

:00445DFD 885006      mov byte ptr \[eax+06\], dl    //eax\[6\]=regcode\[11\]

:00445E00 8D45F0      lea eax, dword ptr \[ebp\-10\]

:00445E03 E824DEFBFF      call 00403C2C

:00445E08 8B55F4      mov edx, dword ptr \[ebp\-0C\]

:00445E0B 8A520B      mov dl, byte ptr \[edx+0B\]

:00445E0E 885008      mov byte ptr \[eax+08\], dl    //eax\[8\]=regcode\[12\]

:00445E11 8D45F4      lea eax, dword ptr \[ebp\-0C\]

:00445E14 8B55F0      mov edx, dword ptr \[ebp\-10\]

:00445E17 E85CDAFBFF      call 00403878

:00445E1C 8B75E8      mov esi, dword ptr \[ebp\-18\]

:00445E1F 85F6      test esi, esi

:00445E21 7E7D      jle 00445EA0

:00445E23 BF01000000      mov edi, 00000001

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:00445E9E(C)

|

:00445E28 8B45F4      mov eax, dword ptr \[ebp\-0C\]

:00445E2B 0FB64438FF      movzx eax, byte ptr \[eax+edi\-01\]

:00445E30 BB39000000      mov ebx, 00000039

:00445E35 2BD8      sub ebx, eax

:00445E37 83C330      add ebx, 00000030

:00445E3A 80FB37      cmp bl, 37

:00445E3D 7722      ja 00445E61

:00445E3F 33C0      xor eax, eax      //计算第 2 步

:00445E41 8AC3      mov al, bl      //详细请看

:00445E43 8BD7      mov edx, edi      //下面的

:00445E45 D1FA      sar edx, 1      //注释

:00445E47 7903      jns 00445E4C

:00445E49 83D200      adc edx, 00000000

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:00445E47(C)

|



### 反汇编代码分析

```asm
:00445E4C 42                  inc edx                 //

:00445E4D 33C2                xor eax, edx            //

:00445E4F 8945EC              mov dword ptr [ebp-14], eax

:00445E52 8D45F4              lea eax, dword ptr [ebp-0C]

:00445E55 E8D2DDFBFF          call 00403C2C

:00445E5A 8B55EC              mov edx, dword ptr [ebp-14]

:00445E5D 885438FF            mov byte ptr [eax+edi-01], dl

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:00445E3D(C)

|

:00445E61 80FB38              cmp bl, 38

:00445E64 750D                jne 00445E73

:00445E66 8D45F4              lea eax, dword ptr [ebp-0C]

:00445E69 E8BEDDFBFF          call 00403C2C

:00445E6E C64438FF39          mov [eax+edi-01], 39

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:00445E64(C)

|

:00445E73 80FB39              cmp bl, 39

:00445E76 750D                jne 00445E85

:00445E78 8D45F4              lea eax, dword ptr [ebp-0C]

:00445E7B E8ACDDFBFF          call 00403C2C

:00445E80 C64438FF38          mov [eax+edi-01], 38

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:00445E76(C)

|

:00445E85 8B45F4              mov eax, dword ptr [ebp-0C]

:00445E88 807C38FF27          cmp byte ptr [eax+edi-01], 27

:00445E8D 750D                jne 00445E9C

:00445E8F 8D45F4              lea eax, dword ptr [ebp-0C]

:00445E92 E895DDFBFF          call 00403C2C

:00445E97 C64438FF24          mov [eax+edi-01], 24

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:00445E8D(C)

|

:00445E9C 47                  inc edi

:00445E9D 4E                  dec esi

:00445E9E 7588                jne 00445E28

:00449595 8B45FC              mov eax, dword ptr [ebp-04]

:00449598 8B804C030000        mov eax, dword ptr [eax+0000034C]

:0044959E 8B55FC              mov edx, dword ptr [ebp-04]

:004495A1 8B9250030000        mov edx, dword ptr [edx+00000350]

:004495A7 E8C0A5FBFF          call 00403B6C

:004495AC 0F85B3040000        jne 00449A65            //if jump badguy

:004495FD E86AA5FBFF          call 00403B6C

:00449602 0F855C010000        jne 00449764            //if jump invalid register

* Possible StringData Ref from Code Obj ->"正式注册成功"

                              |

:0044973B B8589C4400          mov eax, 00449C58

:00449740 E8EBAEFFFF          call 00444630

:00449745 33C0                xor eax, eax

:00449747 5A                  pop edx

:00449748 59                  pop ecx

:00449749 59                  pop ecx

* Possible StringData Ref from Code Obj ->"非法注册成功"

                              |

:004499DE B8949C4400          mov eax, 00449C94

:004499E3 E848ACFFFF          call 00444630

:004499E8 33C0                xor eax, eax

:004499EA 5A                  pop edx

:004499EB 59                  pop ecx

:004499EC 59                  pop ecx

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:004495AC(C)

|

* Possible StringData Ref from Code Obj ->"注册失败,请输入正确的注册号"

                              |

:00449A65 B8C49C4400          mov eax, 00449CC4

:00449A6A E8C1ABFFFF          call 00444630
```

### 注册算法逻辑

上面的代码无非就是对我们输入的 `Regcode` 进行一系列的变换。

由 `GetText()` 取得，`LstrCat()` 连结，`Trim()` 去掉首位 `0` 后，剩下来的 `13` 位数中，取 `12` 位用来变换，过程如下。

比如我输入的 `Regcode=02-6448-2915-8742`

```
         || |||| |||| |||
1、     || |||| |||| |||
    不要  <----|| |||| |||| |||
    存到第 8 位上<----| |||| |||| |||
        12  <-------|||| |||| |||
     1    <--------||| |||| |||
     11  <---------|| |||| |||
     10  <----------| |||| |||
     4   <------------|||| |||
     6   <-------------||| |||
     3   <--------------|| |||
     5   <---------------| |||
     2   <-----------------|||
         7   <------------------||
         9   <-------------------|

结果变成 4812-5972-4846
```

`2、` 把上面得到的 `12` 位数按如下规律再一次转换。先取第一个数，这里是 `4`，和 `0`、`1` 比较，是 `1` 变成 `9`，是 `0` 变成 `8`，都不是的话变成这样：`0x39-0x34^(i/2)+1` (`i` 是 `4` 在上面字符串中的位置)。结果变成 `4394-7462-0734`

`3、` 倒序。将上面得到的字符倒序排列变成 `4370-2647-4934`

`4、` 在首位加一个 `0`。变成 `0-4370-2647-4934`

`5、` 和你的机器号比较，不一样就没戏了，一样的话继续下一步。

`6、` 取你的 `Regcod` 的 `2` 个数和最后一个数比较，如果相等则正式注册成功。否则会出现非法注册成功。

### 注册机源码

`**** start here ****`

```c
#include <stdio.h>

main()
{
    char a[13], regcode[14];
    int i=1;
    char t, temp;

    clrscr();

    printf("Keymaker by CoolBob/China cracker group\n\n");
    printf("Machine code: ");
    scanf("%s", a);

    for(i=12; i>6; i--)
    {
        t=a[i];
        a[i]=a[12-i+1];
        a[12-i+1]=t;
    };

    for(i=1; i<13; i++){
        if(a[i]==0x39) a[i]=0x31;
        else if(a[i]==0x38) a[i]=0x30;
        else
        {
            temp=(a[i])^((i/2)+1);
            a[i]=0x39+0x30-temp;
        }
    };

    regcode[0]='0';
    regcode[1]=a[8];
    regcode[2]=a[12];
    regcode[3]=a[1];
    regcode[4]=a[11];
    regcode[5]=a[10];
    regcode[6]=a[4];
    regcode[7]=a[6];
    regcode[8]=a[3];
    regcode[9]=a[5];
    regcode[10]=a[2];
    regcode[11]=a[7];
    regcode[12]=a[9];
    regcode[13]=regcode[1];
    regcode[14]=0;

    printf("Regcode is:  %s\n\n\n", regcode);
    printf("Press any key to exit!\n");
    getch();
}
```

`**** End here ****`

`written by CoolBob[CCG]/2001.4.20`
`4dd466d0e1e0cc123afbe504da2462ca`
