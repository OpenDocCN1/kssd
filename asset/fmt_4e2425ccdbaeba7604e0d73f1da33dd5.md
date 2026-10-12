
### 汇编分析

```
:0040B7ED 3BD6                 cmp edx, esi ;比较个数是否小于 0

:0040B7EF 7E17                 jle 0040B808

:0040B7F1 53                   push ebx

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:0040B805(C)

|

:0040B7F2 0FBE0439             / movsx eax, byte ptr [ecx+edi] ;字符依次送给 eax
:0040B7F6 8D5801               | lea ebx, dword ptr [eax+01] ;加 1 给 ebx
:0040B7F9 0FAFD8               | imul ebx, eax ;相乘送给 ebx
:0040B7FC 43                   | inc ebx ;加 1
:0040B7FD 0FAFD8               | imul ebx, eax ;在送给 ebx
:0040B800 03F3                 | add esi, ebx ;放入 esi 保存
:0040B802 41                   | inc ecx
:0040B803 3BCA                 | cmp ecx, edx ;是否是最后一个
:0040B805 7CEB                 jl 0040B7F2
:0040B807 5B                   pop ebx

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:0040B7EF(C)

|

:0040B808 8D4C2418             lea ecx, dword ptr [esp+18]
:0040B80C C7442410FFFFFFFF      mov [esp+10], FFFFFFFF
:0040B814 E8748A0600           call 0047428D
:0040B819 8B4C2408             mov ecx, dword ptr [esp+08]
:0040B81D 8BC6                 mov eax, esi
:0040B81F 33D2                 xor edx, edx
:0040B821 5F                   pop edi
:0040B822 F7742418             div [esp+18]
:0040B826 64890D00000000       mov dword ptr fs:[00000000], ecx
:0040B82D 5E                   pop esi ;? esi 为假注册码
:0040B82E 83C40C               add esp, 0000000C
:0040B831 C3                   ret
```

### 流程总结

以上就是我跟踪的流程....

好了在把整个过程在分析一遍,我们输入姓名和注册码,先把我们输入的名字和 `"AnimateIt"`, `"STANDARD"`, 连接起来形成 `"AnimateItSTANDARDdengkeng"`,然后在把小写转换成大写 `"ANIMATEITSTANDARDDENGKENG"`,最后把整个字符串做运算!上面有分析,具体的算法看上面!最后的运算结果存放在 `esi` 中,最后附值给 `eax`.使 `eax` 和存放假注册码的 `esi` 相比较如果成功就注册为 Standard Edition,如果不是就继续比较看是否是 Power Edition,因为 Standard 和 Power 的算法都一样只是字符变成了 `"ANIMATEITPOWERDENGKENG"`.好了下面给出注册机.VC 下编译成功!

### 注册机源代码

```c
#include "stdio.h"
#include "iostream.h"
#include "windows.h"
#include "conio.h"
#include "string.h"

int main()
{
  char Name[256] = "";
  char StandardCopy[256] = "ANIMATEITSTANDARD";
  char PowerCopy[256] = "ANIMATEITPOWER";
  long NameLen = 0;
  long NameLenPower = 0;
  long Serial = 0;

  cout << "##############################################" << endl;
  cout << "AnimateIt Screen Saver KeyGen Made By dengkeng" << endl;
  cout << "QQ:28895751   E-Mail:shellc0de@sohu.com" << endl;
  cout << "##############################################" << endl;
  cout << endl;
  cout << "Registered Name:";

  cin.getline(Name, 256);
  strcat(StandardCopy, Name);
  strcat(PowerCopy, Name);

  NameLen = strlen(StandardCopy);
  NameLenPower = strlen(PowerCopy);

  strupr(StandardCopy);
  strupr(PowerCopy);

  if (NameLen < 1)
  {
    cout << "Your name is too short!" << endl;
  }
  else
  {
    __asm
    {
      PUSH    EBP
      XOR     ESI, ESI
      XOR     EBX, EBX
              LEA     EDI, StandardCopy
      MOV     EDX, [NameLen]
      MOV     ECX, 00000000h
      Loc_0040B7F2:
              MOVSX   EAX, BYTE PTR [EDI+ECX]
              LEA     EBX, [EAX+1]
              IMUL    EBX, EAX
              INC     EBX
              IMUL    EBX, EAX
              ADD     ESI, EBX
              INC     ECX
      CMP     ECX, EDX
      JNZ     Loc_0040B7F2
      MOV     [Serial], ESI
      POP     EBP
    }

    cout << "Registration Key For Standard Edition:" << Serial << endl;

    __asm
    {
      PUSH    EBP
      XOR     ESI, ESI
      XOR     EBX, EBX
              LEA     EDI, PowerCopy
      MOV     EDX, [NameLenPower]
      MOV     ECX, 00000000h
      Loc_0040B7F3:
              MOVSX   EAX, BYTE PTR [EDI+ECX]
              LEA     EBX, [EAX+1]
              IMUL    EBX, EAX
              INC     EBX
              IMUL    EBX, EAX
              ADD     ESI, EBX
              INC     ECX
      CMP     ECX, EDX
      JNZ     Loc_0040B7F3
      MOV     [Serial], ESI
      POP     EBP
    }

    cout << "Registration Key For Power Edition:" << Serial << endl;
  }

  getch();
  return 0;
}
```

### 运行结果

结果如下:

```
Registered Name:dengkeng
Registration Key For Standard Edition:10337368
Registration Key For Power Edition:9499310
```

Made By dengkeng

E-mail:shellc0de@sohu.com

欢迎转载,请保持文章的完整性

`4dd466d0e1e0cc123afbe504da2462ca`
