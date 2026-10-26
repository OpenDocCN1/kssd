
``` 
   00401000    60              pushad

   00401001    E8 00000000     call   00401006

   00401006    5B              pop     ebx

   00401007    83EB 06         sub    ebx, 6

   0040100A    81C3 00080000   add    ebx, 800

   00401010    8BFB            mov     edi, ebx

   00401012    8B7424 24       mov    esi, dword ptr [esp+24] //pExceptionRecord

   00401016    B9 06000000     mov    ecx, 6

   0040101B    F3:A5           rep     movs dword ptr es:[edi], dword ptr [esi]

   0040101D    8BFB            mov     edi, ebx

   0040101F    83C7 30         add    edi, 30

   00401022    8B7424 2C       mov    esi, dword ptr [esp+2C] //pContextRecord

   00401026    B9 18000000     mov    ecx, 18

   0040102B    F3:A5           rep     movs dword ptr es:[edi], dword ptr [esi]

   0040102D    8B83 90000000   mov    eax, dword ptr [ebx+90]

   00401033    8D8B A0000000   lea    ecx, dword ptr [ebx+A0]

   00401039    C1E0 02         shl    eax, 2

   0040103C    03C8            add    ecx, eax

   0040103E    8B7424 24       mov    esi, dword ptr [esp+24]

   00401042    8B46 0C         mov    eax, dword ptr [esi+C]

   00401045    8901            mov     dword ptr [ecx], eax

   00401047    FF83 90000000   inc    dword ptr [ebx+90]

   0040104D    8D7B 30         lea    edi, dword ptr [ebx+30]

   00401050    8B83 E8070000   mov    eax, dword ptr [ebx+7E8] //这里考虑可以在hook setthreadcontext中赋值

   015A0056    8947 04         mov    dword ptr [edi+4], eax

   015A0059    8B83 EC070000   mov    eax, dword ptr [ebx+7EC]

   015A005F    8947 08         mov    dword ptr [edi+8], eax

   015A0062    8B83 F0070000   mov    eax, dword ptr [ebx+7F0]

   015A0068    8947 0C         mov    dword ptr [edi+C], eax

   015A006B    8B83 F4070000   mov    eax, dword ptr [ebx+7F4]

   015A0071    8947 10         mov    dword ptr [edi+10], eax

   015A0074    8B83 F8070000   mov    eax, dword ptr [ebx+7F8]

   015A007A    8947 18         mov    dword ptr [edi+18], eax

   015A007D    8B4424 30       mov    eax, dword ptr [esp+30]

   015A0081    8B4C24 28       mov    ecx, dword ptr [esp+28]

   015A0085    8D7B 30         lea    edi, dword ptr [ebx+30]

   015A0088    50              push    eax

   015A0089    57              push    edi

   015A008A    51              push    ecx

   015A008B    53              push    ebx

   015A008C    8B8B FC070000   mov    ecx, dword ptr [ebx+7FC]

   015A0092    33C0            xor     eax, eax

   015A0094    33DB            xor     ebx, ebx

   015A0096    33FF            xor     edi, edi

   015A0098    33F6            xor     esi, esi

   015A009A    FFD1            call    ecx

   015A009C    61              popad

   015A009D    C3              retn
```

以下为代码的16进制值：

```
60 E8 00 00 00 00 5B 83 EB 06 81 C3 00

08 00 00 8B FB 8B 74 24 24 B9 06 00 00 00 F3 A5 8B FB 83

C7 30 8B 74 24 2C B9 18 00 00 00 F3 A5

8B 83 90 00 00 00 8D 8B A0 00 00 00 C1 E0 02 03 C8 8B 74

24 24 8B 46 0C 89 01 FF 83 90 00 00 00

8D 7B 30 8B 83 E8 07 00 00 89 47 04 8B 83 EC 07 00 00 89

47 08 8B 83 F0 07 00 00 89 47 0C 8B 83

F4 07 00 00 89 47 10 8B 83 F8 07 00 00 89 47 18 8B 44 24

30 8B 4C 24 28 8D 7B 30 50 57 51 53 8B

8B FC 07 00 00 33 C0 33 DB 33 FF 33 F6 FF D1 61 C3 00 00
```

