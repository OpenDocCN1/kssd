
# [原创]鼓起勇气－我的第一篇破文，特级菜．异或运算！

- 软件名称：`netmonitor.exe`，软件版本：3.0
- 软件大小：468 KB
- 下载地址：a6aK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4y4A6M7X3W2#2M7$3!0X3N6q4)9J5k6h3&6W2N6q4)9J5c8R3`.`.
- 软件简介：著名的木马软件netspy（网络精灵）的客户端。netspy是继“冰河”之后又一优秀国产木马，可用IE控制，可惜生不逢时，所有光芒全被当时无人不知的CIH病毒掩盖。
- 软件限制：未注册时只能在列表中显示两台服务端机器。
- 破解声明：给刚入门的人自信心，学会后就是Hacker+Cracker
- 破解工具：`W32dasm8.93`、`OD1.10`、`regmon`

---

## 破解过程

程序没有加壳，（2002年3月11日发行，好像壳还没怎么流行吧），也没有任何多余的语句和花指令，VC编写。

该程序有预注册码（即硬件识别码），输入错误的注册码后没有任何提示，故打开`regmon`，监视程序启动，发现读注册表项`Software\Sirius\NetMonitor\KEY`，用`W32dasm`静态反汇编，查找串式参考`KEY`，有两处相关，其一为`RegQueryValueExA`，当然是启动时读注册信息了，另一处为写注册表，自然可想到是输入注册码了。

基本上直接静态分析就能搞定，如下：

