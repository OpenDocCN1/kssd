
3. F9运行，发现断在`0x1F0895`（此时就有代码了）。

   ![image-20191120191815132](https://bbs.kanxue.com/upload/attach/202002/868592_P2XF2FJRS4DBAHE.jpg)

4. 这样就可保证，每次都会断在填充IAT处（不像一开始有时候断成功，有时候失败）。

## 04-壳代码中自定义了API

1. `GetProcAddr`用来获取API地址，若要填充IAT（别管是不是修改后再填充，都会用到此）。

2. `ctrl+g`，搜索，下断，堆栈中，非主模块的调用放行，测试发现，主模块并没有调用此API（一直放行，都放到填充IAT指令那了都没出现），推测：壳代码中自己实现了一个`GetProcAddress`。

3. 同样的思路，`GetModuleHandleA`和`GetModuleHandleW`下断，也没找到（真实OEP之后的调用肯定不是，在到达真实OEP前，已经填充完IAT了，也就已经调用完`GetModuleHandleA`了）。

## 05-找获取API地址的地方

1. 每次运行，都要获取壳代码申请的内存空间的首地址，此时为`1f0000`（有随机性，每次申请的都不一样，故如此（系统有伪随机性，何况是在虚拟中，尽管如此，也要获取，确保万无一失））。

   ![image-20191122192636153](https://bbs.kanxue.com/upload/attach/202002/868592_T2EYQ2EPWA95AJN.jpg)

2. 填充IAT的位置偏移`0895` + 基地址`1f0000` = `1f0895`，下硬件执行，运行到此。

   ![image-20191122193041387](https://bbs.kanxue.com/upload/attach/202002/868592_H7JT6U7FPV2W6AN.jpg)

   ![image-20191122193108680](https://bbs.kanxue.com/upload/attach/202002/868592_WU6TRW4436M5CTP.jpg)

3. F7单步向下执行，发现进入循环，暂不看代码窗口，只关注堆栈窗口，发现是挨个字符的判断（一个函数名中全部字符的循环）。

4. 运行到向上跳的`jmp`时，就是循环语句，在本`jmp`指令与它跳往的地方，二者之间，找`JXX`条件跳转语句，`JXX`一般就是跳出循环的。

   ![image-20191120202646938](https://bbs.kanxue.com/upload/attach/202002/868592_PF4U9BD8HN3Z9GQ.jpg)

5. 在跳出的`xx1ccc`下断，F9测试一下，发现就不是一个函数名中各个字符的循环了，而是一个个函数名的循环，表示确实是跳出了内层的循环。

6. 在4的基础上继续F7单步，继续找负责本层循环的`jxx`条件跳转语句（这一个逻辑正好与4相反，4:跳则出循环，5:不跳才是出循环，但实质相同）。

   ![image-20191120203141485](https://bbs.kanxue.com/upload/attach/202002/868592_MV2CW3VSXF36G54.jpg)

7. 4的循环时一个函数名字符串中各个字符的循环，5的循环是多个函数名的循环，猜测，接下来就是多个dll的循环了（函数名称字符串有多个字符、一个dll有多个函数、一个exe有多个dll）。

8. 在5的基础上，继续F7单步，此时就要慢点了，快要到关键点了，要时刻关注寄存器窗口，肯定会出现`xxx.yyy`此种形式；发现`1911`执行后，`eax`中出现`xxx.yyy`。

   ![image-20191120204525809](https://bbs.kanxue.com/upload/attach/202002/868592_R9YARB9C83GWAJS.jpg)

9. 这就是所谓获取API地址的地方（不管这个值是怎么出来的，我只知道，运行到此时，`eax`寄存器中有我想要的东西：原始的API地址）。

## 06-OD脚本自动化修复IAT

将真实API的地址添加到IAT，而非壳加密后的值。

要运行至壳OEP时再执行脚本，前面那个系统断点用OD的F9跳过。

```
// 基础模版



// 1. 找到三个地址

MOV dwGetAPIAddr,004385BF

MOV dwWriteIATAddr, 004385F0

MOV dwOEP,00409486



// 2. 设置断点（设置之前先清除所有）

BC    // 清除所有软件断点

BPHWC // 清除所有硬件断点

BPMC  // 清除所有内存断点



BPHWS dwGetAPIAddr, "x" //当执行到此地址时产生中断.

BPHWS dwWriteIATAddr, "x" //当执行到此地址时产生中断.

BPHWS dwOEP, "x" //当执行到此地址时产生中断.



// 3. 循环

LOOP_START:

  RUN   // 运行，即F9

  CMP dwGetAPIAddr,eip

  JNZ case1

  MOV dwTmp,eax// 将真实API地址保存至临时变量

  JMP LOOP_START

case1:

  CMP dwWriteIATAddr,eip

  JNZ case2

  MOV [edi],dwTmp// 将真实API地址填充至IAT表

  JMP LOOP_START

case2:

  CMP dwOEP,eip

  JNZ LOOP_START  // 继续循环

  MSG "到达OEP!脚本结束"// 若达到OEP则结束
```

```
// 依据不同的壳，在基础版本上加以修改

// 1. 找到三个地址

MOV dwGetAPIAddr, 001A36        // 获取真实API地址（偏移）

MOV dwWriteIATAddr, 000897    // 填充IAT（偏移）

MOV dwOEP, 0047148B                    // OEP（真实地址）

MOV dwBase,0047A37F                    // 壳申请的内存空间首地址（真实地址）



// 2. 设置断点

BC    // 清除所有软件断点

BPHWC // 清除所有硬件断点

BPMC  // 清除所有内存断点



BPHWS dwOEP, "x"     //OEP设置断点，此地址不会变

BPHWS dwBase, "x" //申请内存空间返回基地址，是后面两个位置的基础



// 3. 循环

LOOP_START:

  RUN       // 继续运行，直至下个断点



  CMP dwBase,eip// 内存空间申请完成，返回基地址

  JNZ case0

  ADD dwGetAPIAddr,eax  // 基址+偏移 = 最终地址

  ADD dwWriteIATAddr,eax

  BPHWS dwGetAPIAddr, "x" // 有了最终地址，再下断点

  BPHWS dwWriteIATAddr, "x"

  JMP LOOP_START // 断点都下完了，继续执行

case0:

  CMP dwGetAPIAddr,eip

  JNZ case1

  MOV dwTmp,eax // 获取真实API地址，赋值给临时变量

  JMP LOOP_START // 继续执行

case1:

  CMP dwWriteIATAddr,eip

  JNZ case2

  MOV [edx],dwTmp// 将临时变量中的真实API地址填充到IAT

  JMP LOOP_START//继续

case2:

  CMP dwOEP,eip  // 若到达OEP，则前面都运行完，结束

  JNZ LOOP_START  // 否则继续执行

  MSG "到达OEP!脚本结束"// 弹窗，标示已结束
```

如图，修复完成（OEP是结束条件，所以正好运行至OEP）。

![image-20191122213958158](https://bbs.kanxue.com/upload/attach/202002/868592_UT6UME5KA8H8CH4.jpg)

## 07-dump内存

- 此时IAT表是正常的，dump下来后，通过此正常的IAT来修复输入表

  ![image-20191122214212076](https://bbs.kanxue.com/upload/attach/202002/868592_947SDPAWXD233MF.jpg)

## 08-修复输入表

![image-20191122215400920](https://bbs.kanxue.com/upload/attach/202002/868592_PNHF9JKU7YY43RS.jpg)

脱壳成功

![image-20191122215429251](https://bbs.kanxue.com/upload/attach/202002/868592_UW65FHC5F6393DH.jpg)

## 09-参考

- 15PB 脱壳课程

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)