## 2. Inline Hook ExecuteHandler2

Hook在这里

`.text:7C9232A6                call   ecx`

改成Jmp to // hook executehandler2

### Patch vm:ds[imm]

这个地址是前面已经找到的，由于暂时还没找到HASH检测的代码位置，所以不能直接改代码，我的方法是，用脚本在这个位置上设置一个硬件执行断点在，当脚本断下后，脚本自动修改代码跳转到我们自己分配的空间内执行一段代码，在代码的开头立即修复之前被脚本改掉的代码，然后产生一个single step异常给系统，再执行之前由于改成跳转没有执行到的程序源码后跳转到原程序下一行继续。

这个方法是确定是要被占用一个硬件断点。

以下是我构造的代码：

//跳转来自0E8A7C8

```
00390100   60              pushad

00390101   BE 00023900     mov     esi, 390200

00390106   BF C8A7E800     mov     edi, 0E8A7C8

0039010B   B9 05000000     mov     ecx, 5

00390110   F3:A4           rep     movs byte ptr es:[edi], byte ptr [esi] //还原ds[imm]被我修改用来跳转的代码

00390112   61              popad

00390113   9C              pushfd                     \
00390114   66:810C24 0001  or      word ptr [esp], 100    3条指令用来产生一个80000004 的EXCEPTION_SINGLE_STEP
0039011A   9D              popfd                      /    就是通过符号位中的单步标记来产生一个异常
0039011B   8B10            mov     edx, dword ptr [eax]  --读取数值，由于硬件执行断点的那行没有执行
0039011D - E9 A8A6AF00     jmp     乔巴.00E8A7CA      --返回执行下一行代码
```

### Patch Shadow_SetThreadContext

为了防止硬件断点被覆盖，这个Shadow函数必须Patch掉，当然，如果前面的部分没有Patch而直接来改这个函数，SE就不会获得需要检测的异常，造成程序的退出。

1、搜索代码

通过之前patch Shadow_GetThreadContext获得的调用KiFastSystemCall前代码区域，查找调用服务号为0D5的调用，

1. 修改代码

   只需要把跳入KiFastSystemCall的位置直接nop掉就行

FFE2           jmp    edx//这里是跳入KiFastSystemCall，直接nop

### Patch掉HASH计算

通过上面几步，现在我可以下硬件断点了，能断下来的第一件事，当然是找HASH效验代码。

1.在前面vm:ds[imm]的位置后面几行下个硬件读取断点

1. 断下之后：

```
.sedata:00E6323F                push    esi

.sedata:00E63240

.sedata:00E63240 loc_E63240:                             ; CODE XREF:

.sedata:00E6324Ej

.sedata:00E63240                movzx   esi, byte ptr [ecx+edx]

.sedata:00E63244                rol     eax, 7 //断在这里

.sedata:00E63247                xor     eax, esi

.sedata:00E63249                inc     ecx

.sedata:00E6324A                cmp     ecx, [esp+8]

.sedata:00E6324E                jb      short loc_E63240

.sedata:00E63250                 pop    esi

.sedata:00E63251                 jmp     short locret_E6325C
```

2.Run Trace后获得整个循环体，直接找到跳出循环的位置来到这里：

```
00E63247 Main    xor    eax, esi                         ; FL=PZ, EAX=F4E889ED, ECX=00093F9F, ESI=000000D000E63250

 Main    pop    esi                               ; ESP=0012FEC8, ESI=F4E889ED00E63251

 Main    jmp    short 00E6325C00E6325C

 Main    retn   4                                ; ESP=0012FED0           //hash值保存在eax
```

3.重新加载几次之后，发现HASH计算的参数和结果是不变的，我只能说，呵呵

检测开始位置：0E7C91D

检测大小：    0093F9F

返回EAX：   F4E889ED

4.偷懒直接patch掉

//------ Fix HASH------

```
00E63240    B8 ED89E8F4     mov    eax, F4E889ED

00E63245    90              nop

00E63246    90              nop

00E63247    90              nop

00E63248    90              nop

00E63249    90              nop

00E6324A    90              nop

00E6324B    90              nop

00E6324C    90              nop

00E6324D    90              nop

00E6324E    90              nop

00E6324F    90              nop
```

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)
