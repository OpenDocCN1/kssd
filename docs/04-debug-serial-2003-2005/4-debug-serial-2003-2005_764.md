
# netspy 3.0

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-144.htm`](https://www.pediy.com/kssd/pediy07/pediy7-144.htm)

【软件名称】：`netmonitor.exe`  软件版本：3.0

【软件大小】：468 KB

【下载地址】：http://www.siriusoft.net/

【软件简介】：著名的木马软件 `netspy`（网络精灵）的客户端。`netspy` 是继“冰河”之后又一优秀国产木马，可用 IE 控制，可惜生不逢时，所有光芒全被当时无人不知的 `CIH` 病毒掩盖。

【软件限制】：未注册时只能在列表中显示两台服务端机器。

【破解声明】：给刚入门的人自信心，学会后就是 `Hacker` + `Cracker`

【破解工具】：`W32dasm8.93`、`OD1.10`、`regmon`

———————————————————————————————————————————

【破解过程】：

程序没有加壳，（2002 年 3 月 11 日发行，好像壳还没怎么流行吧），也没有任何多余的语句和花指令，`VC` 编写。

该程序有预注册码（即硬件识别码），输入错误的注册码后没有任何提示，故打开 `regmon`，监视程序启动，发现读注册表项 `Software\Sirius\NetMonitor\KEY`，用 `W32dasm` 静态反汇编，查找串式参考“`KEY`”，有两处相关，其一为 `RegQueryValueExA`，当然是启动时读注册信息了，另一处为写注册表，自然可想到是输入注册码了。

基本上直接静态分析就能搞定，如下：

```
* Possible StringData Ref from Data Obj ->"Software\Sirius\NetMonitor"
                     |
:0041D481 68D0B64300           push 0043B6D0
:0041D486 6802000080           push 80000002
* Reference To: ADVAPI32.RegCreateKeyExA, Ord:015Fh       <--读注册表
                     |
:0041D48B FF1510204300        Call dword ptr [00432010]
:0041D491 85C0                test eax, eax
:0041D493 0F85DF000000        jne 0041D578                <-----没找到主键时跳转
:0041D499 C745FC03000000      mov [ebp-04], 00000003
:0041D4A0 C745E406000000      mov [ebp-1C], 00000006
:0041D4A7 8D55E4              lea edx, dword ptr [ebp-1C]
:0041D4AA 52                 push edx                     <-----6 个字节空间
:0041D4AB 8D45F0              lea eax, dword ptr [ebp-10]
:0041D4AE 50                 push eax                     <-----读出数据的地址
:0041D4AF 8D4DFC              lea ecx, dword ptr [ebp-04]
:0041D4B2 51                 push ecx                     <-----数据类型，二进制值
:0041D4B3 6A00                push 00000000
* Possible StringData Ref from Data Obj ->"KEY"             〈-------键名
                     |
:0041D4B5 68ECB64300         push 0043B6EC
:0041D4BA 8B55E8             mov edx, dword ptr [ebp-18]
:0041D4BD 52                push edx
* Reference To: ADVAPI32.RegQueryValueExA, Ord:017Bh     <-------读注册表键值，参数在上面
                     |
:0041D4BE FF1514204300       Call dword ptr [00432014]
:0041D4C4 85C0               test eax, eax
:0041D4C6 0F8599000000       jne 0041D565          <-------相应键不存在时跳转
注：假设注册表中值为 X0、X1、X2、X3、X4、X5、X6、X7，

:0041D4CC 6A01               push 00000001
:0041D4CE 8D45F0             lea eax, dword ptr [ebp-10]   
:0041D4D1 50                push eax
:0041D4D2 8D4DCC             lea ecx, dword ptr [ebp-34]
:0041D4D5 51                push ecx
* Reference To: MSVCRT.memcpy, Ord:0297h
                     |
:0041D4D6 E821270100         Call 0042FBFC        <-----内存拷贝 1 字节，[ebp-10]拷到[ebp-34]
                                                     <---对应注册表数据第 1 位，即 X0
:0041D4DB 83C40C             add esp, 0000000C
:0041D4DE 6A01               push 00000001
:0041D4E0 8D55F1             lea edx, dword ptr [ebp-0F]   
:0041D4E3 52                push edx
:0041D4E4 8D45D8             lea eax, dword ptr [ebp-28]
:0041D4E7 50                push eax
* Reference To: MSVCRT.memcpy, Ord:0297h
                     |
