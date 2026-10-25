
# 算法CALL(2)

```asm
0042CD8E  PUSH    EBP
0042CD8F  MOV     EBP, ESP
0042CD91  PUSH    ECX********************************G的ASCII码值47入ECX
0042CD92  MOVZX   EAX, BYTE PTR SS:[EBP+8]****扩展成00000047
0042CD96  MOV     ECX, [ARG.2]********************赋ECX的初始值为0
0042CD99  MOV     [LOCAL.1], EAX
0042CD9C  MOV     EAX, ECX************************ECX的值移入EAX
0042CD9E  IMUL    EAX, ECX************************EAX=EAX*ECX=0*0
0042CDA1  LEA     EAX, DWORD PTR DS:[EAX+EAX*2+7]****EAX+EAX*2+7的值7给EAX
0042CDA5  IMUL    EAX, ECX************************EAX=EAX*ECX=7*0
0042CDA8  ADD     EAX, 0D**************************EAX=EAX+0D=D
0042CDAB  IMUL    EAX, ECX************************EAX=EAX*ECX=D*0=0
0042CDAE  PUSH    ESI******************************序列号位数F入栈!
0042CDAF  LEA     ESI, DWORD PTR DS:[ECX+5]
0042CDB2  PUSH    ESI
0042CDB3  MOV     [ARG.1], EAX
0042CDB6  LEA     EAX, [ARG.1]
0042CDB9  PUSH    4
0042CDBB  PUSH    EAX
0042CDBC  CALL    SuperPIM.0042CD2E****************这个CALL也在计算,我实在不想跟进!
0042CDC1  PUSH    ESI
0042CDC2  LEA     EAX, [LOCAL.1]
0042CDC5  PUSH    4
0042CDC7  PUSH    EAX
0042CDC8  CALL    SuperPIM.0042CD5E****************这个CALL也是计算CALL,烦!不跟了!
0042CDCD  MOV     EAX, [ARG.1]
0042CDD0  ADD     ESP, 18
0042CDD3  XOR     EAX, [LOCAL.1]********************这里赋EAX的值38000002
0042CDD6  POP     ESI
0042CDD7  LEAVE
0042CDD8  RETN
```

算法CALL(2)结束

```asm
0042CF34  POP     ECX
0042CF35  POP     ECX
0042CF36  PUSH    2B************************************常数2B入栈!
0042CF38  XOR     EDX, EDX
0042CF3A  POP     ECX************************************把常数2B赋给ECX
0042CF3B  DIV     ECX************************************除,EAX/ECX=38000002/2B=014D6535,余数1B入EDX
0042CF3D  MOV     ECX, EDX********************************结果1B入ECX
0042CF3F  ADD     CL, 30**********************************CL=CL+30=1B+30=4B(对应的字符串是K)
0042CF42  CMP     CL, 39**********************************比较是不是数字9
0042CF45  MOV     BYTE PTR SS:[EBP-14], CL****************保存字符串K
0042CF48  JLE     SHORT SuperPIM.0042CF55****************小于就跳走!
0042CF4A  CMP     CL, 41**********************************比较是不是A
0042CF4D  JGE     SHORT SuperPIM.0042CF55****************大于就跳走
0042CF4F  ADD     CL, 0F6
0042CF52  MOV     BYTE PTR SS:[EBP-14], CL
0042CF55  CMP     EBX, [ARG.3]****************************跳到这里!比较0和14
0042CF58  JGE     SHORT SuperPIM.0042CF67****************大于等于就跳走!
0042CF5A  PUSH    [LOCAL.5]
0042CF5D  LEA     ECX, [LOCAL.4]
0042CF60  CALL    SuperPIM.00417EAF
0042CF65  JMP     SHORT SuperPIM.0042CFA5****************无条件跳
0042CF67  MOV     EAX, EBX
0042CF69  CDQ
0042CF6A  IDIV    [ARG.3]
0042CF6D  MOVSX   ECX, CL
0042CF70  PUSH    2B
0042CF72  MOV     ESI, EDX
0042CF74  MOVSX   EAX, BYTE PTR DS:[ESI+EDI]
0042CF78  LEA     EAX, DWORD PTR DS:[EAX+ECU-60]
0042CF7C  CDQ
0042CF7D  POP     ECX
0042CF7E  IDIV    ECX
0042CF80  ADD     DL, 30
0042CF83  CMP     DL, 39
0042CF86  MOV     BYTE PTR SS:[EBP-14], DL
0042CF89  JLE     SHORT SuperPIM.0042CF96
0042CF8B  CMP     DL, 41
0042CF8E  JGE     SHORT SuperPIM.0042CF96
0042CF90  ADD     DL, 0F6
0042CF93  MOV     BYTE PTR SS:[EBP-14], DL
0042CF96  PUSH    [LOCAL.5]
0042CF99  LEA     ECX, [LOCAL.4]
0042CF9C  PUSH    ESI
0042CF9D  CALL    SuperPIM.0042CE4B
0042CFA2  MOV     ESI, [LOCAL.8]
0042CFA5  MOV     EDI, [LOCAL.4]****************************上面跳到这里!
0042CFA8  INC     EBX****************************************EBX加1
0042CFA9  CMP     EBX, [LOCAL.6]****************************比较1和14
0042CFAC  JL      SuperPIM.0042CF1B************************小于就跳,循环,序列号只有15位,这里循环20次,取完后又从第一位取出!
********************************************************************循环结束后得到的值是:K3L2LPBDW1F2H4B8S0UY这就是我们要的注册码!
0042CFB2  MOV     ECX, [ARG.1]
0042CFB5  LEA     EAX, [LOCAL.4]
0042CFB8  PUSH    EAX
0042CFB9  CALL    SuperPIM.00401F36
0042CFBE  LEA     ECX, DWORD PTR DS:[EDI-10]
0042CFC1  CALL    SuperPIM.00401B5D
0042CFC6  MOV     ECX, [LOCAL.3]
0042CFC9  MOV     EAX, [ARG.1]
0042CFCC  POP     EDI
0042CFCD  POP     ESI
0042CFCE  POP     EBX
0042CFCF  MOV     DWORD PTR FS:[0], ECX
0042CFD6  LEAVE
0042CFD7  RETN
```

写这篇文章的目的是想说明如何在OD下万能断点，算法过程太烦琐，就不想多跟了！希望能对大家有点启发，如果觉得这篇破文还行帮我顶一下吧！也是希望有更多的人能看到，对更多的人有益，我也是一只菜鸟，请大家多多指点！

我的序列号：`GK342QZ0C6RE03L`
注册码：`K3L2LPBDW1F2H4B8S0UY`