```asm
* Possible StringData Ref from Data Obj ->"Software\Sirius\NetMonitor"

                                  |

:0041D481 68D0B64300              push 0043B6D0

:0041D486 6802000080              push 80000002

* Reference To: ADVAPI32.RegCreateKeyExA, Ord:015Fh               <----读注册表

                                  |

:0041D48B FF1510204300            Call dword ptr [00432010]

:0041D491 85C0                    test eax, eax

:0041D493 0F85DF000000            jne 0041D578                    <-----没找到主键时跳转

:0041D499 C745FC03000000          mov [ebp-04], 00000003

:0041D4A0 C745E406000000          mov [ebp-1C], 00000006

:0041D4A7 8D55E4                  lea edx, dword ptr [ebp-1C]

:0041D4AA 52                      push edx                        <-----6个字节空间

:0041D4AB 8D45F0                  lea eax, dword ptr [ebp-10]

:0041D4AE 50                      push eax                        <-----读出数据的地址

:0041D4AF 8D4DFC                  lea ecx, dword ptr [ebp-04]

:0041D4B2 51                      push ecx                        <-----数据类型，二进制值

:0041D4B3 6A00                    push 00000000

* Possible StringData Ref from Data Obj ->"KEY"                  〈-------键名

                                  |

:0041D4B5 68ECB64300              push 0043B6EC

:0041D4BA 8B55E8                  mov edx, dword ptr [ebp-18]

:0041D4BD 52                      push edx

* Reference To: ADVAPI32.RegQueryValueExA, Ord:017Bh        <-------读注册表键值，参数在上面

                                  |

:0041D4BE FF1514204300            Call dword ptr [00432014]

:0041D4C4 85C0                    test eax, eax

:0041D4C6 0F8599000000            jne 0041D565              <-------相应键不存在时跳转

注：假设注册表中值为X0、X1、X2、X3、X4、X5、X6、X7，

:0041D4CC 6A01                    push 00000001

:0041D4CE 8D45F0                  lea eax, dword ptr [ebp-10]

:0041D4D1 50                      push eax

:0041D4D2 8D4DCC                  lea ecx, dword ptr [ebp-34]

:0041D4D5 51                      push ecx

* Reference To: MSVCRT.memcpy, Ord:0297h

                                  |

:0041D4D6 E821270100              Call 0042FBFC       <-----内存拷贝1字节，[ebp-10]拷到[ebp-34]

                                                      <---对应注册表数据第1位，即X0

:0041D4DB 83C40C                  add esp, 0000000C

:0041D4DE 6A01                    push 00000001

:0041D4E0 8D55F1                  lea edx, dword ptr [ebp-0F]

:0041D4E3 52                      push edx

:0041D4E4 8D45D8                  lea eax, dword ptr [ebp-28]

:0041D4E7 50                      push eax

* Reference To: MSVCRT.memcpy, Ord:0297h

                                  |

:0041D4E8 E80F270100              Call 0042FBFC      <-----内存拷贝1字节，[ebp-0F]拷到[ebp-28]

                                                     <---对应注册表数据第2位，即X1

:0041D4ED 83C40C                  add esp, 0000000C

:0041D4F0 6A04                    push 00000004

:0041D4F2 8D4DF2                  lea ecx, dword ptr [ebp-0E]

:0041D4F5 51                      push ecx

:0041D4F6 8D55D0                  lea edx, dword ptr [ebp-30]

:0041D4F9 52                      push edx

* Reference To: MSVCRT.memcpy, Ord:0297h

                                  |

:0041D4FA E8FD260100              Call 0042FBFC     <-----内存拷贝4字节，[ebp-0E]拷到[ebp-30]

                                                    <---对应注册表数据第3-6位，X2X3X4X5

:0041D4FF 83C40C                  add esp, 0000000C

:0041D502 8D45D0                  lea eax, dword ptr [ebp-30]

:0041D505 8945DC                  mov dword ptr [ebp-24], eax     <-----4字节的存放位置指针

:0041D508 C745D400000000          mov [ebp-2C], 00000000          <------循环计数器

:0041D50F EB09                    jmp 0041D51A

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:0041D533(U)

|

:0041D511 8B4DD4                  mov ecx, dword ptr [ebp-2C]

:0041D514 83C101                  add ecx, 00000001

:0041D517 894DD4                  mov dword ptr [ebp-2C], ecx

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:0041D50F(U)

|

:0041D51A 837DD404                cmp dword ptr [ebp-2C], 00000004   <-------共循环4次，

                                                                      对应上面的4个字节

:0041D51E 7D15                    jge 0041D535                  <-----4字节处理完后跳转

:0041D520 8B55DC                  mov edx, dword ptr [ebp-24]

:0041D523 0355D4                  add edx, dword ptr [ebp-2C]   <----从低位开始

:0041D526 8A02                    mov al, byte ptr [edx]

:0041D528 3245D8                  xor al, byte ptr [ebp-28]     <---逐位与[ebp-28]异或

:0041D52B 8B4DDC                  mov ecx, dword ptr [ebp-24]

:0041D52E 034DD4                  add ecx, dword ptr [ebp-2C]   <----运算结果放在原处

:0041D531 8801                    mov byte ptr [ecx], al

:0041D533 EBDC                    jmp 0041D511

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:0041D51E(C)

|

:0041D535 8A55D8                  mov dl, byte ptr [ebp-28]

:0041D538 3255CC                  xor dl, byte ptr [ebp-34]     <---[ebp-28]与[ebp-34]异或

:0041D53B 8855D8                  mov byte ptr [ebp-28], dl

:0041D53E 8A45D8                  mov al, byte ptr [ebp-28]

:0041D541 344C                    xor al, 4C                    <---结果再与4C异或

:0041D543 8845D8                  mov byte ptr [ebp-28], al

:0041D546 8B4DD0                  mov ecx, dword ptr [ebp-30]

:0041D549 3B4DE0                  cmp ecx, dword ptr [ebp-20]   <---比较[ebp-20]与刚才的4位运算

:0041D54C 7409                    je 0041D557                   <---相等则跳转

:0041D54E C745F802000000          mov [ebp-08], 00000002        <---否则设置为2

:0041D555 EB0C                    jmp 0041D563

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:0041D54C(C)

|

:0041D557 8B55D8                  mov edx, dword ptr [ebp-28]

:0041D55A 81E2FF000000            and edx, 000000FF             <---得到允许控制的机器数目

:0041D560 8955F8                  mov dword ptr [ebp-08], edx

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:0041D555(U)

|

:0041D563 EB07                    jmp 0041D56C

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:0041D4C6(C)

|

:0041D565 C745F802000000          mov [ebp-08], 00000002                  <-----赋值为2

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:0041D563(U)

|

:0041D56C 8B45E8                  mov eax, dword ptr [ebp-18]

:0041D56F 50                      push eax

* Reference To: ADVAPI32.RegCloseKey, Ord:015Bh

                                  |

:0041D570 FF1518204300            Call dword ptr [00432018]

:0041D576 EB07                    jmp 0041D57F

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:0041D493(C)

|

:0041D578 C745F802000000          mov [ebp-08], 00000002

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:0041D576(U)

|

:0041D57F 8B45F8                  mov eax, dword ptr [ebp-08]        <-----[ebp-08]就是限制数

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:0041D467(U)

|

:0041D582 8BE5                    mov esp, ebp

:0041D584 5D                      pop ebp

:0041D585 C3                      ret
```

计算过程非常简单，只有一处还不明确，

```
:0041D549 3B4DE0    cmp ecx, dword ptr [ebp-20]
```