:0041D4E8 E80F270100         Call 0042FBFC       <-----内存拷贝 1 字节，[ebp-0F]拷到[ebp-28]
                                                  <---对应注册表数据第 2 位，即 X1
:0041D4ED 83C40C             add esp, 0000000C
:0041D4F0 6A04               push 00000004
:0041D4F2 8D4DF2             lea ecx, dword ptr [ebp-0E]
:0041D4F5 51                push ecx
:0041D4F6 8D55D0             lea edx, dword ptr [ebp-30]
:0041D4F9 52                push edx
* Reference To: MSVCRT.memcpy, Ord:0297h
                     |
:0041D4FA E8FD260100         Call 0042FBFC      <-----内存拷贝 4 字节，[ebp-0E]拷到[ebp-30]
                                                  <---对应注册表数据第 3-6 位，X2X3X4X5
:0041D4FF 83C40C             add esp, 0000000C
:0041D502 8D45D0             lea eax, dword ptr [ebp-30]
:0041D505 8945DC             mov dword ptr [ebp-24], eax     <-----4 字节的存放位置指针
:0041D508 C745D400000000     mov [ebp-2C], 00000000          <------循环计数器
:0041D50F EB09               jmp 0041D51A

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0041D533(U)
|
:0041D511 8B4DD4             mov ecx, dword ptr [ebp-2C]
:0041D514 83C101             add ecx, 00000001
:0041D517 894DD4             mov dword ptr [ebp-2C], ecx

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0041D50F(U)
|
:0041D51A 837DD404           cmp dword ptr [ebp-2C], 00000004   <-------共循环 4 次，
                                                      对应上面的 4 个字节
:0041D51E 7D15               jge 0041D535                <-----4 字节处理完后跳转
:0041D520 8B55DC             mov edx, dword ptr [ebp-24]
:0041D523 0355D4             add edx, dword ptr [ebp-2C]   <----从低位开始
:0041D526 8A02               mov al, byte ptr [edx]
:0041D528 3245D8             xor al, byte ptr [ebp-28]     <---逐位与[ebp-28]异或
:0041D52B 8B4DDC             mov ecx, dword ptr [ebp-24]
:0041D52E 034DD4             add ecx, dword ptr [ebp-2C]   <----运算结果放在原处
:0041D531 8801               mov byte ptr [ecx], al
:0041D533 EBDC               jmp 0041D511

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0041D51E(C)
|
:0041D535 8A55D8             mov dl, byte ptr [ebp-28]
:0041D538 3255CC             xor dl, byte ptr [ebp-34]     <---[ebp-28]与[ebp-34]异或
:0041D53B 8855D8             mov byte ptr [ebp-28], dl
:0041D53E 8A45D8             mov al, byte ptr [ebp-28]
:0041D541 344C               xor al, 4C                    <---结果再与 4C 异或
:0041D543 8845D8             mov byte ptr [ebp-28], al
:0041D546 8B4DD0             mov ecx, dword ptr [ebp-30]
:0041D549 3B4DE0             cmp ecx, dword ptr [ebp-20]   <---比较[ebp-20]与刚才的 4 位运算
:0041D54C 7409               je 0041D557                  <---相等则跳转
:0041D54E C745F802000000    mov [ebp-08], 00000002        <---否则设置为 2
:0041D555 EB0C               jmp 0041D563

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0041D54C(C)
|
:0041D557 8B55D8             mov edx, dword ptr [ebp-28]
:0041D55A 81E2FF000000       and edx, 000000FF             <---得到允许控制的机器数目
:0041D560 8955F8             mov dword ptr [ebp-08], edx

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0041D555(U)
|
:0041D563 EB07               jmp 0041D56C

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0041D4C6(C)
|
:0041D565 C745F802000000     mov [ebp-08], 00000002                <-----赋值为 2

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0041D563(U)
|
:0041D56C 8B45E8             mov eax, dword ptr [ebp-18]
:0041D56F 50                push eax
* Reference To: ADVAPI32.RegCloseKey, Ord:015Bh
                     |
