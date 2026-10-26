
# IAT加密过程分析

这一段就是IAT加密过程了，重点分析一下。

## 流程分析

见代码中的注释。

## 使用的数据组分析

### API字符串基本数据组

```
EBX=012B04D0

012B04D0  B8 2D 26 01 9C 8C 2B 01 87 1F 11 BD 3B AE 95 94

012B04E0  4C BA 7B 5E 8C 79 70 8D 9A E1 A9 25 D9 B1 AF BF

012B04F0  37 1E B6 59 2D FB 00 00 B9 BE A4 40 2A 00 00 00

012B0500  30 05 2B 01 F9 85 BA A1 BD 30 96 F7 C7 00 00 00

   EBX +              34 35 36 37 38 39 3A 3B

012B0510  77 1D 80 7C FC E2 D5 F7 A0 AD 80 7C 2B 52 D5 F7

012B0520  00 00 40 00
```

含义：

+0:  `dword`     not used

+4:  `dword`     导入表数据组

+8:  `byte*A*3`  字符串加解密时所用的XOR种子，第3组似乎没用到

+26: `byte*2`    not used

+28: `dword`     从2E34到3F10这一段数据和代码的HASH

+2C: `dword`     上一次所用DLL名的HASH

+30: `dword`     not used

+34: `byte*9`    字符串加解密方式的标志位

+3D: `byte*3`    not used

+40: `dword`     `LoadLibraryA`

+44: `dword`     `LoadLibraryA`函数的HASH

+48: `dowrd`     `GetProcAddress`

+4C: `dword`     `GetProcAddress`函数的HASH

+50: `dword`     程序基址

### 导入表字符串数据组分析

+0:  `byte*9`   字符串解密标志位

+9:  `dowrd`    从DLL获取API地址后存放位置的起始偏移地址

{
  +0:  `dowrd`    当前DLL字符串数据的总长度
  +4:  `word`     当前DLL的HASH
  +6:  `byte*len(DLL name)`,\0    DLL名称，明文
  {
    +0:  `byte`     API名称的加密方式标志位
    +1:  `word`     API名称HASH
    +3:  `word`     API名称长度
    +5:  `byte*len(API name)`    加密后的API名称字符串
  }*n
}*n

基本数据组的标志位从导入表数据组前9字节复制而来，含义为：

`34`：循环结束

`35`：不保存IAT，使用`0125759C`替换

`36`：加密方式二：使用+8开始的XOR种子解密，直接保存

`37`：不需解密直接保存字符串数据，API硬编码

`38`,`39`：加密方式三：替换为壳调用，并校验代码HASH。只能对应一个API

`3A`：加密方式二：使用+8开始的XOR种子解密，替换成壳里调用

`3B`：加密方式一：使用+12开始的XOR种子解密，直接保存

`3C`：not used

## IAT修复流程归纳

修复IAT需要更改保存API地址的`call 01263B84`中的代码流程，先归纳一下相关数据：

1. 使用到了`key=[ebx+35]`,`[ebx+36]`,`[ebx+37]`,`[ebx+38]`,`[ebx+39]`,`[ebx+3A]`,`[ebx+3B]`，仅对这几个标志位的IAT进行计算。
2. `key=[ebx+3B]`时使用+12开始的XOR种子解密。
3. `key=[ebx+37]`时没有API名称，直接用字符串数据填入到IAT中，相当于API硬编码。
4. `key=[ebx+38]`时使用`01262E2C`替换，在数据窗口中分析`01262E2C`处的代码可以得出实际调用的是`[012679D0]=kernel32.RaiseException`。
5. `key=[ebx+39]`时使用`01257574`替换，在数据窗口中分析`01257574`处的代码可以得出实际调用的是`[01269274]=kernel32.GetProcAddress`。
6. `key=[ebx+35]`,`[ebx+36]`,`[ebx+3A]`时都可以解密出API地址，可以直接填入IAT中。

所以需要调整代码流程为：

1. `key=[ebx+3B]`流程不变。
2. `key=[ebx+35]`,`[ebx+36]`,`[ebx+3A]`全部指向`[ebx+36]`流程。
3. `key=[ebx+38]`修改保存地址为`[012679D0]`，并跳过代码HASH自校验过程。
4. `key=[ebx+39]`修改保存地址为`[01269274]`，并跳过代码HASH自校验过程。

### 需要修改的代码

```
01263C27    74 34              je      short 01263C5D               ; ******je 01263C5D

01263C30    74 2B              je      short 01263C5D               ; ******je 01263C5D

01263C39    74 65              je      short 01263CA0               ; ******je 01263CA0

01263C48    EB 13              jmp     short 01263C5D               ; ******jmp 01263C5D

01263C9E    EB 45              jmp     short 01263CE5               ; ******jmp 01263CE5

01263CA5    EB 3E              jmp     short 01263CE5               ; ******jmp 01263CE5

01263D0B    A1 D0792601        mov     eax, dword ptr [12679D0]     ; ******mov eax,[12679D0]

01263D17    EB 7B              jmp     short 01263D94               ; ******jmp 01263D94

01263D55    A1 74922601        mov     eax, dword ptr [1269274]     ; ******mov eax, [1269274]

01263D61    EB 31              jmp     short 01263D94               ; ******jmp 01263D94
```

当外层循环执行完毕后需要把修改过的代码还原，因为在后面其他地方还有自校验。

再次到达`OEP`后用`ImportREC`打开程序进程，填入

`OEP`: `289BE0`

`RVA`: `29CB98`

`SIZE`: `9B8`

搜索导入表，一个无效链接都没有了，心里感觉那叫一个爽快！

敬请期待下一篇：修复跳转表

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)