:0041D570 FF1518204300       Call dword ptr [00432018]
:0041D576 EB07               jmp 0041D57F

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0041D493(C)
|
:0041D578 C745F802000000     mov [ebp-08], 00000002

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
```



# 逆向分析与注册机逻辑

## 汇编代码片段分析

```
|:0041D576(U)  
|  
:0041D57F 8B45F8                 mov eax, dword ptr [ebp-08]        <-----[ebp-08]就是限制数  
* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:0041D467(U)  
|  
:0041D582 8BE5                  mov esp, ebp  
:0041D584 5D                    pop ebp  
:0041D585 C3                    ret  
```

计算过程非常简单，只有一处还不明确，

```
:0041D549 3B4DE0    cmp ecx, dword ptr [ebp-20]  
```

`[ebp-20]` 是什么呢？只有动态跟踪了，看到是一个十六进制数值，再想到注册时要用的预注册码，换算成十进制后果然相等。至此已不难写出注册机程序。

## 注册机算法推导

假设注册表中注册值为 `X0`、`X1`、`X2`、`X3`、`X4`、`X5`

预注册值换算成十六进制为 `Y1Y2Y3Y4`

想要控制的机器为 `Z` 台，则有以下关系式：

```
(X0 xor X1) xor 0x4C=Z

Y1 xor X1 = X5

Y2 xor X1 = X4

Y3 xor X1 = X3

Y4 xor X1 = X2
```

`X1` 随便取个值，就可以确定其他各参数。

编程将计算的结果写入注册表即可，不需经过注册对话框。

如果你有耐心的话，可以进一步推算从注册码输入框输入的内容转换成注册表中十六进制值的过程，写出另一个注册机，跟作者手上的一样：-）

## 爆破与汇编注入

偶是懒人，连写上面的注册机都嫌烦，所以最喜欢爆破，

```
:0041D57F 8B45F8   mov eax, dword ptr [ebp-08]    <----这里是个爆破点，赋值即可
                                             可惜只有 3 字节，太少
```

再往上看看函数开始，

```
* Referenced by a CALL at Addresses:
|:0040D28A   , :0040F3CC   , :0041D982   , :0041DA65        <-----调用的地方还挺多的
|
:0041D434 55                       push ebp     <---从这里开始，填入汇编机器码
                                             33C0         xor eax,eax
                                             35FF000000   xor eax,0ff   搞个 255 吧
                                             C3           ret

:0041D435 8BEC                     mov ebp, esp  
:0041D437 83EC34                   sub esp, 00000034  
:0041D43A C745E000000000           mov [ebp-20], 00000000  
:0041D441 E8EA33FFFF               call 00410830  
:0041D446 8945E0                   mov dword ptr [ebp-20], eax  
:0041D449 837DE000                 cmp dword ptr [ebp-20], 00000000  
:0041D44D 7508                     jne 0041D457  
:0041D44F E885020000               call 0041D6D9  
:0041D454 8945E0                   mov dword ptr [ebp-20], eax  
```

## 注册表写入分析

附后：一不小心被加了精，只好把剩下的部分写完：

前面已提到，用 `W32dasm` 静态反汇编，查找串式参考“`KEY`”，有两处相关，另一处为写注册表，见下

```
* Possible StringData Ref from Data Obj ->"KEY"
                         |
:0041526A 687CB24300               push 0043B27C  
:0041526F 8B4DEC                   mov ecx, dword ptr [ebp-14]  
:00415272 51                       push ecx  

* Reference To: ADVAPI32.RegSetValueExA, Ord:0186h
                         |  
:00415273 FF150C204300             Call dword ptr [0043200C]  
```

由此向上找到函数开始，

```
:004151D5 55                       push ebp  
:004151D6 8BEC                     mov ebp, esp  
:004151D8 83EC1C                   sub esp, 0000001C  
```

## 注册码转换与总结

参照前面的方法静态分析或动态跟踪都可以，发现

注册码第一个框实际就是 `X0` 的十进制数值，相应地，第二个框为 `X1`，第三个框为 `X2X3X4X5`，

### 总结

`X1` 随便取值，如 `FF`，则第二个框填 `255`

我的机子预注册码为 `1205016028`，换算为十六进制为 `47D315DC`，与 `FFFFFFFF` 异或得到 `B82CEA23`，换成十进制 `3089951267`，填入第三个框

如果你要注册控制 `3` 台机器，则

`X0 = X1 xor 0x4C xor 0x3 = 0xB0 = 176` 填入第一个框

Ｏ．Ｋ．注册后重运行，再点注册，提示已注册成功，不要重新输注册码，添加机算机试试，当添加第 `4` 台时，提示“用户数目超出”。

当然，如果你更聪明一点的话可以看到

当 `X1` 取 `0` 时，即第二个框填 `0`，则第三个框把预注册码抄上去就可以了（和 `0` 异或嘛），第一个框 `X0` 随便填都无所谓，注册后机器数限制＝`X0 xor 0x4C`。

全文完

`4dd466d0e1e0cc123afbe504da2462ca`
